# 知识点汇总篇章三

## 知识点7：Runtime

**【扩展 7-1】Runtime如何实现weak变量自动置为nil的？**(待优化)

runtime对注册的类会进行布局，对于weak对象会放入一个hash表中。用weak指向的对象内存地址作为key，当这个对象的引用计数为0的时候会dealloc函数，假如weak指向的对象内存地址是a，那么就会以a为键，在这个weak表中搜索，查找到所有以a为键的weak对象，并将查找到的这些weak对象设置为nil。

