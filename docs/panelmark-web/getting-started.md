# Getting Started with panelmark-web

`panelmark-web` is a live web runtime for the panelmark ecosystem. It provides the
transport and rendering infrastructure to host `panelmark.Interaction` objects in a
browser: WebSocket-driven keyboard input, focus transitions, and live rendering of
interaction content inside panel bodies.

## Prerequisites

- Python 3.11+
- `panelmark` installed
- `panelmark-html` installed

## Install

```bash
pip install panelmark-web[fastapi]   # FastAPI / Starlette
pip install panelmark-web[flask]     # Flask + flask-sock
```

Or from source:

```bash
cd panelmark-web
pip install -e ".[fastapi]"   # or .[flask]
```

---

## Step 1 — Define your interactions

Implement the `Interaction` ABC from `panelmark`. Any interaction whose `render()` returns
`WriteCmd` and `FillCmd` commands works out of the box. The built-in interactions in
`panelmark_web.interactions` can also be used directly.

```python
from panelmark.interactions.base import Interaction
from panelmark.draw import WriteCmd, FillCmd, RenderContext


class EchoEditor(Interaction):
    def __init__(self):
        self._text = ""

    def render(self, context: RenderContext, focused: bool = False) -> list:
        line = (self._text + ("_" if focused else " "))[: context.width]
        line = line.ljust(context.width)
        return [
            FillCmd(row=0, col=0, width=context.width, height=context.height),
            WriteCmd(row=0, col=0, text=line, style={"reverse": True} if focused else None),
        ]

    def handle_key(self, key: str) -> tuple:
        if key == "KEY_BACKSPACE" and self._text:
            self._text = self._text[:-1]
            return True, self._text
        if len(key) == 1 and key.isprintable():
            self._text += key
            return True, self._text
        return False, self._text

    def get_value(self) -> str:
        return self._text

    def set_value(self, value) -> None:
        self._text = str(value)
```

---

## Step 2 — Define a shell and a factory

```python
from panelmark import Shell

SHELL_DEF = """
|===========|
|{40R $editor$ }|
|===========|
"""


def make_shell() -> Shell:
    shell = Shell(SHELL_DEF)
    shell.assign("editor", EchoEditor())
    shell.set_focus("editor")
    return shell
```

`shell_factory` is called once per WebSocket connection, so each browser tab gets its own
independent shell state.

---

## Step 3 — Wire up the server (FastAPI)

```python
import pathlib
from fastapi import FastAPI, WebSocket
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from panelmark_html import render_document
from panelmark_web.server import handle_connection
from panelmark_web.adapters import StarletteAdapter
from panelmark_web.page import prepare_page

app = FastAPI()

# Serve client.js from panelmark_web/static/
_static = pathlib.Path(__file__).parent / "panelmark_web" / "static"
app.mount("/static", StaticFiles(directory=str(_static)), name="static")


@app.get("/", response_class=HTMLResponse)
async def index():
    shell = make_shell()
    html = render_document(shell)
    return prepare_page(html, ws_url="/ws", script_src="/static/client.js")


@app.websocket("/ws")
async def ws_endpoint(websocket: WebSocket):
    await websocket.accept()
    await handle_connection(StarletteAdapter(websocket), shell_factory=make_shell)
```

`prepare_page` injects `data-pm-ws-url` on the shell root element and a `<script>` tag
before `</body>`. The browser client reads `data-pm-ws-url` to know which WebSocket
endpoint to connect to.

`StarletteAdapter` translates Starlette's `receive_text()` / `send_text()` API into the
`recv()` / `send()` interface that `handle_connection` expects. `WebSocketDisconnect` is
caught automatically and exits the connection cleanly.

---

## Step 4 — Run

```bash
uvicorn myapp:app --reload
```

Open `http://localhost:8000/` in a browser. The panel body populates on connect and updates
as you type.

---

## Flask / flask-sock

Use `handle_connection_sync` with `flask-sock`:

```python
from flask import Flask
from flask_sock import Sock
from panelmark_html import render_document
from panelmark_web.server import handle_connection_sync
from panelmark_web.page import prepare_page

app = Flask(__name__)
sock = Sock(app)

@app.get("/")
def index():
    html = render_document(make_shell())
    return prepare_page(html, ws_url="/ws", script_src="/static/client.js")

@sock.route("/ws")
def ws_endpoint(ws):
    handle_connection_sync(ws, shell_factory=make_shell)
```

`handle_connection_sync` blocks the thread for the entire WebSocket lifetime. For
production use gevent:

```bash
pip install gunicorn gevent
gunicorn -k gevent -w 4 "myapp:app"
```

For simpler production deployment without the gevent dependency, use the FastAPI path with
`uvicorn` — async handles many concurrent WebSocket connections without threading
configuration.

---

## WebSocket URL configuration

By default `prepare_page` sets `ws_url="/ws"`. Override it for apps mounted under a
subpath or using a different endpoint:

```python
# App mounted at /myapp
prepare_page(html, ws_url="/myapp/ws", script_src="/myapp/static/client.js")

# Absolute URL (cross-origin or non-standard port)
prepare_page(html, ws_url="wss://ws.example.com/ws")
```

---

## Theming

Override CSS custom properties to theme the shell without modifying `panelmark-html`
output:

```css
:root {
  --pm-border-color: #333;
  --pm-focused-border-color: #00aaff;
  --pm-focused-border-width: 2px;
}
```

See [panelmark-html Hook Contract — CSS custom properties](../panelmark-html/hook-contract.md#css-custom-properties)
for the full property list.

---

## See also

- [panelmark-web Overview](overview.md)
- [Protocol](protocol.md)
- [Hook Usage](hook-usage.md)
- [Interaction Coverage](interaction-coverage.md)
- [panelmark-web package overview](../packages/panelmark-web.md)
