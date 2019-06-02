---
title: Spring WebFlux 浅谈
date: 2019-04-26 17:02:35
tags: 
     - Java 
     - 框架
keywords:
description:
---


![1981_0.png](/images/03_spring_flux_begin/1981_0.png)

>要了解 WebFlux ，首先了解下什么是 Reactive Streams。
### Reactive Streams

 Reactive Streams 是 JVM 中面向流的库标准和规范：
* 处理可能无限数量的元素
* 按顺序处理
* 组件之间异步传递
* 强制性非阻塞背压（Backpressure）


##### Backpressure - 背压
背压是一种常用策略，使得发布者拥有无限制的缓冲区存储元素，用于确保发布者发布元素太快时，不会去压制订阅者。


##### Reactive Streams - 响应式流
一般由以下组成：

* 发布者：发布元素到订阅者

* 订阅者：消费元素

* 订阅：在发布者中，订阅被创建时，将与订阅者共享

* 处理器：发布者与订阅者之间处理数据


#####  Reactive Streams的特点 - Responsive
**可响应的。要求系统尽可能做到在任何时候都能及时响应。**

* Resilient: 可恢复的。要求系统即使**出错了，也能保持可响应性**。

* Elastic: 可伸缩的。要求系统在各种**负载下都能保持可响应性**。

* Message Driven: 消息驱动的。要求系统通过**异步消息连接各个组件**。


##### 什么是函数式
函数式编程是种**编程方式**，它将电脑运算视为函数的计算。函数编程语言最重要的基础是λ演算（lambda calculus），而且λ演算的函数可以接受函数当作输入（参数）和输出（返回值）。webFlux 是响应式编程框架。


##### 什么是响应式
简单来说响应式编程是**关于异步的事件驱动的**需要少量线程的垂直扩展而非水平扩展的**无阻塞应用**。响应式编程是一种面向数据流和变化传播的**编程范式**。这意味着可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型会自动将变化的值通过数据流进行传播。






### 什么是 webFlux


WebFlux 模块的名称是 spring-webflux，名称中的 Flux 来源于 Reactor 中的类 Flux。Spring webflux 有一个全新的非堵塞的函数式 Reactive Web 框架，可以用来构建异步的、非堵塞的、事件驱动的服务，在伸缩性方面表现非常好。

我们知道传统的Web框架，比如说：struts2，springmvc等都是基于Servlet API与Servlet容器基础之上运行的，在Servlet3.1之后才有了异步非阻塞的支持。而**WebFlux是一个典型非阻塞异步的框架，它的核心是基于Reactor的相关API实现的**。

![1983_0.png](/images/03_spring_flux_begin/1983_0.png)


#### Reactor

Java 领域的响应式编程库中，最有名的算是 Reactor 了。Reactor 也是 Spring 5 中反应式编程的基础，Webflux 依赖 Reactor 而构建。

Reactor 是一个基于 JVM 之上的异步应用基础库。为 Java 、Groovy 和其他 JVM 语言提供了构建基于事件和数据驱动应用的抽象库。Reactor 性能相当高，在最新的硬件平台上，使用无堵塞分发器每秒钟可处理 1500 万事件。


**简单说，Reactor 是一个轻量级 JVM 基础库，帮助你的服务或应用高效，异步地传递消息。Reactor 中有两个非常重要的概念 Flux 和 Mono** 。

![1985_0.png](/images/03_spring_flux_begin/1985_0.png)

##### 上图所示，webFlux各个模块：

* Router Functions: 对标@Controller，@RequestMapping等标准的Spring MVC注解，提供一套函数式风格的API，用于创建Router，Handler和Filter。

* WebFlux: 核心组件，协调上下游各个组件提供响应式编程支持。

* Reactive Streams: 一种支持背压（Backpressure）的异步数据流处理标准，主流实现有RxJava和Reactor，Spring WebFlux默认集成的是Reactor。

#### Flux 和 Mono

Flux 和 Mono 是 Reactor 中的两个基本概念。**Flux 表示的是包含 0 到 N 个元素的异步序列**。在该序列中可以包含三种不同类型的消息通知：正常的包含元素的消息、序列结束的消息和序列出错的消息。当消息通知产生时，订阅者中对应的方法 onNext(), onComplete()和 onError()会被调用。



**Mono 表示的是包含 0 或者 1 个元素的异步序列**。该序列中同样可以包含与 Flux 相同的三种类型的消息通知。Flux 和 Mono 之间可以进行转换。对一个 Flux 序列进行计数操作，得到的结果是一个 Mono对象。把两个 Mono 序列合并在一起，得到的是一个 Flux 对象。

#### WebFlux 优点
根据官方的说法，webflux主要在如下两方面体现出独有的优势：　　

**1. 非阻塞式　　　　**

其实在servlet3.1提供了非阻塞的API，WebFlux提供了一种比其更完美的解决方案。使用非阻塞的方式可以利用较小的线程或硬件资源来处理并发进而提高其可伸缩性　　

**2. 函数式编程端点　　**　　

老生常谈的编程方式了，Spring5必须让你使用java8，那么函数式编程就是java8重要的特点之一，而WebFlux支持函数式编程来定义路由端点处理请求。







#### WebFlux性能
[Spring WebFlux性能测试](https://blog.csdn.net/get_set/article/details/79492439)