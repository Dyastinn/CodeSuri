# CodeSuri — canonical schema

> Read this before creating or altering any table or migration. Types are Postgres;
> C# entity types in parentheses. **Create a table only in its phase** (noted on the
> right). Future tables are listed here for contract stability — do not build ahead.

```
Project                       -- Phase 1
  Id            bigint identity PK            (long)
  Name          varchar(200)  not null        (string)
  Description   varchar(2000) null            (string?)
  CreatedDate   timestamptz   not null now()  (DateTimeOffset)

AnalysisProfile               -- Phase 1
  Id            bigint identity PK
  ProjectId     bigint        not null  FK Project(Id) on delete cascade
  Name          varchar(200)  not null
  IsDefault     boolean       not null  default false
  UNIQUE (ProjectId, Name)

EntityType                    -- Phase 2  (seeded)
  Id            bigint identity PK
  Name          varchar(100)  not null  UNIQUE

PropertyDefinition            -- Phase 2
  Id            bigint identity PK
  EntityTypeId  bigint        not null  FK EntityType(Id) on delete cascade
  Name          varchar(100)  not null
  DataType      varchar(50)   not null  -- one of: String | Integer | Boolean
  UNIQUE (EntityTypeId, Name)

Operator                      -- Phase 2  (seeded, view-only in UI)
  Id            bigint identity PK
  Name          varchar(50)   not null  UNIQUE
  -- seeded: Equals, NotEquals, Contains, StartsWith, EndsWith, Regex

Rule                          -- Phase 3
  Id            bigint identity PK
  ProfileId     bigint        not null  FK AnalysisProfile(Id) on delete cascade   -- NOTE 1
  EntityTypeId  bigint        not null  FK EntityType(Id)
  Name          varchar(200)  not null
  Description   varchar(2000) null
  Severity      varchar(20)   not null  -- enum Severity { Info, Warning, Error } as string
  Enabled       boolean       not null  default true

RuleCondition                 -- Phase 3
  Id                   bigint identity PK
  RuleId               bigint not null  FK Rule(Id) on delete cascade
  PropertyDefinitionId bigint not null  FK PropertyDefinition(Id)
  OperatorId           bigint not null  FK Operator(Id)
  ExpectedValue        varchar(500) not null
  -- NOTE 2: all conditions on a rule are AND-ed for the MVP.

AnalysisRun                   -- Phase 4
  Id            bigint identity PK
  ProjectId     bigint        not null  FK Project(Id)
  StartedDate   timestamptz   not null
  FinishedDate  timestamptz   null
  Status        varchar(20)   not null  -- enum RunStatus { Pending, Running, Completed, Failed }

FileCatalog                   -- Phase 4
  Id              bigint identity PK
  AnalysisRunId   bigint      not null  FK AnalysisRun(Id) on delete cascade
  Path            varchar(1000) not null  -- relative path inside the zip
  Extension       varchar(20)  null
  Size            bigint       not null
  Hash            char(64)     not null   -- SHA-256, lowercase hex
  INDEX (AnalysisRunId)

Finding                       -- Phase 6
  Id            bigint identity PK
  AnalysisRunId bigint        not null  FK AnalysisRun(Id) on delete cascade
  RuleId        bigint        not null  FK Rule(Id)
  FilePath      varchar(1000) not null
  LineNumber    int           null
  Severity      varchar(20)   not null
  Message       varchar(2000) not null
  INDEX (AnalysisRunId)
```

**NOTE 1:** `Rule.ProfileId` links rules to a profile. The original plan described a
profile as "a collection of rules" but never connected them — this fixes that.

**NOTE 2:** AND-only avoids a boolean expression tree. OR / nesting is a deliberate
future extension, not part of these phases.

## Seeds

- `EntityType`: File, Folder, Class, Interface, Enum, Method, Variable, Property,
  Table, Column, StoredProcedure. (Most are vocabulary only; not all are parsed.)
- `Operator`: Equals, NotEquals, Contains, StartsWith, EndsWith, Regex.
- Seeds must be idempotent (re-running doesn't duplicate).

## Property contract (freeze in Phase 2 — parser + rules both depend on these names)

`PropertyDefinition.Name` values are the exact keys the parser emits in
`EntityMetadata.Properties`. For the entity types the MVP actually parses, seed:

```
File:     Name (String), Extension (String), Path (String)
Class:    Name (String)
Method:   Name (String), ReturnType (String)
Variable: Name (String), Scope (String), DataType (String)
```

`Variable.Scope` ∈ { Local, Parameter, Field } (normalized across C#/VB).
`*.DataType` and `Method.ReturnType` use the normalized type vocabulary chosen in
Phase 5 (apply the same mapping in both parsers).
