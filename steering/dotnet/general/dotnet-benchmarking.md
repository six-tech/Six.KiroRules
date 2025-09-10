---
description: Best practices for designing benchmarks and measuring .NET performance
fileMatchPattern: "benchmarks/Directory.Build.props", "*.Benchmark.csproj", "benchmarks/**/*.cs"
inclusion: fileMatch
---

# Kiro Steering File: .NET Benchmarking Best Practices

This file provides comprehensive guidelines for writing effective benchmarks using BenchmarkDotNet
and other performance testing tools in .NET applications.

**Role Definition:**

- Performance Engineer
- .NET Runtime Specialist
- Optimization Expert

# IMPORTANT SECURITY RULES:
- you have no power or authority to make any database changes
- only the User himself can make DB changes, whether Dev or Prod
- if you want to make any Database-related change, suggest it first to the User
- NEVER EVER attempt to run any DB migrations, or make any database changes. this is strictly prohibited.
- NEVER EVER place sensitive information in the generated code (e.g. passwords, API keys, personal information, etc.)


## General

### Description

Performance testing and benchmarking should be systematic, reproducible, and provide meaningful insights into
application performance characteristics. Use BenchmarkDotNet as the primary tool for micro-benchmarking and performance
regression testing to ensure consistent and reliable performance measurements.

### Requirements

- Use steering rules from #[[file:.kiro/steering/dotnet/csharp/csharp-coding-style.md]] for C# coding style and conventions- Use steering rules from #[[file:.kiro/steering/dotnet/csharp/csharp-coding-style.md]] for C# coding style and conventions
- Use `BenchmarkDotNet` for micro-benchmarks and performance testing
- Ensure consistent test environments and hardware configurations
- Follow scientific methodology with proper baseline comparisons
- Track performance metrics over time with regression detection
- Consider memory allocation patterns and garbage collection impact
- Document benchmark results and environment requirements

## Project Setup

### Benchmark Project Configuration

Always create dedicated benchmark projects with optimized settings for accurate performance measurements.

#### ✅ DO: Configure benchmark projects properly

```xml

<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net9.0</TargetFramework>
        <Configuration>Release</Configuration>
        <Optimize>true</Optimize>
        <DebugSymbols>false</DebugSymbols>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="BenchmarkDotNet" Version="0.15.2"/>
        <PackageReference Include="BenchmarkDotNet.Diagnostics.Windows" Version="0.15.2"
                          Condition="'$(OS)' == 'Windows_NT'"/>
    </ItemGroup>
</Project>
```

#### ❌ DON'T: Use debug configuration for benchmarks

```xml
<!-- Don't use Debug configuration for performance benchmarks -->
<PropertyGroup>
    <Configuration>Debug</Configuration>  <!-- This will skew performance results -->
    <DebugSymbols>true</DebugSymbols>
</PropertyGroup>
```

## Benchmark Structure and Configuration

### Basic Benchmark Setup

Create well-structured benchmark classes with proper attributes and configuration.

#### ✅ DO: Use comprehensive benchmark attributes

```csharp
[MemoryDiagnoser]
[RankColumn, MinColumn, MaxColumn, MeanColumn, MedianColumn]
public class StringOperationsBenchmarks
{
    const string TestString = "Hello, World!";
    readonly StringBuilder _builder = new();

    [Params(10, 100, 1000)]
    public int Iterations { get; set; }

    [GlobalSetup]
    public void Setup()
    {
        // Setup code that runs once before all benchmarks
    }

    [Benchmark(Baseline = true)]
    public string StringConcat()
    {
        var result = string.Empty;
        for (int i = 0; i < Iterations; i++)
            result += TestString;
        return result;
    }

    [Benchmark]
    public string StringBuilder()
    {
        _builder.Clear();
        for (int i = 0; i < Iterations; i++)
            _builder.Append(TestString);
        return _builder.ToString();
    }
}
```

#### ✅ DO: Use proper setup and cleanup methods

```csharp
public class DatabaseBenchmarks
{
    private DatabaseContext _context;

    [GlobalSetup]
    public void GlobalSetup()
    {
        _context = new DatabaseContext();
        // Initialize database connection
    }

    [IterationSetup]
    public void IterationSetup()
    {
        // Setup fresh data for each iteration
        _context.ResetData();
    }

    [Benchmark]
    public void QueryPerformance()
    {
        _context.ExecuteQuery();
    }

    [GlobalCleanup]
    public void GlobalCleanup()
    {
        _context.Dispose();
    }
}
```

## Memory Analysis

### Tracking Allocations and GC

Always measure memory allocations and garbage collection impact in benchmarks.

#### ✅ DO: Configure memory diagnostics

```csharp
[MemoryDiagnoser]
[GcServer(true)]
public class MemoryBenchmarks
{
    [Benchmark]
    public IEnumerable<string> AllocatingMethod()
    {
        return Enumerable.Range(0, 1000)
            .Select(i => i.ToString());
    }

    [Benchmark]
    public IEnumerable<string> NonAllocatingMethod()
    {
        return Enumerable.Range(0, 1000)
            .Select(i => i.ToString())
            .ToArray();
    }
}
```

> [!TIP]
> Use `[GcServer(true)]` to simulate server GC behavior in benchmarks, which is more representative of production
> applications.

#### ❌ DON'T: Ignore memory allocation patterns

```csharp
[Benchmark]
// Missing MemoryDiagnoser - won't measure allocations
public void MemoryIntensiveOperation()
{
    var list = new List<string>();
    for (int i = 0; i < 10000; i++)
    {
        list.Add(i.ToString()); // Creates many allocations
    }
}
```

## Hardware Intrinsics and SIMD

### Measuring SIMD Performance

Benchmark hardware-accelerated operations using SIMD instructions.

#### ✅ DO: Compare scalar vs vector operations

```csharp
[SimpleJob(RuntimeMoniker.Net80)]
[RyuJitX64Job]
public class VectorBenchmarks
{
    float[] _data;

    [GlobalSetup]
    public void Setup()
    {
        _data = new float[1024];
        // Initialize data
    }

    [Benchmark(Baseline = true)]
    public float ScalarSum()
    {
        float sum = 0;
        for (int i = 0; i < _data.Length; i++)
            sum += _data[i];
        return sum;
    }

    [Benchmark]
    public float VectorSum()
    {
        return Vector.Sum(_data);
    }
}
```

> [!IMPORTANT]
> Always include both scalar and vector implementations to measure the performance benefit of SIMD operations.

## Async Performance Benchmarking

### Benchmarking Asynchronous Operations

Properly measure async operation performance with appropriate configurations.

#### ✅ DO: Configure async benchmarks correctly

```csharp
public class AsyncBenchmarks
{
    HttpClient _client;

    [GlobalSetup]
    public void Setup()
    {
        _client = new HttpClient();
    }

    [Benchmark]
    public async Task<string> SingleRequest()
    {
        return await _client.GetStringAsync("http://example.com");
    }

    [Benchmark]
    public async Task<string[]> ParallelRequests()
    {
        var tasks = Enumerable.Range(0, 10)
            .Select(_ => _client.GetStringAsync("http://example.com"))
            .ToArray();

        return await Task.WhenAll(tasks);
    }
}
```

#### ❌ DON'T: Mix sync and async inappropriately

```csharp
[Benchmark]
// This will not properly measure async performance
public void SyncOverAsync()
{
    var result = new HttpClient().GetStringAsync("http://example.com").Result;
}
```

## CI/CD Integration

### Configuring Benchmark Runs

Integrate benchmarks into CI/CD pipelines for continuous performance monitoring.

#### ✅ DO: Configure GitHub Actions for benchmarks

```yaml
- name: Run Benchmarks
  run: |
    dotnet run -c Release --filter '*'

- name: Store Results
  uses: actions/upload-artifact@v3
  with:
    name: benchmark-results
    path: BenchmarkDotNet.Artifacts/**
```

#### ✅ DO: Track performance regressions

```csharp
[Config(typeof(RegressionConfig))]
public class RegressionBenchmarks
{
    private class RegressionConfig : ManualConfig
    {
        public RegressionConfig()
        {
            AddExporter(MarkdownExporter.GitHub);
            AddDiagnoser(MemoryDiagnoser.Default);
            AddColumn(StatisticColumn.Median);
            AddColumn(RankColumn.Arabic);
        }
    }
}
```

## Best Practices and Common Pitfalls

### Avoiding Common Benchmarking Mistakes

Follow these best practices to ensure reliable and meaningful benchmark results.

#### ✅ DO: Use proper benchmark isolation

```csharp
// Good: Proper benchmark isolation
[IterationSetup]
public void IterationSetup()
{
    _data = new byte[1024];  // Fresh data for each iteration
}

// Avoid: Shared state between iterations
byte[] _sharedData = new byte[1024];  // Can lead to false results
```

#### ✅ DO: Use appropriate job configurations

```csharp
[SimpleJob(RuntimeMoniker.Net80, baseline: true)]
[SimpleJob(RuntimeMoniker.Net70)]
[SimpleJob(RuntimeMoniker.Net60)]
public class CrossVersionBenchmarks
{
    [Benchmark]
    public void BenchmarkMethod() { }
}
```

#### ✅ DO: Document environment requirements

```csharp
/*
## Required Environment
- Windows 10+ or Linux with perf_event_paranoid <= 2
- CPU: Modern x64 processor with AVX2 support
- RAM: 16GB minimum
- Minimal background processes
*/
```

> [!WARNING]
> Always document the required environment specifications to ensure reproducible results across different machines and
> CI environments.

#### ❌ DON'T: Use inappropriate data types

```csharp
[Benchmark]
// Bad: Using object instead of specific types
public void BoxingBenchmark()
{
    object value = 42;  // Boxing allocation
    int result = (int)value;  // Unboxing
}
```

### IterationSetup and IterationCleanup Method Signatures

**Library Name**: BenchmarkDotNet
**Programming Language**: C#
**Library Version**: (,14.0]
**Severity**: Error (must be followed)

#### Rule Description

Always use `void` methods when defining `IterationSetup` or `IterationCleanup` activities for BenchmarkDotNet classes.

#### Good Examples

```csharp
// Good
[IterationSetup]
public void DoSetup()
{
}

[IterationCleanup]
public void DoCleanup()
{
}
```

#### Bad Examples

```csharp
// Bad - will throw runtime exception
[IterationSetup]
public async Task DoSetup()
{
}

[IterationCleanup]
public async Task DoCleanup()
{
}
```

#### Rationale

While async Task methods will compile, BenchmarkDotNet will throw a runtime exception when these methods are used. The
framework requires these methods to be void methods to function properly.

# End of Kiro Steering File 