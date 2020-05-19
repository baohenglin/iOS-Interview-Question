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

id是一个 objc_object结构体指针，定义是 typedef struct objc_object * id。id可以理解为指向对象的指针。所有的OC对象都可以使用id来指向它们，而且编译器不会做类型检查，id调用任何存在的方法都不会在编译阶段报错。如果id指向的对象没有这个方法，则会崩溃。

NSObject * 指向的必须是NSObject的子类，调用的也只能是NSObject里面的方法，否则要做强制类型转换。此外，需要注意的是，不是所有的OC对象都是NSObject的子类，还有一些继承自NSProxy。因此NSObject * 可指向的类型是id可指向类型的子集。

**【1-12】NSProxy和NSObject的区别？**

相同点：NSObject和NSProxy都是Foundation框架中的基类，且均遵守NSObject协议。

不同点：NSProxy一般用来作为消息转发的代理类(因为NSProxy是一个抽象类，自身能够处理的方法极少(仅<NSObject>接口中定义的部分方法))。
        

**【1-13】nil、Nil、null、NSNull的区别？**

[nil、Nil、null、NSNull的区别](https://blog.csdn.net/wzzvictory/article/details/18413519)

[nil、Nil、null、NSNull的区别](https://www.jianshu.com/p/2b44e1c346e7)

**【1-14】OC的多态特性**

[多态特性](https://www.cnblogs.com/wendingding/p/3705428.html)

**【1-15】方法和选择器有什么不同？**

method 是一个结构体，包含了方法名、方法字符串编码和方法实现；selector 是一个方法的名称。

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

**【1-16】Core Foundation 的内存管理**

凡是由带有 Create、Copy、Retain 等关键字的函数创建出来的对象，都需要在最后做一次 release 操作。比如 CFRunLoopObserverCreate 对象的 release 函数：CFRelease(对象);



