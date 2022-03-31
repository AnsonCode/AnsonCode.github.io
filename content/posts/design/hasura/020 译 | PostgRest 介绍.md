---
title: "020 译 | PostgREST 介绍"
date: 2022-03-23T01:27:45+08:00
tags: ["技术翻译", "PostgREST"]
categories: ["研发提效"]
featured_image:
description:
draft: true
---

翻译自：[ PostgREST Documentation ](https://postgrest.org/en/stable/)

PostgREST 是一个单体 web 服务，它可以将 PostgreSQL 数据库直接变成 RESTful API。数据库中的结构约束和权限决定 API 端点和操作。

## 动机

PostgREST 是手动 CRUD 编程的替代品。自定义 API 服务存在问题。写业务逻辑经常会重复，忽略或阻碍数据库结构。对象关系映射是一个有缺陷的抽象，导致减慢命令式代码。PostgREST 理念建立了单一声明式数据源：数据本身。

## 声明式变成

要求 PostgREST 为你去连接数据很容易，并且让查询规划器理解细节而不是你自己逐行遍历。为数据对象分配权限比在控制器中增加守卫更容易。（对于数据依赖中的级联权限来说尤其如此）设置约束比在代码使用合理性检查更容易。

## Leak-proof Abstraction

这与 ORM 无关。在 SQL 中创建新视图具有已知的性能影响。数据库管理员现在可以从头开始创建 API，而不需要自定义编程。

## 做好一件事

PostgREST 有一个集中的作用域。它和其他工具例如 Nginx 配合的很好。这促使你清楚地从其他问题中分离中心数据的 CRUD 操作。使用一组锋利的工具，而不是构建一个大的泥球。

**===待续===**

## API

### 表和视图

公开模式且可访问的所有视图和表

## 角色系统概述
