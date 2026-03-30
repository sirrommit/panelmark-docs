# Glossary

Definitions for terms used across the panelmark ecosystem.

## Core terms

**shell** — the top-level layout container defined by the panelmark shell language. A shell declares named regions and their spatial relationships. At runtime a `panelmark.Shell` instance manages the layout model, focus state, and key dispatch.

**region** — a named rectangular area within a shell. Regions are the unit of layout, focus, and interaction assignment. Each region has a resolved row, column, width, and height.

**panel** — the visual content displayed inside a region. Panels are driven by interactions.

**interaction** — a stateful object assigned to a region. An interaction implements `render()` to produce draw commands and `handle_key()` to respond to input. The `Interaction` ABC is defined in `panelmark` core.

**draw command** — a low-level output instruction produced by an interaction's `render()` method and consumed by a renderer. The three types are `WriteCmd` (write text at a position), `FillCmd` (fill a rectangle), and `CursorCmd` (position the cursor). Draw commands use region-relative coordinates.

**RenderContext** — the context passed to `interaction.render()`. Carries the region's `width`, `height`, and a `capabilities` frozenset of feature flags supported by the renderer (e.g. `'color'`, `'unicode'`).

**renderer** — a package that hosts a `panelmark.Shell`, drives the event loop, and executes draw commands on a specific output surface (terminal, HTML document, browser).

**shell language** — the ASCII-art DSL used to define a shell layout. Written as a multi-line string; parsed by `panelmark` core into a tree of `HSplit`, `VSplit`, and `Panel` nodes.

## Portability terms

**portable library** — the set of interactions and widgets whose constructor signatures, value semantics, and lifecycle behaviour are standardised by the renderer spec. A renderer claiming `portable-library-compatible` status implements all of them.

**portable-library-compatible** — a renderer compatibility level. Means the renderer implements the core contract plus all required interactions and widgets from the portable standard library.

**renderer-specific** — behavior, interactions, or widgets that are defined by a renderer package and not required to match across renderers. Renderer-specific additions must be clearly marked as such and live under the renderer's own namespace (e.g. `panelmark_tui.Toast`, not `panelmark.Toast`).

**core-compatible** — the minimum renderer compatibility level. Means the renderer implements the core shell and interaction hosting contract but makes no claims about the portable library.

## Package terms

**panelmark** — the core package. Zero-dependency. Provides the shell language, parser, layout model, `Shell` class, and `Interaction` ABC.

**panelmark-tui** — the terminal renderer. Uses `blessed` to render to a real terminal. Claims `portable-library-compatible` status.

**panelmark-html** — the static HTML/CSS renderer. Produces HTML strings from a shell; does not handle live interaction or browser events. Pre-alpha.

**panelmark-web** — the live web runtime. Adds WebSocket sessions, keyboard relay, and server-side draw-command-to-HTML rendering on top of `panelmark-html`. Claims `portable-library-compatible` status.

## See also

- [Ecosystem](ecosystem.md)
- [Renderer Spec](renderer-spec/overview.md)
- [Extensions and Compatibility](renderer-spec/extensions.md)
- [Shell Language Overview](shell-language/overview.md)
