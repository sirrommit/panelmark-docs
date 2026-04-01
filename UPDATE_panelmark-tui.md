# UPDATE_panelmark-tui.md

This file gives instructions for updating the `panelmark-tui` documentation in
the docs site after `panelmark-tui/PORTABLE_TODO.md` has been completed.

Assumption for this document:

- the portable semantic mismatches have already been fixed in code
- `panelmark-tui` can now cleanly claim `portable-library-compatible` status


## Goal

Clarify the `panelmark-tui` renderer documentation so it communicates the right
distinction:

- portable semantic contract
- renderer-specific implementation shape

The docs should make clear that `panelmark-tui` provides TUI-specific
implementations of portable interactions/widgets, and that this is the intended
architecture rather than a contradiction.


## Primary source files to update or migrate from

Use these files as source material:

- [renderer-implementation.md](/home/sirrommit/claude_play/panelmark-tui/docs/renderer-implementation.md)
- [interactions.md](/home/sirrommit/claude_play/panelmark-tui/docs/interactions.md)
- [widgets.md](/home/sirrommit/claude_play/panelmark-tui/docs/widgets.md)
- [portable-library.md](/home/sirrommit/claude_play/panelmark/docs/renderer-spec/portable-library.md)

Destination in the docs site should be the `panelmark-tui` package pages under:

- `/home/sirrommit/claude_play/panelmark-docs/docs/`


## Main clarification to make

The old confusion was:

- top-level docs said `panelmark-tui` was `portable-library-compatible`
- deeper text said all built-in interactions/widgets were renderer-specific and
  therefore “not portable”

After the code fixes, the docs should say this instead:

- `panelmark-tui` is `portable-library-compatible`
- it achieves this by providing TUI-specific implementations of the portable
  interactions/widgets
- those implementations are renderer-specific in shape and runtime behavior
  details
- they are still portable in the semantic/API sense because they conform to the
  portable-library contract


## Required documentation changes

### 1. Update the renderer-implementation explanation

Wherever the docs site presents the renderer implementation details for
`panelmark-tui`, make sure it says:

- `panelmark-tui` claims `portable-library-compatible`
- required portable interactions/widgets are implemented
- portable does **not** mean renderer-neutral code
- portable means constructor/semantic compatibility across renderers

Recommended wording pattern:

- “`panelmark-tui` provides blessed/TUI implementations of the portable
  interaction and widget contracts.”

Do not use wording like:

- “all built-ins are not portable”

That phrasing should be reserved only for true extras outside the portable
library.


### 2. Split built-ins into two categories in docs

For both interactions and widgets, the docs should distinguish:

#### Portable items implemented by panelmark-tui

Required interactions:

- `MenuReturn`
- `NestedMenu`
- `RadioList`
- `CheckBox`
- `TextBox`
- `FormInput`
- `DataclassFormInteraction`
- `StatusMessage`

Required widgets:

- `Alert`
- `Confirm`
- `InputPrompt`
- `ListSelect`
- `FilePicker`
- `DataclassForm`

These should be described as:

- portable in semantic contract
- implemented in a TUI-specific way

#### TUI-only additions

Examples:

- `Function`
- `MenuFunction`
- `ListView`
- `TreeView`
- `TableView`
- `DatePicker`
- `Progress`
- `Toast`
- `Spinner`

These should be described as:

- renderer-specific additions
- not guaranteed portable unless/until standardized


### 3. Clarify the meaning of portability

Add or strengthen one short explanation in the docs site:

- shape is renderer-specific
- semantics are portable

Recommended concept paragraph:

- “A portable interaction/widget does not have to look the same in every
  renderer. It is portable when constructor shape, current-state semantics,
  submit semantics, and return values match the portable-library spec.”


### 4. Keep TUI extensions visible but properly labeled

If `panelmark-tui` supports extra parameters or action features beyond the
portable spec, document them as:

- `panelmark-tui` extensions
- optional
- not required for cross-renderer portability

Examples that may need this framing:

- TUI-only action options
- layout-width parameters
- modal presentation helpers


## Suggested docs-site placement

Apply this clarification in:

- the `panelmark-tui` package overview page
- the `panelmark-tui` renderer-implementation page
- the detailed interactions page
- the detailed widgets page

The package overview page should give the short version.
The renderer-implementation page should give the precise compatibility framing.
The interactions/widgets pages should label individual items correctly.


## Verification checklist

Do not consider this docs-site update done until all of the following are true:

- [x] `panelmark-tui` is described as `portable-library-compatible`
- [x] portable built-ins are described as portable in semantic/API terms
- [x] TUI-only extras are clearly labeled as renderer-specific
- [x] no page says or implies that all built-ins are non-portable
- [x] no page implies portability requires identical visual shape across renderers
- [x] the package overview and renderer-implementation pages agree with each other


## Final rule

The docs should communicate this clearly:

`panelmark-tui` is portable-library-compatible because it implements the
portable contracts, even though the concrete classes are TUI-specific
implementations.
