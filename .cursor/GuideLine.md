# AIBaseRules — Quick guide

Reusable AI coding rules for Cursor and Claude. Edit content in `.ai/`; Cursor loads rules from `.cursor/rules/`.

## Folder layout

```
AIBaseRules/
├── .ai/                    # Source of truth (edit these)
│   ├── base.md             # Always-on behavioral rules
│   ├── backend.md          # .NET / GCP (file-scoped)
│   └── frontend.md         # Angular (file-scoped)
├── .cursor/
│   ├── GuideLine.md        # This file
│   └── rules/              # Cursor entry points
│       ├── base.mdc
│       ├── backend.mdc
│       └── frontend.mdc
└── CLAUDE.md               # Pointer for Claude Code
```

## What each rule does

| File | When it applies |
|------|------------------|
| `base.md` | Every chat (`alwaysApply: true`) — think first, keep changes small, define success criteria |
| `backend.md` | When working under `backend/**/*.cs`, Dockerfiles, or `k8s/**/*.yaml` |
| `frontend.md` | When working under `frontend/**/src/**/*.{ts,html,css,scss}` |

## How to use in a project

1. Copy `.ai/` and `.cursor/rules/` into your repo (or submodule this repo).
2. Keep the same folder names (`backend/`, `frontend/`) so globs match, or adjust `globs` in `.ai/backend.md` and `.ai/frontend.md`.
3. Edit rule text in `.ai/*.md` only — keep one place for content.
4. In Cursor: **Settings → Rules** and confirm `base`, `backend`, and `frontend` appear and are enabled.

## Cursor `.mdc` files

Each file in `.cursor/rules/` should point at the matching `.ai` file. Prefer:

- YAML **frontmatter** in the `.mdc` (`description`, `globs`, `alwaysApply`) — Cursor reads this from `.mdc`, not from `.ai/`.
- Body: `@.ai/base.md` (or `backend.md` / `frontend.md`) so the full rule content is included.

Example for `base.mdc`:

```markdown
---
description: Behavioral guidelines to reduce common LLM coding mistakes.
alwaysApply: true
---

@.ai/base.md
```

## Claude Code

`CLAUDE.md` at the repo root references `.ai/base.md`. Add lines for `backend.md` and `frontend.md` if you want the same stack rules there.

## Tips

- Keep rules short and actionable (under ~50 lines per topic when possible).
- Change globs when your repo layout differs from `backend/` and `frontend/`.
- After edits, start a new Agent chat or @-mention a rule to confirm it loads.
