# Active context

## Current focus

Migrating Memory Bank from legacy `memory-bank/` directory to canonical `.memory-bank/` structure with index-based routing.

## Known work items

- **Documentation rework needed** — existing docs are outdated or incomplete and need a thorough rework.

## Evidence

- Old `memory-bank/` directory contains 6 files (no index.md, no promptHistory.md)
- New `.memory-bank/` directory exists but is empty
- Project has unreleased features: conditional ResolutionPrecedence, knockout support, Pester 5 migration, multiple bug fixes

## Next step

Complete migration, verify health, then remove old `memory-bank/` directory.
