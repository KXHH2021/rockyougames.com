---
title: Kernel Memory 入门:Quick Start
date: 2023-12-20 20:00:00
categories:
  - Kernel Memory
tags:
  - Quick Start
  - NuGet
  - Microsoft.KernelMemory.Core
description: Kernel Memory 入门教程：Quick Start
cover: https://s2.loli.net/2024/01/07/PLj1zRAEdqWxv8G.png
---
![image.png](https://s2.loli.net/2024/01/07/PLj1zRAEdqWxv8G.png)

## Kernel Memory 入门：Quick Start

了解了用户问答和文档预处理的流程之后，我们就可以直接开始使用Kernel Memory了。

## 1. 安装

项目中只需要通过NuGet安装**Microsoft.KernelMemory.Core**包即可。
```undefined
dotnet add package Microsoft.KernelMemory.Core
```

## 2. 构建
Kernel Memory的构建过程非常简单，只需要调用**KernelMemoryBuilder**的**Build**方法即可。
```undefined
var memory = new KernelMemoryBuilder()
                 .WithOpenAIDefaults(Env.Var("OPENAI_API_KEY"))
                 .Build<MemoryServerless>();
```
其中默认需要配置LLM和Embedding服务，如果使用OpenAI的服务，可以只需要通过WithOpenAIDefaults方法传入OpenAI的API Key即可。

默认使用的是gpt-3.5-turbo-16k作为文本生成模型，使用text-embedding-ada-002作为Embedding模型。

如果想要自定义其他的模型的话，可以使用以下方式：
```undefined
var memory = new KernelMemoryBuilder()
             .WithOpenAITextGeneration(new OpenAIConfig()
             {
                 APIKey = Env.Var("OPENAI_APIKEY"),
                 TextModel = "gpt-3.5-turbo"
             })
             .WithOpenAITextEmbeddingGeneration(new OpenAIConfig()
             {
                 APIKey = Env.Var("OPENAI_APIKEY"),
                 EmbeddingModel = "text-embedding-ada-002"
             })
             .Build<MemoryServerless>();
```
如果是使用的Azure OpenAI的话，通过以下方式配置：

 ```undefined
var memory = new KernelMemoryBuilder()
             .WithAzureOpenAITextGeneration(new AzureOpenAIConfig(){
                 APIKey = Env.Var("AZURE_OPENAI_API_KEY"),
                 Auth = AzureOpenAIConfig.AuthTypes.APIKey,
                 Endpoint = Env.Var("AZURE_OPENAI_ENDPOINT"),APIType = AzureOpenAIConfig.APITypes.ChatCompletion,
                 Deployment = "gpt-35-turbo-16k"
             })
             .WithAzureOpenAITextEmbeddingGeneration(new AzureOpenAIConfig()
             {
                 APIKey = Env.Var("AZURE_OPENAI_API_KEY"),
                 Auth = AzureOpenAIConfig.AuthTypes.APIKey,
                 Endpoint = Env.Var("AZURE_OPENAI_ENDPOINT"),
                 APIType = AzureOpenAIConfig.APITypes.EmbeddingGeneration,
                 Deployment = "text-embedding-ada-002"
             })
             .Build<MemoryServerless>();
```
这里忽略了文件存储、消息队列、向量数据库的配置，默认会使用内存的方式进行存储。

## 3. 内容导入
Kernel Memory提供了三种内容的导入的方式，第一种就是文件导入,最简单的方法就是直接指定文件的路径，然后调用ImportDocumentAsync方法即可。
```undefined
await memory.ImportDocumentAsync("./sample-SK-Readme.pdf");
```
如果不是本地文件的话，也可以使用文件流的方式进行导入。
```undefined
var fileStream = File.OpenRead("./sample-SK-Readme.pdf");
await memory.ImportDocumentAsync(fileStream, "sample-SK-Readme.pdf");
```
如果想一次性导入多个文档的话，可以使用Document的方式进行导入。
```undefined
var document = new Document();
document.AddFile("./sample-SK-Readme.pdf");
await memory.ImportDocumentAsync(document);
```
Document本身可以添加多个文件，也可以添加文件流。另外Document本身也可以添加筛选的标签。

除了文件的导入，还可以直接导入文本。
```undefined
var text = """
           some content
           """;
await memory.ImportTextAsync(text);
```
还有一种就是直接导入URL，这种方式会自动从URL中获取内容。
```undefined
var url = "https://raw.githubusercontent.com/microsoft/kernel-memory/main/README.md";
await memory.ImportWebPageAsync(url);
```
## 4. 问答和查询
文档导入完成之后，就可以进行问答或者查询了。

问答的话，只需要调用AskAsync方法即可。
```undefined
var question = "What's Kernel Memory?";
var answer = await memory.AskAsync(question);
```
其中**answer**包含了答案的内容，以及相关文档以及相关度。

例如使用以下方式获取答案的内容：

 ```undefined
Console.WriteLine(answer.Result + "\n");
 
foreach (var x in answer.RelevantSources)
{
    Console.WriteLine($"  * {x.SourceName} -- {x.Partitions.First().LastUpdate:D}");
}
 
/* OUTPUT */
/*
Kernel Memory is an open-source service and plugin specialized in the efficient indexing of datasets through custom continuous data hybrid pipelines. It enables natural language querying for obtaining answers from the indexed data, complete with citations and links to the original sources. Kernel Memory enhances data-driven features in applications built for popular AI platforms. It can be used as a library or as a Docker container.
 
  * content.url -- 2023年12月19日
  * sample-SK-Readme.pdf -- 2023年12月19日
*/
```

如果只想查询有哪些相关文档的话，可以使用SearchAsync 方法
```undefined
var question = "What's Kernel Memory?";
var results = await memory.SearchAsync(question);
```
同样的，从Results中可以获取到相关的文档。
```undefined
foreach (var x in results.Results)
{
    Console.WriteLine($"  * {x.SourceName} -- {x.Partitions.First().LastUpdate:D}");
}
/* OUTPUT */
/*
  * content.url -- 2023年12月19日
  * sample-SK-Readme.pdf -- 2023年12月19日
*/
```
参考
**Kernel Memory README**
