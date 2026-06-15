# CodeSuri — Claude Code setup

Drop these into your repo root. Layout:

```
CLAUDE.md                       <- repo root. Loaded every session (keep lean).
docs/
  schema.md                     <- typed schema. Read on demand for migrations.
  contracts.md                  <- frozen interfaces + operator semantics.
  decisions-to-resolve.md       <- open design questions. Resolve before/early.
  phases/phase-1..6.md          <- one phase per session. NOT in CLAUDE.md.
```

## How to run a phase

In a fresh Claude Code session (CLAUDE.md loads automatically), start with:

```
Implement docs/phases/phase-1-foundation.md.
Follow CLAUDE.md. Before creating tables, read docs/schema.md.
```

For phases that touch the engine/parser, also point it at `docs/contracts.md`.
Referencing a file with `@docs/...` in your prompt loads it for that session only,
which keeps context small — that's the point of keeping these out of CLAUDE.md.

## Notes

- `CLAUDE.md` is intentionally < 200 lines (longer files reduce adherence and burn
  context every session). The heavy reference lives in `docs/` and loads on demand.
- Don't paste all six phases into CLAUDE.md — feed one per session.
- Resolve `docs/decisions-to-resolve.md` first (especially which profile a run
  evaluates), then fold the decisions into CLAUDE.md as one or two lines.
- Stack facts verified June 2026. Before Phase 1, do a 30-second sanity check on
  nuget.org for the current `Npgsql.EntityFrameworkCore.PostgreSQL` 10.x and Roslyn
  5.x versions rather than pinning a patch that may have moved.
