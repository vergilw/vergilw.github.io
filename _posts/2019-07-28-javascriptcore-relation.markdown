---
layout: post
title: iOS Framework JavaScriptCore
date: 2019-07-28
---

> JavaScriptCore 为原生编程语言 Objective-C、Swift 提供调用 JavaScript 程序的动态能力，还能为 JavaScript 提供原生能力来弥补前端所缺能力。  正是因为 JavaScriptCore 的这种桥梁作用，所以出现了很多使用 JavaScriptCore 开发 App 的框架 ，比如 React Native、Weex、小程序、WebView Hybird 等框架。

### JavaScriptCore
JavaScriptCore 框架主要由 JSVirtualMachine 、JSContext、JSValue 类组成。  

**JSVirturalMachine** 的作用，是为 JavaScript 代码的运行提供一个虚拟机环境。在同一时间内，JSVirtualMachine 只能执行一个线程。如果想要多个线程执行任务，你可以创建多个 JSVirtualMachine。每个 JSVirtualMachine 都有自己的 GC（Garbage Collector，垃圾回收器），以便进行内存管理，所以多个 JSVirtualMachine 之间的对象无法传递。

**JSContext** 是 JavaScript 运行环境的上下文，负责原生和 JavaScript 的数据传递。  

**JSValue** 是 JavaScript 的值对象，用来记录 JavaScript 的原始值，并提供进行原生值对象转换的接口方法。

SVirtualMachine、JSContext、JSValue 之间的关系，如下图所示：
![JavaScriptCore Relation](/assets/images/javascriptcore-relation.png)

可以看出，JSVirtualMachine 里包含了多个 JSContext， 同一个 JSContext 中又可以有多个 JSValue。

每个 JavaScriptCore 中的 JSVirtualMachine 对应着一个原生线程，同一个 JSVirtualMachine 中可以使用 JSValue 与原生线程通信，遵循的是 JSExport 协议：原生线程可以将类方法和属性提供给 JavaScriptCore 使用，JavaScriptCore 可以将 JSValue 提供给原生线程使用。

JSVirtualMachine 是一个抽象的 JavaScript 虚拟机，是提供给开发者进行开发的，而其核心的 JavaScriptCore 引擎则是一个真实的虚拟机，包含了虚拟机都有的解释器和运行时部分。其中，解释器主要用来将高级的脚本语言编译成字节码，运行时主要用来管理运行时的内存空间。当内存出现问题，需要调试内存问题时，你可以使用 JavaScriptCore 里的 Web Inspector，或者通过手动触发 Full GC 的方式来排查内存问题。


### JavaScriptCore 引擎
JavaScriptCore 引擎的组成 JavaScriptCore 内部是由 Parser、Interpreter、Compiler、GC 等部分组成，其中 Compiler 负责把字节码翻译成机器码，并进行优化。你可以点击这个链接，来查看 WebKit 官方对 JavaScriptCore 引擎的介绍。  JavaScriptCore 解释执行 JavaScript 代码的流程，可以分为两步。  第一步，由 Parser 进行词法分析、语法分析，生成字节码。  第二步，由 Interpreter 进行解释执行，解释执行的过程是先由 LLInt（Low Level Interpreter）来执行 Parser 生成的字节码，JavaScriptCore 会对运行频次高的函数或者循环进行优化。优化器有 Baseline JIT、DFG JIT、FTL JIT。对于多优化层级切换， JavaScriptCore 使用 OSR（On Stack Replacement）来管理。
