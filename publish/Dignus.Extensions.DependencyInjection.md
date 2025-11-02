# Dignus.Extensions.DependencyInjection

**Extension module that integrates `Dignus.DependencyInjection` with `Microsoft.Extensions.DependencyInjection`.**  
Provides seamless registration of `[Injectable]` types into the standard ASP.NET Core `IServiceCollection`,  
enabling hybrid use of Dignus' high-performance DI features within the .NET dependency injection ecosystem.

---

## Features

| Feature | Description |
| :--- | :--- |
| **Automatic Registration** | Scans assemblies for `[Injectable]` attributes and registers services automatically. |
| **Hybrid DI Support** | Bridges `Dignus.DependencyInjection` and `Microsoft.Extensions.DependencyInjection`. |
| **Property Injection** | Automatically injects `[Inject]` properties when resolving dependencies. |
| **LifeScope Mapping** | Maps Dignus `LifeScope` (`Transient` / `Singleton`) to .NET service lifetimes. |
| **Zero Reflection Overhead** | Uses prebuilt constructor delegates for fast service activation. |

---

## Core API

| Method | Description |
| :--- | :--- |
| `RegisterDependencies(this IServiceCollection serviceCollection, Assembly assembly)` | Scans the specified assembly for `[Injectable]` types and registers them into the service collection. |

---

## Example

```csharp
using Dignus.DependencyInjection.Attributes;
using Dignus.Extensions.DependencyInjection;
using Microsoft.Extensions.DependencyInjection;
using System.Reflection;

[Injectable(LifeScope.Singleton, typeof(IMyService))]
public class MyService : IMyService
{
    [Inject] private ILogger _logger { get; set; }
}

var serviceCollection = new ServiceCollection();

// Automatically register all [Injectable] classes from the assembly
serviceCollection.RegisterDependencies(Assembly.GetExecutingAssembly());

var serviceProvider = serviceCollection.BuildServiceProvider();

var myService = serviceProvider.GetService<IMyService>();

```

## Design Principles

- **Optimized Activation:** Services are instantiated using precompiled delegates, avoiding reflection overhead.
- **Property Injection:** `[Inject]` attributes are applied recursively for dependency resolution.  
- **Compatibility Layer:** Enables existing .NET Core apps to leverage Dignus' DI system.  
- **Extensible & Non-Intrusive:** Works transparently with the standard service registration flow.  
- **Zero Runtime Configuration:** All bindings are determined automatically at startup.  

---