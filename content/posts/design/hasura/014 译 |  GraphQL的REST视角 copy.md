---
title: "014 译 | GraphQL的REST视角"
date: 2022-03-02T02:29:45+08:00
tags: ["技术翻译", "Hasura"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

翻译自：[A REST View of GraphQL](https://hasura.io/blog/rest-view-of-graphql/)

软件设计经常对比[GraphQL](https://hasura.io/graphql/)（一种用于定义、查询和更新数据的语言规范）和 REST（一种描述 Web 的架构风格）。我们将探索为什么这个对比没有意义，并且我们应该问什么问题。本文，我们将谈论：

1. REST 架构风格意味着什么？
2. REST 如何用最佳实践实现?
3. GraphQL 是什么？它解决什么问题？
4. GraphQL 与 REST 如何不兼容？

## REST 是什么？

### REST 架构风格

REST 是一个架构风格，起源于 Roy Fielding 在 2000 年的博士理论。这项工作研究了使万维网成功的属性，并且推导出保留这些属性的约束。Roy Fielding 也是 HTTP/1.O 和 HTTP/1.1 工作委员会的成员。部分约束也被添加到了 HTTP 和 HTML 特性中。

### 什么组成了 Web？

理解 REST 之前，看看网络上不同类型的参与者是有用的：

1. 网站：为人类提供内容并在浏览器上消费和交互的程序。

2. 浏览器：主要供人类和网站交互的程序。
3. API 服务提供者：让其他程序消费和数据交互的程序。
4. API 客户端：用于消费和与 API 服务提供者进行数据交互的程序。

注意一个程序可以扮演多个角色。例如：API 服务提供者也可以是一个消费来自其他 API 提供者 API 的客户端。

也要注意，互联网和万维网是不一样的。互联网还有一些其他参与者，这里我们不做讨论（邮箱服务、种子客户端、基于区块链的应用等）。

### “架构风格“是什么？

架构风格是一组命名的、协调的架构约束集。架构约束是一个作用于架构组件上的限制条件，以便实现所需属性。一个例子是 UNIX 实用程序设计中使用的[统一管道和过滤架构。](https://github.com/lawgives/encyclopedia-of-architecture/wiki/Style%3A-Uniform-Pipe-and-Filter)UNIX 实用程序的推荐实践是：

1. 每个实用工具应该作为一个独立进程运行
2. 实用工具间仅使用`stdin`和 stdout 通过文本接口进行交流

遵循这些约束将带来一些好处：

1. 简单性：对新手来说容易一起使用实用程序
2. 复用性：如果第二个能够处理第一个的数据，允许混合使用任意两个使用程序。例如，我可以从 `cat` 或 `ls` 或 `ps` 到 `grep` 传送输出。`grep` 的作者不用担心输入来自哪里。

但是在这个约束下，我们为处理增加了延迟，因为每个工具必须把输出写入 stdout，并且下个工具必须从 stdin 读取它。

对`cat` 、 `ls` 、 `ps`等的一个可替代设计是变成一个具有很好定义接口的库。然后期望终端用户写集成不同库的程序解决他们的问题。系统将会有更好的性能，并且与之前的系统大致相同，但是将变得更复杂去使用。

通常，我们增加到设计中的每个约束都有取舍。

> 软件设计是关于确定最佳满足需求的设计约束的集合。

### web 架构约束

让我们谈谈构建一个"RESTful"服务时你应该遵守的约束：

- 使用[客户端/服务端](https://en.wikipedia.org/wiki/Client%E2%80%93server_model)模式通讯

- 让每个请求发送处理该请求所需的所有内容，从而让保持服务无状态

- 对请求的响应必须被标记为可缓存或不可缓存

- 统一接口：

  - 服务必须使用唯一 ID 公布资源
  - 检索或操作资源必须使用声明（媒体类型）
  - 请求和响应必须有能解释他们的所有信息，例如，他们必须是**自描述的**
  - 超媒体作为应用状态引擎（HATEOAS）:客户端必须仅仅依赖请求的响应去决定能采取的下一步。必须没有与此相关的额外通讯。

- 分层系统：系统的每个组件必须只依赖它直接交互的系统的行为。

- 定制代码：这是可选的约束。服务可以发送被客户端执行的代码，去扩展客户端的功能（例如，JavaScript）

这些约束的目的是让网络易开发、可扩容、高效，并且支持客户端和服务端独立发展。[这个](https://codewords.recurse.com/issues/five/what-restful-actually-means)文章用更多细节解释了这些约束。

### REST 实践

HTTP 是实现 REST 架构风格的首选协议。一些约束被集成到 HTTP 中，例如客户端/服务端模式，将资源作为缓存和分层系统。Others need to be explicitly followed.(没想好怎么翻译)

统一接口尤其 HATEOAS 是最常违反的 REST 约束。让我们看下每个子约束：

![The different components of a REST Request](https://hasura.io/blog/content/images/2020/07/Frame-9-1.svg)

#### 服务必须暴露带唯一 ID 的资源

按照惯例，URI 扮演资源 ID 的角色，并且 HTTP 方法是统一的操作集合，它可以在任何资源上执行。当使用 HTTP 时，第一个约束根据定义自动遵守。大多数后端框架（Rails,Diango 等)也引导你遵守第二个约束。

#### 使用声明检索和操作资源

客户端使用媒体类型如 HTML 或 JSON 检索和操作资源。媒体类型如 HTML 也可以包含客户端能对资源执行的操作，例如，使用表单。这允许客户端和服务端独立发展。任何理解特定规范的客户端可以与任何支持该规范的服务一起使用。

如果你正期望有多个服务相似类型数据的服务，并且多个客户端访问他们，这个约束是有用的。例如，任何 WEB 浏览器可以渲染任何类型为 `text/html` 的页面。相似的，任何 RSS 阅读器将和任何支持`application/rss+xml`媒体类型的服务一起工作。

然而，很多场景不需要这种灵活性。

#### 自描述信息：请求和响应必须有所有能够解释他们的信息

同样，这允许服务端和客户端独立发展，因为客户端不需要假设一个特定的响应结构。这在 WEB 浏览器和 RSS 阅读器中工作的很好。然而，很多 API 客户端为访问特定服务而构建，并且被限制了响应语义。在这种场景下，上述维护自描述信息不总是有用的。

#### 超媒体作为应用状态引擎（HATEOAS）

![EST Response with possible next actions the client can make](https://hasura.io/blog/content/images/2020/07/Frame-7--1-.svg)

这个约束也允许服务端和客户端独立发展，因为客户端不用硬编码可用的下一步。如果消费者是使用浏览器的终端用户，HATEOAS 将很有意义。浏览器将伴随用户采取的操作（表单和锚标记）简单的渲染 HTML。用户将然后理解什么在页面上，并且采取他们更喜欢的行动。如果你更改用户可用的下一步操作集合的 URL 或者表单参数，浏览器无需做任何更改。用户将仍然能够阅读页面，理解发生了什么（或许不情愿），并采取正确的行动。

不幸的是，这对 API 客户端不起作用：

1. 如果你为 API 改变需要的参数，客户端程序开发者或许可能必须理解变化的语义，并且更改客户端去适应这些改变。

2. 对于客户端开发者来说，通过逐个浏览来探索 API 不是非常有用。全面的 API 文档（例如 Swagger 或者 Postman 集合）更有意义。

3. 在一个带有微服务的分布式系统中，下一步操作通常由完全不同的系统采取（通过监听当前操作触发的事件）。所以，为当前客户端返回可用操作的列表是无用的。

所以对 API 客户端，遵循 HATEOAS 约束不意味着独立演化。

很多 API 客户端为单个后台而编写，并且一定数量的耦合将总是存在于 API 客户端和后台之间。在这个案例中，我们可以仍然减小耦合的数量，通过确保：

1. 为 API 调用增加新的参数不应破坏现存客户端
2. 为响应增加新的字段不破坏现存客户端
3. 应该可以为新客户端增加一个新的序列化格式（例如 JSON 到 HTML 或 protobuf）

上述#1 和#2 通常通过 API 版本控制和演化实现。Phil Sturgeon 有一些关于[ API 版本控制](https://apisyouwonthate.com/blog/api-versioning-has-no-right-way)和[演化](https://phil.tech/2018/api-evolution-for-rest-http-apis/)的很好的文章，他们谈论了各种最佳实践。#3 可以通过遵循 HTTP `Accepts`头来实现。

## GraphQL 是什么？

GraphQL 是一个定义数据模式、查询和更新的语言，由 Facebook 在 2012 年发布，于 2015 年开源。GraphQL 的关键思想是：与实现各种资源端点来获取和更新数据不同，你定义了可用数据的总体模式，以及可能的关系和变更（更新）。然后客户端可以查询他们需要的数据。

下面是 GraphQL 模式的示例：

```gql
# Our schema consists of products, users and orders. We will first define these types and relationships
type Product {
  id: Int!
  title: String!
  price: Int!
}

type User {
  id: Int!
  email: String!
}

type OrderItem {
  id: Int!
  product: Product!
  quantity: Int!
}

type Order {
  id: Int!
  orderItems: [OrderItem!]!
  user: User!
}

# Some helper types for defining the query types
type ProductFilter {
  id: Int
  title_like: String
  price_lt: Int
  price_gt: Int
}

type OrderFilter {
  id: Int
  userEmail: String
  userId: Int
  productTitle: String
  productId: Int
}

# Define what can be queried
type Query {
  #Query user
  user(email: String!): User!

  #query products
  products(where: ProductFilter, limit: Int, offset: Int): [Product!]!

  #query orders
  orders(where: OrderFilter, limit: Int, offset: Int): [Order]
}

# Helper types for defining mutations (updates)
type OrderItemInput {
  product: Product!
  quantity: Int!
}

type OrderInput {
  orderItems: [OrderItemInput!]!
  user: User!
}

scalar Void

# Define possible updates
type Mutation {
  insertOrder(input: OrderInput!): Order!
  updateOrderItem(id: Int!, quantity: Int!): OrderItem!
  cancelOrder(id: Int): Void
}
```

下面是 GraphQL 查询和响应的示例：

![Sample GraphQL Requests and Responses](https://hasura.io/blog/content/images/2020/07/Frame-8--1-.svg)

### 为什么使用 GraphQL？

GraphQL 的主要优势是一旦定义数据模式和解析器：

1. 客户端可以获取他们需要的准确数据，减少所需网络带宽（缓存可能会变得很棘手，后续详细说明）
2. 前端团队可以在执行时很少依赖后端团队，因为后端几乎暴露了所有可能的数据。这允许前端团队更快的执行
3. 因为模式是有类型的，它可能产生类型安全的客户端，减少类型错误

然而，在后端，写解析器涉及一些挑战：

1. 为了真正实现#1 和#2，解析器将通常需要携带各种过滤、分页和排序选项
2. 优化解析器很麻烦，因为不同的客户端将请求不同的数据子集
3. 当实现解析器时，需要担心避免 N+1 查询
4. 对客户端很容易构建复杂的嵌套（和潜在的循环）查询，这使得服务器做很多工作。这样可能导致其他客户端的 DOS 攻击

Paypal 在[这篇文章](https://medium.com/paypal-engineering/graphql-resolvers-best-practices-cd36fdbcef55)中更详细的描述了这些问题。

GraphQL 使得前端开发花费更少的精力去迭代，但是代价是后端开发者必须提前付出额外的努力。

使用 Hasura，你不必担心前 3 个问题，因为 Hasura 直接编译你的 GraphQL 查询为 SQL 查询。所以，当使用 Hasura 时,你不用写任何解析器。使用 SQL `JOIN`生成查询获取相关数据避免 N+1 查询问题。[允许列表特性](https://hasura.io/docs/latest/graphql/core/deployment/allow-list.html)也为#4 提供了一个解决方案。

## GraphQL vs REST

[理解了 GraphQL 和 REST 是什么，你可以看到“GraphQL vs REST”一种错误的比较。](https://hasura.io/blog/adding-rest-endpoints-to-hasura-cloud/)相反，我们需要问下列的问题：

### GraphQL 如何打破了 REST 的架构风格？

GraphQL 与 REST 客户端/服务端，无状态，分层系统和按需代码约束是一致的，因为 GraphQL 通常使用 HTTP，而 HTTP 早已强制满足这些约束。但是它破坏了一致接口约束，并且在某种程度破坏了缓存约束。

#### GraphQL 和缓存

缓存约束声明请求的响应必须被标记为可缓存和不可缓存。在实践中，通过使用 HTTP `GET`方法实现，并且使用 `Cache-Control`, `Etags` 和 `If-None-Match` 头。

理论上，你可以使用 GraphQL 并且保持与缓存约束一致，如果你对查询使用 HTTP `GET`，并且使用正确的头部。然而，在事件中，如果你有大量不同的被不同客户端发送的查询，你将使得缓存主键爆炸（因为每个查询结果将需要独立缓存）并且缓存层的用途将丢失。

GraphQL 客户端像 [Apollo](https://www.apollographql.com/docs/react/) 和 [Relay](https://relay.dev/) 实现了解决该问题的客户端缓存。这些客户端将分解请求的响应并缓存单个对象。当下次查询触发时，仅仅不在缓存中的对象需要被重新获取。这确实比 HTTP 缓存带来更好的缓存利用，因为响应的部分也能被复用。所以，如果你正在浏览器、安卓或 IOS 使用 GraphQL，你不需要担心客户端端缓存。

如果你需要共享缓存（CDN、Varnish 等），你需要确保不会遇到缓存键爆炸。

Hasura Cloud 通过为查询增加一个@cached 指令支持数据缓存。阅读 Hasura 支持所有的[查询缓存和数据缓存](https://hasura.io/blog/announcing-hasura-cloud-managed-graphql-for-your-database-and-services/#caching)的更多信息。

#### GraphQL 和统一接口

GraphQL 破坏统一接口资源约束。我们在上面讨论了为什么 API 客户端在特定场景下破坏这个约束是可行的。统一资源约束预期会带来下述属性：

1. 简单化：因为一切都是资源，并且有相同的 HTTP 方法集合适用于它
2. 前后端独立演化
3. 前后端解耦

如果你正在建立 GraphQL 服务，你应该将 APIs 建模为 带有唯一 IDs 的资源，并且有一组统一操作访问他们。这使开发者更容易浏览 APIs。如果你正在建立后端 GraphQL 服务，那么可以遵循 [Relay 服务规范](https://relay.dev/docs/)。

我们最近已经为 Hasura 添加了 [Relay 支持](https://hasura.io/blog/adding-relay-support-to-hasura/)。因为 Hasura 自动从数据库模式生成 GraphQL 查询和变更，你也自动为每个资源获得一组统一操作。

### 我应该什么时候使用 GraphQL？

1990 和 2000 年的 WEB 与今天的 WEB 大相径庭。

1. 我们今天构建的前端应用更丰富，并且提供更多功能。
2. 用 JavaScript 框架如 React, Vue, Angular 而不是从后端渲染模板在持续增长。然后，JavaScript 应用变成了后台的 API 客户端。
3. 前端应用也越来越多通过手机网络访问，他们经常又慢又不稳定。

当设计当代系统时，我们需要考虑这些。GraphQL 帮助在这种上下文中构建高性能的应用程序。

正如在 GraphQL 部分描述的那样，使用 GraphQL 的关键优势是在前端的数据查询变得更容易，并且这使得前端迭代更快。权衡是：

1. 统一资源约束被破坏。对于绝大多数客户端来说，这不应该是一个问题。
2. 需要确保每个可能被触发的查询将被高效服务。
3. HTTP 缓存架构将可能不工作。

使用 Hasura 通过使用权限系统[禁止聚合查询](https://hasura.io/docs/latest/graphql/core/auth/authorization/permission-rules.html#aggr-query-permissionsS)或者设置一个确切的[允许查询列表](https://hasura.io/docs/latest/graphql/core/deployment/allow-list.html)你可以解决上述#2。

Hasura cloud 也为[缓存问题](https://hasura.io/blog/announcing-hasura-cloud-managed-graphql-for-your-database-and-services/#caching)提供了一个解决方案。

### 我能否使用参数去指定我需要的确切数据，而不破坏 REST 吗？

是的，你可以，例如通过支持[稀疏字段设置规范](https://jsonapi.org/format/#fetching-sparse-fieldsets)。你最好在实践中使用 GraphQL，因为：

1. 任何种类的查询语言将需要在后端解析和实现。GraphQL 有很多更好的工具去做这些。
2. 使用 GraphQL，响应形状和请求形状一致，让客户端更容易访问响应。

然而，如果你确实需要遵循统一接口约束，这是你应该采取的方法。注意，使用稀疏字段也可能导致缓存爆炸，破坏共享缓存。

## 结论

我们已经考虑了 GraphQL 和 REST,并且无论比较他们是否有意义。系统设计过程如下：

1. 理解所有功能和非功能需求
2. 推到满足需求的设计约束（尤其是非功能的）
3. 选择帮助我们实现这些约束的技术

REST 描述 web 的这些约束。例如，很多约束对很多系统时恰当的。例如，通过资源组织 API，为每个资源赋予 IDs，并且暴露一组公共操作集合，也是 GraphQL API 设计的最佳实践。（以及[基于 RPC ](https://google.aip.dev/121)的系统也是）

如果你像尝试 GraphQL，前往[学习课程](https://hasura.io/learn/graphql/intro-graphql/introduction/)。Hasura 是一个快速的方式去学习和把玩 GraphQL，因为你的后端被自动设置。
