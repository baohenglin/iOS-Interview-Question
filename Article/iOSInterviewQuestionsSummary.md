# 面试题题集一

## 题目一：简述OC的反射机制

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

## 题目二：block的本质是什么？block的内存结构是怎样的？一共有几种block？都是什么情况下生成的？

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

使用__block修饰的自动变量传递到block变量所指向的结构体内部的是 指向变量内存的指针，故可以在Block内部改变变量值。

此外，block内部可以修改捕获到的静态变量、全局变量和静态全局变量。全局变量和静态全局变量由于作用域是全局，且存储在全局区，供所有函数调用，所以可以在block内被修改；静态变量之所以可以在block内被修改是因为传递给block的是内存的指针。全局变量、静态全局变量和函数参数它们并没有变成Block结构体__main_block_impl_0的成员变量，并不会被Block持有，所以也不会增加retainCount的值。

综上所述，在声明Block之后、调用Block之前对局部变量进行修改，那么调用Block时，捕获到的局部变量值是修改之前的旧值(原因是在声明Block时便已经将局部变量的值传递到Block变量所指向的结构体)；默认情况下，在Block中不可以直接修改局部变量。


**【扩展 2-2】在block内修改自动变量的值有几种方式？如何修改？**

在block内修改自动变量的值有两种方式。1）将内存地址指针传递到block中(使用__block来修饰)；2）改变存储方式，可以修改为static静态变量、静态全局变量或全局变量

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

block本身可以像对象那样被retain，和release。但是，block在创建的时候，它的内存是分配在栈(stack)上，而不是在堆(heap)上。他本身的作于域是属于创建时候的作用域，一旦在创建时候的作用域之外调用block将导致程序崩溃。使用retain也可以，但是block的retain行为默认是用copy的行为实现的，因为block变量默认是声明为栈变量的，为了能够在block的声明域之外使用，就必须要把block从栈(stack)拷贝（copy）到堆(heap)，所以说为了block属性声明和实际的操作一致，最好声明为copy。
































