# panelmark-html

The static HTML/CSS rendering layer. Renders a panelmark shell into an HTML structure with stable hooks and layout classes. Serves as the DOM substrate on which `panelmark-web` builds live sessions.

## What it provides

- renders shell structure into HTML
- emits stable panel hooks and layout classes
- provides base CSS for shell structure
- defines the hook contract consumed by `panelmark-web`

## What it does not include

- browser event handling
- live interaction runtime
- server or session management

## Who should use it

- authors building static HTML output from panelmark shells
- `panelmark-web` (as a dependency)
- renderer authors who want to understand the HTML substrate

Application authors building live browser applications should use `panelmark-web` rather than `panelmark-html` directly.

## Detailed docs

- [Overview](../panelmark-html/overview.md)
- [Rendering API](../panelmark-html/rendering-api.md)
- [Hook Contract](../panelmark-html/hook-contract.md)

## See also

- [panelmark-web](panelmark-web.md)
- [Ecosystem](../ecosystem.md)
