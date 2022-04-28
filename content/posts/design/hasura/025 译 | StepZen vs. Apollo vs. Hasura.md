---
title: "025 译 | StepZen vs. Apollo vs. Hasura"
date: 2022-04-28T23:59:45+08:00
tags: ["技术翻译", "StepZen", "Hasura", "GraphQL"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

翻译自：[StepZen vs. Apollo vs. Hasura](https://stepzen.com/blog/stepzen-hasura-apollo)

后端团队正在努力为前端团队构建和部署 APIs，以便快速构建应用。他们被前端团队的请求驱动，去提供更好的性能以及增强的开发体验。

在[为什么后端开发者也应该爱上 GraphQL](https://thenewstack.io/why-backend-developers-should-fall-in-love-with-graphql-too/)，Anant Jhingran 总结了一些我们在与负责构建和发布这些 APIs 开发者的对话中听到的一些担忧：

- 我和我的团队都不具备构建 GraphQL APIs 的全部技能。
- GraphQL 代表了一种新的妥协 vector。我们锁定了 SQL 注入，然后是 REST，现在我必须担心其他事情？
- 当前在 GraphQL 的工具是的混合业务逻辑和应用逻辑更容易，然而这是反范式的。
- 我在 REST 端点投入数年，我需要扔掉这些从新开始吗？

这些担心通常是合理的，但是任何新技术都通常如此，实际情况可能不像它看起来那样糟糕。此外，使用 GraphQL 提供服务是一个动态软件开发领域，开源工具和像 Apollo, Hasura, 和 StepZen 这样的公司，正开始解决这些问题。

这让我想到一个经常被问的问题：StepZen 如何与 Hasura 或者 Apollo (Server)比较。

## Hasura vs. StepZen

Hasura 是一个 WEB 服务，它连接新的或现存的数据库，快速创建一个 GraphQL API。如果你的出发点是数据库，它是一个很棒的产品。Hasura 提供数据库数据到 GraphQL 的快速转换，通过它的易用的 web UI，托管 GraphQL 端点，以及一个慷慨的免费套餐。

在 StepZen 中，在 GraphQL 模式中，你可以使用几行代码或 [GraphQL 指令 @dbquery](https://stepzen.com/docs/connecting-backends/how-to-connect-a-mysql-database) 为数据库创建 GraphQL API ，而不是写很多行代码或者通过 UI 点击。

```gql
type Article {
  id: ID!
  title: String!
  description: String
}

type Query {
  articles(username: String!): [Article]
    @dbquery(type: "mysql", table: "articles")
}
```

此外，我们和很多处理过异构后端的开发者，不仅数据库，REST 后端，私有或公开的 SaaS REST APIs 甚至不断增加的 GraphQL 后端，交谈。StepZen 使得组合这些各种后端很容易。

```gql
type Author {
  name: String!
}

type Article {
  id: ID!
  title: String!
  description: String
  authorID: ID!
  author: Author
    @materializer(
      query: "userByID"
      arguments: [{ name: "id", field: "authorID" }]
    )
}

type Query {
  articles(username: String!): [Article]
    @dbquery(type: "mysql", table: "articles")
  authorByID(id: ID!): Author @rest(endpoint: "https://dev.to/api/users/$id")
}
```

在 StepZen 中，创建 API 的代码与你需要写的运行，管理和维护 API 的代码大致相同。你可以发布一个极佳的，安全的和性能良好的 GraphQL API，而不需要写更多代码。API 应该为后端扩展，维护一个固定 IP 地址，并且自动修补最新的安全漏洞。StepZen 云，拥有 99.99%的可用性，为你做所有这些。

## Apollo vs. StepZen

Apollo 提供 GraphQL 功能好几年了，并且有成熟的工具集让你顺利访问 GraphQL API。使用 Apollo，你写连接后端的代码。所以，你必须写自定义解析器代码，而不是声明式连接数据库或 API。

在 Apollo 文档的示例中，你首先创建代码去创建爱你 GraphQL 模式——客户端可以执行名称为 `books`的查询，Apollo 服务返回一个空数组或多个`Books`。

```js
const { ApolloServer, gql } = require("apollo-server");

const typeDefs = gql`
  type Book {
    title: String
    author: String
  }

  type Query {
    books: [Book]
  }
`;
```

然后这个代码定义数据集。

```js
const books = [
  {
    title: "The Awakening",
    author: "Kate Chopin",
  },
  {
    title: "City of Glass",
    author: "Paul Auster",
  },
];
```

然后，创建解析器去定义获取模式中定义类型的技术。

```JS
const resolvers = {
  Query: {
    books: () => books,
  },
};
```

然后创建 Apollo Server 实例。

```JS
const server = new ApolloServer({ typeDefs, resolvers });

server.listen().then(({ url }) => {
  console.log(`🚀  Server ready at ${url}`);
});
```

在 StepZen 中，通过向模式（SDL 文件）添加几行构建图，对于 REST，充分发挥 GraphQL 自定义指令[@rest ](https://stepzen.com/docs/connecting-backends/how-to-connect-a-rest-service)的优势，去连接 REST 后端。

```gql
type Article {
  id: ID!
  title: String!
  description: String
}
type Query {
  articles(username: String!): [Article]
    @rest(endpoint: "https://dev.to/api/articles?username=$username")
}
```

在 GraphQL 中真的成功意味着，GraphQL API 必须可扩展部署，保护，高性能以及处理变化。当然如果你使用 Apollo 你可以做到所有，你将必须学习技能，导入新的库，并且部署更多架构去实现这些。

使用 StepZen，你声明式构建 GraphQL 层，就像你构建声明式构建数据库，不需要写应用层代码。你写的创建 API 的代码与你需要写去运行，管理和维护 API 的代码是相似的。在 StepZen 上运行 GraphQL，所以你必须要管理架构，你获得内建性能，花费和可靠性优化。

## 概述

构建其他人（甚至你）使用的 APIs，最好使用 GraphQL。GraphQL 帮你为用户交付更易懂的 API。将功能缝合在一起帮你高效混合和匹配来自后端的响应。GraphQL 片段帮助抽象和细节连续性，并且具有自文档能力的内省节省你的时间。

StepZen 的使命是让开发者获得优秀的，安全的以及高性能的 GraphQL API ，不需要写和维护更多代码，同时 API 应该是可伸缩的，为后端维护固定 IP 地址，并且自动修补最新的安全漏洞。StepZen 云，拥有 99.99%的可用性，为你做所有这些，不需要你做任何事情。

为了帮你开始，我们提供预构建 SaaS APIs，和跨数据库和 REST 的内省，让你通过加入一些声明式代码构建图，然后坐下来，享受一个高性能 GraphQL API。在 GraphQL Studio 中，核查[ StepZen 文档](https://stepzen.com/docs/design-a-graphql-schema)和预构建[ SaaS APIs](https://graphql.stepzen.com/)。我们总是想要看到你正在构建什么，并且在 Discord 中回答问题。
