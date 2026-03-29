# panelmark Documentation Plan

This document is a work plan for an AI coding agent to build out the
`panelmark-docs` site.

The goal is not just to generate pages, but to create one canonical
documentation source for the `panelmark` ecosystem that is:

- accurate
- current
- explicit about boundaries between packages
- explicit about portable vs renderer-specific behavior


## Primary objective

Turn the current scattered markdown documentation across:

- `panelmark/`
- `panelmark-tui/`
- `panelmark-html/`
- `panelmark-web/`

into a coherent MkDocs site under:

- [panelmark-docs](/home/sirrommit/claude_play/panelmark-docs)

without inventing new architecture or drifting from the current implementation.


## Core principles

When writing or migrating docs, follow these rules:

- The docs site is the canonical cross-package documentation surface.
- Package-local READMEs should stay concise and point back to the docs site.
- Core `panelmark` docs own:
  - shell language
  - renderer contract
  - portability model
  - ecosystem architecture
- Renderer package docs own:
  - concrete APIs
  - implementation details
  - limitations
  - examples
- Do not promise features that are not implemented.
- When a renderer does not meet a spec tier, say so explicitly.


## Current source material to mine

The agent should use these files as the current source material.

### Core docs

- [panelmark/README.md](/home/sirrommit/claude_play/panelmark/README.md)
- [panelmark/ECOSYSTEM.md](/home/sirrommit/claude_play/panelmark/ECOSYSTEM.md)
- [panelmark/docs/shell-language.md](/home/sirrommit/claude_play/panelmark/docs/shell-language.md)
- [panelmark/docs/draw-commands.md](/home/sirrommit/claude_play/panelmark/docs/draw-commands.md)
- [panelmark/docs/custom-interactions.md](/home/sirrommit/claude_play/panelmark/docs/custom-interactions.md)
- [panelmark/docs/renderer-spec/overview.md](/home/sirrommit/claude_play/panelmark/docs/renderer-spec/overview.md)
- [panelmark/docs/renderer-spec/contract.md](/home/sirrommit/claude_play/panelmark/docs/renderer-spec/contract.md)
- [panelmark/docs/renderer-spec/portable-library.md](/home/sirrommit/claude_play/panelmark/docs/renderer-spec/portable-library.md)
- [panelmark/docs/renderer-spec/extensions.md](/home/sirrommit/claude_play/panelmark/docs/renderer-spec/extensions.md)
- [panelmark/docs/renderer-spec/readiness.md](/home/sirrommit/claude_play/panelmark/docs/renderer-spec/readiness.md)

### panelmark-tui docs

- [panelmark-tui/README.md](/home/sirrommit/claude_play/panelmark-tui/README.md)
- [panelmark-tui/docs/interactions.md](/home/sirrommit/claude_play/panelmark-tui/docs/interactions.md)
- [panelmark-tui/docs/widgets.md](/home/sirrommit/claude_play/panelmark-tui/docs/widgets.md)
- [panelmark-tui/docs/renderer-implementation.md](/home/sirrommit/claude_play/panelmark-tui/docs/renderer-implementation.md)
- [panelmark-tui/docs/getting-started.md](/home/sirrommit/claude_play/panelmark-tui/docs/getting-started.md)
- [panelmark-tui/CONTRIBUTING.md](/home/sirrommit/claude_play/panelmark-tui/CONTRIBUTING.md)
- [panelmark-tui/KNOWN_LIMITATIONS.md](/home/sirrommit/claude_play/panelmark-tui/KNOWN_LIMITATIONS.md)

### panelmark-html docs

- [panelmark-html/README.md](/home/sirrommit/claude_play/panelmark-html/README.md)
- [panelmark-html/docs/hook-contract.md](/home/sirrommit/claude_play/panelmark-html/docs/hook-contract.md)
- [PANELMARK_HTML_IMPLEMENTATION_PLAN.md](/home/sirrommit/claude_play/PANELMARK_HTML_IMPLEMENTATION_PLAN.md)

### panelmark-web docs

- [panelmark-web/README.md](/home/sirrommit/claude_play/panelmark-web/README.md)
- [panelmark-web/docs/getting-started.md](/home/sirrommit/claude_play/panelmark-web/docs/getting-started.md)
- [panelmark-web/docs/hook-contract-web.md](/home/sirrommit/claude_play/panelmark-web/docs/hook-contract-web.md)
- [panelmark-web/docs/interaction-coverage.md](/home/sirrommit/claude_play/panelmark-web/docs/interaction-coverage.md)


## Site structure to build out

The initial MkDocs skeleton already exists. Expand it rather than replacing it.

Current files:

- [mkdocs.yml](/home/sirrommit/claude_play/panelmark-docs/mkdocs.yml)
- [docs/index.md](/home/sirrommit/claude_play/panelmark-docs/docs/index.md)
- [docs/getting-started.md](/home/sirrommit/claude_play/panelmark-docs/docs/getting-started.md)
- [docs/ecosystem.md](/home/sirrommit/claude_play/panelmark-docs/docs/ecosystem.md)
- [docs/renderer-spec.md](/home/sirrommit/claude_play/panelmark-docs/docs/renderer-spec.md)
- package overview pages under `docs/packages/`

Recommended final top-level structure:

```text
docs/
├── index.md
├── getting-started.md
├── ecosystem.md
├── glossary.md
├── shell-language/
│   ├── overview.md
│   ├── syntax.md
│   └── examples.md
├── renderer-spec/
│   ├── overview.md
│   ├── contract.md
│   ├── portable-library.md
│   ├── extensions.md
│   └── readiness.md
├── packages/
│   ├── panelmark.md
│   ├── panelmark-tui.md
│   ├── panelmark-html.md
│   └── panelmark-web.md
├── panelmark-tui/
│   ├── getting-started.md
│   ├── interactions.md
│   ├── widgets.md
│   ├── renderer-implementation.md
│   └── limitations.md
├── panelmark-html/
│   ├── overview.md
│   ├── rendering-api.md
│   └── hook-contract.md
└── panelmark-web/
    ├── overview.md
    ├── getting-started.md
    ├── protocol.md
    ├── hook-usage.md
    └── interaction-coverage.md
```

The agent does not need to build all of this in one pass, but should use this
as the target structure.


## Work phases

### Phase 1 — Make the docs site structurally complete

Goal:

- convert the current placeholder pages into useful landing pages
- expand `mkdocs.yml` nav to reflect the actual document set

Tasks:

- [ ] Review existing placeholder pages in `panelmark-docs/docs/`
- [ ] Add missing directories/files for the major sections listed above
- [ ] Update `mkdocs.yml` nav so every created page is reachable
- [ ] Keep URLs clean and predictable

Do not:

- duplicate the same content in multiple places
- leave orphaned pages out of nav


### Phase 2 — Migrate core architecture and renderer spec docs

Goal:

- move the important core docs into the site first

Tasks:

- [ ] Build `docs/ecosystem.md` from `panelmark/ECOSYSTEM.md`, but remove stale
  implementation inventories and dead references
- [ ] Create `docs/renderer-spec/overview.md`
- [ ] Create `docs/renderer-spec/contract.md`
- [ ] Create `docs/renderer-spec/portable-library.md`
- [ ] Create `docs/renderer-spec/extensions.md`
- [ ] Create `docs/renderer-spec/readiness.md`
- [ ] Create shell-language pages from `panelmark/docs/shell-language.md` and `panelmark/docs/draw-commands.md`

Editorial rules:

- Preserve the normative content of the renderer spec
- Clean up stale references while migrating
- Do not overcompress the spec into one page if it is clearer as multiple pages


### Phase 3 — Build package overview pages

Goal:

- make each package page an honest high-level entry point

Tasks:

- [ ] Expand `docs/packages/panelmark.md`
- [ ] Expand `docs/packages/panelmark-tui.md`
- [ ] Expand `docs/packages/panelmark-html.md`
- [ ] Expand `docs/packages/panelmark-web.md`

Each package overview page should answer:

- what the package does
- what it does not do
- who should use it
- what its current status is
- where to go next in the docs site


### Phase 4 — Migrate panelmark-tui docs

Goal:

- bring over the concrete TUI documentation into the site

Tasks:

- [ ] Create `docs/panelmark-tui/getting-started.md`
- [ ] Create `docs/panelmark-tui/interactions.md`
- [ ] Create `docs/panelmark-tui/widgets.md`
- [ ] Create `docs/panelmark-tui/renderer-implementation.md`
- [ ] Create `docs/panelmark-tui/limitations.md`

Source material:

- `panelmark-tui/docs/*.md`
- `panelmark-tui/README.md`
- `panelmark-tui/KNOWN_LIMITATIONS.md`

Important:

- fix any known drift while migrating
- keep interaction and widget inventories accurate
- clearly label what is portable vs TUI-specific


### Phase 5 — Migrate panelmark-html docs

Goal:

- document `panelmark-html` as the static structural substrate

Tasks:

- [ ] Create `docs/panelmark-html/overview.md`
- [ ] Create `docs/panelmark-html/rendering-api.md`
- [ ] Create `docs/panelmark-html/hook-contract.md`

Source material:

- `panelmark-html/README.md`
- `panelmark-html/docs/hook-contract.md`
- `PANELMARK_HTML_IMPLEMENTATION_PLAN.md`

Important:

- keep the static-vs-live boundary clear
- document only the actual stable hook contract
- do not reintroduce contradictions about stability


### Phase 6 — Migrate panelmark-web docs

Goal:

- document `panelmark-web` honestly as it exists today

Tasks:

- [ ] Create `docs/panelmark-web/overview.md`
- [ ] Create `docs/panelmark-web/getting-started.md`
- [ ] Create `docs/panelmark-web/protocol.md`
- [ ] Create `docs/panelmark-web/hook-usage.md`
- [ ] Create `docs/panelmark-web/interaction-coverage.md`

Source material:

- `panelmark-web/README.md`
- `panelmark-web/docs/getting-started.md`
- `panelmark-web/docs/hook-contract-web.md`
- `panelmark-web/docs/interaction-coverage.md`

Important:

- do not imply full portable-library compatibility unless it is actually true
- clearly distinguish:
  - generic live runtime support
  - built-in interaction/widget coverage


### Phase 7 — Add cross-cutting explanatory pages

Goal:

- make the docs site navigable for new users

Tasks:

- [ ] Add `docs/glossary.md`
- [ ] Add a shell-language examples page
- [ ] Add an explanation of portable vs renderer-specific concepts
- [ ] Add a “Which package do I need?” explanation somewhere prominent


## Writing standards

Every migrated page should be:

- concise at the top
- explicit about scope
- linked to related pages
- free of stale file paths and removed-document references

When a page is normative:

- say so clearly

When a page is descriptive/current-state:

- say what is implemented today

When a feature is not implemented:

- say that explicitly rather than implying future intent is current reality


## Specific issues to avoid

The agent should actively avoid these mistakes:

- Copying stale counts of interactions/widgets from package READMEs without
  verifying exports.
- Repeating dead references like `renderer-boundary.md`.
- Claiming a renderer is `portable-library-compatible` when it is not.
- Mixing implementation plans with current product docs without labeling them.
- Describing `panelmark-html` as a full live renderer.
- Describing `panelmark-web` as having a built-in interaction library if it
  still only hosts arbitrary draw-command interactions.


## Recommended implementation order

If the work must be broken into multiple PRs or agent passes, use this order:

1. Core ecosystem + renderer spec pages
2. Package overview pages
3. `panelmark-tui` pages
4. `panelmark-html` pages
5. `panelmark-web` pages
6. Cross-cutting glossary/tutorial pages
7. README back-links from package repos to the new site


## Verification checklist

Before considering the docs site pass complete, verify:

- [ ] `mkdocs.yml` nav matches the files that actually exist
- [ ] no page links to removed/stale core docs
- [ ] package status claims match the current implementation
- [ ] portable-library claims are truthful
- [ ] `panelmark-html` and `panelmark-web` boundaries are stated clearly
- [ ] `panelmark-web` interaction coverage is documented honestly
- [ ] renderer-spec pages reflect the current `panelmark/docs/renderer-spec/`
  content
- [ ] package-local docs and site docs do not obviously contradict each other


## Optional follow-up after the docs site exists

Once the site content is in place:

- [ ] update package READMEs to link back to the canonical docs site
- [ ] add a docs badge/link once Read the Docs is configured
- [ ] consider moving or de-emphasizing duplicated repo-local docs where the
  site becomes authoritative
