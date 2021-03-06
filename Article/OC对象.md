## OC对象

**【扩展 1-1】一个NSObject对象占用多少内存？**

系统分配了16个字节给NSObject对象（通过malloc_size函数获得），但NSObject对象内部只使用了8个字节的空间（64bit环境下，可以通过class_getInstanceSize函数获得）。

**【扩展 1-2】OC的类信息存放在哪里？**

* (1)对象方法、属性、成员变量、协议信息，存放在class对象中；
* (2)类方法，存放在meta-class对象中；
* (3)成员变量的具体值，存放在instance对象中。

**【扩展 1-3】对象的isa指针指向哪里？**

* instance对象的isa指向class对象
* class对象的isa指向meta-class对象
* meta-class对象的isa指向基类(Root class)的meta-class对象

**【扩展 1-4】对象的superclass指针指向哪里？**

* class(类对象)的superclass指针指向父类的class(类对象)；如果没有父类，那么superclass指针为nil。
* meta-class(元类对象)的superclass指针指向父类的meta-class。特别需要注意的是基类的meta-class(元类对象)的superclass指针指向基类的class对象(类对象)。

**【扩展 1-5】instance对象(实例对象)调用对象方法的原理？**

由于对象方法存储在class对象（类对象）的内存中，因此instance对象(实例对象)调用对象方法的原理是首先通过instance(实例对象)的isa指针找到它自己的class对象（类对象），查找自己的class对象中是否存在打算调用的对象方法。如果存在，则进行调用；如果不存在该对象方法，那么就会通过自己class对象中的superclass指针找到它的父类的class对象并查找其中是否存在将要调用的对象方法，如果存在，进行调用；如果不存在，那么就再通过这个父类的class对象的superclass指针，继续往上查找...当查找到基类(NSOject)的class对象（类对象）时，如果此时存在该对象方法就立刻调用，如果基类中也不存在该对象方法，那么意味着自始至终都没有找到该对象方法，此时会报错“unrecognized selector sent to instance”。

**【扩展 1-6】class对象(类对象)调用类方法的原理？**

由于类方法都存储在meta-class对象（元类对象）中，因此class对象(类对象)调用类方法的原理是首先通过class对象的isa指针找到自己的meta-class对象，查看是否存在将要调用的类方法，如果存在，立即调用；如果不存在，则通过该meta-class对象中的superclass指针找到父类的meta-class对象（元类对象）并查找其中是否存在将要调用的类方法，如果存在，立刻调用，如果不存在，则通过superclass指针继续一层一层往上查找，直至找到基类(Root class)的元类对象，并查找基类(Root class)的元类对象中是否存在将要调用类方法，如果存在立即调用，如果也不存在，此时并不会报“unrecognized selector sent to instance”错误，而是会继续通过superclass指针找到基类的class对象（类对象），如果基类的class对象中存在将要调用的类方法就立刻调用，如果不存在，此时才会报错(“unrecognized selector sent to instance”)。

**【扩展 1-7】讲一下实例对象、类对象、元类对象结构体的组成以及他们是如何相关联的？为什么对象方法没有保存在实例对象结构体里而是保存在类对象的结构体里？**

**【扩展 1-8】class_ro_t和class_rw_t的区别？**

**【扩展 1-9】class A 继承 class B，class B 继承 NSObject。画出完整的类图。**

[参考链接](https://www.jianshu.com/p/e033edeeeb6c)

**【扩展 1-10】下面的代码输出什么？**

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

**【1-11】id和 NSObject * 的区别是什么？**

id是一个 objc_object结构体指针，定义是 typedef struct objc_object * id。id 类型的指针可以理解为**指向任何OC对象的指针**。所有的OC对象都可以使用id来指向它们，而且编译器不会做类型检查，id调用任何存在的方法都不会在编译阶段报错。如果id指向的对象没有这个方法，则会崩溃。

NSObject * 指向的必须是NSObject的子类，调用的也只能是NSObject里面的方法，否则要做强制类型转换。此外，需要注意的是，不是所有的OC对象都是NSObject的子类，还有一些继承自NSProxy。因此NSObject * 可指向的类型是id可指向类型的子集。

**【1-12】NSProxy和NSObject的区别？**

相同点：NSObject和NSProxy都是Foundation框架中的基类，且均遵守NSObject协议。

不同点：NSProxy一般用来作为消息转发的代理类(因为NSProxy是一个抽象类，自身能够处理的方法极少(仅<NSObject>接口中定义的部分方法))。
        

**【1-13】nil、NULL、Nil、NSNull的区别？（重点）**

[nil、Nil、null、NSNull的区别](https://blog.csdn.net/wzzvictory/article/details/18413519)

[nil、Nil、null、NSNull的区别](https://www.jianshu.com/p/2b44e1c346e7)

* nil：表示 OC 对象的指针(引用)为空。在 Objective-C 中，nil 对象调用任何方法表示什么也不执行，也不会崩溃。
* NULL：表示 C 语言中指向基础数据类型的指针为空。对于 Objective-C 来说，NULL就表示 ((void *)0)。在非 ARC 中，nil 和 NULL 可以互换，但是在 ARC 中，普通指针和对象引用被严格限制，不能互换。
* Nil：用于表示空类。比如 Class myClass = Nil; Nil对于我们 Objective-C开发来说，Nil 也代表 ((void *)0)。
* NSNull：NSNull 是继承于 NSObject 的类型。它是很特殊的类。它表示空，什么也不存储，但是它却是对象，只是一个占位对象。使用场景是：当服务端接口中在 key 为空时，传空，

```
NSDictionary (parametersDic = @{@"arg1":@"value1",@"arg2":arg.isEmpty ? [NSNull null] : arg2};
```

nil、NULL、Nil 这三者对于 Objective-C 来说，值都是一样的，都是 (void *)0，那么为什么还要区分呢？它们又和 NSNull 有什么区别呢？

* NULL 是宏，是对于 C 语言指针而使用的，表示空指针；
* nil 是宏，是对于 Objective-C 中的**对象**而使用的，表示对象的指针指向空（对象的引用为空）；
* Nil 是宏，是对于 Objective-C 中的**类**而使用的，表示**类指向空**；
* NSNull：是类类型，是用于表示空的占位对象，与 JS 或者服务端的 null 类似。

**【1-14】OC的多态特性**

[多态特性](https://www.cnblogs.com/wendingding/p/3705428.html)

**【1-15】方法和选择器有什么不同？**

method 是一个结构体，包含了方法名、方法字符串编码和方法实现；selector 是一个方法的名称。

```
struct method_t {
    SEL name;//函数名
    const char *types;//字符串编码，里面存放着返回值类型、参数类型。
    IMP imp;//指向函数的指针（函数地址）

    struct SortBySELAddress :
        public std::binary_function<const method_t&,
                                    const method_t&, bool>
    {
        bool operator() (const method_t& lhs,
                         const method_t& rhs)
        { return lhs.name < rhs.name; }
    };
};
```

**【1-16】Core Foundation 的内存管理**

凡是由带有 Create、Copy、Retain 等关键字的函数创建出来的对象，都需要在最后做一次 release 操作。比如 CFRunLoopObserverCreate 对象的 release 函数：CFRelease(对象);

**【1-17】malloc 和 new 的区别是什么？**

[new 和 malloc 的区别](https://www.cnblogs.com/shilinnpu/p/8945637.html)

* 从“申请的内存所在的位置”来看，new 操作符从自由存储区（free store）上为对象动态分配内存空间，而 malloc 函数从堆上动态分配内存。
* 从“返回类型安全性”方面来看，new 操作符内存分配成功时，返回的是对象类型的指针，类型严格与对象匹配，无须进行类型转换，故 new 是符合类型安全性的操作符。而 malloc 内存分配成功时返回 void *，需要通过强制类型转换将 void * 指针转换成我们需要的类型。
* 从“内存分配失败时的返回值”方面来看，new 内存分配失败时，会抛出 bac_alloc 异常，它不会返回 NULL；malloc 分配内存失败时返回 NULL。
* 从“是否需要指定内存大小”方面来看，使用 new 操作符申请内存分配时无需指定内存块的大小，编译器会根据类型信息自行计算，而 malloc 则需要显式地指出所需内存的大小；
* 从“是否调用构造函数/析构函数”方面来看，new 不止是分配内存，而且会调用类的构造函数（new 可以认为是 malloc 加构造函数的执行），同理 delete 会调用类的析构函数，而 malloc 则只分配内存，不会进行初始化类成员的操作，同样 free 也不会调用析构函数；
* 从“是否可以被重载”方面来看，new 出来的可以被重载，malloc 的不能被重载；
* 从“是否能够重新分配内存”方面来看，new 无法直观的处理，而 malloc 可以通过 realloc 函数进行内存重新分配以实现内存的扩充。
* 从“是否可以互相调用”方面来看，new 的实现可以通过调用 malloc 实现，而 malloc 的实现不能调用 new。 
* new 是 C++ 中的操作符，malloc 是 C 语言中的一个函数；
* 内存泄漏对于 malloc 和 new 都可以检查出来，区别在于 new 可以指明是哪个文件的哪一行，而 malloc 没有这些信息；

**【1-18】什么是 SEL？如何声明一个 SEL？通过哪些方法能够调用 SEL 包装起来的方法？**

**SEL概念**：

SEL 是方法名或函数名，也称作方法选择器。在内存中每个类的对象方法都存储类对象中，每个方法底层结构体重都有一个与之对应的 SEL 类型的 name（方法名），根据这个方法名就可以找到对应的存储着方法实现的 IMP imp，进而调用该方法。

**声明 SEL**：

```
SEL s1 = @selector(test1); //将 test1方法封装成 SEL 对象
SEL s2 = NSSelectorFromString(@"test1"); //将一个字符串方法转换成 SEL 对象
```

**SEL 类型方法的调用**：

```
//方式1：直接通过方法名来调用
[person text1]; 

//方式2：间接地通过 SEL 数据来调用
SEL methodA = @selector(test);
[person performSelector:methodA];
```

**【1-19】协议中<NSObject>表示什么意思？子类继承了父类，那么子类会遵守父类中遵守的协议吗？协议中能够定义成员变量吗？如何约束一个对象类型的变量要存储的地址是遵守一个协议对象？**
        
 * 遵守了 NSObject 协议。
 * 会。
 * 能。但是只能在头文件中声明，编译器不会自动生成实例变量，需要自己处理 getter 和 setter 方法。
 * id<XXX>
        
**【1-20】NS/CF/CG/CA/UI 这些前缀分别是什么含义？**

* NS 表示函数归属于 cocoa Fundation 框架；
* CF 表示函数归属于 core Fundation 框架；
* CG 表示函数归属于 CoreGraphics.frameworks 框架；
* CA 表示函数归属于 CoreAnimation.frameworks 框架；
* UI 表示函数归属于 UIKit 框架。

**【1-21】面向对象都有哪些特征以及你对这些特征的理解**

* 封装：封装是把数据和操作数据的方法绑定起来，对数据的访问只能通过已定义的接口。我们在类中编写的方法就是对实现细节的封装。我们编写一个类就是对数据和数据操作的封装。总之，封装就是隐藏一切可隐藏的东西，只向外界提供最简单的编程接口。
* 继承：继承是从已有类得到继承信息创建新类的过程。提供继承信息的类被称为父类（超类、基类）；得到继承信息的类被称为子类（派生类）。继承让变化中的软件系统有了一定的延续性，同时继承也是封装“程序中可变因素”的重要手段。
* 多态：多态性是指**不同对象以自己的方式响应相同的消息的能力**。多态性分为编译时的多态性和运行时的多态性。方法重载（overload）实现的是编译时的多态性（也称为前绑定），而方法重写（override）实现的是运行时的多态性（也称为后绑定）。运行时的多态性是面向对象最精髓的东西，要实现多态需要做两件事：一是“方法重写”（子类继承父类并重写父类中已有的或抽象的方法），二是“对象造型”（用父类型引用子类型对象，这样同样的引用，调用同样的方法就会根据子类对象的不同而表现出不同的行为）。假设生物类（Life）都拥有一个相同的方法 -eat；Person 和 Dog 类都继承自 Life，Person 类和 Dog 类实现了各自的 eat 方法，person 实例对象调用 Person 类中的 eat 方法，dog 实例对象调用 Dog 类中的 eat 方法，也就是不同的对象以自己的方式响应了相同的消息（都响应了 eat 这个选择器）。
* 抽象：抽象是将一类对象的共同特征总结出来构造类的过程，包括数据抽象和行为抽象两方面。抽象只关注对象有哪些属性和行为，并不关注这些行为的细节是什么。

**【1-22】什么是懒加载（lazy loading）？懒加载的好处是什么？**

懒加载是指只有在用到的时候才去初始化。也可以理解为延时加载。

**懒加载的好处**：

* 降低系统的内存占用率；
* 对象的实例化在 getter 方法中，各司其职，降低了耦合性；
* 不需要将对象的实例化全部写到 ViewDidLoad 中，可以简化代码，增强代码的可读性。

**【1-23】OC有多继承吗？如果没有的话，可以用什么方法替代？**

多继承是指**一个子类可以有多个父类，它继承了多个父类的特性**。

Objective-C 的类不支持多继承，只支持单继承。如果要实现多继承，可以通过**类别**和**协议**的方式来实现。protocol（协议）可以实现多个接口，通过实现多个接口可以实现多继承；通过 Category（类别）去重写类的方法（仅对本 Category 有效，不会影响到其他类和原有类的关系），也可以实现多继承。

**【1-24】Objective-C 有私有方法吗？有私有变量吗？如果没有的话，有没有什么代替的方法？（重点）**

Objective-C **没有私有方法**，只有静态方法（类方法）和实例方法（对象方法）这两种方法。但是可以通过把方法的声明和定义都放在 .m 文件中来实现一种表面上的私有方法。

Objective-C **有私有变量**，可以通过 @private 来修饰或者把声明放到 .m 文件中。在 Objective-C 中，所有实例变量默认都是私有的，所有实例方法默认都是公有地。

**【1-25】include 与 #import 的区别？#import 和 @class 的区别是什么？**

import 指令是 Objective-C 针对 C 语言的 #include 的改进版本，**import 确保引用的文件只会被引用一次**。

import 与 @class 二者的区别：

* import 会链入该头文件的全部信息，包括实例变量和方法等；而 @class 只是告知编译器，其后面声明的名称是类的名称，至于这些类是如何定义的，定义了哪些实例变量和方法，暂时不用考虑；
* 在编译效率方面，@class 比 #import 速度快。
* 如果在类的实现里面需要用到这个引用类内部的实例变量和方法，就需要使用 #import 来引入头文件；
* 如果存在循环依赖关系，如 A->B，B->A这样的相互依赖关系，如果使用 #import 来互相包含，那么就会出现编译错误，如果使用 @class，则不会报编译错误。

**【1-26】什么是谓词？如何使用？**

谓词就是通过 NSPredicate 给定的逻辑条件作为约束条件来完成对数据的过滤筛选。

* (1)但条件过滤：

```
//定义谓词对象，谓词对象中包含了过滤条件
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"age<%d",30];
//使用谓词条件过滤数据中的元素，过滤之后返回查询的结果
NSArray *array = [personArr filteredArrayUsingPredicate:predicate];
```

* (2)可以使用 && 进行多条件过滤：

```
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"name='l' && age>40"];
//使用谓词条件过滤数据中的元素，过滤之后返回查询的结果
NSArray *array = [personArr filteredArrayUsingPredicate:predicate];
```

* (3)包含语句的使用

```
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"self.name IN {'1','2','4'} || self.age IN {30, 40}"];
```

* (4)筛选字符开头和字符结尾包含指定的字符的数据

```
//name 以 a 开头的
predicate = [NSPredicate predicateWithFormat:@"name BEGINSWITH 'a'"];

//name 以 ba 结尾的
predicate = [NSPredicate predicateWithFormat:@"name ENDSWITH 'ba'"];
```

* (5)其他

```
//name中包含字符 a 的
predicate = [NSPredicate predicateWithFormat:@"name CONTAINS 'a'"];
//name 中只要有 s 字符就满足条件
predicate = [NSPredicate predicateWithFormat:@"name like 's'"];
//?代表一个字符，下面的查询条件是：name 中第二个字符是 s 的
predicate = [NSPredicate predicateWithFormat:@"name like '?s'"];
```

**【1-27】常见的 OC 数据类型有哪些？和 C 的基本类型有什么区别？**

OC 常见的数据类型有 NSInteger、CGFloat、NSString、NSArray、NSData。

* NSInteger 根据 32 位或者 64 位系统决定本身是 int 还是 long；
* CGFloat 根据 32 位或者 64 位系统决定本身是 float 还是 double；
* NSString、NSNumber、NSArray、NSData 都是指针类型的对象，在堆中分配内存，C 语言中的 char int 等都是在栈中分配空间。

**【1-28】向一个 nil 对象发送消息会发生什么？（重点）**

* 向 nil 发送消息是完全有效的。只是在运行时不会有任何作用。
* 如果一个方法返回值是一个对象，那么发送给 nil 的消息将返回 0（nil）；
* 如果方法返回值为指针类型，其指针大小为小于或等于 sizeof(void *)，float，double，long double 或者 long long 的整型标量，发送给 nil 的消息将返回 0；
* 如果方法返回值为结构体，发送给 nil 的消息将返回 0。结构体中各个字段的值将都是0。
* 如果方法的返回值不是上述提到的几种情况，那么发送给 nil 的消息的返回值将是未定义的。

**【1-29】类方法和实例方法的本质区别和联系是什么？**

* 类方法属于类对象，实例方法属于实例对象；
* 类方法只能由类对象调用，实例方法只能由实例对象调用；
* 类方法的 self 是类对象，实例对象的 self 是实例对象；
* 类方法可以调用其他类方法，实例方法可以调用实例方法；
* 类方法不能访问成员变量，实例方法可以访问成员变量；
* 类方法不能直接调用对象方法，实例方法可以直接调用类方法。

**【1-30】写一个 NSString 类的实现（重点 没看明白）**

[参考链接1](https://blog.csdn.net/dayuqi/article/details/8101099?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase)

**【1-31】@property 的本质是什么？ivar、getter、setter 是如何生成并添加到这个类中的？**

**@property 的本质**：@property = ivar（实例变量）+ getter（取方法）+ setter（存方法）

**ivar、getter、setter 是如何生成并添加到这个类中的**：这是编译器自动合成的，通过 @synthesize 关键字指定，若不指定，默认为 @synthesize propertyName = _propertyName; 若手动实现了 getter/setter 方法，则不会自动合成。

**【1-32】这个写法会有什么问题：@property(copy) NSMutableArray *array;**

* (1)没有指明为 nonatomic，因此就是 atomic 即原子操作，会影响性能。该属性使用了同步锁，会在创建时生成一些额外的代码用于帮助编写多线程程序，这会带来性能问题。通过声明 nonatomic 可以节省这些虽然很小但是不必要的额外开销。在我们的应用程序中，几乎都是使用 nonatomic 来修饰的，因为使用 atomic 并不能保证绝对的线程安全，如果要绝对保证线程安全，还需要使用更高级的方式来处理，比如 NSSpinLock 等。
* (2)因为使用的是 copy，所得到的实际是 NSArray 类型，它是不可变的，若在使用中使用了“增、删、改”等操作，则会 crash。

**【1-33】@property 中有哪些属性关键字？**

1. 是否原子性：

```
nonatomic
atomic
```

2. 读写权限：

```
readwrite
readonly
```

3. 内存管理：

```
assign
strong
weak
unsafe_unretained
copy
```

4. getter、setter

**【1-34】isa 指针**

isa 是一个 Class 类型的指针，每个实例对象都有一个 isa 指针，该指针指向该实例对象的类对象，此外 Class 里面也有一个 isa 指针，指向 metaClass（元类）。元类保存了类方法的列表。需要注意的是，元类（meta class）也是类，也有 isa 指针，它的 isa 指针最终指向的是一个根元类（root meta class），根元类的 isa 指针指向本身。

**【1-35】如何访问并修改一个类的私有属性？（重点）**

方式1：通过 KVC 获取

```
HLPerson *person = [[HLPerson alloc]init];
[person setValue:@"FQY" forKeyPath:@"name"]; //使用 KVC 方式访问，可以修改，前提是必须知道变量名。
```

方式2：通过 runtime 访问并修改私有属性，代码如下：

```
#import <objc/runtime.h>
HLPerson *person = [[HLPerson alloc]init];

/*** 方法1：知道变量名时 ***/
//获取一个实例变量信息
Ivar nameIvar = class_getInstanceVariable([HLPerson class], "_name");
//设置成员变量的值
object_setIvar(person, nameIvar, @"BHL");
//获取成员变量的值
NSString *name = object_getIvar(person, nameIvar);
NSLog(@"name=%@",name);//name=BHL

/*** 方法2：不知道变量名时,通过遍历获取并打印所有变量，来确定要修改的是哪个私有变量 ***/
unsigned int count;
Ivar *ivars = class_copyIvarList([HLPerson class], &count);
for (int i = 0; i < count; i++) {
  // 获取成员变量
  Ivar ivar = ivars[i];
  NSMutableString *name = [NSMutableString stringWithUTF8String: ivar_getName(ivar)];
  //需要通过打印来确定要获取的 变量名 是ivars 数组中的第几个元素
  NSLog(@"使用 runtime：%@", name);
}
Ivar name = ivars[0];
object_setIvar(person, name, @"FQY666");
//获取成员变量的值
NSString *name = object_getIvar(person, ivars[0]);
NSLog(@"name=%@",name);//name=BHL
```

**【1-36】如何为 Class 定义一个对外只读对内可读写的属性？**

在头文件中将属性定义为 readonly，在 .m 文件中将属性重新定义为 readwrite。

**【1-37】Objective-C 的 class 是如何实现的？Selector 是如何转化为 C 语言的函数调用的？（重点 待优化）**

**Objective-C 的 class 是如何实现的**？

当一个类被正确的编译后，在这个编译成功的类里面，存在一个变量用于保存这个类的信息。我们可以通过 [NSClassFromString ] 或 [obj class] 方法来获取该类。这样的机制允许我们在运行时可以获取该实例对象的类对象，也可以在动态地生成一个在编译阶段无法确定的一个对象。

**Selector 是如何转化为 C 语言的函数调用的？**

@selector()基本可以等同于 C 语言的中函数指针,只不过 C 语言中，可以把函数名直接赋给一个函数指针，而 Objective-C 的类不能直接应用函数指针，这样只能做一个 @selector 语法来获取.

```
@interface foo
-(int)add:int val;
@end

SEL class_func ; //定义一个类方法指针
class_func = @selector(add:int);
```

* @selector是查找当前类的方法，而[object @selector(方法名:方法参数..) ] ;是或取 object 对应类的相应方法；
* 查找类方法时，除了方法名,方法参数也查询条件之一；
* 可以用字符串来找方法 SEL　变量名　=　NSSelectorFromString(方法名字的字符串);
* 可以在运行时用 SEL 变量反向查出方法名字符串。NSString　*变量名　=　NSStringFromSelector(SEL参数);
* 获取到 selector 的值以后，执行seletor。通过 performSelecor 方法来执行 SEL

```
[对象　performSelector:SEL变量　withObject:参数1　withObject:参数2];
```

**【1-38】对于语句 NSString *obj = [[NSData alloc] init];，编译时和运行时 obj 分别是什么类型？**

编译时是 NSString 类型，运行时是 NSData 类型。

**分析**：

首先，声明 NSString * obj 是告诉编译器，obj 是一个指向某个 Objective-C 对象的指针。因为不管指向的是什么类型的指针，一个指针所占的内存空间都是固定的，所以这里声明成任何类型的对象，最终生成的可执行代码都是一样的。实际上编译的时候 NSString * obj 等同于 id obj，编译器都会在栈空间分配一个 id 类型的数据。id 数据实际上就是一个 struct objc_object 结构体指针：

```
/// Represents an instance of a class.
struct objc_object {
Class isa OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```

这里限定了 NSString 只不过是告诉编译器，要把 obj 当做一个 NSString 类型来检查，如果后面调用了非 NSString 的方法，会产生警告。

接着创建了一个 NSData 对象，然后把这个对象所在的内存地址保存在 obj 变量里，那么在运行时，obj 变量指向的内存空间就是一个 NSData 对象，可以把 obj 当做一个 NSData 对象来用。

**【1-39】@synthesize 和 @dynamic 分别有什么用？**

* @property 有两个对应的词：@synthesize 和 @dynamic。如果 @synthesize 和 @dynamic 都没写，那么默认的就是 @synthesize var = _var;
* @synthesize 的语义是如果你没有手动实现 setter 方法和 getter 方法，那么编译器会自动为你加上这两个方法的实现。
* @dynamic 的语义是告知编译器，属性的 setter 方法和 getter 方法由用户自己实现，不要自动生成。假如一个属性被声明为 @dynamic var，然后你没有提供 setter 方法和 getter 方法，编译的时候没问题，但是当程序运行到 instance.var = someVar，由于缺少 setter 方法会导致程序崩溃；或者当运行到 someVar = var 时，由于缺少 getter 方法同样会导致崩溃。编译时没有问题，运行时才执行相应的方法，这就是所谓的动态绑定。

**【1-40】把字符串“2020-05-20”格式化日期转为 NSDate 类型**

```
NSString *timeStr = @"2020-05-20";
NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
formatter.dateFormat = @"yyyy-MM-dd";
formatter.timeZone = [NSTimeZone defaultTimeZone];
NSDate *date = [formatter dateFromString:timeStr];
```

**【1-41】怎样使用 performSelector 传入 3 个以上参数，其中一个为结构体？**

系统提供的方法包括：

```
- (id)performSelector:(SEL)aSelector;
- (id)performSelector:(SEL)aSelector withObject:(id)object;
- (id)performSelector:(SEL)aSelector withObject:(id)object1 withObject:(id)object2;
```

系统提供的 performSelector 的 api 中，并没有提供三个参数的，那么我们能不能传数组或者字典呢？由于数组和字典只能存入对象类型，而结构体并不是对象类型。故不能使用数组或字典。因此我们只能通过把对象放入结构体作为属性来传递了。

```
typedef struct HLStudent {
   int a;
   int b;
} *my_struct;

@interface HLStudent : NSObject

@property (nonatomic, assign) my_struct arg3;
@property (nonatomic, assign) NSString *arg1;
@property (nonatomic, assign) NSString *arg2;
@end

@implementation HLStudent

//在堆上分配的内存，需要手动释放
- (void)dealloc {
   free(self.arg3);
}
@end
```

测试：

```
my_struct str = (my_struct)(malloc(sizeof(my_struct)));
str->a = 1;
str->b = 2;
HLStudent *student = [[HLStudent alloc] init];
student.arg1 = @"arg1";
student.arg2 = @"arg2";
student.arg3 = str;
[self performSelector:@selector(call:) withObject:student];

- (void)call:(HLStudent *)obj {
  NSLog(@"%d %d",obj.arg3->a,obj.arg3->b);
}
```

**【1-42】若一个类有实例变量 NSString * _foo，调用 setValue:forKey: 时，可以以 foo 还是以 _foo 作为key？**

foo 和 _foo 都可以。

**setValue:forKey:底层实现原理如下**：

（1）首先会按照顺序依次查找setKey:方法和_setKey:方法，只要找到这两个方法当中的任何一个就直接传递参数，调用方法；

（2）如果没有找到setKey:和_setKey:方法，那么这个时候会查看 + (BOOL)accessInstanceVariablesDirectly 方法的返回值是 YES 还是 NO，默认返回 YES，如果返回的是NO（也就是不允许直接访问成员变量），那么会调用setValue:forUndefineKey:方法，并抛出异常“NSUnknownKeyException”；

（3）如果accessInstanceVariablesDirectly方法返回的是YES，也就是说可以访问其成员变量，那么就会按照顺序依次查找 **_key、_isKey、key、isKey** 这四个成员变量，如果查找到了，就直接赋值；如果依然没有查到，那么会调用setValue:forUndefineKey:方法，并抛出异常“NSUnknownKeyException”。

**【1-43】类 NSObject 的哪些方法经常被使用？**

NSObject 是 Objective-C 的基类，其由 NSObject 类及一系列协议构成。其中常用的类方法包括 alloc、class、description；对象方法包括 init、dealloc、performSelector:withObject:afterDelay:

**【1-44】什么是简便构造方法？**

简便构造方法一般由 CocoaTouch 框架提供，如 NSNumber 的 下列方法：

```
+ (NSNumber *)numberWithBool:(BOOL)value;
+ (NSNumber *)numberWithChar:(char)value;
+ (NSNumber *)numberWithInt:(int)value;
+ (NSNumber *)numberWithFloat:(float)value;
+ (NSNumber *)numberWithDouble:(double)value;
```

Foundation 框架下大部分类均有简便构造方法，我们可以通过简便构造方法，获得系统给我们创建好的对象，并且不需要手动释放。

**【1-45】什么是构造方法，使用构造方法有什么注意点？**

构造方法是对象初始化一个实例的方法。**构造方法的作用**：一般在构造方法里，对类进行一些初始化操作。

注意点：方法开头必须以 init 开头，接下来名称要大写，比如 initWithName，initLayout。

**【1-46】创建一个对象需要经过哪些步骤？**

* (1)开辟内存空间
* (2)初始化参数
* (3)返回内存地址值

**【1-47】Get 方法的作用是什么？Set 方法的作用是什么？使用 Set 方法的好处是什么？**

Get 方法的作用是为调用者返回对象内部的成员变量。

Set 方法的作用：为外界提供一个设置成员变量值的方法。Set 方法的好处：不让数据暴露在外，保证了数据的安全性。

**【1-48】id 类型是什么？instancetype 是什么？二者有什么区别？**

id类型：万能指针，能作为参数、方法的返回类型；

instancetype：只能作为方法的返回类型，并且返回的类型是当前定义类的类类型。

**【1-49】截取字符串“20|http://www.baidu.com” 中，“|”字符前面和后面的数据，分别输出它们。（重点 手写代码）**

```
NSString *str = @"20|http://www.baidu.com";
NSArray *array = [str componentsSeparatedByString:@"|"];
for (int i = 0; i < [array count]; i++) {
  NSLog(@"%d=%@",i,[array objectAtIndex:i]);
}
```

**【1-50】编写一个函数，实现递归删除指定路径下的所有文件**

```
+ (void)deleteFiles:(NSString *)path {
   //1. 判断是文件还是目录
   NSFileManager *fileManager = [NSFileManager defaultManager];
   BOOL isDir = NO;
   BOOL isExist = [fileManager fileExistsAtPath:path isDirectory: &isDir];
   if (isExist) {
      //2. 判断是不是目录
      if (isDir) {
        NSArray *dirArray = [fileManager contentsOfDirectoryAtPath:path error: nil];
        NSString *subPath = nil;
        for (NSString *str in dirArray) {
          subPath = [path stringByAppendingPathComponent:str];
          BOOL issubDir = NO;
          [fileManager fileExistsAtPath:subPath isDirectory: &issubDir];
          [self deleteFiles:subPath];
        }
      } else {
        [manager removeItemAtPath:filePath error:nil];
      }
   } else {
     NSLog(@“目录不存在”)；
   }
}
```

**【1-51】Objective-C 与 C、C++ 之间的联系和区别是什么？**

 **联系**：Objective-C 和 C++ 都是 C 的面向对象的超集。
 
**Objective-C 与 C++ 的区别**主要是：

* Objective-C 是完全动态的，支持在运行时动态类型决议（dynamic typing），动态绑定（dynamic binding）和动态装载（dynamic loading）；而 C++ 是部分动态的，编译时静态绑定，通过嵌入类（多重继承）和虚函数（虚表）来模拟实现。
* Objective-C 在语言层次上支持动态消息转发，其消息发送语法为 [object function]; 而 C++ 为 object->function()。两者的语义也不同，在 Objective-C 中表示向对象发送一条消息，可能会经过消息发送阶段，动态方法解析阶段和消息转发阶段这三大阶段；而在 C++ 里表示对象进行了某个操作，如果对象没有这个操作的话，要么编译会报错（静态绑定时），要么程序会 crash（动态绑定时）。

**【1-52】谈谈对目标-动作机制（target-Action）的理解**

目标-动作机制中，目标是动作消息的接收者，动作是控件发送给目标的消息。程序需要通过某些机制来进行事件和指令的翻译，这种机制就是目标-动作机制。目标-动作机制的优点在于可以减少模块之间代码的耦合性，并增强模块内代码之间的内聚性。

```
UIBarButtonItem *saveBtn = [[UIBarButtonItem alloc]initWithTitle:@"Save" style:UIBarButtonItemStyleDone target:self action:@selector(saveClick:)];
```

在这里，按钮按下以后会调用 target(也就是 self)上的 saveClick: 方法，按照 Objective-C 的习惯来说就是当 click 事件发生后，会给 self 对象发送一条 message，进而 self 对象上的 saveClick 方法被调用。

Target-Action 主要用于 MVC 设计模式中 V 和 C 之间进行通信。V只负责向 target 发送相应的 action，并不关心 target 具体做了什么。这降低了模块之间的耦合性。


