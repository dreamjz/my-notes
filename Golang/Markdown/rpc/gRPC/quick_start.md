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
// The greeting service definition
service Greeter {
	rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name
message HelloRequest {
	string name = 1;
}

// The response message containing the greetings
message HelloReply {
	string message = 1;
}
```

Open `hellowordl/helloword.proto` and add a new `SayHelloAgain` method, with the same request and response types:

```protobuf
// The greeting service definition
service Greeter {
	// Sends a greeting
	rpc SayHello (HelloRequest) returns (HelloReply) {}
	// Sends another greeting
	rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}
```

## 5. Regenerate gRPC code

Before you can use the new service method, you need to recompile the updated `.proto` file.

While still in the `examples/helloworld` directory, run the following command :

```sh
$ protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    helloworld/helloworld.proto
```

- `paths=source_relative`: the output file is placed in the same relative directory as the input file. For example, an input file `protos/buzz.proto` results in an output file at `protos/buzz.pb.go`

This will regenerate the `helloworld/helloworld.pb.go` and `helloworld/helloworld_grpc.pb.go` files, which contain:

- Code for populating, serializing, and retrieving `HelloRequest` and `HellReply` message types
- Generated client and server code

## 6. Update and run the application

You have regenerated server and client code, but you still need to implement and call the new method in the human-written parts of the example application.

### 6.1 Update the server

Open `server/main.go` and add the following function to it:

```go
// SayHelloAgain implements helloworld.GreeterServer
func (s *server) SayHelloAgain(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
	return &pb.HelloReply{Message: "Hello again " + in.GetName()}, nil
}
```

### 6.2 Update the client

Open `client/main.go` to add the following code to the end of the `main()` function body:

```go
	r, err = c.SayHelloAgain(ctx, &pb.HelloRequest{Name: *name})
	if err != nil {
		log.Fatalf("could not greet: %v", err)
	}
	log.Printf("Greeting: %s", r.GetMessage())
```

Run the server:

```sh
$ go run server/main.go
```

From another terminal, run the client. 

```sh
$ go run clinet/main.go
2021/12/29 11:38:25 Greeting: Hello world
2021/12/29 11:38:25 Greeting: Hello again world
```

## Reference

1. [quick_start](https://grpc.io/docs/languages/go/quickstart/) gRPC docs
2. [Go Generated Code](https://developers.google.com/protocol-buffers/docs/reference/go-generated) protobuf docs









