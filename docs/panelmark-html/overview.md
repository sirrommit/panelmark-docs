# panelmark-html Overview

`panelmark-html` is the static HTML/CSS renderer for the panelmark ecosystem. Given a
`panelmark` shell and its assigned interactions, it produces an HTML document or fragment
representing the shell's panel structure.

**Package maturity:** Pre-alpha. The Python API (`render_fragment`, `render_document`,
`get_base_css`, `HTMLRenderer`) and higher-level rendering features may still evolve.

**Hook contract:** The region-level DOM hooks and CSS classes are documented as stable and
will not change without a major version bump. See [Hook Contract](hook-contract.md).

## What panelmark-html provides

- Renders shell structure into HTML with stable panel hooks and layout classes
- Emits base CSS for shell layout
- Defines the DOM hook contract consumed by `panelmark-web`

## What panelmark-html does not provide

- Browser event handling, keyboard or mouse input, or focus transitions
- Interaction-internal state (item lists, form fields, text content) — panel bodies are
  empty placeholders; filling them with live content is the job of `panelmark-web`
- Server, socket, HTTP routes, or session management
- Any JavaScript dependency — the output is valid HTML without JS

## Relationship to panelmark-web

`panelmark-html` and `panelmark-web` are separate packages with distinct scopes.

| Package | Role |
|---------|------|
| **panelmark-html** | Static structure: panel layout, borders, headings, stable DOM hooks |
| **panelmark-web** | Live layer: browser events, interaction rendering, server sessions |

`panelmark-web` depends on `panelmark-html` for its rendered structure. The DOM hooks and
CSS classes defined here are the stable contract between the two packages.

## Installation

```
pip install panelmark-html
```

**Dependencies:** `panelmark` core only. No web framework, no JavaScript build step.

## Quick example

```python
from panelmark import Shell
from panelmark_html import render_document

LAYOUT = """
|=== Dashboard ===|
|{ $menu$         }|
|{ $content$      }|
|=================|
"""

shell = Shell(LAYOUT)
# Assign interactions if desired; panel bodies will be empty placeholders.
html = render_document(shell, title="My App")
```

Output is a complete `<!DOCTYPE html>` document with:

- A `.pm-shell` root wrapping the layout
- `.pm-split` containers for each split
- `<section class="pm-panel">` elements with stable `data-pm-region` hooks for each
  named region

See [Rendering API](rendering-api.md) for the full function reference.

## In this section

- [Rendering API](rendering-api.md)
- [Hook Contract](hook-contract.md)

## See also

- [panelmark-html package overview](../packages/panelmark-html.md)
- [panelmark-web Overview](../panelmark-web/overview.md)
- [Hook Usage in panelmark-web](../panelmark-web/hook-usage.md)
