---
title: "Grpc Gateway"
date: 2022-11-16T12:49:38+08:00
draft: true
tags: ["Go","ms"]
categories: ["Go"]
---

# grpc-gateway

## introduce

gRPC-Gateway is a plugin of [protoc](https://github.com/protocolbuffers/protobuf). It reads a [gRPC](https://grpc.io/) service definition and generates a reverse-proxy server which translates a RESTful JSON API into gRPC. This server is generated according to [custom options](https://cloud.google.com/service-infrastructure/docs/service-management/reference/rpc/google.api#http) in your gRPC definition.

## About

The gRPC-Gateway is a plugin of the Google protocol buffers compiler [protoc](https://github.com/protocolbuffers/protobuf). It reads protobuf service definitions and generates a reverse-proxy server which translates a RESTful HTTP API into gRPC. This server is generated according to the [`google.api.http`](https://github.com/googleapis/googleapis/blob/master/google/api/http.proto#L46) annotations in your service definitions.

This helps you provide your APIs in both gRPC and RESTful style at the same time.

![grpc-gateway](https://raw.githubusercontent.com/grpc-ecosystem/grpc-gateway/475b8d96f7d79887691fea8a550cc207ef5183b1/docs/assets/images/architecture_introduction_diagram.svg)

## Docs

You can read our docs at:

- [https://grpc-ecosystem.github.io/grpc-gateway/](https://grpc-ecosystem.github.io/grpc-gateway/)

## Background

gRPC is great -- it generates API clients and server stubs in many programming languages, it is fast, easy-to-use, bandwidth-efficient and its design is combat-proven by Google. However, you might still want to provide a traditional RESTful JSON API as well. Reasons can range from maintaining backward-compatibility, supporting languages or clients that are not well supported by gRPC, to simply maintaining the aesthetics and tooling involved with a RESTful JSON architecture.

This project aims to provide that HTTP+JSON interface to your gRPC service. A small amount of configuration in your service to attach HTTP semantics is all that's needed to generate a reverse-proxy with this library.

## Basic Usage Examples

### Use protobuf to definition gRPC service

Create a new project greeter, Execute go mod init in project director to complete go module initialize.

> go mod init greeter

In project director to create one proto file greeter/proto/hello_world/hello_world.proto

```
syntax = "proto3";

package hello_world;

option go_package="greeter/proto/hello_world";

// defined one Greeter service
service Greeter {
  // Hello Method
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// defined request message
message HelloRequest {
  string name = 1;
}

// defined reply message
message HelloReply {
  string message = 1;
}
```
### Generate code

```
$ protoc -I=proto \
   --go_out=proto --go_opt=paths=source_relative \
   --go-grpc_out=proto --go-grpc_opt=paths=source_relative \
   hello_world/hello_world.proto
```

Use go mod tidy to Solving the dependency problem.

After generating pb and gRPC related codes, register the RPC service in the main function and start the gRPC Server.

```
package main

import (
	"context"
	"log"
	"net"

	"google.golang.org/grpc"

	"greeter/proto/hello_world"
)

type server struct {
	hello_world.UnimplementedGreeterServer
}

func NewServer() *server {
	return &server{}
}

func (s *server) SayHello(ctx context.Context, in *hello_world.HelloRequest) (*hello_world.HelloReply, error) {
	return &hello_world.HelloReply{Message: in.Name + " world"}, nil
}

func main() {
	// Create a listener on TCP port
	lis, err := net.Listen("tcp", ":8080")
	if err != nil {
		log.Fatalln("Failed to listen:", err)
	}

	// create one gRPC server object
	s := grpc.NewServer()
	// register Greeter service to server
	hello_world.RegisterGreeterServer(s, &server{})
	// start gRPC Server
	log.Println("Serving gRPC on 0.0.0.0:8080")
	log.Fatal(s.Serve(lis))
}
```

file tree
```
.
├── go.mod
├── go.sum
├── main.go
└── proto
    └── hello_world
        ├── hello_world_grpc.pb.go
        ├── hello_world.pb.go
        └── hello_world.proto
```


### Add gRPC-Gateway annotations to existing proto files

now we have a working Go gRPC server, Next you need to add the gRPC-Gateway annotation. These annotations define how gRPC services map to JSON requests and responses. When using protocol buffers, each RPC service must use the google.api.HTTP annotation to define the HTTP method and path.

Therefore, we need to import google/api/http.proto into the proto file. We also need to add the required HTTP->gRPC mapping. In this example, we map POST /v1/hello_world/echo to a SayHello RPC.

```
syntax = "proto3";

package hello_world;

import "google/api/annotations.proto";

option go_package="greeter/proto/hello_world";

// defined one Greeter service
service Greeter {
  // Hello Method
  rpc SayHello (HelloRequest) returns (HelloReply) {
    option(google.api.http) = {
      post: "/v1/hello_world/echo"
      body: "*"
    };
  }
}

// defined request message
message HelloRequest {
  string name = 1;
}

// defined reply message
message HelloReply {
  string message = 1;
}
```

### Generate gRPC-Gateway stubs

#### Import dependent packages

Before we can use protoc to generate stubs, we need to copy some dependencies into our proto file structure. Copy a subset of googleapis from the [official repository](https://github.com/googleapis/googleapis) into your local archetype file structure. The copied directory should look like this:

```
.
├── go.mod
├── go.sum
├── main.go
└── proto
    ├── google
    │   └── api
    │       ├── annotations.proto
    │       └── http.proto
    └── hello_world
        ├── hello_world_grpc.pb.go
        ├── hello_world.pb.go
        └── hello_world.proto
```

### Install the gRPC-Gateway tool

> go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway@v2

Now we need to add the gRPC-Gateway generator to the calling command of protoc:

```
$ protoc -I=proto \
   --go_out=proto --go_opt=paths=source_relative \
   --go-grpc_out=proto --go-grpc_opt=paths=source_relative \
   --grpc-gateway_out=proto --grpc-gateway_opt=paths=source_relative \
   hello_world/hello_world.proto
```

### add HTTP Server code

We also need to add and start the gRPC-Gateway mux in the main.go file. Modify our main function as shown in the code below.

```
package main

import (
	"context"
	"log"
	"net"
	"net/http"

	"greeter/proto/hello_world"

	"github.com/grpc-ecosystem/grpc-gateway/v2/runtime" // v2
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
)

type server struct {
	hello_world.UnimplementedGreeterServer
}

func NewServer() *server {
	return &server{}
}

func (s *server) SayHello(ctx context.Context, in *hello_world.HelloRequest) (*hello_world.HelloReply, error) {
	return &hello_world.HelloReply{Message: in.Name + " world"}, nil
}

func main() {
	// Create a listener on TCP port
	lis, err := net.Listen("tcp", ":8080")
	if err != nil {
		log.Fatalln("Failed to listen:", err)
	}

	// create one gRPC server object
	s := grpc.NewServer()
	// register Greeter service to server
	hello_world.RegisterGreeterServer(s, &server{})
	// by 8080 to start gRPC Server
	log.Println("Serving gRPC on 0.0.0.0:8080")
	go func() {
		log.Fatalln(s.Serve(lis))
	}()

	// Create a client connection to the gRPC server we just started
	// gRPC-Gateway uses it to proxy requests (convert HTTP requests to RPC requests)
	conn, err := grpc.DialContext(
		context.Background(),
		"0.0.0.0:8080",
		grpc.WithBlock(),
		grpc.WithTransportCredentials(insecure.NewCredentials()),
	)
	if err != nil {
		log.Fatalln("Failed to dial server:", err)
	}

	gwmux := runtime.NewServeMux()
	// Register Greeter
	err = hello_world.RegisterGreeterHandler(context.Background(), gwmux, conn)
	if err != nil {
		log.Fatalln("Failed to register gateway:", err)
	}

	gwServer := &http.Server{
		Addr:    ":8090",
		Handler: gwmux,
	}
	// 8090 prot provide gRPC-Gateway server
	log.Println("Serving gRPC-Gateway on http://0.0.0.0:8090")
	log.Fatalln(gwServer.ListenAndServe())
}
```

#### Notice

- The imported "github.com/grpc-ecosystem/grpc-gateway/v2/runtime" is the v2 version.
- The gRPC service needs to be started with a separate goroutine.

#### start server
```
go run main.go
```

#### use curl to send http request

```
curl -X POST -k http://localhost:8090/v1/hello_world/echo -d '{"name": " hello"}'
```

#### get message

```
{"message":"hello world"}
```

### references

https://www.liwenzhou.com/posts/Go/grpc-gateway/
