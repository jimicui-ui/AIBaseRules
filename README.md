# AIBaseRules

Reusable AI coding rules for [Cursor](https://cursor.com) and [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Edit rule content in `.ai/`; Cursor loads rules from `.cursor/rules/`.

## What's included

| Rule | Applies when | Focus |
|------|----------------|--------|
| **base** | Every chat (`alwaysApply: true`) | Think before coding, keep changes surgical, define verifiable success criteria |
| **backend** | `backend/**/*.cs`, Dockerfiles, `k8s/**/*.yaml` | .NET microservices, GCP, containers |
| **frontend** | `frontend/**/src/**/*.{ts,html,css,scss}` | Angular (standalone, signals, OnPush) |

## Repository layout

```
AIBaseRules/
├── .ai/                    # Source of truth — edit these files
│   ├── base.md
│   ├── backend.md
│   └── frontend.md
├── .cursor/
│   ├── GuideLine.md        # Detailed setup notes
│   └── rules/              # Cursor entry points (.mdc)
│       ├── base.mdc
│       ├── backend.mdc
│       └── frontend.mdc
└── CLAUDE.md               # Pointer for Claude Code
```

## Quick start

### Use in your project

1. Copy `.ai/` and `.cursor/rules/` into your repo (or add this repo as a submodule).
2. Keep folder names `backend/` and `frontend/` so the globs match, **or** update `globs` in `.ai/backend.md` and `.ai/frontend.md` (and the matching `.mdc` frontmatter) for your layout.
3. Edit rule text only in `.ai/*.md` — one place for content.
4. In Cursor: **Settings → Rules** and confirm `base`, `backend`, and `frontend` appear and are enabled.

### Cursor rules (`.mdc`)

Each file under `.cursor/rules/` should:

- Set **frontmatter** in the `.mdc` (`description`, `globs`, `alwaysApply`) — Cursor reads this from `.mdc`, not from `.ai/`.
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

`CLAUDE.md` at the repo root points at `.ai/base.md`. To load stack-specific rules in Claude Code as well, add references to `.ai/backend.md` and `.ai/frontend.md` in `CLAUDE.md`.

## Customization

- **Repo layout:** Change `globs` in `.ai/backend.md` and `.ai/frontend.md`, and mirror them in the corresponding `.mdc` files.
- **Content:** Keep rules short and actionable (roughly under ~50 lines per topic when possible).
- **Verify:** After edits, start a new Agent chat or @-mention a rule to confirm it loads.

## More detail

See [.cursor/GuideLine.md](.cursor/GuideLine.md) for extended setup notes and examples.

## License

Add a license file if you plan to share this repository publicly.
