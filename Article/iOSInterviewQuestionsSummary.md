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

（3）动态的调用方法：

假设有一天公司产品要实现一个需求：根据后台推送过来的数据，进行动态页面跳转，跳转到页面后根据返回到数据执行对应的操作。遇到这样奇葩的需求，我们当然可以问产品都有哪些情况执行哪些方法，然后写一大堆if else判断或switch判断。但是这种方法实现起来太low了，而且不够灵活，假设后续版本需求变了，还要往其他已有页面中跳转，这样写也不利于代码维护与扩展。此时，反射机制就派上用场了，我们可以用反射机制动态的创建类并执行某方法。此外，我们也可以通过runtime来实现这个功能。

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

Block本质上是Objective-C实例对象。这是因为Block内部有一个isa指针。更确切地说，block是封装了函数调用以及函数调用环境的OC的对象。一个block实例实际上由以下几部分组成的：

* 1）isa指针
* 2）flags：用于按bit位表示一些block的附加信息
* 3）reserved：保留变量
* 4）invoke：函数指针，指向具体block实现的函数调用地址
* 5）descriptor：表示该block的附加描述信息，主要是结构体内存的大小(size)以及copy和dispose函数的指针
* 6）variables：捕获的变量，block能够访问它外部的局部变量，就是因为将这些变量（或变量的地址）复制到了结构体中。


在Objective-C语言中，根据存储位置可以分为3种类型的block：

* 1）NSGlobalBlock(_NSConcreteGlobalBlock)   全局静态block（产生条件：**没有访问auto变量**）
* 2）NSStackBlock(_NSConcreteStackBlock)    保存在栈中的block，当函数返回(超出函数作用域)时会被销毁。(产生条件：**在MRC下，访问了auto变量**)
* 3）NSMallocBlock(_NSConcreteMallocBlock)  保存在堆中的block，当引用计数为0时会被销毁。(产生条件：**NSStackBlock调用了copy方法，就会将block内存搬到堆上，变成了__NSMallocBlock**)

其实，这3种类型的Block都继承自NSBlock类型。

Block是苹果在iOS4开始引入的对C语言的扩展，用来实现匿名函数的特性。Block是一种特殊的数据类型，其可以正常定义变量、作为参数、作为返回值。特殊地，Block还可以保存一段代码，在需要的时候调用，目前Block已经广泛应用于iOS开发中，常用于GCD、动画、排序以及各类回调。

【注意】在MRC的情况下，如果访问了auto变量，那么生成的是__NSStackBlock。__NSStackBlock存在一个问题，也就是超出作用域后变量就会被系统自动销毁，变量被系统自动销毁后如果再访问该变量那么就会存在安全问题。如果开启了ARC，那么编译器就会自动进行copy操作，将__NSStackBlock转变为__NSMallocBlock。

**【扩展 2-1】默认情况下能修改被block捕获的自动变量么？为什么？__block修饰词都做了什么？**

默认情况下，不能修改block捕获的自动变量值。因为block捕获自动变量（局部变量）传递到block变量所指向的结构体内部的是自动变量的值（也就是“值传递”），而不是指向自动变量内存的指针（不是“地址传递”），所以block内部不能改变block捕获的变量。

使用__block修饰的自动变量传递到block变量所指向的结构体内部的是 指向变量内存的指针，故可以在Block内部改变变量值。

此外，block内部可以修改捕获到的静态变量、全局变量和静态全局变量。全局变量和静态全局变量由于作用域是全局，且存储在全局区，供所有函数调用，所以可以在block内被修改；静态变量之所以可以在block内被修改是因为传递给block的是内存的指针。全局变量、静态全局变量和函数参数它们并没有变成Block结构体__main_block_impl_0的成员变量，并不会被Block持有，所以也不会增加retainCount的值。

综上所述，在声明Block之后、调用Block之前对局部变量进行修改，那么调用Block时，捕获到的局部变量值是修改之前的旧值(原因是在声明Block时便已经将局部变量的值传递到Block变量所指向的结构体)；默认情况下，在Block中不可以直接修改局部变量。





















