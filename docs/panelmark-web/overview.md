# panelmark-web Overview

`panelmark-web` is a live web runtime built on top of `panelmark-html`. It provides WebSocket-based session management, browser keyboard input transport, and server-side rendering of panel bodies from draw commands.

*Full content is being migrated from `panelmark-web/README.md`. See the documentation plan.*

## What panelmark-web provides

- WebSocket session management (server-side)
- browser keyboard input transport (client-side JavaScript)
- server-side rendering of panel content from draw commands via `panelmark-html`
- live focus and update propagation to the browser

## What panelmark-web does not provide

- a built-in portable interaction/widget library (interactions are hosted generically via the draw-command path)

## In this section

- [Getting Started](getting-started.md)
- [Protocol](protocol.md)
- [Hook Usage](hook-usage.md)
- [Interaction Coverage](interaction-coverage.md)

## See also

- [panelmark-web package overview](../packages/panelmark-web.md)
- [panelmark-html Overview](../panelmark-html/overview.md)
- [panelmark-html Hook Contract](../panelmark-html/hook-contract.md)
