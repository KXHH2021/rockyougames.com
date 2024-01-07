---
title: Kernel Memory 入门系列：文档的管理
date: 2023-12-26 20:00:00
categories:
  - Kernel Memory
tags:
  - Document管理
  - SECURITY_FILTERS
description: Kernel Memory 入门系列： 文档的管理
cover: https://s2.loli.net/2024/01/07/NOUKQIl96mEnyuh.png
---
![image.png](https://s2.loli.net/2024/01/07/NOUKQIl96mEnyuh.png)
## Kernel Memory 入门系列： 文档的管理
在Quick Start中我们了解到如何快速直接地上传文档。但是实际中，往往会面临更多的问题，例如文档如何更新，如何划定查询范围等等。这里我们将详细介绍在Kernel Memory文档的管理。

## 使用Document管理一组文件
当我们需要批量上传一组文件的时候，可以使用Document来管理。
```undefined
var document = new Document();
document.AddFile("./sample-SK-Readme.pdf");
document.AddFile("./sample-KM-Readme.md");
await memory.ImportDocumentAsync(document);
```
其中Document 作为一个对象，可以将多个文件归结到一起，可以自行指定对应的DocumentId，如果不指定的话，会生成一个随机的DocumentId，这个DocumentId后续可以用来查询文档的处理状态或者用于更新删除文档。

后续的使用和管理，将会以Document为基本的文档单位进行管理。

## 使用Tag进行文档标记
当我们需要对上传的文档进行范围划定时，可以使用Tag来进行标记。Tag可以理解为一个文档的属性，可以充分的自定义，例如标记文档的类型、标记文档的来源、上传的用户、所属的项目、所属的领域等等。
```undefined
var document = new Document();
document.AddFile("./sample-SK-Readme.pdf");
document.AddTag("type", "pdf");
document.AddTag("domain", "llm");
document.AddTag("user", "xbotter");
await memory.ImportDocumentAsync(document);
```
如果导入的是单文件，或者文件流的话，可以通过另外一种方式来添加Tag。
```undefined
var tags = new TagCollection();
tags.Add("type", "pdf");
tags.Add("domain", "llm");
tags.Add("user", "xbotter");
await memory.ImportFileAsync("./sample-SK-Readme.pdf", tags: tags);
```
同样也适用于文本和网页的导入：
```undefined
var tags = new TagCollection();
await memory.ImportTextAsync("这是一段文本", tags: tags);
 
await memory.ImportUrlAsync("https://www.github.com", tags: tags);
```
## 检索时进行筛选

使用Tag标记的最大用途就是在检索时候进行范围的筛选，例如我们可以指定，仅在所有pdf文档范围内搜索：
```undefined
await memory.AskAsync("What's the SK?", filters: MemoryFilters.ByTag("type", "pdf"));
```
当然也可以指定文档进行筛选：
```undefined
await memory.AskAsync("What's the SK?", filters: MemoryFilters.ByDocument("documentId"));
```
## 复杂筛选条件

当我们需要复杂的筛选条件时，可以使用MemoryFilters来实现And和Or的组合方式。
MemoryFilters多次的添加ByTag条件，表示为And的关系。
```undefined
await memory.AskAsync("What's the SK?", filters: MemoryFilters.ByTag("type", "pdf")
                                                              .ByTag("domain", "llm"));
```
添加多个MemoryFilters，表示为Or的关系。
```undefined
await memory.AskAsync("What's the SK?", filters: new List<MemoryFilter>() {
                                                MemoryFilters.ByTag("type", "pdf"),
                                                MemoryFilters.ByTag("domain", "llm")
                                            });
```
## 更新文档

前面提及Document概念的时候已经提到，DocumentId用来指定一个文档，当我们需要更新文档的时候，可以直接指明DocumentId，然后上传新的文档即可。
```undefined
var document = new Document(docId);
document.AddFile("./sample-SK-Readme.pdf");
await memory.ImportDocumentAsync(document);
```
此时，Kernel Memory会自动将原有的文档进行替换，实现文档的更新。

## 删除文档

当我们需要删除文档的时候，可以使用DeleteDocumentAsync方法，指定DocumentId即可。
```undefined
await memory.DeleteDocumentAsync(docId);
```
## 使用Index进行隔离

上传文档和搜索的时候，另外一个需要指定的参数是index，index在向量存储中，可以理解为一个命名空间，可以用来隔离不同的文档，而且在检索的时候，也是无法跨index进行检索的。

当上传和检索文档时未指定index的时候，会使用默认的index。

