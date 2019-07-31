# 知识点汇总篇章二

## 知识点四：OC对象

**【扩展 4-1】一个NSObject对象占用多少内存？**

系统分配了16个字节给NSObject对象（通过malloc_size函数获得），但NSObject对象内部只使用了8个字节的空间（64bit环境下，可以通过class_getInstanceSize函数获得）。

**【扩展 4-2】对象的isa指针指向哪里？**

* instance对象的isa指向class对象
* class对象的isa指向meta-class对象
* meta-class对象的isa指向基类(Root class)的meta-class对象

**【扩展 4-3】OC的类信息存放在哪里？**

* (1)对象方法、属性、成员变量、协议信息，存放在class对象中；
* (2)类方法，存放在meta-class对象中；
* (3)成员变量的具体值，存放在instance对象中。
