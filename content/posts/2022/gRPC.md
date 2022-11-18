---
title: 'gRPC'
date: 2022-11-17T10:44:45+08:00
draft: true
tags: ['gRPC', 'Go']
categories: ['Go']
---

> only how to use

## install gRPC

```
go get google.golang.org/grpc@latest
```

## install Protocol Buffers v3

[Source](https://github.com/protocolbuffers/protobuf/releases)

## install plugin of go

```
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
```

## install grpc plugin

```
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

[protocol-buffers 官方 Go 教程](https://developers.google.com/protocol-buffers/docs/gotutorial)

## check

```
$ protoc --version
libprotoc 3.21.8

$ protoc-gen-go --version
protoc-gen-go v1.28.1

$ protoc-gen-go-grpc --version
protoc-gen-go-grpc 1.2.0
```

## full code

https://github.com/jokereven/gRPC

### tips

```
protoc --go_out=. --go_opt=paths=source_relative \
--go-grpc_out=. --go-grpc_opt=paths=source_relative \
proto/hello_world.proto
```

windows:

```
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative proto/hello_world.proto
```

```
python3 -m grpc_tools.protoc -Iproto --python_out=. --grpc_python_out=. proto/hello_world.proto
```

-Iproto is -I + proto file directory, and the python client import package name is like hello_world_pb2 and hello_world_pb2_grpc your proto file name + \_pb2 and + \_pb2_grpc

## references

[RPC 原理与 Go RPC](https://www.liwenzhou.com/posts/Go/rpc/)

[Protocol Buffers V3 中文语法指南[翻译]](https://www.liwenzhou.com/posts/Go/Protobuf3-language-guide-zh/)

[protobuf 中使用 oneof、WrapValue 和 FieldMask](https://www.liwenzhou.com/posts/Go/oneof-wrappers-field_mask/)

[gRPC 教程](https://www.liwenzhou.com/posts/Go/gRPC/)

[gRPC-Gateway 使用指南](https://www.liwenzhou.com/posts/Go/gRPC)

[gRPC Transcoding](https://www.liwenzhou.com/posts/Go/grpc-transcoding/)

[Golang gRPC Middlewares: interceptor chaining, auth, logging, retries and more.](https://github.com/grpc-ecosystem/go-grpc-middleware)
