## 知识点：分类(category)和类扩展(Extension)

**【扩展 1-1】分类(category)应用场景有哪些？分类有哪些局限性？分类的结构体里面有哪些成员？**

Category是Objective-C 2.0之后增加的语言特性，Category的主要作用是为已经存在的类添加方法。Category是对装饰模式的典型实践（装饰模式(Decorator)是指在不修改原有类的前提下，动态地给这个类添加一些方法）。

**分类(Category)的应用场景包括以下几个**：

* (1)可以在不获悉，不改变原来代码的情况下为现有类添加新的方法（只能添加新的方法，不能删除和修改方法）；
* (2)模拟多继承。[参考链接](https://www.jianshu.com/p/425ff8133f9e)
* (3)拆分复杂类的实现（将类的实现根据功能分散到多个不同文件中，避免单个类的臃肿，更易于管理代码）
* (4)把framework的私有方法公开
* (5)调用类的私有方法。[参考链接](https://www.jianshu.com/p/425ff8133f9e)

**分类(Category)的局限性**包括以下几点：

* (1)Category只能给某个现有的类添加方法，不能添加成员变量；
* (2)Category中也可以添加属性，只不过@property只会生成setter和getter的声明，不会生成setter方法和getter方法的实现，也不会自动合成带下划线的成员变量；
* (3)如果category中的方法和类中原有方法同名，运行时会优先调用category中的方法。也就是，category中的方法会覆盖掉类中原有的方法。所以开发中尽量保证不要让分类中的方法和原有类中的方法名相同。避免出现这种情况的解决方案是给分类的方法名统一添加前缀。比如category_。

分类的结构体struct_category_t里面的成员变量包括：对象方法列表、类方法列表、属性列表和协议列表等信息。


**【扩展 1-2】分类(category)的实现原理？**

Category编译之后的底层结构是struct_category_t类型的的结构体，这个结构体里面存储着分类的对象方法、类方法、属性、协议等信息。在程序运行的时候，runtime会将Category的数据，合并到类信息中（类对象、元类对象中）。runtime加载Category的大致过程是这样的：首先通过Runtime加载某个类的所有Category数据;然后把所有Category的方法、属性、协议数据，合并到一个大数组中，后面参与编译的Category数据，会写到数组的最前面；最后将合并后的所有分类数据（方法、属性、协议），插入到类原来数据的前面。

【注意】category并没有完全的替换掉原有类的同名方法，category的方法被放置在新方法列表的前面，而原来类的方法被放到后面，在runtime中，遍历方法列表查找时，找到了category的方法后，就会停止遍历，这就是我们平时所说的“覆盖”方法。此外，某个类如果存在两个category，而且这两个Category中存在方法名相同的方法，那么根据buildPhases->Compile Sources里面的从上至下的编译顺序，会调用后参与编译的那个分类的方法。

[分类(category)的实现原理详解](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E5%BC%80%E5%8F%91%E4%B9%8BCategory%E6%8E%A2%E7%A9%B6.md)


**【扩展 1-3】分类(Category)和类扩展(Class Extension)有什么区别？**

分类(Category)是Objective-C 2.0之后增加的语言特性，Category的主要作用是为已经存在的类添加方法。

类扩展（Extension）是category的一个特例，也称为匿名分类（类扩展与分类相比只少了分类的名称）。类扩展的作用是为一个类添加一些额外的私有属性、成员变量和私有方法。

**不同点：**

* (1)Class Extension是在编译期决定，Category由运行时决定。也就是说Class extension在编译的时候，它的数据就已经包含在类信息中；而Category（分类）是在运行时，才会将分类数据合并到类信息中（类对象、元类对象中） 
* (2)类扩展即可以添加方法又可以添加成员变量；分类(Category)中只能添加方法，不能直接添加成员变量、属性(属性仅仅是声明，并没有真正实现)
* (3)**在类扩展(Extension)中添加的新方法一定要实现**，而分类(category)中没有这种限制
* (4)类扩展中声明的方法没被实现，编译器会报警告，但是分类(Category)中的方法没被实现编译器是不会有任何警告的（因为类扩展是在编译阶段被添加到类中，而类别是在运行时添加到类中）
* (5)分类有名字，类扩展没有分类名字，即匿名分类，类扩展使一种特殊的分类。

【注意】继承可以增加、修改和删除方法，且可以添加属性。


**【扩展 1-4】Category能否给类添加成员变量？如果可以,如何给Category添加成员变量？**

默认情况下，由于分类底层结构的限制，不能直接给Category添加成员变量,但是可以间接实现。通过“关联对象”的方式来间接给Category添加成员变量。也就是通过“<objc/runtime.h>”中提供的关联对象的API来实现。“<objc/runtime.h>”中提供的关联对象的API有以下3个：

(1)添加关联对象

```
void objc_setAssociatedObject(id object, const void * key,id value, objc_AssociationPolicy policy)
```

(2)获得关联对象

```
id objc_getAssociatedObject(id object, const void * key)
```

(3)移除所有的关联对象

```
void objc_removeAssociatedObjects(id object)
```

例如给HLPerson类的Category分类HLPerson+Test类添加一个_name成员变量和一个_weight成员变量。代码如下：

```
#import "HLPerson.h"
@interface HLPerson (Test)
@property (copy, nonatomic) NSString *name;
@property (assign, nonatomic) int weight;
@end
```

```
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

**【扩展 1-5】Category中有load方法吗？load方法是什么时候调用的？load方法能继承吗？**

Category中有+load方法。load方法在runtime（运行时）加载类、分类的时候调用。+load方法能继承。而且+load方法一般都由系统自动调用，不手动调用。

**【扩展 1-6】load方法和initialize方法的异同点是什么？**

**load方法和initialize方法的相同点**：

如果父类和子类的load方法或initialize方法都被调用,那么父类的调用一定在子类之前。

**load方法和initialize方法的区别**：

* （1）调用方式不同。load是根据函数地址直接调用，而initialize是通过objc_msgSend调用
* （2）调用时刻不同。load是在runtime加载类、分类的时候由系统自动调用（在main函数执行之前被调用而且每个类的load方法只会调用1次），而initialize是类第一次接收到消息的时候调用，每一个类只会initialize一次（父类的initialize方法可能会被调用多次，这是因为有些子类可能没有实现initialize方法，那么在初始化子类时就会调用父类的initialize方法）。

**【扩展 1-7】load方法和initialize方法在category中的调用顺序是怎样的？以及出现继承时它们之间的调用过程是怎样的？**


**+load方法在Category中的调用顺序如下**：

* (1)先调用类的+load方法，再调用分类的+load方法；
* (2)先调用父类的+load方法，再调用子类的+load方法；
* (3)先编译“先参与编译的类”，后编译“后参与编译的类”；
* (4)当子类未实现load方法时,不会调用父类load方法。

**initialize方法在Category中的调用顺序如下**：

* (1)先执行父类的initialize方法再执行子类的initialize；
* (2)当子类未实现initialize方法时,会调用父类initialize方法,子类实现initialize方法时,会覆盖父类initialize方法;
* (3)当有多个Category都实现了initialize方法,会覆盖类中的方法,只执行一个(会执行Compile Sources 列表中最后一个Category 的initialize方法)


【注意】: 

load调用时机比较早,当load调用时,其他类可能还没加载完成,运行环境不安全，所以我们应该尽量减少load方法的逻辑。load方法是线程安全的，它使用了锁，我们应该避免线程阻塞在load方法。

在initialize方法收到调用时,运行环境基本健全。 initialize内部也使用了锁，所以是线程安全的。但同时要避免阻塞线程，不要再使用锁。

**【扩展 1-8】+load方法的使用场景？**

**+load方法的使用场景**：交换两个方法的实现

```
//摘自MJRefresh
+ (void)load
{
    [self exchangeInstanceMethod1:@selector(reloadData) method2:@selector(mj_reloadData)];
    [self exchangeInstanceMethod1:@selector(reloadRowsAtIndexPaths:withRowAnimation:) method2:@selector(mj_reloadRowsAtIndexPaths:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(deleteRowsAtIndexPaths:withRowAnimation:) method2:@selector(mj_deleteRowsAtIndexPaths:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(insertRowsAtIndexPaths:withRowAnimation:) method2:@selector(mj_insertRowsAtIndexPaths:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(reloadSections:withRowAnimation:) method2:@selector(mj_reloadSections:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(deleteSections:withRowAnimation:) method2:@selector(mj_deleteSections:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(insertSections:withRowAnimation:) method2:@selector(mj_insertSections:withRowAnimation:)];
}

+ (void)exchangeInstanceMethod1:(SEL)method1 method2:(SEL)method2
{
    method_exchangeImplementations(class_getInstanceMethod(self, method1), class_getInstanceMethod(self, method2));
}

```

**【扩展 1-9】+initialize方法的使用场景？**

**+initialize方法的使用场景**：主要用来对一些不方便在编译期初始化的对象进行赋值。比如NSMutableArray这种类型的实例化依赖于runtime的消息发送，所以显然无法在编译器初始化：

```
// int类型可以在编译期赋值
static int someNumber = 0; 
static NSMutableArray *someArray;
+ (void)initialize {
    if (self == [Person class]) {
        // 不方便编译期赋值的对象在这里赋值
        someArray = [[NSMutableArray alloc] init];
    }
}

```

<br />
<br />
<br />

**参考链接：**

* [Objective-C +load vs +initialize](http://blog.leichunfeng.com/blog/2015/05/02/objective-c-plus-load-vs-plus-initialize/)
* [Objective-C Category 的实现原理](http://blog.leichunfeng.com/blog/2015/05/18/objective-c-category-implementation-principle/)
* [iOS类方法load和initialize详解](https://juejin.im/post/5a31dc40f265da4307034712)
