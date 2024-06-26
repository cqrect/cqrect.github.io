---
title: go-micro入门
date: 2024-06-07 09:44:40
categories:
- go
tags:
- go
- go-micro
- 微服务
---

微服务是一种软件架构模式，用于将大型整体应用程序分解为更小的可管理独立服务，这些独立服务通过跨语言的协议进行通信，每个服务都专注于做好一件事情。

go-micro 是一个简化分布式开发的微服务生态系统。它为开发分布式应用程序提供了基本的构建模块。go-micro 的设计哲学是：通过提供组件工具，明确微服务开发的边界，让我们专注于开发业务本身。


<!-- more -->


## 构成组件


### Go Micro

- 用于在 Go 中编写微服务的**插件式 RPC 框架**。它提供了**用于服务发现、客户端负载平衡、编码、同步和异步通信库**。


### API

- 提供并将 HTTP 请求**路由到对应微服务的 API 网关**。它充当单个入口点，可以用作反向代理或将 HTTP 请求转换为 RPC。


### Sidecar

- 一种对语言透明的 RPC 代理（也就是和语言无关），具有 go-micro 作为 HTTP 端点的所有功能。虽然 Go 是构建微服务的优选语言，但商业项目中也经常使用其他语言一起开发微服务。因此 Sidecar 提供了一种将其他语言应用程序集成到 Micro 世界的方法。
- Sidecar 又被称为服务网格，其作用是：第一，微服务治理与业务逻辑的解耦；第二，异构系统的统一治理。


### Web

- 用于 Micro Web 应用程序的仪表板和反向代理。官方认为基于微服务建立 Web 应用是通用场景，因为 Web 被视为微服务领域的一等公民。它的行为非常像 API 反向代理，另外也包括对 Web Sockets 的支持。


### CLI

- 提供命令行界面让我们和微服务进行交互。CLI 还可以使我们利用 Sidecar 作为代理。
- CLI 是 MicroServices Toolkit Micro 的命令行界面，包括获取服务、查看服务、查询服务健康状态等功能。


### Bot

- Hubot 风格 bot。在我们的微服务平台中，可以通过 Slack、HipChat、XMPP 等进行交互。它通过消息传递提供 CLI 的功能。可以添加其他命令来自动执行常见的操作任务。
- Bot 使用它的命名空间监视服务注册中心的服务。默认名称空间是 `go.micro.bot`。该名称空间内的任何服务都将自动添加到可用命令列表中。执行命令时，机器人将使用 `Command.Exec` 方法调用该服务。


## Go Micro 组件架构

> Go Micro 为微服务提供了基本的构建模块，其目标是简化分布式系统开发。

架构图如下：

{% asset_img go_micro_architectures.png Go Micro 组件架构图 %}


### Registry 注册中心

注册中心提供可插入的服务发现库，来查找正在运行的服务。默认的实现方式是 consul。


### Selector 负载均衡

Selector 选择器实现 go-micro 的负载均衡机制。

当客户端向服务器发出请求时，首先查询服务的注册中心，注册中心会返回一个正在运行服务的节点列表，选择器将选择这些节点中的其中一个用于查询请求。

多次调用选择器将使用平衡算法。目前的方法是循环法、随机哈希、黑名单。go-micro 就是通过这种机制实现负载均衡的。


### Broker 事件驱动：发布订阅

Broker 是发布和订阅的可插入接口。


### Transport 消息传输

传输是通过点对点传输消息的可插拔接口。

目前的实现是 http、rabbitmq 和 nats。