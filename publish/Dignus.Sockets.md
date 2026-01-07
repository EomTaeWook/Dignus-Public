# Dignus.Sockets
**Dignus.Sockets** is a high-performance **C# TCP server framework** built on an event-driven async socket architecture.  
Designed for scalable TCP networking, real-time systems, and high-throughput packet processing with zero-copy and GC-free execution.

Built for scalable real-time servers with modular, allocation-free design.

> **Benchmark:** [View Performance Results →](https://github.com/EomTaeWook/ServerPerformanceBenchmark)  
[![Performance Benchmark](https://img.shields.io/badge/Performance-Benchmark-blueviolet?logo=github)](https://github.com/EomTaeWook/ServerPerformanceBenchmark)

---

## Features

| Feature | Description | Components |
| :--- | :--- | :--- |
| **Async Session Model** | Event-driven, per-session TCP model using `SocketAsyncEventArgs`. | `Session`, `ServerBase`, `ClientBase` |
| **Zero-Copy I/O** | Direct buffer read/write without redundant memory copies. | `SendBuffer`, `ArrayQueue` |
| **Pluggable Protocols** | Custom serialization and packet handling. | `IPacketSerializer`, `PacketHandlerBase`, `PacketProcessor` |
| **Async Protocol Pipeline** | Attribute-mapped handlers with middleware extensions. | `ProtocolHandlerMapper`, `ProtocolSessionHandlerMapper`, `ProtocolPipelineInvoker` |
| **Session Extensibility** | Custom components per session for modular logic. | `ISessionComponent` |

---

## Architecture

### 1. I/O Layer
Non-blocking I/O using `SocketAsyncEventArgs` and pooled buffers  
for **allocation-free** and **thread-safe** network operations.

---

### 2. Protocol Layer
Handles **packet framing, serialization, and dispatching** between the socket and user-defined logic.  
The layer provides clear separation between **Send** and **Receive** for performance and flexibility.

#### Send Path
- Implemented via `IPacketSerializer`
- Converts a structured `IPacket` into a sendable byte buffer
- Used by `Session.Send()` to enqueue data into the `SendBuffer`
- The send path remains **zero-copy** and lock-protected for thread safety

```csharp
public interface IPacketSerializer
{
    ArraySegment<byte> MakeSendBuffer(IPacket packet);
}
```

#### Receive Path

Incoming bytes are processed by the internal receive loop
implemented in `PacketProcessor`.

The framework manages:
- receive loop execution
- buffer advancing
- session lifetime safety
- async execution boundaries

Users define packet handling behavior
by implementing either `PacketProcessor` or `PacketHandlerBase`.

### 3. Packet Handling

### PacketProcessor

Provides full access to `ISession` during:
- packet framing
- packet processing

Use this when packet handling logic
requires direct interaction with the session.

```csharp
public abstract class PacketProcessor
{
    protected abstract bool TakeReceivedPacket(ISession session, ArrayQueue<byte> buffer,
        out ArraySegment<byte> packet, out int consumedBytes);

    protected abstract Task ProcessPacketAsync(ISession session, ArraySegment<byte> packet);
}
```

### PacketHandlerBase

A packet-centric abstraction built on top of `PacketProcessor`.

- Does not expose `ISession` in method signatures
- Suitable when session access is not required directly

```csharp
public abstract class PacketHandlerBase : PacketProcessor
{
    public abstract bool TakeReceivedPacket(ArrayQueue<byte> buffer,
        out ArraySegment<byte> packet, out int consumedBytes);

    public abstract Task ProcessPacketAsync(ArraySegment<byte> packet);
}
```

### Difference

| Type | Session Access | Responsibility |
|----|---------------|----------------|
| PacketProcessor | Yes | Session-aware packet framing and processing |
| PacketHandlerBase | No | Packet-centric processing without session dependency |

---

### 4. Dispatch Layer
Now fully **asynchronous**.

`ProtocolHandlerMapper` dynamically binds and caches handler delegates (`Task` or `Action`) for each protocol.
`ProtocolSessionHandlerMapper` dynamically binds and caches handler delegates (`Task` or `Action`) for each protocol.
`ProtocolPipelineInvoker` executes those handlers through an async pipeline.

```csharp
await ProtocolHandlerMapper.InvokeHandlerAsync(handler, protocol, body);
```
```csharp
await ProtocolSessionHandlerMapper.InvokeHandlerAsync(handler, protocol, session, body);
```
```csharp
await ProtocolPipelineInvoker<MyContext, MyHandler, string>.ExecuteAsync(protocol, ref context);
```

---

### 5. Direct Mapper Usage
If you prefer not to use a binder or pipeline, you can bind handlers manually using mappers.

```csharp
// Session-less handler example
ProtocolHandlerMapper<MyHandler, string>.BindProtocol<MyProtocol>();
await ProtocolHandlerMapper<MyHandler, string>.InvokeHandlerAsync(handler, protocol, body);

// Session-based handler example
ProtocolSessionHandlerMapper<MyHandler, string>.BindProtocol<MyProtocol>();
await ProtocolSessionHandlerMapper<MyHandler, string>.InvokeHandlerAsync(handler, protocol, session, body);
```

**When to use:**  
- Rapid testing or simple systems  
- No middleware required  
- You want direct control over binding and invocation

---

### 6. Binder Interfaces and Implementations

For large-scale or modular servers, binders automate handler binding and invoker creation.

```csharp
// IProtocolInvokerBinder.cs
public interface IProtocolInvokerBinder<TContext> where TContext : struct
{
    HandlerInvokerDelegate<TContext> BindAndCreateInvoker<TProtocol>()
        where TProtocol : struct, Enum;
}
```

```csharp
// ProtocolHandlerBinder.cs
public class ProtocolHandlerBinder<TContext, THandler, TBody> : IProtocolInvokerBinder<TContext>
    where TContext : struct, IPipelineContext<THandler, TBody>
    where THandler : IProtocolHandler<TBody>
{
    public HandlerInvokerDelegate<TContext> BindAndCreateInvoker<TProtocol>()
        where TProtocol : struct, Enum
    {
        ProtocolHandlerMapper<THandler, TBody>.BindProtocol<TProtocol>();

        return (ref TContext context) =>
        {
            return ProtocolHandlerMapper<THandler, TBody>.InvokeHandlerAsync(
                context.Handler,
                context.Protocol,
                context.Body);
        };
    }
}
```

```csharp
// ProtocolSessionHandlerBinder.cs
public class ProtocolSessionHandlerBinder<THandler, TBody, TContext> 
    : IProtocolInvokerBinder<TContext>
    where TContext : struct, IPipelineSessionContext<THandler, TBody>
    where THandler : IProtocolHandler<TBody>
{
    public HandlerInvokerDelegate<TContext> BindAndCreateInvoker<TProtocol>()
        where TProtocol : struct, Enum
    {
        ProtocolSessionHandlerMapper<THandler, TBody>.BindProtocol<TProtocol>();

        return (ref TContext context) =>
        {
            return ProtocolSessionHandlerMapper<THandler, TBody>.InvokeHandlerAsync(
                context.Handler,
                context.Protocol,
                context.Session,
                context.Body);
        };
    }
}
```

---

### 7. Configuring Pipelines (Recommended)

Use the **binder-based pipeline builder** for automatic protocol registration  
and middleware composition in one fluent chain.

```csharp
ProtocolPipelineInvoker<MiddlewareContext, CGProtocolHandler, string>
    .Bind<MyProtocol>(new ProtocolHandlerBinder<MiddlewareContext, CGProtocolHandler, string>())
    .Use((method, pipeline) =>
    {
        // Optional middleware
        pipeline.Use((ref MiddlewareContext context, ref AsyncPipelineNext<MiddlewareContext> next) =>
        {
            Console.WriteLine($"Protocol {context.Protocol} invoked");
            return next.InvokeAsync(ref context);
        });
    })
    .Build();
```

**Pipeline Steps**
- `Bind<TProtocol>()` → Selects protocol type and binder (session or non-session).  
- `Use()` → Registers optional middleware (logging, filters, validation).  
- `Build()` → Finalizes and stores the async pipeline for runtime execution.

---

## 8.Example Flow

### Session Setup
```csharp
_serverModule = new ServerModule(
    new SessionConfiguration(MakeSerializersFuncForClient),
    ServerEndpointInfo.Port);
```

#### Context Definition Example
```csharp
// 1. Session-less Context
public struct MiddlewareContext : IPipelineContext<MyHandler, string>
{
    public int Protocol { get; init; }
    public MyHandler Handler { get; init; }
    public string Body { get; init; }
}

// 2. Session Context
public struct SessionContext : IPipelineSessionContext<MyHandler, string>
{
    public int Protocol { get; init; }
    public MyHandler Handler { get; init; }
    public string Body { get; init; }
    public ISession Session { get; init; }
}
```

#### Configuring Pipelines (Recommended)
```csharp
var binder = new ProtocolHandlerBinder<MiddlewareContext, CGProtocolHandler, string>();
//var sessionBinder = new ProtocolSessionHandlerBinder<SessionContext, CGProtocolHandler, string>();

ProtocolPipelineInvoker<MiddlewareContext, CGProtocolHandler, string>
    .Bind<MyProtocol>(binder)
    .Use((method, pipeline) =>
    {
        pipeline.Use((ref MiddlewareContext context, ref AsyncPipelineNext<MiddlewareContext> next) =>
        {
            Console.WriteLine($"Protocol {context.Protocol} invoked");
            return next.InvokeAsync(ref context);
        });
    })
    .Build();
```

---


### Handler Example
```csharp
[Auth]
[ProtocolName("JoinGameRoom")]
public void Process(JoinGameRoom req) 
{
    // Custom logic here
}
```

---

### 9. Dispatch Flow Example (Runtime Execution)

The packet handler connects the low-level socket receive loop  
with the high-level protocol execution pipeline.

```csharp
using Dignus.Collections;
using Dignus.Log;
using Dignus.Sockets;
using Dignus.Sockets.Interfaces;
using Dignus.Sockets.Processing;
using System.Text;

internal class PacketHandler : StatelessPacketHandlerBase
{
    private const int HeaderSize = sizeof(int);
    private const int ProtocolSize = sizeof(ushort);

    private readonly CSProtocolHandler _protocolHandler;
    public PacketHandler(CSProtocolHandler csProtocolHandler)
    {
        _protocolHandler = csProtocolHandler;
    }

    public override bool TakeReceivedPacket(ISession session, ArrayQueue<byte> buffer,
        out ArraySegment<byte> packet, out int consumedBytes)
    {
        packet = null;
        consumedBytes = 0;
        if (buffer.Count < HeaderSize)
            return false;

        var bodySize = BitConverter.ToInt32(buffer.Peek(HeaderSize));
        if (buffer.Count < HeaderSize + bodySize)
            return false;

        buffer.TryReadBytes(out _, HeaderSize);
        consumedBytes = bodySize;
        return buffer.TrySlice(out packet, bodySize);
    }

    public override async Task ProcessPacketAsync(ISession session, ArraySegment<byte> packet)
    {
        var protocol = BitConverter.ToInt16(packet);

        if (ProtocolHandlerMapper.ValidateProtocol<CSProtocolHandler>(protocol) == false)
        {
            LogHelper.Error($"[Server] Invalid protocol: {protocol}");
            return;
        }

        var body = Encoding.UTF8.GetString(
            packet.Array, packet.Offset + ProtocolSize, packet.Count - ProtocolSize);

        var context = new PipeContext()
        {
            Handler = _protocolHandler,
            Session = session,
            Protocol = protocol,
            Body = body
        };

        await ProtocolPipelineInvoker<PipeContext, CSProtocolHandler, string>
            .ExecuteAsync(protocol, ref context);
    }
}
```

---

## Design Principles

- Async and Zero-GC Networking  
- Clear separation of Send / Receive paths  
- Precompiled or delegate-based protocol dispatch  
- Middleware-based handler extension  
- Modular, allocation-free architecture  

---
