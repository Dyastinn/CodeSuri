# CodeSuri — frozen contracts

> Read before touching the rule engine, operators, or parsers. Do not change these
> signatures across phases. Do not create parallel types.

## Shared metadata type

One shape, used by **both** the parser (output) and the rule engine (input).

```csharp
// CodeSuri.Domain (or Application) — shared
public sealed record EntityMetadata(
    string EntityTypeName,                              // matches EntityType.Name exactly
    int? LineNumber,
    IReadOnlyDictionary<string, string?> Properties);  // keys match PropertyDefinition.Name exactly
```

## Rule engine (Phase 3)

```csharp
public sealed record RuleEvaluationResult(bool Passed, string? FailedConditionMessage);

public interface IRuleEvaluator
{
    // One rule vs one entity. All conditions AND-ed.
    RuleEvaluationResult Evaluate(Rule rule, EntityMetadata entity);
}

// One implementation per Operator.Name. Resolve by name; fail loudly if unknown.
public interface IConditionOperator
{
    string OperatorName { get; }                 // "Equals", "Regex", ...
    bool Matches(string? actualValue, string expectedValue);
}
```

## Parser (Phase 5)

```csharp
public interface IParser
{
    bool CanParse(string fileExtension);          // ".cs", ".vb"
    IReadOnlyList<EntityMetadata> Parse(string filePath, string sourceText);
}
```

## Operator semantics (freeze)

- All comparisons are **string** comparisons of the entity's property value
  (`actualValue`) against `ExpectedValue`. `DataType` is metadata only; **no
  numeric/boolean coercion** (there are no numeric operators).
- **Case-sensitive, ordinal** comparison.
- Null `actualValue`:
  - `Equals` → match only if `expectedValue` is empty string
  - `NotEquals` → match unless `expectedValue` is empty string
  - `Contains` / `StartsWith` / `EndsWith` / `Regex` → no match (false)
- `Regex` uses `System.Text.RegularExpressions.Regex` with a **mandatory 1-second
  `matchTimeout`** (ReDoS guard). A bad pattern is a rule-config error, not a crash.
