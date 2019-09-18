## 知识点11 内存管理

**【扩展 11-1】介绍下内存的几大区域？** 

![内存布局示意图.png](https://upload-images.jianshu.io/upload_images/4164292-910a6915d3906206.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* BSS段：用来存储**未初始化的全局变量、静态变量**。一旦全局变量或静态变量被初始化就会被回收，并转存到数据段中。
* .text区：也称**代码段**，主要是用来**存放代码的二进制文件(用来存储程序的代码/指令)**，程序结束时系统会自动回收存储在代码段中的数据，内存区域较小。
* .data区：也称**数据段(常量区)**，用来存储 **已经初始化的全局变量、静态变量，常量数据**，直到程序结束时才会被收回。
* 堆区(heap)：用来动态分配内存（需要程序员申请内存，也需要程序员自己管理内存），比如alloc/new/malloc/calloc/realloc出来的对象一般都存储在堆段。分配的内存空间地址越来越大。
* 栈区(stack)：用来**存放局部变量（auto变量）和实参**，特点是编译器自动分配内存，并且在超出其作用域时系统会自动销毁该变量的内存。分配的内存空间地址越来越小

其中.BSS段(未初始化全局区)和.data区(初始化全局区)合起来组成了**全局区(static静态区)**。

**【扩展 11-2】简述对iOS内存管理的理解？**

在iOS中，使用引用计数来管理OC对象的内存。一个新创建的OC对象引用计数默认是1，当引用计数减为0，OC对象就会销毁，释放其占用的内存空间。调用retain会让OC对象的引用计数+1，调用release会让OC对象的引用计数-1。当调用alloc、new、copy、mutableCopy方法返回了一个对象，在不需要这个对象时，要调用release或者autorelease来释放它。

在Objective-C中采用Automatic Reference Counting（ARC）机制，也就是自动引用计数，让编译器来进行内存管理。在新一代Apple LLVM编译器中设置ARC为有效状态，就无需再次键入retain或者release代码，这在降低程序崩溃、内存泄漏等风险的同时，很大程度上减少了开发程序的工作量。编译器完全清楚目标对象，并能立刻释放那些不再被使用的对象。如此一来，应用程序将具有可预测性，且能流畅运行，速度也将大幅提升。

换言之，若满足以下条件，就无需手动输入retain和release代码了，**编译器将自动进行内存管理**:

* 使用Xcode4.2或以上版本；
* 使用LLVM编译器3.0或以上版本；
* 编译器选项中设置ARC有效；

**内存管理的几条原则**如下：

* (1)自己生成的对象，自己所持有；
* (2)非自己生成的对象，自己也能持有；
* (3)不再需要自己持有的对象时，需要释放；
* (4)非自己持有的对象，无法释放。

**【扩展 11-3】ARC都帮我们做了什么(ARC对代码做了什么优化)(网易)？** 

ARC(Automatic Reference Counting)，自动引用计数，实际就是编译时期自动在已有代码中插入合适的内存管理代码以及在Runtime做一些优化。

* 自动插入合适的内存管理代码：

(1)如果是需要自己生成持有的对象，会自动插入Retain代码，并在对象将要释放时插入release代码；

(2)不是自己生成持有的对象会将对象加入autoreleasePool。

* ARC在runtime运行时的优化：

* (1)运行时会在 dealloc 的时候把 weak 修饰的指针自动置 nil。

(2)合并对称的引用计数操作。比如将 +1/-1/+1/-1 直接置为 0；

(3)巧妙地跳过某些情况下autorelease机制的调用。比如当返回值被返回之后，紧接着就需要被retain的时候，没有必要进行autorelease+retain，直接什么都不要做就好了。



**【扩展 11-4】weak指针的实现原理是什么？**

[weak指针的实现原理参考链接](https://www.jianshu.com/p/3c5e335341e0)

weak指针实现原理参考**《Objective-C高级编程 iOS与OS X多线程和内存管理》一书1.4.2节 weak修饰符** 第67页

对象的弱引用关系会存入 SideTable 里面的弱引用散列表中，这个列表中的 key 是对象的地址，value 是所有弱引用了这个对象的的指针的数组。当对象调用 release 的时候，会判断引用计数是否为零，如果为零的话就会调用 dealloc 方法，然后就会通过对象的地址在 SideTable 的弱引用表中找到这个数组，遍历这个数组对数组里边的指针清空置 nil。

Runtime维护了一个weak表(弱引用表)，用于存储指向某个对象的所有weak指针。weak表其实是一个hash表，Key是所指对象的地址，value是weak指针的地址（这个地址的值是所指对象指针的地址）数组。为什么value是数组？因为一个对象可能被多个弱引用指针指向。

weak 的实现原理可概括三步：

* (1)初始化时：runtime会调用objc_initWeak函数，初始化一个新的weak指针指向对象的地址。
* (2)添加引用时：objc_initWeak函数会调用 objc_storeWeak() 函数， objc_storeWeak() 的作用是更新指针指向，创建对应的弱引用表。
* (3)释放时，调用clearDeallocating函数。clearDeallocating函数首先根据"对象地址"获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个entry从weak表中删除，最后清理对象的记录。

**【扩展 11-5】autorelease对象在什么时机会被调用release？(比如在一个vc的viewDidLoad中创建)(阿里)** 

* 手动添加AutoreleasePool的情况下，对象会在当前作用域大括号结束时释放。

* 在没有手加Autorelease Pool的情况下，Autorelease对象是在当前的runloop迭代结束时释放的(Autorelease对象出了作⽤域之后，会被添加到最近一次创建的自动释放池中，并会在当前的runloop迭代结束时释放)。

如果在一个vc的viewDidLoad中创建一个 Autorelease对象，那么该对象会在 viewDidAppear ⽅法执行前就被销毁了。

[Autorelease实现原理](https://blog.sunnyxx.com/2014/10/15/behind-autorelease/)

[Autorelease底层实现](https://blog.leichunfeng.com/blog/2015/05/31/objective-c-autorelease-pool-implementation-principle/)

**【扩展 11-6】方法里有局部对象，出了方法后会立即释放吗？** 

在 arc 下，如果编译器帮我们添加的代码是在出方法前执行[obj release]，那么就是出了方法会立即释放；如果编译器帮我们添加的代码是在创建的时候添加 autorelease，那么就不是立即 release 的。它会在当前runloop 休眠前进行 release。



**【扩展 11-7】造成内存泄漏的原因可能有哪些？** 

* 1）第三方库使用不当，例如AFNetworking，需要将AFHttpSessionManager声明为单例形式，保证只有一个；
* 2）block引起的循环引用；
* 3）没有使用weak修饰delegate，使用的是strong；
* 4）NSTimer没有关闭（参照链接)[https://www.jianshu.com/p/e18bb7651064]；
* 5）非OC对象的内存处理，如c语言中malloc需要free；
* 6）地图类 需要释放相关代理

**【扩展 11-8】Autoreleasepool所使用的数据结构是什么？AutoreleasePoolPage的结构体了解么？** 

 AutoreleasePool 没有单独的结构，是由若干个 AutoreleasePoolPage 以双链表的形式组成的。(待优化)

**【扩展 11-9】什么情况下需要手动创建@autoreleasepool？** 

使用场景：当有大量临时变量的时候，需要手动创建@autoreleasepool，这样可以避免内存峰值过高。典型的例子是读入大量图像的同时改变其尺寸。

**【扩展 11-10】谈谈你对 ARC 的理解。(百度)** 

ARC 是编译器完成的，依靠引用计数，谈谈几个属性修饰符的内存管理策略，什么情况下会内存泄露。

**【扩展 11-11】野指针是什么，iOS 开发中什么情况下会有野指针？(百度)** 

野指针是不为 nil，但是指向已经被释放的内存的指针。 使用__ unsafe_unretain或者assign修饰的指针，对象释放后会出现野指针。 一般情况下oc使用了weak指针，在对象销毁时指针会置nil。

**【扩展 11-12】autoreleasepool 的使用场景和原理** 

【延伸】自动释放池是什么，如何工作的？

自动释放池是cocoa提供的帮助我们管理对象内存的一个工具。当我们像一个对象发送autorelease消息时，这个对象就自动加入到最新的自动释放池中，当自动释放池被销毁的时候，会自动向自动释放池中的所有对象发送一条release消息。也就是说我们不再需要手动向每一个对象发送release消息以释放对象，而是将其加入到自动释放池中最后统一释放。使用自动释放池也可以避免一些人为原因导致的内存泄漏。

**【扩展 11-13】怎么判断某个 cell 是否显示在屏幕上？(阿里)** 

```
1.  - (NSArray*)visibleCells;
//UITableView的方法，这个最直接，返回一个UITableviewcell的数组。
对于自定制的cell，之后的处理可能稍微繁琐些。
```

```
2.- (NSArray*)indexPathsForVisibleRows;
//UITableview的又一个方法，这个比较好用了，返回一个NSIndexPath的数组,可以直接用indexpath.row去调你的table_related_Array里的数据了。比较方便用于自定制的cell。
```

```
3.- (CGRect)rectForRowAtIndexPath:(NSIndexPath*)indexPath;
CGRect cellR = [myTV rectForRowAtIndexPath:indx];
if (myTV.contentOffset.y - cellR.origin.y < myCell.frame.size.height || cellR.origin.y - myTV.contentOffset.y >myTV.size.height) {
//这个时候myCell应该是不在myTV的可视区域了。
} else {//myCell在可视区域时，业务处理
}
//这个方法可以用在代理回调较多的设计中。
```

**【扩展 11-14】ARC的本质？(阿里)** 

ARC是编译器的特性，它并没有改变OC采用引用计数技术来管理内存的本质，更不是GC(垃圾回收)，底层实现依然依赖引用计数，只不过ARC模式下编译器会自动帮我们管理。

* 打开ARC：-fobjc-arc
* 关闭ARC：-fno-objc-arc



## 知识点12 性能优化

**【扩展 12-1】你在项目中是怎么进行内存优化（性能优化）的？** 

[优化内存](https://www.jianshu.com/p/8462b7559e48)

* (1)采用ARC(Automatic Reference Counting，自动引用计数)来管理内存。ARC除了可以避免内存泄露外，ARC还有助于提高程序性能；
* (2)在正确的地方使用reuseIdentifier；如果不使用reuseIdentifier的话，每显示一行table view就不得不设置全新的cell。这对性能的影响可是相当大的。⾃iOS6起，除了UICollectionView的cells和补充views，你也应该在header和footer views中使用reuseIdentifiers。
* (3)尽量把View设置为完全不透明。如果有透明的Views，应该把opaque(不透明)属性设置为YES，这样可以减小开销，对内存有好处。opaque属性给渲染系统提供了了一个如何处理这个view的提示。如果设为 YES， 渲染系统就认为这个view是完全不透明的，这使得渲染系统优化一些渲染过程进而提高性能。如果设置为NO，渲染系统正常地和其它内容组成这个View。opaque属性的默认值是 YES。

在相对比较静止的画面中，设置这个属性不会有太⼤影响。然而当这个view嵌在 scroll view里边，或者是一个复杂动画的一部分，不设置opaque这个属性的话会在很⼤大程度上影响app的性能。只要一个视图不透明度小于1，就会导致blending操作在iOS的图形处理器（GPU）中进行混合像素颜色的计算。blending主要是指混合像素颜色的计算。比如当我们把两个图层叠加在一起时，如果第一个图层有透明效果，那么最终像素颜色的计算需要将第二个图层也考虑进来。这一过程即为Blending。

**为什么Blending会导致性能的损失呢？**因为一个图层如果是完全不透明的，那么系统会直接显示该图层的颜色；而如果图层是带透明效果的，那么就会进行更多的计算，需要把下面的图层也考虑进来，进行混合颜色色值的计算。

* (4)避免庞大的XIB：当你加载一个XIB的时候所有内容都被放在了内存里，包括任何图片。如果有一个不会即刻用到View，就存在内存资源的浪费。
* (5)不要阻塞主线程。大部分阻塞主线程的情形是你的app 在做一些牵涉到读写外部资源的I/O操作，比如存储或者网络。
* (6)在ImageViews中调整图片大小。如果在UIImageVIew中显示一个来自bundle的图片，你应保证图片的大小和UIImageVIew的大小相同。因为在运行中缩放图片是很耗资源的，特别是UIImageVIew嵌套在UIScrollVIew中的情况下。 如果图片是在远端服务器加载的你不能控制图片的大小，你可以在下载完成后，最好用background thread，缩放一次，然后在UIImageView中使用缩放后的图片。
* (7)选择正确的collection(集合)。比如NSArray、NSDictionary、NSSet等。Arrays是有序的一组值，使用index来查找很快，使用value 查找很慢， 插入/删除很慢；Dictionaries用来存储键值对，⽤键来查找比较快；Sets是无序的一组值。⽤值来查找很快，插入/删除很快。
* (8)打开gzip压缩。减小文档的方式就是在服务端和你的app中打开gzip。这对于文字这种能有高压缩率的数据来说会有更显著的效用。另外，iOS已经在NSURLSession 中默认支持了gzip压缩，当然AFNetWorking这些框架也是支持的。
* (9)重用和延迟加载。
* (10)Cache 缓存。缓存那些不大可能改变但经常使用的东西。 我们缓存什么呢？远端服务器的响应，图片，甚至计算结果，比如UItableView的行高。
* (11)处理内存警告。一旦系统内存过低，iOS会通知所有运行中app。如果你的app收到了内存警告，它就需要尽可能释放更多的内存。最佳的方式是移除缓存。 幸运的是，UIKit的提供了集中收集内存警告的方法：1）在appdelegate中使用applicationDidReceiveMemoryWarning：的方法； 2）在你自定义UIViewController的子类中覆盖didReceiveMemoryWarning； 3）注册并接受 UIApplicationDidReceiveMemoryWarningNotification的通知，一旦接受到通知你就需要释放任何不必要的内存使用。
* (12)重用大开销对象。一些objects的初始化很慢，比如NSDateFormatter 和NSCalendar。然而你又不可避免的使用它们，比如从JSON和XML中解析数据。想要避免使用这个对象的瓶颈你就需要重用它们，可以通过添加属性到你的class里或者创建静态变量来实现。
* (13)优化TableView。为了保证TableVIew有更好的滚动性能，可以采取以下措施： （1）正确使用ruseIdentifier来重用cells。（2）采用懒加载即延迟加载的方式加载cell上的控件。（3）当TableView滑动的时候不加载（这个我会在接下的文章中写具体的代码实现）（4）缓存cell的高度。在呈现cell前，把cell的高度计算好缓存起来，避免每次加载cell的时候都要计算。（5）尽量使用不透明的UI控件（6）使用drawRect绘制。
* (14)使用Autorelease Pool。当有大量临时变量的时候，需要手动创建@autoreleasepool，这样可以避免内存峰值过高。
* (15)选择是否缓存图片。常见的从bundle中加载图片的方式有两种，一个是imageNamed，另一个时imageWithContentOfFile。既然有两种方式那它们之间有什么差别呢？先说第一种方式他的优点是当加载是它会缓存图片。相反imageWithContentOfFile的仅仅加载图片。如果你加载一个大的图片而且仅仅使用一次的话就没必要缓存图片。

**【扩展 12-2】性能优化从哪些方面来着手？**

可以从卡顿与否（流畅度）、耗电情况、启动时间以及安装包瘦身等四个方面来着手。详见[性能优化](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E4%B9%8B%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.md)

**【扩展 12-3】列表卡顿的原因可能有哪些？如何来优化？(UITableView的相关优化)**

详见[卡顿原因分析及卡顿解决方法](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E4%B9%8B%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.md)

[iOS保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)

**【扩展 12-4】App启动优化策略？最好结合启动流程来说（main()函数的执行前后都分别说一下，知道多少说多少）** 

详见[3.启动时间优化](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E4%B9%8B%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.md)

**【类似问题】执行main()函数之前经历了怎样的过程？**

[iOS 程序 main 函数之前发生了什么](https://blog.sunnyxx.com/2014/08/30/objc-pre-main/)

* 1)dyld 开始将程序二进制文件初始化；
* 2)交由ImageLoader 读取 image，其中包含了我们的类，方法等各种符号 (Class、Protocol 、Selector、 IMP)
* 3)由于runtime 向dyld 绑定了了回调，当image加载到内存后，dyld会通知runtime进行处理
* 4)runtime 接⼿后调⽤map_images做解析和处理
* 5)接下来load_images 调⽤call_load_methods方法，遍历所有加载进来的 Class，按继承层次依次调用Class的+load和其他Category的+load方法
* 6)⾄此 所有的信息都被加载到内存中
* 7)最后dyld调用真正的main函数

**【扩展 12-5】AppDelegate如何瘦身？** 

[AppDelegate瘦身指南](https://juejin.im/entry/5b2a101cf265da597236c295)

* (1)FRDModuleManager。FRDModuleManager是豆瓣开源的轻量级模块管理工具。它通过减小AppDelegate的代码量来把很多职责拆分至各个模块中去，这样 AppDelegate 会变得容易维护。
* (2)利用Category（分类）将不同的逻辑代码分布到其他文件中以减少APPDelegate的代码行数。
* (3)JLRoutes。可以很方便的处理不同URL Scheme以及解析它们的参数，并通过回调block来处理URL对应的操作。我们可以通过定义URL的规则来定制我们的页面跳转或其他逻辑。



**【扩展 12-6】App无痕埋点的思路了解么？你认为理想的无痕埋点系统应该具备哪些特点？** 

[无侵入的埋点方案如何实现？](https://time.geekbang.org/column/article/87925)

**【扩展 12-7】你知道有哪些情况会导致App崩溃，分别可以用什么方法拦截并化解？** 

造成App崩溃的原因有很多，包括以下这些：

* (1)向某个对象发送其无法响应的方法。报错信息"unrecognized selector sent to instance"。

**解决方法**：可以在写一个方法的时候，判断一下调用者的类型，不符合类型的不让其调用，也可以使用runtime对常见的方法调用做一下错误兼容。比如调用了NSString中不存在的方法而造成崩溃，可以像下面这样解决：

```
@implementation NSString (NSRangeException)

+ (void)load{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        @autoreleasepool {
            [objc_getClass("__NSCFConstantString") swizzleMethod:@selector(objectForKeyedSubscript:) swizzledSelector:@selector(replace_objectForKeyedSubscript:)];
        }
    });
}

- (id)replace_objectForKeyedSubscript:(NSString *)key {
    return nil;
}

@end
```
* (2)NSRangeException 异常。该异常是由数组越界或者string访问越界引起的。

**解决方法**：利用runtime的Swizzle Method特性，可以实现从框架层面杜绝这类的崩溃问题，这样做的好处有两点：一是开发人员忘了写判断越界的逻辑，也不会造成app的崩溃；二是不需要修改现有的代码，对现有代码的侵入性降低到最低，不需要添加大量重复的逻辑判断代码。
* (3)集合类中添加nil对象。NSDictionary插入nil对象会造成崩溃，但是插入NSNull对象是不会造成崩溃的。

**解决方法**：只要利用runtime的Swizzle Method把nil对象给转换成NSNull对象就可以把该问题给解决了。创建一个NSDictionary的类别，利用runtime的Swizzle Method来替换系统的方法。
* (4)SIGSEGV 异常。当去访问没有被开辟的内存或者已经被释放的内存时，就会发生这样的异常。另外，在低内存的时候，也可能会产生这样的异常。我们使用Xcode自带的Leaks工具检测相关的内存泄漏问题。

**解决方法**：在使用C语言对象的时候，一定要记得在不使用的时候给释放掉，ARC并不能释放掉这块内存。

[示例源码链接](https://github.com/guoshimeihua/RuntimeDemo)

* (5)KVO不合理的移除关联key。

**【扩展 12-8】你知道有哪些情况会导致App卡顿，分别可以用什么方法来避免？** 

[卡顿原因分析及卡顿解决方法](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E4%B9%8B%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.md)

**【扩展 12-9】如何对 UITableView 调优？(百度)** 

**思路**：一方面是通过 instruments 检查影响性能的地方，另一方面是估算高度并在 runloop 空闲时缓存。

详见[卡顿原因分析及卡顿解决方法](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E4%B9%8B%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.md)

[iOS保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)

[UITableView优化](https://www.jianshu.com/p/dbcf3665fad8)

**【扩展 12-10】如何监测以及定位卡顿？有哪些方案？**

**【扩展 12-11】具体如何通过RunLoop来监控卡顿？**

⭐️⭐️⭐️⭐️⭐️


[iOS实时卡顿监控](http://www.tanhao.me/code/151113.html/)

[如何利用 RunLoop 原理去监控卡顿？](https://time.geekbang.org/column/article/89494)

[RunLoop的应用场景（四）- App卡顿监测](http://hl1987.com/2018/04/27/RunLoop%E6%80%BB%E7%BB%93%EF%BC%9ARunLoop%E7%9A%84%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF%EF%BC%88%E5%9B%9B%EF%BC%89App%E5%8D%A1%E9%A1%BF%E7%9B%91%E6%B5%8B/)

[微信读书iOS性能优化总结](https://wereadteam.github.io/2016/05/03/WeRead-Performance/)

[iOS卡顿检测](https://allluckly.cn/%E6%8A%95%E7%A8%BF/tougao73)

[iOS UI卡顿检测](https://blog.csdn.net/u010262501/article/details/79616963)

**【扩展 12-12】如何对安装包进行瘦身？**

详见[4.安装包瘦身](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E4%B9%8B%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.md)

**【扩展 12-13】如何定位和分析项目中影响性能的地方？**

使用**Instruments工具**。在进行iOS App性能分析时，首先考虑借助Instruments这个利器来定位和分析性能问题。比如要查看程序中哪些部分最耗时，可以使用Time Profiler，要查看内存是否泄漏，可以使用Leaks；借助 Energy Log来进行耗电检测；借助 Network来进行流量检测；借助Instruments的Core Animation来进行离屏渲染、图层混合等GPU耗时监测。

**【扩展 12-14】imageNamed和imageWithContextOfFile的区别？哪个性能高？**

* 1.用imageNamed的方式加载时，图片使用完毕后缓存到内存中，内存消耗多，加载速度快。即使生成的对象被 autoReleasePool释放了，这份缓存也不释放，如果图像比较大，或者图像比较多，用这种方式会消耗很大的内存。imageNamed采用了缓存机制，如果缓存中已加载了图片，那么直接从缓存读取就可以了，不必每次都去读文件了，效率会更高。 
* 2.imageWithContextOfFile加载图片是不会缓存的，加载速度慢。
* 3.大量使用imageNamed方式会在不需要缓存的地方额外增加开销CPU的时间.当应用程序需要加载一张比较大的图片并且使用一次性，那么其实是没有必要去缓存这个图片的，用imageWithContentsOfFile是最为经济的方式,这样不会因为UIImage元素较多情况下，占用过多CPU资源.

## 知识点13 设计模式与架构

**【扩展 13-1】讲讲MVC、MVVM、MVP区别和联系，以及你在项目里具体是怎么使用的？**

[浅析MVC、MVP、MVVM](https://github.com/baohenglin/HLBlog/blob/master/Articles/%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1-MVC%E3%80%81MVP%E3%80%81MVVM%E8%AF%A6%E8%A7%A3.md)

[MVC、MVP、MVVM详解](https://draveness.me/mvx)

**【扩展 13-2】用过哪些设计模式？**

[23种设计模式和6大设计原则](https://juejin.im/entry/58e45768ac502e4957a22909)

**【扩展 13-3】一般开始一个项目，你的架构是如何思考的？** 

[iOS应用架构现状分析](http://mrpeak.cn/blog/ios-arch/)

[今日头条：iOS 架构设计杂谈](https://juejin.im/post/5b2b1a73e51d4558b27782c0)


**【扩展 13-4】说一下简单工厂模式、工厂模式和抽象工厂模式？** 

**【扩展 13-5】iOS 系统框架里使用了哪些设计模式？至少说6个** 

* 单例模式。在Cocoa框架中大量使用单例模式，如：

```
[NSNotificationCenter defaultcenter]
[UIApplication sharedApplication]
[NSUserDefaults standardUserDefaults]
```
* MVC
* 观察者模式。单例模式中的通知中心，就是典型的观察者模式：

```
// 移除观察者
[[NSNotificationCenter defaultCenter] removeObserver:self name:@"saveMessage" object:nil];
// 添加观察者
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(saveMessage) name:@"saveMessage" object:nil];
// 派发通知
[[NSNotificationCenter defaultCenter] postNotificationName:@"postData" object:saveImageArray];
```

* 责任链模式。Cocoa中对触摸事件的处理就是使用的责任链模式。UIView，UIApplication，UIViewController都直接或间接地继承自UIResponder类。事件的响应者链是一系列连接在一起的UIResponder对象，事件在响应者链中向上传递，直到找到处理它的响应者。
* 委托模式。UITableViewDelegate中的各种方法，实际上是UITableView委托的实现。
* 中介者模式。
* 工厂模式。NSFoundation框架中的。大量的objective-c对象都采用了这种模式。比如NSString的stringWith系列：

```
+ stringWithCharacters:length:
+ stringWithString:
+ stringWithCString:encoding:
+ stringWithUTF8String:
```

**【扩展 13-6】谈谈对单例模式的理解（定义、优缺点），有几种实现方式？** 

**单例模式的定义**：简单来说，一个单例类，在整个程序中只有一个实例，并且提供一个类方法供全局调用，在编译时初始化这个单例类，然后一直保存在内存中，直到程序退出时由系统自动释放这部分内存。

**单例模式的优点：**

* (1)在单例模式中，对单例类的所有实例化得到的都是相同的一个实例。这样就防止其它对象对自己的实例化，确保所有的对象都访问同一个实例；
* (2)由于在整个程序中只会实例化一次，故如果出现了问题，可快速定位；
* (3)避免对共享资源的多重占用，节省了系统内存资源，提高了程序的运行效率（因为在整个程序中只存在一个实例化对象）。
* (4)灵活性：因为类控制了实例化过程，所以类可以更加灵活修改实例化过程。 

**单例模式的缺点：**

* (1)单例一旦创建，对象指针保存在静态区，单例对象在堆中分配的内存空间只有等程序结束才能释放，所以过多的单例会增加内存的消耗。
* (2)由于单利模式中没有抽象层，因此不易扩展。
* (3)单例类的职责过重，在一定程度上违背了“单一职责原则”。

**单例使用场景：**

* 单例模式用来限制一个类只能创建一个对象，那么此对象的属性可以存储全局共享的数据。所有类都可以访问、设置此单例对象中的属性数据；
* 需要频繁实例化然后销毁的对象。 
* 创建对象时耗时过多或者耗资源过多，但又经常用到的对象。也就是说如果一个类创建的时候非常的耗费资源或影响性能，那么此对象可以设置为单例以节约资源和提高性能。

**实现方式有以下几种：**

**方式（1）**：通过**GCD的dispatch_once**来实现单例，同样可以在保证线程安全的前提下来实现单例(推荐)

```
+(instancetype)sharedSingleton{
     static id _instance = nil;
     static dispatch_once_t onceToken;
     dispatch_once(&onceToken, ^{
     //下面这个函数在整个程序运行期间只会执行一次，保证了线程安全。
     _instance = [[self alloc] init];
     });
     return _instance;
}
```

[dispatch_once实现单例的原理分析](https://juejin.im/post/5b15020a5188257d4f0d7c53)


**方式（2）**：直接**使用@synchronized**来保证线程安全。

```
+(instancetype)sharedSingleton{
     static id _instance = nil;
     //在@synchronized大括号中的代码，在同一时间内，只能有一个对象访问。
     @synchronized (self){
       if(_instance == nil){
       _instance = [[self alloc] init];
       }
     }
     return _instance;
}
```

**方式（3）**：使用**同步锁NSLock**

```
+(instancetype)sharedSingleton {
     static id _instance = nil;
     NSLock *lock = [[NSLock alloc]init];
     //关闭线程锁
     [lock lock];
     if (_instance == nil) {
     _instance = [[self allock]init];
     }
     //开启线程锁
     [lock unlock];
     return _instance;
}
```

**iOS 系统中使用的单例类**：

* UIApplication
* NSNotificationCenter
* NSFileManager
* NSUserDefaults
* NSURLCache
* NSHTTPCookieStorage

**【扩展 13-7】设计模式是为了解决什么问题的？**

使用设计模式是为了重用代码、让代码更容易被他人理解、保证代码可靠性，增加可维护性。

**【扩展 13-8】设计模式的成员构成以及工作机制是什么？**

**【扩展 13-9】设计模式的优缺点是什么？**

**【扩展 13-10】面向对象的几个设计原则了解么？最好可以结合场景来说**

六大设计原则。

**【扩展 13-11】MVC 具有什么样的优势，各个模块之间怎么通信，比如点击 Button 后 怎么通知 Model？**

**MVC优势**：

* 低耦合性。
* 利于组件可重用。
* 可维护性强。

经典MVC示意图如下：


![经典MVC示意图.png](https://upload-images.jianshu.io/upload_images/4164292-cce4c649d2e05e71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**MVC各个模块之间通信过程**分析：

* Controller 访问 Model：可以直接单向通信。Controller 需要将 Model 呈现给用户，因此需要知道模型的一切，还需要有同 Model 完全通信的能力，并且能任意使用 Model 的公共 API。
* Model 访问 Controller：由于Model 是独立于 UI 存在的，因此无法直接与 Controller 通信，但是当 Model 本身信息发生了改变的时候，会通过Notification & KVO & Block等方式进行间接通信。
* Controller 访问 View：可以直接单向通信。Controller 通过 View 来布局用户界面。
* View 访问 Controller：通过间接方式来进行通信。View通过Target-action、delegate、dataSource等方法与Controller间接通信。
* Model 和 View：二者完全互相隔离，不能直接通信。View 通过 Controller 获取 Model 数据。

**点击 Button 后通知 Model的过程**：

首先来分析一下向View上的Button添加点击事件的代码：

```
[self.currentBtn addTarget:self action:@selector(btnClick) forControlEvents:UIControlEventTouchUpInside];
```

* self 指目标对象为当前对象，即当前的ViewController；
* action 指的是向目标对象发送消息的方法名；
* UIControlEventTouchUpInside表示方法调用时机，即单击时。

由上可知，当点击Button时，当前View上的self.currentBtn会向目标对象ViewController发送一条消息，即objc_msgSend(self,@selector(btnClick)),此时就会触发btnClick方法，通过在btnClick方法内调用Model的相关API进而更新数据模型model中的数据。


**【扩展 13-11】你觉得框架和设计模式的区别是什么？**

**框架**是framework，是一种为特定的领域内的应用提供可扩展模板的架构实例，也就是说框架是对一组相关联问题的解决方法的抽象设计（架构）的实例集合；

**设计模式**简单的讲就是可以复用的设计范例。是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结，它强调的是一个设计问题的解决方法。往往一个架构由多个设计模式组成。

**架构**简单的说架构就是一个蓝图，是一种设计方案，将客户的不同需求抽象成为抽象组件，并且能够描述这些抽象组件之间的通信和调用。

**【扩展 13-12】看过哪些第三方框架的源码？它们是怎么设计的？设计好的地方在哪里？不好的地方在哪里？如何改进？（这道题的后三个问题难度已经很高了，如果不是太牛的公司不建议深究）**

**【扩展 13-13】最喜欢哪个设计模式？为什么？**

**【扩展 13-14】可以说几个重构的技巧么？你觉得重构适合什么时候来做？**

[最实用的10个重构小技巧](https://www.cnblogs.com/zuoxiaolong/p/pattern27.html)

重构的技巧包括：

* 重复代码的提炼
* 冗长方法的分割
* 嵌套条件分支的优化
* 去掉一次性的临时变量
* 消除过长参数列表
* 提取类或继承体系中的常量
* 让类提供应该提供的方法
* 拆分冗长的类
* 提取继承体系中重复的属性与方法到父类

重构是一个不断优化的过程。重构可以在增加新功能的时候进行，也可以在扩展起来不方便的时候进行。


**【扩展 13-15】说说你对 MVC 和 MVVM 的理解。(百度)**

MVC 的 C 太臃肿，可以和 V 合并，变成 MVVM 中的 V，而 VM 用来将 M 转化成 V 能用的数据。

**【扩展 13-16】介绍一下 MVVM 和 RAC。数据的双向绑定怎么做？bind 函数了解过么？(网易)**

[iOS架构由浅入深 | MVVM](https://juejin.im/post/5b66a8c251882519790caae8)

**【扩展 13-17】如何设计图片缓存？(阿里)**(待优化)

[iOS高性能图片架构与设计](https://zhuanlan.zhihu.com/p/20273299)


图片缓存组件由HLImageView、HLImageManager、HLImageCache、HLImageLoader、HLImageProcessor五大部分组成，它们分别负责图片显示，请求管理，缓存，数据加载，数据处理。




**【扩展 13-18】有没有自己设计过网络控件？(阿里)**(待优化)

[交互设计](https://merlinwu330387414.wordpress.com/2018/02/02/%E4%BA%A4%E4%BA%92%E8%AE%BE%E8%AE%A1-%E6%8E%A7%E4%BB%B6%E8%AE%BE%E8%AE%A1/)

**【扩展 13-19】谈谈对组件化的理解**

[iOS应用架构谈 组件化方案](https://casatwy.com/iOS-Modulization.html)














