---
title: "分布式缓存"
date: 2021-10-01T12:40:40+08:00
draft: true
toc: true
images:
tags: 
  - distributed
---

## Cache

缓存：存储在计算机上的一个原始数据复制集，以便于访问。

属于存储领域，目的是为了让算法便捷高速运行，提升用户体验、系统性能。

Linux 的页表（page table）和内存管理单元（MMU，物理硬件）负责将页的虚拟地址映射到物理地址。页表负责记录哪些是物理页，哪些是虚拟页，以及这些页的页表条目(PTE)。MMU 负责进行虚拟地址和物理地址的翻译，翻译过程中需要从页表获取页的PTE，MMU也会使用翻译后备缓存器（TLB）的缓存页号。

缓存无处不在，根据所处位置不通，可以分为

- 客户端缓存
- 服务端缓存
- 网络中缓存

根据规模和部署方式缓存可以分为

- 单体缓存
- 缓存集群
- 分布式缓存

### 客户端页面缓存

页面缓存可以是对页面自身某些元素或全部元素进行缓存，或者是服务端将静态页面或动态页面的元素进行缓存。将之前渲染的页面保存为文件，再次访问可以避开网络连接，从而减少负载，提升性能，这种事实现自身的缓存和离线应用缓存。

随着**单页面应用**(Single Page Application,SPA**）的广泛应用，加上HTML5支持了**离线缓存和本地存储**，大部分BS应用的页面缓存都可以举重若轻了。

```css
localStorage.setItem/getItem/removeItem/clear
```

页面缓存涉及到需要缓存的资源列表清单文件 **manifest** text/cache-manifest

### 浏览器缓存

浏览器缓存是根据一套与服务器约定的规则进行工作的，检查确保副本是最新的，通常只要一次会话。

HTTP 1.0 服务端侧 HTTP头部设置 **Expires** 来告诉客户端**在重新请求文件之前缓存多久是安全的**，可以通过 if-modified-since 的条件请求来使用缓存。发送的时间是文件最初被下载的时间，**如果文件没有改变，服务器可以用 304-Not Modified 来应答**，客户端收到304代码，就可以使用缓存的文件版本了。

HTTP1.1又了较强增强，**缓存系统被形式化，引入了实体标签 e-tag（文件或对象的唯一标识）**，可以请求一个资源，以及提供所持有的文件，然后询问服务器这个文件是否有变化。如果文件的e-tag 是有效的，服务器会生成304-Not Modified 应答，并提供正确文件的e-tag，否则发送200-OK应答。如果是**集群**，则每次都是不同的ID，不建议使用ETag。

配置了**Last-Modified/ETag** 的情况下，浏览器再次访问统一URL时，还是会发送请求到服务器询问文件是否已经修改，如果没有，服务器只会发送一个304回给浏览器，浏览器则直接从本地缓存取数据，如果有变化，就将整个数据重新发给浏览器。

Last-Modified/ETag 和 Cache-Control/Expires 的作用是不一样的，**如果检测到本地的缓存还在有效的时间范围内，浏览器则直接使用本地缓存，不会发送任何请求**。**两者一起使用时，Cache-Control/Expires 的优先级要高于Last-Modfied/ETag**。当本地副本根据 Cache-Control/Expires 发现还在有效期内时，不会再次发送请求去服务器询问修改时间(Last-Modified)或实体标识(e-tag)了。

Cache-Control 与 Expires 的功能一致，都是指明当前资源的有效期，控制浏览器是直接从浏览器缓存取数据还是重新发请求到服务器存取数据。Cache-Control 的选择更多、更细致，如果同时设置，优先级高于Expires。

Cache-Control 设置相对时间，max-age 指明以秒为单位的缓存时间。

Expires 设置以分钟为单位的绝对过期时间。优先级比Cache-Control低，同时设置Expires 和 Cache-Control ，后者生效。

一般情况会一起使用 Cache-Control/Expires 、Last-Modified/ETag，这样即使服务端设置了缓存时间，当用户刷新时，浏览器会忽略缓存继续向服务器发送请求，这时 Last-Modified/ETag 能够很好利用服务端的返回码304,从而减少开销。

HTML页面节点 meta 标签可以告诉浏览器当前页面不被孩奴承诺，每次访问都需要取服务器拉取，但是只有部分浏览器可以支持，一般缓存代理服务器都不支持，因为代理不解析HTML内容本身。

**nginx no-cache**

more_set_headers 'Cache-Control: no-cache'

```html
<meta http-equiv="Pramma" CONTENT="no-cache">
<meta http-equiv="Cache-Control" content="no-cache" />
<meta http-equiv="expires" content="Wed, 26 Feb 2555 08:21:57 GMT" />
<meta http-equiv="Cache-Control" content="no-transform" />
<meta http-equiv="Cache-Control" content="no-siteapp" />
<meta http-equiv="X-UA-Compatible" content="IE=edge" />
```

no-cache: 数据内容不能被缓存, 每次请求都重新访问服务器, 若有max-age, 则缓存期间不访问服务器.

no-store: 不仅不能缓存, 连暂存也不可以(即: 临时文件夹中不能暂存该资源)

private(默认): 只能在浏览器中缓存, 只有在第一次请求的时候才访问服务器, 若有max-age, 则缓存期间不访问服务器.

public: 可以被任何缓存区缓存, 如: 浏览器、服务器、代理服务器等

max-age: 相对过期时间, 即以秒为单位的缓存时间.

no-cache, private: 打开新窗口时候重新访问服务器, 若设置max-age, 则缓存期间不访问服务器.

private, 正数的max-age: 后退时候不会访问服务器

no-cache, 正数的max-age: 后退时会访问服务器

点击刷新: 无论如何都会访问服务器.

地址栏回车、连接跳转、新开窗口、前进后退都会影响缓存，F5刷新不会影响。

还有浏览器通过页面指令设置HTTP缓存。。。

### 网络中的缓存



