# panelmark

The core package. Defines the shell language, parser, layout model, interaction base contract, and draw command abstractions. All renderer packages depend on `panelmark`.

## What it provides

- shell language parsing and layout model
- region resolution and shell state machine
- interaction base contract
- draw command types (`FillCmd`, `WriteCmd`, `CursorCmd`)
- core portability contracts used by the renderer spec

## What it does not include

- renderer-specific event loops
- terminal, web, or desktop implementation details
- surface-specific widget libraries

## Who should use it directly

- renderer authors implementing a new surface
- application authors who need low-level draw command control
- contributors to core panelmark behavior

Application authors targeting an existing renderer typically interact with that renderer's package rather than `panelmark` directly.

## Detailed docs

- [Shell Language](../shell-language/overview.md)
- [Renderer Spec](../renderer-spec/overview.md)

## See also

- [Ecosystem](../ecosystem.md)
- [panelmark-tui](panelmark-tui.md)
- [panelmark-html](panelmark-html.md)
- [panelmark-web](panelmark-web.md)
