---
title: "017 译 | 使用Hasura增加自定义业务逻辑"
date: 2022-03-15T21:09:45+08:00
tags: ["技术翻译", "Hasura"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

翻译自：[Adding custom business logic with Hasura](https://hasura.io/blog/custom-business-logic/)

Hasura 在 Postgres 上提供具有身份验证引擎的即时实时 GraphQL。这使得设置一个快速的安全且可扩展的 GraphQL API 变得更容易，包括开箱即用的 GraphQL 订阅！

如何增加自定义逻辑呢？

- 数据驱动逻辑
- 自带 GraphQL API
- 事件驱动业务逻辑（写优先工作流）

本文涵盖了定制和扩展 Hasura 的不同方法，所以你可以容易地增加业务逻辑，使用你最喜欢的工具或框架。

你将注意到当你考虑为应用增加新的或现存业务逻辑时，Hasura 鼓励你利用云原生模式（无状态微服务，serverless 函数，事件驱动）。

## 事件驱动逻辑

通常，在数据库层做一些业务逻辑比应用层更好（或更容易）。例如，复杂高性能聚合（假设连续时间序列聚合）或者事务存储进程。

Hasura 使得使用 Postgres 视图（预定义查询）和函数为 GraphQL API 增加新的类型和字段更容易。Hasura 也使得自定义和扩展在 GraphQL API 中生成的类型和字段更加容易。

- 视图和函数
- 计算字段
- 自定义 GraphQL schema 和字段名称（你可以容易的使用元数据 APIs 或 Hasura 控制面板做到）

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220315191343.png)

Hasura 很容易充分利用你的数据库，同时强制和重用身份授权系统，所以你不必为数据优先的任务写代码。

## 自带 GraphQL 服务：自定义业务逻辑

Hasura 提供了一个容易的方式增加业务逻辑，通过使用“remote schemas"”使用自己的 GraphQL API。

（当你需要将不是 Hasura 早已指定数据库类型引入 GraphQL 模式时，这是理想的方式。）

特定于领域的业务逻辑经常存在于微服务中，它或许与其他数据库，外部 SaaS APIs 通讯或者需要复杂应用层处理和转换。

- 远程 Schemas
- 远程连接

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220315204555.png)

Hasura 可以直接将他们集成到一个统一 GraphQL API 中 ，并且允许你通过配置“远程连接”配置不同 GraphQL 服务公开类型之间的语义关系。

## 事件驱动业务逻辑（写优先工作流）

随着微服务和无服务函数激增“后端技术栈”，事件驱动模式正成为处理日益复杂问题的规范。

用事件驱动业务逻辑可以在不影响可伸缩性的情况下更容易实现解耦和容错。

[Hasura Actions](https://hasura.io/blog/introducing-actions/)提供了一个容易的方式创建自定义 GraphQL 变更规范，并且将他们映射到微服务 APIs 或无服务函数（甚至 Postgres 函数）的事件处理器中。

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220315210053.png)

Hasura 在内部关注事件管道，包括事件捕捉，持久化和投递。

阅读更多[ Hasura actions](https://hasura.io/blog/introducing-actions/)(已翻译)！
