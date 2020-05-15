## 知识点：Block

**【扩展 1-0】block的本质是什么？block的内存结构是怎样的？一共有几种block？都是什么情况下生成的？（重点）**

Block本质上是Objective-C对象。这是因为Block内部有一个isa指针。更确切地说，block是封装了函数调用(函数指针)以及函数调用环境(捕获到的参数)的OC的对象。一个block实例实际上由以下几部分组成的：

* 1）isa指针
* 2）flags：用于按bit位表示一些block的附加信息
* 3）reserved：保留变量
* 4）invoke：函数指针，指向具体block实现的函数调用地址
* 5）descriptor：表示该block的附加描述信息，主要是结构体内存的大小(size)以及copy和dispose函数的指针
* 6）variables：捕获的变量，block能够访问它外部的局部变量，就是因为将这些变量（或变量的地址）复制到了结构体中。


在Objective-C语言中，根据存储位置可以分为3种类型的block：

* 1）NSGlobalBlock(_NSConcreteGlobalBlock)   全局静态block（产生条件：**没有访问auto变量**）
* 2）NSStackBlock(_NSConcreteStackBlock)    保存在栈中的block，当函数返回(超出函数作用域)时会被销毁。(产生条件：**在MRC下，访问了auto变量**)
* 3）NSMallocBlock(_NSConcreteMallocBlock)  保存在堆中的block，当引用计数为0时会被销毁。(产生条件：**NSStackBlock调用了copy方法，就会将block内存拷贝到堆上，变成了__NSMallocBlock**)

其实，这3种类型的Block都继承自NSBlock类型。

Block是苹果在iOS4开始引入的对C语言的扩展，用来实现匿名函数的特性。Block是一种特殊的数据类型，其可以正常定义变量、作为参数、作为返回值。特殊地，Block还可以保存一段代码，在需要的时候调用，目前Block已经广泛应用于iOS开发中，常用于GCD、动画、排序以及各类回调。

【注意】在MRC的情况下，如果访问了auto变量，那么生成的是__NSStackBlock。__NSStackBlock存在一个问题，也就是超出作用域后变量就会被系统自动销毁，变量被系统自动销毁后如果再访问该变量那么就会存在安全问题。如果开启了ARC，那么编译器就会自动进行copy操作，将__NSStackBlock转变为__NSMallocBlock。

**【扩展 1-1】默认情况下能修改被block捕获的自动变量么？为什么？__block修饰词都做了什么？**

默认情况下，不能修改block捕获的自动变量值。因为block捕获自动变量（局部变量）传递到block变量所指向的结构体内部的是自动变量的值（也就是“值传递”），而不是指向自动变量内存的指针（不是“地址传递”），所以block内部不能改变block捕获的变量。

使用__block修饰的自动变量传递到block变量所指向的结构体内部的是 **指向变量内存地址的指针**，故可以在Block内部改变变量值。

此外，block内部可以修改捕获到的静态变量、全局变量和静态全局变量。全局变量和静态全局变量由于作用域是全局，且存储在全局区，供所有函数调用，所以可以在block内被修改；静态变量之所以可以在block内被修改是因为传递给block的是内存的指针。全局变量、静态全局变量和函数参数它们并没有变成Block结构体__main_block_impl_0的成员变量，并不会被Block持有，所以也不会增加retainCount的值。

综上所述，在声明Block之后、调用Block之前对局部变量进行修改，那么调用Block时，捕获到的局部变量值是修改之前的旧值(原因是在声明Block时便已经将局部变量的值传递到Block变量所指向的结构体)；默认情况下，在Block中不可以直接修改局部变量。


**【扩展 1-2】在block内修改自动变量的值有几种方式？如何修改？**

在block内修改自动变量的值有两种方式。1）将内存地址指针传递到block中，也就是使用__block来修饰局部变量(推荐)；2）改变存储方式，可以修改为static静态变量、静态全局变量或全局变量（不推荐，因为此方法会使变量一直占用内存，不会释放内存）

**【扩展 1-3】这三种block是在什么情况下产生的？以及它们之间的区别和联系是什么？**

1）从捕获外部变量的角度上来看：

 * _NSConcreteGlobalBlock **没有访问局部变量**（自动变量/auto变量）或只访问了全局变量、静态变量的block为_NSConcreteGlobalBlock，生命周期从创建到应用程序结束。
 * _NSConcreteStackBlock 访问了auto变量，并且没有强指针引用和copy操作的block都是StackBlock，StackBlock的生命周期由系统控制，一旦返回之后，就被系统销毁了；
 * _NSConcreteMallocBlock 有强指针引用或 copy修饰的属性 引用的block会被复制到堆中而成为MallocBlock，没有强指针引用即销毁，生命周期由程序员控制
     
2)从持有对象的角度上来看：

* _NSConcreteGlobalBlock是不持有对象的
* _NSConcreteStackBlock也是不持有对象的
* _NSConcreteMallocBlock是持有对象的。

**【扩展 1-4】什么情况下ARC会自动将Block从栈上拷贝到堆上呢？**

在ARC环境下，编译器会根据情况自动将栈上的block复制(copy)到堆上。一般来说有以下5种情况：

* (1)block作为函数返回值时：

```
typedef void(^HLBlock) (void);
//test方法
HLBlock test()
{
    //定义block,
    int age = 1111;
    HLBlock block = ^{
        //block访问了auto变量age，是__NSStackBlock__类型的block，存储在栈上
        NSLog(@"block作为函数的参数返回值的情况---%d",age);
    };
    
    //block作为函数的返回值
    return block;
    //在MRC环境下，由于block访问了auto变量age，存储在栈上，当超出test函数的作用域时，block在栈上的内存会被系统自动回收。那么在test函数外再调用block时就会出问题。这种情况下，就需要手动进行一次copy操作。
    //在ARC环境下，编译器会自动执行copy操作，将栈上的block拷贝到堆上，由__NSStackBlock__变为__NSMallocBlock__。
    //return [block copy];
  
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        HLBlock block = test();
        //调用block
        block();
        NSLog(@"----%@",[block class]);
        //2019-06-17 10:50:31.038489+0800 Block的copy操作[1003:37627] ----__NSMallocBlock__

    }
    return 0;
}
```

* (2)将block赋值给__strong指针时。

将block赋值给强指针的情况，在ARC环境下，编译器对block自动进行一次copy操作[block copy]，block的类型由__NSStaticBlock__变为了__NSMallocBlock__。代码如下：

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        int age = 1110;
        //将block赋值给强指针的情况
        HLBlock block = ^{
            //由于block访问了auto变量，因此是__NSStaticBlock__类型
            NSLog(@"block：age---%d",age);
        };
        block();
        NSLog(@"----%@",[block class]);
        //2019-06-17 11:01:11.834323+0800 Block的copy操作[1058:40766] ----__NSMallocBlock__
       //由以上打印结果可知：将block赋值给强指针时，在ARC环境下，编译器对block自动进行一次copy操作[block copy]，block的类型由__NSStaticBlock__变为了__NSMallocBlock__。

    }
    return 0;
}
```

再对比看一下没有将block赋值给强指针的情况。没有将block赋值给__strong指针时，在ARC环境下编译器不会自动执行copy操作。代码如下：

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {        
        int age = 1110;
        NSLog(@"----%@",[^{
            //由于block访问了auto变量，因此是__NSStaticBlock__类型
            NSLog(@"block：age---%d",age);
            //2019-06-17 11:13:06.358166+0800 Block的copy操作[1139:44714] ----__NSStackBlock__
            //由以上打印结果可知：没有将block赋值给强指针时，在ARC环境下编译器不会自动执行copy操作
        } class]);
    }
    return 0;
}
```

* (3)block作为Cocoa的API中方法名含有usingBlock的方法的参数时。代码如下：

```
NSArray *arr1 = @[@"100",@"20",@"31",@"42",@"56"];
[arr1 enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
    NSLog(@"obj=%@-------idx=%lu",obj,(unsigned long)idx);
}];
```

* (4)block作为GCD API的方法参数时。比如GCD的一次性函数或者是延迟执行的函数，执行完block操作之后系统才会对block进行release操作。

GCD的一次性函数，代码如下：

```
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
    
});
```
GCD的延迟执行的函数，代码如下：

```
dispatch_after(dispatch_time(DISPATCH_TIME_NOW,(int64_t)(1.0 * NSEC_PER_SEC)),dispatch_get_main_queue(),^{
     
});
```

* (5)手动调用copy
             
**【扩展 1-5】在Block中修饰词__block和__weak的区别？**

二者区别如下：

* __block不管是ARC还是MRC模式下都可以使用，可以修饰对象，也可以修饰基本数据类型；
* __weak只能在ARC模式下使用，且只能修饰对象（如：NSString、NSArray），不能修饰基本数据类型（如int）；
* __block修饰的对象可以在block被重新赋值，__weak修饰的对象不可以；
* __block对象在ARC下可能会导致循环引用，MRC模式下可用来解决循环引用问题；
* __weak只能在ARC模式下使用，用来解决循环引用问题。

**【扩展 1-6】在ARC环境下，为什么使用__weak来修饰Objective-C对象，可以避免循环引用的问题？**

因为使用__weak修饰的对象是弱引用，编译器不会执行retain操作，对象的引用计数不会+1，在对象释放的时候__weak会将引用的对象置为nil，不会导致野指针的产生。需要注意的是，在多线程的情况下，除了在Block外使用__weak对对象进行弱引用外，我们还需要在Block内部对弱引用对象进行一次强引用（__strong）,这是因为，仅用__weak修饰的对象，在多线程情况下，可能会被释放，那么这个对象在Block执行的过程中就会变为nil，从而引起崩溃。

**【扩展 1-7】在MRC环境下，为什么使用__block来修饰Objective-C对象，可以避免循环引用的问题？**

因为当Block存储在堆上时，如果在Block内部引用了外部的对象，会对所引用的对象进行一次retain操作，加上__block可以避免对该对象执行retain操作。


**【扩展 1-8】为什么在声明Block时使用copy来修饰？**

block本身可以像对象那样被retain，和release。但是，block在创建的时候，它的内存是分配在栈(stack)上，而不是在堆(heap)上。他本身的作于域是属于创建时候的作用域，一旦在创建时候的作用域之外调用block将导致程序崩溃。使用retain也可以，但是block的retain行为默认是用copy的行为实现的，因为block变量默认是声明为栈变量的，为了能够在block的声明域之外使用，就必须要把block从栈(stack)拷贝（copy）到堆(heap)，所以说为了block属性声明和实际的操作一致，最好声明为copy。此外，通过copy操作来保证block被拷贝到堆上也是为了开发者可以控制block的生命周期，并对该block进行内存管理。

**【扩展 1-9】Block与代理的联系和区别？**

Block和代理的**共同点**：

* 1）二者都可以实现文件间（一般是**一对一**）的数据通信（数据回调传值）；
* 2）都可能导致循环引用，所以在定义代理属性时一定要使用weak来修饰。在block中一定要弱引用外部对象，比如__weak typeof(self)；
* 3）使用时，都需要先判断。调用代理方法时，先判断if([self.delegate respondsToSelector:])，调用Block时,要先判断if(self.myBlock){}，不判断的话，有可能会崩溃。

**不同点**：

* 1）从使用方面来说，代理相对来说比较繁琐，需要先创建代理协议、声明代理方法、声明代理属性，然后再实现代理方法、遵守代理协议、设置代理对象、调用代理方法。而Block就比较灵活，只需要声明和调用。另外代理一般只能在两个类之间通信，如果在多个类之间通信的话，实现起来非常繁琐，但是Block却可以无限的在多个类之间回调传值。
* 2）从编译运行方面来说，Block的运行成本高。Block出栈时，需要将使用的数据从栈内存拷贝到堆内存，对象的引用计数+1，使用完或者Block置为nil后会将Block变量销毁；而delegate只是保存了一个对象指针（一定要使用weak修饰delegate，不然也会循环引用），直接调用，没有额外消耗。
* 3）从安全性方面来说，Block更容易造成循环引用，解决的方法是使用__weak关键词修饰变量构成若引用。
* 4）从代码维护成本角度来说，代理的代码比较分散，调用和实现分别写在不同的类中。而Block是一种轻量级的回调，能够直接访问上下文，代码比较紧凑，方便阅读，易于维护。

【总结】Block和代理各有优缺点，我们需要根据具体场景，选择合适的回调方式。默认优先使用Block（简单回调和异步线程中推荐使用Block）。如果回调函数多于3个(即公共接口比较多时)，推荐使用代理；如果回调很频繁，次数很多，像UITableView，每次初始化、滑动、点击都会回调，推荐使用代理。

【延伸1】通知(NSNotificationCeter)，通知可以实现**一对多**的传值。通知实现步骤：注册监听者(addObserver方法)、发布通知(postNotificationName方法)、销毁监听对象(removeObserver方法)。

【延伸2】为什么代理要使用weak来修饰？

答：因为使用weak可以打破循环引用。

【延伸3】代理的delegate和dataSource有什么区别？

答：delegate是一个类委托另一个类实现某些方法，协议里面的方法主要是与操作相关的。dataSource的作用主要是一个类通过dataSource将数据发送给需要接受委托的类，协议里面的方法主要是和数据内容有关的。

【延伸4】说说常用的传值方式有哪些？

* (1)方法传值
* (2)属性传值：常用在从上一个页面向下一个页面传值，需要在下一个页面添加属性
* (3)delegate传值：需要定义协议方法，遵守协议、设置代理、实现代理。适用于一对一的使用场景。
* (4)通知传值：可用于一对多的场景。当通知不需要时要从通知中心移除。子线程发通知可能导致内存泄漏。
* (5)单例传值
* (6)Block传值：一般应用于需要回调的场景
* (7)数据存储传值：比如使用userDefaults、sql等。

**【扩展 1-10】block访问对象类型的auto变量时的内存管理原理是怎样的？**

当block内部访问了对象类型的auto变量时，如果block是在栈上（也就是_NSConcreteStackBlock类型的block），不论是ARC还是MRC，也不论是__strong修饰还是__weak修饰，那么都不会对auto变量产生强引用。

当block被拷贝到堆上时，那么会自动调用block内部的copy函数(__main_block_copy_0函数)，__main_block_copy_0函数内部会调用_Block_object_assign函数，然后_Block_object_assign函数会根据auto变量的修饰符(__strong、__weak、__unsafe_unretained)做出相应的操作，类似于retain（形成强引用或者弱引用）。

当block从堆中被移除时，会调用block内部的dispose函数(__main_block_dispose_0函数)，__main_block_dispose_0函数内部会调用_Block_object_dispose函数，_Block_object_dispose函数会自动释放引用的auto变量类似于release操作。

**【扩展 1-11】对象类型的auto变量和__block变量的异同？**

**相同点**：

* 当block在栈上时，对它们都不会产生强引用；
* 当block拷贝到堆上时，都会通过copy函数来处理它们；
* 当block从堆上移除时，都会通过dispose函数来释放它们。

**不同点**： 不同点主要在于引用方面（强引用还是弱引用）。

* 对于OC对象类型的auto变量来说，如果block是通过弱引用来访问OC对象的话，那么block对OC对象产生的弱引用；如果block是通过强引用来访问OC对象的话，那么block对OC对象产生的是强引用。
* 对于__block变量来说，block对__block变量直接产生的就是强引用。

**【扩展 1-12】ARC环境下解决循环引用问题的方法有哪些？应该首选哪种？为什么？**

ARC环境下解决循环引用的方法有以下3种：

* (1)__unsafe_unretained（不推荐）

```
//方法(1)
__unsafe_unretained typeof(self) weakSelf = self;
self.block = ^{
	NSLog(@"%p", weakSelf);
};
```

* (2)__block（不推荐）

```
//方法(2)
__block id weakSelf = self;
self.block = ^{
	NSLog(@"%p",weakSelf);
	weakSelf = nil;//必须写
};
self.block();//必须写
```

* (3)__weak（推荐）

```
//方法(3)
__weak typeof(self) weakSelf = self;
self.block = ^{
   __strong typeof(self)strongSelf = weakSelf;
   NSLog(@"%p", strongSelf);
};
```

ARC环境下，可以通过 __ unsafe_unretained 修饰符来解决，但是由于__ unsafe_unretained是不安全的，当指针指向的对象销毁时，指针存储的地址值不变，也就是不会自动将指针置为nil，从而产生野指针；所以**不推荐使用 __unsafe_unretained**。

ARC环境下，也可以通过__block来解决循环引用。缺点是必须要调用block，而且在block内部要将指向对象的指针置为nil。

ARC环境下，可以通过 __weak 修饰符来解决循环引用。__weak是安全的，当指针指向的对象销毁时，会自动将指针置为nil。

综上所述，优先**推荐使用__weak修饰符**来解决循环引用问题。

此外需要注意的是，**在多线程的情况下**，除了在 Block 外使用 __weak 对对象进行弱引用外，我们还需要在 Block 内部对弱引用对象再进行一次强引用（__strong）,这是因为，仅用__weak 修饰的对象，在多线程情况下，可能会被释放，那么这个对象在 Block 执行的过程中就会变为 nil，从而引起崩溃。



**【扩展 1-13】MRC环境下解决循环引用问题的方法有哪些？**

MRC环境下，不支持__weak，不能通过__weak来解决循环引用。但是可以通过 __unsafe_unretained 和 __block 来解决MRC环境下的循环引用问题。

* (1)__unsafe_unretained

```
__unsafe_unretained typeof(self) weakSelf = self;
self.block = ^{
	NSLog(@"%p", weakSelf);
};
```

* (2)__block

MRC环境下， __block修饰的对象，不会被 __block变量的结构体对象强引用，也就打破了循环引用。

```
__block id weakSelf = self;
self.block = ^{
	NSLog(@"%p",weakSelf);
};
```

**【扩展 1-14】ARC环境下 __ unsafe_unretained和__weak的异同点？**

**相同点**：

__ unsafe_unretained和__weak都是弱引用，不会产生强引用。

**不同点**：

__ unsafe_unretained是不安全的，当指针指向的对象销毁时，指针存储的地址值不变，也就是不会自动将指针置为nil，从而产生野指针；__weak是安全的，当指针指向的对象销毁时，会自动将指针置为nil。

**【扩展 1-15】__block的作用是什么?使用时需要注意什么?**

__block可以用于解决block内部无法修改auto变量值的问题。一旦使用 __block，那么编译器会将 __block变量包装成一个对象 ( __Block _byref _变量名 _0)。该对象内部包含isa指针以及与外部auto变量同名且同类型的成员变量。

使用注意点：注意 __block的内存管理问题；再就是在MRC环境下，使用__block修饰的对象类型不会被block强引用。

**【扩展 1-16】在block内部修改NSMutableArray时，需不需要加__block？**

不需要。

```
NSMutableArray *array = [NSMutableArray array];
HLBlok block = ^{
    [array addObject:@"111"];
};
```
原因：这并不是修改array指针所指向的内存里面的值，而是拿array指针来使用。这与使用赋值符"="不同。所以这种情况下不需要使用__block来修饰array。

**【扩展 1-17】说说你对 block 的理解。**

可以从以下几方面切入：三种 block，栈上的自动复制到堆上，block 的属性修饰符是 copy，循环引用的原理和解决方案。

**【扩展 1-18】KVO、Notification、delegate 各自的优缺点，效率还有使用场景（阿里）**

**【扩展 1-19】block 为什么会有循环引用？（阿里）**

**【扩展 1-20】使用系统的某些block api（如UIView的block版本写动画时），是否也要考虑循环引用问题？**

系统的某些block api中，UIView的 block 版本写动画时不需要考虑，但是也有一些api需要考虑循环引用问题。

所谓“循环引用”是指双向的强引用。所以那些“单向的强引用”（block强引用self）不会产生循环引用问题。比如：

```
//(1)
[UIView animateWithDuration:duration animations:^{ 
	[self.superview layoutIfNeeded];	
}];
//(2)
[[NSOperationQueue mainQueue] addOperationWithBlock:^{ 
	self.someProperty = xyz; 
}];
//(3)
[[NSNotificationCenter defaultCenter] addObserverForName:@"someNotification" object:nil queue:[NSOperationQueue mainQueue] usingBlock:^(NSNotification * notification) {
	self.someProperty = xyz; 
}];
```

以上三种情况只是block强引用self，即单向强引用，不会产生循环引用问题。

但是如果你使用一些参数中可能含有 ivar 的系统 api ，如 GCD 、 NSNotificationCenter就要⼩心一点:比如GCD 内部如果引⽤了 self，⽽且 GCD 的 其他参数是 ivar，则要考虑到循环引用问题。比如下面两种情况：

```
//情况1
__weak __typeof__(self) weakSelf = self;
dispatch_group_async(_operationsGroup, _operationsQueue, ^ {
	__typeof__(self) strongSelf = weakSelf; 
	[strongSelf doSomething];
	[strongSelf doSomethingElse];
});
//情况2
__weak __typeof__(self) weakSelf = self;
_observer = [[NSNotificationCenter defaultCenter] addObserverForName:@"testKey"
								object:nil queue:nil
								usingBlock:^(NSNotification *note) { 
	__typeof__(self) strongSelf = weakSelf;
	[strongSelf dismissModalViewControllerAnimated:YES]; 
}];
//self --> _observer --> block --> self 显然这也是⼀一个循环引用。
```

检测代码中是否存在循环引用问题，可使⽤ Facebook 开源的一个检测工具 FBRetainCycleDetector 。


**【扩展 1-21】使用 block 有什么好处？如何使用 NSTimer 写出一个使用 block 显示（在 UILabel 上）秒表？**

使用 block 的好处：代码紧凑；传值和回调都很方便；相比代理，代码更简洁。

```
NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:1.0 repeate:YES callback:^{
   weakSelf.secondsLabel.text = ...
}];
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

**【扩展 1-22】解释以下代码产生内存泄漏的原因？**

```
@implementation HLTestViewController

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
   HLTestCell *cell = [tableView dequeueReuseableCellWithIdentifier:@"testCell" forIndexPath:indexPath];
   [cell setTouchBlock:^(HLTestCell *cell){
      [self refreshData];
   }];
   return cell;
}
```

**内存泄漏的原因**：存在循环引用。在给 cell 设置的 touchBlock 中，使用了 __strong 修饰的 self，由于 Block 的底层实现原理，当 touchBlock 从栈复制到堆中时，self 会被一同复制到堆中，retain 一次，此时 self 被 touchBlock 持有（强引用），而 touchBlock 又是被 cell 持有的，cell 又被 tableView 持有，tableView 又被 self 持有，这样一来，self 间接持有 touchBlock，touchBlock 持有 self，两个对象都被彼此强引用，引用计数始终不能变为0，无法释放，因此形成了循环引用。


解决方法：

```
__weak __typeof__(self) weakSelf = self;
[cell setTouchBlock:^(HLTestCell *cell){
      [weakSelf refreshData];
   }];
```

