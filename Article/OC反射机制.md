## 知识点：OC反射机制

反射机制是指类名、方法名、属性名等可以和字符串相互转化（反射），而这些转化是发生在运行时的，所以我们可以用这个机制来动态地获取类、方法或属性，从而动态的创建类对象、调用方法、或给属性赋值、判断类型等。OC的反射机制类似于Java的反射机制 ，这种动态机制可以让OC语言变得更加灵活。OC的反射机制有三个用途：（1）通过类名的字符串来实例化对象;（2）将类名转化为字符串；（3）动态的调用方法；（4）SEL反射；（5）将方法变成字符串

（1）通过类名的字符串来实例化对象

```
Class className = NSClassFromString(@"HomeViewController");
NSLog(@"className = %@",className);
```

（2）将类名转化为字符串：

```
Class class = [HomeViewController class];
NSString *className = NSStringFromClass(class);
```

（3）动态地调用方法：

动态调用方法的应用场景：假设有一天公司产品要实现一个需求：根据后台推送过来的数据，进行动态页面跳转，跳转到页面后根据返回到数据执行对应的操作。遇到这样奇葩的需求，我们当然可以问产品都有哪些情况执行哪些方法，然后写一大堆if else判断或switch判断。但是这种方法实现起来太low了，而且不够灵活，假设后续版本需求变了，还要往其他已有页面中跳转，这样写也不利于代码维护与扩展。此时，反射机制就派上用场了，我们可以用反射机制动态的创建类并执行某方法。此外，我们也可以通过runtime来实现这个功能。

```
// 假设从服务器获取JSON串，通过这个JSON串获取需要创建的类的类名为FirstViewController，进而调用FirstViewController类的getDataList方法。
Class class = NSClassFromString(@"FirstViewController");
SEL selector = NSSelectorFromString(@"getDataList");
[[[class alloc] init] performSelector:selector];
```

（4）SEL反射(将字符串转化为方法)

```
Class className = NSClassFromString(@"HomeViewController");
HomeViewController *homeVC1 = [[class alloc]init];
//SEL反射
SEL selector = NSSelectorFromString(@"setName:");
[homeVC1 performSelector:selector withObject:@"zhang"];
```

（5）将方法变成字符串

```
NSString *selectorName1 = NSStringFromSelector(@selector(setName:));
```

**【拓展 1-1】获取Class类的三种方法**：

* 1)通过字符串来获得Class，此方法用到了反射机制。

```
Class className = NSClassFromString(@"HomeViewController");
NSLog(@"className = %@",className);
```
* 2)通过实例对象来获得Class

```
HomeViewController *mainVC = [[HomeViewController alloc]init];
NSLog(@"className = %@",[mainVC class]);
```
* 3)通过类来获得Class

```
NSLog(@"className=%@",[HomeViewController class]);
```

**【拓展 1-2】检查继承关系的方法**

```
HomeViewController *testVC = [[HomeViewController alloc]init];
NSLog(@"[testVC class]=%@",[testVC class]);
//2.判断对象是否为某个类的实例对象
NSLog(@"testVC是否为ViewController的实例对象:%d",[testVC isMemberOfClass:HomeViewController.class]);
//3.判断实例对象是否为某个类及其子类的实例
NSLog(@"[testVC isKindOfClass:[ViewController class]] = %d",[testVC isKindOfClass:[HomeViewController class]]);
//4.判断实例对象是否实现了指定的协议
NSLog(@"判断是否实现了指定协议=%d",[testVC conformsToProtocol:@protocol(UITableViewDelegate)]);
```
