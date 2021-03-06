## 知识点 RunLoop

[RunLoop 参考链接1](https://www.jianshu.com/p/ac05ac8428ac)

**【扩展 1-1】RunLoop 是什么？RunLoop 的作用是什么？**

RunLoop 顾名思义也就是**运行循环**，在程序运行过程中循环执行某些任务。

RunLoop是一种让线程能随时处理事件但不退出的机制。RunLoop实际上是一个对象，这个对象管理着其需要处理的事件和消息，并提供了一个入口函数来执行 Event Loop 的逻辑。线程执行了这个函数后，就会一直处于这个函数内部“接收消息->等待消息->处理消息”的循环中，直到这个循环结束(比如传入quit消息)，函数返回。RunLoop 机制会使线程在没有消息需要处理的时候休眠，在有消息到来时，线程立刻被唤醒。这样有效节约了系统资源。

OS X 和 iOS 系统中，提供了两套 API 来访问和使用 RunLoop：CFRunLoopRef 和 NSRunLoop。其中 CFRunLoopRef 是在 Core Foundation 框架内的，它提供了纯 C 函数的 API，所有这些 API 都是线程安全的。NSRunLoop 是 Foundation 框架内的，是基于 CFRunLoopRef 的一层 OC 封装，提供了面向对象的 API，但是这些 API 不是线程安全的。

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
* 主线程的RunLoop已经自动获取（创建），子线程默认没有开启 RunLoop，需要手动创建开启。

**【扩展 1-3】如何获取 RunLoop 对象？**

获得**当前线程的 RunLoop 对象**，代码如下：

```
//方法1：Foundation
[NSRunLoop currentRunLoop];

//方法2： Core Foundation
CFRunLoopGetCurrent();
```

获得**主线程的 RunLoop 对象**，代码如下：

```
//方法1：Foundation
[NSRunLoop mainRunLoop];

//方法2： Core Foundation
CFRunLoopGetMain();
```


**【扩展 1-3】程序中添加每 3 秒响应一次的 NSTimer，当滑动 tableView 时，timer 可能无法响应，这该怎么解决？**

NSTimer 在滑动时失效的原因是 NSTimer 默认是工作在 NSDefaultRunLoopMode 模式下，而当我们滑动时，RunLoop 会退出 NSDefaultRunLoopMode 模式，并进入 UITrackingRunLoopMode 模式，所有NSTimer失效。

【注意】[NSTimer scheduledTimerWithTimeInterval: repeats:block:]方法会自动将定时器添加到主线程的 NSDefaultRunLoopMode 模式下，如果要自定义 RunLoop 模式的话，可以使用 timerWithTimeInterval 方法创建定时器对象，并将定时器添加到当前线程的 NSRunLoopCommonModes 模式下(实际上是将timer 定时器添加到了 NSRunLoopCommonModes 模式下的 CFMutableSetRef _commonModeItems数组中)，这样就能解决timer失效的问题。代码如下：

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

**mode的作用**是：

* 将不同模式下的Source0/Source1/Timer/Observer隔离开来，互不影响，这样就提高了执行效率和滑动流畅性。
* 指定事件在运行循环中的优先级。

**【扩展 1-7】timer和RunLoop是怎样的关系？**

* 从底层数据结构来看，RunLoop的__CFRunLoop结构体中存储着 CFMutableSetRef _modes,_modes是一个类似数组的集合，其所储存的数据类型是CFRunLoopModeRef, CFRunLoopModeRef结构体中存放着CFMutableArrayRef _timers。此外，如果timer被设置为 kCFRunLoopCommonModes模式，那么timer也将被存放在 __CFRunLoop结构体中的 CFMutableSetRef _commonModeItems数组中。
* 从RunLoop的运行逻辑来讲，timer会通知Observers结束休眠，唤醒线程来处理timer消息。

**【扩展 1-8】RunLoop内部实现逻辑(实现原理)是怎样的？**

[RunLoop的运行逻辑](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E5%BC%80%E5%8F%91%E4%B9%8BRunLoop%E6%8E%A2%E7%A9%B6.md)

**【扩展 1-9】RunLoop的应用场景有哪些？(平时在项目中是如何使用 RunLoop 的？)**

* (1)控制线程生命周期即**线程保活**（线程常驻）

**线程保活的应用场景**：在 iOS 开发中，有时一些耗时操作会阻塞主线程，导致界面卡顿，那么我们就会创建一个子线程，然后将耗时操作放在子线程中来处理。可是当子线程中的任务执行完毕后，子线程就会立刻被销毁掉。如果频繁执行一个任务或者多个串行（非并发）任务，这种情况下宜采用线程保活的方式。线程保活的方式比传统的“创建线程-销毁线程-再创建线程-再销毁...”更节省CPU资源且更高效。比如 AFNetworking 中后台网络请求就使用了线程保活这种技术。

[线程保活示例](https://github.com/baohenglin/RunLoopThreadLive)

* (2)**解决当滑动 UITableView 或 UIScrollView 时 NSTimer 停止工作(失效)的问题**

&emsp;&emsp;NSTimer 在滑动时失效的原因是 NSTimer 默认是工作在 NSDefaultRunLoopMode 模式下，而当我们滑动时，RunLoop 会退出NSDefaultRunLoopMode 模式，并进入 UITrackingRunLoopMode 模式，所有 NSTimer 失效。

【注意】[NSTimer scheduledTimerWithTimeInterval: repeats:block:]方法会自动将定时器添加到主线程的 NSDefaultRunLoopMode 模式下，如果要自定义 RunLoop 模式的话，可以改为使用 timerWithTimeInterval 方法创建定时器对象，并将定时器添加到当前线程的 NSRunLoopCommonModes 模式下(实际上是将timer 定时器添加到了 NSRunLoopCommonModes 模式下的 CFMutableSetRef _commonModeItems数组中)，这样就能解决timer失效的问题。代码如下：

```
NSTimer *timer = [NSTimer timerWithTimeInterval:self.completionDelay target:self selector:@selector(completionDelayTimerFired) userInfo:nil repeats:YES];

[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

* (3)**UIImageView 延迟加载图片**以便使 UITableView 或 UICollectionView 滑动更流畅

在 UITableView 的 cell 上显示网络图片，一般需要两步，第一步异步下载网络图片；第二步将下载好的网络图片更新到 UIImageView 上。在需要加载大量网络图片或图片很大的情况下，如果边滑动边加载显示网络图片，毫无疑问会造成界面卡顿（为什么会卡顿？首先需要明确的是这种卡顿并不是异步下载导致的（已经放到子线程中去下载了），而是主线程更新 UI 导致的。因为**解压缩和渲染**很多的大图片是一种非常耗时的操作，如果在滑动情况下，也就是在 UITrackingRunLoopMode 中加载（解压缩并渲染）这些图片，势必会造成卡顿）。面对这种问题，我们可以这样处理：**在滑动结束后再加载网络图片**，即**延迟加载图片**。那么“延迟加载图片”的这种解决方案具体如何实现呢？

我们可以利用 RunLoop 的 Mode 特性来实现。RunLoop 有五种 Mode，默认情况下处于 NSDefaultRunLoopMode，当滑动 tableView 时，会退出当前的 NSDefaultRunLoopMode，进入 UITrackingRunLoopMode，为了避免加载图片对页面的流畅性产生负面影响，可以通过 performSelector:whithObject:afterDelay:inModes 方法在主线程的 NSDefaultRunLoopMode 里为 UIImageView 设置图片（加载显示图片）。具体代码如下所示：

```
//还没弄清楚这块
UIImage *downloadedImage = ....;
[self.myImageView performSelector:@selector(setImage:) withObject:downloadedImage afterDelay:0 inModes:@[NSDefaultRunLoopMode]];
```
 
* (4)**监控应用卡顿**

[利用 RunLoop 监控界面卡顿详解](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS界面卡顿监测.md)

**【扩展 1-10】RunLoop的底层数据结构是怎样的？**

[RunLoop的底层数据结构](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E5%BC%80%E5%8F%91%E4%B9%8BRunLoop%E6%8E%A2%E7%A9%B6.md)

**【扩展 1-11】RunLoop的作用是什么？它的内部的工作机制了解么？（最好结合线程和内存管理来说）**

考察知识点：RunLoop 的作用、内部运行逻辑

主线程的 RunLoop 已经默认开启，程序创建子线程的时候，才需要手动开启 RunLoop。另外在多线程中，你需要判断是否需要 RunLoop，如果需要，那么你要手动创建 RunLoop 并启动。并不是任何情况下都需要去启动 RunLoop，比如，使用子线程去处理一个预先定义好的耗时任务时，这种情况下，就不需要启动 RunLoop。

**【扩展 1-12】RunLoop 的基本概念，它是怎么休眠的？(阿里)**

**【扩展 1-13】利用 runloop 解释一下页面的渲染过程**

当我们调用 [UIView setNeedsDisplay] 时，这时会调用当前 View.layer 的 [view.layer setNeedsDisplay]方法。这等于给当前的 layer 打上了一个脏标记，而此时并没有直接进行绘制工作。而是会到当前的 Runloop 即将休眠，也就是 beforeWaiting 时才会进行绘制工作。紧接着会调用 [CALayer display]，进入到真正绘制的工作。CALayer 层会判断自己的 delegate 有没有实现异步绘制的代理方法 displayer:，这个代理方法是异步绘制的入口，如果没有实现这个方法，那么会继续进行系统绘制的流程，然后绘制结束。

CALayer 内部会创建一个 Backing Store，用来获取图形上下文。接下来会判断这个 layer 是否有 delegate。如果有的话，会调用 [layer.delegate drawLayer:inContext:]，并且会返回给我们 [UIView DrawRect:] 的回调，让我们在系统绘制的基础之上再做一些事情。如果没有 delegate，那么会调用 [CALayer drawInContext:]。以上两个分支，最终 CALayer 都会将位图提交到 Backing Store，最后提交给 GPU。至此绘制的过程结束。

**【扩展 1-14】performSelector:withObject:afterDelay:内部大概是怎么实现的？使用该方法时有什么注意事项？**

```
[self performSelector:(nonnull SEL) withObject:(nullable id) afterDelay:(NSTimeInterval)]; 
```

**实现原理**：创建一个定时器，在指定的时候结束后系统会利用 runtime 通过方法名（Selector本质上就是方法名）去方法列表中查找到对应的方法实现并调用。

**使用注意事项**：

* （1）调用 performSelector:withObject:afterDelay: 方法时，需要先判断希望调用的方法是否存在（通过 respondsToSelector: 方法判断）；
* （2）performSelector:withObject:afterDelay: 方法是异步方法，必须在主线程调用，在子线程调用不会执行。

[参考1](https://www.iteye.com/blog/grayheart-2289984)

[参考2](https://www.jianshu.com/p/7583ff0181c2)


**【扩展 1-15】下面的代码打印输出结果是什么？为什么？**

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"1");
        [self performSelector : @selector(printLog)
                   withObject : nil
                   afterDelay : 0];
        NSLog(@"3");
});

- (void)printLog {
    NSLog(@"2");
}
```

打印结果是：

```
1 3
```

原因分析： 

```
//参数 aSelector：调用的方法
//参数 anArgument：传递给方法的参数，如果方法没有参数的话传 nil
//参数 delay：消息发送之前的最小时间。为 0 时，selector 也不一定立即执行，selector 仍然在线程的运行循环 RunLoop 中排队并尽快执行。
- (void)performSelector:(SEL)aSelector withObject:(id)anArgument afterDelay:(NSTimeInterval)delay;
```

performSelector:withObject:afterDelay: 该方法在当前线程的运行循环（RunLoop）中设置一个计时器（timer）来执行 aSelector 消息。该计时器 Mode 为 NSDefaultRunLoopMode。当触发计时器时，线程会尝试从 RunLoop 的队列中将该消息出列并执行该 selector。如果 RunLoop 正在运行并且处于 NSDefaultRunLoopMode，则成功；否则，该计时器将等待，直到 RunLoop 处于 NSDefaultRunLoopMode 状态。

我们知道只有主线程中的 RunLoop 是默认开启的，而子线程刚创建时并没有 RunLoop，如果不主动获取，那么子线程一直不会有 RunLoop。由于 performSelector:withObject:afterDelay: 方法在一个子线程中执行，而且该子线程中并没有开启 RunLoop，所以 performSelector:withObject:afterDelay: 方法会失效，也就不会执行 aSelector 了。

**【扩展 1-16】autorelease对象在什么时机会被释放？(比如在一个vc的viewDidLoad中创建)(阿里)** 

分两种情况：手动干预释放和系统自动释放

* **手动添加 AutoreleasePool 的情况**下，对象会在**当前作用域大括号结束时**立即释放。

* **非手动添加 Autorelease Pool的情况下**，Autorelease 对象是在**当前的 runloop 迭代结束时**释放的(Autorelease对象出了作⽤域之后，会被添加到最近一次创建的自动释放池中，并会在当前的runloop迭代结束时释放)。

如果在一个vc的viewDidLoad中创建一个 Autorelease对象，那么该对象会在 viewDidAppear ⽅法执行前就被销毁了。

```
kCFRunLoopEntry(1);  //第一次进入会自动创建一个 autorelease
kCFRunLoopBeforeWaiting(32); //进入休眠状态前会自动销毁一个 autorelease，然后重新创建一个新的 autorelease
kCFRunLoopExit(128); //退出 runloop 时会自动销毁最后一个创建的 autorelease
```

[Autorelease实现原理](https://blog.sunnyxx.com/2014/10/15/behind-autorelease/)

[Autorelease底层实现](https://blog.leichunfeng.com/blog/2015/05/31/objective-c-autorelease-pool-implementation-principle/)

**【扩展 1-17】使用 NSTimer 时有哪些注意事项？**

* 注意将 timer 添加到 RunLoop 时应该设置为哪种 Mode；
* 注意 timer 在不需要时，一定要调用 - (void)invalidate; 方法使定时器失效，否则释放不掉，会造成内存泄漏。而且这种内存泄漏使用 Instrument 工具检测不出来。

```
@property (nonatomic, strong) NSTimer *timer;
// 懒加载的方式创建 timer
- (NSTimer *)timer {
    if (_timer == nil) {
        _timer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(timerAction) userInfo:nil repeats:YES];
        [[NSRunLoop mainRunLoop] addTimer:_timer forMode:NSRunLoopCommonModes];
    }
    return _timer;
}
//销毁 timer
- (void)invalidate {
    [self.timer invalidate];
    self.timer = nil;
}
```

