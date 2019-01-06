---
layout:     post
title:      springboot
subtitle:   springboot 异步提高并发处理能力
date:       2019-01-06
author:     LICSLAN
header-img: img/zjm.jpg
catalog: true
tags:
     - tec
     - spring boot
---


何为异步请求

在Servlet 3.0之前，Servlet采用Thread-Per-Request的方式处理请求，即每一次Http请求都由某一个线程从头到尾负责处理。如果一个请求需要进行IO操作，比如访问数据库、调用第三方服务接口等，那么其所对应的线程将同步地等待IO操作完成， 而IO操作是非常慢的，所以此时的线程并不能及时地释放回线程池以供后续使用，在并发量越来越大的情况下，这将带来严重的性能问题。其请求流程大致为：<br>
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/同步io_servlet.jpg)<br>

而在Servlet3.0发布后，提供了一个新特性：异步处理请求。可以先释放容器分配给请求的线程与相关资源，减轻系统负担，释放了容器所分配线程的请求，其响应将被延后，可以在耗时处理完成（例如长时间的运算）时再对客户端进行响应。其请求流程为：<br>
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/异步io_servlet3.0.jpg)<br>
随着Spring5发布，提供了一个响应式Web框架：Spring WebFlux。之后可能就不需要Servlet容器的支持了。以下是其先后对比图：
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/springboot_reactor.jpg)<br>
原生异步请求，Servlet方式实现异步请求<br>
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/servlet_asyn01.jpg)<br>
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/servlet_asyn01.jpg)<br>
注意：异步请求时，可以利用ThreadPoolExecutor自定义个线程池。<br>

Spring方式实现异步请求<br>
在Spring中，有多种方式实现异步请求，比如callable、DeferredResult或者WebAsyncTask。每个的用法略有不同，可根据不同的业务场景选择不同的方式。以下主要介绍一些常用的用法<br>

Callable<br>

![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/callable.jpg)<br>
控制台输出日志：
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/callable_log.jpg)<br>
超时、自定义线程设置

从控制台可以看见，异步响应的线程使用的是名为：MvcAsync1的线程。第一次再访问时，就是MvcAsync2了。若采用默认设置，会无限的创建新线程去处理异步请求，所以正常都需要配置一个线程池及超时时间。<br>
编写一个配置类：CustomAsyncPool.java<br>
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/CustomAsyncPool.jpg)<br>
自定义一个超时异常处理类：CustomAsyncRequestTimeoutException.java<br>
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/CustomAsyncRequestTimeoutException.jpg)<br>
同时，在统一异常处理加入对CustomAsyncRequestTimeoutException类的处理即可，这样就有个统一的配置了。<br>
之后，再运行就可以看见使用了自定义的线程池了，超时的可以自行模拟下：<br>
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/CustomAsyncRequestTimeoutException_log.jpg)<br>

DeferredResult<br>

相比于callable，DeferredResult可以处理一些相对复杂一些的业务逻辑，最主要还是可以在另一个线程里面进行业务处理及返回，即可在两个完全不相干的线程间的通信。
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/DeferredResult.jpg)<br>
注意：返回结果时记得调用下setResult方法。
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/DeferredResult_log.jpg)<br>
题外话：利用DeferredResult可实现一些长连接的功能，比如当某个操作是异步时，我们可以保存这个DeferredResult对象，当异步通知回来时，我们在找回这个DeferredResult对象，之后在setResult会结果即可。提高性能。<br>

WebAsyncTask<br>

使用方法都类似，只是WebAsyncTask是直接返回了。觉得就是写法不同而已<br>
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/WebAsyncTask.jpg)<br>
控制台输出：<br>
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/WebAsyncTask_log.jpg)<br>

主要是讲解了异步请求的使用及相关配置，如超时，异常等处理。设置异步请求时，记得要设置超时时间。

