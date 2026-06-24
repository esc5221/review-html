---
name: review-html
description: Turn any content (a doc, plan, code, spec, or review target) into a single self-contained HTML review file with per-item approve/reject toggles and comment boxes. Each item gets ✓Approve/✗Reject buttons and a comment field; a "Copy results" button exports the verdict (with item id + status) as Markdown; everything autosaves to localStorage. Light theme, no external libraries, opens straight from file://. Use when the user asks for a "review html", "review doc", "approve/reject", "comment per item", or "make this reviewable".
---

# Review HTML

Turn a review target into a **single self-contained HTML file with inline comments**.
The comment / save / copy logic is already complete in `assets/template.html`; your
only job is to **slice the content into items and drop them in**.

## How to build

1. Read `assets/template.html`.
2. Replace the 4 placeholders:
   - `{{DOC_TITLE}}` — document title (appears in 2 places, replace both)
   - `{{DOC_ID}}` — a stable slug for the localStorage namespace (e.g. `my-plan-v1`).
     **Pick once and never change it** — changing it disconnects saved comments.
   - `{{GENERATED}}` — subtitle (context/date). May be empty.
   - `{{ITEMS}}` — the item blocks (format below)
3. Save the result to the user's path (default `/tmp/<doc-id>.html`) and report it.

## Item (`{{ITEMS}}`) format

Each review unit is one `section.ritem`:

```html
<section class="ritem" id="unique-slug" data-title="Human-readable item title">
  <h2 class="ritem__title">Human-readable item title <span class="tag">optional badge</span></h2>
  <div class="ritem__body">
    <!-- arbitrary HTML: <p>, <ul>, <pre><code>, <table>, etc. -->
  </div>
</section>
```

- Make `id` a **unique** ASCII slug. It is embedded in the copy output as `#id`, the item identifier.
- `data-title` is used in the copy header (`## [n] Title (#id) — ✅ Approved`).
- The approve/reject buttons and comment `textarea` are appended automatically by JS. **Do not add them yourself.**
- Escape code in the body as `&lt; &gt; &amp;`.

## How to split items (not too wide, not too narrow)

One item = **one independently judgeable chunk** — small enough to settle with a single comment.

```
Target              Split unit
─────────────────── ──────────────────────────────
Prose / plan doc    by subheading (H2/H3) section
Rules / guidelines  by rule group (not per bullet — too narrow)
Code                by function/class/logical block (not per line)
Skill / prompt      by section (not the whole thing — too wide)
Design / UI         by screen / component
```

Too narrow (per bullet) = comment fatigue; too wide (one blob) = ambiguous target.
**5–20 items** is usually the sweet spot.

## Guarantees

- Zero external dependencies, light theme. Opens as a single file (`file://` OK).
- Each item has a ✓Approve / ✗Reject toggle (click again to clear) + optional comment. A click alone records a verdict → fast approve/reject.
- Verdicts and comments save to localStorage instantly (comment `reviewhtml:<doc-id>:<item-id>`, status `…:status`). The left color bar shows state (approved green, rejected red, commented teal).
- "Copy results" → only items with a verdict or comment, as Markdown. Each carries `[n] Title (#id) — ✅Approved/❌Rejected/💬Comment`.
- Top progress counter (approved · rejected · commented / total), "Reset" button (clears all after confirm).
