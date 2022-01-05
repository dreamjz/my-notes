---
title: 'Basics tutorial'
date: '2021-12-30'
categories:
  - golang
  - rpc
tags:
  - grpc
publish: true
---

# Basics tutorial

## 1. Why use gRPC 

The example is a simple route mapping application that lets clients get information about features on their route, create a summary of their route, and exchange route information such as traffic updates with the server and other clients.

With gRPC we can define our service onec in a `.proto` file and generate clients and servers in any of gRPC’s supported languages, which in turn can be run in environments ranging from servers inside a large data center to your own tablet -- all the complexity of communication between different languages and environments is handled for you by gRPC. We also get all the advantages of working with protocol buffers, including efficient serialization, a simple IDL, and easy interface upadting.

## 2. Defining the service

To define a service, you specify a named `service` in your `.proto` file:

```protobuf
service RouteGuide {
	...
}
```

Then you define `rpc` methods inside your service definition, specifying their request and response types. gRPC lets you define four kinds of service method, all of which are used in the `RouteGuide` service :

- A simple RPC where the client sends a request to the server using the stub and waits for a response to come back, just like a normal function call.

  ```protobuf
  // Obtains the feature at a given position.
  rpc GetFeature(Point) returns (Feature) {}
  ```

- A server-side streaming RPC where the client sends a request to the server and gets a stream to read a sequence of messages back. The client reads from the returned stream until there are no more messages. As you can see in the example, you specify a server-side streaming method by placing the `stream` keyword brefore the *response* type.

  ```protobuf
  // Obtains the Features available within the given Rectangle.
  // Results are streamed rather than returned at once (e.g.
  // in a response message with a repeated field), as the rectangle
  // may cover a large area and contain a huge number of features.
  rpc ListFeatures(Rectangle) returns (stream Feature) {}
  ```

- A client-side streaming RPC where the client writes a sequence of messages and sends them to the server, again using a provided stream. Once the client has finished writing the message, it waits for the server to read them all and return its response. You specify a client-side streaming method by placing the `stream` keyword before the *request* type.

  ```protobuf
  // Accepts a stream of Points on a route being traversed, returning a 
  // RouteSummary when traversal is completed.
  rpc RecordRoute(stream Point) returns (RouteSummary) {}
  ```

- A bidirectional streaming RPC where both side send a sequence of messages using a read-write stream. The two streams operate independently, so clients and servers can read and write in whatever order they like: for example, the server could wait to receive all the client messages before writing its responses, or it could alternately read a message then write a message, or some other combination of reads and writes. The order of messages in each stream is preserved. You specify this type of method by placing the `stream` keyword before both the request and the response.

  ```protobuf
  // Accepts a stream of RouteNotes sent while a route is being traversed,
  // while receiving other RouteNotes (e.g. from other users)
  rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
  ```
  

The `.proto` file also contains protocol buffer message type definitions for all the request and response types used in our service methods - for example, here’s the `Point` message type:

```protobuf
// Points are represented as latitude-longitude pairs in the E7 representation
// (degrees multiplied by 10**7 and rounded to the nearest integer).
// Latitudes should be in the range +/- 90 degrees and longitude should be in
// the range +/- 180 degrees (inclusive).
message Point {
  int32 latitude = 1;
  int32 longitude = 2;
}
```

## 3. Generating client and server code

From the `examples/route_guide` directory, run the following command:

```sh
$ protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    routeguide/route_guide.proto
```

Running this command generates the following files in the `routeguide` directory:

- `route_guide.pb.go`: which contains all the protocol buffer code to populate, serialize, and retrieve request and response message types.
- `route_guide_grpc.pb.go`: which contains the following:
  - An interface type (or stub) for clients to call with the methods defined in the `RouteGuide` service.
  - An interface type for servers to implement, also with the methods defined in the `RouteGuide` service.

## 4. Creating the server

There are two parts to making our `RouteGuide` service do its job:

- Implementing the service interface generated from our service definition: doing the actual “work” of our service.
- Running a gRPC server to listen for request from clients and dispatch them to the right service implementation.

### 4.1 Implementing RouteGuide

As you can see, our server has a `routeGuideServer`  struct type that implements the generated `RouteGuideServer` interface :

```go
type routeGuideServer struct {
        ...
}
...

func (s *routeGuideServer) GetFeature(ctx context.Context, point *pb.Point) (*pb.Feature, error) {
        ...
}
...

func (s *routeGuideServer) ListFeatures(rect *pb.Rectangle, stream pb.RouteGuide_ListFeaturesServer) error {
        ...
}
...

func (s *routeGuideServer) RecordRoute(stream pb.RouteGuide_RecordRouteServer) error {
        ...
}
...

func (s *routeGuideServer) RouteChat(stream pb.RouteGuide_RouteChatServer) error {
        ...
}
...
```

### 4.2 Simple RPC

The `routeGuideServer` implements all our service methods. Let’s look at the simplest type first, `GetFeature`, which just gets a `Point` from the client and returns the corresponding feature information from its database in a `Feature`.

```go
func (s *routeGuideServer) GetFeature(ctx context.Context, point *pb.Point) (*pb.Feature, error) {
    for _, feature := range s.savedFeatures {
        if proto.Equal(feature.Location, point) {
            return feature, nil
        }
    }
    // No feature was found, return an unnamed feature
    return &pb.Feature{Location: point},nil
}
```

The method is passed a context object for the RPC and the client’s `Point` protocol buffer request. It returns a `Feature` protocol buffer object with the response information and an `error`. In the method we populate the `Feature` with the appropriate information, and then return it along with an nil error to tell gRPC that we’ve finished dealing with the RPC and that the `Feature` can be returned to the client.

### 4.3 Server-side streaming RPC

`ListFeatures` is a server-side streaming RPC, so need to send back multiple `Feature`s to our client.

```go
func (s *routeGuideServer) ListFeatures(rect *pb.Rectangle, stream pb.RouteGuide_ListFeaturesServer) error {
    for _,feature := range s.saveFeatures {
        if inRange(feature.Location,rect){
            if err := stream.Send(feature);err != nil {
            	return err
        	}
       	}
    }
    return nil
}
```

As you can see, instead of getting simple request and response objects in our method parameters, this time we get a request object (the `Rectangle` in which our client wants to find `Feature`s) and a special `RouteGuide_ListFeaturesServer` object to write our responses.

In the method, we populate as many `Feature` objects as we need to return, writing them to the `RouteGuide_ListFeaturesServer` using its `Send()` method. Finally, as in our simple RPC, we return a nil error to tell gRPC that we have finished writing responses. Should any error happen in this call, we return a non-nil error; the gRPC layer will translate it into an appropriate RPC status to be sent on the wire.

### 4.4 Client-side streaming RPC

The client-side streaming method `RecordRoute`, where we get a stream of `Point`s from the client and return a single `RouteSummary` with information about the trip. As you can see, this time the method doesn’t have a request parameter at all. Instead, it gets a `RouteGuide_RecordRouteServer` stream, which the server can use to both read *and* write messages - it can receive client messages using its `Recv()` method and return its single response using its `SendAndClose()` method :

```go
func (s *routeGuideServer) RecordRoute(stream pb.RouteGuide_RecordRouteServer) error {
    var pointCount, featureCount, distance int32
    var lastPoint *pb.Point
    startTime := time.Now()
    for {
        point, err := stream.Recv()
        if err == io.EOF {
            endTime := time.Now()
            return stream.SendAndClose(&pb.RouteSummary{
                PointCount: pointCount,
                FeatureCount: featureCount,
                Distance: distance,
                ElapsedTime: int32(endTime.Sub(startTime).Seconds())
            })
        }
        if err != nil {
            return err
        }
        pointCount++
        for _, feature := range s.savedFeatures {
            if proto.Equal(feature.Location, point) {
                featureCount++
            }
        }
        if lastPoint != nil {
            distance += calcDistance(lastPoint, point)
        }
        lastPoint = point
    }
}
```

In the method body we use the `RouteGuide_RecordRouteServer`'s `Recv()` method to repeatedly read in our client’s requests to a request object (in this case a `Point`) until there are no more messages: the server needs to check the error returned from `Recv()` after each call. If this is nil, the stream is still good and it can continue reading; if it is `io.EOF` the message stream has ended and the server can return its `RouteSummary`. If it has any other value, we return the error “as is ” so that it’ll be translated to an RPC status by the gRPC layer.

### 4.5 Bidirectional streaming RPC

Finally, let’s look at our bidirectional streaming RPC `RouteChat()`:

```go
func (s *routeGuideServer) RouteChat(stream pb.RouteGuide_RouteChatServer) error {
    for {
        in, err := stream.Recv()
        if err == io.EOF {
            return nil
        }
        if err != nil {
            return err
        }
        key := serialize(in.Location)
        ... // look for notes to be sent to client
        for _, note := range s.routeNotes[key] {
            if err := stream.Send(note);err != nil {
                return err
            }
        }
    }
}
```

This time we get a `RouteGuide_RouteChatServer` stream that, as in our client-side streaming example, can be used to read and write messages. However, this time we return values via our method’s stream while the client is still writing messages to *their* message stream.

The syntax for reading and writing here is very similar to our client-side streaming method, except the server uses the stream’s `Send()` method rather than `SendAndClose()` because it’s writing multiple responses. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order -- the streams operate completely independently.

## 5. Starting the server

Once we have implemented all our methods, we also need to start up a gRPC server so that clients can actually use our service. The following snippet show how we do this for our `RouteGuide` service:

```go
flag.Parse()
lis, err := net.Listen("tcp",fmt.Sprintf("localhost:%d",*port))
if err != nil {
    log.Fatalf("failed to listen: %v",err)
}
var opts []grpc.ServerOption
...
grpcServer := grpc.NewServer(opts...)
pb.RegisterRouteGuideServer(grpcServer, newServer())
grpcServer.Serve(lis)
```

To build and start a server, we:

1. Specify the port we want to use to listen for client requests using:
   `lis, err := net.Listen(...)`
2. Create an instance of the gRPC server using
   `grpc.NewServer()`.
3. Register our service implementation with the gRPC server.
4. Call `Serve()` on the server with our port details to do a blocking wait until the process is killed or `Stop()` is called.

## 6. Creating the client

### 6.1 Creating a stub

To call service methods, we first need to create a gRPC *channel* to communicate with the server. We create this by passing the server address and port number to `grpc.Dial()` as follows:

```go
var opts []grpc.DialOption
...
conn, err := grpc.Dial(*serverAddr, opts...)
if err != nil {
    ...
}
defer conn.Close()
```

You can use `DialOptions` to set the auth credentials (for example, TLS, GCE credentials, or JWT credentials) in `grpc.Dial` when a service requires them. The `RouteGuide` service doesn’t require any credentials.

Once the gRPC *channel* is setup, we need a client stub to perform RPCs. We get it using the `NewRouteGuideClient` method provided by the `pb` package generated from the example `.proto` file:

```go
client := pb.NewRouteGuideClient(conn)
```

### 6.2 Calling service methods

Now let’s look at how we call our service methods. Note that in gRPC-Go, RPCs operate in a blocking/synchronous mode, which means that the RPCd call waits for the server to respond, and will either return a response or an error.

### 6.3 Simple RPC

Calling the simple RPC `GetFeature` is nearly as straightforward as calling a local method.

```go
feature, err := client.GetFeature(context.Background(),&pb.Point{409146138, -746188906})
if err != nil {
    ...
}
```

As you can see, we call the method on the stub we got earlier. In our method parameters we create and populate a request protocol buffer object (in our case `Point`). We also pass a `context.Context` Object which lets us change our RPC’s behavior if necessary, such as time-out/cancel an RPC in flight. If the call doesn’t return an error, then we can read the response information from the server from the first return value.

### 6.4 Server-side streaming RPC

Here’s where we call the server-side streaming method `ListFeatures`, which returns a stream of geographical `Feature`s. If you’ve already read Creating the server some of this may look very familiar - streaming RPCs are implemented in a similar way on both side.

```go
rect := &pb.Rectangle{...} // initialize a pb.Rectangle
stream, err := client.ListFeatures(context.Background(),rect)
if err != nil {
    ...
}
for {
    feature, err := stream.Recv()
    if err == io.EOF {
        break
    }
    if err != nil {
        log.Fatalf("%v.ListFeatures(_) = _, %v",client, err)
    }
    log.Println(feature)
}
```

As in the simple RPC, we pass the method a context  and a request. However, instead of getting a response object back, we get back an instance of `RouteGuide_ListFeaturesClient`. The Client can use the `RouteGuide_ListFeaturesClient` stream to read the server’s response.

We use the `RouteGuide_ListFeaturesClient`'s' `Recv()` method to repeatedly read in the server’s response to a response protocol buffer object (in this case a `Feature`) until there are no more messages: the client needs to check the error `err` returned from `Recv()` after each call; if it’s `io.EOF` then the message stream has ended; otherwise there must be an RPC error, which is passed over through `err`.

### 6.5 Client-side streaming RPC

The client-side streaming method `RecordRoute` is similar to the server-side method, except that we only pass the method a context and get a `RouteGuide_RecordRouteClient` stream back, which we can use to both write and read messages.

```go
// Create a random number of random points
r := rand.New(rand.NewSource(time.Now().UnixNano()))
pointCount := int(r.Int31n(100)) + 2 // Traverse at least two points
var points []*pb.Point
for i := 0; i < pointCount; i++ {
    points = append(points,randomPoint(r))
}
log.Printf("Traversing %d points.",len(points))
stream, err := client.RecordRoute(context.Background())
if err != nil {
    log.Fatalf("%v.RecordRoute(_) = _, %v"),client, err)
}
for _, point := range points {
    if err := stream.Send(point); err != nil {
        log.Fatalf("%v.Send(%v) = %v",stream, point, err)
    }
}
reply, err := stream.CloseAndRecv()
if err != nil {
    log.Fatalf("%v.CloseAndRecv() got error %v, want %v",stream, err, nil)
}
log.Printf("Route summary: %v",reply)
```

The `RouteGuide_RecordRouteClient` has a `Send()` method that we can use to send requests to the server. Once we have finished writing our client’s requests to the stream using `Send()`, we need to call `CloseAndRecv()` on the stream to let gRPC know that we’ve finished writing and are expecting to receive a response. We get our RPC status from the `err` returned from `CloseAndRecv()` . if the status is `nil`, then the first return value from `CloseAndRecv()` will be a valid server response.

### 6.6 Bidirectional streaming RPC

Finally, let’s look at our bidirectional streaming RPC `RouteChat()`. As in the case of `RecordRoute`, we only pass the method a context object and get back a stream that we can use to both write and read messages. However, this time we return values via our method’s stream while the server is still writing messages to their message stream.

```go
stream, err := client.RouteChat(context.Background())
waitc := make(chan struct{})
go func() {
    for {
        in, err := stream.Recv()
        if err == io.EOF {
            // read done
            close(waitc)
            return 
        }
        if err != nil {
            log.Fatalf("Failed to receive a note: %v",err)
        }
        log.Printf("Got message %s at point(%d,%d)",in.Message, in.Location.Latitude, in.Location.Longitude)
    } 
}()
for _, note := range notes {
    if err := stream.Send(note); err != nil {
        log.Fatalf("Failed to send a note: %v", err)
    }
}
stream.CloseSend()
<- waitc
```

The syntax for reading and writing here is very similar to client-side streaming method, except we use the stream’s `CloseSend()` method once we’ve finished our call. Although each side will always get the other’s messages in the order they were written, both the client and server can read and write in any order -- the streams operate completely independently.

## 7. Try it out

1. Run the server:

   ```sh
   $ go run server/server.go -json_db_file ../testdata/route_guide_db.json
   ```

2. Run the client:

   ```sh
   $ go run client/client.go
   ...
   2022/01/05 17:28:59 Traversing 34 points.
   2022/01/05 17:28:59 Route summary: point_count:34
   2022/01/05 17:28:59 Got message First message at point (0, 1)
   2022/01/05 17:28:59 Got message Second message at point (0, 2)
   2022/01/05 17:28:59 Got message Third message at point (0, 3)
   2022/01/05 17:28:59 Got message First message at point (0, 1)
   2022/01/05 17:28:59 Got message Fourth message at point (0, 1)
   2022/01/05 17:28:59 Got message Second message at point (0, 2)
   2022/01/05 17:28:59 Got message Fifth message at point (0, 2)
   2022/01/05 17:28:59 Got message Third message at point (0, 3)
   2022/01/05 17:28:59 Got message Sixth message at point (0, 3)
   ```

## Reference

1. [Basic Tutorial](https://grpc.io/docs/languages/go/basics/) gRPC docs
