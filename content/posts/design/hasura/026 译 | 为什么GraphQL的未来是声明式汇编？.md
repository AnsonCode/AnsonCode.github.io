---
title: "026 译 | 为什么GraphQL的未来是声明式汇编？"
date: 2022-04-28T23:59:45+08:00
tags: ["技术翻译", "StepZen", "Hasura", "GraphQL"]
categories: ["研发提效"]
featured_image:
description:
draft: true
---

翻译自：[Why the Future of GraphQL is Declarative Assembly](https://stepzen.com/blog/future-of-graphql-is-declarative)

In an earlier blog What GraphQL Can Learn from the World of Databases, we described how a declarative construction of a GraphQL API is the right approach - it enables simplicity of code writing, but equally importantly, it enables optimizations at runtime. In this article, I'll address a companion design pattern - the assembly, or composition, of GraphQL APIs.

How would you build a GraphQL API that supports an eCommerce backend with queries like this?

```gql
{
  customer(email: "john.doe@example.com") {
    name
    orders {
      createdOn
    }
    weather {
      temp
    }
  }
}
```

In StepZen, you assemble the API from each of its components. The code tree might look like this.

```
.
├── config.yaml
├── customer
│   └── Customer.graphql
├── customer-order
│   └── customer-order.graphql
├── customer-weather
│   └── customer-weather.graphql
├── index.graphql
├── order
│   └── order.graphql
└── weather
    └── weather.graphql

```

Each directory represents either a backend or a domain. In other words, it is basically a .graphql file that represents the GraphQL view of the backend or domain. So there is a customer GraphQL subgraph, whose data might be coming from a REST API. There is an order subgraph whose data might be coming from a SQL database.

And then there are "extensions" that connect the various subgraphs together. So for example, customer-order is the stitching of the customer and order subgraphs.

## Each subgraph is built declaratively, and subgraphs are stitched together declaratively

Each subgraph is built declaratively, and subgraphs are stitched together declaratively. For example, customer.graphql is the subgraph that builds out customer from an Airtable backend.

```gql
type Customer {
  name: String
  email: String
  address: String
}
type Query {
  customer(email: String): Customer
    @rest(
      endpoint: "https://api.airtable.com/v0/$baseid/Customers?filterByFormula={email}=%22$email%22&apikey=$apikey"
    )
}
```

The above customer.graphql uses @rest declaration, and order.graphql uses @dbquery declaration.

Three convenient StepZen CLI commands makes it easy to write these subgraphs, by introspecting the backends and generating the schemas for you. See the [Getting Started](https://stepzen.com/getting-started) wizard to try it for yourself using stepzen import curl, stepzen import mysql, stepzen import graphql, and more.

The stitching of subgraphs also happens declaratively. For example, the code in customer-order.graphql is:

```gql
extend type Customer {
  """
  Take Customer's Id and feed it into the Orders Subgraph
  """
  orders: [Order]
    @materializer(
      query: "ordersByCustomer"
      arguments: [{ name: "customerId", field: "id" }]
    )
}
```

In effect, a new field is created in Customer and it is populated by calling a query in the ordersubgraph, passing it the right data from the customersubgraph. That's all there is to it!

## Summary

So as you saw, building subgraphs and composing a supergraph out of them can all be done declaratively. It leads to simpler, cleaner, easier code to write and maintain. And it delivers all the benefits of a better runtime.

Here's how we like to think of how GraphQL should be built:

![](https://stepzen.com/images/blog/supergraph-subgraph-assembly.png)
