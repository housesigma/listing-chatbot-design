# HouseSigma AI Chat — v1: Fact Lookup (Design Intent)

> Version: v1 (Fact Lookup) · Status: Superseded by v2 · Last updated: 2026-03
>
> Successor: see [`v2-nlqa.md`](v2-nlqa.md) for the repositioning to
> Listing-aware natural-language Q&A, and [`v2-nlqa-spec.md`](v2-nlqa-spec.md)
> for the chat-surface design spec backing v2.

---

## 1. Design Brief

### Problem

When a buyer is browsing a listing detail page, specific questions come up naturally — HOA fees, basement dimensions, parking setup — but finding the answer requires hunting through dense listing copy or leaving the page to contact an agent. There is no fast, in-context path to get a factual answer without friction.

### Design Intent

> Give buyers an instant, trustworthy way to look up facts from a listing — with answers that are direct, grounded, and verifiable.

---

## 2. Core Principle: Chat as Interface, Retrieval as Behavior

The product uses a conversational UI. This is intentional — chat patterns are familiar, low-friction, and natural for questions. But the UI form does not define what the product *does*.

What it does is retrieval: fact lookup from a single listing's data. This governs every behavior decision, independent of the interface layer.

These are two distinct layers:

| Layer | What it means | Design consequence |
|-------|--------------|-------------------|
| **Interface (Form)** | Chat UI — bubbles, turns, streaming | Use familiar patterns; don't fight them |
| **Behavior (Semantics)** | Retrieval — fact-first, source-attributed, listing-bounded | Answers are short, direct, grounded; never advisory |


### What "retrieval behavior" means in practice

- Every answer is **fact-first**: state the fact directly, no preamble
- Every answer is **source-attributed**: a section label anchors where the data came from
- Answers are **short by design**: if the data exists, a good answer is 1–3 sentences
- The AI does not advise, recommend, compare, or speculate
- Out-of-scope queries get a **constructive redirect**, not a refusal

### How the principle is expressed in copy

| Surface | Copy | Why |
|---------|------|-----|
| Entry hero subtitle | "Look up details in this listing" | "Look up" frames the task as retrieval; "details" scopes capability without implying unlimited breadth |
| Input placeholder | "Look up details in this listing..." | Consistent with subtitle; "details" preferred over "anything" to avoid implying a general assistant |
| Quick reply chips | "How many beds & baths?", "When was it built?", etc. | Phrased as real questions, not field names — demonstrate capability by example |
| Generating state | "Looking up details..." | Retrieval framing, not generation framing |
| Source label | "HOA & fees", "Parking", "Building info" | Names the listing section, not the AI process |

Words intentionally avoided: *help*, *suggest*, *recommend*, *generating*, *anything you want*, *anything* — these imply a broader or more generative capability than exists.

### Entry state design

The entry state does not use an AI greeting. The primary visual is the action intent — "Look up details in this listing" — not the property address. The address and city appear below as a secondary scope label, confirming which listing the tool is scoped to.

This hierarchy reflects the context: the bottom sheet is embedded in the listing detail page, so the user already knows which property they are viewing. Repeating the address as the dominant element would be redundant. What the entry state needs to communicate first is *what this tool does*, not *where you are*.

The star-four symbol serves as the product's identity mark across all states.

### MVP Scope Boundary

Out of scope for this release:

- Comparing this listing to other properties
- Mortgage or affordability calculations
- Market trend analysis
- Agent recommendations
- Booking or scheduling actions

---
