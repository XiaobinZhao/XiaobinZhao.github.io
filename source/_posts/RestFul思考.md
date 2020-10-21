---
title: RESTful思考
date: 2020-10-21 17:49:01
description: Richardson的成熟度模型以及RESTful API设计的一些思考
tag: 
- RESTful
- etag
- Richardson成熟度模型
- RMM
categories:
- [架构]
---

# Richardson的成熟度模型

![RIchardson成熟度模型1](/images/restful-grade.png)
![RIchardson成熟度模型2](/images/RIchardson-restful.png)

## 零级
- 服务有**单个的URI**，并且使用**单个的HTTP方法**(通常是P0ST)。
- 该模型的出发点是使用HTTP作为远程交互的传输系统，但是不会使用Web中的任何机制。本质上这里你是为了使用你的远程交互而利用HTTP作为隧道机制(Tunneling Mechanism)，通常是基于远程过程调用(Remote Procedure Invocation)的。
- XML-RPC和普通老式XML( Plain Old XML，POX)使用了类似的方法: Http P0ST请求，和传递到单个URI端点的XML载荷，以及作为HTTP响应的一部分以XML格式交付的回复。

## 一级
- 使用了**很多URI**，但是只使用**单个HTTP动词**。这种基本的服务和零级服务之间的关键区别点在于，这种级别的服务暴露出了很多逻辑上的资源，而零级服务将所有的交互埋入了单个(大型的、复杂的)资源过将操作名称和参数插入到URI中，然后将该URI传递给一个远程服务(通常通过HTTP GET)，操作被埋藏了起来。
- 在Richardson成熟度模型中，通往真正REST的第一步是引入资源(Resource)这一概念。所以相比将所有的请求发送到单个服务端点(Service Endpoint)，现在我们会和单独的资源进行交互。
- 注意：*Richardson声明，如今大多数描述自己为“ RESTful”的服务实际上常常是一级服务。一级服务可以是很有用的，即使它们并没有严格遵守 RESTfull的束，因此它有可能通过使用一个动词(GET)意外地破坏数据，而这个动词原本不应该有这种副作用*。

## 二级
- 二级服务使用了**大量的可通过URI寻址的资源**。这样的服务支持多个HTTP动词来暴露资源。包含在这个级别的是CRUD( Create Read Update Delete，创建、读、更新和删除)服务，通常表示的是业务实体的资源状态，能够通过网络来操作。
- 在LEVEL 0和LEVEL 1中一直使用的是HTTP POST来完成所有的交互，但是有些人会使用GET作为替代。在目前的级别上并不会有多大的区别，GET和POST都是作为隧道机制(Tunneling Mechanism)让你能够通过HTTP完成交互。LEVEL 2避免了这一点，它会尽可能根据HTTP协议定义的那样来合理使用HTTP动词。
- 注意：*很重要的是，二级服务使用了HTTP动词和状态代码来协调交互，这意味着它们为实现健壮性而使用了web*。


## 三级
最Web感知(web- aware)的服务级别，支持**超媒体作为应用状态的引擎**的观念，这个概念的缩写是不那么好看的HATEOAS(Hypertext As The Engine Of Application State)。即，表述包含了消费者可能感兴趣的到其他资源的URI链接。这种服务通过追踪资源来引导消费者，结果是引起应用状态的迁移
注意：_“超媒体作为应用状态的引擎”来自 Fielding关于REST架构风格的工作。我们倾向于使用“超媒体约束”这个术语，因为它更短，而且传递了以下含义:使用超媒体来管理应用的状态，这是大规模计算系统的有利方面_

# Levels的意义(The Meaning of the Levels)
我应该强调一下，**Richardson成熟度模型([Richardson Maturity Model](http://martinfowler.com/articles/richardsonMaturityModel.html)，简称RMM)虽然是思考REST中有哪些元素的好方法，但是它并不直接定义REST中的级别**。Roy Fielding也阐明了这一点：Level 3 RMM是REST的前置条件。和软件中的众多专有名词一样，REST也有很多的定义，但是由于Roy Fielding创造了这个名词，他的定义会权威很多。

**RMM的用处在于它提供了一个层层递进的思考RESTful背后本质思想的方法**。正因为如此，我将它视为一个工具来帮助我们学习概念，而不是作为某种评估机制.

RMM和通用设计方法之间关系:
- Level 1 解释了如何通过分治法(Divide and Conquer)来处理复杂问题，将一个大型的服务端点(Service Endpoint)分解成多个资源。
- Level 2 引入了一套标准的动词，用来以相同的方式应对类似的场景，移除不要的变化。
- Level 3 引入了可发现性(Discoverability)，它可以使协议拥有自我描述(Self-documenting)的能力。
结果就是，这一模型帮助我们思考我们想要提供的HTTP服务是何种类型的，同时也勾勒出人们和它进行交互时的期望。

# web 开发中一些注意点

1. 善用[ETag](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/ETag)
HTTP响应头是资源的特定版本的标识符。这可以让缓存更高效，并节省带宽，因为如果内容没有改变，Web服务器不需要发送完整的响应。而如果内容发生了变化，使用ETag有助于防止资源的同时更新相互覆盖（“空中碰撞”）。
如果给定URL中的资源更改，则一定要生成新的Etag值。 因此Etags类似于指纹，也可能被某些服务器用于跟踪。 比较etags能快速确定此资源是否变化，但也可能被跟踪服务器永久存留。
如果我们不希望得到整个资源的表述，只是想要检查HTTP头信息，则可以使用动词HEAD。动词HEAD允许我们基于指定资源的上下文来决定如何做进一步的处理，这样做能够避免因为转移整个资源的表述而付出网络传输方面的额外代价。

2. [put还是patch](https://juejin.im/post/5ca83c6351882544183367e2)
PATCH 请求中的实体保存的是修改资源的指令，该指令指导服务器来对资源做出修改，所以不是幂等的。
PUT 用做更新操作的时候是提交一整个更新后的实体，而不是需要修改的实体中的部分属性。当 URI 指向一个存在的资源，服务器要做的事就是查找并替换。

3. HTTP不是RPC
人们常常错误地将HTTP称作一种远程过程调用(RPC)机制，仅仅是因为它也包括了请求和响应。RPC与其他形式的基于网络应用的通信的区别之处在于，从概念上讲它是在调用远程机器上的一个过程( procedure)。在RPC协议中，调用方识别出过程并且传递一组固定的参数，然后等待在使用相同接口返回的一个消息中提供的回答。远程方法调用(RMI)也是类似的，差异仅仅是将过程标识为一个{对象，方法}的组合，而不是一个简单的服务过程( service procedure)。被代理的RMI( Brokered rMi)添加了名称服务的间接层( name service indirection)和少量其他把戏( a few other tricks)，但是接口基本上是相同的。
将HTTP与RPC区分开的并不是语法，甚至也不是使用一个流作为参数所获得的不同的特性，尽管它帮助解释了为何现有的RPC机制对于Web而言是不可用的。HTTP与RPC之间的重大区别的是:**请求是被定向到使用一个有标准语义的通用接口的资源，中间组件能够采用与提供服务的机器几乎完全相同的方式来解释这些语义**。其结果是使得一个应用能够支持转换的分层( layers of transformation)和独立于信息来源的间接层( indirection that are independent of the
information origin)，这对于一个需要满足互联网规模、多个组织、无法控制的可伸缩性需求的信息系统来说，是非常有用的。与之相比较，RPC的机制是根据语言的API( language API)来定义的，而不是根据基于网络应用的需求来定义的。

# url 资源设计
决定哪些部分应该被分解成独立甚至重叠的资源，这是服务的设计流程的一部分。在进行这些决策时，我们需要考虑以下几个设计因素:
- 表述的大小：载荷会有多大?是否值得分解成多个资源来优化网络访问和缓存?
- 原子性：由于一种资源与其他资源处于一种组合关系之中，那么应用是否有可能进入不一致的状态?资源的整个表述需要打包在同一个载荷中吗?
- 信息的重要性： 我们真的需要将所有信息作为一个原子块( atomic block)来发送吗?我们可以允许消费者决定他们需要请求哪些链接资源吗?
- 性能/可伸缩性： 这个资源会被频繁地访问吗?要生成它的表述是否需要昂贵的计算量或者事务?
- 可缓存性： 资源表述可以被缓存和复制吗?与该资源相关的不同信息条目的变化频率是否是不同的?哪些信息项依赖于请求的上下文?哪些中立于该上下文?

回答这些问题有助于根据新的标准来分割资源，允许它的一些表述被缓存很长一段时间，而其他表述则是针对每一个请求而临时生成。


# 参考文献

- [ Richardson成熟度模型(Richardson Maturity Model) - 通往真正REST的步骤 ](https://blog.csdn.net/dm_vincent/article/details/51341037 )
- 《REST实战+中文版-4》

