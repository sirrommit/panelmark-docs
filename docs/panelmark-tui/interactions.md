# panelmark-tui Interactions

panelmark-tui provides 13 built-in interaction types, all importable from `panelmark_tui.interactions`.

```python
from panelmark_tui.interactions import (
    MenuFunction, MenuReturn,
    TextBox,
    ListView,
    CheckBox,
    Function,
    FormInput, DataclassFormInteraction,
    StatusMessage,
    TreeView,
    RadioList,
    TableView,
    NestedMenu, Leaf,
)
```

These interactions implement the `panelmark.Interaction` ABC defined in the [Renderer Contract](../renderer-spec/contract.md). For guidance on writing your own interactions, see [Shell Language Examples](../shell-language/examples.md).

Interactions marked **Portable** implement the [Portable Library](../renderer-spec/portable-library.md) spec — their constructor signatures and value semantics are standardized across compliant renderers. Interactions marked **Frequently implemented** follow the portable spec's recommended API but are not required for `portable-library-compatible` status.

## API contract

Every interaction follows a three-concept model.

**Current logical state** — the value that best represents what this interaction currently contains or has selected. Always accessible via `get_value()` and restorable via `set_value()`.

**Submitted result** — a value produced when the user explicitly accepts or submits. Returned through `signal_return()`, not through `get_value()`.

**Display model** — the backing content of a display-only interaction. Display-only interactions expose their display model through `get_value()`.

## Interaction summary

| Interaction | `get_value()` | `set_value()` | `signal_return()` | Portable |
|---|---|---|---|---|
| `MenuReturn` | highlighted label | highlight label | mapped payload on accept | ✅ |
| `MenuFunction` | highlighted label | highlight label | none | Frequently impl. |
| `CheckBox` | full checked-state dict | replace checked-state dict | none | ✅ |
| `RadioList` | current selected value | select by value | selected value on accept | ✅ |
| `TreeView` | highlighted path tuple | highlight path | path on leaf accept | Frequently impl. |
| `NestedMenu` | highlighted path tuple | navigate to path | mapped payload on leaf accept | ✅ |
| `TableView` | active row index | set active row index | none | Frequently impl. |
| `TextBox` | current text | replace text | text on accept (submit mode) | ✅ |
| `ListView` | current items list | replace items list | none | Frequently impl. |
| `StatusMessage` | `(style, message)` or `None` | replace status payload | none | ✅ |
| `FormInput` | current field-state dict | replace field-state dict | submitted dict on submit | ✅ |
| `DataclassFormInteraction` | current field-state dict | replace field-state dict | action result on submit | ✅ |
| `Function` | internal `_value` (escape hatch) | sets `_value` | never | — |

---

## MenuReturn

> **Portable** — part of the [Portable Standard Library](../renderer-spec/portable-library.md).

A scrollable menu that immediately returns a value when the user selects an item.

```python
MenuReturn(items: dict)
```

`items` maps display labels to return values. When the user presses Enter on a label, the shell exits with the corresponding value.

```python
from panelmark_tui.interactions import MenuReturn

menu = MenuReturn({
    "New file":  "new",
    "Open...":   "open",
    "Save":      "save",
    "Quit":      "quit",
})
sh.assign("menu", menu)
result = sh.run()    # returns "new", "open", "save", or "quit"
```

**Keys:** `↑`/`↓` or `k`/`j` to navigate; `Page Up`/`Page Down` to jump by a full page; `Home`/`End` to jump to first/last item; `Enter` to select.

---

## MenuFunction

> **Frequently implemented** — follows the [portable spec's recommended API](../renderer-spec/portable-library.md#menufunction-interaction).

A scrollable menu that calls a Python function when the user selects an item. Does not exit the shell automatically.

```python
MenuFunction(items: dict)
```

`items` maps display labels to callables. Each callable receives the shell as its first argument: `callback(shell)`.

```python
from panelmark_tui.interactions import MenuFunction

def open_file(sh):
    from panelmark_tui.widgets import FilePicker
    path = FilePicker().show(parent_shell=sh)
    if path:
        sh.update("status", ("success", f"Opened {path}"))

menu = MenuFunction({
    "Open file":  open_file,
    "Settings":   lambda sh: show_settings(sh),
    "Quit":       lambda sh: sh.handle_key("\x11"),   # Ctrl+Q
})
sh.assign("menu", menu)
```

**`get_value()`:** returns the currently highlighted label (not the last invoked one).

**`last_activated`:** read-only property — label most recently invoked, or `None` before any item has been activated.

**Keys:** same navigation as `MenuReturn`.

---

## TextBox

> **Portable** — part of the [Portable Standard Library](../renderer-spec/portable-library.md).

A multi-line (or single-line) text input with cursor navigation and optional word wrap.

```python
TextBox(
    initial: str = "",
    wrap: Literal["word", "anywhere", "extend"] = "word",
    readonly: bool = False,
    enter_mode: Literal["newline", "submit", "ignore"] = "newline",
)
```

**Wrap modes:**

- `"word"` — wrap at word boundaries (default)
- `"anywhere"` — wrap mid-word when the line is full
- `"extend"` — no wrap; newlines only on Enter (best for single-line inputs)

**Enter modes:**

- `"newline"` (default) — Enter inserts a newline character
- `"submit"` — Enter fires `signal_return()` with the current text; does not insert a newline
- `"ignore"` — Enter is silently discarded

```python
# Single-line entry that submits on Enter
entry = TextBox(wrap="extend", enter_mode="submit")
sh.assign("entry", entry)
result = sh.run()   # returns the typed text when Enter is pressed

# Read-only display
display = TextBox(initial="some text", readonly=True)
sh.assign("display", display)
```

**Keys:** printable characters to type; `Backspace`/`Delete` to erase; `←`/`→` to move cursor; `Home`/`End` to jump to buffer start/end.

**`signal_return()`:** fires with `(True, text)` when Enter is pressed in `enter_mode="submit"`. Resets after the first call — fires exactly once per press.

---

## ListView

> **Frequently implemented** — follows the [portable spec's recommended API](../renderer-spec/portable-library.md#listview-interaction).

A display-only scrollable list. Not focusable — used to show read-only content updated programmatically.

```python
ListView(items: list[str])
```

```python
from panelmark_tui.interactions import ListView

log = ListView(["Starting...", "Loading config...", "Ready"])
sh.assign("log", log)

# Append a line later:
current = sh.get("log")
sh.update("log", current + ["New log entry"])
```

**Value:** `list[str]` — the current list of display lines.

---

## TreeView

> **Frequently implemented** — follows the [portable spec's recommended API](../renderer-spec/portable-library.md#treeview-interaction).

An interactive, keyboard-navigable tree with expand/collapse support.

```python
TreeView(tree: dict, *, initially_expanded: bool = False)
```

`tree` is a nested dict where `None` values are leaves and dict values are branches:

```python
from panelmark_tui.interactions import TreeView

tree = TreeView({
    'Documents': {
        'report.pdf': None,
        'notes.txt':  None,
    },
    'Pictures': {
        'photo.jpg': None,
    },
    'README.md': None,
})
sh.assign('tree', tree)
result = sh.run()   # e.g. ('Documents', 'report.pdf')
```

**Keys:**

| Key | Action |
|-----|--------|
| `↑` / `k` | Move cursor up |
| `↓` / `j` | Move cursor down |
| `Page Up` / `Page Down` | Jump one page |
| `Home` / `End` | Jump to first/last item |
| `Enter` / `Space` on branch | Toggle expand/collapse |
| `Enter` / `Space` on leaf | Select — shell exits with path tuple |

**Value:** `tuple[str, ...]` — path of the currently highlighted item, e.g. `('Documents', 'report.pdf')`. Returns `None` if the tree is empty.

---

## NestedMenu

> **Portable** — part of the [Portable Standard Library](../renderer-spec/portable-library.md).

A hierarchical drill-down menu. Selecting a branch item descends into it; selecting a leaf item returns its mapped value and exits the shell.

```python
NestedMenu(items: dict)
```

Non-dict values are leaf return values; dict values are branches. Use `Leaf(value)` to wrap a dict as a leaf payload.

```python
from panelmark_tui.interactions import NestedMenu, Leaf

menu = NestedMenu({
    "File": {
        "New":  "file:new",
        "Open": "file:open",
        "Save": "file:save",
    },
    "Edit": {
        "Cut":   "edit:cut",
        "Copy":  "edit:copy",
        "Paste": "edit:paste",
    },
    "Open Recent": Leaf({"command": "open_recent"}),
    "Quit": "quit",
})
sh.assign("menu", menu)
result = sh.run()   # e.g. "file:save" or "quit"
```

**Keys:**

| Key | Action |
|-----|--------|
| `↑` / `k`, `↓` / `j` | Navigate |
| `Page Up` / `Page Down`, `Home` / `End` | Jump |
| `Enter` / `Space` on branch | Descend into submenu |
| `Enter` / `Space` on leaf | Accept — shell exits with mapped value |
| `←` / `h` | Go back to parent level |

**Rendering:** branch items show a `▶` suffix. Inside a submenu a breadcrumb header (`← Parent › Child`) is shown at the top of the region.

**Malformed input:** `ValueError` is raised at construction for empty root, empty branch, duplicate sibling labels, or `None` as a leaf payload.

---

## CheckBox

> **Portable** — part of the [Portable Standard Library](../renderer-spec/portable-library.md).

A scrollable checkbox list supporting multi-select and single-select modes.

```python
CheckBox(items: dict[str, bool], mode: str = "multi")
```

`items` maps labels to initial checked states.

```python
options = CheckBox({
    "Enable logging":   True,
    "Dark mode":        False,
    "Auto-save":        True,
})
sh.assign("options", options)
result = sh.get("options")   # dict[str, bool]
```

In **single mode** (`mode="single"`), checking a new item automatically unchecks the previous one. Checked items show `(●)`, unchecked `( )`.

**Keys:** `↑`/`↓` or `k`/`j` to navigate; `Page Up`/`Page Down`/`Home`/`End` to jump; `Space` or `Enter` to toggle.

**Value:** `dict[str, bool]` — label → checked state for all items.

> Prefer `RadioList` for single-select when items map to distinct return values. `CheckBox(mode="single")` is appropriate only when the input and output are both checked-state dicts.

---

## RadioList

> **Portable** — part of the [Portable Standard Library](../renderer-spec/portable-library.md).

A single-select list with radio-button visuals (`(●)` / `( )`). The cursor position is the selection — moving the cursor immediately changes which item shows `(●)`.

```python
RadioList(items: dict)
```

`items` maps display labels to return values.

```python
from panelmark_tui.interactions import RadioList

sh.assign("size", RadioList({
    "Small":  "s",
    "Medium": "m",
    "Large":  "l",
}))
result = sh.run()   # returns "s", "m", or "l"
```

**Keys:** `↑`/`↓` or `k`/`j` to navigate; `Page Up`/`Page Down`/`Home`/`End` to jump; `Enter` or `Space` to accept.

**`get_value()`:** the value of the currently selected item.

**`signal_return()`:** `(True, value)` on accept.

---

## TableView

> **Frequently implemented** — follows the [portable spec's recommended API](../renderer-spec/portable-library.md#tableview-interaction).

A multi-column read-only display table with a sticky header row.

```python
TableView(columns: list, rows: list)
```

- `columns` — `[(header_label, width_in_chars), ...]`
- `rows` — list of rows; each row is a list of values (converted via `str()`)

```python
from panelmark_tui.interactions import TableView

sh.assign("results", TableView(
    columns=[("Name", 20), ("Status", 10), ("Score", 6)],
    rows=[
        ["Alice",   "active", "95"],
        ["Bob",     "idle",   "72"],
        ["Charlie", "active", "88"],
    ],
))
```

**Keys:** `↑`/`↓` or `k`/`j` to move active row; `Page Up`/`Page Down`/`Home`/`End` to jump.

**`get_value()`:** 0-based active row index.

---

## FormInput

> **Portable** — part of the [Portable Standard Library](../renderer-spec/portable-library.md).

A structured data-entry form with typed fields, validation, and a Submit button.

```python
FormInput(fields: dict)
```

```python
form = FormInput({
    "name": {
        "type":        "str",
        "descriptor":  "Your name",
        "required":    True,
        "validator":   lambda v: True if len(v) >= 2 else "Too short",
    },
    "age": {
        "type":       "int",
        "descriptor": "Age",
        "required":   True,
    },
    "role": {
        "type":    "choices",
        "descriptor": "Role",
        "options": ["Admin", "User", "Guest"],
        "default": "User",
    },
    "active": {
        "type":       "bool",
        "descriptor": "Active",
        "default":    True,
    },
})
sh.assign("form", form)
result = sh.run()   # dict with field values, or None on cancel
```

**Portable field types:** `"str"`, `"int"`, `"float"`, `"bool"`, `"choices"`

**Keys:** `↑`/`↓` to move between fields; field-specific keys for editing; `Enter` on Submit to confirm.

**Value:** `dict[str, value]` on successful submit; `None` on cancel.

---

## DataclassFormInteraction

> **Portable** — part of the [Portable Standard Library](../renderer-spec/portable-library.md).

A structured form interaction driven by a Python dataclass instance. Use this when embedding a form in a larger shell alongside other regions. For a standalone popup, use [`DataclassForm`](widgets.md#dataclassform) instead.

```python
DataclassFormInteraction(
    dataclass_instance,
    actions: list | None = None,
    on_change: callable | None = None,
)
```

`actions` is an optional list of action dicts, each with `"label"`, `"shortcut"`, and `"action"` keys.

```python
import dataclasses
from panelmark_tui.interactions import DataclassFormInteraction

@dataclasses.dataclass
class Config:
    host:  str  = "localhost"
    port:  int  = 8080
    debug: bool = False

interaction = DataclassFormInteraction(
    Config(),
    actions=[
        {"label": "Save",   "shortcut": "s", "action": lambda vals: vals},
        {"label": "Cancel", "shortcut": "q", "action": lambda vals: None},
    ],
)
sh.assign("config", interaction)
result = sh.run()
```

**`signal_return()`:** `(True, action_result)` when an action fires; `(False, None)` otherwise.

---

## StatusMessage

> **Portable** — part of the [Portable Standard Library](../renderer-spec/portable-library.md).

A display-only single-line status and feedback area. Not focusable.

```python
StatusMessage()
```

Updated programmatically via `shell.update(name, value)`:

```python
sh.update("status", ("error",   "File not found"))   # red  ✗ message
sh.update("status", ("success", "Saved"))             # green ✓ message
sh.update("status", ("info",    "3 items selected"))  # default ℹ message
sh.update("status", None)                             # clear
```

A plain `str` is treated as `("info", str)`.

---

## Function

**Escape hatch.** Use only when no built-in interaction fits and you need full control over drawing and key handling. Does not follow the standard `get_value()` / `set_value()` / `signal_return()` contract.

```python
Function(handler: Callable)
```

Handler signature: `handler(shell, context, key)` — returns `list[DrawCommand]` on render calls (`key` is `None`), or `None` on key events.

```python
from panelmark_tui.interactions import Function
from panelmark.draw import WriteCmd

def clock_handler(shell, context, key):
    if key is None:   # render
        import datetime
        now = datetime.datetime.now().strftime("%H:%M:%S")
        return [WriteCmd(row=0, col=0, text=now.ljust(context.width))]

sh.assign("clock", Function(clock_handler))
```

If you find yourself re-implementing navigation, selection, or text editing inside a `Function` handler, a custom `Interaction` subclass is the right path.

---

## Scrolling behaviour

`MenuReturn`, `MenuFunction`, and `CheckBox` inherit from `_ScrollableList`, which provides automatic scroll tracking.

| Key | Action |
|-----|--------|
| `↑` / `k` | Move up one item |
| `↓` / `j` | Move down one item |
| `Page Up` | Jump up by one full page |
| `Page Down` | Jump down by one full page |
| `Home` | Jump to the first item |
| `End` | Jump to the last item |

The scroll offset is kept in sync automatically. The viewport always contains the active item and is updated on every `render()` call, so the interaction correctly tracks the viewport even after a terminal resize.

## See also

- [Widgets](widgets.md)
- [Getting Started](getting-started.md)
- [Portable Library](../renderer-spec/portable-library.md)
- [Shell Language Examples](../shell-language/examples.md)
