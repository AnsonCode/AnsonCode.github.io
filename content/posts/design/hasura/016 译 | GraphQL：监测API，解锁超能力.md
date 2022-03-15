---
title: "016 译 | GraphQL：监测API，解锁超能力"
date: 2022-03-02T21:29:45+08:00
tags: ["技术翻译", "Hasura"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

翻译自：[GraphQL: Instrumenting your API and unlocking superpowers](https://medium.com/paypal-tech/graphql-instrumenting-your-api-and-unlocking-superpowers-c0bc3a9dc451)

本文是在 PayPal 构建 GraphQL APIs 所做的最佳实践和观察系列的第二部分。在日后的文章中，我们将分享我们关于：方案设计，错误处理，生产可视性，优化客户端集成和团队工具的想法。

你或许已经看过我们之前的文章：GraphQL: PayPal Checkout 成功的故事，它讲述了 PayPal 从 REST 到 GraphQL 的旅程。本文将详细介绍插桩以及为什么它是重要的，为什么监测 API 解锁超能力以及一些监测 API 的策略。

**插桩**

1. 检测或测量产品性能水平，诊断错误，写追踪信息的能力
2. 怎么样理解你的用户如何使用 API，并且怎么随着时间自信地演化 API

插桩是在应用程序代码中增加合适数量的日志，以便于你确切的知道它是如何被使用的。

## 解锁超能力

使用 GraphQL，客户端必须在字段级别明确要求他们需要的数据。他们只接收这些字段，不多也不少。例如：客户端不能简单的要求 USER 类型的所有字段。由于设计约束，插桩在 GraphQL 中比在传统 REST APIs 中更有用。

## 满怀信心的改造 API

在 REST 中，客户端请求资源或对象。你或许知道客户端调用 GET /project 端点多少次，但是你不知道他们真的需要资源的哪一个字段。你无法洞察资源的哪个部分是有问题的或者较慢的。更坏的是，你无法洞察客户端如何使用 APIs，所以你更可能保持扩展或增长 API，因为你太害怕做出突破性的改变。

在 GraphQL 中，可以随着时间推移逐渐进行更改。你不需要创建新的主要版本（例如：/V3）并且维护一个 API 的多个版本。你不需要启动一个大型项目迁移所有的客户端到一个新的主要版本上。

如果你想重命名或者移除字段，并且你的插桩说没人使用它？好极了！你可以自信地移除它。

如果你想完全重构 schema，你也可以渐进迁移！创建并部署新的结构，然后使用@deprecated 指令标记旧的字段，这样新的客户端就看不到他们了，通知正请求旧字段的客户端，稍后安全的移除它。

## 强大的工具

插桩为强大的工具打开了大门！

现在，你确切知道哪个字段太慢或者造成太多错误。你也确切知道你的字段被请求了多少次，以及被谁请求。

首先，你可以将数据传输到像 Grafana 这样的工具中。

![Example of latency per resolver/field](https://gitee.com/caoyanbin/picgo/raw/master/img/20220313161940.png)

![Example of # of errors per resolver/field](https://gitee.com/caoyanbin/picgo/raw/master/img/20220313162015.png)

![Example of # of times a resolver/field is executed](https://gitee.com/caoyanbin/picgo/raw/master/img/20220313162033.png)

接下来， Marc-André Giroux 在 [Continuous Evolution of Schemas](https://speakerdeck.com/xuorig/continuous-evolution-of-graphql-schemas-at-github?slide=52) 上的访谈展示了一个 GitHub 内部使用的拉取请求机器人。如果你想移除字段并且你有客户端查询这些字段，你的更改将被拒绝。如果没人使用那个字段，好极了！去移除它吧！

![From Marc-Andre Giroux’s talk on Continuous Evolution of Schemas](https://gitee.com/caoyanbin/picgo/raw/master/img/20220314192304.png)

最终，如果你想移除字段，并且你知道你的客户端正在使用它，你应该使用@deprecated 指令废弃它，而不是移除它。

因为你知道哪个客户端请求哪个字段，当你 废弃他们使用的一些东西时，你可以主动地通知他们。在 PayPal，我们发现通过积极主动并且小幅度做出改变，产品团队更可能快速迁移。大项目以及带有复杂计划流程的大迁移会让人望而生畏并且破坏开发者的工作流。

## 应用实践

在实践中，最容易的方法是检测被每个查询调用的解析器函数。

在 Node.JS 中，你使用另一个函数封装全部的解析器函数，并且获取开始到结束的时间差值。如果你正在使用 Node.js 的 apollo 服务，你可以简单开启跟踪标识，当[创建 ApolloServer 实例](https://www.apollographql.com/docs/apollo-server/api/apollo-server.html#constructor-options-lt-ApolloServer-gt)时，然后采用响应的“跟踪”部分，并且做你想用它做的事情。

其他语言绑定（例如： Java 的[ graphql-java ](https://github.com/graphql-java/graphql-java)）也有选项！

尽管监测解析器函数是最容易开始的方式，他不是 100%精确。更好的方法是评估客户端请求了哪个查询。可能存在一种情形“父”字段（例如：user）当前被破坏，执行了一半，并且 name 解析器没有被调用。如果你使用解析器方法，你或许认为 name 字段从未被请求过并且认为移除它很安全。然后，修复 user 解析器并且破坏了客户端。当然！它是有点极端的案例，但还是安全为好。

一个更好的方法是评估客户端发送了哪个查询，而不是执行了什么解析器。这不像监测解析器那样简单，并且目前没有任何知名开源解决方案。如果这听起来像一个有趣的问题，你或许想要解决并开源，我们的社区将感谢你。

值得一提的 Apollo Engine 也是一个很好的解决该问题的付费服务。Engine 不仅能捕获字段和整体查询的时间和错误率，而且它也有一些真的很好的可视化效果。

![From apollographql.com](https://gitee.com/caoyanbin/picgo/raw/master/img/20220315135550.png)

## 继续收听

思考下？我们想要听到你团队关于插槽的想法。这是一个非常有力和有趣的话题，但创新的时机仍然成熟（有机会创新）。
