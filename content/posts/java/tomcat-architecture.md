---
title: "Tomcat Architechure"
date: 2022-01-24T23:00:06+08:00
draft: true
toc: true
tags: 
  - java
---

基于 Java 语言的轻量级**应用服务器**，完全开源免费的 Servlet 容器实现，最初由 sun 公司开发（JavaWebServer），作为 Servlet 容器的参考实现，1999 年与 JServe （Apache）项目合并为 Tomcat，以 Apache License 许可发布。

> 微服务架构 https://martinfowler.com/articles/microservices.html

> 十二要素应用（构建 SAAS APP 方法论）https://12factor.net/

> JPDA （Java Platform Debugger Architecture） https://docs.oracle.com/javase/8/docs/technotes/guides/jpda/architecture.html

## 8.5 feat

- Servlet 3.1，JSP2.3，EL3.0，WebSocket1.1，Servlet4.0(9.0)
- Servlet4Preview
- 8.0 默认 HTTP、AJP 链接器采用 NIO，8.5 移除了 BIO 的支持
- 新的资源实现，采用单独、一致的方法配置 Web 应用的附加资源，可以用于实现覆盖，eg: 将一个 WAR 作为多个 Web 应用的基础，同时 Web 应用拥有自己的定制功能。
- 8.0 链接器新增支持 JDK7 的 NIO2，HTTP/2 
- 默认采用异步日志处理

## 设计

### 1.0

1. 接受、解析请求，处理，返回响应
2. socket  监听指定端口



### 2.0

1. **Connector，Container 将请求监听和请求处理分离**，应对多种网络协议（HTTP，AJP）
2. Connector 负责开启 socket 监听客户端请求，返回响应数据
3. Container 负责具体的请求处理
5. 二者分别拥有自己的 start()，stop() 方法来释放维护的资源
5. **一个 server 可以包含多个 Connector，Container**

![tomcat-server](https://s2.loli.net/2022/01/25/lMJSHQafgvuox6E.png)

### 3.0

一个 server 包含多个 service （互相独立，共享一个 JVM 以及系统类库）

**一个 service 负责维护（mapping）多个 container 和 一个 container** ,这样来自 connector 的请求只能由它所属 service 维护的 container 处理。



![container-engine](https://s2.loli.net/2022/01/25/SAce4noDp8PIJHC.png)


### 4.0

Engine （整个 servlet 引擎），**负责请求的处理，不需要考虑请求链接、协议等的处理**。

**应用服务器是一个运行环境**，需要在 Engine 容器中支持管理 Web 应用，接收到 Connector 的处理请求时，Engine 容器能够找到一个合适的 Web 应用来处理。

context 表示一个 web 应用，一个 engine 可以包含多个 context。

![container-context](https://s2.loli.net/2022/01/25/Ma1QPcoVsxLWmSq.png)


每个组件通过 start()，stop() 方法在启动时加载资源和停止时释放资源，使得组件充分解耦，提高服务器的可拓展性和可维护性。

### 5.0

如果**一个服务实例要对多个域名提供服务**，可以将每个域名视为一个**虚拟**的主机，每个虚拟主机下包含多个 web 应用。

Host 表示虚拟主机，一个 Host 可以包含多个 context。

![container-host](https://s2.loli.net/2022/01/25/8IM9kb4Hcd5uTwj.png)

### 6.0

Servlet 规范中一个 Web 应用中可以包含多个 Servlet 实例处理不同链接的请求，需要一个组件的概念来表示 Servlet 定义，Tomcat 中 Servlet 定义被称为 Wrapper。

![container-wrapper](https://s2.loli.net/2022/01/25/WaZrNIB4FT9A3HV.png)



### 7.0

Engine，context 组件的作用就是处理接受客户端的请求并返回数据，具体操作可能会委派到子组件完成。

使用 container 表示容器，container 可以添加并维护子容器，Engine，Host，Context，Wrapper 均继承自 Container。

嵌入式方式启动 Tomcat ,运行简单的请求处理，不必支持多 Web 场景，可以只在 Service 中维护一个简化版的 Engine。

默认实现。

![tomcat-container](https://s2.loli.net/2022/01/25/FNUAHyYm4nL1Jci.png)


### 后台处理

很多情况下，container 需要执行一些异步处理，而且是定期执行（web 文件变更的扫描），Tomcat 针对后台处理，在 Container 上定义了 backgroundProcess() 方法，其基础抽象类（ContainerBase）确保子啊启动组件的同时，异步启动后台处理，大多数情况，各个容器组件仅需要实现 Container 的 backgound-Process() 方法即可，不必考虑创建异步线程。

## Lifecycle
