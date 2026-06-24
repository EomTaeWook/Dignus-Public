# Dignus.Core

**Shared runtime foundation for the Dignus ecosystem.**

Provides common runtime components used across Dignus libraries, including collections, dependency injection, coroutine scheduling, object pooling, and pipeline utilities.

---

## Overview

* Base runtime library shared by Dignus modules.
* Includes performance-oriented collections, lightweight dependency injection, deterministic coroutine scheduling, and reusable framework utilities.
* Designed for high-throughput runtime systems with minimal allocation in steady-state execution paths.

---

## Namespaces

| Namespace                      | Description                                                                     |
| :----------------------------- | :------------------------------------------------------------------------------ |
| **Dignus.Collections**         | Array-based collections and concurrent queue implementations.                   |
| **Dignus.Coroutine**           | Lightweight coroutine scheduling with deterministic updates.                    |
| **Dignus.DependencyInjection** | Minimal dependency injection container with constructor and property injection. |
| **Dignus.Framework**           | Object pooling, singleton, and pipeline utilities.                              |

---

## Modules

### Dignus.Collections

Contains array-based collections and multi-producer, single-consumer queues.

* `ArrayQueue<T>` — expandable queue optimized for sequential reads and writes.
* `CompactableArrayQueue<T>` — queue that reuses cleared slots through compaction.
* `UniqueSet<T>` — custom hash-based unique collection.
* `SynchronizedArrayQueue<T>` / `SynchronizedUniqueSet<T>` — synchronized wrappers for concurrent access.
* `MpscBoundedQueue<T>` — fixed-capacity multi-producer, single-consumer queue with lock-free enqueue and single-threaded dequeue.
* `MpscUnboundedQueue<T>` — unbounded multi-producer, single-consumer queue with lock-free enqueue and single-threaded dequeue.

### Dignus.Coroutine

Implements deterministic coroutine scheduling.

* `CoroutineHandle` — represents an active coroutine.
* `CoroutineHandler` — manages coroutine execution, lifecycle, and nested enumerators.
* `DelayInSeconds`, `DelayInMilliseconds`, `WaitWhile` — built-in wait conditions.

### Dignus.DependencyInjection

Provides lightweight service registration and resolution.

* `ServiceContainer`, `ServiceProvider`, `ServiceCollection` — container infrastructure.
* `ServiceRegistration` — registration metadata for services and factories.
* `LifeScope` — service lifetime definition: `Transient` or `Singleton`.
* `ConstructorDelegateFactory` — creates and caches constructor delegates.
* `InjectableAttribute`, `InjectAttribute`, `InjectConstructorAttribute` — attribute-based registration and injection.
* `ServiceContainerExtensions`, `ServiceCollectionExtensions`, `ServiceProviderExtensions` — registration and resolution helpers.

### Dignus.Framework

Provides reusable memory and execution utilities.

* `ObjectPoolBase<T>` / `ObjectPool<T>` — reusable object pooling infrastructure.
* `Singleton<T>` — thread-safe lazy singleton helper.
* `AsyncPipeline<TContext>` — async middleware pipeline using ref-based context passing.

  * `AsyncPipelineDelegate<TContext>` — middleware delegate signature.
  * `AsyncPipelineNext<TContext>` — continuation used to invoke the next middleware.
  * `IAsyncMiddleware<TContext>` / `AsyncHandlerMiddleware<TContext>` — middleware interface and delegate adapter.
