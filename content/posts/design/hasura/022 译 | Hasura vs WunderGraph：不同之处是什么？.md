---
title: "022 译 | Hasura vs WunderGraph：不同之处是什么？"
date: 2022-04-19T23:59:45+08:00
tags: ["技术翻译", "WunderGraph", "Hasura"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

翻译自：[Hasura vs WunderGraph comparison: What are the differences?](https://wundergraph.com/docs/overview/comparison/vs_hasura)

从高层次角度来看，Hasura 和 WunderGraph 都给你“所有数据的即时 GraphQL ”，正如 Hasura 所说。本文，我们将分解这两个方案，并逐个特性对比他们。

虽然我们可能有偏见，我们会尽力考虑每个解决方案的优缺点并尽可能保持客观。

我们尊重 Hasura 公司，他们为 GraphQL 社区做了很多事情。从产品的角度，我们认为两个遵循不同理念的解决方案在一些方面有重叠之处。
本文将发现两者相似和不同，并且想给你何时使用哪个方案的指导。

## 特性对比

### 所有数据的即时 GraphQL

Hasura 的主要焦点是在数据库上面生成 GraphQL API。他们支持 PostgreSQL, MS SQL Server, Amazon Aurora 和 Google BigQuery。然而，看这个文档，他们主要集中在 PostgreSQL 上。

在 APIs 方面，他也可能使用 REST, GraphQL 和 OData。最近，Hasura 增加支持将生成的 GraphQL API 转成 REST 端点。

WunderGraph 支持 PostgreSQL, MySQL, AWS RDS, AWS Aurora, AWS Aurora Serverless, MariaDB, SQLite, Microsoft SQL Server, Azure SQL, MongoDB and Planetscale, CockroachDB 也即将支持。在底层，WunderGraph 使用 Prisma 集成数据库。这意味着，无论何时数据源在 Prisma 中可用，我们就能在 WunderGraph 中支持。

在 APIs 方面，WunderGraph 支持 REST APIs 和 GraphQL ，包括带订阅功能的 Apollo Federation 。这意味着，WunderGraph 不仅支持 GraphQL 上游，而且还可以作为 Apollo Federation 的网关，联合多个其他服务的子图。

正如我们所知，Hasura 的远程 Schema 功能不支持订阅。使用 WunderGraph，支持 GraphQL 订阅，甚至多个 GraphQL 上游和 Apollo Federation。

至此，我们看到这两者有一些不同，但是两个工具提供或多或少相同的特性。

然而，早有一个大的不同。Hasura 围绕有数据库的想法构建。没有数据库，你不能创建 Hasura API。

在另一方面，WunderGraph 完全不依赖数据库。你完全可以从 REST 和 GraphQL APIs 中自由的组合你的 schema。这是因为我们的理念略有不同。

Hasura 把数据库作为每件事的中心。你从中心数据库 schema 开始，并且可以使用 APIs 扩展它。

WunderGraph 把 API 作为中心。一个 API 可能是任何事情，一个 REST API, 一个 GraphQL API, 一个或多个联合子图 或 一个 database。从这层意义上，数据库仅仅是另一种 API。

### 远程 Schema

Hasura 有远程 Schema 的概念。如果一些东西在远处，它意味着也有一个非远程的 Schema，中心数据 Schema。此外，有一个构建介于远程 Schema 字段和中心数据库关系的方式。

WunderGraph 不区远程或非远程 Schema 。反而，每个东西都是远程 Schema。这也意味着，你可以构建介于任何 API 之间的关系。你可以使用 REST API 组合 GraphQL API，并且构建这两者的关系。

正如早前提到的，Hasura Remote Schemas 有一些限制，例如，缺乏 Subscriptions 支持。

### 设置和配置

在配置方面，两个工具遵循完全不同的范例。

尽管 Hasura 可以通过 REST API 配置，配置的主要方法是他们的用户界面。从增加数据库到配置远程 schemas 和关系，所有的配置可以并且应该使用用户界面完成。这个模式提供了很好的上手体验，因为容易学习。

WunderGraph 没有为配置提供用户界面。这是因为我们不相信基础设置，像 APIs，应该使用 UI 配置。APIs 是一个进化的系统，他们总是改变并且前进。我们认为，跟踪所有正在发生的变化是十分重要的。我们认为你想要在不同的环境中操作 APIs，例如 开发，测试和生产。我们也认为，表单和按钮有太多限制对于配置复杂的 API 环境。

这是为什么我们决定反对通过用户界面配置。反而，WunderGraph 提供一个强大的 TypeScript SDK。这个 SDK 允许你非常容易的配置环境并且在 git 中存储配置。你可以为配置获得开箱即用的版本控制，你可以容易的审查和回滚，并且为不同环境使用不同分支变得可能。一旦变化在测试环境中测试，他可以合并到生产环境。WunderGraph 的设计就是为了支持这些流程。

所以，如果你是一些想要使用他们接口配置 APIs 的人，你或许倾向于 Hasura。如果你喜欢“基础架构即代码”，WunderGraph 更适合你。

### 身份验证

Hasura 不为你处理身份验证，所以你必须集成其他服务或自己构建解决方案。使用 WunderGraph，我们已经额外付出努力去构建一个令人吃惊的身份验证开发体验。我们不仅为身份验证支持 OpenID Connect，我们也生成一个完全类型安全的客户端，它知道如何使用配置的授权供应商验证身份。简要来说，你配置身份验证提供商，像 Keycloak, Auth0, Okta 等……这生成处理身份验证流的后端部分，以及处理客户端的客户端部分。所有你必须做的就是将`login()`函数连接到用户界面，并且你的登录就完成了。你可以在[文档](https://wundergraph.com/docs/overview/features/authentication)中阅读更多该特性。

### 鉴权

Hasura 通过基于角色的模型实现鉴权。此外，它有可能使用行级安全，对于一个如此接近数据库的工具，这是有意义的。权限可以使用 Hasura 面板配置。对于远程 schemas，事情有点难办。你可以使用 SDL 为每个角色定义特定的 schema 去限制访问。作为替代方案，他也可能将头部转发给远程 schema。

从我的角度来看，基于角色的授权似乎是一种可靠的解决方案，尤其当使用像 PostgreSQL 这样支持行级安全的数据库时。也就是说，远程 schema 鉴权是一种边缘情况，系统没有对此优化。

WunderGraph 对于鉴权有不同的选择。我们为鉴权集成了 OpenID Connect，意味着用户有一个开箱即用的身份。该身份不是被 WunderGraph 驱动。真实的来源是身份提供商。这意味着你可以从你的身份供应商中配置并管理用户。依赖你为用户赋予的角色和权限，他们将有不同的“claims”，当和 WunderGraph APIs 集成时可以被使用。

使用我们的 claims 注入特性，你能从身份提供商那里使用 claims 并直接将他们注入到 GraphQL 操作中。**[你可以在这里阅读更多 claims 注入的主题。](https://wundergraph.com/docs/overview/features/authorization_claims_injection)**让我们给你一个简短的示例，说明该特性的强大。

```gql
mutation (
  $name: String! @fromClaim(name: NAME)
  $email: String! @fromClaim(name: EMAIL)
  $message: String! @jsonSchema(pattern: "^[a-zA-Z 0-9]+$")
) {
  createOnepost(
    data: {
      message: $message
      user: {
        connectOrCreate: {
          where: { email: $email }
          create: { email: $email, name: $name }
        }
      }
    }
  ) {
    id
    message
    user {
      id
      name
    }
  }
}
```

当执行该操作时，代表用户创建了一篇文章。`NAME`和 `EMAIL`变量自动注入到操作中。

这意味着，如果你的数据库支持行水平安全（RLS)，例如 PostgreSQL，你可以注入用户的邮箱，并且让他执行。

此外，我们有钩子，允许你通过写一个 TypeScript 函数在执行操作前验证用户 claims。这意味着，你可以通过评估身份提供者定义的用户角色和权限限制任何 API，文件上传，变更等的访问。这给你完全的灵活性，同时让配置容易维护。

除此之外，WunderGraph 包含一个 RBAC 实现。当用户首次鉴权时，你可以使用 TypeScript 函数为他们分配角色。例如，你可以给他们不同的角色，基于他们的邮箱地址。由于 TypeScript 钩子只是 NodeJS，你也能够从外部系统或者甚至一个数据库中获取数据，并为用户分配角色。一旦角色被分配，你可以使用 `@rbac`指令 保护你的操作。

这个示例展示了仅仅使用`superadmin`角色的用户被允许通过用户邮箱删除消息。

```gql
mutation ($email: String!) @rbac(requireMatchAll: [superadmin]) {
  deleteManymessages(where: { users: { is: { email: { equals: $email } } } }) {
    count
  }
}
```

总结该章节，Hasura 和 WunderGraph 两者都提供很多方式为 APIs 实现鉴权。我做的唯一区分是 WunderGraphs 鉴权系统被设计和很多“远程 schemas”更好的工作，在另一方面当你想使用 PostgreSQL 原生特性时，Hasura 可以更好的工作。

### API 安全

公开发布一个 GraphQL API 意味着，你必须认真考虑安全。[我再博客文章中额外写了该主题](https://wundergraph.com/blog/the_complete_graphql_security_guide_fixing_the_13_most_common_graphql_vulnerabilities_to_make_your_api_production_ready)，如果你想更深入了解保护 GraphQL API 的复杂性。

Hasura 提供很多特性去解决这个问题。其中包括深度限制，节点限制，节点限制，速率限制，在生产中禁用内省，允许列表。

他们误解的一件事是[ CORS 的使用](https://hasura.io/graphql/security/)。CORS 不能保护服务，他可以保护用户。如果用户被授权并且访问不同的域名，CORS 允许你控制是否接收来自用户的授权信息，即使他们在不同的域名。然而，正如 Hasura 正在使用 JWT 进行身份验证，这种场景不太可能发生，因为攻击者只需要盗取 JWT 令牌。当使用基于 Cookie 的身份验证是，CORS 是有意义的。

关于 API 安全，WunderGraph 有不同的想法。我们的观点时，暴露 GraphQL API 是一个你不想让自己进入的兔子洞。它是一个猫鼠游戏。它几乎不可能证明你的整个 Schema 是安全的。即使你认为你已经做了你可以做的所有事情，黑客或许仍然发现利用你系统的机会。

代替暴露 GraphQL API 并且尝试保护它安全，我们只是公开一个 JSON-RPC API。在开发期间，你定义你需要的所有操作。然后，立即将他们转换成 JSON-RPC API。JSON-RPC API 仅仅接收变量，它受到身份验证和 JSON-Schema 验证中间件的保护。

因为 JSON-RPC 有点难以使用，我们也创建了一个代码生成器基于 JSON-RPC 生成一个 100%类型安全的客户端。我们稍后在讨论这个客户端。

### 输入校验

如上一章所述，WunderGraph 为你定义的每个 GraphQL 操作创建一个专用的端点，但这还不是全部。操作的输入和响应都能被解析为 JSON Object。WunderGraph 解析每个操作的 GraphQL AST 并且为它生成 JSON Schema。用这个方式，我们可以容易的[校验操作的输入](https://wundergraph.com/docs/overview/features/json_schema_validation)。

幸亏自定义`@jsonSchema`指令，它是可能的直接在操作定义中去扩展 JSON-Schema 校验。这儿有一个增加标题、描述和正则模式扩展 JSON-Schema 的示例。

```gql
mutation (
  $message: String!
    @jsonSchema(
      title: "Message"
      description: "Write something meaningful"
      pattern: "^[a-zA-Z 0-9]+$"
    )
) {
  createPost(message: $message) {
    id
    message
  }
}
```

该方法最美之处在于你能够在程序其他部分复用生成的 JSON-Schema，例如前端。这个最棒的一个用途是和`[react-jsonschema-form](https://github.com/rjsf-team/react-jsonschema-form)`的集成。它允许你生成完整的 React Forms，仅仅通过写一个 GraphQL 操作，因为库也可以选择 JSON-Schema。

Hasura 提出使用 Postgres 核查约束或者触发进行输入校验。另一个选项是使用自定义 Hasura Actions，这将在下个章节描述。

我个人不认为输入校验应该依赖数据库。不是所有的数据库管理系统提供相同的功能。而且，我想尽可能早地停止操作，如果输入非法。最后，我不想依赖特定数据库当构建我的应用时。在我的选择中， 在 API 网关水平的 JSON-Schema 校验是干净的方法对于输入校验问题。

### 自定义 API

当生成 API 时，自定义总是一个重要的方面。WunderGraph 和 Hasura 两者为扩展提供了不同的方式，让我们拆解他们。

首先，Hasura 有一个叫做“Action”的功能。Action 是对 Schema 的自定义添加，由 REST API 支持。所以，你可以使用 GraphQL SDL 自定义扩展然后配置 REST API 钩子实现它。 REST API 需要被用户自己开发，对开发和运维增加了一些复杂性。面板也提供一个代码生成器去简化构建 actions。

除了 Actions 之外，Hasura 也有事件和定时器触发。这些可以在特定事件中被调用，例如，当数据库中插入新行或基于计划任务定义。

最后，正如我们之前讨论的，Hasura 允许你添加 Remote Schemas。

接下来是 WunderGraph 的解决方案。首先，我们在开始已经概括了，在 WunderGraph 中没有 Remote Schemas 的概念。每个 Schemas 都是远程的。WunderGraph 也通过内省 OpenAPI Specification (OAS)支持 REST APIs 。所以，代替使用 SDL 扩展 Schema，然后使用 WebHook 实现这个更新，你也可以反过来。使用 OAS 或 GraphQL 定义 REST API 或 GraphQL API，并且简单地将它合并到图上。这样方便多了，并且容易增加现存的 REST APIs 到统一图中。

第二，WunderGraph 有一个与 Actions 相似的概念，Hooks！因为所有操作被 JSON-Schema 定义，我们能够复用 Schema 去支撑 hooks 的结构。所以，无论何时你定义一个新的操作或者修改一个现存的，我们重新生成定义 Hooks 的代码。当前，我们允许你使用 TypeScript 写 Hooks，但是如果需要的话，我们可以用任何语言生成他们。因为 hooks 的类型定义是自动生成的，你只需要实现所需 hooks，例如变更前，查询后，同步或异步，依赖你是否想要修改输入或响应。

你也不必部署 hooks。WunderGraph 自动运行 TypeScript 代码，并和网关一起运行它。Hooks 使用普通的 NodeJS，所以你可以使用任何你想用的 npm 包。所有这些使得 Hooks 十分强大，理想且容易使用。

### 模拟

APIs 生命周期的重要方面是模拟。有时，你不想测试真实的系统，或者，在开发环境，系统或许不可用。

为此，我们在 WunderGraph 中内嵌了针对模拟的第一类支持。由于每个操作有被 JSON-Schema 定义的清晰的输入和输出，对我们来说很容易生成 TypeScript 骨架，用户可以使用模拟操作的响应。

你只需要实现一个或多个函数，这样模拟就已经准备好了。使用 TypeScript 生成骨架，所以当写模拟的时候，你可以凭借类型安全。此外，由于它只在幕后使用 NodeJS，你想要实现模拟时，可以使用任何 npm 包，例如 faker js。你也不必关注部署，框架为你管理这些。

查看 Hasura 的文档，我没有发现模拟的信息。因为他们的主要用例是在 PostgreSQL 数据库之前，数据库不存在的场景非常罕见，所以在这个用例中，模拟并不重要。

我认为，尤其在你集成很多 APIs 的场景下，模拟对于开发过程是必不可少的，当在开发期间你并不总是能够使用所有“真实”APISs 时，例如当构建用户转账界面时。

### 本地开发

我们精心设计 WunderGraph，去最好的支持本地开发，并且创建良好的开箱体验。我们为每个主流平台编译了二进制，包括 MacOS arm64 和 Windows。

从开始创建一个新的 WunderGraph 项目如下面这样简单：

```bash
yarn global add @wundergraph/wunderctl
wunderctl init --template nextjs-starter
```

这将创建一个完整支持 WunderGraph 的 NextJS 应用程序，并提供大量示例。

使用 Hasura，你必须跟随多各个步骤启动并运行本地环境。两种服务在本质上提供相同的东西，只是我们在 WunderGraph 上实现了整个引导流程，使它尽可能流畅。

### 监控和可观测性

Hasura 在可观测性上提供了大量信息。查询时间，执行时间，客户端 IP 地址，识别慢查询，错误跟踪，分布式追踪，WebSocket 连接监控。

我们在 WunderGraph 上解决可观测性的计划是双重的。其一，我们想要为流行的分析和可观测性解决方案构建集成，像 Datadog, Splunk, OpenTracing 和 OpenTelemetry。其二，我们想要在未来提供管理解决方案给你一个很好的开箱即用的分析解决方案。如果你对这个解决方案感兴趣，请联系我们！

### 缓存

缓存对每个应用都是重要的部分。如果它可以帮助应用程序提升弹性和性能，应用程序的每个层应该使用缓存。

Hasura 允许使用 LRU 缓存来缓存 GraphQL 操作。他们创建一个自定义指令，允许用户缓存特定时间内的每个操作。牢记缓存层适用于 GraphQL 执行。（没太理解）

Hasura 也使用 PostgreSQL 预处理语句提升性能。通过该方式，可以缓存 SQL 查询计划，以便于复用他们。此外，它也能够缓存远程 GraphQL APIs。

最后，Hasura 建议生成 REST APIs 去启用“传统”缓存。我个人不认为它是传统的。如果你观察网络的不同层级，你会意识到浏览器，代理，CDNs。

### 实时订阅

Hasura 的核心能力之一是使用轮训在 PostgreSQL 上实现 GraphQL Subscriptions。你可以在每个操作级别配置轮训间隔，意味着你可以为你的每个操作调整“存活性”。此外，轮训不仅支队数据库生效，而且对每个数据源生效，包括 GraphQL 和 REST。

另一个区别是传输。Hasura 依赖 WebSockets，一个不再维护的 HTTP 1.1 特性。WebSockets 在 web 中提供一个非常糟糕的 API，例如他不可能使用 升级 请求发送 Headers。因此，你必须找到自定义解决方案，通过 WebSockets 实现身份验证。这些解决方案通常向浏览器公开令牌，这通常是不安全的。

在另一方面，WunderGraph 暴露使用 HTTP/2 流的 Subscriptions，如果 HTTP/2 不可用，使用块编码回退。（不是很精当）通过这种方式，我们可以使用强加密的安全 HTTP cookies，避免 API 令牌跑到浏览器之外。

Hasura 和 WunderGraph 都为实时 Subscriptions 提供了解决方案，而 WunderGraph 更关注安全性。

### 文件上传

构建盈余公意味着你不仅处理结构化数据，也包括图片,PDF 或其他形式的文件。我们意识到，使用不同系统管理文件将花费大量开销。如果那样的话，你必须实现身份验证两次，增加了系统复杂性。

不是把这个负担给我们的用户，我们付出了额外努力，为 WunderGraph 增加了原生文件管理支持。你可以接入任何 S3 兼容的文件存储，允许你上传文件到你的自建 Minio, 云端 AWS S3 或者任何其他 S3 API 兼容存储。[你可以在这里阅读更多这个主题。](https://wundergraph.com/docs/overview/features/easy_file_uploads_to_s3_compatible_storage_providers)

Hasura 决定不支持非结构化数据，而是集中在 GraphQL API。有时，专注在一件事上是一个好的决定。有了 WunderGraph，我们希望提供一个端到端的解决方案，这样我们的用户不必把太多依赖关系混在一起。

### 客户端和服务端紧密耦合

我对生成 GraphQL APIs 最大的批判之一是，他们在客户端和 API 实现上引入了强耦合。如果从数据库生成 API，数据库的改变将自动为客户端引入破坏性更改。如果你只有一个开发者或者小的团队，这没有问题，但是不能在拥有多个团队的组织中扩展。你当然想要在不破坏 API 的情况下改变存储数据的方式。

在 Hasura 中，这个问题非常普遍。改变一个表明或字段，你的 API 消费者必须处理一个破坏性更改。

正如你早已了解到的，WunderGraph 不直接暴露 GraphQL API。而是，我们使用操作定义 JSON-RPC 端点。对于每一个端点，我们能够生成一个 Postman 集合，因为他们的行为类似于具有特定输入和输出的简单 REST APIs 。

在 GraphQL Schema 前面放置这个”中间“RPC API 的额外好处是我们能够使用客户端从 API 约定中抽象这个实现。通过分析，我们能够准确说出来哪个客户端使用哪个”RPC 操作“。这使我们能够去交换 API 的完整实现，不需要破坏 RPC 约定。你可以改变数据库或者甚至使用自定义 GraphQL API 替换它，只要确保实现一个中间件，为现存客户端实现 RPC 约定，并且你可以不破坏任何事情进行更改。我们在[无版本 APIs](https://wundergraph.com/blog/versionless_apis_making_apis_backwards_compatible_forever_to_enable_businesses_to_collaborate)中对此进行了广泛的讨论。

我们认为，无版本 APIs 是使用 APIs 开启业务协作的关键。当介于他们之间的约定稳定时，跨团队或者跨公司 API 集成才可以扩展。让 APIs 长时间向后兼容是构建这些稳定约束的关键。

### 生成客户端

WunderGraph 不仅是一个生成后端的解决方案，而且带有一个代码生成器去生成 100%类型安全的客户端，与您创建的 API 一起工作。一旦你定义好你的操作，WunderGraph 同时生成两者，API 端点和客户端。我们当前支持 TypeScript 和 ReactJS 作为模板，但是你能够使用自己的自定义模板扩展这个，允许你为任何平台，语言或框架生成代码。

### 身份验证感知的数据获取

不仅生成后端，而且生成 API 客户端，带来很多额外的好处。其中之一是 身份验证感知的数据获取。

当 WunderGraph 代码生成器生成客户端时，它知道所有操作，以及他们是否需要对用户进行身份验证。开发者可以可以使用配置参数改变这些。

这个信息让它可能用一个方式生成客户端，这样它就知道一个操作是否需要身份验证。如果操作需要用户验证身份，客户端将自动”等待“，直到需求被满足。而不是尝试这个操作，导致一个 401 未授权，客户端可以立即指出，发起请求的千珏条件还没有满足。此外，一旦用户通过身份验证，Query 或 Subscription 可以立即触发去获取数据。开发者不必写额外的逻辑去处理这些状态。

而 Hasura 没有生成客户端，你必须自己实现这个逻辑。

### CSRF 保护

API 另一个重要的方面是安全。重要的是你确保用户不能陷入无意”变更“状态的困境。也就是说，你想确保 API 用户只在他们想要的时候（sends money）【不理解】。

解决这个问题有标准的方法，著名的 CSRF 保护。如果用户在你的域名中通过身份验证，CSRF 保护确保来自另一个域名的请求不会轻易发生变更。CSRF 保护不容易实现，并且需要一些关于安全的知识。

当我们生成后端和 API 客户端时，我们能够为两者嵌入 CSRF 保护。如果你使用 WunderGraph 定义变更，API 结果自动被 CSRF 保护。你不必做任何事情！

### 跨 API 连接

Hasura 和 WunderGraph 都支持跨 API 连接数据。Hasura 在 Schema 级别进行连接，WunderGraph 在操作级别进行连接。

让我们探索两个解决方案并对比他们，首先从 Hasura 开始。

Hasura 实现跨 API 连接的方式是允许用户为现存数据驱动 API 增加远程 Schema。然后，你能够配置哪个参数（外键）来连接从数据库 Schema 到远程 Schema 的字段。

结果是组合两者为一个新的 Schema，数据驱动 Schema 和远程 Schema。API 用户不会注意到他们从两个不同的 APIs 连接数据，因为它仅仅是一个单 GraphQL API。

这种方法叫做”模式缝合“，并且在社区广泛应用。

在 WunderGraph 中，我们研究模式缝合一段时间了，并且决定我们不想让他成为连接 APIs 的主要方法。WunderGraph 支持模式缝合和 Apollo Federation 两种连接 APIs 方法，但是我们也开发了一个新的方法：嵌套查询类型连接（Nested Query Type Joins）。让我解释下。

在 WunderGraph 中我们的目标是允许 APIs 成为小的构建块，能够组合其他 APIs 解决问题，像乐高积木一样。

我们试验过模式缝合，并且意识到他有一些使它很难扩展的限制。你尝试缝合越多 APIs，你增加更多复杂性。随着加入 API 越来越多，你遇到越来越多的问题，像命名冲突。为了解决这些问题，你必须增加很难维护的自定义缝合逻辑。

针对该问题我们的解决方案是双重的。首先，我们使用`命名空间`避免命名冲突。其次，我们在操作级别实现`跨API连接`。

这意味着什么？来看一个例子：

```gql
query ($code: ID!, $capital: String! @internal) {
  country: countries_country(code: $code) {
    code
    name
    capital @export(as: "capital")
    weather: _join @transform(get: "weather_getCityByName.weather") {
      weather_getCityByName(name: $capital) {
        weather {
          summary {
            title
            description
          }
          temperature {
            actual
          }
        }
      }
    }
  }
}
```

该操作通过编码获取国家，并且从天气 API 关联天气数据。我们使用一些特殊的`指令`使它工作，但是更重要的是，它是一个 100%合法的 GraphQL 操作。现存工具，例如 IDE 可以校验这个操作，提醒并且代码不全也将如期望的那样工作。

这个方法扩展性很好，因为你能够向`[virtual Graph(虚构图)](https://wundergraph.com/docs/reference/concepts/virtual_graph)`中增加尽可能多的 APIs。命名空间将保持每个 API 独立，并且你可以容易的增加新的 APIs，而不必改变现存代码。

### WunderHub——API 包管理器

WunderGraph 不仅是一个更快构建安全应用的开源框架，它也提供强大的工具支持新高度的 API 协作。

想象你正在一个团队中工作，你想要容易地与同事，整个部门甚至公司的开发者分享 APIs。

[WunderHub](https://hub.wundergraph.com/) 给你一个分享 APIs 的容易方式，并且将他们集成到你现存的应用中。它与 WunderGraph 生态系统紧密结合，提供一个伟大的协作体验。

我们不孤立的看待 APIs。APIs 是与其他开发者分享功能的关键，这些功能被压缩在小的构建块中。

WunderHub 允许你容易的分享这些构建块，允许其他人在他们之上构建。

## 技术对比

WunderGraph 和 Hasura 两者都构建在高性能语言之上，Hasura 使用 Haskell 构建，WunderGraph 使用 Golang 实现。

两个语言都是编译型的，因此都非常快。他们也都有自己的调度器，意味着异步工作负载可以快多个 CPU 进程复用，允许两个语言在多个 CPUs 上很好的扩展。与 NodeJS 相比，他们可以容易最大化利用 CPU.

在 TIOBE 排名上，Golang 排名 18 @1.21%。Haskell 排名40@0.24%。找到 Go 开发者比熟悉 Haskell 的开发者容易得多。Golang 也被用于很多大型项目，像：

- kubernetes
- cockroachdb
- etcd
- Docker
- consul
- Terraform
- syncthing

## 为什么 WunderGraph 是一个伟大的 Hasura 替代品？

如果你想做的全部就是从数据库暴露一个纯净的 GraphQL API ，Hasura 是好的选择。

然而，如果你早已知道你将在公开网络上向 WEB 客户端暴露 API，除了公开 GraphQL API 本身，还有很多工作要去做。你必须保护 API 安全，并且在该基础上构建自定义客户端，以处理缓存，身份验证，鉴权，安全数据获取，CSRF 保护，文件上传等。

使用 WunderGraph，你将得到一个维护很好的框架去为你解决这些问题。如果你正在自己解决这些问题，考虑下谁正在维护你的自定义解决方案，谁将更新 GraphQL 的客户端依赖，以及谁保证你的身份验证一定安全？

所有工作能被框架解决，框架可以集体维护。为什么使用你必须自己维护的自定义私有代码，当它可以被抽象到一个框架时？我们提供很多方式去自定义行为，但是核心不应该被个人维护。

我们的目标是标准化我们集成和消费 APIs 的方式，并且使解决方案和开源框架一样可用。通过这种方式，你可以在应用几乎所有层中使用最佳实践和验证的模式，而无需自定义实现。

## 使用 WunderGraph 上手

看下该文档或者尝试[快速上手](https://wundergraph.com/docs/guides/getting_started/quickstart)。

还想收集更多信息？ 了解更多 WunderGraph 提供的[功能](https://wundergraph.com/docs/overview/features/overview)。

了解更多关于 [API-mesh](https://wundergraph.com/overview/api-mesh) 和 [API 安全](https://wundergraph.com/overview/api-security)性的知识，以及向不同的[角色](https://wundergraph.com/overview/by_persona)解释 WunderGraph 的章节。

如果你是一个开发者，开始的最容易方式是复制&粘贴该代码片段到你的终端中。

```sh
yarn global add @wundergraph/wunderctl@latest

mkdir wg-demo && cd wg-demo

wunderctl init --template nextjs-starter

yarn && yarn dev
```
