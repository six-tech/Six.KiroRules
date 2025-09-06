---
description: Comprehensive best practices for .NET build systems, CI/CD pipelines, and automated build processes
fileMatchPattern: "**/Directory.Build.props","**/.azure/*.yaml","**/build-system/*.yaml",**/"RELEASE_NOTES.md", "**/.github/workflows/*.yaml", "**/*.sln,**/*.csproj"
inclusion: manual
---

# Kiro Steering File: .NET Build System Best Practices

## Role Definition

- Build System Architect
- CI/CD Pipeline Expert
- .NET Build Engineer
- DevOps Specialist
- Quality Assurance Lead

## General

### Description

Establish robust, maintainable build systems for .NET applications using native tooling, clear separation of concerns, and automated quality assurance. Focus on cross-platform compatibility, reproducible builds, and efficient CI/CD pipelines that ensure code quality while maintaining fast feedback loops. Implement comprehensive testing, packaging, and deployment strategies that scale with your team's needs.

### Requirements

**- NEVER: Place sensitive information in the generated code (e.g. passwords, API keys, personal information, etc.)**
- Use native `dotnet` CLI as the primary build mechanism
- Implement cross-platform compatible build scripts
- Maintain clear separation between build, test, and deployment processes
- Use PowerShell only for tasks that can't be handled by `dotnet` CLI
- Document all build prerequisites and environment requirements
- Implement automated quality checks and testing
- Ensure reproducible builds across different environments
- Follow security best practices for credential management

## Project Structure

> [!IMPORTANT]
> Don't create new files without first checking existing ones. Build system files should follow a consistent, predictable structure.

### Required Files and Their Purposes
```
├── Directory.Build.props       # Central version and package metadata
├── Directory.Packages.props    # Centralized package version management
├── global.json                # SDK version pinning
├── RELEASE_NOTES.md           # Version history and release notes
├── build.ps1                  # Version management script (minimal)
├── scripts/                   # PowerShell helper scripts
│   ├── getReleaseNotes.ps1    # Parse release notes
│   ├── bumpVersion.ps1        # Update assembly versions
│   ├── integration-tests.ps1  # Integration test runner
│   └── *.ps1                 # Other build helper scripts
├── build-system/             # CI/CD pipeline definitions
│   ├── azure-pipeline.template.yaml  # Shared pipeline template
│   ├── windows-pr-validation.yaml    # Windows PR validation
│   ├── linux-pr-validation.yaml      # Linux PR validation
│   └── windows-release.yaml          # Release pipeline
└── .github/workflows/        # GitHub Actions (if used)
    ├── pr-validation.yaml    # PR validation
    └── release.yaml          # Release workflow
```

### File Responsibilities

#### Directory.Build.props
- Central version management
- Package metadata
- Common build properties
- Shared package versions

Example:
```xml
<Project>
  <PropertyGroup>
    <VersionPrefix>1.0.0</VersionPrefix>
    <PackageReleaseNotes><!-- Auto-updated by build.ps1 --></PackageReleaseNotes>
    <Authors>Your Company</Authors>
    <Copyright>© $([System.DateTime]::Now.Year) Your Company</Copyright>
    <PackageLicenseExpression>Apache-2.0</PackageLicenseExpression>
    <PackageProjectUrl>https://github.com/org/repo</PackageProjectUrl>
    <PackageReadmeFile>README.md</PackageReadmeFile>
    <RepositoryUrl>https://github.com/org/repo.git</RepositoryUrl>
    <RepositoryType>git</RepositoryType>
  </PropertyGroup>
  
  <!-- Package versions -->
  <PropertyGroup>
    <MicrosoftExtensionsVersion>8.0.0</MicrosoftExtensionsVersion>
    <XunitVersion>2.7.0</XunitVersion>
  </PropertyGroup>
</Project>
```

#### RELEASE_NOTES.md
Must follow this exact format for automated parsing:
```markdown
#### 1.2.3 March 14 2024 ####
* First change
* Second change

#### 1.2.2 March 10 2024 ####
* Previous changes
```

> [!WARNING]
> Common mistakes to avoid:
> - Extra blank lines between version and changes
> - Missing space between version and date
> - Incorrect number of # symbols
> - Missing bullet points for changes

## Build Scripts

### build.ps1
Keep this script minimal. Its only job should be version management:

```powershell
[CmdletBinding()]
param()

$ErrorActionPreference = 'Stop'

# Import helper functions
. "$PSScriptRoot\scripts\getReleaseNotes.ps1"
. "$PSScriptRoot\scripts\bumpVersion.ps1"

# Update version information
$releaseNotes = Get-ReleaseNotes -MarkdownFile (Join-Path -Path $PSScriptRoot -ChildPath "RELEASE_NOTES.md")
UpdateVersionAndReleaseNotes -ReleaseNotesResult $releaseNotes -XmlFilePath (Join-Path -Path $PSScriptRoot -ChildPath "src\Directory.Build.props")

Write-Output "Updated version to $($releaseNotes.Version)"
```

### Integration Tests
Place integration tests in a dedicated script with clear error handling:

```powershell
[CmdletBinding()]
param(
    [ValidateSet("Release", "Debug")]
    [string]$Configuration = "Release"
)

# Track test results
$script:hasUnexpectedFailures = $false
$script:totalTests = 0
$script:passedTests = 0
$script:failedTests = 0

function Invoke-Test {
    param(
        [string]$TestName,
        [scriptblock]$TestScript,
        [bool]$ExpectFailure = $false
    )
    
    $script:totalTests++
    try {
        & $TestScript
        if ($LASTEXITCODE -ne 0 -and -not $ExpectFailure) {
            $script:hasUnexpectedFailures = $true
            $script:failedTests++
        } else {
            $script:passedTests++
        }
    }
    catch {
        $script:hasUnexpectedFailures = $true
        $script:failedTests++
    }
}

# Run all tests before exiting
exit $script:hasUnexpectedFailures ? 1 : 0
```

## CI/CD Pipeline Best Practices

### ✅ DO
- Use pipeline templates for shared configuration
- Keep pipeline files in `build-system/` directory
- Use consistent naming conventions
- Include clear display names for all steps
- Set appropriate timeouts
- Configure proper trigger conditions
- Use matrix builds for cross-platform testing
- Publish test results and artifacts
- Set up proper dependency caching

### ❌ DON'T
- Put complex logic in YAML files
- Duplicate steps across pipelines
- Use inline scripts for complex operations
- Hardcode version numbers or configuration
- Mix PR validation and release pipelines
- Ignore test failures
- Skip publishing test results

### Azure Pipeline Template Example
```yaml
parameters:
  name: ''
  vmImage: ''
  timeoutInMinutes: 10
  runIntegrationTests: false

jobs:
  - job: ${{ parameters.name }}
    timeoutInMinutes: ${{ parameters.timeoutInMinutes }}
    pool:
      vmImage: ${{ parameters.vmImage }}
    steps:
      - checkout: self
        clean: false
        submodules: recursive
        
      - task: UseDotNet@2
        displayName: 'Use .NET SDK'
        inputs:
          useGlobalJson: true
          
      # Version bump
      - task: PowerShell@2
        displayName: 'Update version'
        inputs:
          filePath: './build.ps1'
          
      # Build and test
      - script: dotnet build -c Release
        displayName: 'Build solution'
          
      - script: dotnet test -c Release --no-build --logger:trx
        displayName: 'Run unit tests'
          
      # Integration tests
      - task: PowerShell@2
        displayName: 'Run integration tests'
        condition: eq(${{ parameters.runIntegrationTests }}, true)
        inputs:
          filePath: './scripts/integration-tests.ps1'
          arguments: '-Configuration Release'
          
      # Package
      - script: dotnet pack -c Release -o $(Build.ArtifactStagingDirectory)
        displayName: 'Create packages'
        condition: succeeded()
          
      # Publish results
      - task: PublishTestResults@2
        displayName: 'Publish test results'
        condition: always()
        inputs:
          testRunner: VSTest
          testResultsFiles: '**/*.trx'
          failTaskOnFailedTests: true
```

### PR Validation Pipeline
```yaml
trigger:
  branches:
    include: [ dev, master ]
  paths:
    exclude: [ '*.md', 'docs/*' ]

pr:
  autoCancel: true
  branches:
    include: [ dev, master ]

jobs:
  - template: azure-pipeline.template.yaml
    parameters:
      name: 'Windows'
      vmImage: 'windows-latest'
      runIntegrationTests: true
```

### Release Pipeline
```yaml
trigger:
  branches:
    include: [ refs/tags/* ]
  paths:
    exclude: [ '*.md', 'docs/*' ]

variables:
  - group: nuget-keys
  - name: projectName
    value: YourProject

steps:
  - task: UseDotNet@2
    inputs:
      useGlobalJson: true

  - script: ./build.ps1
    displayName: 'Update version'

  - script: |
      dotnet pack -c Release -o $(Build.ArtifactStagingDirectory)
      dotnet nuget push "$(Build.ArtifactStagingDirectory)/*.nupkg" --api-key $(nugetKey) --source https://api.nuget.org/v3/index.json
    displayName: 'Create and publish packages'
```

## Common Patterns and Anti-patterns

### Version Management

✅ DO:
- Use RELEASE_NOTES.md as the single source of truth
- Automate version updates via build.ps1
- Include detailed release notes for each version
- Follow semantic versioning
- Update all version references consistently

❌ DON'T:
- Manually edit version numbers
- Store versions in multiple places
- Skip release notes
- Use inconsistent version formats
- Forget to update package versions

### Build Process

✅ DO:
- Build before running tests
- Use `--no-build` for test/pack after build
- Set appropriate configuration
- Enable deterministic builds
- Cache dependencies
- Use consistent output directories

❌ DON'T:
- Mix Debug/Release artifacts
- Skip test runs
- Ignore build warnings
- Use platform-specific commands
- Hardcode paths

### Testing

✅ DO:
- Run all tests before exit
- Publish test results
- Use appropriate test loggers
- Set test timeouts
- Handle expected failures properly
- Separate unit and integration tests

❌ DON'T:
- Ignore test failures
- Skip result publishing
- Mix test types
- Use platform-specific test runners
- Leave failing tests unhandled

### Package Creation

✅ DO:
- Include symbols packages
- Set appropriate package metadata
- Use consistent output directory
- Verify package contents
- Include documentation files

❌ DON'T:
- Skip symbol packages
- Hardcode version numbers
- Mix package sources
- Ignore package validation
- Skip README files

## Migration Guidelines

When moving from complex build systems to `dotnet` CLI:

1. **Analyze Current Build**
   - List all build tasks
   - Identify custom logic
   - Document dependencies
   - Note platform-specific code

2. **Plan Migration**
   - Map tasks to `dotnet` commands
   - Identify scripts needed
   - Plan folder structure
   - Set up new pipelines

3. **Execute Migration**
   - Create new script structure
   - Convert build tasks
   - Update CI/CD pipelines
   - Test thoroughly
   - Run in parallel with old system

4. **Validate**
   - Cross-platform testing
   - Build verification
   - Package validation
   - Pipeline testing
   - Documentation update

## Best Practices Summary

1. **Script Organization**
   - One purpose per script
   - Clear error handling
   - Consistent naming
   - Proper documentation
   - Cross-platform compatibility

2. **Pipeline Structure**
   - Template-based design
   - Clear step organization
   - Proper condition handling
   - Result publishing
   - Artifact management

3. **Version Control**
   - Single source of truth
   - Automated updates
   - Consistent formatting
   - Clear documentation
   - Proper validation

4. **Testing Strategy**
   - Separate test types
   - Clear result reporting
   - Proper error handling
   - Complete coverage
   - Performance consideration

5. **Documentation**
   - Clear prerequisites
   - Step-by-step guides
   - Troubleshooting tips
   - Example commands
   - Version history

## Version Management Helper Functions

### Release Notes Parser (getReleaseNotes.ps1)
This script parses the RELEASE_NOTES.md file to extract version information and notes:

```powershell
function Get-ReleaseNotes {
    param (
        [Parameter(Mandatory=$true)]
        [string]$MarkdownFile
    )

    # Read markdown file content
    $content = Get-Content -Path $MarkdownFile -Raw

    # Split content based on headers
    $sections = $content -split "####"

    # Output object to store result
    $outputObject = [PSCustomObject]@{
        Version       = $null
        Date          = $null
        ReleaseNotes  = $null
    }

    # Check if we have at least 3 sections (1. Before the header, 2. Header, 3. Release notes)
    if ($sections.Count -ge 3) {
        $header = $sections[1].Trim()
        $releaseNotes = $sections[2].Trim()

        # Extract version and date from the header
        $headerParts = $header -split " ", 2
        if ($headerParts.Count -eq 2) {
            $outputObject.Version = $headerParts[0]
            $outputObject.Date = $headerParts[1]
        }

        $outputObject.ReleaseNotes = $releaseNotes
    }

    return $outputObject
}
```

### Version Updater (bumpVersion.ps1)
This script updates the version and release notes in Directory.Build.props:

```powershell
function UpdateVersionAndReleaseNotes {
    param (
        [Parameter(Mandatory=$true)]
        [PSCustomObject]$ReleaseNotesResult,

        [Parameter(Mandatory=$true)]
        [string]$XmlFilePath
    )

    if (-not (Test-Path $XmlFilePath)) {
        throw "Directory.Build.props not found at: $XmlFilePath"
    }

    try {
        # Load XML
        $xmlContent = New-Object XML
        $xmlContent.Load($XmlFilePath)

        # Update VersionPrefix and PackageReleaseNotes
        $versionPrefixElement = $xmlContent.SelectSingleNode("//VersionPrefix")
        if ($null -eq $versionPrefixElement) {
            throw "VersionPrefix element not found in Directory.Build.props"
        }
        $versionPrefixElement.InnerText = $ReleaseNotesResult.Version

        $packageReleaseNotesElement = $xmlContent.SelectSingleNode("//PackageReleaseNotes")
        if ($null -eq $packageReleaseNotesElement) {
            throw "PackageReleaseNotes element not found in Directory.Build.props"
        }
        $packageReleaseNotesElement.InnerText = $ReleaseNotesResult.ReleaseNotes

        # Save the updated XML
        $xmlContent.Save($XmlFilePath)
    }
    catch {
        throw "Failed to update Directory.Build.props: $_"
    }
}
```

### Finding Directory.Build.props
When working with complex repository structures, you might need to locate the correct Directory.Build.props file. Here's a helper function:

```powershell
function Find-DirectoryBuildProps {
    param (
        [Parameter(Mandatory=$true)]
        [string]$StartPath,
        
        [Parameter(Mandatory=$false)]
        [string]$FileName = "Directory.Build.props"
    )
    
    # First check if file exists in start path
    $directPath = Join-Path $StartPath $FileName
    if (Test-Path $directPath) {
        return $directPath
    }
    
    # Check src directory if it exists
    $srcPath = Join-Path $StartPath "src" $FileName
    if (Test-Path $srcPath) {
        return $srcPath
    }
    
    # Search recursively up to 2 levels deep
    $searchResults = Get-ChildItem -Path $StartPath -Filter $FileName -Recurse -Depth 2 |
        Where-Object { $_.Name -eq $FileName }
    
    if ($searchResults.Count -eq 0) {
        throw "Could not find $FileName in or below $StartPath"
    }
    if ($searchResults.Count -gt 1) {
        Write-Warning "Found multiple $FileName files. Using the first one found."
        $searchResults | ForEach-Object { Write-Warning "Found: $($_.FullName)" }
    }
    
    return $searchResults[0].FullName
}
```

### Complete Version Update Example
Here's how to use these functions together in your build script:

```powershell
[CmdletBinding()]
param(
    [Parameter(Mandatory=$false)]
    [string]$Configuration = "Release"
)

$ErrorActionPreference = 'Stop'

# Import helper functions
. "$PSScriptRoot\scripts\getReleaseNotes.ps1"
. "$PSScriptRoot\scripts\bumpVersion.ps1"

try {
    # Find Directory.Build.props
    $buildPropsPath = Find-DirectoryBuildProps -StartPath $PSScriptRoot
    Write-Host "Found Directory.Build.props at: $buildPropsPath"
    
    # Parse release notes
    $releaseNotesPath = Join-Path $PSScriptRoot "RELEASE_NOTES.md"
    $releaseNotes = Get-ReleaseNotes -MarkdownFile $releaseNotesPath
    
    if ($null -eq $releaseNotes.Version) {
        throw "Failed to parse version from RELEASE_NOTES.md"
    }
    
    Write-Host "Updating to version $($releaseNotes.Version)"
    
    # Update version information
    UpdateVersionAndReleaseNotes -ReleaseNotesResult $releaseNotes -XmlFilePath $buildPropsPath
    
    Write-Host "Successfully updated version and release notes"
}
catch {
    Write-Error "Failed to update version: $_"
    exit 1
}
```

### Common Version Management Issues

#### ✅ DO:
- Validate release notes format before parsing
- Handle multiple Directory.Build.props files gracefully
- Provide clear error messages for parsing failures
- Back up files before making changes
- Log all version updates

#### ❌ DON'T:
- Assume file locations without verification
- Skip error handling in XML operations
- Overwrite files without validation
- Ignore malformed release notes
- Make partial updates

### Troubleshooting Version Updates

1. **Release Notes Not Parsed**
   - Check exact format matches template
   - Verify no extra spaces in header
   - Ensure proper line endings (CRLF vs LF)
   - Validate bullet point format

2. **Directory.Build.props Not Found**
   - Check repository structure
   - Verify search path is correct
   - Look for case sensitivity issues
   - Check file permissions

3. **XML Update Failures**
   - Verify XML is well-formed
   - Check for required elements
   - Ensure proper namespace handling
   - Validate XML schema

4. **Version Format Issues**
   - Ensure semantic versioning
   - Check for pre-release tag format
   - Validate build metadata
   - Verify version string parsing 

## Command Delegation and Error Handling

### When to Use `dotnet` CLI vs PowerShell

#### Use `dotnet` CLI Directly For:
- Building projects and solutions
- Running tests
- Creating NuGet packages
- Publishing applications
- Restoring packages
- Managing project references

Example of proper `dotnet` CLI usage:
```powershell
# Direct command - no need for script wrapping
dotnet build -c Release

# Test with proper logger configuration
dotnet test -c Release --no-build --logger:trx

# Package creation with symbols
dotnet pack -c Release --include-symbols
```

#### Use PowerShell Scripts For:
- Version management and release notes parsing
- Complex integration test orchestration
- Environment setup and validation
- Tasks requiring file system operations
- Multi-step processes with error aggregation
- Custom build task orchestration

### Error Handling Patterns

#### ❌ Anti-Pattern: Shell Script Error Masking
```powershell
# DON'T DO THIS: Masks real exit codes
try {
    dotnet test
    if ($LASTEXITCODE -ne 0) { 
        Write-Warning "Tests failed but continuing..."
        $LASTEXITCODE = 0  # WRONG: Masks the real failure
    }
}
catch {
    Write-Warning "Error occurred but continuing..."
}
```

#### ✅ Correct Pattern: Preserve Exit Codes
```powershell
# DO THIS: Preserves exit codes and handles expected failures properly
function Invoke-Test {
    param(
        [string]$TestName,
        [scriptblock]$TestScript,
        [bool]$ExpectFailure = $false
    )
    
    try {
        & $TestScript
        $exitCode = $LASTEXITCODE
        
        if ($exitCode -ne 0 -and -not $ExpectFailure) {
            Write-Host "Test failed: $TestName (Exit code: $exitCode)" -ForegroundColor Red
            return $false
        }
        if ($exitCode -eq 0 -and $ExpectFailure) {
            Write-Host "Test succeeded unexpectedly: $TestName" -ForegroundColor Red
            return $false
        }
        
        Write-Host "Test completed as expected: $TestName" -ForegroundColor Green
        return $true
    }
    catch {
        Write-Host "Test threw exception: $TestName`n$_" -ForegroundColor Red
        return $false
    }
}
```

### Command Delegation Guidelines

#### ✅ DO:
- Let `dotnet` CLI handle all build operations
- Use `--no-build` when running tests after build
- Pass through exit codes faithfully
- Collect all errors before exiting
- Use proper logging and verbosity settings

#### ❌ DON'T:
- Wrap simple `dotnet` commands in scripts
- Mask or ignore exit codes
- Mix build configurations
- Handle MSBuild properties in scripts
- Reimplement `dotnet` CLI functionality

### Examples of Proper Delegation

#### Build and Test Pipeline
```powershell
# DON'T DO THIS:
function Build-AndTest {
    # Wrong: Unnecessary wrapping of dotnet commands
    dotnet restore
    if ($LASTEXITCODE -eq 0) {
        dotnet build
        if ($LASTEXITCODE -eq 0) {
            dotnet test
        }
    }
}

# DO THIS INSTEAD:
# In pipeline YAML:
steps:
  - script: dotnet build -c Release
    displayName: 'Build'
    
  - script: dotnet test -c Release --no-build
    displayName: 'Test'
```

#### Integration Test Runner
```powershell
# Proper balance of script logic and dotnet commands
function Start-IntegrationTest {
    param(
        [string]$ProjectPath,
        [string]$Configuration = "Release"
    )
    
    # Script handles:
    # 1. Test environment setup
    # 2. Result aggregation
    # 3. Error tracking
    $results = @{
        Passed = 0
        Failed = 0
        Errors = @()
    }
    
    # Let dotnet handle the actual operations
    $tests = @(
        @{ Name = "Basic"; Args = @("--filter", "Category=Basic") }
        @{ Name = "Extended"; Args = @("--filter", "Category=Extended") }
    )
    
    foreach ($test in $tests) {
        Write-Host "Running $($test.Name) tests..."
        
        # Delegate to dotnet for the actual test execution
        dotnet test $ProjectPath -c $Configuration --no-build @($test.Args)
        
        if ($LASTEXITCODE -ne 0) {
            $results.Failed++
            $results.Errors += "Test '$($test.Name)' failed with exit code $LASTEXITCODE"
        } else {
            $results.Passed++
        }
    }
    
    # Script handles result reporting
    if ($results.Failed -gt 0) {
        Write-Host "Test Summary: $($results.Passed) passed, $($results.Failed) failed"
        $results.Errors | ForEach-Object { Write-Host "  $_" -ForegroundColor Red }
        exit 1
    }
    
    Write-Host "All tests passed!" -ForegroundColor Green
    exit 0
}
```

### Error Code Handling

#### Exit Code Principles
1. Never mask unexpected failures
2. Track expected failures separately
3. Run all tests before exiting
4. Provide clear error summaries
5. Use proper exit codes

```powershell
# Example of proper error tracking
$script:hasUnexpectedFailures = $false
$script:expectedFailures = 0
$script:totalTests = 0

try {
    # Run all tests even if some fail
    foreach ($test in $tests) {
        $script:totalTests++
        
        & $test.Command
        if ($LASTEXITCODE -ne 0) {
            if ($test.ExpectFailure) {
                $script:expectedFailures++
            } else {
                $script:hasUnexpectedFailures = $true
            }
        }
    }
    
    # Exit with failure only if we had unexpected failures
    exit $script:hasUnexpectedFailures ? 1 : 0
}
catch {
    Write-Error "Test execution failed: $_"
    exit 1
}
```

### When to Create Helper Scripts

Create PowerShell scripts only when you need to:
1. Orchestrate multiple `dotnet` commands
2. Set up test environments
3. Aggregate results from multiple operations
4. Handle expected failures
5. Provide custom logging or reporting
6. Manage file system operations

Otherwise, use `dotnet` CLI directly in your build pipeline. 


### CI/CD Integration Examples

#### GitHub Actions

```yaml
name: Build and Test

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        global-json-file: global.json
    
    - name: Restore dependencies
      run: dotnet restore --verbosity normal
    
    - name: Build
      run: |
        dotnet build --configuration Release --no-restore \
          /p:ContinuousIntegrationBuild=true \
          /p:TreatWarningsAsErrors=true
    
    - name: Test
      run: dotnet test --configuration Release --no-build --verbosity normal
    
    - name: Upload build logs
      if: failure()
      uses: actions/upload-artifact@v4
      with:
        name: build-logs
        path: '**/*.binlog'
```

#### Azure DevOps

```yaml
trigger:
- main
- develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  DOTNET_CLI_TELEMETRY_OPTOUT: '1'
  DOTNET_NOLOGO: '1'

steps:
- task: UseDotNet@2
  displayName: 'Use .NET SDK from global.json'
  inputs:
    useGlobalJson: true

- task: DotNetCoreCLI@2
  displayName: 'Restore packages'
  inputs:
    command: 'restore'
    verbosityRestore: 'Normal'

- task: DotNetCoreCLI@2
  displayName: 'Build solution'
  inputs:
    command: 'build'
    arguments: |
      --configuration $(buildConfiguration) 
      --no-restore 
      /p:ContinuousIntegrationBuild=true 
      /p:TreatWarningsAsErrors=true
    verbosityBuild: 'Normal'

- task: DotNetCoreCLI@2
  displayName: 'Run tests'
  inputs:
    command: 'test'
    arguments: '--configuration $(buildConfiguration) --no-build'
    verbosityTest: 'Normal'
```


# End of Kiro Steering File 