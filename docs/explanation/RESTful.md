理解RESTful 
=================
* [1.起源](#origin)
* [2.名词解释](#glossary)
    * [2.1 RESTful](#restful)
    * [2.2 资源](#resource)
    * [2.3 表现层](#representation)
    * [2.4 状态转化](#stateTransfer)
    * [2.5 RESTful架构](#restfulArchitect)
* [3.架构属性](#architecturalProperties)
* [4.架构约定](#architecturalConstraints)
* [5.应用到web服务](#appliedToWebServices)

<h2 id="origin">1.起源</h2>
REST这个名词是Roy Thomas Fielding在2000年的博士论文中提出的。其主要目的是在符合架构原理的前提下，理解和评估以网络为基础的应用软件的架构设计，得到一个功能强，性能好，适宜通信的架构。  
在计算机领域，REST(representational state transfer)是一种构建可扩展web服务的软件架构方式。REST为分布式的超媒体系统（hypermedia system）的组件设计提供了一套完整的约束，遵从这些约束可以构建性能更高，更易于维护的系统。   
RESTful系统大部分采用HTTP协议通信，使用HTTP的标准方法(GET, POST, PUT, DELETE等)来获取或者更新数据。RESTful接口通常是带标识符的资源集合，例如`/people/paul`，这样可以使用标准的动作来对资源进行操作，例如 `DELETE /people/paul`。

<h2 id="glossary">2.名词解释</h2>

<h3 id="restful">2.1 RESTful</h3>
符合REST原则的架构就称为RESTful架构， Representational State Transfer(表现层状态转化)。

<h3 id="resource">2.2 资源</h3>
所谓资源是网络上的一个实体，或者说一条具体信息，可以是文本，图片，歌曲，服务等等，是一个具体的存在。我们可以用一个URI（同意资源定位符）指向它，也就是说每种资源对应了一个特定的URI。要访问资源，只需要访问这个URI，反过来说，URI也就成为了每一个资源的地址或独一无二的标识符。

<h3 id="representation">2.3 表现层</h3>
资源既然是是一个具体的实体，那么资源呈现出来的方式，就是它的表现层。

<h3 id="stateTransfer">2.4 状态转化</h3>
通过HTTP协议的标准方法(GET, POST, PUT, DELETE, HEAD, OPTION)对资源进行操作，导致资源的改变，也就实现了表现层状态的转化。

<h3 id="restfulArchitect">2.5 RESTful架构</h3>
1. 资源对应于一个URI
2. 客户端和服务器端传递的是资源的某种表现层。
3. 客户端通过HTTP方法，对服务器端资源进行crud，即实现状态的转化。

<h2 id="architecturalProperties">3.架构属性</h2>
REST架构风格的约束带来的架构属性主要有：
* 性能 —— 在用户可感知的性能和网络效率方面，组件交互可能是最主要的因素。
* 扩展性 —— 支持大量的组件和组件间的交互。
* 接口的简单性
* 随需应变的组件的可修改性（及时应用正在运行）
* 组件间通信的可见性
* 组件的可移植性——跟数据一起迁移代码
* 可靠性 —— 确保错误发生在组件、连接器或者数据中是避免错误发生在系统层面的主要手段。

<h2 id="architecturalConstraints">4.架构约定</h2>
REST架构的属性是通过对组件、连接器以及数据元素上应用特殊的交互约束来实现的。遵守本章定义的REST约束的应用程序可以称为RESTful的。因为遵从这些约束而符合REST架构风格，各种分布式的超媒体系统都会拥有完全的非功能属性(non-functional properties: 系统级的操作而不仅仅是某些特殊行为)，例如性能，可扩展性，简单性，可修改性，可移植性和可靠性。    
REST约束如下：
* 客户端 - 服务器端模式(Client - server)   
    统一的接口将客户端和服务器端分离开。关注点的分离意味着客户端不关注数据的存储，数据存储在服务器内部，这也保证了客户端代码的可移植性。服务器端不关注用户接口或者用户状态，因此服务器端可以更加简化并具有更高可扩展性。服务器端和客户端可以独立替换或者开发，只要他们的接口保持不变。
* 无状态(Stateless)    
    客户端-服务器端的通信被进一步限制：在多个请求之间，服务器端不会保存客户端的上下文。从任何客户端发出的请求都需要包含响应请求的所有必要信息，会话状态（session state）也在客户端保存。会话状态可以在server间转移以确保在一段时间内被固化以允许身份认证。当客户端准备好转移到一个新的状态时，它开始发起请求。当发起一个或多个请求，客户端被认为在转移中（in transition）。每个应用状态的表现层中包含链接，客户端可以在下一次使用它来初始化一个新的状态转移。
* 可缓存的(Cacheable)  
    跟www一样，客户端或者中间件可以缓存响应。因此，响应必须隐式或者显式的声明它们是可以缓存的还是不可以，这样可以避免客户端重新使用陈旧的数据来响应以后的请求。管理良好的缓存可以部分或者全部的消除客户端和服务器端的相互影响，进一步提高系统的可扩展性和性能。
* 分层的系统(Layered system)  
    客户端一般情况下并不知道它是直接连到终端服务器还是连到中间件。中间件服务器可以通过负载均衡或者共享缓存提高系统的可扩展性，并且也可以加强安全策略。
* 按需编码(Code on demand)    
    服务器端可以通过可执行代码的转移来临时扩展或者定制客户端的功能。按需编码是REST架构的唯一可选的约束。
* 统一接口(Uniform interface)    
    统一接口是REST服务设计的基础。统一接口简化并解耦了架构，系统的各个部分可以单独的进化。统一接口有四部分内容：
    * 资源的识别(Identification of resources)    
        请求中每个资源都是可被识别的，例如基于web的REST系统中使用 URI。资源本身在概念上跟返回客户端的表现层是分开的，例如，服务器端可能发送HTML，XML或者JSON格式的数据，但是在服务器内部数据却不是以这些格式保存。
    * 通过表现层的资源的操作(Manipulation of resources through these re(presentations)    
        当一个客户端拥有资源的表现层，包含附加的所有元数据，它就拥有足够的数据来修改或者删除这个资源。
    * 自描述的消息(Self-descriptive message)  
        每个消息包含足够的信息来描述如何处理这条消息，例如，可能通过Internet media type来标记引用什么解析器。响应消息也都明确的标记它们是否可以被缓存。
    * 作为应用状态引擎的超媒体(Hypermedia as the engine of application state)  
        客户端通过服务器端在超媒体中包含的动态的指定的动作来进行状态转移。除了应用程序的简单固定的入口外，客户端不会假设对任何特殊资源的特殊操作是可用的，除非它是早前从服务器端接收的表现层描述的资源。

<h2 id="appliedToWebServices">5.应用到web服务</h2>
符合REST架构约束的web service API就称为RESTful API，基于HTTP的RESTful API主要在这些方面定义：
* 基础URI
* 数据类型，建议JSON，当然XML等也可以
* 标准的HTTP方法
* 引用状态的超链接
* 引用相关资源的超链接

|         resource|              GET |               PUT |              POST |         DELETE|
|----------|----|-----|------|--------|
|集合URI：http://api.example.com/v1/resources/  | 列出集合的成员列表 | 使用另一个集合替换整个集合  | 创建集合中的新的实体，新实体的URI自动分配并且通常被返回给客户端 | 删除整个集合。|
|元素URI：http://api.example.com/v1/resources/item17 | 获取结合中指定的成员的表现层 | 替换集合中的指定的成员，如果不存在，则创建 | 通常很少使用，将指定成员本身作为一个集合，并创建一个新的实体 | 删除集合中的指定成员|

PUT和DELETE操作被认为是幂等的，这意味着无论重复操作多少次，都会产生相同的结果。GET方法是安全的，意味着调用它不会产生任何副作用。也就是说，获取或者访问一条记录不会改变它。   
跟基于SOAP的web service不同，并不存在官方的RESTful标准。这是因为REST只是一种架构样式，而SOAP是一种协议。

