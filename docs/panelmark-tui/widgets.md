# panelmark-tui Widgets

panelmark-tui ships 10 built-in widgets, importable from `panelmark_tui.widgets`.

```python
from panelmark_tui.widgets import (
    Confirm, Alert, InputPrompt,
    ListSelect, FilePicker, DatePicker,
    DataclassForm,
    Progress, Toast, Spinner,
)
```

Widgets marked **Portable** implement the [Portable Library](../renderer-spec/portable-library.md) spec. Widgets marked **Frequently implemented** follow the portable spec's recommended API but are not required for `portable-library-compatible` status.

## Widget families

### Standard modal dialogs

`Confirm`, `Alert`, `InputPrompt`, `ListSelect`, `FilePicker`, `DatePicker`, `DataclassForm`

These are shell-composed widgets. Each builds a small `Shell` internally and runs it as a modal overlay. They block until the user makes a choice.

```python
result = Widget(options...).show(parent_shell=sh)
```

All modal dialog widgets:

- Are auto-centred on screen by default
- Restore the parent shell's display after closing
- Return `None` on Escape or Ctrl+Q (unless noted otherwise)
- Accept `parent_shell=sh` to integrate with a running TUI

### Renderer-managed utility overlays

`Progress`, `Toast`, `Spinner`

These widgets manage their own render cycle. `Toast` displays briefly then removes itself; `Progress` and `Spinner` use a context-manager pattern where the caller drives updates synchronously.

```python
with Progress(title="...", total=n).show(sh) as prog:
    ...
```

These always return `None` — cancellation is checked via `.cancelled`.

---

## Confirm

> **Portable** — part of the [Portable Standard Library](../renderer-spec/portable-library.md).

Asks the user to confirm or deny an action.

```python
Confirm(
    title: str = "Confirm",
    message_lines: list[str] = [],
    buttons: dict = {"OK": True, "Cancel": False},
    width: int = 40,
)
```

```python
result = Confirm(
    title="Delete account",
    message_lines=[
        "This will permanently delete your account.",
        "All data will be lost.",
    ],
    buttons={"Delete": "delete", "Keep": "keep", "Cancel": None},
).show(parent_shell=sh)

if result == "delete":
    do_delete()
```

**Returns:** the dict value for the selected button; `None` on Escape/Ctrl+Q.

---

## Alert

> **Portable** — part of the [Portable Standard Library](../renderer-spec/portable-library.md).

Informational popup with a single OK button. Blocks until dismissed.

```python
Alert(
    title: str = "Alert",
    message_lines: list[str] = [],
    width: int = 40,
)
```

```python
Alert(
    title="Error",
    message_lines=["Could not read file:", "  /etc/myconfig.conf"],
).show(parent_shell=sh)
```

**Returns:** `True` when OK is pressed; `None` on Escape/Ctrl+Q.

---

## InputPrompt

> **Portable** — part of the [Portable Standard Library](../renderer-spec/portable-library.md).

Asks the user to type a single line of text.

```python
InputPrompt(
    title: str = "Input",
    prompt_lines: list[str] = [],
    initial: str = "",
    width: int = 50,
)
```

```python
name = InputPrompt(
    title="Rename",
    prompt_lines=["Enter a new name for this item:"],
    initial="old_name.txt",
).show(parent_shell=sh)

if name is not None:
    rename(name)
```

**Returns:** the entered text string on OK or Enter (may be `""`); `None` on Cancel/Escape/Ctrl+Q.

Focus opens on the text entry box. Enter in the entry box submits immediately. Tab moves focus to the button row.

---

## ListSelect

> **Portable** — part of the [Portable Standard Library](../renderer-spec/portable-library.md).

Lets the user pick one item (single mode) or multiple items (multi mode) from a scrollable list.

```python
ListSelect(
    title: str = "Select",
    prompt_lines: list[str] = [],
    items: list | dict = [],
    multi: bool = False,
    width: int = 40,
)
```

**Single mode** — selecting an item immediately closes the dialog:

```python
colour = ListSelect(
    title="Pick colour",
    items=["Red", "Green", "Blue"],
).show(parent_shell=sh)

# Pass a dict to return values other than the label:
colour = ListSelect(
    items={"Red": "#FF0000", "Green": "#00FF00", "Blue": "#0000FF"},
).show(parent_shell=sh)
```

**Multi mode** — shows checkboxes; user confirms with OK:

```python
tags = ListSelect(
    title="Add tags",
    items={"urgent": True, "bug": False, "feature": False},
    multi=True,
).show(parent_shell=sh)
```

Pass a `dict[str, bool]` to set initial checked states, or a `list` to start all unchecked.

**Single mode returns:** selected label or mapped value; `None` on cancel.

**Multi mode returns:** `dict[str, bool]` of all items on OK; `None` on cancel.

---

## FilePicker

> **Portable** — part of the [Portable Standard Library](../renderer-spec/portable-library.md).

Browse the filesystem and select a file or directory.

```python
FilePicker(
    start_dir: str | None = None,   # defaults to os.getcwd()
    title: str = "Select File",
    dirs_only: bool = False,
    filter: str = "*",
    width: int = 70,
)
```

```python
path = FilePicker(
    start_dir="/home/user/projects",
    title="Open project",
    filter="*.py",
).show(parent_shell=sh)

# Directory picker mode:
dest = FilePicker(title="Choose destination", dirs_only=True).show(parent_shell=sh)
```

**Returns:** absolute path string on OK; `None` on Cancel/Escape/Ctrl+Q.

**Layout:** two-column panel (directory tree left, file list right), filter field (glob pattern), filename field, status bar, OK/Cancel buttons.

Pressing Enter on a directory navigates into it. The file list is filtered live by the glob pattern. Selecting a file copies its path into the filename field, which can also be typed directly.

See [Limitations](limitations.md#filepicker) for known behaviour notes.

---

## DatePicker

> **Frequently implemented** — follows the [portable spec's recommended API](../renderer-spec/portable-library.md#datepicker-widget).

Presents a monthly calendar for date selection.

```python
DatePicker(
    initial: datetime.date | None = None,   # defaults to today
    title: str = "Select Date",
    width: int = 30,
)
```

```python
import datetime

due_date = DatePicker(
    title="Set due date",
    initial=datetime.date.today() + datetime.timedelta(days=7),
).show(parent_shell=sh)
```

**Returns:** `datetime.date` on OK; `None` on Cancel/Escape/Ctrl+Q.

**Keys:** `↑`/`↓`/`←`/`→` to move one day/week; `←`/`→` in the nav row to change month; `Enter` to select.

---

## DataclassForm

> **Portable** — part of the [Portable Standard Library](../renderer-spec/portable-library.md).

Modal form widget driven by a dataclass instance. A thin wrapper around [`DataclassFormInteraction`](interactions.md#dataclassforminteraction) for use as a standalone popup.

```python
DataclassForm(
    dc_instance,
    title: str = "Edit",
    actions: list | None = None,
    on_change: callable | None = None,
    width: int = 60,
)
```

```python
import dataclasses
from panelmark_tui.widgets import DataclassForm

@dataclasses.dataclass
class Config:
    host: str = "localhost"
    port: int = 8080
    debug: bool = False

result = DataclassForm(
    Config(),
    title="Server Settings",
    actions=[
        {"label": "Save",   "shortcut": "s", "action": lambda vals: vals},
        {"label": "Cancel", "shortcut": "q", "action": lambda vals: None},
    ],
).show(parent_shell=sh)
```

**Returns:** the value returned by the triggered action callable; `None` if cancelled.

When you need the form embedded in a larger shell alongside other regions, assign `DataclassFormInteraction` to a region directly instead.

---

## Progress

> **Frequently implemented** — follows the [portable spec's recommended API](../renderer-spec/portable-library.md#progress-widget).

Displays a progress bar during a long operation. Driven programmatically from the caller's loop.

```python
Progress(
    title: str = "Progress",
    total: int = 100,
    cancellable: bool = True,
    width: int = 50,
)
```

```python
with Progress(title="Importing data", total=len(records)).show(sh) as prog:
    for i, record in enumerate(records, 1):
        process(record)
        prog.set_progress(i, f"Processing record {i}/{len(records)}")
        if prog.cancelled:
            break
```

**Handle API:**

| Method / Property | Description |
|-------------------|-------------|
| `prog.set_progress(n, message="")` | Advance bar to step `n`; optionally update message |
| `prog.cancelled` | `True` if the user clicked Cancel |

**Returns:** `None` always.

The bar renders as `[████████░░░░░░] 55%` using Unicode block characters. `set_progress()` pushes updates synchronously without waiting for a keypress.

---

## Toast

> **Frequently implemented** — follows the [portable spec's recommended API](../renderer-spec/portable-library.md#toast-widget).

Transient overlay notification that auto-dismisses after a configurable timeout or on any keypress.

```python
Toast(
    message: str,
    title: str = "Notice",
    duration: float = 2.0,
    width: int = 40,
)
```

```python
def handle_save(sh):
    save_file()
    Toast(message="File saved!", duration=1.5).show(parent_shell=sh)
```

**Returns:** `None` always. Dismisses after `duration` seconds or on the first keypress.

---

## Spinner

> **Frequently implemented** — follows the [portable spec's recommended API](../renderer-spec/portable-library.md#spinner-widget).

Indeterminate-progress popup for operations with no known total. Like `Progress` but shows an animated braille spinner instead of a fill bar.

```python
Spinner(
    title: str = "Working…",
    cancellable: bool = True,
    width: int = 50,
)
```

```python
with Spinner(title="Scanning…").show(parent_shell=sh) as spin:
    for path in paths:
        scan(path)
        spin.tick(f"Scanning {path}")
        if spin.cancelled:
            break
```

**Handle API:**

| Method / Property | Description |
|-------------------|-------------|
| `spin.tick(message="")` | Advance animation frame; optionally update status text |
| `spin.cancelled` | `True` if the user pressed Escape, Ctrl+Q, or Cancel |

**Returns:** `None` always. Animation uses braille frames (`⠋⠙⠹⠸⠼⠴⠦⠧⠇⠏`).

---

## Positioning

By default, modal dialog widgets are centred on screen. Pass explicit `row`, `col`, or `width` to `.show()` to override position:

```python
Alert(message_lines=["Done"]).show(
    parent_shell=sh,
    row=5,
    col=10,
    width=30,
)
```

## See also

- [Interactions](interactions.md)
- [Getting Started](getting-started.md)
- [Portable Library](../renderer-spec/portable-library.md)
- [Limitations](limitations.md)
