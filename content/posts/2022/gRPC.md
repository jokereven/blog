---
title: "gRPC"
date: 2022-11-17T10:44:45+08:00
draft: true
tags: ["gRPC","Go"]
categories: ["Go"]
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
[protocol-buffers 官方Go教程](https://developers.google.com/protocol-buffers/docs/gotutorial)

## check

```
$ protoc --version
libprotoc 3.21.8

$ protoc-gen-go --version
protoc-gen-go v1.28.1

$ protoc-gen-go-grpc --version
protoc-gen-go-grpc 1.2.0
```
