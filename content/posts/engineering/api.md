---
title: "API相关设计使用"
date: 2021-10-01T00:41:08+08:00
draft: true
toc: true
images:
tags: 
  - 工程、工具
---

稍微复杂一点项目现在大部分都会无脑上微服务，微服务就少不了定义API，定义API也是一门艺术。

插入一下 URL 、 URI、URN

- **URL Uniform Resouce Locator**，**统一资源定位符**，简称网址，是因特网上标准的资源的地址-RFC1738。

  **URL 是一种 URI**

  语法可拓展，使用**美国信息交换标准代码**的一部分来表示因特网的地址，// 为层级标记符号，固定不变。

  **标准格式**

  [传送协议类型]://[服务器地址]:[端口号]/[资源层级UNIX文件路径] [文件名]?[查询]#[片段ID]

  **完整格式**

  [传送协议类型]://[访问资源需要的凭证信息]@[服务器地址]:[端口号]/[资源层级UNIX文件路径] [文件名]?[查询]#[片段ID]

  [可省略部分] 访问资源需要的凭证信息、端口号、查询、片段ID。

  协议常见的有 http、ftp、mailto、file

URL 大多数情况下可以省略传送协议，因为大部分内容都是超文本传输协议文件，超文本协议又允许服务器将浏览器重定向到另外一个网址，也可以升学网页地址中的部分,eg: www。

JDBC客户端连接，也是使用URL格式进行连接数据库服务器。

- **URI Uniform Resource Identifier**，**统一资源标志符**，标识互联网资源的**字符串**， **= URL + URN**

- **URN Uniform Resource Name**，**统一资源名称**，统一资源标识(URI) 的历史名字，使用 urn: 作为 URI Schema

  起初是聚合多个命名空间到一个命名空间，期望用一个命名空间去标示资源，达到持久且位置无关的效果，这在资源不可用的情况下是不行的。历史既能见证英雄也会记录匹夫，哈哈哈。

## Design Principles

- 资源 就是数据的子集，订单
- 集合 就是资源的集合，订单列表
- URL  标识资源或者集合的坐标，/orderDetail

## Naming Convention

1. 减少阅读和理解成本、提告清晰度、减少关注度
2. 促成内部一致性，避免内部交叉组合发生冲突
3. 增强产品的审美和专业外观
4. 提供有意义的数据
5. 长时间间隔后重用代码提供良好的支撑

## Naming Style

- param camelCase(lower) 首字母小写
- url 集合复数 eg: GET /users
- url 集合开始，标识符结束 eg: GET /orders/:orderId 保持概念单一性和一致性
- url **远离动词** eg: PUT /users/{userId} **不要使用动词表示意图，使用HTTP方法描述操作**
- **非资源URL使用动词** eg: POST /order/cancel
- JSON 属性使用 lower camelCase
- restful HTTP服务必须实现监控Endpoint ，eg: health、version、debug、status、metrics
- 🈲️止使用表名作为资源名，屏蔽保护底层体系结构
- Resp 包含资源总数
- GET 请求始终接收分页信息
- 获取要查询的字段参数，只暴露必须的信息 eg: GET /orders?fields=orderNo,amount
- URL **不携带身份认证信息**，通过Header传递，**身份信息短暂有效**
- **只对资源使用HTTP - PUT 操作**
- 为公共API支持CORS头部
- 使用HTTPS
- 无效、错误请求返回 4XX HTTP状态码，尽可能一次性验证所有问题，处理所有错误属性

## Google APIs

https://google-cloud.gitbook.io/api-design-guide/
