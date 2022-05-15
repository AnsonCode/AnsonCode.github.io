---
title: "024 译 | GraphQL, REST, JSON-Schema 和 HTTP2的融合"
date: 2022-04-19T23:59:45+08:00
tags: ["技术翻译", "WunderGraph", "GraphQL"]
categories: ["研发提效"]
featured_image:
description:
draft: true
---

翻译自：[The Fusion of GraphQL, REST, JSON-Schema and HTTP2](https://wundergraph.com/blog/the_fusion_of_graphql_rest_json_schema_and_http2)

本文涉及 GraphQL, REST, JSON-Schema 和 HTTP2 的融合。我打算说服你，你不必在 GraphQL 和 REST 中进行选择。反而，我将提出一个解决方案，给你最好的（体验）。

围绕 REST vs GraphQL 的话题有无穷无尽的讨论。事实上，他们都很好，但是如果你选择其中一方，你将意识到这是一个权衡。你可能会陷入“兔子洞”，并且为您的企业在不同的 API 风格间做出艰难的抉择。

但是如果没有必要，为什么要选择？为什么不从每个 API 风格各取所长，并且组合他们？

我们将从常见的误解开始讨论，并看看两个对立的阵营。我们进一步确定这两个方法地优缺点。我们将研究一个结合 REST 和 GraphQL 的解决方案，使用一点 JSON-Schema 和 HTTP2 的益处。

想象下，你能将 REST 的强大和 HTTP 的灵活性与最流行的查询语言结合起来吗？你将意识到，如果你有偏向性，你将错过很多潜力。然而，你不必在两者中做出选择。您所要做的就是重新思考你的 APIs 模型。

先把你的信仰放在一边。试着在阅读的同时，摒弃任何评判。你将看到我们能 使得 GraphQL RESTful 化，并且它将变得更好！

让我们开始！

## 这两个阵营，以及为什么他们很难在一起工作

近几年，我有机会与很多 API 从业者讨论，从自由职业到中小型公司以及大型企业。

我了解到我们通常把人们安置在两个阵营之一。

第一类是使用 REST APIs 的人。他们通常在 API 设计上有非常强烈的意见，他们很了解 REST API 是什么以及他们的优势在那里。他们很熟悉像 OpenAPI 规范之类的工具。他们或许读了 Roy Fielding 关于 REST 的学术论文，并且知道 Richardson Maturity 模型的一些东西。

第一类也有一个弱点。他们太自信了。当你开始与这个组的人讨论 GraphQL 时，你将遇到很多阻力。很多时候，他们有很好的理由反驳，但话说回来，他们通常缺乏倾听的能力。

他们的解决方案是 REST API。几乎不可能说服他们去尝试新事物。

在另一边，是 GraphQL 狂热者。他们都十分称赞 GraphQL。(Most of them praise GraphQL way too hard.)如果你看下他们的论点，很明显他们缺乏 APIs 的基本常识。这类人比第一类人年轻很多。
这就可以理解了，这类人缺乏经验。他们经常称赞 GraphQL 特性优于 REST ，然而在现实中，他们的 REST API 设计没有进行优化。在 GraphQL 中几乎没有任何东西，你不能通过一个好的 REST 设计解决。如果第二类人知道这些，他们的生活将变得容易很多。

除了这两类主要群体外，也有两类更小的群体。

一个是非常有经验的 API 爱好者。他们的主要关注点是 REST APIs，但是他们也对其他 API 风格开放。他们理解不同的 API 风格服务不同的目的。因此，你可以在某种场景下使用 GraphQL。

第二类利基群体是更擅长 GraphQL 的用户。他们经历了最初的炒作周期，并且意识到 GraphQL 不是银弹。他们理解查询语言的优势，但是明白使用它的挑战。他们有很多围绕安全和性能的挑战需要被解决，正如我再[另一篇文章](https://wundergraph.com/blog/the_complete_graphql_security_guide_fixing_the_13_most_common_graphql_vulnerabilities_to_make_your_api_production_ready)写的那样。

如果你看 Facebook 和 GraphQL 的早期采用者，像 Medium, Twitter 和 Netflix，你将意识到 [GraphQL 并不意味着要在网络上公开](https://wundergraph.com/blog/graphql_is_not_meant_to_be_exposed_over_the_internet)。至今，在 GraphQL 社群的很多人构建开源工具恰恰可以做到这些。这些框架直接将 GraphQL 暴露给客户端，忽略定义网络，HTTP 和 REST 主要规范所做的工作。

这导致我们花费数年在网络扩展上做的工作被扔进垃圾箱，然后重新编写以兼容 GraphQL。这是一个巨大的时间和资源的浪费。当我们可以在 REST 之上构建并利用这些现存解决方案时，为什么构建所有这些工具，忽略 REST 的存在？

但是为了理解这些，我们首先必须谈论 RESTful 意味着什么。

## 当 API 是 RESTful 时，意味着什么？

让我们看下 Roy Fielding 的学术论文，和 Richardson Maturity Model （成熟度模型）去更好的理解 RESTful 意味着什么。

简要来说，RESTful API 能够尽可能高效的利用现存 WEB 架构。

REST 不是一个 API 规范，它是一个带有一系列约束的架构风格。如果你遵循这些约束，你将使你的 API 与网络现存的东西兼容。RESTful APIs 可以利用 CDNs，代理，标准 web 服务和框架以及浏览器。同时，我们并不清楚，是否应该遵循所有约束或者哪一个是最重要的。此外，没有 REST API 看起来像另一个，因为约束留下了很大解释空间。（Additionally, no REST API looks like another as the constraints leave a lot of room for interpretation.）

首先，让我们分析下 Fieldings 的论文：

### 客户端-服务器

第一个约束是关于将应用分为客户端和服务端去分离关注点。

### 无状态

介于客户端和服务端的交流应该无状态。也就是说，每个来自客户端到服务端的请求包含服务端处理请求的所有需要的信息。

### 缓存

Responses from the server to the client should be able to be cached on the client side to increase performance. Servers should send caching metadata to the client so that the client understands if a response can be cached, for how long it can be cached and when a response could be invalidated.

### Uniform Interface#统一接口

Both client- and servers should be able to talk over a uniform interface. Implementations on both sides can be language- and framework agnostic. By only relying on the interface, clients and server implementations can talk to eachother even if implemented in different languages.

This is by far one of the most important constraints that make the web work.

### Layered System#分层系统

It should be possible to build multiple layers of systems that complement another. E.g. there should be a way to add a Cache Server in front of an application server. Middleware systems, like API Gateways, could be put in front of an application server to enhance the application capabilities, e.g. by adding authentication.

### Code-On-Demand#可定制代码

We should be able to download more code at runtime to extend the client and add new functionality.

Next, let's have a look at the Richardson Maturity Model. This model defines four levels, from zero to three which indicate the maturity of a REST API.

### Why REST constraints matter#为什么 REST 约束很重要

Why do these constraints matter so much?

> The Web is built on top of REST. If you ignore it, you ignore the Web.

Most of the standardized components of the web acknowledge HTTP and REST as a standard. These components are implemented in ways to make them compatible with existing RFCs. Everything relies on these standards.

CDN services, Proxies, Browsers, Application Servers, Frameworks, etc... All of them adhere to the standards of the Web.

Here's one simple example. If a client is sending a POST request, most if not all components of the web understand that this operation wants to make a change. For that reason, it is generally accepted that no component of the web will cache this request. In contrast, GET requests indicate that a client wants to read some information. Based on the Cache-Control Headers of the response, any intermediary, like a Proxy, as well as a Browser or Android client is able to use standardized caching mechanisms to cache the response.

So, if you stick to these constraints, you're making yourself compatible to the web. If you don't, you'll have to re-invent a lot of tooling to fix the gaps that you've just created.

We'll talk about this topic later, but in a nutshell, this is one of the biggest problems of GraphQL. Ignoring the majority of RFCs by the IETF leads to a massive tooling gap.

### Richardson Maturity Model: Level 0 - RPC over HTTP# 理查森成熟度模型：水平 0 -基于 HTTP 的 RPC

Level 0 means, a client sends remote procedure calls (RPC) to the server using HTTP.

### Richardson Maturity Model: Level 1 - Resources#理查森成熟度模型：水平 1 -资源

Level 1 introduces Resources. So, instead of sending any type of RPC and completely ignoring the URL, we're now specifying Resources using a URL schema.

E.g. the Resource users could be defined as the URL example.com/users. So, if you want to work with user objects, use this URL.

### Richardson Maturity Model: Level 2 - HTTP Verbs#理查森成熟度模型：水平 2- HTTP 动词

Level 3 adds the use of HTTP Verbs. E.g. if you want to add a user, you would send a POST request to /users. If you want to retrieve a user, you could to so by sending a GET request to /users/1, with 1 being the user ID. Deleting a user could be implemented sending a DELETE request to /users/1.

Level 2 of the RMM makes a lot of sense for most APIs. It gives REST APIs a nice structure and allows them to properly leverage the existing infrastructure of the web.

### Richardson Maturity Model: Level 3 - Hypermedia Controls#理查森成熟度模型：水平 3 -超媒体控制

Level 3 is the one that's usually confusing beginners a lot. At the same time, Hypermedia Controls are extremely powerful because they can guide the API consumer through a journey.

Here's a simple example of how they work. Imagine, you're making a REST API call to book a ticket for an event. You'll get a response back from the API that tells you the ticket is booked, awesome! That not all though, the response also contains additional "Hypermedia Controls" that tell you about possible next steps. One possible next step could be that you might want to cancel the ticket because you chose the wrong one. In this case, the response of the booked ticket could contain a link that lets you cancel the event. This way, the client doesn't have to figure out by itself what to do next, the response contains all the information so that the client is able to continue the "API journey".

This sounds like a really nice API consumer experience, right? Well, not really. Hypermedia Controls have an issue. By definition, there's no specification of what exactly these controls are. A response could contain any kind of controls without a client knowing what exactly to expect.

If both client and server are owned by exactly the same people, this pattern could work extremely well. If you add new hypermedia controls to an API response, you can add new code to your client that automatically handles these controls. What if the people who provide the API are not the ones who consume it? How do you communicate these changes? Wouldn't you need a specification for the controls? If you specify the controls, how is it then compatible with the idea that each API response can return whatever Hypermedia controls it wants? It's not, and that's why we don't see many Hypermedia APIs.

As I said before, Level 3 is extremely powerful. At the same time, it's hard to understand and even more complex to get right which is the biggest reason why most people don't even try.

The majority of API practitioners sticks to Level 2. Good URL design, combined with the use of HTTP Verbs, ideally with an OpenAPI definition gets you very far!

Let's recap this section so that we can use the essential takeaways and move forward to analyze GraphQL.

REST is not a specification, it's a set of constraints
Ignoring REST means, you're ignoring the existing infrastructure of the web
At the same time, you'll have to build a lot of new tools to fix the gaps
Not being RESTful means, not being compatible to the web
Alright, now that we've got a common sense of what REST really is about, let's analyze how RESTful GraphQL is.

Once we've done that, we'll look into ways of improving it.

## How RESTful is GraphQL?#RESTful 如何是 GraphQL？

### GraphQL and the Client Server Model# GraphQL 和客户端服务端模型

GraphQL, by definition, divides the implementation into client and server. You have a GraphQL server that implements a GraphQL Schema. On the other side, GraphQL clients can talk to the server using HTTP.

So, yes, GraphQL embraces the client server model.

### Is GraphQL Stateless?#GraphQL 是无状态的吗？

This one is going to be a bit more complex. So, let's quickly recap what stateless means.

This constraint says that each client request contains all the information required by the server to be able to process the request. No Sessions, no "stateful" data on the server, no nothing. Just this one single request and the server is able to return a response.

GraphQL Operations can be divided into three categories. Queries, Mutations and Subscriptions.

For those who don't know too much about GraphQL, Queries let clients ask for data, Mutations let client mutate data, Subscriptions allow clients to get notified when something specific changes.

If you're sending Queries and Mutations over HTTP, these requests are stateless. Send along a cookie or authentication token and the server can process the request and reply with a response.

The issue arises from Subscriptions, and the way most implementations deal with them. Most GraphQL implementations use a standard defined by Apollo to implement Subscriptions over WebSockets. This standard is an absolute nightmare because it will be responsible for technical debt for many more years to come. I'm not blaming the authors. I think it's a good first start and I could have probably come up with a similar solution. That said, I think it's time to revisit the topic and cleanup the technical debt before it's too late.

What's the problem with WebSockets? Wrong question, sorry! What are THE problems with WebSockets?

If a client wants to initiate a WebSocket connection, they start to by doing an HTTP Upgrade Request to which the server has to reply that the protocol change (from HTTP to TCP) was accepted. Once that happened, it's a plain TCP socket with some extras like frames etc... The user can then define their own protocols to send data back and forth between client and server.

The first problem has to do with the WebSocket specification of HTML. More specifically, it's not possible to specify Headers for the Upgrade Request. If your authentication method is to send an Authorization Header with a Bearer Token, you're out of luck with WebSockets.

What are the alternatives?

You could let the client make a login request first and set a cookie. Then, this cookie would be sent alongside the Upgrade Request. This could be a solution, but it's not ideal as it adds complexity and makes the Request non-stateless, as we're depending on a preceding request.

Another solution would be to put the token in the URL as a Query Parameter. In this case, we're risking that some intermediary or middleware accidentally (or intentionally) logs the URL. From a security point of view, this solution should be avoided.

Most users of WebSockets therefore took another route of solving the problem. They've implemented some custom protocol on top of WebSockets. This means, client and server would use specific messages to authenticate the client. From a security standpoint, this is ok, but it adds significant complexity to your application. At the same time, this approach essentially re-implements parts of HTTP over WebSockets. I would always avoid re-inventing wheels. Finally, this approach is also non-stateless. First, you initiate the socket, then you negotiate a custom protocol between client and server, send custom messages to authenticate the user to be then able to start a GraphQL Subscription.

The next issue is about the capabilities of WebSockets and the misfit for GraphQL Subscriptions. The flow of a GraphQL Subscription goes like this: The client sends a Subscription Operation to the server. The server validates it and starts executing it. Once new data is available on the server, it'll be sent to the client. I hope it's obvious but happy to make it very explicit: GraphQL has no requirements for bidirectional communication. With that in mind, WebSockets allow the client to send data to the server all the time. This means, a malicious client could spam the server with garbage messages. If you wanted to solve this problem, you'd have to look into every message and block misbehaving clients. Wouldn't it be better if you just don't have to deal with the problem at all?

It's four issues already, and we haven't even started talking about the GraphQL over WebSockets specification.

I know, we've talked a lot about non GraphQL related problems, but the main topic of this section is about the client server communication being stateless.

So, if we look at the GraphQL over WebSockets protocol again, we'll see that it's everything but not stateless. First, the client has to send an init message, then it can send start and stop messages to manage multiple subscriptions. So, the whole purpose of this specification is to manually multiplex multiple Subscriptions over one single WebSocke connection. I wrote about this topic a while ago if this topic is of special interest to you. If we break this down a bit, we've got all the issues related to WebSockets outlined above, plus a spec to multiplex many subscriptions over a single TCP connection in userspace. By userspace, I mean that this multiplexing code must be implemented by both the client and the server.

I'm pretty sure you've heard about HTTP/2 and HTTP/3. H2 can multiplex multiple Streams out of the box without all the issues described in this paragraph. H3 will improve the situation even further as it eliminates the problem of individual requests blocking each other. We'll come back later to this when talking about the solution. In any case, avoid WebSockets if you can. It's an old HTTP 1.1 specification and there haven't been any attempts to improve it and H2 makes it obsolete.

To sum up the section of statelessness. If all you do is to send Queries and Mutations over HTTP, we could call it stateless. If you add Subscriptions over WebSockets, it's not stateless anymore.

Think about what happens if the user authenticates, then starts the WebSocket connection, then logs out again, and logs in with another account while the WebSocket connection is still alive because you forgot to close it. From the server side perspective, what is the identity of the user that is starting a Subscription over this WebSocket connection? Is it the first user who is already logged out? This shouldn't be.
