# System patterns

## Architecture

Provider-based hierarchical data resolution with these core components:

- **DatumProvider** (base class) → **FileProvider** (built-in, filesystem-based)
- **Datum Structure** (`New-DatumStructure`): Loads hierarchy from `Datum.yml`, creates providers
- **Resolution Engine** (`Resolve-Datum`): Walks ResolutionPrecedence paths with variable substitution
- **Merge Engine** (`Merge-Datum`, `Merge-Hashtable`, `Merge-DatumArray`): Strategy-based merging
- **RSOP Engine** (`Get-DatumRsop`): Full resolved configuration with caching and source tracking
- **Data Handlers** (`Invoke-DatumHandler`, `ConvertTo-Datum`): Extensible regex-matched handlers

### Key design patterns

- Provider Pattern: Abstracted data sources behind DatumProvider interface
- Lazy Evaluation: FileProvider uses ScriptProperty for on-demand loading
- Strategy Pattern: Merge behaviour configurable per-key via lookup_options
- Variable Substitution: Resolution paths use PowerShell expression expansion
- Knockout Pattern: Items prefixed with `--` removed from merged results

### File organization

```
source/
  datum.psd1              # Module manifest
  datum.psm1              # Root module (empty — ModuleBuilder merges all)
  Classes/                # PowerShell classes (numbered for load order)
  Public/                 # Exported functions (14)
  Private/                # Internal functions (10)
  ScriptsToProcess/       # Global Resolve-NodeProperty for DSC context
tests/
  Integration/            # Functional tests (Pester 5) with 7 test data hierarchies
  QA/                     # Module quality and changelog tests
```

### Build system

- Sampler-based with 16 build tasks: Clean → Build_Module → Create_changelog → Pester tests → Coverage
- ModuleBuilder merges source files into single module output
- Azure Pipelines CI/CD, GitVersion for SemVer

## Decisions

### Decision 1: Memory Bank migrated to .memory-bank

- Choice: Use canonical `.memory-bank/` with index-based routing.
- Rationale: Align with current Memory Bank skill structure; enables routed loading and health checks.

### Decision 2: Pester 5 migration complete

- Choice: All tests use Pester 5 syntax (BeforeDiscovery/BeforeAll/It -ForEach).
- Rationale: Modern test patterns, better discovery, data-driven test cases.

### Decision 3: Knockout prefix is `--`

- Choice: Use `--` prefix to remove inherited items from merged arrays.
- Rationale: Consistent with Puppet Hiera convention.
