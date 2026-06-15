# CodeSuri — decisions to resolve

> Resolve these, then fold the chosen answers into `CLAUDE.md` (1–2 lines each).
> Don't leave open questions in persistent memory.

1. **Run-time profile scoping (affects Phase 6).** When an analysis runs, which
   rules apply? The project's **default** profile, a **user-selected** profile, or
   **all** profiles? Default assumption in the briefs: the project's default
   profile. Confirm or change.

2. **Severity: enum vs lookup table.** Currently a string-backed enum
   { Info, Warning, Error }. If you need user-defined severities, it must become a
   table.

3. **AND-only conditions.** Fine for the MVP (no boolean expression tree). If real
   standards need OR / grouping, that's a schema + evaluator change — decide before
   it sneaks in mid-build.

4. **Seeded-but-unparsed entity types** (Folder, Interface, Enum, Property, Table,
   Column, StoredProcedure). They exist as vocabulary but nothing produces them in
   the MVP. Decide whether to hide them in the rule-creation UI so users don't build
   rules that can never fire.
