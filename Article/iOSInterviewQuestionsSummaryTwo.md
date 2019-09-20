# 知识点汇总篇章二

## 知识点4：OC对象

**【扩展 4-1】一个NSObject对象占用多少内存？**

系统分配了16个字节给NSObject对象（通过malloc_size函数获得），但NSObject对象内部只使用了8个字节的空间（64bit环境下，可以通过class_getInstanceSize函数获得）。

**【扩展 4-2】OC的类信息存放在哪里？**

* (1)对象方法、属性、成员变量、协议信息，存放在class对象中；
* (2)类方法，存放在meta-class对象中；
* (3)成员变量的具体值，存放在instance对象中。

**【扩展 4-3】对象的isa指针指向哪里？**

* instance对象的isa指向class对象
* class对象的isa指向meta-class对象
* meta-class对象的isa指向基类(Root class)的meta-class对象

**【扩展 4-4】对象的superclass指针指向哪里？**

* class(类对象)的superclass指针指向父类的class(类对象)；如果没有父类，那么superclass指针为nil。
* meta-class(元类对象)的superclass指针指向父类的meta-class。特别需要注意的是基类的meta-class(元类对象)的superclass指针指向基类的class对象(类对象)。

**【扩展 4-5】instance对象(实例对象)调用对象方法的原理？**

由于对象方法存储在class对象（类对象）的内存中，因此instance对象(实例对象)调用对象方法的原理是首先通过instance(实例对象)的isa指针找到它自己的class对象（类对象），查找自己的class对象中是否存在打算调用的对象方法。如果存在，则进行调用；如果不存在该对象方法，那么就会通过自己class对象中的superclass指针找到它的父类的class对象并查找其中是否存在将要调用的对象方法，如果存在，进行调用；如果不存在，那么就再通过这个父类的class对象的superclass指针，继续往上查找...当查找到基类(NSOject)的class对象（类对象）时，如果此时存在该对象方法就立刻调用，如果基类中也不存在该对象方法，那么意味着自始至终都没有找到该对象方法，此时会报错“unrecognized selector sent to instance”。

**【扩展 4-6】class对象(类对象)调用类方法的原理？**

由于类方法都存储在meta-class对象（元类对象）中，因此class对象(类对象)调用类方法的原理是首先通过class对象的isa指针找到自己的meta-class对象，查看是否存在将要调用的类方法，如果存在，立即调用；如果不存在，则通过该meta-class对象中的superclass指针找到父类的meta-class对象（元类对象）并查找其中是否存在将要调用的类方法，如果存在，立刻调用，如果不存在，则通过superclass指针继续一层一层往上查找，直至找到基类(Root class)的元类对象，并查找基类(Root class)的元类对象中是否存在将要调用类方法，如果存在立即调用，如果也不存在，此时并不会报“unrecognized selector sent to instance”错误，而是会继续通过superclass指针找到基类的class对象（类对象），如果基类的class对象中存在将要调用的类方法就立刻调用，如果不存在，此时才会报错(“unrecognized selector sent to instance”)。

**【扩展 4-7】讲一下实例对象、类对象、元类对象结构体的组成以及他们是如何相关联的？为什么对象方法没有保存在实例对象结构体里而是保存在类对象的结构体里？**

**【扩展 4-8】class_ro_t和class_rw_t的区别？**

**【扩展 4-9】class A 继承 class B，class B 继承 NSObject。画出完整的类图。**

[参考链接](https://www.jianshu.com/p/e033edeeeb6c)

**【扩展 4-10】下面的代码输出什么？**

```
@implementation Son : Father - (id)init
{
        self = [super init];
        if (self) {
                NSLog(@"%@", NSStringFromClass([self class]));
                NSLog(@"%@", NSStringFromClass([super class]));
        }
        return self;
}
@end
```
答案：都输出Son。

[[self class]和[super class]的区别](https://blog.csdn.net/loving_ios/article/details/49884599)

[类似问题：Runtime相关知识(4)](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E4%B9%8BRuntime%E5%8E%9F%E7%90%86%E6%8E%A2%E7%A9%B6.md)

**【4-11】id和 NSObject * 的区别是什么？**

id是一个 objc_object结构体指针，定义是 typedef struct objc_object * id。id可以理解为指向对象的指针。所有的OC对象都可以使用id来指向它们，而且编译器不会做类型检查，id调用任何存在的方法都不会在编译阶段报错。如果id指向的对象没有这个方法，则会崩溃。

NSObject * 指向的必须是NSObject的子类，调用的也只能是NSObject里面的方法，否则要做强制类型转换。此外，需要注意的是，不是所有的OC对象都是NSObject的子类，还有一些继承自NSProxy。因此NSObject * 可指向的类型是id可指向类型的子集。

**【4-12】NSProxy和NSObject的区别？**

相同点：NSObject和NSProxy都是Foundation框架中的基类，且均遵守NSObject协议。

不同点：NSProxy一般用来作为消息转发的代理类(因为NSProxy是一个抽象类，自身能够处理的方法极少(仅<NSObject>接口中定义的部分方法))。
        

**【4-13】nil、Nil、null、NSNull的区别？**

[nil、Nil、null、NSNull的区别](https://blog.csdn.net/wzzvictory/article/details/18413519)

[nil、Nil、null、NSNull的区别](https://www.jianshu.com/p/2b44e1c346e7)

**【4-14】OC的多态特性**

[多态特性](https://www.cnblogs.com/wendingding/p/3705428.html)



## 知识点5：KVO

**【5-1】iOS是如何实现对一个对象的KVO的？（KVO的本质或原理是什么？）**

本质上是利用Runtime和isa混写（isa-swizzling）机制实现的。当某个类的属性对象第一次被观察时，系统会在运行期动态地创建该类的一个子类(NSKVONotifying_原类名称)，并且让instance对象的isa指向这个全新的子类，并且这个全新的子类重写了setter方法、class方法、dealloc方法、_isKVOA方法。当修改instance对象的属性值时，重写的setter方法会调用Foundation的_NSSetXXXValueAndNotify函数，在_NSSetXXXValueAndNotify函数内部会先调用willChangeValueForKey:方法，然后调用父类原来的setter方法，最后调用didChangeValueForKey:方法，并在didChangeValueForKey方法内部触发监听器（Oberser）的监听方法（observeValueForKeyPath:ofObject:change:context:）。

**【5-2】如何手动触发KVO？**

手动调用willChangeValueForKey:和didChangeValueForKey:这两个方法即可。

**【5-3】直接修改成员变量会触发KVO吗？**

直接修改成员变量不会触发KVO(因为直接修改成员变量不会调用该属性的setter方法)。

KVO的实现原理是利用Runtime和isa混写（isa-swizzling）机制动态地生成一个子类，并且让instance对象的isa指向这个全新的子类，并且这个全新的子类重写了setter方法、class方法、dealloc方法、_isKVOA方法。当修改instance对象的属性值时，重写的setter方法会调用Foundation的_NSSetXXXValueAndNotify函数，在_NSSetXXXValueAndNotify函数内部会先调用willChangeValueForKey:方法，然后调用父类原来的setter方法，最后调用didChangeValueForKey:方法，并在didChangeValueForKey方法内部触发监听器（Oberser）的监听方法（observeValueForKeyPath:ofObject:change:context:）。由于直接修改成员变量不会调用该属性的setter方法，所以不会触发KVO。

**【5-4】如何取消系统默认的KVO并手动触发（给KVO的触发设定条件：改变的值符合某个条件时再触发KVO）？**

[详细实现代码](https://blog.csdn.net/IT_ZGC/article/details/50184419)

假设HLPerson.h中有一个name属性，代码如下：

```
@interface HLPerson : NSObject
@property(nonatomic,copy)NSString *name;
@end
```

HLPerson.m文件中代码如下：

```
#import "HLPerson.h"
@implementation Person
+(BOOL)automaticallyNotifiesObserversForKey:(NSString *)key{
    if ([key isEqualToString:@"name"]) {
        //关闭系统默认的KVO
        return NO;
    }else{
        return [super automaticallyNotifiesObserversForKey:key];
    }
}

-(void)setName:(NSString *)name{
    if (_name!=name) {
        [self willChangeValueForKey:@"name"];
        _name=name;
        [self didChangeValueForKey:@"name"];
    } 
}
@end
```

**【5-5】addObserver:forKeyPath:options:context:各个参数答作用分别是什么？**

```
[self.person addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:@"Person Name"];
```

参数代表的意义：(1)观察者，负责处理监听事件的对象；(2)观察的属性；(3)观察的选项；(4)上下文。

**【5-6】observer中需要实现哪个方法才能获得KVO回调？**

```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context;
```

参数代表的意义：(1)观察的属性；(2)观察的对象；(3)change 属性变化字典（新／旧）；(4)上下文，与监听的时候传递的一致,用来区分不同的监听。

**【5-7】从设计模式角度分析代理、通知和KVO的区别？iOS SDK提供的framework使用了哪些设计模式，为什么使用？有哪些优点和缺点？**



**【5-8】KVO、NSNotification、delegate和block的区别？**

[KVO，NSNotification，delegate及block区别](https://www.jianshu.com/p/59e34b91f0a8)

## 知识点6：KVC

KVC的全称是Key-Value Coding，俗称“键值编码”，可以通过一个key来访问某个属性。

**【6-1】KVC常用的API有哪几个？**

* -(void)setValue:(id)value forKeyPath:(NSString *)keyPath;
* -(void)setValue:(id)value forKey:(NSString *)key;
* -(id)valueForKeyPath:(NSString *)keyPath;
* -(id)valueForKey:(NSString *)key;

前两个是用来设置属性值的，后两个是用来获取属性值的。

**【6-2】赋值方法setValue:forKey:的实现原理是什么？**

setValue:forKey:底层实现原理如下：

（1）首先会按照顺序依次查找setKey:方法和_setKey:方法，只要找到这两个方法当中的任何一个就直接传递参数，调用方法；

（2）如果没有找到setKey:和_setKey:方法，那么这个时候会查看accessInstanceVariablesDirectly方法的返回值，如果返回的是NO（也就是不允许直接访问成员变量），那么会调用setValue:forUndefineKey:方法，并抛出异常“NSUnknownKeyException”；

（3）如果accessInstanceVariablesDirectly方法返回的是YES，也就是说可以访问其成员变量，那么就会按照顺序依次查找 _key、_isKey、key、isKey这四个成员变量，如果查找到了，就直接赋值；如果依然没有查到，那么会调用setValue:forUndefineKey:方法，并抛出异常“NSUnknownKeyException”。

**【6-3】取值方法valueForKey:的实现原理是什么？**

valueForKey:方法底层实现原理如下：

（1）首先会按照顺序依次查找getKey:、key、isKey、_key:这四个方法，只要找到这四个方法当中的任何一个就直接调用该方法；

（2）如果没有找到，那么这个时候会查看accessInstanceVariablesDirectly方法的返回值，如果返回的是NO（也就是不允许直接访问成员变量），那么会调用valueforUndefineKey:方法，并抛出异常“NSUnknownKeyException”；

（3）如果accessInstanceVariablesDirectly方法返回的是YES，也就是说可以访问其成员变量，那么就会按照顺序依次查找 _key、_isKey、key、isKey这四个成员变量，如果找到了，就直接取值；如果依然没有找到成员变量，那么会调用valueforUndefineKey方法，并抛出异常“NSUnknownKeyException”。

**【6-4】通过KVC修改属性会触发KVO吗？原理是什么？**

通过KVC修改属性值会触发KVO。即使没有声明属性，只有成员变量，只要accessInstanceVariablesDirectly返回的是YES，允许访问其成员变量，那么不管有没有调用setter方法，通过KVC修改成员变量的值，都能触发KVO。这也说明通过KVC内部实现了willChangeValueForKey:方法和didChangeValueForKey:方法。

**【6-5】KVC和KVO的keyPath一定是属性吗？**

不一定是属性。KVC支持实例变量。KVO只能手动支持实例变量。KVO需要自己在set方法里实现willChangeValueForKey:和didChangeValueForKey:
此外还要自己实现 +(BOOL)automaticallyNotifiesObserversForKey:(NSString *)key方法手动进行监听。


**【6-6】KVC的keyPath中的集合运算符如何使用？**
[KVC的keyPath中的集合运算符浅析](https://www.jianshu.com/p/c8198d24ac6f)

KVC的keyPath中的集合运算符会根据其返回值的不同分为以下三种类型：

* (1)简单的集合运算符：返回的是strings, number, 或者 dates；
      
      @count:返回一个值为集合中对象总数的NSNumber对象。
      @sum:首先把集合中的每个对象都转换为double类型，然后计算其总，最后返回一个值为这个总和的NSNumber对象。
      @avg:首先把集合中的每个对象都转换为double类型，然后计算其平均值，最后返回一个值为该平均值的NSNumber对象。
      @max:使用compare:方法来确定最大值。所以为了让其正常工作，集合中所有的对象都必须支持和另一个对象的比较。
      @min:和@max一样，但是返回的是集合中的最小值。
      
 ```
[products valueForKeyPath:@"@count"]; // 4
[products valueForKeyPath:@"@sum.price"]; // 3526.00
[products valueForKeyPath:@"@avg.price"]; // 881.50
[products valueForKeyPath:@"@max.price"]; // 1699.00
[products valueForKeyPath:@"@min.launchedOn"]; // June 11, 2012
 ```
      
* (2)对象运算符：返回的是一个数组；比如现在有一个数组：

```
NSArray *inventory = @[iPhone5, iPhone5, iPhone5, iPadMini, macBookPro, macBookPro];
```

@unionOfObjects/ @distinctUnionOfObjects: 返回一个由操作符右边的key path所指定的对象属性组成的数组。其中@distinctUnionOfObjects会对数组去重, 而@unionOfObjects不会。

```
[inventory valueForKeyPath:@"@unionOfObjects.name"]; // "iPhone 5", "iPhone 5", "iPhone 5", "iPad Mini", "MacBook Pro", "MacBook Pro"
[inventory valueForKeyPath:@"@distinctUnionOfObjects.name"]; // "iPhone 5", "iPad Mini", "MacBook Pro"
```


* (3)数组和集合运算符：返回的是一个数组或者集合。 

数组和集合操作符跟对象操作符很相似，只不过它是在NSArray和NSSet所组成的集合中工作的。@distinctUnionOfArrays/@unionOfArrays: 返回了一个数组，其中包含这个集合中每个数组对于这个操作符右面指定的key path进行操作之后的值。正如你期望的，distinct版本会移除重复的值。

@distinctUnionOfSets:和@distinctUnionOfArrays差不多, 但是它期望的是一个包含着NSSet对象的NSSet，并且会返回一个NSSet对象。因为集合不能包含重复的值，所以它只有distinct操作。

## 知识点7：关联对象

**【扩展 7-1】关联对象的应用场景？**

关联对象的应用场景是：给Category(分类)间接地添加成员变量。

优势：关联对象给Category(分类)间接地添加成员变量，不会影响到原来类对象的内存结构。

“<objc/runtime.h>”中提供的关联对象的API有以下3个：

(1)添加关联对象

```
void objc_setAssociatedObject(id object, const void * key,id value, objc_AssociationPolicy policy)
```

(2)获得关联对象

```
id objc_getAssociatedObject(id object, const void * key)
```

(3)移除所有的关联对象：用来移除所有的关联对象，因此如果不是想移除所有的，应该用 objc_setAssociatedObject(id object, const void * key,id value, objc_AssociationPolicy policy) 的方式移除某一个关联对象。也就是设置某个关联对象的value为nil，则相当于移除该关联对象。

```
void objc_removeAssociatedObjects(id object)
```

其中关联策略的枚举值有以下几个：

```
OBJC_ASSOCIATION_ASSIGN 其对应的修饰符是assign
OBJC_ASSOCIATION_RETAIN_NONATOMIC 其对应的修饰符是strong,nonatomic
OBJC_ASSOCIATION_COPY_NONATOMIC 对应的修饰符是copy,nonatomic
OBJC_ASSOCIATION_RETAIN 对应的修饰符是strong,atomic
OBJC_ASSOCIATION_COPY 其对应的修饰符是copy,atomic
```

例如给HLPerson类的Category分类HLPerson+Test类添加一个_name成员变量和一个_weight成员变量。代码如下：

```
//HLPerson+Test.h文件
#import "HLPerson.h"
@interface HLPerson (Test)
@property (copy, nonatomic) NSString *name;
@property (assign, nonatomic) int weight;
@end

//HLPerson+Test.m文件
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

**【扩展 7-2】系统如何管理关联对象？**

关联对象并不是存储在被关联对象本身内存中，而是存储在全局统一的AssociationHashMap哈希表中。系统通过管理这个全局哈希表，再通过对象的指针地址和传入的key值经过相应的计算来获取关联对象。根据objc_setAssociatedObject方法传入的关联策略，来对对象进行内存管理。

**【扩展 7-3】关联对象其被释放的时候需要手动将其指针置空么？**

  当对象被释放时，如果设置的关联策略是OBJC_ASSOCIATION_ASSIGN，那么他的关联对象不会减少引用计数，其他的协议都会减少从而释放关联对象。因此不管什么关联策略，对象释放时都无需手动将关联对象置空。
  
**【扩展 7-4】添加关联对象objc_setAssociatedObject的底层实现原理？**

[objc_setAssociatedObject的底层实现原理](https://github.com/baohenglin/HLBlog/blob/master/Articles/OC%E5%85%B3%E8%81%94%E5%AF%B9%E8%B1%A1%E7%AF%87.md)

**【扩展 7-5】获取关联对象objc_getAssociatedObject的底层实现原理？**

**【扩展 7-6】移除所有关联对象objc_removeAssociatedObjects的底层实现原理？**

**【扩展 7-7】使用runtime Associate方法关联的对象，需要在主对象dealloc的时候释放么？**

在ARC和MRC下均不需要。WWDC2011中指出，被关联的对象在生命周期内要比对象本身释放的晚很多。它们会在被NSObject -dealloc调用的object_dispose()方法中释放。

**对象的内存销毁详细过程**，分四个步骤：

* 1. 最后一次调用 -release，此时实例对象的引用计数变为零，对象正在被销毁，生命周期即将结束。调用 [self dealloc]
* 2. 继承关系中最底层的子类先调用 -dealloc，然后一层一层的向上，在其父类中再调用-dealloc方法
* 3. 最后NSObject 调 -dealloc，并调用 Objective-C runtime 中的 object_dispose() 方法
* 4. 调用 object_dispose()，对 C++ 的实例变量们（iVars）调用 destructors函数，对 ARC 状态下的 实例变量们(iVars)调用 -release方法，并且解除所有使用 runtime Associate方法关联的对象，解除所有 __weak 引用，最后调用 free()。







