# 知识点汇总篇章三

## 知识点8：Runtime

**【扩展 8-1】Runtime如何实现weak变量自动置为nil的？**(待优化)

runtime对注册的类会进行布局，对于weak对象会放入一个hash表中。用weak指向的对象内存地址作为key，当这个对象的引用计数为0的时候会dealloc函数，假如weak指向的对象内存地址是a，那么就会以a为键，在这个weak表中搜索，查找到所有以a为键的weak对象，并将查找到的这些weak对象设置为nil。

**【扩展 8-2】OC的消息机制是什么？**

OC的方法调用是指给方法调用者发送消息，也称为**消息机制**。OC的方法调用，本质上都会在运行时被动态地转化为objc_msgSend函数的调用，objc_msgSend(receiver, selector)函数给receiver(方法调用者)发送了一条消息(@selector(方法名))。objc_msgSend函数底层实现可以分为3大阶段。分别是消息发送阶段、动态方法解析阶段和消息转发阶段。


**【扩展 8-3】OC的消息机制的实现原理是什么？**

OC的方法调用是指给方法调用者发送消息，也称为**消息机制**。OC的方法调用，本质上都会在运行时被动态地转化为objc_msgSend函数的调用，objc_msgSend(receiver, selector)函数给receiver(方法调用者)发送了一条消息(@selector(方法名))。objc_msgSend函数底层实现可以分为3大阶段。分别是消息发送阶段、动态方法解析阶段和消息转发阶段。

* （1）**消息发送阶段。**  objc_msgSend(person, @selector(test))，在此阶段会查找test方法是否存在，如果存在，直接调用，如果不存在，再进入动态方法解析阶段。

![objc_msgSend执行流程-消息发送阶段示意图.png](https://upload-images.jianshu.io/upload_images/4164292-f7eb12d7a09294a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**消息发送阶段详细流程**：首先判断receiver是否为nil，为nil的话，直接retrun。如果receiver不为nil，那么就利用receiver的isa找到receiver的Class对象或Meta-Class对象。接下来，先从receiver自己的Class的cache中查找方法，如果找到了该方法，直接调用，并结束查找。如果没有找到方法，那么就从receiver的Class对象的class_rw_t中methods数组(dispatch table 调度表)里查找方法，如果数组已经排序，采用“二分查找”；如果没有排序，则采用普通遍历查找。如果在methods数组里找到了方法，则调用方法，结束查找并将该方法缓存到receiver的Class的cache中；如果没有找到方法，那么就通过receiver Class的superclass指针找到SuperClass，接下来从SuperClass的cache中查找方法，如果找到了方法，则调用该方法，结束查找，并将该方法缓存到receiver的Class对象的cache中；如果没有找到方法，接着会从SuperClass的class _rw _t 中查找方法，同样地，也存在采用“二分查找”和“遍历查找”的判断，如果找到了方法，则调用方法，结束查找并将该方法缓存到receiver的Class的cache中；如果没有找到方法，此时会判断是否存在更高一级的superClass（父类），如果存在superClass，那么继续从superClass的cache中查找方法，继续上面的所述的在superClass中的查找过程；如果上层不存在superClass了，那么此时就会进入动态方法解析阶段。

【注意】为了保证消息发送与执行效率，系统会将全部的selector和使用过的方法的内存地址缓存起来。每个类都有一个独立的缓存，缓存包含有当前类自己的selector以及继承自父类的selector。查找调度表(dispatch table)前，消息发送系统会首先检查receiver对象的缓存。

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

**消息转发阶段的详细流程**：首先调用forwardingTargetForSelector:方法，如果方法返回值不为nil，那么就执行objc_msgSend(返回值,SEL)，如果返回值为nil，则调用methodSignnatureForSelector:方法，如果返回值为nil，则调用doesNotRecognizeSelector:方法并抛出错误"unrecognized selector sent to instance"；如果methodSignnatureForSelector:方法的返回值不为nil，就会调用forwardInvocation:方法，开发者可以在forwardInvocation:方法里自定义任何处理逻辑。

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

如果以上3个阶段都无法完成消息调用，那么将调用doesNotRecognizeSelector:方法报错"unrecognized selector sent to instance XXX"。

**【扩展 8-4】什么是Runtime？平时项目中用过么(Runtime应用场景有哪些)？**

OC是一门动态性比较强的编程语言，允许很多操作推迟到程序运行时再进行。OC的动态性就是由Runtime来支撑的，Runtime是一套C语言的API，封装了很多动态性相关的函数。平时编写的OC代码，底层都是转换成了Runtime API进行调用。

**Runtime的应用场景**：

Runtime应用场景1：**间接动态地给分类(Category)添加成员变量**

Runtime应用场景2：**动态地交换两个方法的实现-Method Swizzling**

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

Runtime应用场景3：**实现NSCoding的自动归档和解档(一键序列化)**

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

OC中没有真正意义上的私有变量和方法，要让成员变量私有，要放在m文件中声明，不对外暴露。如果我们知道这个成员变量的名称，可以通过runtime获取成员变量，再通过getIvar来获取它的值。

```
//获取一个实例变量信息
Ivar nameIvar = class_getInstanceVariable([HLPerson class], "_name");
//设置成员变量的值
object_setIvar(person, nameIvar, @"BHL");
//获取成员变量的值
NSString *name = object_getIvar(person, nameIvar);
NSLog(@"name=%@",name);//name=BHL
```

**【扩展 8-5】runtime中，SEL和IMP的区别？**

[SEL和IMP的区别](https://www.jianshu.com/p/02b12e98fc98)

SEL是类成员方法的指针，表示方法名称，类似字符串(可互转);IMP是函数指针，保存了方法的地址。

IMP和SEL关系：SEL(方法编号)最终会通过Dispatch table表(调度表)寻找到对应的IMP(函数指针)，Dispatch table表存放SEL和IMP的对应。

【延伸】_cmd在Objective-C的方法中表示当前方法的selector，类似self表示当前方法调用的对象实例。

**【扩展 8-6】runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）**

每一个类对象中都有一个对象方法列表（对象方法缓存）；类方法列表是存放在类对象中isa指针指向的元类对象中（类方法缓存）；方法列表中每个方法结构体中记录着方法的名称,方法实现,以及参数类型，其实selector本质就是方法名称,通过这个方法名称就可以在方法列表中找到对应的方法实现；当我们给一个实例对象发送消息时，这条消息会在实例对象的类对象方法列表里查找；当我们给一个类对象发送一条消息时，这条消息会在类的Meta Class对象的方法列表里查找。

**【扩展 8-7】下面代码打印结果是什么？**

```
@interface HLPerson : NSObject
@end

@interface HLStudent : HLPerson
@end

@implementation HLStudent
- (instancetype)init
{
    if (self = [super init]) {
        NSLog(@"[self class]=%@",[self class]);//HLStudent
        NSLog(@"[self superclass]=%@",[self superclass]);//HLPerson
        
        //objc_msgSendSuper((self,[HLPerson Class]), @selector(run));
        //[super class]的消息接收者receiver仍然是子类，也就是HLStudent,只不过查找方法的时候是从父类中查找
        NSLog(@"[super class]=%@",[super class]);//[super class]=HLStudent
        NSLog(@"[super superclass]=%@",[super superclass]);//[super superclass]=HLPerson
    }
    return self;
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
 
 **【扩展 8-8】下面代码打印结果是什么？**
 
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

总结：主要考察isKindOfClass、isMemberOfClass这两个方法对应的对象方法和类方法的区别。需要特别注意的是：NSObject的superclass指针指向NSObject的类对象，也就是基类的superclass指针指向基类的Class对象。这一点比较特殊。

```
-(BOOL)isMemberOfClass：判断左边对象是否刚好等于右边这种类型
+(BOOL)isMemberOfClass：判断左边对象的Meta-Class对象是否等于右边的对象
-(BOOL)isKindOfClass：判断左边对象是否是右边这种类型或者右边对象的子类。
+(BOOL)isKindOfClass：判断左边的对象的Meta-Class对象是否是右边对象或者右边对象的子类
```

**【扩展 8-9】iOS中内省方法有哪些？class方法和objc_getClass方法有什么区别？**

内省是指对象在运行时获取对象详细信息的能力。这些详细信息包括对象在继承树上的位置，对象是否遵循了特定的协议以及对象是否可以相应特定的消息等。

OC中内省的方法有几个方法：

(1)判断对象类型：

*  -(BOOL)isKindOfClass: 判断是否是这个类或其子类的实例
*  -(BOOL)isMemberOfClass: 判断是否是这个类的实例

(2)判断对象/类是否有这个方法：

*  -(BOOL)respondsToSelector: 这是一个实例方法，用来判断该实例对象是否响应某个方法
*  +(BOOL)instancesRespondToSelector: 这是一个类方法，用来判断类该类的实例对象是否响应某个方法

(3)检查对象是否符合某协议

* -(BOOL)conformsToProtocol:(Protocol *)protocol;

**class方法和objc_getClass方法的区别**

* objc_getClass方法：该方法用于获取isa指针。
* class方法：实例对象调用class方法，返回的是类对象，类对象调用class方法，返回的是类对象自身。类对象调用objc_getClass方法，得到的是对应的meta class(元类对象)。



**【扩展 8-10】以下代码能不能执行成功？如果可以，打印结果是什么？**

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
	
	NSString *string = @"123";
	id cls = [HLPerson class];
	void *obj = &cls;
	[(__bridge id)obj print];
}
@end
```

能执行成功。打印结果是“my name is 123”。

**【扩展 8-11】在运行时创建的方法objc_allocateClassPair的方法名尾部为什么是pair（成对的意思）？**

因为要创建一对类，一个是Class(类)，另一个是meta-class(元类)，类和元类总是成对创建的。每一个类都有自己所属的元类。对象方法存储在类对象中，类方法存储在元类对象中。

**【扩展 8-12】说说你对 runtime 的理解？（百度）**

思路：主要是方法调用时如何查找缓存，如何找到方法，找不到方法时怎么转发，对象的内存布局。

**【扩展 8-13】说说对动态绑定的理解**

动态绑定是指在编译阶段无法确定调用哪个方法，只有到了**运行时才能确定去调用哪个方法**。

[深入Objective-C的动态特性](https://onevcat.com/2012/04/objective-c-runtime/)


## 知识点9 RunLoop

**【扩展 9-1】讲讲RunLoop，项目中用过吗？(RunLoop的应用场景)**

* (1)控制线程生命周期（线程保活）

**线程保活的应用场景**：频繁执行一个任务或者多个串行（非并发）任务，可以采用线程保活的方式。线程保活的方式比传统的“创建线程-销毁线程-再创建线程-再销毁...”更节省CPU资源且更高效。比如AFNetworking中后台网络请求就使用了线程保活这种技术。

[线程保活示例](https://github.com/baohenglin/RunLoopThreadLive)

* (2)解决NSTimer在滑动时停止工作(失效)的问题。

**NSTimer在滑动时失效的原因**是NSTimer默认是工作在NSDefaultRunLoopMode模式下，而当我们滑动时，RunLoop会退出NSDefaultRunLoopMode模式，并进入UITrackingRunLoopMode模式，所有NSTimer失效。

解决方法：

```
NSTimer *timer = [NSTimer timerWithTimeInterval:self.completionDelay target:self selector:@selector(completionDelayTimerFired) userInfo:nil repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

* (3)监控应用卡顿
* (4)性能优化

**【扩展 9-2】RunLoop和线程的关系是怎样的？**

RunLoop和线程的关系如下：

* 每条线程都有唯一的一个与之对应的RunLoop对象。
* RunLoop保存在一个全局的Dictionary里，线程作为key，RunLoop作为value。
* 线程刚创建时并没有RunLoop对象，RunLoop会在第一次获取该线程时创建与之对应的RunLoop对象。
* RunLoop会在线程结束时销毁。
* 主线程的RunLoop已经自动获取（创建），子线程默认没有开启RunLoop。

**【扩展 9-3】程序中添加每3秒响应一次的NSTimer，当拖动tableView时，timer可能无法响应要怎么解决？**

NSTimer在滑动时失效的原因是NSTimer默认是工作在NSDefaultRunLoopMode模式下，而当我们滑动时，RunLoop会退出NSDefaultRunLoopMode模式，并进入UITrackingRunLoopMode模式，所有NSTimer失效。

【注意】[NSTimer scheduledTimerWithTimeInterval: repeats:block:]方法会自动将定时器添加到主线程的NSDefaultRunLoopMode模式下，如果要自定义RunLoop模式的话，可以使用timerWithTimeInterval方法创建定时器对象，并将定时器添加到当前线程的NSRunLoopCommonModes模式下(实际上是将timer定时器添加到了NSRunLoopCommonModes 模式下的CFMutableSetRef _commonModeItems数组中)，这样就能解决timer失效的问题。代码如下：

```
NSTimer *timer = [NSTimer timerWithTimeInterval:self.completionDelay target:self selector:@selector(completionDelayTimerFired) userInfo:nil repeats:YES];

[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

**【扩展 9-4】RunLoop是怎么响应用户操作的，具体流程是什么样的？**

当用户点击屏幕时，RunLoop内部的Source1会捕捉到该触屏事件，并将该事件包装成事件队列eventQueue交给Source0中进行处理。

**【扩展 9-5】说说RunLoop的几种状态？**

```
kCFRunLoopEntry = (1UL << 0),   //即将进入RunLoop
kCFRunLoopBeforeTimers = (1UL << 1),//即将处理Timer
kCFRunLoopBeforeSources = (1UL << 2),//即将处理Source
kCFRunLoopBeforeWaiting = (1UL << 5),//即将进入休眠
kCFRunLoopAfterWaiting = (1UL << 6),//即将从休眠中唤醒
kCFRunLoopExit = (1UL << 7),//即将退出RunLoop
```

**【扩展 9-6】RunLoop的mode作用是什么？**

RunLoop常见的mode有2种：kCFRunLoopDefaultMode(NSDefaultRunLoopMode)和UITrackingRunLoopMode。kCFRunLoopDefaultMode是默认模式，通常主线程是在这个模式下运行；UITrackingRunLoopMode用于ScrollView追踪触摸滑动，保证界面滑动时不受其他mode影响。

mode的作用是将不同模式下的Source0/Source1/Timer/Observer隔离开来，互不影响，这样就提高了执行效率和滑动流畅性。

**【扩展 9-7】timer和RunLoop是怎样的关系？**

* 从底层数据结构来看，RunLoop的__CFRunLoop结构体中存储着 CFMutableSetRef _modes,_modes是一个类似数组的集合，其所储存的数据类型是CFRunLoopModeRef, CFRunLoopModeRef结构体中存放着CFMutableArrayRef _timers。此外，如果timer被设置为 kCFRunLoopCommonModes模式，那么timer也将被存放在 __CFRunLoop结构体中的 CFMutableSetRef _commonModeItems数组中。
* 从RunLoop的运行逻辑来讲，timer会通知Observers结束休眠，唤醒线程来处理timer消息。

**【扩展 9-8】RunLoop内部实现逻辑(实现原理)是怎样的？**

[RunLoop的运行逻辑](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E5%BC%80%E5%8F%91%E4%B9%8BRunLoop%E6%8E%A2%E7%A9%B6.md)

**【扩展 9-9】RunLoop的概念以及作用是什么？**

RunLoop顾名思义也就是运行循环，在程序运行过程中循环执行某些任务。

**作用：**

* 保持程序的持续运行；
* 处理App中的各种事件（比如触摸事件、定时器事件等）；
* 节省CPU资源，提高程序性能：有待执行任务时执行任务，不执行任务时休眠。

RunLoop是一种让线程能随时处理事件但不退出的机制。RunLoop实际上是一个对象，这个对象管理者其需要处理的事件和消息，并提供了一个入口函数来执行Event Loop的逻辑。线程执行了这个函数后，就会一直处于这个函数内部“接收消息->等待消息->处理消息”的循环中，直到这个循环结束(比如传入quit消息)，函数返回。RunLoop机制会使线程在没有消息需要处理的时候休眠，在有消息到来时，线程立刻被唤醒。这样有效节约了系统资源。

OS X和iOS系统中，提供了两个对象：CFRunLoopRef和NSRunLoop。其中CFRunLoopRef 是在 CoreFoundation 框架内的，它提供了纯C函数的 API，所有这些 API 都是线程安全的。NSRunLoop 是基于 CFRunLoopRef 的封装，提供了面向对象的 API，但是这些 API 不是线程安全的。

线程和 RunLoop 之间是⼀一对应的，其关系是保存在一个全局的 Dictionary 里。线程刚创建时并没有 RunLoop，如果你不主动获取，那它一直都不会有。RunLoop 的创建是发生在第一次获取时，RunLoop 的销毁是发生在线程结束时。你只能在一个线程的内部获取其 RunLoop(主线程除外)。

**【扩展 9-10】RunLoop的底层数据结构是怎样的？**

[RunLoop的底层数据结构](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E5%BC%80%E5%8F%91%E4%B9%8BRunLoop%E6%8E%A2%E7%A9%B6.md)

**【扩展 9-11】RunLoop的作用是什么？它的内部的工作机制了解么？（最好结合线程和内存管理来说）**

**【扩展 9-12】RunLoop 的基本概念，它是怎么休眠的？(阿里)**

**【扩展 9-13】程序中添加了每3秒响应一次的NSTimer，当拖动tableView或者scrollView时，timer可能无法响应要怎么解决？**

NSTimer在滑动时失效的原因是NSTimer默认是工作在NSDefaultRunLoopMode(kCFRunLoopDefaultMode)模式下，而当我们滑动时，RunLoop会退出NSDefaultRunLoopMode模式，并进入UITrackingRunLoopMode模式(切换成UITrackingRunLoopMode模式是为了保证滑动的流畅)，导致NSTimer失效。

【注意】[NSTimer scheduledTimerWithTimeInterval: repeats:block:]方法会自动将定时器添加到主线程的NSDefaultRunLoopMode模式下，如果要自定义RunLoop模式的话，可以使用timerWithTimeInterval方法创建定时器对象，并将定时器添加到当前线程的**NSRunLoopCommonModes模式**下(实际上是将timer定时器添加到了NSRunLoopCommonModes 模式下的CFMutableSetRef _commonModeItems数组中)，这样就能解决timer失效的问题。代码如下：

```
NSTimer *timer = [NSTimer timerWithTimeInterval:self.completionDelay target:self selector:@selector(completionDelayTimerFired) userInfo:nil repeats:YES];

[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

**【扩展 9-14】利用 runloop 解释一下页面的渲染过程**

当我们调用 [UIView setNeedsDisplay] 时，这时会调用当前 View.layer 的 [view.layer setNeedsDisplay]方法。这等于给当前的 layer 打上了一个脏标记，而此时并没有直接进行绘制工作。而是会到当前的 Runloop 即将休眠，也就是 beforeWaiting 时才会进行绘制工作。紧接着会调用 [CALayer display]，进入到真正绘制的工作。CALayer 层会判断自己的 delegate 有没有实现异步绘制的代理方法 displayer:，这个代理方法是异步绘制的入口，如果没有实现这个方法，那么会继续进行系统绘制的流程，然后绘制结束。

CALayer 内部会创建一个 Backing Store，用来获取图形上下文。接下来会判断这个 layer 是否有 delegate。如果有的话，会调用 [layer.delegate drawLayer:inContext:]，并且会返回给我们 [UIView DrawRect:] 的回调，让我们在系统绘制的基础之上再做一些事情。如果没有 delegate，那么会调用 [CALayer drawInContext:]。以上两个分支，最终 CALayer 都会将位图提交到 Backing Store，最后提交给 GPU。至此绘制的过程结束。

## 知识点10 多线程

**【扩展 10-1】简述你对多线程的理解**

谈多线程之前，我们先来了解以下进程的概念。进程是指在系统中正在运行的一个应用程序(同时打开QQ和Xcode,系统会分别启动2个进程)，其中，每个进程之间是独立的,每个进程均运行在其专用的且受保护的内存空间内。那线程是什么呢？线程是进程的基本执行单元,进程的所有任务都在线程中执行。一个进程想要执行任务,必须得有线程(每个进程至少要有一条线程,即主线程)。那什么是多线程呢？**一个进程中可以开启多条线程,每条线程可以并行(同时)执行不同的任务**。

**多线程的优势：**

* (1)可以提高资源利用率，比如CPU资源利用率和内存资源利用率；
* (2)可以提高程序运行效率;
* (3)可以将耗时操作放到子线程中执行，防止阻塞主线程。

**多线程的缺点：**

* （1）开启线程需要占用一定的内存空间；
* （2）线程越多，CPU在调度线程上的开销就越大；
* （3）程序设计更加复杂:比如线程之间的通信,多线程的数据共享，需要考虑死锁、读写安全等问题。

此外，iOS多线程方案有多种，包括pthread、NSThread、GCD和NSOperation等。

**【扩展 10-2】iOS的多线程方案有哪几种？你更倾向于哪一种？**

iOS的多线程方案有以下这几种：

* (1)pthread：是一套基于C语言的通用的多线程API，适用于Unix、Linux、Windows等系统，可跨平台移植，但是使用难度大；线程生命周期由程序员管理；从使用频率来看几乎不用。
* (2)NSThread：基于OC语言实现；使用更加面向对象，简单易用，可直接操作线程对象；线程生命周期由程序员管理；从使用频率来看偶尔使用。
* (3)GCD：基于C语言实现；旨在替代NSThread，充分利用设备的多核，执行效率更高；自动管理线程生命周期；从使用频率来看经常使用。
* (4)NSOperation：基于OC语言实现；底层是GCD，但比GCD多了一些更简单实用的功能；使用更加面向对象；自动管理线程生命周期；从使用频率来看经常使用。

更倾向于GCD和NSOPeration。

**【扩展 10-3】用过GCD吗？在项目中具体是如何使用的？**



**【扩展 10-4】GCD的队列类型有哪些？**

GCD的队列可以分为两大类型，分别是串行队列(Serial Dispatch Queue)和并发队列(Concurrent Dispatch Queue)。串行队列是指让任务一个接着一个地执行的队列（一个任务执行完毕后，再执行下一个任务）。并发队列是指可以让多个任务并发(同时)执行（自动开启多个线程同时执行任务）的队列。并发功能只有在异步(dispatch_async)函数下才有效。需要注意的是主队列(dispatch_queue queue = dispatch_get_main_queue();)其实也是串行队列。

**【扩展 10-5】说一下NSOperation(结合NSOperationQueue使用)和GCD的区别以及各自的优势？(阿里)**

**NSOperation和GCD的区别：**

* (1)从**底层实现**来看，GCD是基于C语言实现的系统服务，执行和操作简单高效；NSOperation是对GCD更高层次的抽象。GCG是iOS4.0推出的,主要针对多核CPU 做了优化；NSOperation 是 iOS2.0后推出的,iOS4.0之后重写了NSOperation。
* (2)从**依赖关系方面**来看，NSOperation可以设置两个NSOperation之间的依赖，方便的控制执行顺序；GCD无法直接设置这种依赖关系，不过GCD可以通过dispatch_barrier_async来实现这种效果；
* (3)**KVO(键值对观察)**，NSOperation可以容易监听判断Operation当前的状态(是否正在执行isExecuteing，是否取消isCancelled，是否已完成isFinished)，对此GCD无法通过KVO进行监听判断；
* (4)从设置**优先级方面**来看，NSOperation可以设置自身的优先级(但是优先级高的不一定先执行)；GCD只支持FIFO的队列，GCD只能设置队列的优先级，无法在执行的block设置优先级；
* (5)从**执行效率**方面来看，直接使用GCD效率会更高，NSOperation会多一点开销；
* (6)从**功能方面**来看，NSOperation可以方便地控制队列的停止/继续，也可以取消队列中所有的操作；而GCD不具备这些功能。

那么什么情况下使用NSOperation，什么情况下使用GCD呢？

**使用NSOperation的情况**：各个操作之间有依赖关系；操作需要取消暂停、并发管理；控制操作之间优先级；限制同时能执行的线程数量，让线程在某时刻停止/继续等。

**使用GCD的情况**：一般简单的多线程操作，都可以使用GCD，简单高效。

从编程原则来说，一般我们需要尽可能的使用高等级、封装完美的API，在必须时才使用底层的API。当需求简单，简洁的GCD或许是个更好的选择，而Operation queue 为我们提供了更多的选择。



**【扩展 10-6】线程安全的处理手段有哪些？**

[iOS中的10种线程同步方案](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E4%B9%8B%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%BA%95%E5%B1%82%E5%8E%9F%E7%90%86%E6%8E%A2%E7%A9%B6.md)

**【扩展 10-7】OC中的锁你了解哪些？使用以上这些锁需要注意哪些问题？**

[iOS中的10种线程同步方案](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E4%B9%8B%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%BA%95%E5%B1%82%E5%8E%9F%E7%90%86%E6%8E%A2%E7%A9%B6.md)

**【扩展 10-8】自旋锁和互斥锁的异同？**

自旋锁和互斥锁的**相同点**：

二者都能保证同一时间只有一个线程访问共享资源。都能保证线程安全。

自旋锁和互斥锁的**不同点**：

* 自旋锁：如果共享数据已经有其他线程加锁了，**不是通过休眠使线程阻塞，而是在获取锁之前一直处于忙等(自旋)阻塞状态，一直占用CPU资源**。一旦被访问的资源被解锁，则等待资源的线程会立即执行。自旋锁的效率高于互斥锁。自旋锁用在以下情况：锁持有的时间短，而且线程并不希望在重新调度上花太多的成本。
* 互斥锁：如果共享数据已经有其他线程加锁了，线程会**进入休眠状态**等待资源被解锁。一旦被访问的资源被解锁，则等待资源的线程会被唤醒

**【扩展 10-9】什么情况下使用自旋锁比较好？什么情况下使用互斥锁比较好？**

(1)什么情况下使用自旋锁比较划算？

* 预计线程等待锁的时间很短。
* 加锁的代码（临界区）经常被调用，但竞争情况很少发生。
* CPU资源不紧张
* 多核处理器

(2)什么情况下使用互斥锁比较划算？

* 预计线程等待锁的时间较长
* 单核处理器
* 临界区有IO操作（因为IO操作都占用CPU资源比较大）
* 临界区代码复杂或者循环量大
* 临界区竞争非常激烈

**【扩展 10-10】任选C/OC/C++其中一种语言，实现自旋锁和互斥锁？**

**【扩展 10-11】CoreData的使用，如何处理多线程问题？(阿里)**

**【扩展 10-12】使用GCD如何实现这个需求：A、B、C三个任务并发，完成后执行任务D？(阿里)**

使用GCD的 dispatch_group 来实现此需求，代码如下：

```
dispatch_queue_t dispatchQueue = dispatch_queue_create("bhl.queue.next", DISPATCH_QUEUE_CONCURRENT);
    dispatch_group_t dispatchGroup = dispatch_group_create();
    dispatch_group_async(dispatchGroup, dispatchQueue, ^{
        NSLog(@"任务A");
    });
    dispatch_group_async(dispatchGroup, dispatchQueue, ^{
        NSLog(@"任务B");
    });
    dispatch_group_async(dispatchGroup, dispatchQueue, ^{
        NSLog(@"任务C");
    });
    dispatch_group_notify(dispatchGroup, dispatchQueue, ^{
        NSLog(@"任务D");
    });
```

dispatch_group_notify这个方法表示把block(第三个参数)传入队列(第二个参数)中去。而且可以保证第三个参数block执行时，group中所有任务已经全部完成。

**【扩展 10-13】有哪些场景是NSOperation比GCD更容易实现的？（或是NSOperation优于GCD的几点）**

**NSOperation和GCD的区别：**

* (1)从**底层实现**来看，GCD是基于C语言实现的系统服务，执行和操作简单高效；NSOperation是对GCD更高层次的抽象。GCG是iOS4.0推出的,主要针对多核CPU 做了优化；NSOperation 是 iOS2.0后推出的,iOS4.0之后重写了NSOperation。
* (2)从**依赖关系方面**来看，NSOperation可以设置两个NSOperation之间的依赖，方便的控制执行顺序；GCD无法直接设置这种依赖关系，不过GCD可以通过dispatch_barrier_async来实现这种效果；
* (3)**KVO(键值对观察)**，NSOperation可以容易监听判断Operation当前的状态(是否正在执行isExecuteing，是否取消isCancelled，是否已完成isFinished)，对此GCD无法通过KVO进行监听判断；
* (4)从设置**优先级方面**来看，NSOperation可以设置自身的优先级(但是优先级高的不一定先执行)；GCD只支持FIFO的队列，GCD只能设置队列的优先级，无法在执行的block设置优先级；
* (5)从**执行效率**方面来看，直接使用GCD效率会更高，NSOperation会多一点开销；
* (6)从**功能方面**来看，NSOperation可以方便地控制队列的停止/继续，也可以取消队列中所有的操作；而GCD不具备这些功能。

那么什么情况下使用NSOperation，什么情况下使用GCD呢？

**使用NSOperation的情况**：各个操作之间有依赖关系；操作需要取消暂停、并发管理；控制操作之间优先级；限制同时能执行的线程数量，让线程在某时刻停止/继续等。

**使用GCD的情况**：一般简单的多线程操作，都可以使用GCD，简单高效。

从编程原则来说，一般我们需要尽可能的使用高等级、封装完美的API，在必须时才使用底层的API。当需求简单，简洁的GCD或许是个更好的选择，而Operation queue 为我们提供了更多的选择。

**【扩展 10-14】如何用GCD同步若干个异步调用？（比如根据若干个url异步加载多张图片，然后在都下载完成后合成一张整图）**

使用Dispatch Group追加block到Global Group Queue,这些block如果全部执行完毕，就会执行Main Dispatch Queue中的结束处理的block。
    
```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, queue, ^{ //加载图片1 });
dispatch_group_async(group, queue, ^{ //加载图片2 });
dispatch_group_async(group, queue, ^{ //加载图片3});
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
// 合并图片
});
```

**【扩展 10-15】dispatch_barrier_async的作用是什么？**

在并行队列中，为了保持某些任务的顺序，需要等待一些任务完成后才能继续进行，使用 barrier 来等待之前任务完成，避免数据竞争等问题。 dispatch_barrier_async 函数会等待追加到Concurrent Dispatch Queue并行队列中的操作全部执行完之后，然后再执行 dispatch_barrier_async 函数追加的处理，等 dispatch_barrier_async 追加的处理执行结束之后，Concurrent Dispatch Queue才恢复之前的动作继续执行。

注意：使用 dispatch_barrier_async ，该函数只能搭配自定义并行队列 dispatch_queue_t 使用。不能使用： dispatch_get_global_queue ，否则 dispatch_barrier_async 的作用会和 dispatch_async 的作用一模一样。 ）

**【扩展 10-15】如果让你实现 GCD 的线程池，该如何实现？讲一下思路**

**线程池包含如下几个部分**:

* (1)线程池管理器（ThreadPoolManager）:用于创建并管理线程池，是一个单例；
* (2)工作线程（WorkThread）: 线程池中线程；
* (3)任务接口（Task）:每个任务必须实现的接口，以供工作线程调度任务的执行；
* (4)任务队列:用于存放没有处理的任务。提供一种缓冲机制；
* (5)corePoolSize核心池的大小:默认情况下，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；
* (6)maximumPoolSize线程池最大线程数:它表示在线程池中最多能创建多少个线程；
* (7)存活时间keepAliveTime:表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，这是如果一个线程空闲的时间达到keepAliveTime，则会终止直到线程池中的线程数不大于corePoolSize

**具体流程:**

* (1)当通过任务接口向线程池管理器中添加任务时，如果当前线程池管理器中的线程数目小于corePoolSize，则每来一个任务，就会通过线程池管理器创建一个线程去执行这个任务；
* (2)如果当前线程池中的线程数目大于等于corePoolSize，则每来一个任务，会尝试将其添加到任务缓存队列当中，若添加成功，则该任务会等待空闲线程将其取出去执行；若添加失败（一般来说是任务缓存队列已满），则会尝试创建新的线程去执行这个任务；
* (3)如果当前线程池中的线程数目达到maximumPoolSize，则会采取任务拒绝策略进行处理；
* (4)如果线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止，直至线程池中的线程数目不大于corePoolSize。

**【扩展 10-16】串行并行、同步异步的概念**

队列分为串行和并行，任务的执行分为同步和异步。这两两组合就出现了串行队列同步执行、串行队列异步执行、并行队列同步执行和并行队列异步执行四种。

**串行队列**：表示所有任务一个接一个的在当前线程中执行。各个任务按照一定顺序执行，完成任务A后才能进行执行任务B。串行是同步线程的实现方式

**并行队列**：表示所有任务可以同时在不同线程中执行。

**并发**：当有多个线程在操作时，如果系统只有一个CPU，那么它根本不可能真正同时进行一个以上的线程，它只能把CPU运行时间划分成若干个时间片，再将时间片分配给各个线程执行，在一个时间段的线程代码运行时，其它线程处于挂起状态。这种方式我们称之为并发（Concurrent）。

**并发**：当系统有一个以上CPU时，当一个CPU执行一个线程时，另一个CPU可以执行另一个线程，两个线程互不抢占CPU资源，可以同时进行，这种方式我们称之为并行（Parallel）。

**【并行和并发的区别和联系】**：

并行和并发其实是异步线程实现的两种形式。并行其实是真正的异步，多核CPU可以同时开启多条线程供多个任务同时执行，互不干扰。但是并发就不一样了，并发是一个伪异步，在单核CPU中只有一条线程，但是又需要执行多个任务，这个时候，只能在一条线程上不停的切换任务。比如任务A执行了20%，任务A停下来，线程让给任务B，任务B执行30%停下，载让任务A执行。由于CPU处理速度快，看起来好像是同时执行，其实不是，同一时间只会执行单个任务。并发不一定并行，但并行一定并发。

**主队列**:专门⽤来在主线程调度任务的队列，所以主队列的任务都要在主线程来执行，主队列会随着程序的启动一起创建，我们只需get即可。

**全局队列**:是系统为了方便程序员开发提供的，其⼯作表现与并发队列一致。

**同步(dispatch_sync)**：多任务情况下，一个任务A执行结束，才可以执行另一个任务B。只有一个线程。

**异步(dispatch_async)**：多任务情况下，一个任务A正在执行，同时可以执行另一个任务B。任务B不用等待任务A结束才执行。存在多线程。

**队列**：dispatch_queue_t,一种先进先出的数据结构，线程的创建和回收不需要程序员操作，由队列负责。

```
（1）串行队列:队列中的任务(线程)按照一定的顺序执行，不会同时执行（类似跑步）
dispatch_queue q1 = dispatch_queue_create("...",dispatch_queue_serial);
（2）并行队列:队列中的任务(线程)通常会并发执行（类似多人赛跑）
dispatch_queue_t q2 = dispatch_queue_create("...",dispatch_queue_concurrent);
（3）全局队列:由系统创建，直接调用即可。全局队列属于并行队列。
dispatch_queue_t q3 = dispatch_get_global_queue(dispatch_queue_priority_default,0);
（4）主队列:每个应用程序对应唯一的一个主队列，直接调用即可。主队列属于串行队列，在多线程开发中，一定要在主队列中更新UI。
dispatch_queue_t q4 = dispatch_get_main_queue();
```

**【队列和线程的区别】**：

队列是用来管理线程的，包括线程的创建和回收，相当于线程池。队列分为串行队列和并行队列。串行队列队列中的线程按照一定的顺序执行（不会同时执行）；并行队列队列中的线程会并发执行，没有顺序，可能会有一个疑问：队列不是先进先出吗？如果后面的任务执行完了，何时出队列的呢？这里需要强调的是，任务执行完毕后，并不一定出队列，只有前面的任务执行完了，才能出队列。

**【主队列和GCD创建的队列的区别】**：

主线程队列(主队列)比GCD创建的队列优先级高。所以在GCD中的串行队列开启同步任务里面没有嵌套任务是不会阻塞主线程的。只有一种可能导致死锁，就是串行队列里，嵌套开启任务，有可能会导致死锁。主线程队列中不能开启同步，因为主线程队列中开启同步任务会抢占主线程资源，造成死锁，进而阻塞主线程。主线程队列中只能开启异步任务，开启异步任务也不会开启新的线程，只是降低异步任务的优先级，让CPU空闲的时候才去调用。

**【全局队列跟并发队列的区别】**

全局队列无论是ARC还是MRC都不需要考虑释放，因为它是系统提供的，我们只需要get就 可以了。而并发队列在MRC下，并发队列创建出来后，需要手动释放dispatch_release()。


**【在主队列开启同步任务为什么会阻塞主线程？】**

在主线程开启同步任务，因为主队列是串行队列，线程按照一定的顺序执行，先执行完一个线程才执行下一个线程，而主队列始终就只有一个主线程，主线程是不会执行完毕的，除非关闭应用开发程序，因此在主线程开启一个同步任务，同步任务会想抢占执行的资源，而主线程任务一直在执行某些操作。两个任务始终互相等待，最终导致死锁，阻塞线程。

主队列添加同步任务会导致死锁，示例如下：

```
NSLog(@"任务1"）；
dispatch_sync(dispatch_get_main_queue(),^{
	NSLog(@"任务2");
});
NSLog(@"任务3"）；
//运行结果：只打印出“任务1”
```

**执行步骤分析**：首先执行任务1，然后遇到dispatch_sync同步线程，当前线程进入等待，等待同步线程中的任务2执行完再执行任务3，这个任务2是加入到mainQueue主队列中（此线程为同步线程），FIFO原则，主队列将任务加入到队尾，也就是加到任务3之后，那么问题就来了，任务3等待任务2执行完，而任务2加入到主队列的时候，任务2就会等待任务3执行完，这个就造成了死锁。

**【为什么在主线程开启异步任务不会阻塞线程？】**
  
在主队列中开启异步任务，虽然不会开启新的线程，但是他会把异步任务降低优先级，等CPU空闲时，就会在主线程上执行异步任务。

**【扩展10-17】进程与线程的联系和区别**

一个程序至少有一个进程,一个进程至少有一个线程。

**进程**：设备中正在执行中的程序被称为进程。进程是系统进行资源分配和调度的一个独立单元。负责程序运行的内存分配；各个进程之间互不干扰。

**线程**：进程要想执行任务就需要依赖线程，也就是说进程中的最小执行单位就是线程，并且一个进程中至少有一个线程（即主线程）。线程是CPU调度和分配的基本单元。

**联系**：线程是进程的最小组成单元。

**区别**：

* (1)调度方面：线程作为调度和分配的基本单位，进程作为拥有资源的基本单位。
* (2)并发性：不仅进程之间可以并发执行，同一个进程的多个线程之间也可并发执行。
* (3)拥有资源：进程是拥有资源的一个独立单位，线程不拥有系统资源，但可以访问隶属于进程的资源。
* (4)系统开销：在创建或撤消进程时，由于系统都要为之分配和回收资源，导致系统的开销明显⼤于创建或撤消线程时的开销。 

**【扩展10-18】死锁产生的原因**

所谓死锁，通常是指有两个线程T1和T2都卡住了，并且等待对方完成某些操作。T1不能完成是因为它在等待T2执行完成。而T2也不能完成，因为它在等待T1执行完成。也就是二者都处于“等待对方完成”的状态，于是就导致了死锁（DeadLock）。

**【扩展10-19】死锁的4个必要条件**

* (1)互斥条件。一个资源每次只能被一个进程使用，即在一段时间内某 资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。
* (2)请求与保持条件。进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。
* (3)不可剥夺条件。进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能由获得该资源的进程自己来释放（只能是主动释放)。
* (4)循环等待条件。若干进程间形成首尾相接循环等待资源的关系。

这四个条件是死锁的必要条件，只要系统发生死锁，这些条件必然成立，而只要上述条件之一不满足，就不会发生死锁。

**【扩展10-20】如何避免死锁的产生**

我们可以通过破坏死锁产生的4个必要条件来 预防死锁，由于资源互斥是资源使用的固有特性是无法改变的。所以可以从以下三个方面入手：

* 破坏“不可剥夺”条件：一个进程不能获得所需要的全部资源时便处于等待状态，等待期间他占有的资源将被隐式的释放重新加入到 系统的资源列表中，可以被其他的进程使用，而等待的进程只有重新获得自己原有的资源以及新申请的资源才可以重新启动，执行。
* 破坏”请求与保持条件“：第一种方法静态分配即每个进程在开始执行时就申请他所需要的全部资源。第二种是动态分配即每个进程在申请所需要的资源时他本身不占用系统资源。
* 破坏“循环等待”条件：采用资源有序分配其基本思想是将系统中的所有资源顺序编号，将紧缺的，稀少的采用较大的编号，在申请资源时必须按照编号的顺序进行，一个进程只有获得较小编号的进程才能申请较大编号的进程。





















