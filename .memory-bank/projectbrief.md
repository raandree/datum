# Project brief

## Purpose

Datum is a PowerShell module for aggregating DSC (Desired State Configuration) configuration data from multiple hierarchical sources. It enables policy-driven infrastructure management by organizing configuration data in customizable layers with override capabilities — following the DRY principle for DSC configuration data.

## Scope

- In scope: Hierarchical data resolution, merge strategies, data handlers, file provider, RSOP generation, PowerShell 5.1/7 support
- Out of scope: GUI tooling, non-PowerShell runtimes, bundled external handler modules

## Stakeholders

- Author: Gael Colas (gaelcolas), SynEdgy Limited
- Users: Infrastructure teams managing Windows servers with DSC, DevOps engineers, configuration management specialists

## Acceptance criteria

1. Core hierarchy resolution works with configurable layers and merge strategies
2. File provider supports YAML, JSON, and PSD1 formats
3. Data handlers are extensible via external modules (ProtectedData, InvokeCommand)
4. Module works on both PowerShell 5.1 and PowerShell 7
5. Published to PowerShell Gallery with semantic versioning
