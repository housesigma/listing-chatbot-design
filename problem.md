# Listing Lens — Problem Document

> Version: MVP · Date: 2026-03-25

---

## Problem 1: Entry Point Discoverability

**Problem Description:** Users browsing the listing detail page cannot distinguish the AI assistant entry from other bottom-bar actions. The ✦ icon lacks a text label and shares the same visual weight as Watch and Schedule Viewing, resulting in low feature awareness and poor click-through rates.

**Feedback:**
> On the listing detail page, the bottom action bar includes buttons like Watch and Schedule Viewing. The AI assistant entry point (a small icon button in the bottom-right corner) looks similar to other action buttons, making it hard for users to recognize it as an AI chat entry. This leads to low feature visibility and insufficient click-through rates.

**User Journey:**

```
Browse listing photos & price → Scroll through property details → (Discover AI entry) → Tap to open chat sheet
                                                                        ↑ problem here
```

The break occurs between passive browsing and active discovery. If users do not notice the entry point during their natural scroll path, the entire AI feature is invisible to them.

**User Scenario & Segments:**

- **Scenario #1 (First-time buyer):** A user browsing their first listing scans photos, price, and key parameters. Their attention never reaches the bottom bar's ✦ icon because it carries no visual distinction from adjacent buttons — the AI feature goes entirely undiscovered.
- **Scenario #2 (Returning user):** A user who previously used the AI feature on another listing cannot relocate the entry point. The unlabeled ✦ icon does not trigger recognition when scanning the bottom bar alongside Watch and Schedule Viewing.
- **Scenario #3 (Mid-scroll intent):** A user reading through listing details suddenly has a question about HOA fees. The thought occurs while scrolling, but the ✦ icon is anchored to the bottom bar with no spatial relationship to the content being read — the connection between "I have a question" and "that icon can answer it" is never made.

---

## Problem 2: Blank First Screen with No Guidance

**Problem Description:** When users open the AI chat panel, they land on a blank screen with no orientation — no headline explaining the tool's purpose, no visual identity mark, and no indication of what kinds of questions the tool can handle. The only signal is a placeholder string in the input field, which is insufficient to build confidence or communicate capability.

**Feedback:**
> After tapping into the AI Assistant chat panel, users see a mostly blank page with only an input box and the placeholder text "Ask anything about the property." There is no welcome message, feature introduction, or guided prompts, leaving users unsure of what the chatbot can do and likely to drop off before asking a question.

**User Journey:**

```
Tap entry point → Chat sheet opens → (Understand what the tool does) → Type or tap first question
                                            ↑ problem here
```

The break occurs immediately after the sheet opens. Users must independently figure out "what can I do here?" with no visual guidance, creating a cognitive barrier that causes abandonment before any interaction.

**User Scenario & Segments:**

- **Scenario #1 (Curious browser):** A user taps the ✦ icon out of curiosity. The sheet opens to a blank conversation with only a text input. Without any indication of what the tool can do, the user assumes it is a generic chatbot, loses interest, and dismisses the sheet within seconds.
- **Scenario #2 (Intent-driven buyer):** A serious buyer opens the chat hoping to ask about HOA fees. The blank screen offers no signal that this tool specializes in listing data retrieval. The user hesitates, unsure whether this is the right place — the moment of purchase intent is lost to uncertainty.
- **Scenario #3 (Low-confidence user):** A user unfamiliar with AI chat opens the panel. With only a placeholder in the input field and no visual cues, they cannot quickly understand the feature's purpose or how to phrase a question. A headline and examples would have been enough to orient them.

---

## Problem 3: No Suggested Prompts

**Problem Description:** Users who open the chat panel must formulate and type a question entirely on their own. There are no preset prompts to lower the barrier to first interaction, demonstrate the tool's scope, or spark questions the user hadn't thought to ask. This cold-start friction disproportionately filters out casual and uncertain users.

**Feedback:**
> The chat panel does not provide preset quick questions (e.g., "What is the historical sale price of this property?", "What schools are nearby?", "What is the estimated value?"). Users must think of and type questions on their own, raising the interaction barrier — especially for users unfamiliar with AI chat.

**User Journey:**

```
Chat sheet opens → (Decide what to ask) → Type question → Send → Receive AI response
                        ↑ problem here
```

The break occurs at the "first question" moment. Without suggested prompts, users must generate a question from scratch — a high-friction step that filters out users before they ever experience the tool's value.

**User Scenario & Segments:**

- **Scenario #1 (Formatting anxiety):** A first-time user opens the chat and sees only an empty input field. They have questions but don't know how to phrase them for an AI — "Do I ask it like Google? Like a person?" The lack of examples creates uncertainty about the expected input format.
- **Scenario #2 (No specific intent):** A casual browser is curious about the feature but has no particular question. Without suggested prompts to browse, nothing sparks a question. They would have tapped "Square footage?" if it were offered, but won't type it unprompted.
- **Scenario #3 (Repetitive typing across listings):** A power user checking multiple listings wants to quickly compare parking info. Without a one-tap shortcut, they must type the same question manually on every listing — a repetitive friction point that slows their workflow.

---

## Problem 4: Rough Mobile Interaction Experience

**Problem Description:** The chat interface takes over the entire screen on mobile, severing the user's visual connection to the listing they are asking about. Users cannot reference listing content (photos, price, parameters) while reading AI responses, and the lack of quick-dismiss and generation-interrupt controls makes the interaction feel heavy and irreversible.

**Feedback:**
> On mobile, the chat panel covers the listing detail page in full-screen mode, preventing users from viewing listing information and chat content simultaneously. The input box sits at the bottom of the screen, and when the keyboard appears, the visible area is limited. There are no shortcuts (e.g., voice input, one-tap return to the listing page), resulting in poor overall interaction fluency.

**User Journey:**

```
Open chat → Type question → Read AI response → (Reference listing details) → Continue conversation or dismiss
                                                       ↑ problem here
```

The problem spans the entire in-chat experience. Full-screen coverage breaks the spatial relationship between the chat and the listing, and the lack of lightweight dismiss and interrupt controls makes every interaction feel like a commitment.

**User Scenario & Segments:**

- **Scenario #1 (Cross-referencing):** A buyer receives an answer about parking but wants to verify it against the listing photos. The full-screen chat blocks the listing entirely — they must close the chat, scroll to the photos, then reopen the chat, losing conversational flow and scroll position.
- **Scenario #2 (Small screen + keyboard):** A user opens the keyboard to type on a small phone. The keyboard occupies ~50% of the screen, and full-screen chat leaves only 2–3 lines of conversation visible. The user cannot see previous messages while composing a follow-up.
- **Scenario #3 (Wrong question, no cancel):** A user sends a question and immediately realizes it was wrong. There is no stop button — they must wait for the full response or force-close the chat and lose context entirely.

---

## Problem 5: Lack of Contextual Awareness During Conversation

**Problem Description:** Once a conversation begins, the chat panel provides no persistent reference to the listing being discussed. Users cannot see the property address, price, or layout summary within the chat, and AI responses do not indicate which section of the listing data they are drawn from. This forces users to mentally track context or switch between pages to verify details.

**Feedback:**
> Although the input box displays the current listing tag "1521 19th Ave," neither the top of the chat panel nor the conversation area shows any listing summary information (e.g., price, layout, address). Users lack contextual reference during the conversation and need to switch back and forth between pages to confirm details.

**User Journey:**

```
Ask question → Read AI response → (Verify: right listing? Where did this data come from?) → Ask follow-up
                                                     ↑ problem here
```

The problem compounds over multi-turn conversations. Each additional message pushes the user further from the original listing context, increasing confusion and the need to page-switch.

**User Scenario & Segments:**

- **Scenario #1 (Multi-listing comparison):** A buyer comparing several properties opens the AI chat on one listing, then navigates to another. Without a visible address in the conversation, they confuse which answers belong to which property.
- **Scenario #2 (Unexpected answer):** A user asks "How many bathrooms?" and receives "6 bathrooms." The number seems high — the user wants to confirm this is the right listing, but no address or property summary is visible in the chat to verify. They close the chat to check, breaking flow.
- **Scenario #3 (Source trust):** Five messages into a conversation, the AI says "HOA fees are $345/month." The user wonders where this number came from and whether it is current. Without source attribution, the response feels ungrounded and unreliable.

---

## Problem 6: Insufficient AI Response Status Feedback

**Problem Description:** After sending a message, the system provides only a static text indicator with no animation or progress signal. Users have no way to distinguish "actively processing" from "frozen," and there is no mechanism to interrupt a generation in progress. This creates uncertainty during every wait cycle and makes longer response times feel like system failures.

**Feedback:**
> After sending a message, only a static "AI is typing..." text is displayed — there is no typing animation, progress indicator, or estimated wait time. Users lack confidence during the wait and may mistakenly assume the system is frozen, causing them to leave.

**User Journey:**

```
Send message → (Wait for AI response) → Read response → Continue conversation
                     ↑ problem here
```

The problem occurs in the wait state between sending and receiving. Without dynamic visual feedback, even a 2–3 second wait feels uncertain, and waits longer than 5 seconds feel broken.

**User Scenario & Segments:**

- **Scenario #1 (Perceived freeze):** A user sends a question and sees static text "AI is typing..." with no animation. After 3 seconds they wonder if the system is working. After 5 seconds they assume it is frozen and dismiss the chat — even though the response was about to arrive.
- **Scenario #2 (Slow network):** A user on a slow connection sends a question. The response takes 8 seconds. With only static text, they have no way to distinguish "still processing" from "stuck" and abandon the interaction.
- **Scenario #3 (Regret after send):** A user sends the wrong question and immediately wants to cancel. There is no stop or interrupt button — they must wait for the full response before they can retype, creating frustration and wasted time.
