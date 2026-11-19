# Dignus Library

> Licensed under the [MIT License](./LICENSE)  
> © 2021–2026 EomTaeWook

**High-performance modular framework ecosystem focused on runtime efficiency and GC avoidance.**  
All modules are designed for performance-critical environments with zero-copy and zero-GC architecture.

This repository is a public open-source distribution of the Dignus Library ecosystem.  
The original development repository is private, and the project has been maintained since 2021.

---

## Modules

| Module | Description | NuGet |
| :--- | :--- | :--- |
| [**Dignus.Core**](./publish/Dignus.Core.md) | Foundation layer with allocation-free collections, deterministic coroutine scheduling, lightweight DI, and framework utilities | [![NuGet](https://img.shields.io/nuget/v/Dignus.svg)](https://www.nuget.org/packages/Dignus) |
| [**Dignus.Sockets**](./publish/Dignus.Sockets.md) | High-performance C# networking framework supporting **TCP, TLS, and UDP**, built on event-driven async sockets with zero-copy architecture | [![NuGet](https://img.shields.io/nuget/v/Dignus.Sockets.svg)](https://www.nuget.org/packages/Dignus.Sockets) |
| [**Dignus.Actor.Core**](https://github.com/EomTaeWook/Dignus.ActorServer/blob/main/publish/Dignus.Actor.Core.md) | High-performance actor runtime core with dispatcher-based scheduling, actor references, mailbox processing, and request-response support | [![NuGet](https://img.shields.io/nuget/v/Dignus.Actor.Core.svg)](https://www.nuget.org/packages/Dignus.Actor.Core) |
| [**Dignus.ActorServer**](https://github.com/EomTaeWook/Dignus.ActorServer/blob/main/publish/Dignus.Actor.Network.md) | Actor-based networking layer built on top of Dignus.Sockets, supporting TCP/TLS actor server integration | [![NuGet](https://img.shields.io/nuget/v/Dignus.ActorServer.svg)](https://www.nuget.org/packages/Dignus.ActorServer) |
| [**Dignus.Log**](https://github.com/EomTaeWook/Dignus.Log/blob/main/publish/Dignus.Log.md) | Attribute-driven logging system with configurable targets | [![NuGet](https://img.shields.io/nuget/v/Dignus.Log.svg)](https://www.nuget.org/packages/Dignus.Log) |
| [**Dignus.Extensions.Logging**](https://github.com/EomTaeWook/Dignus.Log/blob/main/publish/Dignus.Extensions.Logging.md) | Integration with Microsoft.Extensions.Logging | [![NuGet](https://img.shields.io/nuget/v/Dignus.Extensions.Logging.svg)](https://www.nuget.org/packages/Dignus.Extensions.Logging) |
| [**Dignus.Unity**](https://github.com/EomTaeWook/Dignus.Unity) | Unity integration package distributed through Unity Package Manager | [![NuGet](https://img.shields.io/nuget/v/Dignus.Unity.svg)](https://www.nuget.org/packages/Dignus.Unity) |
| [**Dignus.Godot**](https://github.com/EomTaeWook/Dignus.Godot) | Godot .NET integration with dependency injection, scene lifecycle management, resource loading, node pooling, coroutines, and value change notifications | [![NuGet](https://img.shields.io/nuget/v/Dignus.Godot.svg)](https://www.nuget.org/packages/Dignus.Godot) |
| [**Dignus.Commands**](https://github.com/EomTaeWook/Dignus.Commands) | Modular command framework for local and Telnet-based execution, with hierarchical command paths and middleware pipeline support | [![NuGet](https://img.shields.io/nuget/v/Dignus.Commands.svg)](https://www.nuget.org/packages/Dignus.Commands) |

---

## Highlights

- **Zero Allocation:** Designed to eliminate GC overhead during runtime  
- **Zero-Copy Architecture:** Direct buffer access across networking layers  
- **Precompiled Execution:** Avoids reflection for high-performance runtime paths  
- **Thread-Safe Design:** Synchronized variants for concurrent environments  
- **Extensible and Modular:** Each module can operate independently or as part of a full-stack system
