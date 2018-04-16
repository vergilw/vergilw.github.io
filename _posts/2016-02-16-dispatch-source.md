---
layout: post
title: Dispatch Source
date: 2016-02-16 16:32:24.000000000 +09:00
---

## 关于Dispatch Source

Dispatch Source是GCD中的一种基本数据类型，从字面意思可称其为调度源，它用于处理特定的系统底层事件，即：当一些特定的系统底层事件发生时，调度源会捕捉到这些事件，然后可以做相应的逻辑处理。

Dispatch Source可用来监听以下几类事件：

* Timer Dispatch Source：定时调度源。
* Signal Dispatch Source：监听UNIX信号调度源，比如监听代表挂起指令的SIGSTOP信号。
* Descriptor Dispatch Source：监听文件相关操作和Socket相关操作的调度源。
* Process Dispatch Source：监听进程相关状态的调度源。
* Mach port Dispatch Source：监听Mach相关事件的调度源。
* Custom Dispatch Source：监听自定义事件的调度源。

使用Dispatch Source时，通常是先指定一个希望监听的系统事件类型，再指定一个捕获到事件后进行逻辑处理的闭包或者函数作为回调函数，然后再指定一个该回调函数执行的Dispatch Queue即可。当监听到指定的系统事件发生时，Dispatch Source会将已指定的回调函数作为一个任务放入指定的队列中执行，也就是说当监听到系统事件后就会触发一个任务，并自动将其加入队列执行。  
这里与通常的手动添加任务的模式不同，一旦将Diaptach Source与Dispatch Queue关联后，只要监听到系统事件，Dispatch Source就会自动将任务（回调函数）添加到关联的队列中，直到我们调用函数取消监听。

为了保证监听到事件后回调函数能够都到执行，已关联的Dispatch Queue会被Diaptach Source强引用。

有些时候回调函数执行的时间较长，在这段时间内Dispatch Source又监听到多个系统事件，理论上就会形成事件积压，但好在Dispatch Source有很好的机制解决这个问题，当有多个事件积压时会根据事件类型，将它们进行关联和结合，形成一个新的事件。

## Dispatch Source类型

Dispatch Source一共可以监听六类事件，根据同类事件的不同操作，Dispatch Source分为11个类型：

* DISPATCH_SOURCE_TYPE_DATA_ADD：属于自定义事件，可以通过dispatch_source_get_data函数获取事件变量数据，在我们自定义的方法中可以调用dispatch_source_merge_data函数向Dispatch Source设置数据，下文中会有详细的演示。
* DISPATCH_SOURCE_TYPE_DATA_OR：属于自定义事件，用法同上面的类型一样。
* DISPATCH_SOURCE_TYPE_MACH_SEND：Mach端口发送事件。
* DISPATCH_SOURCE_TYPE_MACH_RECV：Mach端口接收事件。
* DISPATCH_SOURCE_TYPE_PROC：与进程相关的事件。
* DISPATCH_SOURCE_TYPE_READ：读文件事件。
* DISPATCH_SOURCE_TYPE_WRITE：写文件事件。
* DISPATCH_SOURCE_TYPE_VNODE：文件属性更改事件。
* DISPATCH_SOURCE_TYPE_SIGNAL：接收信号事件。
* DISPATCH_SOURCE_TYPE_TIMER：定时器事件。
* DISPATCH_SOURCE_TYPE_MEMORYPRESSURE：内存压力事件。

## 创建Dispatch Source

使用dispatch_source_create函数创建Dispatch Source，函数原型如下：

```objc
dispatch_source_t dispatch_source_create(dispatch_source_type_t type,
										 uintptr_t handle,
										 unsigned long mask,
										 dispatch_queue_t _Nullable queue);
```

dispatch_source_create函数有四个参数：

* type：指定Dispatch Source类型，共有11个类型，特定的类型监听特定的事件。
* handle：取决于要监听的事件类型，比如如果是监听Mach端口相关的事件，那么该参数就是mach_port_t类型的Mach端口号，如果是监听事件变量数据类型的事件那么该参数就不需要，设置为0就可以了。
* mask：取决于要监听的事件类型，比如如果是监听文件属性更改的事件，那么该参数就标识文件的哪个属性，比如DISPATCH_VNODE_RENAME  
。
* queue：设置回调函数所在的队列。

## 
设置事件处理器

前文中提到过，当Dispatch Source监听到事件时会调用指定的回调函数或闭包，该回调函数或闭包就是Dispatch Source的事件处理器。我们可以使用dispatch_source_set_event_handler或dispatch_source_set_event_handler_f函数给创建好的Dispatch Source设置处理器，前者是设置闭包形式的处理器，后者是设置函数形式的处理器：

既然是事件处理器，那么肯定需要获取一些Dispatch Source的信息，GCD提供了三个在处理器中获取Dispatch Source相关信息的函数，比如handle、mask。而且针对不同类型的Dispatch Source，这三个函数返回数据的值和类型都会不一样，下面来看看这三个函数：

* dispatch_source_get_handle：这个函数用于获取在创建Dispatch Source时设置的第二个参数handle。 
    * 如果是读写文件的Dispatch Source，返回的就是描述符。
    * 如果是信号类型的Dispatch Source，返回的是int类型的信号数。
    * 如果是进程类型的Dispatch Source，返回的是pid_t类型的进程id。
    * 如果是Mach端口类型的Dispatch Source，返回的是mach_port_t类型的Mach端口。
* dispatch_source_get_data：该函数用于获取Dispatch Source监听到事件的相关数据。 
    * 如果是读文件类型的Dispatch Source，返回的是读到文件内容的字节数。
    * 如果是写文件类型的Dispatch Source，返回的是文件是否可写的标识符，正数表示可写，负数表示不可写。
    * 如果是监听文件属性更改类型的Dispatch Source，返回的是监听到的有更改的文件属性，用常量表示，比如DISPATCH_VNODE_RENAME等。
    * 如果是进程类型的Dispatch Source，返回监听到的进程状态，用常量表示，比如DISPATCH_PROC_EXIT等。
    * 如果是Mach端口类型的Dispatch Source，返回Mach端口的状态，用常量表示，比如DISPATCH_MACH_SEND_DEAD等。
    * 如果是自定义事件类型的Dispatch Source，返回使用dispatch_source_merge_data函数设置的数据。
* dispatch_source_get_mask：该函数用于获取在创建Dispatch Source时设置的第三个参数mask。在进程类型，文件属性更改类型，Mach端口类型的Dispatch Source下该函数返回的结果与dispatch_source_get_data一样。

## 设置取消处理器

取消处理器就是当Dispatch Source被释放时用来处理一些后续事情，比如关闭文件描述符或者释放Mach端口等。我们可以使用dispatch_source_set_cancel_handler函数或者dispatch_source_set_cancel_handler_f函数给Dispatch Source注册取消处理器。

## 更改目标队列

在上文中，我们说过可以使用dispatch_source_create函数创建Dispatch Source，并且在创建时会指定回调函数执行的队列，那么如果事后想更改队列，比如说想更改队列的优先级，这时我们可以使用dispatch_set_target_queue函数实现。
    
    
    这里需要注意的是，如果在更改目标队列时，Dispatch Source已经监听到相关事件，并且回调函数已经在之前的队列中执行了，那么会一直在旧的队列中执行完成，不会转移到新的队列中去。
    

## 恢复与暂停Dispatch Source

暂停Dispatch Source使用dispatch_suspend函数；恢复Dispatch Source使用dispatch_resume函数。
    
    
    需要注意的是，因为Dispatch Source创建之后，需要进行一些配置，比如设置事件处理器等，所以刚创建好的Dispatch Source是处于暂停状态的，因此使用时需要用dispatch_resume函数将其启动。
    

## 废弃Dispatch Source

如果我们不再需要使用某个Dispatch Source时，可以使用dispatch_source_cancel函数废除，该函数只有一个参数，那就是目标Dispatch Source。

## 关联用户数据

在设置事件处理器中已提到获取Dispatch Source信息的几个函数，但是对于自定义事件类型的Dispatch Source，dispatch_source_merge_data函数设置的数据为unsigned long类型，因而通过dispatch_source_get_data获取的数据也只支持unsigned long类型，严重影响了自定义类型事件的应用范围。  
对于这一点，可以通过给Dispatch Source关联用户数据的来解决：
    
    
    void dispatch_set_context(dispatch_object_t object, void *context);
    

这个函数可以给自定义事件类型的Dispatch Source关联任何我们需要的数据，当执行事件处理器时，调用dispatch_get_context就能获取最近一次关联的数据。
    
    
    注意，如果关联了用户数据，那么不再需要这个数据时就应当及时释放。
    

## Dispatch Source实践

### 定时器

使用定时器时需要调用 dispatch_source_set_timer函数来配置定时器，这个函数有四个参数：

* source：待配置的定时器类型的 Dispatch Source
* start：控制定时器第一次触发的时刻。参数类型是 dispatch_time_t，这是一个opaque类型，我们不能直接操作它。我们得需要 dispatch_time 和 dispatch_walltime 函数来创建它们。另外，常量 DISPATCH_TIME_NOW 和 DISPATCH_TIME_FOREVER 通常很有用。
* interval：触发间隔
* leeway：定时器进度，单位纳秒；如果设为0，系统只是最大程度满足精度需求。精度越高功耗越大。
    
```objc
- (void)timerDispatchSource
{
    dispatch_source_t timerSource = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER,0, 0, dispatch_get_global_queue(0, 0));
    
    if (timerSource)
    {
        dispatch_time_t startTime = dispatch_time(DISPATCH_TIME_NOW, 0 * NSEC_PER_SEC);  //1, the timer dispatch source uses the default system clock to determine when to fire. However, the default clock does not advance while the computer is asleep.
//        dispatch_time_t startTime = dispatch_walltime(NULL, 0 * NSEC_PER_SEC); //2  the timer dispatch source tracks its firing time to the wall clock time
        
        NSString *desc = timerSource.description;
        dispatch_source_set_timer(timerSource, startTime, 1 * NSEC_PER_SEC, 0);
        dispatch_source_set_event_handler(timerSource, ^{
            static NSInteger i = 0;
            ++i;
            NSLog(@"Timer %@ Task: %ld",desc,i);
//            NSLog(@"Timer %@ Task: %ld",timerSource,i);
            
            
        });
        dispatch_source_set_cancel_handler(timerSource, ^{
//            NSLog(@"Timer：%@ canceled",timerSource);
            NSLog(@"Timer:%@ canceled",desc);
        });
        dispatch_resume(timerSource);
    }
    
    _myTimerSource = timerSource; ///< 必须要保存，除非在hander中引用timerSource，否则出了作用域，Timer就会被释放
}
```            


NSTimer 与 GCD Timer比较  
NSTimer

* 依赖NSRunloop
* 容易导致内存泄漏
* NSTimer的创建与撤销必须在同一个线程操作、 performSelector的创建与撤销必须在同一个线程操作

GCD Timer

* 可以被当做对象放入数组或字典中
* GCD Timer必须强引用，否则出了栈就会失效，这种失效不会触发取消处理器
* GCD Timer精度可控
* 如果使用dispatch_walltime来设置定时器的起始时间，定时器默认使用walltime来触发定时器；如果使用dispatch_time来设置定时器的起始时间，定时器默认使用系统时钟来触发定时器，然而当计算机休眠时，系统时钟也是休眠的。对于时间间隔比较大的定时器，使用dispatch_walltime来设置定时器的起始时间

### 监听信号
    

```objc
- (void)signalDispatchSource
{
    signal(SIGCHLD, SIG_IGN);
    
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_source_t signalSource = dispatch_source_create(DISPATCH_SOURCE_TYPE_SIGNAL, SIGCHLD, 0, queue);
    
    if (signalSource)
    {
        dispatch_source_set_event_handler(signalSource, ^{
            static NSInteger i = 0;
            ++i;
            NSLog(@"Signal Detected: %ld",i);
        });
        dispatch_source_set_cancel_handler(signalSource, ^{
            NSLog(@"Signal canceled");
        });
    
        dispatch_resume(signalSource);
    }
    
    _mySignalSource = signalSource; // 不能省，原因同定时器
}
```
    
    
    注意
    * SIGILL, SIGBUS,  SIGSEGV不能监听
    * 只是监听信号，并不处理信号
    

### 读写文件
    
```objc
- (void)writeDispatchSource
{
    NSString *filePath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) objectAtIndex:0];
    NSString *fileName = [filePath stringByAppendingString:@"/test.txt"];
    int fd = open([fileName UTF8String], O_WRONLY | O_CREAT | O_TRUNC,
                  (S_IRUSR | S_IWUSR | S_ISUID | S_ISGID));
    NSLog(@"Write fd:%d",fd);
    if (fd == -1)
        return ;
    fcntl(fd, F_SETFL); // Block during the write.
    
    dispatch_source_t writeSource = nil;
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    writeSource = dispatch_source_create(DISPATCH_SOURCE_TYPE_WRITE,fd, 0, queue);
    
    dispatch_source_set_event_handler(writeSource, ^{
        size_t bufferSize = 100;
        void *buffer = malloc(bufferSize);
        
        static NSString *content = @"Write Data Action: ";
        content = [content stringByAppendingString:@"=New info="];
        
        NSString *writeContent = [content stringByAppendingString:@"\n"];
        void *string = [writeContent UTF8String];
        size_t actual = strlen(string);
        memcpy(buffer, string, actual);
        
        write(fd, buffer, actual);
        NSLog(@"Write to file Finished");
        
        free(buffer);
        // Cancel and release the dispatch source when done.
        //        dispatch_source_cancel(writeSource);
        dispatch_suspend(writeSource);  //不能省,否则只要文件可写，写操作会一直进行，直到磁盘满，本例中，只要超过buffer容量就会崩溃
//        close(fd);   //会崩溃
    });
    dispatch_source_set_cancel_handler(writeSource, ^{
        NSLog(@"Write to file Canceled");
        close(fd);
    });

    if (!writeSource)
    {
        close(fd);
        return;
    }

    _myWriteSource = writeSource;
}

- (void)readDataDispatchSource
{
    if (_myReadSource)
    {
        dispatch_source_cancel(_myReadSource);
    }
    
    NSString *filePath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) objectAtIndex:0];
    NSString *fileName = [filePath stringByAppendingString:@"/test.txt"];
    // Prepare the file for reading.
    int fd = open([fileName UTF8String], O_RDONLY);
    NSLog(@"read fd:%d",fd);
    if (fd == -1)
        return ;
    fcntl(fd, F_SETFL, O_NONBLOCK);  // Avoid blocking the read operation
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_source_t readSource = dispatch_source_create(DISPATCH_SOURCE_TYPE_READ, fd, 0, dispatch_get_main_queue());
    if (!readSource)
    {
        close(fd);
        return ;
    }
    
    
    // Install the event handler
    //只要文件写入了新内容，就会自动读入新内容
    dispatch_source_set_event_handler(readSource, ^{
        long estimated = dispatch_source_get_data(readSource);
        NSLog(@"Read From File, estimated length: %ld",estimated);
        if (estimated < 0)
        {
            NSLog(@"Read Error:");
            dispatch_source_cancel(readSource);  //如果文件发生了截短，事件处理器会一直不停地重复
        }
        
        // Read the data into a text buffer.
        char *buffer = (char *)malloc(estimated);
        if (buffer)
        {
            ssize_t actual = read(fd, buffer, (estimated));
            NSLog(@"Read From File, actual length: %ld",actual);
            NSLog(@"Readed Data: \n%s",buffer);
//            Boolean done = MyProcessFileData(buffer, actual);  // Process the data.
            
            // Release the buffer when done.
            free(buffer);
            
            // If there is no more data, cancel the source.
//            if (done)
//                dispatch_source_cancel(readSource);
        }
    });
    
    // Install the cancellation handler
    dispatch_source_set_cancel_handler(readSource, ^{
        NSLog(@"Read from file Canceled");
        close(fd);
    });
    
    // Start reading the file.
    dispatch_resume(readSource);
    
    _myReadSource = readSource; //can be omitted
}
```
    
    
    注意：
    * 确保非阻塞方式进行读写，否则当读写出错的时候，会导致线程阻塞
    

### 自定义事件

以更新progressView进度为例，定时器每0.05触发一次，随机更新progressView的进度
    
```objc
- (void)customDispatchSourceForProgressView
{
    static dispatch_source_t timerSource = nil;
    static dispatch_source_t source = nil;
    
    _progressView.progress = 0;
    if (timerSource)
    {
        dispatch_source_cancel(timerSource);
        timerSource = nil;
        source = nil;
    }
    else
    {
        source =dispatch_source_create(DISPATCH_SOURCE_TYPE_DATA_ADD, 0, 0, dispatch_get_main_queue());
        dispatch_resume(source);
        dispatch_source_set_event_handler(source, ^{
        // 方法1，获取dispatch_source_merge_data传入的值，只支持unsigned long类型
        // 如果出现合并，那么得到的值是合并的那几次提交传入的值的累加结果
            unsigned long data = dispatch_source_get_data(source);
            CGFloat accumulate = ((CGFloat)data)/100;
            CGFloat progress = _progressView.progress;
            progress += accumulate;
            
            //方法2，获取关联数据
//            char *c = dispatch_get_context(source);
//            NSLog(@"%c",*c);
//            NSNumber *num = (__bridge NSNumber *)(dispatch_get_context(source));
//            NSLog(@"%@",num);

            _progressView.progress = progress;
            if (progress >= 1)
            {
                timerSource = nil;
                source = nil;
            }
        });
        dispatch_async(dispatch_get_global_queue(0, 0), ^
                       {
                           timerSource = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER,0, 0, dispatch_get_global_queue(0, 0));
                           if (timerSource)
                           {
                               dispatch_time_t startTime = dispatch_time(DISPATCH_TIME_NOW, 0 * NSEC_PER_SEC);
                               
                               dispatch_source_set_timer(timerSource, startTime, 0.05 * NSEC_PER_SEC, 0);
                               dispatch_source_set_event_handler(timerSource, ^{
                                   
                                   //关联用户自定义信息
                                   int i = rand()%5;
//                                   char c = i + 65;
//                                   dispatch_set_context(source, &c);
//                                   NSNumber *num = @(i);
//                                   dispatch_set_context(source, (__bridge void * _Nullable)(num));
                                   
                                   dispatch_source_merge_data(source, i);
                               });
                               dispatch_resume(timerSource);
                           }
                       });
    }
}
```
    
    

## 参考

2. [苹果官方文档][2]
3. [选择 GCD 还是 NSTimer][3]
4. [iOS dispatch_source_t的理解][4]]([http://www.cnblogs.com/wjw-blog/p/5903441.html)][5])
5. [iOS_GCD_讲解三_Dispatch Sources][6]

[1]: https://www.jianshu.com#jumpid1
[2]: https://link.jianshu.com?t=https://developer.apple.com/library/content/documentation/General/Conceptual/ConcurrencyProgrammingGuide/GCDWorkQueues/GCDWorkQueues.html#//apple_ref/doc/uid/TP40008091-CH103-SW1
[3]: https://www.jianshu.com/p/0c050af6c5ee
[4]: https://link.jianshu.com?t=%5Bhttp://www.cnblogs.com/wjw-blog/p/5903441.html
[5]: https://link.jianshu.com?t=http://www.cnblogs.com/wjw-blog/p/5903441.html)
[6]: https://link.jianshu.com?t=http://www.dreamingwish.com/frontui/article/default/gcd%E4%BB%8B%E7%BB%8D%EF%BC%88%E4%B8%89%EF%BC%89-dispatch-sources.html

  
