---
title: "018 译 | 远程连接：连接跨数据库和其他数据源的 GraphQL API"
date: 2022-03-16T21:48:45+08:00
tags: ["技术翻译", "Hasura"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

翻译自：[Remote Joins: A GraphQL API to join across your database and other data-sources](https://hasura.io/blog/remote-joins-a-graphql-api-to-join-database-and-other-data-sources/)

今天我们很高兴宣布 Remote Joins 的发布。使用 Hasura Remote Joins 的数据联合现在在 V1.3.0 稳定版本可用。

**使用 Remote Joins ，开发者可以将跨不同数据源的数据视为同一个数据库而不必修改或者影响现存数据源。**

随着开发者和团队开始使用越来越多的第三方 APIs，多数据库和微服务，数据和“真相之源”正在这些数据源间传播。应用开发者很难访问精确的数据切片，并且花费大量研发周期将数据安全的整合到一起。

![Remote Joins: A GraphQL API to join across your database and other data-sources](https://gitee.com/caoyanbin/picgo/raw/master/img/20220315221708.png)

Hasura 的 Remote Joins 扩展跨表连接数据的概念，能够跨表和远程数据源连接数据。一旦创建了数据库类型和 APIs 类型之间的关系，然后你可以通过执行 GraphQL 查询“连接”他们。

Remote joins 可以跨数据库和 APIs 连接。这些 APIs 可以是你写的[自定义 GraphQL 服务](https://hasura.io/graphql/production-ready-existing-apis/)，第三方 SaaS APIs，或者甚至其他 Hasura 实例。

当然，由于 Hasura 旨在成为一个你可以直接暴露到应用上的 GraphQL 服务，当提供远程连接时，Hasura 也处理安全和身份验证。

## 用例#1：客户数据与在 Stripe 中的账户、账单、支付信息连接

如果在你的应用上有使用 Stripe 的支付方法，你客户的很多账户、账单信息存储在 Stripe 中。例如，客户的储值卡、账户余额、账单/发票历史。

![Customer data joined with account, billing, payment-transaction data in Stripe](https://gitee.com/caoyanbin/picgo/raw/master/img/20220315224653.png)

在应用中，你不出所料的想要为用户展示卡片和发票历史。你将写代码从 Stripe API 为正确的用户、发票获取正确的数据，并将其作为 API 响应返回给应用。

这是 Hasura 上的设置：

### 步骤 1：设置 OneGraph 连接到你的 Stripe 账户

![Setup OneGraph to connect to your Stripe account](https://gitee.com/caoyanbin/picgo/raw/master/img/20220315225202.png)

### 步骤 2：将 OneGraph 作为远程模式添加到 Hasura

![Add OneGraph as a remote schema to Hasura](https://gitee.com/caoyanbin/picgo/raw/master/img/20220315225335.png)

### 步骤 3：使用 `stripe_customer_id` 作为“连接”键设置客户表到 Stripe 解析器的远程关系

![Your table should have the stripe_customer_id column](https://gitee.com/caoyanbin/picgo/raw/master/img/20220315225602.png)

![Setup a remote relationship from your customer table to the Stripe resolver using the stripe_customer_id as the "join" key](https://gitee.com/caoyanbin/picgo/raw/master/img/20220315225634.png)

### 步骤 4：发起 GraphQL 查询在一个请求中获取客户和 stripe 信息！

![Make GraphQL queries to fetch customer and stripe information in one shot!](https://gitee.com/caoyanbin/picgo/raw/master/img/20220315225833.png)

## 用例#2：用户数据连接 Auth0 的个人资料信息

当你使用外部身份验证提供商（或者甚至一个你团队不直接拥有的身份验证/用户管理微服务），将有一些信息仅由身份认证服务提供。例如， `last_login`, `verified_email`, `login_count` 等。

在数据库中，你通常有一个具有 `auth0_id` 字段的用户表。为获取特定身份信息，你需要使用这个 ID 和 Auth0 API。

![User data joined with account/profile/auth information in Auth0](https://gitee.com/caoyanbin/picgo/raw/master/img/20220316202126.png)

下面是 Hasura 如何帮助你自动化这个流程：

### 步骤 1：在 [GraphQL 查询](https://hasura.io/learn/graphql/intro-graphql/graphql-queries/)解析器中写一个封装有 Auth0 个人资料 API 的无服务函数

![Write a serverless function that wraps the Auth0 profile API in a GraphQL query resolver. ](https://gitee.com/caoyanbin/picgo/raw/master/img/20220316202440.png)

### 步骤 2：在 Hasura 中将此连接为一个 remote schema

![Connect this as a remote schema in Hasura](https://gitee.com/caoyanbin/picgo/raw/master/img/20220316202634.png)

### 步骤 3：前往 Hasura 中的 `USER` 表，并为上面添加的模式中增加一个“远程关系”

![The auth0_id identifier is the "join" key from your database](https://gitee.com/caoyanbin/picgo/raw/master/img/20220316202913.png)

![Add a "remote relationship" to the auth0 schema](https://gitee.com/caoyanbin/picgo/raw/master/img/20220316203010.png)

### 步骤 4：发起一个 GraphQL 查询同时从`USER` 表和 Auth0 个人资料信息查询数据

![Make a GraphQL query to query data from the user table and the Auth0 profile information simultaneously](https://gitee.com/caoyanbin/picgo/raw/master/img/20220316204258.png)

## 用例#3：跨 2 个独立的数据库连接数据

![Joining data across 2 databases](https://gitee.com/caoyanbin/picgo/raw/master/img/20220316204411.png)

标题说明了一切。这是 Hasura 的工作原理！

### 步骤 1：设置 Hasura 使用正确的权限控制规则指向数据库 1 和数据库 2，以便 GraphQL API 是安全的

![The "remote" Hasura + Postgres](https://gitee.com/caoyanbin/picgo/raw/master/img/20220316204706.png)

![The "primary" Hasura + Postgres](https://gitee.com/caoyanbin/picgo/raw/master/img/20220316204725.png)

### 步骤 2：在你“主”数据库的 Hasura 上，增加第二个 Hasura 作为一个远程模式

![On the Hasura of your "primary" database, add the second Hasura as a remote schema](https://gitee.com/caoyanbin/picgo/raw/master/img/20220316204924.png)

### 步骤 3：建立从“主”数据库任意表到其他数据库解析器 的远程关系

![Set up remote relationships from any table in your "primary" database to resolvers in the other database](https://gitee.com/caoyanbin/picgo/raw/master/img/20220316205300.png)

### 步骤 4：运行跨 2 个数据库连接的 GraphQL 查询

![A GraphQL query that joins data across 2 databases!](https://gitee.com/caoyanbin/picgo/raw/master/img/20220316205447.png)

注意：Hasura Remote Joins 是一个联邦类型，类似 Apollo Federation。主要的不同是 Apollo Federation 目的是帮助拥有多个 GraphQL 团队的组织“联合”一个全局的 GraphQL schema。 Hasura remote joins 帮你跨多个数据源（或 GraphQL & REST microservices）“连接”，以便你获得一个统一的数据联合 GraphQL API。该统一 GraphQL API 或许对为其应用提供多个数据源的团队所必须，也可能是是一个统一的跨多团队的 GraphQL API，由组织的中心团队维护。
