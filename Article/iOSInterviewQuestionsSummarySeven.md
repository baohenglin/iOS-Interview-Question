## 知识点18 第三方库 & 组件化

**【扩展 18-1】介绍自己用过哪些开源库？**

* Masonry：OC版屏幕布局适配的三方框架。
* SnapKit：Swift版屏幕布局适配的三方框架。
* AFNetWorking：一款轻量级的网络请求框架。
* MKNetworkKit：基于objective-C语言的网络请求库。
* Alamofire：基于Swift语言的网络请求库。
* Mantle：Json转Model的框架。
* YYModel：JSON转Model的框架。
* SDWebImage：一款功能强大的网络图片加载及缓存框架。



**【扩展 18-2】读过某个库的源码么？**

* AFNetworking
* SDWebImage
* YYKit

**【扩展 18-3】SDWebImage 下载了图片后为什么要解码？**

一般下载或者从磁盘获取的图片是PNG或者JPG(因为位图体积很大，所以磁盘缓存不会直接缓存位图数据，而是编码压缩后的PNG或JPG数据)，这是经过编码压缩后的图片数据，不是位图，要把它们渲染到屏幕前就需要进行解码转成位图数据，而这个解码操作比较耗时。也就是说，图片在远端存储一定都是编码后存储的，这样体积小，一个图像可以看做是一个图像文件，里面包含了文件头，文件体和文件尾，图像的数据就包含在文件体中，而我们的解码就是运用算法将文件体中的图像数据转化为位图数据，方便渲染和展示。

[SDWebImage探究](https://www.jianshu.com/p/d527ff0c4950)

**【扩展 18-4】项目有没有做过组件化？或者你是否调研过？**

组件化核心技术是一套路由方案实现完全解耦，这样就可以根据自己的业务功能模块化并行开发。主要还是根据自己业务划分不同的组件，组件粒度大小的掌握是难点。组件之间的解耦和依赖是难点。

路由方案可以自己实现也可以借鉴蘑菇街和casetwy的路由方案。casetwy是Category和Target_Action的方式实现组件之间的解耦和组件之间的通信的。CTMediator内部用runtime实现参数的传递，Target实现事件的分发。CTMediator+ACategory实现功能组件的解耦。

[iOS组件化](https://juejin.im/post/58b2aad6b123db0052cc9edd)

[蘑菇街App组件化之路](https://limboy.me/tech/2016/03/10/mgj-components.html)



**【扩展 18-5】如果让你实现 NSNotificationCenter，讲一下思路**

[NSNotificationCenter实现原理](https://www.jianshu.com/p/051a9a3af1a4)

**【扩展 18-6】APNS推送原理**

[APNS推送原理](https://www.jianshu.com/p/032bfc949917)

**【扩展 18-7】服务器能否知道APNS推送后有没有到达客户端的方法？**

APNS是苹果提供的远程推送的服务，APP开发此功能之后，⽤户允许推送之后，服务端可以向安装了此app的用户推送信息。但是APNS推送无法保证100%到达。如果服务器器向APNS服务器推送信息之后，服务器能够接收到APNS是否真的成功向客户端推送成功了某个信息。这样在一定程度上提高了APNS的成功概率。

**【扩展 18-8】iOS IAP内购审核可能失败的原因有哪些？**

* (1)app中在IAP内购中购买的商品，能够通过其他的渠道或者方式购买。⽐如，你在安卓充值100元人民币，那么如果商品一样能够使用在iOS设备上，苹果是不会允许你上线的。
* (2)在审核的时候不能以任何方式，通过活动或者兑换码的形式，能够获取到iAP内购中能够获取到的商品。
* (3)另外就是可能有人会在苹果审核之后隐藏IAP支付，此处提醒下，苹果会扫描你的 app代码中是否有⽀付宝，微信等关于支付的字段。使⽤开关加h5的方式可以通过审核，但是此处也有⻛险，⻛险就是一旦被发现，可能的结果就是苹果直接封掉账号。app无法使用。

**【扩展 18-9】UDID和UUID**

[UDID和UUID](https://www.cnblogs.com/LiLihongqiang/p/5909734.html)

## 知识点19 Swift

**【扩展 19-1】Swift 中 struct(结构体) 和 class(类) 的区别？**

**struct(结构体) 和 class(类) 的区别：**

* 类(class)是引用类型，结构体(struct)是值类型；
* 类存储在堆（heap）上，而结构体实存储在栈（stack）上。相比于栈上的操作，堆上的操作更加复杂耗时，所以苹果官方推荐使用结构体，这样可以提高 App 运行的效率。
* class可以继承，而结构体不能；
* class可以用deinit来释放资源，而结构体不能；
* 结构体构造函数, 会自动生成带参数的构造器。类不会对有初始化赋值的属性, 生成带参数的构造器。

[Swift 中 struct(结构体) 和 class(类) 的区别](https://www.jianshu.com/p/a9420d8bcf40)

**【扩展 19-2】Swift 是如何实现多态的？**

**【扩展 19-3】Swift 和 OC，各自的优缺点有哪些？**

**Swift的优点**：

* 语法更简洁；
* Swift 语言支持函数式编程、面向协议编程、面向对象编程，而OC只有引入ReactiveCocoa这个库才可支持函数式编程；
* Swift更加安全，它是类型安全的语言。
* Swift开源
* Swift可跨平台
* Swift代码更少，简洁的语法，可以省去大量冗余代码。
* Swift速度更快，运算性能更高。

**Swift的缺点**：

* 第三方库的支持不够多；
* 版本不够稳定；
* App体积变大；
* 上线方式改变。不能使用application Loader上传包文件，会提示你丢失了swift support files，应该使用xcode直接上传。


**Swift和OC的区别**

[Swift和OC的区别](https://www.cnblogs.com/wangyf-iOS/p/6568266.html)

* swift是静态类型语言，OC是动态类型语言;
* swift注重面向协议编程、函数式编程、面向对象编程，OC注重面向对象编程;
* swift没有地址/指针的概念。swift注重值类型，OC注重指针和引用；
* swift注重安全，OC注重灵活；


**【扩展 19-4】用 Alamofire 比直接使用 URLSession，优势是什么？**

**【扩展 19-5】Swift中mutating关键字的作用及使用场景**

在Swift中，包含三种类型(type): structure,enumeration,class。其中structure和enumeration是值类型(value type),class是引⽤用类型(reference type)。但是与Objective-C不同的是，structure和enumeration也可以拥有方法(method)， 其中方法可以为实例方法(instance method)，也可以为类方法(type method)，实例方法是和类型的一个实例绑定的。

使用 mutating 关键字修饰方法是为了能在方法中修改 struct 或是 enum 的变量。

场景1：在结构体的实例方法里面修改属性：

```
struct Persion {
var name = ""
mutating func modify(name:String) {
self.name = name }
}
```

场景2：在协议里面，如果继承了协议的结构体或枚举类型想要改变属性值，则必须使用mutating修饰。

```
protocol Persionprotocol {
var name : String {get}
mutating func modify(name:String)
}

struct Persion : Persionprotocol {
var name = ""
mutating func modify(name:String) {
self.name = name }
}
```

场景3：在枚举中直接修改self属性

```
enum Switch {
  case On, Off
  mutating func operatorTion() { 
      switch self {
      case .On:
        self = .Off 
      default:
        self = .On 
      }
  } 
}
var a = Switch.On a.operatorTion()
print(a)
```

**【扩展 19-6】Swift 下的如何使用 KVC?**

* 步骤1: 继承NSObject；

```
class Animal1 : NSObject {
  var name = "Animal1" 
}
```
* 步骤2：在⽅法前添加@objc，然后再执行如下代码：

```
Animal1().setValue("Dog", forKey: "name")
```

**【扩展 19-7】Swift有哪些模式匹配?**

* 通配符模式(Wildcard Pattern)：如果你在 Swift 编码中使⽤了 _ 通配符，就可以表示你使⽤了通配符模式。

```
for _ in 0...10 {
  print("hello") 
}
```

* 标识符模式(Identifier Pattern)：

```
let i = 1 // i 就是⼀一个标识符模式
```

* 值绑定模式(Value-Binding Pattern)：值绑定在 if 语句和 switch 语句中用的较多。⽐如 if let 和 case let, 还有可能是 case var。 let 和 var 分别对应绑定为常量和 变量。

```
if let v = str {
  // 使⽤用v这个常量量做什什么事 (use v to do something) 
  //print("hello")
}
```

* 元组模式(Tuple Pattern)
* 可选模式
* 枚举⽤用例例模式(Enumeration Case Pattern)
* 类型转换模式(Type-Casting Pattern)
* 表达式模式

**【扩展19-8】OC语言的优缺点**

[OC语言的优缺点](https://www.jianshu.com/p/64e1755318cb?utm_campaign)

## 知识点20 逆向安全

**【扩展20-1】怎样防止反编译？**

* (1)本地数据加密。对NSUserDefaults，sqlite存储文件数据加密，保护账号和关键信息。
* (2)URL编码加密。对程序中出现的URL进行编码加密，防止URL被静态分析。
* (3)网络传输数据加密。对客户端传输数据提供加密方案，有效防止通过网络接口的拦截获取数据。
* (4)方法体、方法名混淆加固。对应用程序的方法名和方法体进行混淆，保证源码被逆向后无法解析代码。
* (5)对应用程序逻辑结构进行打乱混排，保证源码可读性降到最低。

## 知识点21 通知

**【扩展 21-1】远程推送通知实现的具体步骤是什么？APNS实现的原理是什么？**

[iOS 推送通知及推送扩展](https://juejin.im/post/5bc9a6e45188254a075e305c)

**【扩展 21-2】本地通知**

iOS最多允许最近本地通知的数量是64个。


## 知识点22 开放性问题

**【扩展 22-1】哪一个项目技术点最能体现自己的技术实力？具体讲一下。**

**【扩展 22-2】你在项目中遇到的最大的问题是什么？你是怎么解决的？**

* 如何监测和定位并解决卡顿？
* 如何优化TableView列表？

[TableView优化具体措施](https://www.jianshu.com/p/dbcf3665fad8)

* 如何监控并优化App的启动速度？
* 如何实现无侵入埋点？
* 如何全面监控各种崩溃？
* 如何获取App中的全量日志？


**【扩展 22-3】你是如何学习 iOS 的？**

* 查阅[苹果官方开发文档](https://developer.apple.com/documentation/)。苹果的官方文档相当详尽，对于不熟悉的 API，阅读官方文档也是最直接有效地方式。
* 观看WWDC视频。由于 iOS 开发在快速发展，每年苹果都会给我们带来很多新的知识。而对于这些知识，第一手的资料就是 WWDC 的视频。
* 通过书籍学习。比如《Objective-C高级编程 iOS与OS X多线程和内存管理》、《编写高质量iOS与OS X代码的52个有效方法》、《图解HTTP》、《程序员的自我修养——链接、装载与库》；
* 通过做项目，多写代码，多思考。“在多写代码的同时，我们也要注意不要 “ 重复造轮子 “，尽量保证每次写的代码都能具有复用性。在代码结构因为业务需求需要变更时，及时重构，在不要留下技术债的同时，我们也要多思考如何设计应用架构，能够保证满足灵活多变的产品需求。”
* 通过阅读优秀技术博客。
* 看开源项目的代码。阅读优秀的开源项目代码，不但可以学习到 iOS 开发本身的基本知识，还能学习到设计模式等软件架构上的知识。
* 写博客。[我的Github技术博客](https://github.com/baohenglin)
* 多和同行交流。可以在一些技术社区交流，比如[stackoverflow](http://www.stackoverflow.com)

**【扩展 22-4】和产品经理、测试产生冲突时，你是怎么解决的？**

* 积极沟通，确定问题出在什么地方。
* 组成小组讨论解决方案。
* 沟通过程中尽量保持心平气和。

**【扩展 22-5】NSCache 和 NSDcitionary的区别？**

[NSCache](http://southpeak.github.io/2015/02/11/cocoa-foundation-nscache/)

NSCache与可变集合有几点不同：

* 1.NSCache类结合了各种自动删除策略，以确保不会占用过多的系统内存。如果其它应用需要内存时，系统自动执行这些策略。当调用这些策略时，会从缓存中删除一些对象，以最大限度减少内存的占用。
* 2.NSCache是线程安全的，我们可以在不同的线程中添加、删除和查询缓存中的对象，而不需要锁定缓存区域。
* 3.不像NSMutableDictionary对象，一个缓存对象不会拷贝key对象。























