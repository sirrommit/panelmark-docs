# panelmark

`panelmark` is a multi-package UI ecosystem centered on a shared shell language. Define a UI layout once and render it across terminal, static HTML, and live web surfaces.

## Core concept

- define a UI layout using the [panelmark shell language](shell-language/overview.md)
- render it via a renderer package appropriate to your surface
- portable interactions and widgets work consistently across compliant renderers

## Packages

| Package | Role |
|---|---|
| [`panelmark`](packages/panelmark.md) | Core shell language, parser, layout model, interaction base |
| [`panelmark-tui`](packages/panelmark-tui.md) | Terminal renderer |
| [`panelmark-html`](packages/panelmark-html.md) | Static HTML/CSS structural renderer |
| [`panelmark-web`](packages/panelmark-web.md) | Live web runtime built on `panelmark-html` |

## Where to start

- New to panelmark? Read [Getting Started](getting-started.md)
- Want the big picture? Read [Ecosystem](ecosystem.md)
- Building a renderer? Read the [Renderer Spec](renderer-spec/overview.md)
- Looking up a term? Check the [Glossary](glossary.md)
