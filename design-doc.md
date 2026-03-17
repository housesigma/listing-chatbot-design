# HouseSigma AI Chat — Design Document

> Version: MVP · Last updated: 2026-03

---

## 1. Design Brief

### Problem

When a buyer is browsing a listing detail page, specific questions come up naturally — HOA fees, basement dimensions, parking setup — but finding the answer requires hunting through dense listing copy or leaving the page to contact an agent. There is no fast, in-context path to get a factual answer without friction.

### Design Intent

> Give buyers an instant, trustworthy way to retrieve facts from a listing — without creating the expectation that AI can do more than the data supports.

### Core Principle: Retrieval, Not Conversation

This could have been a general-purpose AI assistant embedded in the app. It is not. The product is deliberately scoped to a **single listing, factual retrieval only**. This choice was made for two reasons:

1. **Trust** — A scoped tool that cites listing data is verifiably accurate. A general assistant that speculates about affordability or market trends can mislead buyers on high-stakes decisions.
2. **Expectation management** — Overselling AI capability and then failing to deliver (e.g., "I can't help with that") erodes trust faster than a narrower tool done well.

This principle is never stated to the user explicitly. It is expressed entirely through copy and interaction design (see below).

### How the Principle Is Expressed in Copy

| Surface | Copy | Why |
|---------|------|-----|
| AI greeting | "What would you like to **look up** about this property?" | "Look up" signals retrieval, not assistance |
| Input placeholder | "Look up anything in this listing..." | Reinforces the data boundary without disclaimers |
| Quick reply chips | "Bedrooms & bathrooms?", "Year built?", etc. | Demonstrate scope by example, not by exclusion |

Words intentionally avoided: _help_, _suggest_, _recommend_, _anything you want_ — these imply a broader capability that does not exist.

### MVP Scope Boundary

Out of scope for this release:

- Comparing this listing to other properties
- Mortgage or affordability calculations
- Market trend analysis
- Agent recommendations
- Booking or scheduling actions

---

## 2. Design Decisions

### 1. Conversation thread as the initial state

**Decision:** The sheet opens directly into a conversation thread with the AI greeting as the first message — no splash screen, no logo, no search bar.

**Alternatives considered:**
- *Centered hero layout* (logo + tagline + input field): common in AI product onboarding, but frames the interaction as a new product to learn rather than a tool already in context.
- *Search bar style*: implies keyword lookup, not natural language questions.

**Rationale:** Starting in a live conversation thread signals the interaction model immediately. The user does not need to read instructions — the greeting *is* the instruction. The empty space below also doubles as a scroll buffer as the conversation grows.

---

### 2. Quick reply chips anchored above the composer

**Decision:** Suggested question chips sit directly above the input bar and are removed after the first message is sent.

**Alternatives considered:**
- *Chips in the conversation body* (e.g., Google Gemini style): chips become part of the thread, which works for open-ended agents but clutters a retrieval interface.
- *No chips at all*: increases cold-start friction; users unfamiliar with the product don't know what to ask.

**Rationale:** Anchoring chips above the composer creates a spatial relationship between "suggested question" and "place to type it." Removing them after first use prevents persistent chip rows from competing with the actual conversation.

---

### 3. Scope communicated implicitly, not through a disclaimer

**Decision:** No upfront message like "I can only answer questions about this listing." Scope is inferred through copy and UI context alone.

**Alternatives considered:**
- *Explicit disclaimer on open*: "I can only answer questions based on the listing data for [address]." Honest, but creates a defensive first impression and positions the product as limited before the user has tried anything.
- *Hard error on out-of-scope queries*: "I can't answer that." Technically accurate but abrupt and unhelpful.

**Rationale:** Three design layers work together to set the right expectation without a disclaimer: (1) the verb "look up" in the greeting and placeholder, (2) chip labels that reference concrete listing fields, (3) the context header always showing the listing address. When the AI has no data for a query, it responds constructively — redirecting to what it *can* answer rather than surfacing an error.

---

### 4. Bot avatar shown only on the first message per response group

**Decision:** When the AI sends multiple bubbles in sequence, the avatar appears only on the first. Subsequent bubbles are offset 40px left to maintain column alignment.

**Alternatives considered:**
- *Avatar on every bubble*: creates heavy visual repetition in dense threads (similar to why Slack and iMessage suppress repeated avatars in group chats).

**Rationale:** Reduces visual noise at scale while preserving the column alignment that makes consecutive messages feel like a single response. Follows established convention (iMessage, Slack, WhatsApp).

---

### 5. Source attribution: informal in MVP, explicit in V2

**Decision:** MVP responses cite data informally in prose ("Based on the listing, HOA fees are..."). No tappable source link.

**Alternatives considered:**
- *Inline citations in MVP*: "HOA fees: $345/month ↗ [listing data]" — adds trust but increases response complexity and requires additional UI for the link target.
- *No attribution at all*: simplest, but responses feel unanchored; users cannot distinguish AI-inferred content from listing-stated facts.

**Rationale:** Informal citation ("Based on the listing...") achieves the primary trust goal — the answer is grounded — without the engineering overhead of linking citations to specific listing fields. Tappable attribution is planned for V2 once the response schema is stable.

---

### 6. Long responses folded by default

**Decision:** AI responses exceeding a height threshold are clipped with a "Show more" expand control.

**Alternatives considered:**
- *Unconstrained bubble height*: single long response can push the composer bar far off-screen, breaking the interaction loop.
- *Paginated responses*: splits a single answer into steps, adds unnecessary complexity for factual retrieval.

**Rationale:** Keeps the composer bar reachable after any response. Especially important for multi-room property descriptions where the AI may enumerate many details. The fold is a display affordance only — the full response is always one tap away.

---

## 3. States

| State | When | Screen Name |
|-------|------|-------------|
| Initial | No conversation yet | 终极方案 |
| Active Conversation | 1+ message exchanged | 终极方案_对话中 |
| AI Generating | Awaiting AI response | 终极方案_生成中 |
| No Result | AI cannot find the information | 终极方案_错误 |
| Long Content (folded) | Response exceeds height threshold | 终极方案_长内容折叠 |

---

## 4. Interaction Specification

### Entry Point

The AI chat is triggered from the listing detail page via a persistent entry button (e.g., a floating action button or inline CTA labeled "Ask about this listing"). Tapping the button opens the AI chat as a **bottom sheet** sliding up over the listing page. The listing page header remains partially visible above the sheet.

### Sheet Behavior

| Property | Value |
|----------|-------|
| Sheet type | Bottom Sheet (modal) |
| Sheet height | ~580px (leaves ~264px of listing page visible above) |
| Top corners | Rounded (20px radius) |
| Dismiss | Tap the × button in the Context Header, or drag the Drag Indicator downward |
| Entry animation | Slide up from bottom (standard sheet transition) |
| Exit animation | Slide down |

---

### State 1 — Initial State

**Trigger:** Sheet opens for the first time (no conversation history).

**What the user sees:**
- Drag Indicator at top
- Context Header showing the listing address ("167 Tweedsdale Cres · Oakville") and a dismiss (×) button
- One AI greeting bubble: *"What would you like to look up about this property?"*
- "Suggested questions" label and 4 quick reply chips (Bedrooms & bathrooms? / Square footage? / Year built? / Parking & garage?)
- Empty conversation area between the greeting and the chips
- Composer Bar with placeholder: *"Look up anything in this listing..."*

**Interactions:**
- Tap a chip → chip text is sent as a user message → transitions to **State 2 (Conversation)** + immediately triggers AI response → transitions to **State 3 (Generating)**
- Tap the text field → keyboard appears, user types freely → tap send → same flow as chip tap
- Tap × or drag down → sheet dismisses

**Key detail:** The quick reply chips do NOT persist after the first message is sent. They are shown only in the initial state.

---

### State 2 — Active Conversation

**Trigger:** At least one message has been exchanged.

**What the user sees:**
- Conversation thread showing all messages in chronological order (oldest at top)
- AI message bubbles: left-aligned, gray background, with bot avatar on the **first** AI message of each response group
- User message bubbles: right-aligned, teal background, no avatar
- Consecutive AI messages within the same turn: **bot avatar is hidden** on the 2nd+ message; left offset (40px) maintains column alignment
- Composer Bar with placeholder: *"Look up anything in this listing..."*
- Suggested Prompts section is gone

**Interactions:**
- User types in the Composer Bar and taps Send → message appears as a User Turn → AI starts generating → transitions to **State 3 (Generating)**
- Tap × → sheet dismisses (conversation history retained within the session)

**Key detail — Avatar rule:** The bot avatar appears only once per AI response group (i.e., the first bubble). If the AI sends multiple bubbles in sequence (e.g., a short acknowledgment + a detailed answer), only the first shows the avatar. This prevents avatar repetition in dense threads.

---

### State 3 — AI Generating

**Trigger:** User sends a message; AI response is streaming.

**What the user sees:**
- All previous messages remain visible
- A new AI message row appears with the bot avatar and a **Streaming Indicator**: three animated dots + label "Generating answer..."
- Composer Bar is **disabled** (input opacity reduced, field non-interactive)
- Send button replaced by **Interrupt Action** button (white square stop icon on teal circle)

**Interactions:**
- Wait → AI response streams in → transitions to **State 2 (Conversation)**
- Tap the Stop button → generation is interrupted → partial response (if any) is discarded or shown → transitions back to **State 2 (Conversation)** with the Composer Bar re-enabled

**Key detail:** "Generating answer..." uses #6B7280 on #F4F5F7 (~4.7:1 contrast ratio) to ensure readability while maintaining a subdued visual tone.

---

### State 4 — No Result

**Trigger:** AI cannot find the requested information in the listing data.

**What the user sees:**
- The AI responds with a message in the same gray bubble style as a normal response — **no error styling, no warning icons**
- Response copy is constructive: names what is unavailable and redirects to what the AI *can* answer (e.g., *"School district information isn't included in the listing. Try asking about bedrooms, bathrooms, square footage, or HOA fees."*)

**Key detail:** No red/warning styling because the AI is functioning correctly — it simply has no data for that query. Alarming the user would be a false negative.

---

### State 5 — Long Content (Folded)

**Trigger:** AI response text exceeds the defined height threshold of the message bubble.

**What the user sees:**
- The AI bubble is clipped at a fixed height (the text is cut off mid-sentence)
- A **"Show more"** expand link (teal, 13px) appears immediately below the bubble
- The rest of the conversation thread continues normally below

**Interactions:**
- Tap "Show more" → bubble expands to show the full response text → "Show more" is replaced by "Show less" (or link is removed)

**Key detail:** The fold prevents a single long answer from pushing the Composer Bar far out of view. This is especially important for questions about multi-room properties where the AI may list many room details.

---

### Composer Bar Details

| Element | Default State | Generating State |
|---------|--------------|-----------------|
| Text field | Interactive, placeholder shown | Disabled (opacity reduced) |
| Placeholder | "Look up anything in this listing..." | "Look up anything in this listing..." |
| Action button | Send (teal circle, arrow icon) | Stop (teal circle, square icon) |
| Send button | Active when text is entered | N/A |

### Copy & Tone Notes

- **Tone:** Informative, not conversational. The AI does not use filler phrases ("Sure!", "Great question!"). It goes directly to the answer.
- **Scope hint:** The greeting uses "look up" (retrieval verb) rather than "help you" or "answer anything". This creates an implicit expectation boundary without requiring an explicit disclaimer.
- **No hallucination safeguard note:** Since the AI is constrained to listing data, responses should cite "Based on the listing..." to anchor the answer in a verifiable source. This is enforced at the model/prompt layer, not the UI layer.

---

## 5. Content Specification

### UI Copy

#### Persistent Strings

| Location | String | Type | Notes |
|----------|--------|------|-------|
| Context header | 167 Tweedsdale Cres · Oakville | Dynamic | Bound to current listing address |
| AI greeting | What would you like to look up about this property? | Static | Shown in all states as the first message |
| Input placeholder | Look up anything in this listing... | Static | All input states (active, disabled) |
| Streaming label | Generating answer... | Static | Generating state only |
| Fold expand | Show more | Static | Long content state only |
| Chips section label | Suggested questions | Static | Initial state only |

#### Quick Reply Chips

| # | Label |
|---|-------|
| 1 | Bedrooms & bathrooms? |
| 2 | Square footage? |
| 3 | Year built? |
| 4 | Parking & garage? |

> Note: Chips are static in MVP. V2 can generate these dynamically from listing data (e.g., hide "Parking & garage?" if the listing has no parking).

#### Example AI Responses (for engineering reference)

| Scenario | Response |
|----------|----------|
| Information found | Based on the listing, HOA fees are $345/month, covering maintenance, snow removal, and building insurance. |
| Information not found | School district information isn't included in the listing. Try asking about bedrooms, bathrooms, square footage, or HOA fees. |
| Long response (folded) | This property features hardwood floors on the main level, updated kitchen with quartz countertops and stainless steel appliances... *(truncated)* |

---

### Icons

| Icon Name | Library | Location | Purpose |
|-----------|---------|----------|---------|
| `map-pin` | Lucide | Context header | Listing location indicator |
| `x` | Lucide | Context header | Dismiss / close sheet |
| `star-four` | Phosphor Bold | Bot avatar | AI identity mark |
| `send` | Lucide | Composer bar | Send message |
| Square (filled rect) | Custom | Composer bar | Stop / interrupt generation |

---

### Color Tokens

| Name | Hex | Usage |
|------|-----|-------|
| Brand Teal | `#28A3B3` | Bot avatar, user bubble, send button, teal chip border, links |
| Teal Surface | `#E9F6F7` | Context header chip background |
| Bot Bubble | `#F4F5F7` | AI message bubble background |
| Divider | `#F0F0F0` | Header bottom border |
| Input Surface | `#F7F7FC` | Text input field background |
| Placeholder Text | `#BBBBBB` | Input placeholder |
| Body Text | `#333333` | Message content |
| Subtext | `#999999` | "Suggested questions" label |
| Streaming Text | `#6B7280` | "Generating answer..." label |
| Icon Gray | `#808080` | Dismiss icon |

---

### Accessibility Notes

| Element | Contrast | Standard | Status |
|---------|----------|----------|--------|
| Streaming label (#6B7280 on #F4F5F7) | ~4.7:1 | WCAG AA (4.5:1) | ✅ Pass |
| Placeholder (#BBBBBB on white) | ~3.1:1 | Exempt for placeholder per WCAG 1.4.3 | ⚠️ Acceptable |
| Body text (#333333 on white) | ~10.7:1 | WCAG AA | ✅ Pass |
| User bubble (white on #28A3B3) | ~4.6:1 | WCAG AA | ✅ Pass |

---

## 6. Iteration Roadmap

### Current MVP Coverage

- 5 个核心状态：Initial / Conversation / Generating / No Result / Long Content
- 底部 sheet 入口，从 listing detail 页滑出
- Quick reply chips（静态，首次进入展示）
- 流式生成 + Stop 中断操作
- 长内容折叠（Show more / Show less）
- No result 状态：无错误样式，引导用户换问法

---

### V1.1 — Quick Wins

#### 1. Answer Source Attribution
**Background:** AI answers are entirely based on listing data; anchoring credibility is important.
**Proposal:** Append a source reference to each AI response:
> *Based on the listing · MLS #N5310389*

Or inline field attribution:
> "The property has **5+1 bedrooms** *(listing data)* and **6 bathrooms** *(listing data)*."

**Value:** Reduces user skepticism about AI accuracy; aligns with "no hallucination" product promise.

#### 2. AI FAB Tooltip on First Visit
**Background:** The ✦ icon in the bottom bar has low discoverability.
**Proposal:** On first entry to a listing page, show a tooltip bubble above the FAB: *"Ask about this listing"*, auto-dismissing after 1.5s.

#### 3. Dynamic Quick Reply Chips
**Background:** Current 4 chips are static and unrelated to listing data.
**Proposal:** Generate dynamically from listing data, e.g.:
- No parking data → hide "Parking & garage?"
- Has HOA → show "HOA fees?"
- Has basement → show "Basement details?"

**Value:** Reduces frustration from irrelevant suggestions; improves first-interaction conversion.

#### 5. Entry Bar — 嵌入 Listing Details（Proposed）

**背景：** 当前入口为 ✦ FAB（浮动图标），可发现性依赖用户主动注意——如果用户视线没落到图标上，可能完全不知道该功能存在（见 design-decisions.md OQ-1）。

**提议：** 将 ✦ star-four 入口替换为一个**直接嵌入 listing details 内容流的 entry bar**。该 bar 作为详情页的一个内联 section，用户在正常滚动浏览房源信息时自然经过。

**Entry bar 设计：**
- 全宽卡片，包含 ✦ 图标 + "Look up details in this listing" 文案 + 可点击的输入区域
- 视觉权重与其他 listing detail section 对齐（非浮层覆盖，原生嵌入感）
- 建议位置：关键参数区（卧室/面积/停车等）之后，自然出现在用户浏览路径上
- 点击后：展开为完整底部 sheet 对话界面

**取舍分析：**

| | 现有 FAB | Entry Bar |
|---|---|---|
| 可发现性 | 低，依赖用户主动注意图标 | 高，浏览详情时自然遭遇 |
| 持续可访问 | ✅ 始终可见 | ⚠️ 需滚动回到入口位置 |
| 页面融合感 | ⚠️ 浮层感，与内容分离 | ✅ 原生嵌入，感觉是内容的一部分 |
| 上下文关联 | 弱，图标与数据无关联 | 强，物理上紧邻它所引用的 listing 数据 |

**待确认：** entry bar 的滚动位置如何确保第一屏可见性；是否与 sticky 方案并存（首屏可见后 entry bar 出现，FAB 消失）。

---

#### 4. Accessibility Completions
- AI FAB missing text label → add `aria-label="Ask AI about this listing"`
- During streaming, screen reader should announce "AI is generating a response"
- Stop button needs `aria-label="Stop generating"`

---

### V2 — Mid-Term Features

#### 5. Agent Handoff CTA
**Background:** Purchase intent may have risen after AI Q&A.
**Proposal:** When the conversation contains high-intent signals (price, HOA, school district), inject a soft CTA after the AI response:
> *"Want to discuss this in detail? Schedule a viewing with an agent."*

**Value:** Converts AI conversations to leads; directly feeds the "Schedule Viewing" funnel.

#### 6. Conversation History Persistence
**Background:** Conversation history is lost when the session ends.
**Proposal:** Cache conversation history locally (IndexedDB / localStorage) so users can review it on their next visit to the same listing.
**Note:** Requires a "Clear history" option to address privacy concerns.

#### 7. Response Quality Feedback
**Proposal:** Add a lightweight feedback widget (👍 / 👎) to each AI response, used for:
- Continuous model/prompt improvement
- Identifying high-frequency failure scenarios

#### 8. Popular Question Analysis → Chip Optimization
**Proposal:** Collect and analyze real user query data periodically; use it to update default chip copy.
**Long-term goal:** Maintain separate default chip sets by property type (condo / detached / townhouse).

---

### Long-Term Considerations

#### 9. Multimodal Input
- Voice input (especially suited for hybrid app scenarios)
- Image recognition: user photographs a room and asks "What is this room called in the listing?"

#### 10. Cross-Listing Comparison
> "How does this compare to 142 Main St I looked at yesterday?"
Requires AI to hold user browsing history as context — significant architectural change.

#### 11. Proactive AI
AI surfaces relevant hints as the user scrolls through the listing:
> *"This property has no basement — tap to ask about storage options."*

---

## 7. Design Decision Log

| Decision | Rationale |
|----------|-----------|
| No result state uses no error styling | AI is working correctly; data is simply unavailable. Red/warning styling would be a false negative. |
| Chips hidden after first message | Prevents chips from occupying space in the conversation thread |
| FAB tooltip copy: "Ask about this listing" | "Look up" implies a retrieval boundary and sets implicit expectations |
| Sheet height ~580px, listing title bar visible above | Maintains context awareness; user always knows which listing they're on |
| Avatar shown only on first AI message per group | Reduces visual repetition in dense conversation threads |
