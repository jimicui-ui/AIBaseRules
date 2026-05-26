---
name: test-coverage-guard
description: Enforces layered test coverage (100% Domain, Infrastructure, and all Other layers; at least 90% Application), adds or updates tests, refactors for testability, and requires a coverage gap summary when any layer misses its target. Keeps .ai/testing.md and .cursor/rules/testing.mdc accurate. Use when writing tests, changing production code by layer, fixing coverage gaps, or refactoring for testability.
---

# Test coverage guard

Read [.ai/testing.md](../../../.ai/testing.md) first — it is the source of truth. Keep [.cursor/rules/testing.mdc](../../../.cursor/rules/testing.mdc) frontmatter `globs` in sync if paths change.

## When this skill runs

Apply on any task that touches production code or tests. Do not skip coverage verification because tests “probably pass.”

## Classify every changed file

| Classification | Typical paths | Target |
|----------------|---------------|--------|
| **Domain** | `**/Domain/**`, entities, VOs, domain services | 100% line + branch |
| **Infrastructure** | `**/Infrastructure/**`, repos, adapters, EF, HTTP clients | 100% line + branch |
| **Application** | `**/Application/**`, handlers, use cases, orchestration | at least 90% line + branch |
| **Other** | Everything else (Presentation, UI, API host, `Program.cs`, shared utils, etc.) | **100%** line + branch |

Only the three named tiers have custom rules in the table; **Other is not a fourth relaxed tier** — it must hit **100%**.

If folders differ, infer by responsibility (pure rules → Domain; I/O adapters → Infrastructure; orchestration → Application; else → Other).

**Application handlers:** orchestration only. Business rules, calculations, or domain conditionals in a handler → refactor into Domain entities/services **before** writing or extending tests. See [.ai/backend-ddd-solid.md](../../../.ai/backend-ddd-solid.md) and [.ai/testing.md](../../../.ai/testing.md#application-layer-orchestration-only).

## Workflow

```
Coverage guard:
- [ ] Classify each changed file (Domain | Infrastructure | Application | Other)
- [ ] Application: handlers orchestrate only — refactor business logic to Domain before testing
- [ ] Add/update tests first (where practical)
- [ ] Implement or refactor production code
- [ ] Run tests with coverage per layer
- [ ] Met all targets on touched code — OR wrote Coverage gap summary
- [ ] Confirm testing rule still matches repo layout
```

### 1. Establish baseline

**.NET**

```bash
dotnet test --collect:"XPlat Code Coverage" --results-directory ./TestResults
```

**.TypeScript / Angular**

```bash
npm test -- --coverage
```

Report per tier/layer using folder, namespace, or `lcov` paths.

### 2. Close gaps

For each layer below its target on **touched** code:

1. List uncovered branches.
2. Add focused tests; Other and Domain/Infrastructure: no skipped branches.
3. Refactor minimally if untestable (extract pure functions, inject ports).
4. Re-run coverage.

If still below target → **do not mark the task complete** without a [Coverage gap summary](.ai/testing.md#coverage-gap-summary) (copy template from `.ai/testing.md`).

### 3. Coverage gap summary (required when target missed)

Include: **Layer**, **Target**, **Actual**, **What’s uncovered**, **Why not at target**, **What was tried**, **Path to target**.

Forbidden without user approval: lowering thresholds, blanket exclusions, or waiving coverage without documenting gaps.

### 4. Refactor loop (keep rules aligned)

- Layer moves → move tests; update `.ai/testing.md` / `testing.mdc` globs if paths change.
- New test command → document under `## Project commands` in `.ai/testing.md` if non-default.

### 5. Verify exit

Complete only when **all** are true:

- All tests pass.
- Touched **Domain, Infrastructure, Other:** 100% line + branch **or** gap summary delivered.
- Touched **Application:** at least 90% line + branch **or** gap summary delivered.
- No silent partial coverage on Other-layer code.

## Examples

**Presentation component (Other)**

1. Test bindings, branches, and event handlers.
2. Target: 100%. If a branch is unreachable, document in gap summary with evidence.

**Application handler at 88%**

1. Add tests for missing branches; re-run.
2. If still below 90%, gap summary: list handlers/branches, why, next steps.

## Anti-patterns

- Testing thick handlers that encode domain rules — refactor to Domain first, then test orchestration with at least 90% coverage.
- Treating Other as “optional” or UI as exempt from 100%.
- Finishing without summary when Application is below 90% or any layer misses its target.
- `ExcludeFromCodeCoverage` or threshold drops without user approval.
