---
title: Sidecar Pattern
date: 2021-02-19
categories:
  - design-pattern
tags:
    - pattern
---

# [译]Sidecar Pattern

为了将应用的各个组件独立封装部署在不同的进程或者容器中。这个设计模式也可以是允许应用整合多种各类型的组件和技术。

这个设计模式叫*Sidecar*(跨斗模式)，因像模特车上的跨斗而得名。在跨斗设计模式中，跨斗应用依附在主应用上并为主应用提供相应的支持特性。跨斗应用与主应用共享同一个生命周期，”生死与共“。有时候又叫做搭档模式。

<details>
<summary>原文</summary>
Deploy components of an application into a separate process or container to provide isolation and encapsulation. This pattern can also enable applications to be composed of heterogeneous components and technologies.

This pattern is named Sidecar because it resembles a sidecar attached to a motorcycle. In the pattern, the sidecar is attached to a parent application and provides supporting features for the application. The sidecar also shares the same lifecycle as the parent application, being created and retired alongside the parent. The sidecar pattern is sometimes referred to as the sidekick pattern and is a decomposition pattern.
</details>

# 背景与问题

应用和服务通常需要使用相关的功能，例如：监控、日志、配置信息、网络服务。这些外部的功能都可以分为一个单独扽组件或者服务。

如果这些服务或组件都耦合在一起，他们就能运行在同一个进程或者应用中，能更好的共享资源。然而这也意味着他们不够对立，一个组件或者服务的崩溃会导致整个应用的崩溃，而且它们也需要使用同一种编程语言来编写，这样就导致了应用的高耦合。

如果应用被分为了多个服务，每个服务之间就可以使用不同的编程语言与技术框架。尽管这样做提供了很大的灵活性，但是也意味着每个组件都需要管理自己的依赖和对应的系统环境，以及处理组件资源占用问题。除此之外分布式服务还会增加系统应用之间的延迟，增加各种不同语言代码和依赖维护的复杂性，增加运维成本。

<details>
<summary>原文</summary>
Applications and services often require related functionality, such as monitoring, logging, configuration, and networking services. These peripheral tasks can be implemented as separate components or services.

If they are tightly integrated into the application, they can run in the same process as the application, making efficient use of shared resources. However, this also means they are not well isolated, and an outage in one of these components can affect other components or the entire application. Also, they usually need to be implemented using the same language as the parent application. As a result, the component and the application have close interdependence on each other.

If the application is decomposed into services, then each service can be built using different languages and technologies. While this gives more flexibility, it means that each component has its own dependencies and requires language-specific libraries to access the underlying platform and any resources shared with the parent application. In addition, deploying these features as separate services can add latency to the application. Managing the code and dependencies for these language-specific interfaces can also add considerable complexity, especially for hosting, deployment, and management.
</details>

# 解决

把核心应用和跨斗应用分割为两个部分，在跨斗应用中又将子组件独立部署在进程或容器中，通过跨斗应用通过统一的接口来起到跨语言交互。

![](https://docs.microsoft.com/en-us/azure/architecture/patterns/_images/sidecar.png)

跨斗应用不是主应用的核心部分，但是需要链接到主应用上。就是说跨斗应用不会影响到主应用的运行，但是主应用的运行会影响到跨斗应用。

跨斗模式优点：

 - 辅助工具在运行时环境和编程语言方面与其主要应用程序无关，因此您无需为每种语言开发一个辅助工具。

 - 共享系统资源，跨斗应用可以监控自身和核心应用

 - 低延迟，因为它们在同一台服务器上

 - 即使对于不提供扩展机制的应用程序，您也可以使用sidecar来扩展功能，方法是将其作为自己的进程附加到与主应用程序相同的主机或子容器中。


跨斗模式通常与容器一起使用。

<details>
<summary>原文</summary>
Co-locate a cohesive set of tasks with the primary application, but place them inside their own process or container, providing a homogeneous interface for platform services across languages.

A sidecar service is not necessarily part of the application, but is connected to it. It goes wherever the parent application goes. Sidecars are supporting processes or services that are deployed with the primary application. On a motorcycle, the sidecar is attached to one motorcycle, and each motorcycle can have its own sidecar. In the same way, a sidecar service shares the fate of its parent application. For each instance of the application, an instance of the sidecar is deployed and hosted alongside it.

Advantages of using a sidecar pattern include:

A sidecar is independent from its primary application in terms of runtime environment and programming language, so you don't need to develop one sidecar per language.

The sidecar can access the same resources as the primary application. For example, a sidecar can monitor system resources used by both the sidecar and the primary application.

Because of its proximity to the primary application, there’s no significant latency when communicating between them.

Even for applications that don’t provide an extensibility mechanism, you can use a sidecar to extend functionality by attaching it as its own process in the same host or sub-container as the primary application.

The sidecar pattern is often used with containers and referred to as a sidecar container or sidekick container.
</details>

# 问题与注意事项

 - 需要考虑部署和打包方式，最好是在容器中使用该模式。

 - 考虑跨斗应用之间的通讯机制。

 - 跨斗应用与传统应用是否适合当前服务

 - 还考虑是否可以将功能实现为库或使用传统的扩展机制。 特定语言的库可能具有更深层次的集成和更少的网络开销。
<details>
<summary>原文</summary>
Consider the deployment and packaging format you will use to deploy services, processes, or containers. Containers are particularly well suited to the sidecar pattern.

When designing a sidecar service, carefully decide on the interprocess communication mechanism. Try to use language- or framework-agnostic technologies unless performance requirements make that impractical.

Before putting functionality into a sidecar, consider whether it would work better as a separate service or a more traditional daemon.

Also consider whether the functionality could be implemented as a library or using a traditional extension mechanism. Language-specific libraries may have a deeper level of integration and less network overhead.
</details>

# 何时使用

 - 您的主应用程序使用不同种类的语言和框架。 位于sidecar服务中的组件可由使用不同框架以不同语言编写的应用程序使用。

 - 组件由远程团队或其他组织拥有。

 - 组件或功能必须与应用程序位于同一主机上

 - 您需要与主应用程序共享整个生命周期但可以独立更新的服务。

 - 您需要对特定资源或组件的资源限制进行细粒度的控制。 例如，您可能想限制特定组件使用的内存量。 您可以将组件部署为辅助工具，并独立于主应用程序管理内存使用情况。

# 不适合的情况

 - 需要优化进程间通信时。 父应用程序和Sidecar服务之间的通信包括一些开销，特别是调用中的延迟。 对于聊天之类扽接口，这可能不是可接受的折衷方案。

 - 不适合小规模的应用
<details>
<summary></summary>
Use this pattern when:

Your primary application uses a heterogeneous set of languages and frameworks. A component located in a sidecar service can be consumed by applications written in different languages using different frameworks.
A component is owned by a remote team or a different organization.
A component or feature must be co-located on the same host as the application
You need a service that shares the overall lifecycle of your main application, but can be independently updated.
You need fine-grained control over resource limits for a particular resource or component. For example, you may want to restrict the amount of memory a specific component uses. You can deploy the component as a sidecar and manage memory usage independently of the main application.
This pattern may not be suitable:

When interprocess communication needs to be optimized. Communication between a parent application and sidecar services includes some overhead, notably latency in the calls. This may not be an acceptable trade-off for chatty interfaces.
For small applications where the resource cost of deploying a sidecar service for each instance is not worth the advantage of isolation.
When the service needs to scale differently than or independently from the main applications. If so, it may be better to deploy the feature as a separate service.
</details>