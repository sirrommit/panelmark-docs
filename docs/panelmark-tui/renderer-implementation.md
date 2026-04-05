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

### Border rendering

`_render_structure()` recurses through the `LayoutModel` tree and calls `draw_border()` for each `HSplit` border line. `draw_border()` renders the full-width horizontal rule using Unicode box-drawing characters (`─`/`═`), computing correct corner and junction characters based on adjacent vertical dividers. When the border carries a title, it is rendered centred in the line using `render_styled()` to support `<bold>`/colour markup.

Panel headings (`Region.heading`, set via `__text__` in the shell definition) are drawn by `_render_panel_heading()` as a `├─── Heading ───┤` sub-border on the top row of the panel's content area. `render_region` shrinks the region height by 1 and offsets the content region down by 1 row before passing it to the interaction.

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
| `shell.borders` | `list[BorderSpec]` | Internal separator lines resolved from the layout tree |

## Built-in interactions

All interaction types in `panelmark_tui.interactions` implement the `panelmark.Interaction` ABC and are implemented against the blessed TUI surface.

### Portable interactions implemented by panelmark-tui

The following interactions satisfy the **portable standard library** semantic contract defined in the [Portable Library](../renderer-spec/portable-library.md) spec. They are TUI-specific class implementations, but their constructor signatures, `get_value()` / `set_value()` / `signal_return()` semantics, and return values are standardized across compliant renderers:

- `MenuReturn` — single-select scrollable list that returns a mapped value
- `NestedMenu` — hierarchical action menu with `Leaf` sentinel
- `RadioList` — single-select list with radio-button visuals
- `CheckBox` — checkbox list (multi and single-select modes)
- `TextBox` — multi-line text editor with word-wrap and submit-mode options
- `FormInput` — structured data-entry form with typed fields and validation
- `DataclassFormInteraction` — dataclass-introspecting form interaction
- `StatusMessage` — display-only inline status/feedback region

### TUI-only interaction additions

The following interactions are `panelmark-tui`-specific additions with no portable equivalent:

- `MenuFunction` — scrollable menu that invokes callbacks without exiting the shell
- `ListView` — display-only scrollable list
- `TreeView` — interactive collapsible tree
- `TableView` — multi-column display table with sticky header
- `Function` — escape hatch for fully custom rendering and key handling

These are renderer-specific and are not guaranteed portable unless or until standardized.

See [Interactions](interactions.md) for the full reference.

## Built-in widgets

### Portable widgets implemented by panelmark-tui

The following widgets satisfy the **portable standard library** semantic contract:

- `Alert`, `Confirm`, `InputPrompt`, `ListSelect`, `FilePicker`, `DataclassForm`

Their constructor signatures and return-value semantics are standardized across compliant renderers.

### TUI-only widget additions

The following widgets are `panelmark-tui`-specific additions with no portable equivalent:

- `DatePicker` — calendar date selection
- `Progress`, `Toast`, `Spinner` — renderer-managed utility overlays with their own render cycles (depend on `blessed`'s terminal model)

See [Widgets](widgets.md) for the full reference.

## What portability means

A portable interaction or widget does not have to look the same in every renderer. It is portable when constructor shape, current-state semantics, submit semantics, and return values match the portable-library spec. The TUI-specific class implementations satisfy that contract even though they are built against `blessed` and produce terminal output.

## Portability boundaries

| Layer | Source of truth | Portable? |
|-------|----------------|-----------|
| Shell language (DSL syntax) | `panelmark` core | Yes |
| Shell state machine, focus, dirty tracking | `panelmark` core | Yes |
| Interaction ABC (`render`, `handle_key`, `get_value`, `set_value`, `signal_return`) | `panelmark` core | Yes |
| Draw command types (`WriteCmd`, `FillCmd`, `CursorCmd`, `RenderContext`) | `panelmark` core | Yes |
| Required portable interaction implementations | `panelmark-tui` | Yes — semantic contract matches portable spec |
| TUI-specific interaction additions | `panelmark-tui` | No — TUI-specific |
| Required portable widget implementations | `panelmark-tui` | Yes — semantic contract matches portable spec |
| TUI-specific widget additions | `panelmark-tui` | No — TUI-specific |
| `blessed` terminal integration | `panelmark-tui` | No |
| `MockTerminal`, test utilities | `panelmark-tui` | No |

Application code that uses only the portable standard library interactions and widgets (listed above as portable) is portable across any renderer that claims `portable-library-compatible` status. Application code that uses TUI-specific additions (`MenuFunction`, `ListView`, `TreeView`, `TableView`, `Function`, `DatePicker`, `Progress`, `Toast`, `Spinner`) is coupled to the TUI renderer.

## See also

- [Renderer Spec Overview](../renderer-spec/overview.md)
- [Renderer Contract](../renderer-spec/contract.md)
- [Extensions](../renderer-spec/extensions.md)
- [Readiness](../renderer-spec/readiness.md)
- [Limitations](limitations.md)
