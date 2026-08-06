# Product context

## Problem

Managing DSC Configuration Data at scale requires a hierarchical system to avoid data duplication, snowflake servers, merge complexity, and policy drift. Datum provides a Hiera-like hierarchy for PowerShell DSC, enabling Roles & Profiles patterns used in mature configuration management ecosystems.

## Users

- Infrastructure teams managing Windows servers with DSC
- DevOps engineers implementing Infrastructure as Code
- Configuration management specialists migrating from Puppet/Chef/Ansible patterns
- Used in production managing hundreds of machines

## Core workflows

1. New node onboarding: Create a minimal YAML file (name, role, location) — node inherits all role policies
2. Policy updates: Change a Role definition — all implementing nodes get the update
3. Exception handling: Override specific values at node/location/environment level without touching the role
4. RSOP (Resultant Set of Policy): Use `Get-DatumRsop` to see the fully resolved configuration for any node

## Experience goals

- Simple for operators: Define a Role in YAML, assign it to nodes
- Powerful for architects: Full control over merge strategies, resolution precedence, and data handler extensibility
- Self-documenting: The data hierarchy IS the documentation of infrastructure policy
- Version-controlled: All config data lives in files (YAML/JSON/PSD1), ideal for git workflows
