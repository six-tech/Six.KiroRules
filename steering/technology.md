---
description: This file defines the core technologies and frameworks used in .NET projects with Astro.js documentation, including minimum versions and integration guidelines.
inclusion: always
---

# Kiro Steering File: Technology Stack and Framework Guidelines

## General

This document defines the core technology stack for .NET projects with integrated Astro.js documentation. All projects must use .NET 9 or higher as the minimum framework version, with Astro.js serving as the primary documentation platform. This ensures consistency, maintainability, and modern development practices across all solutions.

## Primary Technologies

### .NET Framework

- **Minimum Version**: .NET 9.0
- **Preferred Version**: Latest stable release
- **Language Version**: C# 13
- **Target Framework**: `net9.0` or higher


## In-Repository Documentation Platforms

- **Primary**: Astro.js with Starlight [SIX theme](https://github.com/six-tech/Six.StarlightTheme) and six theme docs: [Six theme docs here](https://six-tech.github.io/Six.StarlightTheme/getting-started/)
- **Content Format**: Markdown (.md) and MDX (.mdx)
- **Location**: `docs/` directory in solution root
  

### In-Repository Documentation Structure

```
docs/
├── astro.config.mjs              # Astro configuration
├── package.json                  # Node.js dependencies
├── src/
│   ├── content/
│   │   └── docs/                 # Documentation pages
│   └── components/               # Custom Astro components
└── public/                       # Static assets
```

###  In-Repository Documentation Development
- **Node.js**: Version 18.0 or higher
- **Package Manager**: pnpm (but it should work with npm, yarn or bun also)
  
#### Astro.js Configuration

Follow guidelines from #[[file:steering/web/astro-coding-style.md]] for:
- Writing new documentation pages
- Custom Astro components
- Component development patterns
- Performance optimization
- Static generation best practices
- SEO and accessibility standards

#### TypeScript Integration

Follow guidelines from #[[file:steering/web/typescript-coding-style.md]] for:
- Custom Astro components









# End of Kiro Steering File