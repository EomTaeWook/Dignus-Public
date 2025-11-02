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
| **Protocol Pipeline** | Attribute-mapped handlers with middleware extension. | `ProtocolHandlerMapper`, `ProtocolPipelineInvoker` |
| **Session Extensibility** | Per-session components for custom logic. | `ISessionComponent` |

---

## Architecture

### 1. I/O Layer
Non-blocking I/O using `SocketAsyncEventArgs` and pooled buffers  
for **allocation-free** and **thread-safe** network operations.

---

### 2. Protocol Layer
Handles **packet framing, serialization, and dispatching** between the socket and user-defined logic.  
This layer separates **Send** and **Receive** clearly for performance and flexibility.

#### Receive Path
- Implemented via `IPacketHandler`
- Parses buffered bytes and invokes registered handlers  
- Supports both **stateful** and **stateless** models

| Type | Description | Base Class |
| :--- | :--- | :--- |
| **Stateful Handler** | Keeps per-session state during packet processing. | `PacketHandlerBase` |
| **Stateless Handler** | Uses session context per packet, reentrant. | `StatelessPacketHandlerBase` |

```csharp
_packetHandler.OnReceived(this, _receivedBuffer);
```

#### Send Path
- Implemented via `IPacketSerializer`
- Converts a structured `IPacket` into a sendable byte buffer

```csharp
public interface IPacketSerializer
{
    ArraySegment<byte> MakeSendBuffer(IPacket packet);
}
```

### 3. Dispatch Layer
Combines **expression-based handler mapping** with **middleware pipelines**  
for high-speed and extensible protocol processing.

```csharp
ProtocolPipelineInvoker<Ctx, Handler, string>.Use<MyProtocol>((m, p) =>
{
    p.Use<LoggingMiddleware>();
});

ProtocolPipelineInvoker<Ctx, Handler, string>.Execute(packetId, ref context);
```

## Example Flow

### Session Setup
```csharp
_serverModule = new ServerModule(
    new SessionConfiguration(MakeSerializersFuncForClient),
    ServerEndpointInfo.Port);
```
```csharp
ProtocolPipelineInvoker<MiddlewareContext, CGProtocolHandler, string>
    .Use<CGSProtocol>((method, pipeline) =>
    {
        var filters = method.GetCustomAttributes<ActionAttribute>();
        var orderedFilters = filters.OrderBy(r => r.Order).ToList();
        var filtersMiddleware = new ProtocolActionMiddleware(orderedFilters);
        pipeline.Use(filtersMiddleware);
    });
```

### Packet Handling
```csharp
public abstract class PacketHandlerBase : IPacketHandler
{
    public abstract bool TakeReceivedPacket(ArrayQueue<byte> buffer,
        out ArraySegment<byte> packet, out int consumedBytes);

    public abstract void ProcessPacket(in ArraySegment<byte> packet);
}
```
Example derived implementation:
```csharp
public override void ProcessPacket(in ArraySegment<byte> packet)
{
    int protocol = BitConverter.ToInt16(packet);
    var body = Encoding.UTF8.GetString(packet.Array, packet.Offset + 2, packet.Count - 2);

    var context = new MiddlewareContext()
    {
        Body = body,
        Handler = _protocolHandler,
        Protocol = protocol,
    };

    ProtocolPipelineInvoker<MiddlewareContext, CGProtocolHandler, string>
        .Execute(protocol, ref context);
}
```

### Handler
```csharp
[ProtocolName("JoinGameRoom")]
public void Process(JoinGameRoom req) { ... }
```

## Design Principles

- Zero-Copy, Zero-GC Networking
- Clear separation of Send / Receive paths
- Precompiled protocol dispatch or Optimized delegate-based dispatch
- Middleware-based handler extension
- Modular and scalable architecture

---

**Dignus.Sockets** — lightweight, allocation-free, and built for real-time performance.

