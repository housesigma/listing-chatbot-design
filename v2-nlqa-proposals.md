# Listing Chatbot v2 NLQ&A — Proposals

> **范围**: 需跨仓字段产出 / 透传 / prompt 改写才能 ship 的提案集合。配套文档 [`v2-nlqa-surface-changes.md`](./v2-nlqa-surface-changes.md) 覆盖可独立落地的 surface 层。
>
> **上游权威源**(同目录):
> - [`v2-nlqa.md`](v2-nlqa.md) — 设计意图、UI patterns、System Prompt Direction、Open Questions、Risk
> - [`chat-design-system.md`](chat-design-system.md) — 设计原则、12-state 状态机、组件视觉规格(本文每个提案的视觉规格 defer 到此,不重写)
>
> **版本轴**: 本文 v1 / v2 = 设计 positioning 版本,不是代码版本号。worker 当前 v5.1.0,IC/STP/RR 已支持 comparison + market reasoning,真正工程 delta 见 §12.1。
>
> **设计基线**: 继承 chat-design-system.md §2 的 6 条原则;每提案独立 feature flag 门控(§11),关旗字节级一致;视觉壳子保持 Variant C。
>
> **代码基线**(2026-05-14):
>
> | 仓库 | 分支 | HEAD |
> |---|---|---|
> | `web-hybrid` | `master` | `ca95f7ba3` |
> | `listing-chatbot` | `main` | `e2426ac` |
> | `realagent-services` | `feature/PLAN-5931-mcp-client-test` | `bf57453` |
> | `prototypes/listing-chatbot-design` | `main` | `83f5e65` |

---

## 1. 索引

| § | 提案 | 服务端门槛 | 推荐优先级 |
|---|---|---|---|
| 2 | Abstain 模式 UI 启用 | chat-service `writeDoneEvent` 加 2 字段(~3 行) | **P1-A**(最低门槛,先做) |
| 3 | Stage-aware 加载文案 | worker per-stage emit user-friendly `ProgressEvent.message` | **P1-A**(前端 0 改动) |
| 4 | Abstain 文案 redirect 语气 | worker system prompt 改写 | **P1-A**(纯 prompt,与 §2 协同) |
| 5 | Source / Grounding Label | worker 新增 `grounding_type` 字段 + chat-service 透传 | **P1-B**(中等,字段绿地) |
| 6 | Follow-up Suggestion Chips | worker 新增 `follow_ups` 字段 + prompt 产出 + chat-service 透传 | **P1-B**(较大,字段+prompt 双绿地) |
| 7 | Idle Re-engagement Prompt | 依赖 §6 数据源 | **P2** |
| 8 | Confidence Indicator | 依赖 §5 + 用户研究门控 | **deferred**(v2 不做) |

---

## 2. 提案 A · Abstain 模式 UI 启用

### 2.1 设计意图

v2 必须在 UI 上**显式区分两种截然不同的失败**,这是 v1 → v2 心智模型升级里最容易被忽略但语义最关键的差异:

| 类型 | 含义 | 用户应当感受到 |
|---|---|---|
| **Abstain** | 助手 *主动选择不答* — 知识范围外、不愿瞎猜 | 这是助手的判断,不是系统坏了 |
| **Error** | 系统 *没能给出答* — 超时、安全拦截、推理报错 | 这是技术故障,可以重试 |

今天 UI 完全没区分,二者都落到普通 bubble。

### 2.2 视觉规格

视觉细节见 `chat-design-system.md §5.9`(Abstain Pattern)+ `v2-nlqa.md §10.4`。本文只标实施关键约束:

- **Marker 在 bubble 内顶部**,文字 "Outside my listing knowledge",`--muted-foreground` 灰
- **Bubble 背景不换色**(仍 `--muted` 灰),区别于 error 的 destructive 文字
- **`nullReason`** 可选,bubble 下方 italic 灰字,仅当非空且 human-readable 时渲染
- **Follow-up chips 留空** — abstain 时 bubble 正文已承担 redirect,不再叠加 chips(与 §4.2 服务端协同)
- **a11y** — marker 文字进 bubble accessible name(`chat-design-system.md §7.5`)

### 2.3 服务端契约

**worker 端 ✅ 已产出**

`listing-chatbot/src/listing_chatbot/worker/kafka_writer.py:256-273` 的 `write_done` payload 已包含:
```python
payload = {
    "finish_reason": finish_reason,
    "abstained": abstained,            # ✅ 已写
    "null_reason": null_reason,        # ✅ 已写
    ...
}
```

**chat-service 端 ⚠️ 接到但未透传**

`realagent-services/apps/chat-service/src/services/orchestrator.service.ts:329` 已读 `message.payload.abstained` 用于业务决策(abstain 不入 conversation 持久化),但 `writeDoneEvent`(:401-411)生成 SSE chunk 时**丢弃**了:

```ts
// 当前(缺字段)
private writeDoneEvent(reply, roundId, model, sessionId): void {
  const oaiChunk = {
    id: roundId,
    object: 'chat.completion.chunk' as const,
    created: Math.floor(Date.now() / 1000),
    model,
    choices: [{ index: 0, delta: {}, finish_reason: 'stop' }],
    sessionId,
  };
  reply.raw.write(`data: ${JSON.stringify(oaiChunk)}\n\n`);
}
```

**需改动**(chat-service 端 ~3 行):

```ts
// 改后
private writeDoneEvent(
  reply, roundId, model, sessionId,
  abstained: boolean,                  // ← 新参数
  nullReason: string | null,           // ← 新参数
): void {
  const oaiChunk = {
    id: roundId,
    object: 'chat.completion.chunk' as const,
    created: Math.floor(Date.now() / 1000),
    model,
    choices: [{ index: 0, delta: {}, finish_reason: 'stop' }],
    sessionId,
    abstained,                                              // ← 新字段
    null_reason: nullReason,                                // ← 新字段
  };
  reply.raw.write(`data: ${JSON.stringify(oaiChunk)}\n\n`);
}
```

调用点 `orchestrator.service.ts:346` 把已经拿到的 `abstained` 和 `message.payload.null_reason` 传进去即可。

### 2.4 前端落地

**Schema 已通**:
- `useChatStream.ts:112-115` `DonePayload` 已含 `abstained` / `nullReason`
- `useAiAssistant.ts:42-44` `ChatMessage` 已含 `abstained` / `nullReason`
- `useAiAssistant.ts:258-259` `onDone` 已写入 message

**只需补 UI 消费 `AppAiChatMessages.vue`**:

```vue
<div
  :ref="(el) => setBubbleRef(m.id, el as HTMLElement)"
  class="message-bubble"
  :class="{ collapsed: ..., abstained: m.role === 'assistant' && m.abstained }"
>
  <div v-if="m.role === 'assistant' && m.abstained" class="abstain-marker">
    <font-awesome-icon icon="fa-solid fa-circle-info" />
    <span>{{ t('aiChat.abstainMarker') }}</span>
  </div>
  <div v-if="m.role === 'assistant'" class="message-content md-content" v-html="renderMarkdown(m.content)" />
  <div v-else class="message-content">{{ m.content }}</div>
</div>

<!-- group 内 source-label sibling 区:已有 m.sourceLabel 渲染之后插入 -->
<span
  v-if="m.role === 'assistant' && m.abstained && m.nullReason"
  class="null-reason"
>
  {{ m.nullReason }}
</span>
```

**新增 CSS**:

```scss
.abstain-marker {
  display: flex;
  align-items: center;
  gap: 6px;
  margin-bottom: 6px;
  font-size: 11px;
  font-weight: 500;
  color: #808080; // --muted-foreground

  .svg-inline--fa { width: 12px; height: 12px; }
}

.null-reason {
  display: inline-block;
  margin-top: 4px;
  margin-left: 16px;
  font-size: 10px;
  font-style: italic;
  color: #808080;
  line-height: 1.4;
}
```

**Feature flag**: `environment.json` 加 `listingChatbotAbstainUi: false`,marker / nullReason 渲染均 gate 在此 flag。

### 2.5 i18n / GA

| 新增 i18n key | EN | ZH |
|---|---|---|
| `aiChat.abstainMarker` | Outside my listing knowledge | 超出我对这套房子的了解 |

| 新增 GA 事件 | 触发 | hs_label |
|---|---|---|
| `ai_chat_abstain_seen`(可选) | abstain bubble 渲染时 | nullReason 类别(`insufficient_capability` / `insufficient_data` / `mcp_unavailable`) |

### 2.6 验收

- chat-service 透传后,带 `abstained: true` 的 done frame 在前端能写到 `m.abstained`
- bubble 顶部出现小灰条「Outside my listing knowledge」;有 `null_reason` 时下方出现 italic 灰字
- bubble 背景仍是 `--muted`,不和 error 视觉混淆
- 关 flag 时 marker 不出现,行为字节级回到今天

---

## 3. 提案 B · Stage-aware 加载文案

### 3.1 设计意图

v1 "Looking up..." 是系统-passive 单一文案,等待期感觉像检索。v2 想让等待期成为可识别的**思考过程**,让用户感到答案是被推理出来的。

> **本提案不改视觉,只切换标签字符串源**。Loading bubble 几何 / dot animation / progress label 排版见 `chat-design-system.md §5.10`,实施者需保证字符串切换不破坏几何。

文案切换序列(参照 `v2-nlqa.md §9.3`):
1. t=0 → 首个 ProgressEvent: `Thinking…`
2. 收到 `intent_classification` / `scope_tool_planning` / `tool_execution` 阶段: `Reading the listing…`
3. 收到 `response_strategy` 阶段: `Considering the question…`
4. 收到 `response_rendering` 阶段: `Composing your answer…`
5. t≥10s 仍无进度: `Still thinking…`
6. 首个 chunk 到达: dots 消失,bubble 转流式

### 3.2 服务端契约

**worker schema ✅ 已就位**

`listing-chatbot/src/listing_chatbot/types.py:415-422` `ProgressEvent`:
```python
class ProgressEvent(BaseModel):
    stage: str          # Node name from graph
    message: str        # User-safe status message
    timestamp_ms: int
```

**需 worker 实际 emit**

`response_strategy.py` / `intent_classification.py` / `tool_execution.py` / `response_rendering.py` 等节点入口处需要主动 `yield ProgressEvent(...)`,且 `message` 是**用户可见的 redirect 友好文案**,不是 stage 名字本身。

推荐 stage → message 映射(由 worker 决定):
- `intent_classification` → "Reading the listing…"
- `scope_tool_planning` → "Reading the listing…"(合并)
- `tool_execution` → "Reading the listing…"(合并)
- `response_strategy` → "Considering the question…"
- `response_rendering` → "Composing your answer…"

要求**每轮至少 2 个 ProgressEvent**,否则文字会卡在 "Thinking…"(v2-nlqa.md §9.3 末尾约束)。

chat-service 端已透传 ProgressEvent — 见 `useChatStream.ts:418-425` `onProgress` 已经在收。

### 3.3 前端落地

**前端 0 改动**。`AppAiChatMessages.vue:115-126` 三段优先级 ladder 已就位:

```ts
const loadingLabel = computed<string>(() => {
  if (environment.listingChatbotProgressEvent && props.progressLabel && props.progressLabel.trim()) {
    return props.progressLabel  // ← worker stage 文案接管
  }
  return props.loadingDuration >= 10 ? t('aiChat.stillThinking') : t('aiChat.thinking')
})
```

**建议**:worker 一旦稳定 emit 后,把 `environment.json` 里 `listingChatbotProgressEvent` flag 打开成默认 true(目前估计是 false 用作 staging gate)。

### 3.4 i18n / GA

i18n 文案由服务端 `ProgressEvent.message` 字段直接下发,**worker 需根据用户语言决定文案语言**(EN / ZH 双语)。前端不再做映射。

| 新增 GA 事件(可选) | 触发 | hs_label |
|---|---|---|
| `ai_chat_progress_stage_seen` | 每个 stage 文案出现时 | stage 名 |

### 3.5 验收

- 用户在 sheet 内发问后,typing-dots 文字从 "Thinking…" 切到 "Reading the listing…" 再切到 "Composing your answer…"
- 若 worker 偶尔某 stage 不 emit,文字保持上一阶段不闪烁
- 关 `listingChatbotProgressEvent` flag 时回到 thinking / stillThinking 静态文案

---

## 4. 提案 C · Abstain 文案 redirect 语气

### 4.1 设计意图

v1 abstain 文案是 data-gap framing(实际生产例):
> "I can't determine the seller's bottom-line price from the listing. Asking price is $524,886."

等于告诉用户"你问的没数据"。v2 改为 redirect 语气:
> "Listing data alone won't tell us — but recent comparable sales might. Want me to compare?"

**核心**:每条 abstain 必须以**一个有用的下一步**收尾,而不是死胡同。

### 4.2 服务端契约

`listing-chatbot/src/listing_chatbot/agent/nodes/finalize.py` 和 RR 节点 system prompt 的 abstain 输出 **shape + 文案语气**调整(不是重写整个 prompt,详见 §12.1 第 3 条):

- 强制 **redirect-shape 输出**:不能只说"我不知道",必须给出 ≥1 条延展建议(asking agent / comparing comparable sales / 换个问题角度)
- 与 §2(UI marker)/ §5(grounding type)/ §6(follow_ups)输出协同 ——
  - **`follow_ups: []`(空数组,不输出 chips)**:abstain 时 bubble 正文已经承担 redirect,不再叠加 follow-up chips,避免语义重复 + 视觉噪音(canvas Frame 4A `X5oYU` 与此一致)
  - **`grounding_type: None`**:abstain 没有 grounding 可分类(可选 `assumption` 但保守起见用 `None`)

### 4.3 前端 / i18n / GA

**无**。文案 LLM 输出、前端 bubble 已能渲染。GA 复用 §2 `ai_chat_abstain_seen`(可附"redirect 含 follow_ups 比例"指标)。

### 4.4 验收

- 抽样 100 次 abstain done frame,bubble 文案 100% 包含 redirect 短语(可用关键词列表自动检测)
- 用户在 abstain 后**继续发问**的比例上升 vs v1 baseline(GA)

---

## 5. 提案 D · Source / Grounding Label

### 5.1 设计意图

v2 助手允许超出 listing payload 推理(市场对比、买家视角、常识建议),用户必须能一眼看出**这条答案来自哪里**。**信任来自透明,不是来自措辞**。

### 5.2 视觉规格

视觉细节见 `chat-design-system.md §5.8` + `v2-nlqa.md §10.1`。实施关键约束:

- 位置:`.message-bubble` 同级 sibling,bubble 下方(CSS `.source-label` `AppAiChatMessages.vue:327-335` 已就位)
- **仅在 done 之后渲染**(流式期间不显示,防 flicker)
- 4 枚举 → i18n key 映射见 §5.5

### 5.3 服务端契约

**worker 需新增字段**:

`listing_chatbot/types.py` `ChatResponse` / done payload 加:
```python
grounding_type: Literal["from_listing", "from_market_context", "general_advice", "assumption"] | None = None
```

`kafka_writer.write_done` payload 同步加 `grounding_type` 字段(snake_case)。

**system prompt 改造**:
- finalize 节点或 RR 输出阶段决定 grounding type,根据答案的**主要**信息来源分类
- 一条答案只有一个 grounding type(取 primary 依据)

**chat-service 透传**:
- `writeDoneEvent` 加 `grounding_type` 字段(类似 §2 abstain 的 ~3 行改动,合并提交)

### 5.4 前端落地

**Hooks 扩展**

`useChatStream.ts` 改动:
- `DonePayload`(:112-115)加 `groundingType?` 字段
- `ChunkPayload`(:354-366)加 `grounding_type?` snake_case 字段
- done 分支(:442-449)透传 `grounding_type → groundingType`

`useAiAssistant.ts` 改动:
- `ChatMessage`(:28-48)加 `groundingType?` 字段
- 模块作用域加 enum → i18n key map:

```ts
const GROUNDING_LABEL_KEY = {
  from_listing: 'aiChat.groundingFromListing',
  from_market_context: 'aiChat.groundingFromMarket',
  general_advice: 'aiChat.groundingGeneralAdvice',
  assumption: 'aiChat.groundingAssumption',
} as const
```

- `onDone`(:255-262)flag on 时写 `m.sourceLabel`:

```ts
if (environment.listingChatbotGroundingLabel && payload.groundingType) {
  m.groundingType = payload.groundingType
  const key = GROUNDING_LABEL_KEY[payload.groundingType]
  if (key) m.sourceLabel = t(key)
}
```

**渲染层 ✅ 已就位**
- `AppAiChatMessages.vue:46-51` 模板已渲染 `m.sourceLabel`
- CSS `.source-label`(:327-335)已定义:Poppins 13 italic、`$color-cyan-strong`、margin-top 4、margin-left 16

**Feature flag**: `environment.json` 加 `listingChatbotGroundingLabel: false`。

### 5.5 i18n / GA

| 新增 i18n key | EN | ZH |
|---|---|---|
| `aiChat.groundingFromListing` | Based on this listing | 基于这套房源 |
| `aiChat.groundingFromMarket` | Based on market comparison | 基于市场对比 |
| `aiChat.groundingGeneralAdvice` | General guidance | 一般性建议 |
| `aiChat.groundingAssumption` | Inferred — verify before relying | 推断 — 请自行核实 |

| 新增 GA 事件 | 触发 | hs_label |
|---|---|---|
| `ai_chat_grounding_seen` | source label 渲染入 DOM | groundingType |

### 5.6 验收

- 每条 assistant done frame 携带 `grounding_type`,前端写到 `m.sourceLabel`
- bubble 下方出现 italic teal 一行
- 流式期间不显示(防 flicker)
- DOM `.source-label` 出现率 = done frame 带 `grounding_type` 的比例
- 关 flag 时不渲染

---

## 6. 提案 E · Follow-up Suggestion Chips

### 6.1 设计意图

v1 单回合心智模型下,用户拿到答案就结束。v2 是会话伙伴,**对话不能 dead-end**。助手给出答案后主动建议 2-3 个自然延展问题,把"问完关闭"变成"还能聊"。

### 6.2 视觉规格

视觉细节见 `chat-design-system.md §5.3` + `v2-nlqa.md §10.3`。chip 形状参考 hero Suggestion Chip 小一档(高度 28-30 vs hero 32-36)。

**关键约束**:**只在最新 assistant bubble 下方显示**,新一轮发起后旧 chips 自动消失 — 避免「3 轮后页面下方堆着十几个 chips」的视觉污染(`lastAssistantId` 守护,见 §6.4)。

### 6.3 服务端契约

**worker 需新增字段**:

`kafka_writer.write_done` payload 加:
```python
follow_ups: list[str] = []   # 0-3 条
```

**system prompt 改造**:
- finalize / RR 节点产出后,worker 让 LLM **顺手生成 2-3 条自然延展问题**
- 问题语气贴合上下文(基于上一轮的 question / answer)
- 单语种(根据用户输入语言决定 EN / ZH)
- 控制长度 ≤ 60 字符,放得下 chip

**chat-service 透传**:
- `writeDoneEvent` 加 `follow_ups: string[]` 字段

### 6.4 前端落地

- **新组件** `packages/app/src/pages/listing/components/AppAiChatFollowUps.vue` — props `{ chips: string[]; disabled?: boolean }`,emit `select`;模板 / SCSS 参照 chat-DS §5.3
- **Hooks 扩展**(与 §5 合并提交效率最高):
  - `useChatStream.ts` `DonePayload` / `ChunkPayload` 加 `followUps?` / `follow_ups?` 并 done 分支透传
  - `useAiAssistant.ts` `ChatMessage` 加 `followUps?`,`onDone` 写 `m.followUps`
- **集成** `AppAiChatMessages.vue`:`<AppAiChatFollowUps>` 渲染条件 `m.id === lastAssistantId && m.done && m.followUps?.length`;`@select` 沿 `AppAiChatSheet.vue` `onChipSelect` 通路 submit
- **Feature flag**: `environment.json` 加 `listingChatbotFollowUps: false`

### 6.5 i18n / GA

**i18n**: chip 内容由服务端下发,**前端不需要 i18n**(worker 根据用户输入语言决定 chip 文案语言)

| 新增 GA 事件 | 触发 | hs_label |
|---|---|---|
| `ai_chat_follow_up_click` | follow-up chip 点击 | chip 文案 |

### 6.6 验收

- done frame 带 `follow_ups: ["xxx", "yyy"]` 时最新 bubble 下方出现 2-3 个 chip
- 新一轮发起后旧 chips 消失(`lastAssistantId` 守护)
- 点击 chip = 重新走一轮带该问题的 chat,GA 出现 `ai_chat_follow_up_click`
- 平均会话深度(rounds per session)上升 vs v1 baseline
- 关 flag 时不渲染

---

## 7. 提案 F · Idle Re-engagement Prompt

### 7.1 设计意图

用户拿到答案后短暂沉默(30s 无新提问)是会话伙伴模型下的**机会窗口**。主动抛出新角度可能把单次问答转成多回合探索。

### 7.2 视觉规格

参照 v2-nlqa.md §10.6:
- 触发: `hasMessages === true` 且 conversation 闲置 30s
- 形式: composer 上方一行 muted 文字 "Still curious?" + 2-3 个新 follow-up chips
- 解除: 任意用户操作(点击 / 滚动 / 输入)
- chip 视觉**复用 §6 的 `AppAiChatFollowUps` 组件**

### 7.3 服务端契约

**首版**:不需要服务端独立 emit,**复用最近一次 done frame 的 `follow_ups`** — 用户已经看到过的 chips 也允许再次浮出,语义是"还在等你点"。

**进阶**(P3):服务端可单独 emit "idle re-engagement" SSE 事件类型,给出**不同**的 chips(避免重复)。首版不做。

### 7.4 前端落地

**`AppAiChatSheet.vue` 加 idle timer**:
- `onSubmit` / `onChipSelect` / 任意 user activity(touchmove / focus / click)时**重置**计时器
- 30s 无活动 → 显示 idle prompt
- 任何后续 activity 立即隐藏

**渲染**:`<AppAiChatFollowUps>` 复用 + "Still curious?" 一行

**Feature flag**: `environment.json` 加 `listingChatbotIdleReengage: false`

### 7.5 i18n / GA

| 新增 i18n key | EN | ZH |
|---|---|---|
| `aiChat.stillCurious` | Still curious? | 还想了解更多? |

| 新增 GA 事件 | 触发 | hs_label |
|---|---|---|
| `ai_chat_idle_reengage_seen` | Idle prompt 显示 | — |
| `ai_chat_idle_reengage_dismissed` | 用户没点 chip 直接动了别处 | dismiss 方式(scroll / close / type) |

### 7.6 验收

- 30s 静默后 composer 上方出现 "Still curious?" + 复用 chips
- 任何 user activity 立即隐藏
- GA `ai_chat_idle_reengage_seen` 出现率合理(不应过频)
- 关 flag 时永不触发
- **30s 阈值是 spec 初始建议**,上线后 A/B 验证(过低 = 被视为骚扰)。依赖见 §9 协同要点。

---

## 8. 提案 G · Confidence Indicator (deferred)

**当前结论:v2 不做。** 视觉规格见 `v2-nlqa.md §10.2`(glyph 三档:实心点 / 空心点 / 三角)。

**门控**:§5 上线 2-4 周后,若用户研究 / GA 显示 source label 仍不足以让用户分辨"可核实 vs 建议性",才评估补这层;否则永远不做(视觉噪音 > 增益)。复用 §5 `grounding_type` 字段,无新服务端契约。

---

## 9. 依赖与落地顺序

按 ROI / 改动量比 + Sprint 切分:

| 顺序 | Sprint | 提案 | 工作量 | ROI |
|---|---|---|---|---|
| 1 | S1 | §2 Abstain UI | chat-service 3 行 + 前端 30 行 + 1 i18n key | 最低门槛,死字段复活,语义区分 abstain/error 是 v2 心智核心 |
| 2 | S1 | §3 Stage-aware 加载 | 0 前端,纯 worker emit | 前端 ladder 已就位,worker 改动易回滚 |
| 3 | S1 | §4 Abstain redirect 语气 | 纯 worker prompt | 与 §2 协同闭环 |
| 4 | S2 | §5 Source Label | worker 字段 + chat-service + 前端 hooks + 4 i18n keys | 信任层建设,CSS / 模板已就位 |
| 5 | S2 | §6 Follow-up Chips | worker 字段+prompt + chat-service + 新组件 | 会话延展性,GA 影响显著 |
| 6 | S3 | §7 Idle Re-engagement | 纯前端定时器(依赖 §6) | 增量小,效果待 A/B 验证 |
| — | — | §8 Confidence Indicator | deferred,见 §8 | — |

**协同要点**:
- §2 + §4 = abstain 闭环(UI + 语气一体)
- §5 + §6 共享 Hooks schema 改动 + chat-service `writeDoneEvent` 改动,合并 PR
- §7 依赖 §6 数据源(`m.followUps`)
- S3 期间 §5 用户研究观察启动,为 §8 deferred 决策提供输入

---

## 10. 推荐落地顺序

*合并入 §9。*

---

## 11. 上线策略

继承自 v2-nlqa.md §14 风险表的 mitigation 原则:

- **每个提案独立 feature flag**(`environment.json`),关旗时与今天行为字节级一致
- 已有同 pattern 的 flag(`listingChatbotProgressEvent`)证明这条路径在项目内可行,直接照抄
- 服务端字段先于前端开放可以兼容 — 新字段对老前端是 unknown field,被忽略不报错
- 前端 hooks 扩展先于服务端字段可发 — `payload.groundingType` 为 undefined 时分支不执行,no-op

**回滚路径**:
- §2: 关 `listingChatbotAbstainUi` flag → marker 不渲染
- §5: 关 `listingChatbotGroundingLabel` flag → onDone 不写 sourceLabel
- §6: 关 `listingChatbotFollowUps` flag → 组件不渲染
- §7: 关 `listingChatbotIdleReengage` flag → 定时器不启动

服务端字段不需要专门 disable — 关旗时前端不消费,字段空转无 cost。

### 11.1 与 `v2-nlqa.md §14` 单 flag 方案的关系

上游 §14 Risk 表建议单一 `listingChatbotNLQA` flag。**本文细化为多 flag**(每提案一个),理由:跟服务端字段就绪时间错配 + 单点回滚 + 可独立 dark-launch / A/B。`listingChatbotNLQA` 保留作 v2 心智模型主开关的**概念分类标签 / GA 维度**,实际控制权下放到子 flag。PR description 显式说明此决策,避免看起来背离上游 spec。

---

## 12. System Prompt 改造方向汇总(从 `v2-nlqa.md §11` System Prompt Direction 整合)

跨 §2-§6 多个提案的 system prompt 改造意图集中在此,**给 `listing-chatbot` worker prompt owner 一个完整视图**,避免分头打补丁。所有要点设计端不写具体 prompt 文字,只给出意图与产出契约。

### 12.1 五条意图(三档工程 delta)

> 标签校正设计语义与生产代码的版本轴错位:**✅** = 生产 prompt 已满足;**⚠️** = 现有 prompt 措辞/shape 微调,无新字段;**🆕** = worker schema + prompt 新增字段,前后端联动。

| # | 意图 | Delta | 关联 | 工程动作 |
|---|---|---|---|---|
| 1 | Framing as conversation partner(非 JSON 查询函数) | ✅ | 背景 | `06_response_rendering_analytical_system.txt` 已要求 "reasoning visible" + 多维度思考 + `reason` 字段。验证保持即可 |
| 2 | Reason beyond payload(market / comparables / 买家视角) | ✅ | §5 枚举 | STP / IC 已要求 comparison + market reasoning;worker 端 9 个 market/comparison MCP 工具长期在产线。"v1 only answer from listing data" 是 v1 surface 设计语义,不是生产现状 |
| 3 | Redirect-shape abstention(永不死胡同) | ⚠️ | §4 | `finalize.py` + RR prompt 的 abstain 输出 shape 与 tone 微调,无新字段 |
| 4 | Emit `grounding_type` per round(4 枚举,取 primary) | 🆕 | §5 | `types.py` 加字段;RR/finalize prompt 加分类步骤;`kafka_writer.write_done` 加字段;chat-service `writeDoneEvent` 透传 |
| 5 | Emit `follow_ups`(≤3,贴合上下文,≤60 字符) | 🆕 | §6 / §7 | 同上;首版复用 finalize/RR LLM call 一次性输出,不另起 call(成本预算见 §13.2) |

### 12.2 输出契约(汇总)

worker `kafka_writer.write_done` payload 需新增字段:

```python
payload = {
    # 已有字段
    "finish_reason": str,
    "sanitized_query": str,
    "abstained": bool,
    "null_reason": str | None,
    # ... 其他已有 telemetry
    # 新增:
    "grounding_type": Literal["from_listing", "from_market_context",
                              "general_advice", "assumption"] | None,
    "follow_ups": list[str],  # 0-3 条
}
```

worker `ProgressEvent` 在每个关键 stage 入口处 emit,`message` 为 user-friendly 文案(EN/ZH 双语,根据 user 输入语言决定):

```python
yield ProgressEvent(
    stage="intent_classification",          # 节点名
    message="Reading the listing…",         # 用户可见文案
    timestamp_ms=time.time_ns() // 1_000_000,
)
```

### 12.3 不在 prompt 改造范围

AI Avatar、Voice input、Multi-listing memory — 见 §13 v3 backlog 信号。

### 12.4 Owner 与协作

- prompt 文字内容:`listing-chatbot` 团队(prompt engineering owner)
- 意图与 tone:design 团队(本文 §12.1 是输入)
- 输出契约:`listing-chatbot` worker schema owner + 前端 `useChatStream.ts` schema owner 共同 review
- 评估:`listing-chatbot/eval/` 现有评估框架追加 v2 prompt 的 `grounding_type` / `follow_ups` 产出准确度场景

---

## 13. 产品待决策项(承接 `v2-nlqa.md §13` Open Questions)

v2 范围内真正悬而未决、影响落地的 3 项:

| # | 决策项 | 影响 | 决策方式 |
|---|---|---|---|
| 13.1 | **Source label 粒度**:4 枚举够不够?是否加 "verified by HouseSigma data" 第 5 tier(AVM-backed) | §5 | 产品 + UX 研究;默认先 4 个,观察 §13.3 用户研究结论 |
| 13.2 | **Follow-up 生成成本预算**:可接受的额外 latency / token 预算? | §6 | 产品 + AI infra;首版**复用** finalize LLM call 一次性输出(不另起 call),之后看 GA 转化率决定 |
| 13.3 | **Abstention discoverability**:显示 "Outside my listing knowledge" 是降信任还是提信任? | §2 | 上线立 A/B(50/50 marker 显示 vs 不显示),衡量 "abstain 后继续发问比例" |

**v3 backlog 信号**(不在 v2,但产品需关注):AI avatar / persona mark、Voice input、Multi-listing memory("和昨天看的那套比" 类 query 频率)。详见 `v2-nlqa.md §13`。

---

## 14. 风险与回滚

**上游 `v2-nlqa.md §14` 5 条产品风险继承**(LLM 幻觉、成本暴涨、范围外问题、factual 用户体验、死字段复活破坏 build)— 不重写,缓解机制已在 §5 / §11 / §12.1 体现。以下为 proposal-specific 工程风险:

| 风险 | 可能性 | 影响 | 缓解 / 回滚 |
|---|---|---|---|
| chat-service `writeDoneEvent` 字段透传破坏老前端 | 低 | 中 | SSE 新字段对老前端 unknown,JSON.parse 不报错;前端 schema 缺字段时分支不执行 |
| §5/§6 服务端字段就绪但前端 hook 未扩展 | 中 | 低 | §11 已说明字段先开放兼容,空转无 cost |
| 多 flag 维护成本 | 低 | 低 | 子 flag 命名前缀统一 `listingChatbot*`,所有 flag 默认 false,逐项灰度 |

**回滚粒度**:见 §11"回滚路径",每个提案独立 flag 控制。

---

## 15. 参考

### 设计源(同目录)
- `v2-nlqa.md` — 设计意图源,本文 §12-§14 对照 §11/§13/§14
- `chat-design-system.md` — 视觉/组件/状态机权威,本文每提案视觉 spec defer 到此
- `listing-chatbot-design.pen` Frame 4A(`c7EJq`)/ 4B(`c565Q`)— 全量 v2 / surface 落地版 12 状态屏
- `chat-system-design.pen` — chat DS canvas(可复用 12 状态)

### 服务端基线
- `realagent-services/apps/chat-service/src/services/orchestrator.service.ts:329, 346, 401-411` — done frame 构建点
- `listing-chatbot/src/listing_chatbot/types.py:415-422, 524-525` — `ProgressEvent` / `abstained` / `null_reason`
- `listing-chatbot/src/listing_chatbot/worker/kafka_writer.py:256-273` — Kafka done payload
- `listing-chatbot/src/listing_chatbot/agent/nodes/finalize.py` — abstain 决策点
- `realagent-services` 分支:`feature/PLAN-5931-mcp-client-test`(非 master)

### web-hybrid 源
- `packages/hook/ai/useChatStream.ts:112-115, 354-366, 442-449` — schema 扩展点
- `packages/hook/ai/useAiAssistant.ts:28-48, 255-262` — message 扩展 + onDone 写入点
- `packages/app/src/pages/listing/components/AppAiChatMessages.vue:46-51` — 已有 sourceLabel 渲染钩子
- `packages/common/i18n/translation/{en,zh}.ts:965-998 / 904-926` — `aiChat.*` namespace

Owner 协作见 §12.4。配套文档:[`v2-nlqa-surface-changes.md`](./v2-nlqa-surface-changes.md)。
