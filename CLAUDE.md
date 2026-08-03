# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository state

This repository currently contains **no application source code** — only a `docs/` folder that scaffolds the project's lifecycle documentation (in Thai). There is no build system, package manifest, linter, or test runner yet. Do not invent build/lint/test commands; when code is added to this repo, this file should be updated with the real commands at that time.

## Documentation architecture (`docs/`)

The `docs/` tree encodes a linear project workflow, one numbered top-level folder per stage:

```
00-archived        deprecated/superseded docs (never delete docs — move them here instead)
01-requirements
  01-spec          source of truth for what the system must do (features, user stories, business rules, scope)
  02-plan          roadmap/phases/milestones derived from 01-spec
  03-task          concrete task breakdown derived from 02-plan
02-design
  01-prototypes    UI/UX wireframes, mockups, user flow, design system basics
  02-technical     architecture, DB schema, API design, tech/library choices
03-testing
  01-test-plan     test cases/scenarios derived from 01-spec + 02-design
  02-test-result   actual pass/fail results and bugs found against 01-test-plan
04-retrospectives  lessons learned per phase/sprint, sourced from 02-test-result and 05-log
05-log             chronological changelog / decision log
```

Each stage's `index.md` states which upstream folder it derives from and which downstream folder its output feeds — treat this as the intended reading/authoring order: `01-requirements` → `02-design` → `03-testing` → `04-retrospectives`, with `05-log` running alongside as a chronological record and `00-archived` as the holding area for anything superseded.

Cross-references between docs use Obsidian-style wikilinks, e.g. `[[../02-plan/index|02-plan]]`. Preserve this linking style when adding or editing docs so the doc graph stays navigable.

**Convention: never delete a doc outright.** Per `docs/00-archived/index.md`, superseded or cancelled documents/plans should be moved into `00-archived/` rather than removed, to preserve the project's decision history.
