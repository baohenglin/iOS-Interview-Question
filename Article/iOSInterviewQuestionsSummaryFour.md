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

**内存管理的思考方式**如下：

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

**【扩展 11-5】autorelease对象在什么时机会被调用release？(阿里)** 

在没有手加Autorelease Pool的情况下，Autorelease对象是在当前的runloop迭代结束时释放的(原因是系统在每个runloop迭代中都加入了自动释放池Push和Pop)，而不是“当前作用域大括号结束时释放”。

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



## 知识点12 性能优化

**【扩展 12-1】你在项目中是怎么优化内存的？** 

**【扩展 12-2】性能优化从哪些方面来着手？**

**【扩展 12-3】列表卡顿的原因可能有哪些？如何来优化？(UITableView的相关优化)**

**【扩展 12-4】App启动优化策略？最好结合启动流程来说（main()函数的执行前后都分别说一下，知道多少说多少）** 

**【扩展 12-5】AppDelegate如何瘦身？** 

**【扩展 12-6】App无痕埋点的思路了解么？你认为理想的无痕埋点系统应该具备哪些特点？** 

**【扩展 12-7】你知道有哪些情况会导致App崩溃，分别可以用什么方法拦截并化解？** 

**【扩展 12-8】你知道有哪些情况会导致App卡顿，分别可以用什么方法来避免？** 

**【扩展 12-9】如何对 UITableView 调优？(百度)** 

一方面是通过 instruments 检查影响性能的地方，另一方面是估算高度并在 runloop 空闲时缓存。




## 知识点13 设计模式与架构

**【扩展 13-1】讲讲MVC、MVVM、MVP区别和联系，以及你在项目里具体是怎么使用的？**

**【扩展 13-2】用过哪些设计模式？**

**【扩展 13-3】一般开始一个项目，你的架构是如何思考的？** 

**【扩展 13-4】说一下简单工厂模式、工厂模式和抽象工厂模式？** 

**【扩展 13-5】iOS 系统框架里使用了哪些设计模式？至少说6个** 

**【扩展 13-6】谈谈对单例模式的理解（定义、优缺点），有几种实现方式？** 

**【扩展 13-7】设计模式是为了解决什么问题的？**

**【扩展 13-8】设计模式的成员构成以及工作机制是什么？**

**【扩展 13-9】设计模式的优缺点是什么？**

**【扩展 13-10】面向对象的几个设计原则了解么？最好可以结合场景来说**

六大设计原则。

**【扩展 13-11】MVC 具有什么样的优势，各个模块之间怎么通信，比如点击 Button 后 怎么通知 Model？**



**【扩展 13-11】你觉得框架和设计模式的区别是什么？**

**【扩展 13-12】看过哪些第三方框架的源码？它们是怎么设计的？设计好的地方在哪里？不好的地方在哪里？如何改进？（这道题的后三个问题难度已经很高了，如果不是太牛的公司不建议深究）**

**【扩展 13-13】最喜欢哪个设计模式？为什么？**

**【扩展 13-14】可以说几个重构的技巧么？你觉得重构适合什么时候来做？**


**【扩展 13-15】说说你对 MVC 和 MVVM 的理解。(百度)**

MVC 的 C 太臃肿，可以和 V 合并，变成 MVVM 中的 V，而 VM 用来将 M 转化成 V 能用的数据。

**【扩展 13-16】介绍一下 MVVM 和 RAC。数据的双向绑定怎么做？bind 函数了解过么？(网易)**

**【扩展 13-17】如何设计图片缓存？(阿里)**

**【扩展 13-18】有没有自己设计过网络控件？(阿里)**














