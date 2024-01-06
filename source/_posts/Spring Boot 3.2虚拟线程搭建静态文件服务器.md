---
title: 有多快！Spring Boot 3.2虚拟线程搭建静态文件服务器 
date: 2024-01-06 20:48:00
categories:
  - 学习笔记
tags:
  - Spring Boot 3.2
  - Native Image
  - JVM
  - RestClient
  - 虚拟线程
  - Spring for Apache Pulsar
description: Spring Boot 3.2 于 2023 年 11 月大张旗鼓地发布，标志着 Java 开发领域的一个关键时刻。这一突破性的版本引入了一系列革命性的功能！
cover: https://s2.loli.net/2024/01/06/N9pB74fhsavoqZy.png
---
## Spring Boot 3.2虚拟线程搭建静态文件服务器

Spring Boot 3.2 于 2023 年 11 月大张旗鼓地发布，标志着 Java 开发领域的一个关键时刻。这一突破性的版本引入了一系列革命性的功能，包括：

- 虚拟线程：利用 Project Loom 的虚拟线程释放可扩展性，从而减少资源消耗并增强并发性。
- Native Image支持：通过Native Image编译制作速度极快的应用程序，减少启动时间并优化资源利用率。
- JVM 检查点：利用 CRaC 项目的 JVM 检查点机制实现应用程序的快速重启，无需冗长的重新初始化。
- RestClient：采用新的 RestClient 接口的功能方法，简化 HTTP 交互并简化代码。
- Spring for Apache Pulsar：利用 Apache Pulsar 的强大功能实现强大的消息传递功能，无缝集成到您的 Spring Boot 应用程序中。
  
其中，虚拟线程是最近 Java 版本中引入的最具变革性的特性之一。正如官方文件所述：虚拟线程是轻量级线程，可减少编写、维护和调试高吞吐量并发应用程序的工作量。线程是可以调度的最小处理单元。它与其他此类单位同时运行，并且在很大程度上独立于其他此类单元运行。它是 java.lang.Thread 的一个实例。有两种线程：平台线程和虚拟线程。平台线程是作为操作系统 （OS） 线程的瘦包装器实现的。平台线程在其底层操作系统线程上运行 Java 代码，平台线程在平台线程的整个生命周期内捕获其操作系统线程。因此，可用平台线程数限制为操作系统线程数。与平台线程一样，虚拟线程也是 java.lang.Thread 的实例。但是，虚拟线程不绑定到特定的操作系统线程。虚拟线程仍在操作系统线程上运行代码。但是，当在虚拟线程中运行的代码调用阻塞 I/O 操作时，Java 运行时会挂起虚拟线程，直到它可以恢复为止。与挂起的虚拟线程关联的操作系统线程现在可以自由地对其他虚拟线程执行操作。虚拟线程适用于运行大部分时间被阻塞的任务，通常等待 I/O 操作完成。但是，它们不适用于长时间运行的 CPU 密集型操作。

虽然人们普遍认为虚拟线程在 I/O 密集型方案中表现出色，但它们在 CPU 密集型任务中的性能仍然是一个问号。本系列文章深入探讨了虚拟线程在各种用例中的潜在优势，从基本的“hello world”到静态文件服务（I/O 密集型）、QR 码生成（CPU 密集型）和多部分/表单数据处理（混合工作负载）等实际应用。

在本系列的开头文章中，我们已经了解了虚拟线程与物理线程相比在最简单（且不切实际）的 hello world 情况下的性能。物理线程和虚拟线程之间几乎没有任何性能或资源使用差异。在本文中，我们将更加“实用”，并针对静态文件服务器情况进行比较。这绝对是一个常见且“真实世界”的案例。让我们看看这次我们发现了什么。

如果大家正在做Spring Boot 2.3升级Spring 3.2，这里顺手给大家推荐Spring Boot 2.x 到 3.2 的升级指南。

## 测试环境

所有测试均在配备 16G RAM、8 个物理内核和 4 个效率内核的 MacBook Pro M2 上执行。测试工具是 Bombardier，它是更快的 HTTP 负载测试器之一（用 Go 编写）。

软件版本为：

- Java v21.0.1
- Spring Boot 3.2.1

## 程序配置

除了主 Java 类之外，不需要编写任何 Java 文件，静态文件服务器只能通过配置就能发挥作用。

**application.properties**文件如下：
```undefined
server.port=3000

spring.mvc.static-path-pattern=/static/**

spring.web.resources.static-locations=file:/Users/mayankc/Work/source/perfComparisons/static/
```
使用虚拟线程时，我们将通过添加以下行来启用它们：
```undefined
spring.threads.virtual.enabled=true
```
**pom.xml**内容：
```undefined
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.1</version>
    <relativePath/>
 </parent>
 <groupId>com.example</groupId>
 <artifactId>demo</artifactId>
 <version>0.0.1-SNAPSHOT</version>
 <name>demo</name>
 <description>Demo project for Spring Boot</description>
 <properties>
   <java.version>21</java.version>
 </properties>
 <dependencies>
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-web</artifactId>
   </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
 </dependencies>
```
## 测试数据
大小完全相同但数据不同的 100K 文件被放置在静态资源目录中。每个文件大小正好是 102400 字节。

文件的命名范围为 1 到 100000。

使用 Bombardier 的修改版本，为每次运行生成随机请求 URL: **http://localhost:3000/static/<file-name>**

## 应用场景
为了确保结果一致，每个测试在开始数据收集之前都会经历 5K 请求预热阶段。

然后，在不同范围的并发连接级别（50、100 和 300）中仔细记录测量结果，每个级别都承受 500 万个请求工作负载。

## 结果评估
除了简单地跟踪原始速度之外，我们还将采用详细的指标框架来捕获延迟分布（最小值、百分位数、最大值）和吞吐量（每秒请求数）。

CPU 和内存的资源使用情况监控将补充此分析，从而提供不同工作负载下系统性能的全面了解。

## 测试结果
结果以图表形式呈现如下：

![image.png](https://s2.loli.net/2024/01/06/Pi32ade9gwZuT4M.png)

![image.png](https://s2.loli.net/2024/01/06/p1SDLEzXagyVG3P.png)


![image.png](https://s2.loli.net/2024/01/06/E2YLRif8jwcAWQm.png)


![image.png](https://s2.loli.net/2024/01/06/s3E5JQvz8u1DBR6.png)


![image.png](https://s2.loli.net/2024/01/06/1qaGKbjhRFlPDVC.png)


![image.png](https://s2.loli.net/2024/01/06/ajPd23B5AT6cZK1.png)

![image.png](https://s2.loli.net/2024/01/06/sTIjwtJRAMqoeV6.png)

![image.png](https://s2.loli.net/2024/01/06/rU7zXY4jEBkQcgd.png)

![image.png](https://s2.loli.net/2024/01/06/m1hXiZvT5zEWxeF.png)

![image.png](https://s2.loli.net/2024/01/06/19ENqeyjd5pUY3Q.png)
## 总结
对静态文件服务的分析表明，物理线程在性能和资源效率方面略胜一筹（与我们的预期相反）。

不过，这种受 I/O 限制的场景可能并不是充分发挥虚拟线程潜力的理想场所。涉及数据库交互的任务可能会显示出更多令人信服的优势。也许负载不足以让虚拟线程发挥出最大的作用。为了找出答案，我们将在接下来的文章中介绍 URL短链（数据库驱动）、二维码生成（CPU受限）和混合工作负载场景（如表单数据处理），旨在揭示虚拟线程真正出类拔萃的案例。
