---
layout: post
title: Flutter 线程管理
date: 2019-06-16
---

### Flutter线程管理简述
Flutter Engine自己不创建管理线程。Flutter Engine线程的创建和管理是由embedder负责的。
注意:Embeder是指将引擎移植的平台的中间层代码。

Flutter Engine要求Embeder提供四个Task Runner。尽管Flutter Engine不在乎Runner具体跑在哪个线程，但是它需要线程配置在整一个生命周期里面保持稳定。也就是说一个Runner最好始终保持在同一线程运行。这四个主要的Task Runner包括：
![Flutter Framework](/assets/images/flutter_framework.png)

### Platform Task Runner
Flutter Engine的主Task Runner，运行Platform Task Runner的线程可以理解为是主线程。类似于Android Main Thread或者iOS的Main Thread。但是我们要注意Platform Task Runner和iOS之类的主线程还是有区别的。



对于Flutter Engine来说Platform Runner所在的线程跟其它线程并没有实质上的区别，只不过我们人为赋予它特定的含义便于理解区分。实际上我们可以同时启动多个Engine实例，每个Engine对应一个Platform Runner，每个Runner跑在各自的线程里。这也是Fuchsia（Google正在开发的操作引擎）里Content Handler的工作原理。一般来说，一个Flutter应用启动的时候会创建一个Engine实例，Engine创建的时候会创建一个线程供Platform Runner使用。



跟Flutter Engine的所有交互（接口调用）必须发生在Platform Thread，试图在其它线程中调用Flutter Engine会导致无法预期的异常。这跟iOS UI相关的操作都必须在主线程进行相类似。需要注意的是在Flutter Engine中有很多模块都是非线程安全的。一旦引擎正常启动运行起来，所有引擎API调用都将在Platform Thread里发生。



Platform Runner所在的Thread不仅仅处理与Engine交互，它还处理来自平台的消息。这样的处理比较方便的，因为几乎所有引擎的调用都只有在Platform Thread进行才能是安全的，Native Plugins不必要做额外的线程操作就可以保证操作能够在Platform Thread进行。如果Plugin自己启动了额外的线程，那么它需要负责将返回结果派发回Platform Thread以便Dart能够安全地处理。规则很简单，对于Flutter Engine的接口调用都需保证在Platform Thread进行。

需要注意的是，阻塞Platform Thread不会直接导致Flutter应用的卡顿（跟iOS android主线程不同）。尽管如此，平台对Platform Thread还是有强制执行限制。所以建议复杂计算逻辑操作不要放在Platform Thread而是放在其它线程（不包括我们现在讨论的这个四个线程）。其他线程处理完毕后将结果转发回Platform Thread。长时间卡住Platform Thread应用有可能会被系统Watchdot强行杀死。

### UI Task Runner Thread（Dart Runner）

UI Task Runner被Flutter Engine用于执行Dart root isolate代码（isolate我们后面会讲到，姑且先简单理解为Dart VM里面的线程）。Root isolate比较特殊，它绑定了不少Flutter需要的函数方法。Root isolate运行应用的main code。引擎启动的时候为其增加了必要的绑定，使其具备调度提交渲染帧的能力。对于每一帧，引擎要做的事情有：

Root isolate通知Flutter Engine有帧需要渲染。

Flutter Engine通知平台，需要在下一个vsync的时候得到通知。

平台等待下一个vsync

对创建的对象和Widgets进行Layout并生成一个Layer Tree，这个Tree马上被提交给Flutter Engine。当前阶段没有进行任何光栅化，这个步骤仅是生成了对需要绘制内容的描述。

创建或者更新Tree，这个Tree包含了用于屏幕上显示Widgets的语义信息。这个东西主要用于平台相关的辅助Accessibility元素的配置和渲染。


除了渲染相关逻辑之外Root Isolate还是处理来自Native Plugins的消息响应，Timers，Microtasks和异步IO。
我们看到Root Isolate负责创建管理的Layer Tree最终决定什么内容要绘制到屏幕上。因此这个线程的过载会直接导致卡顿掉帧。
如果确实有无法避免的繁重计算，建议将其放到独立的Isolate去执行，比如使用compute关键字或者放到非Root Isolate，这样可以避免应用UI卡顿。但是需要注意的是非Root Isolate缺少Flutter引擎需要的一些函数绑定，你无法在这个Isolate直接与Flutter Engine交互。所以只在需要大量计算的时候采用独立Isolate。



### GPU Task Runner
GPU Task Runner被用于执行设备GPU的相关调用。UI Task Runner创建的Layer Tree信息是平台不相关，也就是说Layer Tree提供了绘制所需要的信息，具体如何实现绘制取决于具体平台和方式，可以是OpenGL，Vulkan，软件绘制或者其他Skia配置的绘图实现。GPU Task Runner中的模块负责将Layer Tree提供的信息转化为实际的GPU指令。GPU Task Runner同时也负责配置管理每一帧绘制所需要的GPU资源，这包括平台Framebuffer的创建，Surface生命周期管理，保证Texture和Buffers在绘制的时候是可用的。



基于Layer Tree的处理时长和GPU帧显示到屏幕的耗时，GPU Task Runner可能会延迟下一帧在UI Task Runner的调度。一般来说UI Runner和GPU Runner跑在不同的线程。存在这种可能，UI Runner在已经准备好了下一帧的情况下，GPU Runner却还正在向GPU提交上一帧。这种延迟调度机制确保不让UI Runner分配过多的任务给GPU Runner。



前面我们提到GPU Runner可以导致UI Runner的帧调度的延迟，GPU Runner的过载会导致Flutter应用的卡顿。一般来说用户没有机会向GPU Runner直接提交任务，因为平台和Dart代码都无法跑进GPU Runner。但是Embeder还是可以向GPU Runner提交任务的。因此建议为每一个Engine实例都新建一个专用的GPU Runner线程。



### IO Task Runner
前面讨论的几个Runner对于执行任务的类型都有比较强的限制。Platform Runner过载可能导致系统WatchDog强杀，UI和GPU Runner过载则可能导致Flutter应用的卡顿。但是GPU线程有一些必要操作是比较耗时间的，比如IO，而这些操作正是IO Runner需要处理的。



IO Runner的主要功能是从图片存储（比如磁盘）中读取压缩的图片格式，将图片数据进行处理为GPU Runner的渲染做好准备。在Texture的准备过程中，IO Runner首先要读取压缩的图片二进制数据（比如PNG，JPEG），将其解压转换成GPU能够处理的格式然后将数据上传到GPU。这些复杂操作如果跑在GPU线程的话会导致Flutter应用UI卡顿。但是只有GPU Runner能够访问GPU，所以IO Runner模块在引擎启动的时候配置了一个特殊的Context，这个Context跟GPU Runner使用的Context在同一个ShareGroup。事实上图片数据的读取和解压是可以放到一个线程池里面去做的，但是这个Context的访问只能在特定线程才能保证安全。这也是为什么需要有一个专门的Runner来处理IO任务的原因。获取诸如ui.Image这样的资源只有通过async call，当这个调用发生的时候Flutter Framework告诉IO Runner进行刚刚提到的那些图片异步操作。这样GPU Runner可以使用IO Runner准备好的图片数据而不用进行额外的操作。

用户操作，无论是Dart Code还是Native Plugins都是没有办法直接访问IO Runner。尽管Embeder可以将一些一般复杂任务调度到IO Runner，这不会直接导致Flutter应用卡顿，但是可能会导致图片和其它一些资源加载的延迟间接影响性能。所以建议为IO Runner创建一个专用的线程。

### 各个平台目前默认Runner线程实现

前面我们提到Engine Runner的线程可以按照实际情况进行配置，各个平台目前有自己的实现策略。

#### iOS和Android
Mobile平台上面每一个Engine实例启动的时候会为UI，GPU，IO Runner各自创建一个新的线程。所有Engine实例共享同一个Platform Runner和线程。

#### Fuchsia
每一个Engine实例都为UI，GPU，IO，Platform Runner创建各自新的线程。
