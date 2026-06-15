# CodeSuri — project memory

> Loaded every session. Keep this lean. Detailed reference lives in `docs/` and is
> read on demand (see "Reference docs" below) so it doesn't burn context every run.
> Stack facts verified 2026-06.

## Behavior (apply every session)

- **Don't invent APIs.** If a type, method, NuGet package, or CLI flag isn't named
  here or in the referenced docs, and you aren't certain it exists in the pinned
  versions, STOP and ask. Do not guess a plausible name.
- **Build and run before saying "done."** Run `dotnet build` and `dotnet test`;
  report the real output. "Should work" is not acceptance.
- **Don't add packages** not listed here without first stating package, version, why.
- **Don't build ahead.** Implement only the current phase's tables, pages, and code.
- **Stay inside the architecture limits below.** If a task seems to need a banned
  item, you've misread the task — ask.

## Reference docs (read the relevant one before you touch that area)

- Changing DB schema / writing a migration → **read `docs/schema.md`** and match it
  exactly (types, keys, FKs, nullability). Do not improvise column types.
- Rule engine, operators, or parser → **read `docs/contracts.md`** (frozen
  interface signatures + operator semantics). Do not create parallel types.
- Working a phase → the phase brief is in `docs/phases/phase-N.md`. Do that phase only.

(These are read-on-demand by design. Don't assume their contents from memory.)

## Stack (pinned)

- .NET **10.0** (LTS). ASP.NET Core **MVC** — controllers + Razor views. Not Razor
  Pages, not Blazor, not Minimal APIs for UI.
- EF Core **10.x**; provider `Npgsql.EntityFrameworkCore.PostgreSQL` **10.x**
  (resolve latest stable for EF Core 10). PostgreSQL 16+.
- TailwindCSS **4.x** + DaisyUI **5.x** (CSS-first — see below).
- Tests: **xUnit** + **Testcontainers.PostgreSql**. Never the EF InMemory provider.
- Parser (Phase 5+): Roslyn — `Microsoft.CodeAnalysis.CSharp` /
  `Microsoft.CodeAnalysis.VisualBasic` (~5.3.x, resolve latest stable).
  **Never hand-write a parser.**

## Solution layout

```
CodeSuri.sln
  src/CodeSuri.Web             // MVC: controllers, views, wwwroot, Program.cs
  src/CodeSuri.Application      // services, interfaces, DTOs, engines
  src/CodeSuri.Domain           // entities + enums only; no framework deps
  src/CodeSuri.Infrastructure   // DbContext, EF configs, migrations, providers
  tests/CodeSuri.Tests          // xUnit
```

Dependencies: Web → Application → Domain; Infrastructure → Application/Domain;
Web → Infrastructure (composition root only). Domain depends on nothing.

## Conventions

- Nullable reference types on; warnings as errors in `src`.
- IO/service methods `async` with `CancellationToken`.
- One `CodeSuriDbContext` (Infrastructure). EF configs via
  `IEntityTypeConfiguration<T>`, not data annotations.
- Default EF naming (PascalCase). No snake_case naming package.
- PK type `long` (bigint identity) unless `docs/schema.md` says otherwise.
- Timestamps `timestamptz` / `DateTimeOffset`, server default `now()`.
- Every schema change = a new migration. Never edit an applied migration.

## Architecture limits (hard)

Single monolithic MVC app. Analysis runs **synchronously in-request** for the MVP.

Business configuration lives in **PostgreSQL**, never in code: naming conventions,
coding standards, repository settings, rule definitions, profiles, and the
entity/property/operator metadata. Allowed in code: operator implementations, rule
engine, pipeline engine, parsers, git providers, file/zip/hash handling.

**Banned — do not introduce:** CQRS, MediatR, event sourcing, DDD aggregates,
microservices, message queues, GraphQL, Redis, Kafka, SignalR, background-job
frameworks (Hangfire/Quartz). If something seems to need these, ask first.

## TailwindCSS 4 + DaisyUI 5 (CSS-first — frequent error source)

There is **no JS config file.** Setup, inside `src/CodeSuri.Web`:

```bash
npm i -D tailwindcss@4 @tailwindcss/cli@4 daisyui@5
```

`Styles/app.css`:
```css
@import "tailwindcss";
@plugin "daisyui";
```

`package.json` scripts:
```json
"css:build": "tailwindcss -i ./Styles/app.css -o ./wwwroot/css/app.css --minify",
"css:watch": "tailwindcss -i ./Styles/app.css -o ./wwwroot/css/app.css --watch"
```

Layout: `<link rel="stylesheet" href="~/css/app.css" asp-append-version="true" />`

**DO NOT** (these are Tailwind v3 patterns and will break v4/v5):
- create `tailwind.config.js` or `postcss.config.js`
- run `npx tailwindcss init`
- use `@tailwind base; @tailwind components; @tailwind utilities;`
- add daisyui to a `plugins: []` array or `require("daisyui")`
- use `@tailwindcss/postcss7-compat` or a PostCSS pipeline

## Testing

- Pure unit tests (no DB) for `IRuleEvaluator` and each `IConditionOperator`.
- Service/EF tests against real Postgres via `Testcontainers.PostgreSql`.
  Needs Docker in the run env; if unavailable, fall back to a dedicated test DB +
  `Respawn` and say so in your report. **Never** the EF InMemory provider.
