# panelmark-html Rendering API

The Python API for rendering a `panelmark` shell into HTML.

```python
from panelmark_html import render_fragment, render_document, get_base_css, HTMLRenderer
```

---

## render_fragment

```python
render_fragment(shell, *, include_css: bool = False) -> str
```

Returns an HTML fragment for `shell` — a `<div class="pm-shell">` root element with the
full panel structure inside.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `shell` | `panelmark.Shell` | — | The shell to render |
| `include_css` | `bool` | `False` | If `True`, prepend a `<style>` block with the base CSS |

```python
from panelmark import Shell
from panelmark_html import render_fragment

shell = Shell(LAYOUT)
fragment = render_fragment(shell)
# → '<div class="pm-shell" data-pm-shell>\n  ...\n</div>\n'

fragment_with_css = render_fragment(shell, include_css=True)
# → '<style>\n...\n</style>\n<div class="pm-shell" ...'
```

Use `render_fragment` when embedding the shell inside an existing page template.

---

## render_document

```python
render_document(
    shell,
    *,
    title: str = "panelmark",
    css_href: str | None = None,
    extra_head: str = "",
) -> str
```

Returns a complete `<!DOCTYPE html>` document with `<html>`, `<head>`, and `<body>`.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `shell` | `panelmark.Shell` | — | The shell to render |
| `title` | `str` | `"panelmark"` | Value for the `<title>` element |
| `css_href` | `str \| None` | `None` | Link to an external stylesheet; if given, the base CSS is not inlined |
| `extra_head` | `str` | `""` | Raw HTML appended inside `<head>` before `</head>` |

```python
from panelmark import Shell
from panelmark_html import render_document

shell = Shell(LAYOUT)

# Fully self-contained HTML file (CSS inlined):
html = render_document(shell, title="My App")

# Link to an external stylesheet instead of inlining:
html = render_document(
    shell,
    title="My App",
    css_href="/static/panelmark.css",
)

# Embed additional head content (e.g., a custom script):
html = render_document(
    shell,
    title="My App",
    extra_head='<script src="/static/app.js" defer></script>',
)
```

---

## get_base_css

```python
get_base_css() -> str
```

Returns the base CSS string that styles the shell layout. Use this to write the stylesheet
to a static file for production:

```python
from panelmark_html import get_base_css

with open("static/panelmark.css", "w") as f:
    f.write(get_base_css())
```

The CSS uses custom properties (`--pm-border-color`, `--pm-gap`, etc.) as theming hooks.
See [Hook Contract — CSS custom properties](hook-contract.md#css-custom-properties) for the
full list of overridable properties.

---

## HTMLRenderer

```python
from panelmark_html import HTMLRenderer

renderer = HTMLRenderer()
fragment = renderer.render_fragment(shell, include_css=False)
document = renderer.render_document(shell, title="...", css_href=None, extra_head="")
```

`HTMLRenderer` is the class that backs `render_fragment` and `render_document`. The module-level
functions are thin wrappers that create a fresh `HTMLRenderer()` on each call.

Use the class directly if you need to render multiple shells with the same instance, or if you
want to subclass `HTMLRenderer` to override internal rendering behaviour.

---

## Flask / Django integration

`render_fragment` and `render_document` return plain strings and have no web framework
dependencies.

```python
# Flask
from flask import Flask
from panelmark import Shell
from panelmark_html import render_fragment

app = Flask(__name__)

@app.route("/dashboard")
def dashboard():
    shell = Shell(LAYOUT)
    # assign interactions...
    return render_fragment(shell, include_css=True)
```

```python
# Django
from django.http import HttpResponse
from panelmark import Shell
from panelmark_html import render_document

def dashboard(request):
    shell = Shell(LAYOUT)
    return HttpResponse(render_document(shell, title="Dashboard"))
```

---

## What is rendered

`render_fragment` and `render_document` always render the **structural shell** — layout, borders,
panel containers, and headings. They do **not** render interaction content inside panel bodies.

| Rendered | Not rendered |
|----------|-------------|
| Shell root (`pm-shell`) | Interaction item lists |
| Split containers (`pm-split-h`, `pm-split-v`) | Form field values |
| Panel elements with stable hooks | Text box contents |
| Panel headings (`pm-panel-heading`) | Focused state transitions |
| Interaction metadata attributes (`data-pm-interaction`, etc.) | |
| Empty panel body placeholder (`pm-panel-body`) | |

Panel bodies are intentionally empty `<div class="pm-panel-body"></div>` elements.
`panelmark-web` populates them with live interaction content.

For the stable hooks you can depend on, see [Hook Contract](hook-contract.md).

---

## See also

- [Hook Contract](hook-contract.md)
- [panelmark-html Overview](overview.md)
- [panelmark-web Hook Usage](../panelmark-web/hook-usage.md)
