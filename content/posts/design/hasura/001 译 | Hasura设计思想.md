---
title: "001 译 | Hasura设计思想"
date: 2022-01-30T16:09:15+08:00
tags: ["技术翻译", "Hasura"]
categories:
  - "研发提效"
featured_image:
description:
draft: false
---

翻译自：[Hasura Design Philosophy](https://hasura.io/blog/how-hasura-works/)

## Hasura 设计思想

本文，我打算从产品视角谈论下 Hasura，我们的技术设计思想，与其他技术的比较，以及一些常见问题。

如果你仅仅想快速开始，我们提供了练习指导，包含多种前端框架、Hasura 后端练习，和 GraphQL 的基本介绍。

### 1，主要问题

在 Hasura 中，我们的使命是让应用开发比传统方式更快。随着技术领域的发展，我们相信这个需要被攻克的瓶颈是让数据可用。

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220130160741.png)

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

![数据库发展历程](https://gitee.com/caoyanbin/picgo/raw/master/img/20220130155321.png)

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

![Hasura实现路径](https://gitee.com/caoyanbin/picgo/raw/master/img/20220130155425.png)

让我们深入这些能力和特性的细节，我们将看到 Hasura 设计是如何应对这些技术难题的。

### 3. Hasura 关键能力

#### 3.1 分钟级生产准备

**Hasura 提供的最重要的商业价值是解锁开发者缩短上线时间**

我们做 Hasura 唯一要考虑的事情就是确保生产效率是超快的。

我们设计 Hasura 和围绕它的开发工具去确保意会 Hasura 的概念，并且尽可能快和容易的上手它。

![分钟级准备](https://gitee.com/caoyanbin/picgo/raw/master/img/20220130155523.png)

#### 3.2 数据即时 GraphQL API

当 Hasura 服务运行时，开发者能够动态配置它的元数据。有多种方式可以修改配置，例如 UI、API，或者使用“应用到”Hasura 的 git 仓库中的代码集成 CI/CD pipeline 。

元数据捕获了一些关键内容：

- 1. 连接数据源
- 2. 将模型从数据源映射到 API
- 3. 在相同数据源和跨数据源间模型之间的关系
- 4. 模型或方法级别的鉴权规则

使用元数据，Hasura 生成 JSON API 描述，并且提供可被消费的 web API。

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220130155553.png)

Hasura 通过 GraphQL 提供统一的 JSON API。GraphQL 是相当合适的，因为他原生就是 JSON。

作为一种 API 规范它能够使用多种方法满足操作大量数据模型的灵活性需求，同时提供 web 服务所需的可控边界。

尽管 Hasura 以 web 服务的形式呈现，但它实际上是一个 JIT 编译器，Hasura 通过 HTTP 接受传入的 GraphQL API 调用，然后当把数据查询给下游数据源的时候，尽可能实现理论上最好的性能。

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220130155809.png)

Hasura 能够完全避免实现 GraphQL 后端常见的 N+1 问题。它也能主动记忆，并且向上游数据源发出最少数量的请求。

要深入理解 Hasura 是如何通过 Postgres & SQL 实现这些的，请阅读：[GraphQL 到 SQL 的高性能架构。](https://hasura.io/blog/architecture-of-a-high-performance-graphql-to-sql-server-58d9944b8a87/)

#### 3.3 内置权限

Hasura 的元数据和查询引擎能够有效的将全部数据表示为一个统一可查询的 JSON 文档。

然而，数据权限不能被自动化和处理，除非没有嵌入权限规则（不需要授权）。

开发者将在不安全的环境中使用数据。不像很多年前，数据存在银行大楼中，并且 Java 应用程序运行在可信客户的机器中。

如今作为数据的所有者，我们想要给开发者访问正确范围内数据的权限（和方法），无论他们在哪或者他们打算用数据做什么。

| 开发者类型                  | APIs 类型                                          |
| --------------------------- | -------------------------------------------------- |
| 组织外的开发者              | 公共 APIs                                          |
| 不同团队的开发者            | 带有治理和 Qos 控制的网络 APIs                     |
| 为外部用户构建 APP 的开发者 | 面向外部的 APIs,但带有严格范围限制避免终端用户滥用 |

**Hasura 有一个授权引擎，允许开发者在模型级别嵌入生命行约束。**

这允许 Hasura 自动使用正确的权限规则，对于任何给定的 API 调用，可以通过关系访问模型列表，模型的部分字段，模型等。

![内置权限](https://gitee.com/caoyanbin/picgo/raw/master/img/20220130160357.png)

由于这些权限是声明式的，所以（通过 APIs）在运行时能够完全可用，它允许开发者和 Hasura 为授权系统去提供额外的可观察性。例如：

- 1.授权审计：终端消费者在特定 API 被调用时使用或触发的那些授权规则
- 2.查看或管理授权规则的工具

Hasura 的授权系统与数据库引擎提供的行级别安全的声明系统类似，但是把这个能力带到了应用层。值得一提的是，很多年前，我们构建第一个版本的 Hasura 鉴权系统时，Postgres RLS 也正好发布。多么愉快地巧合呀！

授权规则是能够贯穿 JSON 数据图任何属性和方法的条件：

- 数据及其关联关系的任何属性
  - 例如：如果`document.collaborators.editors`包含` current_user.id`允许访问
- 访问数据用户的任何属性
  - 例如：如果`accounts.organization.id`等于`current_user.organization_id`允许访问
- 规则能够屏蔽、标记或者加密数据模型或方法返回的被标记数据的部分。这个标签通常被称为“角色”。

这允许 Hasura 提供 RBAC、ABAC、分层多租户权限等。例如，这个博文介绍了你如何用 Hasura 实现一个像谷歌云 IAM 一样的授权系统： https://hasura.io/blog/authorization-rules-for-multi-tenant-system-google-cloud/

![授权系统](https://gitee.com/caoyanbin/picgo/raw/master/img/20220130160432.png)

声明式授权系统有极大的优势，因为他允许 Hasura 自动执行一些优化工作：

- 1.事务级的授权+数据访问：常用的数据查询和授权规则核查在同一个系统中实现。这允许数据查询和授权核查“原子性”的发生，这样在查询数据时授权规则就不会失效。
- 2.授权谓词下推[]：只要有可能，Hasura 可以在数据查询中自动下推授权核查。这极大的有利于提升性能，并且避免额外的查询。

_注：谓词下推 Predicate Pushdown（PPD）：简而言之，就是在不影响结果的情况下，尽量将过滤条件提前执行。谓词下推后，过滤条件在 map 端执行，减少了 map 端的输出，降低了数据在集群上传输的量，节约了集群的资源，也提升了任务的性能。_

- 3.授权缓存：Hasura 能根据需要自动更新和缓存授权信息或授权检查，（之前）这是在热查询时人必须做的工作。
- 4.自动私有数据缓存：因为 Hasura 能“思考”授权规则，Hasura 也能自动确定来自不同用户的多个 GraphQL 查询是否最终返回相同的数据。应用级别缓存这部分，原本由开发者手工构建，现在能够被 Hasura 自动处理。

尤其，Hasura 没有提供一个内置的授权构件。在很多现存的环境中，尤其是企业，授权服务经常是一个外部的系统。随着 SAAS、开源身份管理系统以及授权即服务提供商的崛起，Hasura 现在可以继续解决它最擅长的问题。例如，Hasura 可以和 Keycloak, Okta, Auth0, Azure AD, Firebase Auth 无缝对接，并且能完全自定义像 hasura-backend-plus 一样提供授权服务的社区解决方案。上面提到的一点也不详尽，而是打算介绍下现存的且 Hasura 能够很好交互的各种授权方案，

#### 3.4 应用级别缓存

应用级别缓存是这个最主要的原因之一，为什么开发者经常需要手动构建需要提供数据访问的 web APIs。当然，API 调用是及其频繁和昂贵的，计算数据查询对上游数据源来说也是及其昂贵的。

开发者使用知道什么查询需要缓存的领域知识去构建缓存策略，为某些用户或用户组，使用像 Redis 的缓存存储提供 API 缓存。

因为 Hasura 的元数据配置包含所有数据模型和授权规则的详细信息，授权规则包含了哪些用户能访问哪些数据地信息，[Hasura 能针对动态数据自动实现 API 缓存](https://hasura.io/graphql/caching/)。

![自动缓存](https://gitee.com/caoyanbin/picgo/raw/master/img/20220130154453.png)

这个问题的第二部分缓存校验是一个复杂的问题。通常，为了避免担心缓存失效和一致，作为开发者我们使用基于缓存的 TTL，并且让 API 消费者去处理这个不一致问题。

Hasura 与数据源深度交互，并且因为所有数据访问都需要通过 Hasura 或者使用数据源 CDC(数据捕获) 机制，理论上 Hasura 也能提供自动缓存失效。这块的缓存问题与“实例化视图更新”相似。

我们已经迈出了自动化应用级缓存的第一步，并且有很长的路要走，但是这个前景和潜力是非常让人振奋的：

关于构建自动缓存策略，我曾在 GraphQL 提交的几个月前，给出了详细的讨论。如果你想要了解更多关于 Hasura 缓存我们是如何思考的，那可能会引起你的兴趣。

![An Approach to Automated Caching for Public & Private GraphQL APIs
](https://www.youtube.com/watch?v=HJPYnUT5unw)

#### 3.5 驱动无状态的业务逻辑

在我们所拥有的微服务和多语言数据领域，驱动无状态的业务逻辑正开始不断变得越来越困难

事件驱动模式已成为一个解决方案去处理大容量事务负载和编排一系列业务逻辑操作，这些行为只执行一次并且处理失败不需要回滚。

这些模式也允许业务逻辑尽可能的无状态化。 业务逻辑类似在事件中被调用的”纯计算“。在执行期间所需的任何数据都通过 http APIs 访问。

然而，这样做所需的管道对于入门来说是让人望而却步的。（意思是很难）

Hasura 实现了一个事件系统，为开发者提供容易的方式去写无状态业务逻辑，并且通过 HTTP 的方式与 Hasura 集成。

![事件机制](https://gitee.com/caoyanbin/picgo/raw/master/img/20220130160520.png)

Hasura 捕获并且通过 HTTP 投递事件，同时确保（至少 1 次、正好 1 次）去帮助创建工作流，这可以让开发者像在有状态整体中下写代码那样高效的完成无状态业务逻辑。

** Hasura 的事件集成：**

- 1.与数据源的改变数据捕获集成实现自动捕获，并且通过 HTTP 可靠投递数据事件。[阅读更多](https://hasura.io/docs/latest/graphql/core/event-triggers/index.html#event-triggers)
- 2.API 事件：终端用户触发的 API 调用映射到事件上，通过 HTTP，事件被投递到后端业务逻辑中。Hasura 能提供异步或同步边界。[阅读更多](https://hasura.io/blog/introducing-actions/)
- 3.基于时间的事件：Hasura 能基于定时器或者时间戳触发事件，并且通过 HTTP 投递他们到业务逻辑中。[阅读更多](https://hasura.io/docs/latest/graphql/core/scheduled-triggers/index.html)

例如，一个数据改变事件能被 Hasura 捕获并且被投递到一个由无服务函数或微服务支持的 REST 端点上。

**支持传统事务：**然而，当一个 API 消费者与支持事务的单数据源交互时，Hasura 也能通过 websocket 和 GraphQL 导出事务。这允许业务逻辑作者用通常的方式去使用事务，如果需要的话逐步过渡到基于事件的系统。

#### 3.6 实时 APIs(又名订阅)

Hasura 与数据库集成并且为终端 API 消费者提供一个可扩展实时 API。这种类型像一个 WATCH API（观察 API）或者一个 GraphQL 订阅能观察特定资源的变化。

由于它的授权系统，Hasura 能确保它能将正确的事件投递到正确的消费者。

在生产环境中管理基于 websockets 的实时 APIs 是出名痛苦的，因为他们是令人吃惊的有状态并且昂贵。websockets 也需要额外实现网络安全层。

阅读更多关于 Hasura 如何通过 Postgres 提供一个可扩展的订阅 API：https://github.com/hasura/graphql-engine/blob/master/architecture/live-queries.md

![实时订阅](https://gitee.com/caoyanbin/picgo/raw/master/img/20220130160558.png)

#### 3.7 变革管理和 CI/CD

如果你熟悉 [JAMStack](https://hasura.io/blog/best-practices-jamstack-projects-production-scale/) 生态系统，你将理解”预览 URLs“的概念。在各个开发阶段，有一些端点托管静态 APP 版本。在开发、QA 等阶段，这对利益相关者是非常有用的。

蓝绿发布或 A/B 部署也是一种乐趣，因为在专用的端点上那是可用的静态资源。通过一些简单的配置，DNS 规则或者负载均衡配置，你能从这些能力中获益，去同时发布你 APP 的多版本，并在他们之间分配流量。

Hasura 的目标是通过动态 APIs 提供这些能力。

因为 Hasura 是一个 API 系统，他能完全在运行时配置，并且能在外部系统中分离它的元数据配置，你能轻松完成下列事情：

- 1.开发一个你 API 的新版本，针对相同的数据源去预览一个元数据改变（比如新的模型映射或者授权规则）
- 2.针对相同的数据源运行多个 Hasura 版本去确保一个可靠的推出（TODO）。

![部署对比](https://gitee.com/caoyanbin/picgo/raw/master/img/20220130160644.png)

#### 3.8 联邦数据&控制面板

以防你错过它，Hasura 的元数据系统完全是动态的，并且可以在运行时通过 APIs 配置。

我们正在基于 Hasura 的元数据 APIs 构建一套授权系统。这允许你作为一个 API 的所有者去联合控制元数据给相关数据源的所有者。（TODO）

你的在线数据早已横跨多个需要被联合的数据源。Hasura 通过提供缺少的环节，一个联合控制面板让这变得更容易。

阅读更多关于 Hasura 和远程关联的数据联邦内容，[前往](https://hasura.io/blog/remote-joins-a-graphql-api-to-join-database-and-other-data-sources/)

#### 3.9 不局限特定的语言、框架

Hasura 以服务的形式呈现它自己，并且 提供与其他服务的集成点，例如其他数据系统的网络原生链接、消费 API 或者驱动业务逻辑的 HTTP。

这允许 Hasura 用户使用他们最喜欢的语言、运行时、框架或者主机供应商。

这也允许我们在相对独立又解耦的环境中构建 Hasura，同时鼓励采用最新和最优秀的技术趋势。

#### 3.10 空操作

作为一个产品，我们的的愿景是允许开发者尽可能快的进入”无操作“的未来，这也是至关重要的去确保 Hasura 提升商业价值。

因为 Hasura 是一个独立的服务，我们能构建一个能运行的自动伸缩实践，与 APM 和追踪工具建立正确的集成，这样这些数据 API 服务不会带来沉重的操作重担。

甚至更重要的是，Hasura 能够与上游数据源需要的健康核查、可用性、复用性、可伸缩以及断路模式 进行 集成。

### 4. FAQ

#### 4.1 如果我的数据模型改变了怎么办？将 API 和数据源结合起来是一个坏主意吗？

Hasura 不是从从它的数据库中派生 API 的。严格的说，Hasura 从用户配置的元数据中派生它的 APIs。这允许 Hasura 直接处理暴露数据模型产生的所有问题，并从这个方法中获益。

数据模型是要变化的，主要有下面几个原因：

- 1.数据模型随着业务进化（增加或减少）
- 2.为了让写数据更容易而规范化
- 3.为了让读数据更容易而去规范化
- 4.完全的改变数据库

当需要维持终端 APIs 不变时，Hasura 的元数据引擎允许开发者重新定义模型映射。虽然可以使用数据库视图概念，也能在不需要任何额外的 DDL 的情况下实现这些，因为 Hasura 元数据也包含这些正确的映射关系。例如，加入一个完整的表不存在了，Hasura 的元数据能够继续暴露相同的 API 模型的同名查询（类似视图），但是现在从多个表而不是一张表中获取数据。

另一个例子，尤其在 Postgres 上，我们能创建相同的 API，无论你是使用标准表还是带有的 JSON 字段的非标准表。

#### 4.2 我应该如何定制 Hasura 的 APIs?

Hasura 使用你提供的元数据自动产生一些列必须的 APIs，然后允许你通过混合业务逻辑去扩展 API。

Hasura 鼓励 CQRS（命令查询职责分离） 模型，同时提供底层管道增加自定义 APIs ，你也能用自己喜欢的语言、框架、主机提供商增加 API，并且把它 以 REST 路由的方式插入到 Hasura 中。

#### 4.3 Hasura 强制消费者使用 GraphQL 吗？

基于受欢迎的需求，尤其是企业用户，我们将很快增加对 REST APIs 的支持，就像 Hasura 早已提供的 GraphQL 一样。

保持留意！

#### 4.4 对于现存的应用我应该如何使用 Hasura?

我们设计 Hasura 可以立即为任何现存的在线数据存储增加价值。你可以将 Hasura 指向现存的数据源（数据库、GraphQL 或 REST），并且增加一些配置，开始使用它。

很常见，我们的用户使用 Hasura 去增加高性能的 GraphQL 查询和订阅去支持他们现存的应用程序，并且继续使用他们现存的 APIs 去写入数据。

Hasura 为了增加采用被设计，不需要推到重来或者完全重构技术栈。如果你需要帮助思考你的架构，请与我们联系。

#### 4.5 我使用某种网络框架去构建 APIs。我应该如何使用 Hasura？

同 4.2 和 4.4！

### 5.在该领域与其他技术的对比

很快到来。我们将仔细研究 Hasura 如何在你的技术栈中与其他技术对比，以及 Hasura 的优缺点。
