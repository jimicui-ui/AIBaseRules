---
description: Test coverage thresholds by architectural layer. Activates when writing or changing tests, domain logic, infrastructure adapters, or application use cases.
globs: "**/*.{cs,ts}, **/tests/**/*, **/*Tests/**/*, **/*.spec.ts, **/*Test*.cs, **/Domain/**/*, **/Infrastructure/**/*, **/Application/**/*"
alwaysApply: false
---

# Testing & coverage rules

## Three named tiers (only tiers with custom thresholds)

| Tier | Paths (typical) | Line + branch coverage |
|------|-----------------|------------------------|
| **Domain** | `**/Domain/**`, entities, value objects, domain services | **100%** |
| **Infrastructure** | `**/Infrastructure/**`, persistence, messaging, external adapters | **100%** |
| **Application** | `**/Application/**`, handlers, use cases, API/orchestration | **≥ 90%** |

If the repo uses different folder names, map folders to the nearest tier before measuring.

## All other layers

Any production code **not** classified as Domain, Infrastructure, or Application is **Other** (e.g. Presentation, UI, API hosts, composition root, shared helpers, generated or third-party wrappers you own).

| Rule | Requirement |
|------|-------------|
| Coverage | **100%** line + branch on touched Other-layer code |
| Below 100% | **Not allowed to finish silently** — see [Coverage gap summary](#coverage-gap-summary) |

## Coverage gap summary

When touched code in **any** layer does **not** meet its target (Domain/Infrastructure/Other **100%**, Application **≥ 90%**), stop and give the user a short written summary **before** treating the task as done:

```markdown
## Coverage gap summary

**Layer:** [Domain | Infrastructure | Application | Other — name/path]
**Target:** [100% | ≥90%]
**Actual:** [line % / branch % or “unknown” + how measured]
**What’s uncovered:** [files, types, or branches — be specific]
**Why not at target:** [e.g. untestable framework hook, missing test infra, time box, third-party binary, user asked to defer]
**What was tried:** [tests added, refactors attempted]
**Path to target:** [concrete next steps, or “needs user decision”]
```

Do not exclude files, lower thresholds, or add `ExcludeFromCodeCoverage` without explicit user approval. A gap summary is not a substitute for approval to waive coverage.

## Test-first workflow

1. Classify each changed file: **Domain**, **Infrastructure**, **Application**, or **Other**.
2. Add or update tests **before** or **with** production changes (red → green → refactor).
3. Run the project test command with coverage; meet the target per tier/layer on touched code.
4. If any target is missed, deliver the [coverage gap summary](#coverage-gap-summary).
5. Prefer fast unit tests for Domain; fakes/in-memory doubles for Infrastructure; integration tests only where Application or Other behavior needs real wiring.

## What to test

- **Domain:** every branch, invariant, and domain error path — no untested `throw` or guard clause.
- **Infrastructure:** mapping, serialization, retry/error paths, and adapter contract edges (use test doubles for external systems).
- **Application:** happy path, validation failures, authorization boundaries, and orchestration branches.
- **Other:** every branch in touched UI/API/host code; framework-only boilerplate with no logic may be extracted or thin-wrapped so testable code stays at 100%.

## Refactoring constraints

- Refactor for testability when required to meet coverage; keep behavior unchanged unless the user asked for a behavior change.
- When production code moves between layers, move tests accordingly and re-run coverage.

## Success criteria

- All existing tests pass.
- **Domain, Infrastructure, Other:** **100%** line and branch on touched code — or a coverage gap summary.
- **Application:** **≥ 90%** line and branch on touched code — or a coverage gap summary.
- No new untested public surface in Domain, Infrastructure, or Other without a documented gap summary.
