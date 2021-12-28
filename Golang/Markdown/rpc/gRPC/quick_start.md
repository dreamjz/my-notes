---
title: 'Quick Start'
date: '2021-12-28'
categories:
  - golang
  - rpc
tags:
  - grpc
publish: true
---

# Quick Start

This guide gets you started with gRPC in Go with a simple working example.

## 1. Prerequisites

- Go

- Protocol buffer compiler

- Go plugins for the protocol compiler:

  1. Install the protocol compiler plugins for Go using the following commands:

     ```sh
     $ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.26
     $ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.1
     ```
     
  2. Update your `PATH` so that the `protoc` compiler can find the plugins:
  
     ```sh
     $ export PATH="PATH:$(go env GOPATH)/bin"
     ```

## 2. Get the example code

The example code is part of the [grpc-go](https://github.com/grpc/grpc-go) repo.

1. Download the repo as zip file and unzip it, or clone the repo:

   ```sh
   $ git clone -b v1.41.0 https://github.com/grpc/grpc-go
   ```

2. Change to the quick start example directory:

   ```sh
   $ cd grpc-go/examples/helloworld
   ```

## 3. Run the example 

From the `examples/hellowrold` directory:

1. Compile and execute the server code:

   ```sh
   $ go run greeter_server/main.go
   ```

2. From the different terminal, compile and execute the client code to see the client output:

   ```sh
   $ go run greeter_client/main.go
   Greeting: Hello world
   ```

## 4. Update the gRPC service

In this section you’ll update the application with an extra server method. The gRPC service is defined using protocol buffers. The server and client stub have a `SayHello()` RPC method that takes a `HelloRequest` parameter from the client and returns a `HelloReply` from the server, and that the method is defined like this:

```protobuf
```



