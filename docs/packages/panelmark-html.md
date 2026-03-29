# panelmark-html

**panelmark-html** is the static HTML/CSS renderer for panelmark. Given a panelmark shell and its assigned interactions, it produces an HTML document or fragment representing the shell's panel structure.

It is the structural foundation that `panelmark-web` builds live interactivity on top of.

## What it provides

- Renders shell structure into HTML with stable panel hooks and layout classes
- Emits base CSS for shell structure (`get_base_css()`)
- Defines the hook contract — stable DOM attributes and CSS classes — consumed by `panelmark-web`
- Supports server-side rendering into Flask/Django templates, reports, and dashboards

**Public API:** `render_fragment`, `render_document`, `get_base_css`, `HTMLRenderer`

## What it does not include

- Browser event handling, keyboard or mouse input, focus transitions
- HTTP routes, WebSockets, or session management
- Rendering of interaction-internal state (item lists, form fields, text content) — panel bodies are empty placeholders with stable DOM hooks; filling them in with live content is `panelmark-web`'s job
- Any JavaScript requirement for its output to be valid HTML

## Who should use it

- Application authors doing **server-side rendering** into HTML templates without a live browser session
- Authors generating **static dashboards or reports** from panelmark shell layouts
- **panelmark-web** (as a dependency — you typically don't need to install it directly when using `panelmark-web`)
- Renderer authors or testers who want to snapshot-test rendered HTML output

## Current status

**Pre-alpha.** The public Python API and higher-level rendering features may still evolve.

The **hook contract** — the region-level DOM hooks (`data-pm-region`, `id`, `data-pm-*` attributes) and CSS classes (`.pm-shell`, `.pm-split-*`, `.pm-panel`, `.pm-panel-body`) — is the intended stable substrate for `panelmark-web` and will not change without a major version bump.

## Installation

```
pip install panelmark-html
```

Dependencies: `panelmark`. No web framework dependency. No JavaScript build step.

## Relationship to panelmark-web

| Package | Role |
|---------|------|
| **panelmark-html** | Static structure: panel layout, borders, headings, stable DOM hooks |
| **panelmark-web** | Live layer: browser events, interaction rendering, WebSocket sessions |

`panelmark-web` depends on `panelmark-html` for its rendered structure. The DOM hooks and CSS classes defined here are the stable contract between the two packages.

## Detailed docs

- [Overview](../panelmark-html/overview.md)
- [Rendering API](../panelmark-html/rendering-api.md)
- [Hook Contract](../panelmark-html/hook-contract.md)

## See also

- [panelmark-web](panelmark-web.md)
- [Ecosystem](../ecosystem.md)
- [panelmark core](panelmark.md)
