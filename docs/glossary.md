# Glossary

Definitions for terms used across the panelmark ecosystem.

*This page will be expanded as the docs site is built out. See [Phase 7 of the documentation plan](https://github.com/sirrommit/panelmark-docs).*

## Core terms

**shell** — the top-level layout container defined by the panelmark shell language. A shell declares named regions and their spatial relationships.

**region** — a named rectangular area within a shell. Regions are the unit of layout and focus.

**panel** — the content displayed inside a region. Panels are driven by interactions.

**interaction** — a stateful object that renders itself into a region using draw commands and responds to input events.

**draw command** — a low-level instruction (fill, write, cursor) produced by an interaction and consumed by a renderer to produce visible output.

**renderer** — a package that takes a shell state and produces output on a specific surface (terminal, HTML, browser).

**portable library** — the set of interactions and widgets whose constructor signatures and value semantics are specified by the renderer spec and must be consistent across all compliant renderers.

**renderer-specific** — behavior, interactions, or widgets that are defined by a renderer package and not required to be portable across renderers.

## See also

- [Ecosystem](ecosystem.md)
- [Renderer Spec](renderer-spec/overview.md)
