# Phase 4 — ZIP upload & cataloging

> Follow CLAUDE.md. Read `docs/schema.md`.

**Goal:** accept a ZIP, extract safely, catalog files. No parsing, no rules.

## Do

- Create **AnalysisRun**, **FileCatalog** (types per schema).
- Upload page (ZIP only). On upload: create `AnalysisRun` (Status=Running), extract,
  walk files, write `FileCatalog` rows (Path relative to zip root, Extension, Size,
  SHA-256 lowercase-hex Hash), set Status=Completed (or Failed + FinishedDate on error).
- Use `System.IO.Compression.ZipArchive` and `System.Security.Cryptography.SHA256`.
- Extract to a per-run temp dir; clean it up after cataloging.
- UI: Analysis Runs list; Catalog view (files for a run).

## Security (mandatory)

- **Zip-slip guard:** reject/skip any entry whose resolved path escapes the
  extraction root.
- **Decompression-bomb guard:** enforce max entry count, max total uncompressed
  size, max single-file size. Reject the upload with a clear error if exceeded.

## Tests

- Zip extraction (known zip → expected files).
- A zip-slip entry is rejected.
- Hash matches a known SHA-256.
- Catalog rows created with correct Size/Extension/Path.

## Acceptance

- [ ] Valid ZIP uploads, extracts, catalogs; bad/oversized/zip-slip ZIPs rejected
      safely (no traversal, no resource exhaustion).
- [ ] Tests pass. No parser or rule execution.
