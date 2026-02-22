# Datum Handlers

Datum Handlers extend what can be stored and resolved from data files. They intercept values at lookup time and transform them — for example, decrypting credentials or evaluating dynamic expressions.

## How Handlers Work

A handler consists of two functions in a PowerShell module:

1. **Filter function** (`Test-<HandlerName>Filter`) — Returns `$true` if the handler should process the value
2. **Action function** (`Invoke-<HandlerName>Action`) — Transforms and returns the new value

When Datum resolves a value, it pipes the value through each registered handler's filter. If the filter matches, the action function is called and the returned value replaces the original.

Handlers are declared in the `DatumHandlers` section of `Datum.yml`:

```yaml
DatumHandlers:
  <ModuleName>::<HandlerName>:
    CommandOptions:
      Param1: Value1
      Param2: Value2
```

This instructs Datum to:
1. Call `<ModuleName>\Test-<HandlerName>Filter` with the value
2. If it returns `$true`, call `<ModuleName>\Invoke-<HandlerName>Action`

## Action Function Parameters

Datum automatically populates parameters of the action function from available variables. The following are typically available:

| Parameter | Description |
|-----------|-------------|
| `$InputObject` | The raw value being processed |
| `$Datum` | The full Datum tree |
| `$Node` | The current node (if in a node context) |
| `$PropertyPath` | The property path being looked up |
| Any `CommandOptions` key | Values from the Datum.yml configuration |

You do not need to pass these explicitly — Datum matches parameter names to available variables and `CommandOptions` keys.

## Built-In Test Handler

Datum includes a test handler that demonstrates the pattern. It matches values like `[TEST=<data>]`.

### Configuration

```yaml
DatumHandlers:
  Datum::TestHandler:
    CommandOptions:
      Password: P@ssw0rd
      Test: test
```

### Filter

The filter uses a regex match:

```powershell
function Test-TestHandlerFilter {
    param($InputObject)
    $InputObject -is [string] -and
    $InputObject -match '^\[TEST=(?<data>[\w\W])*\]$'
}
```

### Action

The action function receives parameters from both the available variables and `CommandOptions`:

```powershell
function Invoke-TestHandlerAction {
    param(
        $Password,  # from CommandOptions
        $Test,       # from CommandOptions
        $Datum       # from Datum context
    )
    # Returns diagnostic information about what was received
    @"
    Action: $handler
    Node: $($Node | Format-List * | Out-String)
    Params:
    $($PSBoundParameters | ConvertTo-Json)
"@
}
```

### Using the Test Handler for Redirections

You can modify the test handler to follow references to other Datum paths:

```powershell
function Invoke-TestHandlerAction {
    param(
        $Datum,
        $InputObject
    )

    $datumLink = [regex]::Matches(
        $InputObject,
        '^\[TEST=(?<data>[\w\W]*)\]$'
    )[0].groups['data'].value -split '\\'

    [scriptblock]::Create(
        "`$Datum.$($datumLink -join '.')"
    ).Invoke()
}
```

When a value is `'[TEST=Roles\Role1\Shared1\DestinationPath]'`, this resolves and returns `$Datum.Roles.Role1.Shared1.DestinationPath`.

## Datum.ProtectedData — Encrypted Credentials

The [Datum.ProtectedData](https://www.powershellgallery.com/packages/Datum.ProtectedData) module stores encrypted `[PSCredential]` objects in YAML files.

### Installation

```powershell
Install-Module -Name Datum.ProtectedData -Scope CurrentUser
```

### Configuration

```yaml
DatumHandlers:
  Datum.ProtectedData::ProtectedDatum:
    CommandOptions:
      PlainTextPassword: true
      # Certificate or other encryption key options
```

### Usage

Encrypted values in data files are prefixed with `[ENC=`:

```yaml
# In a data file
AdminCredential: '[ENC=PE9ianM... (encrypted blob) ...=]'
```

When Datum resolves this value, the `ProtectedDatum` handler:

1. Detects the `[ENC=` prefix via its filter
2. Decrypts the blob using the configured certificate or key
3. Returns a `[PSCredential]` object

This allows credentials to be stored securely in version control while being transparently decrypted at lookup time.

## Datum.InvokeCommand — Dynamic Expressions

The [Datum.InvokeCommand](https://www.powershellgallery.com/packages/Datum.InvokeCommand) module enables PowerShell expressions to be evaluated dynamically at lookup time.

### Installation

```powershell
Install-Module -Name Datum.InvokeCommand -Scope CurrentUser
```

### Configuration

```yaml
DatumHandlers:
  Datum.InvokeCommand::InvokeCommand:
    SkipDuringLoad: true
```

> **Important:** The `SkipDuringLoad: true` setting ensures expressions are only evaluated during lookup, not when the data file is first loaded into memory.

### Usage

Wrap PowerShell expressions in `[x= ... =]`:

```yaml
# In a data file
CurrentDate: '[x= { Get-Date -Format "yyyy-MM-dd" } =]'
ComputerName: '[x= { $env:COMPUTERNAME } =]'
DynamicPath: '[x= { Join-Path $env:ProgramFiles "MyApp" } =]'
```

When resolved, the scriptblock is executed and the result replaces the marker.

### Known Limitations

- Expressions are evaluated in the current PowerShell session context
- Complex expressions with nested quotes may need careful escaping
- Error handling within expressions is the responsibility of the expression author

## Building Custom Handlers

### Step 1: Create a Module

Create a PowerShell module with two exported functions:

```powershell
# MyDatumHandler.psm1

function Test-MyHandlerFilter {
    param($InputObject)

    # Return $true if this handler should process the value
    $InputObject -is [string] -and
    $InputObject -match '^\[MYPREFIX=.*\]$'
}

function Invoke-MyHandlerAction {
    param(
        $InputObject,
        $Datum,
        $Node,
        $MyConfigParam  # from CommandOptions
    )

    # Extract the data from the marker
    $data = [regex]::Match($InputObject, '^\[MYPREFIX=(?<data>.*)\]$').Groups['data'].Value

    # Transform and return the value
    # ... your transformation logic here ...
    return $transformedValue
}
```

### Step 2: Make the Module Available

Ensure the module is in a path listed in `$env:PSModulePath` or install it from a repository.

### Step 3: Register in Datum.yml

```yaml
DatumHandlers:
  MyDatumHandler::MyHandler:
    CommandOptions:
      MyConfigParam: SomeValue
```

### Naming Convention

The `DatumHandlers` key format is `<ModuleName>::<HandlerName>`:

- Datum calls `<ModuleName>\Test-<HandlerName>Filter` for the filter
- Datum calls `<ModuleName>\Invoke-<HandlerName>Action` for the action

### Tips

- Keep filter functions fast — they run on every resolved value
- Use specific prefixes (e.g. `[ENC=`, `[x=`) to avoid false matches
- Test handlers thoroughly — a handler bug affects all data resolution
- The `SkipDuringLoad` option (when supported) defers handler execution to lookup time

## See Also

- [Datum.yml Reference](DatumYml.md) — Full configuration file reference
- [README - Datum Handlers](../README.md#datum-handlers)
- [Datum.ProtectedData on PowerShell Gallery](https://www.powershellgallery.com/packages/Datum.ProtectedData)
- [Datum.InvokeCommand on PowerShell Gallery](https://www.powershellgallery.com/packages/Datum.InvokeCommand)
