---
title: "002【翻译】Nhost简介及使用"
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
