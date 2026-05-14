# Listing Chatbot v2 NLQ&A — Surface Changes

> **范围**: v2 NLQ&A surface 层**可独立落地、零跨仓依赖**的改动(文案 / 入口 / chip 重组 / coachmark / loading 文案 / 错误文案)。配套 [`v2-nlqa-proposals.md`](./v2-nlqa-proposals.md) 覆盖需服务端配合的 7 个提案。
>
> **上游权威源**(同目录):
> - [`v2-nlqa.md`](v2-nlqa.md) — Positioning Shift、Variants、Copy 表、UI patterns、Prompt direction、Risk
> - [`chat-design-system.md`](chat-design-system.md) — 12-state 状态机、Components、Composition Rules(本文不重写组件视觉)
>
> **版本轴**: v1 / v2 = 设计 positioning 版本,不是代码版本号。
>
> **明确移出范围**(等服务端配合,见 proposals 对应章节):Abstain UI(§2)/ Abstain redirect prompt(§4)/ Source Label(§5)/ Follow-up Chips(§6)/ Stage-aware loading(§3)/ Idle re-engage(§7)/ Confidence Indicator(§8 deferred)
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

## 1. 设计原则(继承自 `chat-design-system.md` §2)

落地必须遵守的 6 条原则。前 5 条本文都体现,第 6 条作为**已决定不动**的项明确:

| 原则 | 在本文的体现 |
|---|---|
| **Conversational over transactional** | 沿用现有 chat-radius / pill / 圆形 send 体系,视觉壳子保持 Variant C(§10 视觉差异表只标待对齐的小幅 deltas) |
| **Assistant-active voice** | "Thinking…" 替 "Looking up…",errorGeneric 改 "Something tripped me up. Try again?" |
| **Redirect, don't refuse** | errorSafetyBlocked 改 "Let's stay focused on this listing — happy to help with anything about the home" |
| **Trust through transparency** | Source/Grounding Label 是 proposals §5 范围,本文等服务端 |
| **Multi-turn by default** | Follow-up Chips / Idle re-engagement 在 proposals §6/§7,本文不做 |
| **Brand-identifiable AI mark**(**已决项**) | Phosphor `star-four` 是 AI 标识的**一次性图标家族例外**(`chat-design-system.md §5.6`)。`AppAiChatHero.vue:4` 当前用 font-awesome `fa-sparkle` 是历史遗留,**视觉等效但非 spec 钦定**。本文不改(留 P2 视觉对齐,见 §10),实施者**不要误以为这是开放讨论项** |

---

## 2. 设计意图回顾

### 2.1 v1 → v2 心智模型差异

| 维度 | v1 lookup (今天) | v2 NLQ&A (目标) |
|---|---|---|
| 用户心智 | 「查一条事实」 | 「就这套房子聊天问任何东西」 |
| 答复尺度 | 单事实、单回合 | 多回合、开放式、含市场 / 建议视角 |
| 文案语气 | 被动系统化 (Looking up…) | 主动助手化 (Thinking…) |
| 兜底方式 | 「信息不在房源中」 | 「这角度回答不了 — 帮你换个角度?」 |

### 2.2 v1 → v2 能力边界扩展

v1 lookup 时代**明确不做**的事(historical scope boundary,v2-nlqa.md §2 演进对照):
- Comparing this listing to other properties
- Mortgage or affordability calculations
- Market trend analysis
- Agent recommendations
- Booking or scheduling actions

v2 把其中 **2 项从"能力存在但用户不可触达"转为"一级入口"**:

> **重要事实校正**: Comparing / Market context **不是 v2 新建的能力**。worker 端 `listing-chatbot/src/listing_chatbot/reference/data_fetcher.py:22-50` 注册的 23 个 MCP 工具里,**已有 9 个 market / comparison 类工具**长期在产线:`get_similar_listings`、`get_nearby_sold_listings`、`get_nearby_for_sale_listings`、`get_nearby_for_rent_listings`、`get_nearby_leased_listings`、`get_community_market_stats`、`get_municipality_market_overview`、`get_property_analytics`、`get_neighbourhood_demographics`。STP(`mcp/stp_context.py:349`)也把 "What have similar homes nearby recently sold for?" 列为 in-scope 高相关示例。eval 场景(`eval/scenarios/citation_selection/comparables_market_position.yaml`)也持续覆盖。**LLM 没有充分用是因为 v1 system prompt 写的是 "answer using listing data only" 把工具调用压着了** —— v2-nlqa.md §11 明确写了要解锁这块。

v2 的 2 项变化都是 **surface + prompt 层** 的:

- **Comparing** → chip 2 从 v1 的 "How long has it been listed?" 换成 v2 的 "How does it compare to nearby listings?"(`chipCompare`),把已有能力 promote 到一级入口
- **Market context** → System prompt 解锁推理(proposals §4 / §12.1 第 2 条)+ Source Label `from_market_context`(proposals §5)让用户看到答案"用了市场数据"

v2 **仍不做**的:Mortgage、Agent recommendations、Booking — 这些由 `safety_blocked` 错误码兜底并 redirect 回主题(`aiChat.errorSafetyBlocked` 已改语气)。

### 2.3 Variant 探索史

详见 `v2-nlqa.md §3`。要点:Variant B(强套 base DS)已否决 — base DS 适配 listings / filters / forms 但不适配 chat,Chat Design System 在 chat surface 内压过 base DS。Variant C(Chat-First)是 canonical,拆分为 4A(全量 v2) + 4B(本文落地版)。

### 2.4 本文覆盖的子集

v2 心智模型升级的"**入口面**" — 用户在打开 sheet、看 hero、敲第一个问题、等回答的几秒钟内,所有视觉文案语气从 lookup 转 NLQ&A。这部分是 v2 心智模型生效与否的**第一印象层**,且完全由 web-hybrid 自主决定,不需要任何服务端字段或数据透传。

**信任层 / 对话延展层**(Source Label、Follow-up Chips、Abstain UI、Stage-aware 进度)在服务端字段就绪后单独立项落地,见 proposals。

---

## 3. 12-state 状态机全景

State / Trigger / Key elements 完整定义见 `chat-design-system.md §4`。本表给出**每个 state 上本文 / proposals 谁负责**,避免来回翻:

| # | State | 本文(surface) | 相关 proposals |
|---|---|---|---|
| 1 | `trigger` | — | — |
| 2 | `hint` | §6.5 文案改 `askAboutThis` | — |
| 3 | `halfsnap` | §6.1 hero 文案 + chips、§6.2 placeholder | — |
| 4 | `entry_hero` | §6.1、§6.2 | — |
| 5 | `retrieving` | §6.3 加载文案 `thinking/stillThinking` | proposals §3 stage-aware 文案 |
| 6 | `streaming` | — | — |
| 7 | `stopped` | — | — |
| 8 | `result` | — | proposals §5 Source Label、§6 Follow-up Chips |
| 9 | `show_more` | — | — |
| 10 | `abstained` | — | proposals §2 Abstain UI、§4 abstain redirect 语气 |
| 11 | `noresult` | §6.6 errorGeneric 文案改值 | — |
| 12 | `error` | §6.6 errorSafetyBlocked / errorGeneric 改值 | — |

**对实施的启示**:本文 P0 改动主要落在 #3/#4/#5/#12 上。其余 state 在本文范围内**不动结构、不动视觉**,只有 #2/#10/#11/#12 的部分文案随 i18n 改值。

---

## 4. Component Composition Rules

详见 `chat-design-system.md §6`。本文对实施的启示:

- **Chat surface = sheet 容器内部**。sheet 内全部用 chat token(软圆角 16-24、pill 输入、圆形 send、`--muted` 灰底);sheet 外(app header、scrim 下 listing)跟 base DS
- **AI FAB 例外**:FAB 本身在 Bottom Action Bar,跟 base DS 与兄弟按钮 cohesion;FAB 触发的 hint popup 虽挂在 FAB wrapper 上,**算 chat surface**,用 chat token
- **base DS ≠ chat DS**(后者是 sister spec 不是 subset);Variant B 强套 base DS 已否决

---

## 5. 改动总览

### 5.1 按区块视图

| 区块 | 影响文件 | 优先级 |
|---|---|---|
| i18n key 重命名 + 改值 | `en.ts` / `zh.ts` | P0 |
| Hero chip 重组 + headline 改名 | `AppAiChatHero.vue` | P0 |
| 入口 placeholder 改名 | `AppAiChatSheet.vue` | P0 |
| Coachmark 文案改名 | `AppAiHintPopup.vue` | P0 |
| 加载文案改名 | `AppAiChatMessages.vue` | P0 |
| 错误文案语气微调 | `en.ts` / `zh.ts` | P0 |
| 视觉对齐(单独立项) | 多个 | P2 |

### 5.2 按文件视图(对齐 `v2-nlqa.md §8`)

本文实际触动的文件(细节见 §6/§7):

| 文件 | 改动 |
|---|---|
| `packages/common/i18n/translation/{en,zh}.ts:965-998 / 904-926` | 删 6 旧 key + 加 6 重命名 + 改 2 错误文案值 |
| `packages/app/src/pages/listing/components/AppAiChatHero.vue:5, 46, 47` | `entryLabel` → `askAnything`;chips 数组改 key |
| `packages/app/src/pages/listing/components/AppAiChatSheet.vue:40` | placeholder key 改 |
| `packages/app/src/pages/listing/components/AppAiChatMessages.vue:124-125` | 加载文案 key 切换 |
| `packages/app/src/pages/listing/components/AppAiHintPopup.vue:4` | `hintTitle` → `askAboutThis` |
| `packages/common/environments/types.ts:73, 77, 78` + `useAiAssistant.ts:78` | JSDoc 字面引用同步(纯文档串) |

**proposals 文档配套需改文件**(本文不动,见 proposals 各章节):`useChatStream.ts` / `useAiAssistant.ts` schema 扩展、`AppAiChatMessages.vue` 渲染 abstain marker / source label / follow-up chips、新建 `AppAiChatFollowUps.vue`、chat-service `orchestrator.service.ts` 透传、listing-chatbot `kafka_writer.py` payload 字段 + worker prompt。

---

## 6. i18n 改动清单

**文件**: `packages/common/i18n/translation/en.ts:965-998` + `zh.ts:904-926`。`aiChat.*` 命名空间共 17 个 key,5 处运行时 `t()` + 1 处 JSDoc 引用(详见 §6.1 同步消费点列)。

**策略**: 重命名 + 改值单 PR 一次性做完,对齐 `v2-nlqa.md §7` Copy Changes。`entryLabel` / `lookingUp` / `stillLooking` / `hintTitle` 带 lookup 语义,留下会持续误导,**不分阶段**。

### 6.1 重命名 + 改值(6 处)

| 旧 key | 新 key | EN | ZH | 同步消费点 |
|---|---|---|---|---|
| `entryLabel` | `askAnything` | Ask anything about this home | 问问这套房子的任何问题 | `AppAiChatHero.vue:5`、`AppAiChatSheet.vue:40` |
| `lookingUp` | `thinking` | Thinking… | 思考中… | `AppAiChatMessages.vue:125` |
| `stillLooking` | `stillThinking` | Still thinking… | 还在思考… | `AppAiChatMessages.vue:124` |
| `hintTitle` | `askAboutThis` | Ask about this home | 问问这套房子 | `AppAiHintPopup.vue:4` |
| `chipListed` | `chipCompare` | How does it compare to nearby listings? | 和附近房源比怎么样? | `AppAiChatHero.vue:46` |
| `chipSchools` | `chipCautions` | Anything I should be cautious about? | 有什么需要注意的地方吗? | `AppAiChatHero.vue:47` |

### 6.2 只改值(key 名保留)

少数 key 名本身没有问题,只调整文案语气:

| Key | 新 EN | 新 ZH |
|---|---|---|
| `errorSafetyBlocked` | Let's stay focused on this home — happy to help with anything about this listing. | 我们聚焦这套房子吧 — 关于这个房源的任何问题我都很乐意帮忙。 |
| `errorGeneric` | Something tripped me up. Try again? | 我这边出了点小问题,要不再试一次? |

`chipAbout`、`errorBusy`、`showMore`、`showLess` 已经足够口语化,**不动**。其他 `errorTtft` / `errorIdle` / `errorRound` / `errorTooLong` / `errorAuthExpired` / `errorCancelled` / `errorMcpError` / `errorInference` 语气调整可后续 sprint 跟进,**不阻塞本次 v2 改造**。

### 6.3 新增 key

无。设计文档原本列的 `noResultGeneric`、`abstainMarker`、`groundingFromListing/FromMarket/GeneralAdvice/Assumption`、`stillCurious` 等 key **均依赖服务端字段或数据源,移出本文范围**(见 proposals)。

### 6.4 操作清单(单次 PR)

`en.ts` / `zh.ts` 内 `aiChat` 命名空间:
1. 删 6 旧 key + 加 6 重命名 key + 填新 EN/ZH 值(§6.1)
2. 改 2 个 key 的值:`errorSafetyBlocked`、`errorGeneric`(§6.2)
3. 同步消费点(见 §6.1 末列)+ JSDoc 字面串(`common/environments/types.ts:73,77,78`)

> **必须单 commit / 单 PR** — i18n 与消费点错位会让 UI 显示 raw key(`aiChat.entryLabel`)。CI 加 i18n key 双向引用校验防呆。

---

## 7. 组件级改动

### 7.1 `AppAiChatHero.vue` — 入口 hero

**当前关键代码**(:44-48):
```ts
const chips = [
  { key: 'aiChat.chipAbout' },
  { key: 'aiChat.chipListed' },
  { key: 'aiChat.chipSchools' },
]
```

**改动**:
- chips 数组中 `chipListed` → `chipCompare`、`chipSchools` → `chipCautions`(`chipAbout` 保留)
- 模板 `:5` 把 `t('aiChat.entryLabel')` 改成 `t('aiChat.askAnything')`

**不动的**:
- `font-awesome-icon icon="fa-solid fa-sparkle"`(:4)— 是 `chat-design-system.md §5.6` Phosphor `star-four` 的 font-awesome 等效占位,**视觉对齐留 §10**
- scope-pill 结构、margin-top auto 锚定到底等布局
- 微视觉差(背景 `#e9f6f7` vs spec `--muted`、字号 40 vs spec 32)留到 P2 视觉对齐 sprint

### 7.2 `AppAiChatSheet.vue` — 容器

**P0**: 把 `:40` 的 `t('aiChat.entryLabel') + '...'` 改成 `t('aiChat.askAnything') + '...'`。

### 7.3 `AppAiChatMessages.vue` — 消息列表(本文范围内只动加载文案)

**当前**(:115-126): 三段优先级 = `progressLabel`(env flag) > `stillLooking`(≥10s) > `lookingUp`(default)。**结构不动**,把:

- `:124` `t('aiChat.stillLooking')` → `t('aiChat.stillThinking')`
- `:125` `t('aiChat.lookingUp')` → `t('aiChat.thinking')`

同步更新 `useAiAssistant.ts:74-78` `progressLabel` 注释块里残留的 `aiChat.lookingUp / aiChat.stillLooking` 字面引用(注释块横跨 5 行,旧 key 名出现在 :78,只是文档串,不影响运行,但避免误导后续读者)。同样的字面引用还出现在 `common/environments/types.ts:73,77,78`(见 §6 i18n 消费点表第 6 行)。

> `progressLabel` 这一段(env flag on + 服务端 stage 文案 emit 时显示)**保持原样不动**。底层机制已就位,但当前 worker 未 per-stage 发送 user-friendly 文案,实际跑起来还是落到 thinking / stillThinking — **不影响本文范围**。

> Abstain marker / Source label / Follow-up chips 的渲染**全部不在本文范围**(见顶部"明确移出范围")。

### 7.4 `AppAiChatComposer.vue` — composer

- aria-label `"Stop generating"`(:17)— 保持(v2-nlqa.md §8 明确说)
- placeholder 通过 `AppAiChatSheet:40` 透传,§7.2 改完 sheet 端引用后,这里自动跟 `askAnything` 更新,本组件 **无需改动**
- **不动**结构、按钮形状、动画
- 视觉差异(背景 `#ededf0` vs spec `#F2F2F2`)留到 P2 视觉对齐

### 7.5 `AppAiHintPopup.vue` — coachmark

- 把 `:4` 的 `t('aiChat.hintTitle')` 改成 `t('aiChat.askAboutThis')`,文案随 i18n 自动更新
- **不动**结构、定位、`localStorage.hint_ai_chat_viewed` 单次显示、自动 2s 关闭

> **挂载位置提示**: `AppAiHintPopup` 不挂在 sheet 内,而是挂在 `AppListingWatchActions.vue:35`(listing 详情页的 Bottom Action Bar 上,作为 AI FAB 的 sibling)。本 §7.5 改动只动 popup 组件本身;不要去改 sheet 内的 hint 引用 —— 那里没有。`localStorage.hint_ai_chat_viewed` 在 `AppListingWatchActions.vue:142, 204` 读写。

---

## 8. GA 事件

| 事件名 | 状态 | 触发 | hs_label |
|---|---|---|---|
| `ai_chat_open` | 已有 | sheet 打开 | source |
| `ai_chat_chip_click` | 已有 | hero chip 点击 | chip 文案(新文案后自动跟随) |
| `ai_chat_send` | 已有 | 提交输入 | — |
| `ai_chat_stop` | 已有 | 点 stop | — |

> 原计划新增的 `ai_chat_follow_up_click`、`ai_chat_grounding_seen` 依赖服务端字段,移出本文。

---

## 9. 落地优先级

### P0 — 当前 sprint 即可发,零服务端依赖

1. **i18n key 重命名 + 改值**(§6,单次 PR):6 个旧 key 删 / 6 个新 key 加 / 2 个 key 改值 / 5 处 `t(...)` 消费点同步
2. **Hero 3 chip 数组更新**(§7.1):`chipListed` → `chipCompare`、`chipSchools` → `chipCautions`
3. **加载状态文案引用切换**(§7.3):`lookingUp` / `stillLooking` → `thinking` / `stillThinking`

> P0 必须**单次 PR**完成,避免 i18n 与消费点错位。改动全部是字符串 + key 名,**不需要 feature flag**,改完就发。GA `ai_chat_chip_click` 可以观察到新 chip 文案的点击分布,作为 v2 心智模型生效与否的早期信号。

### P2 — 视情况再做

4. **视觉差异修复**(radius `16→20`、尾角 `2→4`、scope-pill 背景、Lucide 图标体系)— 视觉对齐通常单独立项,见 §10

### 移出范围(等服务端配合后单独立项)

- Abstain UI 启用 — 等 chat-service `writeDoneEvent` 透传 `abstained` / `null_reason`(改动 ~3 行)
- Source / Grounding Label — 等 worker 产出 `grounding_type` + chat-service 透传
- Follow-up Suggestion Chips — 等 worker 产出 `follow_ups: string[]` + chat-service 透传
- Stage-aware 进度文案 — 等 worker per-stage emit user-friendly `ProgressEvent.message`
- Idle Re-engagement Prompt — 依赖 `follow_ups` 数据源
- Abstain 文案换成「换个角度」语气 — 等 worker system prompt 改写

---

## 10. 未覆盖的视觉差异(参考,P2 单独立项)

| 项 | 当前 | spec |
|---|---|---|
| Bubble 主圆角 | 16px (`AppAiChatMessages.vue:281`) | 20px (`chat-radius-lg`) |
| Bubble 尾角 | 2px (`:307, 313`) | 4px (`chat-tail-radius`) |
| Hero scope-pill 背景 | `#e9f6f7` (`AppAiChatHero.vue:86`) | `--muted` (#F2F2F2) |
| AI sparkle 图标 | FontAwesome `fa-sparkle` | Phosphor `star-four`(`chat-DS §5.6` 钦定;见 §1 原则 6) |
| 通用图标体系 | FontAwesome | Lucide(引入是单独依赖工作) |

对 v2 心智模型**不构成阻塞**,后续视觉对齐 sprint 单独跟进。Composer / Hint popup 圆角 1-3px 极轻微差异已剔除。

---

## 11. 验收要点

| 阶段 | 用户可见的变化 | 观测信号 |
|---|---|---|
| P0 ship 后 | 首次进入看到 "Ask anything about this home" / "问问这套房子的任何问题"; chip 变成 1 通用 + 1 对比 + 1 谨慎; loading 文字变 "Thinking…" | GA `ai_chat_chip_click` 出现 `chipCompare` / `chipCautions` 的点击 |

---

## 12. 参考

### 设计(同目录)
- `v2-nlqa.md` — **v2 NLQ&A 设计意图 + 提案**:positioning shift、Variant 探索史、Design Principles、Copy 变更总表、新增 / 复活 UI patterns、System Prompt direction、Rollout、Risk
- `chat-design-system.md` — **Chat Design System**(sister spec to base DS,chat surface 视觉 / 组件 / token 权威),listing chatbot 是其首个落地 surface
- `listing-chatbot-design.pen` Frame 4B(node `c565Q`)— **本文落地版设计稿**:12 状态屏沿用,只更新 surface 范围内的文案
- `listing-chatbot-design.pen` Frame 4A(node `c7EJq`)— **全量 v2 设计稿**:12 状态屏含 abstain marker、Source Label、Follow-up Chips、stage-aware loading 文案等新组件渲染(见 proposals)
- `chat-system-design.pen` — chat DS 画布(Variant C 12 状态可复用版),含 reusable component library

### web-hybrid 源码
- `packages/app/src/pages/listing/components/AppAiChatSheet.vue` — 容器、snap、状态管理
- `packages/app/src/pages/listing/components/AppAiChatHero.vue` — 入口 hero、3 chip
- `packages/app/src/pages/listing/components/AppAiChatMessages.vue` — 消息列表、loading、source label hook、collapse
- `packages/app/src/pages/listing/components/AppAiChatComposer.vue` — 输入条、send/stop
- `packages/app/src/pages/listing/components/AppAiHintPopup.vue` — 首次进入 coachmark
- `packages/app/src/components/AppListingWatchActions.vue:33-45` — AI FAB 入口
- `packages/app/src/pages/listing/listing.vue:318-324` — chat-sheet 挂载点
- `packages/hook/ai/useAiAssistant.ts` — message 状态机、错误码映射、`stop`、`setMessages`
- `packages/hook/ai/useChatStream.ts` — 原始 SSE 解析、`DonePayload` / `ChunkPayload`、watchdog
- `packages/common/i18n/translation/{en,zh}.ts:965-998 / 904-926` — `aiChat.*` namespace

### 配套
- [`v2-nlqa-proposals.md`](./v2-nlqa-proposals.md) — 服务端依赖项提案存档
