## TCP、HTTP、HTTPS、Socket、WebSocket

#### HTTPS协议 = HTTP协议 + SSL/TLS协议
- SSL 的全称是 Secure Sockets Layer ，即安全套接层协议，是为⽹络通信提供安全及数据完整性的⼀种安全协议。
- TLS的全称是 Transport Layer Security ，即安全传输层协议。即HTTPS是安全的HTTP。

#### HTTPS 
- 全称 Hyper Text Transfer Protocol Secure ，相⽐ HTTP ，多了⼀个 secure ，这⼀个 secure 是怎么来的呢？这是由 TLS（SSL） 提供的！⼤概就是⼀个叫 openSSL 的 library 提供的。 https 和 HTTP 都属于 application layer ，基于TCP（以及UDP）协议，但是⼜完全不⼀样。TCP⽤的port是80， HTTPS⽤的是443（值得⼀提的是，google发明了⼀个新的协议，叫QUIC，并不基于TCP，⽤的port也是443， 同样是⽤来给HTTPS的。⾕歌好⽜逼啊。）总体来说，HTTPS和HTTP类似，但是⽐HTTP安全。

### TCP为什么要三次握⼿，四次挥⼿
#### 为什么需要三次握⼿：
- 为了防⽌已失效的连接请求报⽂段突然⼜传送到了服务端，因⽽产⽣错误，假设这是⼀个早已失效的报⽂段。但 server 收到此失效的连接请求报⽂段后，就误认为是 client 再次发出的⼀个新的连接请求。于是就向 client 发出确认报⽂段，同意建⽴连接。假设不采⽤“三次握⼿”，那么只要server发出确认，新的连接就建⽴了。由于现在 client 并没有发出建⽴连接的请求，因此不会理睬 server 的确认，也不会向 server 发送数据。但 server 却以为新的运输连接已经建⽴，并⼀直等待 client 发来数据。这样， server 的很多资源就⽩⽩浪费掉了。

#### 为什么需要四次挥⼿：
- 因为TCP是全双⼯通信的，在接收到客户端的关闭请求时，还可能在向客户端发送着数据，因此不能再回应关闭链接的请求时，同时发送关闭链接的请求

### WebSocket与TCP Socket的区别

- Socket 是抽象层，并不是⼀个协议，是为了⽅便使⽤TCP或UDP⽽抽象出来的⼀层，是位于应⽤层和传输层(TCP/UDP)之间的⼀组接⼝。

- WebSocket 和HTTP⼀样，都是应⽤层协议，是基于TCP 连接实现的双向通信协议，替代 HTTP轮询 的⼀种技术⽅案。是全双⼯通信协议， HTTP 是单向的，注意WebSocket和Socket是完全不同的两个概念。

	- WebSocket 建⽴连接时通过HTTP传输，但是建⽴之后，在真正传输时候是不需要HTTP协议的。

	- 相对于传统 HTTP 每次请求-应答都需要客户端与服务端建⽴连接的模式， WebSocket 是类似 Socket 的 TCP ⻓连接 的通讯模式，⼀旦 WebSocket 连接建⽴后，后续数据都以帧序列的形式传输。在客户端断开 WebSocket 连接或 Server 端断掉连接前，不需要客户端和服务端重新发起连接请求。在海量并发及客户端与服务器交互负载流量⼤的情况下，极⼤的节省了⽹络带宽资源的消耗，有明显的性能优势，且客户端发送和接受消息是在同⼀个持久连接上发起，实时性优势明显。

		- WebSocket 连接过程 —— 握⼿过程

			1. 浏览器、服务器建⽴TCP连接，三次握⼿。这是通信的基础，传输控制层，若失败后续都不执⾏。

			2. TCP连接成功后，浏览器通过HTTP协议向服务器传送 WebSocket ⽀持的版本号等信息。（开始前的HTTP握⼿）

			3. 服务器收到客户端的握⼿请求后，同样采⽤ HTTP 协议回馈数据。

			4. 当收到了连接成功的消息后，通过 TCP 通道进⾏传输通信。

- Socket.IO 是⼀个为浏览器（客户端）和服务器之间提供实时，双向和基于事件的通信软件库。

##### Socket.IO 是把数据传输抽离成 Engine.IO，内部对轮询（Polling）和 WebSocket 等进⾏了封装，抹平⼀些细节和平台兼容的问题，提供统⼀的 API。

- 注意 Socket.IO 不是 WebSocket 的实现，只是在必要时使⽤ WebSocket 传输数据，并在此基础上会加⼀些 MetaData 。这就是为什么 WebSocket 的客户端/服务器 ⽆法和 Socket.IO 的服务器/客户端进⾏通信。

## KVO KVC
* [GNUstep KVC/KVO探索（一）：KVC的内部实现](https://www.jianshu.com/p/1f9ea894a426)
* [GNUstep KVC/KVO探索（二）：KVO的内部实现](https://www.jianshu.com/p/d6e4ba25acd2)

- KVC 就是利用字符串，来查询到和对象相关联的方法、实例变量，来达到间接访问属性方法或成员变量.
- 由于KVC在访问属性方法或成员变量前加了一层查询机制，所以效率没有直接访问属性方法或成员变量高.
- 由于KVC的使用是通过字符串, 所以极易出错，推荐借鉴 RAC中的 @keyPath宏， [RAC宏分析8：@keypath](https://www.jianshu.com/p/2fe84847e4de).
- 由上面源码得知，如果key相关联信息找不到，获取传入参数类型不匹配都会导致异常，所以使用时，最后要重写相应的方法.
- 容器类包含了很多便捷的keyPath,
`@count`
`@avg`
`@max`
`@min`
`@sum`
`@distinctUnionOfArrays`
`@distinctUnionOfObjects`
`@distinctUnionOfSets`
`@unionOfArrays`
`@unionOfObjects`
`@unionOfSets`
以上keyPath都是通过容器里重写valueForKeyPath:实现，而且还可以通过数组，一次性返回内部对象的某个属性集合.
- 由于这些方法都是通过KVC实现，所以效率比较低，而且由于返回值为对象，所以还进行了不必要的封装

## OC 源码中使用到 Hash 相关的功能 
> 在 iOS开发中 随处可见 Hash 的身影，难道我们不好奇吗？
* [搞iOS的，面试官问Hash干嘛？原因远比我下面要介绍的多](https://juejin.cn/post/6844903778940878861)

## Runtime 

* [Runtime - 基于isa-swizzling实现消息监听,扩展响应式框架](https://www.jianshu.com/p/86f6059af7a0)
* [深入浅出Runtime (一) 什么是Runtime？ 定义？](https://www.jianshu.com/p/291754a90d2b)
* [深入浅出Runtime (二) Runtime的消息机制](https://www.jianshu.com/p/28c742fb8bb3)
* [深入浅出Runtime (三) Runtime的消息转发](https://www.jianshu.com/p/2b61270cd038)
* [深入浅出Runtime (四) Runtime的实际应用之一，字典转模型](https://www.jianshu.com/p/ecdb5fa307f8)

#### Runtime是什么

- 将尽可能多的决策从编译时和链接时推迟到运行时（Apple）
- 运行时系统充当着Object-C语言的操作系统，它使语言能够工作(Apple)
- 特性： 编写的代码具有运行时、动态特性

#### Runtime在Object-C的使用

##### Objective-C程序在三个不同的层次上与运行时系统交互：

- 通过Object-C源代码进行交互
- 通过NSObject类中定义的方法交互
- 通过直接调用运行时函数

##### 用来干什么 基本作用

- 在程序运行过程中，动态的创建类，动态添加、修改这个类的属性和方法；
- 遍历一个类中所有的成员变量、属性、以及所有方法
- 消息传递、转发

##### 用在哪些地方 Runtime的典型事例

- 给系统分类添加属性、方法
- 方法交换
- 获取对象的属性、私有属性
- 字典转换模型
- KVC、KVO
- 归档(编码、解码)
- NSClassFromString class<->字符串
- block
- 类的自我检测

##### 问题
- 类底层代码、类的本质？
- 类底层是如何调用方法？
- Runtime消息传递
- Runtime消息转发
- Runtime起到了什么作用？
- Runtime实际应用

总结

- Rumtime是Objective-C语言动态的核心，Objective-C的对象一般都是基于Runtime的类结构，达到很多在编译时确定方法推迟到了运行时，从而达到动态修改、确定、交换...属性及方法

## Runloop
- [同是天涯打工人](https://juejin.cn/post/7166856920425299976?searchId=202408021038430180F97D8D271BCDD94A)
- [实际开发你想用的应用场景](https://juejin.cn/post/6889769418541252615#heading-13)
- [RunLoop数据结构、RunLoop的实现机制、RunLoop的Mode、RunLoop与NSTimer和线程](https://www.jianshu.com/p/6ac19d86079e)
- [深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/)

- 线程和RunLoop一一对应， RunLoop和Mode是一对多的，Mode和source、timer、observer也是一对多的
- app 启动之后，程序进入了主线程，苹果帮我们在主线程启动了一个 RunLoop。如果是我们开辟的线程，就需要自己手动开启 RunLoop，而且，如果你不主动去获取 RunLoop，那么子线程的 RunLoop 是不会开启的，它是懒加载的形式。
- 另外苹果不允许直接创建 RunLoop，只能通过 CFRunLoopGetMain() 和 CFRunLoopGetCurrent() 去获取，其内部会创建一个 RunLoop 并返回给你（子线程），而它的销毁是在线程结束时。

- Mode，也就是模式，一个 RunLoop 当前只能处于某一种 Mode 中，就比如当前只能是白天或者夜晚一样。图上可以看到，Mode 之间是互不干扰的，平行世界，A Mode 中发生的事情与 B Mode 无关。这也是苹果丝滑的一个关键，因为苹果的滚动和默认状态分别对应两种不同的 Mode，由于 Mode 互不干扰，所以苹果可以在滚动时专心处理滚动时的事情。

	- kCFRunLoopDefaultMode：app 默认 Mode，通常主线程是在这个 Mode 下运行
	- UITrackingRunLoopMode：界面追踪 Mode，比如 ScrollView 滚动时就处于这个 Mode
	- UIInitializationRunLoopMode：刚启动 app 时进入的第一个 Mode，启动完后不再使用
	- GSEventReceiveRunLoopMode：接受系统事件的内部 Mode，通常用不到
	- kCFRunLoopCommonModes：不是一个真正意义上的 Mode，但是如果你把事件丢到这里来，那么不管你当前处于什么 Mode，都会触发你想要执行的事件。

## Block	

## 线程、进程

#### NSThread、GCD、NSOperation

## 分类、扩展

## 深拷贝、浅拷贝

## UIResponder

## 动画

## 音视频相关

## 设计模式

## 性能优化

#### 启动速度

#### 包体积

