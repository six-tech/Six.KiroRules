---
description: This file provides guidelines for writing clean, maintainable, and idiomatic C# single file apps (.NET 10 feature)
fileMatchPattern: "scripts/*.cs"
inclusion: fileMatch
---

# Kiro Steering File: C# Single File Apps (Scripts) Style Guide

Role Definition:

- C# Language Expert
- Software Architect
- Code Quality Specialist

# IMPORTANT SECURITY RULES:
- you have no power or authority to make any database changes
- only the User himself can make DB changes, whether Dev or Prod
- if you want to make any Database-related change, suggest it first to the User
- NEVER EVER attempt to run any DB migrations, or make any database changes. this is strictly prohibited.
- NEVER EVER place sensitive information in the generated code (e.g. passwords, API keys, personal information, etc.)


## 1. Overview & General Principles

### Core Philosophy

C# single file apps (a feature added in `.NET 10`) should be used for simple, single-purpose scripts. This way we can
keep our entire codebase in c# and avoid the need for several different languages/script types.

Also, since this is pure .NET, we can leverage the full power of .NET and available libraries.

### Requirements

- ALWAYS: Use #[[file:.kiro/steering/dotnet/csharp/csharp-coding-style.md]] as the default style rule. This rule extends it.
- ALWAYS: Use `.NET 10` or later.
- ALWAYS: Start all scripts with `#!/usr/bin/env dotnet`
- ALWAYS: Import the necessary packages with c# single file app importing format: `#:package Spectre.Console@0.49.1`
- ALWAYS use `Spectre.Console` library for outputting text to the console.
- ALWAYS use `CliWrap` library for executing external processes.
- 

### Script Organization Patterns

#### Script Entry Point Structure

Organize scripts with a clear separation of concerns:

```csharp
#!/usr/bin/env dotnet
#:package CliWrap@3.6.6
#:package Spectre.Console@0.49.1

using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using CliWrap;
using Spectre.Console;

// Configuration and setup at the top
var scriptConfig = new ScriptConfig();
SetupEnvironment(scriptConfig);

// Main execution flow
await RunMainAsync(scriptConfig);

// Cleanup and finalization
await CleanupAsync(scriptConfig);

// Good: Clear phases with proper error handling
public class ScriptConfig
{
    public string WorkingDirectory { get; set; } = ".";
    public List<string> SearchPaths { get; set; } = new();
    public LogLevel LogLevel { get; set; } = LogLevel.Info;
    public bool DryRun { get; set; }
}

static void SetupEnvironment(ScriptConfig config)
{
    // Environment setup logic
if (Directory.GetCurrentDirectory().EndsWith("scripts"))
{
    Directory.SetCurrentDirectory("..");
        config.WorkingDirectory = Directory.GetCurrentDirectory();
    }

    config.SearchPaths = ["libs", "src", "tests"];
}

static async Task RunMainAsync(ScriptConfig config)
{
    try
    {
        await AnsiConsole.Status()
            .StartAsync("Processing...", async ctx =>
            {
                // Main script logic
                await ProcessItemsAsync(config, ctx);
            });
    }
    catch (Exception ex)
    {
        AnsiConsole.MarkupLine($"[red]Error: {ex.Message}[/]");
        throw;
    }
}

static async Task CleanupAsync(ScriptConfig config)
{
    // Cleanup logic
    AnsiConsole.MarkupLine("[green]Script completed successfully[/]");
}
```

### Error Handling and Logging

#### Script-Specific Error Handling

Handle errors gracefully in script environments:

```csharp
// Good: Comprehensive error handling for scripts
try
{
    var result = await ExecuteBuildAsync();
    if (!result.Success)
    {
        AnsiConsole.MarkupLine($"[red]Build failed: {result.ErrorMessage}[/]");

        // Offer recovery options
        if (await PromptRecoveryAsync())
        {
            await AttemptRecoveryAsync();
        }
        else
        {
            Environment.Exit(1);
        }
    }
}
catch (FileNotFoundException ex)
{
    AnsiConsole.MarkupLine($"[red]Missing file: {ex.FileName}[/]");
    AnsiConsole.MarkupLine("[yellow]Ensure all required files are present[/]");
    Environment.Exit(2);
}
catch (UnauthorizedAccessException ex)
{
    AnsiConsole.MarkupLine($"[red]Access denied: {ex.Message}[/]");
    AnsiConsole.MarkupLine("[yellow]Run with appropriate permissions[/]");
    Environment.Exit(3);
}
catch (Exception ex)
{
    AnsiConsole.MarkupLine($"[red]Unexpected error: {ex.Message}[/]");

    // Log full details for debugging
    if (args.Contains("--verbose"))
    {
        AnsiConsole.WriteException(ex);
    }

    Environment.Exit(99);
}

// Good: Structured result types for scripts
public record ScriptResult(bool Success, string? ErrorMessage = null, int ExitCode = 0);

public static async Task<ScriptResult> ExecuteBuildAsync()
{
    try
    {
        var result = await Cli.Wrap("dotnet")
            .WithArguments("build --configuration Release")
            .ExecuteAsync();

        return result.ExitCode == 0
            ? new ScriptResult(true)
            : new ScriptResult(false, "Build failed", result.ExitCode);
    }
    catch (Exception ex)
    {
        return new ScriptResult(false, ex.Message, -1);
    }
}
```

### User Interaction Patterns

#### Command Line Argument Parsing

Handle script arguments effectively:

```csharp
// Good: Robust argument parsing with Spectre.Console
var command = new CommandApp();

command.AddCommand<BuildCommand>("build")
    .WithDescription("Build the solution")
    .WithExample(new[] { "build", "--configuration", "Release" });

command.AddCommand<DeployCommand>("deploy")
    .WithDescription("Deploy the application")
    .WithExample(new[] { "deploy", "--environment", "production" });

// Good: Using CommandSettings for type safety
[CommandSettings]
public class BuildSettings : CommandSettings
{
    [CommandOption("--configuration")]
    [DefaultValue("Release")]
    public string Configuration { get; set; }

    [CommandOption("--output")]
    public string OutputPath { get; set; }

    [CommandOption("--dry-run")]
    public bool DryRun { get; set; }

    public override ValidationResult Validate()
    {
        if (string.IsNullOrWhiteSpace(Configuration))
            return ValidationResult.Error("Configuration cannot be empty");

        return base.Validate();
    }
}
```

#### Interactive Prompts

Create user-friendly interactive scripts:

```csharp
// Good: Interactive script with Spectre.Console
static async Task<bool> ConfirmDeploymentAsync()
{
    var shouldDeploy = await AnsiConsole.PromptAsync(
        new ConfirmationPrompt("Deploy to production?")
        {
            DefaultValue = false,
            ShowDefaultValue = true
        });

    if (!shouldDeploy)
    {
        AnsiConsole.MarkupLine("[yellow]Deployment cancelled[/]");
        return false;
    }

    return true;
}

// Good: Multi-selection prompts for complex choices
static async Task<List<string>> SelectEnvironmentsAsync()
{
    var environments = await AnsiConsole.PromptAsync(
        new MultiSelectionPrompt<string>()
            .Title("Select target environments:")
            .AddChoices(["development", "staging", "production"])
            .UseConverter(env => env switch
            {
                "development" => "[blue]Development[/]",
                "staging" => "[yellow]Staging[/]",
                "production" => "[red]Production[/]",
                _ => env
            }));

    return environments;
}

// Good: Progress reporting for long-running operations
static async Task DeployToEnvironmentsAsync(List<string> environments)
{
    await AnsiConsole.Progress()
        .StartAsync(async ctx =>
        {
            var tasks = environments.Select(async env =>
            {
                var task = ctx.AddTask($"Deploying to {env}");
                await DeployToEnvironmentAsync(env, task);
                task.Increment(100);
            });

            await Task.WhenAll(tasks);
        });
}
```

### External Process Management

#### Using CliWrap Effectively

Handle external processes properly in scripts:

```csharp
// Good: Comprehensive process execution with CliWrap
public static async Task<ScriptResult> RunTestsAsync(string configuration = "Release")
{
    var stdOutBuffer = new StringBuilder();
    var stdErrBuffer = new StringBuilder();

    try
    {
        var result = await Cli.Wrap("dotnet")
            .WithArguments($"test --configuration {configuration} --logger trx")
            .WithWorkingDirectory(".")
            .WithStandardOutputPipe(PipeTarget.ToStringBuilder(stdOutBuffer))
            .WithStandardErrorPipe(PipeTarget.ToStringBuilder(stdErrBuffer))
            .WithValidation(CommandResultValidation.ZeroExitCode)
            .ExecuteAsync();

        return new ScriptResult(true, stdOutBuffer.ToString());
    }
    catch (Exception ex)
    {
        return new ScriptResult(false,
            $"Test execution failed: {ex.Message}\nOutput: {stdOutBuffer}\nErrors: {stdErrBuffer}",
            -1);
    }
}

// Good: Parallel process execution for performance
public static async Task<ScriptResult> BuildMultipleProjectsAsync(IEnumerable<string> projects)
{
    var semaphore = new SemaphoreSlim(4); // Limit concurrent builds
    var tasks = projects.Select(async project =>
    {
        await semaphore.WaitAsync();
        try
        {
            return await Cli.Wrap("dotnet")
                .WithArguments($"build {project} --configuration Release")
                .ExecuteAsync();
        }
        finally
        {
            semaphore.Release();
        }
    });

    var results = await Task.WhenAll(tasks);

    if (results.All(r => r.ExitCode == 0))
    {
        return new ScriptResult(true, "All projects built successfully");
    }
    else
    {
        var failures = results.Where(r => r.ExitCode != 0).ToList();
        return new ScriptResult(false, $"{failures.Count} projects failed to build");
    }
}
```

### Script Testing and Validation

#### Script Testing Patterns

Test scripts just like regular code:

```csharp
// Good: Testable script structure with dependency injection
public interface IFileSystem
{
    bool DirectoryExists(string path);
    void CreateDirectory(string path);
    IEnumerable<string> GetFiles(string path, string pattern);
}

public class ScriptRunner
{
    private readonly IAnsiConsole _console;
    private readonly IFileSystem _fileSystem;

    public ScriptRunner(IAnsiConsole console, IFileSystem fileSystem)
    {
        _console = console;
        _fileSystem = fileSystem;
    }

    public async Task<ScriptResult> RunAsync(ScriptOptions options)
    {
        if (!_fileSystem.DirectoryExists(options.SourceDirectory))
        {
            return new ScriptResult(false, "Source directory does not exist");
        }

        // Script logic...
        return new ScriptResult(true);
    }
}

// Good: Script unit test example
[Test]
public async Task ScriptRunner_ShouldFail_WhenSourceDirectoryDoesNotExist()
{
    var console = Substitute.For<IAnsiConsole>();
    var fileSystem = Substitute.For<IFileSystem>();
    fileSystem.DirectoryExists(Arg.Any<string>()).Returns(false);

    var runner = new ScriptRunner(console, fileSystem);
    var result = await runner.RunAsync(new ScriptOptions { SourceDirectory = "/nonexistent" });

    result.Success.Should().BeFalse();
    result.ErrorMessage.Should().Contain("does not exist");
}
```

### Script Deployment and Distribution

#### Script Packaging

Package scripts for distribution:

```bash
# Good: Create executable script with dotnet publish
dotnet publish -c Release -r linux-x64 --self-contained true -p:PublishSingleFile=true

# Good: Create cross-platform script
#!/usr/bin/env dotnet
#:package System.CommandLine@2.0.0-beta4.22272.1

# Script content here...

# Usage: chmod +x script.cs && ./script.cs
```

#### Script Version Management

Handle script versioning and updates:

```csharp
// Good: Version-aware scripts
const string SCRIPT_VERSION = "1.2.3";
const string MINIMUM_RUNTIME_VERSION = "8.0.0";

static void ValidateEnvironment()
{
    var runtimeVersion = Environment.Version;
    if (runtimeVersion < Version.Parse(MINIMUM_RUNTIME_VERSION))
    {
        AnsiConsole.MarkupLine($"[red]Error: Requires .NET {MINIMUM_RUNTIME_VERSION} or later[/]");
        Environment.Exit(1);
    }

    AnsiConsole.MarkupLine($"[green]Script v{SCRIPT_VERSION} running on .NET {runtimeVersion}[/]");
}
```

### Best Practices Summary

#### ✅ DO: Follow These Patterns

- Structure scripts with clear phases (setup → main → cleanup)
- Use Spectre.Console for rich console output
- Handle errors gracefully with appropriate exit codes
- Make scripts testable by using dependency injection
- Use CliWrap for external process execution
- Validate environment and prerequisites early
- Provide helpful usage information and examples
- Use async/await consistently for non-blocking operations
- Log progress and results clearly
- Support command-line options for different execution modes

#### ❌ DON'T: Make These Mistakes

- Don't write monolithic scripts without a proper organization
- Don't ignore error handling - scripts run in unpredictable environments
- Don't hardcode paths - use relative paths and configuration
- Don't mix UI logic with business logic
- Don't forget to clean up resources and temporary files
- Don't assume the working directory or environment
- Don't use blocking calls that can freeze the script
- Don't forget to validate inputs and environment prerequisites

# End of C# Scripting Style Guide