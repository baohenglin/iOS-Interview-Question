## 知识点1：KVO

**【1-1】iOS是如何实现对一个对象的KVO的？（KVO的本质或原理是什么？重点）**

KVO是键值观察机制，它提供了观察某一属性变化的方法。

本质上是利用 Runtime 和 isa混写（isa-swizzling）机制实现的。当一个对象(比如person 对象，person对象的类是 HLPerson)的属性（假设 person 的 age）第一次被观察时，系统会在运行期动态地创建该类的一个子类(继承自HLPerson 的 NSKVONotifying_HLPerson)，并且让 instance对象（实例对象）的 isa 指针指向这个全新的子类，并且这个全新的子类重写了setter方法、class方法、dealloc方法、_isKVOA方法。当修改instance对象的属性值 age 的时候，重写的setter 方法会调用 Foundation 的_NSSetXXXValueAndNotify 函数，在_NSSetXXXValueAndNotify 函数内部会先调用 willChangeValueForKey: 方法，然后调用父类原来的setter方法，最后调用 didChangeValueForKey: 方法，并在 didChangeValueForKey 方法内部触发监听器（Oberser）的监听方法（observeValueForKeyPath:ofObject:change:context:）。

**【1-2】如何手动触发KVO？**

手动调用willChangeValueForKey:和didChangeValueForKey:这两个方法即可。

**【1-3】直接修改成员变量会触发KVO吗？**

直接修改成员变量不会触发KVO(因为直接修改成员变量不会调用该属性的 setter 方法)。

KVO的实现原理是利用Runtime和isa混写（isa-swizzling）机制动态地生成一个子类，并且让instance对象的isa指向这个全新的子类，并且这个全新的子类重写了setter方法、class方法、dealloc方法、_isKVOA方法。当修改instance对象的属性值时，重写的setter方法会调用Foundation的_NSSetXXXValueAndNotify函数，在_NSSetXXXValueAndNotify函数内部会先调用willChangeValueForKey:方法，然后调用父类原来的setter方法，最后调用didChangeValueForKey:方法，并在didChangeValueForKey方法内部触发监听器（Oberser）的监听方法（observeValueForKeyPath:ofObject:change:context:）。由于直接修改成员变量不会调用该属性的setter方法，所以不会触发KVO。

**【1-4】如何取消系统默认的KVO并手动触发（给KVO的触发设定条件：改变的值符合某个条件时再触发KVO）？**

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

**【1-5】addObserver:forKeyPath:options:context:各个参数答作用分别是什么？**

```
[self.person addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:@"Person Name"];
```

参数代表的意义：(1)观察者，负责处理监听事件的对象；(2)观察的属性；(3)观察的选项；(4)上下文。

**【1-6】observer中需要实现哪个方法才能获得KVO回调？**

```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context;
```

参数代表的意义：(1)观察的属性；(2)观察的对象；(3)change 属性变化字典（新／旧）；(4)上下文，与监听的时候传递的一致,用来区分不同的监听。

**【1-7】从设计模式角度分析代理、通知和KVO的区别？iOS SDK提供的framework使用了哪些设计模式，为什么使用？有哪些优点和缺点？**



**【1-8】简述 KVO、NSNotification、Delegate和 Block 并说明它们之间的区别？（重点）**

[KVO，NSNotification，delegate及block区别](https://www.jianshu.com/p/59e34b91f0a8)

[参考链接2](https://www.cnblogs.com/oc-bowen/p/8652492.html)

**【1-9】iOS使用KVO设置键值观察依赖键的具体用法是什么？**

[iOS使用KVO设置键值观察依赖键](https://www.jianshu.com/p/22513f8fad8a)

**【1-10】KVO监听数组元素的变化**

[KVO监听数组元素的变化](https://www.jianshu.com/p/31fd5c8fe595)

**【1-11】KVO 的优点和缺点分别是什么？**

KVO 是一个对象能够观察另一个对象的属性的值，并且能够发现值的变化。KVO 适合任何类型的对象监听另外一个任意对象的改变。这是一个对象与另一个对象保持同步的一种方法，即当另外一种对象的状态发生变化时，观察对象马上做出反应。它只能用来对属性做出反应，而不会用来对方法做出反应。

**KVO 优点**：

* 能够提供一种简单的方法实现两个对象间的同步。例如 model 和 view 之间同步；
* 能够对非我们创建的对象，即内部对象的状态改变做出响应，而且不需要改变内部对象的实现；
* 能够提供观察的属性的最新值以及先前值；
* 用 key paths 来观察属性，因此也可以观察嵌套对象；

**KVO 缺点**：

* 我们观察的属性必须使用 string 来定义。因此在编译期不会检查，不会出现警告；
* 对属性重构将导致我们的观察代码不再可用；
* 当释放观察者时需要移除观察者。

## 知识点2：KVC

KVC 的全称是 Key-Value Coding，俗称“键值编码”，它是一种可以直接通过字符串的名字(key)来访问类的某个属性的机制。KVC支持类对象和内置的基本数据类型。

**【2-1】KVC常用的API有哪几个？**

* -(void)setValue:(id)value forKeyPath:(NSString *)keyPath;
* -(void)setValue:(id)value forKey:(NSString *)key;
* -(id)valueForKeyPath:(NSString *)keyPath;
* -(id)valueForKey:(NSString *)key;

前两个是用来设置属性值的，后两个是用来获取属性值的。

valueForUndefinedKey: 它的默认实现是抛出异常，可以重写这个函数来进行错误处理。

setValue: forUndefinedKey:

**【2-2】赋值方法setValue:forKey:的实现原理是什么？**

setValue:forKey:底层实现原理如下：

（1）首先会按照顺序依次查找setKey:方法和_setKey:方法，只要找到这两个方法当中的任何一个就直接传递参数，调用方法；

（2）如果没有找到setKey:和_setKey:方法，那么这个时候会查看accessInstanceVariablesDirectly方法的返回值，如果返回的是NO（也就是不允许直接访问成员变量），那么会调用setValue:forUndefineKey:方法，并抛出异常“NSUnknownKeyException”；

（3）如果accessInstanceVariablesDirectly方法返回的是YES，也就是说可以访问其成员变量，那么就会按照顺序依次查找 _key、_isKey、key、isKey这四个成员变量，如果查找到了，就直接赋值；如果依然没有查到，那么会调用setValue:forUndefineKey:方法，并抛出异常“NSUnknownKeyException”。

**【2-3】取值方法valueForKey:的实现原理是什么？**

valueForKey:方法底层实现原理如下：

（1）首先会按照顺序依次查找getKey:、key、isKey、_key:这四个方法，只要找到这四个方法当中的任何一个就直接调用该方法；

（2）如果没有找到，那么这个时候会查看accessInstanceVariablesDirectly方法的返回值，如果返回的是NO（也就是不允许直接访问成员变量），那么会调用valueforUndefineKey:方法，并抛出异常“NSUnknownKeyException”；

（3）如果accessInstanceVariablesDirectly方法返回的是YES，也就是说可以访问其成员变量，那么就会按照顺序依次查找 _key、_isKey、key、isKey这四个成员变量，如果找到了，就直接取值；如果依然没有找到成员变量，那么会调用valueforUndefineKey方法，并抛出异常“NSUnknownKeyException”。

**【2-4】通过KVC修改属性会触发KVO吗？原理是什么？**

通过KVC修改属性值会触发KVO。即使没有声明属性，只有成员变量，只要accessInstanceVariablesDirectly返回的是YES，允许访问其成员变量，那么不管有没有调用setter方法，通过KVC修改成员变量的值，都能触发KVO。这也说明通过KVC内部实现了willChangeValueForKey:方法和didChangeValueForKey:方法。

**【2-5】KVC和KVO的keyPath一定是属性吗？**

不一定是属性。KVC支持实例变量。KVO只能手动支持实例变量。KVO需要自己在set方法里实现willChangeValueForKey:和didChangeValueForKey:
此外还要自己实现 +(BOOL)automaticallyNotifiesObserversForKey:(NSString *)key方法手动进行监听。


**【2-6】KVC的keyPath中的集合运算符如何使用？**
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

