# Phase 1 — Foundation

> Follow CLAUDE.md. Read `docs/schema.md` before creating tables.

**Goal:** a working MVC app that connects to Postgres and does CRUD for Projects and
Profiles. No analysis concepts yet.

## Do

- Create the solution and 5 projects per CLAUDE.md (layout + dependency directions).
- Add EF Core 10 + Npgsql provider; create `CodeSuriDbContext`.
- Create tables **Project** and **AnalysisProfile** only (types per `docs/schema.md`)
  via an EF migration. Connection string from `appsettings*.json`.
- Set up Tailwind 4 + DaisyUI 5 exactly per CLAUDE.md. Confirm a DaisyUI component
  (e.g. `btn`) renders.
- Services: `ProjectService`, `ProfileService` (interfaces in Application, impls on
  `CodeSuriDbContext`). CRUD, async, `CancellationToken`.
- Controllers + Razor views: Projects (Create/Edit/Delete/List), Profiles
  (Create/Edit/Delete/List). Profiles are scoped to a project.

## Tests (xUnit, Testcontainers per CLAUDE.md)

- `ProjectService`: create, update, delete (assert persisted state).
- `ProfileService`: create, update, delete; linked to a project; `UNIQUE(ProjectId,
  Name)` enforced (duplicate insert fails).

## Acceptance

- [ ] `dotnet build` clean (no warnings in `src`).
- [ ] `dotnet ef database update` applies to a fresh Postgres.
- [ ] Projects + Profiles CRUD work end-to-end in the browser.
- [ ] DaisyUI styling visibly applied.
- [ ] All Phase-1 tests pass — report the actual `dotnet test` summary.

Do **not** create other tables or any analysis/rule code.
