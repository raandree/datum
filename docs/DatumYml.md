# Datum.yml Configuration Reference

The `Datum.yml` file defines the structure, behaviour, and extensions of a Datum hierarchy. It is the central configuration file, placed at the root of your configuration data tree.

## Complete Example

```yaml
DatumStructure:
  - StoreName: AllNodes
    StoreProvider: Datum::File
    StoreOptions:
      Path: ./AllNodes

  - StoreName: Roles
    StoreProvider: Datum::File
    StoreOptions:
      Path: ./Roles

  - StoreName: Environments
    StoreProvider: Datum::File
    StoreOptions:
      Path: ./Environments

  - StoreName: SiteData
    StoreProvider: Datum::File
    StoreOptions:
      Path: ./SiteData

ResolutionPrecedence:
  - 'AllNodes\$($Node.Environment)\$($Node.Name)'
  - 'AllNodes\$($Node.Environment)\All'
  - 'Environments\$($Node.Environment)'
  - 'SiteData\$($Node.Location)'
  - 'Roles\$($Node.Role)'
  - 'Roles\All'

default_lookup_options: MostSpecific

lookup_options:
  Configurations: Unique
  NetworkConfig: hash
  SoftwareBaseline: deep
  SoftwareBaseline\Packages:
    merge_hash_array: DeepTuple
    merge_options:
      tuple_keys:
        - Name

DatumHandlers:
  Datum::TestHandler:
    CommandOptions:
      Password: P@ssw0rd
      Test: test

  Datum.ProtectedData::ProtectedDatum:
    CommandOptions:
      PlainTextPassword: true

  Datum.InvokeCommand::InvokeCommand:
    SkipDuringLoad: true
```

## Sections

### DatumStructure

Defines the root branches (stores) of the Datum tree. Each entry creates a top-level key in the Datum object.

```yaml
DatumStructure:
  - StoreName: <name>
    StoreProvider: <provider>
    StoreOptions:
      <provider-specific options>
```

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `StoreName` | string | Yes | Name of the root branch. Becomes a top-level key in `$Datum`. |
| `StoreProvider` | string | Yes | Provider to use. Built-in: `Datum::File`. |
| `StoreOptions` | hashtable | Yes | Options passed to the provider. |

#### Built-In File Provider Options

| Key | Type | Required | Default | Description |
|-----|------|----------|---------|-------------|
| `Path` | string | Yes | — | Relative or absolute path to the data directory. |

#### Multiple Stores

You can define multiple stores, each becoming a separate root branch:

```yaml
DatumStructure:
  - StoreName: AllNodes
    StoreProvider: Datum::File
    StoreOptions:
      Path: ./AllNodes
  - StoreName: Roles
    StoreProvider: Datum::File
    StoreOptions:
      Path: ./Roles
```

Access: `$Datum.AllNodes`, `$Datum.Roles`

#### Custom Store Providers

External store providers can be created as PowerShell modules. The `StoreProvider` value format is `<ModuleName>::<ProviderName>`, which maps to a `New-Datum<ProviderName>Provider` function in the specified module.

---

### ResolutionPrecedence

An ordered list of path prefixes, from **most specific** to **most generic**. When performing a lookup, Datum tries each path and returns or merges the values found.

```yaml
ResolutionPrecedence:
  - 'AllNodes\$($Node.Environment)\$($Node.Name)'
  - 'AllNodes\$($Node.Environment)\All'
  - 'Environments\$($Node.Environment)'
  - 'SiteData\$($Node.Location)'
  - 'Roles\$($Node.Role)'
  - 'Roles\All'
```

#### Variable Substitution

Paths support PowerShell variable substitution using `$()` syntax. The `$Node` variable is the most commonly used, referring to the current node's metadata:

| Expression | Resolves To |
|-----------|-------------|
| `$($Node.Name)` | The node's name |
| `$($Node.Environment)` | The node's environment |
| `$($Node.Location)` | The node's location |
| `$($Node.Role)` | The node's role |

Any property of the node hashtable can be referenced.

#### Path Format

- Use **backslash** (`\`) as the path separator
- The first segment must match a `StoreName` from `DatumStructure`
- Subsequent segments map to directories or files within the store

#### Resolution Example

For a node with `Name = 'SRV01'`, `Environment = 'DEV'`, `Location = 'London'`, `Role = 'WebServer'`:

```
AllNodes\DEV\SRV01        →  $Datum.AllNodes.DEV.SRV01.<PropertyPath>
AllNodes\DEV\All          →  $Datum.AllNodes.DEV.All.<PropertyPath>
Environments\DEV          →  $Datum.Environments.DEV.<PropertyPath>
SiteData\London           →  $Datum.SiteData.London.<PropertyPath>
Roles\WebServer           →  $Datum.Roles.WebServer.<PropertyPath>
Roles\All                 →  $Datum.Roles.All.<PropertyPath>
```

---

### default_lookup_options

Sets the default merge strategy applied to all lookups. Can be a preset name or a detailed strategy hashtable.

```yaml
# Simple preset
default_lookup_options: MostSpecific

# Detailed configuration
default_lookup_options:
  merge_hash: deep
  merge_basetype_array: Unique
  merge_hash_array: DeepTuple
  merge_options:
    knockout_prefix: '--'
```

#### Available Presets

| Preset | Aliases | Description |
|--------|---------|-------------|
| `MostSpecific` | `First` | Return the first value found (no merge) |
| `hash` | `MergeTopKeys` | Merge top-level hashtable keys |
| `deep` | `MergeRecursively` | Recursively merge all nested structures |

See [Merging Strategies](Merging.md) for complete documentation.

---

### lookup_options

Per-key merge strategy overrides. Keys can be exact property paths or regex patterns.

```yaml
lookup_options:
  # Simple preset for a key
  Configurations: Unique

  # Detailed strategy for a key
  SoftwareBaseline\Packages:
    merge_hash_array: DeepTuple
    merge_options:
      tuple_keys:
        - Name
        - Version

  # Regex pattern (starts with ^)
  ^LCM_Config\\.*: deep
```

#### Key Matching

| Pattern Type | Example | Description |
|-------------|---------|-------------|
| Exact match | `Configurations` | Matches only the `Configurations` key |
| Nested path | `SoftwareBaseline\Packages` | Matches the `Packages` sub-key under `SoftwareBaseline` |
| Regex | `^Security\\.*` | Matches any key under `Security` |

Exact matches always take priority over regex matches.

#### Strategy Properties

| Property | Values | Description |
|----------|--------|-------------|
| `merge_hash` | `MostSpecific`, `hash`, `deep` | How to merge hashtables |
| `merge_basetype_array` | `MostSpecific`, `Unique`, `Sum` | How to merge scalar arrays |
| `merge_hash_array` | `MostSpecific`, `Sum`, `UniqueKeyValTuples`, `DeepTuple` | How to merge hashtable arrays |
| `merge_options` | hashtable | Additional options (see below) |

#### merge_options

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `knockout_prefix` | string | `--` (for hash/deep presets) | Prefix to remove items during merge |
| `tuple_keys` | string[] | — | Keys used to match items in hash arrays |

---

### DatumHandlers

Registers value transformation handlers that process values at lookup time.

```yaml
DatumHandlers:
  <ModuleName>::<HandlerName>:
    CommandOptions:
      <key>: <value>
    SkipDuringLoad: <bool>
```

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `CommandOptions` | hashtable | No | Parameters passed to the handler's action function |
| `SkipDuringLoad` | bool | No | If `true`, handler runs only at lookup time, not during initial file load |

#### Handler Function Naming

The key `<ModuleName>::<HandlerName>` maps to:
- Filter: `<ModuleName>\Test-<HandlerName>Filter`
- Action: `<ModuleName>\Invoke-<HandlerName>Action`

#### Common Handlers

| Handler | Key | Purpose |
|---------|-----|---------|
| Built-in Test | `Datum::TestHandler` | Demonstration/development handler |
| Protected Data | `Datum.ProtectedData::ProtectedDatum` | Decrypt `[ENC=...]` credential values |
| Invoke Command | `Datum.InvokeCommand::InvokeCommand` | Evaluate `[x= ... =]` PowerShell expressions |

See [Datum Handlers](DatumHandlers.md) for detailed documentation.

---

### Other Settings

#### DscLocalConfigurationManagerKeyName

When defined, specifies the key name within each node's data that contains LCM (Local Configuration Manager) settings. This is used during RSOP computation.

```yaml
DscLocalConfigurationManagerKeyName: LCM_Config
```

---

## File System Layout

A typical Datum configuration data tree:

```
ConfigData/
├── Datum.yml                    # This configuration file
├── AllNodes/
│   ├── DEV/
│   │   ├── SRV01.yml           # Node-specific data
│   │   ├── SRV02.yml
│   │   └── All.yml             # Shared data for all DEV nodes
│   └── PROD/
│       ├── SRV03.yml
│       └── All.yml
├── Environments/
│   ├── DEV.yml                  # Environment-level defaults
│   └── PROD.yml
├── SiteData/
│   ├── London.yml               # Location-specific data
│   └── NewYork.yml
└── Roles/
    ├── WebServer.yml            # Role definitions
    ├── FileServer.yml
    └── All.yml                  # Data shared across all roles
```

Each directory under a store becomes an intermediate node. Each `.yml`, `.json`, or `.psd1` file becomes a leaf with its parsed contents.

## See Also

- [README](../README.md) — Overview and getting started
- [Merging Strategies](Merging.md) — Complete merge behaviour reference
- [Datum Handlers](DatumHandlers.md) — Handler system documentation
- [Cmdlet Reference](CmdletReference.md) — All public function documentation
