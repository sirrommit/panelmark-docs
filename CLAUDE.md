# CLAUDE.md — panelmark-docs

This file defines the source repositories and source documents that Claude
should use when implementing the `panelmark-docs` site.

The docs site lives in:

- `/home/sirrommit/claude_play/panelmark-docs`

The source material lives in sibling repositories/directories under:

- `/home/sirrommit/claude_play`


## Primary instruction

When building or updating `panelmark-docs`, Claude should treat
`panelmark-docs` as the destination and the following sibling repos as the
authoritative source material:

- `/home/sirrommit/claude_play/panelmark`
- `/home/sirrommit/claude_play/panelmark-tui`
- `/home/sirrommit/claude_play/panelmark-html`
- `/home/sirrommit/claude_play/panelmark-web`

Claude should not invent documentation architecture or product claims when the
source repositories already contain the relevant material.


## Destination repo

All docs-site work should be written into:

- `/home/sirrommit/claude_play/panelmark-docs`

Key destination files already present:

- `/home/sirrommit/claude_play/panelmark-docs/mkdocs.yml`
- `/home/sirrommit/claude_play/panelmark-docs/.readthedocs.yaml`
- `/home/sirrommit/claude_play/panelmark-docs/DOCUMENTATION_PLAN.md`
- `/home/sirrommit/claude_play/panelmark-docs/docs/`


## Source repos and exact documentation locations

Claude should pull docs from the following locations.

### 1. Core ecosystem docs — `panelmark`

Repo root:

- `/home/sirrommit/claude_play/panelmark`

Primary source files:

- `/home/sirrommit/claude_play/panelmark/README.md`
- `/home/sirrommit/claude_play/panelmark/ECOSYSTEM.md`

Renderer-spec source files:

- `/home/sirrommit/claude_play/panelmark/docs/renderer-spec/overview.md`
- `/home/sirrommit/claude_play/panelmark/docs/renderer-spec/contract.md`
- `/home/sirrommit/claude_play/panelmark/docs/renderer-spec/portable-library.md`
- `/home/sirrommit/claude_play/panelmark/docs/renderer-spec/extensions.md`
- `/home/sirrommit/claude_play/panelmark/docs/renderer-spec/readiness.md`

Other core docs to inspect as needed:

- `/home/sirrommit/claude_play/panelmark/docs/`

Important note:

- If `README.md` or `ECOSYSTEM.md` contradict newer `renderer-spec` docs 
  prefer the newer renderer-spec docs plus current code reality.


### 2. Terminal renderer docs — `panelmark-tui`

Repo root:

- `/home/sirrommit/claude_play/panelmark-tui`

Primary source files:

- `/home/sirrommit/claude_play/panelmark-tui/README.md`
- `/home/sirrommit/claude_play/panelmark-tui/CONTRIBUTING.md`
- `/home/sirrommit/claude_play/panelmark-tui/KNOWN_LIMITATIONS.md`

Renderer/package docs:

- `/home/sirrommit/claude_play/panelmark-tui/docs/getting-started.md`
- `/home/sirrommit/claude_play/panelmark-tui/docs/interactions.md`
- `/home/sirrommit/claude_play/panelmark-tui/docs/widgets.md`
- `/home/sirrommit/claude_play/panelmark-tui/docs/testing.md`
- `/home/sirrommit/claude_play/panelmark-tui/docs/renderer-implementation.md`

Use these for:

- package overview
- TUI getting-started docs
- interaction inventory
- widget inventory
- TUI renderer-spec implementation notes
- limitations/testing guidance


### 3. Static HTML renderer docs — `panelmark-html`

Repo root:

- `/home/sirrommit/claude_play/panelmark-html`

Primary source files:

- `/home/sirrommit/claude_play/panelmark-html/README.md`
- `/home/sirrommit/claude_play/PANELMARK_HTML_IMPLEMENTATION_PLAN.md`

HTML/hook docs:

- `/home/sirrommit/claude_play/panelmark-html/docs/hook-contract.md`

Use these for:

- `panelmark-html` overview page
- rendering API page
- hook-contract page
- static-vs-live boundary explanation

### 4. Live web runtime docs — `panelmark-web`

Repo root:

- `/home/sirrommit/claude_play/panelmark-web`

Primary source files:

- `/home/sirrommit/claude_play/panelmark-web/README.md`

Web docs:

- `/home/sirrommit/claude_play/panelmark-web/docs/getting-started.md`
- `/home/sirrommit/claude_play/panelmark-web/docs/hook-contract-web.md`
- `/home/sirrommit/claude_play/panelmark-web/docs/interaction-coverage.md`

Use these for:

- `panelmark-web` overview
- framework integration docs
- browser/runtime/protocol summary
- hook usage docs
- interaction coverage page


## Source of truth priority

When multiple files disagree, Claude should use this priority order:

1. Current implementation code in the relevant package
2. Current package docs that match implementation
3. Older README / ecosystem narrative docs

This matters because several packages have had documentation drift recently.


## Where implementation reality lives

When documentation claims are unclear, Claude should verify against code in
these locations:

### Core implementation

- `/home/sirrommit/claude_play/panelmark/panelmark/`

### TUI implementation

- `/home/sirrommit/claude_play/panelmark-tui/panelmark_tui/`

### HTML implementation

- `/home/sirrommit/claude_play/panelmark-html/panelmark_html/`

### Web implementation

- `/home/sirrommit/claude_play/panelmark-web/panelmark_web/`


## Documentation tasks Claude is expected to do

Claude may:

- migrate docs from the source repos into `panelmark-docs/docs/`
- reorganize content into the structure defined by
  `/home/sirrommit/claude_play/panelmark-docs/DOCUMENTATION_PLAN.md`
- rewrite wording for clarity and consistency
- remove stale links and stale claims
- add cross-links between pages
- update `mkdocs.yml` nav to match the created pages

Claude should not:

- silently copy known-stale package docs into the docs site unchanged
- claim a renderer supports more than the code and current package docs justify
- invent compatibility claims that are not already supported by the code/spec


## Required reading before major docs work

Before making substantial docs changes in `panelmark-docs`, Claude should read:

- `/home/sirrommit/claude_play/panelmark-docs/DOCUMENTATION_PLAN.md`
- `/home/sirrommit/claude_play/panelmark/ECOSYSTEM.md`
- `/home/sirrommit/claude_play/panelmark/docs/renderer-spec/overview.md`
- `/home/sirrommit/claude_play/panelmark/docs/renderer-spec/portable-library.md`

Then read the package-specific docs relevant to the section being migrated.


## Suggested migration order

Claude should generally work in this order unless the user asks otherwise:

1. Core ecosystem pages
2. Renderer-spec pages
3. Package overview pages
4. `panelmark-tui` detailed pages
5. `panelmark-html` detailed pages
6. `panelmark-web` detailed pages
7. Cross-cutting glossary/tutorial pages


## Final rule

`panelmark-docs` is the destination. The sibling repos listed above are the
source material. Claude should always pull from those exact paths before
writing new docs content.
