## 知识点：Runtime（运行时）

[Runtime 题目参考](https://www.cnblogs.com/Hakim/p/6549474.html)

**【扩展 1-1】Runtime 如何实现 weak 变量自动置为nil的？**(runtime如何实现 weak 属性的？)

首先要清楚 weak 属性的特点。weak 策略表明该属性定义了一种“非拥有关系”（nonowning relationship）。当为 weak 修饰的属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同 assign 类似。但与 assign 不同的是，weak属性所指的对象销毁时，属性值会被清空（ nil out ），即置为 nil。

那么 runtime 是如何实现将 weak 变量自动置为 nil 的呢？

Runtime 对注册的类，会进行布局，会将 weak 对象存入一个 hash 表中。用 weak 指向的对象的内存地址作为 key，当这个对象的引用计数为 0 时会调用该对象的 dealloc 方法。假设 weak 指向的对象内存地址是 a，那么就会以 a 作为 key，在这个 weak hash 表中进行搜索，查找到所有以 a 为 key 的 weak 对象，并将查找到的这些 weak 对象都置为 nil。

**【扩展 1-1.1】weak 属性需要在 dealloc 中置为 nil 吗？**

不需要。

因为在 ARC 环境下，无论是强指针还是弱指针都无需在 dealloc 中设置为 nil，因为 ARC 会自动帮我们处理。即便是编译器不帮我们处理，weak 属性也不需要再 dealloc 中置为 nil。因为在属性所指向的对象销毁时，属性值也会清空（置为nil）。

```
// 模拟 weak 的 setter 方法。 大致如下：
- (void)setObject:(NSObject *)object {
  objc_setAssociatedObject(self, "object", object, OBJC_ASSOCIATION_ASSIGN);
  [object cyl_runArDealloc:^{
  	_object = nil;
  }];
}
```

**【扩展 1-2】OC的消息机制是什么？**

在Objective-C中，**消息机制**是指方法调用时向方法调用者发送消息的机制。

OC的方法调用，本质上都会在运行时被动态地转化为 objc_msgSend 函数的调用，objc_msgSend(receiver, selector)函数给 receiver (方法调用者)发送了一条消息(@selector(方法名))。objc_msgSend 函数底层实现可以分为3大阶段，这三大阶段分别是消息发送阶段、动态方法解析阶段和消息转发阶段。


**【扩展 1-3】OC的消息机制的实现原理是什么？（什么情况下会报 [NSObject(NSObject) doesNotRecognizeSelector:] 这个错误？）**

OC的方法调用是指给方法调用者发送消息，也称为**消息机制**。OC的方法调用，本质上都会在运行时被动态地转化为objc_msgSend函数的调用，objc_msgSend(receiver, selector)函数给receiver(方法调用者)发送了一条消息(@selector(方法名))。objc_msgSend函数底层实现可以分为3大阶段。分别是消息发送阶段、动态方法解析阶段和消息转发阶段。

* （1）**消息发送阶段。**  objc_msgSend(person, @selector(test))，在消息发送阶段会先查找 test 方法是否存在，如果存在，直接调用，如果不存在，再进入动态方法解析阶段。

![objc_msgSend执行流程-消息发送阶段示意图.png](https://upload-images.jianshu.io/upload_images/4164292-f7eb12d7a09294a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**消息发送阶段详细流程**：首先判断 receiver 是否为 nil，为 nil 的话，直接 retrun。如果 receiver 不为 nil，那么就利用 receiver 的 isa 找到receiver 的 Class 对象或 Meta-Class 对象。接下来，先从 receiver 自己的 Class 对象的 cache 中查找方法，如果找到了该方法，直接调用，并结束查找。如果没有找到方法，那么就从 receiver 的 Class 对象的 class_rw_t 中 methods 数组(dispatch table 调度表)里查找方法，如果数组已经排序，采用“二分查找”；如果没有排序，则采用普通遍历查找。如果在 methods 数组里找到了方法，则调用方法，结束查找并将该方法缓存到 receiver 的 Class 对象的cache 中；如果没有找到方法，那么就通过receiver Class 的 superclass 指针找到 SuperClass，接下来从 SuperClass 的 cache 中查找方法，如果找到了方法，则调用该方法，结束查找，并将该方法缓存到 receiver 的 Class 对象的 cache 中；如果没有找到方法，接着会从 SuperClass 的 class_rw_t 中查找方法，同样地，也存在采用“二分查找”和“遍历查找”的判断，如果找到了方法，则调用方法，结束查找并将该方法缓存到 receiver 的 Class 对象的 cache 中；如果没有找到方法，此时会判断是否存在更高一级的 superClass（父类），如果存在 superClass，那么继续从 superClass 的 cache 中查找方法，继续上面的所述的在 superClass 中的查找过程；如果上层不存在 superClass 了，那么此时就会进入动态方法解析阶段。

【注意】为了保证消息发送与执行效率，系统会将全部的 selector 和使用过的方法的内存地址缓存起来。每个类都有一个独立的缓存，缓存包含当前类自己的 selector 以及继承自父类的 selector。查找调度表(dispatch table)前，消息发送系统会首先检查 receiver 对象的缓存。

* （2）**动态方法解析阶段。**  此阶段允许开发者利用runtime运行时机制动态添加方法实现。如果在动态方法解析阶段，开发者没有执行任何操作，那么将进入“消息转发”阶段。

![objc_msgSend执行流程02-动态方法解析流程示意图.png](https://upload-images.jianshu.io/upload_images/4164292-0b4dc6651b207d00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**动态方法解析阶段详细流程**：首先判断“是否已经动态解析”(if(resolver && !triedResolver))，如果没有动态解析过，那么就会调用+resolveInstanceMethod:或者+resolveClassMethod:方法来动态解析方法，开发者可以在+resolveInstanceMethod:或者+resolveClassMethod:方法里面利用runtime API来动态添加方法实现；动态解析完成后，会标记为已经动态解析(triedResolver = YES)，重新执行“消息发送”的流程，也就是重新从receiver Class的cache中查找方法。如果一开始判断为经动态解析过，那么将转入“消息转发阶段”。需要特别注意的是：如果是为“类方法”动态添加实现的话，class_addMethod的第一个参数必须是Meta-Class对象,也就是objc_getClass(self)。

```
//动态添加为test方法的实现
- (void)other
{
    NSLog(@"%s",__func__);
}
//调用+resolveInstanceMethod方法进行动态解析方法，可以在此方法中动态添加方法。
+ (BOOL)resolveInstanceMethod:(SEL)sel
{
    if (sel == @selector(test)) {
        //获取其他方法(比如other方法)：class_getInstanceMethod(Class _Nullable cls, SEL _Nonnull name)
        Method method = class_getInstanceMethod(self, @selector(other));
        //利用runtime为test方法动态添加方法实现（other方法）
        //class_addMethod(Class _Nullable cls, SEL _Nonnull name, IMP _Nonnull imp,const char * _Nullable types),如果是为“类方法”动态添加实现的话，class_addMethod方法的第一个参数必须是Meta-Class对象,也就是objc_getClass(self)
        class_addMethod(self, sel, method_getImplementation(method), method_getTypeEncoding(method));
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
```

* （3）**消息转发阶段。**  消息转发是指将方法转发给其他调用者。

![objc_msgSend的执行流程03-消息转发流程示意图.png](https://upload-images.jianshu.io/upload_images/4164292-9b726d453273809c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**消息转发阶段的详细流程**：首先调用 forwardingTargetForSelector: 方法，如果方法返回值不为 nil，那么就执行objc_msgSend(返回值,SEL)，如果返回值为 nil，则调用 methodSignnatureForSelector: 方法，如果该方法返回值为nil，则调用 doesNotRecognizeSelector: 方法并抛出错误"unrecognized selector sent to instance"；如果 methodSignnatureForSelector: 方法的返回值不为 nil，就会调用 forwardInvocation: 方法，开发者可以在forwardInvocation: 方法里自定义任何处理逻辑。

```
//消息转发
- (id)forwardingTargetForSelector:(SEL)aSelector
{
    if (aSelector == @selector(test)) {
        //objc_msgSend(cat,aSelector);
        return [[Cat alloc]init];
        //如果返回nil，则调用methodSignatureForSelector:方法
//        return nil;
    }
    return [super forwardingTargetForSelector:aSelector];
}
//方法签名：返回值类型、参数类型
/*
- (void) test;
v    16      @     0     :     8
void         id          SEL
解释：16表示参数的占用空间大小，id后面跟的0表示从0位开始存储，id占8位空间。SEL后面的8表示从第8位开始存储，SEL同样占8位空间
*/
/*
- (int)testWithAge:(int)age Height:(float)height;

  i    24    @    0    :    8    i    16    f    20
int         id        SEL       int        float
解释：参数的总占用空间为 8 + 8 + 4 + 4 = 24，id类型的参数从第0位开始占据8位空间，SEL类型的参数从第8位开始占据8位空间，int类型的参数从第16位开始占据4位空间，float类型的参数从第20位开始占据4位空间。
*/
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
{
    if (aSelector == @selector(test)) {
        //方法签名,如果返回的方法签名为空，那么将不再调用-(void)forwardInvocation:(NSInvocation *)anInvocation方法，并抛出错误
        return [NSMethodSignature signatureWithObjCTypes:"v16@0:8"];
        //如果返回nil，调用doesNotRecognizeSelector:方法并抛出错误"unrecognized selector sent to instance";如果返回不为nil，则调用forwardInvocation:方法。
//        return nil;
    }
    return [super methodSignatureForSelector:aSelector];
}
//NSInvocation封装了一个方法调用，包括方法调用者、方法名、方法参数
//anInvocation.target 方法调用者
//anInvocation.selector 方法名
//[anInvocation getArgument:NULL atIndex:0];
-(void)forwardInvocation:(NSInvocation *)anInvocation
{
    [anInvocation invokeWithTarget:[[Cat alloc]init]];
}
```

如果以上3个阶段都无法完成消息调用，那么将调用 doesNotRecognizeSelector: 方法报错"unrecognized selector sent to instance XXX"。

**【扩展 1-4】什么是 Runtime ？平时项目中用过吗( Runtime 应用场景有哪些)？**

OC是一门动态性比较强的编程语言，允许很多操作推迟到程序运行时再进行。OC的动态性就是由 Runtime 来支撑的，Runtime是一套C语言的API，封装了很多动态性相关的函数。平时编写的OC代码，底层都是转换成了Runtime API进行调用。

**Runtime的应用场景**：

Runtime应用场景1：**间接动态地给分类(Category)添加成员变量**

Runtime应用场景2：**动态交换两个方法的实现(Method Swizzling)**

[Method Swizzling](https://nshipster.cn/method-swizzling/)

实际项目中，主要是用来替换系统自带的方法实现。举个例子：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSString *obj = nil;
    NSMutableDictionary *dic = [NSMutableDictionary dictionary];
    dic[@"name"] = @"Jack";
    //当key为nil时，会崩溃。此时就可以利用runtime交换方法的API来处理。
    dic[obj] = @"111";
    NSLog(@"%@",dic);
}


#import "NSMutableDictionary+Extention.h"
#import <objc/runtime.h>
@implementation NSMutableDictionary (Extention)
+(void)load
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
    	//类簇:NSMutableArray、NSString、NSArray的真正类型是其他类型(比如：__NSArrayM).
    	Class cls = NSClassFromString(@"__NSDictionaryM");
    	Method method1 = class_getInstanceMethod(cls, @selector(setObject:forKeyedSubscript:));
    	Method method2 = class_getInstanceMethod(cls, @selector(hl_setObject:forKeyedSubscript:));
    	method_exchangeImplementations(method1, method2);
    }); 
}
- (void)hl_setObject:(id)obj forKeyedSubscript:(id<NSCopying>)key
{
    NSLog(@"111");
    if(key == nil) return;
    [self hl_setObject:obj forKeyedSubscript:key];
}
@end
```

Runtime应用场景3：**实现 NSCoding 的自动归档和解档(一键序列化)**

实现原理：利用runtime提供的函数遍历自身所有属性，并对属性进行encode和decode操作。

```
//归档
- (void)encodeWithCoder:(NSCoder *)aCoder
{
    // 一层层父类往上查找，对父类的属性执行归解档方法
    Class c = self.class;
    while (c &&c != [NSObject class]) {
        //ivar数量
        unsigned int outCount = 0;
        //class_copyIvarList：获取类中的成员变量列表
        Ivar *ivars = class_copyIvarList([self class], &outCount);
        for (int i = 0; i < outCount; i++) {
            //获取ivar
            Ivar ivar = ivars[i];
            //获取属性名称
            //ivar_getName:获取类的成员变量名称
            NSString *key = [NSString stringWithUTF8String:ivar_getName(ivar)];
            
            id value = [self valueForKeyPath:key];
            //归档
            [aCoder encodeObject:value forKey:key];
        }
        //释放内存，C语言中涉及到copy、new、creat等函数的创建的对象都需要手动释放。
        free(ivars);
        c = [c superclass];
    }
}
//解档
- (instancetype)initWithCoder:(NSCoder *)aDecoder
{
    if (self = [super init]) {
        // 一层层父类往上查找，对父类的属性执行归解档方法
        Class c = self.class;
        while (c &&c != [NSObject class]) {
            unsigned int outCount = 0;
            //class_copyIvarList：获取类中的成员变量列表
            Ivar *ivars = class_copyIvarList(c, &outCount);
            for (int i = 0; i < outCount; i++) {
                //取出Ivar
                Ivar ivar = ivars[i];
                //获取属性名称
                NSString *key = [NSString stringWithUTF8String:ivar_getName(ivar)];
                //解档
                id value = [aDecoder decodeObjectForKey:key];
                //KVC 设值
                [self setValue:value forKey:key];
            }
            //释放
            free(ivars);
            c = [c superclass];
        }
    }
    return self;
}
```

Runtime应用场景4：**实现字典转模型的自动转换**

实现原理是使用runtime遍历出模型中的所有属性，根据模型中属性,去字典中取出对应的value再利用KVC给模型属性设置对应的值。

字典转模型第三方库：[MJExtension](https://my.oschina.net/wolx/blog/396925)、JSONModel、Mantle等

Runtime应用场景5：**访问私有变量**

OC中没有真正意义上的私有变量和方法，要让成员变量私有，要放在 .m 文件中声明，不对外暴露。如果我们知道这个成员变量的名称，可以通过runtime获取成员变量，再通过getIvar来获取它的值。

```
//获取一个实例变量信息
Ivar nameIvar = class_getInstanceVariable([HLPerson class], "_name");
//设置成员变量的值
object_setIvar(person, nameIvar, @"BHL");
//获取成员变量的值
NSString *name = object_getIvar(person, nameIvar);
NSLog(@"name=%@",name);//name=BHL
```

**【扩展 1-5】runtime中，SEL和IMP的区别？**

[SEL和IMP的区别](https://www.jianshu.com/p/02b12e98fc98)

SEL是类成员方法的指针，表示方法名称，类似字符串(可互转);IMP是函数指针，保存了方法的地址。

IMP和SEL关系：SEL(方法编号)最终会通过Dispatch table表(调度表)寻找到对应的IMP(函数指针)，Dispatch table表存放SEL和IMP的对应。

【延伸】_cmd在Objective-C的方法中表示当前方法的selector，类似self表示当前方法调用的对象实例。

**【扩展 1-6】runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）**

每一个类对象中都有一个对象方法列表（对象方法缓存）；类方法列表是存放在类对象中isa指针指向的元类对象中（类方法缓存）；方法列表中每个方法结构体中记录着方法的名称,方法实现,以及参数类型，其实selector本质就是方法名称,通过这个方法名称就可以在方法列表中找到对应的方法实现；当我们给一个实例对象发送消息时，这条消息会在实例对象的类对象方法列表里查找；当我们给一个类对象发送一条消息时，这条消息会在类的Meta Class对象的方法列表里查找。

**【扩展 1-7】下面代码打印结果是什么？（☆☆☆☆☆）**

```
@interface HLPerson : NSObject
@end

@interface HLStudent : HLPerson
@end

@implementation HLStudent
- (instancetype)init
{
    if (self = [super init]) {
    	//objc_msgSend(self, @selector(class))，消息接收者receiver(方法调用者) 是 self，即 HLStudent。
        NSLog(@"[self class]=%@",[self class]);//HLStudent
        NSLog(@"[self superclass]=%@",[self superclass]);//HLPerson
        
        //转化为C++底层实现为：objc_msgSendSuper((self,[HLPerson Class]), @selector(run)); 消息接收者receiver 仍然是 self，即 HLStudent。
        //[super class]的作用是：查找方法的时候是从父类开始查找。
        NSLog(@"[super class]=%@",[super class]);//[super class]=HLStudent
        NSLog(@"[super superclass]=%@",[super superclass]);//[super superclass]=HLPerson
    }
    return self;
}
@end 
//NSObject 中 class 方法的实现
@implementation NSObject
- (Class)class 
{
   return object_getClass(self);//获取消息接收者自身对应的类。
}
// superclass 方法的实现
- (Class)superclass
{
   return class_getSuperclass(object_getClass(self));//获取消息接收者的父类。
}
@end
```

打印结果如下：

```
2019-06-21 18:14:08.706538+0800 Interviews_Super_SuperClass[17851:1105221] [self class]=HLStudent
2019-06-21 18:14:08.706728+0800 Interviews_Super_SuperClass[17851:1105221] [self superclass]=HLPerson
2019-06-21 18:14:08.706864+0800 Interviews_Super_SuperClass[17851:1105221] [super class]=HLStudent
2019-06-21 18:14:08.706988+0800 Interviews_Super_SuperClass[17851:1105221] [super superclass]=HLPerson
```

**原因：** 主要是因为消息接收者仍然为子类HLStudent。

**总结：**

```
 [super message]的底层实现：
 （1）消息接收者仍然是子类对象；
 （2）从父类开始查找方法的实现。
 ```
 
 **【扩展 1-8】下面代码打印结果是什么？**
 
 ```
 BOOL res1 = [[NSObject class] isKindOfClass:[NSObject class]];
    BOOL res2 = [[NSObject class] isMemberOfClass:[NSObject class]];//0
    BOOL res3 = [[HLPerson class] isKindOfClass:[HLPerson class]];//0
    BOOL res4 = [[HLPerson class] isMemberOfClass:[HLPerson class]];//0
    BOOL res5 = [[HLPerson class] isKindOfClass:[NSObject class]];//1
    BOOL res6 = [[HLPerson class] isMemberOfClass:[NSObject class]];//1
    NSLog(@"res1=%d,res2=%d,res3=%d,res4=%d,res5=%d,res6=%d",res1,res2,res3,res4,res5,res6);
   
    
    HLPerson *person = [[HLPerson alloc]init];
    BOOL res7 = [person isKindOfClass:[HLPerson class]];//1
    BOOL res8 = [person isKindOfClass:[NSObject class]];//1
    
    BOOL res9 = [person isMemberOfClass:[HLPerson class]];//1
    BOOL res10 = [person isMemberOfClass:[NSObject class]];//0
    NSLog(@"res7=%d,res8=%d,res9=%d,res10=%d",res7,res8,res9,res10);
 ```
 
 打印结果如下：
 
```
res1=1,res2=0,res3=0,res4=0,res5=1,res6=0
res7=1,res8=1,res9=1,res10=0
```

总结：主要考察isKindOfClass、isMemberOfClass这两个方法对应的对象方法和类方法的区别。**需要特别注意的是：NSObject（基类Root class）的superclass 指针指向 NSObject 的类对象(Root class)，也就是基类的 Superclass 指针指向基类的Class对象。这一点非常特殊。这也是 res5 结果为 1 的原因**。

```
-(BOOL)isMemberOfClass：判断左边对象是否正好等于右边这种类型
+(BOOL)isMemberOfClass：判断左边对象的 Meta-Class 对象是否等于右边的对象
-(BOOL)isKindOfClass：判断左边对象是不是右边这种类型或者右边这种类型的子类。
+(BOOL)isKindOfClass：判断左边的对象的 Meta-Class 对象是否是右边对象或者右边对象的子类
```

具体源码实现如下：

```
-(BOOL)isMemberOfClass:(Class)cls {
   return [self class] == cls;
}
+(BOOL)isMemberOfClass:(Class)cls {
   return object_getClass((id)self) == cls;
}

-(BOOL)isKindOfClass:(Class)cls {
  for (Class tcls = [self class]; tcls; tcls = tcls->superclass) {
    if (tcls == cls) return YES;
  }
  return NO;
}
-(BOOL)isKindOfClass:(Class)cls {
  for (Class tcls = object_getClass((id)self); tcls; tcls = tcls->superclass) {
    if (tcls == cls) return YES;
  }
  return NO;
}
```

**【扩展 1-9】iOS中内省方法有哪些？class方法和objc_getClass方法有什么区别？**

内省是指对象在运行时获取对象详细信息的能力。这些详细信息包括对象在继承树上的位置，对象是否遵循了特定的协议以及对象是否可以响应特定的消息等。

OC中内省的方法有几个方法：

(1)判断对象类型：

*  -(BOOL)isKindOfClass: 判断是否是这个类或其子类的实例
*  -(BOOL)isMemberOfClass: 判断是否是这个类的实例

(2)判断对象/类是否有这个方法：

*  -(BOOL)respondsToSelector: 这是一个实例方法，用来判断该实例对象是否响应某个方法
*  +(BOOL)instancesRespondToSelector: 这是一个类方法，用来判断类该类的实例对象是否响应某个方法

(3)检查对象是否符合某协议

* -(BOOL)conformsToProtocol:(Protocol *)protocol;

**class 方法和 objc_getClass 方法的区别**

* objc_getClass方法：用于获取isa指针指向的对象。比如：类对象调用objc_getClass方法，得到的是对应的 meta class(元类对象)。
* class方法：实例对象调用class方法，返回的是类对象，类对象调用class方法，返回的是类对象自身。



**【扩展 1-10】以下代码能不能执行成功？如果可以，打印结果是什么？**

本题目考察的知识点：**super调用的底层本质、函数栈空间分配、消息机制、访问成员变量的本质**。

```
@interface HLPerson : NSObject
@property (nonatomic, copy) NSString *name;
- (void)print;
@end

@implementation HLPerson
- (void)print {
	NSLog(@"my name is %@",self.name);
}
@end

@implemetation ViewController
- (void)viewDidLoad{
	[super viewDidLoad];

	NSString *test = @"123";
	id cls = [HLPerson class];
	void *obj = &cls;
	[(__bridge id)obj print];
}
@end
```

能执行成功。打印结果是“my name is 123”。

两个问题：

* 1.为什么 print 能够调用成功？
  obj -> cls -> [HLPerson class] 等价于 person -> isa -> [HLPerson class]
* 2.为什么 self.name 变成了 ViewController或者“123”等其他内容？

```
// 局部变量存储在栈空间。栈空间分配内容空间时是从高地址到低地址。
void test() 
{
  long long a = 4; // 0x7ffeef615ff8
  long long b = 5; // 0x7ffeef615ff0
  long long c = 6; // 0x7ffeef615fe8
  NSLog(@"%p %p %p", &a, &b, &c);
}
```

上述程序中局部变量栈中内存布局图如下：

![局部变量栈中内存布局图.png](https://upload-images.jianshu.io/upload_images/4164292-6c54f6b36d8d3036.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果上述程序中的 NSString * test = @"123"; 代码注释掉，[super viewDidLoad]; 代码本质上会转化为：

```
objc_msgSendSuper({
  self, 
  [UIViewController class]
  }, @selector(viewDidLoad));
```

也就是：

```
// 内部会生成一个局部变量类型结构体 superStruct
struct hlSuperStruct = {
    self, 
    [UIViewController class]
};
objc_msgSendSuper(hlSuperStruct, sel_registerName("viewDidLoad"));
```

这种情况下局部变量栈中内存布局图如下：

![superViewDidLoad.png](https://upload-images.jianshu.io/upload_images/4164292-a721447e41b71001.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以此时打印的是cls下面的 self 所指的 ViewController。


需要注意的是，[super viewDidLoad] 方法最终调用的方法是 objc_msgSendSuper2，而不是利用Clang命令生成的 C++ 代码中的 objc_msgSendSuper。

```
objc_msgSendSuper2({
  self, 
  [ViewController class]
  }, @selector(viewDidLoad));
```

结构体的第二个参数是当前类 [ViewController class]，而不是当前类的父类 [UIViewController class]。但是在 objc_msgSendSuper2方法的内部会通过结构体第二个参数（[ViewController class]）的superclass 指针查找当前类的父类。

**【扩展 1-11】在运行时创建的方法objc_allocateClassPair的方法名尾部为什么是pair（成对的意思）？**

因为要创建一对类，一个是Class(类)，另一个是meta-class(元类)，类和元类总是成对创建的。每一个类都有自己所属的元类。对象方法存储在类对象中，类方法存储在元类对象中。

**【扩展 1-12】说说你对 runtime 的理解？（百度）**

思路：主要是方法调用时如何查找缓存，如何找到方法，找不到方法时怎么转发，对象的内存布局。

**【扩展 1-13】说说对动态绑定的理解**

动态绑定是指在编译阶段无法确定调用哪个方法，只有到了**运行时才能确定去调用哪个方法**。

[深入Objective-C的动态特性](https://onevcat.com/2012/04/objective-c-runtime/)

