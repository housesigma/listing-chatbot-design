# Listing Chatbot — Chat System Design Proposal

> **Type:** Design Proposal · System Design submission
> **Project codename:** Listing Chatbot
> **Product positioning:** Listing-aware natural-language Q&A
> **Status:** Proposal · ready for review
> **Date:** 2026-05-13
> **Owner:** Listing Chatbot design
> **Canvas:** `chat-system-design.pen` (Variant C — 12 states · `hm8Iu` appHeader reusable)
> **i18n namespace:** `aiChat.*`
> **Production source:** `packages/app/src/pages/listing/components/AppAiChat*.vue` · `packages/hook/ai/useAiAssistant.ts` · `packages/hook/ai/useChatStream.ts`

---

## 1. Executive Summary

This proposal codifies the **first canonical design system for AI chat
surfaces** inside the HouseSigma product. It does two things:

1. **Establishes a sister design system** (`Conversational Components`)
   alongside the existing base DS (`housesigma-design-system.md`).
   Chat / AI surfaces follow industry conversational conventions
   (iMessage / WhatsApp / ChatGPT / Claude / Gemini family) which
   intentionally diverge from the listing-platform DS on radius, fill,
   shape, and icon family.

2. **Specifies 12 chat states** that cover the entire listing-chatbot
   interaction state machine (entry → in-flight → terminal outcomes),
   plus a 13th-state set of new patterns required for the product
   positioning shift from *fact lookup* to *listing-aware natural-
   language Q&A*.

The 12 states are visually realized in `chat-system-design.pen`
(Variant C — Chat-First). Every state references this document by
section number for spec-precise design adjustments.

### What this proposal asks of System Design

- **Adopt** Conversational Components (§5) as a sister spec to the
  base DS, with the explicit override rule: *inside any AI chat
  surface, this document wins over the base DS where they collide.*
- **Acknowledge the icon-family departure**: Phosphor `star-four` for
  the AI Brand Mark (§5.7) is the one place we use a non-Lucide,
  non-custom-SVG icon. This is calibrated to the 2024-25 GenAI
  category convention; reverting to Lucide loses brand legibility.
- **Co-author the Q&A evolution roadmap** (§8) — particularly the
  Source/Grounding Label (§5.9) and Follow-up Suggestion Chips
  (§5.10), both of which revive long-standing dead hooks in the
  production state machine.

---

## 2. Context

### 2.1 Current state

Listing Chatbot ships today as a **fact-retrieval surface** — users
tap a sparkle FAB on a listing detail page; a bottom sheet opens
with a welcome hero, three suggestion chips, and a composer. The
assistant answers questions grounded in the listing payload.

The production state machine is already conversational-shape
(streaming SSE; `abstained` / `nullReason` / `sourceLabel` /
`progressLabel` fields exist on the message) but the **UI and copy
treat the assistant as a JSON-key lookup function**. Several rich
schema fields are unrendered (`sourceLabel`, `abstained`,
`nullReason`).

### 2.2 Why a new spec now

| Trigger | Implication |
|---|---|
| **LLM capability** has crossed the bar for grounded reasoning on property-specific questions, not just keyword retrieval | Surface needs to *feel* conversational, not retrieval-shaped |
| **Existing state machine already conversational** (streaming, abstention, progress events) | The schema is ready; only UI + copy need to catch up |
| **Production dead hooks** (`sourceLabel`, `abstained` UI never rendered) | Activating them is most of the design lift; spec needed to define how |
| **Base DS doesn't cover chat** — Buttons §8.1 / Inputs §8.2 / Radii §6 / Icons §13 all calibrated for listings / filters / forms, not message bubbles / composers / suggestion chips | Without a sister spec, every future AI feature reinvents these patterns ad-hoc |
| **GA signal**: `ai_chat_chip_click` events skew toward open-ended chips ("Tell me about this home") | Users already expect Q&A, not lookup |

### 2.3 Variants explored and rejected

Two earlier variants exist in design history (kept in
`listing-chatbot-design.pen` Frames 1 & 2 for archival reference):

- **Variant A — Production Mirror**: faithful copy of current
  production. Used as baseline.
- **Variant B — DS-Compliant**: forced every chat element through
  base DS §6/§8/§13. Result: composer becomes a search form, send
  becomes a generic submit button, message bubbles become cards,
  suggestion chips become filter badges. **Rejected** — the surface
  no longer reads as chat.

**Variant C — Chat-First** (this proposal): follows Conversational
Components spec. Visually almost identical to production (Variant A)
but with retroactive engineering rigor: every value comes from a
named token, every shape from a documented rule, every icon from an
explicit decision. The proposal codifies *what production already
intuitively does right* and provides the spec foundation for further
evolution.

---

## 3. Design Principles

| Principle | Manifestation |
|---|---|
| **Conversational over transactional** | Soft radii (16-24) over tight radii (5-13). Pill inputs over rectangular forms. Circular send button over square submit. |
| **Assistant-active, not system-passive voice** | "Thinking…" not "Looking up…". "Something tripped me up" not "Generation failed". |
| **Redirect, don't refuse** | Every limitation copy ends with a useful next step (alternate path / agent / comparable sales). |
| **Trust through transparency** | Source / Grounding Label visible on every assistant bubble — users see whether the answer is from listing data, market context, or general advice. |
| **Multi-turn by default** | Follow-up chips after every assistant response. Sheet auto-expands when the conversation starts. |
| **Brand-identifiable AI mark** | Phosphor `star-four` (the GenAI category convention) — one explicit deviation from base DS icon library. |

---

## 4. State Machine — 12 Canonical States

Each state below corresponds to a frame in
`chat-system-design.pen` Variant C container, numbered 1–12. The
state machine is sourced from `useAiAssistant.ts` and
`useChatStream.ts` event handlers.

| # | State | Trigger | Key elements |
|---|---|---|---|
| **1** | `lookup_trigger` | Listing detail page loads; user has not tapped FAB yet | Property hero · Bottom action bar with Watch · Schedule Viewing · **AI FAB** (Phosphor sparkle in teal rounded square) |
| **2** | `lookup_hint` | First-time user, `localStorage.hint_ai_chat_viewed != '1'` after 1.5s mount delay | Variant 1 + Coachmark tooltip pointing at the FAB. Auto-dismisses after 2s or tap. |
| **3** | `lookup_halfsnap` | User taps FAB → sheet opens at half snap (50dvh, min 480px) | Scrim over dimmed listing · Sheet at bottom half · Welcome Hero + Scope Pill + 3 Suggestion Chips + Composer |
| **4** | `lookup_entry_hero` | Sheet expanded to full snap before user sends (rare) | Full-screen sheet · Welcome Hero (Sparkle + headline + Scope Pill) + 3 Suggestion Chips + Composer |
| **5** | `lookup_retrieving` | User sent message; `isLoading=true`; no chunks arrived yet | User Bubble + Typing Indicator (3 dots + progress label) + Composer in stop mode |
| **6** | `lookup_streaming` | First content chunk arrives during round | User Bubble + Assistant Bubble with streaming partial content + Composer in stop mode (no typing dots once content starts) |
| **7** | `lookup_stopped` | User taps stop while round in flight | User Bubble + Assistant Bubble with partial content (settled, `done=true` via AbortError) + Composer back in send mode (inactive if empty) |
| **8** | `lookup_result` | `onDone` fires; full answer rendered | Round complete: User Bubble + Assistant Bubble (full content + Source Label + optional Follow-up Chips) |
| **9** | `lookup_show_more` | Past Assistant Bubble exceeded 160px after `done`; subsequent round started | Previous Assistant Bubble collapsed at 160px with gradient fade + Show More toggle; most-recent bubble never collapsed |
| **10** | `lookup_abstained` | `onDone({abstained:true})` | Assistant Bubble with explanatory redirect text + (proposed) "Outside my listing knowledge" inline marker |
| **11** | `lookup_noresult` | Stream ended with empty content; fallback `errorGeneric` injected | Assistant Bubble with no-result fallback copy |
| **12** | `lookup_error` | `onError` with structured `error.code` | Assistant Bubble with code-specific copy from `aiChat.error*` namespace |

---

## 5. Conversational Components

The sister design system for AI chat surfaces. Reuses base DS color
tokens (`--primary`, `--muted`, `--foreground`, etc.) but defines its
own radius scale, shape rules, icon exceptions, and component
specifications.

### 5.1 Conversational Radius Scale

Parallel scale calibrated for soft, tactile geometry. **Do not mix
with base DS radii inside the chat surface.**

| Token | Value | Usage |
|---|---|---|
| **chat-radius-xs** | 2 px | Sheet grabber (full pill on 4 h) |
| **chat-radius-sm** | 10 px | Coachmark / tooltip body |
| **chat-radius-md** | 16 px | Message bubble main corners (alternate, denser feel) |
| **chat-radius-lg** | 20 px | Message bubble main corners (default), Modal Bottom Sheet top |
| **chat-radius-pill** | half-height | Suggestion chips, scope pill, input field, send/stop button (full round) |
| **chat-tail-radius** | 4 px | Message bubble origin-corner (creates speech-bubble tail) |

**Why depart from base DS §6** (which caps at 13 px):
- 13 px on a message bubble reads as "card." 20 px reads as "bubble"
  — the universal chat metaphor.
- DS §6 reserves >=34 for true pills (toggle tracks, toast). Chips
  and pills in the 28-36 h range fall in a gap; we resolve by naming
  *intent* (`pill`) rather than a fixed value.

### 5.2 Message Bubble

| Property | Value (user) | Value (assistant) |
|---|---|---|
| `cornerRadius` | `[20, 20, 4, 20]` (LG + tail bottom-right) | `[20, 20, 20, 4]` (LG + tail bottom-left) |
| `padding` | `[10, 16]` | `[12, 16]` |
| `maxWidth` | 80% of conversation pane width | 80% |
| `fill` | `--primary` | `--muted` |
| `text fill` | `--primary-foreground` (white) | `--foreground` (#333) |
| `font` | Poppins 14 Regular | Poppins 14 Regular (markdown-rendered) |
| `gap` (multi-line content) | n/a | 4 |

The **4 px tail** is intentional: it preserves the soft bubble shape
while pointing "this came from me / them." Square tails (0) feel
sliced; symmetric radii lose the chat metaphor.

### 5.3 Bubble collapse behavior (`done` past messages)

- Past assistant bubble with `scrollHeight > 160 px` auto-collapsed
  to 160 px.
- Linear gradient fade overlay: transparent → `--muted` over the
  bottom 40 px.
- Followed by `Show More Toggle` (§5.13) as a sibling below the
  bubble, inside the same `message-group`.
- **Most-recent assistant bubble is never collapsed.**
- Streaming bubbles (`done: false`) are never collapsed and never
  show the toggle.

### 5.4 Suggestion Chip

Quick-reply pill in the Entry Hero. Tap submits the chip text as a
user message.

| Property | Value |
|---|---|
| Shape | **Full pill** (`chat-radius-pill`, computed 22 for 32-36 h chip) |
| `padding` | `[8, 16]` |
| Background | `--card` |
| Border | 1 px `--border` |
| Font | Poppins 14 Regular, `--foreground` |
| Pressed | Background `--background-section`, border `#CCCCCC` |
| Layout | Stacked vertical, gap 12, centered |
| Count | 3 (matches production `chipAbout / chipListed / chipSchools`) |

**Why pill, not DS Badge §8.5 (radius 4):** badges signal a
selectable filter state. Chips signal "tap to send a conversation
turn" — pill shape is the universal industry cue (Apple Smart
Reply, Gemini suggestions, Perplexity prompt starters).

### 5.5 Conversational Input (Composer Text Field)

Departs significantly from DS §8.2 Input Fields.

| Property | Value | DS §8.2 (rejected) |
|---|---|---|
| Shape | **Full pill** (radius 24) | 5 px radius rectangle |
| Background | `--muted` (soft gray) | `--input-bg` (white) |
| Border | **None** | 1 px `--border` |
| `padding` | `[0, 16]` | varies |
| Height | 48 | 47 |
| Font | Poppins 14 Regular | 14 Regular |
| Placeholder color | `--muted-foreground` | `--input-placeholder` |
| Width | `fill_container` minus send button | 349 fixed |

**Why depart:** production code (`AppAiChatComposer.vue:83-101`)
uses these exact values. Borderless gray pill is what users
recognize as "chat input" — borders + white triggers form-fill
mental model.

**States:**
- **Idle**: pill with placeholder text
- **Typed**: pill with user text in `--foreground`
- **Disabled (loading)**: container at 60% opacity, input disabled,
  placeholder remains visible

### 5.6 Send / Stop Button Pair

Single 48 × 48 button that swaps glyph based on round state.

| Property | Value |
|---|---|
| Shape | **Circular** (radius 24) |
| Background | `--primary` |
| Icon family | **Lucide** (consistent across both states) |
| Send glyph | `arrow-up`, 20 × 20, `--primary-foreground` |
| Stop glyph | white 16 × 16 rectangle, radius 2 (matches production's `<span class="stop-icon">` div-based square) |

**States:**
- Inactive (empty input, idle): `arrow-up`, opacity 0.35
- Active (typed, idle): `arrow-up`, opacity 1.0
- Loading (round in flight): stop glyph, opacity 1.0

**Why circular, not DS §8.1 (10 px square):** the send button is the
single persistent action of the composer — it earns FAB-like
prominence through shape. A rounded square reads as "submit a
form"; a circle reads as "send a message." Industry convention is
unanimous (every major chat client).

**Why Lucide for both glyphs:** keeping a unified icon family makes
the send ↔ stop swap feel like one button changing state, not two
buttons trading places. Filled-glyph swaps (Material Symbols
`stop`) break this continuity.

### 5.7 AI Brand Mark *(the one icon-family exception)*

| Property | Value |
|---|---|
| Icon | **Phosphor `star-four`** (single 4-point star) |
| Weight | 700 (filled) |
| Size (Hero) | 32 |
| Size (FAB inside) | 24 |
| Color | `--primary` (hero) / `--primary-foreground` (inside teal FAB) |

**Why Phosphor `star-four`, not Lucide `sparkles`:** the 4-point star
is the 2024-25 GenAI category mark (OpenAI / Anthropic / Google AI
/ Notion AI / Linear AI / every AI-feature toggle uses this glyph
or a variant). Lucide `sparkles` is decorative multi-glint —
semantically "celebration / magic," not "AI."

> **Branding fidelity > icon-family consistency.** This is the *one*
> icon-family exception in the chat surface. Everywhere else
> (composer, navigation, controls, status bar) use Lucide.

### 5.8 Modal Bottom Sheet

| Property | Value |
|---|---|
| `cornerRadius` (top corners only) | `[20, 20, 0, 0]` |
| Background | `--card` (white) |
| Width | 100% of viewport |
| Bottom anchor | `0` (always sticks to viewport bottom) |
| Snap points | `half` (50 dvh, min 480 px) · `full` (vh − 50 mobile web / vh hybrid) |
| Sheet z-index | Above Scrim |
| Transition | `height 0.3s ease, top 0.3s ease` |
| Snap auto-expand | `hasMessages === true` → snap = `full` |

#### Sheet Grabber

| Property | Value |
|---|---|
| Width | 40 |
| Height | 4 |
| `cornerRadius` | 2 (`chat-radius-xs`, full pill on 4 h) |
| Color | `--border` |
| Container padding | 12 top / 4 bottom |
| Gesture | Drag to resize; below 60% half-height closes the sheet |

#### Scrim / Backdrop

| Property | Value |
|---|---|
| Coverage | Full viewport, behind sheet |
| Color | `--shadow` (`rgba(0, 0, 0, 0.4)`) |
| Dismiss | Tap to close |
| z-index | Below sheet, above page |

### 5.9 Source / Grounding Label *(revives dead hook)*

Small italic teal label rendered as a sibling **below** the assistant
bubble, inside the same `message-group`. Tells the user *what kind of
source* the answer is grounded in.

| Property | Value |
|---|---|
| Position | Sibling of `messageBubble` |
| Layout | inline-block, margin-top 4, margin-left 16 |
| Font | Poppins 13 Italic, `--primary` |
| Content map | `from_listing` → "Based on this listing"<br>`from_market_context` → "Based on market comparison"<br>`general_advice` → "General guidance"<br>`assumption` → "Inferred — verify before relying" |
| Visibility | Render only when `groundingType` is set on the message; render only on `done` (no flicker during streaming) |

**Implementation:** revives the existing
`AppAiChatMessages.vue:46-51` `<span class="source-label">` hook
(currently rendered when `m.sourceLabel` is set — but the state
machine never writes it). Replace free-form `sourceLabel` string
with structured `groundingType` enum.

### 5.10 Follow-up Suggestion Chips *(new)*

Rendered below the **most-recent** assistant bubble (only the most
recent — past bubbles' follow-ups disappear once a new round
starts).

| Property | Value |
|---|---|
| Container | Horizontal flex, wrap, gap 8, padding-top 8 |
| Chip count | 2-3 (server returns up to 3 in `onDone.followUps`) |
| Shape | Same as entry-hero Suggestion Chip but smaller |
| Size | Height 28-30 (vs entry hero 32-36) |
| Font | Poppins 12 Regular, `--foreground` |
| Background | `--card` |
| Border | 1 px `--border` |
| Radius | `chat-radius-pill` (half-height) |
| Behavior | Tap injects chip text as new user message; chips fade out as round starts |
| GA | `ai_chat_follow_up_click` with chip text label |

**Server contract:** `DonePayload` adds `followUps?: string[]` (up to
3 strings).

### 5.11 Typing Indicator (Loading Bubble)

Same outer shape as an Assistant Message Bubble. Renders during the
gap between user send and first content chunk.

| Property | Value |
|---|---|
| `cornerRadius` | `[20, 20, 20, 4]` (matches assistant bubble) |
| Background | `--muted` |
| `padding` | `[12, 16]` |
| `gap` (dots ↔ label) | 8 |

#### Animated Dots

| Property | Value |
|---|---|
| Count | 3 |
| Size | 6 × 6 ellipses |
| Spacing | gap 4 |
| Color | `--muted-foreground` — **all three same color** |
| Animation | `dotPulse 1.4s ease-in-out infinite`, stagger 0 / 200 / 400 ms, scale 0.6 ↔ 1.0, opacity 0.4 ↔ 1.0 |

#### Progress Label

| Property | Value |
|---|---|
| Font | Poppins 12 Regular |
| Color | `--muted-foreground` |
| Default copy | `Thinking…` |
| Worker stage `intent_classification` | Server-supplied (e.g. `Reading the listing…`) |
| Worker stage `response_rendering` | Server-supplied (e.g. `Composing your answer…`) |
| `loadingDuration ≥ 10s` (no stage event) | `Still thinking…` |
| First chunk arrives | dots disappear, streaming begins |

**Static design convention:** in mockups, render all three dots
**same color, different opacities** (1.0 / 0.7 / 0.4) to imply the
wave animation without making them look like three misaligned dots
of different colors.

### 5.12 Abstain Pattern *(revives dead hook)*

When `abstained: true` arrives, render the assistant bubble with a
small inline marker above the content.

| Element | Spec |
|---|---|
| Marker row | Above bubble content, inside bubble padding-top |
| Marker icon | Lucide `info`, 12 × 12, `--muted-foreground` |
| Marker text | "Outside my listing knowledge" — Poppins 11 Medium, `--muted-foreground` |
| Bubble content | LLM-generated redirect (per §7.5 abstain copy direction) |
| Optional `nullReason` | Poppins 10 italic, below bubble, if non-null and human-readable |

**This is explicitly distinct from §5.16 *error* bubble:** abstain
means the assistant *chose* not to answer; error means the *system*
failed.

### 5.13 Show More Toggle

Inline expand/collapse control for past Assistant bubbles that
exceeded the 160 px collapse threshold.

| Property | Value |
|---|---|
| Type | Text button (no chrome) |
| Font | Poppins 13 Medium |
| Color | `--primary` |
| Position | Below the collapsed bubble, left-aligned with bubble content |
| `padding` | `[4, 16, 0, 16]` |
| Labels | `Show more` (collapsed) ↔ `Show less` (expanded) |
| Tap target | Min 32 × 32 (inflate hit area beyond visible text) |

### 5.14 Coachmark (First-Time Hint Tooltip)

Onboarding tooltip pointing to the AI FAB. Shows once per user
(`localStorage.hint_ai_chat_viewed`).

| Property | Value |
|---|---|
| Pill `cornerRadius` | 10 (`chat-radius-sm`) |
| Pill `padding` | `[10, 12]` |
| Pill background | `--primary` |
| Body font | Poppins 14 Semibold, white |
| Body copy | "Ask about this home" *(updated from production "Look up listing details" per §7 positioning shift)* |
| Close icon | Lucide `x`, 14, white at 70% opacity |
| Pointer | 14 × 10 triangle path, fill `--primary`, anchored to pill bottom, offset 20 px from right edge |
| Shadow | `offset 0,4 · blur 12 · color --shadow` (DS §7 MD equivalent) |
| Auto-dismiss | 2 s after mount, or tap to dismiss |

### 5.15 Scope Pill (Context Chip in Entry Hero)

Static info pill showing the current listing context.

| Property | Value |
|---|---|
| Shape | **Full pill** (`chat-radius-pill`, computed 17 for 34 h chip — matches production) |
| `padding` | `[6, 16, 6, 10]` (extra left for embedded icon) |
| Background | `--bg-cyan-light` (thematic teal tint connecting to chat brand) |
| Border | None (production has no border) |
| `gap` (icon ↔ text) | 8 |
| Icon container | 22 × 22 circle, fill `--primary`, contains 16 × 16 Lucide `map-pin` (white) |
| Text | Poppins 13 Medium, `--foreground` |
| Content | `{address}{city ? \` · ${city}\` : ''}` (single line, truncate with ellipsis) |

### 5.16 Error Bubble *(per `error.code`)*

Same shape as Assistant Bubble; copy and tone vary by error code.
The full code → copy map lives in `aiChat.error*` i18n keys.

| Error code | i18n key | User-facing copy |
|---|---|---|
| `ttft_timeout` | `errorTtft` | "I'm taking longer than expected to respond. Please try again." |
| `idle_timeout` | `errorIdle` | "My response was cut short. Please try asking again." |
| `round_timeout` / `timeout` | `errorRound` | "The request took too long to complete. Please try again." |
| `rate_limit_exceeded` / `session_busy` | `errorBusy` | "I'm a bit busy right now — please try again in a moment." |
| `context_length_exceeded` | `errorTooLong` | "Your message is too long. Please shorten it and try again." |
| `auth_expired` | `errorAuthExpired` | "Your session has expired. Please refresh and sign in again." |
| `safety_blocked` | `errorSafetyBlocked` | "Let's stay focused on this listing — happy to help with anything about the home." *(updated per §7)* |
| `cancelled` | `errorCancelled` | "The request was cancelled. Please try again." |
| `mcp_error` | `errorMcpError` | "Unable to fetch listing data right now. Please try again." |
| `inference_error` | `errorInference` | "The AI service ran into a problem. Please try again." |
| *default* | `errorGeneric` | "Something tripped me up. Try again?" *(updated per §7)* |

### 5.17 AI Entry FAB (Listing Detail page)

Floating sparkle button inside the Bottom Action Bar of the listing
detail page. Single tap opens the chat sheet at `half` snap.

| Property | Value |
|---|---|
| Shape | Rounded square (`chat-radius-sm`, radius 10) |
| Size | 56 × 56 (matches base DS §8.1 button height for visual rhythm in the action bar) |
| Background | `--primary` |
| Icon | Phosphor `star-four`, 24 × 24, white, weight 700 |
| Action bar position | Right-most of three buttons (Watch · Schedule Viewing · AI) |
| Gap from neighbor | 10 |

> **Note:** This is the *one* place chat radii do **not** apply,
> because the FAB sits *inside* the listing platform's Bottom Action
> Bar — a base-DS component. The FAB matches the radii and height
> of its neighbors (Watch outline / Schedule Viewing filled, both
> 10 px @ 56 h) for cohesion with the listing page. Once the sheet
> opens, all chat-radii apply inside the sheet.

---

## 6. Component Composition Rules

| Inside a chat surface | Outside (listing detail, search, etc.) |
|---|---|
| Use this document's tokens | Use `housesigma-design-system.md` |
| Soft radii (16-24) | Tight radii (5-13) |
| Pill inputs, circular send | Rectangular inputs, square buttons |
| Phosphor `star-four` for AI brand | Lucide / custom SVG icons |
| `--muted` gray fills | `--card` white fills |
| 4 px-grid spacing OK to break for bubble feel | Strict 4 px grid |

**Where chat surface ends:**
- Sheet container's edge — anything outside the sheet (status bar,
  app header, listing content behind the scrim) follows base DS.
- The AI FAB's container (Bottom Action Bar) — the FAB itself
  follows base DS for cohesion with sibling CTAs.

---

## 7. NLQ&A Positioning Shift

The product is evolving from a *fact-retrieval surface* to a
*listing-aware conversation partner*. This shift requires
calibrated copy changes across the entire chat surface.

### 7.1 Mental-model shift

| | Lookup tool *(today)* | Listing-aware Q&A *(target)* |
|---|---|---|
| **User mental model** | "Find me a specific piece of info" | "Ask anything about this home in my own words" |
| **Question shape** | Factual, single-turn | Open-ended, conversational, multi-turn |
| **Assistant scope** | Listing payload only | Listing + market context + advisory perspective |
| **Failure mode** | "Info not in listing" *(data-gap framing)* | "Outside my knowledge — try this instead" *(advisory redirect)* |
| **Success signal** | One correct fact retrieved | A conversation that helps the user decide |
| **Voice** | System-passive ("Looking up…") | Assistant-active ("Thinking…") |
| **Value proposition** | Saves a tab switch | Replaces an exploratory call to the agent |

### 7.2 Copy changes table

| Module | Today (lookup) | Target (NLQ&A) | i18n key |
|---|---|---|---|
| Hero headline | "Look up details in this listing" | **"Ask anything about this home"** | `aiChat.entryLabel` → `askAnything` |
| Composer placeholder | "Look up details in this listing…" | **"Ask anything about this home…"** | derived |
| Coachmark tooltip | "Look up listing details" | **"Ask about this home"** | `aiChat.hintTitle` → `askAboutThis` |
| Loading default | "Looking up details…" | **"Thinking…"** | `aiChat.lookingUp` → `thinking` |
| Loading slow (≥10 s) | "Still looking…" | **"Still thinking…"** | `aiChat.stillLooking` → `stillThinking` |
| Worker stage progress | Generic | Stage-specific (`Reading the listing…` / `Considering the question…` / `Composing your answer…`) | server `ProgressEvent` |
| Chip 1 | "Tell me about this home" | **Keep** | `aiChat.chipAbout` |
| Chip 2 | "How long has it been listed?" | **"How does it compare to nearby listings?"** | `aiChat.chipListed` → `chipCompare` |
| Chip 3 | "What schools are nearby?" | **"Anything I should be cautious about?"** | `aiChat.chipSchools` → `chipCautions` |
| No-result | "School info isn't included. Try asking about bedrooms…" | **"I'm not sure about that one. Want me to try a related angle? You could also ask the listing agent."** | new `noResultGeneric` |
| Abstain copy | "I can't determine the seller's bottom-line price from the listing. Asking price is $524,886." | **"Listing data alone won't tell us — but recent comparable sales might. Want me to compare?"** | server-side, system prompt tuned |
| Safety blocked | "This question can't be answered. Please rephrase…" | **"Let's stay focused on this listing — happy to help with anything about the home."** | `aiChat.errorSafetyBlocked` |
| Error generic | "Something went wrong while generating a response…" | **"Something tripped me up. Try again?"** | `aiChat.errorGeneric` |

### 7.3 Voice principles

- **First-person assistant** ("I", "me") not system-passive ("the
  service")
- **Active verbs** (`Thinking`, `Considering`) not passive (`Looking
  up`, `Fetching`)
- **Redirect, don't refuse** — every limitation copy ends with a
  useful next step

### 7.4 System prompt direction

*(AI team owns the prompt text; design owns intent / tone.)*

The worker's system message should be rewritten to:

- Frame the assistant as a **knowledgeable conversation partner**,
  not a JSON-key lookup function.
- Authorize **reasoning beyond the payload**: market context,
  comparable sales (when available), buyer-perspective advice,
  common-sense observations.
- Mandate **redirect-shape abstention**: if the answer truly can't
  form, return `abstained: true` AND include a helpful alternative
  — never a flat "I don't know."
- Emit `groundingType` per round: classify the primary basis of the
  answer.
- Emit `followUps` (up to 3): phrase as natural questions the user
  might ask next, tied to the conversation thread.

---

## 8. Implementation Considerations

### 8.1 Schema additions on `ChatMessage`

```typescript
interface ChatMessage {
  // existing
  id: string
  role: 'user' | 'assistant'
  content: string
  done?: boolean
  abstained?: boolean
  nullReason?: string | null
  errorCode?: string
  createdAt: Date

  // new for NLQ&A
  sourceLabel?: string                // legacy free-form — deprecate
  groundingType?: 'from_listing' | 'from_market_context'
                | 'general_advice' | 'assumption'
  followUps?: string[]                // up to 3
}
```

### 8.2 SSE `DonePayload` additions

```typescript
interface DonePayload {
  abstained?: boolean
  nullReason?: string | null
  // new
  groundingType?: string              // enum above
  followUps?: string[]                // up to 3
}
```

### 8.3 GA event taxonomy (additive)

| Event | Label / params |
|---|---|
| `ai_chat_follow_up_click` | chip text content |
| `ai_chat_new_conversation` | (no params; from sheet header reset) |
| `ai_chat_grounding_seen` | `groundingType` value |

### 8.4 Component file impact

| File | Change |
|---|---|
| `useAiAssistant.ts` | Add `sourceLabel` / `groundingType` / `followUps` writes in `onDone` handler |
| `useChatStream.ts` | Extend `DonePayload` typing; pass new fields through |
| `AppAiChatMessages.vue` | Render Source Label (§5.9), Abstain Pattern (§5.12); the existing `<span class="source-label">` becomes structured |
| `AppAiChatFollowUps.vue` *(new)* | Render Follow-up Chips (§5.10) below most-recent assistant bubble |
| `AppAiChatSheet.vue` | Add New Chat reset button to header |
| `AppAiChatComposer.vue` | No structural change; placeholder copy updated via i18n |
| `AppAiChatHero.vue` | Headline copy + chip composition updated via i18n |
| `AppAiHintPopup.vue` | Tooltip copy updated via i18n |
| `i18n/translation/{en,zh}.ts` | Rename keys per §7.2 + add new keys (`thinking`, `askAboutThis`, `chipCompare`, `chipCautions`, `noResultGeneric`) |

### 8.5 Feature flag

Gate the new render paths and copy changes behind a flag
(`listingChatbotNLQA`) to enable A/B rollout and safe rollback.
Existing flag pattern in `useAiAssistant.ts:117` (`environment
.listingChatbotProgressEvent`) is the template.

---

## 9. Rollout Phases

### Phase 0 — Spec adoption *(System Design review)*

- Adopt Conversational Components as sister spec
- Acknowledge AI Brand Mark icon-family exception (§5.7)
- Approve schema additions (§8.1, §8.2)

### Phase 1 — Cognitive reframing *(Week 1)*

- Copy: hero / placeholder / coachmark / loading default & slow
  (§7.2)
- Suggestion chips: rebalance to 1 broad / 1 comparative / 1
  advisory
- System prompt rewrite (§7.4)
- i18n key renames

**Acceptance:** a first-time user sees "Ask anything about this
home," not "Look up details." Chip-tap GA reflects more open-ended
questions.

### Phase 2 — Trust & extensibility *(Week 2-3)*

- `sourceLabel` / `groundingType` end-to-end (§5.9)
- Follow-up Suggestion Chips (§5.10)
- Abstain Pattern revival (§5.12)
- Server stage-specific progress labels (§5.11)

**Acceptance:** every assistant bubble carries a Source Label;
non-trivial answers carry ≥2 follow-up chips; abstained rounds
redirect instead of dead-ending.

### Phase 3 — Deep conversation support *(Week 4+)*

- New Chat reset button (header)
- Idle re-engagement prompt
- Markdown rich-content validation (lists, tables, inline links in
  long answers)
- Half-snap min-height bumped 450 → 480

**Acceptance:** average conversation depth (rounds per session)
trends up; abandonment after first answer trends down.

---

## 10. Open Questions

1. **Source-label granularity**: 4 enum values enough? Or do we need
   a separate "verified by HouseSigma data" tier for AVM-backed
   claims?
2. **Follow-up generation cost**: server-side LLM call adds latency
   to `onDone`. Acceptable budget?
3. **Abstention discoverability**: does showing "Outside my listing
   knowledge" lower trust, or raise it? A/B candidate.
4. **AI Avatar / persona mark**: should assistant bubbles carry a
   small sparkle prefix? Trade-off: identification vs visual noise.
5. **Voice input affordance** — not in scope for this proposal, but
   flagged for next iteration.
6. **Multi-listing memory**: out of scope here, but worth flagging
   as a user expectation that may emerge.

---

## 11. Risk & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| LLM hallucinates beyond listing data | Medium | High | `groundingType` UI surfaces uncertainty; system prompt mandates "Inferred — verify" tag for assumptions |
| Cost spike from longer answers + follow-up generation | High | Medium | Cap answer length; cache follow-up chips per round; sample LLM call rate |
| Users start asking out-of-scope questions ("what's the weather?") | Medium | Low | `errorSafetyBlocked` copy redirects; conversational tone keeps users in scope |
| Existing factual users get worse experience | Low | Medium | System prompt: "match answer length to question specificity" |
| Dead-hook revival breaks production builds | Low | High | Feature flag (`listingChatbotNLQA`) gates new render paths |

---

## 12. References

### Design artifacts in this proposal

- **`chat-system-design.pen`** — Variant C canvas, 12 chat states +
  `hm8Iu` appHeader reusable
- **`chat-system-design.md`** — this document

### Sibling specs

- `housesigma-design-system.md` — base platform DS (lives in the
  housesigma design system repository)
- `v1-fact-lookup.md` — original v1 fact-lookup era design intent
  (superseded; preserved for historical context)
- `v2-nlqa.md` — v2 NLQ&A repositioning design intent (this document
  builds on it; §7 here consolidates its key tables)
- `v2-nlqa-spec.md` — v2 Conversational Components spec (this
  document inlines its component specs in §5 for the system-design
  review)

### Production source of truth

- `packages/app/src/pages/listing/components/AppAiChatSheet.vue`
- `packages/app/src/pages/listing/components/AppAiChatHero.vue`
- `packages/app/src/pages/listing/components/AppAiChatMessages.vue`
- `packages/app/src/pages/listing/components/AppAiChatComposer.vue`
- `packages/app/src/pages/listing/components/AppAiHintPopup.vue`
- `packages/hook/ai/useAiAssistant.ts`
- `packages/hook/ai/useChatStream.ts`
- `packages/common/i18n/translation/en.ts` (`aiChat.*` namespace)

### Companion work to author after acceptance

- System prompt design intent doc *(AI team owns content; design
  owns intent / tone)*
- AI Avatar persona mark spec *(if open question 4 resolves toward
  yes)*

---

*End of proposal.*
