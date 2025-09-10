---
description: Best practices for building high-performance ASP.NET Core APIs using Fast Endpoints, focusing on developer experience and maintainability
fileMatchPattern: "*.cs", "*.csproj", "Program.cs", "appsettings.json"
inclusion: manual
---

# Kiro Steering File: ASP.NET Core API Development with Fast Endpoints

## Role Definition

- Senior .NET Backend Developer
- Fast Endpoints Expert
- ASP.NET Core Specialist
- API Performance Architect

# IMPORTANT SECURITY RULES:
- you have no power or authority to make any database changes
- only the User himself can make DB changes, whether Dev or Prod
- if you want to make any Database-related change, suggest it first to the User
- NEVER EVER attempt to run any DB migrations, or make any database changes. this is strictly prohibited.
- NEVER EVER place sensitive information in the generated code (e.g. passwords, API keys, personal information, etc.)


## General

### Description

Build high-performance, maintainable ASP.NET Core APIs using Fast Endpoints - a developer-friendly alternative to
traditional controllers and Minimal APIs. Fast Endpoints provide better performance, cleaner code organization, and
enhanced developer experience while maintaining full ASP.NET Core compatibility. Focus on the proper endpoint structure,
validation, error handling, and performance optimization.

### Requirements

- **USE** Fast Endpoints library exclusively for API development
- **DO NOT** use Minimal APIs or traditional controllers for API development
- Use steering rules from #[[file:.kiro/steering/dotnet/csharp/csharp-coding-style.md]] for C# coding style and conventions
- Use steering rules from #[[file:.kiro/steering/dotnet/general/dotnet-testing.md]] for testing guidelines and best practices
- Leverage built-in ASP.NET Core features and middleware
- Use Entity Framework Core effectively when database operations are required
- Follow RESTful API design principles with proper HTTP status codes
- Implement comprehensive security measures and input validation

## Project Setup and Configuration

### Fast Endpoints Installation and Setup

Install and configure Fast Endpoints for optimal performance and developer experience.

#### ✅ DO: Install Fast Endpoints with proper configuration

```bash
# Install Fast Endpoints
dotnet add package FastEndpoints
dotnet add package FastEndpoints.Swagger
```

```csharp
// Program.cs - Fast Endpoints setup
var builder = WebApplication.CreateBuilder(args);

// Add Fast Endpoints services
builder.Services.AddFastEndpoints();
builder.Services.AddSwaggerDoc(); // Add Swagger support

var app = builder.Build();

// Use Fast Endpoints
app.UseFastEndpoints();
app.UseSwaggerGen(); // Enable Swagger UI

app.Run();
```

#### ❌ DON'T: Mix Fast Endpoints with controllers or Minimal APIs

```csharp
// DON'T mix approaches - choose one
builder.Services.AddFastEndpoints();
// builder.Services.AddControllers(); // DON'T do this
// DON'T add Minimal API mappings alongside Fast Endpoints
```

## Endpoint Architecture and Design

### Fast Endpoints Structure

Create well-structured endpoints following Fast Endpoints conventions.

#### ✅ DO: Create properly structured Fast Endpoints

```csharp
// Request DTO
public class CreateProductRequest
{
    [Required]
    [StringLength(100, MinimumLength = 1)]
    public string Name { get; set; } = string.Empty;

    [Range(0.01, double.MaxValue)]
    public decimal Price { get; set; }

    [Required]
    public Guid CategoryId { get; set; }
}

// Response DTO
public class CreateProductResponse
{
    public Guid Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public DateTime CreatedAt { get; set; }
}

// Endpoint Definition
public class CreateProductEndpoint : Endpoint<CreateProductRequest, CreateProductResponse>
{
    private readonly IProductService _productService;

    public CreateProductEndpoint(IProductService productService)
    {
        _productService = productService;
    }

    public override void Configure()
    {
        Post("/api/products");
        AllowAnonymous(); // Or configure authorization
        Description(b => b
            .WithName("CreateProduct")
            .WithSummary("Creates a new product")
            .WithDescription("Creates a new product in the catalog"));
    }

    public override async Task HandleAsync(CreateProductRequest req, CancellationToken ct)
    {
        var result = await _productService.CreateProductAsync(req, ct);

        await SendCreatedAtAsync(
            new CreateProductResponse
            {
                Id = result.Id,
                Name = result.Name,
                Price = result.Price,
                CreatedAt = result.CreatedAt
            },
            $"/api/products/{result.Id}",
            cancellation: ct);
    }
}
```

#### ❌ DON'T: Create endpoints without proper structure

```csharp
// DON'T create unstructured endpoints
public class BadProductEndpoint : Endpoint<object, object>
{
    public override void Configure()
    {
        Post("/api/products");
        // Missing validation, documentation, proper types
    }

    public override async Task HandleAsync(object req, CancellationToken ct)
    {
        // No proper request/response types - bad practice
        await SendAsync(new { message = "Created" });
    }
}
```

> [!TIP]
> **Fast Endpoints Benefits:**
>
> - Strongly-typed request/response objects with automatic validation
> - Built-in OpenAPI/Swagger documentation generation
> - Better performance than traditional controllers
> - Cleaner separation of concerns with dedicated endpoint classes
> - Automatic model binding and validation
> - Built-in support for dependency injection

## Validation and Error Handling

### Request Validation

Implement comprehensive validation using Fast Endpoints' built-in features.

#### ✅ DO: Use Fast Endpoints validation features

```csharp
public class UpdateProductRequest
{
    [Required]
    public Guid Id { get; set; }

    [StringLength(100, MinimumLength = 1)]
    public string? Name { get; set; }

    [Range(0.01, double.MaxValue)]
    public decimal? Price { get; set; }
}

public class UpdateProductEndpoint : Endpoint<UpdateProductRequest>
{
    private readonly IProductService _productService;

    public UpdateProductEndpoint(IProductService productService)
    {
        _productService = productService;
    }

    public override void Configure()
    {
        Put("/api/products/{Id}");
        AllowAnonymous();
    }

    public override async Task HandleAsync(UpdateProductRequest req, CancellationToken ct)
    {
        // Fast Endpoints automatically validates the request
        // If validation fails, it returns 400 Bad Request automatically

        var result = await _productService.UpdateProductAsync(req, ct);
        await SendOkAsync(result, ct);
    }
}
```

#### ✅ DO: Implement custom validation logic

```csharp
public class CreateUserRequest
{
    [Required, EmailAddress]
    public string Email { get; set; } = string.Empty;

    [Required, MinLength(8)]
    public string Password { get; set; } = string.Empty;
}

public class CreateUserEndpoint : Endpoint<CreateUserRequest>
{
    public override void Configure()
    {
        Post("/api/users");
        AllowAnonymous();
    }

    public override async Task HandleAsync(CreateUserRequest req, CancellationToken ct)
    {
        // Custom validation logic
        if (await UserExistsAsync(req.Email, ct))
        {
            AddError("Email already exists");
            await SendErrorsAsync(409, ct); // Conflict
            return;
        }

        // Password strength validation
        if (!IsPasswordStrong(req.Password))
        {
            AddError("Password must contain at least one uppercase letter, one lowercase letter, and one number");
            await SendErrorsAsync(400, ct);
            return;
        }

        // Create user logic here
        await SendCreatedAtAsync(
            new { message = "User created successfully" },
            $"/api/users/{userId}",
            ct);
    }

    private async Task<bool> UserExistsAsync(string email, CancellationToken ct)
    {
        // Check if user exists
        return false; // Implementation here
    }

    private bool IsPasswordStrong(string password)
    {
        return password.Length >= 8 &&
               password.Any(char.IsUpper) &&
               password.Any(char.IsLower) &&
               password.Any(char.IsDigit);
    }
}
```

#### ❌ DON'T: Skip validation or handle errors poorly

```csharp
public class BadValidationEndpoint : Endpoint<object>
{
    public override void Configure()
    {
        Post("/api/users");
    }

    public override async Task HandleAsync(object req, CancellationToken ct)
    {
        // DON'T skip validation
        // DON'T use dynamic objects
        // DON'T handle errors poorly
        try
        {
            // Risky operation without validation
            await CreateUserAsync(req, ct);
            await SendOkAsync("Created");
        }
        catch (Exception ex)
        {
            // DON'T expose internal errors
            await SendAsync(ex.Message, 500);
        }
    }
}
```

## Security Implementation

### Authentication and Authorization

Configure security with Fast Endpoints' built-in features.

#### ✅ DO: Implement JWT authentication with Fast Endpoints

```csharp
// Program.cs - Authentication setup
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!))
        };
    });

builder.Services.AddAuthorization();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();
```

#### ✅ DO: Configure endpoint-level authorization

```csharp
public class AdminOnlyEndpoint : Endpoint<AdminRequest>
{
    public override void Configure()
    {
        Post("/api/admin/action");
        Roles("Admin"); // Require Admin role
        Description(b => b
            .WithSummary("Admin action")
            .WithDescription("Requires admin privileges"));
    }

    public override async Task HandleAsync(AdminRequest req, CancellationToken ct)
    {
        // Only accessible to users with Admin role
        await SendOkAsync(new { message = "Admin action completed" }, ct);
    }
}

public class UserProfileEndpoint : EndpointWithoutRequest
{
    public override void Configure()
    {
        Get("/api/profile");
        Claims("UserId"); // Require authenticated user with UserId claim
    }

    public override async Task HandleAsync(CancellationToken ct)
    {
        var userId = User.FindFirst("UserId")?.Value;
        var profile = await GetUserProfileAsync(userId!, ct);
        await SendOkAsync(profile, ct);
    }
}
```

#### ❌ DON'T: Skip authorization checks

```csharp
public class InsecureEndpoint : Endpoint<object>
{
    public override void Configure()
    {
        Post("/api/admin/delete-all");
        // DON'T forget to add authorization!
        // AllowAnonymous(); // This is dangerous for admin actions
    }

    public override async Task HandleAsync(object req, CancellationToken ct)
    {
        // DON'T perform admin operations without proper authorization
        await DeleteAllDataAsync(ct);
        await SendOkAsync("All data deleted", ct);
    }
}
```

> [!WARNING]
> **Security Best Practices:**
>
> - Always configure proper authorization for sensitive endpoints
> - Use roles and claims for fine-grained access control
> - Never expose sensitive data in error messages
> - Implement rate limiting for public endpoints
> - Use HTTPS in production environments
> - Validate JWT tokens properly

## Performance Optimization

### Efficient Data Access

Optimize database operations and implement caching with Fast Endpoints.

#### ✅ DO: Implement efficient data access patterns

```csharp
public class GetProductsEndpoint : Endpoint<GetProductsRequest, PagedResult<ProductResponse>>
{
    private readonly IProductService _productService;

    public GetProductsEndpoint(IProductService productService)
    {
        _productService = productService;
    }

    public override void Configure()
    {
        Get("/api/products");
        AllowAnonymous();
        Description(b => b
            .WithSummary("Get paginated products")
            .WithDescription("Retrieves products with pagination and filtering"));
    }

    public override async Task HandleAsync(GetProductsRequest req, CancellationToken ct)
    {
        var result = await _productService.GetProductsAsync(
            new ProductFilter
            {
                Page = req.Page,
                PageSize = req.PageSize,
                CategoryId = req.CategoryId,
                SearchTerm = req.SearchTerm
            }, ct);

        await SendOkAsync(result, ct);
    }
}

// Request DTO with pagination
public class GetProductsRequest
{
    [QueryParam, Range(1, int.MaxValue)]
    public int Page { get; set; } = 1;

    [QueryParam, Range(1, 100)]
    public int PageSize { get; set; } = 20;

    [QueryParam]
    public Guid? CategoryId { get; set; }

    [QueryParam, MaxLength(100)]
    public string? SearchTerm { get; set; }
}
```

#### ✅ DO: Implement caching for frequently accessed data

```csharp
public class CachedProductsEndpoint : EndpointWithoutRequest<List<ProductSummary>>
{
    private readonly IProductService _productService;
    private readonly IDistributedCache _cache;

    public CachedProductsEndpoint(IProductService productService, IDistributedCache cache)
    {
        _productService = productService;
        _cache = cache;
    }

    public override void Configure()
    {
        Get("/api/products/summary");
        AllowAnonymous();
    }

    public override async Task HandleAsync(CancellationToken ct)
    {
        const string cacheKey = "products_summary";

        var cached = await _cache.GetStringAsync(cacheKey, ct);
        if (cached is not null)
        {
            var products = JsonSerializer.Deserialize<List<ProductSummary>>(cached);
            await SendOkAsync(products!, ct);
            return;
        }

        var products = await _productService.GetProductSummariesAsync(ct);

        await _cache.SetStringAsync(
            cacheKey,
            JsonSerializer.Serialize(products),
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
            },
            ct);

        await SendOkAsync(products, ct);
    }
}
```

## API Documentation and Discovery

### OpenAPI/Swagger Integration

Configure comprehensive API documentation using Fast Endpoints' Swagger integration.

#### ✅ DO: Configure OpenAPI documentation

```csharp
// Program.cs - Swagger configuration
builder.Services.AddSwaggerDoc(swagGenOptions =>
{
    swagGenOptions.DocumentName = "v1";
    swagGenOptions.Title = "Product API";
    swagGenOptions.Version = "v1.0";
    swagGenOptions.Description = "A comprehensive product management API built with Fast Endpoints";
    swagGenOptions.Contact = new NSwag.OpenApiContact
    {
        Name = "API Support",
        Email = "support@yourcompany.com"
    };
}, swaggerUiOptions =>
{
    swaggerUiOptions.Path = "/swagger";
    swaggerUiOptions.DocumentPath = "/swagger/v1/swagger.json";
});

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseOpenApi();
    app.UseSwaggerUi();
}
```

#### ✅ DO: Enhance endpoint documentation

```csharp
public class GetProductEndpoint : Endpoint<GetProductRequest, ProductResponse>
{
    public override void Configure()
    {
        Get("/api/products/{Id}");
        AllowAnonymous();
        Description(b => b
            .WithName("GetProduct")
            .WithSummary("Get a specific product by ID")
            .WithDescription("Retrieves detailed information about a product")
            .Produces<ProductResponse>(200, "Success")
            .Produces(404, "Product not found")
            .WithTags("Products"));
    }

    public override async Task HandleAsync(GetProductRequest req, CancellationToken ct)
    {
        var product = await GetProductAsync(req.Id, ct);

        if (product is null)
        {
            await SendNotFoundAsync(ct);
            return;
        }

        await SendOkAsync(product, ct);
    }
}

public class GetProductRequest
{
    [QueryParam, Required]
    public Guid Id { get; set; }
}
```

## Testing Fast Endpoints

### Unit Testing Endpoints

Write comprehensive tests for Fast Endpoints.

#### ✅ DO: Test Fast Endpoints properly

```csharp
public class CreateProductEndpointTests
{
    [Fact]
    public async Task CreateProduct_WithValidData_ReturnsCreated()
    {
        // Arrange
        var endpoint = Factory.Create<CreateProductEndpoint>();
        var request = new CreateProductRequest
        {
            Name = "Test Product",
            Price = 29.99m,
            CategoryId = Guid.NewGuid()
        };

        // Act
        await endpoint.HandleAsync(request, CancellationToken.None);

        // Assert
        endpoint.HttpContext.Response.StatusCode.Should().Be(201);
        var response = endpoint.Response;
        response.Should().NotBeNull();
        response.Id.Should().NotBeEmpty();
    }

    [Fact]
    public async Task CreateProduct_WithInvalidData_ReturnsValidationErrors()
    {
        // Arrange
        var endpoint = Factory.Create<CreateProductEndpoint>();
        var request = new CreateProductRequest
        {
            Name = "", // Invalid - empty name
            Price = -10, // Invalid - negative price
            CategoryId = Guid.Empty // Invalid - empty GUID
        };

        // Act
        await endpoint.HandleAsync(request, CancellationToken.None);

        // Assert
        endpoint.HttpContext.Response.StatusCode.Should().Be(400);
        endpoint.ValidationFailed.Should().BeTrue();
    }
}
```

> [!TIP]
> **Fast Endpoints Testing Benefits:**
>
> - Built-in test factory for creating endpoint instances
> - Easy mocking of dependencies
> - Automatic validation testing
> - Integration with existing .NET testing frameworks
> - Clear separation of unit and integration tests

# End of Kiro Steering File

