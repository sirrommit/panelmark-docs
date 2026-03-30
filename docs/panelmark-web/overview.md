# panelmark-web Overview

`panelmark-web` is the live web runtime for the panelmark ecosystem. It sits on top of
`panelmark-html`'s static structure and adds a real-time browser interface: WebSocket
sessions, keyboard input relay, and server-side rendering of interaction content into panel
bodies.

**Compatibility:** `panelmark-web` claims **`portable-library-compatible`** status — it
implements all 8 required portable interactions and all 6 required portable widgets.

## What panelmark-web provides

- A WebSocket server handler that drives a `panelmark.Shell` instance per browser tab
- A draw-command-to-HTML renderer that populates `.pm-panel-body` elements (`WriteCmd` and `FillCmd` supported; `CursorCmd` ignored)
- A vanilla-JS browser client that relays keyboard input and applies DOM updates
- Framework adapters for FastAPI/Starlette (async) and Flask/flask-sock (sync)
- Built-in implementations of all required portable interactions (`panelmark_web.interactions`)
- Built-in implementations of all required portable widgets (`panelmark_web.widgets`)

## What panelmark-web does not provide

- A standalone HTTP server — you supply the framework (FastAPI, Flask, etc.)
- `CursorCmd` support — cursor rendering is ignored in HTML

## How sessions work

Each browser tab corresponds to one `Shell` instance on the server. The WebSocket
connection is the session lifetime. On connect, the server calls the supplied
`shell_factory()` to create a fresh shell, renders all panels, and sends the initial
HTML to the browser. From then on, every key event from the browser triggers a
`Shell.handle_key()` call, and only dirty regions are re-rendered and sent back.

When the shell signals exit (Escape / Ctrl+Q by default), the server sends an `exit`
message and closes the connection.

## Web note on widget semantics

The portable-library spec describes widgets as blocking: `widget.show(sh)` returns after
the user acts. In `panelmark-web`, sessions are async — there is no call that blocks until
a widget is dismissed.

**Web-appropriate pattern:** assign the widget to a panel region and rely on
`signal_return()` to deliver the result when the user acts. Constructor signatures,
`get_value()` / `set_value()` / `signal_return()` semantics, and return values match the
portable spec exactly.

## Dependencies

| Package | Role |
|---------|------|
| `panelmark` | Shell, Interaction ABC, draw commands |
| `panelmark-html` | Static page structure and base CSS |
| `fastapi` / `starlette` *(optional)* | Async ASGI server adapter |
| `flask` + `flask-sock` *(optional)* | Sync WSGI server adapter |

## Installation

```bash
pip install panelmark-web[fastapi]   # FastAPI / Starlette
pip install panelmark-web[flask]     # Flask + flask-sock
```

## In this section

- [Getting Started](getting-started.md)
- [Protocol](protocol.md)
- [Hook Usage](hook-usage.md)
- [Interaction Coverage](interaction-coverage.md)

## See also

- [panelmark-web package overview](../packages/panelmark-web.md)
- [panelmark-html Overview](../panelmark-html/overview.md)
- [panelmark-html Hook Contract](../panelmark-html/hook-contract.md)
- [Portable Library spec](../renderer-spec/portable-library.md)
