# panelmark-web Interaction Coverage

`panelmark-web` claims **`portable-library-compatible`** status. All required portable
interactions and widgets are implemented with one noted difference from the spec's
blocking-modal semantics — see [Web note](#web-note) below.

---

## Web note

The portable-library spec describes widgets as blocking: `widget.show(sh)` returns only
after the user acts. In `panelmark-web` the session is async — there is no call that
blocks a coroutine until a widget is dismissed.

**Web-appropriate pattern:** assign the widget to a panel region and rely on
`signal_return()` to fire when the user acts. The WebSocket session lifecycle delivers the
result automatically.

All constructor signatures, `get_value()` / `set_value()` / `signal_return()` semantics,
and return values match the portable spec exactly.

---

## Available interactions and widgets

```python
# Interactions
from panelmark_web.interactions import (
    StatusMessage, MenuReturn, RadioList, CheckBox,
    TextBox, NestedMenu, FormInput, DataclassFormInteraction,
    Leaf,
    MenuFunction, ListView, TableView,   # frequently implemented
)

# Widgets
from panelmark_web.widgets import (
    Alert, Confirm, InputPrompt, ListSelect,
    DataclassForm, FilePicker,
)
```

---

## Required interactions

| Interaction | Status | Module |
|-------------|--------|--------|
| `StatusMessage` | Implemented | `panelmark_web.interactions.StatusMessage` |
| `MenuReturn` | Implemented | `panelmark_web.interactions.MenuReturn` |
| `NestedMenu` | Implemented | `panelmark_web.interactions.NestedMenu` |
| `RadioList` | Implemented | `panelmark_web.interactions.RadioList` |
| `CheckBox` | Implemented | `panelmark_web.interactions.CheckBox` |
| `TextBox` | Implemented | `panelmark_web.interactions.TextBox` |
| `FormInput` | Implemented | `panelmark_web.interactions.FormInput` |
| `DataclassFormInteraction` | Implemented | `panelmark_web.interactions.DataclassFormInteraction` |

---

## Required widgets

| Widget | Status | Notes |
|--------|--------|-------|
| `Alert` | Implemented | `panelmark_web.widgets.Alert` |
| `Confirm` | Implemented | `panelmark_web.widgets.Confirm` |
| `InputPrompt` | Implemented | `panelmark_web.widgets.InputPrompt` |
| `ListSelect` | Implemented | `panelmark_web.widgets.ListSelect` |
| `DataclassForm` | Implemented | `panelmark_web.widgets.DataclassForm` |
| `FilePicker` | Implemented | `panelmark_web.widgets.FilePicker` — server-side filesystem browser |

---

## Frequently implemented (optional)

| Interaction / Widget | Status | Notes |
|----------------------|--------|-------|
| `MenuFunction` | Implemented | `panelmark_web.interactions.MenuFunction` |
| `ListView` | Implemented | `panelmark_web.interactions.ListView` |
| `TableView` | Implemented | `panelmark_web.interactions.TableView` |
| `TreeView` | Not implemented | — |
| `DatePicker` | Not implemented | — |
| `Progress` | Not implemented | — |
| `Spinner` | Not implemented | — |
| `Toast` | Not implemented | — |

---

## Draw-command renderer

| Draw command | Support |
|--------------|---------|
| `WriteCmd` | Full — text, position, style (bold, italic, underline, color, bg, reverse) |
| `FillCmd` | Full — rectangular fill with character and style |
| `CursorCmd` | Ignored — HTML renderers do not render a text cursor |

Any custom `Interaction` whose `render()` produces only `WriteCmd` and `FillCmd` commands
will work out of the box.

---

## See also

- [panelmark-web Overview](overview.md)
- [Portable Library spec](../renderer-spec/portable-library.md)
- [Readiness](../renderer-spec/readiness.md)
