---
description: Best practices for building modern, maintainable Blazor applications with proper component architecture, state management, and performance optimization
fileMatchPattern: "*.razor", "*.razor.cs"
inclusion: fileMatch
---

# Kiro Steering File: Best Practices for Blazor Development

## Role Definition

- Software Architect
- Senior Blazor .NET Developer
- C# Language Expert
- ASP.NET Core Expert
- UI/UX Specialist

## General

### Description

Build modern, maintainable Blazor applications using component-based architecture, proper state management, and
performance optimization techniques. Focus on creating responsive user interfaces with clean separation of concerns,
efficient data binding, and scalable component patterns. Leverage Blazor's server-side and WebAssembly capabilities to
deliver optimal user experiences across different deployment scenarios.

### Requirements

- **NEVER** place sensitive information in generated code (passwords, API keys, personal data)
- Use steering rules from #[[file:.kiro/steering/dotnet/csharp/csharp-coding-style.md]] for C# coding style and conventions
- Use steering rules from #[[file:.kiro/steering/dotnet/general/dotnet-testing.md]] for testing guidelines and best practices
- Follow Blazor component naming conventions and file organization patterns
- Implement proper state management and data flow patterns
- Use dependency injection for services and maintain testability
- Optimize for performance with proper rendering and caching strategies
- Ensure accessibility and responsive design principles

## Component Architecture and Design

### Component Structure and Organization

Create well-structured Blazor components following best practices for maintainability and reusability.

#### ✅ DO: Create properly structured components

```csharp
// Counter.razor
@page "/counter"
@inject ILogger<Counter> Logger

<PageTitle>Counter</PageTitle>

<h1>Counter</h1>

<p role="status">Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    private void IncrementCount()
    {
        currentCount++;
        Logger.LogInformation("Counter incremented to {Count}", currentCount);
    }
}
```

#### ❌ DON'T: Mix UI and business logic in components

```csharp
@code {
    // DON'T put business logic directly in components
    private async Task SaveUserAsync()
    {
        // DON'T make direct database calls from components
        using var context = new ApplicationDbContext();
        await context.Users.AddAsync(newUser);
        await context.SaveChangesAsync();
    }
}
```

## Data Binding and State Management

### Effective Data Binding Patterns

Use Blazor's data binding features efficiently with proper state management.

#### ✅ DO: Use proper data binding with validation

```csharp
<EditForm Model="@userModel" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <div class="mb-3">
        <label for="name" class="form-label">Name</label>
        <InputText id="name" @bind-Value="userModel.Name" class="form-control" />
        <ValidationMessage For="@(() => userModel.Name)" />
    </div>

    <button type="submit" class="btn btn-primary" disabled="@isSubmitting">
        @if (isSubmitting)
        {
            <span class="spinner-border spinner-border-sm"></span>
            Saving...
        }
        else
        {
            Save Changes
        }
    </button>
</EditForm>

@code {
    private UserModel userModel = new();
    private bool isSubmitting = false;

    private async Task HandleValidSubmit()
    {
        isSubmitting = true;
        try
        {
            await UserService.UpdateUserAsync(userModel);
        }
        finally
        {
            isSubmitting = false;
        }
    }
}
```

## Performance Optimization

### Component Rendering Optimization

Optimize component rendering to improve application performance.

#### ✅ DO: Implement proper rendering optimizations

```csharp
@if (ShouldRender())
{
    <div class="product-list">
        @foreach (var product in products)
        {
            <ProductCard Product="product" OnUpdate="HandleProductUpdate" />
        }
    </div>
}

@code {
    private List<Product> products = new();

    protected override bool ShouldRender() => shouldRender;

    private void HandleProductUpdate(Product updatedProduct)
    {
        var index = products.FindIndex(p => p.Id == updatedProduct.Id);
        if (index >= 0)
        {
            products[index] = updatedProduct;
            StateHasChanged(); // Only when necessary
        }
    }
}
```

#### ❌ DON'T: Cause unnecessary re-renders

```csharp
@code {
    // DON'T call StateHasChanged() unnecessarily
    private void UnnecessaryUpdate()
    {
        StateHasChanged(); // This causes unnecessary re-renders
    }
}
```

## API Integration and Error Handling

### HttpClient Usage and Error Handling

Implement robust API communication with proper error handling.

#### ✅ DO: Use HttpClient with proper error handling

```csharp
@inject HttpClient Http
@inject ILogger<WeatherComponent> Logger

@code {
    private WeatherForecast[]? forecasts;
    private bool isLoading = false;
    private string errorMessage = string.Empty;

    private async Task LoadForecastsAsync()
    {
        isLoading = true;
        errorMessage = string.Empty;

        try
        {
            forecasts = await Http.GetFromJsonAsync<WeatherForecast[]>("api/weatherforecast");
        }
        catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.Unauthorized)
        {
            errorMessage = "You are not authorized to view this data.";
            Logger.LogWarning(ex, "Unauthorized access attempt");
        }
        catch (HttpRequestException ex)
        {
            errorMessage = "Failed to load weather data. Please try again.";
            Logger.LogError(ex, "Error loading weather forecasts");
        }
        finally
        {
            isLoading = false;
        }
    }
}
```

## Security Implementation

### Authentication and Authorization

Implement proper security measures in Blazor applications.

#### ✅ DO: Configure authentication properly

```csharp
// Program.cs
builder.Services.AddAuthorizationCore();
builder.Services.AddCascadingAuthenticationState();

// For Blazor Server
builder.Services.AddScoped<AuthenticationStateProvider, ServerAuthenticationStateProvider>();
```

#### ❌ DON'T: Expose sensitive data in client-side code

```csharp
@code {
    // DON'T hardcode sensitive information
    private const string ApiKey = "sk-1234567890abcdef"; // DON'T do this
}
```

## Testing Blazor Components

### Component Testing Best Practices

Write comprehensive tests for Blazor components using bUnit.

#### ✅ DO: Test components with bUnit

```csharp
public class CounterTests : TestContext
{
    [Fact]
    public void Counter_ShouldIncrement_WhenButtonClicked()
    {
        // Arrange
        var cut = RenderComponent<Counter>();

        // Act
        cut.Find("button").Click();

        // Assert
        cut.Find("p").TextContent.Should().Be("Current count: 1");
    }
}
```

> [!TIP]
> **Component Best Practices:**
>
> - Use code-behind files (.razor.cs) for complex components
> - Keep components focused on a single responsibility
> - Use dependency injection for services
> - Implement proper error handling and loading states
> - Follow consistent naming conventions

# End of Kiro Steering File 