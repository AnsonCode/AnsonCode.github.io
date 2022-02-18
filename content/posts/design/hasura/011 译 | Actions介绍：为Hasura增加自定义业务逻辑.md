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

它允许每个作者发送命令根本地处理与“领域操作”有关的业务逻辑。有 CRUD 的模型或许变得非常复杂，并且开始封装与模型完全无关的逻辑。例如，插入 X 应该发送一封邮件。

通常，设置使用微服务基于 CQS/CQRS 的管道需要花费一些工作。使用 Hasura，该架构开箱即用，并且它非常容易跨团队分配工作。

而且，从安全角度来看，它容易考虑权限控制。

1. 在领域实体层级：谁有能力访问该模型或操作（action）
2. 在用户层级：这个用户有权限访问什么模型和操作

Hasura 的安全/权限系统由 API 驱动，所以你能仅通过一个 API 调用轻松的导出上面所有的信息。

### 没有供应商锁定

我们设计 Hasura 是为了在新的或现存的数据库中工作。当你使用 Hasura 时，你最终得到的数据模型非常自然的是你作为一个好的数据库用户将放置的数据模型。这意味着，如果你想明天“驱逐”Hasura，你的数据仍然是你的，随时准备被你想要的应用程序使用。

通过 Actions，我们将这种思想扩展到业务逻辑。如今 GraphQL 仍然是一个年轻和成熟的生态系统。GraphQL 框架要么由供应商支持，要么由没有太多贡献者社区的独立开发者开发。此外，很多 GraphQL 工作被限制在 JavaScript 生态系统。

然而，REST 不受这些问题困扰。构建、维护和运行 REST APIs 的工具是成熟的，而且相当出色。REST 运行时环境在最近几年里，也快速发展。Serverless 运行时也不断地变得成熟，并且能运行以轻量级 REST 端点形式呈现的函数。然而，GraphQL 服务不适合 Serverless 部署。生产环境的 GraphQL 服务需要保持一定状态去验证、缓存、限制流入请求速率。GraphQL 订阅，维护起来相当复杂，在 Serverless 环境几乎是不可能的。

通过 Hasura，我们的目的是帮助用户继续使用他们最喜欢的技术和工具（就像他们拥有他们自己的数据和数据库一样），用一个即使他们不用 Hasura 也基本相同的方式，去增加业务逻辑！

使用 Hasura actions，你的业务逻辑可以是你最喜欢技术的任何组合。

![Bring your favourite language x framework x runtime x deployment for Hasura Actions](https://hasura.io/blog/content/images/2020/05/Screenshot-2020-05-05-at-8.51.56-PM.png)

如果你想从 Hasura 迁移，然后你可以完全退出并且构建自己的 GraphQL 服务，甚至完全从 GraphQL 迁移出去，而不必重做任何工作。你的数据，你的数据模型，你的业务逻辑将继续推进。

### 性能最佳实践

当你使用 actions，Hasura 帮助你陷入功成名就的深渊。

例如：GraphQL 变更的最佳实践是委托输入信息到一个能处理它的函数中，然后给出“写后读”的语义，让 GraphQL 层为变更生成正确的响应（取决于用户的请求）。

Hasura actions 就是那样，但是以一种对微服务、无服务（serverless）架构友好的方式。你的 REST API 执行业务逻辑，并且仅仅返回正确的数据或数据的正确的引用。Hasura 高效地将这些引用与剩下的数据图连接，并且生成你客户端需要的 GraphQL 响应。

如果你正在构建你对自己的 GraphQL 服务，你一定担心 N+1 查询问题，校验/[缓存](https://hasura.io/graphql/caching/)查询计划和 Hasura 帮你处理的其他性能优化。这些问题在 serverless 环境更难去定位。然而，使用 Hasura 的 Actions 架构，serverless REST 完美适配了它。

### 利用 Hasura 安全性

Hasura’s GraphQL[安全](https://hasura.io/graphql/security/)涵盖了下列事情：

1. 身份验证规则（在模型水平声明）
2. 允许列表
3. 速率限制
4. 审计

当你构建你的业务逻辑和你的 REST API 处理器时，你不需要担心上述任何事情。Hasura actions 的设计也让它很容易去利用和复用在 Hasura 层面指定的身份验证规则。

例如，你可以指定与终端用户相符的 Hasura 身份验证规则。从 Actions 端点，你可以作为终端用户角色调用 Hasura GraphQL APIs。这让编写“后端”代码更容易，它和“前端”代码具有相同的执行权限，但是附加的步骤和验证是不可能在前端运行的。你的“模型”级别安全保持在同一个地方。

### 与 Serverless 很好的匹配

Hasura Actions 被 REST APIs 支持，这些 APIs 可能被一个服务或 serverless 函数或任何能服务 REST API 的其他东西提供。

无论从架构/心智模型角度考虑还是从性能、可扩展角度考虑，Serverless 都是非常合适的。 每个 Hasura action 都能被一个 serverless 函数支持（暴露 REST 端点）。这个 serverless 函数能被独立的开发和部署。

这是一个简洁的关注点分离，因为 Hasura 为消费者暴露了一个统一连贯的 GraphQL API ，无论 serverless 函数如何构建和运行。

Hasura 也基于数据库暴露了一系列 GraphQL 端点，它能被业务逻辑用于去读写数据到数据库。这对 serverless 特别好，因为直接连接到数据库具有挑战。

你写的业务逻辑现在对性能和扩展来说都是非常成功的，因为

1. Hasura 保证 GraphQL/数据库性能
2. 运行 serverless 函数的云提供商保证可扩展性

### 相关的架构和范式

使用 Actions 的驱动业务逻辑的架构与 [CQS/CQRS](https://martinfowler.com/bliki/CQRS.html) 和 三要素[3factor](https://3factor.app/) 架构非常相似（阅读更多[使用事件驱动构建现代 3 要素应用](https://hasura.io/event-driven-programming/)）

![Image from https://martinfowler.com/bliki/CQRS.html](https://hasura.io/blog/content/images/2019/10/cqrs.png)

### 额外阅读

- [Creating a GraphQL API for Notion with Hasura Actions](https://hasura.io/blog/creating-a-graphql-api-for-notion-with-hasura-actions/)
