---
title: "008【翻译】在前端客户端上处理JWTs的终极指南（GraphQL）"
date: 2022-02-11T22:52:37+08:00
tags: ["技术翻译", "Hasura"]
categories: ["研发提效"]
featured_image:
description:
draft: false
---

[The Ultimate Guide to handling JWTs on frontend clients (GraphQL)](https://hasura.io/blog/best-practices-of-using-jwt-with-graphql/)

JWTs (JSON Web Token, pronounced 'jot') 正变成一个流行的处理身份认证的方式。本文目标是阐明 JWT 是什么，讨论它的优缺点以及涵盖在客户端实现 JWT 的最佳实践，牢记安全。

尽管，我们使用 [GraphQL](https://hasura.io/graphql/) 客户端演示，但是这个概念适用于任何前端客户端。

> 注意：该指导最早发布于 2019 年 9 月 9 日。 最后更新于 2022 年 1 月 4 日。

# 介绍：什么是 JWT?

## 安全性考虑

## JWT 结构

# 基础：登录

现在，我们对什么是 JWT 有了基础的理解，让我们创建一个简单的登录流程，并且提取 JWT。下面是我们想要实现的内容：

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220211234729.png)

**所以，我们如何开始？**

这个登录过程和你通常做的实际上没有什么不同。例如，这儿有一个登录表单，提交一个用户名、密码到身份验证端点，并且从响应中获取 JWT 令牌。这可以是使用外部提供者登录、OAuth 和 OAuth2 的步骤。它真的不重要，只要客户端最终在最终登录成功步骤的响应中获取一个 JWT 令牌。

首先，我们将建立一个简单的登录表单给我们的登录服务器发送用户名和密码。服务将签发 JWT 令牌，并且我们将在内存中存储它。在这个练习中，我们不将注意力放在身份授权服务的后端，但是欢迎你在本文的示例仓库中查看它。

下面是登录按钮的`handleSubmit`处理函数的样子：

```js
async function handleSubmit() {
  //...
  // Make the login API call
  const response = await fetch(`/auth/login`, {
    method: "POST",
    body: JSON.stringify({ username, password }),
  });
  //...
  // Extract the JWT from the response
  const { jwt_token } = await response.json();
  //...
  // Do something the token in the login method
  await login({ jwt_token });
}
```

登录 API 返回一个**令牌**，然后我们把令牌传递给来自`/utils/auth`的`login`函数，在那里，我们可以决定一旦我们获取令牌后我们如何处理它。

```js
import { login } from "../utils/auth";
await login({ jwt_token });
```

**所以，我们已经得到了令牌，现在我们要把令牌存储在那里？**

我们需要在某处保存我们的 JWT 令牌，以便于我们能把它作为一个头部传递给我们的 API。你或许尝试在 `localstorage`中持久化它。不要这样做！这很容易受到 XSS 攻击。

**把它存在 cookie 中怎么样？**
在客户端创建 cookies 保存 JWT 也容易受到 XSS。如果它可以在客户端被来自应用外的 JavaScript 读取，它可能被窃取。你或许认为 HttpOnly cookie（被服务端创建而不是客户端）将有帮助，但是 cookies 容易受到 [CSRF](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) 攻击。值得注意的是，HttpOnly 和合理的 CORS 策略不能避免 CSRF 表单提交攻击，并且使用 cookies 需要适当的 CSRF 缓和策略。

注意， [SameSite cookie](https://owasp.org/www-community/SameSite)将 使基于 cookie 的方法免受 CSRF 攻击。如果你的 Auth 和 API 服务托管在不同的域名下，它或许不是一个解决方案，否则它将真的工作的很好。

**那我们把它存在那里呢？**
[The OWASP JWT Cheatsheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.md) 和 [OWASP ASVS (Application Security Verification Standard)](https://github.com/OWASP/ASVS) 制定了处理和存储令牌的指导方针。

这部分与在 JWT Cheatsheet 中的 `Token Storage on Client Side `和 `Token Sidejacking` 议题，以及 ASVS 的 第三章 (`Session Management`) 和 第八章 (`Data Protection`) 相关。

来自备忘单，`议题：客户端令牌存储`：

> 当应用以一种表现下列行为的方式存储令牌令牌时，这种情况就会出现：

- 由浏览器自动发送（Cookie 存储）
- 即使浏览器重启也能找到（使用浏览器 localStorage 容器）
- 在出现 XSS 问题时检索（Cookie 可以被 JavaScript 代码访问或令牌存储在浏览器本地或 session 存储中）

  如何避免：

- 使用浏览器`sessionStorage`容器存储令牌
- 当调用服务时，使用 JavaScript 把它添加到 Bearer HTTP `Authentication` 头中
- 为令牌增加`fingerprint指纹`信息

将令牌存储在浏览器 sessionStorage 容器中，它暴露了令牌，使它能被 XSS 攻击窃取。然而，为令牌增加指纹避免被盗的令牌在攻击者自己的机器上重复使用。为了关闭一个攻击者最大限度的攻击面，为浏览器增加`Content Security Policy（内容安全策略）`去强化执行上下文。

`fingerprint指纹`是参考`Token Sidejacking `议题以下指导方针的实现。

> 当令牌被攻击者拦截或盗取时，攻击出现，他们使用令牌获取目标用户身份的系统权限。

如何避免：

一种方式是在 token 中增加“用户上下文”。用户上下文由以下信息组成：

- 随机字符串在身份验证阶段生成。它将作为一个加强的 cookie 被发送到客户端（标识：HttpOnly + Secure + SameSite + cookie 前缀）。
- 随机字符串的 SHA256 hash 将存储到令牌中（而不是原始值），以防止任何 XSS 问题，允许攻击者读随机的字符串值并且设置预期的 cookie。

IP 地址不应该被使用，因为在一些合理的场合下，IP 地址会在同一个会话中发生变化。例如，当用户通过移动设备访问应用时，手机运营商在交换期间改变，然后 IP 地址或许（经常）变化。而且，使用 IP 地址会造成符合欧洲 GDPR 法规的潜在问题。

在令牌校验期间，如果接收到的令牌不包含正确的上下文（例如，如果它被重放），它一定会被拒绝。

在客户端的一个实现可能如下：

```js
// Short duration JWT token (5-10 min)
export function getJwtToken() {
  return sessionStorage.getItem("jwt");
}

export function setJwtToken(token) {
  sessionStorage.setItem("jwt", token);
}

// Longer duration refresh token (30-60 min)
export function getRefreshToken() {
  return sessionStorage.getItem("refreshToken");
}

export function setRefreshToken(token) {
  sessionStorage.setItem("refreshToken", token);
}

function handleLogin({ email, password }) {
  // Call login method in API
  // The server handler is responsible for setting user fingerprint cookie during this as well
  const { jwtToken, refreshToken } = await login({ email, password });
  setJwtToken(jwtToken);
  setRefreshToken(refreshToken);

  // If you like, you may redirect the user now
  Router.push("/some-url");
}
```

是的，当用户切换 tabs 时，令牌将为空，但是我们将稍后处理它。

**好的！现在我们我们有了令牌，我们能用它做什么？**

- 在 API 客户端使用它作为一个 header 传递给每个 API 调用
- 通过看 JWT 变量是否被设置，核查用户是否登录
- 可选的，我们甚至能在客户端解码 JWT 访问负载中的数据。

假设在客户端我们需要 user_id 或用户名，我们能从 JWT 中获取。

**如果我们已经登录，我们如何核查？**

如果令牌变量已设置，继续执行，不然的话跳转到登录页。

```js
const jwtToken = getJwtToken();
if (!jwtToken) {
  Router.push("/login");
}
```

# 基础：客户端设置

现在，是时候设置我们的 GraphQL 客户端了。从我们设置的变量中获取令牌，如果它存在，我们把它传递给 GraphQL 客户端。

![Using the JWT in a GraphQL client](https://gitee.com/caoyanbin/picgo/raw/master/img/20220212193853.png)

假定你的 GraphQL API 以`Authorization`头的形式接收 JWT 授权令牌，你需要去做的就是设置你的客户端从变量中使用 jwt 令牌设置 http 头。

以下是使用 Apollo GraphQL 客户端的 ApolloLink 中间件设置。

```js
import { useMemo } from "react"
import { ApolloClient, InMemoryCache, HttpLink, ApolloLink, Operation } from "@apollo/client"
import { getMainDefinition } from "@apollo/client/utilities"
import { WebSocketLink } from "@apollo/client/link/ws"
import merge from "deepmerge"

let apolloClient

function getHeaders() {
    const headers = {} as HeadersInit
    const token = getJwtToken()
    if (token) headers["Authorization"] = `Bearer ${token}`
    return headers
}

function operationIsSubscription(operation: Operation): boolean {
    const definition = getMainDefinition(operation.query)
    const isSubscription = definition.kind === "OperationDefinition" && definition.operation === "subscription"
    return isSubscription
}

let wsLink
function getOrCreateWebsocketLink() {
    wsLink ??= new WebSocketLink({
        uri: process.env["NEXT_PUBLIC_HASURA_ENDPOINT"].replace("http", "ws").replace("https", "wss"),
        options: {
            reconnect: true,
            timeout: 30000,
            connectionParams: () => {
                return { headers: getHeaders() }
            },
        },
    })
    return wsLink
}

function createLink() {
    const httpLink = new HttpLink({
        uri: process.env["NEXT_PUBLIC_HASURA_ENDPOINT"],
        credentials: "include",
    })

    const authLink = new ApolloLink((operation, forward) => {
        operation.setContext(({ headers = {} }) => ({
            headers: {
                ...headers,
                ...getHeaders(),
            },
        }))
        return forward(operation)
    })

    if (typeof window !== "undefined") {
        return ApolloLink.from([
            authLink,
            // Use "getOrCreateWebsocketLink" to init WS lazily
            // otherwise WS connection will be created + used even if using "query"
            ApolloLink.split(operationIsSubscription, getOrCreateWebsocketLink, httpLink),
        ])
    } else {
        return ApolloLink.from([authLink, httpLink])
    }
}

function createApolloClient() {
    return new ApolloClient({
        ssrMode: typeof window === "undefined",
        link: createLink(),
        cache: new InMemoryCache(),
    })
}

export function initializeApollo(initialState = null) {
    const _apolloClient = apolloClient ?? createApolloClient()

    // If your page has Next.js data fetching methods that use Apollo Client, the initial state
    // get hydrated here
    if (initialState) {
        // Get existing cache, loaded during client side data fetching
        const existingCache = _apolloClient.extract()

        // Merge the existing cache into data passed from getStaticProps/getServerSideProps
        const data = merge(initialState, existingCache)

        // Restore the cache with the merged data
        _apolloClient.cache.restore(data)
    }

    // For SSG and SSR always create a new Apollo Client
    if (typeof window === "undefined") return _apolloClient
    // Create the Apollo Client once in the client
    if (!apolloClient) apolloClient = _apolloClient

    return _apolloClient
}

export function useApollo(initialState) {
    const store = useMemo(() => initializeApollo(initialState), [initialState])
    return store
}
```

正如你从代码中看到的，无论何时有一个令牌，它作为一个 header 被传递到每个请求中。

**但是，如果没有令牌将发生什么？**

他依赖于你应用的流程。假设你将用户重定向到登录页：

```js
else {
 Router.push('/login')
}
```

**如果令牌在我们使用时过期了，将发生什么？**

假设我们的令牌仅仅 15 分钟有效。在这种场景下，我们或许将从我们的 API 中获得一个拒绝请求的错误（假设 401 ：未授权错误）。切记，知道如何去使用 JWT 的每个服务能够独立验证它并且核查它是否过期。

让我们为应用增加错误处理去处理该场景。我们将写为每个 API 响应运行且核查错误的代码。当我们从 API 收到令牌过期或非法错误，我们触发注销或跳转到登录工作流。

如果我们正使用 Apollo 客户端，这是代码的样子：

```js
import { onError } from "apollo-link-error";

const logoutLink = onError(({ networkError }) => {
  if (networkError.statusCode === 401) logout();
});

if (typeof window !== "undefined") {
  return ApolloLink.from([
    logoutLink,
    authLink,
    ApolloLink.split(
      operationIsSubscription,
      getOrCreateWebsocketLink,
      httpLink
    ),
  ]);
} else {
  return ApolloLink.from([logoutLink, authLink, httpLink]);
}
```

你或许注意到，这将导致相当糟糕的用户体验。用户将在每次令牌过期时不断被要求重新授权。这是为什么应用实现了一个静默刷新流程，在后台不断刷新 JWT 令牌。在下面的部分会有更多相关内容！

# 基础：注销

使用 JWTs，“注销”只是在客户端删除令牌，以便它不能被之后的 API 调用使用。

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220212200912.png)

**所以，是否根本没有`注销` API 调用？**

`注销`端点真的不需要，因为任何接收 JWTs 的微服务将持续接收它。如果你的认证服务删除了 JWT，它也没关系，因为不管怎样其他的服务将保持接收它（因为所有的 JWTs 端点不需要中心协调）。

**令牌依然合法，且能被使用。如果我需要确保令牌不能被再次使用，怎么做？**

这是为什么 将 JWT 到期值保持小的值是很重要。并且，这是为什么确保你的确保你的 JWTs 不被盗是更重要的了。令牌合法（即使你在客户端删除它），但是只能在短期内降低它被非法使用的可能性。

此外，你可以为 JWTs 增加一个禁用列表。这样，你可以有一个`/login`API 调用，并且你的认证服务把令牌放入“非法列表”。然而，所有的 API 服务消费 JWT，现在需要为他们的 JWT 验证增加一个额外的步骤去检查中心化的“禁用列表”。这又引入了中心状态，并且带我们回到了使用 JWTs 之前的情况。

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220212203116.png)

**禁用列表抵消了 JWT 不需要任何中心存储的好处，不是吗？**

在某种方式，他确实如此。如果你担心令牌被窃取和滥用，它是你能采取的可选措施，但是它也增加了必须核查的验证的数量。如你所想，这在网络上引起了很多[不满](https://stackoverflow.com/questions/21978658/invalidating-json-web-tokens/52407314#52407314)。

**如果我在不同的 选项卡 登录将发生什么？**

解决该问题的一个方式是在 localstorage 中引入一个全局事件监听。一旦我们在某个选项卡 的 localstorage 中更新注销关键字，在其它 选项卡的监听者将启动,并触发“注销”，同时重定向用户到登录界面。

```js
window.addEventListener('storage', this.syncLogout)

//....

syncLogout (event) {
  if (event.key === 'logout') {
    console.log('logged out from storage!')
    Router.push('/login')
  }
}
```

以下是我们现在需要为注销做的 2 件事：

1. 取消令牌
2. 在本地存储中设置`注销`条目

```js
import { useEffect } from "react";
import { useRouter } from "next/router";
import { gql, useMutation, useApolloClient } from "@apollo/client";
import { setJwtToken, setRefreshToken } from "../lib/auth";

const SignOutMutation = gql`
  mutation SignOutMutation {
    signout {
      ok
    }
  }
`;

function SignOut() {
  const client = useApolloClient();
  const router = useRouter();
  const [signOut] = useMutation(SignOutMutation);

  useEffect(() => {
    // Clear the JWT and refresh token so that Apollo doesn't try to use them
    setJwtToken("");
    setRefreshToken("");
    // Tell Apollo to reset the store
    // Finally, redirect the user to the home page
    signOut().then(() => {
      // to support logging out from all windows
      window.localStorage.setItem("logout", Date.now());
      client.resetStore().then(() => {
        router.push("/signin");
      });
    });
  }, [signOut, router, client]);

  return <p>Signing out...</p>;
}
```

在这种情况下，当你从一个选项卡注销时，事件监听者将触发所有其他选项卡，并且重定向他们到登录界面。

**这可以跨选项卡工作。但是，我如何“强制注销”不同设备上的所有会话？**

我们在后面的章节：强制注销，中讨论这个主题的更多细节。

# 静默刷新

基于应用的 JWT 用户仍然将面对 2 个主要问题：

1. 在 JWTs 上设置更短的过期时间，用户将每 15 分钟注销一次。这是相当糟糕的体验。理想做法，我们或许想让用户长时间登录。
2. 如果用户关闭应用并再次打开它，他们需要再一次登录。他们的会话没有持久化，因为我们没有在客户端的任何地方保存 JWT 令牌。

为了解决这个问题，很多 JWT 提供商，提供一个刷新令牌。刷新令牌有 2 个属性：

1. 在前一个 JWT 过期前，它被用于发起 API 调用（`/refresh_token`）去获取一个新的 JWT 令牌。
2. 它可以安全的在客户端跨会话持久化。

**刷新令牌如何工作？**

令牌作为身份验证处理的一部分和 JWT 一起签发。验证服务应该保存刷新令牌并且把它关联到数据库中的一个特定用户上，以便它能处理更新 JWT 逻辑。

在客户端，前一个 JWT 令牌过期前，我们连接我们的应用，去创建`/refresh_token`端点，并且获取一个新的 JWT。

**刷新令牌如何在客户端安全的持久化？**

同 JWT 令牌一样。

**所以，新的“登录”流程懒起来像什么？**

除了刷新令牌随着 JWT 一起发送之外，没有什么变化。我们再看一下登录处理流程图，但是，现在加上了`refresh_token`功能。

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220212231755.png)

1. 用户使用登录 API 调用登录
2. 服务产生 JWT 令牌和`刷新令牌`以及`指纹`
3. 服务返回 JWT 令牌和刷新令牌以及在令牌声明中的指纹 `SHA256` 哈希版本
4. 生成指纹的非哈希版本被存储在客户端上加固的 `HttpOnly` cookie 中
5. 当 JWT 令牌过期时，静默刷新将发生。这是客户端调用 `/refresh` 令牌端点的时机。

**现在，静默刷新看起来像什么？**

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220212232827.png)

发生了什么：

1. 刷新端点必须核查指纹 cookie 的存在，并且验证令牌声明中的哈希值与 cookie 中的非哈希值对比完全相同。
2. 如果这两个条件没有被满足，刷新请求被拒绝。
3. 否则，刷新令牌被接受，并且授予一个新的 JWT 访问令牌，重置静默刷新过程。

使用 `apollo-link-token-refresh` 包实现该工作流的过程如下所示。

使用它作为非终止链接将自动核查 JWT 的有效性，并且当运行任何操作是，如果需要的话，尝试静默刷新。

```js
import { TokenRefreshLink } from "apollo-link-token-refresh";
import { JwtPayload } from "jwt-decode";
import { getJwtToken, getRefreshToken, setJwtToken } from "./auth";
import decodeJWT from "jwt-decode";

export function makeTokenRefreshLink() {
  return new TokenRefreshLink({
    // Indicates the current state of access token expiration
    // If token not yet expired or user doesn't have a token (guest) true should be returned
    isTokenValidOrUndefined: () => {
      const token = getJwtToken();

      // If there is no token, the user is not logged in
      // We return true here, because there is no need to refresh the token
      if (!token) return true;

      // Otherwise, we check if the token is expired
      const claims: JwtPayload = decodeJWT(token);
      const expirationTimeInSeconds = claims.exp * 1000;
      const now = new Date();
      const isValid = expirationTimeInSeconds >= now.getTime();

      // Return true if the token is still valid, otherwise false and trigger a token refresh
      return isValid;
    },
    // Responsible for fetching refresh token
    fetchAccessToken: async () => {
      const jwt = decodeJWT(getJwtToken());
      const refreshToken = getRefreshToken();
      const fingerprintHash =
        jwt?.["https://hasura.io/jwt/claims"]?.["X-User-Fingerprint"];

      const request = await fetch(process.env["NEXT_PUBLIC_HASURA_ENDPOINT"], {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          query: `
                  query RefreshJwtToken($refreshToken: String!, $fingerprintHash: String!) {
                    refreshJwtToken(refreshToken: $refreshToken, fingerprintHash: $fingerprintHash) {
                      jwt
                    }
                  }
                `,
          variables: {
            refreshToken,
            fingerprintHash,
          },
        }),
      });

      return request.json();
    },
    // Callback which receives a fresh token from Response.
    // From here we can save token to the storage
    handleFetch: (accessToken) => {
      setJwtToken(accessToken);
    },
    handleResponse: (operation, accessTokenField) => (response) => {
      // here you can parse response, handle errors, prepare returned token to
      // further operations
      // returned object should be like this:
      // {
      //    access_token: 'token string here'
      // }
      return { access_token: response.refreshToken.jwt };
    },
    handleError: (err) => {
      console.warn("Your refresh token is invalid. Try to reauthenticate.");
      console.error(err);
      // Remove invalid tokens
      localStorage.removeItem("jwt");
      localStorage.removeItem("refreshToken");
    },
  });
}
```

回到本节，地址：“如果我在多个选项卡登录，将发生什么?”，使用 `sessionStorage` 来实现意味着，我们不能在新的选项卡或窗口授权。

在仍然安全的情况下，潜在的解决方案是再一次使用 `localStorage`作为一个事件触发，并且在加载相同基础 URL 选项卡之间同步 `sessionStorage`。

这可以通过使用如下的脚本实现：

```js
if (!sessionStorage.length) {
  // Ask other tabs for session storage
  localStorage.setItem("getSessionStorage", String(Date.now()));
}

window.addEventListener("storage", (event) => {
  if (event.key == "getSessionStorage") {
    // Some tab asked for the sessionStorage -> send it
    localStorage.setItem("sessionStorage", JSON.stringify(sessionStorage));
    localStorage.removeItem("sessionStorage");
  } else if (event.key == "sessionStorage" && !sessionStorage.length) {
    // sessionStorage is empty -> fill it
    const data = JSON.parse(event.newValue);
    for (let key in data) {
      sessionStorage.setItem(key, data[key]);
    }
  }
});
```

# 会话持久化

会话持久化违反了 OWASP 的客户端和令牌身份验证的安全指导规则：

> https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.md#symptom-4

创作本文时，没有一种方法可以在浏览器关闭并重新打开后允许用户会话持久化，除非浏览器实现保留选项卡 session 状态（`sessionStorage`）。

为了跨浏览器重启持久化会话，你或许选择在 `localStorage` 中或者 `Cookie` 存储你的令牌，但是一定要慎重。

> 有关该主题的讨论，见：https://github.com/OWASP/ASVS/issues/1141

# 强制登出，又名注销所有会话/设备

现在，用户永久登录了并且在会话之间保持登录态，出现一个新的问题，我们需要关心：强制登出或者注销所有的会话和设备

上述章节刷新令牌的实现表明，我们能持久化会话，并且保持登录。

在这种情形下，一个简单地“强制登出”实现是要求认证服务使特定用户相关的所有刷新令牌失效。

这是在认证服务后端的主要实现，并且不需要在客户端任何特别的处理。除了你应用上的“强制注销”按钮。

# 服务端渲染（SSR）

在处理 JWT 令牌时，在服务端渲染涉及额外的复杂性。

这是我们想要的：

1. 浏览器向应用 URL 发起请求
2. SSR 服务基于用户身份渲染页面
3. 用户获得渲染页，然后继续使用应用作为单页应用（SPA）

## 如果用户登录，SSR 服务如何知道？

浏览器需要向 SSR 服务发送一些关于当前用户身份信息。做这的唯一方式是通过 cookie。

因为我们早已通过 cookies 实现了刷新令牌工作流，当我们向 SSR 服务发起请求时，我们需要确保刷新令牌也一起发送。

> **注意**：对于经过认证的 SSR 页面，认证 API 的域（以及 refresh_token cookie 的域）与 SSR 服务的域相同是至关重要的。否则，我们的 cookies 不能发送到 SSR 服务！

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220213002655.png)

这是 SSR 服务做的：

1. 在收到渲染特定页面的请求时，SSR 服务捕获 refresh_token cookie
2. SSR 服务使用 refresh_token cookie 为用户获取一个新的 JWT
3. SSR 服务使用新的 JWT 令牌，发起所有授权的 GraphQL 请求去获取正确的数据

**一旦 SSR 页面加载完毕后，用户能继续发起授权请求吗？**

不，不幸的是，如果没有额外的折腾，就不会这样!（意思是还是要额外处理）

一旦 SSR 服务返回渲染的 HTML，浏览器上仅存的关于用户身份的标识是旧的刷新令牌 cookie，它已经被 SSR 服务使用过了。

如果我们应用代码尝试使用这个刷新令牌 cookie 去获取新的 JWT，这个请求将失败并且用户将退出登录。

为了解决这个，SSR 服务渲染页面之后需要发送最新的 `refresh token cookie`，以便于浏览器使用它。

**整个 SSR 流程，首尾相连**

![](https://gitee.com/caoyanbin/picgo/raw/master/img/20220213003641.png)

# 本文的代码（已完成的应用）

本文示例代码包含端到端工作应用，以及 SSR 能力。

https://github.com/hasura/jwt-guide

该仓库也包含身份验证后端代码示例。

# 尝试它

使用 Hasura Cloud 设置免费的 GraphQL 后端，尝试它！

确保你的版本在 1.3 及之上，你可以出发了！

# 参考

- JWT.io
- OWASP notes on XSS, CSRF and similar things
  - [OWASP JWT Cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
  - [OWASP Application Security Verification Standard, v5](https://github.com/OWASP/ASVS)
- The Parts of JWT Security Nobody Talks About | Philippe De Ryck

- [Optimistic UI and Clobbering](https://hasura.io/blog/optimistic-ui-and-clobbering/)

# 总结

一旦你按照所有上述章节操作，你的应用将具有现代应用的能力，使用 JWT 并且能避免 JWT 实现过程中的主要安全缺陷。

如果你有任何问题，建议或者反馈，请在 [Twitter](https://twitter.com/HasuraHQ) 或者在下面评论区告诉我们！
