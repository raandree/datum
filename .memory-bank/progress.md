# Progress

## Current status

Active development with unreleased features accumulated since v0.40.1 (April 2023). Build version 0.41.0 pre-release.

## Recent milestones

- 2026-03-07: Conditional ResolutionPrecedence entries with InvokeCommand expressions
- 2026-02-25: PR #154 — JSON/YAML equivalence tests (33 tests), JSON error context
- 2026-02-23: Documentation fixes (AllNodes iteration, -IncludeSource output, mutual exclusivity)
- 2026-02: Issue #136 fixed — configurable `default_json_depth` in Datum.yml

## Stable capabilities

- Core hierarchy resolution: `New-DatumStructure`, `Resolve-Datum`, `Resolve-NodeProperty`
- File provider with YAML, JSON, PSD1 support
- Merge strategies: MostSpecific, MergeTopKeys, MergeRecursively
- Array merge: Sum/Add, Unique, DeepTuple, UniqueKeyValTuples
- RSOP generation with caching and source tracking
- Data handler system with regex-based matching
- Knockout prefix (`--`) for basetype arrays
- PowerShell 7 full compatibility

## Open work

- Release next version with all unreleased changes
- Fix merge logic bug (3 skipped tests — DSCFile01 Ethernet 3 with InvokeCommand handler)
- Code coverage improvements (threshold currently 0)
- Additional store providers beyond FileProvider (community-driven)
