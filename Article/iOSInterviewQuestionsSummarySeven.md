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


## 知识点19 Swift

**【扩展 19-1】Swift 中 struct 和 class 的区别？**

**【扩展 19-2】Swift 是如何实现多态的？**

**【扩展 19-3】Swift 和 OC，各自的优缺点有哪些？**

**【扩展 19-4】用 Alamofire 比直接使用 URLSession，优势是什么？**


## 知识点20 开放性问题

**【扩展 20-1】哪一个项目技术点最能体现自己的技术实力？具体讲一下。**

**【扩展 20-2】你在项目中遇到的最大的问题是什么？你是怎么解决的？**

**【扩展 20-3】你是如何学习 iOS 的？**

**【扩展 20-4】和产品经理、测试产生冲突时，你是怎么解决的？**
