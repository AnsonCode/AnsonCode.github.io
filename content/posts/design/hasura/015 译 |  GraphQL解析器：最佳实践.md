---
title: "015 译 | GraphQL解析器：最佳实践"
date: 2022-03-02T21:29:45+08:00
tags: ["技术翻译", "Hasura"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

翻译自：[GraphQL Resolvers: Best Practices](https://medium.com/paypal-tech/graphql-resolvers-best-practices-cd36fdbcef55/)

本文是在 PayPal 构建 GraphQL APIs 所做的最佳实践和观察系列的第一部分。在日后的文章中，我们将分享我们关于：方案设计，错误处理，生产可视性，优化客户端集成和团队工具的想法。

你或许已经看过我们之前的文章：GraphQL: PayPal Checkout 成功的故事，它讲述了 PayPal 从 REST 到 GraphQL 的旅程。本文将详细介绍一些构建解析器的最佳实践，解析器是快速的，可测试的，有弹性的。

## 解析器是什么？

让我们从相同的基准线开始。解析器是什么？

> 解析器：一个在 schema 中解析类型或字段值的函数或方法。

> 每种类型的每个字段被一个叫做解析器的函数支持。

解析器是一个在 schema 中解析类型或字段值的函数。解析器可以返回对象或标量如 Strings, Numbers, Booleans 等。如果返回一个对象，执行继续到下一个子字段。如果返回标量（通常在叶子结点），执行完成。如果返回 null，执行停止并不再继续。

解析器也可以是异步的。他们可以从其他 REST API，数据库，缓存，常量等，解析值。

稍后，我们将遍历一系列示例，图解如何构建快速的、可测试的、有弹性的解析器。

## 执行查询

为了更好理解解析器，你需要了解查询如何执行。

每个 GraphQL 查询经过三个阶段。解析、校验和执行查询。

1. 解析：解析查询为一个抽象语法树（或者 AST）。ASTs 难以置信的强大，并且位于 ESLint, babel 等工具之后。如果你想看 GraphQL AST 看起来像什么，看看[astexplorer.net](https://astexplorer.net/)，并且将 JavaScript 改成 GraphQL。你将看看到查询在左侧，AST 在右侧。
2. 校验：根据 schema 验证 AST。检查正确的查询语法以及字段是否存在。
3. 执行：运行时遍历 AST，从树的根节点开始，调用解析器，收集结果，并且输出 JSON。

例如，我们将参考这个查询：

![Query for later reference](https://gitee.com/caoyanbin/picgo/raw/master/img/20220302230121.png)

当解析该查询时，它被转化为一个 AST 或者树。

![Query represented as a tree](https://gitee.com/caoyanbin/picgo/raw/master/img/20220302230229.png)

根 Query 类型是树的入口点，并且包含两个根字段，`user` 和 `album`。并行执行`user` 和 `album`解析器（在所有运行时中国呢都是常见的）。宽度优先执行树，意味着必须解析`user`，在它的子节点 `name` 和 `email` 执行之前。如果解析器时异步的，`user`分支延迟到它被执行。一旦解析所有子节点:`name`, `email`,` title`，执行就完成了。

并行执行根 Query 字段，像`user` 和 `album`，但是没有特定的顺序。通常，按照他们在查询中出现的顺序执行字段，但是做这样的假定是不保险的。因为并行执行字段，他们假定为原子的、幂等的、边际效应无关的。

### 仔细观察解析器

在接下来的部分，我们将使用 JavaScript，但是可以用很多语言来写 GraphQL 服务。

![Resolvers with four arguments — root, args, context, info](https://gitee.com/caoyanbin/picgo/raw/master/img/20220302231655.png)

在某种形式中，每种语言的每个解析器都接收 4 个参数：

- root：来自上一个或父类型的结果

- args：为字段提供的参数
- context：提供给所有解析器的可变对象
- info：与查询有关的特定于字段的信息（很少使用）

这 4 个参数是理解解析器间数据流的核心。

## 默认解析器

我们继续之前，值得注意的是 GraphQL 服务有内建的默认解析器，所以你不必为每个字段指明解析函数。默认解析器会在根节点寻找和字段同名的属性。一个可能的实现：

![Default resolver implementation](https://gitee.com/caoyanbin/picgo/raw/master/img/20220302233544.png)

## 在解析器中查询数据

我们应该在哪里查询数据？我们的选择有哪些权衡？

在接下来的示例中，我们将参考这个 schema：

![An event field has a required id argument, returns an Event](https://gitee.com/caoyanbin/picgo/raw/master/img/20220302233753.png)

## 在解析器间传递数据

context 是一个提供给所有解析器的可变对象。在每个请求间创建和销毁它。它是存储公用 Auth（身份鉴权）数据、APIs 和数据库通用模型/加载器等的好去处。在 PayPal，我们是一个基于 Express 架构的 Node.js 商店，所以我们在那里存储 Express 的 req。

当你第一次学习 context 时，初始的想法或许是使用 context 作为通用缓存。这不被推荐，但是实现可能是这样的。

![Passing data between resolvers using context. This is not recommended!](https://gitee.com/caoyanbin/picgo/raw/master/img/20220302235616.png)

当 调用 `title`，我们在 `context` 中存储 `event` 结果。当调用 `photoUrl`，我们从 `context` 获取 `event`，并且使用它。这个代码是不可靠的。不能保证 `title` 在 `photoUrl` 之前执行。

我们可以改进所有解析器，核查 `event` 是否存在于 `context`。如果这样，使用它。否则，我们获取它并且稍微存储它，但是这仍然有很大出错空间。

反而，我们应该避免在解析器内部修改 `context`。我们应该避免知识和焦虑混杂在一起，以便于我们的解析器容易理解、调试和测试。

## 从父到子传递数据

root 参数用于从父解析器到子解析器传递数据。

例如，如果你正在构建 Event 类型，在那里 Event 的所有字段都依赖相同数据，你或许想要在 event 字段中获取它一次而不是在 每个 Event 字段。

似乎是一个好的注意，对吗？这是开始建立解析器的快速方式，但是你或许会遇到一些问题。让我们理解为什么。

如下示例，我们将使用有两个字段的 Event 类型。

![Event type with two fields: title and photoUrl](https://gitee.com/caoyanbin/picgo/raw/master/img/20220303000954.png)

Event 的很多字段可以从 Event API 获取，所以我们可以在顶级 event 解析器中获取它，并为我们的 title 和 photoUrl 解析器提供结果。

![Top-level event resolver fetches data, provides results to title and photoUrl field resolvers](https://gitee.com/caoyanbin/picgo/raw/master/img/20220303001126.png)

甚至，我们不需要指明下面的两个解析器。

我们可以使用默认解析器，因为被` getEvent()`返回的对象有 `title` 和 `photoUrl` 属性。

![id and title are resolved using default resolvers](https://gitee.com/caoyanbin/picgo/raw/master/img/20220303001346.png)

这有什么问题吗?

有两种场景你或许会重复查询……

### 场景#1：多层数据查询

假设有一些需求进来，你需要展示一个 event 的参与者。我们开始为 Event 增加 attendees 字段。

![Event type with an additional attendees field](https://gitee.com/caoyanbin/picgo/raw/master/img/20220303001650.png)

当你查询 attendees 详情时，你有两个选择：在 event 解析器中查询数据或者 attendees 解析器。

我们将测试第一个选项：将其添加到 event 解析器中。

![event resolver calls two APIs, fetching event details and attendees details](https://gitee.com/caoyanbin/picgo/raw/master/img/20220303001846.png)

如果客户端仅查询 title 和 photoUrl, 而没有 attendees。现在你开始低效并且向 Attendees API 发起无效的请求。

这不是你的错误，这是我们的工作方式。我们识别模式并复制他们。如果参与者看到数据查询在 event 解析器中完成，他们将可能增加任何其他数据查询，而不需要考虑太多。

我们还有一个选项测试在 attendees 解析器中获取 attendees。

![attendees resolver fetches attendees details from the Attendees API](https://gitee.com/caoyanbin/picgo/raw/master/img/20220303002420.png)

如果我们的客户端仅查询 attendees，而不是 title 和 photoUrl。我们仍然效率低下，因向 Events API 发起了不必的请求。

### 场景#2：N+1 问题

由于数据在字段级查询，所以我们有过渡获取的风险。在 GraphQL 世界中，过渡获取和 N+1 问题是常见的话题。[Shopify](https://medium.com/u/bab76dfc19b0?source=post_page-----cd36fdbcef55-----------------------------------) 有一个很赞的文章很好的解释了 N+1 问题。

这对我们有什么影响？

为了更好的阐明它，我们将增加一个新的 events 字段，返回所有 events。

![An events field returns all events.](https://gitee.com/caoyanbin/picgo/raw/master/img/20220303003158.png)

![Query for all events w/ their title and attendees](https://gitee.com/caoyanbin/picgo/raw/master/img/20220303003216.png)

如果客户端查询所有 events 和他们的 attendees，我们有过渡获取的风险，因为 attendees 可以参与不止一个 event。我们或许重复查询相同的 attendee。

该问题在大型组织中被放大，在那里请求可能扩散并且给系统造成不必要的压力。

为了解决它，我们需要批量并且删除重复请求！

在 JavaScript，一些受欢迎的选项是 [ 数据加载器](https://github.com/graphql/dataloader) 和 [Apollo 数据源](https://www.apollographql.com/docs/apollo-server/features/data-sources.html).

如果你正使用其他语言，你可能会发现一些其他东西。所以，在你自己解决这个之前，先找一圈。

最重要的是，这些位于数据访问层之上的库，将缓存和删除重复请求，使用消除抖动和缓存。如果你好奇异步缓存，看看 Daniel Brain 的[大作](https://medium.com/@bluepnume/async-javascript-is-much-more-fun-when-you-spend-less-time-thinking-about-control-flow-8580ce9f73fc)。

### 在字段级获取数据

早些时候，我们看到“头重脚轻“的父到子解析器很容易导致过渡获取。

**有没有更好的选择？**

让我们再次梳理下父到子选项。如果我们反过来，让子字段负责获取他们自己的数据呢？

![Fields are responsible for their own data fetching.](https://gitee.com/caoyanbin/picgo/raw/master/img/20220303004804.png)

**为什么这是一个更好的选择？**

这段代码很容易理解。你确切的知道邮件在那里获取。这很容易调试。

这段代码是可测试的。当你只想测试 title 解析器时，你不必测试 event 解析器。

在某种程度上，getEvent 重复或许看起来像坏代码的味道。但是， 拥有简单、易于推理和更易于测试的代码是值得重复的。

但是，这仍然有一个潜在的问题。如果客户端查询 title 和 photoUrl,我们用 getEvent 在
Event API 上发起额外的请求。正如我们之前在 N+1 问题中看到的，我们应该使用像[ 数据加载器](https://github.com/graphql/dataloader) 和 [Apollo 数据源](https://www.apollographql.com/docs/apollo-server/features/data-sources.html)一样的库在框架层删除重复请求。

如果我们在字段级获取数据并且删除重复请求，我们的代码容易调试和测试，并且我们可以不用考虑优化查询数据。

## 最佳实践

- 从父到子的数据获取和解析应该谨慎使用
- 使用像[ 数据加载器](https://github.com/graphql/dataloader)这样的库删除重复下游请求
- 注意对数据源造成的任何压力
- 不要修改”context。确保代码一致,Bug 更少
- 写可读的、可维护的和可测试的解析器。不要太过聪明
- 确保解析器尽可能的薄。为可重用的异步函数提供数据获取逻辑

## 继续学习！

思考？我们将喜欢听到你的团队构建解析器的最佳实践和学习。这不是经常讨论的话题，但对构建持久的 GraphQL APIs 至关重要。

在即将到来的文章中，我们将分享我们的思考关于：方案设计，错误处理，生产可视性，优化客户端集成和团队工具。
