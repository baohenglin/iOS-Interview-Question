# 知识点汇总篇章1

## 知识点一：简述OC的反射机制

反射机制是指类名、方法名、属性名等可以和字符串相互转化（反射），而这些转化是发生在运行时的，所以我们可以用这个机制来动态地获取类、方法或属性，从而动态的创建类对象、调用方法、或给属性赋值、判断类型等。OC的反射机制类似于Java的反射机制 ，这种动态机制可以让OC语言变得更加灵活。OC的反射机制有三个用途：（1）通过类名的字符串来实例化对象;（2）将类名转化为字符串；（3）动态的调用方法；（4）SEL反射；（5）将方法变成字符串

（1）通过类名的字符串来实例化对象

```
Class className = NSClassFromString(@"HomeViewController");
NSLog(@"className = %@",className);
```

（2）将类名转化为字符串：

```
Class class = [HomeViewController class];
NSString *className = NSStringFromClass(class);
```

（3）动态地调用方法：

动态调用方法的应用场景：假设有一天公司产品要实现一个需求：根据后台推送过来的数据，进行动态页面跳转，跳转到页面后根据返回到数据执行对应的操作。遇到这样奇葩的需求，我们当然可以问产品都有哪些情况执行哪些方法，然后写一大堆if else判断或switch判断。但是这种方法实现起来太low了，而且不够灵活，假设后续版本需求变了，还要往其他已有页面中跳转，这样写也不利于代码维护与扩展。此时，反射机制就派上用场了，我们可以用反射机制动态的创建类并执行某方法。此外，我们也可以通过runtime来实现这个功能。

```
// 假设从服务器获取JSON串，通过这个JSON串获取需要创建的类的类名为FirstViewController，进而调用FirstViewController类的getDataList方法。
Class class = NSClassFromString(@"FirstViewController");
SEL selector = NSSelectorFromString(@"getDataList");
[[[class alloc] init] performSelector:selector];
```

（4）SEL反射(将字符串转化为方法)

```
Class className = NSClassFromString(@"HomeViewController");
HomeViewController *homeVC1 = [[class alloc]init];
//SEL反射
SEL selector = NSSelectorFromString(@"setName:");
[homeVC1 performSelector:selector withObject:@"zhang"];
```

（5）将方法变成字符串

```
NSString *selectorName1 = NSStringFromSelector(@selector(setName:));
```

**【拓展一】获取Class类的三种方法**：

* 1)通过字符串来获得Class，此方法用到了反射机制。

```
Class className = NSClassFromString(@"HomeViewController");
NSLog(@"className = %@",className);
```
* 2)通过实例对象来获得Class

```
HomeViewController *mainVC = [[HomeViewController alloc]init];
NSLog(@"className = %@",[mainVC class]);
```
* 3)通过类来获得Class

```
NSLog(@"className=%@",[HomeViewController class]);
```

**【拓展二】检查继承关系的方法**

```
HomeViewController *testVC = [[HomeViewController alloc]init];
NSLog(@"[testVC class]=%@",[testVC class]);
//2.判断对象是否为某个类的实例对象
NSLog(@"testVC是否为ViewController的实例对象:%d",[testVC isMemberOfClass:HomeViewController.class]);
//3.判断实例对象是否为某个类及其子类的实例
NSLog(@"[testVC isKindOfClass:[ViewController class]] = %d",[testVC isKindOfClass:[HomeViewController class]]);
//4.判断实例对象是否实现了指定的协议
NSLog(@"判断是否实现了指定协议=%d",[testVC conformsToProtocol:@protocol(UITableViewDelegate)]);
```

## 知识点二：block的本质是什么？block的内存结构是怎样的？一共有几种block？都是什么情况下生成的？

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

**【扩展 2-1】默认情况下能修改被block捕获的自动变量么？为什么？__block修饰词都做了什么？**

默认情况下，不能修改block捕获的自动变量值。因为block捕获自动变量（局部变量）传递到block变量所指向的结构体内部的是自动变量的值（也就是“值传递”），而不是指向自动变量内存的指针（不是“地址传递”），所以block内部不能改变block捕获的变量。

使用__block修饰的自动变量传递到block变量所指向的结构体内部的是 **指向变量内存地址的指针**，故可以在Block内部改变变量值。

此外，block内部可以修改捕获到的静态变量、全局变量和静态全局变量。全局变量和静态全局变量由于作用域是全局，且存储在全局区，供所有函数调用，所以可以在block内被修改；静态变量之所以可以在block内被修改是因为传递给block的是内存的指针。全局变量、静态全局变量和函数参数它们并没有变成Block结构体__main_block_impl_0的成员变量，并不会被Block持有，所以也不会增加retainCount的值。

综上所述，在声明Block之后、调用Block之前对局部变量进行修改，那么调用Block时，捕获到的局部变量值是修改之前的旧值(原因是在声明Block时便已经将局部变量的值传递到Block变量所指向的结构体)；默认情况下，在Block中不可以直接修改局部变量。


**【扩展 2-2】在block内修改自动变量的值有几种方式？如何修改？**

在block内修改自动变量的值有两种方式。1）将内存地址指针传递到block中，也就是使用__block来修饰局部变量(推荐)；2）改变存储方式，可以修改为static静态变量、静态全局变量或全局变量（不推荐，因为此方法会使变量一直占用内存，不会释放内存）

**【扩展 2-3】这三种block是在什么情况下产生的？以及它们之间的区别和联系是什么？**

1）从捕获外部变量的角度上来看：

 * _NSConcreteGlobalBlock **没有访问局部变量（自动变量/auto变量）**或只访问了全局变量、静态变量的block为_NSConcreteGlobalBlock，生命周期从创建到应用程序结束。
 * _NSConcreteStackBlock 访问了auto变量，并且没有强指针引用和copy操作的block都是StackBlock，StackBlock的生命周期由系统控制，一旦返回之后，就被系统销毁了；
 * _NSConcreteMallocBlock 有强指针引用或 copy修饰的属性 引用的block会被复制到堆中而成为MallocBlock，没有强指针引用即销毁，生命周期由程序员控制
     
2)从持有对象的角度上来看：

* _NSConcreteGlobalBlock是不持有对象的
* _NSConcreteStackBlock也是不持有对象的
* _NSConcreteMallocBlock是持有对象的。

**【扩展 2-4】什么情况下ARC会自动将Block从栈上拷贝到堆上呢？**

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
             
**【扩展 2-5】在Block中修饰词__block和__weak的区别？**

二者区别如下：

* __block不管是ARC还是MRC模式下都可以使用，可以修饰对象，也可以修饰基本数据类型；
* __weak只能在ARC模式下使用，且只能修饰对象（如：NSString、NSArray），不能修饰基本数据类型（如int）；
* __block修饰的对象可以在block被重新赋值，__weak修饰的对象不可以；
* __block对象在ARC下可能会导致循环引用，MRC模式下可用来解决循环引用问题；
* __weak只能在ARC模式下使用，用来解决循环引用问题。

**【扩展 2-6】在ARC环境下，为什么使用__weak来修饰Objective-C对象，可以避免循环引用的问题？**

因为使用__weak修饰的对象是弱引用，编译器不会执行retain操作，对象的引用计数不会+1，在对象释放的时候__weak会将引用的对象置为nil，不会导致野指针的产生。需要注意的是，在多线程的情况下，除了在Block外使用__weak对对象进行弱引用外，我们还需要在Block内部对弱引用对象进行一次强引用（__strong）,这是因为，仅用__weak修饰的对象，在多线程情况下，可能会被释放，那么这个对象在Block执行的过程中就会变为nil，从而引起崩溃。

**【扩展 2-7】在MRC环境下，为什么使用__block来修饰Objective-C对象，可以避免循环引用的问题？**

因为当Block存储在堆上时，如果在Block内部引用了外部的对象，会对所引用的对象进行一次retain操作，加上__block可以避免对该对象执行retain操作。


**【扩展 2-8】为什么在声明Block时使用copy来修饰？**

block本身可以像对象那样被retain，和release。但是，block在创建的时候，它的内存是分配在栈(stack)上，而不是在堆(heap)上。他本身的作于域是属于创建时候的作用域，一旦在创建时候的作用域之外调用block将导致程序崩溃。使用retain也可以，但是block的retain行为默认是用copy的行为实现的，因为block变量默认是声明为栈变量的，为了能够在block的声明域之外使用，就必须要把block从栈(stack)拷贝（copy）到堆(heap)，所以说为了block属性声明和实际的操作一致，最好声明为copy。此外，通过copy操作来保证block被拷贝到堆上也是为了开发者可以控制block的生命周期，并对该block进行内存管理。

**【扩展 2-9】Block、代理的联系和区别？**

Block和代理的**共同点**：

* 1）二者都可以实现文件间（一般是**一对一**）的数据通信（数据回调传值）；
* 2）都可能导致循环引用，所以在定义代理属性时一定要使用weak来修饰。在block中一定要弱引用外部对象，比如__weak typeof(self)；
* 3）使用时，都需要先判断。调用代理方法时，先判断if([self.delegate respondsToSelector:])，调用Block时,要先判断if(self.myBlock){}，不判断的话，有可能会崩溃。

**不同点**：

* 1）从使用方面来说，代理相对来说比较繁琐，需要先创建代理协议、声明代理方法、声明代理属性，然后再实现代理方法、遵守代理协议、设置代理对象、调用代理方法。而Block就比较灵活，只需要声明和调用。另外代理一般只能在两个类之间通信，如果在多个类之间通信的话，实现起来非常繁琐，但是Block却可以无限的在多个类之间回调传值。
* 2）从编译运行方面来说，Block的运行成本高。Block出栈时，需要将使用的数据从栈内存拷贝到堆内存，对象的引用计数+1，使用完或者Block置为nil后会将Block变量销毁；而delegate只是保存了一个对象指针（一定要使用weak修饰delegate，不然也会循环引用），直接调用，没有额外消耗。
* 3）从安全性方面来说，Block更容易造成循环引用，解决的方法是使用__weak关键词修饰变量构成若引用。
* 4）从代码维护成本角度来说，代理的代码比较分散，调用和实现分别写在不同的类中。而Block是一种轻量级的回调，能够直接访问上下文，代码比较紧凑，方便阅读，易于维护。

【总结】Block和代理各有优缺点，我们需要根据具体场景，选择合适的回调方式，默认优先使用Block。如果回调函数多余3个，推荐使用代理；如果回调很频繁，次数很多，像UITableView，每次初始化、滑动、点击都会回调，推荐使用代理。

【延伸】通知(NSNotificationCeter)，通知可以实现**一对多**的传值。通知实现步骤：注册监听者(addObserver方法)、发布通知(postNotificationName方法)、销毁监听对象(removeObserver方法)。

**【扩展 2-10】block访问对象类型的auto变量时的内存管理原理是怎样的？**

当block内部访问了对象类型的auto变量时，如果block是在栈上（也就是_NSConcreteStackBlock类型的block），不论是ARC还是MRC，也不论是__strong修饰还是__weak修饰，那么都不会对auto变量产生强引用。

当block被拷贝到堆上时，那么会自动调用block内部的copy函数(__main_block_copy_0函数)，__main_block_copy_0函数内部会调用_Block_object_assign函数，然后_Block_object_assign函数会根据auto变量的修饰符(__strong、__weak、__unsafe_unretained)做出相应的操作，类似于retain（形成强引用或者弱引用）。

当block从堆中被移除时，会调用block内部的dispose函数(__main_block_dispose_0函数)，__main_block_dispose_0函数内部会调用_Block_object_dispose函数，_Block_object_dispose函数会自动释放引用的auto变量类似于release操作。

**【扩展 2-11】对象类型的auto变量和__block变量的异同？**

**相同点**：

* 当block在栈上时，对它们都不会产生强引用；
* 当block拷贝到堆上时，都会通过copy函数来处理它们；
* 当block从堆上移除时，都会通过dispose函数来释放它们。

**不同点**： 不同点主要在于引用方面（强引用还是弱引用）。

* 对于OC对象类型的auto变量来说，如果block是通过弱引用来访问OC对象的话，那么block对OC对象产生的弱引用；如果block是通过强引用来访问OC对象的话，那么block对OC对象产生的是强引用。
* 对于__block变量来说，block对__block变量直接产生的就是强引用。

**【扩展 2-12】ARC环境下解决循环引用问题的方法有哪些？应该首选哪种？为什么？**

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
	NSLog(@"%p", weakSelf);
};
```

ARC环境下，可以通过 __ unsafe_unretained 修饰符来解决，但是由于__ unsafe_unretained是不安全的，当指针指向的对象销毁时，指针存储的地址值不变，也就是不会自动将指针置为nil，从而产生野指针；所以**不推荐使用 __unsafe_unretained**。

ARC环境下，也可以通过__block来解决循环引用。缺点是必须要调用block，而且在block内部要将指向对象的指针置为nil。

ARC环境下，可以通过 __weak 修饰符来解决循环引用。__weak是安全的，当指针指向的对象销毁时，会自动将指针置为nil。

综上所述，优先**推荐使用__weak修饰符**来解决循环引用问题。


**【扩展 2-13】MRC环境下解决循环引用问题的方法有哪些？**

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

**【扩展 2-14】ARC环境下 __ unsafe_unretained和__weak的异同点？**

**相同点**：

__ unsafe_unretained和__weak都是弱引用，不会产生强引用。

**不同点**：

__ unsafe_unretained是不安全的，当指针指向的对象销毁时，指针存储的地址值不变，也就是不会自动将指针置为nil，从而产生野指针；__weak是安全的，当指针指向的对象销毁时，会自动将指针置为nil。

**【扩展 2-15】__block的作用是什么?使用时需要注意什么?**

__block可以用于解决block内部无法修改auto变量值的问题。一旦使用 __block，那么编译器会将 __block变量包装成一个对象 ( __Block _byref _变量名 _0)。该对象内部包含isa指针以及与外部auto变量同名且同类型的成员变量。

使用注意点：注意 __block的内存管理问题；再就是在MRC环境下，使用__block修饰的对象类型不会被block强引用。

**【扩展 2-16】在block内部修改NSMutableArray时，需不需要加__block？**

不需要。

```
NSMutableArray *array = [NSMutableArray array];
HLBlok block = ^{
    [array addObject:@"111"];
};
```
原因：这并不是修改array指针所指向的内存里面的值，而是拿array指针来使用。这与使用赋值符"="不同。所以这种情况下不需要使用__block来修饰array。

## 知识点三：分类(category)和类扩展(extension)

**【扩展 3-1】分类(category)应用场景有哪些？分类有哪些局限性？分类的结构体里面有哪些成员？**

Category是Objective-C 2.0之后增加的语言特性，Category的主要作用是为已经存在的类添加方法。Category是对装饰模式的典型实践（装饰模式(Decorator)是指在不修改原有类的前提下，动态地给这个类添加一些方法）。

**分类(Category)的应用场景包括以下几个**：

* (1)可以为现有类添加新的方法；
* (2)模拟多继承
* (3)将一个类中的代码分散出来管理（将类的实现分布到几个不同文件中）
* (4)把framework的私有方法公开

**分类(Category)的局限性**包括以下几点：

* (1)Category只能给某个现有的类添加方法，不能添加成员变量；
* (2)Category中也可以添加属性，只不过@property只会生成setter和getter的声明，不会生成setter方法和getter方法的实现，也不会自动合成带下划线的成员变量；
* (3)如果category中的方法和类中原有方法同名，运行时会优先调用category中的方法。也就是，category中的方法会覆盖掉类中原有的方法。所以开发中尽量保证不要让分类中的方法和原有类中的方法名相同。避免出现这种情况的解决方案是给分类的方法名统一添加前缀。比如category_。

分类的结构体struct_category_t里面的成员变量包括：对象方法列表、类方法列表、属性列表和协议列表等信息。


**【扩展 3-2】分类(category)的实现原理？**

Category编译之后的底层结构是struct_category_t类型的的结构体，这个结构体里面存储着分类的对象方法、类方法、属性、协议等信息。在程序运行的时候，runtime会将Category的数据，合并到类信息中（类对象、元类对象中）。runtime加载Category的大致过程是这样的：首先通过Runtime加载某个类的所有Category数据;然后把所有Category的方法、属性、协议数据，合并到一个大数组中，后面参与编译的Category数据，会写到数组的最前面；最后将合并后的所有分类数据（方法、属性、协议），插入到类原来数据的前面。

【注意】category并没有完全的替换掉原有类的同名方法，category的方法被放置在新方法列表的前面，而原来类的方法被放到后面，在runtime中，遍历方法列表查找时，找到了category的方法后，就会停止遍历，这就是我们平时所说的“覆盖”方法。此外，某个类如果存在两个category，而且这两个Category中存在方法名相同的方法，那么根据buildPhases->Compile Sources里面的从上至下的编译顺序，会调用后参与编译的那个分类的方法。

[分类(category)的实现原理详解](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E5%BC%80%E5%8F%91%E4%B9%8BCategory%E6%8E%A2%E7%A9%B6.md)


**【扩展 3-3】分类(Category)和类扩展(Class Extension)有什么区别？**

分类(Category)是Objective-C 2.0之后增加的语言特性，Category的主要作用是为已经存在的类添加方法。

类扩展（Extension）是category的一个特例，也称为匿名分类（类扩展与分类相比只少了分类的名称）。类扩展的作用是为一个类添加一些额外的私有属性、成员变量和私有方法。

**不同点：**

* (1)类扩展即可以添加方法又可以添加成员变量；分类(Category)中只能添加方法，不能添加成员变量
* (2)在类扩展(Extension)中添加的新方法一定要实现，而分类(category)中没有这种限制
* (3)类扩展中声明的方法没被实现，编译器会报警告，但是分类(Category)中的方法没被实现编译器是不会有任何警告的（因为类扩展是在编译阶段被添加到类中，而类别是在运行时添加到类中）
* (4)Class Extension是在编译期决定，Category由运行时决定。也就是说Class extension在编译的时候，它的数据就已经包含在类信息中；而Category（分类）是在运行时，才会将分类数据合并到类信息中（类对象、元类对象中）

**【扩展 3-4】Category能否给类添加成员变量？如果可以,如何给Category添加成员变量？**

默认情况下，由于分类底层结构的限制，不能直接给Category添加成员变量,但是可以间接实现。通过“关联对象”的方式来间接给Category添加成员变量。也就是通过“<objc/runtime.h>”中提供的关联对象的API来实现。“<objc/runtime.h>”中提供的关联对象的API有以下3个：

(1)添加关联对象

```
void objc_setAssociatedObject(id object, const void * key,id value, objc_AssociationPolicy policy)
```

(2)获得关联对象

```
id objc_getAssociatedObject(id object, const void * key)
```

(3)移除所有的关联对象

```
void objc_removeAssociatedObjects(id object)
```

例如给HLPerson类的Category分类HLPerson+Test类添加一个_name成员变量和一个_weight成员变量。代码如下：

```
#import "HLPerson.h"
@interface HLPerson (Test)
@property (copy, nonatomic) NSString *name;
@property (assign, nonatomic) int weight;
@end
```

```
#import "HLPerson+Test.h"
#import <objc/runtime.h>
@implementation HLPerson (Test)

- (void)setName:(NSString *)name
{
    objc_setAssociatedObject(self, @selector(name), name, OBJC_ASSOCIATION_COPY_NONATOMIC);
}
- (NSString *)name
{
    // 隐式参数
    // _cmd == @selector(name)
    return objc_getAssociatedObject(self, _cmd);
}

- (void)setWeight:(int)weight
{
    objc_setAssociatedObject(self, @selector(weight), @(weight), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
- (int)weight
{
    // _cmd == @selector(weight)
    return [objc_getAssociatedObject(self, _cmd) intValue];
}
@end
```

**【扩展 3-5】Category中有load方法吗？load方法是什么时候调用的？load方法能继承吗？**

Category中有+load方法。load方法在runtime（运行时）加载类、分类的时候调用。+load方法能继承。而且+load方法一般都由系统自动调用，不手动调用。

**【扩展 3-6】load方法和initialize方法的区别是什么？它们在category中的调用顺序是怎样的？以及出现继承时它们之间的调用过程是怎样的？**

**load方法和initialize方法的相同点**：

如果父类和子类的load方法或initialize方法都被调用,那么父类的调用一定在子类之前。

**load方法和initialize方法的区别**：

* （1）调用方式不同。load是根据函数地址直接调用，而initialize是通过objc_msgSend调用
* （2）调用时刻不同。load是在runtime加载类、分类的时候由系统自动调用（在main函数执行之前被调用而且每个类的load方法只会调用1次），而initialize是类第一次接收到消息的时候调用，每一个类只会initialize一次（父类的initialize方法可能会被调用多次，这是因为有些子类可能没有实现initialize方法，那么在初始化子类时就会调用父类的initialize方法）。

**+load方法在Category中的调用顺序如下**：

* (1)先调用类的+load方法，再调用分类的+load方法；
* (2)先调用父类的+load方法，再调用子类的+load方法；
* (3)先编译“先参与编译的类”，后编译“后参与编译的类”；
* (4)当子类未实现load方法时,不会调用父类load方法。

**initialize方法在Category中的调用顺序如下**：

* (1)先执行父类的initialize方法再执行子类的initialize；
* (2)当子类未实现initialize方法时,会调用父类initialize方法,子类实现initialize方法时,会覆盖父类initialize方法;
* (3)当有多个Category都实现了initialize方法,会覆盖类中的方法,只执行一个(会执行Compile Sources 列表中最后一个Category 的initialize方法)


【注意】: 

load调用时机比较早,当load调用时,其他类可能还没加载完成,运行环境不安全，所以我们应该尽量减少load方法的逻辑。load方法是线程安全的，它使用了锁，我们应该避免线程阻塞在load方法。

在initialize方法收到调用时,运行环境基本健全。 initialize内部也使用了锁，所以是线程安全的。但同时要避免阻塞线程，不要再使用锁。

**+load方法的使用场景**：交换两个方法的实现

```
//摘自MJRefresh
+ (void)load
{
    [self exchangeInstanceMethod1:@selector(reloadData) method2:@selector(mj_reloadData)];
    [self exchangeInstanceMethod1:@selector(reloadRowsAtIndexPaths:withRowAnimation:) method2:@selector(mj_reloadRowsAtIndexPaths:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(deleteRowsAtIndexPaths:withRowAnimation:) method2:@selector(mj_deleteRowsAtIndexPaths:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(insertRowsAtIndexPaths:withRowAnimation:) method2:@selector(mj_insertRowsAtIndexPaths:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(reloadSections:withRowAnimation:) method2:@selector(mj_reloadSections:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(deleteSections:withRowAnimation:) method2:@selector(mj_deleteSections:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(insertSections:withRowAnimation:) method2:@selector(mj_insertSections:withRowAnimation:)];
}

+ (void)exchangeInstanceMethod1:(SEL)method1 method2:(SEL)method2
{
    method_exchangeImplementations(class_getInstanceMethod(self, method1), class_getInstanceMethod(self, method2));
}

```

**+initialize方法的使用场景**：主要用来对一些不方便在编译期初始化的对象进行赋值。比如NSMutableArray这种类型的实例化依赖于runtime的消息发送，所以显然无法在编译器初始化：

```
// int类型可以在编译期赋值
static int someNumber = 0; 
static NSMutableArray *someArray;
+ (void)initialize {
    if (self == [Person class]) {
        // 不方便编译期赋值的对象在这里赋值
        someArray = [[NSMutableArray alloc] init];
    }
}

```



<br />
<br />
<br />

**参考链接：**

* [Objective-C +load vs +initialize](http://blog.leichunfeng.com/blog/2015/05/02/objective-c-plus-load-vs-plus-initialize/)
* [Objective-C Category 的实现原理](http://blog.leichunfeng.com/blog/2015/05/18/objective-c-category-implementation-principle/)
* [iOS类方法load和initialize详解](https://juejin.im/post/5a31dc40f265da4307034712)




