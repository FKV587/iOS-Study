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

#### 主要区别

| 特性 | struct | class |
|------|---------|--------|
| 类型 | 值类型 | 引用类型 |
| 存储位置 | 栈（大数据量可能在堆上） | 堆 |
| 传递方式 | 值拷贝（每次赋值创建新副本） | 引用传递（多个引用指向同一对象） |
| 继承 | 不支持 | 支持 |
| let 限制 | 不能修改属性 | var 属性可修改 |
| deinit 方法 | 无 | 有 |
| 线程安全 | 安全（不会共享状态） | 可能不安全（需要同步） |

#### 详细说明

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

#### 适用场景
- struct 适用于轻量级数据模型，如 CGPoint、CGRect、URL 等。
- class 适用于需要共享状态、管理生命周期的对象，如 UIViewController、NSObject 子类等。

### 3. GCD 和 NSOperation 的区别
**问题：** 你如何理解 GCD 和 NSOperation，它们有什么区别？什么时候用 GCD，什么时候用 NSOperationQueue？

**答案：**

#### GCD 和 NSOperation 的区别

| 特性 | GCD | NSOperation |
|------|-----|-------------|
| API 级别 | C 语言底层 API，轻量高效 | Objective-C / Swift 封装，面向对象 |
| 任务管理 | 直接提交任务到队列，不支持取消、依赖、优先级 | 任务可管理，可设置依赖、取消、优先级 |
| 执行方式 | 同步（sync）/异步（async），支持并行或串行队列 | 封装成 NSOperation 对象，添加到 NSOperationQueue |
| 线程控制 | 不能手动暂停、取消任务 | 任务可以暂停、取消 |
| 并发控制 | 依赖于队列类型（串行/并行） | 可控制最大并发数 |

#### 使用场景

##### 使用 GCD 的情况
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

##### 使用 NSOperationQueue 的情况
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

#### 总结
- GCD 更底层、更高效，适合简单异步任务，但无法取消、管理依赖。
- NSOperation 更高级、更灵活，适合复杂任务管理（依赖、取消、优先级）。

### 4. RunLoop 的作用
**问题：** 解释 RunLoop 的作用，它在 iOS 开发中的应用有哪些？

**答案：**

#### RunLoop 的基本概念
RunLoop 是 iOS 应用程序中的一个重要机制，它负责管理和调度线程的工作。每个线程都有一个对应的 RunLoop，主线程的 RunLoop 是自动创建和运行的，而子线程的 RunLoop 需要手动创建和运行。

#### RunLoop 的主要作用

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

#### RunLoop 的工作原理

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

#### RunLoop 在 iOS 开发中的应用

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

#### RunLoop 的运行模式

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

#### 注意事项

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

**答案：**

#### 事件传递和响应链的基本概念

1. **事件传递（Hit-Testing）**
   - 从上到下寻找目标视图
   - 从 UIApplication 开始，经过 UIWindow，最终找到最合适的目标视图
   - 类似于水滴从高处滴落的过程

2. **事件响应链（Responder Chain）**
   - 从下到上寻找可以处理事件的对象
   - 从目标视图开始，沿着父视图、控制器、窗口、应用程序逐级向上
   - 类似于抛球向上传递的过程

#### 事件传递机制（Hit-Testing）

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

#### 事件响应链机制（Responder Chain）

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

#### 常见应用场景

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

#### 注意事项

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

### 7. 列表性能优化
**问题：** 你如何优化 UITableView/UICollectionView 的滚动性能？

### 8. Copy-On-Write 机制
**问题：** 介绍下 Swift 中的 Copy-On-Write 机制，它如何影响 Array 的性能？

### 9. 通信机制对比
**问题：** KVO、NotificationCenter 和 Delegate 三者的区别，分别适用于什么场景？

### 10. 多线程数据竞争
**问题：** 你在 Swift 中如何安全地处理多线程数据竞争问题？

## 实战问题（考察项目经验和解决问题的能力）

### 11. 启动速度优化
**问题：** 你有没有遇到过 iOS 应用启动速度慢的问题？是如何优化的？

### 12. 崩溃分析
**问题：** 你如何做 App 崩溃分析？线上崩溃如何排查？

### 13. 业务优化经验
**问题：** 介绍一个你曾经优化过的 iOS 业务逻辑或者技术方案，优化后带来了哪些改进？

### 14. 模块化架构设计
**问题：** 你如何设计一个 iOS 模块化架构？在团队开发中如何保证代码的高可维护性？

### 15. 依赖管理
**问题：** 你如何管理第三方依赖库？如何处理 Pod 或 SPM 的版本冲突？

## Swift & 高级技术

### 16. Codable 机制
**问题：** Codable 是如何工作的？如何处理 Codable 解析失败的问题？

### 17. 响应式编程框架对比
**问题：** Combine 和 RxSwift 有什么异同？你更推荐使用哪种？

### 18. 异步编程方案对比
**问题：** async/await 相比 GCD 有什么优势？哪些场景更适合 async/await？

### 19. SwiftUI 状态管理
**问题：** 在 SwiftUI 中，@State、@Binding、@ObservedObject、@EnvironmentObject 有什么区别？

### 20. DiffableDataSource
**问题：** 如何使用 DiffableDataSource 优化列表数据源的管理？

## 项目经验相关

### 实战追问示例
如果是更偏实战的面试，我也可以根据你的项目经验，围绕你实际做过的事情进行深入追问，比如：

- 你如何在 TalkHire 这个项目中优化数据流和 UI 交互的性能？
- 你的 Swift 项目从 OC 迁移时，遇到了哪些坑？怎么解决的？