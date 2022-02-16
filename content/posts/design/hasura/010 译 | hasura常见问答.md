---
title: "010 译 | hasura常见问答"
date: 2022-02-15T22:45:45+08:00
tags: ["技术翻译", "Hasura"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

翻译自：[Frequently Asked Questions](https://hasura.io/blog/optimistic-ui-and-clobbering/)

## 1.Hasura 支持什么类型的工作负载和数据库？

- PostgresQL (和 Postgres 数据库家族: Yugabyte, Timescale, Citus, Aurora)
- SQL Server
- Big Query
- Mysql(beta)

## 2.Hasura 如何工作？

尽管 Hasura 把它自己作为一个 web 服务，Hasura 是一个十分好的 JIT 编译器，Hasura 接收通过 HTTP 传入的 GraphQL API 调用，然后在将数据查询委托给下游数据源时，尝试实现理论最优性能。你能在这片文章读到更多 Hasura 设计思想。

## 3.Hasura 节省多少时间和精力？

Hasura 为开发者减少 50-80%的时间。你可以从[案例研究](https://hasura.io/case-studies/)中发现更多。

## 4.如果早已有现存应用或 API，我如何使用 Hasura？

Hasura 为增量采用而设计，不必推到重来或者完全重建你的技术栈。你可以增量迁移你的应用到 Hasura。使用 Hasura 首先使用现存数据为你的应用构建任何新的特性，还有一个高性能读取层，适用于读业务重的场景，因为它不需要花时间设置。你也可以通过 Hasura Actions 在现存应用中使用任何业务逻辑。这将给你时间迁移任何遗留代码或使用 Hasura 重写现存的微服务。

## 5.把业务逻辑放在哪里？

Hasura 通过高性能灵活的 API 暴露领域数据和业务逻辑。如果你把 Hasura 指向一个数据库，你或许想要知道，你应该在哪里写 API 需要的必须的业务逻辑。

注意，Hasura 消除了为内部或外部授权规则写任何所需代码的需要。

Hasura 提供了 4 种方法暴露你的领域中现存或新增的业务逻辑：

- 事件触发：无论上游数据库何时变化，Hasura 可以将该变化作为事件捕获，并且投递到能处理数据变化事件的 HTTP webhook 上，异步对其响应。除了给事件附加特殊逻辑片段之外，如果你正考虑构建端到端实时响应程序，它特别有用。

  - 在[https://3factor.app](https://3factor.app/)上读更多架构
  - 在 Hasura[文档](https://hasura.io/docs/latest/graphql/core/event-triggers/index.html#event-triggers)上读更多事件触发
  - 在[https://hasura.io/learn/](https://hasura.io/learn/)上通过一个快速教程学习如何发送事件触发

- REST APIs：如果你有新的或现存的服务领域数据或逻辑的 REST APIs，你可以容易地将 Hasura 连接到他们，并且扩展 Hasura 暴露的 GraphQL schema。这是有用的，不仅当你有包含很多事务或应用逻辑的遗留 APIs 时，而且当你想去使用云原生代码部署为容器或 serverless 函数去构建和服务自定义逻辑时。

  - 阅读更多[Hasura Actions](https://hasura.io/docs/latest/graphql/core/actions/index.html#actions).

- GraphQL APIs：如果你有扩展 schema 的新的或现存的 GraphQL 服务，比如，集成自定义逻辑的自定义变更，或者你想要使用来自一个你不是直接拥有的服务的“子图”扩展 GraphQL API，你可以在 Hasura 中使用“Remote Schemas”，把作为数据或逻辑提供的 GraphQL 服务带到你的 GraphQL API 中。

  - 阅读更多关于[Remote Schemas](https://hasura.io/docs/latest/graphql/core/remote-schemas/index.html#remote-schemas)

- 数据库存储过程/函数：存储过程和函数是一个写入和存储高性能业务逻辑或事务逻辑的通用的方式，这很接近数据。作为 Hasura 通过数据库暴露的 GraphQL API 的一部分，Hasura 允许你在 GraphQL schema 中以字段的方式暴露存储过程或函数。如果你熟悉数据库，或者写自定义、高性能逻辑（自定义函数），是引入位于你数据库的现存业务逻辑的很好方式。

  - 阅读更多关于[custom functions](https://hasura.io/docs/latest/graphql/core/databases/postgres/schema/custom-functions.html#custom-sql-functions)

根据你的现存业务逻辑和你想它在未来的位置，选择上述一个或更多方法。

例如，你或许在 Java 或 .NET 中有同步 REST APIs 的现存业务逻辑，但是你或许想要写新的逻辑作为响应事件触发器，部署在 Javascript 或 Python 或 Go 的 serverless 函数(或 lambdas)中。

## 6.我能使用 REST 代替 GraphQL APIs 吗？

Hasura 2.0 增加 REST APIs 的支持。Hasura 2.0 允许用户基于 GraphQL 模板创建惯用的 REST 端点。点击[查看](https://hasura.io/docs/latest/graphql/core/api-reference/restified.html#restified-api-reference)详情。

## 7.Hasura 能和授权系统集成吗？

Hasura 认为不应该限制身份验证只有特定的提供者，因此，我们让你提供自己的身份验证系统非常容易。最受喜爱的机制是通过 JWT。Hasura 可以接收来自任何标准 JWT 提供者的 JWT 令牌。例如自定义身份验证系统，Hasura 也支持身份验证 webhook，允许你阅读自定义格式的 cookies 或者令牌。我们为一些流行的身份验证提供商提供了指导。阅读[更多](https://hasura.io/docs/latest/graphql/core/auth/authentication/index.html#authentication)。

## 8.Hasura 也能自动缓存查询或数据去提升性能吗？

查询响应缓存（可以在 Hasura Cloud & Hasura EE 获得）可以通过指定启用，使用@cached 指令缓存查询。

## 9.Hasura 如何处理 ABAC,RBAC 类型的授权策略？

Hasura 通过自动发布不同 GraphQL schema 实现 RBAC，这代表正确的查询、字段和变更对这个角色是可用的。

对于 ABAC，会话变量可以被用作属性，创建使用请求属性中任何动态变量的权限规则。

## 10.Hasura 有其他安全特性，尤其是针对生产环境中的 GraphQL？

Hasura 拥有多个安全特性，可以最好地利用 GraphQL 引擎的强大力量。提供了服务水平安全、身份验证和授权，允许列表，速率和响应限制等功能。在[这儿](https://hasura.io/learn/graphql/hasura-advanced/security/)学习更多 Hasura 安全。

## 11.编译器方法如何提供优越性能？

通常，当你想到处理查询的 GraphQL 服务时，你想到在查询获取数据中有大量的解析器调用。该方法可能导致多次数据库访问，并且带有明显的相关约束。使用数据加载器进行批处理可以通过减少调用次数改善情况。

Hasura 在内部解析 GraphQL 查询，获取 GraphQL AST 的内部表示。然后，被转换为 SQL AST，并且通过必须的变换和变量形成最终的 SQL 。

> GraphQL Parser -> GraphQL AST -> SQL AST -> SQL

基于该编译器的方法允许 Hasura 为任意深度的 GraphQL 查询构成单 SQL 查询。

## 12.Hasura 如何水平和垂直缩放？

Hasura Cloud 能让你自动缩放应用，不必考虑实例、核心、内存、并发等的数量。你可以保持增加并发用户的数量和 API 调用的次数，并且 Hasura Cloud 将自动进行优化。Hasura Cloud 可以通过读副本负载均衡查询和订阅，而将所有变更和元数据 API 调用发送到主机。学习更多 Hasura 水平扩展，[这儿](https://hasura.io/learn/graphql/hasura-advanced/performance/2-horizontal-scaling/)。

## 13.如何提升慢执行 API 调用的性能？

Hasura 允许分析查询去分析慢运行调用，并且使用 Indexes（索引）提升性能。学习更多，[这儿](https://hasura.io/learn/graphql/hasura-advanced/performance/3-analyze-query-plans/)。
