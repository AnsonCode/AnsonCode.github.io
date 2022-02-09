---
title: "003【翻译】GraphQL到SQL高性能架构"
date: 2022-02-01T16:32:14+08:00
tags: ["技术翻译", "Hasura"]
categories:
  - "研发提效"
featured_image:
description:
draft: false
---

翻译自：[Architecture of a high performance GraphQL to SQL engine](https://hasura.io/blog/architecture-of-a-high-performance-graphql-to-sql-server-58d9944b8a87/)

更新：在旧金山的 [2019 GraphQL 峰会](https://www.youtube.com/watch?v=HOKMJkBYaqQ)上，Tanmai Gopal（Hasura 联合创始人\CEO）讨论了更多细节。

Hasura 平台的数据微服务提供了一个 HTTP API， 在权限安全的方式下 使用 GraphQL 或 JSON 去查询 Postgres。

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

数据微服务查询经过这几个阶段：

- 1.Session resolution：请求到达授权密钥网关（如果有的话），增加`user-id`和`role`头，然后代理请求到数据服务。
- 2.查询解析：数据服务接收请求，解析头部去获取`user-id`和`role`，解析请求体为 GraphQL AST
- 3.查询验证：核查查询是否是语义正确的，然后强制执行角色定义的权限
- 4.查询执行：合法的查询被转化为一个 SQL 语句，并且在 Postgres 中执行。
- 5.响应生成：来自 Postgres 的结果被处理，并且发送至客户端（如果需要的话，网关增加 gzip 压缩）

### 目标

大概需求如下：

- 1.HTTP 栈应该只增加很少的开销，并且应该能够处理大量高吞吐的并发请求
- 2.快速的查询转换（GraphQL 到 SQL）
- 3.编译的 SQL 查询应该对 Postgres 有效
- 4.Postgres 的结果必须被有效的返回

### 处理 GraphQL 请求

以下是获取 GraphQL 查询所需数据的各种方法。

#### 原生解析器

GraphQL 查询执行通常涉及为每个字段执行一个解析器。在示例查询中，我们将调用函数去查询 2018 年发布的唱片，然后对于每一个唱片，我们将调用函数去获取歌曲，这是典型的 N+1 查询问题。随着查询的深度大量的查询将指数倍增长。

在 Postgres 中执行的查询如下所示：

```sql
SELECT id,title FROM album WHERE year = 2018;
```

这给我们所有的唱片。让这大量的唱片返回是 N。对于每一个唱片，我们将执行这个查询（所以是，N 查询）：

```sql
SELECT id,title FROM tracks WHERE album_id = <album-id>
```

这将总共 N+1 个查询，去获取所有需要的数据。

#### 批量查询

像 dataloader 这样的项目目的是通过批量查询解决 N+1 查询问题。请求的数量不再依赖结果集合的尺寸大小，他们将替代为依赖在 GraphQL 查询中节点的数量。本例中的示例查询将需要 2 次 Postgres 查询去获取必须数据。

这个在 Postgres 上的查询执行情况如下：

```sql
SELECT id,title FROM album WHERE year = 2018
```

这给我们所有的唱片。去获取所有需要唱片的歌曲。

```sql
SELECT id, title FROM tracks WHERE album_id IN {the list of album ids}
```

这里总共有 2 个请求。我们避免发起请求为每一个唱片获取歌曲信息，而且代替使用 where 语句在单个查询张浩工获取所有需要唱片的歌曲。

#### 连接

Dataloader 被设计用于跨不同的数据源工作，不能利用单数据源的特性。在我们的示例中，我们唯一的数据源是 Postgres，和所有关系数据库一样， Postgres 提供了一个方式在单查询中从几个表中去收集数据，也就是所谓的连接。我们能决定 GraphQL 查询需要的这些表，使用连接生成一个 SQL 查询去获取所有数据。所以，GraphQL 查询需要的数据能从一个 SQL 查询中获取。这些数据在发送到客户端之前已经进行了适当转化。

查询将如下所示：

```sql
SELECT
  album.id as album_id,
  album.title as album_title,
  track.id as track_id,
  track.title as track_title
FROM
  album
LEFT OUTER JOIN
  track
ON
  (album.id = track.album_id)
WHERE
  album.year = 2018
```

这将给我们如下所示的数据：

```
album_id, album_title, track_id, track_title
1, Album1, 1, track1
1, Album1, 2, track2
2, Album2, NULL, NULL
```

这些数据必须被转换为如下结构的 JSON 响应。

```json
[
  {
    "title": "Album1",
    "tracks": [
      { "id": 1, "title": "track1" },
      { "id": 2, "title": "track2" }
    ]
  },
  {
    "title": "Album2",
    "tracks": []
  }
]
```

#### 优化响应生成

我们发现处理请求的大部分时间被花费在转换函数上（转换 sql 结果到 json 结果）。尝试一些方法去优化转换函数之后，我们决定通过把转换推给 Postgres，来移除这些函数。Postgres 9.4 （在第一个数据微服务发布的时候发布）增加了 json 聚合函数，帮助我们推送转换到 Postgres。这个生产的 SQL 将如下所示：

```sql
SELECT json_agg(r.*) FROM (
  SELECT
    album.title as title,
    json_agg(track.*) as tracks
  FROM
    album
  LEFT OUTER JOIN
    track
  ON
    (album.id = track.album_id)
  WHERE
    album.year = 2018
  GROUP BY
    album.id
) r
```

这个查询的结果将有一列和一行，这个值也被发送到客户端不需要进一步的转换。根据我们的压测，这个方法比 Haskell 转换函数大概快 3-6 倍。

#### 预处理语句

根据这个查询嵌套级别和使用的 where 条件，生成的 SQL 语句可能很大和复杂。通常，任何前端应用有一系列带有不同参数的重复查询。例如，下面的查询将执行 2017 年而不是 2018 年。预处理语句最适用于这些使用的用例，例如当你有复杂的 SQL 语句，他们仅仅在参数上有一些变化，其他部分是重复的。

因此，第一次执行 GraphQL 查询时：

```gql
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

我们准备 SQL 语句而不是直接执行它，所以这个生成的 SQL 将如下（注意这个$1 ）：

```sql
PREPARE prep_1 AS SELECT json_agg(r.*) FROM (
  SELECT
    album.title as title,
    json_agg(track.*) as tracks
  FROM
    album
  LEFT OUTER JOIN
    track
  ON
    (album.id = track.album_id)
  WHERE
    album.year = $1
  GROUP BY
    album.
```

接下来执行这个预生成语句：

```sql
EXECUTE prep_1('2018');
```

当 graphql 查询变更为 2017 年时，我们只需要直接执行预处理语句：

```sql
EXECUTE prep_1('2017');
```

依据 GraphQL 查询的复杂性，这大概提升 10-20%。

#### Haskell

Haskell 适用的多种理由：

- 高性能编译语言（[这里](https://stackoverflow.com/questions/35027952/why-is-haskell-ghc-so-darn-fast)）
- 非常高效的 http 栈（[wrap，wrap 架构](http://www.aosabook.org/en/posa/warp.html)）
- 之前有很多针对该语言的经验

#### 总结

所有这些优化放在一起会带来一系列性能的好处。这里有一些 Hasura 架构与 Prisma 和 Postgraphile 的对比。

##### 警告

更新在 2018 年 12 月 13 日：这个文章第一次发布于 2018 年 4 月，这个压力测试现在被更新了。下面介绍的所有项目均已进行了性能优化，我们将很快更新这个文章的最新数据。

![Hasura’s API与Postgraphile和Prisma的对比](https://gitee.com/caoyanbin/picgo/raw/master/img/20220206132921.png)

事实上，当与直接查询 postgres 对比， 他的内存占用低，延迟可忽略，对于很多服务端代码用例，你甚至可以使用 GraphQL API 去替换 ORM。

##### 压力测试

设置细节：

- 1.一个 8GB RAB，i7 笔记本
- 2.Postgres 运行在相同的机器上
- 3.[wrk](https://github.com/wg/wrk) 被用于作为压力测试工具，对于不同的请求，我们尝试“最大化”每秒的请求
- 4.一个单实例 Hasura 引擎被查询
- 5.连接赤子大小：50
- 6.数据集：[chinook](https://github.com/lerocha/chinook-database)

**查询 1:tracks_media_some**

```gql
query tracks_media_some {
  tracks(where: { composer: { _eq: "Kurt Cobain" } }) {
    id
    name
    album {
      id
      title
    }
    media_type {
      name
    }
  }
}
```

- 每秒请求：1375req/s
- 延时：17.5ms
- CUP：～ 30%
- RAM：~30MB (Hasura) + 90MB (Postgres)

**查询 2:tracks_media_all**

```gql
query tracks_media_all {
  tracks {
    id
    name
    media_type {
      name
    }
  }
}
```

- 每秒请求：410req/s
- 延时：59ms
- CUP：～ 100%
- RAM：~30MB (Hasura) + 130MB (Postgres)

**查询 3:album_tracks_genre_some**

```gql
query albums_tracks_genre_some {
  albums(where: { artist_id: { _eq: 127 } }) {
    id
    title
    tracks {
      id
      name
      genre {
        name
      }
    }
  }
}
```

- 每秒请求：1029req/s
- 延时：24ms
- CUP：～ 30%
- RAM：~30MB (Hasura) + 90MB (Postgres)

**查询 4:album_tracks_genre_all**

```gql
query albums_tracks_genre_all {
  albums {
    id
    title
    tracks {
      id
      name
      genre {
        name
      }
    }
  }
}
```

- 每秒请求：328req/s
- 延时：73ms
- CUP：～ 100%
- RAM：~30MB (Hasura) + 130MB (Postgres)

##### 尝试它！

如果你没有运行 Hasura，使用 [Hasura 云](https://hasura.io/docs/latest/graphql/cloud/getting-started/index.html#utm_source=Hasura%20Blog&utm_medium=Blog%20Article&utm_campaign=Project%20Cloud&utm_content=introducing%20actions)去设置一个 GraphQL 工程，读这个[文档](https://hasura.io/docs/latest/graphql/cloud/index.html)去开始吧。

如果你早已运行了 Hasura，确保你的版本是 1.2 及以上，你可以出发了！
