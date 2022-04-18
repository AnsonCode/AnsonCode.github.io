---
title: "013 译 | 使用Hasura作为Nuxt.js的BaaS"
date: 2022-02-18T22:22:45+08:00
tags: ["技术翻译", "Hasura"]
categories: ["研发提效"]
featured_image:
description:
draft: true
---

翻译自：[Using Hasura as BaaS for Nuxt.js](https://hasura.io/blog/using-hasura-as-baas-for-nuxt/)

本文由 [Migsar Navarro](https://twitter.com/migsarnavarro) 所写，是“Hasura 技术作家计划”的一部分。如果你想要在我们的博客上发布关于 Hasura 或 GraphQL 的文章，在这儿申请。

Hasura 是一个真的很酷的工具，从现存数据库中获得实时 GraphQL API，我很少听说一个事情是，它对于早期原型是一个很棒的工具（意思是不只是原型工具而是生产工具），允许你在本地几分钟内为站点在创建一个很好的后端即服务（BaaS）。最终，你可以放心，它将很容易迁移到一个强健的架构上，像 Hasura Cloud ，只需要很小的改变。

本文假定你早已拥有：

- 安装并运行 Docker 和 docker-compose
- Node.js 和 Nuxt.js 命令行工具

我们将使用 Fastify 作为我们的服务，但是你可以使用 Express，Nest 或者任何 serverless 函数提供商。它将在 Docker 内部运行，和 Hasura 一样，所以你不需要安装任何额外的软件。

这儿的想法是打算给你快速的介绍下 Hasura，并且为你展示它是如何足够灵活的用于前端开发，开箱即用，没有学习曲线。我们将关注核心概念和架构决策，尽管如此，我们将最终得到一个简单但是功能齐全的应用。

## 目录

Local Hasura setup with Docker Compose
Nuxt.js app setup
Authentication with Hasura
Hasura’s actions

全部示例功能在[ Gitlab](https://gitlab.com/migsar/hasura-nuxt-auth)上可用。

## 如何开始？

你可以一直使用 Hasura Cloud 获取即时后端，但是在这个练习中，我将为你展示如何在本地使用它。/

让我们首先看看，由 Hasura 提供的安装清单，[这儿](https://github.com/hasura/graphql-engine/tree/master/install-manifests)，该清单在 graphql-engine 仓库的`安装清单`文件夹中。我们可以看到他们包含了所有常用选项，我们将使用`docker-compose/docker-compose.yaml`。在命令行运行下列命令：

```sh
mkdir hasura-test
cd hasura-test
wget -O server/docker-compose.yaml https://raw.githubusercontent.com/hasura/graphql-engine/master/install-manifests/docker-compose/docker-compose.yaml
docker-compose up
```

就是这样，我们让我们的实例运行，我们可以在 http://localhost:8080 看到管理员面板，并且我们可以开始在 http://localhost:8080/v1/graphql 上点击 GraphQL API ，当然，我们仍然没有任何模型。不要担心，这只是一个快速的开始，让我们一瞥它是如何容易，但实际上我们将尝试在 Nuxt 项目内实现它。正如你刚刚所见，Hasura 启动仅仅是一个 docker-compose 文件，所以让我们创建一个干净的 Nuxt.js 应用，并且把该文件移到里面。

按下` Ctrl-C`停止项目，并且然后安装 Nuxt.js ：

```
# yarn create-nuxt-app <project-name>
npx create-nuxt-app hasura-as-baas

# This is not the full output, just some config options
# create-nuxt-app v3.4.0
# ? Programming language: JavaScript
# ? Rendering mode: Universal (SSR / SSG)
# ? Deployment target: Static (Static/JAMStack hosting)
# 🎉  Successfully created project hasura-as-baas
#  To get started:
#        cd hasura-as-baas
#       yarn dev
```

开始开发前，让我们创建一个服务文件夹，并且把`docker-compose.yaml`文件放在那儿：

```
mkdir server
wget -O server/docker-compose.yaml https://raw.githubusercontent.com/hasura/graphql-engine/master/install-manifests/docker-compose/docker-compose.yaml
```

现在，我们已经安装了前端和后端，让我们帮助他们开始谈话。

## 应用的数据模型

我们将使用用户身份验证创建一个非常基础购物列表，那意味着，一些列表是公开的，同时向所有用户展示，但用户将能够拥有他们自己的私有列表。

我们将专注于身份验证，用户体验将缺乏一些基础功能，但是这将允许我们通过移除所有前端交互降低很多复杂性，因此，有明确的原则，容易适配你自己的项目。首先，我们将使用一个 webhook（网络钩子）来实现身份验证。

### 角色

- 注册用户
- 匿名用户

### 模型（表）

- 角色（需要在 Hasura 中实现角色）
- 用户
- 产品
- 列表

### 创建角色

Hasura 拥有比 Postgres 枚举更适合的枚举表版本，因为他们允许更多灵活性。我们可以把任何表转换成枚举，如果它满足一些规则，它可以有两个字段，`Text `类型的 `value` 和 `description`，并且必须至少有一行，这是为什么以下任务的顺序是重要的。你可以在这个枚举[文档](https://hasura.io/docs/latest/graphql/core/schema/enums.html#enums-in-the-hasura-graphql-engine)中发现更多细节。

![Create roles table](https://hasura.io/blog/content/images/2021/02/Image-1.png)

### 创建表

1. 在 http://localhost:8080 打开 GraphQL 面板
2. 点击顶部的“Data”
3. 点击位于“Schema”标题下的“Create table”按钮
4. 将“roles”写为名称
5. 增加 2 个“Text”类型的列，并且点击“Create table”

![Interactively add roles](https://hasura.io/blog/content/images/2021/02/Img2.png)

### 增加值

1. 增加‘registered’, ‘A registered user’ 作为第一行
2. 增加 ‘anonymous’, ‘A non-registered user’ 作为第二行

![Use table as enum](https://hasura.io/blog/content/images/2021/02/img-3.png)

### 把表设为枚举

1. 返回“Modify”选项卡，并且几乎在选项底部有一个切换键“Set table as enum”，激活它。

## 创建用户（users）

现在我们可以创建`users`表，我们可以在 `roles` 之前创建它，然后返回去修改它，它是个人喜好问题，但是 Hasura 没有强加约束，创建之后修改它和创建时定制它一样容易。一个真的好的事情是在“Insert row ”选项卡创建表之后，你将能够通过下拉选择角色。

![Create users table](https://hasura.io/blog/content/images/2021/02/img-4.png)

### 创建表

1. 使用`users`作为名称
2. 增加`UUID`类型的`id`，默认值`gen_random_uuid() `
3. 增加 `name`,`email`和`Text`类型的`role`
4. 设置`id`和`email`为`unique`（唯一的）。
5. 增加一个外键，从`role`到`value`引用`roles`表

![Adding roles as foreign key](https://hasura.io/blog/content/images/2021/02/img5.png)

![Enums are displayed as dropdown](https://hasura.io/blog/content/images/2021/02/img-6.png)

## 关系：产品和列表

创建表时相似的，我们将需要他们的`id`,`name`和`description`。对于`list`，我们将额外需要与`users`关联的`userId`。然后，自从我们有一个介于列表和产品的多对多关系，我们将需要一个中间表`list_products`，它拥有 `list_id` 和 `product_id`以及每个表分别的引用。

![Creating relationships](https://hasura.io/blog/content/images/2021/02/img6.png)

最后一件事是去创建两表之间的关系，这可以在每个表的“Relationships”选项卡中实现，并且它将允许我们创建嵌套对象请求，
也就是说， 同时修改不止一张表的 GraphQL 查询和变更，在下一节中，这将证明非常方便。

我们可以创建 6 种关系，Hasura 自动检测他们，并且仅仅需要他们中 4 种，最后一个在在中间表中创建，通常不会单独使用它。

1. 用户的列表。允许我们检索给定用户地全部列表对象。
2. 列表的所有者和产品。
3. 产品的列表。
4. ListProducts（关联表） 的列表和产品。

内有玄机，当你创建这种类型关系时， GraphQL schema 将封装嵌套对象到另一个对象中，他没有什么大不了的，但是当你写查询时，你必须牢记在心。我们将在下一节讨论这些。

## 创建示例数据

现在我们有我们的 schema，在那里我们可以插入一些示例数据，去测试我们的身份验证和授权策略，并且也可以测试我们通过 action 增加的服务。

这里所有的代码意味着在由 Hasura 提供的 Graphiql 控制台中使用。你可以使用**Explorer**帮你写这些查询，我将在这儿做个笔记，因为我发现这种行为有一点棘手，若要创建变更和订阅，请清空 GraphQL 文本框，然后在面板底部的下拉框中选择期望的选项，然后点击“+”按钮，面板的内容将更改，并且你将可以开始使用它。

让我们开始创建一个用户：

```gql
mutation InsertUser {
  insert_users(
    objects: [
      {
        email: "migsar.navarro@gmail.com"
        name: "Migsar Navarro"
        role: registered
      }
    ]
  ) {
    returning {
      id
    }
  }
}
```

现在，我们有一个用户，我们将创建一个带有一些产品的列表：

```gql
mutation InsertList {
  insert_lists_one(
    object: {
      name: "My shopping list"
      description: "Let's party"
      owner_id: "1f5c2814-a3c0-456e-b528-7490e2f45930"
      products: {
        data: [
          {
            product: {
              data: { name: "Beers", description: "My favorite beers." }
            }
          }
          { product: { data: { name: "Gummies" } } }
          { product: { data: { name: "Salty snacks" } } }
          {
            product: {
              data: { name: "Oranges", description: "I like oranges" }
            }
          }
        ]
      }
    }
  ) {
    id
  }
}
```

我们硬编码了所有信息，我们本可以使用变量，但是让它简单点，需要注意的一件事是如何嵌套对象有一点费解，`products`是一个带有`data`属性的对象，它包含一个产品数组，但是因为它仅仅引用中间表，而不是产品表它自己，我们正在声明的是从中间表引用产品的数组，所以我们需要再一次使用包装器。

## 业务逻辑功能

一旦你决定你可以在 Hasura 中获得全部 API，你仍然需要选择你想使用哪个语言和框架实现这些业务逻辑。他是一个真的灵活的解决方案，因为接口是常用的 HTTP 服务，实现完全取决于你。

这儿，我们正打算使用 Node.js Fastify。功能的整体结构是非常相似的，但是在负载形式和期望响应上也有一些不同。我们的服务将有一个单路由，根路径。

为了开始这，我已经在 Github 上放置了[fastify-docker-starter](https://github.com/migsar/fastify-docker-starter)仓库，你可以克隆它，然后移除 git 文件：

```sh
cd server
git clone https://github.com/migsar/fastify-docker-starter my-function
rm -rf my-function/.git
```

这些函数可以在本地执行，因为 Hasura 早已使用容器，所以把这些函数打包在那里是一个更好的方法，把这些东西放在一起将帮助你快速迁移到一个云基础设施提供商。我们将限制我们需要的 Docker 命令的数量，并且为了方便起见把他们都包含在 package.json 中。
按照习惯，我们将命名镜像为 hasura-baas-*，*用服务的名字替换。

你可以容易开始使用路由在单个容器中构建你自己的 API。我更喜欢单路由非常特定的容器，因为那通常完全解耦，并且容易在不同项目中复用。这一切都是为了找到手头项目的最佳点。

## 身份验证

至此，所有的请求都未身份验证，或者换一种说法，每个人都有数据库管理员权限。显而易见，这不是最好的选择。我们需要某种形式的身份验证。

![Basic authentication flow](https://hasura.io/blog/content/images/2021/02/img7.png)

Hasura 允许钩子（webhooks）和 JWT 身份验证，本文我们将仅仅覆盖第一种。需要强调的是 Hasura 实例将和我们的服务沟通，而不是前端。身份验证对一些项目来说非常明确，此外也有一些复杂的主题，如[缓存](https://hasura.io/graphql/caching/)和令牌存储，不在本文涉及范围。

开箱即用，Hasura 的安装时开放的，然后 Hasura 提供一个方式跳过身份验证，它是一个叫做`HASURA_GRAPHQL_ADMIN_SECRET`的环境变量，在`docker-compose.yaml`中设置，它早已包含在内，但是被注释掉了，一旦开启，如果传了该参数，它将不再检查任何其他的事情，并且允许你做任何事情。如果不是，它将禁用访问或者寻找其他方法。

Webhooks（网络钩子）是这些可替代方法之一；配置它我们将需要设置 2 个环境变量：

- HASURA_GRAPHQL_AUTH_HOOK: 网络钩子端点
- HASURA_GRAPHQL_AUTH_HOOK_MODE: 请求方法 GET 或者 POST

我们将在 `docker-compose.yaml`增加全部 3 个变量，并且为开发配置管理员秘钥，为生产配置钩子。如果我们想更进一步，我们可以在根目录中创建一个`.env` 文件，移除那儿的环境变量，并且从我们的`docker-compose.yaml`中引用它，确保将`:`变为`=`，并且删除 `true` 周围的`" `。我真的喜欢这样，去确保环境变量从不会进入仓库。因为`.env` 位于公共 `.gitignore`配置中。如果你像学习更多，核查 [Docker Compose 文档](https://docs.docker.com/compose/environment-variables/).

一旦你有了`.env`文件：

```YAML
# Replate this:
    environment:
      HASURA_GRAPHQL_DATABASE_URL: postgres://postgres:postgrespassword@postgres:5432/postgres
      HASURA_GRAPHQL_ENABLE_CONSOLE: "true"
      HASURA_GRAPHQL_DEV_MODE: "true"
      HASURA_GRAPHQL_ENABLED_LOG_TYPES: startup, http-log, webhook-log, websocket-log, query-log

# By this:
    env_file:
      - ../.env
```

我们的身份验证服务将非常基础，但是牢记它是一个普通的 HTTP 端点，那有两种重要的含义。第一你一点也不被限制去使用 Node.js，第二是你可以直接插入一些早已经过测试的东西，像通行证和它有的任何策略。这里我们不会那样做。我们将为请求增加一个头`x-email`，如果它出现，并且邮箱早已存在我们的数据库，然后它就是一个**注册用户**，否则是**匿名用户**。

**/server/docker-compose.yaml**

```YAML
# Not all the code is displayed
services:
  auth:
    image: hasura-auth:v1
    restart: always
    env_file:
      - ../.env
```

**/server/auth/src/index.mjs**

```JS
/ Not all the code is displayed
import pg from 'pg';
const { Pool } = pg;

const {
  AUTH_SERVER_HEADER_KEY: headerKey,
  HASURA_GRAPHQL_DATABASE_URL: connectionString,
} = process.env;

// Read-only SQL
const getRoleQuery = ({ email }) =>
  `SELECT role FROM users WHERE email='${email}'`

const pool = new Pool({ connectionString });

server.post('/', async (req, reply) => {
  const { headers } = req.body;
  if (!headers) {
    return reply.code(400).send(new Error(Errors.NO_DATA));
  }

  if (!headers[headerKey]) {
    // Here we can return any other session variable
    // Just add another key with the form X-Hasura-*
    reply.code(200).send({ 'X-Hasura-Role': Roles.ANONYMOUS })
  }

  try {
    const { rows } = await pool.query(getRoleQuery({ email: headers[headerKey] }));
    const role = rows.length > 0 ? rows[0].role :  Roles.ANONYMOUS;
    reply.code(200).send({ 'X-Hasura-Role': role })
    reply.code(200).send();
  } catch (error) {
    return reply.code(400).send(error)
  }
})
```

最后一步，我们需要使用我们在 `docker-compose.yaml` 文件引用的名字创建镜像 `docker build -t hasura-auth:v1 `。在`/server/auth`，下次我们启动 `Docker Compose up` 时将拥有身份验证服务。

此时你可能会问，”我们有一个用户，但是新用户如何注册？“。这是一个好问题，现在这个答案是，他们不会有人添加他们，无论是注册用户还是使用管理员秘钥头连接上的某些人。那是不同的策略，并且事实是，这高度依赖于实现，所以很难提出适用所有场景的解决方案，2 个可能的策略是：

1. 配置 `anonymous`角色对很多资源拥有只读权限，但是对 `users` 表拥有写权限，这将在内部应用中可用，因为你的用户在某种程度上是可信的。
2. 在写入角色上混合使用 actions 和仅后端权限去增加额外的验证层。

你可以总是跳过这个并且创建直接把用户写入数据库的服务，但是我认为这是最不优雅的解决方案。

## 授权、权限控制

我们现在
