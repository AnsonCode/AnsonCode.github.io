---
title: "021 译 | Supabase - 后端即服务：一个真正开源的Firebase替代品"
date: 2022-03-31T22:27:45+08:00
tags: ["技术翻译", "PostgREST"]
categories: ["研发提效"]
featured_image:
description:
draft: true
---

翻译自：[ Supabase - Backend as a service： A truly open source alternative to Firebase ](https://flaming.codes/posts/supabase-backend-as-a-service-firebase-alternative)

## Supabase，一个功能多样的后端套件

Supabase 是一组后端功能，允许你将后端作为服务来管理。它的特性集与 Firebase 特别相近，谷歌的一个建立在”谷歌云平台“上的产品，并且也允许你运行 Baas。

## Firebase 替代品

由于 Supabase 和 Firebase 有核心功能的巨大重叠，Supabase 可以被视为 Firebase 的替代品。提供的特性如下：

- 身份验证管理
- 用于持久存储的数据库
- 文件资产存储，例如图片，视频或文档
- serverless function，至今还没上线

正如你看到的，Supabase 提供的最重要的特性，未来将会有 serverless function。你可以使用 Supabase 的托管服务，同时使用免费套餐在几分钟内启动并运行它。对于更多的使用，你当然可以升级到付费计划。

Supabase 也提供了在开发阶段通过他们 CLI 使用本地设置进行测试的选项。实际上，这意味着它为你提供内建脚手架和生产环境去配置：本地脚手架，云端生产。当然这相当简单，但是我想说明它为完成本地测试提供一个 CLI，这是很棒的。

## Supabase 和 Firebase 之间的不同

尽管拥有相同的特性集，他们的实现和开发策略与 Google’s Firebase 相比是相当不同的。首要的是，Supabase 是完全开源的，这意味着你可以为项目的每个方面做贡献。Firebase 只提供了客户端和管理端 SDK 作为开源方案，隐藏了服务的真实实现。

Supabase 的公开的方法有一个副作用，就是你可以在你的基础设施中托管它。这对于 Firebase 来说是一个巨大的优势，如果需要的话，你可以选择 fork 并开发你的自定义服务。在考虑产品核心组件的寿命时，这是一个重要的方面。

而且，很重要的不同是 [Supabase 使用 PostgreSQL](https://supabase.com/database)，而不是像 Firebase 一样的 NoSQL 数据库。考虑到 Firestore 和 Firebase 数据库 是完全私有的，这也是一个巨大的不同。

PostgreSQL 是一个通用的存储解决方案，允许你在未来在技术上将你的数据从 Supabase 迁移到任何其他 PostgreSQL-DB。

至于 Firebase 提供的其他服务，例如通知和机器学习解决方案，Supabase 一点也不相似。你将仍然需要其他的供应商去开发这些功能，例如 AWS, Azure 实例 或, 当然还有 Firebase.

## 结论

如你所见，Supabase 是一个可行的替代品，如果你喜欢 Firebase 提供的功能，但是不像使用谷歌的产品。同时如果你对未来数据迁移或者 Firebase 服务的私有代码有顾虑，Supabase 作为一个开源产品在它的开发中是完全透明的。正如所述，现在是公测版，但是可以在中等规模的项目中使用。
