# panelmark-web

**panelmark-web** is the live web runtime for panelmark. It sits on top of `panelmark-html`'s static structure and adds a real-time browser interface driven by WebSockets.

Each browser tab corresponds to one `Shell` instance on the server. The WebSocket connection is the session lifetime.

## What it provides

| Component | Description |
|-----------|-------------|
| WebSocket server handler | Drives a `panelmark.Shell` instance per session |
| Draw-command renderer | Translates `WriteCmd` / `FillCmd` into HTML for `.pm-panel-body` elements |
| Browser client | Vanilla-JS client that relays keyboard input and applies DOM updates |
| Framework adapters | FastAPI/Starlette (async) and Flask/flask-sock (sync) |
| Portable interactions | `StatusMessage`, `MenuReturn`, `RadioList`, `CheckBox`, `TextBox`, `NestedMenu`, `FormInput`, `DataclassFormInteraction` |
| Portable widgets | `Alert`, `Confirm`, `InputPrompt`, `ListSelect`, `DataclassForm`, `FilePicker` |

Custom `Interaction` objects — any `render()` method that returns `WriteCmd` and `FillCmd` commands — work out of the box alongside the built-in library.

## What it does not include

- Desktop or terminal rendering
- A built-in portable `MenuFunction`, `ListView`, `TreeView`, `TableView`, `DatePicker`, `Progress`, `Spinner`, or `Toast` (see [Interaction Coverage](../panelmark-web/interaction-coverage.md))

## Who should use it

Application authors building live browser-based panelmark applications with Python server frameworks.

## Compatibility status

`panelmark-web` implements the **core renderer contract** and claims **`portable-library-compatible`** status:

- shell hosting via WebSocket session
- draw-command execution (`WriteCmd`, `FillCmd`; `CursorCmd` is ignored — no cursor concept in the browser renderer)
- focus routing and dirty-region tracking
- exit signal handling
- all 8 required portable interactions (`panelmark_web.interactions`)
- all 6 required portable widgets (`panelmark_web.widgets`)

**Web note on widget semantics:** The portable-library spec describes widgets as blocking (`widget.show(sh)` returns after the user acts). In `panelmark-web` the session is async — assign the widget to a panel region and `signal_return()` delivers the result when the user acts. Constructor signatures and value semantics match the spec exactly.

See [Interaction Coverage](../panelmark-web/interaction-coverage.md) for the full status matrix.

## Installation

```bash
pip install panelmark-web[fastapi]   # FastAPI / Starlette
pip install panelmark-web[flask]     # Flask + flask-sock
```

Dependencies: `panelmark`, `panelmark-html`, and the chosen web framework.

## Detailed docs

- [Overview](../panelmark-web/overview.md)
- [Getting Started](../panelmark-web/getting-started.md)
- [Protocol](../panelmark-web/protocol.md)
- [Hook Usage](../panelmark-web/hook-usage.md)
- [Interaction Coverage](../panelmark-web/interaction-coverage.md)

## See also

- [panelmark-html](panelmark-html.md)
- [Portable Library](../renderer-spec/portable-library.md)
- [Readiness](../renderer-spec/readiness.md)
- [Ecosystem](../ecosystem.md)
