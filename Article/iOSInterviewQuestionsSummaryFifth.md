## 知识点14  修饰符

**【扩展 14-1】讲一下iOS属性修饰符atomic的实现机制(内部是怎么实现的)；为什么不能保证绝对的线程安全（最好可以结合场景来说）？**

atomic表示原子操作，系统会为setter方法加锁，具体适用@synchronized(self){//code},保证读写互斥，即读取的时候不能修改值，修改值的时候不能读取，以保证线程安全。

使用atomic并不能保证绝对的线程安全。因为atomic只能保证对属性的读写是原子性的，但是仍然可能出现线程错误。比如：当线程A进行写操作时，其他线程的读或者写操作会因为该操作而等待，当A线程的写操作结束后，B线程进行写操作，然后当A线程需要读操作时，却获取了在B线程中的值，这就破坏了线程安全。如果有线程C在A线程读操作之前release了该属性，那么将会导致程序崩溃。所以仅仅使用atomic并不能保证线程安全，只是保证了属性getter和setter方法的线程安全。

**【扩展 14-2】成员变量、实例变量和属性的区别和联系？**

* （1）成员变量：成员变量是定义在{}中的变量，如果成员变量的数据类型是一个类，则称该变量为实例变量(实例变量是针对类而言的)，所以实例变量是成员变量的一种特殊情况。成员变量不会生成set、get方法，所以成员变量无法被外界访问（只用于类内部），这个也就是所谓的类私有变量。

* （2）属性：使用@property声明的变量是属性，属性可以被外界访问。例如：@property (nonatomic, strong) UIButton *myButton;编译器会自动地生成一个实例变量"_myButton"，同时会生成myButton属性的getter/setter方法。在.m文件中可以直接使用实例变量_myButton，也可以通过属性self.myButton，这里的self.myButton其实是调用myButton属性的getter/setter方法。这与C++中的点方法的使用是有区别的，C++中的点方法可以直接访问成员变量。声明属性myButton后，如果.m文件中写了@synthesize myButton;那么自动生成的实例变量为myButton而不是_myButton。@synthesize的作用是生成与属性对应的实例变量，并让编译器自动生成setter和getter方法。

【注意】类与分类(Category)中添加的属性要区分开来，因为类别中只能添加方法，不能添加实例变量。即使在类别中添加了属性，也不会自动生成带下划线的实例变量，这里其实只是添加的getter与setter方法的声明，并没有getter和setter方法的实现。此外类扩展（匿名类别）是可以添加实例变量的。

【使用建议】:(1)如果只是单纯的private变量，最好声明在implementation里；（2）如果是类的public属性，就用@property写在.h文件里；（3）如果自己内部需要setter和getter来实现一些功能，就在.m文件里用property来声明。

**【扩展 14-3】被weak修饰的对象在被释放的时候会发生什么？是如何实现的？知道sideTable么？里面的结构可以画出来么？**

[参考链接](https://www.jianshu.com/p/10c0f49f4755)

[weak原理](http://cloverkim.com/ios_weak-principle.html)

**【扩展 14-4】实现 isEqual 和 hash 方法时要注意什么？**

**【扩展 14-5】property 的常用修饰词有哪些？weak 和 assign 的区别？weak 的实现原理是什么？**

**【扩展 14-6】如何令自己所写的对象具有拷贝功能？**

如果想让⾃己的类具备copy方法，并返回不可变类型，必须遵循NSCopying协议，并且实现 - (id)copyWithZone:(NSZone *)zone方法。

如果想让⾃己的类具备mutableCopy方法，并且返回可变类型，必须遵守 NSMutableCopying协议，并实现 - (id)mutableCopyWithZone:(nullable NSZone *)zone方法。

注意:再此说的copy对应不可变类型和mutableCopy对应可变类型⽅方法，都是遵从系统规则⽽已。如果你想实现⾃己的规则，也是可以的。

**【扩展 14-7】谈谈对weak属性的理解**

释放时，会调用clearDeallocating函数。clearDeallocating函数首先根据对象地址获取所有weak指针地址的数组，然后遍历这个数组并把其中的数据设置为nil，最后把这个entry(条目)从weak表中删除，最后清理对象的记录。

**追问问题1：weak修饰的属性，为什么对象释放后会自动置为nil**？

Runtime对注册的类，会进行布局。对于weak对象会放入一个hash表中。用weak指向的对象内存地址作为key，当此对象的引用计数为0时会dealloc，假如weak指向的对象内存地址是a，那么就会以a为键，在这个weak表中搜索，找到所有以a为键的weak对象，从而设置为nil。

**追问问题2：当weak引用指向的对象被释放时，又是如何去处理weak指针的呢？**

* (1)调用objc_release
* (2)因为对象的引用计数为0，所以执行dealloc；
* (3)在dealloc中，调用了_objc_rootDealloc函数；
* (4)在_objc_rootDealloc中，调用了object_dispose函数；
* (5)调用objc_destructInstance；
* (6)最后调用objc_clear_deallocating。

objc_clear_deallocating函数内部具体实现如下：

* ✅a.从weak表中获取废弃对象的地址为键值的记录；
* ✅b.将包含在记录中的所有附有 weak 修饰符变量的地址置为nil；
* ✅c.将weak表中该记录删除；
* ✅d.从引用计数表中删除以废弃对象的地址为键值的记录。

**【扩展14-8】@synthesize和@dynamic分别有什么作用？**

@property有两个对应的词，一个是@synthesize（合成实例变量），一个是@dynamic。如果@synthesize和@dynamic都没有写，那么默认的就是@synthesize = _var;

@synthesize作用是：如果属性没有手动实现setter和getter方法，编译器会自动实现setter和getter方法。一般地，在类的实现代码里可以通过 @synthesize语法来指定实例变量的名字，如@synthesize i = _i;

@dynamic 的作用是：告诉编译器属性的 setter 与 getter 方法由开发者自己实现，不自动生成，避免编译期间产生警告。

假如一个属性被声明为 @dynamic var，而且你没有提供 @setter方法和 @getter 方法，编译的时候没问题，但是当程序运行到 instance.var = someVar，由于缺 setter 方法会导致程序崩溃；或者当运行到 someVar = var 时，由于缺 getter 方法同样会导致崩溃。编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定。

**【扩展14-9】用@property声明的NSString（或NSArray、NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题？**

用 @property 声明 NSString、NSArray、NSDictionary 经常使用 copy 关键字，是因为他们有对应的可变类型:NSMutableString、NSMutableArray、 NSMutableDictionary，他们之间可能进行赋值操作，为确保对象中的字符串串值不会无意间遭人更改，应该在设置新属性值时拷⻉⼀份。

例如当属性类型为NSString时，由于传递给Setter方法的新值有可能指向一个NSMutableString类的实例，这个类是NSString的子类，此时若是不copy字符串，那么设置完属性值之后，字符串的值就可能在对象不知情的情况下遭人更改。为了防止这种错误，故使用copy修饰。

**【扩展 14-10】浅拷贝和深拷贝**

**浅拷贝**是指针拷贝。对一个对象进行浅拷贝，相当于对指向对象的指针进行复制，产生一个新的指向这个对象的指针，那么就是有两个指针指向同一个对象，这个对象销毁后两个指针都应该置空。

**深拷贝**是对一个对象进行拷贝，相当于对对象进行复制，产生一个新的对象，那么就有两个指针分别指向两个对象。当一个对象改变或者被销毁后拷贝出来的新的对象不受影响。实现深拷贝需要实现NSCopying协议，实现-(id)copyWithZone:(NSZone * )zone 方法。

浅拷贝与深拷贝的区别在于浅拷贝本质上没有产生新对象，深拷贝产生了新的对象。

[copy和MutableCopy](https://github.com/baohenglin/HLBlog/blob/master/Articles/iOS%E4%B9%8B%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E5%88%86%E6%9E%90.md)

**【扩展 14-11】static的作用**

* (1)用**static声明局部变量时**，改变变量的存储方式（生命期），使变量成为静态的局部变量，即编译时就为变量分配内存，直到程序退出才释放存储单元。这样，使得该局部变量有记忆功能，可以记忆上次的数据，不过由于仍是局部变量，因而只能在代码块内部使用（作用域不变）。与auto变量不同的是，static变量的内存只被分配一次。
* (2)**static声明全局变量时**，该变量可以被该模块中的所有函数访问，但是不能被模块外的其他函数访问。
* (3)**使用static用于函数定义时**，对函数的连接方式产生影响，使得函数只在本文件内部有效，对其他文件是不可见的。这样的函数又叫作静态函数。使用静态函数的好处是，不用担心与其他文件的同名函数产生干扰，另外也是对函数本身的一种保护机制。如果想要其他文件可以引用本地函数，则要在函数定义时使用关键字extern，表示该函数是外部函数，可供其他文件调用。
* (4)在类中的static成员变量属于整个类所拥有，对类的所有对象只有一份拷贝。
* (5)在类中的static成员函数属于整个类所拥有，这个函数不接收this指针，因而只能访问类的static成员变量。 

**【扩展 14-12】关键字const的作用**

```
//a是一个常整型数
const int a;
//a是一个常整型数
int const a;
//a是一个指向常整型数的指针（也就是，整型数是不可修改的，但指针可以）
const int *a;
//a是一个指向整型数的常指针（也就是说，指针指向的整数是可以修改的，但指针是不可修改的）
int * const a;
//a是一个指向常整型数的常指针（也就是说，指针指向的整型数和指针都是不可修改的）
int const * a const;
```

对指针来说，可以指定指针本身为const，也可以指定指针所指的数据为const，或者二者同时指定为const；在一个函数声明中，const可以修饰形参，表明它是一个输入参数，在函数内部不能改变其值；对于类的成员函数，若指定其为const类型，则表明其是一个常函数，不能修改类的成员变量。

使用关键字const能产生更紧凑的代码。合理地使用const关键字可以使编译器很自然地保护那些不希望被改变的参数，防止其被无意的代码修改，可以减少bug的出现。

**【扩展 14-13】关键字volatile的作用是什么？列举三个不同的例子**

一个定义为volatile的变量表明该变量可能会被意想不到地改变，优化器在用到该变量时必须每次都小心地重新读取这个变量的值，而不是使用保存在寄存器里的备份。

下面是volatile变量的三个例子：

* 并行设备的硬件寄存器（例如状态寄存器）
* 一个中断服务子程序中会访问到的非自动变量（Non-automatic variables）
* 多线程应用中被几个任务共享的变量。

**【扩展14-14】写一个setter方法用于完成@property(nonatomic,retain)NSString * name，写一个setter方法用于完成@property(nonatomic,copy)NSString * name**

```
- (void)setName:(NSString *)str {
	if(_name != str) {
		[_name release];
		_name = [str retain];
	}
}

- (void)setName:(NSString *)str {
	if(_name != str) {
		[_name release];
		_name = [str copy];
	}
}
```

**【扩展14-15】原子(atomic)和非原子(nonatomic)属性有什么区别？**

[atomic和nonatomic属性的区别](https://www.jianshu.com/p/63b15067351c)


## 知识点15  UI视图

**【扩展 15-1】哪些场景可以触发离屏渲染？（知道多少说多少）**

**触发离屏渲染的场景**：

* 设置圆角。maskToBounds 同时设置；
* shadows（阴影）；
* shouldRasterize（光栅化）；
* masks（遮罩）；
* group opacity（不透明）；
* 渐变；
* edge antialiasing（抗锯齿）。

**当前屏幕渲染**：On-Screen Rendering(当前屏幕渲染)，指的是GPU的渲染操作是在当前用于显示的屏幕缓冲区中进行的。

**离屏渲染**：Off—Screen Rendering(离屏渲染)指的是GPU在当前屏幕缓冲区外新开辟一个缓冲区进行渲染操作。

**为什么要避免离屏渲染？**

在触发离屏渲染的时候，会增加GPU的工作量，而增加GPU的工作量很可能会导致GPU和CPU的工作总耗时超出了16.7ms,即屏幕的FPS小于60，从而导致UI的掉帧或卡顿，所以要避免离屏渲染。


**【扩展 15-2】说说你对 OC 中 load 方法和 initialize 方法的异同。——主要说一下执行时间，各自用途，没实现子类的方法会不会调用父类的？**

**+load方法的特点：**

* +load方法会在runtime加载类、分类时调用其对应的+load方法。
* 每个类、分类的+load方法，在程序运行过程中只调用一次。

**+load方法的调用顺序：**

1.先调用类的+load方法
	*	按照编译的先后顺序调用（先编译，先调用）；
	*	调用子类的+load方法之前会先调用父类的+load。
2.再调用分类的+load方法
	*	按照编译先后顺序调用（先编译，先调用）
  
  
**+initialize方法的特点：**+initialize方法会在类第一次接收到消息的时候调用。

**+initialize方法的调用顺序**：先调用父类的+initialize，再调用子类的+initialize。也就是先初始化父类，再初始化子类，且每个类只会初始化1次。

**+initialize和+load的最大区别**是+initialize是通过objc_msgSend进行调用的，而+load方法是通过函数指针直接调用+load方法。正因为+initialize是通过objc_msgSend进行调用的，所以+initialize有以下特点：

*	如果子类没有实现+initialize，会调用父类的+initialize（所以父类的+initialize可能会被调用多次）；
*	如果分类实现了+initialize，就覆盖类本身的+initialize调用。

**【扩展 15-3】如果页面 A 跳转到 页面 B，A 的 viewDidDisappear 方法和 B 的 viewDidAppear 方法哪个先调用？**

A页面跳转到B页面有2个方法,push和present。

**push**：先执行A页面的viewWillDisappear和viewDidDisappear,然后执行B页面的viewWillAppear和viewDidAppear.
**present**：先执行A页面的viewWillDisappear,随后执行B页面的viewWillAppear和viewDidAppear,最后执行A页面的viewDidDisappear.

**⭐️⭐️⭐️⭐️⭐️【扩展 15-4】细致地讲一下事件传递流程**

**事件传递流程**如下：

事件的传递是自上到下的顺序，即 UIApplication->window->处理该触摸事件最合适的 view。

当点击屏幕时，压力转为电信号，iOS系统将产生UIEvent对象，记录事件产生的时间和类型，然后系统的消息循环(runloop)会将接收到的触摸事件加入到一个由UIApplication管理的事件消息队列中，接下来UIApplication会从消息队列中取出触摸事件并将其分发下去。首先传给UIWindow，UIWindow会使用hitTest:withEvent:方法在视图层次结构中查找到一个最合适的视图来处理此次触摸事件，找到这个视图之后他就会调用视图的touchesBegan:withEvent:方法来处理事件。这些touches方法默认是将事件沿着响应者链条向上传递，将事件交给上一个响应者进行处理。一般事件的传递是从父控件传递到子控件的。如果父控件接收不到该触摸事件，那么子控件就不可能接收到触摸事件。

```
- (UIView * )hitTest:(CGPoint)point withEvent:(UIEvent * )event;
```

**UIView不能接收触摸事件的三种情况**：

* (1)用户交互权限关闭：userInteractionEnabled = NO;
* (2)隐藏：hidden = YES;
* (3)透明：alpha = 0.0~0.01

**响应者链条**：由很多响应者对象（继承自UIResponder的对象）一起组合起来的链条称之为响应者链条。一般默认的做法是控件将事件沿着响应者链条向上传递，将事件交给上一个响应者进行处理。那么如何判断当前响应者的上一个响应者是谁呢？有以下两个规则：

* (1)判断当前view是否是控制器的View，如果是控制器的View，上一个响应者就是控制器。
* (2)如果不是控制器的View，那么上一个响应者就是父控件。

当有view能够处理触摸事件后，开始响应事件。系统会调用view的以下方法：

```
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;
```

可以多对象共同响应事件。只需要在以上方法中调用super方法。

UIApplication –> UIWindow –> 递归找到最合适处理触摸事件的控件 –> 控件调用 touches 方法 –> 判断是否实现 touches方法 –> 没有实现默认会将事件传递给上一个响应者 –> 找到上一个响应者 –> 找不到方法作废。

[iOS事件传递与响应机制](https://juejin.im/entry/58451fff61ff4b006c36eba6)

[iOS UI事件传递与响应者链](https://www.jianshu.com/p/1a4570895df5)


**【扩展 15-5】谈一下对三种布局方式 frame、Auto Layout 以及 UIStackView的理解**

[iOS9 UIStackView 简介](https://swift.gg/2016/03/31/ios9-uistackview-guide-swift/)


**【扩展 15-6】UIView和CALayer是什么关系？**

[UIView和CALayer的区别和联系](https://blog.csdn.net/liushuo19920327/article/details/77851062)

**【扩展 15-7】UIView 生命周期**

[UIView 生命周期](https://blog.csdn.net/Bolted_snail/article/details/98960564#UIView_95)

```
//构造方法,初始化时调用,不会调用init方法
- (instancetype)initWithFrame:(CGRect)frame;
//添加子控件时调用。添加视图调用addSubview方法会触发didAddSubview
- (void)didAddSubview:(UIView *)subview ;
//构造方法,内部会调用initWithFrame方法。不会调用initWithCoder和awakeFromNib方法;
- (instancetype)init;
//xib归档初始化视图后调用,如果xib中添加了子控件会在didAddSubview方法调用后调用。xib归档创建视图会触发initWithCoder和awakeFromNib方法,不再调用init和initWithFrame方法;
- (instancetype)initWithCoder:(NSCoder *)aDecoder;
//唤醒xib,可以布局子控件
- (void)awakeFromNib;
//父视图将要更改为指定的父视图,当前视图被添加到父视图时调用
- (void)willMoveToSuperview:(UIView *)newSuperview;
//父视图已更改
- (void)didMoveToSuperview;
//其窗口对象将要更改
- (void)willMoveToWindow:(UIWindow *)newWindow;
//窗口对象已经更改
- (void)didMoveToWindow;
//布局子控件
- (void)layoutSubviews;
//绘制视图
- (void)drawRect:(CGRect)rect;
//从父控件中移除
- (void)removeFromSuperview;
//销毁
- (void)dealloc;
//将要移除子控件
- (void)willRemoveSubview:(UIView *)subview;
```

**【扩展 15-8】UIViewController 的生命周期**

[UIViewController 生命周期](https://blog.csdn.net/Bolted_snail/article/details/98960564#UIView_95)

按照执行顺序依次是：

```
//类的初始化方法
+ (void)initialize;
//通过xib来初始化控制器
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil;
// 对象初始化方法
- (instancetype)init;
// 从归档初始化
- (instancetype)initWithCoder:(NSCoder *)coder;
//nib文件被加载时，会发送一个awakeFromNib的消息到nib文件中的每个对象。
-(void)awakeFromNib;
//视图控制器开始加载view属性时调用此方法。当访问UIViewController的view属性时，view如果此时是nil，那么VC会自动调用loadView方法来初始化一个UIView并赋值给view属性。此方法用在初始化关键view，需要注意的是，在view初始化之前，不能先调用view的getter方法，否则将导致死循环（除非先调用了[super loadView];）如果没有重载loadView方法，则UIViewController会从nib或StoryBoard中查找默认的loadView，默认的loadView会返回一个空白的UIView对象。
-(void)loadView;
//视图控制器的view被加载完成
- (void)viewDidLoad;
//视图控制器的view即将显示在window上。在view即将添加到视图层级中（显示给用户）且任意显示动画切换之前调用,此时self.view.superview为nil.这个方法中完成任何与视图显示相关的任务，例如改变视图方向、状态栏方向、视图显示样式等。实现该方法时确保调用[super viewWillAppear:]
-(void)viewWillAppear:(BOOL)animated;
//视图控制器的view开始更新AutoLayout约束。
updateViewConstraints：
//视图控制器的view 将要布局子视图。
-(void)viewWillLayoutSubviews;
//视图控制器的view 已经布局子视图。
-(void)viewDidLayoutSubviews;
//视图控制器的view已经显示在window上。实现该方法时确保调用[super viewDidAppear:]
-(void)viewDidAppear:(BOOL)animated;
//视图控制器的view将要从window上消失。
-(void)viewWillDisappear:(BOOL)animated;
//视图控制器的view已经从window上消失。
-(void)viewDidDisappear:(BOOL)animated;
//接收到内存警告时会被调用
- (void)didReceiveMemoryWarning;
//视图即将被释放时调用。已经被废弃
- (void)viewWillUnload;
//视图被释放时调用。已经被废弃
- (void)viewDidUnload;
//视图被销毁
-(void)dealloc;
```

【延伸】App的生命周期

[App的生命周期](https://www.jianshu.com/p/ecd4917ce407)


**【扩展 15-9】UITableView和UICollection的异同**

[UITableView和UICollection的异同](https://www.jianshu.com/p/66190e352faa)

**【扩展 15-10】UIView的setNeedsDisplay和setNeedsLayout方法的异同？**

**相同点**：setNeedsDisplay和setNeedsLayout这两个方法都是异步执行的。

**不同点**：

* (1)setNeedsDisplay会自动调用drawRect方法实现View的绘制。而setNeedsLayout则调用layoutSubView来实现view中subView的重新布局。(这样设计的目的是为了避免重复调用造成的资源浪费。)
* (2)setNeedsDisplay方法的作用是将接收器的整个边界矩形标记为需要重绘。可以使用setNeedsDisplay或setNeedsDisplayInRect：来通知系统某视图内容需要重绘。此方法(setNeedsDisplay)记录请求并立即返回。在下一个绘制周期之前，视图实际上不会重绘，此时所有无效视图都会更新。只有在视图的内容或外观发生更改时，才应使用此方法请求重绘视图。如果只是更改视图的几何图形，则通常不会重绘视图。而是根据视图的contentMode属性中的值调整其现有内容。通过避免重绘未更改的内容的需要，重新显示现有内容可以提高性能。
* (3)setNeedsLayout的作用是使接收器的当前布局无效并在下一个更新周期触发布局更新。如果要调整视图子视图的布局，请在应用程序的主线程上调用此方法。此方法记录请求并立即返回。由于此方法不强制立即更新，而是等待下一个更新周期，因此可以在更新任何视图之前使用它来使多个视图的布局无效。此行为允许您将所有布局更新合并到一个更新周期，这通常会提高性能。

【延伸1】- (void)layoutIfNeeded;方法的作用：如果布局更新处于待处理状态，则立即布局子视图。

苹果文档有关layoutIfNeeded方法的描述如下：

```
Use this method to force the view to update its layout immediately. When using Auto Layout, 
the layout engine updates the position of views as needed to satisfy changes in constraints. 
Using the view that receives the message as the root view, this method lays out the view 
subtree starting at the root. If no layout updates are pending, this method exits without 
modifying the layout or calling any layout-related callbacks.
```

使用此方法可强制视图立即更新其布局。 使用“自动布局”时，布局引擎会根据需要更新视图的位置，以满足约束的更改。 使用以根视图接收消息的视图，此方法从根开始布局视图子树。 如果没有待处理的布局更新，则此方法退出而不修改布局或调用任何与布局相关的回调。

**【扩展 15-11】layoutSubViews & drawRects**

layoutSubViews在以下情况下会被调用(视图位置变化时触发)：

* 1. init初始化不会触发layoutSubViews
* 2. addSubView会触发layoutSubViews
* 3. 设置view的Frame会触发layoutSubviews,当然前提是frame的值设置前后发生了变化。
* 4. 滚动UIScrollView会触发layoutSubviews。
* 5. 旋转UIScreen会触发父视图上的layoutSubviews事件。
* 6. 改变一个UIView大小的时候也会触发父View上的layoutSubviews事件。
* 7. 直接调用setLayoutSubviews方法会触发layoutSubViews。

drawRects方法是用来绘图的。drawRects方法在以下情况下会被调用：

* 1. 如果在UIView初始化时没有设置Rect大小，将直接导致drawRect不被自动调用。drawRect调用是在loadView、ViewDidLoad两个方法之后，ViewWillAppear和ViewDidAppear之间。
* 2. 该方法在调用sizeToFit后被调用，所以可以先调用sizeToFit计算出size，然后系统自动调用drawRect：方法。
* 3. 通过设置contentMode属性值为UIViewContentModeRedraw，那么将在每次设置或更改frame时自动调用drawRect。
* 4. 直接调用setNeedsDisplay或者setNeedsDisplayInRect:触发drawRect:，但是前提条件是rect不能为0。

**drawRect方法使用注意点**：

* 1.若使用UIView绘图，只能在drawRect:方法中获取相应的contextRef并绘图。在其他方法中获取到的ref不能用于画图。
* 2.drawRect:方法不能手动显示调用，必须通过调用setNeedsDisplay或者setNeedsDisplayInRect，让系统自动调用该方法。
* 3.若使用CALayer绘图，只能在drawInContext:中（类似于drawRect）绘制，或者在delegate中相应方法绘制。同样也是调用setNeedDisplay方法间接调用以上方法。
* 4.若要实时画图，不能使用gestureRecognizer，只能使用touchbegan等方法来调用setNeedsDisplay实时刷新屏幕。

**【扩展 15-12】创建视图控制器有哪几种方式？**

[iOS 加载视图控制器的三种方式](https://www.jianshu.com/p/636600daf1e2)

**【扩展 15-13】创建视图UIView有哪几种方式？**

[控制器View的六种创建方式](https://blog.csdn.net/imkata/article/details/78759977)

**【扩展 15-14】xib文件的构成分别是哪3个图标？都具有什么功能？**

* File's Owner：它表示从磁盘加载nib文件的对象。
* First Responder：表示用户当前正在与之交互的对象。
* View：显示用户界面；完成用户交互；是UIView类或其子类。

**【扩展 15-15】Cocoa Touch提供了哪几种Core Animation过渡类型？**

[几种动画过渡效果](https://www.cnblogs.com/huangzs/p/10617254.html)

过渡动画通过type设置不同的动画效果，CATransition有多种过渡效果，但其实Apple官方的SDK只提供了四种：

* fade：淡出效果，默认
* moveIn：覆盖原图
* push：推出
* reveal：底部显示出来

**【扩展 15-16】Quatrz 2D的绘图功能的三个核心概念是什么并简述其作用**

* 上下文：主要用于描述图形写入哪里
* 路径：是在图层上绘制的内容
* 状态：用于保存配置变换的值、填充和轮廓，alpha值等。

**【扩展 15-17】frame和bounds的区别**

frame指的是该view在父view坐标系统中的位置和大小。（参考点是父视图的坐标系统）。

bounds指的是该view在本身坐标系统中的位置和大小。（参考点是本身的坐标系统）。

[frame和bounds的区别](https://blog.csdn.net/mad1989/article/details/8711697)

**【扩展15-18】延时加载**

延时加载是指只在用到的时候才去初始化。这样做可以避免内存占用率过高。

**【扩展15-19】UIView的动画效果有哪些？**

```
[UIView animateWithDuration: delay: options: animations: completion:^(BOOL finished) {}];
//转场动画一般使用这个方法。该方法效果是插入一面视图，移除一面视图，期间可以使用一些转场动画效果。
[UIView transitionFromView: toView: duration: options: completion:^(BOOL finished) {}];
```

[UIView的动画效果](https://www.cnblogs.com/xiaobajiu/p/4084747.html)


## 知识点16  计算机网络及网络安全

**【扩展 16-1】App网络层有哪些优化策略？**

* (1)优化DNS解析和缓存

APP内置Server IP列表，该列表可以在App启动服务中下发更新。App启动后的首次网络服务会从Server IP列表中取一个IP地址进行TCP连接，同时DNS解析会并行进行，DNS成功后，会返回最适合用户网络的Server IP，那么这个Server IP会被加入到Server IP列表中被优先使用。

Server IP列表有权重机制的，DNS解析返回的IP很明显具有最高的权重，每次从Server IP列表中取IP会取权重最高的IP。列表中IP权重也是动态更新的，根据连接或者服务的成功失败来动态调整，这样即使DNS解析失败，用户在使用一段时间后也会选取到适合的Server IP。

* (2)网络质量检测（根据网络质量来改变策略）

根据用户是在2G/3G/4G/Wi-Fi的网络环境来设置不同的超时参数，以及网络服务的并发数量。比如在环境比较差的情况下:

✅将并发数设置为一个，改成串行；

✅动态设置超时时间；

✅throttle 进行节流。AFNetworking中的throttle方法

```
- (void)throttleBandwidthWithPacketSize:(NSUInteger)numberOfBytes 
```

✅管道化连接。如果服务器不支持管线化的话，那么响应就会乱序。所以服务器要支持。 AFNetworking中通过HTTPShouldUsePipelining属性来设置，默认为NO。

* (3)减少数据传输量：选择合适的数据格式进行传输，比如使用Protocol Buffer数等，使用WebP图片格式。
* (4)提供网络服务重发机制：当第一次网络请求失败的时候，自动尝试再次重发
* (5)使用HTTP缓存。

[iOS网络层详解和优化](http://www.cocoachina.com/articles/22608)

**【扩展 16-2】TCP为什么要三次握手，四次挥手？三次挥手不行吗？**

**TCP三次握手的原因**：

假如现在客户端想向服务端进行握手，它发送了第一个连接的请求报文，但是由于网络信号差或者服务器负载过多，这个请求没有立即到达服务端，而是在某个网络节点中长时间的滞留了，以至于滞留到客户端连接释放以后的某个时间点才到达服务端，那么这就是一个失效的报文，但是服务端接收到这个失效的请求报文后，就误认为客户端又发了一次连接请求，服务端就会想向客户端发出确认的报文，表示同意建立连接。
假如不采用三次握手，那么只要服务端发出确认，表示新的建立就连接了。但是现在客户端并没有发出建立连接的请求，其实这个请求是失效的请求，一切都是服务端在自相情愿，因此客户端是不会理睬服务端的确认信息，也不会向服务端发送确认的请求，但是服务器却认为新的连接已经建立起来了，并一直等待客户端发来数据，这样的情况下，服务端的很多资源就没白白浪费掉了。
采用三次握手的办法就是为了防止上述这种情况的发生，比如就在刚才的情况下，客户端不会向服务端发出确认的请求，服务端会因为收不到确认的报文，就知道客户端并没有要建立连接，那么服务端也就不会去建立连接，这就是三次握手的作用。

**TCP四次挥手的原因**：

TCP协议是一种面向连接的、可靠的、基于字节流的运输层通信协议。TCP是全双工模式，这就意味着，在客户端想要断开连接时，客户端向服务端发送FIN报文，只是表示客户端已经没有数据要发送了，但是这个时候客户端还是可以接收来自服务端的数据。

当服务端接收到FIN报文，并返回ACK报文，表示服务端已经知道了客户端要断开连接，客户端已经没有数据要发送了，但是这个时候服务端可能依然有数据要传输给客户端。当服务端的数据传输完之后，服务端会发送FIN报文给客户端，表示服务端也没有数据要传输了，服务端同意关闭连接，之后，客户端收到FIN报文，立即发送给客户端一个ACK报文，确定关闭连接。在之后，客户端和服务端彼此就愉快的断开了这次的TCP连接。

为什么服务端的ACK报文和FIN报文都是分开发送的，但是在三次握手的时候却是ACK报文和SYN报文是一起发送的，因为在三次握手的过程中，当服务端收到客户端的SYN连接请求报文后，可以直接发送SYN+ACK报文。其中ACK报文是用来应答的，SYN报文是用来同步的。但是在关闭连接时，当服务端接收到FIN报文时，很可能并不会立即关闭SOCKET，所以只能先回复一个ACK报文，告诉客户端，你发的FIN报文我收到了，只有等到服务端所有的数据都发送完了，才能发送FIN报文，因此ACK报文和FIN报文不能一起发送。所以断开连接的时候才需要四次挥手来完成。

三次挥手不行。因为当服务端接收到FIN报文，并返回ACK报文，仅表示服务端已经知道了客户端要断开连接，客户端已经没有数据要发送了，但是这个时候服务端可能依然有数据要传输给客户端。需要将这种特殊情况考虑进去。

**【扩展 16-3】对称加密和非对称加密的区别？分别有哪些算法来实现这两种加密？**

**对称加密**又称**公开密钥加密**，加密和解密都会用到同一个密钥，加解密速度快。由于密钥需要在网络中传输，如果密钥被攻击者获得，此时加密就失去了意义。所以安全性不高。常见的对称加密算法有DES、3DES、AES、Blowfish、IDEA、RC5、RC6。

**非对称加密**又称**共享密钥加密**，使用一对非对称的密钥，一把叫做私钥，另一把叫做公钥；公钥加密只能用私钥来解密，私钥加密只能用公钥来解密。加解密速度慢，但安全性更高。常见的公钥加密算法有：RSA、ElGamal、背包算法、Rabin（RSA的特例）、迪菲－赫尔曼密钥交换协议中的公钥加密算法、ECC(椭圆曲线加密算法）。

**【扩展 16-4】HTTPS的握手流程？为什么密钥的传递需要使用非对称加密？双向认证了解么？**

HTTPS=HTTP+SSL/TLS，在HTTP的基础上又加了一层安全加密处理。HTTPS握手流程可细分为8步：

* 1）客户端向服务器端发起HTTPS请求。
* 2）服务器端接受请求。此时服务器端会生成一对密钥，即公钥和私钥，用来进行非对称加密使用。服务器端保存私钥，公钥可以发送给任何人。
* 3）服务器端将自己的公钥和数字证书发送给客户端。
* 4）客户端收到服务器端返回的数字证书后，会对数字证书的合法性进行校验。如何数字证书合法，客户端会生成一个随机值，这个随机值就是用于进行对称加密的密钥，我们将该密钥称为客户端密钥（Client key）。然后用服务器端返回的公钥对客户端密钥进行非对称加密，这样客户端密钥就变成密文了。
* 5）客户端将加密之后的客户端密钥发送给服务器端
* 6）服务器接收到客户端发来的密文后，会用自己的私钥对其进行非对称解密，解密之后的明文就是客户端密钥，然后用客户端密钥对数据进行对称加密，这样数据就变成了密文。
* 7）然后服务器端将加密后的密文发送给客户端
* 8）客户端接收到服务器发过来的密文，用客户端密钥（Client key）对其进行对称解密，得倒服务器发过来的数据。

鉴于非对称性加密速度比对称加密慢，但安全性更高，所以密钥的传输使用非对称加密，数据内容的传输使用对称加密。双向认证除了客户端验证服务器端数字证书的合法性之外，增加了服务器端验证客户端数字证书的合法性，确保可信任的客户端才可以访问服务器。

**【扩展 16-5】HTTPS是如何实现验证身份和验证完整性的？**

**HTTPS校验双方身份的真实性**：

HTTPS通过数字证书来验证双方身份。数字证书主要包含的信息有证书颁发机构、证书颁发机构签名、证书绑定的服务器域名、证书版本、有效期、签名使用的加密算法（非对称算法，如RSA）、公钥等。客户端收到服务器的响应后，先向CA验证证书的合法性（根据证书的签名、绑定的域名等信息），如果校验不通过，就会中止连接，向用户提示证书不安全。

**HTTPS验证数据的完整性**：

网络传输过程中需要经过很多中间节点，虽然数据无法被解密，但可能被篡改，那如何校验数据的完整性呢？通过校验数字签名来验证。

服务器在发送报文之前，会先通过哈希算法(哈希算法能够将任意长度的字符串转化为固定长度的字符串，该过程不可逆，可用来作数据完整性校验)对报文提取定长摘要；然后再用私钥对报文摘要进行加密，作为数字签名；再将数字签名附加到报文末尾发送给客户端。

客户端接收到报文后，先用公钥对服务器下发的数字签名进行解密；然后用与服务端同样的Hash算法重新计算出报文的数字签名；比较解密后的签名与自己计算的签名是否一致，如果不一致，说明数据被篡改过。

同样，客户端发送数据时，通过公钥加密报文摘要，服务器用私钥解密，用同样的方法校验数据的完整性。

[HTTPS身份真实性校验与数据完整性校验](https://www.jianshu.com/p/fb6035dbaf8b)

**【扩展 16-6】使用Charles抓HTTPS的包，其中的原理和流程是什么？**


**Charles抓HTTPS包的原理**：

Charles作为一个“中间人代理”，也就是将自己设置成系统的网络访问代理服务器。当浏览器和服务器通信时，Charles接收服务器的证书，但动态生成一张证书发送给浏览器，也就是说Charles作为中间代理在浏览器和服务器之间通信，所以通信的数据可以被Charles拦截并解密。由于Charles更改了证书，浏览器校验不通过会给出安全警告，所以必须安装Charles的证书后才能进行正常访问。

HTTPS抓包的原理简单来说就是Charles作为“中间人代理”，拿到了 服务器证书公钥 和 HTTPS连接的对称密钥，前提是客户端选择信任并安装Charles的CA证书，否则客户端就会“报警”并中止连接。

**Charles抓HTTPS包的流程**：

![Charles抓取HTTPS包的流程示意图.png](https://upload-images.jianshu.io/upload_images/4164292-b94b70db5cfeed41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* (1)客户端向服务器发起HTTPS请求
* (2)Charles拦截客户端的请求，伪装成客户端向服务器进行请求
* (3)服务器向“客户端”（实际上是Charles）返回服务器的CA证书
* (4)Charles拦截服务器的响应，获取服务器证书公钥，然后自己制作一张证书，将服务器证书替换后发送给客户端。（这一步，Charles拿到了服务器证书的公钥）
* (5)客户端接收到“服务器”（实际上是Charles）的证书后，生成一个对称密钥，用Charles的公钥加密，发送给“服务器”（Charles）
* (6)Charles拦截客户端的响应，用自己的私钥解密对称密钥，然后用服务器证书公钥加密，发送给服务器。（这一步，Charles拿到了对称密钥）
* (7)服务器用自己的私钥解密对称密钥，向“客户端”（Charles）发送响应
* (8)Charles拦截服务器的响应，替换成自己的证书后发送给客户端
* (9)至此，连接建立，Charles获取了服务器证书的公钥，也获取了客户端与服务器协商的对称密钥，之后就可以解密或者修改加密的报文了。

[浅谈Charles抓取HTTPS原理](https://www.jianshu.com/p/405f9d76f8c4)

**【扩展 16-7】如何使用Charles抓HTTPS的包？**

[使用Charles进行HTTPS抓包的具体操作](https://www.jianshu.com/p/7a88617ce80b)


**【扩展 16-8】什么是中间人攻击？如何避免？**

**中间人攻击(MITM攻击)的定义**：中间人攻击即所谓的Main-in-the-middle attack(MITM)，是指攻击者与通讯的两端分别创建独立的联系，并交换其所收到的数据，使通讯的两端认为他们正在通过一个私密的连接与对方直接对话，但事实上整个会话都被攻击者完全控制。

简单来说，攻击者在请求和响应传输途中，拦截并篡改内容。对于 HTTP 来说，由于设计的简单，不需要太多步骤就可以进行监听和修改报文；在这里主要是针对 HTTPS，HTTPS 使用了 SSL 加密协议，是一种非常安全的机制，目前并没有方法直接对这个协议进行攻击，一般都是在建立 SSL 连接时，利用中间人获取到 CA证书、非对称加密的公钥、对称加密的密钥；有了这些条件，就可以对请求和响应进行拦截和篡改。

**避免措施**：

* (1)对于SSL证书欺骗攻击，客户端不要轻易信任证书，只使用默认的系统校验；如果是使用WebView浏览网页，需要在UIWebView中加入较强的授权校验，禁止用户在校验失败的情况下继续访问。
* (2)对于SSL剥离攻击（SSLStrip），在WebView中打开网页同样需要注意，在非全网HTTPS的网站，建议对WebView中打开的URL做检查，检查应该使用 “https://” 的URL是否被篡改为 “http://” ；也建议服务端在配置HTTPS服务时，加上“HTTP Strict Transport Security”配置项。
* (3)针对SSL算法进行攻击，不要随意连入公共场合内的WiFi，也不要使用未知代理服务器；不要安装不可信或突然出现的描述文件，信任伪造的证书；App内部需对服务器证书进行单独的对比校验，确认证书不是伪造的；

[MITM攻击(中间人攻击)](https://www.jianshu.com/p/a825de42ccbc)


**【扩展 16-9】TCP和UDP的区别是什么，他们位于哪一层？**

**TCP协议**:是面向有连接的协议，也就是说在使用TCP协议传输数据之前一定要在发送方和接收方之间建立连接。建立连接后，通过数据超时重传、流量控制、拥塞控制等功能，TCP协议能够正确处理丢包问题，保证接收方能够收到数据，同时还能有效利用网络带宽。

**UDP协议**:是面向无连接的协议，它只会把数据传递给接收端，但不会关注接收端是否收到数据。

TCP和UDP的区别：

* (1)连接性：TCP协议是面向有连接的协议，要先确保发送发和接收方之间建立连接(三次握手)才能进行通信；UDP协议是面向非连接的协议，也就是说发送数据之前不需要建立连接；
* (2)传输可靠性：TCP传输可靠，UDP传输不可靠。TCP协议提供可靠性传输服务，可保证数据无差错、不丢失、不重复且按序发送，按序到达，提供超时重传来保证可靠性，但是UDP不保证按序到达，甚至不保证到达，只是尽最大努力交付，即便是按序发送的序列，也不保证按序送到。
* (3)应用场景：TCP适用于传输大量的数据，对可靠性要求较高的场景；UDP适用于对高速传输和实时性有较高要求的场景（视频、音频等多媒体通信（即时通信））；
* (4)速度方面：TCP传输速度慢；UDP传输速度快
* (5)控制机制：TCP有流量控制、拥塞控制和重发控制等机制，UDP没有，网络拥堵不会影响发送端的发送速率
* (6)开销方面：TCP开销大，TCP首部需20个字节（不算可选项）；UDP开销小，首部字段只需8个字节。
* (7)服务性质方面：TCP是一对一、点到点的连接，而UDP则可以支持一对一，一对多，多对一和多对多的交互通信。
* (8)传输内容方面：TCP面向的是字节流的服务，TCP把数据看成一连串无结构的字节流；UDP面向的是报文的服务，UDP没有拥塞控制，因此网络出现拥塞不会使源主机的发送速率降低。
* (9)信道方面：TCP是全双工的可靠信道；UDP是不可靠信道。 

TCP和UDP都位于OSI七层模型的第四层：**传输层**

OSI七层协议：应用层、表示层、会话层、传输层、网络层、数据链路层、物理层。

TCP/IP的体系结构：应用层、传输层(TCP、UDP)、网际层IP、网络接口层



**【扩展 16-10】路由器和交换机的工作原理大概是什么，他们分别用到什么协议，位于哪一层？**

[路由器和交换机的工作原理](https://www.jianshu.com/p/5553ada4a881)

**【扩展 16-11】描述TCP 协议三次握手，四次挥手的过程。**

**三次握手(3 way handshake):**

* 第一次握手：建立连接时首先客户端将标志位SYN置为1，并随机生成一个序列值seq = x，并将该数据包发送给服务端,客户端进入SYN_SENT状态，等待服务端确认；
* 第二次握手：服务端收到客户端发过来的数据包后由标志位SYN=1可知客户端请求建立连接，服务端将标志位SYN和ACK都置为1，ack = x + 1，随机产生一个值seq = y, 并将该数据包发送给客户端以确认连接请求，服务端进入SYN_RCVD状态；
* 第三次握手：客户端收到确认后，检查ack是否为x+1，ACK是否为1，如果符合的话，则将标志位ACK置为1，ack = y + 1, 并将该数据包发送给服务端,服务端检查ack是否为y+1，ACK是否为1，如果都正确则连接建立成功，客户端和服务端进入ESTABLISHED状态，完成三次握手，随后客户端与服务端之间开始进行数据传输。

**四次挥手：**

* 第一次挥手：客户端发出连接释放的报文，并且停止发送数据。将释放数据报文首部的FIN置为1，序列号seq 置为u（等于前面已经传送过来的数据的最后一个字节的序号加1），此时，客户端进入FIN-WAIT-1（终止等待1）状态。 TCP规定，FIN报文段即使不携带数据，也要消耗一个序号。
* 第二次挥手：服务端收到报文后，检查到首部的FIN为1，知道客户端请求释放连接，服务端发出确认报文，并将报文首部的ACK置为1，ack置为u + 1,并且带上自己的序列号v,此时服务端进入CLOSE-WAIT(关闭等待状态)。客户端收到服务端的确认报文后，检查ACK是否为1，ack是否为u+1,如果都正确，客户端进入FIN-WAIT-2(终止等待2)状态。等待服务端发送连接释放报文。
* 第三次挥手：服务端将最后的数据发送完毕后，就向客户端发送连接释放报文，FIN=1,ack = u+1,序列号为seq = w(因为在半关闭状态，服务器很可能又发送了一些数据，假定此时的序列号为seq=w)，此时服务端进入LASK-ACK(最后确认)状态，等待客户端确认。
* 第四次挥手：客户端接收服务器的报文后，检查FIN为1，知道服务端请求释放连接，发出确认报文，ACK = 1， ack = w + 1, seq = u + 1, 此时客户端进入TIME-WAIT(时间等待)状态。服务端只要收到客户端发出的确认报文，检查ACK是否为1，ack 是否为 w + 1, 如果都正确，立即进入CLOSE状态。

[三次握手和四次挥手详解](https://www.jianshu.com/p/5553ada4a881)

**【扩展 16-12】TCP 协议是如何进行流量控制，拥塞控制的？**

**TCP流量控制原理:**

流量控制以动态调整发送空间大小(滑动窗口)的形式来反映接收端接收消息的能力，反馈给发送端以调整发送速度，避免发送速度过快导致的丢包或者过慢降低整体性能。

这里采用滑动窗口机制，一是不用每次发送完成都需要等待收到确认消息才能继续发送，二是参考接收端的接收能力，限制发送数据段大小，避免丢失现象。

**TCP拥塞控制原理:**

连接建立的初期，如果窗口比较大，发送方可能会突然发送大量数据，导致网络瘫痪。因此，在通信一开始时，TCP 会通过慢启动算法得出窗口的大小，对发送数据量进行控制。

流量控制是由发送方和接收方共同控制的。接收方会把自己能够承受的最大窗口长度写在TCP 首部中，实际上在发送方这里，也存在流量控制，它叫拥塞窗口。TCP 协议中的窗口是指发送方窗口和接收方窗口的较小值。

**慢启动过程**如下：

![拥塞控制窗口大小调整.png](https://upload-images.jianshu.io/upload_images/4164292-f38ca5ade0b122ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 通信开始时，发送方的拥塞窗口大小为1。每收到一个 ACK确认后，拥塞窗口翻倍。
* 由于指数级增长非常快，很快地，就会出现确认包超时。(超时是因为数据量大导致网络拥塞)。
* 此时设置一个“慢启动阈值”，它的值是当前拥塞窗口大小的一半。同时将拥塞窗口大小设置为1，重新进入慢启动过程
* 由于现在“慢启动阈值”已经存在，当拥塞窗口大小达到阈值时，不再翻倍，而是线性增加。
* 随着窗口大小不断增加，可能收到三次重复确认应答，进入“快速重发”阶段。(快速重发: 当发送端连续收到三个重复的ack时，表示该数据段已经丢失，需要重发。当收到三个表示同一个数据段的ack时，不需要等待计时器超时，即重新发送数据段（当时这三个ack要在超时之前到达发送端），因为能够收到接收端的ack确认信息，所以数据段只是单纯的丢失，而不是因为网络拥塞导致。)
* 这时候，TCP 将“慢启动阈值”设置为当前拥塞窗口大小的一半，再将拥塞窗口大小设置成阈值大小（也有说加 3）。
* 拥塞窗口又会线性增加，直至下一次出现三次重复确认应答或超时。



**【扩展 16-13】为什么建立连接时是三次握手，两次行不行？如果第三次握手失败了怎么处理？**

**建立连接时是三次握手的原因：** 

由于网络是不可靠的，数据包是可能丢失的。假如现在客户端想向服务端进行握手，它发送了第一个连接的请求报文，但是由于网络信号差或者服务器负载过多，这个请求没有立即到达服务端，而是在某个网络节点中长时间的滞留了，以至于滞留到客户端连接释放以后的某个时间点才到达服务端，那么这就是一个失效的报文，但是服务端接收到这个失效的请求报文后，就误认为客户端又发了一次连接请求，服务端就会想向客户端发出确认的报文，表示同意建立连接。

假如不采用三次握手，那么只要服务端发出确认，表示新的建立就连接了。但是现在客户端并没有发出建立连接的请求，其实这个请求是失效的请求，一切都是服务端在自相情愿，因此客户端是不会理睬服务端的确认信息，也不会向服务端发送确认的请求，但是服务器却认为新的连接已经建立起来了，并一直等待客户端发来数据，这样的情况下，服务端的很多资源就没白白浪费掉了。

采用三次握手的办法就是为了防止上述这种情况的发生，比如就在刚才的情况下，客户端不会向服务端发出确认的请求，服务端会因为收不到确认的报文，就知道客户端并没有要建立连接，那么服务端也就不会去建立连接，这就是三次握手的作用。

**第三次握手失败后的处理措施：**

按照 TCP 协议处理丢包的一般方法，服务端会重新向客户端发送数据包，直至收到ACK 确认为止。但实际上这种做法有可能遭到SYN 泛洪攻击。所谓的泛洪攻击，是指发送方伪造多个 IP 地址，模拟三次握手的过程。当服务器返回 ACK 后，攻击方故意不确认，从而使得服务器不断重发 ACK。由于服务器长时间处于半连接状态，最后消耗过多的CPU和内存资源导致死机。

正确处理方法是**服务端发送RST 报文，进入 CLOSE状态**。这个 RST 数据包的 TCP 首部中，控制位中的 RST 位被设置为1。这表示连接信息全部被初始化，原有的 TCP通信不能继续进行。客户端如果还想重新建立TCP 连接，就必须重新开始第一次握手。

**【扩展 16-14】关闭连接时，第四次握手失败怎么处理？**(待优化)

实际上，在第三步中，客户端收到 FIN 包时，它会设置一个计时器，等待相当长的一段时间。如果客户端返回的 ACK 丢失，那么服务端还会重发 FIN 并重置计时器。假设在计时器失效前服务器重发的 FIN 包没有到达客户端，客户端就会进入 CLOSE 状态，从而导致服务端永远无法收到 ACK 确认，也就无法关闭连接。


**【扩展 16-15】你怎么理解分层和协议？**

**分层**

分层的优点：

* 独立性强。通过分层，每一层只接受下一层提供的特定服务，并且负责为上一层提供特定服务，上下层之间进行交互所遵循的约定叫“接口”，同一层之间的交互所遵循的约定叫做“协议”。每一层可以独立使用，及时系统中某些层次发生变化，也不会波及系统。
* 灵活性好。对于任何一层的改动，只要上下层接口不变，都不会造成系统的问题，有利于每一层功能的扩展和变动。
* 易于实现和维护。将大问题简化为小问题，大系统简化为小层次。比如将网络的通信过程划分为小一些、简单一些的部件,因此有助于各个部件的开发、设计和故障排除。
* 能促进标准化工作。通过分层，定义在模型的每一层实现什么功能,有利于鼓励产业的标准化，同时允许多个供应商进行开发。

分层的原则：

* 各个层之间有清晰的边界,便于理解;
* 每个层实现特定的功能;
* 层次的划分有利于国际标准协议的制定;
* 层的数目应该足够多，以避免各个层功能重复。

**协议**

协议实际上是一种通信双方共同遵守的规范。比如我需要把性别和年龄传递给另外一台主机，那么我可以定义一个"A 协议"，协议规定数据的前 4 个字节表示性别，后四个字节表示年龄。这样对方主机接收时就知道前 4 个字节是性别，而不会错把它当成年龄来处理。

协议的规范和共同遵守，有利于各个分层之间的交流和处理，也有利于促进协议的标准化过程。

**【扩展 16-16】HTTP 请求中的 GET 和 POST 的区别，Session 和 Cookie 的区别。**


**GET 和 POST 的区别：**

* (1)GET使用URL或Cookie传参，而POST将参数放在BODY中。
* (2)GET方式提交的数据有长度限制，则POST没有限制。其实HTTP协议对GET和POST都没有对长度的限制。HTTP协议明确地指出了，HTTP头和Body都没有长度的要求。之所以说GET提交的数据有长度限制是因为：一方面早期的浏览器会对URL长度做限制，另一方面多数服务器出于安全性、稳定性方面的考虑，会给URL长度加限制。但是这个限制是针对所有HTTP请求的，与GET、POST没有关系。
* (3)POST 请求仅比 GET 请求安全。因为POST请求的参数不会被保存在浏览器历史记录或 web 服务器日志中。此外，GET的数据在 URL 中对所有人都是可见的。POST的数据不会显示在 URL中。通过GET提交数据，用户名和密码将明文出现在URL上，因为登录页面有可能被浏览器缓存，其他人查看浏览器的历史纪录，那么别人就可以拿到你的账号和密码了，除此之外，使用GET提交数据还可能会造成Cross-site request forgery攻击。

以上三点只是二者在使用方面的区别。GET和POST的本质区别是**GET请求是幂等性的，POST请求不是幂等性的**。什么是幂等性？幂等性是指一次和多次请求某一个资源应该具有同样的副作用。简单来说意味着对同一URL的多个请求应该返回同样的结果。正因为二者有这样的区别，**所以不应该且不能用get请求做数据的增删改这些有副作用的操作。因为get请求是幂等的，在网络不好的隧道中会尝试重试。如果用get请求增数据，会有重复操作的风险**，而这种重复操作可能会导致副作用（浏览器和操作系统并不知道你会用get请求去做增操作）。


**Session 和 Cookie 的区别：**

* (1)cookie 保存在客户端，而 session 则保存在服务器中。
* (2)cookie不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗，考虑到安全应当使用session。 
* (3)session 可以设置超时时间，超过这个时间后就失效，以免长期占用服务端内存。
* (4)单个 cookie 的大小有限制（4 Kb），很多浏览器都限制一个站点最多保存20个cookie。
* (5)客户端每次都会把 cookie 发送到服务端，因此服务端可以知道 cookie，但是客户端不知道 session。当服务器接收到 cookie 后，会根据 cookie 中的 SessionID 来找到这个客户的 session。如果没有，则会生成一个新的 SessionID 发送给客户端。
* (6)将登陆信息等重要信息存放为SESSION，其他信息如果需要保存，可以放在COOKIE中。


**token和session的区别：**

* (1)Session 是一种HTTP存储机制，目的是为无状态的HTTP提供的持久机制。Session的状态是存储在服务器端，客户端只有session id；而Token的状态是存储在客户端的。
* (2)session需要严格保密，不应该共享给第三方。如果你的用户数据可能需要和第三方共享，或者允许第三方调用 API 接口，用token。

**cookie的定义**：cookie是保存在本地终端的数据。cookie由服务器生成，发送给客户端，客户端把cookie以键值对形式保存下来，下一次请求同一个URL时会把该cookie发送给服务器。cookie的组成有：名称(name)、值(value)、有效域(domain,可以访问该cookie的域名)、path(域的路径，一般设置为全局:"/")、失效时间、secure(指定后，cookie只有在使用SSL连接时才发送到服务器(https))。

**session的定义**：session的中文翻译是“会话”，当用户打开某个web应用时，便与web服务器产生一次session。服务器使用session把用户的信息临时保存在了服务器上，用户离开网站后session会被销毁。这种用户信息存储方式相对cookie来说更安全，可是session有一个缺陷：如果web服务器做了负载均衡，那么下一个操作请求到了另一台服务器的时候session会丢失。

**token的定义**：token的意思是“令牌”，是用户身份的验证方式，最简单的token组成:uid(用户唯一的身份标识)、time(当前时间的时间戳)、sign(签名，由token的前几位+盐以哈希算法压缩成一定长的十六进制字符串，可以防止恶意第三方拼接token请求服务器)。

**【扩展 16-17】谈谈你对 HTTP 1.1，2.0 和 HTTPS 的理解**

**HTTP**

HTTP（超文本传输协议，HyperText Transfer Protocol)是应用层的协议，目前在互联网中应用广泛。它被设计用于Web浏览器和Web服务器之间的通信，但它也可以用于其他目的。 HTTP遵循经典的客户端-服务端模型，客户端打开一个连接以发出请求，然后等待它收到服务器端响应。HTTP是无状态协议，意味着服务器不会在两个请求之间保留任何数据（状态）。

**HTTP/1.0 ——构建可扩展性**

HTTP/1.0规定浏览器与服务器只保持短暂的连接，浏览器的每次请求都需要与服务器建立一个TCP连接，服务器完成请求处理后立即断开TCP连接，服务器不跟踪每个客户也不记录过去的请求。

**HTTP/1.1 ——标准化的协议**

HTTP/1.0的多种不同的实现运用起来有些混乱，HTTP1.1是第一个标准化版本，重点关注的是校正HTTP1.0设计中的结构性缺陷：

* 连接可以重复使用，节省了多次打开它的时间，以显示嵌入到单个原始文档中的资源。
* 增加流水线操作，允许在第一个应答被完全发送之前发送第二个请求，以降低通信的延迟。
* 支持响应分块。
* 引入额外的缓存控制机制。
* 引入内容协商，包括语言，编码，或类型，并允许客户端和服务器约定以最适当的内容进行交换。
* 添加Host 头，能够使不同的域名配置在同一个IP地址的服务器。
* 安全性得到了提高。

在http/1.1中，client和server都是默认对方支持长链接的， 如果不希望使用长链接，则需要在header中指明connection:close。

**HTTP/2.0**

HTTP/2.0在HTTP/1.1有以下几点不同:

* HTTP/2.0是二进制协议而不是文本协议。
* HTTP/2.0是一个复用协议。并行的请求能在同一个链接中处理，移除了HTTP/1.x中顺序和阻塞的约束。
* 压缩了headers。因为headers在一系列请求中常常是相似的，其移除了重复和传输重复数据的成本。
* 其允许服务器在客户端缓存中填充数据，通过一个叫服务器推送的机制来提前请求。

**HTTPS**

我们知道HTTP 协议直接使用了TCP 协议进行数据传输。由于数据没有加密，都是直接明文传输，所以存在以下三个风险：

* 窃听风险：由于通信使用明文(不加密)，内容可能会被窃听；
* 篡改风险：无法证明报文的完整性，所以有可能已遭篡改；
* 冒充风险：不验证通信方的身份，因此有可能遭遇伪装。

HTTPS 协议旨在解决以上三个风险，因此它可以：

* 保证所有信息加密传输，无法被第三方窃取。
* 为信息添加校验机制，如果被第三方恶意破坏，可以检测出来。
* 配备身份证书，防止第三方伪装参与通信。

HTTPS(HTTP Secure安全套接字层超文本传输协议)，为了数据传输的安全，HTTPS在HTTP的基础上加入了SSL(Secure Socket Layer)协议，SSL依靠证书来验证服务器的身份，并为客户端和服务器之间的通信加密。即HTTPS=HTTP+SSL/TLS。

HTTPS握手流程可细分为8步：

* 1）客户端向服务器端发起HTTPS请求。
* 2）服务器端接受请求。此时服务器端会生成一对密钥，即公钥和私钥，用来进行非对称加密使用。服务器端保存私钥，公钥可以发送给任何人。
* 3）服务器端将自己的公钥和数字证书发送给客户端。
* 4）客户端收到服务器端返回的数字证书后，会对数字证书的合法性进行校验。如何数字证书合法，客户端会生成一个随机值，这个随机值就是用于进行对称加密的密钥，我们将该密钥称为客户端密钥（Client key）。然后用服务器端返回的公钥对客户端密钥进行非对称加密，这样客户端密钥就变成密文了。
* 5）客户端将加密之后的客户端密钥发送给服务器端
* 6）服务器接收到客户端发来的密文后，会用自己的私钥对其进行非对称解密，解密之后的明文就是客户端密钥，然后用客户端密钥对数据进行对称加密，这样数据就变成了密文。
* 7）然后服务器端将加密后的密文发送给客户端
* 8）客户端接收到服务器发过来的密文，用客户端密钥（Client key）对其进行对称解密，得倒服务器发过来的数据。

鉴于非对称性加密速度比对称加密慢，但安全性更高，所以密钥的传输使用非对称加密，数据内容的传输使用对称加密。双向认证除了客户端验证服务器端数字证书的合法性之外，增加了服务器端验证客户端数字证书的合法性，确保可信任的客户端才可以访问服务器。


