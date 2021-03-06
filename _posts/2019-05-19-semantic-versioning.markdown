---
layout: post
title: Semantic Versioning
date: 2019-05-19
---

> 大部分开发平台都有包管理工具，这让我们可以很方便的管理第三方的工具集。由于第三方工具集也在不断更新迭代，为了保证第三方工具集的更新不会影响项目的稳定性，很多时候我们会使用包管理工具依赖指定的第三方工具版本，而不是最新版本。很多包管理工具使用的版本号命名方式遵从Semantic Versioning。所以无论是使用第三方工具，还是自己发布第三方工具，了解Semantic Versioning十分必要。

### Semantic Versioning介绍
Semantic Versioning版本号`X.Y.Z`主要由三部分组成： **`MAJOR.MINOR.PATCH`**

分别代表了主版本，次版本，补丁版本。Semantic Versioning提出了一套简单的规则和要求，规定如何分配和递增版本号。任何的第三方工具都会将自身的功能封装成一些接口方法供程序调用，所以公开的接口方法必须清晰准确。然后使用特定部分的版本号增加来传达更新的内容。

* **MAJOR主版本号增加**：当你的公共接口方法修改后不向下兼容 
* **MINOR次版本号增加**：当你新增了功能，公共接口方法可以向下兼容
* **PATCH补丁版本号增加**：当你修复了错误，公共接口方法可以向下兼容


### Semantic Versioning具体使用
我们可能会想当然的将新开发的第三方工具第一个版本定义为1.0.0，但是推荐的做法是0.1.0。因为我们没法保证我们刚开始就设计了一个完美且稳定的第三方工具，通过版本号的方式可以明确的传达给使用者，我们的公共接口方法可能不会那么稳定，当前正处于频繁迭代的初期。在1.0.0之前当我们对公共接口方法进行了修改且不向下兼容时，仅增加MINOR次版本号。当我们经过了多个初期版本迭代，进行大量测试后，终于有了一个稳定的公共接口方法。这时我们可以改为1.0.0，告诉使用者可以放心的依赖我们的工具，因为我们已经非常稳定了。所以说在1.0.0版本以前MINOR和PATCH的含义会有不同。


### 包管理工具
很多包管理工具都针对Semantic Versioning做了处理，例如FlutterPackages里的YAML文件dependencies引入了`^`符号。当你指定第三方工具版本号`^1.2.3`等同于>=1.2.3且<2.0.0，当你指定版本号`^0.1.2`等同于>=0.1.2且<0.2.0。这符合上面所介绍的规则。包管理工具这样的处理极大的方便了我们的第三发工具集的管理，能保证第三方工具集的更新不会影响项目的稳定性。


#### **参考文档**
* [Semantic Versioning 2.0.0规范](https://semver.org/)
* [Dart语言版本管理](https://dart.dev/tools/pub/dependencies#version-constraints)

