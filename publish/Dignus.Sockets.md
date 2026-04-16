````md
# Dignus.Sockets

**Dignus.Sockets** is a high-performance C# TCP/TLS/UDP server framework.  
It provides async socket processing, custom packet serialization, protocol-based dispatch, and middleware-capable pipelines.

> Benchmark: https://github.com/EomTaeWook/ServerPerformanceBenchmark

---

## Features

- Async TCP/TLS/UDP networking
- Custom packet framing and serialization
- Protocol-based handler dispatch
- Direct mapper-based execution
- Binder-based pipeline system
- Middleware composition support
- Session component model

---

## Quick Start

A server built with `Dignus.Sockets` typically consists of:

1. Packet format
2. Packet definition
3. Dispatch layer
4. Direct mapper or pipeline
5. Protocol handler
6. Packet processor
7. Session setup
8. Server start

---

## 1. Packet Format

This example uses:

```text
[length:int][protocol:int][body:byte[]]
```

### Fields

- length  
  size of `[protocol + body]`

- protocol  
  message identifier

- body  
  payload (JSON in this example)

### Example

```text
[12][1001]["hello"]
```

### Notes

- This format is only an example.
- Protocol type can be changed.
- Body format can be changed.
- Length-based framing must be preserved.

---

## 2. Packet

```csharp
using Dignus.Sockets.Interfaces;
using System.Text;

internal class Packet : IPacket
{
    public int Protocol { get; }
    public byte[] Body { get; }

    public Packet(int protocol, byte[] body)
    {
        Protocol = protocol;
        Body = body;
    }

    public Packet(int protocol, string value)
    {
        Protocol = protocol;
        Body = Encoding.UTF8.GetBytes(value);
    }

    public int GetLength()
    {
        return Body.Length + sizeof(int);
    }
}
```

---

## 3. Dispatch Layer

The dispatch layer maps protocol values to handler methods.

### Binding

```csharp
ProtocolHandlerMapper<EchoHandler, string>.BindProtocol<CSProtocol>();
```

This step:

- scans handler methods
- reads mapping attributes
- builds protocol-to-method bindings

### Invocation

```csharp
await ProtocolHandlerMapper<EchoHandler, string>.InvokeHandlerAsync(
    handler,
    protocol,
    body);
```

This step:

- finds the mapped handler method
- executes it asynchronously

### Available Mapper APIs

```csharp
await ProtocolHandlerMapper<MyHandler, string>.InvokeHandlerAsync(handler, protocol, body);

await ProtocolSessionHandlerMapper<MyHandler, string>.InvokeHandlerAsync(handler, protocol, session, body);
```

---

## 4. Direct Mapper Usage

If you do not need a binder or middleware pipeline, you can bind handlers directly with a mapper.

```csharp
ProtocolHandlerMapper<MyHandler, string>.BindProtocol<MyProtocol>();
await ProtocolHandlerMapper<MyHandler, string>.InvokeHandlerAsync(handler, protocol, body);
```

### When to use

- simple servers
- rapid testing
- no middleware
- direct control over binding and invocation

---

## 5. Pipeline

`Dignus.Sockets` provides a binder-based pipeline system for protocol execution.

It supports:

- automatic protocol binding
- middleware composition
- async execution pipeline

### Pipeline Builder (Recommended)

Use the binder-based builder for automatic protocol registration and middleware composition.

```csharp
var binder = new ProtocolHandlerBinder<MiddlewareContext, CGProtocolHandler, string>();

// var binder = new ProtocolHandlerBinder<MiddlewareContext, CGProtocolHandler, string, ISession>();

ProtocolPipelineInvoker<MiddlewareContext>
    .Bind<CGProtocolHandler, string, MyProtocol>(binder)
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

### Pipeline Execution

#### Recommended

```csharp
await ProtocolPipelineInvoker<MyContext>.ExecuteAsync(ref context);
```

#### Legacy

```csharp
await ProtocolPipelineInvoker<MyContext, MyHandler, string>.ExecuteAsync(ref context);
```

### Pipeline Steps

- `Bind<TProtocol>()`  
  selects the protocol type and binder

- `Use()`  
  registers middleware such as logging, validation, or filtering

- `Build()`  
  finalizes the pipeline configuration

- `ExecuteAsync()`  
  runs the pipeline for the given context

### When to use

- middleware is required
- logging, validation, or filtering should be centralized
- protocol execution needs pre/post processing
- handler invocation should be composed declaratively

### Notes

- `ProtocolPipelineInvoker<MyContext>` is the recommended API for new code.
- `ProtocolPipelineInvoker<MyContext, MyHandler, string>` is the legacy form.
- Both execute the configured async pipeline.

---

## 6. Handler

```csharp
using Dignus.Sockets.Attributes;
using Dignus.Sockets.Interfaces;
using System.Text.Json;

internal class EchoHandler : IProtocolHandler<string>, ISessionComponent
{
    private ISession _session;

    public T DeserializeBody<T>(string body)
    {
        return JsonSerializer.Deserialize<T>(body);
    }

    [ProtocolName("EchoMessage")]
    public void Process(EchoMessage echo)
    {
        var body = JsonSerializer.Serialize(echo);
        _session.SendAsync(new Packet((int)SCProtocol.EchoMessageResponse, body));
    }

    public void OtherMessage(OtherMessage otherMessage)
    {
        var body = JsonSerializer.Serialize(otherMessage);
        _session.SendAsync(new Packet((int)SCProtocol.OtherMessageResponse, body));
    }

    public void SetSession(ISession session)
    {
        _session = session;
    }

    public void Dispose()
    {
        _session = null;
    }
}
```

### Key Points

- `BindProtocol()` reads handler metadata during initialization.
- Protocol values are automatically mapped to handler methods.
- `InvokeHandlerAsync()` executes the mapped method.
- `ISessionComponent` allows the handler to receive the current session.

---

## 7. Packet Processor

```csharp
using Dignus.Collections;
using Dignus.Sockets.Interfaces;
using Dignus.Sockets.Processing;
using System.Text;

internal class PacketProcessor(EchoHandler echoHandler)
    : SessionlessPacketProcessor, IPacketSerializer
{
    private const int HeaderSize = sizeof(int) * 2;
    private const int SizeToInt = sizeof(int);

    public override async Task ProcessPacketAsync(ArraySegment<byte> packet)
    {
        var protocol = BitConverter.ToInt32(packet.Array, packet.Offset);
        var size = packet.Count - SizeToInt;
        var bodyString = Encoding.UTF8.GetString(packet.Array, packet.Offset + SizeToInt, size);

        await ProtocolHandlerMapper<EchoHandler, string>.InvokeHandlerAsync(
            echoHandler,
            protocol,
            bodyString);
    }

    public ArraySegment<byte> MakeSendBuffer(IPacket packet)
    {
        var sendPacket = (Packet)packet;

        var packetSize = sendPacket.GetLength();
        byte[] buffer = new byte[packetSize + SizeToInt];

        Buffer.BlockCopy(BitConverter.GetBytes(packetSize), 0, buffer, 0, SizeToInt);
        Buffer.BlockCopy(BitConverter.GetBytes(sendPacket.Protocol), 0, buffer, SizeToInt, SizeToInt);
        Buffer.BlockCopy(sendPacket.Body, 0, buffer, HeaderSize, sendPacket.Body.Length);

        return buffer;
    }

    public override bool TakeReceivedPacket(
        ArrayQueue<byte> buffer,
        out ArraySegment<byte> packet,
        out int consumedBytes)
    {
        packet = null;
        consumedBytes = 0;

        if (buffer.Count < SizeToInt)
            return false;

        var length = BitConverter.ToInt32(buffer.Peek(SizeToInt));

        if (buffer.Count < length + SizeToInt)
            return false;

        if (buffer.TryReadBytes(out _, SizeToInt) == false)
            return false;

        if (buffer.TrySlice(out packet, length) == false)
            return false;

        consumedBytes = length;
        return true;
    }
}
```

---

## 8. Session Setup

```csharp
static SessionSetup PacketHandlerSetupFactory()
{
    EchoHandler handler = new();
    PacketProcessor processor = new(handler);

    return new SessionSetup(
        processor,
        processor,
        [handler]);
}
```

### Structure

- first argument  
  receive-side processor

- second argument  
  send-side serializer

- third argument  
  session components

---

## 9. Server

```csharp
using Dignus.Sockets;
using Dignus.Sockets.Tcp;

internal class EchoServer : TcpServerBase
{
    public EchoServer(SessionConfiguration config) : base(config)
    {
        ProtocolHandlerMapper<EchoHandler, string>.BindProtocol<CSProtocol>();
    }
}
```

---

## 10. Start

```csharp
var config = new SessionConfiguration(PacketHandlerSetupFactory);
var server = new EchoServer(config);

server.Start(5000);
```

---

## Runtime Flow

1. receive bytes
2. split one packet
3. parse protocol and body
4. dispatch to the mapped handler
5. serialize the response
6. send the response

---

## Choosing an Execution Style

### Direct Mapper

Use this when:

- the server is simple
- middleware is unnecessary
- direct binding is preferred

### Pipeline

Use this when:

- middleware is needed
- cross-cutting concerns should be centralized
- protocol execution should be composed through a fluent builder

---

## Summary

- Packet  
  defines protocol and body

- Packet Processor  
  handles framing and parsing

- Mapper  
  dispatches protocols directly

- Pipeline  
  composes protocol execution with middleware

- Handler  
  contains business logic

- Session  
  sends and receives data

---