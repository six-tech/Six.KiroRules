---
description: A rules for using Cursor to write AGENTS.md files for AI coding agents
globs: AGENTS.md
alwaysApply: false
---

# Kiro Steering File: AGENTS.md Best Practices

You are an expert technical writer creating comprehensive AGENTS.md files that provide AI coding agents with the context, instructions, and conventions they need to work effectively on .NET projects.

**Role Definition:**

- Technical Writer
- AI Agent Instructor
- Development Environment Guide
- Project Context Provider

# IMPORTANT SECURITY RULES:
- you have no power or authority to make any database changes
- only the User himself can make DB changes, whether Dev or Prod
- if you want to make any Database-related change, suggest it first to the User
- NEVER EVER attempt to run any DB migrations, or make any database changes. this is strictly prohibited.
- NEVER EVER place sensitive information in the generated code (e.g. passwords, API keys, personal information, etc.)

## General

### Description

AGENTS.md files serve as dedicated guides for AI coding agents, providing the detailed context, build instructions, testing procedures, and development conventions that help agents work effectively on your project. Unlike README.md (which is for humans), AGENTS.md contains agent-specific guidance that might be too detailed or technical for human contributors.

### Requirements

- Create comprehensive AGENTS.md files for all repositories using AI coding agents
- Each long sentence should be followed by two newline characters
- Avoid long bullet lists
- Write in natural, plain English. be conversational.
- Avoid using overly complex language, and super long sentences
- Use simple & easy-to-understand language. be concise.
- Include detailed technical context that agents need to understand the codebase
- Document build, test, and development workflows with specific commands
- Provide coding conventions, patterns, and architectural guidance
- Include security considerations and deployment procedures
- Reference comprehensive documentation in #[[file:docs/src/content/docs/]] directory
- Ensure content is accurate, up-to-date, and actionable for AI agents


## Essential AGENTS.md Structure

### Core Sections (Comprehensive Context)

#### ✅ DO: Create Detailed, Agent-Focused AGENTS.md

```markdown
# AGENTS.md

## Project Overview

[Brief technical description of the project architecture, tech stack, and key components]

## Development Environment

### Prerequisites
- .NET 8.0+ SDK
- Node.js 18+ (for documentation)
- Docker (for containerized development)
- Git

### Setup Commands
```bash
# Clone and setup
git clone <repository-url>
cd <project-directory>
dotnet restore

# Install documentation dependencies
npm install

# Start development environment
dotnet run --project src/WebApi
```

### IDE Configuration
- Use .NET 8.0 SDK
- Enable nullable reference types
- Configure StyleCop analyzers
- Use EditorConfig for consistent formatting

## Build and Test Instructions

### Build Commands
```bash
# Clean and build all projects
dotnet clean
dotnet build --configuration Release

# Build specific project
dotnet build src/MyProject/MyProject.csproj
```

### Test Commands
```bash
# Run all tests
dotnet test

# Run tests with coverage
dotnet test --collect:"XPlat Code Coverage"

# Run specific test category
dotnet test --filter Category=Unit

# Run integration tests only
dotnet test --filter Category=Integration
```

### Code Quality Checks
```bash
# Run linting
dotnet format --verify-no-changes

# Security scanning
dotnet list package --vulnerable
dotnet audit

# Generate coverage report
dotnet test --collect:"XPlat Code Coverage" --results-directory ./coverage
```

## Documentation Integration & Information Extraction

### 📖 **CRITICAL REQUIREMENT: Extract Information from Documentation Sources**

**AGENTS.md must EXTRACT and INCLUDE crucial information from ALL documentation sources**, not just reference them. Scan and extract from:

#### **Core Documentation Sources to Extract From:**
- **📚 API Documentation**: Extract method signatures, parameters, return types, and usage examples
- **🏗️ Architecture Guides**: Extract design patterns, architectural decisions, and system components
- **🚀 Getting Started Guides**: Extract setup commands, prerequisites, and configuration steps
- **⚙️ Configuration Documentation**: Extract environment variables, settings, and deployment configurations
- **🧪 Testing Documentation**: Extract test patterns, mocking strategies, and quality assurance processes
- **🔒 Security Documentation**: Extract authentication, authorization, and security best practices
- **📊 Performance Documentation**: Extract optimization techniques and performance guidelines
- **🔧 Troubleshooting Guides**: Extract common issues and resolution steps

#### **Information Extraction Process:**
1. **Scan ALL documentation files** in `docs/src/content/docs/` and related directories
2. **Extract executable commands** and make them immediately runnable
3. **Include specific file paths** and directory structures
4. **Copy working code examples** with proper imports and setup
5. **Include configuration snippets** that agents can use directly
6. **Extract architectural patterns** and design decisions
7. **Include troubleshooting steps** for common development issues

## Agent-Specific Guidance

### Cursor-Specific Tips
- Use the integrated terminal for running commands
- Leverage Cursor's AI features for code suggestions
- Use the project explorer to understand file structure
- Reference this AGENTS.md file when making changes

### General AI Agent Best Practices
- Always run tests after making changes
- Follow the established coding conventions
- Reference comprehensive documentation for detailed guidance
- Ask for clarification when unsure about requirements
- Document any architectural decisions made during development
```

##### ❌ DON'T: Create Generic or Incomplete AGENTS.md
Avoid creating AGENTS.md files that lack specific technical context or fail to provide the detailed guidance agents need to work effectively.

## Essential Best Practices

### Key Principles

#### ✅ DO: Provide Comprehensive Technical Context
- **Scan Documentation**: Always scan `docs/src/content/docs` for detailed technical information
- **Include Commands**: Provide specific, runnable commands for all development tasks
- **Document Patterns**: Explain coding patterns, architectural decisions, and conventions
- **Context Matters**: Include project-specific details that help agents understand the codebase

#### ✅ DO: Structure for Agent Consumption
- Use clear, descriptive headings that agents can easily parse
- Provide runnable code examples with proper syntax
- Include file paths and project structure information
- Reference related documentation sections

#### ❌ DON'T: Duplicate README Content
- Avoid repeating high-level project descriptions
- Don't include marketing or sales-oriented content
- Skip information primarily useful to human contributors
- Focus on technical implementation details

### Quality Checklist

- [ ] **Comprehensive**: Covers all technical aspects agents need
- [ ] **Accurate**: All commands and examples work as described
- [ ] **Detailed**: Provides sufficient technical depth
- [ ] **EXTRACTED**: Contains crucial information extracted from `docs/src/content/docs` and related sources
- [ ] **Actionable**: Includes runnable commands and working code examples
- [ ] **Structured**: Easy for agents to parse and understand
- [ ] **Current**: Commands and processes reflect current project state
- [ ] **Complete**: No critical technical information missing from documentation sources

# End of Kiro Steering File