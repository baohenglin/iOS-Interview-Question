## 知识点 RunLoop

**【扩展 1-1】RunLoop 是什么？RunLoop 的作用是什么？**

RunLoop 顾名思义也就是**运行循环**，在程序运行过程中循环执行某些任务。

RunLoop是一种让线程能随时处理事件但不退出的机制。RunLoop实际上是一个对象，这个对象管理着其需要处理的事件和消息，并提供了一个入口函数来执行 Event Loop 的逻辑。线程执行了这个函数后，就会一直处于这个函数内部“接收消息->等待消息->处理消息”的循环中，直到这个循环结束(比如传入quit消息)，函数返回。RunLoop 机制会使线程在没有消息需要处理的时候休眠，在有消息到来时，线程立刻被唤醒。这样有效节约了系统资源。

OS X 和 iOS 系统中，提供了两个对象：CFRunLoopRef 和 NSRunLoop。其中 CFRunLoopRef 是在 CoreFoundation 框架内的，它提供了纯 C 函数的 API，所有这些 API 都是线程安全的。NSRunLoop 是基于 CFRunLoopRef 的封装，提供了面向对象的 API，但是这些 API 不是线程安全的。

线程和 RunLoop 之间是⼀一对应的，其关系是保存在一个全局的 Dictionary 里。线程刚创建时并没有 RunLoop，如果你不主动获取，那它一直都不会有。RunLoop 的创建是发生在第一次获取时，RunLoop 的销毁是发生在线程结束时。你只能在一个线程的内部获取其 RunLoop(主线程除外)。

**RunLoop 的作用：**

* 保持程序的持续运行；
* 处理 App 中的各种事件（比如触摸事件、定时器事件等）；
* 节省 CPU 资源，提高程序性能：有待执行任务时执行任务，不执行任务时休眠。

**【扩展 1-2】RunLoop和线程的关系是怎样的？**

RunLoop和线程的关系如下：

* 每条线程都有唯一的一个与之对应的RunLoop对象。
* RunLoop保存在一个全局的Dictionary里，线程作为key，RunLoop作为value。
* 线程刚创建时并没有RunLoop对象，RunLoop会在第一次获取该线程时创建与之对应的RunLoop对象。
* RunLoop会在线程结束时销毁。
* 主线程的RunLoop已经自动获取（创建），子线程默认没有开启RunLoop。

**【扩展 1-3】程序中添加每3秒响应一次的NSTimer，当拖动tableView时，timer可能无法响应要怎么解决？**

NSTimer在滑动时失效的原因是NSTimer默认是工作在NSDefaultRunLoopMode模式下，而当我们滑动时，RunLoop会退出NSDefaultRunLoopMode模式，并进入UITrackingRunLoopMode模式，所有NSTimer失效。

【注意】[NSTimer scheduledTimerWithTimeInterval: repeats:block:]方法会自动将定时器添加到主线程的NSDefaultRunLoopMode模式下，如果要自定义RunLoop模式的话，可以使用timerWithTimeInterval方法创建定时器对象，并将定时器添加到当前线程的NSRunLoopCommonModes模式下(实际上是将timer定时器添加到了NSRunLoopCommonModes 模式下的CFMutableSetRef _commonModeItems数组中)，这样就能解决timer失效的问题。代码如下：

```
NSTimer *timer = [NSTimer timerWithTimeInterval:self.completionDelay target:self selector:@selector(completionDelayTimerFired) userInfo:nil repeats:YES];

[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

**【扩展 1-4】RunLoop是怎么响应用户操作的，具体流程是什么样的？**

当用户点击屏幕时，RunLoop内部的Source1会捕捉到该触屏事件，并将该事件包装成事件队列eventQueue交给Source0中进行处理。

**【扩展 1-5】说说RunLoop的几种状态？**

```
kCFRunLoopEntry = (1UL << 0),   //即将进入RunLoop
kCFRunLoopBeforeTimers = (1UL << 1),//即将处理Timer
kCFRunLoopBeforeSources = (1UL << 2),//即将处理Source
kCFRunLoopBeforeWaiting = (1UL << 5),//即将进入休眠
kCFRunLoopAfterWaiting = (1UL << 6),//即将从休眠中唤醒
kCFRunLoopExit = (1UL << 7),//即将退出RunLoop
```

**【扩展 1-6】RunLoop的mode作用是什么？**

RunLoop常见的mode有2种：kCFRunLoopDefaultMode(NSDefaultRunLoopMode)和UITrackingRunLoopMode。kCFRunLoopDefaultMode是默认模式，通常主线程是在这个模式下运行；UITrackingRunLoopMode用于ScrollView追踪触摸滑动，保证界面滑动时不受其他mode影响。

mode的作用是将不同模式下的Source0/Source1/Timer/Observer隔离开来，互不影响，这样就提高了执行效率和滑动流畅性。

**【扩展 1-7】timer和RunLoop是怎样的关系？**

* 从底层数据结构来看，RunLoop的__CFRunLoop结构体中存储着 CFMutableSetRef _modes,_modes是一个类似数组的集合，其所储存的数据类型是CFRunLoopModeRef, CFRunLoopModeRef结构体中存放着CFMutableArrayRef _timers。此外，如果timer被设置为 kCFRunLoopCommonModes模式，那么timer也将被存放在 __CFRunLoop结构体中的 CFMutableSetRef _commonModeItems数组中。
* 从RunLoop的运行逻辑来讲，timer会通知Observers结束休眠，唤醒线程来处理timer消息。

**【扩展 1-8】RunLoop内部实现逻辑(实现原理)是怎样的？**

[RunLoop的运行逻辑](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E5%BC%80%E5%8F%91%E4%B9%8BRunLoop%E6%8E%A2%E7%A9%B6.md)

**【扩展 1-9】平时在项目中是如何使用 RunLoop 的？(RunLoop的应用场景)**

* (1)控制线程生命周期（线程保活）

**线程保活的应用场景**：频繁执行一个任务或者多个串行（非并发）任务，可以采用线程保活的方式。线程保活的方式比传统的“创建线程-销毁线程-再创建线程-再销毁...”更节省CPU资源且更高效。比如 AFNetworking 中后台网络请求就使用了线程保活这种技术。

[线程保活示例](https://github.com/baohenglin/RunLoopThreadLive)

* (2)解决NSTimer在滑动时停止工作(失效)的问题。

**NSTimer 在滑动时失效的原因**是 NSTimer 默认是工作在 NSDefaultRunLoopMode 模式下，而当我们滑动时，RunLoop会退出 NSDefaultRunLoopMode 模式，并进入 UITrackingRunLoopMode 模式，所有 NSTimer 失效。

解决方法：

```
NSTimer *timer = [NSTimer timerWithTimeInterval:self.completionDelay target:self selector:@selector(completionDelayTimerFired) userInfo:nil repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

* (3)监控应用卡顿
* (4)性能优化

**【扩展 1-10】RunLoop的底层数据结构是怎样的？**

[RunLoop的底层数据结构](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E5%BC%80%E5%8F%91%E4%B9%8BRunLoop%E6%8E%A2%E7%A9%B6.md)

**【扩展 1-11】RunLoop的作用是什么？它的内部的工作机制了解么？（最好结合线程和内存管理来说）**

**【扩展 1-12】RunLoop 的基本概念，它是怎么休眠的？(阿里)**

**【扩展 1-13】程序中添加了每3秒响应一次的NSTimer，当拖动tableView或者scrollView时，timer可能无法响应要怎么解决？**

NSTimer在滑动时失效的原因是NSTimer默认是工作在NSDefaultRunLoopMode(kCFRunLoopDefaultMode)模式下，而当我们滑动时，RunLoop会退出NSDefaultRunLoopMode模式，并进入UITrackingRunLoopMode模式(切换成UITrackingRunLoopMode模式是为了保证滑动的流畅)，导致NSTimer失效。

【注意】[NSTimer scheduledTimerWithTimeInterval: repeats:block:]方法会自动将定时器添加到主线程的NSDefaultRunLoopMode模式下，如果要自定义RunLoop模式的话，可以使用timerWithTimeInterval方法创建定时器对象，并将定时器添加到当前线程的**NSRunLoopCommonModes模式**下(实际上是将timer定时器添加到了NSRunLoopCommonModes 模式下的CFMutableSetRef _commonModeItems数组中)，这样就能解决timer失效的问题。代码如下：

```
NSTimer *timer = [NSTimer timerWithTimeInterval:self.completionDelay target:self selector:@selector(completionDelayTimerFired) userInfo:nil repeats:YES];

[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

**【扩展 1-14】利用 runloop 解释一下页面的渲染过程**

当我们调用 [UIView setNeedsDisplay] 时，这时会调用当前 View.layer 的 [view.layer setNeedsDisplay]方法。这等于给当前的 layer 打上了一个脏标记，而此时并没有直接进行绘制工作。而是会到当前的 Runloop 即将休眠，也就是 beforeWaiting 时才会进行绘制工作。紧接着会调用 [CALayer display]，进入到真正绘制的工作。CALayer 层会判断自己的 delegate 有没有实现异步绘制的代理方法 displayer:，这个代理方法是异步绘制的入口，如果没有实现这个方法，那么会继续进行系统绘制的流程，然后绘制结束。

CALayer 内部会创建一个 Backing Store，用来获取图形上下文。接下来会判断这个 layer 是否有 delegate。如果有的话，会调用 [layer.delegate drawLayer:inContext:]，并且会返回给我们 [UIView DrawRect:] 的回调，让我们在系统绘制的基础之上再做一些事情。如果没有 delegate，那么会调用 [CALayer drawInContext:]。以上两个分支，最终 CALayer 都会将位图提交到 Backing Store，最后提交给 GPU。至此绘制的过程结束。

**【扩展 1-15】performSelector:withObject:afterDelay:内部大概是怎么实现的？使用该方法时有什么注意事项？**

```
[self performSelector:(nonnull SEL) withObject:(nullable id) afterDelay:(NSTimeInterval)]; 
```

**实现原理**：创建一个定时器，在指定的时候结束后系统会利用 runtime 通过方法名（Selector本质上就是方法名）去方法列表中查找到对应的方法实现并调用。

**使用注意事项**：

* （1）调用 performSelector:withObject:afterDelay: 方法时，需要先判断希望调用的方法是否存在（通过 respondsToSelector: 方法判断）；
* （2）performSelector:withObject:afterDelay: 方法是异步方法，必须在主线程调用，在子线程调用不会执行。

[参考1](https://www.iteye.com/blog/grayheart-2289984)

[参考2](https://www.jianshu.com/p/7583ff0181c2)











