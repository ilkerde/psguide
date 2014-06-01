Powershell Serious Scripting Guidelines
=======================================

_Or: How to develop solid Powershell applications in teams._

### Indent Style

**Powershell prefers spaces.** Use spaces instead of tabs. Use a consistent indent level (i.e. 4 spaces). Period.

### Brace Style

**Powershell uses K&R 1TBS bracing style.** It helps using script blocks and seamlessly integrating DSLs.

Example:

    function Hello {
        return 'Hello World'
    }

    # single line braces
    if ($writeHello) { Write-Host Hello }

    Invoke-Psake -Init {
        Write-Output 'Initializing Psake...'
    }

    task Compile -Depends Clean {
        exec { msbuild $solution }
    }

### Name Style

**Powershell is case-insensitive.** This requires a consistent casing and naming style.

* Language keywords are always lowercased.
* Only commands (cmdlets) always follow _Verb-Noun_ convention.
* Public functions are always pascal-cased.
* Public variables are always pascal-cased.
* Private functions are always camel-cased.
* Private variables are always camel-cased.
* Named parameters are always pascal-cased.
* Known type names are always lowercased.
* Functions do not use _Verb-Noun_ convention if they are not intended to be used in command line.
* Variables do not encompass type information in their name.

Example:

    function LoadConfig {
        param (
            [string] $File,
            [switch] $ValidateIfExists
        )

        function notExists {
            param (
                [string] $File
            )

            $exists = Test-Path $File
            return -not $exists
        }

        if ($ValidateIfExists -and notExists $File) {
            throw 'Configuration file not found.'
        }

        $config = Get-Content $File
        return $config
    }

### Punctuation Style

**Powershell is command-line-oriented.** It is designed to express commands in one line. Semicolons are not necessary, quotations have meaning.

* Avoid semicolons at all costs. Follow the _one line, one instruction_ mantra.
* Express explicit strings with single quotes.
* Use double quotes only for interpolated strings.
* Write paths and wildcards _as-is_.
* Always surround keywords with spaces.
* Avoid command continuation backticks.
* Avoid chaining pipelines over multiple lines.
* If you chain pipeline over multiple lines, backtick to start with a pipe.

Example:

    function LoadConfig {
        $config = Get-Content configurations\urls.json | ConvertFrom-Json
        return $config
    }

    if (Test-Path **\*.json) {
        Write-Output 'Found json files.'
    }

    $message = "Hello $name"

    # Bad - avoid this:
    $bad = "Hi $name"; Write-Host $bad

    # Bad - avoid this:
    Invoke-Psake `
        -BuildScript $buildScript `
        -TaskList $taskList `
        -Framework '4.0'

    # Bad - avoid this:
    if(Test-Path **\*.json){Write-Output 'Found json files.'}

    # Exception - only do this if you **really** know what you are doing:
    Get-Content $file `
        | Select-String *.cs -Encoding utf8 `
        | Tee-Object -Variable csFiles `
        | Sort-Object `
        | Out-File csharpFiles.txt

### Definition Style

**Powershell likes functions.** Functions are essential for modularity and clarity. Use this tool.

* Use `function` for regular functions, `filter` for processing pipeline items.
* Always use `param` to define parameters.
* Every parameter is defined on a separate line (at least).
* Parameters are always annotated with expected types.
* Always `return` from a function explicitly if it has something to return.

Example:

    function SaveConfig {
        param (
            [string] $Path,
            [switch] $Force
        )

        # Save configuration here
    }

    filter Even {
        $even = if ($_ % 2 -eq 0) { $_ }
        return $even
    }

    # Bad - avoid this:
    function SaveConfig($Path, $Force) {
        # Save configuration here
    }

    # Bad - avoid this:
    filter Even {
        if ($_ % 2 -eq 0) { $_ }
    }

### Call Style

**Powershell is graceful.** Be explicit during development, be merciful during usage.

* Never use aliases for commands (cmdlets).
* Call functions with positional parameters first if supported.
* Fallback to named version on positional parameters for clarification.
* Do not abbreviate named parameters.
* Always append parameters or switches to the arguments.
* Call .NET natives exactly as you would in C# (without semicolon).

Example:

    Copy-Item $from $to -Recurse

    Invoke-Psake $buildScript $taskList -Framework '4.0'

    $readmeFile = Get-ChildItem readme.md
    $acl = $readmeFile.GetAccessControl()

    # Bad - avoid this:
    cp -r $from $to

### Documentation Style

**Powershell is well-documented.** Document your commands.

* Always document your commands using inline help.
* Avoid externalizing your help to MAML, except you want to provide multiple cultures.
* Prefer single line comments for inline commentary.

Example:

    function Test-Assembly {
    <#
        .SYNOPSIS

        Tests a C# assembly with NUnit.

        .PARAMETER Path

        The path to the assemblies to test.

        .EXAMPLE

        Test-Assembly *.dll

        .NOTES

        This is just an example.
    #>
        param (
            [string[]] $Path
        )

        # Test your assembly here
    }

_Note: Consult the Powershell documentation on what is possible with inline help._

### Idioms

#### Use `%{}` and `?{}` aliases

Always prefer single character aliases for `Foreach-Object` and `Where-Object`. These are the only exceptions to the otherwise explicit call style.

Example:

    # Good - use this:
    $files | ?{ $_ -like '*.zip' } | %{ Copy-Item $_ Z:\archive\$_ -Recurse -Force }

    # Bad - avoid this:
    $files | Where-Object { $_ -like '*.zip' } | Foreach-Object { Copy-Item $_ Z:\archive\$_ -Recurse -Force }

#### Use shorthands for arrays and hashes

Always use the shorthand syntax for arrays and hashes where appropiate.

Example:

    # Good - array initialization:
    $set = @()

    # Good - ranges:
    $twenty = 1..20

    # Good - force array return:
    $files = @() + Get-ChildItem *.cs

    # Good - compose sets with csv:
    $colors = Red, Green, Blue, Cyan, Magenta

    # Good - hash initialization:
    $hash = @{}

    # Good - hash values:
    $hash = @{
        orders = 2;
        runtime = 'new';
        colors = $colors
    }

#### Use PS3 automatics if possible

Prefer the automatic variables provided by Powershell version 3 if possible.

Example:

    # Good - directory of executing script
    $here = $PSScriptRoot

    # Good - the full script path
    $me = $PSCommandPath

    # Good - find out if script is the root script (not dot-sourced)
    $main = -not $MyInvocation.PSCommandPath

#### Use type accelerators for object creation if possible

Type accelerators are awesome. Use the default type accelerators.

Example:

    [datetime]$date = '01/01/1970'
    [mailaddress]$email = 'john.doe@example.com'
    [regex]$rx = '^H(i|o).*$'
    [xml]$document = Get-Content doc.xml
    [version]$v = '1.4.1534'

#### Avoid `Write-Host` in functions

Just prefer `Write-Output` over `Write-Host`. Only use `Write-Host` if you know that your function is not going to be reused.

#### Prefer `string[]` over `string` for paths

For path parameters, prefer to use `string[]` in order to signal participation in batch processing.

#### Avoid setting preferences

Never try to set preferences of Powershell in your scripts, especially keep hands off from `$ErrorActionPreference`!

### Remarks

This "guideline" was written because I did not find any Powershell coding or scripting guidelines out there. It is by far not complete or perfect. As always with guidelines, the most important thing is _consistency_. Remember: style is debatable, consistency is not. I happen to be an irregular Powershell user and scripter for a few years now, but just recently got into more "serious" Powershell scripting. If you have any suggestions, feel free to fork, change and open a pull request.

_(c) 2014 Ilker Cetinkaya, MIT License._

