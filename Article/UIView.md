## 知识点  UI视图

**【扩展 1-1】哪些场景可以触发离屏渲染？（知道多少说多少）**

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


**【扩展 1-2】说说你对 OC 中 load 方法和 initialize 方法的异同。——主要说一下执行时间，各自用途，没实现子类的方法会不会调用父类的？（重点）**

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

**【扩展 1-3】如果页面 A 跳转到 页面 B，A 的 viewDidDisappear 方法和 B 的 viewDidAppear 方法哪个先调用？**

A页面跳转到B页面有2个方法,push和present。

**push**：先执行A页面的viewWillDisappear和viewDidDisappear,然后执行B页面的viewWillAppear和viewDidAppear.
**present**：先执行A页面的viewWillDisappear,随后执行B页面的viewWillAppear和viewDidAppear,最后执行A页面的viewDidDisappear.

**⭐️⭐️⭐️⭐️⭐️【扩展 1-4】细致地讲一下事件传递流程**

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


**【扩展 1-5】谈一下对三种布局方式 frame、Auto Layout 以及 UIStackView的理解**

[iOS9 UIStackView 简介](https://swift.gg/2016/03/31/ios9-uistackview-guide-swift/)


**【扩展 1-6】UIView 和 CALayer 是什么关系？**

[UIView和CALayer的区别和联系](https://blog.csdn.net/liushuo19920327/article/details/77851062)

**【扩展 1-7】UIView 生命周期**

[UIView 生命周期](https://blog.csdn.net/Bolted_snail/article/details/98960564#UIView_95)

```
//创建对象，分配空间
+ (instancetype)alloc;
//构造方法,初始化对象时调用,不会调用init方法
- (instancetype)initWithFrame:(CGRect)frame;
//添加子控件时调用。添加视图调用addSubview方法会触发didAddSubview
- (void)didAddSubview:(UIView *)subview ;
//构造方法,内部会调用initWithFrame方法。不会调用initWithCoder和awakeFromNib方法;
- (instancetype)init;
//xib归档初始化视图后调用,如果xib中添加了子控件会在didAddSubview方法调用后调用。xib归档创建视图会触发initWithCoder和awakeFromNib方法,不再调用init和initWithFrame方法;
- (instancetype)initWithCoder:(NSCoder *)aDecoder;
//唤醒 xib,可以布局子控件
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
//视图被销毁时调用。此处需要对你在 init 和 viewDidLoad 中创建的对象进行释放。
- (void)dealloc;
//将要移除子控件
- (void)willRemoveSubview:(UIView *)subview;
```

**【扩展 1-8】UIViewController 的生命周期（重点）**

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
//视图控制器的 view 已经显示在 window 上时调用。实现该方法时确保调用[super viewDidAppear:]
-(void)viewDidAppear:(BOOL)animated;
//视图控制器的view将要从window上消失。
-(void)viewWillDisappear:(BOOL)animated;
//视图控制器的view已经从 window 上消失时调用。
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


**【扩展 1-9】UITableView和UICollection的异同**

[UITableView和UICollection的异同](https://www.jianshu.com/p/66190e352faa)

**【扩展 1-10】UIView的setNeedsDisplay和setNeedsLayout方法的异同？**

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

**【扩展 1-11】layoutSubViews & drawRects**

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

**【扩展 1-12】创建视图控制器有哪几种方式？**

[iOS 加载视图控制器的三种方式](https://www.jianshu.com/p/636600daf1e2)

**【扩展 1-13】创建视图UIView有哪几种方式？**

[控制器View的六种创建方式](https://blog.csdn.net/imkata/article/details/78759977)

**【扩展 1-14】xib文件的构成分别是哪3个图标？都具有什么功能？**

* File's Owner：它表示从磁盘加载nib文件的对象。
* First Responder：表示用户当前正在与之交互的对象。
* View：显示用户界面；完成用户交互；是UIView类或其子类。

**【扩展 1-15】Cocoa Touch提供了哪几种Core Animation过渡类型？**

[几种动画过渡效果](https://www.cnblogs.com/huangzs/p/10617254.html)

过渡动画通过type设置不同的动画效果，CATransition有多种过渡效果，但其实Apple官方的SDK只提供了四种：

* fade：淡出效果，默认
* moveIn：覆盖原图
* push：推出
* reveal：底部显示出来

**【扩展 1-16】Quatrz 2D的绘图功能的三个核心概念是什么并简述其作用**

* 上下文：主要用于描述图形写入哪里
* 路径：是在图层上绘制的内容
* 状态：用于保存配置变换的值、填充和轮廓，alpha值等。

**【扩展 1-17】frame和bounds的区别**

frame指的是该view在父view坐标系统中的位置和大小。（参考点是父视图的坐标系统）。

bounds指的是该view在本身坐标系统中的位置和大小。（参考点是本身的坐标系统）。

[frame和bounds的区别](https://blog.csdn.net/mad1989/article/details/8711697)

**【扩展1-18】延时加载**

延时加载是指只在用到的时候才去初始化。这样做可以避免内存占用率过高。

**【扩展1-19】UIView的动画效果有哪些？**

```
[UIView animateWithDuration: delay: options: animations: completion:^(BOOL finished) {}];
//转场动画一般使用这个方法。该方法效果是插入一面视图，移除一面视图，期间可以使用一些转场动画效果。
[UIView transitionFromView: toView: duration: options: completion:^(BOOL finished) {}];
```

[UIView的动画效果](https://www.cnblogs.com/xiaobajiu/p/4084747.html)

**【扩展1-20】Auto Layout是如何进行自动布局的？(Auto Layout的生命周期)**

Auto Layout拥有一套运行时的 Layout Engine 引擎，由它来统一管理布局的创建、更新和销毁。App启动后，主线程的 RunLoop 会一直处于监听状态。每个视图在得到自己的布局之前，Layout Engine会将视图、约束、优先级、固定大小通过计算转换成最终的大小和位置。当约束发生变化后会触发 Deffered Layout Pass(延迟布局传递)，在里面做容错处理（约束丢失等情况）并把view标识为dirty状态，然后 RunLoop再次进入监听阶段。当下一次刷新屏幕动作来临（或者是调用 layoutIfNeeded）时，Layout Engine会从上到下调用 LayoutSubviews()，通过 Cassowary算法计算各个子视图的位置，算出来之后将子视图的frame从 Layout Engine 拷贝出来，接下来的绘制、渲染过程就跟手写frame是一样的了。所以，使用 Auto Layout和手写布局的区别，就是多了布局上的这个计算过程。

**【扩展1-21】Auto Layout的性能问题**

Auto Layout在iOS 12得到了优化。优化后的性能已经基本和手写布局一样可以达到性能随着视图嵌套的数量呈现线性增长了。而在此之前的 Auto Layout，视图嵌套的数量对性能的影响是呈指数级增长的。

实际上，iOS 12 之前，很多约束变化时都会重新创建一个计算引擎 NSISEnginer 将约束关系重新添加进来，然后重新计算。这样，当涉及的约束关系变多时，就会导致新的计算引擎需要重新计算，从而计算量呈指数级增加。总的来说，iOS12的 Auto Layout 更多地利用了 Cassowary 算法的界面更新策略，使其真正完成了高效的界面线性策略计算。

iOS12使得 Auto Layout具有了和手写布局几乎相同的高性能后，我们就可以放心的使用 Auto Layout了。使用 Auto Layout一定要注意多使用 Compression Resistance Priority 和 Hugging Priority，利用优先级的设置，让布局更加灵活，代码更少，更易于维护。

**【1-22】什么是 key window？**

如果一个窗口当前能接收键盘和非触摸事件（触摸事件会被传递到触摸发生的那个窗口），那么这个窗口就是主窗口（key window）。同一时刻只能有一个主窗口。在 iOS 开发中，我们可以通过设置 UIWindowLevel 的数值来设置最前端的窗口为哪个，Level 数值越高的窗口越靠前，如果两个窗口的 Level 等级相同，则我们可以通过 makeKeyAndVisible 来显示 KeyWindow。

**【1-23】项目中，你是怎么封装 View 的？**

[自定义 View 封装](https://www.jianshu.com/p/91ad5a343622)

思路：如果一个 view 内部的子控件比较多，一般会考虑自定义一个 view，把它内部的子控件创建封装起来。外界可以传入相应的数据模型给自定义的 view，view 获取到模型数据后给内部的子控件设置对应的数据。

具体做法：

* 新建一个继承 UIView 的类；
* 在 initWithFrame: 方法中添加子控件（也可以使用懒加载）；
* 重写属性的 setter 方法，在 setter 方法中设置属性值并添加到子控件上；
* 在 - (void)layoutSubViews 方法中设置子控件的 frame（一定要调用 [super layoutSubviews]）

【注意】哪些情况会触发 layoutSubviews 方法：

```
(1)init 不会触发 layoutSubviews
(2)addSubview 会触发 layoutSubviews
(3)设置 view 的 Frame 会触发 layoutSubviews，当然前提是 frame 的值设置前后发生了变化
(4)滑动一个 UIScrollView 会触发 layoutSubviews
(5)旋转 Screen 会触发父 UIView 上的 layoutSubviews；
(6)改变一个 UIView 大小的时候会触发父 UIView 上的 layoutSubviews
```

**【1-24】简单说一下 App 的启动过程，从 main 文件开始说起**

程序启动分为两类：有 storyboard 和没有 storyboard。

有 storyboard 的情况：

```
1. main 函数
2. UIApplicationMain：创建 UIApplication 对象，创建 UIApplication 的 delegate 对象
3. 根据 Info.plist 获得 Main.storyboard 的文件名，加载 Main.storyboard（有 storyboard）
4. 创建 UIWindow
5. 创建和设置 UIWindow 的 rootViewController
6. 显示窗口
```

没有 storyboard 的情况：

```
1. main 函数
2. UIApplicationMain：创建 UIApplication 对象，创建 UIApplication 的 delegate 对象
3. delegate 对象开始处理（监听）系统事件（没有 storyboard），程序启动完毕的时候，就会调用代理的 application:didFinishLauchingWithOptions: 方法
4. 在 application:didFinishLauchingWithOptions:中创建 UIWindow
5. 创建和设置 UIWindow 的 rootViewController
6. 显示窗口
```

**【1-25】UIButton 和 UITableView 的继承体系是怎样的？**

UIButton：UIButton -> UIControl -> UIView -> UIResponder -> NSObject

UITableView：UITableView ->UIScrollView -> UIView -> UIResponder -> NSObject

**【1-26】设置 scrollView 的 contentSize 能在 Viewdidload里设置吗？为什么？**

一般情况下可以设置在 viewDidLoad 中，但是在 autoLayout 下，系统会在 viewDidAppear 之前根据 subview 的 constraint 重新计算 scrollview 的 contentsize。这就是在 viewDidLoad 中设置 contentsize 不起作用的原因。

解决方法：

* 去除 autoLayout 选项，自己手动设置 contentsize；
* 如果使用 AutoLayout，需要在 viewDidAppear 里面手动设置 contentsize。






