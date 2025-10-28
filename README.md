# Dignus Framework Suite

**A high-performance, modular framework ecosystem for real-time servers, network engines, and Unity integration.**  
All components are designed for speed, extensibility, and zero-GC operation.

---

## 🧱 Projects

| Project | Description |
| :--- | :--- |
| **[Dignus.Core](#dignuscore)** | Foundation layer with zero-GC collections, coroutine scheduler, lightweight DI, and framework utilities |
| **[Dignus.Sockets](#dignussockets)** | High-performance, event-driven async socket library optimized for scalability |
| **[Dignus.Unity](#dignusunity)** | Lightweight Unity integration with coroutine, pooling, DI, and scene management |

---

## ⚙️ Common Highlights

- **Zero Allocation Runtime** — GC-free collections, buffer management, and coroutine systems  
- **Expression Tree Compilation** — No runtime reflection overhead  
- **Thread-Safe Components** — Synchronized variants for concurrent systems  
- **Lightweight & Extensible** — Modular design, no external dependencies  
- **MIT Licensed** — Free for commercial and open-source use  

---

## Dignus.Core

**High-performance modular core library for the Dignus.**  
Zero-GC collections, coroutine scheduler, lightweight DI container, and extensible framework.

### Modules
| Module | Description |
| :--- | :--- |
| **Collections** | Zero-GC, high-throughput data structures for real-time and network systems |
| **Coroutine** | Lightweight async coroutine system without async/await overhead |
| **DependencyInjection** | Minimal DI container using expression tree compilation for fast runtime resolution |
| **Framework** | Middleware pipeline, object pooling, and singleton utilities |

### Highlights
- Zero-GC memory model  
- Expression-based constructor compilation  
- Modular and composable design  
- Thread-safe collection variants  
- No external dependencies (pure .NET Standard)

---

## Dignus.Sockets

**High-performance, event-driven async socket library built for extensibility and speed.**  
[![Performance Benchmark](https://img.shields.io/badge/Performance-Benchmark-blueviolet?logo=github)](https://github.com/EomTaeWook/ServerPerformanceBenchmark)

### Key Features
| Feature | Description |
| :--- | :--- |
| **Async Session Model** | Event-driven, per-session TCP architecture using SAEA |
| **Pluggable Protocols** | Replaceable serializers and packet handlers |
| **Protocol Pipeline** | Attribute-based mapping of packet IDs to handler methods |
| **Custom Middleware** | Authentication, logging, and security pipeline hooks |
| **Session Extensibility** | Component-based session customization |

### Core APIs
`Session`, `ServerBase`, `ClientBase`, `ISessionComponent`,  
`ProtocolHandlerMapper<THandler, TBody>`, `ProtocolPipelineInvoker<TContext, THandler, TBody>`

### Design Focus
- SAEA-based I/O for max throughput  
- ArrayQueue-based send buffer for minimal GC  
- Expression Tree dispatch for handler performance  
- Fully event-driven async session model  

---

## Dignus.Unity

**Lightweight Unity extension for the Dignus framework.**

| Feature | Description |
| :--- | :--- |
| **Coroutine Manager** | Efficient `IEnumerator`-based coroutine system |
| **Object Pooling** | GameObject & Component pooling for performance |
| **Dependency Injection** | DI-ready service container for Unity context |
| **Scene System** | Async loading, scene controller base, and event hooks |
| **BindableProperty** | Reactive property system for data binding |
| **Resource Manager** | Centralized prefab and resource management |

### Core APIs
`DignusUnityCoroutineManager`  
`DignusUnityObjectPool`  
`SceneControllerBase<TScene, TModel>`  
`DignusUnityServiceContainer`  
`DignusUnityResourceManager`  
`BindableProperty<T>`  
`DignusUnitySceneManager`

---
