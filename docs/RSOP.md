# RSOP — Resultant Set of Policy

The **Resultant Set of Policy** (RSOP) computes the fully resolved configuration data for each node in your Datum hierarchy. It applies all hierarchy layers, merge strategies, and datum handlers to produce the final, merged configuration that a node would receive.

## Overview

`Get-DatumRsop` is the primary function for computing RSOP. It:

1. Iterates over all nodes (or a filtered subset)
2. Resolves the **composition key** (default: `Configurations`) to determine which configurations apply to each node
3. For each configuration, resolves its data through the hierarchy
4. Returns the fully merged result per node

## Basic Usage

```powershell
# Load the hierarchy
$Datum = New-DatumStructure -DefinitionFile .\Datum.yml

# Build the AllNodes array from the Datum tree
$AllNodes = @(
    $Datum.AllNodes.psobject.Properties | ForEach-Object {
        $Node = $Datum.AllNodes.($_.Name)
        (@{} + $Node)
    }
)

# Compute RSOP for all nodes
$rsop = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes
```

The result is an array of hashtables, one per node, containing the fully resolved configuration data.

## Filtering Nodes

Use the `-Filter` parameter to compute RSOP for specific nodes:

```powershell
# Single node
$rsop = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes -Filter {
    $_.NodeName -eq 'DSCFile01'
}

# By environment
$rsop = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes -Filter {
    $_.Environment -eq 'DEV'
}

# By role
$rsop = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes -Filter {
    $_.Role -eq 'WebServer'
}
```

The filter scriptblock receives each node hashtable as `$_`.

## Source Tracking

The `-IncludeSource` switch adds metadata showing which data file each value originated from:

```powershell
$rsop = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes -IncludeSource
```

This is invaluable for debugging — when a value is unexpected, the source information tells you exactly which file in the hierarchy provided it.

### Example Output with Source

When `-IncludeSource` is used, each resolved value includes a `__source` key indicating the file path:

```yaml
# RSOP output for DSCWeb01
NodeName: DSCWeb01
Configurations:
  - FileDSC
  - Shared1
Shared1:
  DestinationPath: C:\MyRoleParam.txt
  Param1: This is the Role Value!
  __source: Roles/Role1.yml
```

### Removing Source Data

Use `-RemoveSource` to strip source tracking metadata from the output after it has been added:

```powershell
$rsop = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes -IncludeSource -RemoveSource
```

## Caching

RSOP results are **cached per node name** for performance. This is important when calling `Get-DatumRsop` repeatedly or when computing RSOP in a build pipeline.

### Viewing the Cache

```powershell
$cache = Get-DatumRsopCache
```

Returns the internal cache hashtable keyed by node name.

### Clearing the Cache

After modifying data files, clear the cache to get fresh results:

```powershell
Clear-DatumRsopCache
```

### Ignoring the Cache

Force recalculation without clearing the cache for other nodes:

```powershell
$rsop = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes -IgnoreCache
```

## Composition Key

The `-CompositionKey` parameter controls which key in the node's data lists the configurations to resolve. The default is `Configurations`.

```powershell
# Default (uses 'Configurations' key)
$rsop = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes

# Custom composition key
$rsop = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes -CompositionKey 'DscConfigurations'
```

The composition key's value should be an array of configuration names. Each name is then resolved through the hierarchy to get its data.

### How It Works

For a node with:

```yaml
Configurations:
  - Shared1
  - SoftwareBaseline
```

`Get-DatumRsop` will:

1. Resolve the `Configurations` key → `['Shared1', 'SoftwareBaseline']`
2. For each configuration name, resolve its data through the hierarchy
3. Merge all resolved data into the final RSOP

## Practical Examples

### Comparing Nodes

```powershell
$Datum = New-DatumStructure -DefinitionFile .\Datum.yml
$AllNodes = @(
    $Datum.AllNodes.psobject.Properties | ForEach-Object {
        (@{} + $Datum.AllNodes.($_.Name))
    }
)

# RSOP for two different nodes
$web01 = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes -Filter { $_.NodeName -eq 'DSCWeb01' }
$web02 = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes -Filter { $_.NodeName -eq 'DSCWeb02' }

# Compare their configurations
Compare-Object $web01.Configurations $web02.Configurations
```

### Exporting RSOP to YAML

```powershell
$rsop = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes

foreach ($nodeRsop in $rsop) {
    $nodeRsop | ConvertTo-Yaml | Set-Content -Path ".\RSOP\$($nodeRsop.NodeName).yml"
}
```

### Validating Data Changes

```powershell
# Before changes
$before = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes
Clear-DatumRsopCache

# ... make data file changes ...

# After changes - reload the hierarchy
$Datum = New-DatumStructure -DefinitionFile .\Datum.yml
$after = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes

# Compare
# (use your preferred comparison method)
```

## Troubleshooting

### Stale Results

If RSOP results don't reflect recent data changes:

1. Call `Clear-DatumRsopCache`
2. Reload the Datum hierarchy with `New-DatumStructure`
3. Re-run `Get-DatumRsop`

### Unexpected Values

Use `-IncludeSource` to identify which file provides each value:

```powershell
$rsop = Get-DatumRsop -Datum $Datum -AllNodes $AllNodes -IncludeSource -Filter {
    $_.NodeName -eq 'ProblemNode'
}

# Inspect the source of a specific key
$rsop.SomeKey.__source
```

### Performance

- RSOP caching is automatic — repeated lookups for the same node are fast
- Use `-Filter` to compute RSOP only for the nodes you need
- Call `Clear-DatumRsopCache` only when data has changed

## See Also

- [README - RSOP](../README.md#rsop-resultant-set-of-policy)
- [Cmdlet Reference - Get-DatumRsop](CmdletReference.md#get-datumrsop)
- [Merging Strategies](Merging.md) — How values are merged across layers
- [Datum.yml Reference](DatumYml.md) — Hierarchy configuration
