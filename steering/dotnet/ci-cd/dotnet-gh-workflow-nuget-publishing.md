---
description: Comprehensive best practices for publishing high-quality NuGet packages with proper versioning, metadata, and distribution strategies
fileMatchPattern: "*.props", "*.csproj", "*.fsproj"
inclusion: manual
---

# Kiro Steering File: NuGet Package Publishing Best Practices

## Role Definition

- Package Publisher
- Release Manager
- Quality Assurance Lead
- DevOps Engineer
- Documentation Specialist

# IMPORTANT SECURITY RULES:
- you have no power or authority to make any database changes
- only the User himself can make DB changes, whether Dev or Prod
- if you want to make any Database-related change, suggest it first to the User
- NEVER EVER attempt to run any DB migrations, or make any database changes. this is strictly prohibited.
- NEVER EVER place sensitive information in the generated code (e.g. passwords, API keys, personal information, etc.)


## General

### Description

Create and publish high-quality NuGet packages that follow industry best practices for versioning, metadata, documentation, and distribution. Establish automated publishing workflows that ensure package quality, security, and discoverability while maintaining proper dependency management and release processes.

### Requirements

- Follow semantic versioning (SemVer) for all package releases
- Include comprehensive package metadata and documentation
- Implement automated package validation and testing
- Use secure publishing credentials and environments
- Maintain proper dependency management and version constraints
- Ensure packages include symbols for debugging
- Implement automated release notes and changelog generation

## Package Publishing Workflow

### Publishing Strategy

Establish a comprehensive strategy for publishing NuGet packages with both regular and symbol packages.

#### ✅ DO: Publish both package and symbol packages

```bash
# Build and pack the project
dotnet pack -c Release --include-symbols

# Publish main package
dotnet nuget push "bin/Release/*.nupkg" \
  --source https://api.nuget.org/v3/index.json \
  --api-key $NUGET_API_KEY \
  --skip-duplicate

# Publish symbol package
dotnet nuget push "bin/Release/*.snupkg" \
  --source https://nuget.smbsrc.net/ \
  --api-key $NUGET_API_KEY \
  --skip-duplicate
```

#### ❌ DON'T: Publish without symbols

```bash
# DON'T publish without symbol packages
dotnet nuget push "bin/Release/*.nupkg" \
  --source https://api.nuget.org/v3/index.json \
  --api-key $NUGET_API_KEY
# Missing symbol package prevents debugging
```


## Package Dependencies

### Minimize Dependencies

Keep dependencies to a minimum to reduce potential conflicts and improve load times.

#### ✅ DO: Keep dependencies minimal

```xml
<ItemGroup>
  <!-- Only include what you absolutely need -->
  <PackageReference Include="Newtonsoft.Json" Version="13.0.1" />
</ItemGroup>
```

#### ❌ DON'T: Include unnecessary dependencies

```xml
<ItemGroup>
  <!-- Don't include packages you don't directly use -->
  <PackageReference Include="Newtonsoft.Json" Version="13.0.1" />
  <PackageReference Include="System.Data.SqlClient" Version="4.8.3" />
  <PackageReference Include="Microsoft.Extensions.Logging" Version="6.0.0" />
</ItemGroup>
```

### Use Appropriate Version Ranges

Specify version ranges that balance flexibility with stability.

#### ✅ DO: Use specific version ranges

```xml
<ItemGroup>
  <!-- Exact version -->
  <PackageReference Include="ExactPackage" Version="1.2.3" />
  
  <!-- Minimum version (1.0.0 or higher) -->
  <PackageReference Include="MinimumPackage" Version="1.0.0" />
  
  <!-- Range with minimum and maximum (>= 2.0.0 and < 3.0.0) -->
  <PackageReference Include="RangePackage" Version="[2.0.0,3.0.0)" />
  
  <!-- Specific major and minor, any patch (>= 1.2.0 and < 1.3.0) -->
  <PackageReference Include="MinorRangePackage" Version="[1.2.*,)" />
</ItemGroup>
```

#### ❌ DON'T: Use overly broad version ranges

```xml
<ItemGroup>
  <!-- Too broad, accepts any version -->
  <PackageReference Include="AnyVersionPackage" Version="*" />
  
  <!-- Too broad, accepts any version from 1.0.0 onwards -->
  <PackageReference Include="TooFlexiblePackage" Version="1.*" />
</ItemGroup>
```

### Split Functionality into Separate Packages

For complex libraries, consider splitting functionality into separate packages.

#### ✅ DO: Split functionality logically

Example package structure:
- `MyCompany.MyLibrary.Core` - Core functionality
- `MyCompany.MyLibrary.Data` - Data access components
- `MyCompany.MyLibrary.AspNetCore` - ASP.NET Core integration
- `MyCompany.MyLibrary.All` - Metapackage that references all the above

#### ✅ DO: Create metapackages for convenience

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <PackageId>MyCompany.MyLibrary.All</PackageId>
    <Description>Metapackage that includes all MyLibrary components</Description>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="MyCompany.MyLibrary.Core" Version="1.0.0" />
    <PackageReference Include="MyCompany.MyLibrary.Data" Version="1.0.0" />
    <PackageReference Include="MyCompany.MyLibrary.AspNetCore" Version="1.0.0" />
  </ItemGroup>
</Project>
```

## Versioning

### Follow Semantic Versioning (SemVer)

Use SemVer to communicate the impact of changes to your package.

#### Version Components

- **Major (X.y.z)**: Breaking changes
- **Minor (x.Y.z)**: New features, non-breaking
- **Patch (x.y.Z)**: Bug fixes only

#### ✅ DO: Increment version numbers appropriately

```xml
<PropertyGroup>
  <!-- Initial version -->
  <Version>1.0.0</Version>
  
  <!-- After adding new features -->
  <Version>1.1.0</Version>
  
  <!-- After fixing bugs -->
  <Version>1.1.1</Version>
  
  <!-- After making breaking changes -->
  <Version>2.0.0</Version>
</PropertyGroup>
```

#### ✅ DO: Use version suffixes for pre-release versions

```xml
<PropertyGroup>
  <!-- Alpha release -->
  <Version>1.0.0-alpha.1</Version>
  
  <!-- Beta release -->
  <Version>1.0.0-beta.2</Version>
  
  <!-- Release candidate -->
  <Version>1.0.0-rc.1</Version>
  
  <!-- Final release -->
  <Version>1.0.0</Version>
</PropertyGroup>
```

#### ❌ DON'T: Make breaking changes without incrementing the major version

```xml
<!-- DON'T: This suggests no breaking changes -->
<PropertyGroup>
  <Version>1.2.0</Version>
</PropertyGroup>
```

### Version in Directory.Build.props

For multi-package solutions, manage versions centrally.

#### ✅ DO: Centralize version management

```xml
<!-- In Directory.Build.props -->
<Project>
  <PropertyGroup>
    <VersionPrefix>1.2.3</VersionPrefix>
    <VersionSuffix Condition="'$(Configuration)' == 'Debug'">preview</VersionSuffix>
  </PropertyGroup>
</Project>
```

## Build and Pack Commands

### Generate Packages with dotnet pack

Use `dotnet pack` to generate both `.nupkg` and `.snupkg` files.

#### ✅ DO: Pack with appropriate configuration

```shell
# Basic pack command
dotnet pack -c Release

# Pack with specific version
dotnet pack -c Release /p:Version=1.2.3

# Pack with version suffix
dotnet pack -c Release --version-suffix preview.1

# Pack multiple projects
dotnet pack MySolution.sln -c Release
```

### Verify Package Contents

Always verify package contents before publishing.

#### ✅ DO: Inspect package contents

```shell
# Install the NuGet Package Explorer CLI
dotnet tool install -g NuGet.PackageExplorer.CLI

# View package contents
nuget-pe view MyPackage.1.0.0.nupkg

# Or use the nuget CLI
nuget verify MyPackage.1.0.0.nupkg
```

### Publish Packages

Publish packages to NuGet.org or a private feed.

#### ✅ DO: Publish to NuGet.org

```shell
# Push package to NuGet.org
dotnet nuget push MyPackage.1.0.0.nupkg -s https://api.nuget.org/v3/index.json -k YOUR_API_KEY

# Push symbol package
dotnet nuget push MyPackage.1.0.0.snupkg -s https://api.nuget.org/v3/index.json -k YOUR_API_KEY
```

#### ✅ DO: Publish to a private feed

```shell
# Push to Azure Artifacts
dotnet nuget push MyPackage.1.0.0.nupkg -s https://pkgs.dev.azure.com/myorg/_packaging/myfeed/nuget/v3/index.json -k az

# Push to GitHub Packages
dotnet nuget push MyPackage.1.0.0.nupkg -s https://nuget.pkg.github.com/myorg/index.json -k YOUR_GITHUB_TOKEN
```

## Quality Checks

### Pre-publish Checklist

Always run through this checklist before publishing:

1. **Metadata Verification**
   - Package ID is correct and follows naming conventions
   - Version is appropriate (SemVer)
   - Description is clear and informative
   - Authors and copyright information is correct
   - License expression is valid
   - Project URL and repository URL are correct
   - Tags are relevant and helpful for discoverability

2. **Content Verification**
   - README.md is included and renders correctly
   - All images in README use approved domains
   - XML documentation is generated and included
   - No unnecessary files are included in the package

3. **Symbol and Source Verification**
   - Symbol package (`.snupkg`) is generated
   - SourceLink is configured correctly
   - Source debugging works as expected

4. **Dependency Verification**
   - Dependencies are minimal and necessary
   - Version ranges are appropriate
   - No conflicting dependencies

5. **Functional Verification**
   - Package installs successfully in a new project
   - Basic functionality works as expected
   - No runtime errors or exceptions

### Automated Quality Checks

Implement automated checks in your CI/CD pipeline.

#### ✅ DO: Add package validation to CI

```yaml
# Example GitHub Actions workflow
name: Package Validation

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: 9.0.x
    - name: Restore dependencies
      run: dotnet restore
    - name: Build
      run: dotnet build --no-restore -c Release
    - name: Test
      run: dotnet test --no-build -c Release
    - name: Pack
      run: dotnet pack --no-build -c Release
    - name: Validate Package
      run: |
        dotnet tool install -g NuGet.PackageExplorer.CLI
        nuget-pe validate bin/Release/*.nupkg
```

## CI/CD Integration

### Automated Publishing Workflows

#### ✅ DO: Implement automated publishing pipelines

```yaml
# GitHub Actions publishing workflow
name: Publish to NuGet

on:
  release:
    types: [published]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: 9.0.x

    - name: Restore dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --no-restore -c Release

    - name: Test
      run: dotnet test --no-build -c Release

    - name: Pack
      run: dotnet pack --no-build -c Release --include-symbols

    - name: Publish to NuGet
      run: |
        dotnet nuget push "bin/Release/*.nupkg" \
          --source https://api.nuget.org/v3/index.json \
          --api-key ${{ secrets.NUGET_API_KEY }} \
          --skip-duplicate

    - name: Publish Symbols
      run: |
        dotnet nuget push "bin/Release/*.snupkg" \
          --source https://nuget.smbsrc.net/ \
          --api-key ${{ secrets.NUGET_API_KEY }} \
          --skip-duplicate
```

#### ✅ DO: Include version management in CI/CD

```yaml
- name: Update version
  run: |
    # Extract version from tag (e.g., v1.2.3)
    VERSION=${GITHUB_REF#refs/tags/v}
    echo "VERSION=$VERSION" >> $GITHUB_ENV

- name: Pack with version
  run: dotnet pack -c Release --include-symbols /p:Version=$VERSION
```

> [!WARNING]
> **Publishing Security:**
>
> - Never commit API keys to source control
> - Use environment variables or secret management systems
> - Implement proper access controls for publishing operations
> - Regularly rotate publishing credentials

## Additional Resources

- [NuGet Documentation](https://docs.microsoft.com/en-us/nuget)
- [SPDX License List](https://spdx.org/licenses)
- [SourceLink Documentation](https://github.com/dotnet/sourcelink)
- [SemVer Specification](https://semver.org)
- [NuGet Package Explorer](https://github.com/NuGetPackageExplorer/NuGetPackageExplorer)

# End of Kiro Steering File 