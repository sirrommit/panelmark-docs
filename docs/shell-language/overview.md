# Shell Language Overview

panelmark layouts are defined using an ASCII-art shell definition string. The parser converts this string into a tree of `HSplit`, `VSplit`, and `Panel` nodes, which are then resolved into flat `Region` objects (row, col, width, height) at a given viewport size.

A developer writes the shell once. The renderer is chosen at deployment time.

## What the shell language defines

- named regions and their spatial arrangement
- how panels split horizontally and vertically
- which panels have headings
- initial focus behavior (managed by the shell state machine)

## What the shell language does not define

- how a region's content is rendered — that is the renderer's job
- interaction-specific behavior or input handling
- surface-specific styling beyond border titles

## Shell, region, panel

A **shell** is the top-level layout container. It is defined by the shell definition string and manages a collection of named regions.

A **region** is a named rectangular area within the shell. Regions are the unit of layout, focus, and interaction assignment.

A **panel** is the content specification inside `{...}` blocks in the shell definition. Each panel becomes a region once the shell resolves geometry for its viewport.

An **interaction** is assigned to a region to make it live. See [Custom Interactions](../packages/panelmark.md) and the [Portable Library](../renderer-spec/portable-library.md).

## Shell state machine

Once parsed, a `Shell` instance manages the layout and routes input to interactions.

| Method | Description |
|--------|-------------|
| `shell.assign(name, interaction)` | Attach an `Interaction` to a named region |
| `shell.unassign(name)` | Remove the interaction from a region |
| `shell.get(name)` | Get the current value of a region |
| `shell.update(name, value)` | Programmatically set a region's value (marks dirty) |
| `shell.on_change(name, cb)` | Register a callback fired when a region's value changes |
| `shell.bind(source, target)` | Mirror source value → target (with optional transform) |
| `shell.set_focus(name)` | Move focus to a named region |
| `shell.handle_key(key)` | Dispatch a key event; returns `('exit', value)` or `('continue', None)` |
| `shell.dirty_regions` | Set of region names that need re-rendering |
| `shell.mark_all_clean()` | Clear the dirty set after rendering |

### Built-in key bindings

| Key | Action |
|-----|--------|
| `Tab` | Move focus to next focusable region |
| `Shift+Tab` (`KEY_BTAB`) | Move focus to previous focusable region |
| `Escape` or `Ctrl+Q` | Exit — `handle_key` returns `('exit', None)` |

All other keys are dispatched to the currently focused interaction's `handle_key()` method. Focus order follows reading order (top-to-bottom, left-to-right) by region position.

## In this section

- [Syntax](syntax.md) — shell definition grammar: splits, blocks, specifiers, parser rules
- [Examples](examples.md) — annotated complete examples and custom interaction patterns

## See also

- [Ecosystem](../ecosystem.md)
- [Renderer Spec](../renderer-spec/overview.md)
- [Glossary](../glossary.md)
- [panelmark package](../packages/panelmark.md)
