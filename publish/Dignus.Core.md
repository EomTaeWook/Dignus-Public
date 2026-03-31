# Dignus.Core

**Shared runtime foundation for the Dignus ecosystem**  
Provides the low-level building blocks used across Dignus libraries, including collections, dependency injection, coroutine scheduling, object pooling, and pipeline utilities.

---

## Overview

- Base runtime library used by other Dignus modules.  
- Includes performance-oriented collections, lightweight dependency injection, deterministic coroutine scheduling, and reusable framework utilities.  
- Designed for performance-oriented runtime systems, with a focus on high-throughput execution.

---

## Namespaces

| Namespace | Description |
| :--- | :--- |
| **Dignus.Collections** | Zero-GC collections including ArrayQueue, CompactableArrayQueue, and UniqueSet. |
| **Dignus.Coroutine** | Lightweight coroutine handler with deterministic updates. |
| **Dignus.DependencyInjection** | Minimal DI container with constructor and property injection support. |
| **Dignus.Framework** | Object pooling, singleton, and pipeline utilities for performance-critical systems. |

---

## Modules

### Dignus.Collections
Contains zero-allocation, array-based and set-based data structures.

- `ArrayQueue<T>` — expandable queue optimized for sequential reads/writes.  
- `CompactableArrayQueue<T>` — auto-compacting queue that reuses cleared slots.  
- `UniqueSet<T>` — custom hash-based unique collection with minimal allocation.  
- `SynchronizedArrayQueue<T>` / `SynchronizedUniqueSet<T>` — thread-safe wrappers.
- `MpscBoundedQueue<T>` — fixed-capacity multi-producer, single-consumer queue optimized for lock-free enqueue and single-threaded dequeue.

### Dignus.Coroutine
Implements allocation-free coroutine scheduling.

- `CoroutineHandle` — represents an active coroutine instance.  
- `CoroutineHandler` — manages coroutine lifecycles and nested enumerators.  
- `DelayInSeconds`, `DelayInMilliseconds`, `WaitWhile` — built-in wait conditions.

### Dignus.DependencyInjection
Minimal, high-speed dependency injection framework.

- `ServiceContainer`, `ServiceProvider`, `ServiceCollection` — container infrastructure.  
- `ServiceRegistration` — metadata for registered services and factories.  
- `LifeScope` — defines lifetime (`Transient` / `Singleton`).  
- `ConstructorDelegateFactory` — builds and caches fast constructor delegates for registered services.  
- `InjectableAttribute`, `InjectAttribute`, `InjectConstructorAttribute` — attribute-driven registration.  
- `ServiceContainerExtensions`, `ServiceCollectionExtensions`, `ServiceProviderExtensions` — helper APIs for simplified registration and resolution.

### Dignus.Framework
Utility layer providing reusable object and execution management systems.

- `ObjectPoolBase<T>` / `ObjectPool<T>` — memory-stable pooling mechanism.  
- `Singleton<T>` — thread-safe lazy singleton helper.  
- `AsyncPipeline<TContext>` — allocation-free async middleware pipeline using ref-based context passing.  
  - `AsyncPipelineDelegate<TContext>` — delegate signature for middleware functions.  
  - `AsyncPipelineNext<TContext>` — continuation struct controlling next middleware execution.  
  - `IAsyncMiddleware<TContext>` / `AsyncHandlerMiddleware<TContext>` — interface and adapter for composing pipelines.  
