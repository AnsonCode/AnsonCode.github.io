---
title: "002【翻译】GraphQL到SQL高性能架构"
date: 2022-01-30T16:32:14+08:00
tags: [”技术翻译“, "Hasura"]
categories:
  - "研发提效"
featured_image:
description:
draft: true
---

翻译自：[Architecture of a high performance GraphQL to SQL engine](https://hasura.io/blog/architecture-of-a-high-performance-graphql-to-sql-server-58d9944b8a87/)

更新：在旧金山的 [2019 GraphQL 峰会](https://www.youtube.com/watch?v=HOKMJkBYaqQ)上，Tanmai Gopal（Hasura 联合创始人\CEO）讨论了更多细节。

Hasura 平台的数据微服务提供了一个 HTTP API 在权限安全的方式下 使用 GraphQL 或 JSON 去查询 Postgres。

你可以在 Postgres 中运用外键约束通过单个请求查询分层数据。例如，你能运行查询去获取“专辑”和他们所有的“歌曲”（提供的“歌曲“表需要有指向“专辑”表的外键）：

```graphql
{
  album(where: { year: { _eq: 2018 } }) {
    title
    tracks {
      id
      title
    }
  }
}
```

正如你所猜测的，这个查询可以遍历表到任意的深度。这个查询接口接合了权限，能够不写任何后端代码让前端应用程序查询 Postgres。

这个 api 被设计为最快的响应时间，并且处理大量吞吐（每秒请求），但是轻量的资源消耗（低 cpu 和内存使用）。我们将讨论让我们实现它的架构选型。

### 查询生命周期
