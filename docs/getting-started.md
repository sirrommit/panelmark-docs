# Getting Started

## Which package do I need?

| Goal | Install | Start here |
|------|---------|-----------|
| Build a **terminal** (TUI) application | `pip install panelmark-tui` | [panelmark-tui: Getting Started](panelmark-tui/getting-started.md) |
| Build a **web** application (FastAPI / Flask) | `pip install panelmark-web[fastapi]` | [panelmark-web: Getting Started](panelmark-web/getting-started.md) |
| Generate **static HTML** from a shell (server-side rendering, reports) | `pip install panelmark-html` | [panelmark-html Overview](panelmark-html/overview.md) |
| Implement a **new renderer** | `pip install panelmark` | [Renderer Spec](renderer-spec/overview.md) |
| Use the **shell language** without a renderer | `pip install panelmark` | [Shell Language](shell-language/overview.md) |

`panelmark` (core) is a dependency of all renderer packages. You do not need to install it separately.

---

## Choose your path

**I want to build a terminal application**

1. Read [Ecosystem](ecosystem.md) for the overall picture
2. Read [panelmark-tui: Getting Started](panelmark-tui/getting-started.md)
3. Browse the [shell language](shell-language/overview.md) reference

**I want to build a web application**

1. Read [Ecosystem](ecosystem.md) for the overall picture
2. Read [panelmark-web: Getting Started](panelmark-web/getting-started.md)
3. Review [panelmark-html Overview](panelmark-html/overview.md) to understand the static substrate

**I want to implement a new renderer**

1. Read [Ecosystem](ecosystem.md)
2. Read the full [Renderer Spec](renderer-spec/overview.md)
3. Review [panelmark-tui Renderer Implementation](panelmark-tui/renderer-implementation.md) as a reference

**I want to understand the shell language**

1. Read [Shell Language Overview](shell-language/overview.md)
2. See [Shell Language Examples](shell-language/examples.md)

---

## Portable vs renderer-specific

Some interactions and widgets are **portable** — their constructor signatures, value
semantics, and lifecycle behaviour are standardised by the [portable library spec](renderer-spec/portable-library.md).
Any renderer claiming `portable-library-compatible` status implements all of them. If you
use only portable APIs, your application logic is not coupled to a specific renderer.

Other interactions and widgets are **renderer-specific** — they are defined by the renderer
package and only work on that surface. For example, `panelmark_tui.widgets.Toast` is
specific to the terminal renderer. Renderer-specific additions are clearly marked in each
renderer's documentation.

| Kind | Defined in | Works across renderers? |
|------|-----------|------------------------|
| Portable interaction / widget | `panelmark` renderer spec | Yes — on any `portable-library-compatible` renderer |
| Renderer-specific interaction / widget | Renderer package (e.g. `panelmark_tui`) | No — only on that renderer |
| Shell language | `panelmark` core | Yes — all renderers parse the same shell DSL |
| Draw commands (`WriteCmd`, `FillCmd`, `CursorCmd`) | `panelmark` core | Yes — custom interactions using only core draw commands are portable |

---

## Audiences

- **application authors** — use the package guides for your target surface
- **renderer authors** — use the renderer spec and existing renderer implementations as reference
- **contributors to `panelmark` core** — read the ecosystem overview and renderer spec first
