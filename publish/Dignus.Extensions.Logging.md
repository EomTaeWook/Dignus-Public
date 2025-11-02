# Dignus.Extensions.Logging

**Integration module connecting `Dignus.Log` with `Microsoft.Extensions.Logging`.**  
Provides a lightweight, zero-GC logging bridge that enables seamless use of Dignus’ high-performance logger  
within the standard ASP.NET Core and .NET logging infrastructure.

---

## Features

| Feature | Description |
| :--- | :--- |
| **ASP.NET Core Integration** | Enables Dignus logging through the `ILogger` interface and `ILoggerFactory`. |
| **Zero-GC Logging** | Inherits the allocation-free design of `Dignus.Log` for high throughput. |
| **Category Support** | Adds log category support (`${category}`) for structured log output. |
| **Scope Handling** | Fully compatible with .NET’s `IExternalScopeProvider`. |
| **Automatic Registration** | `AddDignusLog()` extension method integrates logging and configuration automatically. |

---

## Core APIs

| API | Purpose |
| :--- | :--- |
| `AddDignusLog(this IServiceCollection services, LogConfiguration logConfiguration)` | Registers the Dignus logging provider and initializes log configuration. |
| `DignusLoggerProvider` | Implements `ILoggerProvider` to route .NET log events into the Dignus logger. |
| `DignusLogger` | Handles `ILogger` calls, converts log levels, and formats categorized log entries. |

---

## Example

```csharp
using Dignus.Extensions.Logging;
using Dignus.Log;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

var services = new ServiceCollection();

// Load Dignus logging configuration
var logConfig = LogConfigXmlReader.Load("log.config");

// Integrate with Microsoft.Extensions.Logging
services.AddLogging(builder =>
{
    builder.ClearProviders();
    builder.AddDignusLog(logConfig);
});

var provider = services.BuildServiceProvider();
var logger = provider.GetRequiredService<ILogger<Program>>();

logger.LogInformation("Hello from Dignus Logging!");
```

## Design Principles

- **Zero Allocation:** All log formatting and dispatch avoid GC allocations.  
- **Category Awareness:** Extends `LogEvent` to include category metadata for structured logging.  
- **Optimized Formatting:** Uses cached delegates for element rendering to minimize overhead.
- **Full .NET Integration:** Works natively with ASP.NET Core’s logging pipeline.  
- **Scoped Logging Support:** Handles `IExternalScopeProvider` and `BeginScope()` seamlessly.  

---