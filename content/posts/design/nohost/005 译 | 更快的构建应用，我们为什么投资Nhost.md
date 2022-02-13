---
title: "005 译 | 更快的构建应用，我们为什么投资Nhost"
date: 2022-02-09T20:01:09+08:00
tags: ["技术翻译", "Nhost"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

翻译自：[Build apps faster, or why we invested in Nhost!](https://medium.com/nauta-capital/build-apps-faster-or-why-we-invested-in-nhost-5fb602d87559)

我们非常相信并且支持开源软件。因此，我们很激动能在 Nhost 领投 3 百万美金种子轮融资，一个坐落于 Stockholm 和 Berlin 的初创企业，和令人称奇的创始人 Johan 和 Nuno 一起踏上他们令人兴奋的旅程，去挑战现在的竞争对手 [Firebase](https://firebase.google.cn/)。

![Nhost创始人Johan 和 Nuno](https://gitee.com/caoyanbin/picgo/raw/master/img/20220209205925.png)

自从在风险投资行业工作，我就没有像我喜欢的那样从事编程了。因此，我非常兴奋去尝试一个推荐给我叫做 [Nhost](https://nhost.io/) 的平台。我喜欢写代码，我对建立和维护基础设施不感兴趣。我对 Nhost 的易用性感到惊喜，几分钟内我就拥有了一个带着 GraphQL API 和身份验证的 PostgreSQL 数据库。

没有太多前端开发经验，我跟着 [Nhost 文档](https://docs.nhost.io/get-started)(已翻译)快速建立了一个 React 应用。我最喜欢的项目是去建立一个简单的 web 应用去搜索我通过 GitHub API 获取的开源项目列表。我使用 Netlify 让我的项目随时可用。（部署到 Netlify 上）

为这个项目，我主要需要用到 3 个工具：Nhost, [Netlify](https://medium.com/u/5250f9d9bd2f?source=post_page-----5fb602d87559-----------------------------------) 和 [GitHub](https://medium.com/u/8df3bf3c40ae?source=post_page-----5fb602d87559-----------------------------------)(公平一点，我还会写一点 Python 脚本获取数据)。这个事实让我不能更兴奋了，仅仅几个月 Nauta 资本 和 GitHub 创始人 [Tom Preston-Werner](https://twitter.com/mojombo) 、 [Scott Chacon](https://twitter.com/chacon)以及 Netlify 创始人 [Christian Bach](https://twitter.com/chr_bach) 、 [Mathias Biilmann](https://twitter.com/biilmann)(还有其他强大的天使投资人) 一起投资 Nhost。

# 关于 Nhost

Nhost 是一家总部位于 Stockholm 和 Berlin-based 的技术公司，为开发者开始构建他们的 web 或移动应用提供所有必须的基础架构。

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220209144145.png)

构建并维护一个应用程序的后端是耗时并且需要专业知识的。开发者需要

- (i)建立，管理并配置他们的云基础架构
- (ii)编写应用层（建立用户管理、AIIs、存储、确定技术栈）
- (iii)维护应用（安全、性能、扩展性、更新）

然而，全球仅大概有 27 百万软件开发者（占总人口的 3.3%），在美国有 500,000+的软件工程职位空缺，并且在该领域薪水不断增加。同时， web 应用的 80%由样板代码组成，例如权限管理，用户管理，消息或者通知。一次又一次的建立这些功能减少了开发者集中在他们软件与竞争对手不同之处的时间。

此外，管理后端需要时间和精力，很多应用从用户的视角开始规划和设计。前端是第一次也是最重要的触点，将在很多场景下指导应用程序开发。在这种情况下，后端变成了商品（通用），前端是主要的区别点。

Nhost 自动启动一个后端，带有关系数据库、GraphQL API、身份验证以及存储（权限、用户管理）。为此，他们使用久经考验和开源的技术，[PostgreSQL](https://github.com/postgres/postgres) 作为关系数据库，[Hasura](https://github.com/hasura/graphql-engine) 管理 API 层， [Minio](https://github.com/minio/minio) 和一系列强大的权限规则作为存储。此外，Nhost 为完善这个包增加身份验证，并且把它的核心技术作为开源项目[ hasura-backend-plus](https://github.com/nhost/hasura-backend-plus)发布。

所有的业务操作和基础架构是自动化的，并由 Nhost 平台处理。在这种方式下，前端开发者能够使用他们喜欢的技术，例如 React 或者 Next.js ，立即构建他们的 web 应用。

[YouTube 演示视频](https://youtu.be/v3I5_2t1cco)

# 喜欢 Nhost 的多种原因

作为一名投资者，你总是对杰出的创业者感到兴奋。在 Nhost 的例子中，Johan 和 Nuno 绝对是我们投资它的主要原因。

Johan 是一个非常具有企业家精神的人，在高中的时候就建立了他的第一家企业。他在瑞典布京理工学院学习计算机科学，主要研究信息技术安全。不是找一个高薪的工作，他决定帮助他的父亲扩大家族生意。作为兼职做过一些 web 应用，他开始建立 Nhost。他对企业解决方案充满热情，并且有清晰的使命。

他的联合创始人 Nuno 是一个强大的技术领导者。在 Técnico 获得计算机工程学士学位后，他搬到 Berlin 在几家快速增长的企业像 Delivery Hero 和 Marley Spoon 去积累经验。他在基础架构企业像 CodeShip 和 Cloudbees 有宝贵的经验，并且知道如何去建立一个商业化开源公司。

下面是一个不完整的列表，关于 Nhost 让我们感到兴奋的点：

1. **初期引入**：没有任何市场推广，截止到 2020 年底早已有 1400+用户注册了 Nhost。截止到现在，在他们的平台
   已经创建超过 700 个项目。
2. **开源**：我们预计开源产品将在软件市场占据主导地位。我们感觉有很多依赖私有产品（Google Firebase）的开发者开始不安起来，我们也相信一个开源的玩家将颠覆这个市场。
3. **技术**：公司将久经考验的技术像 PostgreSQL 与新兴的开发模式像 GraphQL 结合，正好这也是应用开发者一直在寻找的组合。
4. **以用户为中心的设计**：很多应用从用户的视角开始规划和设计。前端是第一个也是最重要的触点，将在很多情况下指导用户开发。在这种场景下，后端或许变的商品化，前端是主要的区别点。对此，后端即服务（BaaS）是一个最好的答案。
5. **时机把握**：Nhost 发现它正处在一些大趋势中间（大势所趋）。由于在开发者训练营的教育中断，产生了很多前端开发者，而且这个数字还在不断增长（后端没学好去学了前端）。同时，前端框架像 React 开始越来越强大，并且允许在客户端做一些之前只能在后端做的事情，最终，提供了更多标准的后端（升级为 BaaS 玩家）和前端服务（Netlify 和 Vercel 等 Jamstack 平台）。此外，开发者对现有的框架如 Firebase 充满怀疑，由于谷歌关闭了大量[老产品](https://killedbygoogle.com/)，并且不通知平台的价格变化。因此，开发者寻找更小供应商锁定的解决方案，例如开源，同时根据价值观，优先考虑他们信任的企业。最终，GraphQL 的新 API 范例以及它的管理使用强有力（开源）平台像 Hasura 或 Apollo ，正在创造前所有未有的生产力。

亲自体验了让应用程序运行在 Nhost 上是多么简单，我们非常期待看到公司为我们制定的未来规划。

最初在 2021 年 4 月 23 日在https://nautacapital.com上发布。
