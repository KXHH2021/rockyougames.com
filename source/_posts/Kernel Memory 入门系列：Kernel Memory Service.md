---
title: Kernel Memory 入门系列：Kernel Memory Service
date: 2023-01-10 20:00:00
categories:
  - Kernel Memory
tags:
  - Kernel Memory Service
  - Kernel Memory OpenAPI
description: Kernel Memory 入门系列：Kernel Memory Service
cover: https://s2.loli.net/2024/01/07/nJwL8RtczWBTlSA.png
---
![image.png](https://s2.loli.net/2024/01/07/nJwL8RtczWBTlSA.png)
## Kernel Memory 入门系列：Kernel Memory Service

在 Kernel Memory 的 GitHub 仓库中有一个单独的项目 Service , 用于提供独立的Kernel Memory OpenAPI 服务。可以用于独立的部署后台，也可以为其他语言提供接口调用服务。

## 1. 初始化
使用 Kernel Memory Service 需要先下载该项目的源码。使用Git Clone 命令下载源码到本地即可。
```undefined
git clone https://github.com/microsoft/kernel-memory.git
```
然后打开/service/Service目录，根据操作系统选择执行使用setup.sh或者setup.cmd，就可以进行快速的初始化了。

也可以直接执行dotnet run setup命令进行初始化。

这个过程会调用初始化服务，将所有选择的配置项都存储到appsettings.Development.json文件中。

当然也可以直接通过修改appsettings.json文件，来修改配置项。该文件中包含详细的配置说明。

这其中的大部分的配置项都和Kernel Memory的组件有关，例如ContentStorageType、TextGeneratorType、

MemoryDbTypes等等。大部分的组件将会在后续的文章中进行介绍。如果测试的话，可以选择Simple开头的选项，基本都是使用本地文件作为存储。

这其中需要注意的 TextGeneratorType 和 EmbeddingGeneratorType 及其相关配置是必须的，因为这部分是调用大语言模型的需要的配置。

## 2. 启动服务

完成配置之后，就可以启动服务了。运行项目下的run.cmd或者run.sh即可启动服务。

![image.png](https://s2.loli.net/2024/01/07/d39vZmgs1ClY2Sk.png)

这里为了演示方便，启用了Swagger和同步处理，使用了本地的文件存储和向量存储。

## 3. 使用服务

服务其中后，如果是在本地运行，可以通过**http://localhost:9001/swagger/index.html**访问服务的Swagger页面。

![image.png](https://s2.loli.net/2024/01/07/14gAV7pFhMWUPIf.png)

其他语言可以使用Swagger.json 生成OpenAPI的客户端代码，也可以直接使用HTTP请求调用接口。

C# 可以使用 Microsoft.KernelMemory.WebClient 包来调用接口。
```undefined
var memory = new MemoryWebClient("http://127.0.0.1:9001");
```
这里的memory 的使用和 ServerlessMemory 一样。

## 4. 一些其他的工具脚本

在该项目的另外一个目录下提供了一些工具脚本,可以更加方便的使用 Kernel Memory Service。

- run-qdrant.sh 用于启动 Qdrant 服务，该服务是用于提供向量数据库，用于存储生成的向量。使用Docker 运行，因此需要提前安装好Docker。

- run-rabbitmq.sh 用于启动 RabbitMQ 服务，该服务是用于提供消息队列服务，用于异步管理文件上传和处理进度。也是使用Docker 运行。

- setup-service.sh 用于初始化 Kernel Memory Service，和上面的setup.sh是一样的。

- run-service.sh 用于启动 Kernel Memory Service，和上面的run.sh是一样的。

- upload-file.sh 用于上传文件到 Kernel Memory Service，需要提供服务地址（默认是http://localhost:9001）和文件路径。

- ask.sh 用于向 Kernel Memory Service 提问，需要提供服务地址和问题文本。

- search.sh 用于向 Kernel Memory Service 搜索，需要提供服务地址和问题文本。

**参考**

Kernel Memory Service:**https://github.com/microsoft/kernel-memory/tree/main/service/Service**

Kernel Memory Tools:**https://github.com/microsoft/kernel-memory/tree/main/tools**
