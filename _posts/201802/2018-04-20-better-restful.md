---
layout: post
title:  "RESTful & JSON API"
date:   2018-04-20 12:00:00

categories: tool
tags: api
author: "Victor"
---

正在读《RESTful Web APIs中文版》之后会再次补充此文。

## 前提

**永远不要跟业界通行的方法背着干，这不是个人偏好的问题，是项目生死的大事。**

所谓代码风格、接口形式、各种林林总总的格式规定，其实都是为了在团队内部形成共识、防止个人习惯差异引起的混乱。

JSON-RPC 当然也是有规范的，但相比 REST 实在宽松太多了。

注意：多数时候我们提到的规范一词都是 specification 的意思，不是 regulation 的意思，其本身并没有约束力。

## REST

REST -- Resource Representational State Transfer 直接翻译：资源在网络中以某种表现形式进行状态转移。抱歉，这个中文我完全不懂什么意思。

### 起源

它首次出现在 2000 年 Roy Fielding 的博士论文中，Roy Fielding 是 HTTP 规范的主要编写者之一。

他在论文中提到：“我这篇文章的写作目的，就是想在符合架构原理的前提下，理解和评估以网络为基础的应用软件的架构设计，得到一个功能强、性能好、适宜通信的架构。REST 指的是一组架构约束条件和原则。”

REST 本身并没有创造新的技术、组件或服务，而隐藏在 RESTful 背后的理念就是使用 Web 的现有特征和能力，更好地使用现有 Web 标准中的一些准则和约束。虽然 REST 本身受 Web 技术的影响很深，但是理论上 REST 架构风格并不是绑定在 HTTP 上，只不过目前 HTTP 是唯一与 REST 相关的实例。 所以我们这里描述的 REST 也是通过 HTTP 实现的 REST。

**REST 远远不仅是指 API 的风格，它是一种网络应用的架构风格。**它被定义为 [6 个限制](https://en.wikipedia.org/wiki/Representational_state_transfer#Architectural_constraints)，满足这 6 个限制，能够获得诸多好处。

### 详解

这一段太长不想看，那只要记住这句 **就是用 URL 定位资源，用 HTTP METHOD 描述操作，看 HTTP STATUS CODE 就知道结果如何。**

* Resource：资源，可以是一段文本、图片、歌曲或者 **一种服务**。比如 newsfeed，friends 等；
* Representational：某种表现形式，比如 JSON，XML，JPEG 等；
* State Transfer：状态变化。通过 HTTP 动词实现。
  * 取东西就要 GET - GET就是安全的，不会修改服务资源
  * 新增就要 POST - POST就是不安全的
  * 修改就要 PUT - PUT就要幂等
  * 删除就是 DELETE - DELETE就要幂等

REST 的使用场景是 Machine-to-machine 的系统集成，目标是让服务发布者和消费者在最小约束下自由演化。

这个约束是指服务契约，简单讲就是服务输入输出的语义。消费者只需知道服务的根资源的 URI，就可以由根资源引导到所需的资源。换句话说，消费者和发布者的耦合只在于根资源的 URI 以及各资源及其操作的语义。

## RESTful API

我们浏览一个网站，说到底就是与这个网站中的资源进行互动（获取、提交、更新、删除）。前端的工作，就是为用户从服务端获取资源、展示资源、请求服务端改变资源。

很多人在尚未真正理解 REST 的一些核心概念的情况下，就到处吹自己设计的 API 是 RESTful API，大佬 Fielding 忍无可忍发了一篇博客意思是，RESTful API 必须是超文本驱动的，也就是 HATEOAS 这个概念。

**只要把 HTTP 标准和扩展的 RFC 看完，就会觉得 RESTful 就是按 HTTP 说的那样用。只是很多人的 HTTP 没按 HTTP 说的那样用。**

### 理解RESTful

一个架构符合 REST 原则，就称它为 RESTful 风格。

要理解 RESTful 架构，需要理解 Representational State Transfer 这个词组到底是什么意思，它的每一个词都有些什么涵义。

下面我们结合 REST 原则，围绕资源展开讨论，从资源的定义、获取、表述、关联、状态变迁等角度，列举一些关键概念并加以解释。

* 资源与 URI
* 统一资源接口
* 资源的表述
* 资源的链接
* 状态的转移

### 资源 Resource

REST 全称是表述性状态转移，那究竟指的是什么的表述? 其实指的就是资源。

* 任何事物，只要有被引用到的必要，它就是一个资源。
* 资源可以是实体(例如手机号码)，也可以只是一个抽象概念(例如手机号码的价值) 。
* 资源是任何可以被操作（获取、提交、更新、删除）的数据，比如一个文档（document）、一张图片
* 资源的集合也是一种资源，比如 blogs

#### URI

要让一个资源可以被识别，需要有个唯一标识，在 Web 中这个唯一标识就是 URI Uniform Resource Identifier。

* URI 既可以看成是资源的地址，也可以看成是资源的名称。
* 如果某些信息没有使用 URI 来表示，那它就不能算是一个资源，只能算是资源的一些信息而已。
* URI 的设计应该遵循可寻址性原则，具有自描述性，需要在形式上给人以直觉上的关联。

进行资源操作的时候，用 URI 来指定被操作的资源。如果一个 URI 不仅能标识一个网络上的资源，还能够定位这个资源，那么这个 URI 也叫 URL(是 URI 的子集)。关于 URI, URN, URL 的区别可参考相关阅读。

#### 用 URL 定位资源

既然 URL 表示一个资源，自然就不应该包含动词，它应该由名词组成。一个 not RESTful 的例子是通过向 `api/delete/resource` 发送 GET 请求来删除一个资源。

#### URI 设计上的一些技巧

* 使用 `_` 或 `-` 来让 URI 可读性更好
* 使用 `/` 来表示资源的层级关系
* 使用 `?` 用来过滤资源，这种 URL 通常对应的是一些特定条件的查询结果或算法运算结果
* `,` 或 `;` 可以用来表示同级资源的关系，现在 github 是使用 `...` 来做这个事情的

```
http://www.oschina.net/news/38119/oschina-translate-reward-plan
/pulls?state=closed
/git/git/compare/master...next
```

#### 操作资源

RESTful 架构应该遵循统一接口原则，统一接口包含了一组受限的预定义的操作，不论什么样的操作，都是通过使用相同的接口进行资源的访问。

接口应该使用标准的 HTTP 方法如 GET，PUT 和 POST，并遵循这些方法的语义。

| HTTP METHOD | SQL    | 备注                                           | 安全性 | 幂等性 |
| ----------- | ------ | ---------------------------------------------- | ------ | ------ |
| GET         | SELECT | 获取资源（一个资源或资源集合）                 | √      | √      |
| HEAD        |        | 获取资源的元数据                               | √      | √      |
| POST        | CREATE | 新建一个资源                                   |        |        |
| PUT         | UPDATE | 更新资源（客户端提供改变后的完整资源）         |        | √      |
| PATCH       | UPDATE | 更新资源（客户端提供改变的部分）               |        |        |
| DELETE      | DELETE | 删除资源                                       |        | √      |
| OPTIONS     |        | 获取信息，关于资源的哪些属性是客户端可以改变的 |        | √      |

* 幂等方法：对于同一个内容的请求，发出n次的效果与发出1次的效果相同
* 安全方法：不会造成服务器上资源的改变

#### 一些特殊情况

统一接口是否意味着不能扩展带特殊语义的方法?

* 统一接口并不阻止你扩展方法，只要方法对资源的操作有着具体的、可识别的语义即可，并能够保持整个接口的统一性。
* 像 WebDAV 就对 HTTP 方法进行了扩展，增加了 LOCK、UPLOCK 等方法。

如果GET请求增加计数器，这是否违反安全性?

* 安全性不代表请求不产生副作用，例如像很多 API 开发平台，都对请求流量做限制。但客户端不是为了追求副作用而发出这些 GET 或 HEAD 请求的，产生副作用是服务端 "自作主张" 的。所以服务端在设计时，也不应该让副作用太大。

#### 通过 HTTP 状态码表示操作的结果

虽然 HTTP 状态码设计的本意就是表示操作结果，但是有时候人们往往没有很好的利用它，RESTful API 要求充分利用 HTTP 状态码

### 资源的表述 Representational

客户端和服务端之间传送的也是资源的表述，而不是资源本身。

* 资源是一个抽象的概念，资源无法被传输，只能传输资源的表示 representation，格式可能是 XML，JSON 等等
* 表示 representation 包括数据 data 和元数据 metadata，所以服务端返回的 JSON 字符串并不是完整的 representation，整个 HTTP 响应才是 representation。
* 客户端可以通过 `Accept` 头请求一种特定格式的表述，服务端则通过 `Content-Type` 告诉客户端资源的表述形式。

也有通过 URI 后缀来区分表述格式，比如 `users.json`，但是这种方式混淆了资源的名称和资源的表述形式。

### 状态的转移 State Transfer

有了上面的铺垫，再讨论 REST 里边的状态转移就会很容易理解了。REST 原则中的无状态通信原则。是指服务端不应该保存客户端状态，而不是客户端应用不能有状态。

#### 应用状态与资源状态

状态应该区分应用状态和资源状态，客户端负责维护应用状态，而服务端维护资源状态。

* 客户端与服务端的交互必须是无状态的，并在每一次请求中包含处理该请求所需的一切信息。
* 服务端不需要在请求间保留应用状态，只有在接受到实际请求的时候，服务端才会关注应用状态。

#### 应用状态的转移

状态转移到这里已经很好理解了，“会话”状态不是作为资源状态保存在服务端的，而是被客户端作为应用状态进行跟踪的。
客户端应用状态在服务端提供的超媒体的指引下发生变迁。服务端通过超媒体告诉客户端当前状态有哪些后续状态可以进入。

## HATEOAS 和 JSON:API

REST 还有一个核心的概念：HATEOAS (Hypermedia As The Engine Of Application State)。

对于客户端的资源请求，服务器不仅要返回所请求的资源，而且要返回客户端所处的状态和可转移的状态。（客户端有状态）

客户端不需要提前知道应用有哪些状态，而是根据服务端响应的“可转移的状态”，提供给用户选择，从而发生状态转移。

用简单的话来说，在严格的 RESTful 架构中，客户端不需要提前知道服务端的 API 有哪些、怎么调用，在客户端与服务器通信的过程中，服务端会告诉客户端：在你当前所处的状态下，有哪些 API 可以使用、可以转移到哪些状态。

### 举例说明

我们正常浏览网页的过程，你访问首页就返回了很多可供点击的子导航，点击子导航之后，进入详情页，还可以点击相关的新闻之类的。

10086 的电话自动语音应答服务也是一个不错的例子。你想要查询你的手机流量，只需要会拨打 10086，对方会提示你按下哪些按键就能进入哪些状态。进入下一个状态以后，又会有语音提示你接下来能够按哪些按键。最终，你能进入到你想要的那个状态（流量查询服务）。你需要记住的仅仅是 10086 这个号码而已。

### JSON:API

现在再来看 JSON:API 的返回就会觉得，特别棒了吧。

```json
{
  "links": {
    "self": "http://example.com/articles",
    "next": "http://example.com/articles?page[offset]=2",
    "last": "http://example.com/articles?page[offset]=10"
  },
  "data": [{
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "JSON:API paints my bikeshed!"
    },
    "relationships": {
      "author": {
        "links": {
          "self": "http://example.com/articles/1/relationships/author",
          "related": "http://example.com/articles/1/author"
        },
        "data": { "type": "people", "id": "9" }
      },
      "comments": {
        "links": {
          "self": "http://example.com/articles/1/relationships/comments",
          "related": "http://example.com/articles/1/comments"
        },
        "data": [
          { "type": "comments", "id": "5" },
          { "type": "comments", "id": "12" }
        ]
      }
    },
    "links": {
      "self": "http://example.com/articles/1"
    }
  }],
  "included": [{
    "type": "people",
    "id": "9",
    "attributes": {
      "firstName": "Dan",
      "lastName": "Gebhardt",
      "twitter": "dgeb"
    },
    "links": {
      "self": "http://example.com/people/9"
    }
  }, {
    "type": "comments",
    "id": "5",
    "attributes": {
      "body": "First!"
    },
    "relationships": {
      "author": {
        "data": { "type": "people", "id": "2" }
      }
    },
    "links": {
      "self": "http://example.com/comments/5"
    }
  }, {
    "type": "comments",
    "id": "12",
    "attributes": {
      "body": "I like XML better"
    },
    "relationships": {
      "author": {
        "data": { "type": "people", "id": "9" }
      }
    },
    "links": {
      "self": "http://example.com/comments/12"
    }
  }]
}
```

### RESTful API 有哪些好处

总的来说就是 **简单、低耦合**，因为在复杂、紧耦合的情况下事情更容易失控。

举例来说好处的话，大概有这么几条：

* 透明性，暴露资源存在，面向资源，服务自解释
* 充分利用 HTTP 协议本身语义。
* 无状态，幂等。这点非常重要。在调用一个接口（访问、操作资源）的时候，可以不用考虑上下文，不用考虑当前状态，极大的降低了复杂度。
* HTTP 本身提供了丰富的内容协商手段，无论是缓存，还是资源修改的乐观并发控制，都可以以业务无关的中间件来实现。
* 降低消费者对服务内部实现细节的耦合

## 一些相关的概念

### 服务器无状态

根据 REST 的架构限制，RESTful 的服务器必须是无状态的，这意味着来自客户的每一个请求必须包含服务器处理该请求所需的所有信息，服务器不能利用任何已经存储的 上下文(context，在这里表示用户的状态) 来处理新到来的请求，会话状态只能由客户端来保存，并且在请求时一并提供。

无状态意味着分布式系统能够更好地工作，负载均衡器可以自由地将请求分发到任意的服务器。让系统的横向拓展能力强大。

### HTTP 幂等性

幂等性是分布式系统设计中十分重要的概念，而 HTTP 的分布式本质也决定了它在 HTTP 中具有重要地位。简单的说 **除 POST 之外的所有方法都是幂等** 的。

有一种看法 PATCH 不一定是幂等的。PATCH 的实现方式有可能是 **提供一个用来替换的数据**，也有可能是 **提供一个更新数据的方法** (比如 `data++`)。如果是后者，那么 PATCH 不是幂等的。

* [HTTP请求-幂等、安全概念](https://blog.csdn.net/zcmuczx/article/details/79397029)
* [编程中的幂等性 —— HTTP幂等性](https://blog.csdn.net/scratlc/article/details/78205363)
* [理解HTTP幂等性](https://www.cnblogs.com/weidagang2046/archive/2011/06/04/2063696.html)

### HTTP 状态码

2XX/3XX 都是请求成功，但是结果不同。4XX 是请求出错，5XX 是服务器处理出现错误。

相关阅读：

* [关于 RESTful API 中 HTTP 状态码的定义的疑问](https://www.zhihu.com/question/58686782/answer/159603453)
* [10 Status Code Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)
* [List of HTTP status codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#cite_note-15)
* [Choosing an HTTP Status Code — Stop Making It Hard](http://racksburg.com/choosing-an-http-status-code/)

### RESTful 常见的一些问题

1. `login` 和 `logout` 应该怎么 REST 化
2. 多条件复合搜索在 GET 里写不下怎么办
3. 大量资源的删除难道要写几千个 `DELETE`

其实在理解了 REST 后，这些都不是什么无解的难题，只是思维方式要转换一下：

1. `login` 和 `logout` 只是对 `session` 资源的创建和删除
2. search 本身就是个资源，使用 POST 创建，如果不需持久化，可以直接在 Response 中返回结果，如果需要（如翻页、长期缓存等），直接保存搜索结果并 303 跳转到资源地址就行
3. id 多到连 url 都写不下的请求，应该创建 task，用 GET 返回 task 状态甚至执行进度



## JSON-RPC vs RESTful API

### RPC

RPC 的思想是把本地函数映射到 API，也就是说一个 API 对应的是一个 function，我本地有一个 `getAllUsers`，远程也能通过某种约定的协议来调用这个 `getAllUsers` 至于这个协议是 Socket、是 HTTP 还是别的什么并不重要；

RPC 中的主体都是动作，是个动词，表示我要做什么。

### RESTful

URL 主体是资源，是个名词。而且也仅支持 HTTP 协议，规定了使用 HTTP Method 表达本次要做的动作，类型一般也不超过那四五种。这些动作表达了对资源仅有的几种转化方式。

### 比较

两者没有高下之分，无非是一种约定俗成的标准。习惯用 RPC 就用 RPC，能理解 REST 就用 REST。

* JSON-RPC 比较符合直观，格式也相对宽松
* REST 流行，有设计规范

JSON-RPC 无法像 REST 一样享受 HTTP 的各种优点(standard interface, stateless, cache..)，又必须承担 HTTP 作为基于文本的协议，payload 过大传输的成本以及序列化反序列化的开销。

如果你想寻求一种 RPC 框架，Thrift 或 protobuf 无疑更合适。

## RFC

Request For Comments（RFC），是一系列以编号排定的文件。文件收集了有关因特网相关资讯，以及 UNIX 和因特网社群的软件文件。

所有关于 Internet 的正式标准都是以 RFC 文档形式出版。但大量的 RFC 文档都不是正式的标准，出版目的都是为了提供信息。

RFC 文档在成为官方标准前一般至少要经历 4 个阶段：因特网草案、建议标准、草案标准、因特网标准。

[RFC文档目录](http://man.chinaunix.net/develop/rfc/default.htm)

## 相关

### 工具

* [apiary.io](https://apiary.io/)
* [api blueprint](https://apiblueprint.org/)

### 范例

* [Teambition Core Api](https://docs.teambition.com/#me-get_preference)

### 相关阅读

* [Architectural Styles and the Design of Network-based Software Architectures](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)
* [从消费者的角度评估REST的价值](http://hippoom.github.io/blogs/value-of-hypermedia-from-client-perspective.html)
* [理解本真的REST架构风格](http://www.infoq.com/cn/articles/understanding-restful-style)
* [理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful)
* [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)
* [怎样用通俗的语言解释REST，以及RESTful？](https://www.zhihu.com/question/28557115/answer/48094438)
* [RESTful风格的优势是什么](https://blog.csdn.net/wlchn/article/details/48369233)
* [WEB开发中，使用JSON-RPC好，还是RESTful API好](https://www.zhihu.com/question/28570307/answer/163638731)
* [举例说明，RESTful 到底有哪些好处？](https://www.zhihu.com/question/20130130)
* [URL和URI区别](https://www.cnblogs.com/wuyun-blog/p/5706703.html)
* [RESTful 架构详解](https://www.runoob.com/w3cnote/restful-architecture.html)
