# Shell Language Examples

## Complete shell definition

```python
SHELL = """
|=== <bold>File Manager</> ===|
|{30% 20R $tree$  }|{20R $files$       }|
|--- <bold>Filter</> ----------|
|{2R $filter$                 }|
|--- <bold>Filename</> --------|
|{2R $filename$               }|
|-----------------------------|
|{1R  $buttons$               }|
|=============================|
"""
```

This produces:

- A top double-line border titled "File Manager" (bold)
- A 20-row VSplit: `$tree$` gets 30% width, `$files$` takes the remaining 70%
- A single-line border titled "Filter" (bold)
- A 2-row `$filter$` region
- A single-line border titled "Filename" (bold)
- A 2-row `$filename$` region
- A plain single-line divider
- A 1-row `$buttons$` region
- A closing double-line border

## Using the shell

```python
from panelmark_tui import Shell
from panelmark_tui.interactions import MenuReturn, TextBox

SHELL = """
|=== File Manager ===|
|{30% $tree$}|{$files$}|
|------------|
|{2R $filter$}|
|=============|
"""

sh = Shell(SHELL)
sh.assign("tree",   MenuReturn({"Documents": "docs", "Downloads": "dl", "Desktop": "desk"}))
sh.assign("files",  MenuReturn({}))
sh.assign("filter", TextBox(wrap="extend", enter_mode="ignore"))

result = sh.run()
```

## Custom interaction: display label

A display-only label shows a single line of text. It is not focusable and ignores all keys.

```python
from panelmark import Interaction
from panelmark.draw import DrawCommand, RenderContext, WriteCmd, FillCmd


class Label(Interaction):
    """Display-only single-line text label."""

    def __init__(self, text: str = ""):
        self._text = text

    @property
    def is_focusable(self) -> bool:
        return False

    def render(self, context: RenderContext, focused: bool = False) -> list[DrawCommand]:
        line = self._text[:context.width].ljust(context.width)
        cmds = [WriteCmd(row=0, col=0, text=line)]
        if context.height > 1:
            cmds.append(FillCmd(row=1, col=0,
                                width=context.width, height=context.height - 1))
        return cmds

    def handle_key(self, key: str) -> tuple:
        return False, self.get_value()

    def get_value(self) -> str:
        return self._text

    def set_value(self, value) -> None:
        self._text = str(value)
```

## Custom interaction: toggle button

A toggle button cycles between on/off states and signals exit when confirmed.

```python
from panelmark import Interaction
from panelmark.draw import DrawCommand, RenderContext, WriteCmd


class Toggle(Interaction):
    """Simple on/off toggle. Press Enter to confirm and exit."""

    def __init__(self, initial: bool = False):
        self._value = initial
        self._confirmed = False

    def render(self, context: RenderContext, focused: bool = False) -> list[DrawCommand]:
        indicator = "[ON ]" if self._value else "[OFF]"
        text = indicator.ljust(context.width)
        style = {"reverse": True} if focused else None
        return [WriteCmd(row=0, col=0, text=text, style=style)]

    def handle_key(self, key: str) -> tuple:
        if key in (' ', 'KEY_ENTER', 'KEY_LEFT', 'KEY_RIGHT'):
            self._value = not self._value
            if key == 'KEY_ENTER':
                self._confirmed = True
            return True, self.get_value()
        return False, self.get_value()

    def get_value(self) -> bool:
        return self._value

    def set_value(self, value) -> None:
        self._value = bool(value)

    def signal_return(self) -> tuple:
        if self._confirmed:
            return True, self._value
        return False, None
```

## Accessing the shell from inside an interaction

When an interaction is assigned to a shell via `shell.assign()`, the shell sets `interaction._shell = self`. You can call any public shell method from inside an interaction — for example to update another region or read its value:

```python
def handle_key(self, key: str) -> tuple:
    if key == 'KEY_ENTER':
        entry = self._shell.get('entry')
        self._shell.update('status', ('success', f'Saved: {entry}'))
        return True, self.get_value()
    return False, self.get_value()
```

## Using capabilities for portable rendering

Use `context.supports()` to provide fallbacks on renderers that lack a feature:

```python
def render(self, context: RenderContext, focused: bool = False) -> list[DrawCommand]:
    cmds = []
    for i, item in enumerate(self._items):
        text = item[:context.width].ljust(context.width)
        if i == self._active and focused:
            if context.supports('color'):
                style = {"bg": "blue", "color": "white"}
            else:
                style = {"reverse": True}   # mono fallback
        else:
            style = None
        cmds.append(WriteCmd(row=i, col=0, text=text, style=style))
    return cmds
```

## Testing interactions

Because `render()` returns a plain list of data objects, interactions are easy to unit test without any terminal dependency:

```python
from panelmark.draw import RenderContext, WriteCmd, FillCmd

def test_label_renders_text():
    label = Label("Hello, world")
    ctx = RenderContext(width=20, height=1)
    cmds = label.render(ctx, focused=False)
    assert isinstance(cmds[0], WriteCmd)
    assert "Hello, world" in cmds[0].text

def test_label_clips_long_text():
    label = Label("A" * 100)
    ctx = RenderContext(width=10, height=1)
    cmds = label.render(ctx, focused=False)
    assert len(cmds[0].text) == 10
```

## See also

- [Shell Language Overview](overview.md)
- [Syntax](syntax.md)
- [Renderer Contract](../renderer-spec/contract.md) — full draw command reference
- [Portable Library](../renderer-spec/portable-library.md) — built-in interactions and widgets
