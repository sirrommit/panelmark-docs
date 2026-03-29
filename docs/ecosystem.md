# Ecosystem

The panelmark ecosystem is organized as a core library plus renderer packages. The core library defines the shell language and the renderer contract. Renderer packages implement that contract for specific surfaces.

*Full content is being migrated from `panelmark/ECOSYSTEM.md` in Phase 2.*

## Package map

| Package | Role | Surface |
|---|---|---|
| `panelmark` | Core shell language, parser, layout model, interaction base, draw commands | (surface-agnostic) |
| `panelmark-tui` | Terminal renderer | Terminal (blessed) |
| `panelmark-html` | Static HTML/CSS structural renderer | HTML/CSS |
| `panelmark-web` | Live web runtime on top of `panelmark-html` | Browser (WebSocket) |

## Design principles

- keep `panelmark` core renderer-agnostic
- standardize semantics before renderer shape
- let renderer packages add surface-appropriate behavior
- document portable guarantees separately from renderer-specific additions

## Renderer compliance

Renderers declare their compliance level against the [Renderer Spec](renderer-spec/overview.md). The spec defines:

- the required [renderer contract](renderer-spec/contract.md)
- the [portable library](renderer-spec/portable-library.md) of interactions and widgets
- how [extensions](renderer-spec/extensions.md) should be documented
- how to assess [readiness](renderer-spec/readiness.md)

## See also

- [Renderer Spec Overview](renderer-spec/overview.md)
- [Glossary](glossary.md)
