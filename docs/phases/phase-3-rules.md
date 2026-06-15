# Phase 3 — Rule system

> Follow CLAUDE.md. Read `docs/schema.md` and `docs/contracts.md`.

**Goal:** create configurable rules and evaluate them against **in-memory**
`EntityMetadata`. No files.

## Do

- Create **Rule**, **RuleCondition** (types per schema, incl. `Rule.ProfileId`).
- UI: create/edit a Rule (pick Profile, EntityType, Severity, Enabled) with N
  conditions (PropertyDefinition + Operator + ExpectedValue). All dropdowns from DB
  rows — nothing hardcoded.
- Implement `IRuleEvaluator` and one `IConditionOperator` per operator, per the
  frozen signatures and semantics in `docs/contracts.md`. Resolve operators by
  `Operator.Name`; fail loudly on an unknown operator.

## Tests (pure unit, no DB)

Each with a passing and failing case: Equals, NotEquals, Contains, StartsWith,
EndsWith, Regex. Plus: a multi-condition rule proving AND semantics; the
null-handling rules from `docs/contracts.md`; the Regex `matchTimeout` path.

## Acceptance

- [ ] Rules + conditions created via UI, persisted, all from DB rows.
- [ ] `IRuleEvaluator` returns correct Pass/Fail for every operator.
- [ ] Tests pass. No file/parser code exists yet.
