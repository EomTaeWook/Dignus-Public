
# Dignus Library

> Licensed under the [MIT License](./LICENSE)  
> © 2021 EomTaeWook
> 

**High-performance modular framework ecosystem focused on runtime efficiency and GC avoidance.**  
All modules are designed for performance-critical environments with zero-copy and zero-GC architecture.

---

## Modules

| Module | Description |
| :--- | :--- |
| [**Dignus.Core**](./publish/Dignus.Core.md) | Foundation layer with allocation-free collections, deterministic coroutine scheduling, lightweight DI, and framework utilities |
| [**Dignus.Sockets**](./publish/Dignus.Sockets.md) | Event-driven async socket framework with zero-copy I/O and expression-compiled protocol pipeline |
| [**Dignus.Log**](./publish/Dignus.Log.md) | Attribute-driven logging system with configurable targets and zero-GC rendering |
| [**Dignus.Unity**](./publish/Dignus.Unity.md) | Lightweight Unity integration with coroutine, pooling, DI, and reactive scene architecture |

---

## Highlights

- **Zero Allocation:** Designed to eliminate GC overhead during runtime  
- **Zero-Copy Architecture:** Direct buffer access across networking layers  
- **Expression Tree Compilation:** Reflection-free execution for critical paths  
- **Thread-Safe Design:** Synchronized variants for concurrent environments  
- **Extensible and Modular:** Each module can operate independently or as part of a full-stack system  

---