---
title: "004 译 | 使用Gatsby和Hasura动态化JAMStack"
date: 2022-02-07T12:33:13+08:00
tags: ["技术翻译", "Hasura"]
categories:
  - "研发提效"
featured_image:
description:
draft: false
---

翻译自：[Dynamic JAMStack with Gatsby and Hasura GraphQL](https://hasura.io/blog/dynamic-jamstack-with-gatsby-hasura-graphql/)

[JAMStack](https://jamstack.org/)是构建 web 应用的现代架构。与基于经验的传统 CMS 相比，开发者工作流有了很大的提升。

由 JAMStack 驱动的 web 应用，由于预构建标记获得更快的性能，通过 CDN 交付可伸缩应用的能力，应用 UI 和 API 数据之间关注点和安全的明确分离，最终通过自动构建和 git 集成轻松部署。

但是，还有什么比通过 Hasura's GraphQL APIs 让站点动态化更让它具有扩展性和有力的呢？在这个文章中，我们将寻找：如何使用 Hasura 和 GraphQL 构建一个具有 JAMStack 所有优点的动态应用。

![架构](https://hasura.io/blog/content/images/2019/10/dynamic-jamstack-hasura-1.png)

## 入门指南

JAMStack 对使用的工具没有任何意见，只要输出是一个好的旧静态站点。如果你是一个想要加入 JAMStack 工作流的前端开发者，你或许感兴趣你的流行框架是否支持它？现代前端框架（像 React 和 Vue）周围有很多受欢迎的**静态站点生成器**。

本文，我们将看下这个最受欢迎的 React 静态站点生成器——[Gatsby](https://www.gatsbyjs.com/)。

## Gatsby 的静态站点

让我们创建一个新的 Gatsby 站点，并且建立一个简单静态站点去展示文章。

1. 安装 gatsby CLI

```sh
npm install -g gatsby-cli
```

2. 创建一个新的站点

```sh
gatsby new gatsby-site
```

3. 开始开发

```sh
gatsby develop
```

4. 导航至 _http://localhost:8000_ 看应用运行

## 构建阶段的元数据，数据改变时重建

在构建阶段，静态站点生成元数据，生成一个完全静态的站点。现在，你有站点在运行中，在*src/pages*中改变任何文件，去看实时改变的重建发生。在开发设置中这当然做了一个改变的实时重载。

让我们生成输出 HTML 文件的产品构建。

执行下面的命令：

```sh
gatsby build && gatsby serve
```

去建立和服务静态站点。你将看到像下面这样的页面：

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220207154348.png)

Gatsby 在构建阶段生成 HTML。所以，当你运行`gatsby build`时，底层发生了什么呢？Gatsby 遍历应用的每个路径，编译出普通的 HTML 文件。它使用特定的数据源去编译 HTML。上面运行的站点有直接用 HTML 写的静态数据。在 HTML 中任何数据变化将在你下次运行重建命令时被重建。

棒极了！所以，任何静态内容变成了 HTML，能被简单的 HTTP 服务提供服务。但是，这对于很多应用是不足够的。不是吗？通常，你将需要来自数据库的数据、特定用户的内容或者受制于特定付费门槛甚至在页面集成数据处理的交互操作。数据需要快速变成动态的（todo）。

## 增加动态数据

在上述静态站点，假若这个文章列表来自一个数据库，而不是在每篇文章中手工编写，会怎么样？HTML 页面必须为站点中尽可能多的文章构建。像 Gatsby 这样的静态站点生成器能连接到外部数据源。GraphQL 是其中之一。

Gatsby 有原生的 GraphQL 支持，让你在众多外部数据源列表中增加任何 GraphQL API 作为数据源插件。GraphQL APIs 也能使用**GraphiQL**运行`http://localhost:8000/___graphql`导出.

## Hasura 如何适配动态 JAMStack

Hasura 是一个开源引擎，能在新的或现存的 Postgres 数据库中提供实时 GraphQL APIs，内置支持拼接自定义 GraphQL APIs。

Hasura 通过它的统一的 GraphQL API 端点变成 Gatsby 的数据源，来适应 JAMStack。它也支持授权（不在本文的范围）。

![](https://hasura.io/blog/content/images/2019/09/hasura-gatsby.png)

跟着[文档](https://hasura.io/docs/latest/graphql/core/deployment/deployment-guides/heroku.html)的介绍去部署 Hasura，并且创建这个 app 需要的`users`和`articles`表。记录 GraphQL Endpoin 的 Heroku URL。你将在 Gatsby 应用中配置这个，接下来去获取数据。

现在，让我们添加 Hasura 数据源，去从 Postgres 获取内容到 Gatsby。

我们使用 gatsby 插件 [gatsby-source-graphql](https://www.gatsbyjs.org/docs/third-party-graphql/)连接任何外部 GraphQL API。请注意，所有的静态站点生成器都可以通过一些方式去连接外部 API 获取数据。

在`gatsby-config.js`中，插件配置看起来像这样：

```json
{
     resolve: 'gatsby-source-graphql',
     options: {
       typeName: 'HASURA',
       fieldName: 'hasura',
       url: `${ process.env.HASURA_GRAPHQL_URL }`,
     },
   }
```

现在，导航到`http://localhost:8000/___graphql`与`users`和`articles`一起探索 hasura 源。

在应用中，导航到`src/pages/index.js`，增加下列 GraphQL 查询去从 Postgres 获取数据。

```js
export const query = graphql`
  query ArticleQuery {
    hasura {
      articles {
        id
        title
      }
    }
  }
`;
```

现在改变文章列表为：

```js
const IndexPage = ({ data }) => (
  <Layout>
    <SEO title="Home" />
    <ul>
      {data.hasura.articles.map((article) => (
        <Link key={article.id} to={"/" + article.id}>
          <li>{article.title}</li>
        </Link>
      ))}
    </ul>
  </Layout>
);
```

这应该现在从 Postgres 数据库获取带有它的`id`和`title`的文章。

![](https://hasura.io/blog/content/images/2019/09/gatsby-data-graphql.png)

太棒了！因此，文章现在来自数据库并且在构建时获取（？）。

这儿是参考代码的[源码仓库](https://github.com/praveenweb/dynamic-jamstack-gatsby-hasura/tree/master/dynamic-source-build)。

## 分离静态和动态数据

现在让我们考虑这个场景，列表页有没有授权就不能访问的文章。只有用户登录，私有的文章才能被他们访问。对于内容对不同登录用户不同的动态应用，这是一个典型的用例。账户页也仅能被登录用户访问。显而易见，这些页面不能在编译时间被构建。对此我们有两个组件：

- 设置身份验证（例如：Auth0)
- 为用户配置权限

### 设置身份验证

第一步是为应用启动授权。前往[文档](https://hasura.io/docs/latest/graphql/core/guides/integrations/auth0-jwt.html)去设置 Auth0 。你可以用其他任何身份验证提供者或写自定义身份验证去替换它。一旦身份验证提供者就位，gatsby 实现是基于用户是否授权来限制特定路由的内容。

### 受保护路由

之前设置的`gatsby-source-graphql`配置将在构建阶段生效。如果我们需要在客户端发起授权请求，我们需要在客户端设置`react-apollo`，并且在发起请求前设置恰当的头部。这是一个例子，如何在客户端动态获取文章。

```js
import React from "react";
import { useQuery } from "@apollo/react-hooks";
import gql from "graphql-tag";

// This query is executed at run time by Apollo.
const APOLLO_QUERY = gql`
  {
    articles {
      id
      title
      user {
        id
        name
      }
    }
  }
`;

export default ({ token }) => {
  const { loading, error, data } = useQuery(APOLLO_QUERY, {
    context: { headers: { Authorization: "Bearer " + token } },
  });

  return (
    <div>
      {loading && <p>Loading...</p>}
      {error && <p>Error: ${error.message}</p>}
      {data &&
        data.articles &&
        data.articles.map((article) => {
          return (
            <div key={article.id}>
              {article.id} - {article.title}
            </div>
          );
        })}
    </div>
  );
};
```

如果你尝试不登录访问文章页，你将被重定向到登录页。现在刷新页面登录，看是否文章可见 。

第二块是去设置用户权限。让我们看下如何使用 Hasura 做到这一点。

## Hasura 基于角色的权限控制

Hasura 有基于角色的权限控制，数据获取可与基于登录用户的角色限制在特定条件下。现在这可以用来限制内容，让它仅能被应用程序用户访问。

在上述应用中，我们有一个使用权限保护的文章列表页。但是现在，让我们限制每一个登录用户可以看到的内容。

在此之前，我们将为`user`角色设置一个选择权限去限制用户 ID 等于会话变量`x-hasura-user-id`。（这将设置在 JWT token 中）

前往 Hasura 控制面板设置如下权限：

![](https://hasura.io/blog/content/images/2019/09/user-permission.png)

权限已经设置完毕。现在尝试为不同用户插入一些数据。

刷新页面，你将能够仅看到自己的文章。这是因为在`user`角色的权限中应用了过滤器。

在上述应用，我们从一个列出所有文章的简单静态页面开始，继续为 Gatsby 集成 Hasura 作为数据源。我们已经可以从数据库获取文章。然后，我们能够基于登录的用户限制生成的内容。

这是供参考的[源码仓库](https://github.com/praveenweb/dynamic-jamstack-gatsby-hasura/tree/master/dynamic-auth-client)。

## 部署

动态应用现已准备完毕！让我们看如何部署它。正如你现在所知，尽管原始内容来自数据库，但它是一个静态站点，每个页面都是 HTML 文件。它能被部署到不同提供商之上。

但是部署提供商要适配 JAMStack，它需要支持 Git 自动化，源代码托管在`git`，`git push`应触发一个自动化构建，并且在云端部署最新的改变。

受欢迎的选择是 Netlify, Github Pages 和 Heroku，而对 CDN 来说 Cloudflare 是受欢迎的选择。我建议使用 Netlify 部署任何 JAMStack 站点。

### Netlify 部署

在 Netlify 上的部署体验是无缝衔接的。它额外提供了 CDN。Netlify 支持 Git 自动化。在 Netlify 上自动构建，只需要连接你的 git 仓库。

另一个有趣的选项是 Heroku 和 Zeit Now。静态站点也可以使用像 Cloudfront 之类的 CDN 托管在 AWS S3 上。

注意静态网站生成器编译的越快，部署体验越好。

### 其他静态站点生成器

假若，你正在寻找围绕 React 和 Vue 的静态站点生成器，这儿有一些选项：

除了 Gatsby 之外，基于 React 的框架包括[react-static](https://github.com/react-static/react-static) ([Hasura 指导](https://hasura.io/blog/build-react-static-sites-using-hasura-graphql-on-postgres/))。

基于 Vue 的框架包括[Nuxt.js](https://nuxtjs.org/)([Hasura 指导](https://hasura.io/blog/create-nuxt-js-universal-apps-using-graphql-on-postgres/)) 和 [Gridsome](https://gridsome.org/) ([Hasura 指导](https://hasura.io/blog/build-deploy-vuejs-static-sites-gridsome-graphql/)).

一些其他的流行静态网站生成器，包括 ekyll, [Hugo](../../201用hugo搭建免费的个人博客并自动化部署.md), Hexo, VuePress。
他们没有充分利用框架或数据源集成的力量，但是仍然是可靠的开始选择。
