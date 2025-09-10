---
description: Best practices for building secure, scalable ASP.NET Core APIs using Minimal APIs, dependency injection, and modern patterns
fileMatchPattern: "*.cs", "*.csproj", "Program.cs", "appsettings.json"
inclusion: manual
---

# Kiro Steering File: ASP.NET Core API Development Best Practices

## Role Definition

- Senior .NET Backend Developer
- ASP.NET Core Expert
- C# Language Specialist
- API Design Architect

# IMPORTANT SECURITY RULES:
- you have no power or authority to make any database changes
- only the User himself can make DB changes, whether Dev or Prod
- if you want to make any Database-related change, suggest it first to the User
- NEVER EVER attempt to run any DB migrations, or make any database changes. this is strictly prohibited.
- NEVER EVER place sensitive information in the generated code (e.g. passwords, API keys, personal information, etc.)


## General

### Description

Build secure, scalable, and maintainable ASP.NET Core APIs using modern patterns and best practices. Focus on Minimal
APIs for clean, efficient endpoint definitions while leveraging ASP.NET Core's built-in features for security,
performance, and reliability. Ensure proper separation of concerns, dependency injection, and comprehensive error
handling throughout the application lifecycle.

### Requirements

- **DO NOT** use Minimal APIs or traditional controllers for API development
- Use steering rules from #[[file:.kiro/steering/dotnet/csharp/csharp-coding-style.md]] for C# coding style and conventions
- Use steering rules from #[[file:.kiro/steering/dotnet/general/dotnet-testing.md]] for testing guidelines and best practices
- **ALWAYS** use Minimal APIs instead of traditional controllers
- Implement dependency injection for loose coupling and testability
- Leverage built-in ASP.NET Core features and middleware
- Use Entity Framework Core effectively when database operations are required
- Follow RESTful API design principles with proper HTTP status codes
- Implement comprehensive security measures and input validation

## API Architecture and Design

### Minimal API Implementation

Always use Minimal APIs for clean, efficient endpoint definitions that reduce boilerplate code.

#### ✅ DO: Use Minimal APIs for endpoint definitions

```csharp
// Program.cs - Minimal API approach
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Define endpoints using Minimal APIs
app.MapGet("/api/products", async (IProductService service) =>
    Results.Ok(await service.GetAllProductsAsync()))
.WithName("GetProducts")
.WithOpenApi();

app.MapPost("/api/products", async (ProductDto product, IProductService service) =>
{
    var result = await service.CreateProductAsync(product);
    return Results.Created($"/api/products/{result.Id}", result);
})
.WithName("CreateProduct")
.WithOpenApi();

app.Run();
```

#### ❌ DON'T: Use traditional controllers

```csharp
// DON'T create controllers - use Minimal APIs instead
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // Traditional controller approach - avoid this pattern
}
```

> [!TIP]
> **Benefits of Minimal APIs:**
>
> - Reduced boilerplate code and ceremony
> - Better performance with less overhead
> - Improved developer experience with fluent API
> - Easier testing and maintenance
> - Built-in OpenAPI/Swagger support

## Error Handling and Validation

### Global Exception Handling

Implement comprehensive error handling with proper HTTP status codes and consistent error responses.

#### ✅ DO: Implement global exception handling middleware

```csharp
// Program.cs - Global exception handling
app.UseExceptionHandler("/error");
app.MapGet("/error", () => Results.Problem());

app.MapGet("/api/test", () =>
{
    // This will be caught by global exception handler
    throw new InvalidOperationException("Test error");
});
```

#### ✅ DO: Use proper error response formats

```csharp
// Custom error response
public record ErrorResponse(string Message, string? Details = null);

app.MapGet("/api/fail", () =>
    Results.BadRequest(new ErrorResponse("Invalid request", "Product ID is required")))
.WithName("FailEndpoint");
```

#### ❌ DON'T: Let exceptions bubble up unhandled

```csharp
// DON'T - unhandled exceptions
app.MapGet("/api/dangerous", () =>
{
    throw new Exception("This will crash the API");
    // No error handling - bad practice
});
```

### Input Validation

Use Data Annotations or Fluent Validation for robust input validation.

#### ✅ DO: Validate inputs with Data Annotations

```csharp
public record CreateProductRequest
{
    [Required]
    [StringLength(100, MinimumLength = 1)]
    public string Name { get; init; } = string.Empty;

    [Range(0.01, double.MaxValue)]
    public decimal Price { get; init; }

    [Required]
    public Guid CategoryId { get; init; }
}

app.MapPost("/api/products", async (CreateProductRequest request, IProductService service) =>
{
    if (!ModelState.IsValid)
        return Results.ValidationProblem(ModelState.ToDictionary());

    var result = await service.CreateProductAsync(request);
    return Results.Created($"/api/products/{result.Id}", result);
});
```

## Security Implementation

### Authentication and Authorization

Implement proper security measures with environment-aware configuration.

#### ✅ DO: Configure environment-specific security settings

```csharp
// Program.cs - Security configuration
if (app.Environment.IsDevelopment())
{
    // Allow HTTP and relaxed CORS in development
    app.UseCors(policy => policy.AllowAnyOrigin().AllowAnyMethod().AllowAnyHeader());
}
else
{
    // Enforce HTTPS and strict CORS in production
    app.UseHttpsRedirection();
    app.UseCors("ProductionPolicy");
}

// Configure CORS policy for production
builder.Services.AddCors(options =>
{
    options.AddPolicy("ProductionPolicy", policy =>
    {
        policy.WithOrigins("https://yourdomain.com")
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});
```

#### ✅ DO: Implement JWT authentication for APIs

```csharp
// Add JWT authentication
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

// Use authentication and authorization
app.UseAuthentication();
app.UseAuthorization();
```

> [!WARNING]
> **Security Best Practices:**
>
> - Never hardcode sensitive data (use configuration and environment variables)
> - Always use HTTPS in production environments
> - Implement proper CORS policies for production
> - Validate JWT tokens and handle token expiration
> - Use secure password hashing for user authentication

## Performance Optimization

### Asynchronous Operations

Use async/await patterns for all I/O-bound operations to maintain application responsiveness.

#### ✅ DO: Use async operations for I/O-bound work

```csharp
app.MapGet("/api/products/{id}", async (Guid id, IProductService service) =>
{
    var product = await service.GetProductByIdAsync(id);
    return product is not null ? Results.Ok(product) : Results.NotFound();
})
.WithName("GetProduct");
```

#### ✅ DO: Implement caching strategies

```csharp
// Add caching services
builder.Services.AddMemoryCache();
builder.Services.AddDistributedMemoryCache(); // Or Redis/SQL Server

app.MapGet("/api/products", async (IProductService service, IDistributedCache cache) =>
{
    const string cacheKey = "products_list";

    var cached = await cache.GetStringAsync(cacheKey);
    if (cached is not null)
        return Results.Ok(JsonSerializer.Deserialize<List<Product>>(cached));

    var products = await service.GetAllProductsAsync();
    await cache.SetStringAsync(cacheKey, JsonSerializer.Serialize(products),
        new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
        });

    return Results.Ok(products);
});
```

#### ❌ DON'T: Block on async operations

```csharp
// DON'T block on async calls
app.MapGet("/api/products", (IProductService service) =>
{
    var products = service.GetAllProductsAsync().Result; // Blocking call - bad!
    return Results.Ok(products);
});
```

## Data Access Patterns

### Entity Framework Core Usage

Implement efficient data access patterns with proper query optimization.

#### ✅ DO: Use EF Core with proper query patterns

```csharp
public class ProductService : IProductService
{
    private readonly ApplicationDbContext _context;

    public async Task<List<Product>> GetAllProductsAsync()
    {
        return await _context.Products
            .AsNoTracking() // No change tracking for read-only queries
            .Include(p => p.Category) // Eager loading to avoid N+1
            .ToListAsync();
    }

    public async Task<Product?> GetProductByIdAsync(Guid id)
    {
        return await _context.Products
            .Include(p => p.Category)
            .FirstOrDefaultAsync(p => p.Id == id);
    }
}
```

#### ✅ DO: Implement pagination for large datasets

```csharp
public async Task<PagedResult<Product>> GetProductsPagedAsync(int page = 1, int pageSize = 20)
{
    var query = _context.Products
        .Include(p => p.Category)
        .AsNoTracking();

    var totalCount = await query.CountAsync();
    var items = await query
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .ToListAsync();

    return new PagedResult<Product>(items, totalCount, page, pageSize);
}
```

> [!IMPORTANT]
> **Database Performance Tips:**
>
> - Use `AsNoTracking()` for read-only queries to improve performance
> - Implement eager loading with `Include()` to prevent N+1 query problems
> - Use pagination for large result sets
> - Implement proper indexing on frequently queried columns
> - Consider using compiled queries for frequently executed queries

## API Documentation

### OpenAPI Integration

Use ASP.NET Core's built-in OpenAPI support for comprehensive API documentation.

#### ✅ DO: Configure OpenAPI with proper metadata

```csharp
// Program.cs - OpenAPI configuration
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Product API",
        Version = "v1",
        Description = "A comprehensive product management API",
        Contact = new OpenApiContact
        {
            Name = "API Support",
            Email = "support@yourcompany.com"
        }
    });

    // Include XML comments
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    options.IncludeXmlComments(xmlPath);
});

var app = builder.Build();

// Enable Swagger UI
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

#### ✅ DO: Add XML documentation comments

```csharp
/// <summary>
/// Retrieves all products from the catalog
/// </summary>
/// <returns>A list of all available products</returns>
/// <response code="200">Returns the list of products</response>
app.MapGet("/api/products", async (IProductService service) =>
    Results.Ok(await service.GetAllProductsAsync()))
.WithName("GetProducts")
.WithOpenApi(operation => new(operation)
{
    Summary = "Get all products",
    Description = "Retrieves a complete list of products from the catalog"
});
```

## Background Processing

### Hosted Services Implementation

Use background services for long-running or scheduled tasks.

#### ✅ DO: Implement background services for async operations

```csharp
// Background service for processing orders
public class OrderProcessingService : BackgroundService
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<OrderProcessingService> _logger;

    public OrderProcessingService(
        IServiceProvider serviceProvider,
        ILogger<OrderProcessingService> logger)
    {
        _serviceProvider = serviceProvider;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                using var scope = _serviceProvider.CreateScope();
                var orderService = scope.ServiceProvider.GetRequiredService<IOrderService>();

                await orderService.ProcessPendingOrdersAsync(stoppingToken);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error processing orders");
            }

            await Task.Delay(TimeSpan.FromMinutes(5), stoppingToken);
        }
    }
}

// Register the service
builder.Services.AddHostedService<OrderProcessingService>();
```

## Configuration Management

### Environment-Specific Settings

Properly configure applications for different environments.

#### ✅ DO: Use environment-specific configuration

```csharp
// Program.cs - Environment-aware configuration
var builder = WebApplication.CreateBuilder(args);

// Add environment-specific configuration
builder.Configuration
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .AddJsonFile($"appsettings.{builder.Environment.EnvironmentName}.json", optional: true, reloadOnChange: true)
    .AddEnvironmentVariables();

// Configure services based on environment
if (builder.Environment.IsDevelopment())
{
    builder.Services.AddDatabaseDeveloperPageExceptionFilter();
}
else
{
    builder.Services.AddHsts(options =>
    {
        options.Preload = true;
        options.IncludeSubDomains = true;
        options.MaxAge = TimeSpan.FromDays(365);
    });
}
```

> [!TIP]
> **Configuration Best Practices:**
>
> - Use strongly-typed configuration classes
> - Separate environment-specific settings
> - Never commit sensitive data to source control
> - Use Azure Key Vault or similar for production secrets
> - Validate configuration on application startup

# End of Kiro Steering File

