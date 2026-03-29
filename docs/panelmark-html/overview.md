# panelmark-html Overview

`panelmark-html` renders a panelmark shell into a static HTML/CSS structure. It is the DOM substrate on which `panelmark-web` builds live browser sessions.

*Full content is being migrated from `panelmark-html/README.md`. See the documentation plan.*

## What panelmark-html provides

- renders shell structure into HTML with stable panel hooks and layout classes
- emits base CSS for shell structure
- defines the hook contract used by `panelmark-web`

## What panelmark-html does not provide

- browser event handling
- live interaction runtime
- server or session management

## In this section

- [Rendering API](rendering-api.md)
- [Hook Contract](hook-contract.md)

## See also

- [panelmark-html package overview](../packages/panelmark-html.md)
- [panelmark-web Overview](../panelmark-web/overview.md)
