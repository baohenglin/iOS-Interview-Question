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


## 知识点6：KVC
