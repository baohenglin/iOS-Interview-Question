
## 知识点23  其他问题

**【扩展 23-1】CPU和GPU**

CPU：中央处理器(Central Processing Unit)是一台计算机的运算核心和控制核心。CPU、内部存储器和输入/输出设备是电子计算机的三大核心部件。其功能主要是解释计算机指令以及处理计算机软件中的数据。

GPU：图形处理器(Graphic Processing Unit)。一个专门的图形核心处理器。GPU是显示卡的大脑，决定了该显卡的档次和大部分性能，同时也是2D显示卡和3D显示卡的区别依据。2D显示芯片在处理3D图像和特效时主要依赖CPU的处理能力，称为“软加速”。3D显示芯片是将三维图像和特效处理功能集中在显示芯片内，也即所谓的“硬件加速”功能。

**【扩展 23-2】点(pt)和像素(px)**

像素（pixels）是数码显示上的最小的计算单位。在同一个屏幕尺寸，更高的PPI（每英寸的像素数目），就能显示更多的像素，同时渲染的内容也会更清晰。点（points）是一个与分辨率无关的计算单位。根据屏幕的像素密度，一个点可以包含多个像素（例如，在标准Retina屏上1pt里有2 * 2个像素）。当为多种显示设备设计时，你应该以“点”为单位作为参考，但设计还是以像素为单位设计的。

**【扩展 23-3】CFSocket的使用步骤**

[CFSocket](https://www.jianshu.com/p/da02ffd2f718)

**【扩展 23-4】Core Foundation中提供了哪几种操作Socket的方法？**

答：CFNetwork 、 CFSocket 和 BSD Socket 

**【扩展 23-5】解析XML文件有哪几种方式？**

答：以 DOM 方式解析 XML 文件；以 SAX 方式解析 XML 文件；

**【扩展 23-6】什么是沙盒模型？哪些操作是属于私有api范畴？**

[沙盒模型](https://www.jianshu.com/p/b3043067b1cf)

**【扩展 23-7】iPhone OS主要提供了几种播放音频的方法？**

* SystemSound Services
* AVAudioPlayer类
* Audio Queue Services
* OpenAL

**【扩展 23-8】使用AVAudioPlayer类调用哪个框架？使用步骤是什么？**

调用AVFoundation.framework框架。

步骤：（1）配置AVAudioPlayer对象；（2）实现AVAudioPlayer类的委托方法；（3）监控AVAudioPlayer类的对象；（4）监控音量水平；（5）回放进度和拖拽播放。

**【扩展 23-8】有哪几种手势通知方法？请手写方法名**

```
-(void)touchesBegan:(NSSet)touchedwithEvent:(UIEvent)event;
-(void)touchesMoved:(NSSet)touched withEvent:(UIEvent)event;
-(void)touchesEnded:(NSSet)touchedwithEvent:(UIEvent)event;
-(void)touchesCanceled:(NSSet)touchedwithEvent:(UIEvent)event;
```

**【扩展 23-9】用预处理指令#define声明一个常数，用以表明1年终有多少秒（忽略闰年问题）**

```
#define SECONDS_PER_YEAR (60 * 60 * 24 * 365)UL
```

预处理指令不能以分号结尾；注意括号的使用；由于该表达式会使一个16位的整型数溢出（最高位表示正负，范围是-2的15次方~2的15次方），故需要使用无符号长整型(UL)。

**【扩展 23-10】写一个标准宏MIN，这个宏输入两个参数并返回较小的一个**

```
#define MIN(A,B)  ((A) <= (B) ? (A) : (B))
```

**【扩展 23-11】iOS中谓词(NSPredicate)的使用**

[谓词(NSPredicate)的使用](https://www.jianshu.com/p/88be28860cde)

**【扩展 23-4】什么是简便构造方法？**

构造方法就是初始化对象的方法。构造方法主要用于在对象创建时为对象的成员变量或属性赋值。简便构造方法一般由CocoaTouch框架提供，如NSNumber的下列简便构造方法：

```
+ (NSNumber *)numberWithChar:(char)value;
+ (NSNumber *)numberWithUnsignedChar:(unsigned char)value;
+ (NSNumber *)numberWithShort:(short)value;
+ (NSNumber *)numberWithUnsignedShort:(unsigned short)value;
+ (NSNumber *)numberWithInt:(int)value;
+ (NSNumber *)numberWithUnsignedInt:(unsigned int)value;
+ (NSNumber *)numberWithLong:(long)value;
+ (NSNumber *)numberWithUnsignedLong:(unsigned long)value;
+ (NSNumber *)numberWithLongLong:(long long)value;
+ (NSNumber *)numberWithUnsignedLongLong:(unsigned long long)value;
+ (NSNumber *)numberWithFloat:(float)value;
+ (NSNumber *)numberWithDouble:(double)value;
+ (NSNumber *)numberWithBool:(BOOL)value;
+ (NSNumber *)numberWithInteger:(NSInteger)value
```

Foundation下大部分类均有简便构造方法，我们可以通过简便构造方法，获得系统给我们创建好的对象，并且不需要手动释放。


**【扩展 23-5】如何使用Xcode设计通用应用(iPhone/iPad)？**

使用MVC模式设计应用，其中Model层完全脱离界面，即在Model层，其可以运行在任何设备上。在Controller层，根据iPhone与iPad（独有UISplitViewController）的不同特点选择不同的ViewController对象。在View层，可根据产品需求来设计，其中以xib文件设计时，将其设置为universal。

**【扩展 23-6】Objective-C有多继承吗？如果没有的话用什么代替？**

Objective-C没有多继承。可以通过Protocol委托代理来实现多继承。

**【扩展 23-7】Objective-C和C/C++如何混用？**

[C和OC混用](https://blog.csdn.net/yuanyuan1314521/article/details/51455166)

**【扩展 23-8】iOS的核心框架和核心机制有哪些？**

**iOS核心框架**：CoreAnimation、CoreGraphics、CoreLocation、AVFoundation、Foundation

**iOS核心机制**：

* UITableView重用机制
* 内存管理；自动释放池、ARC
* RunLoop
* Runtime
* Block的定义、特性、内存区域、如何实现
* Responder Chain
* NSOperation
* GCD

**【扩展 23-9】#import 和 #include 有什么区别？#import<> 和 #import"" 有什么区别？@class有什么作用？**

**#import 和 #include 的区别**：#import是Objective-C导入头文件的关键字，#include是C/C++导入头文件的关键字。使用#import的话，头文件只会自动导入一次，不会重复导入，相当于#include和#pragma once。

**#import<> 和 #import"" 的区别**：#import<>用来包含系统的头文件，#import""用来包含开发者自定义头文件。

**@class的作用**：告知编译器某个类的声明，当执行时，才去查看类的实现文件，可以解决头文件的相互包含的问题。

**【扩展23-10】NSInteger和int的区别**

NSIteger也是基本数据类型，并不是NSNumber的子类，也不是NSObject的子类。NSInteger是基本数据类型int或者long的别名（NSInteger的定义是 typedef long NSInteger），NSInteger与int的区别在于，NSInteger会根据系统是32位还是64位来决定本身是int还是long。

**【扩展23-11】id声明的对象有什么特性？**

id声明的对象具有运行时的特性，即可以指向任意类型的Objective-C的对象。


