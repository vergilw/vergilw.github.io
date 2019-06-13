---
layout: post
title: Dart DevTools
date: 2019-06-09
---

### Dart DevTools是什么
DevTools是Dart和Flutter的一套性能工具。它目前处于预览版本。DevTools支持检查Flutter应用程序的UI布局和状态，诊断Flutter应用程序中的UI性能问题。也支持Flutter或Dart命令行应用程序的源级调试，以及查看有关正在运行的应用程序的常规日志和诊断信息。
它支持AndroidStudio、IntelliJ、VS Code和命令行。

#### Flutter Inspector
Flutter Inspector是可视化和探索Flutter Widget树的强大工具。 Flutter框架使用Widget作为从控件（文本，按钮，切换等）到布局（居中，填充，行，列等）的任何内容的核心构建块。检查器是可视化和探索Flutter这些Widget树的强大工具，可用于：
* 查看布局结构
* 诊断布局问题

#### Timeline View
Timeline View显示有关Flutter帧的信息。它由三部分组成，每个部分的粒度都在增加： 
* 帧渲染图表
* 帧事件图表
* CPU分析器
Timeline View还支持导入和导出。

#### Memory View
Memory View可让您查看isolate在给定时刻如何使用内存。使用Snapshot和Reset可以显示累加器计数。如果您怀疑应用程序正在泄漏内存或者有其他与内存分配有关的错误，则可以使用累加器来研究内存分配的速率。内存分析由四部分组成，每个部分的粒度都在增加：
* 内存总揽图表
* 事件时间线 
* 快照类
* 类实例

#### Debugger
DevTools包括一个完整的源代码级调试器，支持断点，步进和变量检查。 当您打开Debugger调试器选项卡时，您应该会看到屏幕左下角（在“脚本”区域下方）中列出的所有应用程序库，以及主要源区域中加载的应用程序的主要入口点的源。 要浏览更多应用程序源，可以滚动“脚本”区域并选择要显示的其他源文件。

#### Logging View
Logging View显示来自Dart运行时、应用程序框架（如Flutter）和应用程序级日志记录事件的事件。
默认情况下标准记录事件：
* Dart运行时垃圾回收事件
* Flutter框架事件，如创建帧
* 应用程序的stdout和stderr
* 应用程序自定义日志事件
