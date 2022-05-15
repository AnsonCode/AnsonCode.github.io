---
title: "028 译 | 基于Thunk的解析器：如何构建强大和灵活的GraphQL网关"
date: 2022-04-28T23:59:45+08:00
tags: ["技术翻译", "WunderGraph", "GraphQL"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

翻译自：[Thunk-based Resolvers: How to build a powerful and flexible GraphQL Gateway](https://wundergraph.com/blog/thunk_based_resolvers_how_to_build_a_powerful_and_flexible_graphql_gateway)

WunderGraph 的核心是一个非常强大和灵活的 GraphQL API 网关。今天，我们将深入驱动它的模式：基于 Thunk 的 GraphQL 解析器。

学习很多关于基于 Thunk 的解析器模式将帮你理解你可以如何构建你的自定义 GraphQL Gateway。如果你熟悉 Go，你也不必从头开始，因为你可以基于我们的[开源框架](https://github.com/jensneuse/graphql-go-tools)构建。如果你想要用其他语言实现这个主意，你仍然能够学到很多这个模式，因为本文不会关注 GO 的实现。

开始前，让我们定义 GraphQL API Gateway 的目标：

- 它应该容易配置和扩展
- 它应该能够协调不同的服务和协议
- 它应该可能没有代码生成和编译步骤就可以重新部署
- 它应该支持模式缝合
- 它应该支持 Apollo Federation
- 它应该支持订阅
- 它应该支持 REST APIs

构建能支持这些要求的 GraphQL API Gateway 是一个真的挑战。为了更好的理解这个问题以及基于 Thunk 的解析器如何帮助解决它，首先让我们看下介于“普通”解析器和“基于 Thunk”解析器的不同。

## 普通的 GraphQL 解析器

```js
const userResolver = async (id) => {
  const user = await db.userByID(id);
};
```

这是一个简单的用户解析器。它采用用户 ID 作为参数并且返回用户对象。需要注意的是，如果你执行它，这个函数返回数据。

## 基于 Thunk 的 GraphQL 解析器

在另一方面，基于 Thunk 的解析器不立即返回数据。反而，它返回一个你能稍后执行的函数。

这儿有一个示例：

```js
const userResolver = async () => {
  return async (root, args, context, info) => {
    const user = await db.userByID(args.id);
    return user;
  };
};
```

这是一个基于 Thunk 的解析器。它不立即返回数据，它返回一个能稍后使用的函数去加载用户。

然而，这是一个基于 Thunk 解析器在实际情况下的极端简化。

为了增加更多上下文，我们现在看下 GraphQL 服务通常如何解析 GraphQL 查询。

## 一个普通的 GraphQL 服务

在普通 GraphQL 服务中， GraphQL Query 执行流看起来像这样：

1. 解析 GraphQL Query
2. 归一化
3. 校验
4. 执行

在执行阶段，GraphQL 服务将执行解析器并且组装结果。

假定一个简单的 GraphQL Schema……

```gql
type Query {
  user(id: ID!): User
}
type User {
  id: ID!
  name: String!
  posts: [Post]
}
type Post {
  id: ID!
  title: String!
  body: String!
}
```

……带有两个 GraphQL 解析器……

```js
const userResolver = async (userID: string) => {
  const user = await db.userByID(userID);
  return user;
};
const postResolver = async (userID: string) => {
  const post = await db.postsByUserID(userID);
  return post;
};
```

……并且有接下来的 GraphQL 查询：

```gql
query {
  user(id: "1") {
    id
    name
    posts {
      id
      title
      body
    }
  }
}
```

这个查询的执行将看起来像这样：

1. 遍历字段“user”，并使用参数”1“执行”userResolver“
2. 解析”user“对象的字段”id“和”name“
3. 遍历字段”posts"，并使用参数“1”执行“postResolver”
4. 解析”post"对象的字段“id”，“title"和”body“

一旦执行，响应看起来像这样：

```json
{
  "data": {
    "user": {
      "id": "1",
      "name": "John Doe",
      "posts": [
        {
          "id": "1",
          "title": "Hello World",
          "body": "This is a post"
        }
      ]
    }
  }
}
```

这应该有足够的上下文去理解构建 API 网关问题。假定你想要构建支持上述 GraphQL Schema 的 GraphQL API 网关。你应该能够将你的 GraphQL API Gateway 指向你的 GraphQL 服务，并且它应该能够执行 GraphQL 查询。

记得，网关应该没有生成代码或者任何编译步骤就能工作。这意味着，我们补鞥呢像我们在之前示例中做的那样创建解析器。这是因为，网关不能提前知道大将要使用什么 Schema。它需要足够灵活去支持任何你想要使用的 Schema。

所以，我们如何解决这个问题？这是基于 Thunk 解析器发挥作用的地方。

## 基于 Thunk 的 GraphQL 服务

基于 Thunk 的 GraphQL 服务不同于普通 GraphQL 服务，它们将 GraphQL Query 执行分成两个阶段：计划和执行。

在计划阶段，GraphQL Query 将被转换为执行计划。一旦计划好，执行计划可以被执行。值得注意的是，计划可以被缓存，所以未来相同查询的执行不必再次精力计划阶段。

为了让这尽可能容易理解，让我们从执行阶段倒推计划阶段。

## 基于 Thunk 的 GraphQL 服务的执行阶段

执行阶段依赖于执行计划。因为这些计划相当复杂，为了示例的目的我们使用简化版本。

我们实际上可以使用上面的响应，并将其转换为执行计划。

```json
{
  "data": {
    "__fetch": {
      "url": "http://localhost:8080/graphql",
      "body": "{\"query\":\"query{user(id:\\\"1\\\"){id,name,posts{id,title,body}}}\"}",
      "method": "POST"
    },
    "user": {
      "__type": "object",
      "__path": ["user"],
      "fields": [
        {
          "id": {
            "__type": "string",
            "__path": ["id"]
          },
          "name": {
            "__type": "string",
            "__path": ["name"]
          },
          "posts": {
            "__type": "array",
            "__path": ["posts"],
            "__item": {
              "__type": "object",
              "__fields": [
                {
                  "id": {
                    "__type": "string",
                    "__path": ["id"]
                  },
                  "title": {
                    "__type": "string",
                    "__path": ["title"]
                  },
                  "body": {
                    "__type": "string",
                    "__path": ["body"]
                  }
                }
              ]
            }
          }
        }
      ]
    }
  }
}
```

坦白的说，执行引擎创建一个 JSON 响应。它遍历数据字段并且执行 fetch 操作。一旦执行 fetch 操作，它遍历剩余字段并从响应中提取数据。用这种方式，最终构建响应。

注意以下几点：

- 硬编码 userID 为"1"。实际上，这将是一个动态值。所以在真是世界中，你必须将这转变成一个变量并且把它注入到查询中。
- 另一个介于真实和上述示例的不同是，你将通常发起多个嵌套 fetch 操作
- 获取也需要并行或者批量
- Apollo Federation 和 Schema Stitching（模式缝合） 增加了额外的复杂性
- 与数据库交互意味着它不仅是获取而且也有更复杂的请求
- 订阅需要完全不同的对待，因为他们打开了数据流

好了，所以现在我们有了一个执行计划。正如我们之前提到的，我们正在逆向工作。现在让我们移动到计划阶段并且看如何从 GraphQL Query 中生成这样一个执行计划。

## 基于 Thunk 的 Graphql 服务的计划阶段

对于执行计划的第一个要素是配置。配置包含所有关于 GraphQL Schema 以及如何解析查询 的信息。

这儿有一个配置的简化版本：

```json
{
  "dataSources": [
    {
      "type": "graphql",
      "url": "http://localhost:8080/graphql",
      "rootFields": [
        {
          "type": "Query",
          "field": "user",
          "args": {
            "id": {
              "type": "string",
              "path": ["id"]
            }
          }
        }
      ],
      "childFields": [
        {
          "type": "User",
          "field": "id"
        },
        {
          "type": "User",
          "field": "name"
        },
        {
          "type": "User",
          "field": "posts"
        },
        {
          "type": "Post",
          "field": "id"
        },
        {
          "type": "Post",
          "field": "title"
        },
        {
          "type": "Post",
          "field": "body"
        }
      ]
    }
  ]
}
```

坦白的说，配置包含下列：如果查询以根字段“user”开始，数据源将解析它。数据源还负责在"User“类型上的子字段“id”“name"和”posts"，以及在"Post"类型上的字段“id”“title"和”body"。

正如我们知道的，它是“graphql”类型的上游，我们知道我们需要创建 GraphQL Query 去获取数据。

Ok，我们有了这些配置。让我们看下“解析器”。

你如何将 GraphQL Query 转换为我们上面看到的执行计划？

我们通过遍历每个 GraphQL Query 节点做这些。这儿有一个简单的示例：

```js
const document = parse(graphQLOperation);
visit(document, {
  enterSelectionSet(node: SelectionSetNode) {
    // open json object
  },
  leaveSelectionSet(node: SelectionSetNode) {
    // close json object
  },
  enterField(node: FieldNode) {
    // add field to json object
    // if it's a scalar, add the type
    // if it's an object or array, created nested structure
    //
    // if it's a root field, create a fetch and add it to the field
  },
  leaveField(node: FieldNode) {
    // close objects and arrays
  },
});
```

正如你看到的，我们不解析任何数据。反而，我们“遍历”所有解析的 GraphQL Query 的节点。我们正在使用访问者模式，因为它允许我们用可预测的方式访问所有字段和片段集。walker 首先遍历所有节点，并在访问者（visitor）进入或离开节点时调用“回调”。

当遍历节点时，我们可以查看配置，看是否我们需要从上游获取数据。如果我们需要，我们创建一个查询并将其添加到字段中。

所以，这个程序做两个事情。首先，它创建我们将返回给客户端的响应结构。其次，它配置我们将执行获取数据的操作。

由于这些操作仅仅是返回数据并且稍后执行的函数，我们也可以把他们称作“Thunks”（形实转换程序）。这就是该方法得名的原因。

## 构建基于 Thunk GraphQL 框架的挑战

第一次看起来很简单，但确实真的是很难的挑战。我打算提及一些当我们构建该框架时遇到的难题。
你不能只是将数据源“附加”到字段类型元组。

这是第一次最耗时的学习。当构建“引擎”第一个迭代时，我认为直接将数据源附加到类型组合和字段上是可以的。类型元组合字段在一开始看起来似乎是唯一的。
很好，问题是 GraphQL 允许递归操作。例如，如果你有一个类型：User，你可以有一个返回 Post 对象数组的字段“posts”。在另一方面，Post 对象有一个字段"user"，返回 User 对象。

意思是你可以多次有完全相同的类型和字段组合。所以，如果你真的像唯一标识数据源，你需要在操作中考虑类型，字段和路径。

我们如何解决这个问题呢？我们打算“遍历”操作两次。

在第一次遍历中，我们确认我们需要多少数据源，并且实例化他们。第二次遍历用于实际构建执行计划。采用这个方式，“执行计划者”不必担心数据源的边界。

这也是为什么我们决定在规划器配置中设置“根节点”和“子节点”。他们一起定义了每个数据源的边界。如果你在第一次遍历中进入“根节点”，你实例化一个数据源。然后，只要你停留在“子节点”中，就仍然由同一个数据源负责。一旦遇到非子节点的节点，字段的职责将被移交给下一个数据源。

> 手工配置基于 thunk 的解析器几乎不可能。

基于 thunk 解析器最脆弱的部分是配置。如果配置中缺少根或者子节点，操作的执行时不可预测的。

GraphQL Schema 中的一个拼写错误，就会导致整件事的崩溃。这是因为 GraphQL Schema 和 DataSource 配置紧密耦合。他们必须完美匹配，所以手工创建它们不是好主意。

我们将在文章结尾单独用一段来讨论这个问题。

> 你应该区分下游和上游模式。

我们曾面临的另一个问题是扩展方法。对于一个模式，它很简单，但是将很多模式组合起来呢？

连接多个 GraphQL Schemas 是超级强大的，因为它允许你使用一个 GraphQL 操作从多个数据源查询和组合数据。

与此同时，这样做也有一些挑战。一个大的问题域是“命名冲突”，这将经常发生。不用为此想太多，大多数（如果不是全部）模式都有一个类型命名“User”。很明显，Stripe 的 User 与 Salesforce 的 User 不是同一个事情。

所以，像这样一个工具不可避免的事情是，必须考虑解决命名冲突。
我们的第一个方法是创建一个允许用户重命名类型的 API。它能工作，但是它并不是非常用户友好。你总会遇到问题，必须在组合两个模式前手工重命名类型。

另一个问题是对于在根类型 Query, Mutation 和 Subscription 的字段上你将也有命名冲突。首先，我们创建另一个 API，也允许用户重命名字段。然而，该方法与前一个一样用户友好，它是乏味的过程，并且可扩展性不强。

使用这些手工步骤意味着你必须重命名类型和字段，无论 schema 何时改变。

我们回到手绘板上，并且搜索一个能允许我们自动组合多个模式的解决方案，完全不用人工干预。

我们提出的解决方案是“[API 命名空间](https://wundergraph.com/docs/overview/features/api_namespacing)”。通过允许用户为每个 API 选择命名空间，我们可以使用命名空间自动为所有类型和字段增加前缀。该方式，所有命名冲突被自动解决。

也就是说，“API 命名空间”不是免费的。

想象下面的 GraphQL 模式：

```gql
type User {
  id: ID!
  name: String!
}
type Anonymous {
  name: String!
}
union Viewer = User | Anonymous
type Query {
  viewer: Viewer
}
```

如果我们使用前缀“identity”命名这个模式，我们最终将获得这个模式：

```gql
type identity_User {
  id: ID!
  name: String!
}
type identity_Anonymous {
  name: String!
}
union identity_Viewer = identity_User | identity_Anonymous
type Query {
  identity_viewer: identity_User | identity_Anonymous
}
```

现在，让我们假定一个“下游”Query，一个来自客户端到“网关”的查询。

```gql
query {
  identity_viewer {
    ... on identity_User {
      id
      name
    }
    ... on identity_Anonymous {
      name
    }
  }
}
```

我们不能仅发送这个查询到这个“identity”API。我们必须“去掉命名”。

这儿是上游 Query 看起来的样子：

```gql
query {
  viewer {
    ... on User {
      id
      name
    }
    ... on Anonymous {
      name
    }
  }
}
```

正如你看到的，命名 GraphQL 不是那样简单。而且它甚至能变得更复杂。

下游查询中的一些变量定义只在上游查询中使用呢？仅仅存在一些上游模式的命名指令呢？API 命名空间是一个解决该问题的很好解决方案，但实现它不是和听起来一样简单。

## GraphQL 解析器基于 Thunk 方法的好处

负面影响和挑战已经很多了，让我们也谈论下好处。

基于 Thunk 的解析器，如果做得对，意味着你可以从 GraphQL 解析器中移除很多复杂计算逻辑。静态分析允许你从热路径中移除很多代码，更高性能的执行 GraphQL 解析。

这意味着你完全可以从运行时中 移除“GraphQL”。一旦为 Query 创建执行计划，它被缓存并随后执行。在之后的请求中，缓存计划被用于代替原来的查询。这意味着，我们完全可以跳过所有执行 GraphQL 的相关部分，像解析，归一化，校验，计划等……

我们在运行时做的一切就是执行预编译的执行计划，它是一个简单的数据结构，定义了响应形状以及何时执行哪个查询。也就是说，这对我们并不足够。我们更进一步，并且干脆移除 GraphQL。在开发期间，你仍然写 GraphQL 查询和变更，但是在运行时，我们使用 JSON RPC 替换 GraphQL。

我们生成类型安全的客户端，它知道你定义的确切的 GraphQL 操作。开发这体验仍然保持相同，但是它是更高性能和安全的。这不仅是因为生成的客户端仅仅几 KB 大小，该方法也解决 13 个常见的 GraphQL 漏洞。

从这点来看，我们为什么引入虚拟图的概念应该很清晰了。如果你有一个 GraphQL API，组合了多个其他的 APIs，但是仅仅暴露一个生成的 REST/JSON RPC，叫它“虚拟”图讲得通，因为这个组合的 GraphQL Schema 仅仅虚拟存在。

虚拟图的想法是如此的强大，它允许你使用单个统一 GraphQL schema 合并一系列异构 APIs，并且 使用普通 GraphQL Operations 与他们交互。从多个 APIs 连接和缝合数据，就像他们是同一个。一个针对所有服务、APIs 和数据库的统一接口。

基于 thunk 方法的另一个益处是当批量获取时，你可以更加高效。数据加载器是一个等待一小段时间批量获取多个请求的技术。

使用基于 thunk 的解析器，你不必等待“批量窗口”去填充。你可以使用静态分析将批量处理插入到执行计划中。这也意味着你实际上可以在单个“线程”中批量获取多个请求。

这一点很重要，因为多线程的同步是非常复杂的。想象你正在解析一个数组的 10 个子字段，并且对每个数组，你必须发起另一个可以批处理的取回。

使用数据加载器模式，10 线程将被阻塞直到批处理被解析。在另一方面使用静态分析，单线程可以同步解析所有 10 个字段。

当“遍历”所有批处理兄弟节点的第一个字段时，它早已通过静态编译（在编译时）知道需要一个批请求。这意味着，我们可以立即创建批请求并且执行它。一旦查询，我们可以继续同步解析所有兄弟节点字段。

使用基于 thunk 的方法，在这个场景，意味着我们可以更好地利用 CPU，因为我们不创建 CPU 线程必须彼此等待的场景。

## 基于 thunk 的解析器不能替代“经典”解析器方法

有了基于 thunk 方法解析器的所有益处，你或许思考你应该使用基于 thunk 的方法替换所有现存的 GraphQL 解析器。

虽然这是可能的，基于 thunk 解析器不是真的意味着替换“经典”解析器方法。这是一种对构建 API 网关和代理有意义的技术。

如果你正在构建中间件，使用这个方法有很多意义，因为你早已有了 API 实现。

然而，仅仅使用基于 thunk 的解析器构建 GraphQL 服务真的很难，并且我不推荐它。

## 我们如何使基于 thunk 解析器配置更加容易

至此你已经学习了很多关于 thunk 的解析器，并且要明确的一点是，配置他们对于正确执行是至关重要的，但是很难手动做这些。这是为什么我们决定在不牺牲灵活性的基础上它需要适当级别的抽象。

我们对于该问题的解决方案是创建一个允许你用几行代码为基于 Thunk 解析器“生成”配置的 TypeScript SDK。

让我们看一个使用 SDK 组合 3 个 APIs 的示例：

```js
const db = introspect.postgresql({
  apiNamespace: "db",
  databaseURL: "postgresql://admin:admin@localhost:54322/example?schema=public",
});

const countries = introspect.graphql({
  apiNamespace: "countries",
  url: "https://countries.trevorblades.com/",
});

const stripe = introspect.openApi({
  apiNamespace: "stripe",
  statusCodeUnions: true,
  source: {
    kind: "file",
    filePath: "./stripe.yaml",
  },
  headers: (builder) => {
    return builder.addStaticHeader(
      "Authorization",
      `Bearer ${process.env["STRIPE_SECRET_KEY"]}`
    );
  },
});
const myApplication = new Application({
  name: "app",
  apis: [db, countries, stripe],
});
```

这儿，我们正内省一个 PostgreSQL 数据库，一个国家 GraphQL API 和一个 Stripe OpenAPI 规范。正如你看到的，API 命名空间是一个简单的配置字段，所有其他配置将自动生成。

想象你必须手动调整并配置 n 个 APIs 的组合。没有这个框架，配置的复杂性将或许是 O(n^2)。你增加越多 APIs，配置将越复杂。如果单 API 改变，你必须再一次浏览所有步骤重新配置并测试所有东西。

与使用 SDK 的自动化方法形成对比。配置的复杂性看起来更像: O(n)。对于你增加的每个 API，你只需要调整一次配置。

如果原始 API 改变，你可以重新运行内省程序并且更新配置。这甚至可以使用 CI/CD 流水线实现。

如果你对该方法感兴趣，看下我们的快速入门指南，它会使你在几分钟内在本地机器上运行。

## 我们满足需求了吗？

让我们回过头，看下我们定义的需求，看是否我们已经满足了它们。

### 它应该很容易配置和扩展

正如我们展示的，GraphQL Engine 和 Configuration SDK 的组合构成了非常灵活的解决方案，同时让它很容易去配置和扩展。

### 它应该能够协调不同的服务和协议

中间件实现 GraphQL 和客户端的通讯。实际上，它通过 HTTP 使用 JSON RPC，但是让我们先忽略这些。

The Middleware speaks GraphQL to the client. Actually, it speaks JSON RPC over HTTP but let's ignore this for a moment.

基于 Thunk 的解析器允许我们用一个方式实现计划和执行阶段，我们能协调 GraphQL 和任何其他协议，例如 REST, GraphQL, gRPC, Kafka 甚至 SQL。

在“计划阶段”仅仅聚合所有你需要的信息，例如，数据库的表和字段，或者 Kafka 服务的主题和分区，然后再“执行阶段”执行确实与数据库或 Kafka 通讯的 Thunk。

### 应该能够在没有代码生成、编译步骤的情况下重新部署

每次你想改变 schema 时必须重新编译和部署 GraphQL Engine 将会是一个巨大的痛苦。正如我们之前所描述的，计划阶段的输出，在内存中缓存“执行计划”，甚至可以序列化/反序列化。这意味着，我们可以不需要编译步骤改变配置，并且水平缩放引擎。

### 它应该支持 schema 缝合

回想下 GraphQL Engine 如何工作，我们之前描述了你必须定义 GraphQL Schema，然后将数据源附加到类型元组和字段。这是开启 schema 缝合的唯一要求，意味着它开箱即用。

### 它应该支持 Apollo Federatio

用理解 Apollo Federation 规范的方式实现 GraphQL 数据源（它是全部开源的）。你只需要向数据源传递正确的配置参数，剩下的部分自动生效。

这儿是如何配置它的例子：

```js
const federatedApi = introspect.federation({
  apiNamespace: "federated",
  upstreams: [
    {
      url: "http://localhost:4001/graphql",
    },
    {
      url: "http://localhost:4002/graphql",
    },
  ],
});
```

GraphQL+Federation 数据源展示了基于 thunk 方法的力量。它从一个简单的 GraphQL 数据源开始，然后扩展到支持 Apollo Federation 规范。因为计划和执行阶段被分割成 2 段，它也很容易测试。

### 它应该支持订阅

订阅是当实现 GraphQL 网关时，很多开发者逃避的特性之一。尤其是，当涉及到 Federation 时，Apollo 的某个人问我，我们如何管理客户端和 Gateway(下游)之间以及 Gateway 和 subgraph(上游)之间的所有连接。

答案是简单的：分而治之。基于 Thunk 的方法允许我们将难题分成块，例如订阅

当客户端连接到 Gateway，且想要开始第一次订阅时，你完成了计划步骤，并且找出你必须发送给（数据）源什么 GraphQL Operation 。如果至今没有针对源的激活 WebSocket 连接，你使用所有 GraphQL 服务都遵循的协议初始化一个 。当你获得一些返回信息，你将它转发给客户端。如果第二个客户端连接并且想要开启第二个订阅，你可以对源复用相同的 WebSocket 连接。

一旦所有客户端断开源的连接，你可以关闭源的 WebSocket 连接。

简而言之，因为我们可以将任何我们想要的逻辑放入规划器实现和执行器实现，我们也能支持订阅。

### 它应该支持 REST APIs

相同的原则应用到 REST APIs，意味着也能很好的支持它们。REST API 数据源或许是最容易实现的。这是因为你不必“遍历”子字段去识别是否需要修改 REST API 调用。如果你对比 REST 数据源和 GraphQL 数据源的实现，你将看到前者是更简单的。

## 总结

本文，我们涵盖了构建 GraphQL API 网关的秘密：基于 Thunk 的解析器。它应该很清晰，写解析器的经典方法对于写 GraphQL APIs 更有意义，而基于 Thunk 的方法通常是是构建 API 网关，代理和中间件的更好选择。
