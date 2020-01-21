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

