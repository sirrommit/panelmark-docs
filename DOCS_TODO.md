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

- [x] **`docs/shell-language/syntax.md`** — confirmed accurate; added note
  that `Shell.borders` is how renderers receive the data.

- [x] **`docs/shell-language/syntax.md`** — quick-reference table already
  correct; no change needed.

- [x] **`docs/shell-language/overview.md`** — added `Shell.borders` to the
  state machine table.

- [x] **`docs/panelmark-tui/renderer-implementation.md`** — added Border
  rendering and Panel heading sections; added `shell.borders` to API surface
  table.

- [x] **`docs/panelmark-html/rendering-api.md`** — added border elements to
  the "What is rendered" table.

### Docs to update once `panelmark-tui` implements borders

- [x] **`docs/panelmark-tui/renderer-implementation.md`** — done above.

### Docs to update once `panelmark-html` implements borders

- [x] **`docs/panelmark-html/rendering-api.md`** — done above.

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

- [x] **`docs/shell-language/syntax.md`** — both renderers already implement
  `Region.heading`. Docs updated to describe the actual renderer-specific
  visual forms (TUI: `├─── Heading ───┤` sub-border; HTML: `<header>`).

### Once a renderer implements `Region.heading`

- [x] Both renderers implement it. Description updated in `syntax.md` and
  `renderer-implementation.md`.

---

## 3. `Shell.borders` API reference

Once border rendering is live in at least one renderer, add `Shell.borders` to
whichever page documents the `Shell` public API:

- [x] **`docs/shell-language/overview.md`** — `Shell.borders` and
  `Shell.regions` added to the state machine table.

---

## Syntax clarification to preserve

The `{----Label---}` syntax (braces included) is **not** a border.  It is a
content row in a panel definition and produces no `BorderSpec`.  Only bare
`|----Label----|` and `|====Label====|` lines (no `{`) are border rows.

This distinction is already correct in `syntax.md`; do not accidentally blur it
when updating the border docs.
