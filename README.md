# Dignus Library

> Licensed under the [MIT License](./LICENSE)  
> © 2021–2026 EomTaeWook

**High-performance modular framework ecosystem focused on runtime efficiency and GC avoidance.**  
All modules are designed for performance-critical environments with zero-copy and zero-GC architecture.

---

## Modules

| Module | Description | NuGet |
| :--- | :--- | :--- |
| [**Dignus.Core**](./publish/Dignus.Core.md) | Foundation layer with allocation-free collections, deterministic coroutine scheduling, lightweight DI, and framework utilities | [![NuGet](https://img.shields.io/nuget/v/Dignus.svg)](https://www.nuget.org/packages/Dignus) |
| [**Dignus.Sockets**](./publish/Dignus.Sockets.md) | Event-driven async socket framework with zero-copy I/O and optimized protocol pipeline | [![NuGet](https://img.shields.io/nuget/v/Dignus.Sockets.svg)](https://www.nuget.org/packages/Dignus.Sockets) |
| [**Dignus.Log**](./publish/Dignus.Log.md) | Attribute-driven logging system with configurable targets and zero-GC rendering | [![NuGet](https://img.shields.io/nuget/v/Dignus.Log.svg)](https://www.nuget.org/packages/Dignus.Log) |
| [**Dignus.Unity**](./publish/Dignus.Unity.md) | Lightweight Unity integration with coroutine, pooling, DI, and reactive scene architecture | [![NuGet](https://img.shields.io/nuget/v/Dignus.Unity.svg)](https://www.nuget.org/packages/Dignus.Unity) |

---

## Highlights

- **Zero Allocation:** Designed to eliminate GC overhead during runtime  
- **Zero-Copy Architecture:** Direct buffer access across networking layers  
- **Precompiled Execution:** Avoids reflection for high-performance runtime paths 
- **Thread-Safe Design:** Synchronized variants for concurrent environments  
- **Extensible and Modular:** Each module can operate independently or as part of a full-stack system  
