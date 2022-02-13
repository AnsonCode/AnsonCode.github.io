---
title: "002 译 | Nhost简介及使用"
date: 2022-02-01T11:59:39+08:00
tags: ["技术翻译", "Hasura", "Nhost"]
categories:
  - "研发提效"
featured_image:
description:
draft: false
---

# 1. 欢迎来到 Nhost

Nhost 是一个开源的，实时，无服务的后端平台，能够构建可靠的应用，去扩大你的商业规模。

### 组件

Nhost 构建在一系列开源组件之上。

**数据库**

你的应用程序有自己的 PostgreSQL 数据库，这个世界上最先进的关系数据库。

**GraphQL API**
基于 Hasura 的高性能、实时 GraphQL API

**鉴权和存储**
用户管理和文件存储与 Hasura 权限无缝集成

**无服务函数**

在后端用 JavaScript 和 TypeScript 函数运行自定义代码

### 开始

跟着我们的[快速开始](https://docs.nhost.io/get-started/quick-start)指导去构建你的第一个应用。

核查[Nhost Github](https://github.com/nhost/nhost)。给我们一个星星，并尽情讨论任何特性需求。

# 2. 快速启动

# 2.1 创建你的应用

让我们使用 Nhost 创建一个简单的 todo(待做事项)应用。在一个待做事项应用中，一个用户应该能为他们的账户创建列表项（CRUD），并且没有其他任何人看到他们（权限）。

为了在 Nhost 实现一个像这样的应用，我们简单介绍这些主题：

- 在 Nhost 创建一个新的应用
- 定义数据库 schema
- 插入数据
- 设置权限
- 通过 GraphQL API 查询数据

在它的结尾，但愿你有一个更好的理解关于 Nhost 是什么，以及它能为你做什么。

### 登录 Nhost

如果你之前没有账户，前往[happ.nhost.io](https://app.nhost.io/)注册一个新账户。

登录并跳转到 Nhost 控制面板。

### 创建应用

在控制面板的主页，点击“新建应用”按钮。填写名字并且选择一个靠近你用户地区域。你将使用默认空间和免费计划完成设置。

![new app](https://docs.nhost.io/images/quick-start/new-app.png)

设置新的应用将花费大约 20 秒。在这期间，Nhost 将为你的应用设置好全部的基础架构。

一旦启动完成，你将自动看到控制面板，然后你就可以去定义你的应用数据库 schema 了。

# 2.2 定义 schema

为了实现一个管理待做事项的应用，让我们确保我们有存储待做事项和用户地数据库表。

### 打开 Hasura 控制台

Hasura 生成实时 GraphQL APIs，但是它也提供一个网络控制台管理 schema 和数据库数据。

在你的 APP 面板，前往**数据**页签，并且选择**打开 Hasura**。记得去复制这个管理员秘钥。

你 APP 的 Hasura 控制台将在一个新的页签中打开。你可以使用 Hasura 控制台去管理你的应用 schema、数据、权限和时间触发。

![DATA->OPEN Hasura](https://docs.nhost.io/images/quick-start/data-tab.png)

### 用户表

你应该能在屏幕左侧看到你的所有数据表。你应该看到多个不同的像文件夹一样展示的**schemas**。

- 你 APP 自定义表的公共 schema
- NHost 的用户管理和文件存储 schema

如果你打开**auth**schema，将看到你的 app 早已经有用户表了，所以你不需要创建一个。

![To store the users, we already have a users table from the auth schema](https://docs.nhost.io/images/quick-start/list-of-schemas.png)

### 创建 TODOs 表

在 Hasura 控制台中，去**数据**页签，然后点击**创建表**。命名这个表为**todos**。

**增加常用的列**

**id**和**created_at**字段是很常见的，仅仅通过两次点击增加上。点击**常用字段**并且创建他们：

- **id**(uuid)
- **created_at**(timestamp)

这确保这些字段获得正确的命名、类型和默认值。

![Frequently used columns in the Hasura console](https://docs.nhost.io/images/quick-start/frequently-used-columns.png)

**增加自定义字段**

手动增加另外两列字段

- **name**(text)
- **is_completed**(boolean)

确保设置**is_completed**的默认值为**false**

![Create a table in the Hasura console](https://docs.nhost.io/images/quick-start/create-table.png)

这是所有我们需要做的！当你点击**增加表**时，一个新的表将要被创建。

### 插入数据

前往**插入行**页签为你的数据库增加一些数据。

![Inserting todos](https://docs.nhost.io/images/quick-start/insert-todos.gif)

### 查询数据

现在在我们的数据库中有了数据，我们可以通过 GraphQL API 检索他们。在主菜单中，前往**API**页签。这个页面可以用于执行 GraphQL 请求，查询或者变更数据库中的数据。

粘贴下面的 GraphQL 查询到表单中，点击“播放”按钮：

```
query {
  todos {
    id
    created_at
    name
    is_completed
  }
}
```

你将在右侧看到你感兴趣的 todos（待做事项）输出。

#### 管理员角色

上述请求的前提是：Hasura 控制台的所有请求都是**管理员**角色。

# 2.3 JavaScript 客户端

在前一个章节，你使用 Hasura 控制台查询了一列待做事项。现在，你将在你的 Nhost 应用中写一个小型的 JavaScript 客户端去集成和检索待做事项。

#### 前段框架

Nhost 是框架无关的，并且支持你想使用的任何前端。如果你想的话，也可以从你的服务端链接 Nhost。

在这个指南中，我们将保持例子简单。我们不使用前端框架。在一个真实的场景中，你或许将使用一个 React 、VUE、Svelte 或者 React Native 框架构建一个客户端。

### 启动

_注：确保你已经安装[Node.js](https://nodejs.org/en/)和[npm](https://docs.npmjs.com/getting-started/)_

创建一个新的文件夹叫做**nhost-todos**，并且初始化一个新的 JavaScript 应用：

```bash
npm init
```

*注：你或许必须编辑`package.json`文件，并且增加或改变*type*对象为`module`（“type":"module"）*

安装 Nhost JavaScript SDK:

```bash
npm install @nhost/nhost-js
```

### 初始化 Nhost

在新的文件夹中，创建叫做**index.js**的文件。

在文件中输入下列代码。它将初始化一个新的**NhostClient**，能够与你的后端交互：

```js
import { NhostClient } from "@nhost/nhost-js";

const nhost = new NhostClient({
  backendUrl: "https://[app-subdomain].nhost.run", // replace this with the backend URL of your app
});

console.log(nhost.graphql.getUrl());
```

在你的命令行中运行代码。你将看到你应用的 GraphQL 端点 URL:

```BASH
➜ node index.js

https://[app-subdomain].nhost.run/v1/graphql
```

### 查询 todos

如果你现在增加下列的 GraphQL 查询到客户端，当你运行这个更新的版本时，让我们看将发生什么：

```js
import { NhostClient } from "@nhost/nhost-js";

const nhost = new NhostClient({
  backendUrl: "https://[app-subdomain].nhost.run",
})(async () => {
  // nhost.graphql.request returns a promise, so we use await here
  const todos = await nhost.graphql.request(`
    query {
      todos {
        id
        created_at
        name
        is_completed
      }
    }
  `);

  // Print todos to console
  console.log(JSON.stringify(todos.data, null, 2));
})();
```

```console
➜ node index.js

null
```

**null**被输出。为什么是这样？让我们找到原因。

# 2.4 设置权限

Hasura 支持角色权限控制。你为每一个角色、表和操作(select\insert\update\delte)创建规则，这会动态核查会话变量，例如用户 ID。

在上述部分，你能查询 todos，因为当使用 Hasura 控制台时，**管理员**角色默认开启。当构建你的应用时，你将想要使用角色定义权限，当执行请求时，你的用户假定的角色。

## 未授权用户

当你想要一些数据被任何不登录的用户访问时，在权限中使用**public**角色。在所有未授权请求中，这是默认角色。

通常说，这个**public**角色不应该有插入、更新或者删除的权限定义。

### 设置公共权限

在 Hasura 控制台中，前往**Data**页签，点击**todos**表，然后点击**授权**。增加一个叫做**public**的新角色，点击**选择**。这个对于选择操作的权限选项将在下面打开。

增加下列的权限：

![Public role](https://docs.nhost.io/images/quick-start/permissions-public-select.png)

如果你现在再运行程序一次，你将在输出中看到 todos：

```bash
➜ node index.js

{
  "todos": [
    {
      "id": "558b9754-bb18-4abd-83d9-e9056934e812",
      "created_at": "2021-12-01T17:05:09.311362+00:00",
      "name": "write docs",
      "is_completed": false
    },
    {
      "id": "480369c8-6f57-4061-bfdf-9ead647e10d3",
      "created_at": "2021-12-01T17:05:20.5693+00:00",
      "name": "cook dinner",
      "is_completed": true
    }
  ]
}

```

这有 2 个原因，为什么请求成功：

- Nhost 设置**public**角色为每个未授权 GraphQL 请求
- 你明确定义权限为**public**角色

他是重要的去理解 Hasura 有一个**默认禁用所有**的策略确保仅仅你明确定义的角色和权限能被访问。

# 3. 授权用户

在上述章节，你为**public**角色定义**select**权限。现在你将增加**insert**和**create**权限对于授权用户，使用权限验证去保护你应用 GraphQL API 的安全。

_Nhost 的授权服务允许你为你的用户提供无摩擦注册和登录体验。我们支持大量社交账户和不同的方法（无密码、短信等等）_

### 插入一个测试用户

通过你应用顶部菜单的**用户**页签手动创建用户并且点击**增加用户**。

![Add user](https://docs.nhost.io/images/quick-start/add-user.gif)

你现在将使用新创建的用户向 API 发出授权过的请求.

### 授权并查询数据

增加下列的代码登录新的用户并且再一次请求一列 todos:

```js
import { NhostClient } from "@nhost/nhost-js";

const nhost = new NhostClient({
  backendUrl: "https://[app-subdomain].nhost.run",
})(async () => {
  // Sign in user
  const signInResponse = await nhost.auth.signIn({
    email: "joe@example.com",
    password: "securepassword",
  });

  // Handle sign-in error
  if (signInResponse.error) {
    throw signInResponse.error;
  }

  // Get todos
  const todos = await nhost.graphql.request(`
    query {
      todos {
        id
        created_at
        name
        is_completed
      }
    }
  `);

  console.log(JSON.stringify(todos.data, null, 2));
})();
```

为什么返回空值？因为当作为授权用户执行 GraphQL 请求时，这个**用户**角色被假定。

_对于授权请求，总是可以使用其他合法的角色覆盖默认的**用户**角色。_

### 用户权限

**移除公共角色的权限**

我们将不再使用**public**角色，所以，让我们移除这个角色的所有权限。

![Remove public permissions from Hasura](https://docs.nhost.io/images/quick-start/remove-public-permissions.png)

现在，我们将增加**user**角色权限。

_所有登录用户都有**user**角色_

**插入权限**

首先，我们设置**插入权限**。

一个用户仅能插入`name`，因为所有的其他列将被自动设置。更具体的说，`user_id`将被设置为发出请求用户 ID（`x-hasura-user-id`），并在**Column presets**部分配置。见下图：

![User insert permission](https://docs.nhost.io/images/quick-start/user-insert-permission.png)

**选择权限**

对于**选择权限**，设置一个**自定义检查**，以便用户仅能选择`user_id`与他们用户 ID 相同的 todos。换句话说：用户仅能选择他们自己的 todos。见下图：

![User select permission](https://docs.nhost.io/images/quick-start/user-select-permission.png)

现在，再一次运行 APP。新的 todos 被插入，并且仅用户的 todos 能被查询和展示。你的后端成功被保护！

# 4. 工作流命令行

## 4.1 命令行从零到生产环境

在之前的教程中，我们测试了 Nhost 的各部分，例如：

- Database
- GraphQL API
- Permission
- JavaScript SDK
- Authentication

我们为数据库和 API 做的所有更改都直接在 Nhost 应用的生产环境。

在生产环境进行更改是不理想的，因为你或许弄砸事情，将影响应用的所有用户。

反而，在部署这些变更到生产环境之前，更推荐在本地更改和测试应用。

为了在本地进行更改，我们需要有一个完整的 Nhost 应用运行在本地，这是 Nhost CLI 做的事情。

Nhost CLI 在本地环境中匹配生产环境应用，这种方式在部署修改到生产环境前，你可以修改或者测试代码。

### 使用 Nhost 建议的工作流

- 使用 Nhost CLI 本地开发
- 推送变更到 GitHub
- Nhost 部署变更到生产环境

### 你将在该指导中学到什么：

- 使用 Nhost CLI 创建本地环境
- 使用 Nhost 应用连接到 GitHub 仓库
- 部署本地修改到生产环境

## 4.2 工作流程设置

接下来是如何为工作流程设置 Nhost 的详细教程。

### 创建 Nhost 应用

为该练习创建新的 Nhost 应用。

> 为这个指导创建新的应用而不是复用旧的 Nhost 应用是重要的，因为我们想去起推动一个干净的 Nhost 应用。

![Create new app](https://docs.nhost.io/images/cli-workflow/create-app.png)

### 创建新的 GitHub 仓库

为你的新 Nhost 应用创建新的 GitHub 仓库。仓库私有或者公开都可以。

![Create new repo](https://docs.nhost.io/images/cli-workflow/create-repo.png)

### 把 GitHub 仓库连接到 Nhost 应用

在 Nhost 面板，前往你 Nhost 应用的仪表盘并且点击`Connect to GitHub`。

[视频演示](https://docs.nhost.io/videos/cli-workflow/connect-github-repo.mp4)

## 4.3 安装命令行

使用下列命令安装 Nhost CLI：

```sh
sudo curl -L https://raw.githubusercontent.com/nhost/cli/main/get.sh | bash
```

在本地初始化新的 Nhost 应用：

```sh
nhost init -n "nhost-example-app" && cd nhost-example-app
```

并且，在相同文件夹初始化 GitHub 仓库：

```sh
echo "# nhost-example-app" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/[github-username]/nhost-example-app.git
git push -u origin main
```

现在返回 Nhost 面板，点击`Deployments`。你就已经为 Nhost 应用完成了新的部署。

![Deployments tab](https://docs.nhost.io/images/cli-workflow/deployments-tab.png)

如果你点击部署，你可以看到没有任何东西真的部署了。因为，我们仅仅对 README 文件做了变更。

![Deployments details](https://docs.nhost.io/images/cli-workflow/deployments-details.png)

让我们做一些本地后端的变更吧！

## 4.4 本地变更

本地启动 Nhost：

```sh
nhost dev
```

> 确保你已经在自己电脑上安装了 Docker。这是 Nhost 工作所必须的。

`nhost dev` 命令将在你的电脑上自动开始一个完整的本地 Nhost 环境：

- Postgres
- Hasura
- Authentication
- Storage
- Serverless Functions
- Mailhog

在部署变更到生产环境前，你使用本地环境去做更改和测试。

运行`nhost dev` 也启动 Hasura 控制面板。

> 使用 Hasura 面板是重要的，当你变更时，它自动启动。这种方式，自动为你跟踪变化。

![Hasura Console](https://docs.nhost.io/images/cli-workflow/hasura-console.png)

在 Hasura 控制面板，创建一个带有 2 列的新表`customers`:

- id
- name

[视频演示](https://docs.nhost.io/videos/cli-workflow/hasura-create-customers-table.mp4)

当我们创建`customers`表时，有一个迁移被自动创建。迁移被创建在 `nhost/migrations/default`下。

```sh
$ ls -la nhost/migrations/default
total 0
drwxr-xr-x  3 eli  staff   96 Feb  7 16:19 .
drwxr-xr-x  3 eli  staff   96 Feb  7 16:19 ..
drwxr-xr-x  4 eli  staff  128 Feb  7 16:19 1644247179684_create_table_public_customers
```

数据库迁移或许仅被应用于本地，意味着，你在本地创建`customers`表，但是它至今不存在于生产环境。

为了将本地变更应用到生产环境，我们需要提交变更，并且推送它到 GitHub。Nhost 将自动获取仓库变更并应用变更。

> 你可以提交和推送文件在另一个终端，以便`nhost dev`仍然运行。

```sh
git add -A
git commit -m "Initialized Nhost and added a customers table"
git push
```

在 Hasura 面板导航到`Deployments`选项卡看下部署。

![Deployments tab after changes](https://docs.nhost.io/images/cli-workflow/deployments-tab-with-changes.png)

一旦部署完成，`customers`表将在生产环境创建。

![Customers table in Hasura Console](https://docs.nhost.io/images/cli-workflow/hasura-customers-table.png)

现在，我们在 Nhost 完成了推荐的工作流：

- 使用 Nhost CLI 本地开发
- 推送变更到 GitHub
- Nhost 部署变更到生产环境

## 4.5 元数据和 Serverless 函数

在上述章节，我们仅仅创建了一个新的表：`customers`。使用命令行你也可以后端的其他部分。

命令行和 GitHub 集成跟踪和应用到生产环境有有 3 个事情：

- 数据迁移
- Hasura 元数据
- Serverless 函数

本节，我们变更下 Hasura 元数据并且创建一个 Serverless 函数。

### Hasura 元数据

我们为`users`表增加权限，确保用户仅能看到他们自己的数据。为此，前往`auth`模式，点击`users`表。然后点击 `Permissions` 并键入新的角色 `user`，并且为角色创建新的 `select` 权限。

使用`自定义检查`创建权限：

```JSON
{
  "id": {
    "_eq" : "X-Hasura-User-Id"
  }
}
```

选择下列的字段：

- id
- created_at
- display_name
- avatar_url
- email

然后点击`Save permissions`(保存权限)。

[视频演示](https://docs.nhost.io/videos/cli-workflow/hasura-user-permissions.mp4)

现在，我们再做一次`git status`(git 状态检查)去确保我们做的权限更改被跟踪到本地的 Git 仓库中。

![Git status](https://docs.nhost.io/images/cli-workflow/git-status.png)

我们现在可以提交该变更：

```sh
git add -A
git commit -m "added permission for uses"
```

现在，让我们在推送所有变更到 GitHub 之前，创建一个 serverless 函数，以便 Nhost 能部署我们的变更。

### Serverless 函数

Serverless 函数是一块用 JavaScript 或者 TypeScript 写的代码，它能响应 http 请求，并返回响应。

这是一些示例：

```js
import { Request, Response } from "express";

export default (req: Request, res: Response) => {
  res.status(200).send(`Hello ${req.query.name}!`);
};
```

Serverless 函数被放置在仓库的`functions/ `文件夹。每个文件变成他自己的端点。在我们创建 serverless 函数之前，需要安装 `express`，它是 Serverless 函数工作的必要条件。

```sh
npm install express
# or with yarn
yarn add express
```

我们将用到 TypeScript，所以我们也安装两个类型定义。

```sh
npm install -d @types/node @types/express
# or with yarn
yarn add -D @types/node @types/express
```

然后，我们创建一个文件 `functions/time.ts`。

在文件 `time.ts` 中，我们将增加下列的代码去创建我们的 serverless 函数:

```js
import { Request, Response } from "express";

export default (req: Request, res: Response) => {
  return res
    .status(200)
    .send(`Hello ${req.query.name}! It's now: ${new Date().toUTCString()}`);
};
```

我们现在可以在本地测试该函数。本地，后端 URL 是 `http://localhost:1337`。函数位于`/v1/functions `之下。每个函数的路径和名称变成 API 端点。

这意味着我们的函数`functions/time.ts`位于 `http://localhost:1337/v1/functions/time`.

让我们使用 curl 测试我们的新函数：

```sh
curl http://localhost:1337/v1/functions/time
Hello undefined! It's now: Sun, 06 Feb 2022 17:44:45 GMT
```

并且携带一个带有我们名字的查询参数：

```sh
curl http://localhost:1337/v1/functions/time\?name\=Johan
Hello Johan! It's now: Sun, 06 Feb 2022 17:44:48 GMT
```

再一次，让我们使用`git status`去看下创建过 serverless 函数后的变化。

现在，提交变化并且推送到 GitHub。

```sh
git add -A
git commit -m "added serverless function"
git push
```

在 Nhost 控制面板，点击一个新的部署看下详情。

![Deployments details for function](https://docs.nhost.io/images/cli-workflow/details-for-function.png)

Nhost 完成变更部署后，我们可以在生产环境测试他们。首先，确保用户权限已经应用。

![Hasura Console permissions table](https://docs.nhost.io/images/cli-workflow/hasura-permissions-table.png)

然后，确认 serverless 函数被部署。再一次，我们使用 curl:

```sh
curl http://localhost:1337/v1/functions/time\?name\=Johan
```

![Serverless Function test](https://docs.nhost.io/images/cli-workflow/function-test.png)

### 结论

在这个练习中，我们安装了 Nhost CLI 并且创建了本地 Nhost 环境进行本地开发和测试。

在本地环境中，我们变更了数据库和 Hasura’s 元数据，并且创建了 serverless 函数。

我们连接到 GitHub 仓库，并且推送变更到 GitHub。

我们看到了 Nhost 自动部署我们的变更，并且我们验证了更改的应用。

总结，我们使用推荐的 Nhost 工作流建立了生产环境：

- 使用 Nhost CLI 本地开发
- 推送变更到 GitHub
- Nhost 部署变更到生产环境

# 4. 从 V1 升级到 V2(待润色)

从 Nhost V1 版本升级到 V2 版本需要改变数据库 schema 和 hasura 元数据。

**需要帮助？**

如果你需要帮助从 V1 迁移到 V2,请联系 Johan: johan@nhost.io

### 升级步骤

**准备一个新的文件夹**

```sh
mkdir -p nhost-v2/nhost
echo "version: 2" > nhost-v2/nhost/config.yaml
```

**导出 Nhost V1 的当前迁移和元数据**

```sh
hasura migrate create init --from-server --endpoint=[v1-endpoint] --admin-secret=[v1-admin-secret]
hasura metadata export --endpoint=[v1-endpoint] --admin-secret=[v1-admin-secret]
```

**编辑迁移**

编辑被创建的*up.sql*迁移文件。

- 删除所有\*auth.\*\*表和函数
- 删除*public.users*表
- 更新外键引用从*public.users*到*auth.users*
- Add OR REPLACE after CREATE for the public.set_current_timestamp_updated_at function

**编辑元数据**
更新元数据

- 在*auth* schema 中删除所有跟踪的所有表
- 删除跟踪*public.users*表
- 更新对用户的所有从*public*到*auth* schema 的引用

_启动 nhost_
启动 Nhost CLI

_重启授权和存储容器_

打开 Docker UI 并重启 Hasura Auth 和 Hasura Storage。这将 为 auth 和 storage schema 重新应用数据库库迁移和元数据。

_移动数据库迁移_

移动当前*migrations*文件夹到前一个*migration-bk*

_更改配置版本_

更改配置版本从*config 2*到*config 3*在*nhost/config.yaml*文件

_导出本地元数据用新的格式_

```sh
hasura metadata export --endpoint=http://localhost:1337 --admin-secret=nhost-admin-secret
```

**移动迁移文件夹**
为了匹配 hasura 配置 V3,移动所有的迁移进入*migrations/default*文件夹。

```sh
mkdir -p migrations/default
cp -r migrations/* migrations/default/
```

**完成**
数据库迁移和元数据现在从远程 Nhost V1 的后端拉取下来了，为 Nhost V2 编辑，并且为 Hasura Configuration v3 更新。
