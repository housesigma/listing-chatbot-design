# Conversational Components

> Standalone spec for the chat / AI assistant surface of HouseSigma.
>
> **Relationship to `housesigma-design-system.md`:** This document is a *sister*
> spec, not a subset. Chat UI follows industry conversational conventions
> (iMessage / WhatsApp / ChatGPT / Claude / Bard family) that intentionally
> diverge from the listing-platform DS in radius, fill, and shape language.
> Where the base DS and this document disagree, **this document wins inside
> any chat / AI assistant surface**.
>
> **Tokens:** This document re-uses the base DS color tokens (`--primary`,
> `--muted`, `--foreground`, etc.) wherever possible — no new color tokens are
> introduced. Departures are limited to **shape (radius), fill semantics, icon
> family, and component composition.**
>
> Last updated: 2026-05-13.
> Maintainer: Listing Lens design.

---

## 1. Philosophy

Conversational UI carries different visual cues than transactional UI:

| Transactional (base DS) | Conversational (this spec) |
|---|---|
| Cards, forms, filters | Bubbles, scrolling history, live composer |
| Discrete, single-action | Continuous, multi-turn |
| Tight, gridded radii (5/8/10/13) | Soft, organic radii (16-24) |
| Filled inputs with borders | Borderless pill inputs |
| Square icon buttons | Circular send button |
| One-shot interactions | Streaming, abortable, async |

If you find yourself fighting the base DS to make something feel "more chat,"
that's the signal to pull from this document instead.

---

## 2. Conversational Radius Scale

Chat surfaces use a parallel scale calibrated for **soft, tactile** geometry.
Do not mix with base DS radii inside the chat surface.

| Token | Value | CSS Variable | Usage |
|---|---|---|---|
| **chat-radius-xs** | 2px | `--chat-radius-xs` | Sheet grabber, system indicators (full pill on 4px tall element) |
| **chat-radius-sm** | 10px | `--chat-radius-sm` | Coachmark / tooltip body |
| **chat-radius-md** | 16px | `--chat-radius-md` | Message bubble main corners (alternate) |
| **chat-radius-lg** | 20px | `--chat-radius-lg` | Message bubble main corners (default), Modal Bottom Sheet top |
| **chat-radius-pill** | half-height | `--chat-radius-pill` | Suggestion chips, scope pill, conversational input field, send/stop button (full round) |
| **chat-tail-radius** | 4px | `--chat-tail-radius` | Bubble origin-corner (creates speech-bubble tail) |

**Why depart from base DS §6:**
- Base DS caps at 13px (XL). Bubbles at 13 read as "cards stacked," not as
  conversation. 20px reads as "bubble" — the universal chat metaphor.
- Base DS pill rule says ">=34" for full pill — that's calibrated for toggle
  tracks (24h) and toast (40h). For chip-sized elements (28-36h) a pill is
  achieved at half-height (14-18) which falls in the gap. We resolve by
  naming the *intent* (`pill`) instead of a fixed value.

---

## 3. Message Bubble

Carries one message in a conversation. Renders inside `Conversation Pane`.

### 3.1 Geometry

| Property | Value |
|---|---|
| `cornerRadius` (User) | `[20, 20, 4, 20]` — main `chat-radius-lg`, tail (bottom-right) `chat-tail-radius` |
| `cornerRadius` (Assistant) | `[20, 20, 20, 4]` — main `chat-radius-lg`, tail (bottom-left) `chat-tail-radius` |
| `padding` | `[12, 16]` |
| `maxWidth` | 80% of conversation pane width |
| `gap` (multi-line content) | 4 |

The **4px tail** is intentional: it keeps the soft bubble shape while pointing
"this came from me / them." Square tails (radius 0) feel sliced; symmetric
radii lose the chat metaphor.

### 3.2 Fill & Text

| Role | Background | Text Color | Font |
|---|---|---|---|
| **User** | `--primary` | `--primary-foreground` (white) | Poppins 14 Regular |
| **Assistant** | `--muted` (#F2F2F2) | `--foreground` (#333) | Poppins 14 Regular (Markdown-rendered) |
| **Error (within Assistant)** | `--muted` | `--destructive` text inline | Poppins 14 Regular |

### 3.3 Behavior

- **Past Assistant bubble** with `scrollHeight > 160px` is auto-collapsed
  to 160px with a linear gradient fade (transparent → `--muted`) over the
  bottom 40px. Followed by a `Show More Toggle` row.
- **Most-recent Assistant bubble** is **never collapsed**, regardless of
  height — let the user read the freshest answer in full.
- **Streaming Assistant bubble** is never collapsed and never shows the
  Show More toggle (`done: false`).

---

## 4. Suggestion Chip

Quick-reply pill that lives in the Entry Hero. Tapping submits the chip text
as a user message.

| Property | Value |
|---|---|
| Shape | **Full pill** (`chat-radius-pill`) |
| `cornerRadius` (computed) | 22 (for 32-36h chip) |
| `padding` | `[8, 16]` |
| Background | `--card` (white) |
| Border | 1px `--border` (#D9D9D9) |
| Font | Poppins 14 Regular, `--foreground` |
| Pressed state | Background `--background-section` (#F6F6F6), border `#CCCCCC` |
| Layout | Stacked vertically, gap 12, centered |
| Count | Typically 3 (matches production `chipAbout / chipListed / chipSchools`) |

**Why pill, not DS Badge §8.5 (4px radius):** badges signal selectable
filter state. Chips signal "tap to send conversation" — pill shape is the
universal industry cue (Apple Smart Reply, Gemini suggestions, Perplexity
prompt starters all use pills).

---

## 5. Conversational Input (Composer Text Field)

The composer's text field. Departs significantly from DS §8.2 Input Fields.

| Property | Value | DS §8.2 (rejected) |
|---|---|---|
| Shape | **Full pill** | 5px radius rectangle |
| Background | `--muted` (#F2F2F2) — soft gray | `--input-bg` (white) |
| Border | **None** | 1px `--border` |
| `padding` | `[0, 16]` | varies |
| Height | 48 | 47 |
| Font | Poppins 14 Regular | 14 Regular |
| Placeholder color | `--muted-foreground` (#808080) | `--input-placeholder` (#B3B3B3) |
| Width | `fill_container` minus send button | 349 fixed |

**Why depart from §8.2:** the production code (`AppAiChatComposer.vue:83-101`)
uses these exact values. Borderless gray pill is what users recognize as
"chat input" — borders + white triggers form-fill mental model.

### 5.1 States

| State | Visual |
|---|---|
| **Idle** | Pill with placeholder text |
| **Typed** | Pill with user text in `--foreground` |
| **Disabled (loading)** | Container at 60% opacity, input disabled, placeholder remains visible |

---

## 6. Send / Stop Button Pair

A single 48x48 button that swaps glyph based on round state. Lives at the
right of the Composer.

| Property | Value |
|---|---|
| Shape | **Circular** (`chat-radius-pill` = 24) |
| Size | 48 × 48 |
| Background | `--primary` (#28A3B3) |
| Icon family | **Lucide** (consistent across both states) |
| Send glyph | `arrow-up`, 20×20, `--primary-foreground` |
| Stop glyph | `square`, 20×20, `--primary-foreground`, weight 700 (filled-feel) |

### 6.1 States

| State | Glyph | Opacity |
|---|---|---|
| **Inactive** (input empty, idle) | `arrow-up` | 0.35 |
| **Active** (input has text, idle) | `arrow-up` | 1.0 |
| **Loading** (round in flight) | `square` (stop) | 1.0 |

**Why circular, not DS §8.1 (10px square button):** the send button is the
*single* persistent action of the composer — it earns FAB-like prominence
through shape. A rounded square reads as "submit a form," a circle reads as
"send a message." Industry convention is unanimous (every major chat client).

**Why Lucide for both glyphs, not Material Symbols `stop`:** keeping a
unified icon family (Lucide outlined) makes the send ↔ stop swap feel like
one button changing state, not two buttons trading places. Outlined ↔
outlined retains visual continuity that filled-glyph swaps break.

---

## 7. AI Brand Mark

The decorative AI/sparkle icon that anchors the Entry Hero and the FAB.

| Property | Value |
|---|---|
| **Icon** | **Phosphor `star-four`** (single 4-point star) |
| Weight | 700 (filled) |
| Size (Hero) | 32 |
| Size (FAB inside) | 24 |
| Color | `--primary` (hero) / `--primary-foreground` (inside teal FAB) |

**Why phosphor `star-four`, not Lucide `sparkles`:** the 4-point star is the
2024-25 GenAI category mark (OpenAI / Anthropic / Google AI / Notion AI /
Linear AI / every AI-feature toggle in the SaaS world uses a variant). Lucide
`sparkles` is decorative multi-glint — semantically "celebration / magic,"
not "AI." Branding fidelity > icon-family consistency.

**This is the one icon-family exception in the chat surface.** Everywhere
else (composer, navigation, controls) use Lucide.

---

## 8. Modal Bottom Sheet

Container for the conversation. Two snap points.

### 8.1 Geometry

| Property | Value |
|---|---|
| `cornerRadius` (top corners only) | `[20, 20, 0, 0]` (chat-radius-lg) |
| Background | `--card` (white) |
| Width | `100%` of viewport |
| Bottom anchor | `0` (always sticks to viewport bottom) |
| Snap points | `half` (50dvh, min 450px) · `full` (vh − 50 mobile web / vh hybrid) |
| Sheet z-index | Above `Scrim` |
| Transition | `height 0.3s ease, top 0.3s ease` |

### 8.2 Sheet Grabber

| Property | Value |
|---|---|
| Width | 40 |
| Height | 4 |
| `cornerRadius` | 2 (`chat-radius-xs` — full pill on 4h element) |
| Color | `--border` (#D9D9D9) |
| Container padding | 12 top / 4 bottom |
| Gesture | Drag to resize between snap points; drag below 60% half-height closes |

**Why 2, not DS SM=5:** 5px radius on a 4px-tall element looks like a sharp
divider tab, not a draggable indicator. Full pill (radius = half-height)
matches iOS and Material Bottom Sheet conventions.

### 8.3 Scrim / Backdrop

| Property | Value |
|---|---|
| Coverage | Full viewport, behind sheet |
| Color | `--shadow` (rgba(0,0,0,0.4)) |
| Dismiss | Tap to close |
| z-index | Below sheet, above page |

---

## 9. Typing Indicator (Loading Bubble)

Same outer shape as an Assistant Message Bubble. Renders during the gap
between user send and first content chunk.

| Property | Value |
|---|---|
| `cornerRadius` | `[20, 20, 20, 4]` (same as Assistant Bubble) |
| Background | `--muted` |
| `padding` | `[12, 16]` |
| `gap` (dots ↔ label) | 8 |

### 9.1 Animated Dots

| Property | Value |
|---|---|
| Count | 3 |
| Size | 6 × 6 ellipses |
| Spacing (gap) | 4 |
| Color | `--muted-foreground` (#808080) — **all three same color** |
| Animation | `dotPulse 1.4s ease-in-out infinite`, stagger delay 0/200/400ms, scale 0.6 ↔ 1.0, opacity 0.4 ↔ 1.0 |

### 9.2 Progress Label

| Property | Value |
|---|---|
| Font | Poppins 12 Regular |
| Color | `--muted-foreground` |
| Default copy | `Looking up details...` |
| ≥ 10s elapsed | `Still looking...` |
| When server `ProgressEvent` arrives | Replace with worker stage message (e.g. `Analyzing your request...`, `Composing your answer...`) |

**Static design convention:** in mockups, render all three dots the **same
color, different opacities** (1.0 / 0.7 / 0.4) to imply the wave animation
without making them look like three misaligned dots of different colors.

---

## 10. Show More Toggle

Inline expand/collapse control for past Assistant bubbles that exceeded the
160px collapse threshold.

| Property | Value |
|---|---|
| Type | Text button (no chrome) |
| Font | Poppins 13 Medium |
| Color | `--primary` |
| Position | Below the collapsed bubble, left-aligned with bubble content |
| `padding` | `[4, 16, 0, 16]` |
| Labels | `Show more` (when collapsed) ↔ `Show less` (when expanded) |
| Tap target | Min 32 × 32 (inflate hit area beyond visible text) |

---

## 11. Coachmark (First-Time Hint Tooltip)

Onboarding tooltip pointing to the AI FAB. Shows once per user
(`localStorage.hint_ai_chat_viewed`).

| Property | Value |
|---|---|
| Pill `cornerRadius` | 10 (`chat-radius-sm`) |
| Pill `padding` | `[10, 12]` |
| Pill background | `--primary` |
| Body font | Poppins 14 Semibold, white |
| Close icon | Lucide `x`, 14, white at 70% opacity |
| Pointer | 14×10 triangle path, fill `--primary`, attached to pill bottom, offset 20px from right edge |
| Shadow | `offset 0,4 · blur 12 · color --shadow` (DS §7 MD equivalent) |
| Auto-dismiss | 2s after mount, or tap to dismiss |

---

## 12. Scope Pill (Context Chip in Entry Hero)

Static info pill showing the current listing context.

| Property | Value |
|---|---|
| Shape | **Full pill** (`chat-radius-pill`) — computed radius = half-height |
| `padding` | `[6, 14, 6, 10]` (extra left for embedded icon) |
| Background | `--muted` |
| Border | 1px `--border-light` (#EAEAEA) |
| `gap` (icon ↔ text) | 8 |
| Icon container | 22 × 22 circle, fill `--primary`, contains 16×16 Lucide `map-pin` (white) |
| Text | Poppins 12 Medium, `--muted-foreground` |
| Content | `{address}{city ? ` · ${city}` : ''}` (single line, truncate with ellipsis) |

---

## 13. AI FAB (Entry Trigger)

Floating sparkle button inside the Bottom Action Bar of the listing detail
page. Single tap opens the chat sheet at `half` snap.

| Property | Value |
|---|---|
| Shape | Rounded square (`chat-radius-sm` = 10) |
| Size | 56 × 56 (matches DS §8.1 button height for visual rhythm in the action bar) |
| Background | `--primary` |
| Icon | Phosphor `star-four`, 24×24, white, weight 700 |
| Action bar position | Right-most of three buttons (Watch · Schedule Viewing · AI) |
| Gap from neighbor | 10 |

**Note:** This is the *one* place chat radii do **not** apply, because the
FAB sits *inside* the listing platform's Bottom Action Bar — a base-DS
component. The FAB matches the radii and height of its neighbors (Watch
outline / Schedule Viewing filled, both 10px @ 56h) for cohesion with the
listing page. Once the sheet opens, all chat-radii apply inside the sheet.

---

## 14. Component Composition Rules

| Inside a chat surface | Outside (listing detail, search, etc.) |
|---|---|
| Use this document's tokens | Use `housesigma-design-system.md` |
| Soft radii (16-24) | Tight radii (5-13) |
| Pill inputs, circular send | Rectangular inputs, square buttons |
| Phosphor `star-four` for AI brand | Lucide / custom SVG icons |
| `--muted` gray fills | `--card` white fills |
| 4px-grid spacing OK to break for bubble feel | Strict 4px grid |

**Where chat surface ends:**
- Sheet container's edge — anything outside the sheet (status bar, app
  header, listing content behind the scrim) follows base DS.
- The AI FAB's container (Bottom Action Bar) — the FAB itself follows base
  DS for cohesion with sibling CTAs.

---

## 15. Open Questions / Follow-ups

- **AI Avatar / persona mark:** should Assistant bubbles carry a small (e.g.
  20px) brand mark to the left to make AI attribution explicit? Production
  doesn't render one today.
- **Voice input affordance:** a microphone glyph next to send is common in
  consumer chat apps; HouseSigma's hint set is currently text-only.
- **Inline source citations:** the `sourceLabel` rendering hook exists in
  `AppAiChatMessages.vue:46-51` but is never set. If the source attribution
  shipper, this spec should define the visual placement (under-bubble vs
  inline).
- **Abstention badge:** the `abstained` / `nullReason` schema is wired all
  the way through `useAiAssistant.ts` but no UI consumes it. If a future
  release renders it, decide whether it's an inline badge, a footer line,
  or a state on the bubble itself.

---

## 16. References

- Production source of truth:
  - `packages/app/src/pages/listing/components/AppAiChatSheet.vue`
  - `packages/app/src/pages/listing/components/AppAiChatHero.vue`
  - `packages/app/src/pages/listing/components/AppAiChatMessages.vue`
  - `packages/app/src/pages/listing/components/AppAiChatComposer.vue`
  - `packages/app/src/pages/listing/components/AppAiHintPopup.vue`
  - `packages/hook/ai/useAiAssistant.ts`
  - `packages/hook/ai/useChatStream.ts`
- Sibling spec: `housesigma-design-system.md` (base platform DS)
- Design canvas: `listing-lens.pen` — Frame 4 (Variant C — Chat-First)
