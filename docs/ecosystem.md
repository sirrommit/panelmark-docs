# Ecosystem

panelmark is a layout language and rendering framework for Python user interfaces. A developer sketches a UI as an ASCII-art shell definition string, and that same definition renders correctly through any supported renderer. The shell is written once; the renderer is chosen at deployment time.

## Package map

```
panelmark                 # core: layout language, parser, shell state machine
├── panelmark-tui         # terminal renderer (blessed)
├── panelmark-html        # static HTML/CSS renderer
│   └── panelmark-web     # live web renderer (depends on panelmark-html)
```

## panelmark (core)

The core package contains everything that is not renderer-specific:

- The shell definition parser
- The layout model (`HSplit`, `VSplit`, `Panel`, `Region` resolution)
- The `Shell` state machine (`assign`, `update`, `bind`, `on_change`, `handle_key`, `dirty_regions`)
- The `Interaction` abstract base class
- Style tag parsing (`parse_styled`, `styled_plain_text`)
- Exceptions (`RegionNotFoundError`, `CircularUpdateError`)

**Dependencies:** none.

The zero-dependency constraint is intentional. Any renderer can depend on `panelmark` without pulling in `blessed`, a web framework, or a GUI toolkit. The core is also the natural target for future tooling — a layout linter, a shell file previewer, or a headless test harness — none of which should require a rendering backend.

## panelmark-tui

The terminal renderer. Extends `panelmark.Shell` with a `blessed`-powered event loop.

**Adds:**

- `Shell.run()` — fullscreen terminal event loop
- `Shell.run_modal()` — modal popup loop inside an existing terminal context
- `Renderer` — draws box-drawing characters, borders, and region content to a terminal
- Concrete `Interaction` library covering common UI patterns
- Prebuilt modal widgets for common prompts and dialogs
- Testing utilities (`MockTerminal`, `make_key`)

**Dependencies:** `panelmark`, `blessed >= 1.20`.

## panelmark-html

The static HTML/CSS renderer. Given a layout model and assigned interactions, produces an HTML document or fragment representing the current state of the shell.

**Scope:** pure rendering — layout model in, HTML string out. No network layer, no server, no JavaScript beyond what is needed to represent static state. Useful for:

- Server-side rendering of a panelmark shell into a Flask/Django template
- Generating static dashboards or reports
- Automated visual testing (render to HTML, diff against a snapshot)
- The foundation layer for `panelmark-web`

**Dependencies:** `panelmark`. No web framework dependency.

## panelmark-web

The live web runtime. Adds a server-side session layer and a JavaScript client on top of `panelmark-html`.

**Adds over panelmark-html:**

- A WebSocket server integration (Flask, FastAPI) that hosts a panelmark shell as a live session
- A WebSocket channel between the server and browser, carrying key events from the client and shell state updates back
- A JavaScript client that applies incremental updates to the rendered HTML without full page reloads
- Session management and multiplexing for concurrent users

**Why the split from panelmark-html exists:**

`panelmark-html` has a natural, bounded scope: render a layout model to HTML. That scope is useful on its own and carries no server or network dependency. `panelmark-web` adds a full interactive stack on top. Separating them means:

1. The rendering layer can be developed, tested, and used independently of the live session layer.
2. The dependency graph is honest: `panelmark-web` depends on `panelmark-html` for rendering, just as `panelmark-tui` depends on `panelmark` for layout.
3. Users doing server-side rendering without a live connection do not pull in WebSocket dependencies.

**Dependencies:** `panelmark-html`, a web framework (Flask or FastAPI).

## Design principles

As renderers are added, pressure builds to add renderer-specific concepts to the core. The following constraints hold regardless:

- `panelmark.Shell` must not import or reference any renderer. The terminal and web renderers are equal peers from the core's perspective.
- The shell definition language must render correctly through every renderer without modification. Renderer-specific annotations, if ever needed, belong in the renderer's own configuration layer, not in the shell string.
- The `Interaction` base class must not assume a terminal. The `render()` signature uses `RenderContext` and returns `list[DrawCommand]`, keeping interactions renderer-agnostic. Renderer capability queries are available via `context.supports(feature)` — interactions can branch on capabilities without depending on any renderer directly.

## See also

- [Renderer Spec](renderer-spec/overview.md)
- [Shell Language](shell-language/overview.md)
- [Glossary](glossary.md)
