# Unity GC 系列教程完整版：从原理到优化与高级应用

## 第一篇：GC 基础概念与工作原理

### 1.1 什么是 GC (Garbage Collection)? 我们为何需要它?
在编程世界中，内存管理是一个永恒的话题。当我们创建变量、对象、数据结构时，它们都需要占用内存空间。程序运行结束后，或者这些变量、对象不再被需要时，它们所占用的内存就需要被释放，以便其他程序或对象可以使用。

手动管理内存是一项复杂且容易出错的任务。在 C++ 等语言中，开发者需要自己负责内存的分配 (`new`) 和释放 (`delete`)。如果忘记释放内存，就会导致 **内存泄漏 (Memory Leak)**，即程序占用的内存越来越多，最终可能导致系统崩溃或程序运行缓慢。反之，如果过早地释放了还在使用的内存，则可能导致 **悬空指针 (Dangling Pointer)**，引发程序崩溃或不可预测的行为。

为了解决这些手动内存管理带来的痛点，**垃圾回收 (Garbage Collection)** 技术应运而生。GC 的核心思想是：**自动识别并回收程序中不再使用的内存**。开发者不再需要手动编写内存释放的代码，而是由 GC 运行时系统 (在 C# 中是 .NET / Mono 运行时) 自动完成这项工作。

在 C# 和 Unity 中，我们主要处理的是 **托管内存 (Managed Memory)**。这意味着我们创建的大部分对象 (如类实例、字符串、数组等引用类型) 都由 .NET / Mono 运行时进行内存管理。而像直接操作底层硬件或文件句柄等少数情况，可能会涉及到 **非托管内存 (Unmanaged Memory)**，这部分内存通常需要我们手动管理或使用特定的 API 来释放。

**GC 的基本目标是：**
*   **自动化内存管理**：将开发者从繁琐的内存管理任务中解放出来。
*   **防止内存泄漏**：自动回收不再使用的内存，避免程序占用过多资源。
*   **提高程序稳定性**：减少因内存管理错误 (如野指针、二次释放) 导致的崩溃。

### 1.2 GC 的基本原理：可达性分析与常见算法
尽管 GC 的具体实现非常复杂，但其核心原理是基于 **可达性分析 (Reachability Analysis)**。简单来说，GC 会判断程序中的某个对象是否仍然“可达”，即从程序的“根”出发，是否还能找到并访问到这个对象。

#### 1.2.1 GC 根 (GC Roots)
要理解可达性分析，首先要明白什么是 **GC 根 (GC Roots)**。GC 根是程序中那些无论如何都不会被GC回收的对象。它们是GC判断其他对象是否“存活”的起点。常见的 GC 根包括：
*   **栈变量 (Stack Variables)**：当前正在执行的方法中的局部变量。
*   **静态变量 (Static Variables)**：全局可访问的变量，它们在程序整个生命周期内都可能存在。
*   **CPU 寄存器 (CPU Registers)**：CPU 内部存储数据的寄存器。
*   **JNI 引用 (JNI References)** (在 Java 等语言中，但在 Unity 的跨语言交互中也可能遇到类似概念)。
*   **其他特殊系统对象**：例如，由运行时或外部系统 (如操作系统、Unity 引擎本身) 直接引用的对象。

你可以将 GC 根想象成一棵树的树根，所有从这些树根延伸出来的枝干 (引用) 所连接到的叶子 (对象)，都是“活着的”对象。

#### 1.2.2 可达性分析算法
GC 在执行时，会从这些 GC 根开始，遍历所有它们直接或间接引用的对象。所有能够被遍历到的对象，都被认为是“可达的”，也就是“存活的”。而那些无法从任何 GC 根到达的对象，则被认为是“不可达的”，也就是“垃圾”，可以被回收。

#### 1.2.3 常见的 GC 算法简介
*   **标记-清除 (Mark-Sweep)**
    *   **标记 (Mark) 阶段**：从 GC 根开始，遍历所有可达对象，并将它们标记为“存活”。
    *   **清除 (Sweep) 阶段**：遍历整个堆内存，回收所有未被标记 (即不可达) 的对象所占用的内存空间。
    *   **缺点**：内存碎片 (Memory Fragmentation)，GC 暂停 (Stop-The-World)。
*   **复制 (Copying)**
    *   将堆内存分为两个区域：From 空间和 To 空间。GC 执行时，将所有存活对象从 From 空间复制到 To 空间。
    *   **优点**：没有内存碎片。
    *   **缺点**：内存利用率低 (需要两倍空间)，复制开销大。
*   **标记-整理 (Mark-Compact)**
    *   结合了标记-清除和复制算法的优点。标记后，将存活对象向堆的一端移动，紧密排列。
    *   **优点**：无碎片，利用率高。
    *   **缺点**：GC 暂停时间较长 (移动对象涉及内存拷贝和更新引用)。

#### 1.2.4 分代 GC (Generational GC)
现代 GC 算法通常采用 **分代 GC** 策略。基于观察：大部分对象生命周期很短；存活越久的对象，未来存活概率越大。
*   **新生代 (Gen 0)**：存放新创建的对象，通常采用复制算法，频率高，速度快。
*   **老年代 (Gen 1 / Gen 2)**：存放经过多次 GC 仍存活的对象，通常采用标记-整理或标记-清除算法。

### 1.3 Unity 中的 Mono GC
Unity 早期版本 (直到 2019 引入 Incremental GC 之前)，主要使用的是 **阻塞式 (Blocking) GC** (基于 Boehm GC)。这意味着当 GC 需要执行时，它会暂停所有游戏逻辑的执行 (即 **Stop-The-World**)，直到垃圾回收完成。

### 1.4 为什么 GC 会产生性能问题？
1.  **GC 暂停 (GC Pause / Stop-The-World)**：GC 会一次性完成所有工作，导致较长时间的暂停，引发掉帧。
2.  **GC Alloc (内存分配开销)**：频繁在堆上分配内存会触发 GC。GC Alloc 越多，GC 越频繁，卡顿越严重。

---

## 第二篇：Unity 中常见的 GC Alloc 场景与分析工具

### 2.1 什么是 GC Alloc?
**GC Alloc (Garbage Collection Allocation)**，直译就是“垃圾回收器内存分配”。它指的是在程序运行时，**向托管堆 (Managed Heap) 申请分配内存空间的过程**。
当你在 C# 中使用 `new` 关键字创建一个类的实例，或者创建一个数组、字符串等引用类型时，这些操作都会在托管堆上分配内存，从而产生 GC Alloc。

**为什么是问题？**
1.  **频繁的分配**：导致内存很快被填满，触发 GC。
2.  **触发 GC**：堆满时触发垃圾回收，导致卡顿。
3.  **内存碎片**：频繁分配和回收导致内存不连续。

### 2.2 常见的 GC Alloc 场景

#### 2.2.1 装箱 (Boxing)
装箱是指将 **值类型 (Value Types)** 隐式或显式地转换为 **引用类型 (Reference Types)** 的过程。
*   **原理**：当一个值类型 (如 `int`, `float`, `struct`) 需要被当作 `object` 类型或接口类型处理时，C# 运行时会在堆上创建一个临时的 `object` 对象，将值类型的数据复制进去。
*   **常见场景**：
    *   字符串拼接：`"Score: " + 100` (int 被装箱)。
    *   非泛型集合：`ArrayList` 使用 `object` 存储，存入 `int` 会装箱。
    *   `string.Format`：参数是 `object`，传入值类型会装箱。
    *   函数参数为 `object`：`Debug.Log(10)` (int 被装箱)。

#### 2.2.2 字符串操作 (String Operations)
`string` 是引用类型，且是 **不可变 (Immutable)** 的。
*   **字符串拼接 (`+`)**：每次拼接都会创建一个新的字符串对象。
    ```csharp
    // 产生大量垃圾
    string info = "Name: " + name + ", Level: " + level.ToString(); 
    ```
*   **其他方法**：`Split()`, `Substring()`, `Replace()` 都会返回新字符串，产生 Alloc。

#### 2.2.3 集合类型操作
*   `List<T>.ToArray()` / `ToList()`：创建新数组或列表。
*   `Dictionary<K,V>.Keys` / `.Values`：在旧版本 .NET/Mono 中，每次访问都会创建新集合对象 (新版本已优化，但 foreach 可能仍有开销)。
*   **foreach 循环**：在旧版本编译器中，foreach 可能会造成枚举器 (Enumerator) 的装箱。

#### 2.2.4 Unity API 调用
*   `GameObject.GetComponent<T>()`：通常是安全的，但返回数组的版本 `GetComponents<T>()` 会分配新数组。
*   `Mesh.vertices` / `Input.touches`：每次访问属性都会返回一个新的数组副本 (Property 返回数组通常是复制)。
*   `Physics.RaycastAll`：返回 `RaycastHit[]` 数组。

#### 2.2.5 匿名函数与 Lambda 表达式 (Closures)
当 Lambda 表达式捕获了 **外部作用域的变量** 时，编译器会生成一个 **闭包 (Closure)** 类。这个闭包对象是引用类型，并在堆上分配。
```csharp
int localScore = 100;
// 捕获了 localScore，产生 GC Alloc
Action act = () => Debug.Log(localScore); 
```

#### 2.2.6 LINQ
LINQ 非常方便，但大多数 LINQ 查询都在内部创建临时的集合、委托或枚举器，导致 **大量 GC Alloc**。在游戏热路径 (Update) 中应避免使用 LINQ。

### 2.3 分析工具：Unity Profiler
*   **CPU Usage 模块**：关注 **GC Alloc** 列。
*   **Hierarchy 视图**：按 GC Alloc 排序，找出哪行代码分配了内存。
*   **Deep Profile**：能精确定位到具体函数，但开销大。

---

## 第三篇：GC Alloc 优化技巧与实践（上）

### 3.1 减少装箱 (Boxing)
*   **避免字符串拼接**：使用 `StringBuilder`。
*   **泛型集合**：使用 `List<T>` 替代 `ArrayList`。
*   **重写 ToString**：为自定义 struct 重写 `ToString()` 以避免默认的反射实现（尽管 ToString 本身通常返回 string，但重写可控）。
*   **接口与结构体**：当结构体实现接口并以接口类型传递时，会发生装箱。尽量使用泛型约束 `where T : IMyInterface` 来避免。

### 3.2 优化字符串操作
*   **StringBuilder**：对于频繁修改的字符串，使用 `System.Text.StringBuilder`。
*   **缓存 StringBuilder**：不要每次都 `new StringBuilder()`，创建一个成员变量并复用 (`Clear()` 后重用)。
*   **字符串哈希**：比较字符串时，考虑使用 `Animator.StringToHash` 等 ID 比较。

### 3.3 优化集合类操作
*   **预分配容量**：`new List<int>(1000)`。避免 `Add` 时频繁扩容产生的数组拷贝。
*   **复用集合**：
    *   使用 `Clear()` 而不是 `new List()`。
*   **避免 LINQ**：手动写 `for` / `foreach` 循环。

### 3.4 对象池 (Object Pooling) - 核心策略
对象池是优化 GC Alloc 最重要、最有效的策略之一。
*   **原理**：预先创建一批对象，需要时从池中“借用” (`SetActive(true)`)，用完后“归还” (`SetActive(false)`)，而不是 `Destroy`。
*   **适用场景**：子弹、特效、UI 元素、敌人 NPC。
*   **Unity API**：Unity 2021+ 提供了官方的 `UnityEngine.Pool` 类。

### 3.5 缓存 GetComponent 结果
*   在 `Awake` 或 `Start` 中获取组件引用并缓存，不要在 `Update` 中频繁调用 `GetComponent`。

### 3.6 优化物理查询
*   使用 **NonAlloc** 版本的方法。
    *   ❌ `Physics.RaycastAll()` (返回新数组)
    *   ✅ `Physics.RaycastNonAlloc(ray, results, ...)` (传入预分配的数组，无 GC)

---

## 第四篇：GC Alloc 优化技巧与实践（下）与 GC 调优

### 4.1 深入理解和优化 foreach 循环
在早期的 Unity (Mono) 版本中，`foreach` 会导致枚举器装箱。现在的版本 (Unity 5.5+, C# 4+ 编译器) 大多修复了 `List<T>` 的 foreach 问题 (使用 struct 枚举器且采用鸭子类型避免装箱)。
*   **注意**：如果是通过接口 (`IEnumerable<T>`) 遍历，仍然会发生装箱。
*   **最佳实践**：对 `List<T>` 和数组直接 foreach 是安全的；对字典或复杂情况，或为了极致性能，可改用 `for` 循环。

### 4.2 LINQ 的替代方案
*   **手动实现**：使用 `for` 循环和 `List` 操作替代 `Where`, `Select`, `OrderBy`。虽然代码量增加，但零 GC。

### 4.3 避免匿名函数和 Lambda 捕获变量 (Closures)
*   如果 Lambda 表达式不捕获外部变量 (只使用参数)，编译器会缓存一个静态委托，不会每次分配。
*   如果捕获了局部变量 (闭包)，每次执行都会 `new` 一个闭包类实例。
*   **优化**：尽量避免在 `Update` 中创建捕获外部变量的 Lambda。

### 4.4 GC 调优：Incremental GC (增量式 GC)
Unity 2019.3+ 引入。
*   **原理**：将原本一次性完成的 GC 标记工作 (Stop-The-World)，分摊到多个帧中进行 (Time Slicing)。
*   **优点**：大幅减少单帧的最大卡顿时间 (Spikes)，游戏更平滑。
*   **缺点**：总的 GC 开销可能会略微增加 (因为需要写屏障 Write Barrier 来保证分步标记的正确性)；可能会降低整体吞吐量。
*   **设置**：Project Settings -> Player -> Configuration -> **Use Incremental GC**。

### 4.5 手动触发 GC (`System.GC.Collect()`)
*   **建议**：通常不要手动调用。
*   **例外场景**：在加载关卡、黑屏过渡、打开全屏 UI 等玩家不会操作的时机，手动调用一次，清理掉之前关卡的内存，避免在战斗中触发自动 GC。

### 4.6 理解堆内存 (Managed Heap)
*   **只增不减**：Mono 堆内存通常只会在不够用时扩张，但很难自动缩小返还给操作系统 (即使空闲)。
*   **策略**：控制堆内存的峰值。

---

## 第五篇：高级 GC 内核

### 5.1 ScriptableObject：数据存储与共享
*   **作用**：将数据从 MonoBehaviour 中分离出来，作为资产文件存储。
*   **GC 优势**：避免每个 GameObject 实例都复制一份配置数据。所有实例共享同一个 ScriptableObject 引用，极大降低内存占用。

### 5.2 IL2CPP 对 GC 的影响
*   **IL2CPP**：将 C# (IL) 转换为 C++ 代码，再编译为本机代码。
*   **优势**：运行效率更高，结构体操作优化更好。虽然仍有 GC (IL2CPP 集成了一个高性能 GC)，但由于代码执行更快，GC 的压力相对减轻。

### 5.3 C# Job System 与 Burst Compiler
这是 Unity 高性能计算的核心 (DOTS 的一部分)。
*   **Native Container**：`NativeArray`, `NativeList` 等。
*   **非托管内存**：这些容器使用的是 **非托管内存 (Unmanaged Memory)**，完全 **不产生 GC Alloc**。
*   **手动管理**：必须手动调用 `.Dispose()` 释放，否则会内存泄漏 (Unity 编辑器会报错提示)。
*   **Burst Compiler**：将 C# Job 代码编译为高度优化的机器码 (利用 SIMD)，性能极高。

### 5.4 Addressables 与内存管理
*   **引用计数**：Addressables 系统内部使用引用计数来管理资源加载和卸载。
*   **优势**：避免重复加载同一资源；提供了更清晰的内存管理 API (`Release`)，减少资源导致的内存泄漏风险。

### 5.5 .NET GC 发展与 Unity 的演进
*   随着 Unity 升级 .NET 运行时 (CoreCLR 等)，未来将享受到更先进的 GC 算法 (如并发 GC、分代更优、LOH 优化等)。
*   **Value Types 推广**：Unity 可能会继续推广值类型 API，减少引用类型的使用。

---

### 总结
1.  **理解原理**：知道 GC 为什么会卡顿 (STW) 以及什么操作会分配内存 (Alloc)。
2.  **善用工具**：使用 Profiler 定位 GC Alloc 热点。
3.  **基础优化**：避免装箱，慎用字符串拼接，慎用 LINQ。
4.  **进阶优化**：使用 **对象池**，使用 **NonAlloc** API，缓存引用。
5.  **高级技术**：利用 **Job System** 和 **NativeContainer** 彻底避开托管堆。
6.  **策略**：开启 **Incremental GC** 平滑帧率。

> **核心口诀**：
> 热点代码 (Update) **零分配**；
> 常用对象 **用池化**；
> 字符串拼接 **StringBuilder**；
> 物理检测 **NonAlloc**。