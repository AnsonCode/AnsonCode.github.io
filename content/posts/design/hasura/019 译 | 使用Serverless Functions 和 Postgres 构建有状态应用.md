---
title: "019 译 | 使用Serverless Functions 和 Postgres 构建有状态应用"
date: 2022-03-23T01:27:45+08:00
tags: ["技术翻译", "Hasura"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

翻译自：[Building Stateful Apps with Serverless Functions and Postgres](https://hasura.io/blog/building-stateful-apps-using-serverless-postgres-and-hasura/)

![How should your serverless functions interface with a database to manage application state?](https://gitee.com/caoyanbin/picgo/raw/master/img/20220316215809.png)

本文探索当在无服务函数中写业务逻辑时，处理应用状态的不同的方式。当谈论这些不同的方法时，我们将主要使用 Postgres 作为数据库，并且经常使用 Hasura 作为介于无服务函数和 Postgres 数据库之间的粘合剂。然而，本文的观点是通用的，并且可以应用到你最喜欢的数据库解决方案上。

- 方法#1：直接从无服务函数连接到 Postgres 上
- 方法#2：在无服务函数中使用 BaaS APIs
- 方法#3：在 Postgres 事件中使用丰富的事件负载触发无服务函数
- 方法#4：组合上述方法并且使用 Postgres 编排工作流与驱动状态转变的无服务函数

## 介绍

Serverless 平台不允许开发者编写维持长期状态的代码。这是因为 Serverless 运行时按需启动，并且生命周期很短，例如：在请求完成时释放所有的资源。

这是一些示例，你不能用写普通 API 服务相同的方式做的事情，之前可能认为理所当然：

1. 直接写入磁盘，像保存文件到文件夹
2. 保持跨多个请求信息的全局变量，例如一个跟踪该 API 服务已经精力多少次请求的计数器
3. 一个 API 服务维持数据库的连接池 ，甚至在没有请求时

本文，我们探索一些真实世界用例的模式，将在使用无服务函数构建下一个应用中帮到你。让我们一个接一个的谈到他们！

## 1.直接连接 Postgres

对于服务端应用最常用的存储应用状态的方式是使用数据库。然而，直接从无服务函数原生连接到数据库不是明智的决定，因为当你突然调用 1000 无服务函数时，数据库不能突然扩容创建 1000 数据库连接。

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220316225409.png)

这儿有一些从无服务函数连接到 Postgres 去读写数据或运行数据库事务的最佳实践：

1. 跟着供应商的最佳实践创建数据库连接，帮助 serverless 函数通过”调用“（类似连接池）复用数据库连接

   - [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
   - [Google Cloud Functions](https://cloud.google.com/functions/docs/sql#best_practices_for_cloud_sql_in_cloud_functions)
   - [Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/manage-connections?tabs=csharp)

2. 在函数开始时，创建完整的状态

   - 这是为了避免对数据库执行多次调用，这可能造成性能开销以及运行开销。Gwen Shapira 在博文： https://thenewstack.io/managing-state-in-serverless/
     中谈论了更多这种数据库访问模式。

3. 尽可能多的使用无 ORM 数据层

   - 这是因为使用 ORMs，你首先需要定义数据模型，并且你将必须为每个函数做这些。这可能意味着写很多样板代码，并且保持他们与底层数据库同步。当你做这些时，你将 意识到写一个整体的 API 服务比写彼此独立的 serverless 函数更好。
   - 考虑使用好的 SQL 客户端或者像 [massivejs](https://massivejs.org/) 一样的无 ORM 客户端

4. 考虑为数据库使用外部连接池代理，比如 pgBouncer
   - 如果你正在使用 DB 事务，然后牢记数据库并发限制。事务花费巨大并且不容易扩展。依赖于你的负载，或许必须充分利用外部连接池，例如 [pgBouncer](http://www.pgbouncer.org/)。事实上，DigitalOcean 通过内建连接池提供可管理的 Postgres 数据库。

## 2.使用 BaaS APIs

管理状态最容易的方式是不自己管理它？

当处理持久化状态时，后端即服务(BaaS)解决方案抽象了一些复杂的性能和并发问题。你在 serverless 函数中的代码使用了 HTTP APIs(无状态)并且把 BaaS 解决方案当做黑盒。BaaS 实际上自己处理管理和持久化状态，即使是成千的并发代码读、写或者修改数据。

这儿有一个 BaaS 服务的示例，你的 serverless function 或许会用到：

- S3 风格 APIs 去读、写对象或文件类型数据。所有主要的云供应商现在提供 S3 API。
- Hasura GraphQL APIs 从 Postgres 中读、写数据
- [Algolia](https://www.algolia.com/) 去读写一个搜索索引和数据库

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220322224011.png)

使用 BaaS 的好处之一是你的函数看上去好像完全无状态。考虑[下面的代码](https://github.com/hasura/graphql-engine/tree/master/community/sample-apps/serverless-etl)，使用了 Algolia 输入一个搜索索引：

```js

const algoliasearch = require('algoliasearch');

exports.function = async (req, res) => {
  const data = req.body.book;

  var client = algoliasearch(process.env.ALGOLIA_ID, process.env.ALGOLIA_KEY); //from env vars
  var index = client.initIndex('book_index');

  index.addObjects([data], function(err, content) {
    if (err) {
      console.error(err);
      res.json({error: true, data: err});
      return;
    } else {
        res.json({error: false, data: content});
    }
};

```

正如你看到的，该函数本质上向完成繁重状态工作的 BaaS 发起了一个 API 调用。

## 3.使用”丰富“的 事件负载通过 Postgres events 触发 serverless functions

事件是云供应商为 serverless 函数提供资源并运行它的底层机制，通过从事件负载中向函数传递参数。

如果事件负载包含 serverless function 运行的所有数据和必须上下文，然后你不必从外部源中读取数据。

这为我们带来三个写需要状态的 serverless function 的方法。你可以从数据库（像 Postgres ）捕获事件，然后使用他们触发 serverless function。

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220322225209.png)

然而，为了在真实环境中发挥作用，这些事件需要包含所有关联数据和相关上下文。只有 id 或者操作 (insert/update/delete) 细节明显不够。

![A detailed event payload is more useful](https://gitee.com/caoyanbin/picgo/raw/master/img/20220322225435.png)

在我们使用 Postgres 的案例中，事件负载通常应该包含：

1. 行数据
2. （通过其他表连接的）行的关联数据
3. 应用用户会话信息，若可以

Hasura 提供一个在 Postgres 上开箱即用的[事件系统](https://hasura.io/event-triggers/)，使它更容易携带更多如上所述的上下文和细节负载触发事件！

## 4.Postgres 和 serverless functions 在工作流中充当一个状态机

很多复杂的应用在后端拥有工作流或通过状态机转换的流程。思考一个使用下列数据模型的食物订单应用：

![Order table](https://gitee.com/caoyanbin/picgo/raw/master/img/20220323010812.png)

整个订单工作流，包含重要的业务逻辑步骤像校验，支付，审批，派送代理分配，可以认为是一系列对 `order` 表状态的转换。

![Orchestrating a workflow using Postgres and serverless functions](https://gitee.com/caoyanbin/picgo/raw/master/img/20220323011235.png)

在订单模型的每个字段可以被特定 serverless function 更新，它运行复杂的业务逻辑发起适当的转换，并且使用更多信息修改状态。

这个方法然后变成 1/2/3 方法的混合体：

- 在数据库事件中触发 serverless function（更新特定列触发特定 serverless function）
- 当通过#1 和#2 方法运行他们的业务逻辑时，serverless function 可以前往并更新数据库的状态

该想法与 3factor 架构模式相近（[阅读更多](https://hasura.io/event-driven-programming/)关于使用事件驱动编程构建 3factor 应用的内容）

## 结论

使用 serverless function 构建有状态应用对很多用例来说不难。基本的思路是将繁重的工作委托给专家系统。使用一些这里介绍的模式，开发者可以发挥在 serverless funciton 中写业务逻辑的最大价值。
