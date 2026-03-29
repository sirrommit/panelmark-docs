# panelmark

**panelmark** is a zero-dependency Python library for defining UI layouts using a readable ASCII-art shell language, and for implementing the interaction logic that runs inside them.

It is the core library in the panelmark ecosystem. All renderer packages depend on it.

## What it provides

| Component | Description |
|-----------|-------------|
| Shell definition language | ASCII-art DSL for describing layouts |
| Layout model | `HSplit`, `VSplit`, `Panel`, `Region` — resolves geometry at a given viewport size |
| `Shell` state machine | Focus management, dirty tracking, key dispatch, `on_change`, `bind` |
| `Interaction` ABC | The protocol all interactive widgets implement |
| Draw command types | `WriteCmd`, `FillCmd`, `CursorCmd`, `RenderContext` — renderer-agnostic output |
| Style tag parser | Parses `<bold>text</>` markup in border titles |
| Exceptions | `RegionNotFoundError`, `CircularUpdateError` |

**Zero runtime dependencies.** No `blessed`, no web framework, no GUI toolkit. Any renderer can depend on `panelmark` without pulling in surface-specific libraries.

## What it does not include

- A renderer or event loop (those live in `panelmark-tui`, `panelmark-html`, `panelmark-web`)
- Built-in interaction implementations (a Label, Menu, TextBox, etc.) — those are renderer-provided
- Any output surface or display logic

## Who should use it directly

- **Renderer authors** implementing a new rendering surface
- **Custom interaction authors** implementing the `Interaction` ABC for use with any renderer
- **Application authors** who need low-level draw command control

Application authors targeting an existing renderer typically import from that renderer's package (e.g. `panelmark_tui.Shell`) rather than from `panelmark` directly, though the core types are used throughout.

## Current status

| Feature | Status |
|---------|--------|
| Shell definition language | Fully working |
| `#` line comments and `/* */` block comments | Fully working |
| Horizontal splits (`=` / `-` border rows) | Fully working |
| Vertical splits — single (`\|`) and double (`\|\|`) dividers | Fully working |
| Equal-width fill columns | Fully working (columns differ by at most 1 char) |
| Panel headings (`__text__` syntax) | Parsed and stored; renderers must implement display |
| `Shell` state machine | Fully working |
| Draw command abstraction | Fully working |
| `Interaction` base class | Fully working |

## Installation

```
pip install panelmark
```

## Quick start

```python
from panelmark import Shell, Interaction
from panelmark.draw import DrawCommand, RenderContext, WriteCmd

LAYOUT = """
|=== <bold>My App</> ===|
|{10R $sidebar$  }|{$main$        }|
|==================|
|{2R  $status$              }|
|==================|
"""

class Label(Interaction):
    def __init__(self, text: str):
        self._text = text

    @property
    def is_focusable(self) -> bool:
        return False

    def render(self, context: RenderContext, focused: bool = False) -> list[DrawCommand]:
        line = self._text[:context.width].ljust(context.width)
        return [WriteCmd(row=0, col=0, text=line)]

    def handle_key(self, key) -> tuple:
        return False, self.get_value()

    def get_value(self):
        return self._text

    def set_value(self, value) -> None:
        self._text = str(value)

shell = Shell(LAYOUT)
shell.assign("sidebar", Label("Navigation"))
shell.assign("main",    Label("Content area"))
shell.assign("status",  Label("Ready"))

# Drive with a renderer — e.g. panelmark_tui.Shell adds run()
# result = shell.run()
```

## Detailed docs

- [Shell Language](../shell-language/overview.md) — full syntax reference and examples
- [Renderer Spec](../renderer-spec/overview.md) — renderer compatibility contract and portable library

## See also

- [Ecosystem](../ecosystem.md)
- [panelmark-tui](panelmark-tui.md)
- [panelmark-html](panelmark-html.md)
- [panelmark-web](panelmark-web.md)
