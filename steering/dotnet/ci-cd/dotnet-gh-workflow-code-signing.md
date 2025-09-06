---
description: Comprehensive security practices for .NET code signing, certificate management, and secure package distribution
globs: "**/signsettings.json","**/appsettings.json","**/*.nupkg"
inclusion: manual
---

# Kiro Steering File: .NET Code Signing Best Practices

## Role Definition

- Security Engineer
- DevSecOps Specialist
- Code Signing Administrator
- Compliance Officer
- Release Manager

## General

### Description

Implement secure code signing practices for .NET applications and NuGet packages to ensure authenticity, integrity, and compliance. Establish automated signing workflows that protect against tampering while maintaining efficient CI/CD pipelines. Focus on certificate lifecycle management, secure credential handling, and comprehensive audit trails for all signed artifacts.

### Requirements

**- NEVER: Place sensitive information in the generated code (e.g. passwords, API keys, personal information, etc.)**
- Implement automated code signing in CI/CD pipelines
- Use secure credential management for signing certificates
- Maintain proper certificate lifecycle management
- Implement comprehensive audit logging for signing operations
- Ensure compliance with security standards and regulations
- Validate signed packages before distribution
- Monitor certificate expiration and renewal processes

## Prerequisites

These rules only apply when one of the following exists:
- `signsettings.json`
- `appsettings.json` with a `SignClient` section

## Code Signing Configuration

### SignClient Setup

Install and configure SignClient for automated code signing in CI/CD pipelines.

#### ✅ DO: Install SignClient properly

```bash
# Install SignClient globally
dotnet tool install --global SignClient

# Verify installation
signclient --version
```

#### ✅ DO: Configure SignClient settings

```json
// appsettings.json
{
  "SignClient": {
    "AzureAd": {
      "AADInstance": "https://login.microsoftonline.com/",
      "ClientId": "[configured in CI]",
      "TenantId": "[configured in CI]"
    },
    "Service": {
      "Url": "https://signing-service.example.com/",
      "ResourceId": "[configured in CI]"
    }
  }
}
```

#### ❌ DON'T: Hardcode signing credentials

```json
// DON'T do this in source control
{
  "SignClient": {
    "AzureAd": {
      "ClientId": "hardcoded-client-id", // NEVER commit this
      "ClientSecret": "hardcoded-secret" // NEVER commit this
    }
  }
}
```

## Security Best Practices
1. Never commit signing credentials to source control
2. Store signing credentials in CI/CD secret variables
3. Use environment-specific signing configurations
4. Implement proper credential rotation
5. Log signing operations for audit purposes

## CI/CD Integration

### GitHub Actions Example
```yaml
jobs:
  sign:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Install SignClient
      run: dotnet tool install --global SignClient
    
    - name: Sign NuGet Packages
      run: |
        SignClient sign `
          --config signsettings.json `
          --input "**/*.nupkg" `
          --secret "${{ secrets.SIGN_CLIENT_SECRET }}" `
          --name "MyProject" `
          --description "Official release package"
```

### Azure DevOps Example
```yaml
steps:
- task: UseDotNet@2
  inputs:
    useGlobalJson: true

- script: dotnet tool install --global SignClient
  displayName: 'Install SignClient'

- powershell: |
    SignClient sign `
      --config signsettings.json `
      --input "$(Build.ArtifactStagingDirectory)/*.nupkg" `
      --secret "$(SignClientSecret)" `
      --name "$(Build.DefinitionName)" `
      --description "Official release package"
  displayName: 'Sign Packages'
  env:
    SignClientSecret: $(SignClientSecret)
```

## Verification
1. Implement post-signing verification
2. Check signature validity before publishing
3. Include signature verification in test pipeline
4. Document signature verification process

## Package Publishing
1. Only publish signed packages for releases
2. Implement signature verification before publishing
3. Maintain signed package archive
4. Document package signature in release notes

## Troubleshooting
1. Common signing errors and solutions
2. Credential validation steps
3. Service connectivity issues
4. Certificate expiration handling

## Maintenance

### Certificate Lifecycle Management

#### ✅ DO: Implement automated certificate monitoring

```yaml
# Example monitoring workflow
name: Certificate Health Check

on:
  schedule:
    - cron: '0 9 * * 1'  # Weekly on Monday
  workflow_dispatch:

jobs:
  check-certificates:
    runs-on: ubuntu-latest
    steps:
    - name: Check certificate expiration
      run: |
        # Implement certificate expiration checking
        echo "Checking certificate expiration dates..."

    - name: Alert on expiring certificates
      if: failure()
      run: |
        echo "Certificate expiration alert triggered"
```

#### ✅ DO: Implement credential rotation

```powershell
# PowerShell script for credential rotation
param(
    [Parameter(Mandatory=$true)]
    [string]$KeyVaultName,
    [Parameter(Mandatory=$true)]
    [string]$CertificateName
)

# Rotate signing certificate in Azure Key Vault
# Implementation here
```

## Troubleshooting

### Common Signing Issues

#### ❌ DON'T: Ignore signing failures

```yaml
# DON'T disable signing on failure
- name: Sign packages (dangerous)
  continue-on-error: true  # DON'T do this
  run: signclient sign...
```

#### ✅ DO: Handle signing failures properly

```yaml
- name: Sign packages
  run: |
    signclient sign --config signsettings.json --input "**/*.nupkg"
    if ($LASTEXITCODE -ne 0) {
        Write-Error "Package signing failed"
        exit 1
    }
```

> [!WARNING]
> **Security Considerations:**
>
> - Never store signing credentials in source control
> - Always use secure credential management (Azure Key Vault, AWS Secrets Manager, etc.)
> - Implement proper access controls for signing operations
> - Maintain comprehensive audit logs for all signing activities

# End of Kiro Steering File 