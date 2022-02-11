---
title: "006【翻译】Serverless开源框架：OpenFaaS,Knative等"
date: 2022-02-10T21:30:31+08:00
tags: ["技术翻译", "Serverless"]
categories: ["研发提效"]
draft: false
---

https://www.cncf.io/blog/2020/04/13/serverless-open-source-frameworks-openfaas-knative-more/

本文将讨论上面提及的一些框架，并且深入 OpenFaaS 和 Knative 去介绍他们的架构、主要的组件以及基本安装步骤。如果你对这个主题感兴趣，并且计划使用开源平台开发 serverless 应用，本文将让你更好的理解这些解决方案。

过去几年里，serverless 已经迅速流行起来。这个技术的主要优势是能够创建和运行应用程序，但不需要基础架构管理。换句话说，当使用 serverless 架构时，开发者不再需要分配资源、扩展或维护服务器就可以运行应用，或者管理数据库和存储系统。他们唯一的职责就是去写高质量的代码。

有很多建立 serverless 框架的开源项目（Apache OpenWhisk, IronFunctions, Fn from Oracle, OpenFaaS, Kubeless, Knative, Project Riff 等）。此外，由于开源平台提供了 IT 创新的途径，很多开发者对开源解决方案产生兴趣。

# OpenWhisk, Firecracker & Oracle FN

深入研究 OpenFaaS 和 Knative 之前，让我们简单的介绍下这三个平台。

Apache OpenWhisk 是用于 serverless 计算的开放云计算平台，它把云计算资源作为服务。与其他开源项目（Fission, Kubeless, IronFunctions）对比，A[pache OpenWhisk](https://epsagon.com/blog/epsagon-makes-troubleshooting-apache-openwhisk-a-snap/) 有如下特征：庞大的代码库、高质量特性和大量贡献者。然而，该平台过于庞大的工具（CouchDB, Kafka, Nginx, Redis 和 Zookeeper）为开发者带来了困难。此外，该平台在安全性方面不太完美。

Firecracker 是由 Amazon 推出的一个虚拟化技术。这个技术提供最小开销的虚拟机，允许创建和管理隔离的环境和服务。Firecracker 提供叫做微 VMs 的轻量虚拟机，它使用基于硬件的虚拟化技术实现完全隔离，同时提供传统容器级别的性能和灵活性。对开发者的不便之处是，该技术的开发完全采用 Rust 语言。还使用了一个删减的软件环境，其中包含最小的组件集。为了节省内存，减少启动时间同时提高环境安全性，一个修改过的 Linux 内核被启动，其他所有多余的部分都被排除在外。此外，功能和设备支持也减少了。该平台由 Amazon Web Services 开发，旨在提高 AWS Lambda 和 AWS Fargate 平台的性能和效率。

Oracle Fn 是一个开放服务的 serverless 平台，为云系统提供附加抽象层，允许函数即服务（FaaS）。与 Oracle Fn 中其他开放平台一样，开发者在独立的函数中实现逻辑。不像现存的商业 FaaS 平台，例如 Amazon AWS Lambda, Google Cloud Functions, 和 Microsoft Azure Functions，Oracle 的解决方案定位是无供应商锁定。用户能选择任何云解决方案供应商启动 Fn 基础架构，组合不同的云系统，或者在他们自己的设备上运行平台。

Kubeless 是一个基础架构，支持在集群中部署 serverless 函数，同时能够在 Python, Node.js 或 Ruby 代码中执行 HTTP 和事件触发。Kubeless 是一个使用 [Kubernetes](https://epsagon.com/blog/development/how-to-guide-debugging-a-kubernetes-application/) 核心功能，例如部署、服务、配置卡等，构建的平台。
这节省了 Kubeless 的基础代码，而且代码尺寸很小，这也意味着开发者不必重放大部分任务调度逻辑代码，它包含在 Kubernetes 内核中。

Fission 是一个开源平台，在 Kubernetes 上提供 serverless 架构。Fission 优势之一是它负责 Kubernetes 中自动扩展资源的大部分任务，把你从手动资源管理中释放出来。Fission 的第二个优势是你不受单一供应商的束缚，可以自由的从一个换到另一个，前提是他们支持 Kubernetes 集群（和你的程序或许有的任何其他特别的需求）。

# 使用 OpenFaaS 和 Knative 的主要优点

OpenFaaS 和 Knative 公开可用且免费开源环境，能够创建和托管 serverless 函数。这些平台允许你：

- 减少闲置资源
- 快速处理数据
- 对接第三方服务
- 通过对大量请求的深加工来负载均衡

然而，尽管这两种平台和 serverless 计算在总体上都有优势，开发者必须在开始实现前评估应用的逻辑。这意味着，你必须首先拆分逻辑为独立的任务，并且仅仅这样你才能写代码。

为清晰起见，让我们分别考虑这些开源 serverless 解决方案。

# 如何使用 OpenFaaS 构建和部署 Serverless 函数

OpenFaaS 的主要目标是使用 Docker 容器简化 serverless 函数，允许你运行复杂和灵活的基础设施。

## OpenFaas 设计和架构

OpenFaas 架构基于云原生标准，并且包含下列组件： API Gateway（API 网关）, Function Watchdog, and the container orchestrators Kubernetes, Docker Swarm, Prometheus, 和 Docker。根据如下所示的架构，当开发者使用 OpenFaas 时，流程开始于安装 Docker，结束于 Gateway API。

![OpenFaaS Components and Process](https://www.cncf.io/wp-content/uploads/2020/04/pasted-image-0.png)

## API Gateway（API 网关）

通过 API 网关，提供了到所有函数位置的路由，同时通过 [Prometheus](https://www.prometheus.wang/) 监控云原生指标。

![Client to Functions Routing](https://www.cncf.io/wp-content/uploads/2020/04/graf2.jpg)

## Function Watchdog

Watchdog 组件与每个容器集成去支持 serverless 应用，并且提供一个介于用户和函数之间的通用接口。

![OpenFaaS Watchdog Interface](https://www.cncf.io/wp-content/uploads/2020/08/graf3.jpg)

Watchdog 的主要功能之一是组织在 API Gateway 接收的 HTTP 请求，并且调用选定的应用。

## Prometheus（监控程序）

这个组件允许你随时获取指标变化的动态，和其他指标对比，转换他们，在不离开网络界面主页的情况下，用文本格式或图表形式可视化他们。Prometheus 在 RAM 中存储收集的度量指标，并且在到达给定大小限制或一段时间后，把他们存储在磁盘上。

## Docker Swarm 和 Kubernetes

Docker Swarm 和 Kubernetes 是编排引擎。诸如 API Gateway, Function Watchdog, 和 Prometheus 实例工作在这些编排器之上。建议使用 Kubernetes 去开发产品，而 Docker Swarm 更适合创建本地函数。

再者，所有开发的函数、微服务和产品被存储在 Docker 容器中，它充当主要的 OpenFaaS 平台，使用容器为开发者和系统管理员提供开发、部署和运行 serverless 应用。

# 在 Docker 中安装 OpenFaaS 的要点

OpenFaaS API Gateway 依赖所选 Docker 编排工具内建的函数。为了实现它，API Gateway 为所选的编排工具连接合适的插件，在 Prometheus 中记录各种指标，并且基于从 Prometheus 通过 AlertManager 收到的警告扩充函数。

例如，假如你正在用 Linux 系统工作，想去使用 OpenFaaS 在 Docker 集群节点上写一个简单的函数。为了实现它，你需要跟着下列的步骤：

- 安装 Docker CE 17.05 或者更新的版本
- 运行 Docker

```sh
$ docker run hello-world
```

- 初始化 Docker Swarm

```sh
$ docker swarm init
```

- 从 Github 克隆 OpenFaaS

```sh
git clone https://github.com/openfaas/faas && \  cd faas && \ ./deploy_stack.sh
```

- 登录 UI 门户： http://127.0.0.1:8080

Docker 现在已经准备好使用了，当你进一步写函数时，不需要再安装它。

## 为构建函数准备 CLI OpenFaaS

为开发函数，你需要使用脚本安装最新的命令行工具。对于 brew，它是 `$ brew install faas-cli`。对于 curl，你需要使用`$ curl -sL https://cli.get-faas.com/ | sudo sh`。

## 使用 OpenFaas 的不同编程语言

使用 CLI 中的模板创建和部署一个 OpenFaas 函数，你几乎可以用任何编程语言写一个处理器（handler）。例如：

- 创建新的函数：

```sh
$ faas-cli new --lang prog language <<function name>>
```

- 生成堆栈文件和文件夹：

```sh
$ git clone https://github.com/openfaas/faas \
 cd faas \
 git checkout 0.6.5 \
 ./deploy_stack.sh
```

- 建立函数

```sh
$$ faas-cli build -f <<stack file>>
Deploy the function:
$ faas-cli deploy -f <<stack file>>
```

## 在 OpenFaaS UI 中测试函数

从 OpenFaas 的用户界面，你可以用几种方式快速的测试函数，如下所示：

- 前往 OpenFaaS UI：http://127.0.0.1:8080/ui/

- 使用 curl：

  ```sh
  $ curl -d "10" http://localhost:8080/function/fib
  ```

- 使用 UI

初一看，每件事情似乎都很简单。然而，你仍然必须处理很多细微的差别。如果你必须用 Kubernetes，需要很多函数，或者需要为 FaaS 主代码库增加额外的依赖，这更是如此了。

这个一个在 GitHub 上完整的 OpenFaaS [开发者社区](https://github.com/openfaas/faas/tree/master/sample-functions)，你也能找到很多有用的信息。

# OpenFaaS 的优缺点

OpenFaaS 简化了系统的构建。修复错误变得更加容易，并且为系统增加新的功能也比单体应用情况下更快。换句话说，OpenFaaS 允许你随时随地用任何编程语言运行代码。

然而，这儿是一些缺点：

- 对一些编程语言冷启动时间长
- 容器启动时间依赖于供应商
- 函数有限的生命周期，意味着不是所有的系统都能根据 Serverless 工作。（当使用 OpenFaaS 时，计算容器不能在内存中长时间存储可执行的应用代码。平台将自动创建和销毁他们，所以无状态是不可能的。）

# 使用 Knative 部署和运行函数

Knative 允许你开发和部署基于容器的服务端应用，你可以容易的在云服务提供商之间迁移。Knative 是刚开始受欢迎的开源平台，但是现在引起了开发者很大的兴趣。

## Knative 的架构和组件

Knative 架构包含 Building, Eventing, 和 Serving 组件。

![Knative Architecture and Components](https://www.cncf.io/wp-content/uploads/2020/04/graf8.jpg)

### 构建(Building)

Knative Building 组件的职责是确保在集群中容器程序集从源代码中启动。该组件以现存的 Kubernetes 原语为基础，并对其进行扩展。

### 事件(Eventing)

Knative Eventing 组件负责通用的订阅、投递和事件管理，以及在松耦合架构组件之间创建通讯。此外，该组件允许你扩展服务器上负载。

### 服务(Serving)

Serving 组件的主要目标是支持 serverless 应用和特性的开发，自动缩放，Istio 组件的路由和网络编程，以及部署代码和配置的快照。Knative 使用 Kubernetes 作为编排器，Istio 执行查询路由和高级负载均衡的功能。

# 使用 Knative 最简单函数的例子

你可以在 Knative 上使用一些方法创建一个后端应用。你的选择依赖于你的特定技能和使用多种服务的经验，包括 Istio, Gloo, Ambassador, Google, 尤其是 Kubernetes Engine, IBM Cloud, Microsoft Azure Kubernetes Service, Minikube, 和 Gardener。

简单为每个 Knative 组件选择安装文件。以下是三个所需组件的主要安装文件的链接：

- Serving Component：[链接](https://github.com/knative/serving/releases/download/v0.7.0/serving.yamlhttps://github.com/knative/serving/releases/download/v0.7.0/monitoring.yaml)
- Building Component：[链接](https://github.com/knative/build/releases/download/v0.7.0/build.yaml)
- Eventing Component：[链接](https://github.com/knative/eventing/releases/download/v0.7.0/release.yamlhttps://github.com/knative/eventing/releases/download/v0.7.0/eventing.yaml)

这些组件的每一个都由一系列对象构成。关于语法和安装组件的更多细节信息能在[Knative 开发站点](https://knative.dev/docs/)上找到。

# Knative 的优缺点

Knative 有大量的优势。和 OpenFaaS 一样，Knative 允许你使用容器创建 serverless 环境。这反过来允许你获得一个本地的基于事件的架构，其中没有来自公共云服务施加的限制。Knative 也能自动化容器组装进程，从而提供自动伸缩。因此，serverless 函数的能力是基于预定义阈值和事件处理机制。

此外，Knative 允许你在内部、云上或者第三方数据中心创建应用。这意味着，你不必被任何云供应商束缚。并且由于它的操作基于 Kubernetes 和 Istio，Knative 也有一个更高的采用率和更大采用潜能。

Knative 一个主要的缺点是需要独立管理容器架构。简而言之，Knative 不是针对终端用户的。然而，因此，更多的商业化管理的 Knative 服务正开始出现，例如 Google Kubernetes Engine 和 IBM Cloud Kubernetes Service 的 Managed Knative 。

# 结论

尽管开源 serverless 平台越来越多，OpenFaaS 和 Knative 将继续获得开发者的欢迎。值得注意的是，这些平台不容易比较，因为他们为了不同的任务而设计。

尽管 OpenFaas, Knative 不是一个完全成熟的 serverless 平台，但是它是更合适作为一个创建、部署和管理 serverless 工作负载的平台。然而，从配置和维护的角度，OpenFaas 是更简单的。使用 OpenFaas，不需要像 Knative 那样单独安装所有的组件，并且如果需要的组件早已被安装了，你不需要清除之前的设置和新开发的资源。

还是，如上面所提到的，OpenFaaS 的一个显著缺点是容器启动时间依赖提供者，而 Knative 不受制于任何单独的云解决方案提供商。基于上面的优缺点，团队或许也可以选择同时使用 Knative 和 OpenFaaS 来有效实现不同目标。
