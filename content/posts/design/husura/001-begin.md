---
title: "001 【研发提效】后端开发神器——Hasura实战教程"
date: 2022-01-01T11:09:15+08:00
tags: []
categories: [""]
featured_image:
description:
draft: false
---

### 前言

如果你希望为你的数据库获得一个开箱即用的 GraphQL 服务，其他工具可能更适合你的情况：Hasura, Postgraphile

Hasura GraphQL 引擎 能够根据数据库(数据表) 自动生成 GraphQL schema，并且处理 GraphQL 查询, 订阅 和 变更。

#### Schema 生成

我们只需要跟踪表或视图，并且创建他们之间的关联关系，Hasura 引擎将自动生成 GraphQL schema。

##### 表

跟踪表时将自动生成如下内容：

- 关于该表的 GraphQL 类型定义
- 查询：支持 where（条件搜索）、oreder_by（排序）、limit 和 offset（分页）的查询
- 订阅：支持 where（条件搜索）、oreder_by（排序）、limit 和 offset（分页）的*订阅*
- 新增：支持冲突更新的单条新增；支持批量新增；
- 更新：支持 where 条件的批量更新
- 删除：支持 where 条件的批量删除

##### 表/视图（Tables/Views）

跟踪表时将自动生成如下内容：

- 定义：关于该表的 GraphQL 类型定义
- 查询：支持 where（条件搜索）、oreder_by（排序）、limit 和 offset（分页）的查询
- 订阅：支持 where（条件搜索）、oreder_by（排序）、limit 和 offset（分页）的*订阅*

- 新增：支持冲突更新的单条新增；支持批量新增；
- 更新：支持 where 条件的批量更新
- 删除：支持 where 条件的批量删除

* 视图不支持变更，因此只具有表的前 3 条功能

##### 关系（Relationships）

当在 Hasura 引擎中创建表/视图与其他表/视图关系后，引擎自动完成下述内容：

- 在表/视图中增加对象类型的字段，方便通过嵌套的方式查询数据
- 为嵌套对象增加 where 和 order_by 语法，便于过滤和排序

##### 解析器（Resolvers）

Hasura 不需要任何手写任何解析器，它自身就是一个编译器，能够将 GraphQL 查询编译成高效的 SQL 查询语句。

Hasura 针对 SQL 查询做了优化，能够充分利用 SQL 的功能，实现高效的查询。

##### 元数据（Metadata）

Hasura 将 schema 生成所需要的全部信息 以元数据的方式存储 在“目录”中，它实际上是元数据数据库中的一个特殊 Postgres schema。

#### 元数据目录

元数据目录是 Hasura 系统内部的表，被用于管理数据库状态和 GraphQL schema。

Hasura 引擎使用元数据生成能够被不同客户端访问的 GraphQL API。

Hasura 引擎将元数据存储在 Postgres 元数据库中，默认和业务数据所在位置相同。

初始化的时候，Hasura 在元数据库中创建一个叫做`hdb_catalog`的 schema，并且初始化一些表，详情见下文。

##### hdb_catalog schema

Hasura 创建它的目的是为了管理系统内部的状态。

当使用 Hasura 面板或 API 创建或更新表/权限/关系时，Hasura 将捕获该信息，同时将信息存储到相应的表中。

下面的表是 Hasura 用到的：

##### hdb_table table

`hdb_table`存储所有表/视图被创建或追踪的相关信息.

<!-- 当使用 Hasura 面板或 API 创建或跟踪表/视图时， -->

![hdb_table](https://hasura.io/docs/latest/_images/hdb_table1.jpg)

- table_schema:
- table_name:
- is_system_defined:真代表由系统内部创建；假代表由用户创建

##### hdb_relationship table

`hdb_relationship`存储表/视图间的关联信息。

![hdb_relationship](https://hasura.io/docs/latest/_images/hdb_relationship1.jpg)

- table_schema:
- table_name:
- rel_name:关系的名称
- rel_type：权限类型（insert/select/update/delete）
- perm_def：关系定义的相关信息

```
{
  "foreign_key_constraint_on": "user_id"
}
```

- comment:关系的备注
- is_system_defined:真代表由系统内部创建；假代表由用户创建

##### hdb_permission table

`hdb_permission`存储与表或视图权限控制规则相关的信息。

- table_schema:
- table_name:
- role_name:使用该权限的角色名称
- perm_type：权限类型（insert/select/update/delete）
- perm_def：权限定义的相关信息

当一个请求携带上述角色访问上述表时，引擎将先验证被用户请求的字段是否有访问权限。只要请求通过验证，这个 `filter`字段定义的过滤器将返回适当的结果。例如：

```
{
  "columns": ["id", "name"],
  "filter": {
    "id": {
      "_eq": "X-HASURA-USER-ID"
    }
  }
}
```

- comment:权限的备注
- is_system_defined:真代表由系统内部创建；假代表由用户创建

注意：上述内容将持续更新。截止到上一次更新，后续可能有新的表或字段增加到目录中，以便于支持新的特性。

##### Exploring the catalogue

通过 Postgres 客户端访问 `hdb_catalog` schema,你可以核查当前 schema 和目录内容。

##### Catalogue versioning

当目录的 schema 被修改后，系统将自动升级目录版本。

如果有 Hasura 引擎更新期间有新的版本可用，目录版本也将在系统启动时更新。

### 参考

https://hasura.io/docs/latest/_images/auth-webhook-overview1.png

https://hasura.io/learn/zh/graphql/hasura/what-next/

https://www.graphqlweekly.com/

N+1 问题讨论：https://stackoverflow.com/questions/97197/what-is-the-n1-selects-problem-in-orm-object-relational-mapping

https://hasura.io/blog/how-hasura-works/

## Hasura 设计思想

本文，我打算从产品视角谈论下 Hasura，我们的技术设计思想，与其他技术的比较，以及一些常见问题。

如果你仅仅想快速开始，我们提供了练习指导，包含多种前端框架、Hasura 后端练习，和 GraphQL 的基本介绍。

### 1，主要问题

在 Hasura 中，我们的使命是让应用开发比传统方式更快。随着技术领域的发展，我们相信这个需要被攻克的瓶颈是让数据可用。

![s](https://lh4.googleusercontent.com/X2duY5dYkQCJbQKSJlm55Fie2nnvulYMHVDxce6PhQTfpSuZg4YXG4hrkNgXaCFxSz1xzjz0bXHOXUubVReMu3oBLzwzDgW4dlW3jOyflUDEhn74eKztuC1fWm30fpX2AvJv6fyP)

尤其是在企业环境中，采用最佳技术，构建现代化程序或者构建新的能力严重依赖对在线/实时数据的处理能力。

**最近几年有两个重要的趋势已经浮现：**

- 1.数据不在是单源：业务数据不断的被分散到多个源头，例如工作负载数据库、SAAS 解决方案、甚至网络服务。
- 2.不安全无状态化：开发者持续在各种环境中消费数据，包含不安全或者未授权的环境，例如 web/mobile apps 和无状态、高并发的计算环境，例如 web/mobile apps，容器，serverless funciton。

构建数据 API 是十分必要的，它需要能够多数据源兼容、包含明确的业务逻辑，并且提供必要的安全、性能和并发能力。

![能力架构](https://lh3.googleusercontent.com/zKImP3MqPqSsN5eUcbkoRwzvBNC0hDGSXeps_Urpc01HdamTe9fIk1pY17fivYVpKlHZxKeL1Gt9xTIPi-yDX9b8R6BPls_2h3s9Veyou3NO5Zqwo1ssDmlS5UbF9zIQ_4Se-oRA)

这是单调乏味的工作，也是一段糟糕的时光。这种工作是半重复的，例如和不同的数据源交互，并且导出 crud APIs，并且在某些领域也是相似的，例如授权规则。

**Hasura 的惊人的创意是把构建 API 单调乏味的部分变成一个通用的服务**

Hasura 开发者能够声明式的在几分钟内配置一个 web 服务并且暴露生产级别的 API，而不是花费数月写代码去关联大量表，潜入鉴权规则并且构建 APIs。从手工构建到服务生成。

![Hasura provides the data API "as a service" on your data sources.](https://lh3.googleusercontent.com/T9vg204_zn5mHhTOmdMryc5tbvHuAsI5m9VWRrndmVJsJm72TWyrngJ_xhbO1gXxJLxmBP2lpU78cL_EOLJh-vetD94blqtRXKyQGejbAYkzwjmTJ9N8Z_dj_TTYVuetiAsW5zUg)

Hasura 用户能够任何场景下快速完成需求。从印度的开发者、创业公司到财富 10 强，我们一次又一次的听到 Hasura 让他们以创纪录的速度和预测性发布他们的 APIs 和构建现代 apps。

对于所有的新功能必须对接现存的线上数据的企业就更有价值了，他们不需要从头建立应用。

### 2，Hasura 技术架构

设计像 Hasura 这样的产品是充满风险和挑战的事情。提取开发者习惯手工完成的工作，并把其中一部分自动化为一项服务并不是一件简单的事情。

几年前，那时 GraphQL 规范还未发布，我们就开始构建 Hasura。为了找灵感，我们调研了很多同类产品和技术。

#### 2.1 主要工作和设计灵感

如果我们回过头，考虑能够将重复部分的工作抽象为服务，同时捕获特定领域需求，从而大量提升开发者效率的技术，浮现在脑海中最好案例就是：数据库！

数据库提供了运行时的元数据引擎，能够满足开发者的特定需求，并且提供允许开发者操作数据的服务，而且没有处理原始文件带来的任何繁琐工作。例如，考虑下 Lucene 和 Elasticsearch 的不同。

![数据库发展历程](https://lh3.googleusercontent.com/JNwEuyHVrrA0rVpnA9vFD-Fgv5NsSYNGA2kdFRoP4cQKzRT-1qSnQSCOSC1S3lOIq6SXzyXS8_z6noKOC8OuYspMvtjbYGisgOY0ve7PKjlibd9Gx8tBXSj8XYy6k5NYZxkwvo1r)

使用数据库元数据，数据库引擎能够在运行时（作为服务）完全运行，而不需要开发人员进行任何构建时工作。
描述性的元数据系统时有用的，因为数据库引擎能够高效“思考它自己”，并且通过可预测但是灵活的 API 自动化抽象处繁重的工作.

此外，数据库服务能够在性能、安全、可用性以及其他操作方面提供开箱即用的保证和 SLAs。

当我们开始思考 Hasura 技术设计原语时，我们数据库世界中获得了大量启发，尤其是那些广泛被应用的数据库像 Postgres,构建搜索服务的 Lucene，同步数据从标准的到非标准化数据存储以及其他类似的 OLTP 数据任务。

**思考 Hasura 的方便方式是，它是工作在网络层的分布式数据库引擎的前端**支持网络负载。

核查下面这张表，它描述了数据库世界和 Hasura 引擎的相似之处。

| 数据库                                                       | Hasura                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 通过 TCP 传输 SQL                                            | 通过 HTTP 传输 GraphQL                                       |
| 通过关系代数公开模型或方法                                   | 通过统一 JSON 抽象暴露模型或方法                             |
| 元数据描述模型规则                                           | 元数据描述映射、关系和授权规则                               |
| 引擎与存储通讯                                               | 引擎与其他数据库或源通讯                                     |
| 实现事务引擎和带有容错的写优先的日志系统驱动复杂的数据库操作 | 实现带有至少 1 次和仅 1 次语意的事件引擎驱动无状态的业务逻辑 |

#### 2.2 Hasura 实现路径

Hasura 实现了一个元数据引擎，能捕获需要授权的网络服务特定领域需求。

基于这个，Hasura 提供了开箱即用的 APIs,能够灵活、安全的操作数据。

同时，Hasura 也能提供所谓 NFRs（非功能性需求）的服务等级协议和保证，去确保平滑使用。

在下图，你能看到 Hasura 是如何将 web API 概念映射到 声明性网络服务的，即“GraphQL 引擎”。

![Hasura](https://lh4.googleusercontent.com/bswkTBFrM6riII93BCiAzIwFjPxHhhXzajhFnpD590RSb997A2REhdsyjVKZJ3sWp-USRnub2r3FTbSQ1M7GwerJ3wCHnb2NfzwW_eQO3KhlKsjkE5hP6dy0YuWIMcNCWyuXyNLX)

让我们深入这些能力和特性的细节，我们将看到 Hasura 设计是如何应对这些技术难题的。

### 3. Hasura 关键能力

#### 3.1 分钟级生产准备

**Hasura 提供的最重要的商业价值是解锁开发者缩短上线时间**

我们做 Hasura 唯一要考虑的事情就是确保生产效率是超快的。

我们设计 Hasura 和围绕它的开发工具去确保意会 Hasura 的概念，并且尽可能快和容易的上手它。

![分钟级准备](https://lh6.googleusercontent.com/IGLgmDdIun7OxT0IlNgYLyZYm4nxEBtNDGP6RQwDqeXjBCNiXIYv6Khs_F7HkjVhuCVfyGO3McXW3EIBEfpC5JUbwak44U5zBxm4bi3HsZuXLwoaZ7BrTNO9uvRqqwkr9YaaYy4B)

#### 3.2 数据即时 GraphQL API

当 Hasura 服务运行时，开发者能够动态配置它的元数据。有多种方式可以修改配置，例如 UI、API，或者使用“应用到”Hasura 的 git 仓库中的代码集成 CI/CD pipeline 。

元数据捕获了一些关键内容：

- 1. 连接数据源
- 2. 将模型从数据源映射到 API
- 3. 在相同数据源和跨数据源间模型之间的关系
- 4. 模型或方法级别的鉴权规则

使用元数据，Hasura 生成 JSON API 描述，并且提供可被消费的 web API。

![s](https://lh3.googleusercontent.com/mGqRPpruMq_N35pJOr1_ntrJcl9n3vMDAA539bzuu7_269_1hr6bK5_7LRZ-E4gpdVC6swTMWOzVimMsuT1xVkRUTQ2RRe2BQEVykRpxkhHDhYMMDy4va7yJ2EScH1YzRdwa9Abz)

Hasura 通过 GraphQL 提供统一的 JSON API。GraphQL 是相当合适的，因为他原生就是 JSON。

作为一种 API 规范它能够灵活的捕获 xu
