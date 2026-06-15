# Phase 6 — End-to-end

> Follow CLAUDE.md. Read `docs/schema.md` and `docs/contracts.md`.
> Resolve `docs/decisions-to-resolve.md` #1 (run-time profile scoping) before this phase.

**Goal:** ZIP → catalog → parse → evaluate rules → Findings.

## Do

- Create **Finding** (types per schema).
- Implement a synchronous, in-request pipeline (per CLAUDE.md architecture limits):
  for an `AnalysisRun`, read cataloged `.cs`/`.vb` files, parse each via the matching
  `IParser`, evaluate every **Enabled** rule whose `EntityType` matches the entity
  (scoped to the run's profile — see decision #1) via `IRuleEvaluator`, and write a
  `Finding` per failure (FilePath, LineNumber from the entity, Severity from the
  rule, Message describing which condition failed).
- UI: show findings for a run.

## Test

- One full-pipeline test: fixed input ZIP + known rules → asserted exact set of
  Findings (count, file paths, line numbers, messages).

## Acceptance

- [ ] End-to-end run produces the expected Findings for the fixture.
- [ ] Findings persisted and visible in UI.
- [ ] Full-pipeline test passes.
