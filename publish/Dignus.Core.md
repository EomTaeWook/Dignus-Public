# Dignus.Core

**Foundation layer of the Dignus framework**  
Provides core runtime utilities such as dependency injection, coroutine scheduling, collections, object pooling, and pipeline systems.  
All modules in the Dignus ecosystem depend on this base library.

---

## Overview

- Core of the Dignus ecosystem, providing fundamental abstractions and utilities.  
- Includes lightweight DI, coroutine engine, memory-safe collections, thread-safe variants, and object pooling.  
- Designed for **zero-GC**, **deterministic**, and **high-throughput** execution environments.

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
