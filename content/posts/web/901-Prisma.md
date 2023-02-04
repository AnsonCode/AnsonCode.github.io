---
title: "901-Prisma 生成器的原理"
date: 2022-09-07T10:22:03+08:00
tags: ["Prisma"]
categories: ["WEB"]
featured_image:
description: "Prisma 生成器的原理"
draft: false
plantuml: false
---

翻译：https://prismaio.notion.site/Prisma-Generators-a2cdf262207a4e9dbcd0e362dfac8dc0

在 Prisma，我们有一个叫做“生成器（Generator）”的概念。生成器是一个执行程序，它把解析后的 Prisma schema 作为输入，并且可以输出任何东西。

最著名的生成器叫做`prisma-client-js`。它是一个 ORM 客户端，支持 Node.js 中的 TypeScript 和 JavaScript。

然而，它不是唯一的生成器。因为生成器接口是开放的并且语言无关，任何人都能在他们的 Prisma 项目中接入他们自己的生成器，它能完成很棒的事情。

一些其他生成器的示例：

- [Docs generator](https://github.com/pantharshit00/prisma-docs-generator)：用于生成 prisma 文档
- [DBML generator](https://github.com/notiz-dev/prisma-dbml-generator)：用于生成数据库关系图
- [Nestjs Service generator](https://www.npmjs.com/package/@trackyourhealth/nestjs-restful-generator)
- [TypeGraphQL generator](https://github.com/MichalLytek/typegraphql-prisma)：看了没看太懂~大概意思是生成 TypeScript 类型
- [Prisma Go Client generator](https://github.com/prisma/prisma-client-go)：基于 golang 的生成器

生成器将总是当你运行 `prisma generate` 时调用。然而，只有在 `schema.prisma` 文件中共提及的生成器会运行。本文档描述了，生成器架构的样子，以及如何构建自己的生成器。

# Schema

在 schema 中，有一个所谓的 `generator` 块：

```jsx
// Prisma 客户端 TypeScript 生成器
generator mygen {
  provider = "prisma-client-js"
}

// Prisma 客户端 文档生成器
generator gen {
  provider = "node node_modules/prisma-docs-generator"
}

// 官方 Prisma 客户端 Go 生成器
generator db {
  provider = "go run github.com/prisma/prisma-client-go"
}
```

`provider` 也可以是一个命令，例如 `node myscript.js` 或者 预先定义的别名 `prisma-client-js`， `prisma` 命令行界面预封装了它。

# CLI

`prisma generate` 是连接一切的关键。无论何时用户更新项目的 `schema.prisma` 文件，他们运行 `prisma generate` 在新生成的生成器构建中反射更新过的数据模型，比如生成的 TypeScript 客户端。

它这样工作：

1. 解析数据模型，获得所有生成器
2. 为每个生成器生成子进程
3. 从每个生成器获得所谓的 `生成器自述 generator manifest`。它包含信息，该生成器需要哪个引擎和版本。
4. 下载可能丢失的引擎（？）
5. 在每个生成器：调用`generate`，给它传入元信息，例如解析后的数据模型 AST（抽象语法树） 或者引擎在文件系统中的位置

# 架构

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cdac9c4d-93e6-4480-ac46-c2f2e0217803/Blank_diagram.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cdac9c4d-93e6-4480-ac46-c2f2e0217803/Blank_diagram.png)

[https://lucid.app/lucidchart/invitations/accept/d71c6bbf-5bdf-4f5f-b7c9-d9c2ae64d17d](https://lucid.app/lucidchart/invitations/accept/d71c6bbf-5bdf-4f5f-b7c9-d9c2ae64d17d)

# 生成器接口

生成器必须是文件系统中某处的可执行文件。该执行文件，例如 `./mygenerator` ，需要通过 标准输入输出 实现 JSON RPC 接口。

生成器自身可以消费它从 SDK 中通过`stdin`接收的信息。

如果生成器想向 SDK”回复“一个消息，他可以通过`stderr`。

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea923596-fbae-4324-93f4-6fa584b87e4c/Blank_diagram_(1).png](<https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea923596-fbae-4324-93f4-6fa584b87e4c/Blank_diagram_(1).png>)

[https://lucid.app/lucidchart/invitations/accept/d0a7724d-86cf-4060-b474-a7279025982c](https://lucid.app/lucidchart/invitations/accept/d0a7724d-86cf-4060-b474-a7279025982c)

一个消息总是一行 JSON RPC 对象，以换行符 (`\n` )结束。

从生成器返回给 SDK 的回复或信息通过 `stderr` 发送。他们也是一个行 JSON 对象，通过换行符 (`\n` )界定。

这是一个来自生成器 SDK 的示例消息，

```jsx
{"jsonrpc":"2.0","method":"getManifest","params":{},"id":1}\n
```

它遵循[JSON RPC 2.0 规范](https://www.jsonrpc.org/specification)。注意，我们只支持了规范的子集。我们只支持单对象，没有批处理数组。

### RPC 示例流

这是`getManifest` RPC 流看起来的样子：

**Prisma SDK** 把它发送给生成器进程 `stdin`：

```tsx
{"jsonrpc":"2.0","method":"getManifest","params":{...},"id":1}\n
```

The generator process then returns this to `stderr`:

随后，生成器进程把它返回给 `stderr`:

```tsx
{"jsonrpc":"2.0","result":{"manifest":{...}},"id":1}\n
```

在 JSON RPC 中，有**方法**（这个概念）。方法基本是预先商定的 API，这是双方（SDK 和生成器）都理解的。

### 协议

These are the available methods in the current generator interface, their params and their expected answer:

在当前的生成器方法中，这是可用的方法，他们期望的参数和响应：

[生成器 RPC 协议方法](https://www.notion.so/591f19ac980a4450930f22e3e57e01fc)

Here are the TypeScript type definitions of the input and output types together with comments explaining what they're good for:

下面是输入和输出类型的 TypeScript 类型定义以及注释，解释了它们的优点:

### `生成器配置（GeneratorConfig）`

```tsx
/**
 * `GeneratorConfig` 是解析 `generator` 块的AST
 * 在数据模型中，由用户提供
 **/
export interface GeneratorConfig {
  name: string; // 生成器的名称, 例如. "mygen"
  output: string | null; // 自定义输出路径
  isCustomOutput?: boolean;
  provider: string; // 提供商, 例如. "prisma-client-js"
  config: Dictionary<string>; // 块中定义的所有其他字段都放在这里
  binaryTargets: string[]; // 自定义二进制目标
  previewFeatures: string[]; // 预览功能列表
}
```

### `GeneratorManifest`

```tsx
export type EngineType =
  | "queryEngine"
  | "libqueryEngineNapi"
  | "migrationEngine"
  | "introspectionEngine"
  | "prismaFmt";

/**
 * `GeneratorManifest` 是 当生成器被SDK请求时给的回复。
 * 它指定生成器的名称和或许有的需求。
 * 所有东西都在它的可选项中，尽管我们推荐至少提供`prettyName`，正如每个 `prisma generate` 命令展示的那样
 **/
export type GeneratorManifest = {
  // Name printed in `prisma generate`
  prettyName?: string;

  // 如果没有提供输出时，应该输出到哪里？（默认输出地址）
  defaultOutput?: string;

  // 是否有需要禁用的模型或字段名
  denylists?: {
    models?: string[];
    fields?: string[];
  };

  // 一些生成器依赖`prisma-client-js`先生成一次
  requiresGenerators?: string[];

  // 这些生成器依赖特定的引擎，例如 "queryEngine（查询引擎）"?
  requiresEngines?: EngineType[];

  // What's the version of this generator?
  // 生成器的版本？
  version?: string;

  // 如果生成器需要特定引擎，引擎版本是哪个？
  requiresEngineVersion?: string;
};
```

### `生成器配置项（GeneratorOptions）`

```tsx
/**
 * 在每个 `generate` 命令，有一些参数要传给生成器
 **/
export type GeneratorOptions = {
  // 生成器的 AST 配置，见上文
  generator: GeneratorConfig;

  // 其他生成器的AST 配置
  otherGenerators: GeneratorConfig[];

  // `schema.prisma`  文件的位置
  schemaPath: string;

  // 数据模型元格式(the Datamodel Meta Format (DMMF))的AS
  // 这个包含生成的 API Schema，以及解析的数据模型
  dmmf: DMMF.Document;

  // 在数据模型中定义的数据源列表
  datasources: DataSource[];

  // 字符串格式的数据模型
  datamodel: string;

  // 如果在自述中`requiresEngines`属性需要某些引擎，引擎的二进制在这里
  binaryPaths?: BinaryPaths;

  // 二进制文件的版本
  version: string;
};
```

最新的类型定义在[这里](https://github.com/prisma/prisma/blob/main/packages/generator-helper/src/types.ts)[.](https://github.com/prisma/prisma/blob/master/src/packages/generator-helper/src/types.ts)

# 实现

<aside>
⚠️ **WARNING** ⚠️

**The `@prisma/sdk` package and the DMMF are _not_ considered part of
the official API surface and might break unexpectedly with any release.**

</aside>

[Prisma CLI](https://github.com/prisma/prisma/tree/master/src/packages/cli) (`prisma` 的 NPM 包)，包含 `generate` 命令的实现，只是[Prisma SDK](https://github.com/prisma/prisma/tree/master/src/packages/sdk) (`@prisma/sdk`)的一层封装。

SDK 最终负责解析数据模型和运行生成器。

在 SKD 中，我们有一个叫做 `getGenerators` 的函数。该函数是 `prisma generate` 访问生成器的**入口点**。它大致是这样工作的：

```jsx
import { getGenerators } from "@prisma/sdk";

const generators = await getGenerators({
  schemaPath: "./prisma/schema.prisma",
});

for (const generator of generators) {
  console.log(`Generating ${generator.manifest.prettyName}`);
  await generator.generate();
}
```

在底层，它实例化[一堆`GeneratorProcess`](https://github.com/prisma/prisma/blob/main/packages/generator-helper/src/GeneratorProcess.ts#L27)[.](https://github.com/prisma/prisma/blob/master/src/packages/generator-helper/src/GeneratorProcess.ts#L27)

这些生成生成器二进制文件，作为一个长时间运行的进程。这个二进制现在准备通过 `stdin` 接收 RPCs 了。

在生成器端，你也能通过标准输入输出实现 JSON RPC，或者，如果你使用 JavaScript 或 TypeScript，你可以使用我们写的叫做 `@prisma/generator-helper`的帮助库。

它处理实现该接口的所有工作，并给你简单的回调，你可以在那里实现业务逻辑。这里有一个生成器帮助库如何在 Prisma Client TypeScript 中使用示例：

```tsx
import Debug from "@prisma/debug";
import { enginesVersion } from "@prisma/engines-version";
// 引入帮助库
import { generatorHandler } from "@prisma/generator-helper";
import { generateClient } from "./generation/generateClient";
import { getDMMF } from "./generation/getDMMF";
import { externalToInternalDmmf } from "./runtime/externalToInternalDmmf";
const debug = Debug("prisma:client:generator");

const pkg = require("../package.json");
const clientVersion = pkg.version;

// 在回调中写具体的逻辑
generatorHandler({
  // 回复自述调用
  onManifest(config) {
    const requiredEngine =
      config?.previewFeatures?.includes("napi") ||
      process.env.PRISMA_FORCE_NAPI === "true"
        ? "libqueryEngineNapi"
        : "queryEngine";
    debug(`requiredEngine: ${requiredEngine}`);
    return {
      defaultOutput: "@prisma/client",
      prettyName: "Prisma Client",
      requiresEngines: [requiredEngine],
      version: clientVersion,
      requiresEngineVersion: enginesVersion,
    };
  },
  // 调用生成时的逻辑
  async onGenerate(options) {
    return generateClient({
      datamodel: options.datamodel,
      datamodelPath: options.schemaPath,
      binaryPaths: options.binaryPaths!,
      datasources: options.datasources,
      outputDir: options.generator.output!,
      copyRuntime: Boolean(options.generator.config.copyRuntime),
      dmmf: options.dmmf,
      generator: options.generator,
      engineVersion: options.version,
      clientVersion,
      transpile: true,
      activeProvider: options.datasources[0]?.activeProvider,
    });
  },
});

export { getDMMF, externalToInternalDmmf };
```

然而，你可以在任何语言实现这个，只要你遵循上面提及的 JSON RPC 接口。作为 TS 实现的替代方案，你可以看下[Go 客户端](https://github.com/prisma/prisma-client-go/blob/master/generator.go)生成器接口的实现。
