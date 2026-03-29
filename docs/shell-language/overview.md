# Shell Language Overview

The panelmark shell language is a declarative format for describing terminal/web UI layouts. It defines the structure of a shell — named regions, their positions, and how they relate — independently of any renderer.

*Full content is being migrated from `panelmark/docs/shell-language.md`. See the documentation plan.*

## What the shell language defines

- named regions and their spatial arrangement
- which region holds initial focus
- how panels are associated with regions at runtime

## What the shell language does not define

- how a region's content is rendered (that is the renderer's job)
- interaction-specific behavior or input handling

## In this section

- [Syntax](syntax.md) — shell language grammar and structure
- [Examples](examples.md) — annotated shell definitions

## See also

- [Ecosystem](../ecosystem.md)
- [Renderer Spec](../renderer-spec/overview.md)
- [panelmark package](../packages/panelmark.md)
