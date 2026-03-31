# panelmark-tui

**panelmark-tui** is the blessed-powered terminal renderer for panelmark. It turns a panelmark layout definition and a set of interaction assignments into a live, keyboard-driven TUI application.

## What it provides

| Component | Description |
|-----------|-------------|
| `Shell` | Fullscreen terminal event loop (`run()`) and modal overlay loop (`run_modal()`) |
| 8 portable interactions | `MenuReturn`, `NestedMenu`, `RadioList`, `CheckBox`, `TextBox`, `FormInput`, `DataclassFormInteraction`, `StatusMessage` — constructor and value semantics are standardized across compliant renderers |
| 5 TUI-only interactions | `MenuFunction`, `ListView`, `TreeView`, `TableView`, `Function` — renderer-specific, not guaranteed portable |
| 6 portable widgets | `Confirm`, `Alert`, `InputPrompt`, `ListSelect`, `FilePicker`, `DataclassForm` — standardized across compliant renderers |
| 4 TUI-only widgets | `DatePicker`, `Progress`, `Toast`, `Spinner` — renderer-specific additions |
| Testing utilities | `MockTerminal`, `make_key` — for test suites that don't need a real terminal |

`panelmark-tui` is **`portable-library-compatible`**: it provides TUI-specific implementations of the portable interaction and widget contracts. Portable does not mean renderer-neutral code — it means constructor and semantic compatibility. See [Renderer Implementation](../panelmark-tui/renderer-implementation.md) for the precise framing.

## What it does not include

- Web or desktop rendering
- Browser event handling or WebSocket transport

## Who should use it

Application authors building terminal UIs with panelmark.

## Current status

`panelmark-tui` is the most mature renderer in the ecosystem.

| Feature | Status |
|---------|--------|
| Fullscreen event loop (`Shell.run()`) | Fully working |
| Modal overlay (`Shell.run_modal()`) | Fully working |
| Tab / Shift+Tab focus movement | Fully working |
| `MenuReturn`, `MenuFunction` | Working — up/down/j/k/Enter/Page Up/Page Down/Home/End |
| `TextBox`, `ListView`, `CheckBox`, `FormInput`, `StatusMessage` | Working |
| `TreeView` | Interactive collapsible tree — expand/collapse, full keyboard navigation |
| `RadioList` | Single-select with `(●)` / `( )` visuals; returns value on Enter/Space |
| `TableView` | Multi-column display table; sticky header; scrollable |
| `DataclassFormInteraction`, `DataclassForm` | Dataclass-driven form and modal widget |
| Standard modal widgets (`Confirm`, `Alert`, `InputPrompt`, `ListSelect`, `FilePicker`, `DatePicker`) | Working |
| `Progress` | Context-manager progress bar; renderer-managed update cycle |
| `Toast` | Transient overlay notification; auto-dismisses after timeout or keypress |
| `Spinner` | Indeterminate-progress popup; animated braille frames; cancellable |
| Panel headings (`__text__` syntax) | Rendered as `├─── Heading ───┤` at top of panel content area |

See [Limitations](../panelmark-tui/limitations.md) for known issues.

## Installation

```
pip install panelmark-tui
```

Dependencies: `panelmark`, `blessed`

## Quick start

```python
from panelmark_tui import Shell
from panelmark_tui.interactions import MenuReturn, StatusMessage

LAYOUT = """
|=== <bold>Pick a colour</> ===|
|{10R $menu$                   }|
|------------------------------|
|{2R  $status$                 }|
|==============================|
"""

sh = Shell(LAYOUT)
sh.assign("menu", MenuReturn({
    "Red":   "red",
    "Green": "green",
    "Blue":  "blue",
}))
sh.assign("status", StatusMessage())
sh.update("status", ("info", "Arrow keys to navigate, Enter to select"))

result = sh.run()
print(f"You chose: {result}")
```

## Using widgets

Most widgets follow the modal pattern:

```python
from panelmark_tui.widgets import Confirm, Alert

confirmed = Confirm(
    title="Delete file",
    message_lines=["Are you sure?", "This cannot be undone."],
).show(parent_shell=sh)

if confirmed:
    do_delete()
    Alert(title="Done", message_lines=["File deleted."]).show(parent_shell=sh)
```

`Progress` and `Spinner` use a context-manager pattern:

```python
from panelmark_tui.widgets import Progress

with Progress(title="Processing", total=len(items)).show(sh) as prog:
    for i, item in enumerate(items, 1):
        process(item)
        prog.set_progress(i)
        if prog.cancelled:
            break
```

## Detailed docs

- [Getting Started](../panelmark-tui/getting-started.md)
- [Interactions](../panelmark-tui/interactions.md)
- [Widgets](../panelmark-tui/widgets.md)
- [Renderer Implementation](../panelmark-tui/renderer-implementation.md)
- [Limitations](../panelmark-tui/limitations.md)

## See also

- [panelmark core](panelmark.md)
- [Renderer Spec](../renderer-spec/overview.md)
- [Portable Library](../renderer-spec/portable-library.md)
- [Ecosystem](../ecosystem.md)
