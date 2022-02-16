---
title: "011 译 | Actions介绍：为Hasura增加自定义业务逻辑"
date: 2022-02-16T22:22:45+08:00
tags: ["技术翻译", "Hasura"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

翻译自：[Introducing Actions: Add custom business logic to Hasura](https://hasura.io/blog/introducing-actions/)

Actions 让你容易把新的或现存的业务逻辑带入 Hasura。本文，我将向你介绍 actions 是什么，以及他们为什么被这样设计！

## 为什么 actions？

使用 actions 你可以容易的增加各种类型的业务逻辑到 GraphQL API。Hasura 处理 CRUD，你的 actions 出来剩下的一切！

Actions 和 Hasura 的效果非常好，所以你可以选择，那部分让 Hasura 处理，以及哪部分你自己处理。它是一个频谱，你可以随着应用演化来回移动。

就像 Hasura 和数据库工作的方式，我们针对 Actions 的目标是去帮助目标用户从一开始就使用最佳实践去实现目的。

| 特性          | 描述                                                                                                          |
| ------------- | ------------------------------------------------------------------------------------------------------------- |
| 可扩展架构    | 一个增加复杂领域逻辑的可扩展模式                                                                              |
| 没有锁定      | 不锁定任何语言、框架、运行时去实现你的业务逻辑                                                                |
| 最佳性能实践  | 即使是你写的业务逻辑，Hasura 仍然处理所有 GraphQL 信息。像往常一样快。Hasura 确保你的业务逻辑自动进入最好性能 |
| Hasura 安全性 | Hasura 拥有非常强大的身份验证系统。这个系统能够容易地在 Actions 中从你的业务逻辑中重用                        |
| serverless    | Hasura actions 与 serverless 特别匹配，因为每个 action 能被一个独立的函数支撑                                 |

## Actions 如何工作？

![Hasura Actions: The ease of writing REST, and the joy of consuming GraphQL](https://hasura.io/blog/content/images/2020/05/Screenshot-2020-05-05-at-7.18.46-PM.png)

- SETP1：你可以在 Hasura 中指定查询或变更 GraphQL 合约
- SETP2：Hasura 验证和代理进来的 GraphQL 请求到 REST API（又名 Action 处理器）
- SETP3：REST API 做一些事情并返回最小的输出
- SETP4：Hasura 根据用户在最终 GraphQL 响应中想要的内容，使用数据图的剩余部分丰富输出。因为 Hasura 将数据连接到一起，你可以获得 Hasura 超快 GraphQL 执行的所有益处。

尝试下 Actions，使用 Hasura Cloud 启动 GraphQL 项目，并且阅读 Actions [文档](https://hasura.io/docs/latest/graphql/core/actions/index.html#utm_source=Hasura%20Blog&utm_medium=Blog%20Article&utm_campaign=Project%20Cloud&utm_content=introducing%20actions)去开始。

核查在 4 月 #ActionPackedApril 系列的 youtube 播放列表，看你能用 actions 做的所有不同的事情。

[An example of using Hasura actions with NextJS API routes!](https://www.youtube.com/embed/videoseries?list=PLTRTpHrUcSB9e9m43VKDrO0UBDQfZmW90&enablejsapi=1&origin=https://hasura.io)

## 可扩展架构

很多 APIs 由混合的 CRUD 和非 CRUD APIs 组成。为非 CRUD 设计的可扩展模式是基于“命令&查询”架构的模式。

当我们想到 CRUD，我们想到模型，并且我们想到使用一些 API 端点（或 GraphQL 字段）读取和写入他们。

当我们想到命令和查询设计，你想到运行一个“命令”，它是一个在你的领域中对用户有意义的操作或工作流。该命令的处理程序然后运行处理请求的逻辑，然后继续更新模型视图。用户发起请求的最终响应是对这些更新模型的一个“查询”。

![A command & query architecture for GraphQL with Hasura](https://hasura.io/blog/content/images/2020/05/Screenshot-2020-05-05-at-7.22.20-PM.png)

### 为什么该架构是有好处的？
