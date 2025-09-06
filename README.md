![Six Kiro Rules](.kiro/six-kiro-rules-banner-wide-1980-01.png)

# Six Kiro Rules

A comprehensive collection of Kiro AI assistant rules designed to enhance software development quality, consistency,
and productivity across multiple technology stacks. This project provides structured guidelines covering:

- **Technical documentation** - Standards for writing and maintaining project documentation
- **.NET ecosystem** - ASP.NET, Avalonia, Blazor, CI/CD, and related technologies
- **C# development** - Language-specific coding standards and best practices
- **Web technologies** - Astro, TypeScript, and modern web development frameworks
- **Specification-driven development** - Structured approaches to requirements and implementation
- **General software engineering practices** - Coding standards, testing, benchmarking, dependency management, and package publishing

## What are Kiro Rules?

Kiro Rules are configuration files that guide AI assistants in Kiro IDE to follow specific development practices, coding standards, and workflow patterns. These include:

- **Steering Files** (`.md` format) - Provide contextual guidance and best practices for specific file types or development scenarios
- **Hook Files** (`.kiro.hook` format) - Automate AI assistant actions based on file changes or user interactions

These rules ensure consistent code quality, proper documentation, and adherence to industry best practices across different domains and technologies.

## Installation

### Steering Files
To use these Kiro Steering rules in your project:

1. **Copy the steering rules** - Copy the desired `*.md` rule files from this repository to your project's `.
kiro/steering/` directory
2. **Create the directory** - If the `.kiro/steering/` directory doesn't exist, create it in your solution root
3. **Restart Kiro** - Restart Kiro IDE to ensure the new rules are loaded
4. **Verify activation** - The AI assistant will automatically apply these rules when working on your project

### Hook Files
To use these Kiro Hooks in your project:

1. **Copy the hooks** - Copy the desired `*.kiro.hook` files from this repository to your project's `.kiro/hooks/`
   directory
2. **Create the directory** - If the `.kiro/hooks/` directory doesn't exist, create it in your solution root
3. **Restart Kiro** - Restart Kiro IDE to ensure the new rules are loaded
4. **Verify activation** - The hooks will automatically trigger based on their configured conditions


## Available Steering Rules

### .NET Development

#### 🔧 **General .NET**

##### [Library Development](steering/dotnet/general/dotnet-library.md)
Best practices for creating high-quality .NET libraries with proper metadata, SourceLink integration, and NuGet publishing. Focuses on package discoverability, documentation, and following NuGet conventions for maximum ecosystem compatibility.

##### [Dependency Management](steering/dotnet/general/dotnet-dependency-management.md)
Guidelines for managing NuGet dependencies with security focus, version control, and migration strategies. Addresses package vulnerabilities, license compliance, and maintaining healthy dependency graphs across .NET projects.

##### [Testing Best Practices](steering/dotnet/general/dotnet-testing.md)
Comprehensive testing strategies including unit, integration, and performance testing with xUnit, BenchmarkDotNet, and Coverlet. Emphasizes test isolation, Arrange-Act-Assert patterns, and automated testing workflows.

##### [Benchmarking Guidelines](steering/dotnet/general/dotnet-benchmarking.md)
Performance benchmarking best practices using BenchmarkDotNet for measuring and optimizing .NET code performance. Covers memory analysis, async performance testing, and CI/CD integration for continuous performance monitoring.

##### [Documentation Standards](steering/dotnet/general/dotnet-docs.md)
Standards for writing technical documentation in `docs/steering/content/docs/` with proper structure and formatting. Ensures consistent documentation quality and discoverability across .NET projects.


#### 💻 **C#**
##### [C# Coding Style](steering/dotnet/csharp/csharp-coding-style.md)
Guidelines for writing clean, maintainable, and idiomatic C# code with functional patterns and modern language features. Emphasizes readability, proper abstractions, and leveraging C# 13+ features for optimal code quality.

##### [C# Scripting](steering/dotnet/csharp/csharp-scripting-style.md)
Best practices for C# scripting with .NET, including single-file applications, package directives, and rich console output. Covers Spectre.Console for CLI applications and CliWrap for external process management.



#### 🏗️ **ASP.NET Core Development**
##### [ASP.NET Core API](steering/dotnet/aspnet/aspnet-api.md)
Best practices for building secure, scalable ASP.NET Core APIs using Minimal APIs and modern patterns. Covers dependency injection, error handling, security, performance optimization, and comprehensive API documentation.

##### [Fast Endpoints API](steering/dotnet/aspnet/aspnet-api-fast-endpoints.md)
Guidelines for building high-performance ASP.NET Core APIs using Fast Endpoints framework. Focuses on endpoint architecture, validation, security, performance optimization, and comprehensive testing strategies.

##### [Orleans Development](steering/dotnet/aspnet/aspnet-orleans.md)
Best practices for developing distributed applications using Microsoft Orleans framework. Covers grain development, clustering, persistence, and integration with .NET Aspire for local orchestration.

#### 🎨 **UI & Desktop Development**
##### [Avalonia MVVM](steering/dotnet/avalonia/avalonia-mvvm.md)
Comprehensive guidelines for building cross-platform desktop applications using Avalonia UI with MVVM architecture. Covers reactive programming with ReactiveUI, XAML best practices, and cross-platform compatibility for Windows, macOS, and Linux.

##### [Blazor Development](steering/dotnet/blazor/blazor-coding-style.md)
Best practices for building modern web applications with Blazor, covering component architecture, data binding, performance optimization, and integration with ASP.NET Core APIs for full-stack development.




#### 🔄 **CI/CD & DevOps**
##### [.NET Build System](steering/dotnet/ci-cd/dotnet-gh-workflow-build.md)
Comprehensive best practices for .NET build systems, CI/CD pipelines, and automated processes. Covers cross-platform builds, version management, testing integration, and both GitHub Actions, and Azure DevOps pipelines.

##### [Code Signing](steering/dotnet/ci-cd/dotnet-gh-workflow-code-signing.md)
Security practices for .NET code signing with SignClient integration in CI/CD pipelines. Ensures authenticity, integrity, and compliance through automated signing workflows and certificate management.

##### [NuGet Publishing](steering/dotnet/ci-cd/dotnet-gh-workflow-nuget-publishing.md)
Best practices for publishing high-quality NuGet packages with proper versioning, metadata, and distribution. Covers semantic versioning, package validation, security, and automated publishing workflows.



### 🌐 **Web Technologies**
#### [TypeScript Development](steering/web/typescript/typescript-coding-style.md)
Guidelines for writing type-safe, maintainable TypeScript code with modern JavaScript features and functional patterns. Focuses on proper typing, performance optimization, and scalable architecture for web applications.

#### [Astro Framework](steering/web/astro/astro-coding-style.md)
Best practices for building fast, content-focused websites using Astro framework. Covers component development, performance optimization, and integration with modern web technologies for optimal user experience.



### 📋 **Technical Documentation**
#### [README Documentation](steering/documentation/readme-md.md)
Standards for writing comprehensive README files with proper structure, formatting, and content organization. Ensures consistent documentation quality across all projects and repositories.

#### [AI Agents Documentation](steering/documentation/agents-md.md)
Guidelines for documenting AI agents and their capabilities within development projects. Ensures clear communication of AI-assisted workflows and automated processes.


## Available Hooks

### 📚 **Documentation & Code Quality**

#### [C# Documentation Scanner](hooks/six-template-doc-scan-hook.kiro.hook)
Automatically scans project documentation when working with C# files to provide relevant context and knowledge for better task execution. Triggers when C# files are edited and searches for related documentation in the project to enhance AI assistant responses.

#### [In-Repository Documentation Updater](hooks/six-template-update-docs-hook.kiro.hook)
Automatically updates documentation files when source code, configuration files, or other project files are modified. Ensures documentation stays in sync with codebase changes by reviewing modifications and updating relevant documentation files, code comments, and user-facing guides.





## Writing New Steering Rules Using .meta.md

### What is `.meta.md`?

`.meta.md` represents the **meta-framework** and **standardized structure** used to create consistent, high-quality Kiro rules across the Six Kiro Rules collection. 

It's not a single file, but rather a **methodology and template** that ensures all rules follow the same proven structure for optimal AI assistant guidance.

### Why Use the `.meta.md` Approach?

The `.meta.md` methodology provides several key benefits:

- **🔄 Consistency**: All rules follow the same structure, making them predictable and easy to understand
- **🎯 Clarity**: Clear role definitions and requirements help AI assistants understand their scope
- **📝 Documentation**: Standardized format ensures comprehensive coverage of topics
- **🔧 Maintainability**: Common structure makes rules easier to update and maintain
- **🎨 Best Practices**: Incorporates proven patterns for effective AI-assisted development

### How to Write New Rules Using `.meta.md`

#### 1. **Front Matter Structure**
Every steering rule starts with standardized front matter:

```yaml
---
description: Brief description of what this rule covers
fileMatchPattern: "*.cs", "*.ts", "*.md"  # File patterns where rule applies
inclusion: fileMatch  # Options: always, fileMatch, manual
---
```

#### 2. **Title Format**
Use the standardized title format:
```markdown
# Kiro Steering File: [Descriptive Rule Name]
```

#### 3. **Role Definition Section**
Define the AI assistant's roles and expertise areas:

```markdown
**Role Definition:**
- Primary Expert Role
- Secondary Expert Role
- Specialized Skill Area
```

#### 4. **General Section Structure**
Include these essential subsections:

```markdown
## General

### Description
Clear explanation of the rule's purpose and scope.

### Requirements
- **NEVER**: Critical restrictions (e.g., security, sensitive data)
- Specific technical requirements
- Best practices to follow
- Tools or frameworks to use
```

#### 5. **Domain-Specific Sections**
Organize content into logical sections based on the domain:

```markdown
## [Domain Area 1]
Content specific to the first major area...

## [Domain Area 2]
Content for the second major area...

### [Subsection]
Detailed guidance within a domain area...
```

#### 6. **DO/DON'T Patterns**
Use clear success/failure patterns:

```markdown
### ✅ DO: Recommended Approach
```code
// Good example
```

### ❌ DON'T: Avoid This Pattern
```code
// Bad example to avoid
```
```

#### 7. **File Organization**
Place new rules in appropriate directories:

```
steering/
├── dotnet/           # .NET ecosystem rules
├── web/             # Web technologies
├── specs-generation/# Specification rules
└── documentation/   # Documentation standards
```

### Best Practices for Rule Creation

#### **Content Guidelines**
- **Be Specific**: Use concrete examples and actionable guidance
- **Include Context**: Explain why practices matter, not just what to do
- **Progressive Disclosure**: Start with basics, then advanced concepts
- **Cross-References**: Link to related rules when relevant

#### **Technical Requirements**
- **File Extensions**: Use `.md` for Kiro steering files, `.kiro.hook` for hook files
- **File Match Patterns**: Define specific file patterns for rule activation using `fileMatchPattern`
- **Security First**: Include security considerations in requirements
- **Testing Focus**: Emphasize testing and validation practices

#### **Documentation Standards**
- **Clear Descriptions**: Front matter descriptions should be concise but informative
- **Comprehensive Coverage**: Address common scenarios and edge cases
- **Regular Updates**: Keep rules current with evolving best practices

### Example Rule Template

```markdown
---
description: Guidelines for [specific technology/practice]
fileMatchPattern: "*.extension", "path/patterns/**"
inclusion: fileMatch
---

# Kiro Steering File: [Technology/Practice] Best Practices

**Role Definition:**
- Primary Technology Expert
- Best Practices Specialist
- Implementation Guide

## General

### Description
Brief overview of what this rule covers and why it matters.

### Requirements
- **NEVER**: Critical restrictions
- Key technical requirements
- Best practices to follow

## [Main Topic Area]
Detailed guidance for the primary focus area...

## [Secondary Topic Area]
Additional guidance for related concepts...

## Special Thanks
```

### Rule Validation Checklist

Before finalizing a new rule, ensure it includes:

- [ ] Clear, descriptive front matter
- [ ] Well-defined role definitions
- [ ] Comprehensive requirements section
- [ ] Practical examples and code snippets
- [ ] DO/DON'T patterns for clarity
- [ ] Cross-references to related rules
- [ ] Security considerations
- [ ] Testing guidance
- [ ] Proper file organization

This `.meta.md` approach ensures that all Six Kiro Rules maintain high quality, consistency, and effectiveness in guiding AI-assisted development across the entire technology stack.

## Writing New Hooks

### Hook Structure

Kiro hooks are JSON configuration files with `.kiro.hook` extension that define automated AI assistant actions. Each hook consists of:

#### Basic Hook Template

```json
{
  "enabled": true,
  "name": "Hook Display Name",
  "description": "Brief description of what this hook does",
  "version": "1",
  "when": {
    "type": "fileEdited",
    "patterns": [
      "*.cs",
      "*.ts"
    ]
  },
  "then": {
    "type": "askAgent",
    "prompt": "Instructions for the AI assistant when this hook triggers"
  }
}
```

#### Hook Trigger Types

**File-based Triggers:**
- `fileEdited` - Triggers when specified file patterns are modified
- `fileSaved` - Triggers when files are saved
- `fileCreated` - Triggers when new files are created

**Manual Triggers:**
- `manual` - Creates a button in the UI for manual execution

#### Hook Actions

**AI Assistant Actions:**
- `askAgent` - Sends a prompt to the AI assistant with specific instructions
- `runCommand` - Executes shell commands or scripts

#### Best Practices for Hook Creation

1. **Clear Naming**: Use descriptive names that explain the hook's purpose
2. **Specific Patterns**: Target specific file types to avoid unnecessary triggers
3. **Contextual Prompts**: Include relevant file references using `#[[file:path]]` syntax
4. **Performance**: Consider the frequency of triggers to avoid overwhelming the system
5. **Documentation**: Always include clear descriptions of what the hook accomplishes

#### Example Hook Categories

**Documentation Hooks:**
- Auto-update README files when code changes
- Sync API documentation with code modifications
- Generate changelog entries from commit messages

**Code Quality Hooks:**
- Run tests when source files change
- Update code comments when interfaces change
- Validate coding standards on file save

**Project Management Hooks:**
- Update project metadata when dependencies change
- Sync version numbers across project files
- Generate release notes from version changes

## Special Thanks

**Some of these rules are derived from:**
[.NET Cursor Rules by Aaronontheweb](https://github.com/Aaronontheweb/dotnet-cursor-rules)
[Cursor Official Directory](https://cursor.directory/rules)

## Related Resources

- [Kiro IDE Documentation](https://kiro.dev/docs)
- [C# Coding Standards](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- [NuGet Package Publishing](https://docs.microsoft.com/en-us/nuget/create-packages/overview-and-workflow)
- [Semantic Versioning](https://semver.org/)
- [SPDX License List](https://spdx.org/licenses/)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
