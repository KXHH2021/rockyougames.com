---
title: Kernel Memory 入门系列：异步管道
date: 2024-01-03 20:00:00
categories:
  - Kernel Memory
tags:
  - RabbitMQ
  - Azure Queue
description: Kernel Memory 入门系列：异步管道
cover: https://s2.loli.net/2024/01/07/MdgQA8JCs9YGo52.png
---
## Kernel Memory 入门系列：异步管道
前面所介绍的处理流程都是基于同步管道的，即文档导入的时候，会等到文档处理完成之后才会返回。

但是在实际的应用中，文档很多，而且文档的处理时间也不确定，如果采用同步的方式，那么就会导致整个流程的处理时间过长，也会导致整个流程的阻塞。因此，我们需要一种异步的方式来处理这种情况。

## 注册消息队列
当我们使用异步管道的时候，需要先注册消息队列，Kernel Memory中默认提供了几种消息队列的实现，包括：

- RabbitMQ
- Azure Queue
- Simple Queue (file based, for testing)

这里以Simple Queue为例，在构建Kernel Memory的时候，可以通过WithSimpleQueuesPipeline方法来注册Simple Queue。
```undefined
var _ = new KernelMemoryBuilder(appBuilder.Services)
    //...
    .WithSimpleQueuesPipeline() // <- register simple queue
    .Build();
```
默认注册消息队列之后，处理流程就会以后台异步的方式进行处理。

## 后台任务
使用了异步队列之后，自定义的处理流程就注册方法就需要发生一些改变。

Kernel Memory提供的方式是将所有的异步处理流程都注册为HostedService, 也就是后台任务。

当注册自定义的处理流程的时候，就需要调用AddHandlerAsHostedService方法。
```undefined
builder.Services.AddHandlerAsHostedService<MyHandler>("my_step");
```
注册好的后台任务会监听消息队列，当有消息到达的时候，就会触发对应的处理流程。

其余的导入、自定义处理流程和同步管道的方式一样。

## 构建异步处理服务
通过分布式的文件存储、向量存储和消息队列服务，就可以将文件的导入和处理流程进行分离，从而实现异步的处理流程。
![image.png](https://s2.loli.net/2024/01/07/MdgQA8JCs9YGo52.png)

整体的处理流程如下：

- 1.文件导入后，会保存到分布式的文件存储中，同时会发送消息到消息队列中。
- 2.后台任务会监听消息队列，当有消息到达的时候，就会触发对应的处理流程。
- 3.客户端可以通过GetDocumentStatusAsync方法来获取文档的处理状态。

由此就可以实现异步的处理流程。
