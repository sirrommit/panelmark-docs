# Shell Language Syntax

## Structure

Every shell definition is a block of lines. Each line must be enclosed in outer pipe characters `|...|`. Lines that are empty or contain only whitespace (no `|...|`) are ignored and do not affect layout geometry.

Two comment forms are supported:

- `# line comment` — everything from `#` to the end of the line is removed before parsing
- `/* block comment */` — C-style block comments may span multiple lines and can appear inline inside a shell row

```
|=== Title ===|         ← horizontal border (double-line style, with title)
|{$left$  }|{$right$}|  ← content rows with a vertical split
|-------------|         ← horizontal border (single-line style, no title)
|{$footer$   }|         ← content row spanning full width
|=============|         ← closing border
```

The parser reads the inner content of each line (everything between the outer `|` walls) and builds a layout tree recursively.

---

## Horizontal splits

A **horizontal split** (top/bottom) is produced by a **border row** — a line whose content starts with `=` or `-` and contains no `{` blocks.

```
|================|     ← double-line style  (= characters)
|----------------|     ← single-line style  (- characters)
```

The first such row found splits the definition into a `top` block (all lines above it) and a `bottom` block (all lines below it). The border is rendered as a full-width horizontal rule by every compliant renderer. Renderers access border data via `shell.borders`, which returns a list of `BorderSpec` objects — one per internal separator — each carrying its row, column, width, style, and optional title.

**Optional title** — text anywhere in the border row (between the fill characters) becomes the border's title, rendered centred:

```
|=== My Section ===|   ← "My Section" displayed centred in the border
|--- Details ------|   ← "Details" displayed centred in a single-line border
```

**Style markup in titles** — titles support a limited inline style syntax:

```
|=== <bold>Important</> ===|
```

Supported tags: `<bold>`, `<italic>`, `<underline>`, `<reverse>`, `<red>`, `<green>`, `<yellow>`, `<blue>`, `<magenta>`, `<cyan>`. Close with `</>`.

---

## Vertical splits

A **vertical split** (left/right) is produced by a structural column divider that appears **outside** any `{...}` block and is consistent across **all** content rows.

```
|{$left$  }|{$right$}|
```

The inner `|` between the two `{...}` blocks is the structural divider. It must appear in exactly the same structural position in every content row of the block.

- `|` produces a **single-line** vertical divider (`│`)
- `||` produces a **double-line** vertical divider (`║`)

Splits can be nested to any depth. The parser resolves them recursively:

```
|{$a$}|{$b$}|{$c$}|    ← two structural | chars → two vertical splits
```

---

## Content blocks

A **content row** is a line whose inner content is a single `{...}` block (after the outer border pipes are removed). Everything inside `{...}` is the panel specification.

```
|{specifiers $name$ }|
```

A panel can span multiple definition rows — each additional row adds another `1R` to its implicit height. Specifiers are only read from the **first** definition row of a panel.

### Region name

A **region name** identifies the panel for interaction assignment. Use `$name$` syntax. Names must match `[a-z0-9_]+` and must be unique across the shell.

```
|{$my_region$  }|
```

If omitted, the panel exists in the layout but cannot have an interaction assigned.

### Fixed width

A leading integer sets a fixed character width for the panel:

```
|{20 $sidebar$}|{$main$   }|
```

`$sidebar$` is always 20 characters wide. `$main$` takes the remaining space.

### Percentage width

A leading integer followed by `%` sets a proportional width:

```
|{30% $sidebar$}|{70% $main$}|
```

Percentages are computed relative to the available content width (terminal width minus the two outer border walls and any internal dividers).

> At least one panel in a VSplit must be either fixed or percentage; the other may be fill (no width spec). If all panels in a split are fill, they share equally.

### Fixed row count

`NR` inside the block sets the panel to exactly N rows tall:

```
|{10R $list$  }|
|{3R  $status$}|
```

### Percentage row count

`N%R` sets a proportional height:

```
|{50%R $top$   }|
|{50%R $bottom$}|
```

### Heading

`__text__` inside the block attaches a heading string to the panel. The string is stored in `Region.heading` and passed to the renderer, which displays it at the top of the panel's content area. The visual form depends on the renderer:

- **panelmark-tui** — draws a `├─── Heading ───┤` sub-border on the first row of the region, consuming one row of the panel's height.
- **panelmark-html** — emits a `<header class="pm-panel-heading">` element above the panel body.

```
|{__Navigation__ $sidebar$}|
```

### Equal fill columns

When all columns in a vertical split group are fill-width (no fixed character count or percentage specified), the available content space is divided **equally** among them. Any remainder that cannot be divided evenly falls to the rightmost columns, so columns differ in width by at most one character.

```
|{$a$}|{$b$}|{$c$}|    ← three fill columns share space equally
```

### Implicit height

If no row count specifier is given, the panel's height equals the number of `{...}` content rows that make up its slot in the layout. Each additional content row (even one with no specifiers or name) expands the panel's height by one row.

```
|{$panel$   }|   ← 1 content row → height 1
|{          }|   ← 2nd content row for the same panel → height 2
|{          }|   ← 3rd content row → height 3
```

Note: lines that are entirely empty (no `|...|`) are stripped before parsing and never count toward implicit height.

---

## Parser rules summary

| Syntax | Effect |
|--------|--------|
| `\|...\|` | Outer border — required on every line |
| `=...=` or `---` | Horizontal split border (no `{` on that line) |
| `=== Title ===` | Horizontal border with centred title |
| `<bold>text</>` | Style markup in border titles |
| `\|` between blocks | Single-line vertical split divider |
| `\|\|` between blocks | Double-line vertical split divider |
| `{...}` | Content block specifying a panel |
| `$name$` | Region name (inside `{...}`) |
| `20` at start | Fixed 20-char width (inside `{...}`) |
| `30%` at start | Percentage width (inside `{...}`) |
| `5R` | Fixed 5-row height (inside `{...}`) |
| `50%R` | Percentage height (inside `{...}`) |
| `__text__` | Panel heading (inside `{...}`) |
| Empty lines (no `\|...\|`) | Ignored — do not affect layout geometry |
| `# comment` | Line comment — stripped to end of line before parsing |
| `/* comment */` | Block comment — may span lines; newlines preserved |

---

## See also

- [Shell Language Overview](overview.md)
- [Examples](examples.md)
- [Renderer Contract](../renderer-spec/contract.md) — how draw commands are produced and executed
