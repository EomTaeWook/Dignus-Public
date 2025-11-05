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
Used when a session maintains state across multiple packets.

```csharp
public abstract class PacketHandlerBase : IPacketHandler
{
    public abstract bool TakeReceivedPacket(ArrayQueue<byte> buffer,
        out ArraySegment<byte> packet, out int consumedBytes);

    public abstract Task ProcessPacketAsync(ArraySegment<byte> packet);
}
```

#### Stateless Packet Handler
Used when packets can be processed independently of session state.

```csharp
public abstract class StatelessPacketHandlerBase : IPacketHandler
{
    public abstract bool TakeReceivedPacket(ISession session, ArrayQueue<byte> buffer,
        out ArraySegment<byte> packet, out int consumedBytes);

    public abstract Task ProcessPacketAsync(ISession session, ArraySegment<byte> packet);
}
```

Both implementations internally handle buffering, advancing, and async processing  
via `IPacketHandler.OnReceivedAsync()`.

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

#### Centralized Initialization (`BindAndCreateInvoker`)

For simple and safe pipeline setup, use **`BindAndCreateInvoker`** to automatically **bind handler methods**  
and **create the terminal invoker delegate** in one call.

```csharp
// Session-less Handler
var invoker = ProtocolHandlerMapper<MyHandler, string>
    .BindAndCreateInvoker<MyContext, MyProtocol>();

// Session Handler
var sessionInvoker = ProtocolSessionHandlerMapper<MyHandler, string>
    .BindAndCreateInvoker<MySessionContext, MyProtocol>();
```
---

## Example Flow

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
// 1. Get the pre-bound invoker
var handlerInvoker = ProtocolHandlerMapper<CGProtocolHandler, string>
    .BindAndCreateInvoker<MiddlewareContext, MyProtocol>();

// 2. Configure pipeline with middleware
ProtocolPipelineInvoker<MiddlewareContext, CGProtocolHandler, string>
    .Use<MyProtocol>(
        handlerInvoker,
        (method, pipeline) =>
        {
            // Optional middleware
            pipeline.Use(async (ref MiddlewareContext ctx, ref AsyncPipelineNext<MiddlewareContext> next) =>
            {
                Console.WriteLine($"Protocol {ctx.Protocol} invoked");
                await next.InvokeAsync(ref ctx);
            });
            // Final handler already bound by handlerInvoker
        });
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

### 5. Dispatch Flow Example (Runtime Execution)

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
