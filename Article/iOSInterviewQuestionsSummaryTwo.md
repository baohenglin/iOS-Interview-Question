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

## 知识点5：KVO

**【5-1】iOS是如何实现对一个对象的KVO的？（KVO的本质或原理是什么？）**

本质上是利用Runtime和isa混写（isa-swizzling）机制动态地生成一个子类，并且让instance对象的isa指向这个全新的子类，并且这个全新的子类重写了setter方法、class方法、dealloc方法、_isKVOA方法。当修改instance对象的属性值时，重写的setter方法会调用Foundation的_NSSetXXXValueAndNotify函数，在_NSSetXXXValueAndNotify函数内部会先调用willChangeValueForKey:方法，然后调用父类原来的setter方法，最后调用didChangeValueForKey:方法，并在didChangeValueForKey方法内部触发监听器（Oberser）的监听方法（observeValueForKeyPath:ofObject:change:context:）。

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








