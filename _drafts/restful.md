---
title: REST and RESTful
layout: post
---

## REST
REST 是 Representational State Transfer 的简称，由 Roy Fielding 博士在2000年他的博士论文中提出来的一种软件架构风格。

*Roy Fielding*
参与了 HTTP1.0，HTTP 1.1，URI 相关规范的制定，

> 这个名称“表述性状态转移”是有意唤起人们对于一个良好设计的 Web 应
> 用如何运转的印象:一个由网页组成的网络(一个虚拟状态机),用户通过选择链接(状态
> 转移)在应用中前进,导致下一个页面(代表应用的下一个状态)被转移给用户,并且呈现
> 给他们,以便他们来使用。

## REST 架构约束

1. 客户端-服务器架构(Client-Server)
2. 无状态服务端(Stateless)
3. 缓存(Cache)
  * User Agent 缓存
  * 服务器缓存
4. 统一接口(Uniform Interface)
  * 资源标识(Indentification of resources)
  * 通过表述对资源进行操作(Manipulation of resources through representations) MIME
  * 子描述消息(Self-descriptive messages), HTTP 报文
  * 超媒体作为应用状态引擎(Hypermedia as the engine of application state, HATEOAS)
5. 分层系统(Layered system)，中间件缓存，代理，负载均衡
6. 按需代码(Code-on-demand) (optional)

## RESTful API 建议

1. Design URL based on ROA(Resource-Oriented Architecture)
  * 名词 vs 动词：REST 对于信息的核心抽象是资源。因此 URL 中使用名词，尽量不使用动词。
  * 单数 vs 复数：个体使用单数，集合使用复数

            GET     /articles
            GET     /articles/:id
            GET     /articles/:id/new
            POST    /articles
            GET     /articles/:id/edit
            PUT     /articles/:id
            DELETE  /articles/:id
2. Use HTTP method
  * 安全方法(Safe method) 请求不会没有副作用，不会修改资源在服务器上的状态。
  * 幂等方法(Idempotence method) 多次请求结果和一次请求结果相同

3. Use HTTP Status Code
  * 2xx  成功
    * 200
    * 201
    * 204
  * 3xx  重定向
  * 4xx  客户端错误
    * 401
    * 403
    * 404
    * 422
  * 5xx  服务器错误
4. Pagination
5. Use JSON replace XML

## Rails 中的 RESTful 实践

## 现实
http://developer.github.com
http://developers.douban.com

**参考**

1. [最佳实践：更好的设计你的 REST API](https://www.ibm.com/developerworks/cn/web/1103_chenyan_restapi/)
