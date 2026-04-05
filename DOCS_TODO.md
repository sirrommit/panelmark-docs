# DOCS_TODO.md

This file tracks documentation updates needed in this repo. Each section has a
checklist; tick items off as the corresponding renderer work is completed.

---

## 1. Border rendering (`BorderSpec` / `Shell.borders`)

**Background.**
`panelmark` core now returns `(regions, borders)` from `LayoutModel.resolve()`
instead of just `regions`.  The `borders` list contains `BorderSpec` objects —
one per internal HSplit separator — carrying absolute `row`, `col`, `width`,
`style` (`'single'`/`'double'`), and optional `title`.  `Shell.borders` exposes
the same list as a read-only property.

Outer frame lines (`|=====|` at the top and bottom of the shell) are **not**
included; only separator lines between two content panels produce a
`BorderSpec`.

Each renderer must consume `shell.borders` and draw the lines.  See
`BORDER_FIX_TODO.md` in `panelmark-tui` and `panelmark-html`.

### Docs to update once both renderers implement border rendering

- [ ] **`docs/shell-language/syntax.md`** — the section beginning
  "The first such row found splits the definition…" currently says borders are
  "rendered as a full-width horizontal rule" but this was aspirational.  Once
  renderers implement it, verify that statement is accurate and add a note
  that `Shell.borders` is how renderers receive the data.

- [ ] **`docs/shell-language/syntax.md`** — the quick-reference table near the
  bottom already lists `=== Title ===` and `--- Details ---` correctly.  No
  change needed there unless wording is improved.

- [ ] **`docs/shell-language/overview.md`** — if it mentions borders, check
  that it does not imply borders were always rendered.

- [ ] **`docs/panelmark-tui/renderer-implementation.md`** — add a short section
  explaining that `shell.borders` is read after layout resolution and each
  `BorderSpec` is drawn before (or after) rendering interactions.

- [ ] **`docs/panelmark-html/`** (equivalent renderer-implementation page,
  once that section exists) — same treatment for the HTML renderer.

### Docs to update once `panelmark-tui` implements borders

- [ ] **`docs/panelmark-tui/renderer-implementation.md`** — mark border
  rendering as implemented; describe the `TUICommandExecutor` / `BorderSpec`
  flow if relevant.

### Docs to update once `panelmark-html` implements borders

- [ ] **`docs/panelmark-html/`** — mark border rendering as implemented.

---

## 2. Panel heading (`__text__` / `Region.heading`) — stale docs

**Background.**
`docs/shell-language/syntax.md` line ~135 currently says:

> `` `__text__` inside the block attaches a heading string to the panel.
> The heading is rendered as a centred title in a single-line
> `├─── Heading ───┤` border at the top of the panel's content area,
> consuming one row of the panel's height.``

This is **not yet implemented in any renderer**.  The `__text__` syntax is
parsed and stored in `Region.heading`, but no renderer reads it.

The docs are currently ahead of the implementation.

### Immediate fix (no renderer change needed)

- [ ] **`docs/shell-language/syntax.md`** — add a note to the `__text__`
  paragraph marking it as "not yet rendered by any current renderer" or move
  it to a "planned features" section, so the docs do not document unimplemented
  behaviour as if it works.

### Once a renderer implements `Region.heading`

- [ ] Remove the caveat and restore the full description with an example.
- [ ] Update the renderer's implementation page to describe how `Region.heading`
  is consumed (typically: reserve the first row, draw the heading text centred,
  pass `context` with `height - 1` to the interaction).

---

## 3. `Shell.borders` API reference

Once border rendering is live in at least one renderer, add `Shell.borders` to
whichever page documents the `Shell` public API:

- [ ] **`docs/shell-language/overview.md`** or wherever `Shell.regions`,
  `Shell.dirty_regions`, etc. are listed — add `Shell.borders` with a one-line
  description: "read-only list of `BorderSpec` objects for all internal
  separator lines; updated whenever the layout is resolved."

---

## Syntax clarification to preserve

The `{----Label---}` syntax (braces included) is **not** a border.  It is a
content row in a panel definition and produces no `BorderSpec`.  Only bare
`|----Label----|` and `|====Label====|` lines (no `{`) are border rows.

This distinction is already correct in `syntax.md`; do not accidentally blur it
when updating the border docs.
