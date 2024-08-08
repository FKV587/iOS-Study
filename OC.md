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

Objective-C Runtime 是一个强大的底层库，它提供了许多用于动态操作对象和类的功能。Objective-C 语言本身是一种动态语言，许多高级特性（如动态方法调用、消息传递、类和方法的动态添加和替换等）都是通过 Runtime 库实现的。

#### Objective-C Runtime 的概念

在 Objective-C 中，许多操作（如消息传递、方法调用等）并不是在编译时静态决定的，而是在运行时动态决定的。Objective-C Runtime 提供了一组 API，用于在运行时操作类和对象。

#### Runtime 的主要功能

1. **消息传递**：
   在 Objective-C 中，方法调用本质上是向对象发送消息。消息传递是通过 `objc_msgSend` 函数实现的。
   
   ```objective-c
   [object method]; // 相当于 objc_msgSend(object, @selector(method));
   ```

2. **动态类型信息**：
   可以在运行时获取类、方法、属性和协议的详细信息。
   
   ```objective-c
   Class cls = [MyClass class];
   const char *className = class_getName(cls);
   ```

3. **动态方法解析和替换**：
   可以在运行时动态添加、替换或交换方法实现。
   
   ```objective-c
   class_addMethod([MyClass class], @selector(newMethod), (IMP)newMethodImplementation, "v@:");
   ```

4. **动态创建类和对象**：
   可以在运行时动态创建类及其实例。
   
   ```objective-c
   Class newClass = objc_allocateClassPair([NSObject class], "NewClass", 0);
   objc_registerClassPair(newClass);
   ```

5. **关联对象**：
   可以动态为现有对象添加关联数据（类似于给对象添加额外的属性）。
   
   ```objective-c
   objc_setAssociatedObject(object, key, value, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
   ```

#### Runtime 示例

以下是一些使用 Objective-C Runtime 的示例代码：

##### 动态获取类信息

```objective-c
#import <objc/runtime.h>

void printClassInfo(Class cls) {
    NSLog(@"Class: %s", class_getName(cls));
    
    unsigned int methodCount;
    Method *methods = class_copyMethodList(cls, &methodCount);
    for (unsigned int i = 0; i < methodCount; i++) {
        SEL methodName = method_getName(methods[i]);
        NSLog(@"Method: %@", NSStringFromSelector(methodName));
    }
    free(methods);
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        printClassInfo([NSString class]);
    }
    return 0;
}
```

##### 动态方法添加

```objective-c
#import <objc/runtime.h>

void dynamicMethod(id self, SEL _cmd) {
    NSLog(@"Dynamic method called");
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Class cls = objc_allocateClassPair([NSObject class], "DynamicClass", 0);
        class_addMethod(cls, @selector(dynamicMethod), (IMP)dynamicMethod, "v@:");
        objc_registerClassPair(cls);
        
        id obj = [[cls alloc] init];
        [obj performSelector:@selector(dynamicMethod)];
    }
    return 0;
}
```

##### 方法交换（Method Swizzling）

```objective-c
#import <objc/runtime.h>

@interface MyClass : NSObject
- (void)originalMethod;
@end

@implementation MyClass
- (void)originalMethod {
    NSLog(@"Original method called");
}
@end

void swizzledMethod(id self, SEL _cmd) {
    NSLog(@"Swizzled method called");
    // 调用原始方法实现
    SEL originalSelector = @selector(originalMethod);
    Method originalMethod = class_getInstanceMethod([MyClass class], originalSelector);
    void (*originalImp)(id, SEL) = (void (*)(id, SEL))method_getImplementation(originalMethod);
    originalImp(self, _cmd);
}

void swizzle(Class class, SEL originalSelector, SEL swizzledSelector) {
    Method originalMethod = class_getInstanceMethod(class, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
    
    BOOL didAddMethod = class_addMethod(class,
                                        originalSelector,
                                        method_getImplementation(swizzledMethod),
                                        method_getTypeEncoding(swizzledMethod));
    
    if (didAddMethod) {
        class_replaceMethod(class,
                            swizzledSelector,
                            method_getImplementation(originalMethod),
                            method_getTypeEncoding(originalMethod));
    } else {
        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        MyClass *obj = [[MyClass alloc] init];
        
        SEL originalSelector = @selector(originalMethod);
        SEL swizzledSelector = @selector(swizzledMethod);

        // 动态添加新方法
        class_addMethod([MyClass class], swizzledSelector, (IMP)swizzledMethod, "v@:");
        
        // 交换方法实现
        swizzle([MyClass class], originalSelector, swizzledSelector);
        
        // 调用原始方法（实际执行的是交换后的方法实现）
        [obj originalMethod];
    }
    return 0;
}
```

##### 总结

Objective-C Runtime 是一个功能强大的底层库，提供了丰富的动态特性，使得开发者可以在运行时操作类和对象。通过 Runtime，可以实现许多高级功能，例如动态方法解析、方法交换和动态创建类等。这些特性为开发框架、调试工具和动态行为提供了强大的支持。

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
- [一文彻底理解iOS block的知识点](https://juejin.cn/post/7342798550898704425?searchId=20240802140647BA4AF5248F4828F58084)
- [iOS核心基础与面试题之block](https://juejin.cn/post/7080002745385615368?searchId=20240802140647BA4AF5248F4828F58084)
- block本质上是一个OC对象（内部有个isa指针）,block是封装了函数调用以及函数调用环境的OC对象
- 堆、栈、全局，当 Block 发生拷贝时，全局 Block 不会发生变化，栈 Block 会被复制到堆上，并成为堆 Block，而堆 Block 的引用计数会增加。在ARC下编译器会进行判断，判断是否有需要将block从栈复制到堆，如果有就自动生成将block从栈复制到堆的代码。block复制操作执行的是copy实例方法，block只要调用copy方法，栈块就会变成堆块。
	- _NSConcreteGlobalBlock：全局 Block，存储在全局数据区，不需要手动管理内存。
	- _NSConcreteStackBlock：栈 Block，存储在栈上，当离开定义它的作用域时会被自动销毁。
	- _NSConcreteMallocBlock：堆 Block，存储在堆上，需要手动管理内存（在非 ARC 环境下）。

## 线程、进程
- [iOS面试题-常问多线程问题](https://www.jianshu.com/p/bec1476c2b65)
- 多线程是指实现多个线程并发执行的技术,进而提升整体处理性能。

#### [自旋锁和互斥锁](https://juejin.cn/post/6844903716114415624?searchId=202408051101361A45F5E0B956C0962050)
- 互斥锁: 同一时刻只能有一个线程获得互斥锁,其余线程处于挂起状态.
- 自旋锁: 当某个线程获得自旋锁后,别的线程会一直做循环,尝试加锁,当超过了限定的次数仍然没有成功获得锁时,线程也会被挂起.

#### NSThread、GCD、NSOperation
- NSThread是苹果官方提供面向对象操作线程的技术，简单方便，可以直接操作线程对象，不过需要自己控制线程的生命周期。在平时使用很少，最常用到的无非就是 [NSThread currentThread]获取当前线程
- GCD(Grand Central Dispatch), 又叫做大中央调度, 它对线程操作进行了封装,加入了很多新的特性,内部进行了效率优化,提供了简洁的C语言接口, 使用更加高效,也是苹果推荐的使用方式
- NSOperation是基于GCD的上封装,将线程封装成要执行的操作,不需要管理线程的生命周期和同步,比GCD可控性更强


## 分类、扩展
在 Objective-C 中，分类（Category）和扩展（Extension）是两种用于向现有类添加方法或属性的机制。它们有一些相似之处，但也有重要的区别。下面详细介绍它们的定义、用法及主要区别。

### 分类（Category）

#### 定义
分类（Category）是 Objective-C 提供的一种机制，允许开发者在不修改现有类源代码的情况下向该类添加方法。

#### 用法
使用分类可以将方法添加到任何现有的类中，包括系统类和自定义类。分类无法添加实例变量。

#### 语法
```objective-c
// MyClass+CategoryName.h
#import "MyClass.h"

@interface MyClass (CategoryName)
- (void)newMethod;
@end

// MyClass+CategoryName.m
#import "MyClass+CategoryName.h"

@implementation MyClass (CategoryName)
- (void)newMethod {
    NSLog(@"New method from category");
}
@end
```

#### 优点
1. 代码模块化：可以将不同功能的代码分开，实现代码的模块化。
2. 无需访问源代码：可以向现有类添加方法而无需访问或修改其源代码。
3. 扩展系统类：可以向系统类（如 `NSString`、`NSArray` 等）添加自定义方法。

### 扩展（Extension）

#### 定义
扩展（Extension）通常称为匿名分类，是一种特殊的分类。它只能用于类的实现文件中，通常用于向类添加私有方法和属性。

#### 用法
扩展的主要用途是声明私有方法和属性。扩展可以添加实例变量，但仅限于类的实现部分。

#### 语法
```objective-c
// MyClass.h
#import <Foundation/Foundation.h>

@interface MyClass : NSObject
- (void)publicMethod;
@end

// MyClass.m
#import "MyClass.h"

@interface MyClass ()
@property (nonatomic, strong) NSString *privateProperty;
- (void)privateMethod;
@end

@implementation MyClass

- (void)publicMethod {
    NSLog(@"Public method");
}

- (void)privateMethod {
    NSLog(@"Private method");
}

@end
```

#### 优点
1. 隐藏实现细节：可以将私有方法和属性隐藏在类的实现文件中，保护类的封装性。
2. 添加实例变量：可以在类的实现部分添加实例变量，用于内部使用。

### 分类与扩展的主要区别

1. **作用范围**：
   - 分类：可以在任何地方使用，包括头文件和实现文件。
   - 扩展：通常只在实现文件中使用，属于类的私有部分。

2. **访问权限**：
   - 分类：方法在分类中是公开的，可以被外部访问。
   - 扩展：方法和属性在扩展中是私有的，只能在类的实现部分访问。

3. **添加内容**：
   - 分类：只能添加方法，不能添加实例变量。
   - 扩展：可以添加方法和属性，并且可以添加实例变量。

4. **编译时间**：
   - 分类：在编译时与类一起编译。
   - 扩展：在编译时与类的实现部分一起编译。

### 示例代码对比

#### 分类示例
```objective-c
// MyClass+CategoryName.h
#import "MyClass.h"

@interface MyClass (CategoryName)
- (void)categoryMethod;
@end

// MyClass+CategoryName.m
#import "MyClass+CategoryName.h"

@implementation MyClass (CategoryName)
- (void)categoryMethod {
    NSLog(@"Category method");
}
@end
```

#### 扩展示例
```objective-c
// MyClass.h
#import <Foundation/Foundation.h>

@interface MyClass : NSObject
- (void)publicMethod;
@end

// MyClass.m
#import "MyClass.h"

@interface MyClass ()
@property (nonatomic, strong) NSString *privateProperty;
- (void)privateMethod;
@end

@implementation MyClass

- (void)publicMethod {
    NSLog(@"Public method");
}

- (void)privateMethod {
    NSLog(@"Private method");
}

@end
```

### 结论

分类和扩展在 Objective-C 中都用于向现有类添加功能，但它们有不同的用途和限制。分类主要用于向类添加公开方法，而扩展主要用于向类添加私有方法和属性。理解它们的区别和用法，可以帮助开发者更好地组织代码，实现更高的封装性和代码复用性。

## 深拷贝与浅拷贝

### 相同点

1. **目的**：两者的主要目的是复制对象，创建新的对象引用。
2. **方法**：在 Objective-C 中，`copy` 和 `mutableCopy` 方法都可以用于复制对象。
3. **实现**：都需要遵循 `NSCopying` 或 `NSMutableCopying` 协议。

### 不同点

1. **复制的深度**：
   - **浅拷贝**：仅复制对象的引用（指针），新对象和原对象共享同一份数据。即，新对象指向原对象的数据内存地址。
   - **深拷贝**：复制对象及其所有子对象，即新对象和原对象完全独立，数据内存地址不同，互不影响。

2. **内存使用**：
   - **浅拷贝**：因为仅复制引用，所以内存占用较少。
   - **深拷贝**：因为复制了所有对象及其子对象，所以内存占用较多。

3. **独立性**：
   - **浅拷贝**：新对象和原对象指向同一份数据，修改其中一个对象会影响另一个对象。
   - **深拷贝**：新对象和原对象完全独立，修改其中一个对象不会影响另一个对象。

4. **适用场景**：
   - **浅拷贝**：适用于需要多个引用共享同一份数据的场景，如读操作较多的情况。
   - **深拷贝**：适用于需要独立副本的场景，如需要对副本进行修改而不影响原对象的情况。

### 浅拷贝（Shallow Copy）

浅拷贝只复制对象的引用，不复制实际的对象内容，因此新对象和原对象共享同一份数据。在浅拷贝中，以下是一些常见对象的行为：

1. **不可变对象**（如 `NSString`、`NSArray`、`NSDictionary` 等）：
   - 对于不可变对象，浅拷贝和深拷贝的效果相同，因为这些对象本质上是不可变的。
   ```objective-c
   NSString *string = @"Hello, World!";
   NSString *stringCopy = [string copy]; // 浅拷贝
   ```

2. **可变对象**（如 `NSMutableString`、`NSMutableArray`、`NSMutableDictionary` 等）：
   - 对于可变对象，浅拷贝只复制对象的引用，修改其中一个对象会影响另一个对象。
   ```objective-c
   NSMutableArray *originalArray = [NSMutableArray arrayWithObjects:@"one", @"two", @"three", nil];
   NSMutableArray *shallowCopyArray = [originalArray copy]; // 浅拷贝
   ```

### 深拷贝（Deep Copy）

深拷贝复制对象及其所有子对象，因此新对象和原对象完全独立。以下是一些常见对象在深拷贝时的行为：

1. **不可变对象**（如 `NSString`、`NSArray`、`NSDictionary` 等）：
   - 对于不可变对象，深拷贝会创建对象的独立副本，但由于不可变对象的特性，其效果与浅拷贝相同。
   ```objective-c
   NSArray *originalArray = @[@"one", @"two", @"three"];
   NSArray *deepCopyArray = [[NSArray alloc] initWithArray:originalArray copyItems:YES]; // 深拷贝
   ```

2. **可变对象**（如 `NSMutableString`、`NSMutableArray`、`NSMutableDictionary` 等）：
   - 对于可变对象，深拷贝会创建对象及其子对象的独立副本，修改其中一个对象不会影响另一个对象。
   ```objective-c
   NSMutableArray *originalArray = [NSMutableArray arrayWithObjects:@"one", @"two", @"three", nil];
   NSMutableArray *deepCopyArray = [[NSMutableArray alloc] initWithArray:originalArray copyItems:YES]; // 深拷贝
   ```

### 特殊对象

1. **自定义对象**：
   - 自定义对象需要实现 `NSCopying` 和 `NSMutableCopying` 协议，才能进行浅拷贝和深拷贝。对于深拷贝，需要在 `copyWithZone:` 方法中实现深度复制逻辑。
   ```objective-c
   @interface MyObject : NSObject <NSCopying>
   @property (nonatomic, strong) NSString *name;
   @end

   @implementation MyObject
   - (id)copyWithZone:(NSZone *)zone {
       MyObject *copy = [[[self class] allocWithZone:zone] init];
       copy.name = [self.name copy];
       return copy;
   }
   @end
   ```

2. **集合对象**（如 `NSArray`、`NSDictionary`、`NSSet` 等）：
   - 集合对象在深拷贝时，可以使用 `copyItems:YES` 参数来递归地复制其子对象。
   ```objective-c
   NSArray *originalArray = @[@"one", @"two", @"three"];
   NSArray *deepCopyArray = [[NSArray alloc] initWithArray:originalArray copyItems:YES]; // 深拷贝
   ```

### 总结

- **浅拷贝**：适用于不可变对象和需要共享数据的场景，只复制对象引用，不复制实际内容。
- **深拷贝**：适用于可变对象和需要独立副本的场景，复制对象及其所有子对象，使新对象与原对象完全独立。

根据具体需求选择合适的复制方式，浅拷贝仅复制对象引用，适用于共享数据场景，深拷贝复制对象及其子对象，适用于需要独立副本的场景，从而更好地管理对象生命周期和内存使用。

## View
- [iOS面试题-常问UI问题](https://www.jianshu.com/p/ca1ff37499d5)

### UIView 相关底层逻辑知识点

1. **视图层级和渲染**：
   - 视图层级关系
   - 渲染顺序和绘制流程

2. **Core Animation 集成**：
   - 与 CALayer 的关系
   - Layer-backed 视图

3. **事件处理**：
   - 事件传递机制（Hit-Testing）
   - 事件响应链

4. **布局系统**：
   - Auto Layout 机制
   - 布局约束的解析和更新
   - layoutSubviews 的调用时机

5. **重绘机制**：
   - setNeedsDisplay 和 setNeedsLayout
   - drawRect: 的调用时机

6. **动画**：
   - UIView 动画封装
   - 隐式动画和显式动画

### CALayer 相关底层逻辑知识点

1. **图层树**：
   - 图层树的结构
   - 视图树和图层树的关系

2. **寄宿图（Backing Image）**：
   - contents 属性
   - 图层内容的设置和更新

3. **离屏渲染**：
   - 离屏渲染的触发条件
   - 离屏渲染的性能影响

4. **图层几何学**：
   - 坐标系和位置属性
   - Anchor Point 的作用

5. **动画系统**：
   - Core Animation 动画模型
   - 动画事务（CATransaction）

6. **图层行为**：
   - 隐式动画和显式动画
   - CAAction 协议

7. **渲染流水线**：
   - 图层的合成和渲染过程
   - GPU 和 CPU 的合作

8. **图层优化**：
   - shouldRasterize 和 rasterizationScale
   - 影子路径（shadowPath）优化


## 动画

### 动画相关知识点

1. **基本动画**：
   - CABasicAnimation
   - CAKeyframeAnimation
   - CAAnimationGroup

2. **隐式动画**：
   - 自动触发的属性动画
   - Layer 属性变化的隐式动画

3. **显式动画**：
   - 创建和配置显式动画
   - 动画的开始和结束

4. **动画事务**：
   - CATransaction
   - 事务的开始和提交
   - 动画的延迟和时间调整

5. **动画曲线**：
   - 动画时长
   - 动画节奏（Ease In, Ease Out, Linear）
   - UIView 动画曲线

6. **关键帧动画**：
   - CAKeyframeAnimation 的使用
   - 使用关键帧定义动画路径

7. **自定义动画**：
   - 使用 Core Animation 创建自定义动画
   - 动画的过渡效果

8. **动画组合**：
   - 使用 CAAnimationGroup 组合多个动画
   - 动画之间的协调和同步

9. **动画中断和取消**：
   - 动画的中断处理
   - 取消进行中的动画

10. **图层与动画的关系**：
    - UIView 动画与 CALayer 动画的区别
    - 动画性能优化

11. **UIView 动画方法**：
    - [UIView animateWithDuration:]
    - [UIView animateWithDuration:animations:]
    - [UIView transitionWithView:duration:options:animations:completion:]

12. **时间和延迟**：
    - 动画时间控制
    - 动画延迟处理

13. **动画状态的恢复**：
    - 保存和恢复动画状态
    - 动画完成后的处理

14. **GPU 渲染**：
    - 使用 GPU 提高动画性能
    - 避免 CPU 渲染带来的性能问题

15. **交互动画**：
    - 动画与手势识别的结合
    - 动态响应用户输入

## 音视频相关

### 音视频相关知识点

1. **音频格式与编解码器**：
   - 常见音频格式（如 WAV, MP3, AAC, FLAC）及其特性
   - 编解码器（Codec）的概念及工作原理
   - 音频压缩技术（如动态范围压缩、均衡器）

2. **视频格式与压缩**：
   - 常见视频格式（如 AVI, MP4, MOV, MKV）及其特性
   - 视频压缩算法（如 H.264, H.265）及其帧内外编码机制
   - 视频流的分段处理与编码（如 GOP 结构）

3. **音频采样与信号处理**：
   - 采样定理（Nyquist Theorem）和量化误差
   - 音频信号处理基础（如 FIR 和 IIR 滤波器、频域分析）
   - 频谱分析与信号处理算法（如时域和频域转换）

4. **视频处理与播放**：
   - AVFoundation 框架的使用（AVPlayer, AVPlayerLayer）
   - 视频录制（AVCaptureSession）和编辑（AVMutableComposition）
   - 使用 Core Graphics 和 Core Animation 进行视频渲染

5. **实时音视频通讯**：
   - WebRTC 的概念和应用
   - RTSP、RTP、UDP 和 TCP 的使用与区别
   - 实时音视频处理的协议与实现

6. **音视频同步**：
   - 时间码（Timecode）和音视频同步原理
   - 播放时的同步处理技术（如 AVAudioTime 和 AVPlayerItem）

7. **流媒体技术**：
   - HTTP Live Streaming (HLS) 和 Dynamic Adaptive Streaming over HTTP (DASH) 的工作原理
   - 媒体容器格式的结构及其元数据处理
   - RTMP 协议与流媒体服务器的实现

8. **音效处理与视频特效**：
   - 使用 AVAudioEngine 进行音效处理
   - Core Image 和 GPUImage 进行视频特效处理
   - 3D 音效和空间音频的实现

9. **编码与解码机制**：
   - 有损与无损压缩原理
   - 常见编解码器的内部结构（如 H.264 的宏块和帧类型）
   - 媒体转换和转码技术（如 FFmpeg 的应用）

10. **网络传输与缓冲管理**：
    - 音视频流的网络传输协议（RTP, RTCP）
    - 视频缓冲区的管理与优化（如自适应流媒体的缓冲策略）
    - 网络带宽管理与流控制技术

11. **硬件加速与驱动**：
    - GPU 和 DSP 的利用在音视频处理中的应用
    - Core Audio 的底层架构和音频单位（Audio Units）
    - 使用 VideoToolbox 进行硬件加速的编解码

12. **用户权限与安全性**：
    - 麦克风和摄像头的访问权限处理
    - 在 Info.plist 中配置权限请求
    - 数据隐私和用户授权的最佳实践

13. **低延迟处理**：
    - 实时音视频传输的延迟来源及优化技术
    - 音视频处理管道的构建与管理
    - 使用音视频队列的技术（如 AVSampleBufferDisplayLayer）

14. **自定义与扩展**：
    - 自定义音视频处理组件的实现
    - 扩展 AVFoundation 功能的技巧
    - 自定义播放器和录制器的设计模式

15. **媒体信息提取与处理**：
    - 使用 AVAsset 和 AVMetadataItem 提取媒体信息
    - 媒体文件的封装格式和解读
    - 数据库与媒体库的集成（如 Photos 框架）

16. **音频混音与合成**：
    - 多音轨混音的实现
    - 使用 AVAudioMix 和 AVAudioMixInputParameters
    - 音频合成和处理技术

## 设计模式

### iOS 常用设计模式

1. **单例模式（Singleton）**：
   - 确保一个类只有一个实例，并提供一个全局访问点。
   - 适用于共享状态或资源的管理，如网络管理器、数据库连接等。

2. **观察者模式（Observer）**：
   - 定义对象间一对多的依赖关系，当一个对象的状态改变时，所有依赖于它的对象都会得到通知并自动更新。
   - 在 iOS 中常用于 NotificationCenter 和 KVO（键值观察）。

3. **代理模式（Delegate）**：
   - 通过定义一个代理协议来实现对象间的通信，允许一个对象委托另一个对象来完成某个任务。
   - 常见于 UITableView 和 UICollectionView 的数据源和代理。

4. **适配器模式（Adapter）**：
   - 将一个接口转换成客户端所期望的另一种接口，使得原本因接口不兼容而无法一起工作的类能够一起工作。
   - 在 iOS 中常用于 UIKit 和 Core Data 的接口适配。

5. **策略模式（Strategy）**：
   - 定义一系列算法，将每一个算法封装起来，使它们可以互换。
   - 在 iOS 中常用于处理不同的行为策略，如图像处理、排序算法等。

6. **命令模式（Command）**：
   - 将一个请求封装成一个对象，从而使您可以用不同的请求对客户进行参数化。
   - 常用于处理用户输入和操作的管理，如撤销/重做功能。

7. **构建者模式（Builder）**：
   - 使用多个简单的对象一步步构建成一个复杂的对象，尤其适合构建可变对象。
   - 在 iOS 中常用于构建复杂的 UI 组件或配置对象。

8. **模板方法模式（Template Method）**：
   - 定义一个算法的骨架，而将一些步骤延迟到子类中。
   - 在 iOS 中常用于 UIViewController 的生命周期管理。

9. **状态模式（State）**：
   - 允许一个对象在其内部状态改变时改变它的行为。
   - 在 iOS 中常用于管理视图控制器的状态或应用程序的状态。

10. **访问者模式（Visitor）**：
    - 通过将操作封装在访问者中，允许对对象结构中的元素执行操作而不需要修改这些元素的类。
    - 在 iOS 中较少使用，但在复杂的数据结构操作时可以考虑。

11. **组合模式（Composite）**：
    - 将对象组合成树形结构以表示部分-整体层次结构。
    - 在 iOS 中常用于构建复杂的视图层次结构，如自定义 UI 组件。

12. **桥接模式（Bridge）**：
    - 将抽象部分与其实现部分分离，使它们可以独立变化。
    - 在 iOS 中适合于支持不同平台或不同外观的 UI 设计。

## 性能优化

### iOS 性能优化（优先级排列）

1. **内存管理**：
   - 使用 ARC（自动引用计数）和手动内存管理相结合，避免强引用循环，使用 weak 或 unowned 引用。
   - 优化内存分配，使用 `malloc`、`free` 和 `posix_memalign` 进行底层内存管理。
   - 使用对象池和 `NSCache` 管理可回收的内存资源，避免频繁的内存分配和释放。

2. **减少启动时间**：
   - 优化启动时加载的资源，延迟加载不必要的模块，使用 App Thinning 和 On-Demand Resources 减少应用包体积。

3. **减少 CPU 占用**：
   - 使用 Instruments 监测 CPU 使用情况，找出性能瓶颈，避免在主线程上执行繁重计算。

4. **异步处理**：
   - 使用 GCD（Grand Central Dispatch）和 NSOperationQueue 进行并发处理，避免主线程阻塞。
   - 使用低级线程 API（如 `pthread`）直接控制线程创建和销毁。

5. **优化绘制性能**：
   - 减少视图层次，避免不必要的视图嵌套，使用 CAShapeLayer 和 CATiledLayer 进行复杂绘制。
   - 使用 `CATransaction` 控制动画提交，减少图层的绘制次数，利用图层的 `shouldRasterize` 属性优化复杂图层。

6. **网络请求优化**：
   - 使用 NSURLSession 进行高效的网络请求，支持数据缓存和后台下载，使用 CFNetwork 和 libcurl 进行底层网络请求管理。
   - 减少请求次数，使用合适的 HTTP 方法和数据压缩，进行批量请求以减少带宽消耗。

7. **减少内存占用**：
   - 通过 Instruments 监测内存使用情况，找出内存泄漏和不必要的内存占用，使用固定大小的对象减少内存碎片。

8. **动画性能优化**：
   - 使用 Core Animation 进行高效动画处理，避免不必要的绘制和计算，使用关键帧动画和物理仿真来实现复杂动画效果。

9. **图片和资源管理**：
   - 使用合适的图片格式和分辨率，异步加载图片，使用 SDWebImage 等库进行缓存。
   - 对大文件进行懒加载，避免一次性加载大量资源，使用静态或全局变量存储频繁使用的对象。

10. **数据持久化优化**：
    - 使用 Core Data 或 SQLite 进行高效的数据存储和查询，使用批量插入和更新，避免频繁的数据库操作。

11. **利用懒加载和按需加载**：
    - 使用懒加载技术减少初始加载时的内存和性能开销，在需要时才加载和处理数据。

12. **性能分析工具**：
    - 使用 Instruments 的 Time Profiler 和 Allocations 工具，分析性能瓶颈，利用内存图工具定位内存泄漏。

13. **文件 I/O 优化**：
    - 使用 `mmap` 映射文件到内存，进行批量读写操作，减少文件操作次数，提高效率。

14. **精简渲染管线**：
    - 使用 Metal 或 OpenGL ES 进行底层图形渲染，优化渲染性能，避免不必要的状态切换和绘制调用。
