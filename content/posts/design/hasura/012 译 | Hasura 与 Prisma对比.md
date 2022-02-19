---
title: "012 译 | Hasura 与 Prisma对比"
date: 2022-02-19T22:22:45+08:00
tags: ["技术翻译", "Hasura"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

翻译自：[Hasura vs Prisma](https://hasura.io/blog/hasura-vs-prisma-9ffc7271eda8/)

过去几周，很多人问我 Hasura 与 Prisma.js 的区别。

在我们用户如何使用 Hasura 的帮助下，我总结了以下几点，并罗列了一些区别。这些都是定性的观点，由于时间问题我不能面面俱到，但我将不断完善它。在此期间，如果你有任何问题，请随时在 [Twitter](https://twitter.com/tanmaigo?lang=en) 或 [discord](https://discord.gg/vBPpJkS) 上联系我。

表面上看，Hasura 和 Prisma 在 Postgres 上 提供 GraphQL API。然而，从 Hasura 在你技术栈的理想位置和他们如何为项目增加价值方面来看，是有重大区别的。

## Hasura 是面向前端应用的 GraphQL

Prisma 是一个针对 [GraphQL](https://hasura.io/graphql/) (或 REST)服务和非前端应用的 GraphQL ORM，更像 JDBC, SQL Alchemy, ActiveRecord 等的替代品。

作为基于数据库之上的 GraphQL API，Prisma 不能直接被前端应用使用。

![](https://hasura.io/blog/content/images/downloaded_images/hasura-vs-prisma-9ffc7271eda8/1-EMkrtmQxqh1S0K5BXhIZ0g.png)

也就是说，如果你需要，你可以使用 Hasura 作为你微服务的 GraphQL API。尤其，如果你正在寻找可扩展和强健的订阅。

## 增加业务逻辑

Prisma 为 nodejs/typescript/go 提供了客户端和绑定，让你去构建自己的业务逻辑，与数据库对话。因为 Prisma 是一个 ORM，而不是使用 nodejs 和 knex/sequelize 构建 GraphQL 服务，你可以使用 nodejs 和 Prisma 客户端与 Prisma 服务对话，进而与数据库对话。你的代码可以像通常那样执行业务逻辑，就像从头构建 GraphQL 服务时一样。

**使用 Hasura**，有两种写业务逻辑的方式：

1. 用任何语言或框架编写自己的 GraphQL 服务，使用你最喜欢的 ORM（如果你需要与数据库对话），并且使用 Hasura 的 GraphQL 合并 GraphQL 服务，创建一个统一的 GraphQL 端点。

2. 写一个 webhook，理想情况下作为 serverless 函数，无论数据库何时发生变更，它都被触发。注意，你可以在数据库中为任何变化设置变化捕捉，不仅是 GraphQL 变更时。

Hasura 目标是**为你节省写 CRUD 和可扩展实时后端的繁重的工作**，而不影响你如何写业务逻辑。

![Adding business logic with Hasura](https://hasura.io/blog/content/images/downloaded_images/hasura-vs-prisma-9ffc7271eda8/1-cIboOGvlbRJK_tBKS0DJaA.png)

## 身份验证、访问控制和 GraphQL 安全

由于 Hasura 是一个针对应用的 GraphQL API，Hasura 拥有权限控制和身份验证特性，让你能[安全](https://hasura.io/graphql/security/)访问数据库。我们仔细地设计权限控制系统使其能处理简单到复杂的用户场景。从用户能够 CRUD 他们自己的数据到企业级多租户多角色应用。

Hasura 可以集成任何身份验证系统，包括 JWT 系统像 [Auth0](https://auth0.com/), [firebase](https://firebase.google.com/docs/auth/), [forgerock](https://www.forgerock.com/)。

**GraphQL 安全：**我们早已支持限制响应大小去避免爬虫和拒绝服务，并且我们正在根据用户优先级排序增加更多安全特性。在我们的路线图上，还有有很多其他特性，可以帮助用户安全的使用 GraphQL。

## 性能和开销

在架构中，Hasura 不使用解析器或者数据加载器。而是，Hasura 将[ GraphQL 查询](https://hasura.io/learn/graphql/intro-graphql/graphql-queries/)编译为单语句，非常可读的 SQL 查询，并且包含权限控制语句（`where user_id = <session-user-id>`）。这使得 Hasura 非常快，尤其是对于复杂的查询。在这儿阅读更多关于高性能架构：[（已翻译 003 译 | GraphQL 到 SQL 高性能架构）](https://hasura.io/blog/architecture-of-a-high-performance-graphql-to-sql-server-58d9944b8a87)

由于 Hasura 实际上是一个针对网络服务的编译器，Hasura 的资源占用特别低，尤其与 Prisma 运行所需要的 JVM 对比。

在 100MB RAM 以内，Hasura 可以很好的满足几千条复杂 GraphQL 查询每秒。在运行 Postgres 和 Hasura 的情况下，构建一个可以扩展 1000 用户并发的业余项目，需要花费 5 美元/月。由于它的镜像尺寸和资源消耗，Prisma 不能免费运行在 Heroku 和 Zeit 上。

相比之下，Prisma 使用数据加载器方法，仅仅为小查询做了优化，并且不能有效处理复杂查询。

## 简单缩放和自动缩放

Hasura 被设计完全保持无状态化，并且没有主从架构。这让它可以无痛缩放 Hasura：

1. 纵向缩放：增加 CUP/RAM
2. 水平缩放：增加副本数量
3. 自动缩放：在你最喜欢的容器提供商上设置自动缩放，依赖流量从 1 到 n 扩展

水平扩展 Prisma 是更乏味的，并且不像改变配置变量去增加副本或者在云供应商面板上开启自动扩展（或者重声明配置文件）那样简单。

## 无缝支持 Postgres

Hasura 被设计 Postgres 优先。Postgres 是一个非常成熟的数据库，并且 Hasura 持续确保你可以以一种你想要的方式，使用一个现存的 Postgres 实例，而不需要对你团队的工作流做任何改变。Hasura 支持 Postgres 丰富的类型系统和操作，创建视图的能力，使用自定义 SQL 函数，并且甚至物理化了视图！

## 自动管理 GraphQL schema

Prisma 只能让你通过 GraphQL SDL 管理数据库，它不支持很多应用最终需要的 UP/DOWN/自定义数据库迁移。例如，在 GraphQL SDL 驱动的数据库系统中：创建 User GraphQL 类型等同于创建 user 表。

Hasura 不能通过 GraphQL schema 定义语言驱动你的数据模型。相反，Hasura 帮你使用你的数据库，并且提供一个简单的 UI 管理你的数据库模型，然后基于额外提供的元数据生成 GraphQL schema。你可以使用你自己的数据库迁移系统或者 Hasura 的迁移系统（灵感来自 ActiveRecord ala Rails）去建模你的数据库。
