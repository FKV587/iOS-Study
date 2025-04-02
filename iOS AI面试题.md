# iOS 面试题

## 基础问题（考察核心概念）

### 1. ARC（自动引用计数）的理解
**问题：** 说说你对 ARC（自动引用计数）的理解，什么情况下会导致循环引用？如何解决？

**答案：** 
自动引用计数（ARC）由系统维护一个引用计数表来管理对象的内存释放。当一个对象被持有时，引用计数加 1；当引用计数降为 0 时，系统会自动释放该对象。

如果对象 A 持有对象 B，同时对象 B 也强引用对象 A，就会导致循环引用，或者多个对象之间相互强引用，从而形成引用循环，导致内存无法释放。常见的循环引用场景包括 delegate 和 block 的强引用问题。

通常，可以通过让对象 A 对对象 B 使用 weak（弱引用），或者让对象 B 对对象 A 使用 weak，从而打破循环引用，避免内存泄漏。

### 2. struct 和 class 的区别
**问题：** struct 和 class 有什么区别？它们在内存中的存储方式有什么不同？

**答案：**

**主要区别**

| 特性 | struct | class |
|------|---------|--------|
| 类型 | 值类型 | 引用类型 |
| 存储位置 | 栈（大数据量可能在堆上） | 堆 |
| 传递方式 | 值拷贝（每次赋值创建新副本） | 引用传递（多个引用指向同一对象） |
| 继承 | 不支持 | 支持 |
| let 限制 | 不能修改属性 | var 属性可修改 |
| deinit 方法 | 无 | 有 |
| 线程安全 | 安全（不会共享状态） | 可能不安全（需要同步） |

**详细说明**

1. **值类型 vs 引用类型**
   - struct（结构体）是值类型，存储在栈区，但如果结构体包含大量数据，编译器可能会优化，将其存储到堆区。赋值或传递时会进行值拷贝，生成新的副本。
   - class（类）是引用类型，存储在堆区。赋值或传递时是引用传递，多个变量可以指向同一个对象。

2. **继承**
   - struct 不支持继承，适用于数据封装和轻量级对象。
   - class 支持继承，可以实现面向对象编程中的继承关系。

3. **变量修改**
   - struct 使用 let 声明后不可修改，即使结构体内部的属性是 var，整体不可变。
   - class 即使使用 let 声明，也可以修改其 var 属性，但不能更改对象的引用。

4. **释放机制**
   - struct 没有 deinit 方法，因为它的生命周期由作用域自动管理。
   - class 有 deinit 方法，用于在对象释放时执行清理操作。

5. **线程安全**
   - struct 是线程安全的，因为值拷贝不会共享状态，每个线程持有独立的副本。
   - class 线程不安全，多个线程可以同时修改同一个对象，需要手动同步。

**适用场景**
- struct 适用于轻量级数据模型，如 CGPoint、CGRect、URL 等。
- class 适用于需要共享状态、管理生命周期的对象，如 UIViewController、NSObject 子类等。

### 3. GCD 和 NSOperation 的区别
**问题：** 你如何理解 GCD 和 NSOperation，它们有什么区别？什么时候用 GCD，什么时候用 NSOperationQueue？

**GCD 和 NSOperation 的区别**

| 特性 | GCD | NSOperation |
|------|-----|-------------|
| API 级别 | C 语言底层 API，轻量高效 | Objective-C / Swift 封装，面向对象 |
| 任务管理 | 直接提交任务到队列，不支持取消、依赖、优先级 | 任务可管理，可设置依赖、取消、优先级 |
| 执行方式 | 同步（sync）/异步（async），支持并行或串行队列 | 封装成 NSOperation 对象，添加到 NSOperationQueue |
| 线程控制 | 不能手动暂停、取消任务 | 任务可以暂停、取消 |
| 并发控制 | 依赖于队列类型（串行/并行） | 可控制最大并发数 |

**使用场景**

**使用 GCD 的情况**
- 简单异步任务（如网络请求、后台任务）
- 高性能、轻量级任务，不需要额外的管理
- 避免 Objective-C 复杂性，直接使用 DispatchQueue

示例代码：
```swift
DispatchQueue.global(qos: .background).async {
    print("在后台执行任务")
    DispatchQueue.main.async {
        print("回到主线程更新 UI")
    }
}
```

**使用 NSOperationQueue 的情况**
- 任务之间有依赖关系（如先下载，再处理，再存储）
- 需要手动取消任务（如用户取消下载任务）
- 更好地控制并发数

示例代码：
```swift
let queue = OperationQueue()
let operation1 = BlockOperation {
    print("任务 1")
}
let operation2 = BlockOperation {
    print("任务 2 依赖 1")
}
operation2.addDependency(operation1)
queue.addOperations([operation1, operation2], waitUntilFinished: false)
```

**总结**
- GCD 更底层、更高效，适合简单异步任务，但无法取消、管理依赖。
- NSOperation 更高级、更灵活，适合复杂任务管理（依赖、取消、优先级）。

### Runtime
**问题：** 消息发送流程
1. **Objective-C/Swift（@objc）的方法调用** 是基于 **Runtime 消息传递**，底层使用 `objc_msgSend`。
2. **查找顺序**：**方法缓存 → 类方法列表 → `superclass` 继承链**。
3. **找不到方法时**，Runtime 触发 **方法解析 & 消息转发**，防止 `unrecognized selector` 崩溃：
   - **`resolveInstanceMethod:`**（动态添加方法）。
   - **`forwardingTargetForSelector:`**（转发给另一个对象）。
   - **`methodSignatureForSelector:` + `forwardInvocation:`**（完整消息转发）。

### 4. RunLoop 的作用
**问题：** 解释 RunLoop 的作用，它在 iOS 开发中的应用有哪些？
**RunLoop 的基本概念**
RunLoop 是 iOS 应用程序中的一个重要机制，它负责管理和调度线程的工作。每个线程都有一个对应的 RunLoop，主线程的 RunLoop 是自动创建和运行的，而子线程的 RunLoop 需要手动创建和运行。

**RunLoop 的主要作用**

1. **事件处理**
   - 处理输入源（如用户触摸事件、系统事件）
   - 处理定时器事件
   - 确保应用程序能够及时响应用户操作
   - 管理各种事件源（Source）和观察者（Observer）

2. **线程管理**
   - 维持线程的生命周期
   - 有任务时唤醒线程
   - 空闲时进入休眠状态
   - 优化 CPU 资源使用

3. **性能优化**
   - 避免线程频繁创建和销毁
   - 减少系统资源消耗
   - 提高应用程序响应性

**RunLoop 的工作原理**

1. **运行循环**
   - 检查是否有待处理的事件
   - 如果有事件，处理事件
   - 如果没有事件，进入休眠状态
   - 被唤醒后继续检查事件

2. **事件源类型**
   - Source0：非基于 Port 的事件源
   - Source1：基于 Port 的事件源
   - Timer：定时器事件
   - Observer：观察者，用于监听 RunLoop 状态变化

**RunLoop 在 iOS 开发中的应用**

1. **主线程任务管理**
```swift
// 主线程的 RunLoop 自动运行，不需要手动管理
DispatchQueue.main.async {
    // UI 更新等主线程任务
    self.updateUI()
}
```

2. **定时器（Timer）管理**
```swift
// 创建定时器
let timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { timer in
    print("定时器触发")
}

// 添加到 RunLoop
RunLoop.current.add(timer, forMode: .common)
RunLoop.current.run()
```

3. **常驻线程**
```swift
class ThreadManager {
    private var thread: Thread?
    
    func createThread() {
        thread = Thread { [weak self] in
            let runLoop = RunLoop.current
            let port = Port()
            runLoop.add(port, forMode: .default)
            
            // 保持线程存活
            while !Thread.current.isCancelled {
                runLoop.run(mode: .default, before: Date(timeIntervalSinceNow: 0.1))
            }
        }
        thread?.start()
    }
    
    func stopThread() {
        thread?.cancel()
    }
}
```

4. **自动释放池（AutoreleasePool）**
```swift
// 在大量临时对象创建的场景中使用
autoreleasepool {
    // 创建大量临时对象
    for i in 0..<10000 {
        let obj = SomeObject()
        // 使用对象
    }
} // 自动释放池结束时释放对象
```

**RunLoop 的运行模式**

1. **Default Mode**
   - 默认模式，处理大多数事件
   - 包含 Timer、网络请求等事件
   - 最常用的运行模式

2. **Tracking Mode**
   - 用于处理 UI 相关事件
   - 在用户交互时自动切换到该模式
   - 确保 UI 事件的及时响应

3. **Common Mode**
   - 包含 Default 和 Tracking 模式的事件
   - 用于需要同时处理 UI 和其他事件的场景
   - 最全面的运行模式

**注意事项**

1. **主线程 RunLoop**
   - 主线程的 RunLoop 是自动创建和运行的
   - 不需要手动管理主线程的 RunLoop
   - 主线程的 RunLoop 不能被手动停止

2. **子线程 RunLoop**
   - 需要手动创建和运行
   - 注意及时停止，避免内存泄漏
   - 合理使用 RunLoop 模式
   - 确保线程安全

3. **性能考虑**
   - 避免在 RunLoop 中执行耗时操作
   - 合理使用 RunLoop 模式，避免不必要的模式切换
   - 注意内存管理，及时释放不需要的资源
   - 避免创建过多的常驻线程

4. **常见问题**
   - Timer 在滚动时失效（需要添加到 Common Mode）
   - 子线程 RunLoop 未正确停止导致内存泄漏
   - 主线程阻塞导致界面卡顿

### 5. 事件传递和响应链
**问题：** 说说 iOS 事件传递和响应链的工作机制。
**事件传递和响应链的基本概念**

1. **事件传递（Hit-Testing）**
   - 从上到下寻找目标视图
   - 从 UIApplication 开始，经过 UIWindow，最终找到最合适的目标视图
   - 类似于水滴从高处滴落的过程

2. **事件响应链（Responder Chain）**
   - 从下到上寻找可以处理事件的对象
   - 从目标视图开始，沿着父视图、控制器、窗口、应用程序逐级向上
   - 类似于抛球向上传递的过程

**事件传递机制（Hit-Testing）**

1. **传递流程**
   - UIApplication → UIWindow：事件最先传递到 UIApplication
   - UIWindow → Root View：窗口从根视图开始查找
   - 递归遍历子视图：从最上层子视图开始，直到找到最深的子视图

2. **关键方法**
```swift
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    // 1. 检查视图是否可交互
    if !self.isUserInteractionEnabled || self.isHidden || self.alpha <= 0.01 {
        return nil
    }
    
    // 2. 检查点击点是否在视图范围内
    if !self.point(inside: point, with: event) {
        return nil
    }
    
    // 3. 从最顶层子视图开始遍历
    for subview in self.subviews.reversed() {
        let convertedPoint = subview.convert(point, from: self)
        if let hitView = subview.hitTest(convertedPoint, with: event) {
            return hitView
        }
    }
    
    // 4. 没有更深的视图，当前视图接收事件
    return self
}
```

**事件响应链机制（Responder Chain）**

1. **响应链顺序**
   ```
   目标视图 (UIView)
      ⬆
   父视图 (Superview)
      ⬆
   视图控制器 (UIViewController)
      ⬆
   窗口 (UIWindow)
      ⬆
   应用程序 (UIApplication)
      ⬆
   应用程序代理 (AppDelegate)
   ```

2. **响应者对象**
   - UIView 及其子类
   - UIViewController 及其子类
   - UIWindow
   - UIApplication
   - AppDelegate

3. **事件处理方法**
```swift
// 触摸事件
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    super.touchesBegan(touches, with: event)
    print("当前视图响应了触摸事件")
}

// 修改响应链
override var next: UIResponder? {
    return superview // 自定义响应链顺序
}
```

**常见应用场景**

1. **扩大点击区域**
```swift
override func point(inside point: CGPoint, with event: UIEvent?) -> Bool {
    // 扩大点击区域到按钮周围 20 点
    let expandedBounds = bounds.insetBy(dx: -20, dy: -20)
    return expandedBounds.contains(point)
}
```

2. **穿透点击**
```swift
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    // 让点击事件穿透当前视图
    let hitView = super.hitTest(point, with: event)
    return hitView == self ? nil : hitView
}
```

3. **自定义响应链**
```swift
class CustomView: UIView {
    override var next: UIResponder? {
        // 自定义响应链顺序
        return superview?.superview
    }
}
```

**注意事项**

1. **性能考虑**
   - hitTest 方法会被频繁调用，注意性能优化
   - 避免在 hitTest 中执行耗时操作
   - 合理使用 isUserInteractionEnabled 控制事件传递

2. **常见问题**
   - 子视图超出父视图范围时的事件处理
   - 多个重叠视图的事件传递顺序
   - 自定义响应链可能影响系统默认行为

3. **最佳实践**
   - 优先使用系统提供的事件处理方法
   - 谨慎修改响应链顺序
   - 注意内存泄漏问题
   - 合理使用事件拦截和穿透

## 进阶问题（考察性能优化与底层原理）

### 6. MVVM 和 MVC 的区别
**问题：** 说说 MVVM 和 MVC 的区别，如何在 iOS 项目中更好地应用 MVVM？

**MVVM 和 MVC 的区别**

| 特性  | MVC（Model-View-Controller） | MVVM（Model-View-ViewModel） |
|------|-----------------------------|-----------------------------|
| **核心思想** | 由 Controller 负责处理 UI 逻辑，直接与 Model 交互 | 通过 ViewModel 处理逻辑，View 仅监听数据变化 |
| **代码组织** | ViewController 既处理 UI，又处理数据逻辑，容易变得臃肿 | 业务逻辑被拆分到 ViewModel，使 ViewController 更轻量 |
| **数据绑定** | 需要手动更新 UI | 通过数据绑定（KVO、Combine、RxSwift）自动更新 UI |
| **适用场景** | 适合小型项目，代码简单直观 | 适合大型项目，降低耦合，便于测试和维护 |

**如何在 iOS 项目中更好地应用 MVVM？**

1. **使用 ViewModel 处理 UI 逻辑**
   - ViewModel 负责数据处理、网络请求、业务逻辑，而 ViewController 仅负责 UI 展示。
   - 例如：在一个列表页中，ViewModel 负责获取数据并转换成适合 UI 显示的格式。

2. **利用数据绑定**
   - 可以使用 **Combine**、**RxSwift**、**KVO** 或者 **闭包回调** 实现 View 和 ViewModel 之间的数据绑定。
   - 例如：
     ```swift
     class ViewModel {
         @Published var items: [String] = []
         
         func fetchData() {
             // 模拟网络请求
             DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
                 DispatchQueue.main.async {
                     self.items = ["苹果", "香蕉", "橙子"]
                 }
             }
         }
     }
     ```

3. **减轻 ViewController 负担**
   - 在 MVC 中，ViewController 可能既要处理 UI，又要处理数据请求、解析等逻辑，而在 MVVM 中，ViewController 只负责 UI 交互，业务逻辑交给 ViewModel。

4. **使用依赖注入（Dependency Injection）**
   - 让 ViewController 通过初始化传递 ViewModel，而不是直接在内部创建。
   - 这样可以提高代码的可测试性和灵活性。

**总结**
- **MVC** 适用于小型项目，但容易导致 ViewController 过于臃肿。
- **MVVM** 通过拆分逻辑，让 ViewModel 处理数据，View 仅负责显示，降低了耦合性，提高了可维护性和测试性。
- 在 iOS 项目中，使用 **Combine**、**RxSwift** 或 **KVO** 进行数据绑定，可以更好地发挥 MVVM 的优势。

### 7. 列表性能优化
**问题：** 你如何优化 UITableView/UICollectionView 的滚动性能？
- 优化 UITableView/UICollectionView 滚动性能的关键在于减少不必要的计算和渲染开销。首先，确保正确复用 cell，避免频繁创建销毁。其次，预计算 cell 高度并使用 automaticDimension 让系统自动调整尺寸。同时，减少视图嵌套，避免使用透明 UIView、圆角和阴影，以防止离屏渲染影响性能。对于图片加载，推荐使用异步加载方式（如 SDWebImage），并限制图片大小。此外，在 iOS 13 及以上可以使用 diffableDataSource 进行更高效的数据更新。综合这些优化手段，可以显著提升列表的流畅度，减少卡顿问题。 🚀

### 8. Copy-On-Write 机制
**问题：** 介绍下 Swift 中的 Copy-On-Write 机制，它如何影响 Array 的性能？
- 在 Swift 中，**Copy-On-Write（COW）** 机制是一种优化值类型（如 `Array`、`Dictionary`、`Set`）的内存管理方式。它的核心思想是：**只有在写入（修改）数据时才会进行复制**，如果多个变量共享同一个数据且未发生修改，则它们仍然指向同一块内存，避免不必要的复制，提高性能。  

**Copy-On-Write 如何影响 `Array` 的性能？**  
1. **避免不必要的复制**：当 `Array` 发生赋值时，Swift 不会立即创建新的副本，而是多个变量共享同一块内存，直到其中一个发生修改才进行复制。  
2. **降低内存开销**：如果 `Array` 只是传递给函数或赋值给其他变量，但不进行修改，系统不会复制数据，从而减少内存使用。  
3. **提高性能**：对于大数组，COW 机制能显著减少数据拷贝次数，提高执行效率，避免不必要的性能损耗。  

**示例代码**  
```swift
var array1 = [1, 2, 3]  
var array2 = array1  // 此时 array2 只是引用相同的底层存储，未发生复制  
array2.append(4)      // 发生写入操作，此时才真正进行复制，array1 和 array2 变成独立的对象  

print(array1)  // [1, 2, 3]  
print(array2)  // [1, 2, 3, 4]  
```
当 `array2.append(4)` 时，Swift 检测到 `array2` 发生了修改，这时才会进行真正的复制，确保 `array1` 不受影响。

**如何优化 `Array` 性能？**
- **避免不必要的修改**：如果 `Array` 不需要改变，尽量使用 `let` 声明，这样可以避免触发 COW。  
- **使用 `reserveCapacity` 预分配空间**：如果事先知道数组大概的大小，可以调用 `array.reserveCapacity(n)` 预分配内存，减少扩容带来的性能损耗。  
- **使用 `UnsafeMutableBufferPointer` 直接操作内存**：对于性能敏感的场景，可以绕过 COW，直接修改底层数据（但要注意安全性）。  

**总结**
Swift 的 COW 机制在保证值类型安全性的同时，提供了高效的性能优化，使 `Array` 在大多数情况下都能保持高效运行。理解并善用 COW，可以帮助开发者在大数据处理时写出更加高性能的代码。 🚀

### 9. 通信机制对比
**问题：** KVO、NotificationCenter 和 Delegate 三者的区别，分别适用于什么场景？
- KVO 适用于监听对象属性的变化，NotificationCenter 适用于一对多的全局事件通知，而 Delegate 适用于一对一的回调，常用于组件间的直接通信。

1. **KVO（键值观察）**  
   - **特点**：监听对象属性的变化，自动触发回调，无需手动调用。  
   - **适用场景**：适合监视某个对象属性的变化，如监听 `UIScrollView` 的 `contentOffset` 变化。  
   - **示例**：
     ```swift
     class Observer: NSObject {
         var objectToObserve: SomeObject
         init(object: SomeObject) {
             self.objectToObserve = object
             objectToObserve.addObserver(self, forKeyPath: "someProperty", options: [.new, .old], context: nil)
         }
         override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
             print("值发生变化: \(change?[.newKey])")
         }
     }
     ```

2. **NotificationCenter（通知中心）**  
   - **特点**：一种一对多的全局广播机制，任何对象都可以监听通知，发送方和接收方相互独立。  
   - **适用场景**：适用于无直接关系的对象之间的消息传递，如全局事件（如用户登录成功的通知）。  
   - **示例**：
     ```swift
     NotificationCenter.default.addObserver(self, selector: #selector(receiveNotification(_:)), name: .someEvent, object: nil)
     
     NotificationCenter.default.post(name: .someEvent, object: nil)
     ```

3. **Delegate（代理模式）**  
   - **特点**：一种一对一的通信方式，常用于对象之间的定向回调，要求双方建立直接的关系。  
   - **适用场景**：适用于需要明确指定回调对象的情况，如 `UITableView` 通过 `delegate` 回调 `cellForRowAt` 方法。  
   - **示例**：
     ```swift
     protocol CustomDelegate: AnyObject {
         func didUpdateData(_ data: String)
     }

     class Sender {
         weak var delegate: CustomDelegate?
         func update() {
             delegate?.didUpdateData("更新数据")
         }
     }
     ```

**总结**  
| 机制 | 通信关系 | 适用场景 | 适合的示例 |
|------|------|------|------|
| **KVO** | 一对一 | 监听对象属性变化 | 监听 `UIScrollView` 滚动 |
| **NotificationCenter** | 一对多 | 广播全局事件 | 用户登录、退出通知 |
| **Delegate** | 一对一 | 组件之间回调 | `UITableViewDelegate` |

👉 **选择合适的方式**：  
- **如果是监听某个对象的属性变化** ➝ **KVO**  
- **如果是多个对象需要接收消息** ➝ **NotificationCenter**  
- **如果是两个对象间的直接通信** ➝ **Delegate**

### 10. 多线程数据竞争
**问题：** 你在 Swift 中如何安全地处理多线程数据竞争问题？
- 使用 actor 进行线程安全的数据管理，还可以使用 DispatchQueue（串行队列）来同步访问共享资源，或者使用 NSLock、Semaphore 等锁机制来控制并发访问。此外，@MainActor 也可以确保代码在主线程执行，避免 UI 相关的竞争问题。

## 实战问题（考察项目经验和解决问题的能力）

### 11. 启动速度优化
**问题：** 你有没有遇到过 iOS 应用启动速度慢的问题？是如何优化的？
- 将可异步执行的任务放到后台处理。  

优化 iOS 应用启动速度的方法包括：  

1. **减少 `AppDelegate` 和 `SceneDelegate` 的阻塞操作**：  
   - 延迟初始化不必要的对象  
   - 避免在 `didFinishLaunchingWithOptions` 中执行复杂计算  

2. **优化动态库加载**：  
   - 减少 `dyld` 需要加载的动态库数量  
   - 合并多个小的动态库  

3. **减少启动时的 I/O 操作**：  
   - 只加载必要的数据，避免启动时读取大文件  
   - 使用 `NSUserDefaults` 代替数据库查询  

4. **优化 Swift 代码**：  
   - 避免大量使用 `lazy var`，它们在首次访问时初始化可能影响性能  
   - 使用 `@preconcurrency` 让一些对象提前初始化  

5. **使用 Instruments 分析**：  
   - 使用 Xcode 的 **Time Profiler** 和 **App Launch** 工具检查启动耗时  

如果遇到具体的启动慢问题，可以用 Instruments 来找出真正的瓶颈，再针对性优化。

### 12. 崩溃分析
**问题：** 你如何做 App 崩溃分析？线上崩溃如何排查？
- 线上崩溃分析一般依赖 **崩溃日志** 和 **用户反馈**。可以使用 **Crashlytics、Bugly 或 Xcode Organizer** 收集崩溃信息，并 **上传符号表（dSYM）** 解析崩溃堆栈。然后按 **崩溃频率、影响范围、触发场景** 分类，使用 **测试设备或 Xcode 复现**，结合 **线程分析、异常断点和 Instruments** 深入排查。对于偶发性崩溃，可增加日志埋点，或让用户提供详细操作步骤，最终找到并修复问题。  

**详细解释崩溃排查流程**  
1. **收集崩溃日志**  
   - 使用 **Crashlytics、Bugly、Sentry** 等第三方工具收集崩溃数据  
   - Xcode Organizer 也能查看 TestFlight 用户的崩溃记录  
   - 确保上传 **dSYM 符号表**，否则日志无法解析  

2. **分析崩溃日志**  
   - 查找 **崩溃线程**，定位具体的代码位置  
   - 判断是 **空指针、数组越界、死锁、内存泄漏** 还是其他类型的崩溃  
   - 关注 **iOS 版本、设备型号**，找出是否是特定机型/系统问题  

3. **复现和调试**  
   - 通过 **日志埋点** 记录关键数据（如参数、线程信息）  
   - 使用 **Xcode 断点和 Instruments** 找出性能问题或竞态条件  
   - 在不同网络、设备、系统版本上测试  

4. **解决崩溃并监控效果**  
   - 修复代码后发布新版本，并在 Crashlytics 等平台监控是否仍然存在类似问题  
   - 增加异常保护，如 **非空判断、捕获异常、线程安全优化**  

**示例：崩溃定位案例**  
- **崩溃日志**：某 App 在 iOS 17 设备上 `main queue` 触发 `EXC_BAD_ACCESS`  
- **排查过程**：查找堆栈，发现 `UITableView reloadData` 期间访问了已释放的对象  
- **解决方案**：使用 `weak self` 避免循环引用，并增加对象生命周期管理  

这个流程可以帮助你高效排查线上崩溃，减少用户流失！

### 13. 业务优化经验
**问题：** 介绍一个你曾经优化过的 iOS 业务逻辑或者技术方案，优化后带来了哪些改进？
- 你还可以说自己在项目中如何发现性能瓶颈（比如使用 Instruments、Xcode Performance 监测）。
- 也可以举例说 如何减少 API 请求的次数，或者 如何优化本地缓存，这也是常见的优化点。
- 使用 MVP 模式 让 UI 和业务逻辑解耦，适用于 同一 UI 需要支持不同交互逻辑 的场景，如 不同用户角色、不同模式（编辑/查看）、不同业务分支。

### 14. 模块化架构设计 组件
**问题：** 你如何设计一个 iOS 模块化架构？在团队开发中如何保证代码的高可维护性？
- 在 iOS 项目中设计模块化架构时，我通常会根据业务需求和项目规模来拆分模块，遵循 **高内聚低耦合** 的原则，确保各模块之间的职责清晰，减少模块间的依赖，从而提升开发效率和代码的可维护性。具体设计思路如下：

 **1. 模块化架构层次划分：**

1. **基础层（基础模块）：**  
   存放一些公共的、跨模块共享的代码，如：网络请求（`NetworkManager`）、公共工具类、扩展方法、日志记录等。这些模块是独立且高复用的，任何模块都可以依赖它们。

2. **UI 层（公共 UI 组件）：**  
   包含各种 UI 组件，例如：自定义控件、通用视图、常见的弹框、列表展示、动画等。UI 组件模块只专注于界面展示，避免和业务逻辑混杂。

3. **中间层（业务相关模块）：**  
   比如用户信息模块、认证模块、路由模块（`Router`）、数据存储模块等。这些模块处理具体的业务逻辑，但不会直接和 UI 界面耦合。每个模块都可以独立开发和测试。

4. **上层业务模块（具体功能模块）：**  
   根据项目的需求，将业务模块划分成多个较小的功能模块，如订单管理、支付模块、用户中心等。每个业务模块仅依赖基础层和公共 UI 层，确保功能的独立性。

**2. 模块间解耦：**

- **依赖注入（DI）：**  
   采用依赖注入的方式，模块之间通过接口（protocol）而非直接引用来进行通信，避免了模块间的紧耦合。例如，`ViewController` 和 `Presenter` 之间通过协议交互，不依赖具体实现。

- **模块化路由（Router）：**  
   引入统一的路由管理器（`Router`），用来管理页面跳转和视图控制器的创建，减少了界面层的耦合度，且易于扩展。

- **协议与抽象：**  
   为了增加模块之间的灵活性，我们通常会在接口层使用协议而不是直接引用具体类。这允许不同模块根据不同场景替换实现，且不影响其他模块。

**3. 保证高可维护性的策略：**

1. **清晰的职责分离：**  
   每个模块的职责必须清晰，遵循单一职责原则（SRP）。通过合理的划分模块，避免同一个模块承担过多责任。

2. **模块化测试：**  
   各模块的功能可以单独进行单元测试，确保每个模块的功能在独立状态下能够正确运行，从而提高系统的稳定性。

3. **版本控制与发布：**  
   使用子模块或 CocoaPods 进行模块管理，确保每个模块能够独立版本管理，提升团队开发效率。当某一模块发生变化时，其他模块能够及时得到更新和回归测试。

4. **模块独立性：**  
   避免某个模块对其他模块的修改影响过大。尽量减少直接依赖，使用协议、委托、闭包等方式减少模块间的耦合度。

5. **文档与规范：**  
   为每个模块写清楚接口文档，并规定编码规范，确保每个开发人员能够高效协作，减少无谓的沟通成本。

**4. 具体的技术栈和工具：**
- **CocoaPods / Carthage / Swift Package Manager：** 用于管理第三方依赖和模块化架构的依赖。
- **Swift Protocols + Dependency Injection：** 用于确保模块间解耦，并方便单元测试。
- **Storyboard + XIB / SwiftUI：** UI 层的设计，可以通过分模块的方式管理每个视图控制器。

### **📌 组件化与 CTMediator 解耦方案**  

在 iOS **组件化开发** 中，通常需要解决 **模块间的解耦问题**。CTMediator 是一种 **基于目标-动作（Target-Action）模式** 的路由方案，可以动态调用不同组件的功能，而不直接依赖它们，从而实现 **模块解耦**。  

**1️⃣ 为什么使用 CTMediator？**
在组件化开发中，**各业务模块（如登录、用户中心、支付等）应当独立，不应直接互相引用**。  
如果 **A 模块需要调用 B 模块**：
- **❌ 直接 `import BModule`**：会增加 **强依赖**，导致**编译依赖关系复杂**。
- **✅ 通过 CTMediator**：A **不直接依赖 B**，而是通过 **中介者（CTMediator）** 进行调用，避免直接依赖关系。

**2️⃣ CTMediator 组件化架构**
CTMediator 主要采用 **Target-Action 方式** 进行解耦：
1. **每个组件** 提供 `Target_xxx` 类，并在其中定义可供外部调用的方法（Action）。
2. **通过 CTMediator** 调用 `Target_xxx` 提供的方法，而不是直接 `import` 目标组件。
3. **组件可以单独运行**，且业务间低耦合，提升模块复用性。

**📌 组件通信示意图**
```
App (CTMediator)
 ├──> A 组件 (Target_A)
 ├──> B 组件 (Target_B)
 ├──> C 组件 (Target_C)
```
**3️⃣ CTMediator 使用步骤**
**🔹 1. 组件内部创建 Target_xxx**
**每个组件都需要一个 `Target_xxx` 类**，用于暴露可调用方法。

**示例：`BModule` 组件**
```swift
import UIKit

@objc class Target_BModule: NSObject {
    
    @objc func Action_BViewController(_ params: [String: Any]) -> UIViewController {
        let vc = BViewController()
        vc.param = params["info"] as? String
        return vc
    }
}
```

**🔹 2. 使用 CTMediator 进行路由调用**
**A 组件想调用 B 组件的 `BViewController`**：
```swift
import CTMediator

let params: [String: Any] = ["info": "从A传来的参数"]
if let viewController = CTMediator.sharedInstance()?.performTarget("BModule", action: "BViewController", params: params, shouldCacheTarget: false) as? UIViewController {
    navigationController?.pushViewController(viewController, animated: true)
}
```
- `"BModule"` 👉 `Target_BModule`（去掉 `Target_`）。
- `"BViewController"` 👉 `Action_BViewController`（去掉 `Action_`）。
- **无须 import `BModule`**，仅通过字符串动态查找，完全解耦。

**🔹 3. 远程调用（支持 URL 路由）**
CTMediator 还支持 **远程调用**，可用于 **H5 调用 Native 组件**：
```swift
let url = "app://BModule/BViewController?info=来自H5的参数"
CTMediator.sharedInstance()?.performAction(withUrl: URL(string: url)!, completion: { result in
    print("回调结果: \(result)")
})
```

**4️⃣ CTMediator 组件化架构的优点**
✅ **低耦合**：各业务模块互不依赖，提升代码可维护性。  
✅ **可动态扩展**：新增模块只需新增 `Target_xxx` 类，无须改动其他模块。  
✅ **支持远程调用**：支持 `URL Scheme` 调用组件，方便 **H5/Native 交互**。  
✅ **独立运行**：每个模块都可作为 **独立 App 运行**，提升开发效率。  

**🎯 总结**
- **CTMediator 通过 Target-Action 方式，避免 `import` 其他模块，从而实现组件解耦**。
- **使用 `CTMediator.sharedInstance()?.performTarget(action:)` 方式调用目标组件**。
- **支持远程调用（URL Scheme），适用于 H5 调用 Native 组件**。

### 15. 依赖管理
**问题：** 你如何管理第三方依赖库？如何处理 Pod 或 SPM 的版本冲突？
- 使用工具（如 CocoaPods、SPM、Carthage）管理依赖，并且通过版本锁定（如 `Podfile.lock` 或 `Package.resolved`）来确保依赖一致性。
- 通过 `tag` 管理自定义库的版本，避免引入不稳定的版本。
- 对于版本冲突，采取版本范围、手动调整依赖版本、使用 `post_install` 钩子等方式来解决。
- 定期检查过时依赖，及时更新，保持依赖库的健康。

## Swift & 高级技术

### 16. Codable 机制
**问题：** Codable 是如何工作的？如何处理 Codable 解析失败的问题？
- Codable 通过 Encodable 和 Decodable 协议简化了对象与 JSON 数据之间的转换。
- 如果解析失败，可以通过自定义 decode 方法、使用 try? 和 do-catch 语句来处理解析错误。
- 使用 CodingKeys 可以在 JSON 键和模型属性名不一致时进行映射。

### 17. 响应式编程框架对比
**问题：** Combine 和 RxSwift 有什么异同？你更推荐使用哪种？

**Combine 和 RxSwift 的异同对比及推荐使用场景**  
RxSwift 和 Combine 都是响应式编程框架，能够处理异步事件流，但它们在 API 设计、功能特性、错误处理、线程管理等方面存在明显区别。以下是它们的主要异同，并结合实际场景推荐如何选择。  

**1. 主要区别**
 **① 线程管理（Schedulers）**
- **Combine**  
  - 通过 `DispatchQueue` 和 `RunLoop` 进行线程调度，提供 `subscribe(on:)` 指定数据在哪个队列处理，`receive(on:)` 指定在哪个队列接收数据。  
  - 线程控制能力较基础，不如 RxSwift 细粒度。  

- **RxSwift**  
  - 提供更丰富的调度器，如 `MainScheduler`（主线程）、`ConcurrentDispatchQueueScheduler`（GCD 线程池）、`OperationQueueScheduler`（基于 NSOperationQueue）。  
  - `observe(on:)` 指定流的下游线程，`subscribe(on:)` 指定流的上游线程，灵活性更高。  

✅ **结论**：RxSwift 在复杂的多线程调度上更灵活，Combine 适用于一般 UI 绑定和轻量级任务。  

**② 错误处理（Error Handling）**
- **Combine**  
  - 使用 `Failure` 泛型强制处理错误，流一旦发生错误 (`failure`)，就会终止，无法恢复。  
  - 提供 `catch(_:)` 处理错误，`retry(_:)` 进行重试。  

- **RxSwift**  
  - 通过 `onError` 事件处理错误，并提供 `onErrorResumeNext(_:)` 允许流在发生错误后继续运行。  
  - 也支持 `catchErrorJustReturn(_:)` 设置默认值，`retry(_:)` 进行重试。  

✅ **结论**：RxSwift 的错误恢复能力更强，而 Combine 遇到错误后流会直接终止，适用于更严格的错误处理需求。  

**③ 操作符（Operators）**
- **Combine**  
  - 操作符较少，核心基于 `Publisher` 和 `Subscriber`，提供 `map`、`flatMap`、`filter`、`combineLatest` 等基础功能。  
  - 不支持 RxSwift 的 `amb`、`withLatestFrom` 等高级操作符，复杂需求需手动实现。  

- **RxSwift**  
  - 提供完整的 Rx 操作符集合，支持 `merge`、`flatMapLatest`、`combineLatest`、`debounce`、`throttle`、`withLatestFrom`、`buffer` 等多种组合操作。  
  - 适用于复杂的数据流转换，如多个数据源合并、节流、防抖等。  

✅ **结论**：RxSwift 在数据流处理上更强大，适合高频交互需求，Combine 适用于一般 UI 绑定。  

---

**④ 内存管理（Memory Management）**
- **Combine**  
  - 使用 `AnyCancellable` 进行订阅管理，必须手动存储到变量中，否则订阅会立即释放。  
  - `store(in:)` 方法可以将多个 `AnyCancellable` 存入集合，便于批量管理。  

- **RxSwift**  
  - 依赖 `DisposeBag` 进行资源回收，所有订阅存入 `DisposeBag` 后会自动销毁。  
  - 需要注意 `weak self` 以避免循环引用导致的内存泄漏。  

✅ **结论**：Combine 的 `AnyCancellable` 更直观，RxSwift 的 `DisposeBag` 适用于批量管理订阅。  

**⑤ 背压（Backpressure）处理**
- **Combine**  
  - 提供 `buffer(size:options:)` 控制数据流速率，但功能有限。  
  - `throttle` 和 `debounce` 可减少数据量，但缺乏 RxSwift 的 `buffer` 和 `window` 等高级操作符。  

- **RxSwift**  
  - 提供 `buffer(timeSpan:count:scheduler:)`、`window` 等完整的背压处理方案，适用于高吞吐量场景，如 WebSocket、传感器数据流。  

✅ **结论**：RxSwift 在高频数据流和背压处理上更强，Combine 适合普通 UI 绑定。  

**⑥ Swift 版本 & 生态支持**
- **Combine**  
  - 由 Apple 官方推出，仅支持 iOS 13+，生态较新，与 SwiftUI 深度集成。  
  - 未来会持续更新，但目前操作符较少，部分场景需要手动实现。  

- **RxSwift**  
  - 支持 iOS 9+，适用于老项目，社区活跃，有大量开源库（如 RxCocoa、RxRelay）支持 UIKit 和 SwiftUI。  
  - 更适合需要跨平台（如 macOS、tvOS）或 UIKit 项目。  

✅ **结论**：Combine 更适合 SwiftUI + iOS 13+ 项目，RxSwift 更适合老项目迁移和复杂业务场景。  

**2. 推荐使用场景**
| 需求 | Combine | RxSwift |
|------|---------|---------|
| **SwiftUI 项目** | ✅ 推荐 | ❌ 不推荐 |
| **iOS 13+ 新项目** | ✅ 推荐 | ✅ 可选 |
| **iOS 9-12 兼容** | ❌ 不支持 | ✅ 推荐 |
| **复杂的异步数据流** | ❌ 操作符较少 | ✅ 丰富的操作符 |
| **高级错误处理** | ❌ 遇错终止 | ✅ 可恢复错误 |
| **复杂多线程控制** | ❌ 线程调度有限 | ✅ 线程调度灵活 |
| **老项目过渡** | ❌ 需重构 | ✅ 可无缝使用 |
| **官方支持** | ✅ Apple 官方 | ❌ 第三方库 |

**3. 总结**
- **使用 Combine**：如果项目基于 SwiftUI，或是 iOS 13+ 的新项目，推荐使用 Combine，官方支持且与 Swift 生态融合紧密。  
- **使用 RxSwift**：如果需要支持老版本 iOS，或对异步数据流有高要求（如复杂操作符、线程调度、错误恢复），RxSwift 仍然是更好的选择。  

**如果你的项目是 UIKit + RxSwift，没必要切换到 Combine；但如果是全新 SwiftUI 项目，Combine 是更自然的选择。**

### 18. 异步编程方案对比
**问题：** async/await 相比 GCD 有什么优势？哪些场景更适合 async/await？

**async/await 相比 GCD 的优势**  
1. **代码更清晰、可读性更高**  
   - `async/await` 采用同步代码的写法，让异步代码更直观，避免了 GCD 中的嵌套回调（回调地狱）。  
2. **自动管理线程切换**  
   - `async/await` 由 Swift 运行时自动管理任务的调度，避免手动切换 `DispatchQueue.main.async {}` 这样的代码。  
3. **更好的错误处理**  
   - `async/await` 可以结合 `try/catch` 进行**同步风格的异常处理**，而 GCD 需要通过回调或者 Result 处理错误。  
4. **任务结构化管理**  
   - `async/await` 允许使用 `Task` 进行任务管理，支持**任务取消**、**优先级控制**等，而 GCD 的 `DispatchQueue.async` 不能直接取消任务。  
5. **提升性能**  
   - `async/await` 采用**协作式多任务**，线程调度更高效，而 GCD 可能会创建不必要的线程，增加 CPU 负担。  

**更适合 async/await 的场景**
1. **多个异步任务需要顺序执行**（避免 GCD 回调嵌套）  
   ```swift
   async func fetchData() async -> String {
       return "数据加载完成"
   }
   
   async func processData() async -> String {
       let data = await fetchData()
       return "处理后的 \(data)"
   }
   ```
2. **需要错误处理的异步任务**（比 GCD 更优雅）  
   ```swift
   async func loadData() async throws -> String {
       if Bool.random() {
           throw URLError(.badServerResponse)
       }
       return "数据成功加载"
   }
   
   Task {
       do {
           let data = try await loadData()
           print(data)
       } catch {
           print("发生错误: \(error)")
       }
   }
   ```
3. **异步任务需要取消**（GCD 任务无法直接取消）  
   ```swift
   let task = Task {
       await fetchData()
   }
   task.cancel()  // 取消任务
   ```
4. **SwiftUI 结合 `@MainActor` 更新 UI**  
   ```swift
   @MainActor
   func updateUI() { ... }
   Task {
       await updateUI()  // 确保 UI 更新在主线程
   }
   ```

**总结**
✅ **如果是新项目，建议优先使用 `async/await`**，它更易读、更易维护，并且支持任务取消和错误处理。  
✅ **如果是老项目使用 GCD，可以逐步迁移到 `async/await`**，提高代码质量和可维护性。

### 19. SwiftUI 状态管理
**问题：** 在 SwiftUI 中，@State、@Binding、@ObservedObject、@EnvironmentObject 有什么区别？

`@State` 用于管理**本地私有状态**，`@Binding` 用于**父子视图之间的双向绑定**，`@ObservedObject` 用于**监听可观察对象的变化**，`@EnvironmentObject` 用于**在整个视图层级中共享对象**。  

**详细解释：**
- **`@State`**：适用于**简单值类型**（如 `Int`、`String`、`Bool`）的状态管理，变量值变化时 SwiftUI 会重新渲染视图，仅限当前视图使用。
- **`@Binding`**：用于**父子视图之间的状态共享**，子视图可以通过 `@Binding` 修改父视图的 `@State`，但不会持有数据本身。
- **`@ObservedObject`**：用于引用类型的对象（遵循 `ObservableObject` 协议），可以监听对象的变化，并在发生变更时刷新视图，适合管理复杂数据。
- **`@EnvironmentObject`**：类似 `@ObservedObject`，但适用于**全局状态管理**，可以在多个视图间传递，不需要手动传递参数，但使用前需要 `.environmentObject()` 注入对象。

**适用场景：**
- `@State` 适合**简单局部状态**，例如按钮的选中状态。
- `@Binding` 适合**父子组件共享数据**，如 Slider 的当前值。
- `@ObservedObject` 适合**需要在多个视图间传递的对象**，但仍需手动传递实例。
- `@EnvironmentObject` 适合**全局共享数据**，如用户信息或主题设置。

### 20. DiffableDataSource
**问题：** 如何使用 DiffableDataSource 优化列表数据源的管理？

`DiffableDataSource` 通过 `NSDiffableDataSourceSnapshot` 实现高效、自动化的数据变更管理，简化 `UITableView` 和 `UICollectionView` 的数据更新，避免手动调用 `reloadData()`。  

**详细解释**
`DiffableDataSource`（`UICollectionViewDiffableDataSource` 和 `UITableViewDiffableDataSource`）是 Apple 在 iOS 13 引入的新数据源 API，相比传统 `UITableViewDataSource` / `UICollectionViewDataSource`，它提供了更**高效、安全**的数据管理方式。  

**核心概念**
1. **`DiffableDataSource` 取代传统数据源**
   - 传统 `UITableViewDataSource` 需要手动管理 `cellForRowAt` 和 `numberOfRows`，而 `DiffableDataSource` 通过快照 (`NSDiffableDataSourceSnapshot`) 直接管理数据，并自动计算 UI 变化。

2. **通过 `NSDiffableDataSourceSnapshot` 进行数据更新**
   - 传统方式更新数据需要 `reloadData()`，可能导致 UI 闪烁，而 `DiffableDataSource` 通过 **快照（Snapshot）** 自动计算增删改差异，实现**流畅动画更新**。

**示例代码**
```swift
class ViewController: UIViewController {
    enum Section { case main }
    
    var tableView: UITableView!
    var dataSource: UITableViewDiffableDataSource<Section, String>!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        tableView = UITableView(frame: view.bounds, style: .plain)
        view.addSubview(tableView)
        
        // 配置 DiffableDataSource
        dataSource = UITableViewDiffableDataSource<Section, String>(tableView: tableView) { tableView, indexPath, item in
            let cell = tableView.dequeueReusableCell(withIdentifier: "Cell") ?? UITableViewCell(style: .default, reuseIdentifier: "Cell")
            cell.textLabel?.text = item
            return cell
        }
        tableView.dataSource = dataSource
        
        // 初始化数据
        updateSnapshot(items: ["Apple", "Banana", "Cherry"])
    }
    
    func updateSnapshot(items: [String]) {
        var snapshot = NSDiffableDataSourceSnapshot<Section, String>()
        snapshot.appendSections([.main])
        snapshot.appendItems(items)
        dataSource.apply(snapshot, animatingDifferences: true)
    }
}
```

**优势**
- **自动计算增删改，避免 `reloadData()`**
- **性能优化**：只更新变化的部分，提高列表滚动流畅度
- **代码更简洁**：无需维护 `indexPath` 计算逻辑

**适用场景**
- 需要频繁**更新列表**（如消息列表、动态刷新）
- 复杂的数据管理（如**多 Section、多层级结构**）
- 需要高性能**流畅动画**（如 iOS 大型数据列表）

在现代 iOS 开发中，`DiffableDataSource` 是推荐的列表管理方式，尤其适用于 **UITableView 和 UICollectionView** 需要动态更新数据的场景。

## 项目经验相关

### 实战追问示例
如果是更偏实战的面试，我也可以根据你的项目经验，围绕你实际做过的事情进行深入追问，比如：

#### **你如何在 TalkHire 这个项目中优化数据流和 UI 交互的性能？**
是的，我在 iOS 开发过程中遇到过多个复杂的性能优化问题，其中一个典型的案例是 **复杂 UI 页面导致的性能瓶颈**。  

**问题背景**  
在某个招聘类 App 开发中，我们的 **职位详情页面** 结构非常复杂，包含 **嵌套的 UITableView + 多层 UIStackView + 大量的图片和动态内容**。用户滑动时会明显感受到卡顿，特别是在低端设备上，CPU 和内存占用飙升，影响了用户体验。  

**分析过程**  
我首先使用 **Instruments（Time Profiler、Memory Leaks、GPU Frame Capture）** 进行分析，发现以下几个主要问题：  
1. **UIStackView 嵌套过深**，导致 `layoutSubviews` 频繁调用，计算成本高。  
2. **图片加载和解码占用大量 CPU**，同时部分图片未正确使用缓存，导致重复加载。  
3. **Cell 复用机制未完全优化**，一些动态内容导致 Cell 频繁创建，影响滚动流畅度。  
4. **主线程阻塞**，某些数据处理逻辑（如 JSON 解析、文本计算）直接放在主线程执行。  

**优化方案**  
针对这些问题，我采取了如下优化措施：  
1. **减少 UIStackView 嵌套**  
   - 通过 **改用 CALayer 直接绘制部分背景**，减少 UIView 层级，提高布局计算效率。  
   - 使用 **手写 Auto Layout 约束** 代替部分 UIStackView，提高布局性能。  

2. **优化图片加载**  
   - 使用 **SDWebImage + WebP 格式** 代替 PNG/JPEG，减少解码开销。  
   - 通过 **压缩图片尺寸** 并使用 `imageWithRenderingMode(.alwaysOriginal)` 避免 iOS 额外的渲染消耗。  
   - **开启预加载**，提前加载即将进入屏幕的图片，减少滚动时的卡顿。  

3. **优化 Cell 复用**  
   - 避免 `cellForRowAt` 里执行额外的计算逻辑，提前计算好 Cell 高度（使用 **UITableView 的 estimatedRowHeight**）。  
   - 采用 **异步加载数据**，并在 `willDisplayCell` 时动态填充内容，避免一次性加载过多数据。  

4. **优化主线程负担**  
   - **JSON 解析、文本排版计算、数据预处理** 迁移到子线程（使用 `DispatchQueue.global().async`）。  
   - 在主线程 **只做 UI 渲染相关操作**，确保流畅度。  

**优化效果**  
经过优化后，我们的页面 **CPU 占用降低了约 40%**，**帧率由 30FPS 提升到 55-60FPS**，用户滑动时的流畅度明显提升，低端设备也能较好运行。这次优化大幅提升了用户体验，同时也为后续项目提供了可复用的性能优化方案。  

**总结**：iOS 性能优化需要从 **布局优化、图片优化、数据处理优化、线程优化** 等多个维度入手，善用 **Instruments 工具** 分析瓶颈，有针对性地优化，才能实现流畅的用户体验。

#### **你的 Swift 项目从 OC 迁移时，遇到了哪些坑？怎么解决的？**
在将 **Objective-C（OC）项目迁移到 Swift** 时，会遇到以下常见问题，并对应提供解决方案：  

**1. 头文件和桥接问题**
**问题**：Swift 不能直接使用 `.h` 头文件中的 OC 代码，需要桥接。  
**解决方案**：  
- 在 Swift 项目中，创建 **`Bridging-Header.h`**，然后在其中 `#import` OC 头文件。  
- 在 OC 代码中，使用 `@import` 或 `#import <Module/Module.h>`，避免 `#import "SomeFile.h"` 造成循环引用。  
- 如果 Swift 代码需要被 OC 调用，使用 `@objc` 或 `@objcMembers` 公开给 OC。

**2. Nullability（空值）处理**
**问题**：OC 允许 `nil` 但 Swift 需要显式 `Optional` 处理。  
**解决方案**：  
- 在 OC 代码的属性和方法参数中，使用 **`nonnull` / `nullable`** 修饰符：
  ```objc
  @property (nonatomic, strong, nullable) NSString *name;
  ```
- 在 Swift 中要小心 **隐式解包 Optional（!）**，尽量使用 `if let` 或 `guard let` 进行安全解包。

**3. 宏定义替换**
**问题**：OC 的 `#define` 宏定义在 Swift 中不能直接使用。  
**解决方案**：  
- 用 `let` 或 `enum` 替代：
  ```swift
  // OC: #define kMaxCount 10
  let kMaxCount = 10
  ```
- 对于字符串宏，改用 `static let`：
  ```swift
  struct Constants {
      static let apiBaseURL = "https://api.example.com"
  }
  ```

**4. KVO 监听方式不同**
**问题**：OC 的 `addObserver:forKeyPath:` 方式不适用于 Swift。  
**解决方案**：  
- 使用 **`@objc dynamic`** 让 Swift 兼容 KVO：
  ```swift
  class MyClass: NSObject {
      @objc dynamic var name: String = ""
  }
  ```
- 或者**使用 Combine** 监听：
  ```swift
  myObject.publisher(for: \.name)
      .sink { newValue in
          print("Name changed to \(newValue)")
      }
  ```

**5. GCD 和异步调用的区别**
**问题**：OC 使用 `dispatch_async`，Swift 推荐用 `async/await`。  
**解决方案**：
  ```swift
  // OC:
  dispatch_async(dispatch_get_main_queue(), ^{
      NSLog(@"Hello from main queue");
  });

  // Swift:
  Task { @MainActor in
      print("Hello from main queue")
  }
  ```

**6. `SEL` 选择子和方法调用**
**问题**：OC 用 `SEL` 调用方法，Swift 需要 `#selector`。  
**解决方案**：
  ```swift
  // OC:
  [self performSelector:@selector(doSomething)];

  // Swift:
  self.perform(#selector(doSomething))
  ```

**7. Category 迁移到 Extension**
**问题**：OC 的 `Category` 不能直接转换成 Swift 扩展。  
**解决方案**：
- 如果 Category 只是扩展功能，直接改用 **Swift `extension`**：
  ```swift
  extension UIView {
      func addBorder() {
          self.layer.borderWidth = 1
      }
  }
  ```
- 如果 Category 需要添加存储属性，需要使用**关联对象（Associated Object）**。

**8. Block 与 Closure 兼容问题**
**问题**：OC `Block` 不能直接与 Swift `Closure` 兼容。  
**解决方案**：
- 在 OC 代码中，尽量使用 `typedef` 定义 Block：
  ```objc
  typedef void (^CompletionBlock)(NSString * _Nonnull result);
  ```
- 在 Swift 中使用 `@escaping` 处理：
  ```swift
  func fetchData(completion: @escaping (String) -> Void) {
      completion("Success")
  }
  ```

**9. `id` 类型替换**
**问题**：OC `id` 类型在 Swift 中缺少具体类型信息。  
**解决方案**：
- **尽量避免 AnyObject**，改用具体类型：
  ```swift
  // OC: id obj;
  var obj: Any  // Swift 需要明确类型
  ```

**10. `NS_ASSUME_NONNULL_BEGIN` / `END` 的影响**
**问题**：OC 代码如果使用 `NS_ASSUME_NONNULL_BEGIN`，Swift 可能误判 `nil` 处理。  
**解决方案**：
- 在 Swift 中显式使用 `Optional` 处理可空值：
  ```swift
  var name: String? = someOCObject.name
  ```

**总结**
Swift 迁移 OC 项目时，需要特别关注：
1. **桥接头文件**：使用 `Bridging-Header.h` 或 `@objc` 进行兼容。
2. **Nullability 处理**：用 `nullable` / `nonnull` 修饰符避免 `nil` 崩溃。
3. **GCD 与异步调用**：推荐用 `async/await` 取代 `dispatch_async`。
4. **Block 与 Closure**：Block 需要 `@escaping` 兼容。
5. **Category 替换为 Extension**：扩展功能用 `extension`，存储属性用 `objc_setAssociatedObject`。

