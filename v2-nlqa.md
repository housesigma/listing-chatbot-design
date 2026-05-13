# Listing Chatbot — v2: Listing-aware NLQ&A (Design Intent)

> Design direction and change spec for the positioning shift from
> **"Look up facts from a listing"** to
> **"Listing-aware natural language Q&A"**.
>
> Version: v2 (NLQ&A) · Status: Proposal · Owner: Listing Chatbot design · Last updated: 2026-05-13
>
> Predecessor: [`v1-fact-lookup.md`](v1-fact-lookup.md) — original fact-retrieval intent.
>
> Companion documents:
> - [`v2-nlqa-spec.md`](v2-nlqa-spec.md) — chat-first component spec backing this v2 intent
> - `housesigma-design-system.md` — base platform DS
> - `listing-chatbot-design.pen` — Variant C canvas (current chat-first state)

---

## 1. Executive Summary

Listing Chatbot started as a **fact-retrieval tool** — users ask for a
specific datum and the assistant looks it up from the listing payload.
The evolution to **listing-aware natural-language Q&A** repositions
the surface as a *conversation partner* that reasons about the home,
grounds itself in listing data, but can also draw on market context,
buyer perspective, and common-sense advice.

This document captures every layer that needs to move for the shift to
land: **copy**, **modules**, **design patterns**, in priority order
P0 → P2. The base interaction shell (sheet, bubbles, composer,
streaming) stays — Variant C in the canvas is the visual foundation.

---

## 2. Positioning Shift

|  | Lookup Tool *(today)* | Listing-aware Q&A *(target)* |
|---|---|---|
| **User mental model** | "Find me a specific piece of info" | "Ask anything about this home in my own words" |
| **Question shape** | Factual, single-turn | Open-ended, conversational, multi-turn |
| **Assistant scope** | Listing payload only | Listing + market context + advisory perspective |
| **Failure mode** | "Info not in listing" (data-gap framing) | "Outside my knowledge — try this instead" (advisory redirect) |
| **Success signal** | One correct fact retrieved | A conversation that helps the user decide |
| **Voice** | System-passive ("Looking up…") | Assistant-active ("Thinking…") |
| **Value proposition** | Saves a tab switch | Replaces an exploratory call to the agent |

---

## 3. Why Now

- **LLM capability** has crossed the bar for grounded reasoning on
  property-specific questions, not just keyword retrieval.
- **Existing schema is already conversational**: `useAiAssistant.ts`
  state machine handles `abstained`, `nullReason`, `progressLabel`,
  streaming — designed for Q&A but currently fed lookup-shape prompts.
- **Production dead hooks** (`sourceLabel`, `abstained`, `nullReason`)
  exist precisely because the original architecture anticipated this
  evolution. Activating them is most of the lift.
- **User signal**: chip taps (`ai_chat_chip_click` GA) cluster on
  open-ended chips like *"Tell me about this home"* far more than
  factual ones like *"How long has it been listed?"* — early evidence
  that users want conversation, not retrieval.

---

## 4. Scope

### 4.1 What changes
- All entry-point copy (hero / placeholder / coachmark)
- Loading and progress copy
- Suggestion chip composition
- System prompt
- Abstain / no-result message framing
- Several i18n keys (rename, not just retranslate)
- A handful of new schema fields (`sourceLabel`, `followUps`,
  `groundingType`)
- 3 new UI patterns (follow-up chips, source label, abstain pattern)

### 4.2 What stays
- Sheet structure (half-snap → full-snap auto-expand)
- Bubble geometry, composer pill input, circular send/stop
- Streaming protocol (`useChatStream.ts` is already general-purpose)
- Error code catalog (just tone-shift on copy)
- GA event names for the core flow (additive new events only)
- Variant C as the canonical visual baseline

---

## 5. Copy Changes

| Module | Today (lookup) | Target (NLQ&A) | Source / i18n key |
|---|---|---|---|
| **Hero headline** | "Look up details in this listing" | **"Ask anything about this home"** | `aiChat.entryLabel` → rename to `askAnything` |
| **Composer placeholder** | "Look up details in this listing…" | **"Ask anything about this home…"** | derived `entryLabel + '…'` |
| **Coachmark tooltip** | "Look up listing details" | **"Ask about this home"** | `aiChat.hintTitle` → rename to `askAboutThis` |
| **Loading — default** | "Looking up details…" | **"Thinking…"** | `aiChat.lookingUp` → rename to `thinking` |
| **Loading — slow (≥10s)** | "Still looking…" | **"Still thinking…"** | `aiChat.stillLooking` → rename to `stillThinking` |
| **Worker progress stages** | Generic loading copy | Stage-specific:<br>`Reading the listing…`<br>`Considering the question…`<br>`Composing your answer…` | `useChatStream.ts` ProgressEvent — server emits per stage |
| **Suggestion chip 1** | "Tell me about this home" | **Keep** — broadest NLQ&A entry | `aiChat.chipAbout` |
| **Suggestion chip 2** | "How long has it been listed?" | **"How does it compare to nearby listings?"** | `aiChat.chipListed` → `chipCompare` |
| **Suggestion chip 3** | "What schools are nearby?" | **"Anything I should be cautious about?"** | `aiChat.chipSchools` → `chipCautions` |
| **No-result fallback** | "School info isn't included. Try asking about bedrooms, bathrooms, square footage, or HOA fees." | **"I'm not sure about that one. Want me to try a related angle? You could also ask the listing agent."** | New string `noResultGeneric` |
| **Abstain copy (LLM-generated)** | "I can't determine the seller's bottom-line price from the listing. Asking price is $524,886." | **"Listing data alone won't tell us — but recent comparable sales might. Want me to compare?"** *(LLM-side; tuned via system prompt)* | server-side `abstained=true` payload |
| **Safety blocked** | "This question can't be answered. Please rephrase or ask something else." | **"Let's stay focused on this listing — happy to help with anything about the home."** | `aiChat.errorSafetyBlocked` |
| **Error generic** | "Something went wrong while generating a response. Please try again." | **"Something tripped me up. Try again?"** | `aiChat.errorGeneric` |
| **Error busy** | "I'm a bit busy right now — please try again in a moment." | **Keep** — already conversational | `aiChat.errorBusy` |

### Voice principles
- **First-person assistant** ("I", "me") not system-passive ("the
  service")
- **Active verbs** (`Thinking`, `Considering`) not passive (`Looking
  up`, `Fetching`)
- **Redirect not refuse** — every limitation copy ends with a useful
  next step

---

## 6. Module / Code Changes

| Layer | Change | File / Owner |
|---|---|---|
| **i18n keys** | Rename `entryLabel`, `lookingUp`, `stillLooking`, `hintTitle`, `chipListed`, `chipSchools`. Add `thinking`, `stillThinking`, `askAboutThis`, `chipCompare`, `chipCautions`, `noResultGeneric` | `packages/common/i18n/translation/{en,zh}.ts` |
| **System prompt** | Rewrite from "answer using listing data only" → "reason about the home; ground answers in listing data first, may incorporate market context / buyer perspective; abstain helpfully with a redirect, not flatly" | Server-side worker config |
| **Revive `sourceLabel`** | Schema and render hook exist (`AppAiChatMessages.vue:46-51`); state machine never writes. Add server-side emission + `useAiAssistant.ts:255-262` `onDone` writes `m.sourceLabel` | `useAiAssistant.ts` ChatMessage + `useChatStream.ts` `DonePayload` |
| **New field `followUps`** | Server emits 2-3 follow-up question strings on `onDone`. Frontend renders as Follow-up Suggestion Chips below assistant bubble | `useChatStream.ts` `DonePayload` + new `AppAiChatFollowUps.vue` |
| **New field `groundingType`** | Enum: `from_listing` / `from_market_context` / `general_advice` / `assumption`. Drives Source Label rendering + Confidence Indicator | `useChatStream.ts` `ChunkPayload` |
| **Activate `abstained` UI** | Schema is wired through end-to-end (`useAiAssistant.ts:42-44`, `258-259`) but no UI consumes it. Add Abstain Pattern (see §8.4) | `AppAiChatMessages.vue` |
| **New Chat reset** | `useAiAssistant.ts` already exposes `setMessages([])` — no UI entry today. Add header button | `AppAiChatSheet.vue` — sheet header |
| **GA additive events** | `ai_chat_follow_up_click` (with chip text label), `ai_chat_new_conversation`, `ai_chat_grounding_seen` (with type) | `ga.service` |
| **Stop-button aria-label** | "Stop generating" → keep (still accurate) | `AppAiChatComposer.vue:17` |

---

## 7. Design Changes

Built on top of Variant C (canvas Frame 4) and
[`v2-nlqa-spec.md`](v2-nlqa-spec.md) chat-radius tokens.

### 7.1 Entry hero — chip composition

| Today | Target |
|---|---|
| 3 factual chips | 1 broad + 1 comparative + 1 advisory:<br>• Tell me about this home<br>• Compare to nearby listings<br>• Things to watch out for |

Chip *shape* stays — full pill, `--card` fill, 1px `--border`,
cornerRadius `chat-radius-pill`. Only the labels (and i18n keys)
change.

### 7.2 Hero sparkle and headline

No visual change. Copy update only ("Ask anything about this home"
replaces the headline). Phosphor `star-four` AI Brand Mark continues
to serve as the persistent visual identity.

### 7.3 Loading bubble — stage-aware progress label

The typing-dots bubble already accepts a `progressLabel` prop. Visual
unchanged; copy becomes stage-aware:
1. **t=0 → first progress event**: `Thinking…`
2. **first ProgressEvent `intent_classification`**: `Reading the
   listing…`
3. **first ProgressEvent `response_rendering`**: `Composing your
   answer…`
4. **t ≥ 10s with no progress**: `Still thinking…`
5. **First chunk arrives**: dots disappear, streaming begins

Server contract: emit at least 2 `ProgressEvent`s per round so the
label is never stuck at `Thinking…`.

---

## 8. New Design Patterns

These are new components to add to [`v2-nlqa-spec.md`](v2-nlqa-spec.md)
as additional sections (proposed §17-21).

### 8.1 Source / Grounding Label

A small italic teal label rendered as a sibling **below** the
assistant bubble. Tells the user *what kind of source* the answer is
grounded in.

| Property | Value |
|---|---|
| Position | Sibling of `messageBubble`, inside `messageGroup` |
| Layout | Inline-block, margin-top 4, margin-left 16 |
| Font | Poppins 13 Italic, `--primary` |
| Content map | `from_listing` → "Based on this listing"<br>`from_market_context` → "Based on market comparison"<br>`general_advice` → "General guidance"<br>`assumption` → "Inferred — verify before relying" |
| Visibility | Render only when `groundingType` is set; render only on `done` (no flicker during streaming) |

Implementation: revives the dead `sourceLabel` hook with a richer
schema (`groundingType` enum instead of free-form string).

### 8.2 Confidence Indicator (optional, P2)

Tiny inline indicator at the start of the source label, distinguishing
verifiable claims from advisory opinion.

| Glyph | When |
|---|---|
| ● (filled dot, primary) | `from_listing` — verifiable |
| ○ (outlined dot, muted) | `from_market_context` — derived |
| △ (triangle, amber) | `general_advice` / `assumption` |

Only ship if user research shows source label alone is insufficient.

### 8.3 Follow-up Suggestion Chips

Rendered below the **most-recent** assistant bubble (only the most
recent — past bubbles' follow-ups disappear once a new round starts).

| Property | Value |
|---|---|
| Container | Horizontal flex, wrap, gap 8, padding-top 8 |
| Chip count | 2-3 (server returns up to 3) |
| Shape | Same as entry-hero Suggestion Chip but smaller |
| Size | Height 28-30 (vs entry hero 32-36) |
| Font | Poppins 12 Regular, `--foreground` |
| Background | `--card` |
| Border | 1px `--border` |
| Radius | `chat-radius-pill` (half-height for full pill) |
| Behavior | Tap → injects chip text as new user message; chips fade out as round starts |
| GA | `ai_chat_follow_up_click` with chip text |

Server contract: `onDone.followUps?: string[]` (up to 3 strings).

### 8.4 Abstain Pattern (revived)

When `abstained: true` arrives, render the assistant bubble with a
small inline marker above the content.

| Element | Spec |
|---|---|
| Marker row | Above bubble content, inside bubble padding-top |
| Marker icon | Lucide `info` 12×12, `--muted-foreground` |
| Marker text | "Outside my listing knowledge" — Poppins 11 Medium, `--muted-foreground` |
| Bubble content | LLM-generated redirect text (per §5 abstain copy direction) |
| Optional `nullReason` | Rendered as Poppins 10 italic below bubble if non-null and human-readable |

This explicitly differs from §6's *error* bubble — abstain means the
assistant chose not to answer; error means the system failed.

### 8.5 New Chat Button

Sheet-header trailing action to clear history and return to entry
hero.

| Property | Value |
|---|---|
| Position | Top-right of sheet, outside drag-grabber area |
| Tap target | 32 × 32 |
| Icon | Lucide `message-circle-plus` 18×18, `--primary` |
| Visibility | Hidden when `messages.length === 0` |
| Action | Clears `messages`, resets snap to `half`, focuses composer |
| GA | `ai_chat_new_conversation` |

### 8.6 Idle Re-engagement Prompt (P2)

After 30 s of conversation idleness with `hasMessages === true`, show
a muted "Still curious?" line above the composer with 2-3 fresh
follow-up chips. Dismisses on any user action.

---

## 9. System Prompt Direction

(Owner: AI team — design provides intent, not the prompt text.)

Shift the worker's system message to:

- **Frame the assistant as a knowledgeable conversation partner**, not
  a JSON-key lookup function.
- **Authorize reasoning beyond the payload**: market context,
  comparable sales (when available), general buyer-perspective advice,
  common-sense observations.
- **Mandate redirect-shape abstention**: if the answer truly can't be
  formed, return `abstained: true` AND include a helpful
  alternative — never a flat "I don't know."
- **Emit `groundingType` per round**: classify the *primary* basis of
  the answer.
- **Emit `followUps`** *(up to 3)*: phrase as natural questions the
  user might ask next, tied to the conversation thread.

---

## 10. Priority Tiers & Roadmap

### P0 — Cognitive Reframing (Week 1)
Users see this surface as conversational starting day 1.

- [ ] Copy: hero, placeholder, coachmark, loading default & slow
- [ ] Suggestion chips: rebalance to 1 broad / 1 comparative / 1
  advisory
- [ ] System prompt rewrite
- [ ] i18n key renames

**Acceptance:** A first-time user sees "Ask anything about this home,"
not "Look up details." Chip taps reflect more open-ended questions in
GA (`ai_chat_chip_click` event payload).

### P1 — Trust & Extensibility (Week 2-3)
Users can tell where answers come from; conversations don't
dead-end.

- [ ] `sourceLabel` + `groundingType` end-to-end: server emit → state
  machine write → UI render (§8.1)
- [ ] Follow-up Suggestion Chips (§8.3)
- [ ] Abstain Pattern revival (§8.4)
- [ ] Server stage-specific progress labels (§7.3)

**Acceptance:** Every assistant bubble carries a Source Label;
non-trivial answers carry ≥2 follow-up chips; abstained rounds
redirect instead of dead-ending.

### P2 — Deep Conversation Support (Week 4+)

- [ ] New Chat Button (§8.5)
- [ ] Idle Re-engagement Prompt (§8.6)
- [ ] Confidence Indicator (§8.2) — gate on user research
- [ ] Half-snap min-height bumped 450 → 500 to fit longer answers
- [ ] Markdown rich-content validation (lists, tables, inline links in
  long answers)

**Acceptance:** Average conversation depth (rounds per session)
trends up; abandonment after first answer trends down.

---

## 11. Open Questions

- **Source label granularity**: 4 enum values enough? Or do we need a
  separate "verified by HouseSigma data" tier for things like
  AVM-backed claims?
- **Follow-up chip generation cost**: server-side LLM call adds
  latency to `onDone`. Acceptable budget?
- **Abstention discoverability**: does showing "Outside my listing
  knowledge" lower trust, or raise it? A/B candidate.
- **AI Avatar / persona mark**: should bot bubbles carry a small
  sparkle prefix? Trade-off: identification vs visual noise. Defer to
  user testing.
- **Voice input** affordance — not in scope for this evolution, but
  worth flagging for the next iteration.
- **Multi-listing memory**: if a user asks "how does this compare to
  the one I looked at yesterday?" — out of scope, but a strong signal
  user already expects more than single-listing Q&A.

---

## 12. Risk & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| LLM hallucinates beyond listing data ("this neighborhood has the best schools" without evidence) | Medium | High — trust | `groundingType` UI surfaces uncertainty; system prompt mandates "Inferred — verify" tag for assumptions |
| Cost spike from longer answers + follow-up generation | High | Medium | Cap answer length; cache follow-up chips per round; sample LLM call rate |
| Users get accustomed to NLQ&A and ask out-of-scope questions ("what's the weather?") | Medium | Low | `errorSafetyBlocked` copy redirects; conversational tone keeps users in scope |
| Existing factual users get worse experience (answer too verbose) | Low | Medium | System prompt: "match answer length to question specificity" |
| Dead-hook revival breaks production builds | Low | High | Feature flag (`listingChatbotNLQA`) to gate the new render paths |

---

## 13. References

### Production code
- `packages/app/src/pages/listing/components/AppAiChatSheet.vue`
- `packages/app/src/pages/listing/components/AppAiChatHero.vue`
- `packages/app/src/pages/listing/components/AppAiChatMessages.vue`
- `packages/app/src/pages/listing/components/AppAiChatComposer.vue`
- `packages/app/src/pages/listing/components/AppAiHintPopup.vue`
- `packages/hook/ai/useAiAssistant.ts` *(`ChatMessage`,
  `friendlyErrorMessage`, `stop`, `handleSubmit`)*
- `packages/hook/ai/useChatStream.ts` *(`ChunkPayload`,
  `DonePayload`, `ProgressFieldPayload`)*
- `packages/common/i18n/translation/en.ts` *(`aiChat.*` namespace)*

### Design artifacts
- [`v2-nlqa-spec.md`](v2-nlqa-spec.md) — chat-first component spec
  (companion)
- `housesigma-design-system.md` — base platform DS
- `listing-chatbot-design.pen` — design canvas:
  - Frame 1: Variant A (Production Mirror — current)
  - Frame 2: Variant B (DS-Compliant — rejected path)
  - Frame 3: Chat Wireframe Annotation Reference
  - Frame 4: Variant C (Chat-First — current canonical)

### Companion specs to author next
- [`v2-nlqa-spec.md`](v2-nlqa-spec.md) §17-21: Source/Grounding Label,
  Confidence Indicator, Follow-up Suggestion Chips, Abstain Pattern,
  New Chat Button
- A separate **system prompt design doc** (AI team owns content;
  design owns intent / tone direction)
