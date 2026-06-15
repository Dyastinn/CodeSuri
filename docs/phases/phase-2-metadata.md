# Phase 2 — Metadata

> Follow CLAUDE.md. Read `docs/schema.md` (incl. seeds + property contract).

**Goal:** manage the analyzer's vocabulary. No analysis.

## Do

- Create **EntityType**, **PropertyDefinition**, **Operator** (types per schema).
- Seed `EntityType` and `Operator` per `docs/schema.md` (idempotent seeding).
- UI: EntityType CRUD; PropertyDefinition CRUD (scoped to an entity type, `DataType`
  constrained to String/Integer/Boolean); Operator **view-only**.

## Critical contract (freeze now)

`PropertyDefinition.Name` values are the exact keys the parser will emit later. Seed
the property definitions listed under "Property contract" in `docs/schema.md`
exactly (File/Class/Method/Variable + their properties, `Variable.Scope` values).

## Tests

- Create EntityType; create PropertyDefinition; operators load (all 6 present).

## Acceptance

- [ ] Metadata manageable via UI; operators read-only.
- [ ] Seeds present and idempotent.
- [ ] The property-contract definitions exist exactly as named.
- [ ] Tests pass.
