# Design Decisions — Listing Chatbot

设计变迁记录，按迭代顺序追溯每个关键决策的触发原因与最终结论。

---

## 起点：初始设计假设

- 对话线程作为初始状态（无引导界面）
- Bot avatar 标识 AI 身份
- Context header 显示地址 + 关闭按钮
- 聊天区从 greeting 开始

---

## 关键决策节点

### 决策 1 — 确立核心原则：Chat as Interface, Retrieval as Behavior

**触发**：讨论"行为上查找、交互上聊天是否矛盾"

**结论**：两者是不同层——Form（chat UI）vs Semantics（retrieval）。不矛盾，但有张力需要在 copy 层面管理。产品形态是 chat，但语义是 look up，不是 ask for help。

---

### 决策 2 — 移除 Bot Avatar

**触发**：视觉审查

**结论**：avatar 强化了"AI 助手"感，与 retrieval 定位冲突。移除后气泡排版更紧凑，左侧 padding 随之归零。

---

### 决策 3 — 入口引入 Hero Section（选方案 B）

**触发**：初始状态空白，缺乏引导感

**探索**：
- 方案 A：搜索框启动器（通用搜索入口感）
- 方案 B：房源锚定——地址 + 城市 + 副标题

**结论**：方案 B 建立房源上下文，强化"scoped to this listing"的定位感。

---

### 决策 4 — 移除 Context Header，改用 Drag Indicator

**触发**：底部 sheet 是嵌入式的，header 里的地址与 hero 里的地址重复

**结论**：嵌入式 sheet 不需要关闭按钮和独立 header；drag indicator 足以传达"可关闭"；地址保留在 hero 中提供上下文即可。

---

### 决策 5 — 加入 Star-Four 符号

**触发**：希望建立产品识别符号，与纯文字 UI 区分

**结论**：star-four（✦）成为跨状态的身份标识，出现在入口 hero 和对话气泡提示中，形成视觉一致性。

---

### 决策 6 — 统一四个状态的视觉语言

**触发**：对话状态（result / retrieving / no-result）仍沿用旧版 full-height sheet，与 init-v2b 不一致

**结论**：全部对齐 init-v2b 规范：
- teal 背景满高
- 白色 sheet 从 y:70 开始
- drag indicator
- 无 context header
- input bar 无 shadow

---

### 决策 7 — 对话状态移除 Greeting

**触发**：lookup_result 等对话状态里仍有 "What would you like to look up?" greeting

**结论**：对话状态从用户第一条消息开始，不显示 greeting。Greeting 只属于 entry state，以 hero subtitle 的形式保留。

---

### 决策 8 — 副标题 vs Chips Section Label 冲突

**触发**："Look up any detail from this listing" 和 "Suggested questions" 同时出现，引导意图重复

**结论**：保留 subtitle（用于范围框定），删除 "Suggested questions" 标签。Chips 本身已足够表意，不需要额外 label。

---

### 决策 9 — 文案统一："Look up details in this listing"

**触发**：subtitle 和 placeholder 用词不一致；"anything" 暗示无限能力，与 retrieval 定位不符

**演变路径**：
```
"Look up any detail"
  → "Look up anything"
  → "Look up details"
```

**结论**：`details` 弱化了宽泛感，与 retrieval 原则一致。subtitle 和 placeholder 统一使用同一核心短语 "Look up details in this listing"。

---

### 决策 10 — Entry Hero 层级反转：Action Intent 优先于 Address

**触发**：反思"地址作为主标题"是否合理——用户进入嵌入式 sheet 时已知当前房源

**结论**：入口应首先传达"工具能做什么"而非"你在哪里"。

最终层级：
- 主标题（20px semibold）：**"Look up details in this listing"**
- 作用域标签（13px 灰色）：地址
- Chips：建议问题
- Input bar

---

## Open Questions

### OQ-1 — 入口按钮的可发现性

**问题**：房源详情页上触发底部 sheet 的入口按钮，用户可能不容易注意到。

**背景**：当前设计中，入口是嵌入在详情页内的一个触发区域。用户如果没有主动滚动到该区域，或者视觉上没被吸引过去，可能完全不知道这个功能存在。

**待解决的方向**：
- 入口是否需要更强的视觉权重（颜色、动效、固定位置）？
- 是否考虑 sticky / floating 入口，始终可见？
- 如何在不干扰详情页主体内容的前提下提高曝光？

---

## 最终设计状态

### 四个状态命名
| 状态 | 说明 |
|------|------|
| `lookup_entry_hero` | 入口，无对话历史 |
| `lookup_result` | 有回答的对话状态 |
| `lookup_retrieving` | 检索中，含前一轮历史 |
| `lookup_noresult` | 无匹配结果 |

### 核心视觉规范
- 背景：teal 满高
- Sheet：白色，从 y:70 开始
- 顶部：drag indicator，无 context header，无 shadow
- Input bar：无 shadow

### Entry 层级（从上到下）
1. Action intent — "Look up details in this listing"（主标题）
2. Scope label — 地址（灰色小字）
3. Suggested chips
4. Input bar

### 对话状态原则
- 从用户第一条消息开始，无 greeting
- Retrieving 状态包含前一轮历史对话，体现多轮上下文

### 文案系统
- 核心短语：**"look up details"**
- 避免使用：help / suggest / anything
- subtitle 与 placeholder 保持一致
