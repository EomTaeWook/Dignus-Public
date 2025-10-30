# Dignus.Sockets

> **Benchmark:** [View Performance Results →](https://github.com/EomTaeWook/ServerPerformanceBenchmark)

**High-performance, event-driven async socket framework focused on throughput, zero-copy I/O, and GC-free execution.**  
Built for scalable real-time servers through modular, allocation-free architecture.

[![Performance Benchmark](https://img.shields.io/badge/Performance-Benchmark-blueviolet?logo=github)](https://github.com/EomTaeWook/ServerPerformanceBenchmark)

---

## Features

| Feature | Description | Key Components |
| :--- | :--- | :--- |
| **Async Session Model** | Event-driven, object-per-session TCP model using `SocketAsyncEventArgs` (SAEA) for high concurrency. | `Session`, `ClientBase`, `ServerBase` |
| **Zero-Copy I/O** | Direct buffer-based read/write path without redundant memory copies. | `SendBuffer`, `ArrayQueue` |
| **Pluggable Protocols** | Replaceable serialization and packet handling logic. | `IPacketSerializer`, `IPacketHandler` |
| **Protocol Pipeline** | Attribute-based mapping of packet IDs to handler methods via compiled delegates, optionally extended with per-protocol middleware. | `ProtocolHandlerMapper`, `ProtocolPipelineInvoker` |
| **Session Extensibility** | Extend session behavior or attach custom components per connection. | `ISessionComponent` |

---

## Architecture

Dignus.Sockets is optimized for **zero-copy** and **zero-GC** networking by separating concerns into three independent layers:

1. **I/O Layer:**  
   Uses `SocketAsyncEventArgs` and reusable `ArrayQueue` buffers for non-blocking, allocation-free I/O.

2. **Protocol Layer:**  
   Handles packet serialization and deserialization through pluggable `IPacketSerializer` and `IPacketHandler` implementations.

3. **Dispatch Layer:**  
   Combines `ProtocolHandlerMapper` and `ProtocolPipelineInvoker` to create a fast, structured, and middleware-enabled packet handling flow.

---

## Core APIs

| API | Purpose |
| :--- | :--- |
| `Session` | Manages socket state, buffering, and lifecycle per connection. |
| `ServerBase` / `ClientBase` | Base abstractions for async accept/connect loops and session management. |
| `ISessionComponent` | Adds extensibility and per-session customization. |
| `ProtocolHandlerMapper<THandler, TBody>` | Binds protocol enums to handler methods via expression trees for near-zero overhead dispatch. |
| `ProtocolPipelineInvoker<TContext, THandler, TBody>` | Builds and executes handler pipelines with optional middleware using a `ref struct` context. |

---

## Combined Usage: Mapper + Pipeline

`ProtocolHandlerMapper` and `ProtocolPipelineInvoker` are designed to work **together**.  
This allows you to maintain **fast, expression-compiled dispatch** while layering **middleware logic** on top.

- `ProtocolHandlerMapper` maps protocol IDs to strongly-typed handler methods.  
- `ProtocolPipelineInvoker` wraps those handlers into configurable middleware pipelines for tasks like authentication, logging, validation, etc.

```csharp
// Register handlers and build middleware per protocol
ProtocolPipelineInvoker<Context, Handler, IMessage>.Use<MyProtocolEnum>((method, pipeline) =>
{
    pipeline.Use<LoggingMiddleware>();
    if (method.Name != "RequestLogin")
        pipeline.Use<AuthCheckMiddleware>();
});

// On receive: run mapped handler through the configured pipeline
ProtocolPipelineInvoker<Context, Handler, IMessage>.Execute(packetId, ref context);
```

This integration provides both **maximum runtime performance** (via precompiled delegates)  
and **clean separation of concerns** (via middleware composition).

---

## Design Principles

- **Zero-Copy I/O:** Data flows directly from socket buffers to handlers without intermediate copying.  
- **Zero-GC Runtime:** SAEA pooling and buffer reuse eliminate runtime allocations.  
- **Expression-Based Dispatch:** Reflection-free mapping via precompiled expression trees.  
- **Middleware Extensibility:** Per-protocol customization through the pipeline system.  
- **Thread-Safe Scalability:** Designed for large-scale concurrent connections.

---

## Summary

- Event-driven TCP framework based on SAEA  
- Zero-copy, zero-GC networking pipeline  
- Expression-compiled protocol dispatch  
- Middleware-based handler extension (`ProtocolHandlerMapper` + `ProtocolPipelineInvoker`)  
- Ideal for scalable, high-performance real-time servers
