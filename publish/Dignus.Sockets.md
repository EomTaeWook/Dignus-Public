# Dignus.Sockets

**High-performance, event-driven async socket framework**  
focused on **throughput**, **zero-copy I/O**, and **GC-free execution**.  
Built for scalable real-time servers with modular, allocation-free design.

> **Benchmark:** [View Performance Results →](https://github.com/EomTaeWook/ServerPerformanceBenchmark)  
[![Performance Benchmark](https://img.shields.io/badge/Performance-Benchmark-blueviolet?logo=github)](https://github.com/EomTaeWook/ServerPerformanceBenchmark)

---

## Features

| Feature | Description | Components |
| :--- | :--- | :--- |
| **Async Session Model** | Event-driven, per-session TCP model using `SocketAsyncEventArgs`. | `Session`, `ServerBase`, `ClientBase` |
| **Zero-Copy I/O** | Direct buffer read/write without redundant memory copies. | `SendBuffer`, `ArrayQueue` |
| **Pluggable Protocols** | Custom serialization and packet handling. | `IPacketSerializer`, `IPacketHandler` |
| **Async Protocol Pipeline** | Attribute-mapped handlers with middleware extensions. | `ProtocolHandlerMapper`, `ProtocolPipelineInvoker` |
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

Now fully **asynchronous**, using `Task`-based processing for incoming packets.

```csharp
public interface IPacketHandler
{
    Task OnReceivedAsync(ISession session, ArrayQueue<byte> buffer);
}
```

The session calls `OnReceivedAsync()` whenever new bytes are received.

---

### 3. Packet Handler Implementations

#### Stateful Packet Handler
Maintains per-session state while processing packets.

```csharp
public abstract class PacketHandlerBase : IPacketHandler
{
    public abstract bool TakeReceivedPacket(ArrayQueue<byte> buffer,
        out ArraySegment<byte> packet, out int consumedBytes);

    public abstract Task ProcessPacketAsync(ArraySegment<byte> packet);

    async Task IPacketHandler.OnReceivedAsync(ISession session, ArrayQueue<byte> buffer)
    {
        while (true)
        {
            if (session.GetSocket() == null)
                return;

            if (TakeReceivedPacket(buffer, out var packet, out var consumed))
            {
                await ProcessPacketAsync(packet);
                buffer.Advance(consumed);
            }
            else break;

            if (buffer.Count == 0)
                break;
        }
    }
}
```

#### Stateless Packet Handler
Does not maintain internal state; each packet is processed independently.

```csharp
public abstract class StatelessPacketHandlerBase : IPacketHandler
{
    public abstract bool TakeReceivedPacket(ISession session, ArrayQueue<byte> buffer,
        out ArraySegment<byte> packet, out int consumedBytes);

    public abstract Task ProcessPacketAsync(ISession session, ArraySegment<byte> packet);

    async Task IPacketHandler.OnReceivedAsync(ISession session, ArrayQueue<byte> buffer)
    {
        while (true)
        {
            if (session.GetSocket() == null)
                return;

            if (TakeReceivedPacket(session, buffer, out var packet, out var consumed))
            {
                await ProcessPacketAsync(session, packet);
                buffer.Advance(consumed);
            }
            else break;

            if (buffer.Count == 0)
                break;
        }
    }
}
```

---

### 4. Dispatch Layer
Now fully **asynchronous**.

`ProtocolHandlerMapper` dynamically binds and caches handler delegates (`Task` or `Action`) for each protocol.  
`ProtocolPipelineInvoker` executes those handlers through an async pipeline.

```csharp
await ProtocolHandlerMapper.InvokeHandlerAsync(handler, protocol, body);
```

#### Context Definition Example
```csharp
public struct MiddlewareContext : IPipelineContext<MyHandler, string>
{
    public int Protocol { get; init; }
    public MyHandler Handler { get; init; }
    public string Body { get; init; }
}
```

#### Configuring Pipelines
```csharp
ProtocolPipelineInvoker<MiddlewareContext, CGProtocolHandler, string>
    .Use<MyProtocol>((method, pipeline) =>
    {
        // Optional custom middleware
        pipeline.Use(async (ref MiddlewareContext ctx, ref AsyncPipelineNext<MiddlewareContext> next) =>
        {
            Console.WriteLine($"Protocol {ctx.Protocol} invoked");
            await next.InvokeAsync(ref ctx);
        });
    });

// Execute pipeline
await ProtocolPipelineInvoker<MyContext, MyHandler, string>.ExecuteAsync(protocol, ref context);
```

---

## Example Flow

### Session Setup
```csharp
_serverModule = new ServerModule(
    new SessionConfiguration(MakeSerializersFuncForClient),
    ServerEndpointInfo.Port);
```

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

## Design Principles

- Async and Zero-GC Networking  
- Clear separation of Send / Receive paths  
- Precompiled or delegate-based protocol dispatch  
- Middleware-based handler extension  
- Modular, allocation-free architecture  

---
