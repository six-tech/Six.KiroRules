---
description: A rules for using Cursor to write and update in-repository technical documentation files
fileMatchPattern: "/docs/astro.config", "/docs/**/*.md", "/docs/**/*.mdx"
inclusion: fileMatch
---

# Kiro Steering File: .NET Technical Documentation in In-Repository Docs

You are an expert software developer creating technical content for other developers. Your task is to produce clear, in-depth technical documentation that provides practical, implementable knowledge.

## Role Definition

- Technical Writer
- Documentation Specialist
- Software Architect
- .NET Developer

## General

### Description

This document outlines the rules on how to write or update technical documentation for functionalities and features in this repository.

When developing or updating existing functionality, good technical documentation is invaluable to the engineers that will use these new or updated code, ensuring alignment and reducing development iterations.

When asked to generate technical documentation for a functionality or a feature, Cursor should search if there is existing documentation or if it has to create a new one.

### Requirements

**- NEVER: Place sensitive information in the documentation (e.g. passwords, API keys, personal information, etc.)**

- Maintain a #[[file:docs/astro.config]] for repository documentation website (uses Astro.js with Starlight for documentation)
- Maintain (write new and update existing documentation pages) in #[[file:docs/src/content/docs/]] directory
- Prefer `.mdx` format if custom components are needed (if Markdown is not enough). Otherwise, use standard `.md` format.
- Use astro.js steering rules for writing documentation pages and organization from #[[file:.kiro/steering/web/astro-coding-style.md]]
- Use typecript steering rules for writing documentation pages and organization from #[[file:.kiro/steering/web/typescript-coding-style.md]]


## Solution Structure

> [!IMPORTANT]
>
> **DO NOT create new documentation files without first checking existing ones**. If documentation for feature or functionality already exists, update it instead of creating a new one.
>
> When new page is created, ALWAYS add it to the #[[file:docs/astro.config]] file.

### In-Repository Documentation File Structure and Hierarchy

```
Repository (solution) root
└── docs/                                             # Repository documentation (astro.js/starlight)
    ├── astro.config                                  # Standard Astro.js/Starlight configuration.
    └── src/                                          
        └── content/                                  
            └── docs/                                 # Directory where all documentation pages are located
```

### File Responsibilities

#### Astro Configuration in #[[file:docs/astro.config]]

This file is used to configure the Astro.js/Starlight website. It contains the list of all documentation pages to be
rendered and the configuration for the website.

**Rules for adding new documentation pages to the `docs/astro.config`:**

- When defining the `sidebar` in #[[file:docs/astro.config]], organize pages in logical sections, hierarchically organized in the sidebar tree-view. ALWAYS provide a clean and structured hierarchical organization. If a functionality has several chapters (pages), group them under the parent functionality
- When defining the `sidebar` in #[[file:docs/astro.config]], organize pages as a guided tour of the functionality. Therefore, the first page should be the main introduction to the functionality and the last page should be the most complex use case.

#### Documentation pages in #[[file:docs/src/content/docs/]]

#[[file:docs/src/content/docs/]] directory contains all documentation pages for the repository.

These rules describe a logical progression with decision points between phases of how to organize documentation pages,
ensuring each step is properly completed before moving to the next.

- ALWAYS think in "chapters." Each functionality is its own chapter and therefore needs its own page so readers can be focused on it and to avoid searching through long pages.
- If documentation already exists for a specific functionality, examine it and update its contents to reflect new changes. The documentation should be written and observed for continuous refinement and updates when functionality changes.
- If documentation doesn't exist for a specific functionality, write a new one.
- Always group pages in subdirectories based on how they are organized in sidebar (in #[[file:docs/astro.config]])
- Each page is a Markdown file with `.md` or `.mdx` extension. If richer content is needed, use `.mdx` format to be able to use custom components.
- ALWAYS add frontmatter to each page with the `title` and `description`.

## Writing Documentation

### Page Structure

- All main headings should start with H1 (`#`).
- Each page should be organized in a logical section, hierarchically organized in the sidebar tree-view.
- Each page should be a guided tour of the functionality. Therefore, the first page should be the main introduction to
  the functionality and the last page should be the most complex use case.
- Each page should be written in a logical progression, from the basics (for example, how to install, setup, create),
  then how to use each part, document the technical architecture, up to the most complex use cases. These sections
  should be structured for easy understanding.
- Each page should provide code examples of concrete usage.
- Each page should provide DO-s and DON'T-s for security or performance important sections. Always guide the user how to properly implement secure and performant solutions.

## Documentation Best Practices and Examples

### DO and DON'T Guidelines

#### Page Structure and Organization

##### ✅ DO: Use Logical Chapter-Based Organization
Organize complex features into multiple focused pages that build upon each other:

```markdown
# API Authentication

## Overview
This guide covers implementing secure API authentication using JWT tokens.

## Basic Setup
Configure your authentication middleware...

## Token Management
Handle token generation, validation, and refresh...

## Advanced Security
Implement rate limiting, token rotation, and security headers...

## Integration Examples
Real-world integration with your API endpoints...
```

##### ❌ DON'T: Create Monolithic Single Pages
Avoid overwhelming readers with everything in one long document:

```markdown
# Everything About Our API

## Introduction
Our API does authentication, user management, data processing, caching, logging, monitoring, deployment, scaling, security, performance optimization, error handling, testing, documentation, support, and more...

## Authentication
[500 lines of mixed content about basic auth, JWT, OAuth, SAML, custom providers...]

## Security
[300 lines mixing authentication security with data encryption, HTTPS, CORS, CSRF...]

## Performance
[400 lines covering caching, database optimization, API rate limiting, CDN setup...]
```

#### Writing Style and Clarity

##### ✅ DO: Write Technical, Implementation-Focused Content
Focus on practical implementation details with clear explanations:

```markdown
## Implementing Repository Pattern

The repository pattern abstracts data access logic from business logic. Create an interface that defines data operations, then implement it for each data source.

```csharp
public interface IUserRepository
{
    Task<User> GetByIdAsync(Guid id);
    Task<IEnumerable<User>> GetByEmailDomainAsync(string domain);
    Task CreateAsync(User user);
    Task UpdateAsync(User user);
    Task DeleteAsync(Guid id);
}

public class SqlUserRepository : IUserRepository
{
    readonly DbContext _context;

    public SqlUserRepository(DbContext context)
    {
        _context = context;
    }

    public async Task<User> GetByIdAsync(Guid id)
    {
        return await _context.Users
            .Include(u => u.Profile)
            .FirstOrDefaultAsync(u => u.Id == id);
    }

    // Additional implementations...
}
```

This approach provides several benefits:
- Decouples business logic from data access implementation
- Enables easy unit testing with mock repositories
- Supports multiple data sources (SQL, NoSQL, in-memory) through interface implementation
- Centralizes data access patterns across the application
```

##### ❌ DON'T: Write Generic Marketing-Style Content
Avoid vague, non-technical descriptions that don't help developers:

```markdown
## Repository Pattern

The repository pattern is a great way to handle data access in your application. It's very popular and widely used by many developers around the world. This pattern helps you organize your code better and makes your application more maintainable and scalable.

You can use it with any kind of database or data source. It's flexible and powerful. Many successful companies use this pattern in their applications.

Here's some code:

```csharp
public interface IUserRepository
{
    User GetById(Guid id);
    void Save(User user);
}
```

This pattern is crucial for modern application development and will enhance your architecture significantly.
```

#### Code Examples Quality

##### ✅ DO: Provide Complete, Runnable Code Examples
Include all necessary imports, error handling, and context:

```csharp
using Microsoft.EntityFrameworkCore;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace MyApp.Data.Repositories
{
    public class ProductRepository : IProductRepository
    {
        private readonly ApplicationDbContext _context;

        public ProductRepository(ApplicationDbContext context)
        {
            _context = context ?? throw new ArgumentNullException(nameof(context));
        }

        public async Task<Product> GetByIdAsync(Guid id)
        {
            try
            {
                return await _context.Products
                    .Include(p => p.Category)
                    .Include(p => p.Tags)
                    .FirstOrDefaultAsync(p => p.Id == id);
            }
            catch (Exception ex)
            {
                // Log the exception
                throw new RepositoryException("Failed to retrieve product", ex);
            }
        }

        public async Task<IEnumerable<Product>> GetByCategoryAsync(Guid categoryId, int page = 1, int pageSize = 20)
        {
            return await _context.Products
                .Where(p => p.CategoryId == categoryId && p.IsActive)
                .OrderBy(p => p.Name)
                .Skip((page - 1) * pageSize)
                .Take(pageSize)
                .ToListAsync();
        }
    }
}
```

##### ❌ DON'T: Provide Incomplete Code Snippets
Avoid code fragments that leave developers guessing:

```csharp
// Don't do this:
public class ProductRepository
{
    public Product GetById(Guid id)
    {
        return _context.Products.Find(id);  // Missing includes, error handling, async
    }

    // Where's the interface? Constructor? Using statements?
    // How do I handle exceptions? What's the return type for collections?
}
```

#### Frontmatter and Metadata

##### ✅ DO: Include Comprehensive Frontmatter
Provide detailed metadata for better discoverability:

```markdown
---
title: Implementing CQRS with MediatR
description: A comprehensive guide to implementing Command Query Responsibility Segregation (CQRS) pattern using MediatR in ASP.NET Core applications, including commands, queries, handlers, and pipeline behaviors.
keywords: [CQRS, MediatR, ASP.NET Core, Commands, Queries, Clean Architecture]
author: Development Team
tags: [architecture, patterns, aspnetcore, mediatr]
prerequisites: [ASP.NET Core fundamentals, Dependency Injection, Basic CQRS concepts]
---

# Implementing CQRS with MediatR
```

##### ❌ DON'T: Use Minimal or Missing Frontmatter
Avoid inadequate metadata that hurts discoverability:

```markdown
---
title: CQRS
---

# CQRS Pattern

Some information about CQRS...
```

#### Security and Performance Guidance

##### ✅ DO: Include Specific Security Guidance with Examples
Provide actionable security recommendations with concrete examples:

```markdown
## Secure JWT Token Implementation

### Token Storage
Never store JWT tokens in localStorage due to XSS vulnerabilities:

```javascript
// ❌ DON'T: Vulnerable to XSS attacks
localStorage.setItem('authToken', token);

// ✅ DO: Use httpOnly cookies for automatic transmission
// Set cookie from server with httpOnly flag
```

### Token Validation
Always validate tokens on the server side:

```csharp
public class JwtAuthenticationHandler : AuthenticationHandler<JwtBearerOptions>
{
    protected override async Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        var token = Request.Cookies["authToken"];

        if (string.IsNullOrEmpty(token))
        {
            return AuthenticateResult.Fail("Missing token");
        }

        try
        {
            var principal = ValidateToken(token);
            var ticket = new AuthenticationTicket(principal, Scheme.Name);
            return AuthenticateResult.Success(ticket);
        }
        catch (SecurityTokenExpiredException)
        {
            return AuthenticateResult.Fail("Token expired");
        }
        catch (Exception ex)
        {
            return AuthenticateResult.Fail($"Token validation failed: {ex.Message}");
        }
    }
}
```

### Rate Limiting
Implement rate limiting to prevent abuse:

```csharp
[HttpPost("login")]
[EnableRateLimiting("LoginPolicy")]
public async Task<IActionResult> Login([FromBody] LoginRequest request)
{
    // Login logic...
}
```

### Rolling Technical Documentation Concept

The rolling technical documentation concept ensures continuous refinement and synchronization with evolving codebases. This iterative approach maintains documentation accuracy and practical value.

#### Implementation Strategy
- **Version Tracking**: Update documentation alongside code changes using the same pull request
- **Change Documentation**: Document breaking changes, new features, and deprecations immediately
- **Review Integration**: Include documentation updates in code review processes
- **Validation**: Regularly validate examples against current codebase to ensure they remain functional

#### Practical Examples

##### ✅ DO: Update Documentation with Code Changes
When adding a new parameter to an API endpoint:

```csharp
// Code change in PR #123
[HttpPost("users")]
public async Task<IActionResult> CreateUser([FromBody] CreateUserRequest request, bool sendWelcomeEmail = false)
{
    // Implementation...
}
```

Update documentation in the same PR:

```markdown
## Creating Users

Send a POST request to `/users` to create a new user account.

### Parameters
- `name` (string, required): User's full name
- `email` (string, required): User's email address
- `department` (string, optional): User's department
- `sendWelcomeEmail` (boolean, optional): Whether to send welcome email (default: false) *[Added in v2.1]*

### Example Request
```json
{
  "name": "John Doe",
  "email": "john.doe@company.com",
  "department": "Engineering",
  "sendWelcomeEmail": true
}
```
```

##### ❌ DON'T: Let Documentation Become Stale
Avoid documentation that doesn't reflect current implementation:

```markdown
// Current code (v2.1)
[HttpPost("users")]
public async Task<IActionResult> CreateUser([FromBody] CreateUserRequest request, bool sendWelcomeEmail = false)

// Outdated documentation (still shows v1.0 API)
## Creating Users

Send a POST request to `/users` to create a new user account.

### Parameters
- `name` (string, required): User's full name
- `email` (string, required): User's email address
```

## Documentation Types and Templates

### API Reference Documentation

#### Structure Template
```markdown
---
title: User Management API
description: Complete API reference for user management operations including authentication, profiles, and permissions.
---

# User Management API

## Authentication

### POST /auth/login
Authenticate a user and return access tokens.

**Parameters:**
- `username` (string): User's username or email
- `password` (string): User's password

**Response:**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "refresh_token_here",
  "expiresIn": 3600
}
```

**Error Responses:**
- `400 Bad Request`: Invalid credentials
- `429 Too Many Requests`: Rate limit exceeded

## User Profiles

### GET /users/{id}
Retrieve user profile information.

**Path Parameters:**
- `id` (UUID): User identifier

**Query Parameters:**
- `include` (string): Comma-separated list of related data to include

**Response:**
```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "name": "John Doe",
  "email": "john.doe@company.com",
  "department": "Engineering",
  "createdAt": "2024-01-15T10:30:00Z"
}
```
```



### Client Library Documentation

#### Structure Template
```markdown
---
title: Six.Keycloak Admin Client
description: .NET client library for managing Keycloak realms, users, and authentication through the Keycloak Admin REST API.
---

# Six.Keycloak Admin Client

A .NET client library that provides a strongly-typed interface for managing Keycloak realms, users, roles, and authentication flows.

## Installation

```bash
dotnet add package Six.Keycloak
```

## Quick Start

```csharp
using Six.Keycloak;

// Configure the client
var client = new KeycloakAdminClient(new KeycloakOptions
{
    BaseUrl = "https://keycloak.example.com",
    Realm = "master",
    Username = "admin",
    Password = "admin123"
});

// Use the client
var realms = await client.GetRealmsAsync();
```

## Client Configuration

### KeycloakOptions

```csharp
public class KeycloakOptions
{
    public string BaseUrl { get; set; }          // Keycloak server URL
    public string Realm { get; set; }            // Admin realm (usually "master")
    public string Username { get; set; }         // Admin username
    public string Password { get; set; }         // Admin password
    public TimeSpan Timeout { get; set; }        // Request timeout (default: 30s)
    public bool VerifySsl { get; set; }          // SSL certificate validation (default: true)
}
```

## Authentication Methods

### LoginAsync
Authenticates with the Keycloak server and obtains an access token.

```csharp
public async Task<LoginResult> LoginAsync()
```

**Returns:**
```csharp
public class LoginResult
{
    public bool Success { get; set; }
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
    public DateTime ExpiresAt { get; set; }
    public string Error { get; set; }
}
```

**Example:**
```csharp
var loginResult = await client.LoginAsync();
if (loginResult.Success)
{
    Console.WriteLine($"Logged in successfully. Token expires: {loginResult.ExpiresAt}");
}
else
{
    Console.WriteLine($"Login failed: {loginResult.Error}");
}
```

### RefreshTokenAsync
Refreshes an expired access token using the refresh token.

```csharp
public async Task<TokenRefreshResult> RefreshTokenAsync(string refreshToken)
```

**Parameters:**
- `refreshToken` (string): The refresh token obtained from login

**Returns:**
```csharp
public class TokenRefreshResult
{
    public bool Success { get; set; }
    public string NewAccessToken { get; set; }
    public string NewRefreshToken { get; set; }
    public DateTime ExpiresAt { get; set; }
    public string Error { get; set; }
}
```

## Realm Management

### GetRealmsAsync
Retrieves all realms accessible to the authenticated user.

```csharp
public async Task<IEnumerable<Realm>> GetRealmsAsync()
```

**Returns:** Collection of `Realm` objects representing available realms.

**Example:**
```csharp
var realms = await client.GetRealmsAsync();
foreach (var realm in realms)
{
    Console.WriteLine($"Realm: {realm.Name} - {realm.DisplayName}");
}
```

### CreateRealmAsync
Creates a new realm with the specified configuration.

```csharp
public async Task<RealmOperationResult> CreateRealmAsync(RealmConfig config)
```

**Parameters:**
- `config` (RealmConfig): Configuration for the new realm

**Returns:**
```csharp
public class RealmOperationResult
{
    public bool Success { get; set; }
    public string RealmId { get; set; }
    public string Error { get; set; }
}
```

**Example:**
```csharp
var config = new RealmConfig
{
    Name = "my-application",
    DisplayName = "My Application",
    Enabled = true,
    RegistrationAllowed = true
};

var result = await client.CreateRealmAsync(config);
if (result.Success)
{
    Console.WriteLine($"Realm created with ID: {result.RealmId}");
}
```

### UpdateRealmAsync
Updates an existing realm's configuration.

```csharp
public async Task<OperationResult> UpdateRealmAsync(string realmName, RealmConfig config)
```

**Parameters:**
- `realmName` (string): Name of the realm to update
- `config` (RealmConfig): Updated configuration

### DeleteRealmAsync
Deletes an existing realm.

```csharp
public async Task<OperationResult> DeleteRealmAsync(string realmName)
```

**Parameters:**
- `realmName` (string): Name of the realm to delete

## User Management

### GetUsersAsync
Retrieves users from a specific realm with optional filtering.

```csharp
public async Task<IEnumerable<User>> GetUsersAsync(string realmName, UserQueryParameters parameters = null)
```

**Parameters:**
- `realmName` (string): Target realm name
- `parameters` (UserQueryParameters, optional): Query filters

**Query Parameters:**
```csharp
public class UserQueryParameters
{
    public string Search { get; set; }          // Search in username, first/last name, email
    public string Username { get; set; }        // Exact username match
    public string Email { get; set; }           // Email address
    public bool? EmailVerified { get; set; }    // Email verification status
    public int? First { get; set; }             // Pagination offset
    public int? Max { get; set; }               // Maximum results (default: 100)
}
```

**Example:**
```csharp
// Get first 50 users
var users = await client.GetUsersAsync("my-realm", new UserQueryParameters
{
    Max = 50,
    EmailVerified = true
});

// Search for users
var searchResults = await client.GetUsersAsync("my-realm", new UserQueryParameters
{
    Search = "john.doe"
});
```

### CreateUserAsync
Creates a new user in the specified realm.

```csharp
public async Task<UserOperationResult> CreateUserAsync(string realmName, User user)
```

**Parameters:**
- `realmName` (string): Target realm name
- `user` (User): User object with profile information

**Example:**
```csharp
var newUser = new User
{
    Username = "john.doe",
    Email = "john.doe@example.com",
    FirstName = "John",
    LastName = "Doe",
    Enabled = true,
    EmailVerified = false
};

var result = await client.CreateUserAsync("my-realm", newUser);
if (result.Success)
{
    Console.WriteLine($"User created with ID: {result.UserId}");
}
```

### UpdateUserAsync
Updates an existing user's profile information.

```csharp
public async Task<OperationResult> UpdateUserAsync(string realmName, string userId, User user)
```

### DeleteUserAsync
Deletes a user from the realm.

```csharp
public async Task<OperationResult> DeleteUserAsync(string realmName, string userId)
```

## Role Management

### GetRolesAsync
Retrieves all roles defined in a realm.

```csharp
public async Task<IEnumerable<Role>> GetRolesAsync(string realmName)
```

### AssignRoleToUserAsync
Assigns one or more roles to a user.

```csharp
public async Task<OperationResult> AssignRoleToUserAsync(string realmName, string userId, IEnumerable<string> roleNames)
```

**Example:**
```csharp
var result = await client.AssignRoleToUserAsync("my-realm", "user-123", new[]
{
    "admin",
    "user_manager"
});

if (result.Success)
{
    Console.WriteLine("Roles assigned successfully");
}
```

## Error Handling

The client library provides comprehensive error handling through custom exceptions:

### KeycloakException
Base exception for all Keycloak-related errors.

```csharp
public class KeycloakException : Exception
{
    public int StatusCode { get; }
    public string ErrorCode { get; }
    public string ErrorDescription { get; }
}
```

### Specific Exceptions
- `AuthenticationException`: Authentication failures
- `RealmNotFoundException`: Referenced realm doesn't exist
- `UserNotFoundException`: Referenced user doesn't exist
- `RoleNotFoundException`: Referenced role doesn't exist
- `ValidationException`: Invalid input parameters

### Error Handling Example
```csharp
try
{
    var user = await client.GetUserAsync("my-realm", "user-123");
    Console.WriteLine($"User: {user.Username}");
}
catch (AuthenticationException ex)
{
    Console.WriteLine($"Authentication failed: {ex.Message}");
    // Re-authenticate or refresh token
}
catch (UserNotFoundException ex)
{
    Console.WriteLine($"User not found: {ex.Message}");
    // Handle missing user scenario
}
catch (KeycloakException ex)
{
    Console.WriteLine($"Keycloak error ({ex.StatusCode}): {ex.ErrorDescription}");
}
```

## Advanced Configuration

### Retry Policy
Configure automatic retries for transient failures:

```csharp
var client = new KeycloakAdminClient(options, new RetryPolicy
{
    MaxRetries = 3,
    InitialDelay = TimeSpan.FromSeconds(1),
    BackoffMultiplier = 2.0
});
```

### Logging
Enable detailed logging for debugging:

```csharp
using Microsoft.Extensions.Logging;

var logger = LoggerFactory.Create(builder =>
{
    builder.AddConsole();
    builder.SetMinimumLevel(LogLevel.Debug);
});

var client = new KeycloakAdminClient(options, logger: logger);
```

### Connection Pooling
For high-throughput applications, configure connection pooling:

```csharp
var client = new KeycloakAdminClient(new KeycloakOptions
{
    // ... other options
    ConnectionPoolSize = 20,
    ConnectionTimeout = TimeSpan.FromSeconds(10)
});
```

## Best Practices

### Authentication
- Cache access tokens and automatically refresh when expired
- Handle token refresh failures gracefully with fallback strategies
- Store credentials securely using platform-specific credential managers

### Error Handling
- Implement exponential backoff for retry scenarios
- Log errors with appropriate severity levels
- Provide meaningful error messages to application users

### Performance
- Use pagination for large result sets
- Implement connection pooling for concurrent requests
- Cache frequently accessed data when appropriate

### Security
- Always use HTTPS in production environments
- Validate SSL certificates unless in development
- Rotate credentials regularly
- Limit scope of admin operations to minimum required

## Migration Guide

### From v1.x to v2.x

**Breaking Changes:**
- `KeycloakOptions.AdminRealm` renamed to `KeycloakOptions.Realm`
- `CreateUser()` method now returns `UserOperationResult` instead of `string`
- Exception hierarchy simplified with new base `KeycloakException`

**Migration Example:**
```csharp
// v1.x
var options = new KeycloakOptions
{
    BaseUrl = "https://keycloak.example.com",
    AdminRealm = "master"  // Changed in v2.x
};

// v2.x
var options = new KeycloakOptions
{
    BaseUrl = "https://keycloak.example.com",
    Realm = "master"  // New property name
};
```

# End of Kiro Steering File