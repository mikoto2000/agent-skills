---
name: powershell
description: >
  Use this skill when working in PowerShell on Windows or cross-platform PowerShell.
  It provides guidance for correct PowerShell syntax, HTTP requests, JSON handling,
  native command invocation, quoting, pipelines, error handling, filesystem operations,
  and version-specific behavior. Prefer PowerShell-native semantics over Bash or POSIX
  shell assumptions, and verify uncertain command behavior with Get-Command and Get-Help.
enabled: true
---

# PowerShell Skill

## Purpose

Use PowerShell according to PowerShell semantics rather than translating Bash, POSIX shell, or `cmd.exe` patterns mechanically.

PowerShell is an object-oriented shell. Pipelines usually pass .NET objects rather than plain text, and cmdlets have their own parameter binding, error handling, quoting, and output behavior.

When executing commands, prioritize correctness and observability over terseness.

## Core Rules

- Do not assume Bash syntax or behavior applies to PowerShell.
- Do not invent cmdlet parameters.
- Do not guess about uncertain PowerShell syntax when the environment can be inspected.
- Prefer PowerShell cmdlets when they provide the required functionality cleanly.
- Use native executables when they are the better tool, but treat their invocation and error handling differently from cmdlets.
- Prefer readable multi-step PowerShell over fragile one-liners.
- Account for differences between Windows PowerShell 5.1 and PowerShell 7+ when relevant.
- When the PowerShell version is unknown and the distinction matters, inspect `$PSVersionTable`.

## Inspect Before Guessing

PowerShell is highly introspectable. Use that capability instead of hallucinating syntax.

When uncertain about a command:

```powershell
Get-Command <command>
Get-Help <command> -Full
Get-Help <command> -Examples
```

To inspect parameters:

```powershell
(Get-Command Invoke-WebRequest).Parameters.Keys
```

To inspect the PowerShell version:

```powershell
$PSVersionTable
```

To inspect aliases:

```powershell
Get-Alias
Get-Alias curl -ErrorAction SilentlyContinue
Get-Command curl -All
```

If command behavior can be verified locally, verify it before proposing speculative syntax.

## PowerShell Pipelines

PowerShell pipelines generally pass objects, not lines of text.

Prefer:

```powershell
Get-Process |
    Where-Object CPU -gt 100 |
    Sort-Object CPU -Descending |
    Select-Object -First 10 Name, CPU
```

Do not unnecessarily convert structured objects into text and then parse them again.

Avoid patterns such as:

```powershell
Get-Process | Out-String | Select-String ...
```

when the same operation can be performed using object properties.

Common pipeline cmdlets include:

- `Where-Object`
- `ForEach-Object`
- `Select-Object`
- `Sort-Object`
- `Group-Object`
- `Measure-Object`

When accessing object properties, inspect the object if necessary:

```powershell
$result | Get-Member
$result | Format-List *
```

Do not use `Format-Table` or `Format-List` in the middle of a data-processing pipeline. Formatting cmdlets are intended primarily for final presentation.

## HTTP Requests

Understand the distinction between `Invoke-RestMethod` and `Invoke-WebRequest`.

### Prefer Invoke-RestMethod for APIs

Use `Invoke-RestMethod` when interacting with JSON or REST-style APIs.

Example:

```powershell
$response = Invoke-RestMethod `
    -Uri 'https://example.com/api/items' `
    -Method Get
```

JSON responses are normally deserialized into PowerShell objects automatically.

You can then use normal PowerShell property access:

```powershell
$response.items
$response.items | Select-Object id, name
```

### Use Invoke-WebRequest for Raw HTTP Responses

Use `Invoke-WebRequest` when HTTP-level response information is important, such as:

- status codes
- headers
- raw response content
- downloading arbitrary content
- inspecting web responses rather than primarily consuming a JSON API

Example:

```powershell
$response = Invoke-WebRequest `
    -Uri 'https://example.com/' `
    -Method Get

$response.StatusCode
$response.Headers
$response.Content
```

Do not treat `Invoke-WebRequest` as merely PowerShell's spelling of `curl`.

## Sending JSON

When sending JSON, explicitly serialize PowerShell objects.

Prefer:

```powershell
$body = @{
    name    = 'example'
    enabled = $true
}

$response = Invoke-RestMethod `
    -Uri 'https://example.com/api/items' `
    -Method Post `
    -ContentType 'application/json' `
    -Body ($body | ConvertTo-Json)
```

For nested structures, consider an explicit depth:

```powershell
$json = $body | ConvertTo-Json -Depth 10
```

Do not manually construct JSON strings unless there is a specific reason.

Avoid:

```powershell
$body = '{"name":"' + $name + '"}'
```

Prefer creating PowerShell objects and using `ConvertTo-Json`.

## HTTP Headers

Represent headers as a hashtable:

```powershell
$headers = @{
    Authorization = "Bearer $token"
    Accept        = 'application/json'
}

Invoke-RestMethod `
    -Uri $uri `
    -Headers $headers `
    -Method Get
```

Do not translate curl's `-H` syntax directly into PowerShell.

For example, do not write:

```powershell
Invoke-RestMethod -H 'Authorization: Bearer ...'
```

unless inspection confirms that such a parameter actually exists for the command being invoked.

## curl on Windows

Be careful with the command name `curl`.

Historically, Windows PowerShell may expose `curl` as an alias for `Invoke-WebRequest`.

Do not assume:

```powershell
curl ...
```

means the native curl executable.

Inspect it if necessary:

```powershell
Get-Command curl -All
```

When specifically intending to execute native curl on Windows, prefer:

```powershell
curl.exe
```

Example:

```powershell
curl.exe -X POST `
    -H "Content-Type: application/json" `
    -d '{"name":"example"}' `
    https://example.com/api/items
```

Do not mix `curl.exe` options with `Invoke-WebRequest` parameters.

They are different programs with different semantics.

## HTTP Error Handling

For requests where failures matter, use explicit error handling.

Example:

```powershell
try {
    $response = Invoke-RestMethod `
        -Uri $uri `
        -Method Get `
        -ErrorAction Stop
}
catch {
    Write-Error "HTTP request failed: $($_.Exception.Message)"
    throw
}
```

PowerShell versions differ in the HTTP-specific information exposed by exceptions.

Do not assume a particular exception property exists without considering the target PowerShell version.

When debugging, inspect the exception:

```powershell
$_.Exception | Format-List *
```

If running on a known modern PowerShell version, use version-appropriate response information where available.

## Native Commands vs Cmdlets

PowerShell cmdlets and native executables have different error semantics.

For cmdlets:

```powershell
try {
    Get-Item 'C:\does-not-exist' -ErrorAction Stop
}
catch {
    Write-Error $_
}
```

For native executables, inspect the process exit code:

```powershell
git status

if ($LASTEXITCODE -ne 0) {
    throw "git status failed with exit code $LASTEXITCODE"
}
```

Do not use `$LASTEXITCODE` as the primary success indicator for PowerShell cmdlets.

Do not assume `$?` and `$LASTEXITCODE` mean the same thing.

- `$?` describes whether the previous PowerShell operation succeeded.
- `$LASTEXITCODE` contains the exit code of the most recently executed native program or script that sets it.

When correctness matters, test the appropriate mechanism explicitly.

## Native Command Arguments

Do not assume Bash quoting rules apply to native applications launched from PowerShell.

Prefer straightforward argument construction.

For dynamic argument lists:

```powershell
$args = @(
    'commit'
    '-m'
    $message
)

git @args
```

This is generally clearer than constructing a command line as one giant string.

Avoid:

```powershell
$command = "git commit -m `"$message`""
Invoke-Expression $command
```

Do not use `Invoke-Expression` for ordinary command execution.

It introduces quoting problems and unnecessary code-injection risk.

## String Quoting

PowerShell has meaningful differences between single-quoted and double-quoted strings.

Single quotes do not normally expand variables:

```powershell
$name = 'Alice'

'Hello $name'
```

Result:

```text
Hello $name
```

Double quotes expand variables:

```powershell
"Hello $name"
```

Result:

```text
Hello Alice
```

For expressions inside interpolated strings, use `$()`:

```powershell
"Current PID: $($PID)"
```

Prefer single-quoted strings when interpolation is not needed.

## Variables Adjacent to Text

When variable names are immediately followed by characters that could be interpreted as part of the variable name, delimit the variable explicitly.

Prefer:

```powershell
"${name}_suffix"
```

or:

```powershell
"$($name)_suffix"
```

## Paths

Prefer PowerShell path cmdlets and `Join-Path` when constructing paths programmatically.

Example:

```powershell
$configPath = Join-Path $HOME '.config'
$filePath = Join-Path $configPath 'settings.json'
```

Do not concatenate path separators manually unless the path is intentionally literal.

Use:

```powershell
Test-Path $filePath
Get-Item $filePath
Resolve-Path $filePath
```

when appropriate.

PowerShell normally accepts `/` in many filesystem contexts, but do not assume every native Windows application handles paths the same way.

## Files and Text Encoding

Be explicit about text encoding when the encoding matters.

Example:

```powershell
Set-Content `
    -Path $path `
    -Value $content `
    -Encoding utf8
```

PowerShell's encoding defaults differ across versions, particularly between Windows PowerShell 5.1 and PowerShell 7+.

If files are consumed by another application or protocol, choose the encoding deliberately.

## Reading Files

For normal text:

```powershell
$content = Get-Content -Raw -Path $path
```

Use `-Raw` when the whole file should be treated as one string.

Without `-Raw`, `Get-Content` normally emits lines individually into the pipeline.

For JSON:

```powershell
$config = Get-Content -Raw -Path $path | ConvertFrom-Json
```

For CSV:

```powershell
$rows = Import-Csv -Path $path
```

Prefer structured parsers over manually splitting text.

## JSON

Convert objects to JSON:

```powershell
$json = $object | ConvertTo-Json -Depth 10
```

Convert JSON to PowerShell objects:

```powershell
$object = $json | ConvertFrom-Json
```

Do not assume the default `ConvertTo-Json` depth is sufficient for deeply nested API payloads.

If nested data may exist, explicitly choose an appropriate `-Depth`.

## Hashtable vs PSCustomObject

Use hashtables conveniently for constructing parameter sets and JSON payloads:

```powershell
$payload = @{
    user = @{
        name = 'Alice'
    }
}
```

Use `[PSCustomObject]` when an explicit structured object is useful:

```powershell
$item = [PSCustomObject]@{
    Name  = 'example'
    Value = 42
}
```

Do not overcomplicate simple data structures with unnecessary .NET types.

## Error Handling

Understand terminating and non-terminating errors.

A `try`/`catch` block does not automatically catch every PowerShell error.

When an operation must fail into `catch`, use:

```powershell
-ErrorAction Stop
```

Example:

```powershell
try {
    Remove-Item $path -ErrorAction Stop
}
catch {
    Write-Error "Failed to remove '$path': $($_.Exception.Message)"
}
```

Do not add `-ErrorAction Stop` blindly everywhere, but use it deliberately where failure must interrupt control flow.

## Avoid Silently Ignoring Errors

Do not use:

```powershell
-ErrorAction SilentlyContinue
```

merely to make a failing command appear successful.

Use it only when absence or failure is expected and intentionally handled.

For example:

```powershell
$command = Get-Command foo -ErrorAction SilentlyContinue

if ($null -eq $command) {
    Write-Verbose 'foo is not installed'
}
```

## Null Checks

Prefer explicit null checks:

```powershell
if ($null -eq $value) {
    ...
}
```

Putting `$null` on the left avoids surprising behavior when `$value` is an array.

Do not assume truthiness checks distinguish all cases:

```powershell
if ($value) {
    ...
}
```

because values such as `0`, `''`, `$false`, empty collections, and `$null` may require different treatment depending on intent.

## Collections

Create arrays explicitly when useful:

```powershell
$items = @(
    'a'
    'b'
    'c'
)
```

Iterate:

```powershell
foreach ($item in $items) {
    Write-Output $item
}
```

Use `ForEach-Object` primarily when pipeline processing is desirable:

```powershell
$items | ForEach-Object {
    $_.ToUpperInvariant()
}
```

Do not use pipeline constructs merely because they are shorter if a normal `foreach` loop is clearer.

## Boolean Operators

PowerShell boolean operators are:

```powershell
-and
-or
-not
-xor
```

Examples:

```powershell
if ($enabled -and $ready) {
    ...
}
```

Do not write Bash-style:

```text
&&
||
```

without considering the PowerShell version and intended semantics.

PowerShell 7 supports pipeline chain operators `&&` and `||`, but Windows PowerShell 5.1 does not.

If compatibility with Windows PowerShell 5.1 matters, avoid them.

## Comparison Operators

Use PowerShell comparison operators:

```powershell
-eq
-ne
-lt
-le
-gt
-ge
-like
-notlike
-match
-notmatch
-contains
-notcontains
-in
-notin
```

Examples:

```powershell
if ($status -eq 'ready') {
    ...
}

if ($name -like '*.json') {
    ...
}
```

Do not assume shell operators such as `==` behave as they do in Bash or general-purpose programming languages.

## Command Continuation

Prefer natural PowerShell continuation through incomplete syntax:

```powershell
Invoke-RestMethod `
    -Uri $uri `
    -Method Post
```

The backtick can be used for explicit continuation, but it is fragile because trailing whitespace after a backtick breaks continuation.

When possible, use syntax that naturally spans lines:

```powershell
$params = @{
    Uri         = $uri
    Method      = 'Post'
    ContentType = 'application/json'
    Body        = $json
}

Invoke-RestMethod @params
```

For commands with many parameters, splatting is generally preferable to long backtick-heavy commands.

## Splatting

Use splatting for readable dynamic command invocation.

Example:

```powershell
$params = @{
    Uri         = 'https://example.com/api/items'
    Method      = 'Post'
    ContentType = 'application/json'
    Body        = $body | ConvertTo-Json
}

$response = Invoke-RestMethod @params
```

Conditional parameters can be added cleanly:

```powershell
if ($token) {
    $params.Headers = @{
        Authorization = "Bearer $token"
    }
}
```

Prefer this over repeatedly concatenating command strings.

## Environment Variables

Read environment variables through `$env:`:

```powershell
$env:PATH
$env:HOME
```

Set for the current process:

```powershell
$env:MY_VARIABLE = 'value'
```

Do not use Bash syntax:

```text
export MY_VARIABLE=value
```

when generating PowerShell commands.

## Command Existence Checks

Before assuming an optional executable exists:

```powershell
$git = Get-Command git -ErrorAction SilentlyContinue

if ($null -eq $git) {
    throw 'git is not installed or is not available on PATH.'
}
```

Do not rely on `which`.

Use `Get-Command`.

## Process Execution

For ordinary native command execution, invoke the executable directly:

```powershell
git status
```

For more control over process creation, use `Start-Process`.

Example:

```powershell
$process = Start-Process `
    -FilePath 'example.exe' `
    -ArgumentList @('--foo', 'bar') `
    -Wait `
    -PassThru

if ($process.ExitCode -ne 0) {
    throw "example.exe failed with exit code $($process.ExitCode)"
}
```

Do not reach for `Start-Process` when direct invocation provides better stdout/stderr pipeline behavior.

## External Tools Producing JSON

When a native command can emit JSON, prefer consuming that structured output.

Example:

```powershell
$json = some-tool --output json

if ($LASTEXITCODE -ne 0) {
    throw 'some-tool failed'
}

$data = $json | ConvertFrom-Json
```

This is generally more robust than parsing human-oriented console tables.

## Do Not Parse Display Output Without Need

Avoid brittle patterns such as:

```powershell
git status | Select-String ...
```

when the tool offers machine-readable output appropriate for the task.

Prefer stable structured or porcelain interfaces where available.

## Windows PowerShell 5.1 vs PowerShell 7+

Do not silently assume they are equivalent.

Relevant differences may include:

- availability of `&&` and `||`
- HTTP cmdlet behavior
- native argument passing
- default text encodings
- available parameters
- JSON behavior and supported .NET APIs
- platform-specific cmdlets

Inspect:

```powershell
$PSVersionTable.PSVersion
$PSVersionTable.PSEdition
```

When producing reusable scripts, state the minimum PowerShell version if version-specific features are used.

## Destructive Operations

Before destructive filesystem or system operations:

- verify the target value is not null or unexpectedly broad
- resolve or inspect the target when practical
- avoid wildcard deletion unless explicitly intended
- prefer `-LiteralPath` when paths may contain wildcard characters
- use `-WhatIf` when supported and appropriate during validation

Example:

```powershell
if ([string]::IsNullOrWhiteSpace($target)) {
    throw 'Target path is empty.'
}

Remove-Item -LiteralPath $target -Recurse -WhatIf
```

Do not run broad destructive commands based on an unvalidated interpolated variable.

## Prefer LiteralPath for Literal User Paths

When a filesystem path should be interpreted literally, especially if it may contain `[` or `]`, consider:

```powershell
Get-Item -LiteralPath $path
Remove-Item -LiteralPath $path
```

Use `-Path` when wildcard behavior is intentionally desired.

## Scripts

For nontrivial scripts, use normal PowerShell script structure rather than compressing everything into a single command.

Example:

```powershell
param(
    [Parameter(Mandatory)]
    [string] $Uri
)

$ErrorActionPreference = 'Stop'

try {
    $result = Invoke-RestMethod -Uri $Uri -Method Get
    $result
}
catch {
    Write-Error "Request failed: $($_.Exception.Message)"
    exit 1
}
```

Use parameter validation where appropriate.

Example:

```powershell
param(
    [Parameter(Mandatory)]
    [ValidateNotNullOrEmpty()]
    [string] $Name
)
```

## Output Streams

PowerShell has multiple output streams.

Be aware of:

- success/output
- error
- warning
- verbose
- debug
- information

For normal pipeline output:

```powershell
Write-Output $value
```

or simply:

```powershell
$value
```

For diagnostics:

```powershell
Write-Verbose 'Detailed diagnostic message'
Write-Warning 'Potential problem'
Write-Error 'Operation failed'
```

Do not use `Write-Host` as a replacement for normal pipeline output when downstream processing may be expected.

## Common Bash-to-PowerShell Mistakes

Do not generate these blindly:

```text
export FOO=bar
which command
grep
sed
awk
cat file
rm -rf
cp -r
mv
touch
source file
VAR=value command
$(...)
```

PowerShell may have aliases or superficially similar features, but their behavior is not necessarily equivalent.

Typical PowerShell-native forms include:

```powershell
$env:FOO = 'bar'
Get-Command command
Select-String
Get-Content
Remove-Item -Recurse -Force
Copy-Item -Recurse
Move-Item
New-Item
. .\script.ps1
```

Do not mechanically translate every Unix command, either. If a native tool is already available and is the clearest solution, it may be appropriate to invoke that native tool directly.

The important rule is to know which execution model is being used.

## Command Construction Policy

When asked to produce or execute a PowerShell command:

1. Identify whether each command is a PowerShell cmdlet, function, alias, script, or native executable.
2. Do not mix parameter syntax between those categories.
3. Check whether PowerShell-version differences matter.
4. Prefer structured objects over text parsing.
5. Prefer splatting for complex parameter sets.
6. Avoid `Invoke-Expression`.
7. Handle native exit codes when failure matters.
8. Use `-ErrorAction Stop` where cmdlet failures must enter `catch`.
9. Validate destructive targets.
10. If syntax is uncertain and inspection is possible, use `Get-Command` or `Get-Help` before proceeding.

## HTTP Request Decision Guide

For JSON API calls:

```text
Invoke-RestMethod
```

should normally be the first choice.

For raw response inspection:

```text
Invoke-WebRequest
```

is usually appropriate.

For reproducing an existing curl command exactly, or when curl-specific behavior is required:

```text
curl.exe
```

may be preferable on Windows.

Do not automatically convert a working `curl` invocation into `Invoke-WebRequest` unless doing so provides a concrete benefit.

## Before Finishing a PowerShell Task

Verify the following where relevant:

- Is this actually PowerShell syntax?
- Have Bash assumptions leaked into the command?
- Are cmdlet parameters real?
- Is the command a cmdlet or native executable?
- Are quoting rules correct for that category?
- Is JSON serialized correctly?
- Is the appropriate HTTP cmdlet being used?
- Are HTTP or command failures observable?
- Is `$LASTEXITCODE` checked for important native commands?
- Does the command depend on PowerShell 7 features?
- Is object processing being preserved instead of converting everything to text?
- Are destructive paths validated?
- Could `Get-Command` or `Get-Help` resolve any remaining uncertainty?

If any of these are uncertain and the environment is available, inspect the environment before guessing.
