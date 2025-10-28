# Dignus.Unity

**Lightweight Unity extension built on top of the Dignus framework.**  
Provides efficient coroutine management, pooling, and DI-ready scene architecture for scalable Unity projects.

---

## Features

| Feature | Description |
| :--- | :--- |
| **Coroutine Manager** | High-performance `IEnumerator`-based coroutine scheduler for Unity environments |
| **Object Pooling** | Reusable pool system for `GameObject` and `Component` instances to minimize allocations |
| **Dependency Injection** | DI-ready service container for in-game systems and controllers |
| **Scene System** | Base controller architecture with async loading, scene events, and state binding |
| **Bindable Property** | Reactive property system for data-driven UI and gameplay logic |
| **Resource Management** | Centralized prefab and resource loader with path-based access |

---

## Core APIs

| API | Purpose |
| :--- | :--- |
| `DignusUnityCoroutineManager` | Manages coroutines with high performance and low GC pressure |
| `DignusUnityObjectPool` | Generic pooling system for Unity objects |
| `SceneControllerBase<TScene, TModel>` | Base class for structured, DI-friendly scene controllers |
| `DignusUnityServiceContainer` | Lightweight service container for Unity dependency management |
| `DignusUnityResourceManager` | Handles prefab loading, caching, and resource lifecycle |
| `BindableProperty<T>` | Provides reactive, bindable data structures for UI and gameplay |
| `DignusUnitySceneManager` | Coordinates async scene loading and transitions |

---

## Highlights

- **Zero-GC Focus:** Designed to minimize allocations during scene transitions and gameplay  
- **Extensible Architecture:** Works seamlessly with `Dignus.Core` and `Dignus.DependencyInjection`  
- **Runtime Efficiency:** Pooling and binding systems reduce CPU/GPU load spikes  
- **Maintainable Scene Flow:** Structured controllers encourage modular scene logic  
- **Unity-Native Design:** Integrates directly with Unity’s lifecycle without extra dependencies  

---

## Summary

- Lightweight Unity integration layer for the Dignus framework  
- Efficient coroutine, pooling, and DI-based scene control  
- Reactive data binding with `BindableProperty`  
- Optimized for zero-GC, scalable runtime performance  
- Suitable for large Unity projects and service-type architectures  

---