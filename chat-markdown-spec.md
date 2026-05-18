# Chat Markdown Rendering Spec

> **Contract between the listing-chatbot worker (LLM output) and the
> web-hybrid chat surfaces** (mobile `AppAiChatMessages.vue` + desktop
> `PcAiChatMessages.vue`).
>
> This spec normalises the **full CommonMark + GFM HTML tag set (27 tags)**.
> Every tag has an explicit rendering rule — independent of whether the
> worker is currently instructed to emit it. The "worker contract" status
> is metadata, not gating: if the LLM drifts, the renderer still has a
> defined behaviour.
>
> Sister spec to [`chat-design-system.md`](chat-design-system.md). The
> chat DS covers bubble shape, state machine, and visual tokens. This
> document covers only the **markdown subset inside the assistant bubble**.

---

## 1. Scope

### 1.1 What "the tag set" means here

The frontend uses [`marked@^18`](../../web-hybrid/packages/hook/ai/useChatStream.ts)
with `gfm: true` and `breaks: true`. This is **CommonMark 0.31 + GFM
0.29-gfm**, which produces exactly **27 distinct HTML tags** from
markdown syntax:

| Origin | HTML tags | Count |
|---|---|---|
| CommonMark 0.31 | `p` `h1` `h2` `h3` `h4` `h5` `h6` `blockquote` `ul` `ol` `li` `pre` `code` `hr` `a` `img` `em` `strong` `br` | **19** |
| GFM extensions | `table` `thead` `tbody` `tr` `th` `td` `del` `input[type=checkbox]` | **+8** |
| **Total (GFM)** | | **27** |

This spec defines rendering for all 27. Raw HTML the LLM might inject
(`<details>`, `<sub>`, `<kbd>`, etc.) is **not** part of this contract —
DOMPurify handles those via its `FORBID_TAGS` policy (§2) and any
remaining tags fall through to browser default.

### 1.2 Worker contract status (metadata, not gating)

For each of the 27 tags, §4 records one of three states. The states
**do not change the rendering rule** — they document where the tag
comes from and what the operator should expect in production.

| Status | Meaning |
|---|---|
| ✅ **instructed** | `presentation_instruction_*` in [`default.yaml#L332-393`](../../listing-chatbot/src/listing_chatbot/configuration/default.yaml) explicitly tells the LLM to emit this tag |
| ⚠ **leak** | Not in the contract; LLMs occasionally emit it (hedges, alternate headings, mis-formatted sections). Rendered defensively |
| 🚫 **never** | Worker has no path that could emit this; renderer style exists only because the GFM parser accepts it and we refuse to leave any tag unspecified |

Counts: **12 instructed · 13 leak · 2 never** ( `li` counts once;
its status depends on whether it's inside `<ul>` (instructed) or
`<ol>` (leak) ).

### 1.3 Stripped before rendering (security, not formatting)

[`useChatStream.ts::sanitiseChunk`](../../web-hybrid/packages/hook/ai/useChatStream.ts)
strips, on every wire chunk before markdown parsing:

- ANSI CSI escape sequences (`\x1b[...`)
- BiDi override / isolate (U+202A-202E, U+2066-2069)
- Zero-width / formatting (U+200B-200F, U+2060-206F, U+FEFF)
- C0/C1 controls excluding common whitespace

Not a rendering concern — these are stripped to prevent terminal-log
poisoning and visually-deceptive text injection.

---

## 2. Frontend Rendering Pipeline

```
LLM JSON.answer (raw md)
  → sanitiseChunk()           // strip ANSI / BiDi / zero-width / C0-C1
  → marked.parse()            // GFM + breaks (single \n → <br>)
  → DOMPurify.sanitize()      // FORBID style/iframe/object/embed/form
  → v-html → .md-content      // scoped CSS per surface
```

Renderer config in [`useChatStream.ts`](../../web-hybrid/packages/hook/ai/useChatStream.ts):

| Setting | Value | Effect |
|---|---|---|
| `marked.gfm` | `true` | Tables, strikethrough, task lists, autolinks accepted |
| `marked.breaks` | `true` | Single newline → `<br>` (preserves chat prose feel) |
| DOMPurify `FORBID_TAGS` | `style, iframe, object, embed, form` | Active embeds + scripted styles blocked |
| DOMPurify `ADD_ATTR` | `target, rel` | Allows `<a target="_blank">` |
| `afterSanitizeAttributes` hook | Auto-inject `rel="noopener noreferrer"` on `<a target="_blank">` | Prevents reverse tabnabbing |

---

## 3. Per-Surface Bubble Tokens

Identical structure across mobile and desktop. Only sizing tokens
differ — all relative `em` values in §4 auto-scale with the base.

| Token | Desktop (`PcAiChatMessages.vue`) | Mobile (`AppAiChatMessages.vue`) |
|---|---|---|
| Bubble font-size (base) | 14px | 15px |
| Bubble padding | 10px 14px | 12px 16px |
| Bubble radius | 18px | 20px |
| AI bubble max-width | 90% | 90% |
| User bubble max-width | 80% | 80% |
| Inter-message gap | 16px | 20px |
| Inline `<code>` font-size | 12px | 13px |
| Toggle button font-size | 12px | 13px |
| Source label font-size | 12px | 13px |

---

## 4. Tag Specification — 27 Tags

CSS rules below are applied via `.md-content :deep(<tag>)` on both
surfaces. `em` values are relative to the bubble's base font-size
(§3). When the "Style" column reads **"no override"**, the tag relies
on browser default — but the rule is still part of this spec (the
choice to not override is explicit, not accidental).

### 4.1 Block: paragraph (1)

| # | Tag | Markdown | Worker | Style |
|---|---|---|---|---|
| 1 | `p` | (paragraph) | ✅ instructed | `margin: 0 0 8px;` last-child `margin-bottom: 0` |

### 4.2 Block: headings (6)

| # | Tag | Markdown | Worker | Style |
|---|---|---|---|---|
| 2 | `h1` | `# H1` | ⚠ leak | `font-size: 1.07em; font-weight: 700; margin: 0 0 8px` |
| 3 | `h2` | `## H2` | ✅ instructed | `font-size: 1.07em; font-weight: 700; margin: 0 0 8px` |
| 4 | `h3` | `### H3` | ⚠ leak | `font-size: 1em; font-weight: 700; margin: 0 0 8px` |
| 5 | `h4` | `#### H4` | ⚠ leak | (same as `h3`) |
| 6 | `h5` | `##### H5` | ⚠ leak | (same as `h3`) |
| 7 | `h6` | `###### H6` | ⚠ leak | (same as `h3`) |

**Heading clamp rationale** — chat bubbles are not page documents.
H1/H2 at 1.07em gives the worker's `##` section signpost a visible
edge over body text without forcing the bubble to grow vertically.
H3-H6 collapse to body size because they are not in the contract —
capping them prevents an LLM mis-emitting `#### Sub-section` from
producing a giant heading inside a 14-15px bubble.

### 4.3 Block: quotation & separator (2)

| # | Tag | Markdown | Worker | Style |
|---|---|---|---|---|
| 8 | `blockquote` | `> text` | ⚠ leak | `border-left: 3px solid rgba(0,0,0,0.12); padding-left: 10px; margin: 6px 0; color: #555` |
| 9 | `hr` | `---` / `***` / `___` | ⚠ leak | `border: 0; border-top: 1px solid rgba(0,0,0,0.08); margin: 10px 0` |

### 4.4 Block: lists (3)

| # | Tag | Markdown | Worker | Style |
|---|---|---|---|---|
| 10 | `ul` | `- item` | ✅ instructed | `padding-left: 20px; margin: 0 0 8px` |
| 11 | `ol` | `1. item` | ⚠ leak | (same as `ul`) |
| 12 | `li` | list item | ✅ (in `ul`) / ⚠ (in `ol`) | `margin-bottom: 4px` |

### 4.5 Block: code (2)

| # | Tag | Markdown | Worker | Style |
|---|---|---|---|---|
| 13 | `pre` | fenced ``` ``` ``` ``` / indented 4-space | ⚠ leak | `background: #e8eaed; padding: 8px; border-radius: 6px; overflow-x: auto; margin: 0 0 8px` + mono font stack |
| 14 | `code` | `` `inline` `` (or inside `pre`) | ⚠ leak | **Inline**: `background: rgba(0,0,0,0.06); padding: 2px 4px; border-radius: 3px; font-size: 12px (desktop) / 13px (mobile)` + mono font stack. **Inside `<pre>`**: `background: none; padding: 0` |

### 4.6 Block: tables (6, GFM)

| # | Tag | Markdown | Worker | Style |
|---|---|---|---|---|
| 15 | `table` | `\| col \| col \|` + `\|---\|---\|` | ✅ instructed | `display: block; width: 100%; overflow-x: auto; border-collapse: collapse; margin: 8px 0; font-size: 0.93em` |
| 16 | `thead` | header row | ✅ instructed | no override (styling carried by `th`) |
| 17 | `tbody` | body rows | ✅ instructed | no override (styling carried by `td`) |
| 18 | `tr` | row | ✅ instructed | no override |
| 19 | `th` | header cell | ✅ instructed | `padding: 6px 10px; border: 1px solid rgba(0,0,0,0.08); text-align: left; vertical-align: top; white-space: normal; background: rgba(0,0,0,0.04); font-weight: 600` |
| 20 | `td` | body cell | ✅ instructed | `padding: 6px 10px; border: 1px solid rgba(0,0,0,0.08); text-align: left; vertical-align: top; white-space: normal` |

**Table overflow strategy** — tables use `display: block; overflow-x: auto`
so a 3-column table on a narrow mobile viewport scrolls *inside the bubble*
rather than forcing the bubble past `max-width: 90%`. Cells use
`white-space: normal` so multi-line content wraps cleanly. Worker
contract caps tables at 3 columns; horizontal-scroll model is chosen
over column collapsing because the worker chose the comparison shape
for a reason.

### 4.7 Inline: emphasis (3)

| # | Tag | Markdown | Worker | Style |
|---|---|---|---|---|
| 21 | `strong` | `**bold**` / `__bold__` | ✅ instructed | `font-weight: 600` |
| 22 | `em` | `*italic*` / `_italic_` | ⚠ leak | `font-style: italic` |
| 23 | `del` | `~~strike~~` (GFM) | ⚠ leak | `color: #888` |

**Strong weight 600** (not browser default 700) — Poppins 700 at 14/15px
reads as a smudge on small viewports; 600 keeps the emphasis legible.

### 4.8 Inline: link & media (3)

| # | Tag | Markdown | Worker | Style |
|---|---|---|---|---|
| 24 | `a` | `[text](url)` / autolink `<url>` | ⚠ leak | `color: $color-cyan-strong; text-decoration: underline`. `rel="noopener noreferrer"` auto-injected on `target="_blank"` by [`useChatStream.ts`](../../web-hybrid/packages/hook/ai/useChatStream.ts) |
| 25 | `img` | `![alt](url)` | 🚫 never | `max-width: 100%; height: auto; border-radius: 6px; margin: 6px 0; display: block` |
| 26 | `input[type=checkbox]` | `- [ ]` / `- [x]` (GFM task list) | 🚫 never | `pointer-events: none; margin: 0 4px 0 0; vertical-align: middle` |

**`img` & checkbox: explicit, not browser default** — the worker has no
path to emit either, but if a leak ever occurred a raw `<img>` would
blow out the bubble's `max-width: 90%` and a checkbox would render as
an interactive but no-op control. The styles above degrade gracefully.

### 4.9 Inline: whitespace (1)

| # | Tag | Markdown | Worker | Style |
|---|---|---|---|---|
| 27 | `br` | single `\n` (via `breaks: true`) | ✅ implicit (every `\n`) | no override |

---

## 5. Emoji Font Fallback

The base font (`Poppins`, set globally in
[`_common.scss`](../../web-hybrid/packages/desktop/src/assets/style/global/_common.scss))
does not ship color emoji. Without a fallback chain, an emoji the
worker emits as an H2 section signpost renders as **monochrome Symbols
on Windows** or **undefined glyph on Linux**.

### 5.1 Chat font stack

```scss
$chat-font-stack:
  "Poppins", -apple-system, "Helvetica Neue", Helvetica, Arial,
  "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif,
  "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
```

**Order is load-bearing**: Poppins handles all Latin / CJK base glyphs
first; system color-emoji families serve only the characters Poppins
doesn't ship. Reordering (e.g. putting emoji families higher) would
substitute emoji glyphs for any character they happen to include,
breaking Latin/CJK rendering.

### 5.2 Mono font stack (for `code` / `pre`)

```scss
$chat-mono-stack:
  ui-monospace, "SF Mono", Menlo, Monaco, Consolas, monospace,
  "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
```

### 5.3 Where each stack is applied

| Selector | Stack | Why |
|---|---|---|
| `.message-bubble` | `$chat-font-stack` | Both user (plain text) and assistant (markdown) bubbles receive emoji fallback |
| `.md-content :deep(p, h1-h6, strong, em, ul, ol, li, table, th, td, blockquote, del)` | `$chat-font-stack` (re-declared) | [`_common.scss`](../../web-hybrid/packages/desktop/src/assets/style/global/_common.scss) forces `font-family: Poppins` on these exact tags at specificity (0,1,3); the scoped `.md-content[data-v-…] <tag>` at (0,2,1) wins the cascade and keeps the emoji fallback active |
| `.md-content :deep(code, pre)` | `$chat-mono-stack` | Monospace + emoji fallback |

### 5.4 Why two copies (not a shared SCSS variable)

Each `*ChatMessages.vue` declares its own `$chat-font-stack` /
`$chat-mono-stack` at the top of its scoped `<style>` block. The
duplication is intentional — extracting to a shared SCSS file would
introduce a cross-package dependency for two ~15-line variable blocks.
If a third AI surface ships (property valuation Q&A, market analysis),
revisit and extract then.

---

## 6. Source of Truth

| Question | File |
|---|---|
| What does the worker emit (contract)? | [`listing-chatbot/src/listing_chatbot/configuration/default.yaml`](../../listing-chatbot/src/listing_chatbot/configuration/default.yaml) lines 332-393 |
| What does the worker actually emit (empirical)? | Not currently tracked. To add: sample N production traces, parse via `marked` AST, count tag frequencies, append as a column to §4 |
| How is each chunk sanitised before rendering? | [`useChatStream.ts`](../../web-hybrid/packages/hook/ai/useChatStream.ts) `sanitiseChunk` |
| How is markdown parsed and HTML purified? | [`useChatStream.ts`](../../web-hybrid/packages/hook/ai/useChatStream.ts) `renderMarkdownSafe` |
| Desktop rendering rules (code) | [`PcAiChatMessages.vue`](../../web-hybrid/packages/desktop/src/pages/listing/components/PcAiChatMessages.vue) `<style>` block |
| Mobile rendering rules (code) | [`AppAiChatMessages.vue`](../../web-hybrid/packages/app/src/pages/listing/components/AppAiChatMessages.vue) `<style>` block |
| Bubble shape / colors / state machine | [`chat-design-system.md`](chat-design-system.md) |
