---
title: Kernel Memory 入门系列：Semantic Kernel 插件
date: 2023-12-30 20:00:00
categories:
  - Kernel Memory
tags:
  - Semantic Kernel
description: Kernel Memory 入门系列：Semantic Kernel 插件
cover: https://s2.loli.net/2024/01/07/O5Dezq7SLnaKIMP.png
---
![image.png](https://s2.loli.net/2024/01/07/O5Dezq7SLnaKIMP.png)
## Kernel Memory 入门系列：Semantic Kernel 插件

Kernel Memory 本身提供了完整的RAG能力，这部分能力如果通过Semantic Kernel Memory的话，也是可以实现的，但是整体的管理成本会比较高。

因此通过Kernel Memory 构建知识库管理，然后通过插件的方式，将Kernel Memory接入到Semantic Kernel 中，这样就可以有效的提升整体的使用效率。

## 初始化插件
首先需要在Semantic Kernel 的项目中添加对应的Kernel Memory的Semantic Kernel Plugin包。
```undefined
dotnet add package Microsoft.KernelMemory.SemanticKernelPlugin
```
然后就取出构建KernelMemory，构建的步骤和之前的方式一样，也可以使用Kernel Memory Service的 WebClient.
```undefined
var memory = new KernelMemoryBuilder()
               //...
                 .Build<MemoryServerless>();
// OR
 
var memory = new MemoryWebClient("http://127.0.0.1:9001");
```
完成构建之后就可以Semantic Kernel 中导入插件了。
```undefined
var memoryPlugin = kernel.ImportPluginFromObject(new MemoryPlugin(memory), "memory");
```
如果直接使用Kernel Memory Service的话，也可以通过快捷方式直接构建。
```undefined
var memoryPlugin = kernel.ImportPluginFromObject(new MemoryPlugin("http://127.0.0.1:9001"), "memory");
```
如果有使用其中的ApiKey参数的话，也可以参考其中的参数说明进行配置。

另外还可以指定默认的 Index 和文档的 tags。
```undefined
new MemoryPlugin(memory,
        defaultIndex: "default",
        defaultRetrievalTags: 
            new TagCollection()
            {
                {"user", "xbotter"}
            }
        );
```
## 插件的功能
MemoryPlugin 本身提供了近乎完整的Kernel Memory的功能，包括：
- Save
- SaveFile
- SaveWebPage
- Search
- Ask
- Delete
- WaitForDocumentReadiness

也就意味着如果在Semantic Kernel 中，通过插件功能的编排，可以实现动态的知识库管理。

当然也可以通过 plugin 直接调用其中的方法例如：
```undefined
    var context = new KernelArguments
    {
        [MemoryPlugin.FilePathParam] = DocFilename,
        [MemoryPlugin.DocumentIdParam] = "NASA001"
    };
    await memoryPlugin["SaveFile"].InvokeAsync(kernel, context);
```
## 插件的使用
导入到Semantic Kernel 中的插件可以通过在提示词函数中直接使用，最常见的就是在提示词中使用Ask方法和Search方法。
```undefined
var skPrompt = """
                Question to Kernel Memory: {{$input}}
 
                Kernel Memory Answer: {{memory.ask $input}}
 
                If the answer is empty say 'I don't know' otherwise reply with a preview of the answer, truncated to 15 words.
               """;
 
var askFunction = kernel.CreateFunctionFromPrompt(skPrompt);
```
和直接使用 MemoryClient 一样，调用 memory.ask 方法可以直接获取 Kernel Memory的回答，构成提示词的一部分。

如果只是需要检索相关的文档的话，可以使用Search方法。
```undefined
var skPrompt = """
                Facts:
                {{memory.search $input}}
                ======
                Given only the facts above, provide a comprehensive/detailed answer.
                You don't know where the knowledge comes from, just answer.
                If you don't have sufficient information, reply with 'I don't know'.
                Question: {{$input}}
                Answer: 
                """;
var searchFunction = kernel.CreateFunctionFromPrompt(skPrompt);
```
这里其实就相当于是使用 PromptFunction 替换了 KernelMemory中的Ask方法。

另外其中的一些参数都可以通过KernelArguments进行配置，例如：
```undefined
var args = new KernelArguments
{
    [MemoryPlugin.IndexParam ] = "default",
    [MemoryPlugin.LimitParam] = "10",
    [MemoryPlugin.MinRelevanceParam] = "0"
};
```
最后就是Kernel Function 的调用了：
```undefined
var question = "any question";
var answer = await askFunction.InvokeAsync(kernel, question);
 
 // or 
 
args["input"] = question;
var result = await kernel.InvokeAsync(searchFunction,args);

```
