# Tech context

## Stack

- PowerShell 5.1+ / PowerShell 7 (PSCore)
- Script module with classes (PSM1 + PSD1)
- Data formats: YAML (primary), JSON, PSD1
- Build system: Sampler (InvokeBuild-based)
- Testing: Pester 5
- CI/CD: Azure DevOps Pipelines
- Versioning: GitVersion (Semantic Versioning)
- Distribution: PowerShell Gallery

## Environment

- Runtime dependency: `powershell-yaml`
- Optional: `Datum.ProtectedData`, `Datum.InvokeCommand`
- Build deps: InvokeBuild, ModuleBuilder, Sampler 0.119.1, PSScriptAnalyzer
- Output: `output/datum/<version>/`

## Constraints

- Must work on both Windows PowerShell 5.1 and PowerShell 7
- No binary dependencies (pure PowerShell)
- External data handler modules are optional, not bundled
- File provider is the only built-in store provider
- Class loading order matters: `1.DatumProvider.ps1` before `FileProvider.ps1`

## Validation

- `./build.ps1 -AutoRestore -Tasks test` — run Pester tests
- `./build.ps1 -AutoRestore` — full build
- Run build in separate process to avoid VS Code hanging
- Current test results: all passed, 0 failed, 3 skipped (known merge logic bug)
