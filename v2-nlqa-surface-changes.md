# Listing Chatbot v2 NLQ&A — Surface Changes

> **范围**: v2 NLQ&A surface 层**可独立落地、零跨仓依赖**的改动(文案 / 入口 / chip 重组 / coachmark / loading 文案 / 错误文案)。配套 [`v2-nlqa-proposals.md`](./v2-nlqa-proposals.md) 覆盖需服务端配合的提案。
>
> **上游权威源**(同目录):
> - [`v2-nlqa.md`](v2-nlqa.md) — Positioning Shift、Variants、Copy 表、UI patterns、Prompt direction、Risk
> - [`chat-design-system.md`](chat-design-system.md) — 12-state 状态机、Components、Composition Rules(本文不重写组件视觉)
>
> **版本轴**: v1 / v2 = 设计 positioning 版本,不是代码版本号。
>
> **明确移出范围**(等服务端配合,见 proposals 对应章节):Abstain UI / Abstain redirect prompt / Source Label / Follow-up Chips / Idle re-engage / Confidence Indicator(deferred)。Stage-aware loading **已在 master / `release/v3.3.0` 端到端落地**(chat-service commit `c8fc211`),本文 i18n 改名后会作为 stage 文案到达前的 fallback
>
> **代码基线**(2026-05-15):
>
> | 仓库 | 分支 | HEAD |
> |---|---|---|
> | `web-hybrid` | `master` | `30a3ab86c` |
> | `listing-chatbot` | `main` | `e2426ac` |
> | `realagent-services` | `master` | `9df6f56` |
> | `prototypes/listing-chatbot-design` | `main` | `ad56765` |
>
> **基线选择说明**:`realagent-services` 上一版文档误用 `feature/PLAN-5931-mcp-client-test` (HEAD `bf57453`, 2026-04-24) 作为基线 —— 该分支是 ci-only 的空壳分叉点(自有 0 commit,落后 master 31 commit),导致基于它做代码调查会得出与线上完全脱节的结论(典型如 stage-aware 透传:feature 视角是"gateway 没透传",master 视角是"`c8fc211` 早已端到端落地")。本次基线统一改回 `master`,`release/v3.3.0` 是当前线上版本,与 master 各差 1 commit。`listing-chatbot-design` 显式以 `main` 为主(remote HEAD 历史遗留指向 `entire/checkpoints/v1`,不作为基线)。

---

## 设计原则(继承自 `chat-design-system.md`)

落地必须遵守的 6 条原则。前 5 条本文都体现,第 6 条作为**已决定不动**的项明确:

| 原则 | 在本文的体现 |
|---|---|
| **Conversational over transactional** | 沿用现有 chat-radius / pill / 圆形 send 体系,视觉壳子保持 Variant C,只标出待对齐的小幅 deltas |
| **Assistant-active voice** | "Thinking…" 替 "Looking up…",errorGeneric 改 "Something tripped me up. Try again?" |
| **Redirect, don't refuse** | errorSafetyBlocked 改 "Let's stay focused on this listing — happy to help with anything about the home" |
| **Trust through transparency** | Source/Grounding Label 是 proposals 范围,本文等服务端 |
| **Multi-turn by default** | Follow-up Chips / Idle re-engagement 在 proposals,本文不做 |
| **Brand-identifiable AI mark**(**已决项**) | Phosphor `star-four` 是 AI 标识的**一次性图标家族例外**(见 `chat-design-system.md` 对应章节)。Hero 当前用 font-awesome `fa-sparkle` 是历史遗留,**视觉等效但非 spec 钦定**。本文不改(留 P2 视觉对齐),实施者**不要误以为这是开放讨论项** |

---

## 设计意图回顾

### v1 → v2 心智模型差异

| 维度 | v1 lookup (今天) | v2 NLQ&A (目标) |
|---|---|---|
| 用户心智 | 「查一条事实」 | 「就这套房子聊天问任何东西」 |
| 答复尺度 | 单事实、单回合 | 多回合、开放式、含市场 / 建议视角 |
| 文案语气 | 被动系统化 (Looking up…) | 主动助手化 (Thinking…) |
| 兜底方式 | 「信息不在房源中」 | 「这角度回答不了 — 帮你换个角度?」 |

### v1 → v2 能力边界扩展

v1 lookup 时代**明确不做**的事(historical scope boundary):
- Comparing this listing to other properties
- Mortgage or affordability calculations
- Market trend analysis
- Agent recommendations
- Booking or scheduling actions

v2 把其中 **2 项从"能力存在但用户不可触达"转为"一级入口"**:

> **重要事实校正**: Comparing / Market context **不是 v2 新建的能力**。worker 端 data_fetcher 已经长期注册着 9 个 market / comparison 类 MCP 工具(覆盖 similar listings、nearby sold / for-sale / for-rent / leased listings、community market stats、municipality market overview、property analytics、neighbourhood demographics)。STP context 也把「附近相似房源最近成交价」这类问题列为 in-scope 高相关示例,eval 场景中持续有 comparables / market position 覆盖。**LLM 没有充分用是因为 v1 system prompt 写的是 "answer using listing data only" 把工具调用压着了** —— `v2-nlqa.md` 明确写了要解锁这块。

v2 的 2 项变化都是 **surface + prompt 层** 的:

- **Comparing** → chip 2 从 v1 的 "How long has it been listed?" 换成 v2 的 "How does it compare to nearby listings?"(对应 `chipCompare` key),把已有能力 promote 到一级入口
- **Market context** → System prompt 解锁推理(proposals 范围)+ Source Label `from_market_context`(proposals 范围)让用户看到答案"用了市场数据"

v2 **仍不做**的:Mortgage、Agent recommendations、Booking — 这些由 `safety_blocked` 错误码兜底并 redirect 回主题(`aiChat.errorSafetyBlocked` 已改语气)。

### Variant 探索史

详见 `v2-nlqa.md`。要点:Variant B(强套 base DS)已否决 — base DS 适配 listings / filters / forms 但不适配 chat,Chat Design System 在 chat surface 内压过 base DS。Variant C(Chat-First)是 canonical,拆分为 4A(全量 v2) + 4B(本文落地版)。

### 本文覆盖的子集

v2 心智模型升级的"**入口面**" — 用户在打开 sheet、看 hero、敲第一个问题、等回答的几秒钟内,所有视觉文案语气从 lookup 转 NLQ&A。这部分是 v2 心智模型生效与否的**第一印象层**,且完全由 web-hybrid 自主决定,不需要任何服务端字段或数据透传。

**信任层 / 对话延展层**(Source Label、Follow-up Chips、Abstain UI)在服务端字段就绪后单独立项落地,见 proposals。**Stage-aware 进度文案已端到端上线**(chat-service master commit `c8fc211`,在 `release/v3.3.0`),本文范围内只需 i18n 改名(`lookingUp` → `thinking`),改名后会作为 stage 文案到达前的 fallback。

---

## 12-state 状态机全景

State / Trigger / Key elements 完整定义见 `chat-design-system.md`。本表给出**每个 state 上本文 / proposals 谁负责**,避免来回翻:

| # | State | 本文(surface) | 相关 proposals |
|---|---|---|---|
| 1 | `trigger` | — | — |
| 2 | `hint` | 文案改 `askAboutThis` | — |
| 3 | `halfsnap` | hero 文案 + chips、placeholder | — |
| 4 | `entry_hero` | hero 文案 + chips、placeholder | — |
| 5 | `retrieving` | 加载文案 `thinking` / `stillThinking`(stage 文案到达前的 fallback) | Stage-aware 文案已在线上(master `c8fc211`)逐 stage 替换 fallback,本文不动 |
| 6 | `streaming` | — | — |
| 7 | `stopped` | — | — |
| 8 | `result` | — | proposals Source Label、Follow-up Chips |
| 9 | `show_more` | — | — |
| 10 | `abstained` | — | proposals Abstain UI、abstain redirect 语气 |
| 11 | `noresult` | errorGeneric 文案改值 | — |
| 12 | `error` | errorSafetyBlocked / errorGeneric 改值 | — |

**对实施的启示**:本文 P0 改动主要落在 halfsnap / entry_hero / retrieving / error 这几个 state 上。其余 state 在本文范围内**不动结构、不动视觉**,只有 hint / abstained / noresult / error 的部分文案随 i18n 改值。

---

## Component Composition Rules

详见 `chat-design-system.md`。本文对实施的启示:

- **Chat surface = sheet 容器内部**。sheet 内全部用 chat token(软圆角 16-24、pill 输入、圆形 send、`--muted` 灰底);sheet 外(app header、scrim 下 listing)跟 base DS
- **AI FAB 例外**:FAB 本身在 Bottom Action Bar,跟 base DS 与兄弟按钮 cohesion;FAB 触发的 hint popup 虽挂在 FAB wrapper 上,**算 chat surface**,用 chat token
- **base DS ≠ chat DS**(后者是 sister spec 不是 subset);Variant B 强套 base DS 已否决

---

## 改动总览

### 按区块视图

| 区块 | 影响文件 | 优先级 |
|---|---|---|
| i18n key 重命名 + 改值 | `en.ts` / `zh.ts` | P0 |
| Hero chip 重组 + headline 改名 | `AppAiChatHero.vue` | P0 |
| 入口 placeholder 改名 | `AppAiChatSheet.vue` | P0 |
| Coachmark 文案改名 | `AppAiHintPopup.vue` | P0 |
| 加载文案改名 | `AppAiChatMessages.vue` | P0 |
| 错误文案语气微调 | `en.ts` / `zh.ts` | P0 |
| 视觉对齐(单独立项) | 多个 | P2 |

### 按文件视图

本文实际触动的文件:

| 文件 | 改动 |
|---|---|
| `packages/common/i18n/translation/en.ts` 与 `zh.ts` 中 `aiChat` 命名空间 | 删 6 个旧 key、新增对应的 6 个重命名 key、调整 2 条错误文案的文案值 |
| `packages/app/src/pages/listing/components/AppAiChatHero.vue` | hero headline 渲染用的 i18n key 切换、chips 数据数组中两个非通用 chip 的 key 替换 |
| `packages/app/src/pages/listing/components/AppAiChatSheet.vue` | 透传给 composer 的 placeholder 文案的 i18n key 切换 |
| `packages/app/src/pages/listing/components/AppAiChatMessages.vue` | 加载状态文案(常规态 / 长等待态两段)的 i18n key 切换 |
| `packages/app/src/pages/listing/components/AppAiHintPopup.vue` | coachmark 标题渲染用的 i18n key 切换 |
| `packages/common/environments/types.ts` 与 `packages/hook/ai/useAiAssistant.ts` | JSDoc 注释中残留的旧 key 字面引用同步(纯文档串) |

**proposals 文档配套需改文件**(本文不动,见 proposals 各章节):`useChatStream.ts` / `useAiAssistant.ts` schema 扩展、`AppAiChatMessages.vue` 渲染 abstain marker / source label / follow-up chips、新建 `AppAiChatFollowUps.vue`、chat-service `orchestrator.service.ts` 透传、listing-chatbot `kafka_writer.py` payload 字段 + worker prompt。

---

## i18n 改动清单

**文件**: `packages/common/i18n/translation/en.ts` + `zh.ts` 的 `aiChat` 命名空间。namespace 下 key 由 5 个组件文件运行时消费,另由错误码映射函数(`useAiAssistant.ts`)用于把 chat-service 错误码翻成本地化字符串;还有两处 JSDoc 注释里有旧 key 名的字面引用。

**策略**: 重命名 + 改值单 PR 一次性做完,对齐 `v2-nlqa.md` 的 Copy Changes。`entryLabel` / `lookingUp` / `stillLooking` / `hintTitle` 带 lookup 语义,留下会持续误导,**不分阶段**。

### 重命名 + 改值(6 处)

| 旧 key | 新 key | EN | ZH | 同步消费点 |
|---|---|---|---|---|
| `entryLabel` | `askAnything` | Ask anything about this home | 问问这套房子的任何问题 | hero 标题渲染、sheet 透传给 composer 的 placeholder |
| `lookingUp` | `thinking` | Thinking | 思考中 | 消息列表 loading 常规态文案（typing-dots 气泡已自带 3 个动画 dot，文案不再带省略号） |
| `stillLooking` | `stillThinking` | Still thinking | 还在思考 | 消息列表 loading 长等待态文案（同上不带省略号） |
| `hintTitle` | `askAboutThis` | Ask about this home | 问问这套房子 | coachmark popup 标题 |
| `chipListed` | `chipCompare` | How does it compare to nearby listings? | 和附近房源比怎么样? | hero chips 数据数组中"对比类"chip |
| `chipSchools` | `chipCautions` | Anything I should be cautious about? | 有什么需要注意的地方吗? | hero chips 数据数组中"风险提示类"chip |

### 只改值(key 名保留)

少数 key 名本身没有问题,只调整文案语气:

| Key | 新 EN | 新 ZH |
|---|---|---|
| `errorSafetyBlocked` | Let's stay focused on this home — happy to help with anything about this listing. | 我们聚焦这套房子吧 — 关于这个房源的任何问题我都很乐意帮忙。 |
| `errorGeneric` | Something tripped me up. Try again? | 我这边出了点小问题,要不再试一次? |

`chipAbout`、`errorBusy`、`showMore`、`showLess` 已经足够口语化,**不动**。其他 `errorTtft` / `errorIdle` / `errorRound` / `errorTooLong` / `errorAuthExpired` / `errorCancelled` / `errorMcpError` / `errorInference` 语气调整可后续 sprint 跟进,**不阻塞本次 v2 改造**。

### 新增 key

无。设计文档原本列的 `noResultGeneric`、`abstainMarker`、`groundingFromListing/FromMarket/GeneralAdvice/Assumption`、`stillCurious` 等 key **均依赖服务端字段或数据源,移出本文范围**(见 proposals)。

### 操作清单(单次 PR)

`en.ts` / `zh.ts` 内 `aiChat` 命名空间:
1. 删 6 个旧 key、加 6 个重命名 key、填新 EN / ZH 值(见重命名 + 改值表)
2. 改 2 个 key 的值:`errorSafetyBlocked`、`errorGeneric`(见只改值表)
3. 同步运行时消费点(5 个组件 + 错误码映射函数)和 JSDoc 字面引用(`common/environments/types.ts` 加载文案 env 标志的 JSDoc、`useAiAssistant.ts` `progressLabel` 状态变量的注释块)

> **必须单 commit / 单 PR** — i18n 与消费点错位会让 UI 显示 raw key(如 `aiChat.entryLabel`)。建议 CI 加 i18n key 双向引用校验防呆。

---

## 组件级改动

### `AppAiChatHero.vue` — 入口 hero

**改动**:
- 模板中 hero 标题渲染所用的 i18n key 从 `entryLabel` 切到 `askAnything`,文案随 i18n 自动变化
- chips 数据数组中,"通用介绍"chip(`chipAbout`)保留;"上市时长"chip 换成"对比类"(`chipListed` → `chipCompare`);"学校"chip 换成"风险提示类"(`chipSchools` → `chipCautions`)

**不动的**:
- 头部 AI sparkle 图标(font-awesome `fa-sparkle`)— 是 Phosphor `star-four` 的 font-awesome 等效占位,**视觉对齐留到 P2 视觉对齐章节**
- scope-pill 结构、margin-top auto 让 chips 锚定到 hero 底部的布局
- 微视觉差(背景 `#e9f6f7` vs spec `--muted`、icon 字号 40 vs spec 32)留到 P2 视觉对齐 sprint

### `AppAiChatSheet.vue` — 容器

**P0**: composer 的 placeholder prop 由 sheet 透传,把 placeholder 字符串引用的 i18n key 从 `entryLabel` 切到 `askAnything`(末尾追加省略号的拼接逻辑不动)。

### `AppAiChatMessages.vue` — 消息列表(本文范围内只动加载文案)

**当前**: loading 文案有三段优先级 = `progressLabel`(由 env flag `listingChatbotProgressEvent` 控制,服务端 stage 文案到达时显示) > `stillLooking`(等待 ≥10s) > `lookingUp`(默认)。**结构不动**,只把:

- 长等待态使用的 i18n key 从 `stillLooking` 切到 `stillThinking`
- 默认态使用的 i18n key 从 `lookingUp` 切到 `thinking`

同步更新 `useAiAssistant.ts` 中 `progressLabel` 状态变量的注释块,把里面残留的 `aiChat.lookingUp` / `aiChat.stillLooking` 字面引用换成新 key 名(只是文档串,不影响运行,但避免误导后续读者)。同样的字面引用还出现在 `common/environments/types.ts` 中 env 标志的 JSDoc 中(见操作清单同步项)。

> `progressLabel` 这一段(env flag on + 服务端 stage 文案 emit 时显示)**保持原样不动**。端到端机制已在线上落地,各层证据如下:
>
> - **Worker**(listing-chatbot `main`):`src/listing_chatbot/agent/progress.py:51-62` 每 stage emit `ProgressEvent(stage, message, timestamp_ms)`,`message` 来自 `configuration/default.yaml` 的 stage→7 条变体配置(`intent_classification` → "Reading your brief…"、`scope_tool_planning` → "Pulling the right reports…"、`response_rendering` → "Typing up your summary…" 等);`worker/kafka_writer.py:130-159` 完整序列化 `{stage, message, timestamp_ms}` 写 Kafka
> - **Gateway**(realagent-services `master`,commit `c8fc211` "RR streaming passthrough — progress + reset + telemetry projection",2026-05-11,在 `release/v3.3.0` 已部署):`apps/chat-service/src/services/orchestrator.service.ts` 新增 `case 'progress'` envelope handler,经 `coerceProgressPayload` 校验 + `canWriteChunk` 背压 gate + `writeProgressChunk` sanitise(ANSI / BiDi / 零宽字符 / C0+C1 边界清洗)后,写到 SSE chunk 的 `progress` 字段(同 PR 还新增 `case 'reset'` / `case 'heartbeat'` 透传)
> - **Frontend**(web-hybrid `master`):`packages/hook/ai/useChatStream.ts:349-365, 418-422` 的 `ProgressFieldPayload { stage, message, timestamp_ms }` 类型 + `payload.progress` 分支消费;`packages/hook/ai/useAiAssistant.ts:237-241` `stream.onProgress` 回调把 `message` 替换式(非累积)赋值给 `progressLabel.value`;`AppAiChatMessages.vue:115-126` 三级 fallback:**flag on + progressLabel 非空 → stage 文案 verbatim;否则 loadingDuration ≥10s → `aiChat.stillThinking`;否则 → `aiChat.thinking`**(stage 文案一旦到过一次就 latch 住,不会回退到 stillThinking,因为 if 在前面拦截);env flag `listingChatbotProgressEvent` 默认 on
>
> 因此本文 i18n 改名(`lookingUp` → `thinking`,`stillLooking` → `stillThinking`)**只影响 stage 文案到达前的 fallback**(开头几百 ms 还没收到 worker 第一个 ProgressEvent 时,以及 stage 透传失败的极端情况下)。progressLabel 路径本身不动。

> Abstain marker / Source label / Follow-up chips 的渲染**全部不在本文范围**(见顶部"明确移出范围")。

### `AppAiChatComposer.vue` — composer

- stop 按钮 aria-label("Stop generating")— 保持(`v2-nlqa.md` 明确说)
- placeholder 通过 sheet 透传,sheet 端切换 i18n key 后,这里自动跟 `askAnything` 更新,本组件 **无需改动**
- **不动**结构、按钮形状、动画
- 视觉差异(背景 `#ededf0` vs spec `#F2F2F2`)留到 P2 视觉对齐

### `AppAiHintPopup.vue` — coachmark

文案改名:
- 把模板中 popup 标题渲染所用的 i18n key 从 `hintTitle` 切到 `askAboutThis`,文案随 i18n 自动更新

可见性硬化（同 PR 一并修复 popup 被高 z-index overlay 遮挡时静默 dismiss 的问题）:
- 自动关闭时长从 2 秒延长到 4 秒
- 引入 IntersectionObserver 监听 popup 自身可见性。`hint_displayed` GA 事件、`localStorage.hint_ai_chat_viewed='1'` 写入、4 秒倒计时**统一改成只有 IntersectionObserver 确认 ≥50% 可见后才触发**。
- popup 通过新增的 `seen` 事件通知 parent 设置 `aiHintViewed`,原有的 `close` 事件改为纯 UI 隐藏。被遮挡时 popup 静默留在 DOM,直到真正可见或路由切换。one-shot 曝光窗口在被遮挡情形下得到保护,用户下次进 listing 仍有机会看到。
- 保留: 定位、模板结构、AI FAB 点击路径下的 dismiss 仍同时清掉 popup + 写 viewed=1(用户主动开聊是更强的"已感知"信号)

> **挂载位置提示**: `AppAiHintPopup` 不挂在 sheet 内,而是挂在 `AppListingWatchActions.vue`(listing 详情页的 Bottom Action Bar 上,作为 AI FAB 的 sibling)。本节改动只动 popup 组件本身和 parent 的事件 handler,sheet 内没有 hint 引用。`hint_ai_chat_viewed` 这个 localStorage 标志位的读写发生在 `AppListingWatchActions.vue` 内。

---

## GA 事件

| 事件名 | 状态 | 触发 | hs_label |
|---|---|---|---|
| `ai_chat_open` | 已有 | sheet 打开 | source |
| `ai_chat_chip_click` | 已有 | hero chip 点击 | chip 文案(新文案后自动跟随) |
| `ai_chat_send` | 已有 | 提交输入 | — |
| `ai_chat_stop` | 已有 | 点 stop | — |

> 原计划新增的 `ai_chat_follow_up_click`、`ai_chat_grounding_seen` 依赖服务端字段,移出本文。

---

## 落地优先级

### P0 — 当前 sprint 即可发,零服务端依赖

1. **i18n key 重命名 + 改值**(单次 PR):删 6 个旧 key、加 6 个新 key、改 2 个 key 的值、运行时消费点 + JSDoc 字面引用同步
2. **Hero 3 chip 数组更新**:`chipListed` → `chipCompare`、`chipSchools` → `chipCautions`
3. **加载状态文案引用切换**:`lookingUp` / `stillLooking` → `thinking` / `stillThinking`

> P0 必须**单次 PR**完成,避免 i18n 与消费点错位。改动全部是字符串 + key 名,**不需要 feature flag**,改完就发。GA `ai_chat_chip_click` 可以观察到新 chip 文案的点击分布,作为 v2 心智模型生效与否的早期信号。

### P2 — 视情况再做

4. **视觉差异修复**(bubble 主圆角、bubble 尾角、scope-pill 背景、Lucide 图标体系)— 视觉对齐通常单独立项

### 移出范围(等服务端配合后单独立项)

- Abstain UI 启用 — 等 chat-service `writeDoneEvent` 透传 `abstained` / `null_reason`
- Source / Grounding Label — 等 worker 产出 `grounding_type` + chat-service 透传
- Follow-up Suggestion Chips — 等 worker 产出 `follow_ups: string[]` + chat-service 透传
- ~~Stage-aware 进度文案~~ **已上线** — chat-service master commit `c8fc211`(2026-05-11)合入 `case 'progress'` 透传,在 `release/v3.3.0`(2026-05-13)已部署;worker `agent/progress.py` + `configuration/default.yaml` 的 stage→文案配置、前端 `useChatStream.ts` 的 `payload.progress` 消费分支均早已就位;env flag `listingChatbotProgressEvent` 默认 on。本文 i18n `lookingUp` → `thinking` / `stillLooking` → `stillThinking` 改名只是 stage 文案到达前的 fallback,与 stage-aware 路径独立
- Idle Re-engagement Prompt — 依赖 `follow_ups` 数据源
- Abstain 文案换成「换个角度」语气 — 等 worker system prompt 改写

---

## 视觉对齐(随同 PR 一并修复,参 Frame 4B spec)

本次 PR 在 i18n 改动基础上,同步把 Frame 4B 设计稿的几项关键视觉差异落了:

| 项 | 改前 | 改后 / Spec |
|---|---|---|
| Hero chips 布局 | `flex-wrap: wrap` 横向 wrap | `flex-direction: column; align-items: center` 垂直堆叠(spec layout: vertical) |
| Hero chips gap | 8px | 12px |
| Hero chip cornerRadius | 20px | 22px |
| Hero chip padding | 8 / 14 | 8 / 16 |
| Hero icon size | 40px | 32px |
| Bubble 主圆角 | 16px | 20px |
| User bubble 尾角(`border-bottom-right-radius`) | 2px | 4px |
| AI bubble 尾角(`border-bottom-left-radius`) | 2px | 4px |
| Loading bubble 圆角/尾角 | 16 / 2 | 20 / 4 |

Chip 横向 wrap 在长文案 chip 下视觉散乱,垂直堆叠也是 spec 明确意图(`chipsSection.layout: vertical`)。Bubble 圆角/尾角向 spec 对齐,与设计稿一致。

## 未覆盖的视觉差异(后续 P2 单独立项)

| 项 | 当前 | spec |
|---|---|---|
| Hero scope-pill 背景 | `#e9f6f7`(hero 组件 SCSS) | `$A:--bg-cyan-light` 变量(色调相近,变量化是单独工作) |
| Hero scope-pill 内 pin 图标 | font-awesome `fa-location-dot` 直接铺平 | spec 包裹一层 22×22 cyan 圆形背景 + 白色 pin 图标 |
| AI sparkle 图标 | FontAwesome `fa-sparkle` | Phosphor `star-four`(chat-DS 钦定;迁图标字体集是依赖工作) |
| 通用图标体系 | FontAwesome | Lucide(引入是单独依赖工作) |

对 v2 心智模型**不构成阻塞**,后续视觉对齐 sprint 单独跟进。Composer / Hint popup 圆角 1-3px 极轻微差异已剔除。

---

## 验收要点

| 阶段 | 用户可见的变化 | 观测信号 |
|---|---|---|
| P0 ship 后 | 首次进入看到 "Ask anything about this home" / "问问这套房子的任何问题"; chip 变成 1 通用 + 1 对比 + 1 谨慎; loading 文字变 "Thinking"（typing-dots 气泡自带 3 个动画 dot，文案不带省略号） | GA `ai_chat_chip_click` 出现 `chipCompare` / `chipCautions` 的点击 |

---

## Desktop Variant

Same v2 NLQ&A positioning shift,**容器形态**不同 —— desktop 改用浮动 popover 而不是 bottom sheet。业务层(`useAiAssistant` / `useChatStream`)/i18n/chat-stream 协议全部复用,无双轨。详见 `chat-design-system.md` §5.16。

### Surface 拓扑

| Layer | Mobile (`AppAiChatSheet`) | Desktop (`PcAiChat`) |
|---|---|---|
| Entry | host 页 `AppListingWatchActions` 内的 AI 按钮 + 首次 `AppAiHintPopup` coachmark | 视口右下角 fixed FAB (`PcAiChatFab` 嵌在 `PcAiChat` 内部) |
| Container | 全屏 modal bottom sheet,top corners `chat-radius-lg`,snap = half / full,带 drag bar + scrim | 400×600 floating popover,四角全 `chat-radius-md`,固定尺寸,**无 drag bar / 无 snap / 无 scrim**,只靠 `0 12 40 / 22%` shadow 浮起来 |
| Header chrome | 无独立 header(sheet 整页占满,hero 自带 title 区) | 有 header — hero entry 极简(只右上 close),messages 状态 chrome 化(scope pill 左 + close 右 + 1px divider) |
| Close | 拉拽 sheet 到阈值 / 点 scrim / 系统返回 | 右上 `×` button / ESC key |
| z-index | sheet (789) over scrim (788) | popover (1000),FAB (1000) — 二者互斥呈现 |
| Safe-area-inset | composer 加 `env(safe-area-inset-bottom)` | N/A |

### 12-state 映射

复用 `chat-design-system.md` §5.16 表(state 1 / 4 / 5 / 6 / 8 / 9 / 10 / 11 / 12 一致;state 3 `halfsnap` 在 desktop 上**不存在** —— popover 直接打开到 entry hero;state 2 `hint` 在 desktop MVP **暂不做**,因为右下角 FAB 在 desktop 屏幕上足够显眼)。

### 触发的设计决策

1. **Scope pill 漂移** — hero entry 时 pill 留在 hero block 内(沿用 mobile 布局);messages 状态时 pill 上移到 header 作为对话期上下文锚,因为 popover 浮在仍可见的 listing 页面上,但 hero 已经隐去 —— 没 header anchor 用户会"忘记在跟哪套房聊"。Mobile 不需要这一步,sheet 全屏盖住 listing 时上下文是封闭的。

2. **AI bubble max-width** — code 层 `.ai-row .message-group max-width: 90%`,与 mobile 等同;popover 内 messages area 是 368 px(panel 400 − padding 16×2),AI bubble 上限 ≈ 322 px(90%)。

3. **Header brand symbol 去除** — 早期试过 outline `star-four + AI` 文字做品牌 mark,但与 hero h2 / scope pill 信息重叠,最终选 chrome-only(只有 close),sparkle 完全交给 hero icon 表达。

### 文件清单

新增(`packages/desktop/src/pages/listing/components/`):
- `PcAiChat.vue` — 根组件:floating panel 容器 + FAB toggle + `useAiAssistant` 接入 + ESC dismiss
- `PcAiChatHero.vue` — entry hero(40px sparkle 图标 + h2 + scope pill + 3 chips,垂直布局)
- `PcAiChatMessages.vue` — 消息列表 + 加载指示 + markdown 渲染 + collapse / Show more
- `PcAiChatComposer.vue` — 输入条 + send / stop 切换

修改(`packages/desktop/src/pages/listing/`):
- `PcListing.vue` — 在 `<template v-if="house">` 块尾挂 `<PcAiChat>`,gate `listing?.resp?.component_display?.chatbot === 1`

i18n:
- 全部复用 mobile 已对齐的 `aiChat.*` namespace(`askAnything` / `chipCompare` / `chipCautions` / `thinking` / `stillThinking` / `errorGeneric` / `errorSafetyBlocked` 等),无新增 key

业务 hook(共享,无改动):
- `packages/hook/ai/useAiAssistant.ts`
- `packages/hook/ai/useChatStream.ts`

### 设计稿对应

- `listing-chatbot-design.pen` **Frame 4 — Desktop Chatbot (PcAiChat)**(node `Q5s2r0`)— 本节落地版,展示 FAB / hero entry / messages 三个关键状态及 spec 注脚
- `chat-system-design.pen` **Desktop — Floating Popover Variant**(node `g4phu`)— chat DS 画布的 desktop 容器规范,与 §5.16 配套

### 移出 desktop MVP 范围

| 项 | 状态 |
|---|---|
| Coachmark hint popup | 不做(desktop 屏幕大,FAB 直接可见) |
| Sheet snap / drag bar / 安全区适配 | N/A(popover 不需要)|
| `hint_ai_chat_viewed` localStorage 写入 | 不做(无 coachmark)|

### GA 事件

`ai_chat_open` 的 `hs_label` desktop 上发 `"desktop_fab"`(mobile 是 `entry_bar` / `bottom_bar`),其他事件(`ai_chat_chip_click` / `ai_chat_send` / `ai_chat_stop`)与 mobile 完全一致。

---

## 参考

### 设计(同目录)
- `v2-nlqa.md` — **v2 NLQ&A 设计意图 + 提案**:positioning shift、Variant 探索史、Design Principles、Copy 变更总表、新增 / 复活 UI patterns、System Prompt direction、Rollout、Risk
- `chat-design-system.md` — **Chat Design System**(sister spec to base DS,chat surface 视觉 / 组件 / token 权威),listing chatbot 是其首个落地 surface
- `listing-chatbot-design.pen` Frame 4B(node `c565Q`)— **mobile 落地版设计稿**:12 状态屏沿用,只更新 surface 范围内的文案
- `listing-chatbot-design.pen` Frame 4 — Desktop Chatbot(node `Q5s2r0`)— **desktop 落地版设计稿**:FAB / hero entry / messages 三个关键状态 + spec 注脚
- `listing-chatbot-design.pen` Frame 4A(node `c7EJq`)— **全量 v2 设计稿**:12 状态屏含 abstain marker、Source Label、Follow-up Chips、stage-aware loading 文案等新组件渲染(见 proposals)
- `chat-system-design.pen` Desktop — Floating Popover Variant(node `g4phu`)— chat DS 画布的 desktop 容器规范
- `chat-system-design.pen` — chat DS 画布(Variant C 12 状态可复用版),含 reusable component library

### web-hybrid 源码

**Mobile (`packages/app/src/pages/listing/components/`)**:
- `AppAiChatSheet.vue` — 容器、snap、状态管理
- `AppAiChatHero.vue` — 入口 hero、3 chip
- `AppAiChatMessages.vue` — 消息列表、loading、source label hook、collapse
- `AppAiChatComposer.vue` — 输入条、send/stop
- `AppAiHintPopup.vue` — 首次进入 coachmark

**Mobile 挂载点**:
- `packages/app/src/components/AppListingWatchActions.vue` — AI FAB 入口
- `packages/app/src/pages/listing/listing.vue` — sheet 挂载

**Desktop (`packages/desktop/src/pages/listing/components/`)**:
- `PcAiChat.vue` — 浮动 popover 容器 + FAB toggle + `useAiAssistant` 接入
- `PcAiChatHero.vue` — 入口 hero(同 mobile 结构,size/spacing 适配 400px panel)
- `PcAiChatMessages.vue` — 消息列表(逻辑与 mobile 一致)
- `PcAiChatComposer.vue` — 输入条(无 safe-area-inset)

**Desktop 挂载点**:
- `packages/desktop/src/pages/listing/PcListing.vue` — popover 挂载

**共享(两端都用)**:
- `packages/hook/ai/useAiAssistant.ts` — message 状态机、错误码映射、`stop`、`setMessages`
- `packages/hook/ai/useChatStream.ts` — 原始 SSE 解析、`DonePayload` / `ChunkPayload`、watchdog
- `packages/common/i18n/translation/{en,zh}.ts` — `aiChat.*` namespace

### 配套
- [`v2-nlqa-proposals.md`](./v2-nlqa-proposals.md) — 服务端依赖项提案存档
