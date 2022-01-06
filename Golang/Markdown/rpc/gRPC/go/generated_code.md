---
title: 'Generated code'
date: '2022-01-05'
categories:
  - golang
  - rpc
tags:
  - grpc
publish: true
---

# Generated-code reference

**Thread-safety**: note that client-side RPC invocations and server-side RPC handlers are *thread-safe* and are meant to be run on concurrent goroutines. But also note that for individual streams, incoming and outgoing data is bi-directional but serial; so e.g. *inidividual* streams do not support concurrent reads or concurrent writes (but reads are safely concurrent with writes).

## 1. Methods on generated server

On the server side, each `service Bar` in the `.proto` file results in the function:

`func RegisterBarService(s *grpc.Server, srv BarServer)`

The application can define a concrete implementation of the `BarServer` interface and register it with a `gprc.Server` instance (before starting the server instance) by using this function.

### 1.1 Unary methods

These methods have the following signature on the generated service interface:

`Foo(context.Context, *MsgA) (*MsgB, error)`

In this context, `MsgA` is the protobuf message sent from the client, and `MsgB` is the protobuf message sent back from the server.

### 1.2 Server-streaming methods

These methods have the following signature on the generated service interface:

`Foo(*MsgA, <ServiceName>_FooServer) error`

In this context, `MsgA` is the single request from the client, and the `<ServiceName>_FooServer` parameter represents the server-to-client stream of `MsgB` messages.

`<ServiceName>_FooServer` has an embedded `grpc.ServerStream` and the following interface:

```go
type <ServiceName>_FooServer interface {
    Send(*MsgB) error
    grpc.ServerStream
}
```

The server-side handler can send a stream of protobuf messages to the client through this parameter’s `Send` method. End-of-stream for the server-to-client stream is caused by the `return` of the handler method.

### 1.3 Client-streaming methods

These methods have the following signature on the generated service interface:

`Foo(<ServiceName_FooServer) error` 

In this context, `<SerivceName>_FooServer` can be used both to read the client-to-server message stream and to send the single server response message.

`<ServiceName>_FooServer` has an embedded `gprc.ServerStream` and the following interface:

```go
type <ServiceName>_FooServer interface {
    SendAndClose(*MsgA) error
    Recv() (*MsgB, error)
    grpc.ServerStream
}
```

The server-side handler can repeatedly call `Recv` on this parameter in order to receive the full stream of messages from the client. `Recv` returns `(nil, io.EOF)` once it has reached the end of the stream. The single response message from the server is sent by calling the `SendAndClose` method on this `<ServiceName>_FooServer` parameter. Note that `SendAndClose` must be called once and only once.

### 1.4 Bidi-streaming methods

These methods have the following signature on the generated service interface:

`Foo(<ServiceName>_FooServer) error`

In this context, `<ServiceName>_FooServer` can be used to access both the client-to-server message stream and the server-to-client message stream. `<ServiceName>_FooServer` has an embedded `gprc.ServerStream` and the following interface:

```go
type <ServiceName>_FooServer interface {
    Send(*MsgA) error
    Recv() (*MsgB, error)
    grpc.ServerStream
}
```

The server-side handler can repeatedly call `Recv` on this parameter in order to read the client-to-server message stream. `Recv` returns `(nil, io.EOF)` once it has reached the end of the client-to-server stream. The response server-to-client message stream is sent by repeatedly calling the `Send` method of on this `<ServiceName>_FooServer` parameter. End-of-stream for the server-to-client stream is indicated by the `return` of the bidi method handler.

# 2. Methods on generated client interfaces

For client side usage, each `service Bar` in the `.proto` file also results in the function: `func BarClient(cc *grpc.ClientConn) BarClient`, which returns a concrete implementation  of the `BarClient` interface (this concrete implementation also lives in the generated `.pb.go` file).

### 2.1 Unary Methods

These methods have the following signature on the generated client stub:

`Foo(ctx context.Context, in *MsgA, opts ...grpc.CallOption) (*MsgB, error)`

In this context, `MsgA` is the single request from client to server, and `MsgB` contains the response sent back from the server.

### 2.2 Server-Streaming methods

These methods have the following signature on the generated client stub:

`Foo(ctx context.Context, in *MsgA, opts ...grpc.CallOption) (<ServiceName>_FooClient, error)`

In this context, `<ServiceName>_FooClient` represents the server-to-client `stream` of `MsgB` messages.

This stream has an embedded `grpc.ClientStream` and the following interface:

```go
type <ServiceName>_FooClient interface {
    Recv() (*MsgB, error)
    gprc.ClientStream
}
```

The stream begins when the client calls the `Foo` method on the stub. The client can then repeatedly call the `Recv` method on the returned `<ServiceName>_FooClient` stream in order to read the server-to-client response stream. This `Recv` method returns `(nil, io.EOF)` once the server-to-client stream has been completely read through.

### 2.3 Client-Streaming methods

These methods have the following signature on the generated client stub:

`Foo(ctx context.Context,opts ...gprc.CallOption) (<ServiceName>_FooClient, error)`

In this context, `<ServiceName>_FooClient` represents the client-to-server `stream` of `MsgA` messages.

`<ServiceName>_FooClient` has an embedded `grpc.ClientStream` and the following interface:

```go
type <ServiceName>_FooClient interface {
    Send(*MsgA) error
    CloseAndRecv() (*MsgB, error)
    grpc.ClientStream
}
```

The stream begins when the client calls the `Foo` method on the stub. The client can then repeatedly call the `Send` method on the returned `<ServiceName>_FooClient` stream in order to send the client-to-server message stream. The `CloseAndRecv` method on this stream must be called once and only once, in order to both close the client-to-server stream and receive the single response message from the server.

### 2.4 Bidi-Streaming methods

These methods have the following signature on the generated client stub:

`Foo(ctx context.Context, opts ...grpc.Option) (<ServiceName>_FooClient, error)`

In this context, `<ServiceName>_FooClient` represents both the client-to-server and server-to-client message streams.

`<ServiceName>_FooClient` has an embedded `grpc.ClientStream` and the following interface:

```go
type <ServiceName>_FooClient interface {
    Send(*MsgA) error
    Recv() (*MsgB, error)
    grpc.ClientStream
}
```

The stream begins when the client calls the `Foo` method on the stub. The client can then repeatedly call the `Send` method on the returned `<ServiceName>_FooClient` stream in order to send the client-to-server message stream. The client can also repeatedly call `Recv` on this stream in order to receive the full server-to-client message stream.

End-of-Stream for the server-to-client stream is indicated by a return value of `(nil, io.EOF)` on the `Recv` method of the stream. End-of-stream for the client-to-server stream can be indicated from the client by calling the `CloseSend` method on the stream.

## Reference

1. [generated-code](https://grpc.io/docs/languages/go/generated-code/) gRPC docs