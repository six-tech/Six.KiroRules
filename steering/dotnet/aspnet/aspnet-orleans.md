---
description: This file provides comprehensive guidelines for writing high-performance, maintainable Orleans applications with proper grain design, state management, and distributed patterns.
fileMatchPattern: "*.cs", "*.csproj", "Program.cs", "appsettings.json"
inclusion: manual
---

# Kiro Steering File: Microsoft Orleans Coding Style & Best Practices

Role Definition:

- Orleans Distributed Systems Expert
- Performance Optimization Specialist
- Grain Design Architect
- Distributed Computing Specialist

# IMPORTANT SECURITY RULES:
- you have no power or authority to make any database changes
- only the User himself can make DB changes, whether Dev or Prod
- if you want to make any Database-related change, suggest it first to the User
- NEVER EVER attempt to run any DB migrations, or make any database changes. this is strictly prohibited.
- NEVER EVER place sensitive information in the generated code (e.g. passwords, API keys, personal information, etc.)


## 1. Overview & General Principles

### Core Philosophy

Orleans applications should maximize scalability, reliability, and performance through proper grain design, efficient
state management, and optimized communication patterns. Always prioritize distributed system best practices while
maintaining developer productivity.

### Requirements

- Use steering rules from #[[file:.kiro/steering/dotnet/csharp/csharp-coding-style.md]] for C# coding style and conventions
- Use steering rules from #[[file:.kiro/steering/dotnet/general/dotnet-testing.md]] for testing guidelines and best practices
- Design grains for high cohesion and loose coupling
- Implement proper state persistence and recovery patterns
- Use `ValueTask` for async grain methods to optimize performance
- Follow Orleans naming conventions and patterns
- Implement comprehensive error handling and resilience patterns
- Use Orleans clustering and partitioning effectively
- Design for eventual consistency and distributed system constraints

## 2. Project Setup & Configuration

### Orleans Package Dependencies with .NET Aspire

Include essential Orleans packages and Aspire components for orchestration:

```xml
<!-- Core Orleans packages -->
<PackageReference Include="Microsoft.Orleans.Server" Version="9.1.2"/>
<PackageReference Include="Microsoft.Orleans.Client" Version="9.1.2"/>
<PackageReference Include="Aspire.Microsoft.Extensions.Hosting" Version="9.0.0"/>
<PackageReference Include="Microsoft.Extensions.Hosting" Version="9.0.0"/>

<!-- Aspire Orleans integration -->
<PackageReference Include="Aspire.Orleans.Server" Version="9.0.0"/>
<PackageReference Include="Aspire.Orleans.Client" Version="9.0.0"/>

<!-- Optional packages for storage (prefer Aspire-managed services for better local dev) -->
<PackageReference Include="Aspire.Azure.Storage.Blobs" Version="9.0.0"/>
<PackageReference Include="Aspire.MongoDB.Driver" Version="9.0.0"/>
<PackageReference Include="Microsoft.Extensions.Logging.Console" Version="9.0.0"/>
```

### Silo Configuration with .NET Aspire

Configure the Orleans silo using Aspire's service orchestration:

```csharp
// OrleansSilo/Program.cs - Aspire-managed Orleans Silo
var builder = WebApplication.CreateBuilder(args);

// Add Aspire service defaults & health checks
builder.AddServiceDefaults();

// Configure Orleans silo with Aspire orchestration
builder.UseOrleans(orleansBuilder =>
{
    // Aspire-managed clustering (automatic service discovery)
    orleansBuilder.UseAspireClustering();

    // Aspire-managed grain storage (Azure Storage, MongoDB, etc.)
    orleansBuilder.AddAspireGrainStorage("MyGrainStorage");

    // Performance optimizations
    orleansBuilder.Configure<GrainCollectionOptions>(options =>
    {
        options.CollectionQuantum = TimeSpan.FromMinutes(1);
        options.DeactivationTimeout = TimeSpan.FromMinutes(2);
    });

    // Load balancing
    orleansBuilder.Configure<LoadSheddingOptions>(options =>
    {
        options.LoadSheddingEnabled = true;
        options.LoadSheddingLimit = 95;
    });
});

var app = builder.Build();

// Map Aspire endpoints (health checks, metrics, etc.)
app.MapDefaultEndpoints();

app.Run();
```

### AppHost Configuration (.NET Aspire)

Set up your AppHost project to orchestrate Orleans services:

```csharp
// AspireApp.AppHost/Program.cs
var builder = DistributedApplication.CreateBuilder(args);

// Add Orleans silo with scaling
var orleansSilo = builder.AddProject<Projects.OrleansSilo>("orleans-silo")
    .WithReplicas(2); // Scale to multiple silo instances

// Add Orleans client service
var orleansClient = builder.AddProject<Projects.OrleansClient>("orleans-client")
    .WithReference(orleansSilo); // Automatic service discovery

// Add storage resources (managed by Aspire)
var storage = builder.AddAzureStorage("storage");
var blobs = storage.AddBlobs("blobstorage");
var tables = storage.AddTables("tablestorage");

// Add database for Orleans clustering/storage
var sql = builder.AddSqlServer("sql")
    .WithLifetime(ContainerLifetime.Persistent);

var orleansDb = sql.AddDatabase("orleans-db");

// Configure Orleans services
orleansSilo.AddAzureStorage(storage)
           .WithReference(orleansDb);

builder.Build().Run();
```

### Local Development with Aspire Dashboard

Start your entire application stack with Aspire's orchestration:

```bash
# Run the AppHost to start all services
dotnet run --project AspireApp.AppHost

# This automatically starts:
# - Aspire Dashboard (http://localhost:15888)
# - Orleans Silo(s) with clustering
# - Orleans Client service
# - Azure Storage emulator
# - SQL Server container
# - Health monitoring and metrics
```

### Environment-Specific Configuration

Configure Orleans differently per environment using Aspire:

```csharp
// OrleansSilo/Program.cs - Environment-aware configuration
builder.UseOrleans(orleansBuilder =>
{
    // Clustering based on environment
    if (builder.Environment.IsDevelopment())
    {
        // Use local clustering for development
        orleansBuilder.UseLocalhostClustering();
    }
    else
    {
        // Use Aspire-managed clustering for staging/production
        orleansBuilder.UseAspireClustering();
    }

    // Storage based on environment
    if (builder.Environment.IsDevelopment())
    {
        // Use in-memory storage for faster development
        orleansBuilder.AddMemoryGrainStorage("MyMemoryStorage");
    }
    else
    {
        // Use Aspire-managed storage for persistence
        orleansBuilder.AddAspireGrainStorage("MyPersistentStorage");
    }
});
```

### Client Configuration with .NET Aspire

Configure Orleans client using Aspire's automatic service discovery:

```csharp
// OrleansClient/Program.cs - Aspire-managed Orleans Client
var builder = WebApplication.CreateBuilder(args);

// Add Aspire service defaults & health checks
builder.AddServiceDefaults();

// Configure Orleans client with Aspire service discovery
builder.UseOrleansClient(clientBuilder =>
{
    // Aspire-managed clustering (automatic service discovery)
    clientBuilder.UseAspireClustering();

    clientBuilder.Configure<ClusterOptions>(options =>
    {
        options.ClusterId = "aspire-dev";
        options.ServiceId = "OrleansApp";
    });

    clientBuilder.Configure<ClientMessagingOptions>(options =>
    {
        options.ResponseTimeout = TimeSpan.FromSeconds(30);
        options.MaxMessageLength = 10 * 1024 * 1024; // 10MB
    });

    clientBuilder.AddSimpleMessageStreamProvider("SMS");
});

var app = builder.Build();

// Map Aspire endpoints (health checks, metrics, etc.)
app.MapDefaultEndpoints();

// Example: Use Orleans grains in your API endpoints
app.MapGet("/users/{id}", async (string id, IGrainFactory grainFactory) =>
{
    var userGrain = grainFactory.GetGrain<IUserGrain>(id);
    var profile = await userGrain.GetProfileAsync();
    return Results.Ok(profile);
});

app.Run();
```

### Aspire Service Communication

Communicate between Aspire services using Orleans grains:

```csharp
// Example: API service communicating with Orleans silo
public class WeatherApiService
{
    private readonly IGrainFactory _grainFactory;

    public WeatherApiService(IGrainFactory grainFactory)
    {
        _grainFactory = grainFactory;
    }

    public async Task<WeatherData> GetWeatherAsync(string location)
    {
        // Get grain reference (Aspire handles service discovery)
        var weatherGrain = _grainFactory.GetGrain<IWeatherGrain>(location);

        // Call grain method
        return await weatherGrain.GetCurrentWeatherAsync();
    }
}
```

### Service Reference Configuration

Configure service references in AppHost for proper communication:

```csharp
// AspireApp.AppHost/Program.cs - Service communication setup
var builder = DistributedApplication.CreateBuilder(args);

// Add API service that depends on Orleans
var apiService = builder.AddProject<Projects.WeatherApi>("weather-api")
    .WithReference(orleansSilo) // Automatic service discovery
    .WithReference(orleansClient); // If needed

// Add Orleans silo
var orleansSilo = builder.AddProject<Projects.OrleansSilo>("orleans-silo")
    .WithReplicas(2);

// Add Orleans client service
var orleansClient = builder.AddProject<Projects.OrleansClient>("orleans-client")
    .WithReference(orleansSilo);

builder.Build().Run();
```

## 3. Grain Design & Architecture

### Grain Interface Design

Design grain interfaces for clarity and performance:

```csharp
// ✅ GOOD: Clear, focused grain interface
public interface IUserGrain : IGrainWithStringKey
{
    ValueTask<UserProfile> GetProfileAsync();
    ValueTask UpdateProfileAsync(UserProfile profile);
    ValueTask<bool> IsActiveAsync();
    ValueTask DeactivateAsync();
}

// ❌ BAD: Bloated interface with mixed concerns
public interface IUserGrain : IGrainWithStringKey
{
    Task<UserProfile> GetProfile(); // Not ValueTask
    Task UpdateProfile(UserProfile profile);
    Task<bool> IsActive();
    Task Deactivate();
    Task SendEmail(string subject, string body); // Different concern
    Task ProcessPayment(decimal amount); // Different concern
}
```

### Grain Implementation Patterns

Implement grains with proper lifecycle management:

```csharp
// ✅ GOOD: Proper grain implementation with lifecycle
public sealed class UserGrain : Grain, IUserGrain
{
    readonly IPersistentState<UserProfile> _profile;
    readonly ILogger<UserGrain> _logger;

    public UserGrain(
        [PersistentState("profile", "UserProfileStorage")] IPersistentState<UserProfile> profile,
        ILogger<UserGrain> logger)
    {
        _profile = profile;
        _logger = logger;
    }

    public override async ValueTask OnActivateAsync(CancellationToken token)
    {
        await base.OnActivateAsync(token);
        _logger.LogInformation("User grain {GrainKey} activated", this.GetPrimaryKeyString());
    }

    public override async ValueTask OnDeactivateAsync(DeactivationReason reason, CancellationToken token)
    {
        _logger.LogInformation("User grain {GrainKey} deactivated: {Reason}",
            this.GetPrimaryKeyString(), reason.ReasonCode);
        await base.OnDeactivateAsync(reason, token);
    }

    public ValueTask<UserProfile> GetProfileAsync()
    {
        if (!_profile.RecordExists)
            throw new UserNotFoundException(this.GetPrimaryKeyString());

        return ValueTask.FromResult(_profile.State);
    }

    public async ValueTask UpdateProfileAsync(UserProfile profile)
    {
        _profile.State = profile ?? throw new ArgumentNullException(nameof(profile));
        await _profile.WriteStateAsync();
    }

    public ValueTask<bool> IsActiveAsync() =>
        ValueTask.FromResult(_profile.RecordExists && _profile.State.IsActive);

    public async ValueTask DeactivateAsync()
    {
        if (_profile.RecordExists)
        {
            _profile.State.IsActive = false;
            await _profile.WriteStateAsync();
        }
    }
}
```

### Grain Key Strategies

Choose appropriate grain key types based on the use case:

```csharp
// ✅ GOOD: Different key types for different scenarios
public interface IDeviceGrain : IGrainWithStringKey { }      // Device ID as string
public interface IUserGrain : IGrainWithGuidKey { }         // User ID as GUID
public interface ISessionGrain : IGrainWithIntegerKey { }   // Session ID as integer
public interface ILocationGrain : IGrainWithIntegerCompoundKey { } // Lat/Long compound key

// ✅ GOOD: Custom key for complex scenarios
[GenerateSerializer]
public readonly record struct UserDeviceKey(string UserId, string DeviceId);

public interface IUserDeviceGrain : IGrainWith<UserDeviceKey> { }
```

## 4. State Management & Persistence

### Persistent State Patterns

Implement efficient state management with proper error handling:

```csharp
// ✅ GOOD: Robust state management with error handling
public sealed class OrderGrain : Grain, IOrderGrain
{
    readonly IPersistentState<OrderState> _state;
    readonly ILogger<OrderGrain> _logger;

    public OrderGrain(
        [PersistentState("order", "OrderStorage")] IPersistentState<OrderState> state,
        ILogger<OrderGrain> logger)
    {
        _state = state;
        _logger = logger;
    }

    public async ValueTask<OrderState> GetStateAsync()
    {
        try
        {
            if (!_state.RecordExists)
                throw new OrderNotFoundException(this.GetPrimaryKeyString());

            return _state.State;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to get order state for {OrderId}", this.GetPrimaryKeyString());
            throw;
        }
    }

    public async ValueTask UpdateStateAsync(OrderState state)
    {
        ArgumentNullException.ThrowIfNull(state);

        try
        {
            _state.State = state;
            await _state.WriteStateAsync();
            _logger.LogInformation("Order {OrderId} state updated", this.GetPrimaryKeyString());
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to update order state for {OrderId}", this.GetPrimaryKeyString());
            throw;
        }
    }
}
```

### State Versioning & Migration

Handle state schema changes gracefully:

```csharp
// ✅ GOOD: State versioning with migration
[GenerateSerializer]
[Alias("OrderStateV1")]
public sealed class OrderStateV1
{
    [Id(0)] public string CustomerId { get; set; }
    [Id(1)] public List<OrderItem> Items { get; set; }
    [Id(2)] public decimal Total { get; set; }
}

[GenerateSerializer]
[Alias("OrderStateV2")]
public sealed class OrderStateV2
{
    [Id(0)] public string CustomerId { get; set; }
    [Id(1)] public List<OrderItem> Items { get; set; }
    [Id(2)] public decimal Total { get; set; }
    [Id(3)] public DateTimeOffset CreatedAt { get; set; } // New field
    [Id(4)] public string Status { get; set; } // New field
}

public sealed class OrderGrain : Grain, IOrderGrain
{
    readonly IPersistentState<OrderStateV2> _state;

    public OrderGrain([PersistentState("order", "OrderStorage")] IPersistentState<OrderStateV2> state)
    {
        _state = state;
    }

    public override async ValueTask OnActivateAsync(CancellationToken token)
    {
        await base.OnActivateAsync(token);

        // Migration logic
        if (_state.RecordExists && _state.State.CreatedAt == default)
        {
            _state.State.CreatedAt = DateTimeOffset.UtcNow;
            _state.State.Status = "Active";
            await _state.WriteStateAsync();
        }
    }
}
```

### Memory Optimization

Use memory-efficient patterns for large state objects:

```csharp
// ✅ GOOD: Memory-efficient state handling
public sealed class LargeDataGrain : Grain, ILargeDataGrain
{
    readonly IPersistentState<LargeDataState> _state;

    public LargeDataGrain([PersistentState("large", "LargeDataStorage")] IPersistentState<LargeDataState> state)
    {
        _state = state;
    }

    public async ValueTask ProcessLargeDataAsync(Stream dataStream)
    {
        // Process data in chunks to avoid loading everything into memory
        using var reader = new StreamReader(dataStream);
        var buffer = new char[8192];

        while (!reader.EndOfStream)
        {
            var charsRead = await reader.ReadAsync(buffer, 0, buffer.Length);
            // Process chunk
            await ProcessChunkAsync(new string(buffer, 0, charsRead));
        }
    }

    private async ValueTask ProcessChunkAsync(string chunk)
    {
        // Process chunk without keeping it in memory
        var result = await AnalyzeChunkAsync(chunk);

        // Update state incrementally
        _state.State.ProcessedChunks++;
        _state.State.LastResult = result;
        await _state.WriteStateAsync();
    }
}
```

## 5. Communication Patterns

### Request-Response Patterns

Use ValueTask for optimal performance in grain communications:

```csharp
// ✅ GOOD: ValueTask for performance optimization
public interface IUserGrain : IGrainWithStringKey
{
    ValueTask<UserProfile> GetProfileAsync();
    ValueTask UpdateProfileAsync(UserProfile profile);
    ValueTask<bool> IsOnlineAsync();
}

// ✅ GOOD: Batch operations for efficiency
public interface IBatchGrain : IGrainWithStringKey
{
    ValueTask<BatchResult> ProcessBatchAsync(IEnumerable<BatchItem> items);
}

public sealed class BatchGrain : Grain, IBatchGrain
{
    public async ValueTask<BatchResult> ProcessBatchAsync(IEnumerable<BatchItem> items)
    {
        var results = new List<BatchItemResult>();
        var errors = new List<string>();

        foreach (var item in items)
        {
            try
            {
                var result = await ProcessItemAsync(item);
                results.Add(result);
            }
            catch (Exception ex)
            {
                errors.Add($"Failed to process item {item.Id}: {ex.Message}");
            }
        }

        return new BatchResult(results, errors);
    }
}
```

### Event-Driven Communication

Implement pub/sub patterns with streams:

```csharp
// ✅ GOOD: Stream-based event communication
public interface INotificationGrain : IGrainWithStringKey
{
    ValueTask SubscribeAsync(IAsyncStream<string> stream);
    ValueTask UnsubscribeAsync(IAsyncStream<string> stream);
    ValueTask PublishAsync(string message);
}

public sealed class NotificationGrain : Grain, INotificationGrain
{
    readonly IStreamProvider _streamProvider;
    IAsyncStream<string>? _stream;

    public NotificationGrain(IStreamProvider streamProvider)
    {
        _streamProvider = streamProvider;
    }

    public async ValueTask SubscribeAsync(IAsyncStream<string> stream)
    {
        _stream = stream;
        var subscription = await _stream.SubscribeAsync(
            (message, token) => HandleMessageAsync(message, token),
            (exception, token) => HandleErrorAsync(exception, token));

        // Store subscription for cleanup
        this.GetStreamProvider("SMS").GetStream<string>(Guid.NewGuid(), "notifications");
    }

    public ValueTask PublishAsync(string message)
    {
        if (_stream is not null)
            return _stream.OnNextAsync(message);

        return ValueTask.CompletedTask;
    }

    private async ValueTask HandleMessageAsync(string message, StreamSequenceToken token)
    {
        // Process incoming message
        await ProcessNotificationAsync(message);
    }
}
```

### Cross-Grain Communication

Implement efficient cross-grain communication patterns:

```csharp
// ✅ GOOD: Efficient cross-grain communication
public sealed class OrderProcessorGrain : Grain, IOrderProcessorGrain
{
    readonly IGrainFactory _grainFactory;
    readonly ILogger<OrderProcessorGrain> _logger;

    public OrderProcessorGrain(IGrainFactory grainFactory, ILogger<OrderProcessorGrain> logger)
    {
        _grainFactory = grainFactory;
        _logger = logger;
    }

    public async ValueTask ProcessOrderAsync(string orderId)
    {
        var orderGrain = _grainFactory.GetGrain<IOrderGrain>(orderId);
        var userGrain = _grainFactory.GetGrain<IUserGrain>(GetUserIdFromOrder(orderId));

        // Parallel execution for better performance
        var (order, user) = await (orderGrain.GetOrderAsync(), userGrain.GetProfileAsync());

        // Process order with user context
        await ProcessOrderWithUserContextAsync(order, user);
    }

    private async ValueTask ProcessOrderWithUserContextAsync(Order order, UserProfile user)
    {
        // Implementation
        _logger.LogInformation("Processing order {OrderId} for user {UserId}",
            order.Id, user.UserId);
    }
}
```

## 6. Performance Optimization

### Grain Activation Management

Optimize grain lifecycle for performance:

```csharp
// ✅ GOOD: Smart grain activation patterns
public sealed class CacheGrain : Grain, ICacheGrain
{
    readonly IPersistentState<CacheState> _state;
    readonly IGrainActivationContext _context;
    DateTime _lastAccess;

    public CacheGrain(
        [PersistentState("cache", "CacheStorage")] IPersistentState<CacheState> state,
        IGrainActivationContext context)
    {
        _state = state;
        _context = context;
        _lastAccess = DateTime.UtcNow;
    }

    public ValueTask<T> GetAsync<T>(string key)
    {
        _lastAccess = DateTime.UtcNow;

        if (_state.State.Data.TryGetValue(key, out var value))
            return ValueTask.FromResult((T)value);

        throw new KeyNotFoundException($"Key '{key}' not found in cache");
    }

    public async ValueTask SetAsync<T>(string key, T value, TimeSpan? ttl = null)
    {
        _lastAccess = DateTime.UtcNow;

        _state.State.Data[key] = value;
        _state.State.ExpirationTimes[key] = DateTime.UtcNow + (ttl ?? TimeSpan.FromHours(1));

        await _state.WriteStateAsync();
    }

    // Custom deactivation logic
    public override ValueTask OnDeactivateAsync(DeactivationReason reason, CancellationToken token)
    {
        // Clean up expired entries before deactivation
        var expiredKeys = _state.State.ExpirationTimes
            .Where(kvp => kvp.Value < DateTime.UtcNow)
            .Select(kvp => kvp.Key)
            .ToList();

        foreach (var key in expiredKeys)
        {
            _state.State.Data.Remove(key);
            _state.State.ExpirationTimes.Remove(key);
        }

        if (expiredKeys.Any)
            return _state.WriteStateAsync();

        return base.OnDeactivateAsync(reason, token);
    }
}
```

### Memory and Resource Management

Implement efficient resource usage patterns:

```csharp
// ✅ GOOD: Memory-efficient grain patterns
public sealed class StreamingGrain : Grain, IStreamingGrain
{
    readonly ILogger<StreamingGrain> _logger;
    readonly SemaphoreSlim _processingSemaphore = new(1, 1);
    CancellationTokenSource? _processingCts;

    public StreamingGrain(ILogger<StreamingGrain> logger)
    {
        _logger = logger;
    }

    public async ValueTask ProcessStreamAsync(IAsyncEnumerable<DataChunk> stream)
    {
        if (_processingCts is not null)
            throw new InvalidOperationException("Stream processing already in progress");

        _processingCts = new CancellationTokenSource();

        try
        {
            await foreach (var chunk in stream.WithCancellation(_processingCts.Token))
            {
                await _processingSemaphore.WaitAsync(_processingCts.Token);
                try
                {
                    await ProcessChunkAsync(chunk);
                }
                finally
                {
                    _processingSemaphore.Release();
                }
            }
        }
        catch (OperationCanceledException)
        {
            _logger.LogInformation("Stream processing cancelled for grain {GrainId}",
                this.GetPrimaryKeyString());
        }
        finally
        {
            _processingCts?.Dispose();
            _processingCts = null;
        }
    }

    public ValueTask CancelProcessingAsync()
    {
        _processingCts?.Cancel();
        return ValueTask.CompletedTask;
    }

    private async ValueTask ProcessChunkAsync(DataChunk chunk)
    {
        // Process chunk efficiently
        using var memoryStream = new MemoryStream();
        await memoryStream.WriteAsync(chunk.Data);

        // Process without keeping large objects in memory
        var result = await AnalyzeDataAsync(memoryStream);
        await StoreResultAsync(result);
    }
}
```

### Concurrent Processing Optimization

Handle concurrent requests efficiently:

```csharp
// ✅ GOOD: Concurrent processing with proper synchronization
public sealed class ConcurrentGrain : Grain, IConcurrentGrain
{
    readonly SemaphoreSlim _semaphore = new(3, 3); // Allow 3 concurrent operations
    readonly ConcurrentDictionary<string, TaskCompletionSource<bool>> _pendingOperations = new();

    public async ValueTask ProcessAsync(string operationId, Func<Task> operation)
    {
        await _semaphore.WaitAsync();

        try
        {
            var tcs = _pendingOperations.GetOrAdd(operationId, _ => new TaskCompletionSource<bool>());

            try
            {
                await operation();
                tcs.TrySetResult(true);
            }
            catch (Exception ex)
            {
                tcs.TrySetException(ex);
                throw;
            }
        }
        finally
        {
            _semaphore.Release();
        }
    }

    public ValueTask<Task> GetOperationStatusAsync(string operationId)
    {
        if (_pendingOperations.TryGetValue(operationId, out var tcs))
            return ValueTask.FromResult(tcs.Task);

        throw new KeyNotFoundException($"Operation {operationId} not found");
    }
}
```

## 7. Error Handling & Resilience

### Grain-Level Error Handling

Implement comprehensive error handling patterns:

```csharp
// ✅ GOOD: Comprehensive error handling in grains
public sealed class ResilientGrain : Grain, IResilientGrain
{
    readonly ILogger<ResilientGrain> _logger;
    readonly IGrainFactory _grainFactory;
    int _retryCount;

    public ResilientGrain(IGrainFactory grainFactory, ILogger<ResilientGrain> logger)
    {
        _grainFactory = grainFactory;
        _logger = logger;
    }

    public async ValueTask<T> ExecuteWithRetryAsync<T>(Func<ValueTask<T>> operation, int maxRetries = 3)
    {
        var exceptions = new List<Exception>();

        for (int attempt = 0; attempt <= maxRetries; attempt++)
        {
            try
            {
                _retryCount++;
                return await operation();
            }
            catch (TransientException ex) when (attempt < maxRetries)
            {
                exceptions.Add(ex);
                _logger.LogWarning(ex, "Transient error on attempt {Attempt}/{MaxRetries}", attempt + 1, maxRetries + 1);

                // Exponential backoff
                await Task.Delay(TimeSpan.FromSeconds(Math.Pow(2, attempt)));
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Non-retryable error after {Attempts} attempts", attempt + 1);
                throw new AggregateException("Operation failed after retries", exceptions.Concat(new[] { ex }));
            }
        }

        throw new AggregateException("Operation failed after all retries", exceptions);
    }

    public async ValueTask HandleCircuitBreakerAsync(Func<ValueTask> operation)
    {
        try
        {
            await operation();
            ResetCircuitBreaker();
        }
        catch (Exception ex) when (IsCircuitBreakerOpen())
        {
            _logger.LogWarning(ex, "Circuit breaker is open, rejecting request");
            throw new CircuitBreakerOpenException("Service is temporarily unavailable");
        }
        catch (Exception ex)
        {
            RecordFailure();
            throw;
        }
    }

    private bool IsCircuitBreakerOpen() => _retryCount > 5;
    private void RecordFailure() => _retryCount++;
    private void ResetCircuitBreaker() => _retryCount = 0;
}
```

### Exception Propagation

Design exceptions that work well in distributed systems:

```csharp
// ✅ GOOD: Orleans-friendly exception design
[GenerateSerializer]
[Alias("UserNotFoundException")]
public sealed class UserNotFoundException : Exception
{
    [Id(0)] public string UserId { get; }

    public UserNotFoundException(string userId)
        : base($"User with ID '{userId}' was not found")
    {
        UserId = userId;
    }

    public UserNotFoundException(string userId, string message)
        : base(message)
    {
        UserId = userId;
    }

    // Parameterless constructor for serialization
    public UserNotFoundException() : base() { }
}

// ✅ GOOD: Typed error results for better error handling
[GenerateSerializer]
[Alias("OperationResult")]
public readonly record struct OperationResult
{
    [Id(0)] public bool Success { get; init; }
    [Id(1)] public string? ErrorMessage { get; init; }
    [Id(2)] public int ErrorCode { get; init; }

    public static OperationResult Succeeded() =>
        new() { Success = true };

    public static OperationResult Failed(string message, int errorCode = 0) =>
        new() { Success = false, ErrorMessage = message, ErrorCode = errorCode };
}
```

## 8. Testing Patterns

### Unit Testing Grains

Test grains with proper mocking and isolation:

```csharp
// ✅ GOOD: Comprehensive grain testing
public class UserGrainTests
{
    readonly Mock<IPersistentState<UserProfile>> _mockState = new();
    readonly Mock<ILogger<UserGrain>> _mockLogger = new();

    [Fact]
    public async Task GetProfileAsync_WhenUserExists_ReturnsProfile()
    {
        // Arrange
        var expectedProfile = new UserProfile { UserId = "user1", Name = "John Doe" };
        _mockState.Setup(s => s.RecordExists).Returns(true);
        _mockState.Setup(s => s.State).Returns(expectedProfile);

        var grain = new UserGrain(_mockState.Object, _mockLogger.Object);

        // Act
        var result = await grain.GetProfileAsync();

        // Assert
        Assert.Equal(expectedProfile, result);
        _mockState.Verify(s => s.RecordExists, Times.Once);
    }

    [Fact]
    public async Task UpdateProfileAsync_WithNullProfile_ThrowsArgumentNullException()
    {
        // Arrange
        var grain = new UserGrain(_mockState.Object, _mockLogger.Object);

        // Act & Assert
        await Assert.ThrowsAsync<ArgumentNullException>(() =>
            grain.UpdateProfileAsync(null!));
    }
}
```

### Integration Testing

Test grain interactions and persistence:

```csharp
// ✅ GOOD: Integration testing with Orleans test cluster
public class OrleansIntegrationTests : IAsyncLifetime
{
    TestCluster? _cluster;
    IClusterClient? _client;

    public async Task InitializeAsync()
    {
        var builder = new TestClusterBuilder()
            .AddSiloBuilderConfigurator<SiloConfigurator>()
            .AddClientBuilderConfigurator<ClientConfigurator>();

        _cluster = builder.Build();
        await _cluster.DeployAsync();

        _client = _cluster.Client;
    }

    public async Task DisposeAsync()
    {
        if (_client is not null)
            await _client.Close();

        if (_cluster is not null)
            await _cluster.StopAllSilosAsync();
    }

    [Fact]
    public async Task UserGrain_PersistsStateCorrectly()
    {
        // Arrange
        var grain = _client!.GetGrain<IUserGrain>("test-user");
        var profile = new UserProfile { UserId = "test-user", Name = "Test User" };

        // Act
        await grain.UpdateProfileAsync(profile);
        var retrievedProfile = await grain.GetProfileAsync();

        // Assert
        Assert.Equal(profile.UserId, retrievedProfile.UserId);
        Assert.Equal(profile.Name, retrievedProfile.Name);
    }

    class SiloConfigurator : ISiloConfigurator
    {
        public void Configure(ISiloBuilder siloBuilder)
        {
            siloBuilder
                .AddMemoryGrainStorage("TestStorage")
                .ConfigureServices(services =>
                {
                    // Configure test services
                });
        }
    }

    class ClientConfigurator : IClientBuilderConfigurator
    {
        public void Configure(IConfiguration configuration, IClientBuilder clientBuilder)
        {
            // Configure test client
        }
    }
}
```

## 9. Deployment & Monitoring

### Health Checks with .NET Aspire

Aspire provides automatic health monitoring and integrates with the dashboard:

```csharp
// OrleansSilo/Program.cs - Aspire health checks are automatic
var builder = WebApplication.CreateBuilder(args);

// Add Aspire service defaults (includes health checks)
builder.AddServiceDefaults();

// Configure Orleans silo
builder.UseOrleans(orleansBuilder =>
{
    // Orleans configuration...
});

var app = builder.Build();

// Map default endpoints (includes health checks at /health)
app.MapDefaultEndpoints();

// Optional: Add custom Orleans health checks
app.MapHealthChecks("/health/orleans", new HealthCheckOptions
{
    ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse,
    ResultStatusCodes =
    {
        [HealthStatus.Healthy] = StatusCodes.Status200OK,
        [HealthStatus.Degraded] = StatusCodes.Status200OK,
        [HealthStatus.Unhealthy] = StatusCodes.Status503ServiceUnavailable
    }
});

app.Run();
```

### Aspire Dashboard Health Monitoring

Monitor Orleans health through the Aspire dashboard:

```csharp
// OrleansSilo/Program.cs - Add detailed health reporting
builder.Services.AddHealthChecks()
    .AddCheck<OrleansClusterHealthCheck>("orleans-cluster")
    .AddCheck<OrleansGrainHealthCheck>("orleans-grains");

public sealed class OrleansClusterHealthCheck : IHealthCheck
{
    readonly IClusterClient _client;

    public OrleansClusterHealthCheck(IClusterClient client)
    {
        _client = client;
    }

    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken token)
    {
        try
        {
            var managementGrain = _client.GetGrain<IManagementGrain>(0);
            var hosts = await managementGrain.GetHosts();

            var data = new Dictionary<string, object>
            {
                ["ActiveSilos"] = hosts.Count,
                ["TotalActivations"] = hosts.Sum(h => h.Statistics.ActivationCount),
                ["TotalDeactivations"] = hosts.Sum(h => h.Statistics.DeactivationCount)
            };

            return hosts.Any()
                ? HealthCheckResult.Healthy("Orleans cluster operational", data)
                : HealthCheckResult.Unhealthy("No active silos found", data);
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("Cluster connectivity failed", ex);
        }
    }
}
```

### Aspire Health Check Integration

Configure health checks in your AppHost for comprehensive monitoring:

```csharp
// AspireApp.AppHost/Program.cs - Health monitoring setup
var builder = DistributedApplication.CreateBuilder(args);

// Add Orleans silo with health monitoring
var orleansSilo = builder.AddProject<Projects.OrleansSilo>("orleans-silo")
    .WithReplicas(2)
    .WithHealthCheck(); // Enable health checks

// Add client service with health checks
var orleansClient = builder.AddProject<Projects.OrleansClient>("orleans-client")
    .WithReference(orleansSilo)
    .WithHealthCheck();

// Add health check UI (optional)
builder.Services.AddHealthChecksUI();

builder.Build().Run();
```

### Service Discovery Health Integration

Aspire automatically removes unhealthy services from service discovery:

```csharp
// Automatic health-based service discovery
public sealed class WeatherService
{
    readonly IGrainFactory _grainFactory;

    public WeatherService(IGrainFactory grainFactory)
    {
        _grainFactory = grainFactory; // Aspire ensures this connects to healthy silos
    }

    public async ValueTask<WeatherData> GetWeatherAsync(string location)
    {
        // Aspire handles routing to healthy Orleans silos automatically
        var weatherGrain = _grainFactory.GetGrain<IWeatherGrain>(location);
        return await weatherGrain.GetCurrentWeatherAsync();
    }
}
```

### Aspire Dashboard Integration

Monitor your Orleans application through the Aspire dashboard:

```bash
# Start your AppHost project
dotnet run --project AspireApp.AppHost

# Open Aspire Dashboard at http://localhost:15888
# You'll see:
# - Orleans silo health status
# - Service dependencies and communication
# - Resource usage (CPU, memory, network)
# - Log aggregation from all services
# - Real-time metrics and traces
```

### Aspire Service Communication Patterns

Best practices for service communication in Aspire:

```csharp
// ✅ GOOD: Service-to-service communication via Orleans grains
public sealed class OrderProcessingService
{
    readonly IGrainFactory _grainFactory;

    public OrderProcessingService(IGrainFactory grainFactory)
    {
        _grainFactory = grainFactory; // Injected by Aspire DI
    }

    public async ValueTask<OrderConfirmation> ProcessOrderAsync(OrderRequest request)
    {
        // Get Orleans grain (Aspire handles service discovery)
        var orderGrain = _grainFactory.GetGrain<IOrderGrain>(request.OrderId);

        // Call grain method
        var order = await orderGrain.CreateOrderAsync(request);

        // Get user grain for validation
        var userGrain = _grainFactory.GetGrain<IUserGrain>(request.UserId);
        var userProfile = await userGrain.GetProfileAsync();

        // Process payment via payment grain
        var paymentGrain = _grainFactory.GetGrain<IPaymentGrain>(request.OrderId);
        var paymentResult = await paymentGrain.ProcessPaymentAsync(order.Total);

        return new OrderConfirmation(order.Id, paymentResult.Success);
    }
}
```

### Metrics and Monitoring

Implement comprehensive monitoring:

```csharp
// ✅ GOOD: Metrics collection for Orleans applications
public sealed class GrainMetricsGrain : Grain, IGrainMetricsGrain
{
    readonly IGrainFactory _grainFactory;
    readonly ILogger<GrainMetricsGrain> _logger;
    readonly Meter _meter;
    readonly Counter<long> _grainActivations;
    readonly Histogram<double> _grainExecutionTime;

    public GrainMetricsGrain(IGrainFactory grainFactory, ILogger<GrainMetricsGrain> logger)
    {
        _grainFactory = grainFactory;
        _logger = logger;

        _meter = new Meter("Orleans.Grains");
        _grainActivations = _meter.CreateCounter<long>("grain_activations_total", "Total number of grain activations");
        _grainExecutionTime = _meter.CreateHistogram<double>("grain_execution_duration", "ms", "Duration of grain method execution");
    }

    public async ValueTask RecordMetricsAsync(string grainType, string methodName, TimeSpan duration)
    {
        _grainActivations.Add(1, new KeyValuePair<string, object?>("grain_type", grainType));
        _grainExecutionTime.Record(duration.TotalMilliseconds,
            new KeyValuePair<string, object?>("grain_type", grainType),
            new KeyValuePair<string, object?>("method", methodName));

        // Log performance issues
        if (duration > TimeSpan.FromSeconds(5))
        {
            _logger.LogWarning("Slow grain execution: {GrainType}.{MethodName} took {Duration}ms",
                grainType, methodName, duration.TotalMilliseconds);
        }
    }

    public async ValueTask<GrainMetrics> GetMetricsAsync()
    {
        var managementGrain = _grainFactory.GetGrain<IManagementGrain>(0);
        var detailedMetrics = await managementGrain.GetDetailedGrainStatistics();

        return new GrainMetrics
        {
            TotalActivations = detailedMetrics.Sum(s => s.ActivationCount),
            TotalDeactivations = detailedMetrics.Sum(s => s.DeactivationCount),
            ActiveGrains = detailedMetrics.Sum(s => s.ActivationCount - s.DeactivationCount),
            AverageExecutionTime = detailedMetrics.Average(s => s.AverageExecutionTime.TotalMilliseconds)
        };
    }
}
```

### Logging Best Practices

Implement structured logging for distributed systems:

```csharp
// ✅ GOOD: Structured logging in Orleans grains
public sealed class LoggingGrain : Grain, ILoggingGrain
{
    readonly ILogger<LoggingGrain> _logger;

    public LoggingGrain(ILogger<LoggingGrain> logger)
    {
        _logger = logger;
    }

    public async ValueTask ProcessRequestAsync(RequestData request)
    {
        using var scope = _logger.BeginScope(new Dictionary<string, object>
        {
            ["GrainId"] = this.GetPrimaryKeyString(),
            ["GrainType"] = this.GetType().Name,
            ["RequestId"] = request.Id,
            ["UserId"] = request.UserId
        });

        _logger.LogInformation("Processing request {RequestId} for user {UserId}",
            request.Id, request.UserId);

        try
        {
            var startTime = Stopwatch.GetTimestamp();
            var result = await ProcessAsync(request);
            var elapsed = Stopwatch.GetElapsedTime(startTime);

            _logger.LogInformation("Request {RequestId} completed successfully in {Duration}ms",
                request.Id, elapsed.TotalMilliseconds);

            return result;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Request {RequestId} failed with error: {ErrorMessage}",
                request.Id, ex.Message);

            // Log additional context for debugging
            _logger.LogDebug("Request details: {@Request}", request);

            throw;
        }
    }
}
```

## 10. Advanced Patterns & Best Practices

### Grain Placement Strategies

Implement custom grain placement for optimal performance:

```csharp
// ✅ GOOD: Custom grain placement strategy
[AttributeUsage(AttributeTargets.Class)]
public sealed class PreferredSiloAttribute : Attribute
{
    public string SiloId { get; }

    public PreferredSiloAttribute(string siloId)
    {
        SiloId = siloId;
    }
}

public sealed class CustomPlacementDirector : IPlacementDirector
{
    public async ValueTask<PlacementResult> OnAddActivation(
        PlacementStrategy strategy, PlacementTarget target, IPlacementContext context)
    {
        if (target.GrainIdentity.Type.GetCustomAttribute<PreferredSiloAttribute>() is { } attribute)
        {
            // Find silo with preferred ID
            var preferredSilo = context.Silos.FirstOrDefault(s => s.SiloAddress.ToString().Contains(attribute.SiloId));
            if (preferredSilo is not null)
                return new PlacementResult { SiloAddress = preferredSilo.SiloAddress };
        }

        // Default placement
        return await Task.FromResult(new PlacementResult());
    }
}
```

### Circuit Breaker Pattern

Implement circuit breaker for resilient grain communication:

```csharp
// ✅ GOOD: Circuit breaker pattern for grain resilience
public sealed class CircuitBreakerGrain : Grain, ICircuitBreakerGrain
{
    readonly IPersistentState<CircuitBreakerState> _state;
    readonly ILogger<CircuitBreakerGrain> _logger;

    public CircuitBreakerGrain(
        [PersistentState("circuit", "CircuitBreakerStorage")] IPersistentState<CircuitBreakerState> state,
        ILogger<CircuitBreakerGrain> logger)
    {
        _state = state;
        _logger = logger;
    }

    public async ValueTask<T> ExecuteWithCircuitBreakerAsync<T>(Func<ValueTask<T>> operation)
    {
        if (_state.State.Status == CircuitBreakerStatus.Open)
        {
            if (DateTimeOffset.UtcNow - _state.State.LastFailureTime < _state.State.Timeout)
                throw new CircuitBreakerOpenException();

            // Try to close circuit breaker
            _state.State.Status = CircuitBreakerStatus.HalfOpen;
            await _state.WriteStateAsync();
        }

        try
        {
            var result = await operation();
            await OnSuccessAsync();
            return result;
        }
        catch (Exception ex)
        {
            await OnFailureAsync(ex);
            throw;
        }
    }

    private async ValueTask OnSuccessAsync()
    {
        if (_state.State.Status == CircuitBreakerStatus.HalfOpen)
        {
            _state.State.Status = CircuitBreakerStatus.Closed;
            _state.State.FailureCount = 0;
            await _state.WriteStateAsync();
        }
    }

    private async ValueTask OnFailureAsync(Exception ex)
    {
        _state.State.FailureCount++;
        _state.State.LastFailureTime = DateTimeOffset.UtcNow;

        if (_state.State.FailureCount >= _state.State.FailureThreshold)
        {
            _state.State.Status = CircuitBreakerStatus.Open;
            _logger.LogWarning("Circuit breaker opened due to {FailureCount} failures", _state.State.FailureCount);
        }

        await _state.WriteStateAsync();
    }
}

[GenerateSerializer]
public enum CircuitBreakerStatus { Closed, Open, HalfOpen }

[GenerateSerializer]
public sealed class CircuitBreakerState
{
    [Id(0)] public CircuitBreakerStatus Status { get; set; } = CircuitBreakerStatus.Closed;
    [Id(1)] public int FailureCount { get; set; }
    [Id(2)] public int FailureThreshold { get; set; } = 5;
    [Id(3)] public TimeSpan Timeout { get; set; } = TimeSpan.FromMinutes(1);
    [Id(4)] public DateTimeOffset LastFailureTime { get; set; }
}
```

This comprehensive Orleans coding style guide provides best practices for building high-performance, maintainable
distributed applications with proper grain design, state management, communication patterns, and performance
optimization techniques.

# End of Kiro Steering File