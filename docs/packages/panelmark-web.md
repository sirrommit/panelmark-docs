# panelmark-web

The live web runtime for panelmark. Built on top of `panelmark-html`, it adds WebSocket session management, browser keyboard input transport, and server-side rendering of panel bodies from draw commands.

## What it provides

- WebSocket session management (server-side, framework-agnostic)
- browser keyboard input transport (client-side JavaScript)
- server-side rendering of panel content from draw commands via `panelmark-html`
- live focus and update propagation to the browser
- configurable WebSocket URL for subpath deployments

## What it does not include

- a built-in portable interaction/widget library: interactions are hosted generically via the draw-command path, not via a renderer-provided standard library

## Who should use it

Application authors who want to build live browser-based panelmark applications using Python server frameworks (FastAPI, Flask, etc.).

## Detailed docs

- [Overview](../panelmark-web/overview.md)
- [Getting Started](../panelmark-web/getting-started.md)
- [Protocol](../panelmark-web/protocol.md)
- [Hook Usage](../panelmark-web/hook-usage.md)
- [Interaction Coverage](../panelmark-web/interaction-coverage.md)

## See also

- [panelmark-html](panelmark-html.md)
- [Ecosystem](../ecosystem.md)
- [Renderer Spec](../renderer-spec/overview.md)
