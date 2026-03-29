# panelmark-tui Renderer Implementation

This document describes how `panelmark-tui` satisfies the `panelmark` [renderer specification](../renderer-spec/overview.md).

`panelmark-tui` claims **`portable-library-compatible`** status. It implements all required interactions and widgets from the [Portable Standard Library](../renderer-spec/portable-library.md) spec, all frequently-implemented interactions and widgets, and provides additional renderer-specific additions documented as such.

## Module layout

```
panelmark_tui/
├── shell.py          Shell subclass — event loop, terminal setup/teardown, dirty/redraw
├── renderer.py       Draws structural chrome; calls render_region() per interaction
├── context.py        build_render_context() — constructs RenderContext from blessed term
├── executor.py       TUICommandExecutor — translates DrawCommand list to terminal output
├── style.py          render_styled() — applies <bold>/<red>/… tags via blessed
├── testing.py        MockTerminal, make_key — test utilities
├── interactions/     Concrete Interaction subclasses
└── widgets/          Renderer-side convenience widgets
```

## How each contract requirement is met

### Shell hosting

`shell.py` provides a `Shell` subclass with `run()` and `run_modal()` methods. `run()` initializes the `blessed` terminal, enters fullscreen mode, drives the event loop, and tears down the terminal on exit.

### Region rendering

`renderer.py` walks the `LayoutModel` tree to draw structural chrome (borders, headings, dividers), then calls `render_region(region, interaction, focused)` for each named region.

`render_region` builds a `RenderContext` via `context.py`, calls `interaction.render(context, focused)`, and passes the resulting command list to `TUICommandExecutor`.

### Draw command execution

`executor.py` provides `TUICommandExecutor`, which translates each `DrawCommand` into terminal escape sequences via `blessed`. It maps region-relative coordinates to screen-absolute coordinates.

### Input dispatch

The event loop in `shell.py` reads key events from `blessed` and passes them to `shell.handle_key(key)`. On `('exit', value)`, the loop terminates and returns `value`. On `('continue', None)`, it redraws dirty regions and waits for the next event.

### Focus handling

`renderer.py` passes `focused=True` to `render_region` for the region whose name matches the shell's current focus.

### Dirty / redraw

`shell.py` tracks dirty region names in its own `_dirty` set. After each `handle_key()` call, the Shell subclass's `_redraw_dirty(renderer, term)` method iterates the dirty set, re-renders each affected region, and clears the set.

### Shell return semantics

The event loop returns the value from `('exit', value)` to the caller of `run()`. Interaction `signal_return()` behavior is routed through the shell's key dispatch and results in an exit signal when the interaction signals accept.

## Shell API surface

These are the `panelmark.Shell` attributes that `panelmark-tui` accesses directly. All other interaction goes through the public API.

| Attribute | Type | Purpose |
|-----------|------|---------|
| `shell.regions` | `dict[str, Region]` | Populated after `Shell.__init__` |
| `shell.interactions` | `dict[str, Interaction]` | Populated by `assign()` |
| `shell.focus` | `str \| None` | Name of the currently focused region |
| `shell._dirty` | `set[str]` | Region names needing re-render |
| `shell.handle_key(key)` | method | Returns `('exit', value)` or `('continue', None)` |
| `shell.layout` | `LayoutModel` | The parsed layout tree |

## Renderer-specific additions

The following are `panelmark-tui`-specific and are not part of the portable core contract.

### Built-in interactions

All interaction types in `panelmark_tui.interactions` implement the `panelmark.Interaction` ABC but are implemented against the blessed TUI surface. They can be used in any `panelmark-tui` shell but are not portable to other renderers.

See [Interactions](interactions.md) for the full reference.

### Built-in widgets

All widgets in `panelmark_tui.widgets` are renderer-specific. `Progress`, `Toast`, and `Spinner` manage their own render cycles in ways that depend on `blessed`'s terminal model.

See [Widgets](widgets.md) for the full reference.

## Portability boundaries

| Layer | Source of truth | Portable? |
|-------|----------------|-----------|
| Shell language (DSL syntax) | `panelmark` core | Yes |
| Shell state machine, focus, dirty tracking | `panelmark` core | Yes |
| Interaction ABC (`render`, `handle_key`, `get_value`, `set_value`, `signal_return`) | `panelmark` core | Yes |
| Draw command types (`WriteCmd`, `FillCmd`, `CursorCmd`, `RenderContext`) | `panelmark` core | Yes |
| Built-in interaction implementations | `panelmark-tui` | No — TUI-specific |
| Built-in widget implementations | `panelmark-tui` | No — TUI-specific |
| `blessed` terminal integration | `panelmark-tui` | No |
| `MockTerminal`, test utilities | `panelmark-tui` | No |

Application code that stays within the core shell language and the `Interaction` ABC is portable. Application code that directly imports `panelmark_tui.interactions` or `panelmark_tui.widgets` is coupled to the TUI renderer.

## See also

- [Renderer Spec Overview](../renderer-spec/overview.md)
- [Renderer Contract](../renderer-spec/contract.md)
- [Extensions](../renderer-spec/extensions.md)
- [Readiness](../renderer-spec/readiness.md)
- [Limitations](limitations.md)
