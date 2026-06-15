# Phase 5 — Parser (Roslyn) — highest-risk phase

> Follow CLAUDE.md. Read `docs/contracts.md` and `docs/schema.md` (property contract).
> **Use Roslyn — do not hand-write a parser.**

**Goal:** turn `.cs` and `.vb` source into `EntityMetadata`. No rule execution.

## Packages

`Microsoft.CodeAnalysis.CSharp`, `Microsoft.CodeAnalysis.VisualBasic` (~5.3.x,
resolve latest stable). Parse with `CSharpSyntaxTree.ParseText` /
`VisualBasicSyntaxTree.ParseText` and walk the **syntax tree** (syntactic only — no
Compilation/semantic model needed for the MVP property set).

## Recommended split

- **5a — C# parser** (`CSharpParser : IParser`, `CanParse(".cs")`).
- **5b — VB.NET parser** (`VbNetParser : IParser`, `CanParse(".vb")`).

## Extract (exactly the Phase-2 property contract)

- **File**: one per file — Name, Extension, Path.
- **Class**: Name. (C#: `ClassDeclarationSyntax`; VB: `ClassStatementSyntax`/`ClassBlockSyntax`.)
- **Method**: Name, ReturnType. (C#: `MethodDeclarationSyntax`; VB: `MethodStatementSyntax`.)
- **Variable**: Name, Scope (Local|Parameter|Field), DataType. Cover locals,
  parameters, fields.

Each `EntityMetadata` carries a 1-based `LineNumber` from
`node.GetLocation().GetLineSpan()`.

## Cross-language normalization (decide and document)

Map language-specific type names to one shared vocabulary so a rule written once
matches both languages. Minimum: VB `Integer`↔C# `int`, `String`↔`string`,
`Boolean`↔`bool`. Pick a target vocabulary, apply it in both parsers, state the mapping.

## Tests

Real `.cs` and `.vb` sample files in the test project. Verify method / parameter /
local / field / class detection, correct LineNumbers, and that the **same construct
in C# and VB yields the same normalized property values**.

## Acceptance

- [ ] Both parsers emit `EntityMetadata` with keys matching the Phase-2 property
      names exactly.
- [ ] Type normalization verified across the two languages.
- [ ] Tests pass. No rule execution yet.
