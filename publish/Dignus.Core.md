# Dignus

**High-performance modular core library focused on runtime efficiency and GC avoidance.**  
Provides allocation-free collections, deterministic coroutine scheduling, lightweight DI, and extensible framework utilities.

---

## Modules

| Module | Description |
| :--- | :--- |
| **Collections** | High-performance, allocation-free data structures designed to minimize GC overhead |
| **Coroutine** | Lightweight coroutine scheduler for deterministic async operations without async/await |
| **DependencyInjection** | Minimal DI container using expression tree compilation for fast runtime resolution |
| **Framework** | Middleware pipeline, object pooling, and singleton utilities for efficient runtime control |

---

## Highlights

- **Zero Allocation:** Core components designed to avoid garbage collection in steady state  
- **Performance-Oriented:** Expression tree compilation replaces reflection for faster runtime execution  
- **Extensible:** Modular and composable structure adaptable to various architectures  
- **Thread-Safe:** Synchronized variants provided where concurrency is required  
- **Lightweight:** Pure .NET implementation, no external dependencies  

---

## Components

### Collections
- `ArrayQueue<T>`, `CompactableArrayQueue<T>` — zero-copy, expandable queues  
- `UniqueSet<T>` — allocation-free hash set implementation  
- Thread-safe variants: `SynchronizedArrayQueue<T>`, `SynchronizedUniqueSet<T>`

### Coroutine
- `CoroutineHandler`, `CoroutineHandle` — deterministic coroutine scheduler  
- Built-in waits: `DelayInSeconds`, `DelayInMilliseconds`, `WaitWhile`  
- Zero-GC execution, supports nested coroutines and completion callbacks

### DependencyInjection
- Supports constructor and property injection  
- Attribute-based setup via `[Injectable]`, `[InjectConstructor]`  
- Lifetime management: `Transient` / `Singleton`  
- Reflection-free resolution using expression tree-compiled factories  

### Framework
- **Pipeline:** `RefMiddlewarePipeline<TContext>` — ref-based middleware chain without allocation  
- **ObjectPool:** allocation-free instance reuse with internal tracking  
- **Singleton:** thread-safe, lazy initialization utility  

---

## Key Advantages

- Designed for GC-free execution under load  
- Optimized for high-performance, low-latency environments  
- Simple, extensible, and dependency-free architecture  
- Ideal for performance-critical systems and real-time frameworks  

---
