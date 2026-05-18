# HouseSigma Chat Design System

> **Status: Sister-spec proposal.**
>
> This document proposes a chat / AI surface design system for
> HouseSigma. It is a **sister specification** to
> `housesigma-design-system.md` (the base platform DS, living in
> `prototypes/pencil-poc/`), not a subset.
>
> The base DS covers transactional surfaces (listings, filters, forms);
> this document covers conversational surfaces (chat bubbles,
> composers, AI sheets). **Where the two disagree, this document wins
> inside any chat / AI surface.**
>
> Companion canvas: `chat-system-design.pen` (Variant C — Chat-First).
> The markdown spec and the canvas together are the chat-system design
> proposal in this repo.
>
> First implementing surface: Listing Chatbot — see
> [`v2-nlqa.md`](v2-nlqa.md).
>
> Last generated from: `chat-system-design.pen` (Variant C — Chat-First)
> Date: 2026-05-14

---

## 1. Overview

**Chat / AI surfaces** are conversational UIs inside the HouseSigma
product — currently the Listing Chatbot bottom sheet, with more AI
features expected on the roadmap. They follow industry conversational
conventions (iMessage / WhatsApp / ChatGPT / Claude / Gemini family)
which intentionally diverge from the listing-platform DS on radius,
fill, shape, and icon family.

This document defines the **Conversational Components** system:
- The **token departures** from base DS (radius / fill semantics / icon
  exception) — see §3
- The **12 canonical chat states** (entry → in-flight → terminal
  outcomes) every chat surface must handle — see §4
- The **15+ reusable components** that make up a chat surface — see §5
- The **composition rules** for combining base DS and chat tokens
  cleanly — see §6

### 1.1 Relationship to base DS

| Aspect | Base DS (`housesigma-design-system.md`) | Chat DS (this document) |
|---|---|---|
| **Surface type** | Transactional (listings, filters, forms) | Conversational (bubbles, composers, AI sheets) |
| **Radii** | 5-13 px (tight, gridded) | 16-24 px (soft, organic) |
| **Inputs** | Bordered rectangle, white fill | Borderless pill, gray fill |
| **Primary action** | Square button (10 px radius) | Circular button (full radius) |
| **Icon family** | Lucide / custom SVG | Lucide + 1 exception (AI brand) |
| **Mental model** | "Form / card" | "Chat / message" |
| **4 px grid** | Strict | Relaxed for bubble feel |

### 1.2 Token reuse

Chat DS **reuses base DS color tokens** (`--primary`, `--muted`,
`--foreground`, etc.) — no new color tokens introduced. Departures are
limited to:
- **Shape (radius)** — see §3.1
- **Fill semantics** — see §3.2
- **Icon family** for the AI brand mark — see §3.3
- **Component composition** — see §5

This makes Chat DS a strict subset of pencil-poc's overall visual system
from a color / type perspective, and a parallel extension only on shape
and conversational metaphors.

### 1.3 Applicable surfaces

Any surface that exhibits chat-like interaction patterns:
- Listing Chatbot **mobile bottom sheet** (`AppAiChatSheet`, the original canonical user)
- Listing Chatbot **desktop floating popover** (`PcAiChat`, anchored bottom-right
  of the listing-page viewport) — same 12-state machine, different container shell
- Future AI assistants (property valuation Q&A, market analysis,
  agent-buyer chat — if/when shipped)
- Any feature using streaming text + bubble exchange

Surfaces that look conversational but are **not** chat
(e.g. notification toasts, error messages, status banners) follow base
DS, not this document.

---

## 2. Design Principles

| Principle | Manifestation |
|---|---|
| **Conversational over transactional** | Soft radii (16-24) over tight radii (5-13). Pill inputs over rectangular forms. Circular send button over square submit. |
| **Assistant-active, not system-passive voice** | "Thinking…" not "Looking up…". "Something tripped me up" not "Generation failed". |
| **Redirect, don't refuse** | Every limitation copy ends with a useful next step (alternate path / agent / comparable data). |
| **Trust through transparency** | Source / Grounding Label visible on every assistant bubble — users see whether the answer is from primary data, derived context, or general advice. |
| **Multi-turn by default** | Follow-up chips after every assistant response. Sheet auto-expands when the conversation starts. |
| **Brand-identifiable AI mark** | Phosphor `star-four` (the GenAI category convention) — the one explicit deviation from base DS icon library. |

---

## 3. Token Departures from Base DS

### 3.1 Conversational Radius Scale

Parallel scale calibrated for soft, tactile geometry. **Do not mix with
base DS radii inside the chat surface.**

| Token | Value | CSS Variable | Usage |
|---|---|---|---|
| **chat-radius-xs** | 2 px | `--chat-radius-xs` | Sheet grabber (full pill on 4 h element), system indicators |
| **chat-radius-sm** | 10 px | `--chat-radius-sm` | Coachmark / tooltip body |
| **chat-radius-md** | 16 px | `--chat-radius-md` | Message bubble main corners (alternate, denser feel) |
| **chat-radius-lg** | 20 px | `--chat-radius-lg` | Message bubble main corners (default), Modal Bottom Sheet top |
| **chat-radius-pill** | half-height | `--chat-radius-pill` | Suggestion chips, scope pill, conversational input field, send/stop button (full round) |
| **chat-tail-radius** | 4 px | `--chat-tail-radius` | Message bubble origin-corner (creates speech-bubble tail) |

**Why depart from base DS §6** (which caps at 13 px):
- 13 px on a message bubble reads as "card." 20 px reads as "bubble" —
  the universal chat metaphor.
- Base DS §6 reserves `>=34` for true pills (toggle tracks, toast).
  Chips and pills in the 28-36 h range fall in a gap; we resolve by
  naming *intent* (`pill`) rather than a fixed value.

### 3.2 Fill Semantics

| Surface | Base DS fill | Chat DS fill | Why |
|---|---|---|---|
| **Input field idle** | `--input-bg` (white) | `--muted` (#F2F2F2) | Borderless gray pill reads as "chat input"; white + border triggers form-fill mental model |
| **Assistant bubble** | n/a | `--muted` | Subtle distinction from user bubble |
| **User bubble** | n/a | `--primary` | Brand color reinforces user voice as the active interlocutor |
| **Suggestion chip** | n/a | `--card` + 1 px `--border` | Reads as "tap to send conversation" |
| **Loading bubble** | n/a | `--muted` | Same shape and fill as Assistant bubble for continuity |
| **AI brand mark on FAB** | n/a | `--primary` background | Differentiates the AI entry from other action buttons |

### 3.3 Icon Family Exception

Chat DS uses **Lucide** everywhere (composer arrow, stop square, marker
info, navigation, controls — same as base DS), with **one exception**:

| Element | Icon family | Glyph | Why |
|---|---|---|---|
| **AI Brand Mark** | **Phosphor** | `star-four` (single 4-point star, weight 700) | The 4-point star is the 2024-25 GenAI category mark (OpenAI / Anthropic / Google AI / Notion AI / Linear AI all use this glyph or a variant). Lucide `sparkles` is decorative multi-glint — semantically "celebration / magic," not "AI." Branding fidelity > icon-family consistency. |

This is the **one** icon-family exception in the chat surface.
Everywhere else (composer, navigation, controls, status bar) use Lucide.

---

## 4. State Machine — 12 Canonical Chat States

Every chat surface implements this state machine. State transitions are
driven by event handlers in the underlying chat stream (e.g.
`useChatStream` + `useAiAssistant` for listing chatbot). This table is
schema-agnostic — replace product-specific event names with your
chat-stream events.

| # | State | Trigger | Key elements |
|---|---|---|---|
| **1** | `trigger` | Host page loads; user has not opened the chat surface yet | Host UI · AI FAB (Phosphor sparkle on `--primary` rounded square, follows base DS height) |
| **2** | `hint` | First-time user, `localStorage.hint_*_viewed != '1'` after mount delay | State 1 + Coachmark tooltip (§5.12) pointing at the FAB. Auto-dismisses after 2 s or tap. |
| **3** | `halfsnap` | User taps FAB → sheet opens at half snap (50 dvh, min 480 px) | Scrim over dimmed host · Sheet at bottom half · Entry Hero (Sparkle + headline + Scope Pill (§5.13) + 3 Suggestion Chips (§5.2)) + Composer (§5.4) |
| **4** | `entry_hero` | Sheet expanded to full snap before user sends (rare) | Full-screen sheet · same content as state 3 |
| **5** | `retrieving` | User sent message; loading flag true; no chunks arrived yet | User Bubble (§5.1) + Typing Indicator (§5.10) + Composer in stop mode |
| **6** | `streaming` | First content chunk arrives during round | User Bubble + Assistant Bubble with streaming partial content + Composer in stop mode (no typing dots once content starts) |
| **7** | `stopped` | User taps stop while round in flight | User Bubble + Assistant Bubble with partial content (settled, marked `done`) + Composer back in send mode |
| **8** | `result` | `onDone` fires; full answer rendered | Round complete: User Bubble + Assistant Bubble (full content + Source/Grounding Label (§5.8) + optional Follow-up Chips (§5.3)) |
| **9** | `show_more` | Past Assistant Bubble exceeded 160 px after `done`; subsequent round started | Previous Assistant Bubble collapsed at 160 px with gradient fade + Show More toggle (§5.11); most-recent bubble never collapsed |
| **10** | `abstained` | `onDone({abstained: true})` | Assistant Bubble with redirect text + Abstain Pattern marker (§5.9) above content |
| **11** | `noresult` | Stream ended with empty content; fallback generic copy injected | Assistant Bubble with no-result fallback copy |
| **12** | `error` | `onError` with structured `error.code` | Assistant Bubble (§5.14) with code-specific copy from the chat surface's i18n error namespace |

State names above are **logical names**, not symbol names. Each
implementing product may map these to its own state machine identifiers.

---

## 5. Components

### 5.1 Message Bubble

Carries one message in a conversation. Renders inside the conversation
pane.

| Property | User bubble | Assistant bubble |
|---|---|---|
| `cornerRadius` | `[20, 20, 4, 20]` (LG + tail bottom-right) | `[20, 20, 20, 4]` (LG + tail bottom-left) |
| `padding` | `[10, 16]` | `[12, 16]` |
| `maxWidth` | 80 % of conversation pane | 80 % |
| Background | `--primary` | `--muted` |
| Text fill | `--primary-foreground` | `--foreground` |
| Font | Poppins 14 Regular | Poppins 14 Regular (markdown-rendered) |
| `gap` (multi-line) | n/a | 4 |

The **4 px tail** is intentional: it preserves the soft bubble shape
while pointing "this came from me / them." Square tails (radius 0) feel
sliced; symmetric radii lose the chat metaphor.

#### 5.1.1 Bubble collapse behavior (`done` past messages)

- Past assistant bubble with `scrollHeight > 160 px` auto-collapses to
  160 px
- Linear gradient fade overlay: transparent → `--muted` over the bottom
  40 px
- Followed by Show More Toggle (§5.11) as a sibling below the bubble,
  inside the same `message-group`
- **Most-recent assistant bubble is never collapsed**
- Streaming bubbles (`done: false`) are never collapsed and never show
  the toggle

### 5.2 Suggestion Chip (Entry Hero)

Quick-reply pill in the entry hero. Tap submits the chip text as a user
message.

| Property | Value |
|---|---|
| Shape | Full pill (`chat-radius-pill`, computed 22 for 32-36 h chip) |
| `padding` | `[8, 16]` |
| Background | `--card` |
| Border | 1 px `--border` |
| Font | Poppins 14 Regular, `--foreground` |
| Pressed | Background `--background-section`, border `#CCCCCC` |
| Layout | Stacked vertically, gap 12, centered |
| Count | Typically 3 |

**Why pill, not DS Badge (4 px radius):** badges signal a selectable
filter state. Chips signal "tap to send a conversation turn" — pill
shape is the universal industry cue (Apple Smart Reply, Gemini
suggestions, Perplexity prompt starters).

### 5.3 Follow-up Suggestion Chip

Rendered below the **most-recent** assistant bubble (only the most
recent — past bubbles' follow-ups disappear once a new round starts).

| Property | Value |
|---|---|
| Container | Horizontal flex, wrap, gap 8, padding-top 8 |
| Chip count | 2-3 |
| Shape | Same as §5.2 but smaller |
| Size | Height 28-30 (vs entry hero 32-36) |
| Font | Poppins 12 Regular, `--foreground` |
| Background | `--card` |
| Border | 1 px `--border` |
| Radius | `chat-radius-pill` (half-height) |
| Behavior | Tap injects chip text as new user message; chips fade out as round starts |

**Data contract**: chip text strings supplied by the chat stream
(typically on `onDone.followUps` or equivalent). The component is
schema-agnostic about how strings are produced.

### 5.4 Conversational Input (Composer Text Field)

Departs significantly from base DS §8.2 Input Fields.

| Property | Value | Base DS §8.2 (rejected) |
|---|---|---|
| Shape | Full pill (radius 24) | 5 px radius rectangle |
| Background | `--muted` (soft gray) | `--input-bg` (white) |
| Border | None | 1 px `--border` |
| `padding` | `[0, 16]` | varies |
| Height | 48 | 47 |
| Font | Poppins 14 Regular | 14 Regular |
| Placeholder color | `--muted-foreground` | `--input-placeholder` |
| Width | `fill_container` minus send button | 349 fixed |

**Why depart:** borderless gray pill is what users recognize as "chat
input" — borders + white triggers form-fill mental model.

**States:**
- **Idle**: pill with placeholder text
- **Typed**: pill with user text in `--foreground`
- **Disabled (loading)**: container at 60 % opacity, input disabled,
  placeholder remains visible

### 5.5 Send / Stop Button Pair

Single 48 × 48 button that swaps glyph based on round state.

| Property | Value |
|---|---|
| Shape | Circular (radius 24) |
| Background | `--primary` |
| Icon family | Lucide (consistent across both states) |
| Send glyph | `arrow-up`, 20 × 20, `--primary-foreground` |
| Stop glyph | White 16 × 16 rectangle (radius 2) — matches production's div-based square |

**States:**

| State | Glyph | Opacity |
|---|---|---|
| Inactive (empty input, idle) | `arrow-up` | 0.35 |
| Active (typed, idle) | `arrow-up` | 1.0 |
| Loading (round in flight) | Stop square | 1.0 |

**Why circular, not base DS (10 px square):** the send button is the
single persistent action of the composer — it earns FAB-like prominence
through shape. A rounded square reads as "submit a form"; a circle reads
as "send a message." Industry convention is unanimous.

**Why Lucide for both glyphs (not Material `stop`):** keeping a unified
icon family makes the send ↔ stop swap feel like one button changing
state, not two buttons trading places.

### 5.6 AI Brand Mark *(the one icon-family exception)*

The decorative AI / sparkle icon that anchors entry surfaces and AI
identity.

| Property | Value |
|---|---|
| Icon | **Phosphor `star-four`** (single 4-point star) |
| Weight | 700 (filled) |
| Size (Hero) | 32 |
| Size (FAB inside) | 24 |
| Color | `--primary` (hero) / `--primary-foreground` (inside teal FAB) |

See §3.3 for the rationale.

### 5.7 Modal Bottom Sheet

Container for the conversation. Two snap points.

| Property | Value |
|---|---|
| `cornerRadius` (top corners only) | `[20, 20, 0, 0]` (`chat-radius-lg`) |
| Background | `--card` |
| Width | 100 % of viewport |
| Bottom anchor | `0` (always sticks to viewport bottom) |
| Snap points | `half` (50 dvh, min 480 px) · `full` (vh − 50 mobile web / vh hybrid) |
| Sheet z-index | Above Scrim |
| Transition | `height 0.3s ease, top 0.3s ease` |
| Snap auto-expand | `hasMessages === true` → snap = `full` |

#### 5.7.1 Sheet Grabber

| Property | Value |
|---|---|
| Width | 40 |
| Height | 4 |
| `cornerRadius` | 2 (`chat-radius-xs`, full pill on 4 h) |
| Color | `--border` |
| Container padding | 12 top / 4 bottom |
| Gesture | Drag to resize; below 60 % half-height closes the sheet |

#### 5.7.2 Scrim / Backdrop

| Property | Value |
|---|---|
| Coverage | Full viewport, behind sheet |
| Color | `--shadow` (`rgba(0, 0, 0, 0.4)`) |
| Dismiss | Tap to close |
| z-index | Below sheet, above page |

### 5.8 Source / Grounding Label

Small italic teal label rendered as a sibling **below** the assistant
bubble, inside the same `message-group`. Tells the user *what kind of
source* the answer is grounded in.

| Property | Value |
|---|---|
| Position | Sibling of message bubble |
| Layout | inline-block, margin-top 4, margin-left 16 |
| Font | Poppins 13 Italic, `--primary` |
| Visibility | Render only when grounding type is set on the message; render only on `done` (no flicker during streaming) |

**Content map** (logical, surface decides exact strings via i18n):

| Grounding type | Suggested copy |
|---|---|
| `from_listing` (or equivalent primary data) | "Based on this listing" |
| `from_market_context` | "Based on market comparison" |
| `general_advice` | "General guidance" |
| `assumption` | "Inferred — verify before relying" |

**Data contract**: the chat stream supplies a grounding type enum on
`onDone` (e.g. `groundingType`). The component is schema-agnostic about
enum names — surfaces map their own enum to the four-tier conceptual
space above.

### 5.9 Abstain Pattern Marker

When the stream reports the assistant has chosen not to answer
(`abstained: true` or equivalent), render the assistant bubble with a
small inline marker above the content.

| Element | Spec |
|---|---|
| Marker row | Above bubble content, inside bubble padding-top |
| Marker icon | Lucide `info`, 12 × 12, `--muted-foreground` |
| Marker text | "Outside my [scope] knowledge" — Poppins 11 Medium, `--muted-foreground` |
| Bubble content | LLM-generated redirect text (per surface's system prompt direction) |
| Optional reason | Poppins 10 italic, below bubble, if non-null and human-readable |

**This is explicitly distinct from §5.14 *error* bubble:** abstain means
the assistant *chose* not to answer; error means the *system* failed.

### 5.10 Typing Indicator (Loading Bubble)

Same outer shape as an Assistant Message Bubble. Renders during the gap
between user send and first content chunk.

| Property | Value |
|---|---|
| `cornerRadius` | `[20, 20, 20, 4]` (matches assistant bubble) |
| Background | `--muted` |
| `padding` | `[12, 16]` |
| `gap` (dots ↔ label) | 8 |

#### 5.10.1 Animated Dots

| Property | Value |
|---|---|
| Count | 3 |
| Size | 6 × 6 ellipses |
| Spacing | gap 4 |
| Color | `--muted-foreground` — **all three same color** |
| Animation | `dotPulse 1.4s ease-in-out infinite`, stagger 0 / 200 / 400 ms, scale 0.6 ↔ 1.0, opacity 0.4 ↔ 1.0 |

#### 5.10.2 Progress Label

| Property | Value |
|---|---|
| Font | Poppins 12 Regular |
| Color | `--muted-foreground` |
| Default copy | `Thinking…` |
| Stage event (per pipeline) | Server-supplied (e.g. `Reading the listing…`, `Composing your answer…`) |
| `loadingDuration ≥ 10s` (no stage event) | `Still thinking…` |
| First chunk arrives | Dots disappear, streaming begins |

**Static design convention:** in mockups, render all three dots **same
color, different opacities** (1.0 / 0.7 / 0.4) to imply the wave
animation without making them look like three misaligned dots of
different colors.

### 5.11 Show More Toggle

Inline expand/collapse control for past Assistant bubbles that exceeded
the 160 px collapse threshold.

| Property | Value |
|---|---|
| Type | Text button (no chrome) |
| Font | Poppins 13 Medium |
| Color | `--primary` |
| Position | Below the collapsed bubble, left-aligned with bubble content |
| `padding` | `[4, 16, 0, 16]` |
| Labels | `Show more` (collapsed) ↔ `Show less` (expanded) |
| Tap target | Min 32 × 32 (inflate hit area beyond visible text) |

### 5.12 Coachmark (First-Time Hint Tooltip)

Onboarding tooltip pointing to the AI FAB. Shows once per user via
client-side persistence (`localStorage`).

| Property | Value |
|---|---|
| Pill `cornerRadius` | 10 (`chat-radius-sm`) |
| Pill `padding` | `[10, 12]` |
| Pill background | `--primary` |
| Body font | Poppins 14 Semibold, white |
| Close icon | Lucide `x`, 14, white at 70 % opacity |
| Pointer | 14 × 10 triangle path, fill `--primary`, anchored to pill bottom, offset 20 px from right edge |
| Shadow | `offset 0,4 · blur 12 · color --shadow` (base DS §7 MD equivalent) |
| Auto-dismiss | 2 s after mount, or tap to dismiss |

### 5.13 Scope Pill (Context Chip in Entry Hero)

Static info pill showing the current context (e.g. which listing, which
report).

| Property | Value |
|---|---|
| Shape | Full pill (`chat-radius-pill`, computed 17 for 34 h chip) |
| `padding` | `[6, 16, 6, 10]` (extra left for embedded icon) |
| Background | Theme-tinted (e.g. `--bg-cyan-light` for listing chatbot) |
| Border | None |
| `gap` (icon ↔ text) | 8 |
| Icon container | 22 × 22 circle, fill `--primary`, contains 16 × 16 Lucide context icon (e.g. `map-pin` for listings) |
| Text | Poppins 13 Medium, `--foreground` |
| Content | `{contextLabel}` (single line, truncate with ellipsis) |

### 5.14 Error Bubble *(per `error.code`)*

Same shape as Assistant Bubble; copy and tone vary by error code. The
full code → copy map lives in the implementing surface's i18n
namespace.

| Error code (typical) | Suggested user-facing copy direction |
|---|---|
| `ttft_timeout` | Time-to-first-token exceeded; retry-able |
| `idle_timeout` | Stream stalled mid-response; retry-able |
| `round_timeout` / `timeout` | Total round budget exceeded; retry-able |
| `rate_limit_exceeded` / `session_busy` | Conversational "busy" tone; user waits |
| `context_length_exceeded` | Ask user to shorten input |
| `auth_expired` | Refresh / re-login required |
| `safety_blocked` | Conversational redirect to in-scope topics |
| `cancelled` | User cancelled; offer retry |
| `mcp_error` / `inference_error` | Generic-class infrastructure error; retry-able |
| *default* | "Something tripped me up. Try again?" |

**Voice principle**: error copy must follow §2 ("Redirect, don't
refuse") — every error message ends with a useful next step
(retry / rephrase / contact agent / etc.), never a flat dead-end.

### 5.15 AI Entry FAB (Host Page Surface)

Floating sparkle button inside the host page's action bar. Single tap
opens the chat sheet at `half` snap.

| Property | Value |
|---|---|
| Shape | Rounded square (`chat-radius-sm`, radius 10) |
| Size | 56 × 56 (matches base DS button height for visual rhythm in the action bar) |
| Background | `--primary` |
| Icon | Phosphor `star-four`, 24 × 24, white, weight 700 |
| Host bar position | Defined by host page's action-bar layout |
| Gap from neighbor | 10 |

> **Note:** This is the *one* place chat radii do **not** apply, because
> the FAB sits *inside* the host page's action bar — a base-DS
> component. The FAB matches the radii and height of its neighbors
> (typically 10 px @ 56 h) for cohesion with the host page. Once the
> sheet opens, all chat radii apply inside the sheet.

### 5.16 Desktop Floating Popover Container

Alternative to the mobile Modal Bottom Sheet (§5.7). Used on desktop
viewports where the host page stays partially visible behind the chat
surface — context anchoring shifts from sheet-dominates-screen (mobile)
to popover-overlays-corner (desktop). Same 12-state machine, same chat
tokens; the container shell is what differs.

| Property | Value |
|---|---|
| Container | Fixed-position floating panel anchored to viewport bottom-right |
| `cornerRadius` | 16 (`chat-radius-md`, **all four corners** — not sheet-style top-only) |
| Background | `--card` |
| Width | 400 px (`max-width: calc(100vw - 48px)`) |
| Height | 600 px (`max-height: calc(100vh - 48px)`) |
| Right offset | 24 px |
| Bottom offset | 24 px |
| Drop shadow | `0 12 40 / 22%` outer — replaces the mobile scrim |
| Scrim | **None** — host page remains interactive behind the popover |
| z-index | 1000 |
| Transition | `opacity 0.22s, transform 0.22s` (rise from bottom-right with `scale(0.97)`) |
| Snap points | **None** — width/height are fixed, no drag-to-resize gesture |
| Drag bar / grabber | **None** — close exclusively via the `×` button or ESC key |
| Safe-area-inset | **N/A** — desktop has no notch / home indicator |

**FAB / panel toggle:** the desktop FAB lives at the same `bottom: 24px;
right: 24px` slot. When the panel opens, the FAB unmounts (the panel
occupies its anchor); when the panel closes, the FAB re-mounts. Single
floating element at any time, no visual stacking.

**Header chrome**: **always minimal** — only the `×` close button,
right-aligned. No divider, no scope pill. Same treatment for every chat
state, identical to the mobile sheet which has no header chrome at all.

**Why no scope pill migration**: an earlier draft of this spec moved a
scope pill from the entry hero into the header once a conversation
began, on the rationale that the popover floats over the host listing
page and the user might "lose track" of which listing the bot is
grounded on. Two reasons that rationale didn't hold up:

1. The host listing-detail page behind the popover is itself the anchor
   — title, photos, price, and map are all in the same viewport, visible
   alongside the popover at all times. A header scope pill duplicates
   information the page is already showing.
2. The popover is structurally **listing-page-bound** (`PcAiChat`
   instantiates inside `PcListing`, not at the app root), so it cannot
   accidentally "follow" the user to a different listing. There is no
   architectural path for the popover and the page-behind to disagree
   on scope.

The entry hero's scope pill (§5.13) still carries the affordance "this
chat will be grounded on this listing" before the user commits. Once
messages flow, per-bubble Source Label (§5.8) takes over as the
finer-grained grounding signal. This makes desktop and mobile
symmetric: **scope pill appears only in the entry hero on both surfaces,
never in any persistent chrome**.

**ESC dismiss**: pressing ESC closes the panel. Mobile sheet has no ESC
equivalent (touch-first interaction model).

**Mapping to 12-state machine** — same logical states as §4, only the
container shell differs:

| # | Mobile (Sheet, §5.7) | Desktop (Popover, this section) |
|---|---|---|
| 1 `trigger` | Host action bar + AI FAB | Fixed bottom-right FAB |
| 2 `hint` | Coachmark over FAB | (deferred — desktop FAB visible enough on first load) |
| 3 `halfsnap` | Sheet at 50 dvh | **N/A** — popover has no half-snap (opens directly to entry hero) |
| 4 `entry_hero` | Sheet full-snap, hero content | Popover at fixed size, hero content + minimal header (`×` only) |
| 5 `retrieving` | Sheet full-snap, hero replaced by typing indicator | Popover, hero replaced by typing indicator, header unchanged (`×` only) |
| 6 `streaming` | Same body, chrome unchanged | Same body, chrome unchanged |
| 7 `stopped` | User tapped stop | Same |
| 8 `result` | Assistant bubble landed | Same |
| 9 `show_more` | Collapsed bubble expanded | Same |
| 10 `abstained` | Abstain marker rendered | Same |
| 11 `noresult` | Error bubble inside sheet | Error bubble inside popover |
| 12 `error` | Error bubble inside sheet | Error bubble inside popover |

**Components reused inside the popover** *(no visual delta from §5.1 - §5.14)*:
Message Bubble (§5.1), Suggestion Chip (§5.2), Composer (§5.4), Send/Stop
(§5.5), AI Brand Mark (§5.6), Source Label (§5.8), Abstain Marker (§5.9),
Typing Indicator (§5.10), Show More Toggle (§5.11), Scope Pill (§5.13),
Error Bubble (§5.14). Bubble inner-text wrap, padding, gap, and shape
are byte-identical. Only the outer chrome (sheet vs popover) differs.

---

## 6. Component Composition Rules

| Inside a chat surface | Outside (host page, search, etc.) |
|---|---|
| Use this document's tokens | Use `housesigma-design-system.md` |
| Soft radii (16-24) | Tight radii (5-13) |
| Pill inputs, circular send | Rectangular inputs, square buttons |
| Phosphor `star-four` for AI brand | Lucide / custom SVG icons |
| `--muted` gray fills | `--card` white fills |
| 4 px-grid spacing OK to break for bubble feel | Strict 4 px grid |

**Where chat surface ends:**
- Sheet / Popover container's edge — anything outside (status bar, app
  header, listing content behind the scrim or behind the floating popover)
  follows base DS
- The AI FAB's container — mobile: host page's action bar; desktop:
  free-floating `fixed` slot. The FAB itself follows base DS proportions
  in both cases.

**Layering rule** (z-index):

| Layer | z-index | Surface |
|---|---|---|
| Page content | base | both |
| App header | DS-defined | both |
| Coachmark (§5.12) | above app header | mobile |
| Scrim (§5.7.2) | above all page content | mobile only — desktop popover has no scrim |
| Sheet (§5.7) | above scrim | mobile |
| Desktop Popover (§5.16) | 1000 | desktop |
| Desktop FAB | 1000 | desktop |
| Toast (base DS) | above everything | both |

---

## 7. Implementation Notes

### 7.1 Schema-agnostic design

This document specifies **visual + interaction** spec, not data schema.
Each implementing chat surface defines its own:
- Event names (`onChunk` / `onDone` / `onError` / `onProgress`)
- Message field names (`abstained` / `grounding_type` / `follow_ups`)
- Error code catalog (specific to its service backend)

Components in §5 are schema-agnostic in the following sense:
- §5.8 Source Label expects "some enum on done" — it doesn't care if
  the field is `grounding_type` or `source_kind`
- §5.9 Abstain marker expects "some boolean on done" — `abstained` or
  `is_abstain` both work
- §5.3 Follow-up chips expect "some string array on done" — `follow_ups`
  or `next_questions` both work

This keeps Chat DS reusable across multiple chat surfaces with different
backend contracts.

### 7.2 Feature flag pattern

Every component that **renders conditionally on a server-side data
field** should be gated by a feature flag, so the front-end can ship
hooks before / independently of the backend producing the data:

| Component | Gated on | Flag pattern |
|---|---|---|
| §5.8 Source Label | grounding-type field on done | `<feature>SourceLabel` |
| §5.9 Abstain marker | abstain boolean on done | `<feature>AbstainUi` |
| §5.3 Follow-up chips | follow-ups array on done | `<feature>FollowUps` |
| §5.10.2 stage labels | stage progress events | `<feature>ProgressEvent` |

When the flag is OFF, components behave byte-identically to the
"no feature" state. This is essential for safe rollout and rollback.

### 7.3 i18n contract

Every user-visible string is i18n-driven. Components in §5 do not
hardcode copy — they take their content from the implementing surface's
i18n namespace.

Each implementing surface owns:
- An `<feature>.chat.*` namespace (or equivalent)
- Translations for every visible string (entry headline, placeholder,
  chips, loading, errors, abstain marker, source labels, etc.)
- Voice consistency with §2 (assistant-active, redirect-don't-refuse)

### 7.4 Multi-language considerations

Chat copy strings vary widely by language — design with overflow
tolerance:
- Hero headline width: stretches up to 2 lines on long translations
- Chip width: prefer `fit_content`; on overflow, wrap to a new line
  rather than truncating
- Source Label width: italic, single-line, may sit on its own row if
  the bubble is short

### 7.5 Accessibility

| Element | A11y requirement |
|---|---|
| §5.5 Send button (inactive) | aria-disabled, opacity 0.35 conveys state visually |
| §5.5 Stop button | aria-label "Stop generating" |
| §5.9 Abstain marker | Visible icon + text, marker text is part of bubble accessible name |
| §5.8 Source Label | Rendered as readable text, italic style preserved for screen readers as italic |
| §5.12 Coachmark | Focus management on open + close; dismiss key (Esc) supported |
| All buttons | Min 32 × 32 tap target inflation |

### 7.6 Performance budget

| Aspect | Budget |
|---|---|
| First sheet open (cold) | < 200 ms render |
| Typing-dot animation | 1 keyframe @ 60 fps, no layout thrash |
| Bubble streaming | Append-only DOM updates; do not re-layout the whole conversation per chunk |
| Bubble collapse height measurement | Measure only after `done: true` to avoid mid-stream thrash |

---

## 8. Design Rules

### Do's

- **Do** use chat radii (16-24) on all elements inside the sheet — bubbles, chips, composer, FAB-inside-sheet
- **Do** reuse base DS color tokens (`--primary`, `--muted`, etc.) — no new color tokens
- **Do** keep the AI brand mark as Phosphor `star-four` — every other icon is Lucide
- **Do** end every limitation / error copy with a useful next step (§2)
- **Do** gate new conditional-render components on feature flags (§7.2)
- **Do** match send button to message metaphor (circular, not square)
- **Do** treat the most-recent assistant bubble as never-collapsed
- **Do** keep the sheet grabber at full pill (radius = half-height)
- **Do** render the Source Label only on `done` — never during streaming

### Don'ts

- **Don't** use base DS radii (5-13) on bubbles, chips, or composer — they read as cards / forms, not chat
- **Don't** add a border to the composer input — gray pill + no border is the chat signature
- **Don't** swap stop glyph to a filled icon (e.g. Material `stop`) — keep outlined ↔ outlined for visual continuity with send arrow
- **Don't** mix Phosphor glyphs anywhere other than the AI brand mark — Lucide everywhere else
- **Don't** show "I don't know" without a redirect — abstain must offer an alternate path
- **Don't** measure bubble height during streaming — wait for `done`
- **Don't** introduce new color tokens; if a chat-specific tint is needed, theme-extend an existing token

---

## 9. Component Inventory (cross-reference to base DS)

For every chat surface need, the following table tells you whether to
look here or in base DS.

| Need | Source | Section |
|---|---|---|
| Message bubble | Chat DS | §5.1 |
| Suggestion chip | Chat DS | §5.2 |
| Follow-up chip | Chat DS | §5.3 |
| Chat composer | Chat DS | §5.4 |
| Send / stop button | Chat DS | §5.5 |
| AI brand mark | Chat DS | §5.6 |
| Bottom sheet | Chat DS | §5.7 |
| Source label | Chat DS | §5.8 |
| Abstain marker | Chat DS | §5.9 |
| Typing dots | Chat DS | §5.10 |
| Show more toggle | Chat DS | §5.11 |
| Coachmark | Chat DS | §5.12 |
| Scope pill | Chat DS | §5.13 |
| Error bubble | Chat DS | §5.14 |
| AI entry FAB | Chat DS | §5.15 |
| Page button / icon button outside sheet | Base DS | §8.1 |
| Input field (forms) | Base DS | §8.2 |
| Tag / badge | Base DS | §8.3 / §8.5 |
| Tab | Base DS | §8.6 |
| Toast / notification | Base DS | §8.10 / §8.9 |
| App header | Base DS | §10.2 |
| Bottom nav | Base DS | §10.3 |
| Property card | Base DS | §10.4 / §9.8 |
| Typography token | Base DS | §3 |
| Color token | Base DS | §4 |
| Spacing token | Base DS | §5 |
| Shadow token | Base DS | §7 |

---

## 10. References

### Sibling specs in this repo
- `housesigma-design-system.md` — base platform DS

### Design canvas
- `chat-system-design.pen` — Variant C canvas (12 chat states +
  reusable chat components)
- `housesigma.lib.pen` — base DS component library (color / type /
  spacing tokens referenced throughout this document)

### Implementation references (per chat surface)
- Listing Chatbot: [`v2-nlqa.md`](v2-nlqa.md) — v2 NLQ&A design intent.
  Engineering proposals in [`v2-nlqa-proposals.md`](v2-nlqa-proposals.md)
  and [`v2-nlqa-surface-changes.md`](v2-nlqa-surface-changes.md) (same
  directory).

### Out-of-scope (for this document)
- Product-specific positioning (mental model shift, copy table, system
  prompt direction) — see the implementing surface's design intent doc
- Schema definitions for SSE / Kafka / WebSocket — see the implementing
  surface's protocol doc
- Worker / backend / orchestrator concerns — see the implementing
  surface's service docs
