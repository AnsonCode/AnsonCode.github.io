---
title: "022 译 | Hasura vs WunderGraph：不同之处是什么？"
date: 2022-04-19T23:59:45+08:00
tags: ["技术翻译", "WunderGraph", "Hasura"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

翻译自：[Hasura vs WunderGraph comparison: What are the differences?](https://wundergraph.com/docs/overview/comparison/vs_hasura)

从高层次角度来看，Hasura 和 WunderGraph 都给你“所有数据的即时 GraphQL ”，正如 Hasura 所说。本文，我们将分解这两个方案，并逐个特性对比他们。

虽然我们可能有偏见，我们会尽力考虑每个解决方案的优缺点并尽可能保持客观。

我们尊重 Hasura 公司，他们为 GraphQL 社区做了很多事情。从产品的角度，我们认为两个遵循不同理念的解决方案在一些方面有重叠之处。
本文将发现两者相似和不同，并且想给你何时使用哪个方案的指导。

## 特性对比

### 所有数据的即时 GraphQL

Hasura 的主要焦点是在数据库上面生成 GraphQL API。他们支持 PostgreSQL, MS SQL Server, Amazon Aurora 和 Google BigQuery。然而，看这个文档，他们主要集中在 PostgreSQL 上。

在 APIs 方面，他也可能使用 REST, GraphQL 和 OData。最近，Hasura 增加支持将生成的 GraphQL API 转成 REST 端点。

WunderGraph 支持 PostgreSQL, MySQL, AWS RDS, AWS Aurora, AWS Aurora Serverless, MariaDB, SQLite, Microsoft SQL Server, Azure SQL, MongoDB and Planetscale, CockroachDB 也即将支持。在底层，WunderGraph 使用 Prisma 集成数据库。这意味着，无论何时数据源在 Prisma 中可用，我们就能在 WunderGraph 中支持。

在 APIs 方面，WunderGraph 支持 REST APIs 和 GraphQL ，包括带订阅功能的 Apollo Federation 。这意味着，WunderGraph 不仅支持 GraphQL 上游，而且还可以作为 Apollo Federation 的网关，联合多个其他服务的子图。

正如我们所知，Hasura 的远程 Schema 功能不支持订阅。使用 WunderGraph，支持 GraphQL 订阅，甚至多个 GraphQL 上游和 Apollo Federation。

至此，我们看到这两者有一些不同，但是两个工具提供或多或少相同的特性。

然而，早有一个大的不同。Hasura 围绕有数据库的想法构建。没有数据库，你不能创建 Hasura API。

在另一方面，WunderGraph 完全不依赖数据库。你完全可以从 REST 和 GraphQL APIs 中自由的组合你的 schema。这是因为我们的理念略有不同。

Hasura 把数据库作为每件事的中心。你从中心数据库 schema 开始，并且可以使用 APIs 扩展它。

WunderGraph 把 API 作为中心。一个 API 可能是任何事情，一个 REST API, 一个 GraphQL API, 一个或多个联合子图 或 一个 database。从这层意义上，数据库仅仅是另一种 API。

### 远程 Schema

Hasura 有远程 Schema 的概念。如果一些东西在远处，它意味着也有一个非远程的 Schema，中心数据 Schema。此外，有一个构建介于远程 Schema 字段和中心数据库关系的方式。

WunderGraph 不区远程或非远程 Schema 。反而，每个东西都是远程 Schema。这也意味着，你可以构建介于任何 API 之间的关系。你可以使用 REST API 组合 GraphQL API，并且构建这两者的关系。

正如早前提到的，Hasura Remote Schemas 有一些限制，例如，缺乏 Subscriptions 支持。

### 设置和配置

在配置方面，两个工具遵循完全不同的范例。

尽管 Hasura 可以通过 REST API 配置，配置的主要方法是他们的用户界面。从增加数据库到配置远程 schemas 和关系，所有的配置可以并且应该使用用户界面完成。这个模式提供了很好的上手体验，因为容易学习。

WunderGraph 没有为配置提供用户界面。这是因为我们不相信基础设置，像 APIs，应该使用 UI 配置。APIs 是一个进化的系统，他们总是改变并且前进。我们认为，跟踪所有正在发生的变化是十分重要的。我们认为你想要在不同的环境中操作 APIs，例如 开发，测试和生产。我们也认为，表单和按钮有太多限制对于配置复杂的 API 环境。

这是为什么我们决定反对通过用户界面配置。反而，WunderGraph 提供一个强大的 TypeScript SDK。这个 SDK 允许你非常容易的配置环境并且在 git 中存储配置。你可以为配置获得开箱即用的版本控制，你可以容易的审查和回滚，并且为不同环境使用不同分支变得可能。一旦变化在测试环境中测试，他可以合并到生产环境。WunderGraph 的设计就是为了支持这些流程。

所以，如果你是一些想要使用他们接口配置 APIs 的人，你或许倾向于 Hasura。如果你喜欢“基础架构即代码”，WunderGraph 更适合你。

### 身份验证

Hasura 不为你处理身份验证，所以你必须集成其他服务或自己构建解决方案。使用 WunderGraph，我们已经额外付出努力去构建一个令人吃惊的身份验证开发体验。我们不仅为身份验证支持 OpenID Connect，我们也生成一个完全类型安全的客户端，它知道如何使用配置的授权供应商验证身份。简要来说，你配置身份验证提供商，像 Keycloak, Auth0, Okta 等……这生成处理身份验证流的后端部分，以及处理客户端的客户端部分。所有你必须做的就是将`login()`函数连接到用户界面，并且你的登录就完成了。你可以在[文档](https://wundergraph.com/docs/overview/features/authentication)中阅读更多该特性。

### 鉴权

Hasura 通过基于角色的模型实现鉴权。此外，它有可能使用行级安全，对于一个如此接近数据库的工具，这是有意义的。权限可以使用 Hasura 面板配置。对于远程 schemas，事情有点难办。你可以使用 SDL 为每个角色定义特定的 schema 去限制访问。作为替代方案，他也可能将头部转发给远程 schema。

从我的角度来看，基于角色的授权似乎是一种可靠的解决方案，尤其当使用像 PostgreSQL 这样支持行级安全的数据库时。也就是说，远程 schema 鉴权是一种边缘情况，系统没有对此优化。

WunderGraph 对于鉴权有不同的选择。我们为鉴权集成了 OpenID Connect，意味着用户有一个开箱即用的身份。该身份不是被 WunderGraph 驱动。真实的来源是身份提供商。这意味着你可以从你的身份供应商中配置并管理用户。依赖你为用户赋予的角色和权限，他们将有不同的“claims”，当和 WunderGraph APIs 集成时可以被使用。

使用我们的 claims 注入特性，你能从身份提供商那里使用 claims 并直接将他们注入到 GraphQL 操作中。**[你可以在这里阅读更多 claims 注入的主题。](https://wundergraph.com/docs/overview/features/authorization_claims_injection)**让我们给你一个简短的示例，说明该特性的强大。

```gql
mutation (
  $name: String! @fromClaim(name: NAME)
  $email: String! @fromClaim(name: EMAIL)
  $message: String! @jsonSchema(pattern: "^[a-zA-Z 0-9]+$")
) {
  createOnepost(
    data: {
      message: $message
      user: {
        connectOrCreate: {
          where: { email: $email }
          create: { email: $email, name: $name }
        }
      }
    }
  ) {
    id
    message
    user {
      id
      name
    }
  }
}
```

当执行该操作时，代表用户创建了一篇文章。`NAME`和 `EMAIL`变量自动注入到操作中。

这意味着，如果你的数据库支持行水平安全（RLS)，例如 PostgreSQL，你可以注入用户的邮箱，并且让他执行。

此外，我们有钩子，允许你通过写一个 TypeScript 函数在执行操作前验证用户 claims。这意味着，你可以通过评估身份提供者定义的用户角色和权限限制任何 API，文件上传，变更等的访问。这给你完全的灵活性，同时让配置容易维护。

除此之外，WunderGraph 包含一个 RBAC 实现。当用户首次鉴权时，你可以使用 TypeScript 函数为他们分配角色。例如，你可以给他们不同的角色，基于他们的邮箱地址。由于 TypeScript 钩子只是 NodeJS，你也能够从外部系统或者甚至一个数据库中获取数据，并为用户分配角色。一旦角色被分配，你可以使用 `@rbac`指令 保护你的操作。

这个示例展示了仅仅使用`superadmin`角色的用户被允许通过用户邮箱删除消息。

```gql
mutation ($email: String!) @rbac(requireMatchAll: [superadmin]) {
  deleteManymessages(where: { users: { is: { email: { equals: $email } } } }) {
    count
  }
}
```

总结该章节，Hasura 和 WunderGraph 两者都提供很多方式为 APIs 实现鉴权。我做的唯一区分是 WunderGraphs 鉴权系统被设计和很多“远程 schemas”更好的工作，在另一方面当你想使用 PostgreSQL 原生特性时，Hasura 可以更好的工作。

### API 安全

### 输入校验

### 自定义 API

### 模拟

### 本地开发

### 监控和可观测性

### 缓存

### 实时订阅

### 文件上传

### 客户端和服务端紧密耦合

### 生成客户端

### Authentication-Aware Data Fetching

### CSRF 保护

### 跨 API 连接

### WunderHub——API 包管理器

## 技术对比

## 为什么 WunderGraph 是一个伟大的 Hasura 替代品？

## 使用 WunderGraph 上手
