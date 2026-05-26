# AIBaseRules

Reusable AI coding rules for [Cursor](https://cursor.com) and [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Edit rule content in `.ai/`; Cursor loads rules from `.cursor/rules/`. Optional agent workflows live in `.cursor/skills/`.

## What's included

### Rules

| Rule | Source (`.ai/`) | Applies when | Focus |
|------|-----------------|--------------|--------|
| **base** | `base.md` | Every chat (`alwaysApply: true`) | Think before coding, keep changes surgical, define verifiable success criteria |
| **backend** | `backend.md` | `backend/**/*.cs`, `**/Dockerfile`, `**/k8s/**/*.yaml` | .NET microservices, Minimal APIs, DI, async, GCP logging & secrets, containers |
| **backend-ddd-solid** | `backend-ddd-solid.md` | `backend/**/*.cs`, `**/Domain/**`, `**/Application/**`, `**/Infrastructure/**`, `**/Presentation/**` (`.cs`) | DDD layering, aggregates, ports/adapters, SOLID in C# |
| **frontend** | `frontend.md` | `frontend/**/src/**/*.{ts,html,css,scss}` | Angular standalone, OnPush, signals, strict TypeScript |
| **testing** | `testing.md` | Tests and layered code (see `testing.mdc` `globs`) | 100% Domain/Infrastructure/Other, ≥90% Application; coverage gap summary if missed |

### Skills (optional)

Copy `.cursor/skills/` into your project when you want specialized agent workflows. Each skill is a folder with `SKILL.md`.

| Skill | Use when |
|-------|----------|
| **test-coverage-guard** | Writing tests, changing layered production code, fixing coverage gaps; keeps `.ai/testing.md` and `testing.mdc` in sync |
| **mssql-env-db-helper** | SQL Server across prod/beta/dev — schema, queries, migrations |
| **stripe-recurring-payments** | Stripe Subscriptions — monthly billing, payment methods, cancel at period end |
| **angularjs-orange-blog-layout** | AngularJS 1.x blog layout with orange nav, hero, sidebar widgets |

## Repository layout

```
AIBaseRules/
├── .ai/                              # Source of truth — edit rule content here
│   ├── base.md
│   ├── backend.md
│   ├── backend-ddd-solid.md
│   ├── frontend.md
│   └── testing.md
├── .cursor/
│   ├── GuideLine.md                  # Extended setup notes
│   ├── rules/                        # Cursor entry points (.mdc)
│   │   ├── base.mdc
│   │   ├── backend.mdc
│   │   ├── backend-ddd-solid.mdc
│   │   ├── frontend.mdc
│   │   └── testing.mdc
│   └── skills/                       # Optional agent skills
│       ├── test-coverage-guard/
│       ├── mssql-env-db-helper/
│       ├── stripe-recurring-payments/
│       └── angularjs-orange-blog-layout/
└── CLAUDE.md                         # Pointer for Claude Code
```

## Quick start

### Use in your project

1. Copy `.ai/` and `.cursor/rules/` into your repo (or add this repo as a submodule).
2. Optionally copy `.cursor/skills/` for workflows you need.
3. Keep folder names `backend/` and `frontend/` so globs match, **or** update `globs` in the matching `.mdc` files (and any skill that references those paths).
4. Edit rule text in `.ai/*.md` — one place for content. Keep `.mdc` bodies as `@.ai/<name>.md` references.
5. In Cursor: **Settings → Rules** and confirm all rules appear and are enabled.

### Cursor rules (`.mdc`)

Each file under `.cursor/rules/` should:

- Set **frontmatter** in the `.mdc` (`description`, `globs`, `alwaysApply`) — Cursor reads this from `.mdc`, not from optional frontmatter in `.ai/` files.
- Reference the matching source file in the body, e.g. `@.ai/base.md`.

Example `base.mdc`:

```markdown
---
description: Behavioral guidelines to reduce common LLM coding mistakes.
alwaysApply: true
---

@.ai/base.md
```

### Claude Code

`CLAUDE.md` at the repo root points at `.ai/base.md`. To load stack-specific rules in Claude Code as well, add references to `.ai/backend.md`, `.ai/backend-ddd-solid.md`, `.ai/frontend.md`, and `.ai/testing.md` in `CLAUDE.md`.

## Customization

- **Repo layout:** Change `globs` in `.cursor/rules/*.mdc` (and mirror in skills such as `test-coverage-guard` if paths change).
- **Content:** Keep rules short and actionable (roughly under ~50 lines per topic when possible).
- **Verify:** After edits, start a new Agent chat or @-mention a rule to confirm it loads.

## More detail

See [.cursor/GuideLine.md](.cursor/GuideLine.md) for extended setup notes and examples.

## License

Add a license file if you plan to share this repository publicly.
