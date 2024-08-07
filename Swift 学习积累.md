
##### 在Swift结构体中实现写时复制

* 使用 isKnownUniquelyReferenced 方法检查一个类的实例是不是唯一的引用，如果是，我们就不需要对结构体实例进行复制，如果不是，说明对象被不同的结构体共享，这时对它进行更改就需要进行复制
* 在Swift中采用的优化方式叫做写时复制技术，简单的说就是，只有当一个结构体发生了写入行为时才会有复制行为。具体的做法就是，在结构体内部用一个引用类型来存储实际的数据，在不进行写入操作的普通传递过程中，都是将内部的reference的应用计数+1，在进行写入操作时，对内部的reference做一次copy操作用来存储新的数据，防止和之前的reference产生意外的数据共享。

#####  如何获取当前代码的函数名和行号

* #file 用于获取当前文件文件名
* #line 用于获取当前行号
* #column 用于获取当前列编号
* #function 用于获取当前函数名

##### required
required 是 Swift 中用于类的初始化方法的一个修饰符。它的主要作用是确保子类也必须实现这个初始化方法。这在涉及到协议的初始化方法时尤为重要，因为协议中的初始化方法无法被自动继承。

##### inout

* 使用值类型传递参数，直接修改值类型的值

```
func swap( a: inout Int, b: inout Int) {
    let temp = a
    a = b
    b = temp
}
var a = 1
var b = 2
print(a, b)// 1 2
swap(a: &a, b: &b)
print(a, b)// 2 1
```

##### 下面的代码会不会崩溃，说出原因

```
var mutableArray = [1,2,3]
for _ in mutableArray {
  mutableArray.removeLast()
}
```
在每次迭代时，for-in 循环会遍历最初确定的元素。尽管数组在每次迭代中被修改，但循环依然会按照原始的长度（3 次迭代）进行。这是因为 for-in 在开始时决定了要遍历的元素，而不是在每次迭代时重新计算。

##### some
函数返回值 是协议的时候可以直接使用some关键字标记该协议，译器可以根据返回值进行类型推断得到具体类型，但是不能是多个不同类型否则编译器报错

# 底层机制与高级特性
### Swift 面试需要的知识点条目

#### Swift 语言基础
1. **基本语法**
2. **数据类型**
3. **控制流**
4. **函数和闭包**
5. **面向对象编程**
6. **协议与扩展**
7. **错误处理**

## Swift 内存模型

#### 自动引用计数（ARC）的原理和工作机制
- **ARC**：Swift 使用 ARC 来管理应用程序的内存。每当创建一个类实例时，ARC 会自动跟踪并管理该实例的引用次数。
- **引用计数增加**：当有新的引用指向实例时，引用计数增加。
- **引用计数减少**：当引用不再指向实例时，引用计数减少。
- **内存释放**：当引用计数降到零时，ARC 自动释放该实例占用的内存。

#### 内存布局（Stack vs Heap）
- **Stack（栈）**：
  - 用于存储局部变量和函数调用。
  - 具有较快的分配和释放速度。
  - 内存由系统自动管理，生命周期由作用域决定。
- **Heap（堆）**：
  - 用于存储类实例和动态分配的内存。
  - 分配和释放速度较慢，需要手动管理（通过 ARC）。
  - 生命周期由程序控制，通过引用计数管理。

#### 内存管理的优化策略
- **弱引用和无主引用**：
  - 使用 `weak` 和 `unowned` 关键字来避免强引用循环。
  - `weak`：引用不增加引用计数，引用对象被销毁时自动置为 `nil`。
  - `unowned`：引用不增加引用计数，但引用对象被销毁时不置为 `nil`，容易导致野指针错误。
- **延迟初始化**：
  - 使用 `lazy` 关键字延迟属性的初始化，优化内存使用。
- **合适的数据结构**：
  - 选择合适的数据结构以减少内存占用，如使用 `Set` 替代 `Array` 进行查找操作。

#### 内存泄漏和工具检测
- **内存泄漏**：
  - 发生在对象的引用计数无法降到零，从而无法释放内存。
  - 常见原因是强引用循环。
- **工具检测**：
  - **Xcode Instruments**：使用 Instruments 中的 "Leaks" 和 "Allocations" 工具来检测内存泄漏和监视内存使用。
  - **Xcode Memory Graph**：在调试时使用 Xcode 的 Memory Graph 工具查看应用的内存图，识别强引用循环和其他内存问题。

## Swift 编译器与性能

#### Swift 编译器的优化选项
- **编译优化级别**：
  - **`-Onone`**：无优化，主要用于开发和调试。编译速度快，生成的代码便于调试。
  - **`-O`**：基本优化，平衡编译速度和代码性能。适用于日常开发。
  - **`-Ounchecked`**：高性能优化，忽略安全检查。适用于性能关键的生产代码，但可能导致运行时错误。
  - **`-Osize`**：优化代码尺寸，适用于需要减小应用体积的场景。

- **Link-Time Optimization (LTO)**：
  - 在链接时进行跨模块优化，提高最终生成代码的性能。

- **Whole Module Optimization (WMO)**：
  - 整个模块的源代码一起进行优化，能够跨文件进行优化。

#### Swift 编译过程
1. **Lexing**：
   - 将源码转换为词法单元（tokens）。
2. **Parsing**：
   - 将词法单元解析为抽象语法树（AST）。
3. **Semantic Analysis**：
   - 语义分析，类型检查和确定变量作用域。
4. **SIL（Swift Intermediate Language）生成**：
   - 将 AST 转换为中间表示（SIL），便于优化和分析。
5. **SIL 优化**：
   - 对 SIL 进行优化，如内联、消除死代码等。
6. **LLVM IR 生成**：
   - 将优化后的 SIL 转换为 LLVM 中间表示（LLVM IR）。
7. **LLVM 优化**：
   - 对 LLVM IR 进行进一步优化，如循环优化等。
8. **机器代码生成**：
   - 将优化后的 LLVM IR 转换为目标机器代码。
9. **链接**：
   - 将生成的机器代码与系统库和其他模块链接，生成可执行文件。

#### 代码内联和性能优化
- **代码内联（Inlining）**：
  - 编译器将函数调用替换为函数体本身，减少函数调用的开销。
  - 提高了代码执行速度，但可能增加代码尺寸。

- **性能优化技巧**：
  - **避免不必要的对象创建**：减少对象创建频率，降低内存分配和垃圾回收的开销。
  - **使用值类型（struct）代替引用类型（class）**：值类型避免引用计数的开销，适用于小型数据结构。
  - **减少动态派发**：尽量使用静态派发（如 `final`、`private` 方法），减少运行时开销。
  - **高效使用集合类型**：根据实际需求选择合适的集合类型，如 `Set` 代替 `Array` 进行查找操作。
  - **Profile 和 Benchmark**：使用 Instruments 等工具进行性能分析，找出瓶颈并优化。

## Runtime 特性

#### Swift 中的类型信息（Type Metadata）
- **类型元数据（Type Metadata）**：
  - Swift 在运行时维护类型的相关信息，称为类型元数据。
  - 类型元数据包括类型的名称、大小、存储属性、方法表等信息。
  - 可以通过反射机制（`Mirror`）或标准库函数获取类型元数据。

#### Swift 中的具体派发机制

* 值类型总是会使用直接派发, 简单易懂
* 而协议和类的extension都会使用直接派发
* NSObject的extension会使用消息机制进行派发
* NSObject声明作用域里的函数都会使用函数表进行派发
* 协议里声明的, 并且带有默认实现的函数会使用函数表进行派发

1. **Direct Dispatch（直接派发）**
   - **适用场景**：
     - `struct` 和 `enum` 的方法
     - `final class` 的方法
     - `private` 和 `fileprivate` 作用域的方法
   - **实现方式**：
     - 编译时确定具体实现，直接调用函数地址。
     - 没有运行时查找开销，性能最佳。

2. **Table Dispatch（虚方法表派发）**
   - **适用场景**：
     - 非 `final` 类的方法
     - 默认类方法（不带 `@objc` 修饰符）
   - **实现方式**：
     - 在编译时为类生成虚方法表，包含所有虚方法的指针。
     - 在运行时通过虚方法表查找并调用具体实现。
     - 比直接派发开销稍大，但比消息发送开销小。

3. **Message Dispatch（消息发送派发）**
   - **适用场景**：
     - `@objc` 修饰的类和方法
     - Objective-C 兼容的类和方法
   - **实现方式**：
     - 使用 Objective-C 的消息传递机制，通过 `objc_msgSend` 函数在运行时查找并调用方法。
     - 动态性强，允许运行时改变方法实现，但开销较大。

理解 Swift 的派发机制对于优化性能和正确使用动态特性至关重要。在编写性能关键代码时，尽量使用静态派发；而在需要动态行为时，选择合适的派发机制。

#### Swift 的反射机制（Mirror）
- **反射机制**：
  - 反射允许在运行时检查类型、属性和方法。
  - `Mirror` 类型是 Swift 的反射 API，提供了对实例的运行时检查功能。

- **使用 `Mirror`**：
  - 可以通过 `Mirror(reflecting: instance)` 创建 `Mirror` 实例。
  - `Mirror` 提供属性和子元素的详细信息，可以用于调试、序列化等场景。
  ```swift
  let instance = MyClass()
  let mirror = Mirror(reflecting: instance)
  for child in mirror.children {
      print("Property name: \(child.label ?? ""), value: \(child.value)")
  }
  ```

#### 动态方法调用
- **使用 `perform`**：
  - 通过 `NSObject` 的 `perform` 方法动态调用 Objective-C 方法。
  - 适用于与 Objective-C 交互的场景。

- **使用 `Selector`**：
  - 通过 `Selector` 类型动态调用方法。
  - 主要用于 Objective-C 兼容的类和方法。
  ```swift
  class MyClass: NSObject {
      @objc func dynamicMethod() {
          print("Dynamic method called")
      }
  }

  let instance = MyClass()
  let selector = #selector(instance.dynamicMethod)
  instance.perform(selector)
  ```

- **函数指针**：
  - 通过函数指针调用方法，适用于高性能要求的场景。
  ```swift
  func myFunction() {
      print("Function called")
  }

  let functionPointer: () -> Void = myFunction
  functionPointer()
  ```

## Swift 协议和泛型实现

### Swift 协议（Protocol）的底层实现
1. **协议表（Protocol Witness Table，PWT）**：
   - 用于保存协议方法的实现。
   - 当一个类型遵循某个协议时，编译器会生成一个协议表，包含该类型对协议中所有方法的具体实现。
   - 通过协议表实现动态派发，允许不同类型遵循相同协议，提供多态性。

> 在 Swift 中，协议方法的派发机制取决于协议是如何被使用的。具体来说，可以分为以下几种情况：
> #### 1. 静态派发（Static Dispatch）
> 如果协议方法在编译时可以确定具体实现，则会使用静态派发。静态派发没有运行时开销，性能最佳。
> - **适用场景**：
>   - 使用泛型约束的协议方法。
>   - 编译时能够确定具体类型实现的情况。
>   
> - **示例**：
>   ```swift
>   protocol Greetable {
>       func greet()
>   }
>   struct Person: Greetable {
>       func greet() {
>           print("Hello!")
>       }
>   }
>   func greet<T: Greetable>(_ value: T) {
>       value.greet() // 静态派发
>   }
>   let person = Person()
>   greet(person)
>   ```
> ### 2. 表派发（Witness Table Dispatch）
> 表派发用于协议表（Protocol Witness Table, PWT）中存储的方法。通过协议表实现动态派发，允许不同类型遵循相同协议，提供多态性。
> - **适用场景**：
>   - 类型以具体类型遵循协议，但不使用 `@objc` 属性。
>   
> - **示例**：
>   ```swift
>   protocol Greetable {
>       func greet()
>   }
>   struct Person: Greetable {
>       func greet() {
>           print("Hello!")
>       }
>   }
>   let greetable: Greetable = Person()
>   greetable.greet() // 表派发
>   ```
> ### 3. 消息发送派发（Message Sending Dispatch）
> 消息发送派发用于 `@objc` 修饰的协议方法，通过 Objective-C 的运行时系统发送消息来实现动态派发。
> - **适用场景**：
>   - 使用 `@objc` 修饰的协议方法。
>   - Objective-C 兼容的类和方法。
>   
> - **示例**：
>   ```swift
>   @objc protocol Greetable {
>       func greet()
>   }
>   class Person: NSObject, Greetable {
>       func greet() {
>           print("Hello!")
>       }
>   }
>   let greetable: Greetable = Person()
>   greetable.greet() // 消息发送派发
>   ```
> #### 总结
> - **静态派发**：用于编译时可以确定具体实现的协议方法（例如，泛型约束）。
> - **表派发**：用于以具体类型遵循协议但不使用 `@objc` 修饰的方法，通过协议表实现动态派发。
> - **消息发送派发**：用于 `@objc` 修饰的协议方法，通过 Objective-C 的运行时系统实现动态派发。

2. **协议合规性检查**：
   - 在编译时检查类型是否符合协议要求，确保类型实现了协议中所有的方法和属性。
   - 协议的合规性检查在运行时也会通过协议表进行，确保类型动态地满足协议的要求。

#### 泛型的编译和运行时表现
1. **类型擦除（Type Erasure）**：
   - Swift类型擦除 [网页1](https://juejin.cn/post/7056411279946154014) [网页2](https://juejin.cn/post/7034484715541233694) 
   - Swift 通过类型擦除来处理泛型，在编译时将具体的泛型类型转换为一种通用的类型，以便在运行时使用。
   - 类型擦除允许在运行时处理不同类型的值，支持泛型的灵活性。

```swift
protocol Printer {
    associatedtype T
    var value: T { get }
    func print()
}

struct TypeBPrinter<T>: Printer {
    var value: T
    func print() {
        debugPrint("\(value)")
    }
}

func test() {
    test1(TypeBPrinter(value: "字符串"))
    test1(TypeBPrinter(value: 10))
}

func test1<T>(_ p: TypeBPrinter<T>) {
    p.print()
}

func test1<T>(_ p: inout TypeBPrinter<T>, value: T) {
    p.print()
    p.value = value
}

```

2. **专门化（Specialization）**：
   - Swift 编译器会对泛型函数和类型进行专门化处理，根据具体的类型生成优化的代码。
   - 专门化可以提高性能，减少运行时开销。

3. **类型参数传递**：
   - 在运行时，通过元类型（metatype）传递泛型的类型信息。
   - 这种机制允许泛型在运行时仍然保留类型信息，实现类型安全的操作。

#### 关联类型（Associated Types）和存在类型（Existential Types）
1. **关联类型（Associated Types）**：
   - 协议可以定义一个或多个关联类型，用于表示协议中的类型占位符。
   - 关联类型使得协议更加灵活，可以适应不同类型的实现。
   - 使用 `associatedtype` 关键字定义关联类型。
   ```swift
   protocol Container {
       associatedtype Item
       func append(_ item: Item)
       var count: Int { get }
       subscript(i: Int) -> Item { get }
   }
   ```

2. **存在类型（Existential Types）**：
   - 存在类型表示一种可以持有任何符合某协议的类型的值。
   - 使用协议类型作为类型标注，即表示存在类型。
   - 存在类型在运行时通过协议表实现动态派发，支持多态性。
   ```swift
   func printDescription(of value: CustomStringConvertible) {
       print(value.description)
   }
   ```

3. **区别与联系**：
   - 关联类型用于定义协议的类型占位符，使协议适应不同类型的实现，通常在编译时确定具体类型。
   - 存在类型用于表示任意符合协议的类型的值，通过协议表实现动态派发，支持运行时的多态性。

#### 总结

- Swift 的协议通过协议表实现动态派发，提供多态性和灵活性。
- 泛型通过类型擦除和专门化在编译时和运行时实现类型安全和性能优化。
- 关联类型和存在类型分别用于协议的类型占位符和运行时的多态性，实现灵活的类型处理。

## 函数式编程与 Swift

#### 函数式编程的底层实现
函数式编程是一种编程范式，强调使用函数来进行计算。它通常包含以下几个核心概念：

1. **纯函数**：函数的输出只依赖于输入参数，不依赖于外部状态，也不修改外部状态。
2. **高阶函数**：可以接受函数作为参数或返回函数。
3. **不可变性**：数据不可变，即创建后不能修改。
4. **函数组合**：通过组合小的函数来构建复杂的操作。

在 Swift 中，函数式编程的底层实现依赖于 Swift 的类型系统和编译器优化。Swift 提供了丰富的标准库支持函数式编程，例如 `map`、`filter` 和 `reduce` 等高阶函数。Swift 的函数是第一类公民，可以赋值给变量、作为参数传递和返回。

#### 高阶函数的实现原理
高阶函数是指可以接受其他函数作为参数或返回值的函数。Swift 中的高阶函数利用了函数类型的灵活性。以下是几个常用的高阶函数及其实现原理：

1. **map**：对集合中的每个元素应用一个函数，并返回一个新的集合。
    ```swift
    let numbers = [1, 2, 3, 4, 5]
    let doubled = numbers.map { $0 * 2 }
    // doubled: [2, 4, 6, 8, 10]
    ```

2. **filter**：根据条件过滤集合中的元素，返回符合条件的元素组成的集合。
    ```swift
    let numbers = [1, 2, 3, 4, 5]
    let evenNumbers = numbers.filter { $0 % 2 == 0 }
    // evenNumbers: [2, 4]
    ```

3. **reduce**：将集合中的所有元素合并成一个值。
    ```swift
    let numbers = [1, 2, 3, 4, 5]
    let sum = numbers.reduce(0) { $0 + $1 }
    // sum: 15
    ```

高阶函数的实现原理主要依赖于闭包（Closure）和泛型（Generic）。闭包捕获上下文环境中的变量，使得函数可以灵活地传递和组合。泛型使得高阶函数能够处理不同类型的集合和操作。

#### 闭包的捕获列表和内存管理
闭包是自包含的代码块，可以在代码中传递和调用。闭包可以捕获和存储其所在上下文中的常量和变量，称为捕获值。捕获列表用于明确地控制捕获的值。

1. **捕获列表**：在闭包表达式的前面，使用方括号包裹的捕获列表，可以指定捕获的变量和捕获方式（强引用、弱引用或无主引用）。
    ```swift
    class Example {
        var value = 0
        deinit { print("Example deinitialized") }
    }

    var example: Example? = Example()
    let closure = { [weak example] in
        print(example?.value ?? "no value")
    }
    example = nil
    closure()
    // 输出 "no value"
    ```

2. **内存管理**：Swift 使用自动引用计数（ARC）来管理内存。闭包和捕获的变量之间可能会形成强引用循环，导致内存泄漏。使用捕获列表中的弱引用（`weak`）或无主引用（`unowned`）可以避免这种情况。

3. **循环引用**：当闭包和捕获的对象互相引用时，会导致循环引用。
    ```swift
    class Example {
        var value = 0
        lazy var closure: () -> Void = { [unowned self] in
            print(self.value)
        }
    }
    ```

使用 `unowned` 表示捕获的引用不会变为 `nil`，但在对象被释放后访问捕获的引用会导致崩溃。使用 `weak` 表示捕获的引用可以变为 `nil`，需要处理可选类型。

#### 总结
Swift 的函数式编程特性通过高阶函数和闭包提供了强大的工具，使得代码更加简洁和模块化。通过了解底层实现原理，可以更好地掌握和优化这些特性，提高代码的性能和安全性。

6. **并发编程底层原理**：
   - Grand Central Dispatch (GCD) 的底层实现
   - Operation 和 OperationQueue 的底层机制
   - Swift Concurrency 的实现（如 Actor 模型、Async/Await 底层机制）

7. **Swift 语言特性**：
   - Swift 的编译期与运行期特性
   - Swift 的 ABI 稳定性
   - Swift 模块系统和跨模块优化

8. **Swift 访问控制**：
   - 访问控制的底层实现
   - 访问控制对编译和运行的影响

9. **Swift 异常处理**：
   - 错误处理机制的实现
   - 错误传播的底层实现

10. **Swift 中的 C 与 Objective-C 互操作性**：
    - Swift 与 C 代码的互操作
    - Swift 与 Objective-C 的互操作
    - Swift 中的桥接（Bridging）

11. **内存管理与优化工具**：
    - 使用 Instruments 进行内存分析
    - 使用 Address Sanitizer 检测内存错误
    - 使用 Static Analyzer 进行静态代码分析
