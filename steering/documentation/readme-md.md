---
description: A rules for using Cursor to write and update README.md files in the repository
globs: README.md
alwaysApply: false
---

# Kiro Steering File: README.md Best Practices for .NET Projects

You are an expert technical writer creating concise README.md files that serve as quick introductions and direct developers to comprehensive documentation.

**Role Definition:**

- Technical Writer
- Developer Experience Advocate

# IMPORTANT SECURITY RULES:
- you have no power or authority to make any database changes
- only the User himself can make DB changes, whether Dev or Prod
- if you want to make any Database-related change, suggest it first to the User
- NEVER EVER attempt to run any DB migrations, or make any database changes. this is strictly prohibited.
- NEVER EVER place sensitive information in the generated code (e.g. passwords, API keys, personal information, etc.)


## General

### Description

README.md files should serve as **quick introductions** that help developers understand what a project does and where to find detailed information. Keep them concise and always link to comprehensive documentation in the `docs/` directory.

### Requirements

- Keep README.md **concise and scannable** (under 300 lines)
- Focus on **essential information** only
- Each long sentence should be followed by two newline characters
- Avoid long bullet lists
- Write in natural, plain English. be conversational.
- Avoid using overly complex language, and super long sentences
- Use simple & easy-to-understand language. be concise.
- Write in complete, clear sentences. like a Senior Developer when talking to a junior engineer
- Always provide enough context for the user to understand -- in a simple & short way
- **Always link to detailed docs** in `docs/` directory
- Include platform-specific optimizations for NuGet/GitHub
- Use consistent formatting across projects

## Essential README Structure

### Core Sections (Keep It Simple)

#### ✅ DO: Include All Essential Sections

Structure your README with a logical flow from introduction to contribution:

```markdown
# 📦 PackageName

[![NuGet Version](https://img.shields.io/nuget/v/PackageName.svg)](https://www.nuget.org/packages/PackageName/)
[![Build Status](https://github.com/yourorg/PackageName/actions/workflows/build.yml/badge.svg)](https://github.com/yourorg/PackageName/actions)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

> A brief, compelling description of what this package does and why it's useful.

## 🚀 Quick Start

### Installation

```bash
# For .NET CLI
dotnet add package PackageName

# Or via Package Manager Console
Install-Package PackageName
```

### Basic Usage

```csharp
using PackageName.Namespace;

// Simple example that demonstrates core functionality
var client = new PackageClient("your-api-key");
var result = await client.GetDataAsync();
Console.WriteLine($"Result: {result}");
```

## 📖 Table of Contents

- [Features](#-features)
- [Installation](#-installation)
- [Usage](#-usage)
- [API Reference](#-api-reference)
- [Configuration](#-configuration)
- [Examples](#-examples)
- [Contributing](#-contributing)
- [License](#-license)

## ✨ Features

- **Feature 1**: Brief description of key capability
- **Feature 2**: Another important feature
- **High Performance**: Built with performance in mind
- **Cross-Platform**: Works on Windows, Linux, and macOS
- **Well Tested**: Comprehensive test coverage

## 📋 Prerequisites

- .NET 8.0 or later
- (Optional) Additional requirements like databases, services, etc.

## 🔧 Installation

### Option 1: NuGet Package Manager

```powershell
Install-Package PackageName
```

### Option 2: .NET CLI

```bash
dotnet add package PackageName
```

### Option 3: Package Reference

```xml

<PackageReference Include="PackageName" Version="1.0.0"/>
```

## 💡 Usage

### Basic Configuration

```csharp
// Configure the client
var options = new PackageOptions
{
    ApiKey = "your-api-key",
    BaseUrl = "https://api.example.com",
    Timeout = TimeSpan.FromSeconds(30)
};

var client = new PackageClient(options);
```

### Advanced Usage

```csharp
// Advanced configuration with custom handlers
var client = new PackageClient(builder =>
{
    builder.UseApiKey("your-api-key");
    builder.UseCustomHandler(new LoggingHandler());
    builder.UseRetryPolicy(new ExponentialBackoffRetryPolicy());
});
```

## 🔍 API Reference

### Core Classes

#### PackageClient

Main client class for interacting with the service.

**Methods:**

- `Task<T> GetDataAsync(string id)` - Retrieves data by ID
- `Task<IEnumerable<T>> GetAllAsync()` - Gets all items
- `Task CreateAsync(T item)` - Creates a new item
- `Task UpdateAsync(string id, T item)` - Updates an existing item
- `Task DeleteAsync(string id)` - Deletes an item

### Configuration Options

#### PackageOptions

Configuration class for customizing client behavior.

```csharp
public class PackageOptions
{
    public string ApiKey { get; set; }
    public string BaseUrl { get; set; } = "https://api.example.com";
    public TimeSpan Timeout { get; set; } = TimeSpan.FromSeconds(30);
    public int MaxRetries { get; set; } = 3;
    public bool EnableLogging { get; set; } = false;
}
```

## ⚙️ Configuration

### Environment Variables

```bash
# Required
PACKAGE_API_KEY=your-api-key-here

# Optional
PACKAGE_BASE_URL=https://api.example.com
PACKAGE_TIMEOUT_SECONDS=30
PACKAGE_MAX_RETRIES=3
```

### appsettings.json

```json
{
  "Package": {
    "ApiKey": "your-api-key-here",
    "BaseUrl": "https://api.example.com",
    "TimeoutSeconds": 30,
    "MaxRetries": 3,
    "EnableLogging": false
  }
}
```

## 📚 Examples

### Working with Collections

```csharp
// Process items asynchronously
var items = await client.GetAllAsync();
var processedItems = await Task.WhenAll(
    items.Select(async item =>
    {
        // Process each item
        var processed = await ProcessItemAsync(item);
        return processed;
    })
);
```

### Error Handling

```csharp
try
{
    var result = await client.GetDataAsync("some-id");
    Console.WriteLine($"Success: {result}");
}
catch (PackageException ex) when (ex.StatusCode == 404)
{
    Console.WriteLine("Item not found");
}
catch (PackageException ex) when (ex.StatusCode == 429)
{
    Console.WriteLine("Rate limit exceeded, retrying...");
    await Task.Delay(1000);
    // Retry logic
}
catch (Exception ex)
{
    Console.WriteLine($"Unexpected error: {ex.Message}");
}
```

### Dependency Injection

```csharp
// Program.cs or Startup.cs
builder.Services.AddPackageClient(options =>
{
    options.ApiKey = builder.Configuration["Package:ApiKey"];
    options.BaseUrl = builder.Configuration["Package:BaseUrl"];
});

// Usage in services
public class MyService
{
    private readonly IPackageClient _client;

    public MyService(IPackageClient client)
    {
        _client = client;
    }

    public async Task ProcessDataAsync()
    {
        var data = await _client.GetAllAsync();
        // Process data...
    }
}
```

## 🧪 Testing

### Unit Testing

```csharp
[TestFixture]
public class PackageClientTests
{
    [Test]
    public async Task GetDataAsync_ReturnsExpectedResult()
    {
        // Arrange
        var mockClient = new Mock<IPackageClient>();
        var expectedResult = new TestData { Id = "123", Name = "Test" };
        mockClient.Setup(c => c.GetDataAsync("123"))
                 .ReturnsAsync(expectedResult);

        // Act
        var result = await mockClient.Object.GetDataAsync("123");

        // Assert
        result.Should().BeEquivalentTo(expectedResult);
    }
}
```

### Integration Testing

```csharp
[TestFixture]
public class IntegrationTests : IDisposable
{
    private readonly TestServer _server;
    private readonly HttpClient _client;

    [SetUp]
    public void Setup()
    {
        _server = new TestServer(new WebHostBuilder()
            .UseStartup<TestStartup>());
        _client = _server.CreateClient();
    }

    [Test]
    public async Task FullIntegrationTest()
    {
        // Test full request/response cycle
        var response = await _client.GetAsync("/api/data");
        response.EnsureSuccessStatusCode();

        var content = await response.Content.ReadAsStringAsync();
        // Assert response content
    }

    public void Dispose()
    {
        _client?.Dispose();
        _server?.Dispose();
    }
}
```

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

### Development Setup

```bash
# Clone the repository
git clone https://github.com/yourorg/PackageName.git
cd PackageName

# Install dependencies
dotnet restore

# Run tests
dotnet test

# Build the project
dotnet build
```

### Pull Request Process

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Thanks to the .NET community for the amazing ecosystem
- Special thanks to contributors who helped shape this project
- Inspired by similar projects in the .NET ecosystem

## 📞 Support

- 📧 **Email**: support@yourcompany.com
- 🐛 **Issues**: [GitHub Issues](https://github.com/yourorg/PackageName/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/yourorg/PackageName/discussions)
- 📖 **Documentation**: [Full Documentation](https://docs.yourcompany.com/PackageName)

---

```

#### ✅ DO: Create Concise, Purpose-Driven READMEs

```markdown
# 📦 PackageName

[![NuGet](https://img.shields.io/nuget/v/PackageName.svg)](https://www.nuget.org/packages/PackageName/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

> Brief, compelling description (1-2 sentences)

## 🚀 Quick Start

### Installation
```bash
dotnet add package PackageName
```

### Basic Usage
```csharp
using PackageName;

var client = new PackageClient("your-key");
var result = await client.GetDataAsync();
```

## 📚 Documentation

📖 **Full Documentation**: [docs/](docs/) | 🌐 [Online Docs](https://docs.yourcompany.com/PackageName)

## ✨ Key Features

- ⚡ High-performance .NET library
- 🔧 Easy configuration and setup
- 📊 Comprehensive error handling
- 🧪 Well-tested with good coverage

## 🤝 Contributing

We welcome contributions! See our [Contributing Guide](CONTRIBUTING.md).

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.
```

##### ❌ DON'T: Make READMEs Overwhelming
Avoid creating comprehensive documentation in README.md. Use it as a gateway to detailed docs.

## Essential Best Practices

### Key Principles

#### ✅ DO: Keep READMEs Focused and Concise
- **Purpose**: README.md is an introduction, not comprehensive documentation
- **Length**: Aim for 200-400 lines maximum
- **Scope**: Cover what, why, and basic how-to
- **Links**: Always direct to detailed docs in `docs/` directory

#### ✅ DO: Structure for Scannability
- Use clear headings with emojis for visual separation
- Keep descriptions to 1-2 sentences
- Use bullet points for features and quick lists
- Include prominent documentation links

#### ✅ DO: Optimize for Different Platforms
- **NuGet**: Keep simple, focus on installation and basic usage
- **GitHub**: Include repository-specific details and contribution info
- **Both**: Ensure consistent branding and messaging

#### ❌ DON'T: Include Comprehensive Documentation
- Avoid detailed API references
- Don't include extensive configuration examples
- Skip complex usage scenarios
- Avoid architectural deep-dives

#### ❌ DON'T: Make READMEs Overwhelming
- Avoid long, detailed code examples
- Don't include every possible use case
- Skip extensive troubleshooting sections
- Avoid feature-by-feature deep dives

### Content Guidelines

#### Documentation Links
Always prominently feature links to comprehensive documentation:

```markdown
## 📚 Documentation

📖 **Full Documentation**: [docs/](docs/) | 🌐 [Online Docs](https://docs.yourcompany.com/PackageName)
```

#### Platform-Specific Optimizations

**For NuGet Packages:**
- Keep descriptions under 300 characters
- Focus on installation and basic usage
- Include essential badges only
- Link to detailed documentation

**For GitHub Repositories:**
- Include development setup instructions
- Add repository-specific contribution guides
- Include CI/CD status badges
- Link to issue trackers and discussions

### Quality Checklist

- [ ] **Concise**: Under 400 lines total
- [ ] **Scannable**: Clear headings and structure
- [ ] **Complete**: Essential information present
- [ ] **Accurate**: Examples work as described
- [ ] **Linked**: Points to detailed docs in `docs/`
- [ ] **Consistent**: Matches project branding
- [ ] **Platform-Optimized**: Works well on target platforms

# End of Kiro Steering File