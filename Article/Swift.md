## 知识点1 Swift

**【扩展 1-1】Swift 中 struct(结构体) 和 class(类) 的区别？**

**struct(结构体) 和 class(类) 的区别：**

* 类(class)是引用类型，结构体(struct)是值类型；
* 类存储在堆（heap）上，而结构体实存储在栈（stack）上。相比于栈上的操作，堆上的操作更加复杂耗时，所以苹果官方推荐使用结构体，这样可以提高 App 运行的效率。
* class可以继承，而结构体不能；
* class可以用deinit来释放资源，而结构体不能；
* 结构体构造函数, 会自动生成带参数的构造器。类不会对有初始化赋值的属性, 生成带参数的构造器。

[Swift 中 struct(结构体) 和 class(类) 的区别](https://www.jianshu.com/p/a9420d8bcf40)

**【扩展 1-2】Swift 是如何实现多态的？**

**【扩展 1-3】Swift 和 OC，各自的优缺点有哪些？**

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


**【扩展 1-4】用 Alamofire 比直接使用 URLSession，优势是什么？**

**【扩展 1-5】Swift中mutating关键字的作用及使用场景**

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

**【扩展 1-6】Swift 下的如何使用 KVC?**

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

**【扩展 1-7】Swift有哪些模式匹配?**

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

**【扩展1-8】OC语言的优缺点**

[OC语言的优缺点](https://www.jianshu.com/p/64e1755318cb?utm_campaign)

**【1-9】Swift 和 Objective-C 的联系？**

* Swift 与 Objective-C 共用同一套运行时环境。Swift 类型可以桥接到 Objective-C 的类型。反之亦然。
* 同一个工程，Swift 和 Objective-C 可以混编；
* Objective-C 中的很多概念，比如引用计数、ARC、属性、协议、接口、初始化、扩展类、匿名函数等在 Swift 中继续有效。也有一些概念 OC 不支持，Swift 支持。比如泛型。

**【1-10】Swift 比 Objective-C 有什么优势？**

* Swift 代码更简洁，更易读。在 Swift 中不再需要行尾的分号以及 if-else 语句中围绕条件表达式的括号。Swift 中的方法和函数的调用使用行业标准的在一对小括号内使用逗号分隔的参数列表。这样做的好处就是使语言更简洁优雅，更富有表现力。
* Swift 更易于维护：Swift 把 Objective-C 头文件(.h)和实现文件(.m)合并成了一个代码文件（.swift），Xcode 编译器可以自动计算并执行增量构建。
* Swift 更加安全：Swift 中的可选类型——一种针对返回或不返回值的编译时的安全机制。
* Swift 速度更快：Swift 已经接近 C++ 的速度，运行速度是 OC 的 1.4 倍。

**【1-11】Swift 支持面向过程编程吗？**

Swift 采用了 Objective-C 的命名参数以及动态对象模型，可以无缝对接到现有的 Cocoa 框架，并且可以兼容 Objective-C 代码，支持面向过程编程和面向对象编程。

**【1-12】举例说明 Swift 里面有哪些是 Objective-C 中没有的？**

* Swift 中引入了 Objective-C 中没有的一些高级数据类型，例如 tuples(元组)，可以使你创建和传递一组数值。
* Swift 还引入了可选项类型（Optionals），用于处理变量值不存在的情况。Optionals 类似于 Objective-C 中指向 nil 的指针，但是适用于所有的数据类型，而非仅仅局限于类。Optionals 相比 Objective-C 中的 nil 指针更加安全和简明。

**【1-13】Swift 是一门安全语言吗？**

Swift 是一门类型安全的语言，Optionals（可选项）就是代表。Swift 能帮助你在类型安全的环境下工作，如果你的代码中需要使用 String 类型，Swift 的安全机制能阻止开发者错误地将 Int 值传递过来，这可以使你在开发阶段就能及时发现并修改问题。

**【1-14】为什么要在变量类型后面加个问号？**

用来标记这个变量的值是可选的，一般用“!”和“?”定义。

**可选变量的区别**：

用“!”定义的可选变量必须保证转换能够成功，否则报错，但定义的变量可以直接使用，不会封装在 option 里；而用“？”定义的可选变量即使转换不成功本身也不会出错，变量值为 nil，如果转换成功，要使用该变量进行计算时变量名后需要加“!”。

**【1-15】Swift 中的泛型使什么？它们又解决了什么问题？**

泛型是用来使代码能安全工作的。在 Swift 中，泛型可以在函数数据类型和普通数据类型中使用，例如类、结构体或枚举。

泛型解决了**代码复用**的问题。有一种常见的情况，你有一个方法，需要一个类型的参数，你为了适应零一种类型的参数还得重新再写一遍这个方法：

```
func areInEqual(x: Int, _ y: Int) -> Bool {
  return x == y
}
func areStringEqual(x: String, _ y: String) -> Bool {
  return x == y
}
areStringEqual("ray", "ray") //true
areInEqual(1, 1) //true
```

一个 Objective-C 开发者可能会**采用 NSObject** 来解决问题：

```
func areTheyEuaul(x: NSObject, _ y: NSObject) -> Bool {
  return x == y
}
```

这段代码能达到目的，但是编译的时候并不安全。因为它允许一个字符串和一个整型数据进行比较。程序可能不会崩溃，但是允许一个字符串和一个整型数据进行比较可能不会得到预期的结果。

使用**泛型**来解决：通过泛型将上面两个方法合并为一个，同时也保证了数据类型的安全。

```
func areTheyEqual<T: Equatable>(x: T, _ y: T) -> Bool {
  return x == y
}
areStringEqual("ray", "ray") //true
areInEqual(1, 1) //true
```


