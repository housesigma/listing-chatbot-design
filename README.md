# Listing Chatbot — Design

Design repo for HouseSigma's in-listing AI chatbot. The story is the
positioning shift from **v1 (fact lookup)** to **v2 (listing-aware
natural-language Q&A)**, and the Chat Design System proposal that v2
adopts as its visual baseline.

> **Positioning shift:** "Look up facts from a listing" (v1) →
> "Listing-aware natural-language Q&A" (v2).

---

## What lives here

Four artifacts, two proposals.

### Proposal 1 — Listing Chatbot v2 NLQ&A

| File | What it is |
|---|---|
| [`v2-nlqa.md`](v2-nlqa.md) | Design intent + proposal. v1 → v2 positioning, variants explored, design principles, copy table, schema additions, new/revived UI patterns, P0-P2 roadmap, risks. |
| `listing-chatbot-design.pen` | Design canvas. Variant A (Production Mirror), Variant B (DS-Compliant, rejected), Variant C (Chat-First, canonical). |

### Proposal 2 — Chat Design System

A sister-spec proposal to `housesigma-design-system.md` (the base
platform DS, in `prototypes/pencil-poc/`). Covers all chat / AI
surfaces; the listing chatbot is its first implementing surface.

| File | What it is |
|---|---|
| [`chat-design-system.md`](chat-design-system.md) | Markdown spec. Token departures, 12-state chat state machine, 15+ reusable chat components, composition rules. Inside any chat surface, this wins over base DS. |
| `chat-system-design.pen` | Canvas. 12 frames, one per canonical chat state. |

### How they relate

```
   base DS (pencil-poc/housesigma-design-system.md)
                       ^
                       | sister spec, wins inside chat surfaces
                       |
       chat-design-system.md  ←→  chat-system-design.pen
                       ^
                       | adopted as visual baseline
                       |
            v2-nlqa.md  ←→  listing-chatbot-design.pen
```

The Chat DS proposal is **product-agnostic** (any future AI surface
can implement it). The v2 NLQ&A proposal is the first
**product-specific** application of it.

`.pen` files are encrypted — open them with the Pencil MCP tools
(`open_document`, `get_editor_state`, …), not `Read` / `Grep`.

---

## Reading order

1. [`v2-nlqa.md`](v2-nlqa.md) — start here. The story (v1 → v2), what
   changes, what stays, the roadmap.
2. [`chat-design-system.md`](chat-design-system.md) — the visual /
   interaction spec the v2 proposal adopts. Read when you need
   component-level detail.
3. `listing-chatbot-design.pen` — the canvas behind v2-nlqa.md.
   Variant C is canonical.
4. `chat-system-design.pen` — the canvas behind chat-design-system.md.
   12 chat states.

---

## Variants in the listing chatbot canvas

`listing-chatbot-design.pen`:

- **Variant A — Production Mirror.** Baseline. Faithful copy of
  current production.
- **Variant B — DS-Compliant.** Rejected. Forced chat through base
  DS radius / button / icon rules — composer became a search form,
  bubbles became cards, the surface stopped reading as chat.
- **Variant C — Chat-First.** Canonical. Follows the Chat DS.
  Visually near-identical to production but every value comes from a
  named token. Codifies what production already intuitively does
  right.

The full rationale lives in `v2-nlqa.md` §3 and
`chat-design-system.md` §1.

---

## Relation to production code

Production source the v2 proposal touches:

- `packages/app/src/pages/listing/components/AppAiChat*.vue`
- `packages/hook/ai/useAiAssistant.ts` — `ChatMessage`,
  `friendlyErrorMessage`, `stop`, `handleSubmit`
- `packages/hook/ai/useChatStream.ts` — `ChunkPayload`,
  `DonePayload`, `ProgressFieldPayload`
- `packages/common/i18n/translation/{en,zh}.ts` — `aiChat.*` namespace

Most of the v2 lift is **revival** — `sourceLabel`, `abstained`,
`nullReason` fields already exist end-to-end in the state machine
but no UI renders them. The proposal defines how.

---

## Working with the canvases

Pencil MCP tools: `open_document`, `get_editor_state`,
`get_guidelines`, `batch_get`, `batch_design`, `snapshot_layout`,
`get_screenshot`, `get_variables`, `set_variables`,
`find_empty_space_on_canvas`, `search_all_unique_properties`,
`replace_all_matching_properties`, `export_nodes`.

Typical loop: `open_document` → `get_editor_state` for layout / scope
→ `batch_get` for node details → `batch_design` for edits →
`get_screenshot` to verify.

---

## Status

| Artifact | Status |
|---|---|
| `v2-nlqa.md` | Proposal |
| `listing-chatbot-design.pen` (Variant C) | Canonical visual baseline |
| `chat-design-system.md` | Sister-spec proposal |
| `chat-system-design.pen` | Canonical chat-DS canvas |
