# Listing Chatbot — v2: Listing-aware NLQ&A

> Design intent + proposal for the positioning shift from
> **"Look up facts from a listing"** to
> **"Listing-aware natural-language Q&A"**.
>
> Version: v2 (NLQ&A) · Status: Proposal · Owner: Listing Chatbot design · Last updated: 2026-05-14
>
> Companion artifacts (this repo):
> - `listing-chatbot-design.pen` — design canvas (Variants A/B/C, Variant C is canonical)
> - `chat-design-system.md` + `chat-system-design.pen` — Chat Design System proposal that this doc adopts as the visual baseline

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
streaming) stays — Variant C in `listing-chatbot-design.pen` is the
visual foundation, and the component spec lives in
[`chat-design-system.md`](chat-design-system.md).

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

## 3. Variants Explored

Three visual variants were explored on the canvas
(`listing-chatbot-design.pen`). They are kept side-by-side for
contrast, not as alternates.

- **Variant A — Production Mirror.** Faithful copy of current
  production. Baseline reference; not a forward proposal.
- **Variant B — DS-Compliant.** Forced every chat element through
  base DS radii / button shapes / icon family. Result: composer becomes
  a search form, send becomes a generic submit button, bubbles become
  cards, suggestion chips become filter badges. **Rejected** — the
  surface no longer reads as chat.
- **Variant C — Chat-First.** Canonical. Follows the Chat Design System
  ([`chat-design-system.md`](chat-design-system.md)). Visually almost
  identical to Variant A but every value comes from a named token,
  every shape from a documented rule, every icon from an explicit
  decision. **Variant C codifies what production already intuitively
  does right** and provides the spec foundation for further evolution.

The rejection of Variant B is the clearest case for treating chat as a
sister surface to the base DS rather than a strict subset — see
`chat-design-system.md` §1 for the full sister-DS argument.

---

## 4. Design Principles

| Principle | Manifestation |
|---|---|
| **Conversational over transactional** | Soft radii (16-24) over tight radii (5-13). Pill inputs over rectangular forms. Circular send button over square submit. |
| **Assistant-active, not system-passive voice** | "Thinking…" not "Looking up…". "Something tripped me up" not "Generation failed." |
| **Redirect, don't refuse** | Every limitation copy ends with a useful next step (alternate path / agent / comparable sales). |
| **Trust through transparency** | Source / Grounding Label visible on every assistant bubble — users see whether the answer is from listing data, market context, or general advice. |
| **Multi-turn by default** | Follow-up chips after every assistant response. Sheet auto-expands when the conversation starts. |
| **Brand-identifiable AI mark** | Phosphor `star-four` (the GenAI category convention) — one explicit deviation from base DS icon library. |

These principles mirror `chat-design-system.md` §2 because the chat
DS is the source of truth for visual / interaction stance. They are
restated here so the listing-chatbot proposal can be read on its own.

---

## 5. Why Now

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

## 6. Scope

### 6.1 What changes
- All entry-point copy (hero / placeholder / coachmark)
- Loading and progress copy
- Suggestion chip composition
- System prompt
- Abstain / no-result message framing
- Several i18n keys (rename, not just retranslate)
- A handful of new schema fields (`sourceLabel`, `followUps`,
  `groundingType`)
- 3 new UI patterns (follow-up chips, source label, abstain pattern)

### 6.2 What stays
- Sheet structure (half-snap → full-snap auto-expand)
- Bubble geometry, composer pill input, circular send/stop
- Streaming protocol (`useChatStream.ts` is already general-purpose)
- Error code catalog (just tone-shift on copy)
- GA event names for the core flow (additive new events only)
- Variant C as the canonical visual baseline

---

## 7. Copy Changes

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

## 8. Module / Code Changes

| Layer | Change | File / Owner |
|---|---|---|
| **i18n keys** | Rename `entryLabel`, `lookingUp`, `stillLooking`, `hintTitle`, `chipListed`, `chipSchools`. Add `thinking`, `stillThinking`, `askAboutThis`, `chipCompare`, `chipCautions`, `noResultGeneric` | `packages/common/i18n/translation/{en,zh}.ts` |
| **System prompt** | Rewrite from "answer using listing data only" → "reason about the home; ground answers in listing data first, may incorporate market context / buyer perspective; abstain helpfully with a redirect, not flatly" | Server-side worker config |
| **Revive `sourceLabel`** | Schema and render hook exist (`AppAiChatMessages.vue:46-51`); state machine never writes. Add server-side emission + `useAiAssistant.ts:255-262` `onDone` writes `m.sourceLabel` | `useAiAssistant.ts` ChatMessage + `useChatStream.ts` `DonePayload` |
| **New field `followUps`** | Server emits 2-3 follow-up question strings on `onDone`. Frontend renders as Follow-up Suggestion Chips below assistant bubble | `useChatStream.ts` `DonePayload` + new `AppAiChatFollowUps.vue` |
| **New field `groundingType`** | Enum: `from_listing` / `from_market_context` / `general_advice` / `assumption`. Drives Source Label rendering + Confidence Indicator | `useChatStream.ts` `ChunkPayload` |
| **Activate `abstained` UI** | Schema is wired through end-to-end (`useAiAssistant.ts:42-44`, `258-259`) but no UI consumes it. Add Abstain Pattern (see §10.4) | `AppAiChatMessages.vue` |
| **New Chat reset** | `useAiAssistant.ts` already exposes `setMessages([])` — no UI entry today. Add header button | `AppAiChatSheet.vue` — sheet header |
| **GA additive events** | `ai_chat_follow_up_click` (with chip text label), `ai_chat_new_conversation`, `ai_chat_grounding_seen` (with type) | `ga.service` |
| **Stop-button aria-label** | "Stop generating" → keep (still accurate) | `AppAiChatComposer.vue:17` |

---

## 9. Design Changes

Built on top of Variant C (canvas) and `chat-design-system.md`
chat-radius tokens. The visual stack does not change — only the listing-
chatbot's *use* of it does.

### 9.1 Entry hero — chip composition

| Today | Target |
|---|---|
| 3 factual chips | 1 broad + 1 comparative + 1 advisory:<br>• Tell me about this home<br>• Compare to nearby listings<br>• Things to watch out for |

Chip *shape* stays — full pill, `--card` fill, 1 px `--border`,
cornerRadius `chat-radius-pill`. Only the labels (and i18n keys)
change.

### 9.2 Hero sparkle and headline

No visual change. Copy update only ("Ask anything about this home"
replaces the headline). Phosphor `star-four` AI Brand Mark continues
to serve as the persistent visual identity.

### 9.3 Loading bubble — stage-aware progress label

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

## 10. New / Revived UI Patterns

Component specs are owned by `chat-design-system.md`. The sub-sections
below are the **listing-chatbot-specific activation notes**: which
hooks to revive, which strings to emit, which enums map to what.

### 10.1 Source / Grounding Label

Revives the dead `sourceLabel` hook (`AppAiChatMessages.vue:46-51`).
Replaces the free-form string with the structured `groundingType`
enum below.

| `groundingType` | Rendered copy |
|---|---|
| `from_listing` | "Based on this listing" |
| `from_market_context` | "Based on market comparison" |
| `general_advice` | "General guidance" |
| `assumption` | "Inferred — verify before relying" |

Visibility: render only on `done` (no flicker during streaming).
Full visual spec: `chat-design-system.md` §5.8.

### 10.2 Confidence Indicator *(P2, optional)*

Tiny inline indicator at the start of the source label,
distinguishing verifiable claims from advisory opinion.

| Glyph | When |
|---|---|
| ● (filled dot, primary) | `from_listing` — verifiable |
| ○ (outlined dot, muted) | `from_market_context` — derived |
| △ (triangle, amber) | `general_advice` / `assumption` |

Ship only if user research shows source label alone is insufficient.
Not yet in `chat-design-system.md` — this is a listing-chatbot
proposal that may graduate to the chat DS later.

### 10.3 Follow-up Suggestion Chips

Server contract: `onDone.followUps?: string[]` (up to 3 strings).
Rendered below the most-recent assistant bubble; past bubbles' chips
disappear on the next round.

GA: `ai_chat_follow_up_click` with chip text.

Full visual spec: `chat-design-system.md` §5.3.

### 10.4 Abstain Pattern *(revived)*

When `abstained: true` arrives, render the assistant bubble with the
"Outside my listing knowledge" inline marker above the content. The
LLM-generated body follows the abstain copy direction in §7 (redirect,
don't dead-end).

Optional `nullReason` rendered below the bubble if non-null and
human-readable.

Full visual spec: `chat-design-system.md` §5.9. Distinct from the error
bubble (chat DS §5.14): abstain means the assistant *chose* not to
answer; error means the system failed.

### 10.5 New Chat Button

Activates `useAiAssistant.ts`'s existing `setMessages([])`. Add a
sheet-header trailing action, hidden when `messages.length === 0`.

GA: `ai_chat_new_conversation`.

Full visual spec: `chat-design-system.md` §5.16.

### 10.6 Idle Re-engagement Prompt *(P2)*

After 30 s of conversation idleness with `hasMessages === true`, show
a muted "Still curious?" line above the composer with 2-3 fresh
follow-up chips. Dismisses on any user action.

Not yet in `chat-design-system.md` — listing-chatbot-specific until
a second chat surface needs the same behavior.

---

## 11. System Prompt Direction

(Owner: AI team — design provides intent, not the prompt text.)

> **Reading note for engineers**: this section describes the **design-level
> positioning shift** (v1 lookup-tool framing → v2 conversation-partner
> framing) — not a literal worker prompt rewrite. Several intents below
> are **already satisfied by the current production prompt set** (IC /
> STP / RR analytical). For the engineering delta breakdown
> (✅ already-on-track / ⚠️ tune / 🆕 truly new), see
> `listing-chatbot-local-ops/v2-nlqa-proposals.md §12.1`.

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

## 12. Priority Tiers & Roadmap

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
  machine write → UI render (§10.1)
- [ ] Follow-up Suggestion Chips (§10.3)
- [ ] Abstain Pattern revival (§10.4)
- [ ] Server stage-specific progress labels (§9.3)

**Acceptance:** Every assistant bubble carries a Source Label;
non-trivial answers carry ≥2 follow-up chips; abstained rounds
redirect instead of dead-ending.

### P2 — Deep Conversation Support (Week 4+)

- [ ] New Chat Button (§10.5)
- [ ] Idle Re-engagement Prompt (§10.6)
- [ ] Confidence Indicator (§10.2) — gate on user research
- [ ] Half-snap min-height bumped 450 → 500 to fit longer answers
- [ ] Markdown rich-content validation (lists, tables, inline links in
  long answers)

**Acceptance:** Average conversation depth (rounds per session)
trends up; abandonment after first answer trends down.

---

## 13. Open Questions

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

## 14. Risk & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| LLM hallucinates beyond listing data ("this neighborhood has the best schools" without evidence) | Medium | High — trust | `groundingType` UI surfaces uncertainty; system prompt mandates "Inferred — verify" tag for assumptions |
| Cost spike from longer answers + follow-up generation | High | Medium | Cap answer length; cache follow-up chips per round; sample LLM call rate |
| Users get accustomed to NLQ&A and ask out-of-scope questions ("what's the weather?") | Medium | Low | `errorSafetyBlocked` copy redirects; conversational tone keeps users in scope |
| Existing factual users get worse experience (answer too verbose) | Low | Medium | System prompt: "match answer length to question specificity" |
| Dead-hook revival breaks production builds | Low | High | Feature flag (`listingChatbotNLQA`) to gate the new render paths |

---

## 15. References

### Design artifacts in this repo
- `listing-chatbot-design.pen` — Variants A / B / C canvas
- [`chat-design-system.md`](chat-design-system.md) — Chat Design System (sister-spec proposal) backing Variant C
- `chat-system-design.pen` — chat DS canvas (12 chat states + reusable components)

### Production source of truth
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

### Companion work to author after acceptance
- System prompt design intent doc *(AI team owns content; design
  owns intent / tone direction)*
- AI Avatar persona mark spec *(if open question on bubble-prefix
  resolves toward yes)*
