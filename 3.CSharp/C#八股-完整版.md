# C# 面试八股 — 完整知识体系手册

> 整合自 26 个专题 + 6 套面试真题，涵盖 C# 基础、CLR 原理、Unity 实战、性能优化、各厂面试风格。

---

# 第一部分：C# 基础与运行时

## 第一章 C# 与 .NET 基础

### 1.1 C# 与 .NET 是什么关系？

C# 是语言，.NET 是平台。C# 代码编译后在 .NET 运行时上运行。类比：JavaScript 和 Node.js / 浏览器的关系。

```
C# = 编程语言（语法、语义规范）
.NET = 运行时平台（CLR + BCL + 工具链）
```

- C# 由 Anders Hejlsberg 主导设计，2002 年随 .NET Framework 1.0 发布
- C# 是 ECMA/ISO 标准，理论上可以脱离 .NET 实现（如 Mono）
- 现代 .NET 支持 C#、F#、VB.NET 等多种语言，都编译成相同的 IL

**C# 和 Java 的区别**：C# 有更丰富的值类型系统（自定义 struct）、属性、LINQ、async/await 语法糖、unsafe 代码。两者都是托管语言，核心设计理念相似。

### 1.2 CLR 是什么？

CLR（Common Language Runtime）是 .NET 的运行时虚拟机，负责 IL 代码的 JIT 编译、GC、异常处理、线程管理和类型安全。相当于 Java 的 JVM。

| 职责 | 说明 |
|------|------|
| **JIT 编译** | 将 IL 编译成本机机器码 |
| **GC** | 自动内存管理，分代回收 |
| **类型安全** | 编译时 + 运行时双重类型检查 |
| **异常处理** | 结构化异常处理（SEH） |
| **线程管理** | 托管线程池、同步原语 |
| **安全性** | Code Access Security、验证 IL |

```
C# 源码 → Roslyn → IL + Metadata → CLR → JIT → 机器码 → 执行
```

### 1.3 CTS、CLS

| | CTS | CLS |
|------|------|------|
| **全称** | Common Type System | Common Language Specification |
| **作用** | 统一类型系统规范 | 跨语言互操作规范 |
| **范围** | 所有类型 | CTS 的子集 |

- CTS 保证 C# 的 `int` 和 VB.NET 的 `Integer` 都是 `System.Int32`
- CLS 要求避免 public 方法暴露 `uint`（C# 支持但 VB.NET 不支持）

### 1.4 BCL、FCL

- **BCL**（Base Class Library）：.NET 核心基础类库（System.*）
- **FCL**（Framework Class Library）：包括 BCL 在内的完整框架类库（包括 ASP.NET、WPF 等）

```
FCL
├── BCL ← 核心：System.String, System.IO, System.Threading, System.Linq
├── ASP.NET Core
├── Entity Framework Core
├── WPF / WinForms (Windows)
└── MAUI (跨平台 UI)
```

### 1.5 .NET Framework vs .NET Core vs .NET 5+

| | .NET Framework | .NET Core | .NET 5+ |
|------|---------|------|------|
| **跨平台** | ❌ Windows 专属 | ✅ | ✅ |
| **开源** | 部分 | ✅ MIT | ✅ MIT |
| **性能** | 基准 | 更快 | 持续优化 |
| **最新版本** | 4.8（停止） | 3.1（停止） | 9.0（2024.11） |

### 1.6 Mono 是什么？

Mono 是 .NET 的第三方开源跨平台实现，由 Xamarin 主导。Unity 早期使用 Mono 作为脚本运行时。包含 C# 编译器（mcs）、跨平台 CLR 实现、兼容 .NET Framework 的 BCL。

**Unity 的演进**：Unity 早期 → Mono（C# 2.0~6.0 子集）→ Unity 2017+ → IL2CPP + .NET Standard 2.0 → Unity 2021+ → .NET Standard 2.1

### 1.7 IL（MSIL/CIL）

IL 是 CPU 无关的栈式中间语言，所有 .NET 语言最终都编译成 IL。IL 使跨语言互操作成为可能。

```
C# 代码:  int Add(int a, int b) { return a + b; }
IL:       ldarg.1   → 将参数 a 压栈
          ldarg.2   → 将参数 b 压栈
          add       → 弹出两个值，相加后压栈
          ret       → 返回栈顶
```

- IL 是栈式指令，操作数通过栈传递
- IL 包含 Metadata（类型信息、方法签名等全部保留）
- IL 可被反编译：ILSpy、dnSpy、JetBrains dotPeek

### 1.8 JIT 与 AOT

**JIT（Just-In-Time）**：CLR 的即时编译器，在运行时将 IL 按需编译为本机机器码。方法是"首次调用时编译"，编译结果缓存，后续调用直接执行机器码。优势是利用运行时信息做动态优化（热点内联、去虚拟化），代价是首次调用有编译延迟。

| 模式 | 行为 |
|------|------|
| **Normal JIT** | 方法首次调用时编译，结果缓存 |
| **Econo JIT** | 编译后不缓存（已废弃） |
| **ReadyToRun (R2R)** | 发布时预编译大部分 IL，运行时只需少量 JIT |

**AOT / IL2CPP**：运行前将所有 IL 转为本机代码。IL2CPP 是 Unity 的 AOT 方案：IL → C++ → 本机代码。

```
标准 .NET:      IL → CLR (JIT 按需编译) → 机器码
IL2CPP:        IL → IL2CPP.exe (转 C++) → 平台编译器 → 本机代码
```

### 1.9 Unity 为什么使用 IL2CPP？

iOS 禁止 JIT（W^X 内存保护），IL2CPP 提前编译为机器码可上架 App Store。此外：性能提升 1.5-2x、跨平台统一、减少包体（Strip Engine Code）、安全性（IL 不复存在难反编译）。

| 维度 | Mono JIT | IL2CPP |
|------|---------|--------|
| iOS 兼容 | ❌ 不允许 JIT | ✅ 纯 AOT |
| 性能 | 基准 | 1.5-2x 提升 |
| 启动速度 | JIT 预热慢 | 快 |
| 动态代码生成 | ✅ System.Reflection.Emit | ❌ 不支持 |

---

## 第二章 C# 编译流程

### 2.1 完整编译流水线

C# 源码 → Roslyn 编译 → IL（CIL/MSIL）→ CLR 加载 Assembly → JIT 编译 → 本机机器码 → CPU 执行。

```
┌──────┐     ┌──────┐     ┌──────────┐     ┌──────┐     ┌──────┐
│ C#   │ ──→ │Roslyn│ ──→ │ IL +     │ ──→ │ CLR  │ ──→ │ CPU  │
│ 源码  │     │ 编译  │     │ Metadata │     │ JIT  │     │ 执行  │
└──────┘     └──────┘     └──────────┘     └──────┘     └──────┘
                                ↓
                          PE 文件(.dll/.exe)
                          托管模块 (Managed Module)
```

### 2.2 Roslyn 编译器

Roslyn 是 C#/VB.NET 的开源编译器。编译阶段：

```
源码 "int x = 1 + 2;"
├── 词法分析 (Lexer) → Token 流: [int] [x] [=] [1] [+] [2] [;]
├── 语法分析 (Parser) → 语法树 (Syntax Tree)
├── 语义分析 (Binder) → 符号绑定、类型检查
├── IL 生成 (Emitter) → IL + Metadata
└── 优化 (Optimizer) → 常量折叠等
```

Roslyn = "Compiler as a Service"，编译过程完全透明，.NET 工具链基座。IDE 智能提示、重构都直接调用 Roslyn 的 Code Analysis API。

### 2.3 IL 核心指令速查

| 指令 | 含义 |
|------|------|
| `ldarg.0` | 加载第0号参数 |
| `stloc.0` | 存入第0号局部变量 |
| `ldc.i4.5` | 加载整数常量5 |
| `newobj` | 创建对象（分配+构造） |
| `call` / `callvirt` | 静态调用 / 虚调用 |
| `box` / `unbox` | 装箱 / 拆箱 |
| `ret` | 返回 |

### 2.4 Metadata

Metadata 是"数据的数据"，描述了每个类型的名称、方法签名、属性、特性（Attribute）等。.NET 自描述文件的核心——PE 文件中 IL 和 Metadata 共存。

| 类别 | 内容 |
|------|------|
| **类型定义** | 类名、可见性、基类、接口 |
| **成员定义** | 方法、属性、字段、事件的签名 |
| **特性** | 所有 Attribute 信息 |
| **程序集引用** | 依赖哪些外部 Assembly |
| **资源** | 嵌入的资源文件 |

为什么 Metadata 重要？反射依赖 Metadata；跨语言互操作靠 Metadata；IDE 智能提示靠 Metadata；Trim/AOT 需要 Metadata 做静态分析。

### 2.5 Assembly 的结构

Assembly（程序集）= .dll / .exe 文件，是 .NET 的部署、版本控制、安全边界单元。

```
Assembly (.dll / .exe)
├── Manifest（清单）
│   ├── 程序集名称、版本、文化
│   ├── 依赖的外部程序集引用
│   └── 导出类型列表
├── Module（托管模块）
│   ├── IL 代码
│   ├── Metadata
│   └── 资源
└── 非托管资源（图片、文件等）
```

### 2.6 PE 文件

.NET 程序集本质上是 PE（Portable Executable）文件，和 Windows 的 .exe/.dll 同格式。

```
PE 文件布局
┌──────────────────┐
│  PE 头            │ ← Windows 加载器识别 (PE32/PE32+)
├──────────────────┤
│  CLR 头           │ ← CLR 识别入口 (IMAGE_COR20)
├──────────────────┤
│  Metadata        │
├──────────────────┤
│  IL 代码          │
├──────────────────┤
│  资源             │
└──────────────────┘
```

### 2.7 CLR 加载流程

1. 程序启动 / Assembly.Load() → 2. CLR 加载器定位 PE 文件 → 3. 验证 PE 签名 + CLR 头 → 4. 加载 Metadata 表 → 5. 创建内部运行时类型表示（EEClass/MethodTable）→ 6. 类型首次访问 → 类型初始化器执行 → JIT → 执行

### 2.8 JIT 编译过程

方法首次调用时：CLR 查找 MethodDesc → 未编译 → 触发 JIT → IL 编译为机器码（存入 Heap）→ 更新 MethodDesc 入口指向机器码地址 → 执行。后续调用直接执行已缓存的机器码。

JIT 优化包括：内联、常量传播、死代码消除、循环优化（利用运行时数据）。

### 2.9 ReadyToRun (R2R)

R2R 是"预编译大部分 IL"的折中方案。发布时用 crossgen 工具将 IL 预编译为本机代码，运行时 JIT 只需少量补充编译。

| | 标准 JIT | ReadyToRun | IL2CPP (AOT) |
|------|-----|------|------|
| 启动速度 | 慢（预热） | 快 | 最快 |
| 性能峰值 | 高（动态优化） | 较高 | 较高 |
| 动态生成 | ✅ | ✅（剩余 IL） | ❌ |
| 包体 | 小 | 大 | 大 |

### 2.10 IL2CPP 编译流程

```
IL (所有 .dll) → IL2CPP.exe（静态分析 + 优化）→ C++ 代码 → 平台编译器(clang/msvc/gcc) + IL2CPP Runtime → 本机机器码（无 IL 残留）
```

IL2CPP 优化：Strip Engine Code（剔除未使用引擎代码）、泛型共享（值类型各有实例，引用类型共享一份）、死代码消除、字符串 Lazy 初始化。

---

## 第三章 CLR 原理

### 3.1 CLR 核心组件

```
CLR
├── 类加载器           ├── JIT 编译器
├── GC                ├── 异常管理
├── 线程管理           ├── 安全引擎
```

### 3.2 CLR 七大核心职责

| 职责 | 说明 | 重要性 |
|------|------|--------|
| **类型加载** | 按需加载 Assembly，解析类型 | ★★★★ |
| **JIT 编译** | IL → 机器码，首次调用时触发 | ★★★★★ |
| **GC** | 分代回收、内存压缩 | ★★★★★ |
| **线程管理** | 托管线程池、同步原语 | ★★★ |
| **异常处理** | SEH 结构化异常处理 | ★★★ |
| **安全保障** | CAS（历史）、IL 验证 | ★★ |
| **互操作** | P/Invoke、COM 互操作 | ★★★ |

### 3.3 类型加载过程

类型加载是 CLR 在运行时按需加载类型数据的过程——不是启动时全部加载，而是首次访问类型时才加载。

**加载步骤**：
1. 定位 Assembly（已加载则跳过）→ 2. 解析类型 Metadata → 3. 创建 EEClass（内部类型表示）→ 4. 创建 MethodTable（虚方法分派表）→ 5. 分配静态字段内存 → 6. 检查静态构造函数是否需要执行 → 7. 类型就绪

**核心内部结构**：

| 结构 | 作用 |
|------|------|
| **EEClass** | 类型的完整描述（字段布局、GC 信息） |
| **MethodTable** | 虚方法分派表（vtable）+ 接口映射 |
| **MethodDesc** | 每个方法的描述（IL 地址、JIT 状态） |
| **FieldDesc** | 每个字段的描述 |

类型一般不会卸载单个类型。只有整个 Assembly 被卸载（AssemblyLoadContext.Unload）时，所有类型一起被回收。

### 3.4 AppDomain 与 AssemblyLoadContext

AppDomain 是 .NET Framework 的隔离单元——同一进程内多个 AppDomain 各自加载 Assembly，互不影响。.NET Core/5+ 中被 **AssemblyLoadContext** 替代，更轻量、没有序列化开销。

```csharp
// AssemblyLoadContext 的好处：
// - 支持热加载 / 热卸载插件
// - 无序列化跨域通信开销
// - ASP.NET Core 的每个应用用独立 ALC
```

### 3.5 Assembly Load 过程

```
Assembly.Load("MyLib")
↓
1. GAC（全局程序集缓存）—— .NET Framework 场景
2. 应用程序基础目录
3. 子目录（按程序集名称）
4. 自定义探针（AssemblyLoadContext.Resolving）
```

| 方法 | 行为 |
|------|------|
| `Assembly.Load(name)` | 按名称加载，走正常探测 |
| `Assembly.LoadFrom(path)` | 按路径加载，自动加载该目录下的依赖 |
| `Assembly.LoadFile(path)` | 按路径加载，不加载依赖（需手动处理） |

### 3.6 Reflection 在 CLR 中的工作原理

Reflection 是直接查询 CLR 内部 Metadata 的 API。`typeof(T)` 通过 RuntimeTypeHandle 直接定位 MethodTable；`Assembly.GetTypes()` 遍历 Metadata 表。不是"另建数据"，而是读取 CLR 已有的运行时结构。

```csharp
// typeof() 原理：RuntimeTypeHandle → MethodTable* → 返回 RuntimeType（几乎 O(1)，无额外分配）
Type t = typeof(List<int>);

// Assembly.GetTypes()：遍历 Metadata 表，每个类型包装为 RuntimeType
Type[] types = asm.GetTypes();
```

**反射的代价**：不是已有结构有开销，而是 `MethodInfo.Invoke` 要走额外的参数打包-解包-安全检查；每次 GetMethod/GetField 需要按名称查找 → 字符串比较。

### 3.7 类型初始化流程

静态字段在首次访问类型前初始化。CLR 保证"exactly once + 线程安全"。

```csharp
class Foo {
    static int x = 42;               // inline 初始化
    static readonly int y;           // 静态构造赋值
    static Foo() { y = 100; }        // 仅执行一次，线程安全
}
```

| 触发条件 | 是否触发 cctor |
|------|------|
| `Foo.x`（访问静态字段） | ✅ |
| `new Foo()`（首次创建实例） | ✅ |
| `typeof(Foo)` | ❌ 不触发（仅获取 Type 句柄） |
| `Foo[]`（创建数组） | ❌ |

### 3.8 CLR 生命周期

CLR 随托管进程启动而加载，一个进程只有一个 CLR 实例。生命周期：进程启动→加载 CLR DLL→初始化子系统→执行托管代码→进程退出→CLR 清理。CLR 退出时清理所有托管内存，但 `Environment.Exit()` 会跳过 Finalizer。

---

## 第四章 内存模型

### 4.1 C# 进程内存六大区

栈（Stack，值类型/引用指针）、托管堆（Managed Heap，引用对象，GC 管）、非托管堆（Native，手动释放）、静态区（static）、常量区、代码区。GC 只管辖托管堆。

```
高地址  ┌────────────────────┐
       │       栈 Stack       │ ← 值类型、引用地址；函数结束自动弹出
       │   (每线程独立, ~1MB)  │
       ├────────────────────┤
       │   托管堆 Managed Heap │ ← 引用类型对象；CLR 分配，GC 回收
       │  ┌ SOH (小对象堆) ┐  │
       │  │ Gen0/Gen1/Gen2│  │
       │  ├──────────────┤  │
       │  │ LOH (大对象)   │  │ ← ≥85000 字节
       │  └──────────────┘  │
       ├────────────────────┤
       │ 非托管堆 Native Heap │ ← C++/malloc；需手动释放 / IDisposable
       ├────────────────────┤
       │     静态区 Static    │ ← static 变量；程序生命周期
       ├────────────────────┤
       │     常量区           │ ← 字符串常量等；编译时确定
       ├────────────────────┤
       │     代码区           │ ← IL/机器码
低地址  └────────────────────┘
```

### 4.2 托管堆 vs 栈

| | 栈 (Stack) | 托管堆 (Managed Heap) |
|------|------|------|
| **存储内容** | 值类型、引用指针 | 引用类型对象 |
| **管理方式** | 编译器自动（出作用域弹出） | GC 回收 |
| **分配速度** | 极快（栈指针移动） | 快（移动 next 指针），有 GC 暂停 |
| **大小** | ~1 MB（线程） | GB 级别 |
| **GC 参与** | ❌ | ✅ |
| **碎片** | 无 | 压缩后无碎片 |

```csharp
void Example() {
    int x = 42;           // 栈：x 的值 42
    string s = "hello";   // 栈：s 的引用地址；堆："hello" 对象
    object[] arr = new object[100]; // 栈：arr 的引用；堆：object[100]
} // 栈弹出 x,s,arr；堆上的 "hello" 和 object[100] 等下 GC
```

### 4.3 栈帧布局（x64 调用约定）

栈存储局部值类型变量、引用类型变量的引用地址、方法参数、返回地址。每个线程有独立栈。

```
┌─── 上一个栈帧 ──┐
│  返回地址         │
│  参数 b (引用)    │
│  参数 a (值)      │
│  局部变量 c       │
│  局部变量 pos     │  ← 整个 struct 内联存储
│  局部变量 obj     │  ← 仅存储引用地址(8字节)
└────────────────┘
```

### 4.4 静态区

静态区存储所有 static 字段，生命周期 = 程序运行全程。CLR 按类型分配一块连续内存存该类型所有静态字段，在首次访问前初始化。静态字段不在 GC 堆上 → 不受 GC 根追踪，不会被 GC 回收（除非 Assembly Unload）。

### 4.5 LOH（Large Object Heap）

≥ 85000 字节的对象直接分配在 LOH。LOH 在第 2 代 GC 时回收，但默认**不压缩**（因为移动大对象太贵），导致 LOH 碎片风险。

| | SOH（小对象堆） | LOH（大对象堆） |
|------|------|------|
| **阈值** | < 85000 字节 | ≥ 85000 字节 |
| **分代** | Gen0/Gen1/Gen2 | 逻辑 Gen2 |
| **压缩** | ✅ 每次回收压缩 | ❌ 默认不压缩 |
| **碎片** | 无（压缩消除） | **有碎片风险** |

LOH 最佳实践：复用大对象而非反复分配+释放；大对象分解为小对象（如 `List<byte[]>` 而非一个巨型 `byte[]`）；使用 `ArrayPool<T>` 管理大缓冲区。

### 4.6 Pinned Object

Pinned Object 是"钉住"的对象，GC 压缩时不可移动。通过 `fixed` 语句或 `GCHandle.Alloc(obj, GCHandleType.Pinned)` 实现。用于与非托管代码交换指针。钉住的对象阻碍压缩 → 碎片 → 影响 GC 效率。

### 4.7 String Intern Pool

字符串池是 CLR 维护的哈希表，将相同内容的字符串常量复用为同一份堆内存。编译期字面量自动入池，运行时可通过 `string.Intern()` 手动入池。

```csharp
string a = "Hello";            // 自动入池
string b = "Hello";            // 复用池中的 "Hello"
Console.WriteLine(ReferenceEquals(a, b)); // True！同一个对象
```

- ✅ 减少重复字符串的内存占用
- ❌ Intern 的字符串不会释放（GC 不回收池中字符串）
- ❌ 过度 Intern 会撑大线程安全的哈希表，有锁开销

---

## 第五章 值类型 vs 引用类型

### 5.1 struct vs class 核心区别

| | struct（值类型） | class（引用类型） |
|------|------|------|
| **存储内容** | 值本身 | 引用（地址） |
| **默认位置** | 栈（局部） | 托管堆 |
| **赋值** | 复制整个数据 | 复制引用 |
| **继承** | ❌（但可实现接口） | ✅ |
| **继承自** | System.ValueType | System.Object |
| **默认值** | 不可为 null（Nullable 除外） | null |
| **GC** | 在栈上不产生 GC | 产生 GC |

```csharp
// 关键行为差异
var s1 = new PointStruct { X = 1, Y = 2 };
var s2 = s1;          // 完整复制！s2 是独立副本
s2.X = 100;
Console.WriteLine(s1.X); // 1 ← 不受影响

var c1 = new PointClass { X = 1, Y = 2 };
var c2 = c1;          // 复制引用！指向同一对象
c2.X = 100;
Console.WriteLine(c1.X); // 100 ← 被一起改了！
```

### 5.2 struct 到底在栈上？（重要纠正）

这是一个常见误解。准确说：**struct 存在哪里取决于它的声明位置**——局部变量的 struct 在栈上；作为 class 字段的 struct 随对象在堆上；数组中的 struct 在堆上。struct 本身不决定内存位置，声明上下文决定。

### 5.3 struct 什么时候上堆

① struct 作为 class 的字段→随对象在堆；② struct 装箱→在堆上创建副本；③ struct 存储在数组中→在堆；④ 捕获到闭包的 struct→在堆；⑤ 静态 struct 字段→在静态区。

### 5.4 装箱（Boxing）

装箱 = 值类型 → object/接口，在托管堆上分配新对象，复制值。GC 压力来源之一。装箱是隐式发生的，编译器不警告。

```csharp
int value = 42;
object obj = value;  // 装箱：托管堆分配内存 + 复制值

// 隐式装箱场景
IComparable cmp = 10;              // 装箱（值类型→接口）
string s = 42.ToString();          // 不装箱！（int 重写了 ToString）
string s2 = "" + 42;               // 装箱（拼接时调用 object.ToString）
var list = new ArrayList();        // 非泛型集合
list.Add(100);                     // 装箱！
```

### 5.5 拆箱（Unboxing）

拆箱 = object/接口 → 值类型。先检查类型匹配（不匹配抛异常），然后复制堆上的值回栈。拆箱本身比装箱轻量（无堆分配），但类型检查 + 复制有开销。

```csharp
object obj = 42;       // 装箱
int x = (int)obj;      // 拆箱：类型检查 → 复制值

// ❌ 经典错误
object obj = 42L;      // 装箱了 long
int x = (int)obj;      // InvalidCastException！long 不能拆成 int
// ✅ 正确
int x = (int)(long)obj;  // 先拆成 long → 再强制转 int
```

### 5.6 Nullable\<T\>

`Nullable<T>` 让值类型可以为 null。底层是 `Nullable<T> where T : struct` 的 struct，内部有 `bool hasValue` + `T value` 两个字段。`int?` 是 `Nullable<int>` 的语法糖。

```csharp
int? x = null;                    // hasValue = false
int? y = 42;                      // hasValue = true, value = 42
int z = y ?? 0;                   // 空值合并
int w = y!.Value;                 // 断言非 null

// 装箱的 Nullable 特殊处理
int? a = 42;
object obj = a;    // 装箱的是 int 42，不是 Nullable<int>！
int? b = null;
object obj2 = b;   // 返回 null（不装箱！）
```

### 5.7 readonly struct（C# 7.2+）

保证所有字段不可变，**彻底消除防御性复制**。当 struct 作为 `in` 参数或 `readonly` 字段访问时，编译器无需复制。

```csharp
readonly struct Point {
    public readonly int X;
    public readonly int Y;
    public Point(int x, int y) { X = x; Y = y; }
}
// in 参数 + readonly struct = 零拷贝
void Foo(in Point p) {
    Console.WriteLine(p.X);  // 编译器不产生防御性复制！
}
```

### 5.8 ref struct（C# 7.2+）

只允许在栈上存在，**禁止上堆**。不能作为 class 字段、不能装箱、不能存数组、不能被 Lambda 捕获。典型代表：`Span<T>`、`ReadOnlySpan<T>`。

```csharp
ref struct Span<T> {
    private readonly ref T _reference;  // ref 字段（C# 11+）
    private readonly int _length;
}
// 只能活在栈上，保证不被 GC 移动
```

限制：不能是 class 字段、不能装箱、不能放数组中、不能被闭包捕获、不能实现接口、不能是 async 方法的局部变量——原因都是"会到堆上"。

---

## 第六章 GC

### 6.1 为什么需要 GC？

手动内存管理（C++ new/delete）容易产生：内存泄漏、野指针、双重释放。GC 自动追踪对象可达性，回收不可达对象，消除整个类别的 bug。代价是 Stop-the-World 暂停和性能开销。

### 6.2 GC Roots

GC Roots 是 GC 标记的起点——所有从根可达的对象 = 存活，不可达 = 可回收。根包括：栈上的引用、静态字段、CPU 寄存器中的引用、GC Handles、finalization queue。

```csharp
static Player global;  // 静态根
void Foo() {
    var p = new Player();  // 栈上的 p 是根
    global = new Player(); // 静态根
} // p 出作用域，第一个 Player 不可达 → 可回收
```

### 6.3 Mark-Sweep-Compact 三阶段

1. **MARK（标记）**：暂停所有线程 → 从 GC Roots 出发 DFS/BFS → 标记所有可达对象
2. **SWEEP（清除）**：回收未标记对象的内存 → 若有 Finalizer，放入终结队列
3. **COMPACT（压缩）**：移动存活对象 → 消除碎片 → 更新引用地址

### 6.4 分代回收（Generational GC）

基于"绝大多数对象短命"的假设，将堆分三代。Gen0 最年轻，回收最频繁；Gen1 是缓冲；Gen2 是长期存活对象。

| 代 | 阈值(触发回收) | 回收频率 | 存活判定 |
|------|------|------|------|
| **Gen0** | ~256 KB | 最高 | 未回收 → 晋升 Gen1 |
| **Gen1** | ~2 MB | 中等 | 未回收 → 晋升 Gen2 |
| **Gen2** | ~10 MB | 最低 | 继续存活 |
| **LOH** | 每个对象 ≥85KB | 仅 Gen2 回收时 | 默认不压缩 |

为什么分代能提升性能？大多数对象在 Gen0 就死了→只扫描 Gen0 就够了→避免全堆扫描。

### 6.5 Finalizer（终结器）

Finalizer（析构函数 `~类名()`）在 GC 回收对象时执行，用于清理非托管资源。问题是调用时机不确定，且会让对象活得更久。

带 Finalizer 的对象生命周期：分配 → 不可达 → 放入终结队列 → Finalizer 线程执行 → 再次不可达 → 真正回收。至少两次 GC 才能死透。终结队列中的对象及其引用的对象都多活一轮。

### 6.6 IDisposable + using

`IDisposable` 提供确定性资源释放——调用方主动释放，不等 GC。`using` 语句保证 `Dispose()` 被调用（finally 块实现）。这是释放非托管资源的首选模式。

```csharp
public class FileWrapper : IDisposable {
    private FileStream _stream;
    private bool _disposed;

    public void Dispose() {
        Dispose(true);
        GC.SuppressFinalize(this);  // 已经手动清理，GC 不用再调 Finalizer
    }

    protected virtual void Dispose(bool disposing) {
        if (_disposed) return;
        if (disposing) { _stream?.Dispose(); }
        _disposed = true;
    }

    ~FileWrapper() => Dispose(false);  // 兜底
}

// using 语法糖
using var file = new FileWrapper("path");  // 离开作用域自动调 Dispose
```

为什么 `Dispose` 模式需要 `GC.SuppressFinalize`？已经手动释放，Finalizer 不需要再调一遍。同时减少对象在终结队列的额外存活时间。

### 6.7 GC.Collect 该不该手动调？

不该。GC 自己知道什么时候回收最好。手动 `GC.Collect()` 打断分代回收策略，可能让短命对象提前晋升到高代，导致更早触发全堆 GC。

极少数合理场景：场景切换后（加载完成、关闭主菜单）清理加载时产生的垃圾；内存 Benchmark 后确保完全回收后测内存。

### 6.8 Unity GC 特别提示

- 不要在 Update/LateUpdate/FixedUpdate 中 `new` 对象 → 对象池复用
- 用 `StringBuilder` 拼接字符串
- 用 `CompareTag` 代替 `gameObject.tag == "xxx"`
- 用 `List<T>` 代替 `ArrayList`

---

# 第二部分：面向对象与核心特性

## 第七章 引用与参数传递

### 7.1 值传递 vs 引用传递

C# 默认参数传递是**值传递**——值类型传值副本，引用类型传引用副本（但仍能改对象内容）。ref/out 才是**引用传递**——直接传变量本身，修改影响原变量。

```csharp
// 值传递：值类型 → 传值副本
void ModifyValue(int x) { x = 100; }
int a = 5;
ModifyValue(a);
Console.WriteLine(a); // 5 ← 不改变！

// 值传递：引用类型 → 传引用副本（但仍能改对象内容）
void ModifyRef(Player p) { p.name = "New"; }
var p = new Player { name = "Old" };
ModifyRef(p);
Console.WriteLine(p.name); // "New" ← 对象内容改了

// 但赋值不会影响原引用
void ReassignRef(Player p) { p = new Player { name = "Other" }; }
var p2 = new Player { name = "Old" };
ReassignRef(p2);
Console.WriteLine(p2.name); // "Old" ← 引用没变！
```

为什么说"C# 默认传参是值传递"？因为即使传引用类型，也是传引用地址的副本。`p = new Player()` 改变的是副本，不影响外层变量。

### 7.2 ref

ref 是真正的引用传递——方法内对参数的任何修改（重新赋值）都反映到原变量。调用前变量必须初始化。

```csharp
void Swap(ref int a, ref int b) { int t = a; a = b; b = t; }
int x = 10, y = 20;
Swap(ref x, ref y);  // x=20, y=10

// ref 引用类型——重新赋值也影响外部
void Reassign(ref Player p) { p = new Player { name = "New" }; }
var p = new Player { name = "Old" };
Reassign(ref p);
Console.WriteLine(p.name); // "New" ← 引用也变了！
```

### 7.3 out

out 也是引用传递，但调用前不需要初始化。方法内必须赋值。语义："我不给你值，你给我结果"。用于多返回值场景。C# 7+ 支持内联声明 `out` 变量。

```csharp
void Divide(int a, int b, out int quotient, out int remainder) {
    quotient = a / b;
    remainder = a % b;
}
Divide(10, 3, out int q, out int r);  // q=3, r=1

// TryParse 是 out 的经典使用
if (int.TryParse("123", out int result)) { }
```

### 7.4 ref vs out vs in

| | ref | out | in (C# 7.2+) |
|------|------|------|------|
| **调用前初始化** | 必须 | 不必 | 必须 |
| **方法内赋值** | 可选 | 必须 | **不可**（只读） |
| **语义** | 读写 | 只写 | 只读 |
| **目的** | 修改 | 输出多个结果 | 避免大 struct 拷贝 |

```csharp
// in：大 struct 零拷贝只读传递
void PrintPoint(in Point p) { Console.WriteLine($"{p.X}, {p.Y}"); }
```

### 7.5 params

`params` 让方法接受可变数量的参数，调用时直接传逗号分隔的值，编译器自动装箱为数组。

```csharp
void Log(params string[] messages) {
    foreach (var m in messages) Console.WriteLine(m);
}
Log("Error", "File not found", "Retry");  // 编译器生成 new string[]{...}
```

### 7.6 ref return / ref local（C# 7.0+）

ref return 让方法返回变量的引用（而非值的副本）；ref local 声明一个引用变量。用于操作大型集合中的元素，避免复制。

```csharp
ref int Find(int[] arr, int target) {
    for (int i = 0; i < arr.Length; i++)
        if (arr[i] == target) return ref arr[i];
    throw new Exception("Not found");
}
ref int found = ref Find(nums, 2);  // found 引用 arr[1]
found = 99;                          // 直接修改 arr[1]
```

---

## 第八章 类与对象

### 8.1 class

class 是引用类型，分配在托管堆，支持继承和多态。适合复杂对象、需要共享引用的场景。所有 class 最终继承自 System.Object。

### 8.2 struct

struct 是值类型，适合小而不可变的数据（Vector3、Point、Color）。赋值时复制整个值，不产生 GC。不能继承，但可实现接口。

### 8.3 record（C# 9+）

record 是**值相等语义的不可变数据载体**。比较两个 record 时比较属性值（而非引用），支持 with 表达式创建副本。

```csharp
public record Person(string Name, int Age);

var p1 = new Person("Alice", 25);
var p2 = new Person("Alice", 25);
Console.WriteLine(p1 == p2);   // True！值相等！

// with 表达式 —— 创建不可变副本
var p3 = p1 with { Age = 26 };  // 新对象，Name="Alice", Age=26
```

| | class | struct | record |
|------|------|------|------|
| **类型** | 引用 | 值 | 引用（class）/ 值（struct） |
| **相等性** | 引用相等 | 值相等 | 值相等 |
| **可变性** | 可变 | 可变 | 不可变（推荐） |
| **继承** | ✅ | ❌ | record class 可继承 |
| **with** | ❌ | ❌ | ✅ |

### 8.4 sealed

sealed 阻止继承。sealed class 不能被继承，sealed override 阻止进一步重写。提升：编译器可以针对 sealed 调用做去虚拟化优化（直接调用而非虚调用）。

### 8.5 partial

允许一个类型的定义分散在多个文件中。典型用途：Windows Forms 自动生成代码 + 手写代码分离；Unity 脚本生成部分。

### 8.6 nested class

嵌套类定义在另一个类内部，可以访问外部类的 private 成员。常用于仅对外部类有用的小型辅助类。

---

## 第九章 面向对象

### 9.1 四大支柱

| 支柱 | 核心思想 | C# 实现 |
|------|------|------|
| **封装** | 隐藏内部，暴露接口 | private + property |
| **继承** | is-a，复用代码 | `class Dog : Animal` |
| **多态** | 同一接口，不同行为 | virtual + override |
| **抽象** | 隐藏复杂度 | abstract / interface |

### 9.2 封装

封装 = 数据私有 + 公共方法访问。C# 用 `private` 字段 + `public` 属性实现。对调用方隐藏内部状态变更逻辑，降低耦合。

```csharp
public class BankAccount {
    private decimal _balance;               // 不直接暴露
    public decimal Balance => _balance;       // 只读属性
    public void Deposit(decimal amount) {
        if (amount <= 0) throw new ArgumentException();
        _balance += amount;
    }
}
// 调用方不能直接 _balance = -10000，必须通过 Deposit
```

### 9.3 继承

继承 = 子类复用父类的字段和方法，实现代码共享。C# 只支持单继承（一个类只能有一个基类），但可通过接口实现多重行为。

继承的局限：紧耦合（父类改动影响所有子类）、继承链过深难理解、静态绑定。解决：组合优于继承。

### 9.4 SOLID 原则（★★★★★）

> **一分钟回答**：SOLID 是面向对象设计的五个核心原则。SRP（单一职责）= 一个类只做一件事；OCP（开闭原则）= 对扩展开放、对修改关闭；LSP（里氏替换）= 子类必须能完全替换父类；ISP（接口隔离）= 小而专一的接口优于大而全的接口；DIP（依赖倒转）= 依赖抽象而非具体实现。这五个原则共同目标是：**高内聚、低耦合、易扩展**。

---

#### S — 单一职责原则（Single Responsibility Principle）

> **一个类应该有且仅有一个引起它变化的原因。**

核心理念：一个类只负责一件事。当需求变化时，修改的影响范围最小化。

```csharp
// ❌ 违反 SRP：一个类干了三件事（数据、渲染、持久化）
public class Player {
    public string Name { get; set; }
    public int Health { get; set; }
    
    public void Render() { /* 在屏幕上绘制玩家 */ }
    public void SaveToFile() { /* 保存到文件 */ }
}
// 问题：UI 变了要改这个类，存储方式变了也要改这个类 → 不稳定

// ✅ 遵循 SRP：三个类各司其职
public class Player {
    public string Name { get; set; }
    public int Health { get; set; }
}
public class PlayerRenderer {
    public void Render(Player p) { /* 负责渲染 */ }
}
public class PlayerRepository {
    public void Save(Player p) { /* 负责持久化 */ }
}
// 哪个维度变化，只改对应的类，Player 本体不动
```

**如何判断是否违反 SRP？**
- 问自己："这个类做什么？" 如果答案中包含"和"字，可能违反
- 修改一个功能时，是否总是不小心影响另一个功能？
- 类的方法是否使用到了类的不同字段子集？（一个方法只用 A 字段，另一个只用 B 字段→可能该拆）

**Unity 中的 SRP**：
```csharp
// ❌ MonoBehaviour 里塞了太多职责
public class Player : MonoBehaviour {
    // 移动逻辑
    void HandleMovement() { }
    // 动画播放
    void HandleAnimation() { }
    // 音效播放
    void HandleAudio() { }
    // 数据持久化
    void SaveData() { }
}

// ✅ 拆分为独立组件
public class PlayerMovement : MonoBehaviour { }
public class PlayerAnimation : MonoBehaviour { }
public class PlayerAudio : MonoBehaviour { }
public class PlayerData : MonoBehaviour { }
// 每个组件 = 单一职责 → 可单独测试、可复用、改动影响小
```

---

#### O — 开闭原则（Open/Closed Principle）

> **对扩展开放，对修改关闭。**

核心理念：新增功能时，通过**扩展**（加新类/新方法）实现，而不是**修改**已有代码。已有代码经过测试，改它可能引入新 bug。

```csharp
// ❌ 违反 OCP：每加一个新形状都要改 AreaCalculator
public class AreaCalculator {
    public double GetArea(object shape) {
        if (shape is Circle c) return Math.PI * c.Radius * c.Radius;
        if (shape is Rectangle r) return r.Width * r.Height;
        if (shape is Triangle t) return 0.5 * t.Base * t.Height;
        // 每加一个新形状 → 修改这个方法（if-else 链越来越长）
    }
}

// ✅ 遵循 OCP：扩展新形状只需加新类，不需要改 AreaCalculator
public interface IShape {
    double GetArea();  // 每个形状自己实现
}
public class Circle : IShape {
    public double Radius { get; set; }
    public double GetArea() => Math.PI * Radius * Radius;
}
public class Rectangle : IShape {
    public double Width { get; set; }
    public double Height { get; set; }
    public double GetArea() => Width * Height;
}
// 新增 Triangle 时：只需实现 IShape，AreaCalculator 一行都不改！

public class AreaCalculator {
    public double GetArea(IShape shape) => shape.GetArea();  // 永远不变
}
```

**OCP 的核心武器**：
1. **接口/抽象类**：定义扩展点（如 IShape）
2. **virtual 方法**：允许子类重写特定行为
3. **策略模式**：将可变算法封装为独立策略类
4. **插件架构**：通过反射/DI 加载外部实现

**Unity 中的 OCP**：
```csharp
// ✅ 技能系统：新增技能只需加新类，不改 SkillManager
public abstract class Skill {
    public abstract void Execute(Player caster);
}
public class Fireball : Skill {
    public override void Execute(Player caster) { /* 火球 */ }
}
public class Heal : Skill {
    public override void Execute(Player caster) { /* 治疗 */ }
}
// 新增 LightningSkill → 继承 Skill，实现 Execute → SkillManager 不动
```

---

#### L — 里氏替换原则（Liskov Substitution Principle）

> **子类必须能够完全替换其基类，而不破坏程序的正确性。**

核心理念：程序中任何使用基类的地方，换成子类后，行为应该依然正确。子类可以扩展父类行为，但不能改变父类约定的语义。

```csharp
// ❌ 经典反例：正方形继承长方形
public class Rectangle {
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
    public int Area() => Width * Height;
}
public class Square : Rectangle {
    public override int Width {
        set { base.Width = base.Height = value; }  // 设 Width，偷偷改了 Height！
    }
    public override int Height {
        set { base.Width = base.Height = value; }  // 设 Height，偷偷改了 Width！
    }
}

// LSP 被破坏的证据：
void Test(Rectangle rect) {
    rect.Width = 5;
    rect.Height = 10;
    Debug.Assert(rect.Area() == 50);  // 对 Rectangle：5×10=50 ✅
                                       // 对 Square：设 Width=5 → W=H=5
                                       //            设 Height=10 → W=H=10
                                       //            Area = 10×10 = 100 ❌ 断言失败！
}
// Square 不能安全地替代 Rectangle → 违反 LSP
```

**LSP 的常见违反方式**：
1. 子类抛基类不抛的异常（调用方按基类契约编程→炸了）
2. 子类对输入参数要求更严（基类接受 int，子类只接受正数）
3. 子类返回值和基类语义不一致
4. 子类修改了基类方法的前置/后置条件

```csharp
// ✅ 遵循 LSP：正确的抽象
public interface IShape {
    int Area();
}
public class Rectangle : IShape {
    public int Width { get; set; }
    public int Height { get; set; }
    public int Area() => Width * Height;
}
public class Square : IShape {  // 不继承 Rectangle！
    public int Side { get; set; }
    public int Area() => Side * Side;
}
// 各自独立实现 Area()，谁也不能替代谁，但都满足 IShape 契约
```

**面试追问**：LSP 和多态的关系？
> LSP 是多态可以正确工作的**前提条件**。如果子类不能安全替代父类（违反 LSP），多态调用（基类引用调子类方法）就会产生错误行为。多态是语法层面的支持（virtual/override），LSP 是语义层面的约束。

---

#### I — 接口隔离原则（Interface Segregation Principle）

> **客户端不应该被迫依赖它不使用的方法。**

核心理念：大而全的接口应该拆分成小而专一的接口。这样实现类只需关心自己真正需要的接口。

```csharp
// ❌ 违反 ISP：大而全的接口
public interface IEntity {
    void Move();
    void Attack();
    void Fly();
    void Swim();
    void Heal();
}
// 问题：
// Ghost 只需要 Move，但被迫实现 Attack/Fly/Swim/Heal
// Fish 只需要 Swim，但被迫实现 Move/Attack/Fly/Heal
// 每个实现类都带着一堆 throw new NotImplementedException()

// ✅ 遵循 ISP：小而专一的接口
public interface IMovable { void Move(); }
public interface IAttacker { void Attack(); }
public interface IFlyable { void Fly(); }
public interface ISwimmable { void Swim(); }
public interface IHealable { void Heal(); }

// 每个类只实现它需要的接口
public class Ghost : IMovable, IAttacker {
    public void Move() { /* 飘 */ }
    public void Attack() { /* 鬼爪 */ }
}
public class Fish : ISwimmable {
    public void Swim() { /* 游 */ }
}
public class Dragon : IMovable, IAttacker, IFlyable {
    public void Move() { /* 走 */ }
    public void Attack() { /* 喷火 */ }
    public void Fly() { /* 飞 */ }
}
```

**ISP 的判断标准**：
- 接口有多少个方法？超过 5~7 个可能有问题
- 调用方是否只用到了接口中的部分方法？（看传入的接口参数实际调了几个方法）
- 不同实现类是否总有一些方法抛 NotImplementedException？

**Unity 中的 ISP**：
```csharp
// ✅ 小而专一的接口 → 组件查询更精确
public interface IDamageable { void TakeDamage(int dmg); }
public interface IInteractable { void Interact(Player p); }
public interface IPickupable { void OnPickup(Player p); }

// 查询所有可攻击目标，不关心它们能不能被拾取
var targets = FindObjectsOfType<MonoBehaviour>().OfType<IDamageable>();
```

---

#### D — 依赖倒转原则（Dependency Inversion Principle）

> **高层模块不应该依赖低层模块，两者都应该依赖抽象。抽象不应该依赖细节，细节应该依赖抽象。**

核心理念："倒转"意味着反转依赖方向——传统设计中高层依赖低层，DIP 要求两者都依赖抽象（接口）。

```csharp
// ❌ 违反 DIP：高层 Player 直接依赖低层 Sword/Gun 具体类
public class Sword {
    public void Swing() { /* 挥砍 */ }
}
public class Player {
    private Sword _sword = new Sword();  // 硬编码依赖具体武器！
    public void Attack() {
        _sword.Swing();
    }
}
// 问题：想换枪？→ 要改 Player 代码
//      想加新武器？→ Player 要加 if-else
//      想单元测试？→ Sword 是真实对象，没法 mock

// ✅ 遵循 DIP：两者都依赖抽象 IWeapon
public interface IWeapon {
    void Attack();
}
public class Sword : IWeapon {
    public void Attack() => Console.WriteLine("挥砍");
}
public class Gun : IWeapon {
    public void Attack() => Console.WriteLine("射击");
}
public class MagicWand : IWeapon {
    public void Attack() => Console.WriteLine("魔法");
}

public class Player {
    private readonly IWeapon _weapon;  // 依赖抽象！
    
    // 构造函数注入：从外部传入，自己不 new
    public Player(IWeapon weapon) {
        _weapon = weapon;
    }
    
    public void Attack() {
        _weapon.Attack();  // 调接口方法，不关心具体实现
    }
    
    // 运行时也可以换武器
    public void EquipWeapon(IWeapon newWeapon) {
        // _weapon = newWeapon;
    }
}

// 使用
var player = new Player(new Sword());  // 拿剑
player.Attack();  // 挥砍
player.EquipWeapon(new MagicWand());   // 换法杖
player.Attack();  // 魔法

// 单元测试：mock 武器
var mockWeapon = new Mock<IWeapon>();
var player = new Player(mockWeapon.Object);
```

**依赖注入的三种方式**：
```csharp
// 1. 构造函数注入（推荐：明确依赖，初始化后不可变）
public Player(IWeapon weapon) { _weapon = weapon; }

// 2. 属性注入（可选依赖，可运行时切换）
public IWeapon Weapon { get; set; }

// 3. 方法注入（仅当前方法需要的依赖）
public void AttackWith(IWeapon weapon) { weapon.Attack(); }
```

**Unity 中的 DIP / DI 容器**：
```csharp
// 用 Zenject / VContainer 实现依赖注入
public class PlayerInstaller : MonoInstaller {
    public override void InstallBindings() {
        Container.Bind<IWeapon>().To<Sword>().AsSingle();
        Container.Bind<Player>().FromComponentInHierarchy().AsSingle();
    }
}
// Player 不需要知道武器是什么类型，DI 容器自动注入
```

---

#### SOLID 五原则关系总结

```
        ┌──────────────┐
        │     SRP      │ ← 类的粒度：做一件事
        │  单一职责     │
        └──────┬───────┘
               │ 指导如何拆分
               ▼
┌──────────────┐    ┌──────────────┐
│     OCP      │    │     LSP      │
│   开闭原则    │◄───┤  里氏替换     │
│  扩展不修改   │    │ 子类可替换基类 │
└──────┬───────┘    └──────────────┘
       │ 指导如何设计扩展点
       ▼
┌──────────────┐    ┌──────────────┐
│     ISP      │───►│     DIP      │
│  接口隔离     │    │  依赖倒转     │
│ 小接口优于大  │    │ 依赖抽象不具体 │
└──────────────┘    └──────────────┘
       │                    │
       └────────┬───────────┘
                ▼
         高内聚 · 低耦合 · 易扩展
```

| 原则 | 核心问题 | 关键手段 | 面试常考点 |
|------|---------|---------|-----------|
| **SRP** | 类太臃肿，一个改动影响多个功能 | 按职责拆分 | "这个类做什么？" |
| **OCP** | 加新功能要改旧代码，风险大 | 接口/虚方法/策略模式 | "怎么加新功能不改旧代码？" |
| **LSP** | 子类不能安全替代父类，多态失效 | 正确的继承关系设计 | "正方形能继承长方形吗？" |
| **ISP** | 实现类被迫实现不需要的方法 | 大接口拆小接口 | "接口多少方法算大？" |
| **DIP** | 高层依赖低层具体类，无法替换 | DI + 面向接口编程 | "依赖注入有几种方式？" |

**面试追问**：SOLID 在实际项目中有哪些取舍？
> 1. 小型项目/原型不需要严格遵循所有原则→过度设计
> 2. SRP 拆太细则类爆炸→适度拆分，按变化维度而非功能点
> 3. ISP 每个接口 1~2 个方法也可能过度→按"共同被使用"分组
> 4. DIP 每个类都依赖接口→简单类（值对象、DTO）不需要抽象
> 5. 关键判断标准：**"未来这个维度会独立变化吗？"** 如果不会，也许不需要拆。

### 9.5 组合优于继承

组合 = 持有其他类型实例，将功能委托给它（has-a），而非通过继承获得（is-a）。组合更灵活——运行时切换行为，不产生继承链耦合。

```csharp
// ✅ 组合：Player 持有 List<IPlayerBehavior>，运行时随意组合
public interface IPlayerBehavior { void Execute(); }
public class Player {
    private readonly List<IPlayerBehavior> _behaviors = new();
    public void AddBehavior(IPlayerBehavior b) => _behaviors.Add(b);
    public void Update() { foreach (var b in _behaviors) b.Execute(); }
}
```

### 9.6 interface vs abstract

| | interface | abstract class |
|------|------|------|
| 多实现/继承 | ✅ 可实现多个 | ❌ 单继承 |
| 字段 | ❌ | ✅ |
| 构造函数 | ❌ | ✅ |
| 默认实现 | C# 8+ 支持 | ✅ 任意方法 |
| 访问修饰符 | public only | 任意 |
| 使用场景 | "能做什么" | "是什么" |

---

## 第十章 多态原理

### 10.1 virtual / override 怎么工作？

virtual 标记方法可被子类重写；override 标记子类重写该实现。CLR 通过 vtable（虚方法表）在运行时找到正确的 override 版本——这称为动态绑定。调用虚方法 = `callvirt` 指令 → 查 vtable → 跳转。

```csharp
class Animal {
    public virtual void Speak() => Console.WriteLine("Animal");
}
class Dog : Animal {
    public override void Speak() => Console.WriteLine("汪汪");
}
Animal a = new Dog();
a.Speak();  // "汪汪" — 运行时根据实际类型 Dog 决定
```

### 10.2 new vs override

override = 重写虚方法，运行时动态绑定；new = 隐藏基类方法，编译时静态绑定。用 override，通过基类引用也能调子类方法；用 new，基类引用调的是基类方法。

```csharp
Base b = new Derived();
b.Virtual();  // "Derived.Virtual"（override 是多态！）
b.Normal();   // "Base.Normal"（new 不是多态！基类引用 → 基类版本）
```

### 10.3 abstract 方法

abstract 方法 = 没有实现的虚方法，强制非抽象子类提供 override 实现。在 vtable 中占据 slot，但入口指向 CLR 的"纯虚调用"标记。

### 10.4 动态绑定的底层原理

CLR 的 MethodTable 包含 vtable（虚方法表）。每个虚方法在 vtable 中有固定 slot 索引。虚方法调用：`callvirt` → 对象引用 → MethodTable → vtable[slot_index] → 跳转到实际代码。

```
运行时调用 a.Speak() 的过程：
1. a 引用 → 找到 Dog 对象
2. Dog 对象头 → syncblock → MethodTable*（指向 Dog 的 MethodTable）
3. Dog.MethodTable → vtable[Speak_slot]
4. vtable[Speak_slot] → Dog.Speak 方法代码地址
5. 跳转执行
```

**vtable 布局**：

```
System.Object MethodTable:
  slot 0: ToString, slot 1: Equals, slot 2: GetHash, slot 3: Finalize

Animal MethodTable (extends Object):
  slot 0-3: (继承), slot 4: Animal.Speak ← 新增

Dog MethodTable (extends Animal):
  slot 0-3: (继承), slot 4: Dog.Speak ← override 覆盖！
  slot 5: Dog.Eat ← 新增
```

关键：slot 索引在继承链路中不变 → 祖先代码可以调用子类的方法而无需知道具体类型。

为什么 C# 默认方法不是 virtual？（对比 Java）Java 默认虚方法，每个方法调用都是虚调用——开销大。C# 默认非虚，只有显式标记 virtual 才是虚调用——性能更好，也不鼓励随便重写。

---

## 第十一章 static

### 11.1 static 字段

static 字段属于类而非实例，存储在静态区，生命周期 = 程序运行全程。所有实例共享同一个静态字段。GC 不回收静态字段。

### 11.2 static 方法

static 方法属于类，不依赖实例。只能访问 static 成员，不能访问实例成员。调用时用 `ClassName.Method()`。

### 11.3 static class

static class 只能包含 static 成员，不能实例化、不能继承。适合工具类、扩展方法容器。

### 11.4 new 关键字在内存层面做了什么（★★★★★）

> **一分钟回答**：`new` 在 CLR 中经历三步：① 托管堆分配内存（所有字段零初始化）；② 初始化——类型静态初始化（若未触发）→ 实例字段 inline 赋值 → 基类构造 → 当前类构造；③ 返回对象引用。**内存先于构造函数存在，构造函数只是"初始化已分配的内存块"。**

```
new MyClass() 的完整流程：

1. CLR 计算对象总大小（所有字段 + syncblock 索引 + MethodTable 指针）
   ↓
2. 托管堆分配内存 → 所有字节初始化为 0
   ↓
3. 检查类型是否已静态初始化 → 未初始化则触发静态构造函数（线程安全）
   ↓
4. 设置 MethodTable 指针（指向该类型的 MethodTable）
   ↓
5. 按继承链从顶向下执行构造函数：
   System.Object() → BaseClass() → CurrentClass()
   ↓
6. 返回对象引用给栈上的变量
```

```csharp
// 验证：内存先于构造函数存在——构造函数里 this 已经是可用的引用
public class Demo {
    private int x = 42;  // inline 赋值在构造函数体之前执行
    
    public Demo() {
        Console.WriteLine(this.x);  // 42 —— 在构造函数体开始时字段已初始化！
        Console.WriteLine(this);     // Demo 对象已存在
    }
}
```

**面试追问**：分配内存时为什么字段是 0/null？
> CLR 分配用的是高效内存分配器（移动 next 指针），不逐个字段清零——整块内存已被 GC 在压缩回收时清零。好处：new 比 C++ 的 malloc 快（无需遍历字段），且编译器能优化掉"先设 0 再赋 42"的双写。

---

### 11.5 构造函数的继承链执行顺序（★★★★★）

> **一分钟回答**：继承链中构造函数**从顶向下**执行：System.Object() → Base() → Derived()。每个构造函数在调用自己的代码之前，先隐式或显式调用基类构造。字段 inline 赋值在每个构造函数的**函数体之前**执行，且顺序也遵循"基类 inline → 基类构造体 → 子类 inline → 子类构造体"。

#### 完整执行序列（图解）

```
class Base {
    int b1 = B1();           // 步骤 4
    public Base() {           // 步骤 5
        B2();
    }
}

class Derived : Base {
    int d1 = D1();           // 步骤 6
    public Derived() {        // 步骤 7
        D2();
    }
}

new Derived() 的执行顺序：
┌─────────────────────────────────┐
│ 1. 托管堆分配 Derived 所需大小   │
│ 2. 所有字段 = 0 / null           │
│ 3. 检查 Derived 静态初始化       │
│                                 │
│ ┌── Base 部分 ──────────────┐   │
│ │ 4. Base 实例字段 inline    │   │  ← b1 = B1()
│ │ 5. Base() 构造函数体       │   │  ← B2()
│ └───────────────────────────┘   │
│                                 │
│ ┌── Derived 部分 ───────────┐   │
│ │ 6. Derived 实例字段 inline │   │  ← d1 = D1()
│ │ 7. Derived() 构造函数体    │   │  ← D2()
│ └───────────────────────────┘   │
└─────────────────────────────────┘
```

#### 代码验证

```csharp
public class Animal {
    private int _id = GenerateId();   // 步骤 1
    public Animal() {                  // 步骤 2
        Console.WriteLine("Animal ctor");
        Console.WriteLine(_id > 0);     // true —— inline 已在构造体前执行
    }
    private static int GenerateId() {
        Console.WriteLine("Animal field init");
        return 1;
    }
}

public class Dog : Animal {
    private string _name = GetName();  // 步骤 3
    public Dog() {                      // 步骤 4
        Console.WriteLine("Dog ctor");
    }
    private static string GetName() {
        Console.WriteLine("Dog field init");
        return "Buddy";
    }
}

// 执行 new Dog()
// 输出顺序：
// Animal field init     ← 基类 inline 字段
// Animal ctor           ← 基类构造函数体
// Dog field init        ← 子类 inline 字段
// Dog ctor              ← 子类构造函数体
```

#### base() 显式调用

```csharp
public class Dog : Animal {
    public Dog() : base() { }        // 默认隐式调用基类无参构造
    public Dog(string name) : base(name) { }  // 显式调用基类有参构造
    // 如果没有 base()，编译自动插入 base()（前提是基类有无参构造）
    // 如果基类没有无参构造 → 必须显式 base(...)
}
```

#### 关键面试点

| 顺序 | 说明 |
|------|------|
| **字段 inline 先于构造体** | `int x = 42;` 在构造函数体 `{ }` 之前执行 |
| **基类先于子类** | Base 全部完成后才开始 Derived |
| **static 先于实例** | 类型首次访问时，static 初始化先于任何实例构造 |
| **内存先于一切** | 分配+置零最先发生，然后才有初始化代码 |

---

### 11.6 静态构造函数时机详解（★★★★★）

> **一分钟回答**：静态构造在**类首次被访问前**自动执行一次，CLR 保证线程安全。关键区别在 **BeforeFieldInit**：如果类没有显式写静态构造，CLR 标记 BeforeFieldInit → 静态字段可以"随时"初始化（可能在首次访问前更早），性能更好；如果有显式静态构造 → CLR 必须严格在首次访问前的那一刻才执行。

#### BeforeFieldInit 的精确语义

```csharp
// 情况 1：只有 inline 赋值，没有显式静态构造
public class A {
    public static int X = 1;  // 没有 static ctor → BeforeFieldInit
}
// CLR 可在首次访问 A 之前任意时刻初始化 X
// 可能在你调 A.X 的前一刻，也可能程序启动时就初始化了

// 情况 2：有显式静态构造
public class B {
    public static int X = 1;
    static B() { }  // 显式静态构造 → 非 BeforeFieldInit
}
// CLR 必须严格在"首次访问 B 的任何成员或创建实例"的前一刻才执行
// 不能更早
```

```
性能影响：
┌─────────────────────────────────────────────────────────┐
│ BeforeFieldInit（无显式 cctor）                           │
│  • JIT 编译时可以直接内联静态字段值                          │
│  • CLR 可以提前（eager）初始化，避免访问时的锁检查             │
│  • 更快：省去了"是否已初始化"的运行时检查                      │
│                                                         │
│ 非 BeforeFieldInit（有显式 cctor）                         │
│  • 每次访问静态成员都要先检查"是否已初始化"                    │
│  • 必须精确在首次访问前一刻执行 → 有锁开销                    │
│  • 更慢，但初始化时机确定（适合依赖顺序的场景）                  │
└─────────────────────────────────────────────────────────┘
```

#### 静态构造的触发条件

| 操作 | 是否触发 cctor |
|------|------|
| `ClassName.StaticField`（访问静态字段） | ✅ |
| `new ClassName()`（首次创建实例） | ✅ |
| `ClassName.StaticMethod()` | ✅ |
| `typeof(ClassName)` | ❌ 不触发 |
| `ClassName[] arr = new ClassName[10]`（创建数组） | ❌ 不触发 |
| `ClassName obj = null`（声明变量） | ❌ 不触发 |

```csharp
// 验证：typeof 不触发静态构造
public class HeavyInit {
    public static int X;
    static HeavyInit() {
        Console.WriteLine("cctor 执行！");
        X = 42;
    }
}

Type t = typeof(HeavyInit);          // 不打印
var arr = new HeavyInit[5];           // 不打印（只是分配数组空间）
Console.WriteLine(HeavyInit.X);       // 此时才打印 "cctor 执行！"
```

#### 静态构造的线程安全

```csharp
// CLR 内部对静态构造加锁：多线程同时首次访问同一个类时，
// 只有一个线程执行 cctor，其他线程阻塞等待它完成。
// 这确保了：
// 1. cctor 只执行一次
// 2. 所有线程都看到 cctor 完成后的值

public class Singleton {
    public static readonly Singleton Instance;
    static Singleton() {
        Instance = new Singleton();  // 线程安全的单例！
    }
}
// 利用 CLR 对 cctor 的锁 = 最简单的线程安全单例
```

---

### 11.7 完整初始化顺序表（面试版）

> **★★★★★ 这是面试最高频的考察点。一句话：静态优于实例，基类优于子类，inline 优于构造体。**

```
完整的 new Derived() 时间线：

0. 程序启动 → 静态区预留空间（所有 static 字段 = 0）

1. [首次访问 Derived 的任何成员时]
   ├── 1a. Base static 字段 inline 赋值
   ├── 1b. Base static 构造函数执行
   ├── 1c. Derived static 字段 inline 赋值
   └── 1d. Derived static 构造函数执行

2. [new Derived()]
   ├── 2a. 托管堆分配内存（所有字段 = 0/null）
   ├── 2b. Object 实例字段 inline 赋值
   ├── 2c. Object 构造函数体
   ├── 2d. Base 实例字段 inline 赋值
   ├── 2e. Base 构造函数体
   ├── 2f. Derived 实例字段 inline 赋值
   └── 2g. Derived 构造函数体

3. 返回对象引用
```

```csharp
// 考试题：输出顺序是什么？
public class A {
    static int sa = Log("A.sa");
    int ia = Log("A.ia");
    static A() { Log("A.static ctor"); }
    public A() { Log("A.ctor"); }
    public static int Log(string msg) { Console.WriteLine(msg); return 0; }
}

public class B : A {
    static int sb = Log("B.sb");
    int ib = Log("B.ib");
    static B() { Log("B.static ctor"); }
    public B() { Log("B.ctor"); }
}

new B();
// 输出：
// A.sa              ← 首次访问 B → 先初始化基类 A 的 static
// A.static ctor
// B.sb              ← 然后是 B 自身的 static
// B.static ctor
// A.ia              ← 开始构造实例：A inline → A ctor
// A.ctor
// B.ib              ← B inline → B ctor
// B.ctor
```

#### 与 C++ 的关键区别

| | C# | C++ |
|------|------|------|
| **内存分配** | CLR 分配+自动清零 | 手动 new / 栈对象 |
| **基类构造调用** | 编译器自动插入 `: base()` | 初始化列表中显式写 |
| **字段 inline 初始化** | 在构造函数体之前自动执行 | 在初始化列表中执行 |
| **虚方法在构造中** | 调用**最派生类的 override**（危险！） | 调用**当前类的版本** |
| **静态初始化** | CLR 保证线程安全且仅一次 | 手动控制、不保证线程安全 |

```csharp
// ⚠️ C# 的虚方法陷阱：构造函数中调虚方法 → 调的是最终 override 版本！
public class Base {
    public Base() { Init(); }  // 构造函数中调虚方法！
    public virtual void Init() => Console.WriteLine("Base.Init");
}
public class Derived : Base {
    private string _name = "Bob";
    public override void Init() => Console.WriteLine($"Derived.Init, name={_name}");
    //                                          
    // 注意：此时 _name 的值是什么？
}
new Derived();  // 输出：Derived.Init, name=      ← _name 还是 null！
// 原因：Base 构造 → Derived.Init 被调 → 此时 Derived 的字段 inline 还没执行！
// 绝对不要在构造函数中调虚方法！
```

---

### 11.8 Unity 中 static 的陷阱

> **一分钟回答**：Unity 场景切换时普通 GameObject 被销毁，但 static 字段不销毁 → 持有已销毁对象的引用 → NullReferenceException。值类型 static OK（int、bool），引用 MonoBehaviour/GameObject 的 static 要特别小心。

```csharp
// ❌ 典型陷阱
public class Player : MonoBehaviour {
    public static Player Instance;
    void Awake() { Instance = this; }
} // 原场景销毁 → Instance 仍指向已 Destroy 的 GameObject → 野指针！

// ✅ 安全方案
public class Player : MonoBehaviour {
    public static Player Instance { get; private set; }
    
    void OnEnable() {
        if (Instance != null && Instance != this) {
            Destroy(gameObject);
            return;
        }
        Instance = this;
        DontDestroyOnLoad(gameObject);
    }
    
    void OnDestroy() {
        if (Instance == this) Instance = null;  // 清理
    }
}

// ✅ 更安全：不存 MonoBehaviour 引用，存数据
public static class GameState {
    public static int Score;        // 值类型完全安全
    public static bool IsPaused;    // 值类型完全安全
    // 不存任何对场景对象的引用
}
```

---

## 第十二章 委托

### 12.1 delegate 的本质

委托 = 类型安全的函数指针。本质是一个类，继承自 MulticastDelegate → Delegate。包含两个核心字段：`_target`（目标对象，静态方法为 null）和 `_methodPtr`（函数地址）。

```csharp
public delegate int MathOperation(int a, int b);
MathOperation op = Add;
op += Multiply;

// 编译器生成：
// class MathOp : MulticastDelegate {
//     int Invoke(int a, int b);
//     IAsyncResult BeginInvoke(...);
//     int EndInvoke(...);
// }
```

### 12.2 MulticastDelegate 的多播

委托支持 `+=`/`-=` 来注册/移除多个方法。内部维护 `_invocationList`（Delegate[]），调用时按注册顺序依次执行。返回最后一个方法的返回值。

注意事项：返回值只返回最后注册的方法的返回值（前面的被丢弃）；某个方法抛异常，后面的不执行；空委托调用抛 NullReferenceException → 始终用 `?.Invoke()`。

### 12.3 Action、Func、Predicate

内置泛型委托，现代 C# 很少自定义委托。

- **Action**：无返回值（0~16 个参数）
- **Func**：有返回值（最后一参数是返回类型）
- **Predicate**：返回 bool（单参数）

```csharp
Action<string> print = s => Console.WriteLine(s);
Func<int, int, int> add = (a, b) => a + b;
Predicate<int> isPositive = n => n > 0;
```

**委托演进史**：C# 1.0 → delegate 关键字手动声明 → C# 2.0 → 匿名方法 → C# 3.0 → Lambda + Action/Func → 基本不再自定义委托

### 12.4 委托 vs 函数指针

| | delegate | 函数指针 |
|------|------|------|
| **类型安全** | ✅ | ❌ |
| **多播** | ✅ | ❌ |
| **绑定 this** | ✅（实例方法） | ❌ |
| **闭包** | ✅ | ❌ |
| **GC** | 被 GC 追踪 | 不受 GC 管 |
| **性能** | 有虚调用开销 | 直接跳转 |

---

## 第十三章 Event

### 13.1 event 是什么？

event 是对 delegate 的包装，添加了权限控制——外部只能 `+=`/`-=`（订阅/取消订阅），不能 `=`（赋值覆盖）也不能从外部 `Invoke`（触发）。发布-订阅模式的核心。

```csharp
public class Player {
    public event Action<int> OnHealthChanged;  // 外部只能 +=/-=
    
    public void TakeDamage(int damage) {
        health -= damage;
        OnHealthChanged?.Invoke(health);  // 只有类内部能触发
    }
}
```

### 13.2 为什么 event 比 delegate 安全？

1. 外部不能 `=` 覆盖（防止清空所有订阅）
2. 外部不能 `Invoke`（防止外部伪造事件触发）
3. event 明确语义："这是一个事件，请订阅"（delegate 可能是回调/策略）

event 底层：编译后 = private delegate 字段 + public add/remove 方法。`+=` 调 add（内部 `Delegate.Combine`），`-=` 调 remove（`Delegate.Remove`）。

### 13.3 标准事件模式

`public event EventHandler<TEventArgs> EventName` —— TEventArgs 继承 `EventArgs`，发送方参数 `object sender`。这是 .NET 约定俗成的命名规范。

### 13.4 Observer 模式

事件天然实现 Observer（发布-订阅）模式。发布者不关心谁在订阅；订阅者只关心事件发生，不依赖发布者内部实现。最常用的解耦手段。

### 13.5 Unity Event vs C# Event

| | C# event | UnityEvent |
|------|------|------|
| **Inspector 可见** | ❌ | ✅ |
| **运行时性能** | 极快 | 稍慢（序列化支持） |
| **代码绑定** | ✅ | ✅ |
| **序列化** | ❌ | ✅ |
| **适用场景** | 纯代码逻辑 | 需要策划/美术在 Inspector 配置 |

### 13.6 EventBus

EventBus 是全局事件总线，允许任意组件之间通信而不直接引用。解耦性强但易滥用——事件爆炸、调用链不清晰。适合少数全局事件（如游戏暂停、场景切换）。

### 13.7 常见陷阱

1. 忘记取消订阅 → 内存泄漏：`OnEnable` 中 `+=`，`OnDisable` 中 `-=`
2. 匿名函数/Lambda 隐式捕获 this：Unity 中 lambda 持有 MonoBehaviour 引用 → 即使 Destroy 对象也不会被 GC
3. null 检查：始终用 `?.Invoke()` 而非直接 `()`
4. 多播委托异常：某个订阅者抛异常 → 后续订阅者不被通知

---

## 第十四章 Lambda 与闭包

### 14.1 Lambda 的本质

Lambda 表达式是匿名方法的语法糖。编译器将 Lambda 转为：① 私有静态方法（如果不捕获变量），或 ② DisplayClass 的实例方法（如果捕获变量）。本质是委托（delegate）。

### 14.2 Capture（捕获）

Lambda 引用外部变量称为"捕获"。编译器会创建一个 DisplayClass 对象，将捕获的变量提升为 DisplayClass 的**字段**。Lambda 变为 DisplayClass 的实例方法。

```csharp
int counter = 0;
Action increment = () => counter++;
// counter 被"捕获"了！编译器生成：
// sealed class DisplayClass0 {
//     public int counter;
//     internal void <M>b__0() => counter++;
// }
```

关键后果：Lambda 的寿命和捕获的变量一起被延长了！display class 在堆上，不被 GC 回收。

### 14.3 闭包原理

闭包 = Lambda + 捕获的外部变量。闭包捕获的是**变量的引用**，不是值的快照。

```csharp
List<Action> actions = new List<Action>();
for (int i = 0; i < 3; i++) {
    actions.Add(() => Console.WriteLine(i));
}
foreach (var a in actions) a();  // 输出：3 3 3（不是 0 1 2！）

// 原因：只有一个 i 变量，所有 Lambda 捕获了同一个 i
// 循环结束后 i=3，所有 Lambda 都打印 3

// ✅ 修复：在循环内声明新变量
for (int i = 0; i < 3; i++) {
    int copy = i;
    actions.Add(() => Console.WriteLine(copy));
} // 输出：0 1 2 ✓
```

### 14.4 foreach 闭包陷阱

C# 5.0 之前 foreach 变量定义在循环外，导致闭包捕获同一变量；C# 5.0+ 修复了这个问题，每次迭代有独立变量。

```csharp
// Unity 中常见坑
for (int i = 0; i < buttons.Length; i++) {
    buttons[i].onClick.AddListener(() => OnButtonClicked(i));
    // 所有按钮的 Lambda 都捕获同一个 i！
}
// ✅ 修复
for (int i = 0; i < buttons.Length; i++) {
    int index = i;
    buttons[i].onClick.AddListener(() => OnButtonClicked(index));
}
```

---

## 第十五章 泛型

### 15.1 泛型为什么快？

泛型避免了装箱拆箱（值类型直接存，不需要转为 object），且编译时类型安全（不需要运行时 is/as 检查）。`List<int>` 内部就是 `int[]`，无装箱，无类型检查开销。

### 15.2 泛型约束（where）

| 约束 | 含义 | 例子 |
|------|------|------|
| `where T : class` | 必须是引用类型 | `T obj = null` 合法 |
| `where T : struct` | 必须是值类型 | 不能传 class |
| `where T : new()` | 必须有公共无参构造 | `new T()` 合法 |
| `where T : unmanaged` | 必须是非托管类型 | `fixed(T*)` 合法 |
| `where T : BaseClass` | 必须是某基类的子类 | 访问基类成员 |
| `where T : IFoo` | 必须实现某接口 | 调接口方法 |

### 15.3 CLR 泛型实现

CLR 泛型不是编译期代码生成，而是**运行时特化**。所有引用类型共享同一份 JIT 编译后的机器码（因为指针大小相同）；每个值类型有独立的一份机器码（因为不同 struct 大小不同）。

```
引用类型：
  List<string>   ─┐
  List<Player>   ─┼→ 共享同一份机器码（内部存储 object 引用）
  List<Stream>   ─┘   JIT 只编译一次！

值类型：
  List<int>  → 独立机器码（int 大小 4 字节，需专用指令）
  List<long> → 独立机器码（long 大小 8 字节）
  List<Vector3> → 独立机器码（Vector3 大小 12 字节）
```

为什么引用类型可以共享，值类型不行？引用类型都是 8 字节（64位）指针，操作指令相同。值类型大小不同（int=4、long=8），需要不同的偏移量和存储指令。

### 15.4 C# 泛型 vs C++ 模板

| | C++ Template | C# Generic |
|------|------|------|
| **时机** | 编译期展开 | 运行时特化 |
| **检查** | 实例化时才检查（duck typing） | 编译时检查（需约束） |
| **机器码** | N 种类型 → N 份代码 | 引用类型共享，值类型各一份 |
| **灵活性** | 极高（SFINAE、特化） | 受约束限制 |
| **二进制膨胀** | 可能严重 | 可控 |
| **泛型信息** | 编译后丢失 | 运行时保留（反射可用） |

### 15.5 泛型在 Unity 中的限制

IL2CPP 下泛型必须能在 AOT 编译时确定所有实例化类型。泛型虚方法（`where T : IFoo` 时调接口方法）需要 IL2CPP 提前为所有可能的 T 生成桥接代码。

协变/逆变（C# 4+）：
```csharp
IEnumerable<string> strings = new List<string>();
IEnumerable<object> objects = strings;  // ✅ 协变 out

Action<object> objAction = o => { };
Action<string> strAction = objAction;  // ✅ 逆变 in
```

---

## 第十六章 LINQ（★★★★★）

> **一分钟回答**：LINQ（Language Integrated Query）是 C# 内嵌的查询语言，将类 SQL 的声明式语法编译为对集合的链式方法调用。核心三要素：① 扩展方法（Where/Select/GroupBy 等）挂接在 IEnumerable 上；② Lambda 表达式提供内联谓词；③ 延迟执行——查询定义时不执行，遍历时才执行。

---

### 16.1 查询表达式 vs 方法语法

LINQ 有两种等价写法，编译器将查询表达式翻译为方法语法：

```csharp
// 查询表达式（Query Expression）—— 类 SQL，可读性强
var result = from p in players
             where p.Score > 100
             orderby p.Name
             select new { p.Name, p.Score };

// 方法语法（Method Syntax / Fluent Syntax）—— Lambda 链，功能完整
var result = players
    .Where(p => p.Score > 100)
    .OrderBy(p => p.Name)
    .Select(p => new { p.Name, p.Score });
```

| | 查询表达式 | 方法语法 |
|------|------|------|
| **风格** | 类 SQL 声明式 | Lambda 链式调用 |
| **能力** | 是方法语法的子集 | 全部功能（如 Distinct、Take、Aggregate） |
| **适用** | Join/GroupBy/Let 更直观 | 简单过滤/投影更简洁 |
| **编译器** | 一律翻译为方法语法 | 不翻译，直接编译 |

**不是所有操作都有查询表达式关键字**——Take、Skip、Distinct、Aggregate、First、Count 等只能通过方法语法调用。实际项目中两种风格常混用：

```csharp
var top3 = (from p in players
            where p.IsActive
            orderby p.Score descending
            select p).Take(3).ToList();  // Take 只能方法语法
```

---

### 16.2 核心操作符分类（共 50+ 个）

#### 一、过滤（Filtering）

| 操作符 | 作用 | 签名 |
|------|------|------|
| **Where** | 按条件筛选 | `IEnumerable<T> → IEnumerable<T>` |
| **OfType<T>** | 按类型筛选 | `IEnumerable → IEnumerable<T>` |

```csharp
// Where：条件过滤
var top = players.Where(p => p.Score > 500 && p.IsActive);

// OfType：类型过滤（比如从混合集合中提取特定类型）
var shapes = new List<object> { new Circle(), new Rectangle(), new Circle() };
var circles = shapes.OfType<Circle>();  // 2 个 Circle，跳过 Rectangle
```

---

#### 二、投影（Projection）

| 操作符 | 作用 |
|------|------|
| **Select** | 一对一映射（map） |
| **SelectMany** | 一对多展开（flatMap） |

```csharp
// Select：把 Player 映射为 Name
var names = players.Select(p => p.Name);

// Select + 匿名类型（最常用）
var info = players.Select(p => new { p.Name, p.Score, Level = p.Score / 100 });

// Select + 带索引重载（index 从 0 开始）
var ranked = players.Select((p, i) => new { Rank = i + 1, p.Name, p.Score });

// SelectMany：展平嵌套集合（每个玩家有多件武器 → 得到所有武器列表）
var allWeapons = players.SelectMany(p => p.Weapons);
// 等价于：
var allWeapons = from p in players
                 from w in p.Weapons    // 查询表达式用第二个 from
                 select w;
```

---

#### 三、排序（Ordering）

| 操作符 | 作用 |
|------|------|
| **OrderBy** | 升序排序 |
| **OrderByDescending** | 降序排序 |
| **ThenBy** | 次要排序（升序） |
| **ThenByDescending** | 次要排序（降序） |
| **Reverse** | 反转顺序 |

```csharp
// 单字段排序
var sorted = players.OrderBy(p => p.Score);

// 多字段排序：先按Team升序，同队内按Score降序
var sorted = players
    .OrderBy(p => p.Team)               // 主排序
    .ThenByDescending(p => p.Score);    // 次排序（只有同 Team 才比 Score）

// 注意：不要连续用 OrderBy！第二个 OrderBy 会覆盖第一个！
// ❌ players.OrderBy(p => p.Team).OrderBy(p => p.Score)  ← 等价于只按 Score 排
// ✅ players.OrderBy(p => p.Team).ThenBy(p => p.Score)   ← 正确
```

---

#### 四、分组（Grouping）

| 操作符 | 作用 |
|------|------|
| **GroupBy** | 按 Key 分组 |
| **ToLookup** | 按 Key 分组（立即执行，产生 Lookup<TKey,T>） |

```csharp
// GroupBy：返回 IEnumerable<IGrouping<Key, Element>>
var teams = players.GroupBy(p => p.Team);
foreach (var team in teams) {
    Console.WriteLine($"Team {team.Key}: {team.Count()} players");
    foreach (var p in team) Console.WriteLine($"  - {p.Name}");
}

// GroupBy + 投影：分组后直接聚合
var teamStats = players
    .GroupBy(p => p.Team,
             p => p,                    // 元素选择器
             (key, group) => new {      // 结果选择器
                 Team = key,
                 Count = group.Count(),
                 AvgScore = group.Average(p => p.Score),
                 TopPlayer = group.MaxBy(p => p.Score)?.Name
             });

// 查询表达式中的 group by
var result = from p in players
             group p by p.Team into g
             select new { Team = g.Key, AvgScore = g.Average(p => p.Score) };
```

---

#### 五、连接（Joining）

| 操作符 | 作用 |
|------|------|
| **Join** | 内连接（两表 Key 匹配才出结果） |
| **GroupJoin** | 分组连接（左表每条 + 右表匹配的集合） |

```csharp
// Join：内连接
var playerTeams = players.Join(
    teams,                     // 内表
    p => p.TeamId,             // 外表键选择器
    t => t.Id,                 // 内表键选择器
    (p, t) => new { p.Name, TeamName = t.Name }  // 结果选择器
);

// 查询表达式 Join（更直观）
var result = from p in players
             join t in teams on p.TeamId equals t.Id
             where t.IsActive
             select new { p.Name, TeamName = t.Name };

// GroupJoin：左连接效果（包含没有匹配的关系）
var result = from t in teams
             join p in players on t.Id equals p.TeamId into teamPlayers  // into = GroupJoin
             select new { Team = t.Name, Players = teamPlayers };

// 左连接（包含没有匹配的 Left）
var result = from p in players
             join t in teams on p.TeamId equals t.Id into pt
             from t in pt.DefaultIfEmpty()  // 无匹配时给 null
             select new { p.Name, TeamName = t?.Name ?? "No Team" };
```

---

#### 六、聚合（Aggregation）

| 操作符 | 作用 | 返回值 |
|------|------|------|
| **Count** | 元素数量 | int |
| **Sum** | 求和 | 数值类型 |
| **Average** | 平均值 | double |
| **Min / Max** | 最小/最大值 | T |
| **MinBy / MaxBy** | 按条件取最小/最大元素 | T（返回元素本身） |
| **Aggregate** | 自定义累积（fold/reduce） | 任意 |

```csharp
// 常规聚合
int total = players.Sum(p => p.Score);
double avg = players.Average(p => p.Score);
int count = players.Count(p => p.IsActive);

// MaxBy：取分数最高的玩家本身（不是分数值）
var mvp = players.MaxBy(p => p.Score);  // 返回 Player 对象

// Aggregate（自定义累加）—— 功能最强的聚合
// 累加字符串
string allNames = players.Aggregate(
    "",                          // seed 初始值
    (current, p) => current + p.Name + ", ",  // 累加函数
    result => result.TrimEnd(',', ' ')        // 结果转换
);

// Aggregate 计算阶乘
int factorial = Enumerable.Range(1, 5).Aggregate(1, (acc, n) => acc * n);  // 120
```

---

#### 七、量词（Quantifiers）

| 操作符 | 作用 |
|------|------|
| **Any** | 是否存在任一满足条件（空集合返回 false） |
| **All** | 是否全部满足条件（空集合返回 true） |
| **Contains** | 是否包含指定元素 |

```csharp
bool hasVIP = players.Any(p => p.IsVIP);          // 有没有 VIP 玩家？
bool allAlive = players.All(p => p.Health > 0);   // 所有玩家都活着？
bool hasAlice = players.Select(p => p.Name).Contains("Alice");

// ⚠️ Any() 比 Count() > 0 效率高！
// ❌ if (players.Where(p => p.Score > 500).Count() > 0)  // 遍历所有人
// ✅ if (players.Any(p => p.Score > 500))                  // 找到一个就返回
```

---

#### 八、分区（Partitioning / 分页）

| 操作符 | 作用 |
|------|------|
| **Take** | 取前 n 个 |
| **Skip** | 跳过前 n 个 |
| **TakeWhile** | 一直取直到条件为 false |
| **SkipWhile** | 一直跳直到条件为 false |

```csharp
// 分页（第 page 页，每页 size 条）
var page = players.OrderBy(p => p.Name).Skip((page - 1) * size).Take(size);

// TakeWhile：取连续满足条件的开头部分（一旦不满足就停）
var numbers = new[] { 2, 4, 6, 7, 8, 10 };
var evens = numbers.TakeWhile(n => n % 2 == 0);  // {2, 4, 6}，遇到 7 停止

// SkipWhile：跳过开头满足条件的
var rest = numbers.SkipWhile(n => n % 2 == 0);    // {7, 8, 10}
```

---

#### 九、集合操作（Set Operations）

| 操作符 | 作用 |
|------|------|
| **Distinct** | 去重 |
| **Union** | 并集 |
| **Intersect** | 交集 |
| **Except** | 差集 |

```csharp
var a = new[] { 1, 2, 3, 3 };
var b = new[] { 2, 3, 4 };

a.Distinct();          // {1, 2, 3}
a.Union(b);            // {1, 2, 3, 4}
a.Intersect(b);        // {2, 3}
a.Except(b);           // {1}

// Distinct 按指定字段去重
var uniqueNames = players.Select(p => p.Name).Distinct();
// C# 6+ DistinctBy（更简洁）
var unique = players.DistinctBy(p => p.TeamId);  // 每个 Team 只取一个
```

---

#### 十、元素操作（Element Operations）

| 操作符 | 作用 | 空集合/无匹配行为 |
|------|------|------|
| **First** | 第一个元素 | 抛 InvalidOperationException |
| **FirstOrDefault** | 第一个元素或默认值 | 返回 default(T) |
| **Single** | 唯一元素 | 无元素/多个元素→抛异常 |
| **SingleOrDefault** | 唯一元素或默认值 | 多个元素→抛异常 |
| **Last / LastOrDefault** | 最后一个元素 | 同上 |
| **ElementAt / ElementAtOrDefault** | 按索引取 | 越界→异常/default |

```csharp
var mvp = players.First(p => p.Score > 900);         // 没找到→异常
var mvp = players.FirstOrDefault(p => p.Score > 900); // 没找到→null

// ⚠️ Single vs First
var admin = players.Single(p => p.Role == "Admin");
// Single 断言"有且仅有一个"——如果是 0 个或 2+ 个都抛异常
// 用于"必须唯一"的业务约束

// 性能注意：First 找到一个就停止；Single 即使找到也要确认没有第二个
```

---

#### 十一、生成（Generation）

| 操作符 | 作用 |
|------|------|
| **Range** | 生成连续整数序列 |
| **Repeat** | 重复某元素 n 次 |
| **Empty** | 返回空序列 |

```csharp
var numbers = Enumerable.Range(1, 10);   // {1, 2, 3, ..., 10}
var zeros = Enumerable.Repeat(0, 5);      // {0, 0, 0, 0, 0}
var empty = Enumerable.Empty<int>();       // {}
```

---

#### 十二、转换（Conversion）

| 操作符 | 作用 | 时机 |
|------|------|------|
| **ToList** | → List\<T\> | 立即执行 |
| **ToArray** | → T[] | 立即执行 |
| **ToDictionary** | → Dictionary | 立即执行 |
| **ToLookup** | → ILookup (类似 Dictionary<T, List\<T\>>) | 立即执行 |
| **Cast\<T\>** | 强制转型（非泛型→泛型） | 延迟执行 |
| **AsEnumerable** | 转回 IEnumerable（降级 IQueryable→IEnumerable） | 延迟执行 |

```csharp
// ToDictionary：Key 必须唯一！重复 Key → 抛异常
var dict = players.ToDictionary(p => p.Id);     // Key=Id, Value=Player
var dict = players.ToDictionary(p => p.Id, p => p.Name);  // Key=Id, Value=Name

// ToLookup：允许重复 Key（类似多值字典）
var lookup = players.ToLookup(p => p.Team);  // ILookup<string, Player>
var teamA = lookup["TeamA"];   // IEnumerable<Player>，可为空

// Cast vs OfType
ArrayList list = new ArrayList { 1, "hello", 2 };
list.Cast<int>();           // InvalidCastException！"hello" 不能转 int
list.OfType<int>();         // {1, 2} —— 跳过不能转的元素
```

---

### 16.3 延迟执行 vs 立即执行（★★★★★）

这是 LINQ 最重要的核心概念。操作符分为两类：

| 类型 | 执行时机 | 典型操作符 |
|------|------|------|
| **延迟（Deferred）** | 遍历时才执行，每次遍历都重新算 | Where, Select, OrderBy, GroupBy, Join, Take, Skip, Cast, Distinct, Reverse |
| **立即（Immediate）** | 调用时马上执行，结果缓存在内存 | ToList, ToArray, ToDictionary, Count, Sum, First, Single, Any, All, Max, Min, Aggregate |

```csharp
// 延迟执行的核心机制：返回的是迭代器，不是结果集
var query = players.Where(p => p.Score > 100);
// query 的类型是 IEnumerable<Player>，内部持有 WhereListIterator
// 此时 Where 的 Lambda 还没执行过

// 第一次遍历 → 执行 Where
foreach (var p in query) Console.WriteLine(p.Name);

// 第二次遍历 → 重新执行 Where！
// 如果两次遍历之间 players 变了，结果也不同

// 立即执行 → 拍平为 List
var list = query.ToList();  // 此时执行，结果固定
foreach (var p in list) { } // 不再重新执行
```

**延迟执行的两大陷阱**：

```csharp
// 陷阱 1：捕获变量（闭包问题）
int threshold = 100;
var query = players.Where(p => p.Score > threshold);
threshold = 500;  // 改了！
foreach (var p in query) { }  // 按 threshold=500 执行！
// 修复：立即执行
var result = players.Where(p => p.Score > threshold).ToList();  // 按 100 执行

// 陷阱 2：多次遍历（数据库场景灾难）
IQueryable<Player> query = db.Players.Where(p => p.Score > 100);
var count = query.Count();    // SELECT COUNT(*) ...    ← 1 次 DB 查询
var names = query.ToList();   // SELECT * FROM ...      ← 又 1 次 DB 查询！
// 修复：ToList 一次，后续操作都在内存中
var list = db.Players.Where(p => p.Score > 100).ToList();
var count = list.Count;        // 内存操作
var names = list;               // 内存操作
```

---

### 16.4 IEnumerable vs IQueryable（★★★★★）

> **一分钟回答**：IEnumerable 在客户端内存执行（Linq to Objects），Lambda 编译为 IL 委托直接运行；IQueryable 将 Lambda 捕获为**表达式树（Expression Tree）**，由 Provider 翻译为 SQL/其他查询语言在远端执行。

```csharp
// IQueryable 表达式树原理
IQueryable<Player> query = db.Players.Where(p => p.Score > 100);
// 编译器看到 IQueryable 的 Where → 不把 p => p.Score > 100 编译为委托
// 而是生成 Expression<Func<Player, bool>> 表达式树：
//   Expression.Lambda(
//       Expression.GreaterThan(
//           Expression.Property(parameter, "Score"),
//           Expression.Constant(100)
//       ),
//       parameter
//   )
//
// EF Core 的 Provider 把表达式树 → SQL：
//   SELECT * FROM Players WHERE Score > 100
```

| | IEnumerable\<T\> | IQueryable\<T\> |
|------|------|------|
| **命名空间** | System.Collections.Generic | System.Linq |
| **执行位置** | 客户端内存 | 远端（数据库等） |
| **Lambda 编译为** | IL 委托（直接执行） | Expression Tree（可翻译） |
| **数据量** | 全量加载到内存后再筛选 | 远端筛选后只返回结果 |
| **适用场景** | 内存集合（List, Array, Dictionary） | Entity Framework / LINQ to SQL |

```csharp
// ❌ 典型错误：IQueryable 提前 IEnumerable 化 → 全表扫描
var allPlayers = db.Players.ToList();         // SELECT * 全量加载到内存！
var filtered = allPlayers.Where(p => p.Score > 100);  // 在 C# 内存中筛选（晚矣）

// ✅ 正确：保持 IQueryable 到最后
var filtered = db.Players
    .Where(p => p.Score > 100)
    .OrderBy(p => p.Name)
    .Take(10)
    .ToList();  // 生成 SQL：SELECT TOP 10 * FROM Players WHERE Score>100 ORDER BY Name
```

**面试追问**：AsEnumerable() 的作用？
> 把 IQueryable 降级为 IEnumerable，后续操作走客户端内存而非数据库。用于"数据库做完它擅长的过滤，剩下的在 C# 做"的场景（如调用数据库不支持的 C# 函数）。

---

### 16.5 Let / Into 关键字（查询表达式专属）

```csharp
// let：引入中间变量（避免重复计算）
var result = from p in players
             let bonus = CalculateBonus(p)  // 只算一次
             where bonus > 100
             orderby bonus descending
             select new { p.Name, Bonus = bonus };
// 等价方法语法：Select 创建匿名类型携带中间值
var result = players
    .Select(p => new { Player = p, Bonus = CalculateBonus(p) })
    .Where(x => x.Bonus > 100)
    .OrderByDescending(x => x.Bonus)
    .Select(x => new { x.Player.Name, x.Bonus });

// into：在 group/join/select 后继续查询（投影后接新的查询范围）
var result = from p in players
             group p by p.Team into g          // into g = 把分组结果命名为 g
             where g.Count() >= 3              // 只保留 ≥3 人的组
             select new { Team = g.Key, Count = g.Count() };
```

---

### 16.6 LINQ 内部机制：迭代器组合

每个 LINQ 操作符返回一个新的迭代器，形成**迭代器链**：

```
players (List<Player>)
   │
   ▼ .Where(p => p.Score > 100)
WhereListIterator<Player>        ← 持有 _source=List, _predicate=Lambda
   │
   ▼ .Select(p => p.Name)
SelectListIterator<Player, string> ← 持有 _source=Where迭代器, _selector=Lambda
   │
   ▼ .Take(10)
TakeIterator<string>              ← 持有 _source=Select迭代器, _count=10
   │
   ▼ foreach 触发
MoveNext() 从 Take → Select → Where → List 逐层向上拉数据
```

**每个迭代器只在上游拉数据时才会被执行**——这就是延迟执行的本质。

```csharp
// 可视化迭代器链的执行过程
var result = players
    .Where(p => p.Score > 100)   // 迭代器 1
    .Select(p => p.Name)          // 迭代器 2
    .Take(2);                     // 迭代器 3

foreach (var name in result) { }
// 执行序列：
// 1. Take.MoveNext() → 需要数据
// 2. Select.MoveNext() → 需要上游数据
// 3. Where.MoveNext() → 遍历 players，找到 Score>100 的 → 返回
// 4. Select 对返回值执行 p.Name → 返回给 Take
// 5. Take 计数 → 没到 2 条 → 再拉
// 6. 重复 2~5，直到 Take 拿够 2 条 → 停止
// 注意：不满足 Take 的后续 player 根本不会被 Where 遍历！
```

---

### 16.7 LINQ 性能实践

#### 何时用 LINQ vs 手写循环

| 场景 | 推荐 | 原因 |
|------|------|------|
| 原型/非热路径 | LINQ | 可读性 > 性能 |
| 数据处理/工具 | LINQ | 简洁、不易出错 |
| Unity Update/FixedUpdate | 手写 for | LINQ 每帧分配迭代器/闭包 |
| 数据库查询 | LINQ (IQueryable) | 自动翻译 SQL |
| 高频数组操作 | 手写 for | 避免委托调用开销 |

```csharp
// LINQ 的 GC 分配来源
var result = list
    .Where(x => x > 0)       // 分配：WhereListIterator + 闭包捕获的 DisplayClass
    .Select(x => x * 2)      // 分配：SelectListIterator + 闭包
    .ToList();               // 分配：新 List<T>

// 热路径手写（零分配）
var result = new List<int>();
for (int i = 0; i < list.Count; i++) {
    if (list[i] > 0) result.Add(list[i] * 2);
}
```

#### 常见性能坑

```csharp
// ❌ 坑 1：Count() 比 .Count 属性慢
list.Count();    // LINQ 扩展方法 → 遍历整个集合（O(n)）
list.Count;       // 属性 → O(1)
// .Count 是 ICollection<T> 的属性，直接读取内部 _size 字段

// ❌ 坑 2：重复遍历
var filtered = list.Where(x => x > 0);
var cnt = filtered.Count();       // 遍历 1
var arr = filtered.ToArray();     // 遍历 2
var sum = filtered.Sum();         // 遍历 3
// ✅ 立即拍平
var arr = list.Where(x => x > 0).ToArray();
var cnt = arr.Length;   // O(1)
var sum = arr.Sum();    // 遍历 1（走 IEnumerable → 不可避免）

// ❌ 坑 3：OrderBy().First() 笨重排序
var min = list.OrderBy(x => x).First();  // 排序 O(n log n)
var min = list.Min();                     // 遍历 O(n) —— 好得多！

// ❌ 坑 4：Where().First() vs First(predicate)
var x = list.Where(p).First();  // 创建迭代器 → 拉数据
var x = list.First(p);          // 直接遍历，少一层迭代器包装
```

---

### 16.8 速查表：50+ LINQ 操作符一览

```
┌──────────────────────────────────────────────────────────┐
│  分类          │ 操作符                                   │
├──────────────────────────────────────────────────────────┤
│  过滤          │ Where          OfType                    │
│  投影          │ Select         SelectMany               │
│  排序          │ OrderBy        OrderByDescending        │
│               │ ThenBy         ThenByDescending  Reverse │
│  分组          │ GroupBy        ToLookup                 │
│  连接          │ Join           GroupJoin                │
│  聚合          │ Count  Sum  Average  Min  Max           │
│               │ MinBy  MaxBy    Aggregate               │
│  量词          │ Any  All  Contains                       │
│  分区          │ Take  Skip     TakeWhile  SkipWhile     │
│  集合操作      │ Distinct  Union  Intersect  Except      │
│  元素操作      │ First  FirstOrDefault                   │
│               │ Single  SingleOrDefault                  │
│               │ Last  LastOrDefault                      │
│               │ ElementAt  ElementAtOrDefault            │
│               │ DefaultIfEmpty                           │
│  生成          │ Range  Repeat  Empty                     │
│  转换          │ ToList  ToArray  ToDictionary  ToLookup │
│               │ Cast  OfType  AsEnumerable               │
│  串联          │ Concat  Append  Prepend                  │
│  相等          │ SequenceEqual                            │
│  压缩          │ Zip                                      │
└──────────────────────────────────────────────────────────┘
```

---

## 第十七章 IEnumerable 与 IEnumerator

### 17.1 foreach 底层原理

foreach 是语法糖，编译器将其展开为调用 `GetEnumerator()` → `while (enumerator.MoveNext())` → `current`。任何实现了 IEnumerable 或满足"鸭子类型"模式的类型都可以用 foreach。

```csharp
// foreach 写法
foreach (var item in collection) { Console.WriteLine(item); }

// 编译器展开
IEnumerator<int> enumerator = collection.GetEnumerator();
try {
    while (enumerator.MoveNext()) {
        int item = enumerator.Current;
        Console.WriteLine(item);
    }
} finally { enumerator?.Dispose(); }
```

### 17.2 IEnumerable vs IEnumerator

| | IEnumerable | IEnumerator |
|------|------|------|
| **作用** | 可被 foreach 的条件 | 实际迭代器（游标） |
| **关系** | "给我一个迭代器" | "我是迭代器" |
| **多次遍历** | 每次 GetEnumerator() 新迭代器 | 单向游标 |
| **类比** | 书架 | 书签（第几页） |

### 17.3 yield return — 迭代器状态机

yield return 让方法变成迭代器，编译器自动生成一个实现了 IEnumerator 的状态机类。每次 yield return = 返回一个值并"暂停"，下次 MoveNext = 从上次位置继续。

```csharp
public IEnumerable<int> GetNumbers() {
    yield return 1;   // 暂停点 1
    yield return 2;   // 暂停点 2
}

// 编译器生成的状态机（简化）
class GetNumbers_StateMachine : IEnumerator<int> {
    private int _state;       // 0=初始, 1=暂停点1, 2=暂停点2, -1=结束
    private int _current;

    public bool MoveNext() {
        switch (_state) {
            case 0: _current = 1; _state = 1; return true;
            case 1: _current = 2; _state = 2; return true;
            case 2: _state = -1; return false;
        }
    }
    public int Current => _current;
}
```

### 17.4 foreach vs for 性能对比

- 数组上 for 快 5-15%（直接索引）；foreach 编译器对数组做了优化直接用 for
- List\<T\> 上 for 略快（List 的索引器不能内联）
- 一般区别不大，优先关注可读性
- foreach 对 List 有版本检测（修改集合会抛异常）

### 17.5 Unity 协程与 yield

Unity 协程借用了 IEnumerator + yield return 的暂停/恢复机制，但语义完全不同——Unity 协程的 yield 控制的是**时间**（等一帧 / 等 N 秒），而非数据迭代。

```csharp
// Unity 协程：yield return 表示"暂停，等条件满足再继续"
IEnumerator MyCoroutine() {
    yield return new WaitForSeconds(1);  // 暂停 1 秒
    yield return null;                    // 暂停一帧
    yield return new WaitUntil(() => isReady);  // 等待条件
}
```

---

## 第十八章 async / await

### 18.1 async/await 是什么？

async 标记方法可能是异步的；await 在异步操作完成前"暂停"当前方法，释放线程去干别的，完成后"恢复"继续执行。编译器将 async 方法转为状态机。

### 18.2 Task 的几种状态

Task 状态：Created → WaitingForActivation → Running → RanToCompletion / Faulted / Canceled。Task.Run() 直接进入 Running。

Task vs ValueTask：Task 是引用类型，堆分配；ValueTask（C# 7+）是值类型，可能不分配堆，适合高频调用。

### 18.3 await 背后的状态机

编译器将 async 方法转化为一个实现了 `IAsyncStateMachine` 的结构体。await 处 = 暂停点（将剩余代码作为回调注册到 Task）。Task 完成后 → 回调被调 → 状态机 MoveNext → 继续执行。

```csharp
// 编译器生成（简化）
class FooStateMachine : IAsyncStateMachine {
    int _state = 0;
    TaskAwaiter<string> _awaiter;
    string _r1, _r2;

    void MoveNext() {
        switch (_state) {
            case 0:
                _awaiter = DownloadAsync("url1").GetAwaiter();
                if (_awaiter.IsCompleted) goto case 1;
                _state = 1;
                _awaiter.OnCompleted(MoveNext); // 注册回调
                return;                          // 释放线程！
            case 1:
                _r1 = _awaiter.GetResult();
                // ... 继续下一个 await
        }
    }
}
```

### 18.4 SynchronizationContext

SynchronizationContext 决定 await 后代码"跑在哪个线程"。

| 环境 | SynchronizationContext | await 后回到 |
|------|------|------|
| **WPF/WinForms** | DispatcherSynchronizationContext | UI 线程 |
| **ASP.NET Core** | **null** | ThreadPool 线程 |
| **Unity** | UnitySynchronizationContext | 主线程 |
| **控制台** | null | ThreadPool 线程 |

### 18.5 ConfigureAwait(false)

让 await 后**不回到原 SynchronizationContext**，在任何线程继续执行。库代码应使用此方法避免 UI 死锁；UI/Unity 代码通常不配置（需要回到主线程更新 UI）。

### 18.6 死锁经典场景

UI 线程调 `task.Result` → 阻塞等待 → await 完成后要回 UI 线程执行 → UI 线程被阻塞无法接收 → 死锁。

解决方案：全部 await 到底，不调 .Result/.Wait()；库代码中用 ConfigureAwait(false)。

### 18.7 async void 的危险

async void 方法无法被 await（调用方无法等待），异常无法捕获（抛到 SynchronizationContext → 直接崩溃）。除了 UI 事件处理器，永远不要用 async void。

### 18.8 Unity + async/await + UniTask

Unity 的 SynchronizationContext 让 await 后自动回到主线程，但不能直接 await Unity 的 AsyncOperation 等非 Task 类型。**UniTask** 是社区方案：零 GC 分配、支持 Unity 对象生命周期（Cancel 在 Destroy 时）、比原生 Task 快。

| | Task | UniTask |
|------|------|------|
| **GC 分配** | 每次 await 分配 Task | 零分配（struct） |
| **Unity 生命周期** | ❌ 不感知 | ✅ Destroy 自动取消 |
| **async void 安全** | ❌ | ✅ UniTaskVoid 可捕获异常 |
| **Unity 异步 API** | 需包装 | 直接 await |

---

# 第三部分：线程、反射、集合与 Unity 实战

## 第十九章 Thread

### 19.1 Thread vs ThreadPool vs Task

| | Thread | ThreadPool | Task |
|------|------|------|------|
| **创建开销** | 高（~1MB 栈 + 内核对象） | 低（复用） | 低（基于 ThreadPool） |
| **管理** | 手动 Start/Join/Abort | 自动调度 | async/await 状态机 |
| **返回值** | ❌ | ❌ | ✅ Task\<T\> |
| **异常传播** | 难 | 难 | ✅ AggregateException |
| **取消** | 手动 flag | ❌ | ✅ CancellationToken |
| **适用** | 长期运行的后台线程 | 短任务 | 所有异步场景 |

现代 C# 首选 Task。

### 19.2 lock 的底层实现

lock 是 Monitor.Enter/Exit 的语法糖。编译后 = `try { Monitor.Enter(obj, ref lockTaken); ... } finally { if (lockTaken) Monitor.Exit(obj); }`。锁是基于对象的 sync block（同步块索引）。

```csharp
// ✅ 专用锁对象，不锁 this/typeof/string
private readonly object _lock = new object();
lock (_lock) { }
```

### 19.3 Monitor

Monitor 是 lock 的底层 API。相比 lock 多了 `Monitor.Wait/Pulse/PulseAll`（类似条件变量），用于线程间通知。

### 19.4 Mutex vs Semaphore

| | lock/Monitor | Mutex | Semaphore |
|------|------|------|------|
| **范围** | 单进程 | **跨进程** | 跨进程/单进程 |
| **并发数** | 1 | 1 | N（可配置） |
| **性能** | 最快 | 慢（内核对象） | 中等 |
| **用途** | 临界区保护 | 单实例程序 | 限流 |

### 19.5 ConcurrentDictionary

线程安全的字典，内部用细粒度锁（lock per bucket）而非全局锁，并发性能远超 `lock + Dictionary`。.NET Core 中用更细粒度的 lock-free 操作。`TryAdd/GetOrAdd/AddOrUpdate` 是原子操作。

### 19.6 死锁四个条件

死锁需同时满足四个条件：① 互斥（资源只能独占）、② 持有并等待（持有资源等其他资源）、③ 不可抢占（资源不能被抢走）、④ 循环等待（A 等 B，B 等 A）。破坏任一条件即可避免死锁。

预防方法：所有线程统一加锁顺序。

---

## 第二十章 Reflection

### 20.1 反射是什么？

反射 = 运行时获取类型信息并动态操作（创建对象、调用方法、访问字段/属性）。核心在 `System.Reflection` 命名空间。

| 核心类 | 作用 |
|------|------|
| **Type** | 类型信息（typeof / obj.GetType()） |
| **Assembly** | 程序集入口 |
| **MethodInfo** | 方法信息，动态调用 |
| **PropertyInfo** | 属性信息 |
| **FieldInfo** | 字段信息 |
| **Activator** | 动态创建对象实例 |

获取 Type 的 4 种方式：`typeof(Player)`（编译时已知）、`obj.GetType()`（通过实例）、`Type.GetType("MyApp.Player")`（字符串）、`Assembly.LoadFrom("dll").GetType(...)`（动态加载）。

### 20.2 动态创建对象

```csharp
// 1. Activator：最简单
object obj = Activator.CreateInstance(typeof(Player));

// 2. ConstructorInfo：有参构造
ConstructorInfo ctor = typeof(Player).GetConstructor(new[] { typeof(string), typeof(int) });
object obj = ctor.Invoke(new object[] { "Alice", 25 });

// 3. 泛型 Activator：返回强类型
Player p = Activator.CreateInstance<Player>();

// 4. 表达式树编译：最快！
var ctor = typeof(Player).GetConstructor(Type.EmptyTypes);
var newExpr = Expression.New(ctor);
var lambda = Expression.Lambda<Func<Player>>(newExpr).Compile();
Player p = lambda();  // 几乎跟 new Player() 一样快！
```

### 20.3 反射的性能开销

反射慢的原因：① 按名称查找（字符串比较走 Metadata 表）；② 参数/返回值打包拆包（object[] → 实际类型）；③ 安全检查（调用者权限验证）。核心瓶颈是 MethodInfo.Invoke。

```
new Player()              ~1ns
Activator.CreateInstance  ~100ns (×100)
ctor.Invoke               ~200ns
Expression.Compile()      ~2ns (仅首次编译有开销)
```

### 20.4 性能优化方案

| 方案 | 原理 | 提升 |
|------|------|------|
| **缓存** | Type/MethodInfo 只获取一次 | 避免重复查找 |
| **Delegate.CreateDelegate** | 反射转强类型委托 | ~50x |
| **表达式树编译** | 生成编译后的 Lambda | ~100x |
| **Emit** | 直接生成 IL | ~200x（但复杂） |
| **Source Generator** | 编译时生成代码 | 零反射开销 |

### 20.5 应用场景

插件系统、DI 框架、ORM、序列化、单元测试（访问私有成员）、Attribute 处理。

---

## 第二十一章 Attribute

### 21.1 Attribute 是什么？

Attribute（特性）是附加在类型、方法、属性等元素上的元数据标记，运行时可通过反射获取。不影响代码执行逻辑，由框架/工具按需解析。相当于"代码的标签"。

```csharp
[Serializable]
[Obsolete("旧方法")]
public class Player {
    [SerializeField] private int health;
    [DllImport("kernel32")] static extern IntPtr GetModuleHandle(string name);
}
```

### 21.2 自定义 Attribute

继承 `System.Attribute`，命名以 "Attribute" 结尾。可以定义构造函数参数（位置参数）和属性（命名参数）。

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = true)]
public class AuthorAttribute : Attribute {
    public string Name { get; }
    public string Date { get; set; }
    public AuthorAttribute(string name) { Name = name; }
}

[Author("张三", Date = "2024-01-15")]
public class MyClass { }
```

### 21.3 AttributeUsage

限制 Attribute 可以用在哪些目标上、是否允许多次应用、是否被子类继承。

| 常见 AttributeTargets | 含义 |
|------|------|
| Class、Struct、Enum | 类型 |
| Method、Property、Field | 成员 |
| Parameter | 方法参数 |
| Assembly | 整个程序集 |
| All | 所有目标 |

### 21.4 Unity 常见 Attribute

| Attribute | 作用 |
|------|------|
| `[SerializeField]` | 私有字段在 Inspector 可见 |
| `[HideInInspector]` | public 字段在 Inspector 隐藏 |
| `[Range(0, 100)]` | Inspector 中显示滑动条 |
| `[Header("标题")]` | Inspector 中加标题分隔 |
| `[Tooltip("提示")]` | 鼠标悬停提示 |
| `[RequireComponent(typeof(Rigidbody))]` | 自动添加依赖组件 |
| `[ExecuteAlways]` | 编辑器模式下也运行 |
| `[MenuItem("菜单路径")]` | 编辑器菜单项 |
| `[CreateAssetMenu]` | 创建 ScriptableObject 资源的菜单 |

---

## 第二十二章 Assembly

### 22.1 Assembly 是什么？

Assembly（程序集）= .dll（类库）或 .exe（可执行文件）。是 .NET 的部署、版本控制、安全边界的最小单元。包含 IL 代码 + Metadata + 资源清单。

### 22.2 .dll 与 .csproj

.csproj 是项目文件（描述源码、引用、配置），编译后产出 .dll（Assembly）。一个 .csproj 编译为一个 Assembly。

### 22.3 Unity 的 asmdef

asmdef（Assembly Definition）让 Unity 项目可以拆分为多个独立 Assembly，而非全部打进一个巨大的 Assembly-CSharp.dll。

好处：
1. 加速编译：改了 UI 脚本 → 只重编 UI.dll
2. 强制依赖管理：UI.asmdef 引用 Core.asmdef → UI 能访问 Core
3. 循环依赖检测：A 引用 B，B 引用 A → 编译报错
4. 代码隔离：不同 Assembly 间 internal 不可见

### 22.4 Assembly-CSharp

Unity 项目中所有不在 Editor 文件夹且没配 asmdef 的脚本，默认编译到 Assembly-CSharp.dll（运行时）或 Assembly-CSharp-Editor.dll（编辑器）。

### 22.5 程序集加载机制

CLR 按需加载 Assembly——首次引用某程序集的类型时，触发加载。探测顺序：GAC → 应用基础目录 → 子目录 → AssemblyLoadContext.Resolving。

Unity 中的 Assembly 加载：引擎 Assembly（UnityEngine.dll）启动时自动加载；项目 Assembly 在初始化时加载；asmdef 产生的 Assembly 按依赖顺序加载；第三方 .dll（Plugins 目录）在 Assembly-CSharp 之前加载。

---

## 第二十三章 Unity 与 CLR

### 23.1 Unity 生命周期

MonoBehaviour 有严格的生命周期，由 Unity 引擎按固定顺序调用：

```
初始化阶段:           每帧循环:               销毁阶段:
  Awake()               FixedUpdate()           OnDisable()
  OnEnable()             Update()               OnDestroy()
  Start()                LateUpdate()
```

完整顺序：Awake → OnEnable → Start → ↻ FixedUpdate → ↻ Update → ↻ LateUpdate → OnDisable → OnDestroy

### 23.2 MonoBehaviour 为什么不能 new？

MonoBehaviour 需要绑定到 GameObject 上，由 Unity 引擎内部创建并管理其生命周期。new 出来的 MonoBehaviour 不挂载在任何 GameObject 上 → Awake/Start/Update 都不会被调 → 各内部引用为 null。

```csharp
// ❌ var player = new Player();  // 不能 new！
// ✅ var player = gameObject.AddComponent<Player>();
// ✅ var player = Instantiate(playerPrefab);
```

### 23.3 ScriptableObject

ScriptableObject 是 Unity 的数据容器类，可以在编辑器中创建独立 .asset 文件存储数据。适合配置表、技能数据、道具数据——不需要 GameObject，不占每帧开销。

ScriptableObject vs JSON 配置：引擎内编辑拖拽 vs 外部工具编辑；类型安全引用资源 vs 字符串字段；二进制 .asset vs 文本。

### 23.4 IL2CPP 工作原理

IL2CPP 将 IL 转为 C++ 再编译为本机代码。两步：① IL2CPP.exe：IL → C++（静态分析 + 优化）；② 平台 C++ 编译器：C++ → 机器码。

优点：iOS 兼容（无 JIT）、性能提升 1.5-2x、Strip Engine Code、难以反编译。缺点：System.Reflection.Emit 不支持、编译时间更长、包体略大。

### 23.5 Burst Compiler

Burst 是 Unity 的 LLVM 编译器，将 C# 子集（HPC#）编译为高度优化的本机代码。配合 Jobs System 使用，SIMD 优化 + LLVM 优化通道 → 数倍到数十倍性能提升。

| | 普通 C# (Mono/IL2CPP) | Burst |
|------|------|------|
| **编译器** | Mono JIT / IL2CPP C++ | LLVM |
| **SIMD** | 手动 | 自动向量化 |
| **性能** | 基准 | 数倍~数十倍 |
| **限制** | 完整 C# | HPC# 子集（无 class、无异常处理） |

### 23.6 Jobs System

Unity Jobs System 是多线程任务调度框架，将工作分配到多个 Worker Thread 并行执行。必须用 NativeContainer（NativeArray 等）传递数据，保证线程安全。

```csharp
[BurstCompile]
struct MultiplyParallelJob : IJobParallelFor {
    public NativeArray<float> values;
    public float multiplier;
    public void Execute(int index) { values[index] *= multiplier; }
}
var handle = job.Schedule(array.Length, 64);  // 64 = batch size
handle.Complete();  // 等所有线程完成
```

### 23.7 NativeArray / Unsafe

NativeArray 是非托管的连续内存数组，用于 Job 间传递数据，有 Safety Handle 检测竞态条件。需手动 Dispose。Unsafe 代码允许绕过安全检查直接操作内存。

---

## 第二十四章 常用集合（★★★★★）

> **一分钟回答**：C# 集合体系分三层——① 泛型集合（List\<T\>、Dictionary\<TKey,TValue\>、HashSet\<T\>、Queue\<T\>、Stack\<T\>、LinkedList\<T\>）；② 线程安全集合（ConcurrentDictionary、ConcurrentQueue、ConcurrentBag）；③ 非泛型旧集合（ArrayList、Hashtable，已淘汰）。选型关键看：是否需要索引、是否需要 Key-Value、是否需要排序、是否需要线程安全、容量是否已知。

---

### 24.1 C# 集合 vs C++ STL 总览

```
┌──────────────────────────┬────────────────────────────┐
│        C# (.NET)         │        C++ STL             │
├──────────────────────────┼────────────────────────────┤
│ List<T>                  │ std::vector<T>             │
│ LinkedList<T>            │ std::list<T>               │
│ Dictionary<TKey,TValue>  │ std::unordered_map<K,V>    │
│ SortedDictionary<K,V>    │ std::map<K,V>              │
│ SortedList<K,V>          │ (无直接对应，flat_map 类似)  │
│ HashSet<T>               │ std::unordered_set<T>      │
│ SortedSet<T>             │ std::set<T>                │
│ Queue<T>                 │ std::queue<T>              │
│ Stack<T>                 │ std::stack<T>              │
│ (无直接对应)              │ std::deque<T>              │
│ PriorityQueue (NET 6+)   │ std::priority_queue<T>     │
│ ConcurrentDictionary     │ (需要 tbb/concurrent_unordered_map) │
│ ConcurrentQueue          │ (需要第三方)                │
│ Channel<T> / BlockingCollection │ std::queue + mutex    │
└──────────────────────────┴────────────────────────────┘
```

**核心差异**：

| 维度 | C# | C++ STL |
|------|------|------|
| **内存模型** | 托管堆，GC 自动回收 | 堆/栈均可，手动管理或 RAII |
| **迭代器失效** | foreach 中修改抛异常（版本检测） | 迭代器可能变野指针（UB） |
| **异常安全** | 天然安全（GC + 强异常保证） | 需 RAII + 异常规范 |
| **自定义比较** | IComparer\<T\> 接口 | 模板算子（functor/lambda） |
| **泛型实现** | 运行时特化（引用类型共享） | 编译期展开（每个类型独立一份） |
| **值类型存储** | 直接内联，无需额外分配 | 直接内联 |

---

### 24.2 List\<T\> vs std::vector\<T\>（★★★★★）

#### C# List\<T\>

```csharp
// 内部结构
public class List<T> {
    private T[] _items;       // 底层数组
    private int _size;        // 实际元素数量
    private int _version;     // 版本号，foreach 中检测修改
}

// 扩容
// 初始容量 0 → 第一次 Add → 4 → 8 → 16 → 32...
// newCapacity = oldCapacity == 0 ? 4 : oldCapacity * 2
// 扩容时：新数组 + Array.Copy（最终调 memmove / 托管内部的 BulkMove）

// Clear 的行为
list.Clear();  // 内部：Array.Clear(_items, 0, _size); → 将引用置 null
               // 但 _items 数组容量不缩！内存不还！
               // 要释放内存：list.TrimExcess() 或 list.Capacity = 0
```

#### C++ std::vector\<T\>

```cpp
// 内部：T* _data + size_t _size + size_t _capacity
// 扩容因子通常 2x（MSVC）或 1.5x（GCC/Clang）
// 1.5x 的好处：旧内存块 + 其后的空闲空间可能刚好够新大小 → 原地扩展
// 2x 永远无法原地扩展（新大小 = 旧大小 * 2 > 旧大小 + 空闲）

// C++ 扩容用移动语义（C++11+）→ 比 C# 的 Array.Copy 可能更优
// 因为 C++ 对 string/vector 等可移动，而 C# 对引用类型复制的是引用（O(n) 引用复制）
```

| | C# List\<T\> | C++ std::vector\<T\> |
|------|------|------|
| **底层** | T[] 托管数组 | T* 裸指针+堆分配 |
| **扩容因子** | 2x | 2x (MSVC) / 1.5x (GCC) |
| **扩容代价** | Array.Copy（memmove） | 移动/拷贝 + 释放旧内存 |
| **值类型扩容** | memmove（快） | 移动（快，memmove 或逐元素） |
| **引用类型扩容** | 复制引用（快，O(n)） | N/A（存值） |
| **Clear 后内存** | 不释放（需手动 TrimExcess） | 不释放（需 shrink_to_fit） |
| **迭代器失效** | foreach 中修改 → 异常 | push_back 可能使迭代器失效 |
| **Add 均摊** | O(1) | O(1) |
| **Insert** | O(n)（后移） | O(n)（后移） |

**面试追问**：为什么 GCC 的 vector 扩容用 1.5x 而不是 2x？
> 1.5x 有机会"原地扩展"：释放掉的旧内存块 + 紧跟其后的空闲空间 → 可能刚好 ≥ old_capacity * 1.5。2x 永远不行（新大小 = 已用 + 已用，但已用空间已经占了实际位置）。

---

### 24.3 Dictionary\<TKey,TValue\> vs std::unordered_map（★★★★★）

#### C# Dictionary 内部

```csharp
// 核心结构
class Dictionary<TKey, TValue> {
    struct Entry {
        public int hashCode;    // 缓存的哈希值（避免重复计算）
        public int next;        // 碰撞链：下一个 Entry 的索引（-1 = 尾）
        public TKey key;
        public TValue value;
    }
    int[] _buckets;     // 桶数组（质数大小），存 Entry 索引+1，0=空桶
    Entry[] _entries;   // 所有键值对，连续存储
    int _freeList;      // 已删除 Entry 的空闲链表头
    int _freeCount;     // 空闲 Entry 数量
    int _count;         // 实际元素数量
}
```

```
Dictionary 内存布局：

_buckets (int[7])          _entries (Entry[])
┌─────┐                   ┌────────────────────────┐
│ [0] │──→ 1 ──────────→ │[0] hash=0              │  ← 空
│ [1] │──→ 0             │[1] hash=42,next=-1     │  ← "key1"→value1
│ [2] │──→ 2             │[2] hash=17,next=3      │  ← "key2"→value2
│ [3] │──→ 0             │[3] hash=17,next=-1     │  ← "key3"→value3 (碰撞)
│ [4] │──→ 0             │[4] hash=0              │  ← 空
│ [5] │──→ 0             │                        │
│ [6] │──→ 0             │                        │
└─────┘                   └────────────────────────┘

查找 key3：hash=17 → 17%7=3 → buckets[3]=2 → entries[2]
         → hash 不匹配 → next=3 → entries[3] → hash 匹配 ✅ → 返回值
```

**扩容时机**：`_count >= _buckets.Length * 0.72` → 新大小 = 当前大小×2 的下一个质数 → 重新哈希所有 Entry

**为什么 _freeList 用链表？** 删除的 Entry 不压缩，用 freeList 串联，下次 Add 优先填充空闲位（复用内存）。

#### C++ std::unordered_map 内部

```cpp
// 典型实现：分离链表法（每个桶一个链表头）
template<typename K, typename V>
class unordered_map {
    // buckets：vector<node*>，每个桶指向链表头
    // node：hash + key + value + next*
    // 查找：hash → bucket → 遍历链表
};
```

| | C# Dictionary | C++ unordered_map |
|------|------|------|
| **碰撞解决** | 拉链法（Entry.next 索引） | 拉链法（node* 指针） |
| **Entry 存储** | 连续数组（缓存友好） | 节点分散在堆上 |
| **空闲 Entry 复用** | freeList 链表 | 取决于 allocator |
| **扩容阈值** | ~0.72 (质数) | 通常 1.0 (max_load_factor) |
| **哈希缓存** | ✅ Entry 存 hashCode | 通常 ✅ |
| **迭代顺序** | 插入顺序 | 无保证 |
| **内存占用** | 紧凑（连续 Entry 数组） | 较松散（节点+指针） |

**面试追问**：为什么 C# Dictionary 的 entries 用连续数组？
> 缓存局部性：遍历 Dictionary 时，连续数组 → CPU cache line 命中率高。C++ unordered_map 每个节点单独分配 → 指针跳转 → cache miss。C# 的紧凑设计对现代 CPU 更友好。

---

### 24.4 HashSet\<T\> vs SortedSet\<T\>（★★★★）

```
┌──────────────────────┬───────────────────────┐
│   HashSet<T>         │   SortedSet<T>         │
├──────────────────────┼───────────────────────┤
│ 哈希表（同Dictionary）│ 红黑树                  │
│ O(1) 增删查           │ O(log n) 增删查        │
│ 不保证顺序             │ 始终排序                │
│ 需要 GetHashCode      │ 需要 IComparable       │
│ 对应 unordered_set    │ 对应 std::set          │
└──────────────────────┴───────────────────────┘
```

```csharp
// HashSet：O(1)，无顺序
var set = new HashSet<int> { 3, 1, 2 };
foreach (var x in set) Console.Write(x + " ");  // 可能 1 2 3 或 3 1 2

// SortedSet：O(log n)，始终有序
var sorted = new SortedSet<int> { 3, 1, 2 };
foreach (var x in sorted) Console.Write(x + " ");  // 一定是 1 2 3
```

---

### 24.5 Queue\<T\> / Stack\<T\> — 环形数组实现（★★★★）

#### Queue\<T\>

```csharp
// 内部：环形数组
class Queue<T> {
    private T[] _array;
    private int _head;   // 队首索引（下次 Dequeue 位置）
    private int _tail;   // 队尾索引（下次 Enqueue 位置）
    private int _size;
}

// 环形：Enqueue 到 array[tail]，tail = (tail+1) % capacity
//        Dequeue 从 array[head]，head = (head+1) % capacity
// head==tail 为空或满 → 用 _size 区分
// 扩容：分配新数组，按顺序复制（head→tail 环绕）
```

```
Queue 环形数组示意（capacity=4）：
初始：[ ][ ][ ][ ]  head=0 tail=0 size=0
E×3:  [A][B][C][ ]  head=0 tail=3 size=3
D×2:  [ ][ ][C][ ]  head=2 tail=3 size=1
E×2:  [ ][F][C][D]  head=2 tail=1 size=3  ← tail 绕回到 1
```

#### Stack\<T\>

```csharp
// 内部：普通数组，只有 _size 索引
class Stack<T> {
    private T[] _array;
    private int _size;  // 也等于下一次 Push 的位置
}
// Push → array[_size++] = item
// Pop → return array[--_size]
// 扩容：2x
```

#### C++ 对比

| | C# Queue\<T\> | C++ std::queue\<T\> |
|------|------|------|
| **底层容器** | 环形数组（固定） | 默认 std::deque（可换 vector） |
| **底层实现** | 自实现环形 | std::deque：分段数组（chunk） |
| **内存特点** | 连续+环绕 | 分段连续，不易碎片 |

C++ `std::deque` 值得了解：
```cpp
// std::deque：分段数组（chunk array），每个 chunk 固定大小（如 512 字节）
// [chunk0][chunk1][chunk2]...
// 头尾插入 O(1)，中间插入 O(n)
// 没有 C# 的直接对应 → C# 用 LinkedList 约等于，但缓存局部性差很多
```

---

### 24.6 LinkedList\<T\> vs std::list\<T\>（★★★）

```
C# LinkedList<T> = 双向链表，每个节点 LinkedListNode<T>：
  ┌─── prev
  │ ┌─ next
  │ │ ┌─ value
  ▼ ▼ ▼
┌─────────┐    ┌─────────┐    ┌─────────┐
│ Node    │◄──►│ Node    │◄──►│ Node    │
│ "A"     │    │ "B"     │    │ "C"     │
└─────────┘    └─────────┘    └─────────┘
```

| | C# LinkedList\<T\> | C++ std::list\<T\> |
|------|------|------|
| **节点** | LinkedListNode\<T\> 类（堆分配） | 内部 node（堆分配） |
| **插入/删除** | O(1)（给定节点） | O(1)（给定迭代器） |
| **随机访问** | ❌ | ❌ |
| **缓存友好** | ❌（节点分散） | ❌（节点分散） |

**面试追问**：什么时候用 LinkedList 而不是 List？
> 频繁在中间插入/删除 → LinkedList O(1) vs List O(n)。但实际中 List 的 O(n) 因为缓存局部性好，n<1000 时往往比 LinkedList 快。LinkedList 几乎只在"需要在遍历中修改自身"或"前后都频繁删除"时才用。

---

### 24.7 SortedDictionary vs SortedList（★★★）

| | SortedDictionary\<K,V\> | SortedList\<K,V\> |
|------|------|------|
| **底层** | 红黑树 | 两个排序数组（Key[] + Value[]） |
| **插入** | O(log n) | O(n)（后移） |
| **查找** | O(log n) | O(log n)（二分查找） |
| **索引访问** | ❌ | ✅ `Values[i]` O(1) |
| **内存** | 每节点有指针开销 | 紧凑数组，无指针开销 |
| **适用** | 频繁增删 | 多读少写，需要索引 |
| **C++ 对应** | std::map | （无直接对应，flat_map 类似） |

```csharp
// SortedList 适合：配置表（一次加载，多次二分查找）
var config = new SortedList<int, string>();
config.Add(1, "Level 1");
config.Add(2, "Level 2");
// 内部 Key[]: [1, 2]  Value[]: ["Level 1", "Level 2"]
var level5 = config[5];  // 二分查找
```

---

### 24.8 Concurrent 集合 —— 线程安全（★★★★★）

#### ConcurrentDictionary\<K,V\>

```csharp
// 内部：lock per bucket（不是全局锁！）
// 并发场景下多个线程写不同桶 → 无需等待
// 读操作大部分 lock-free（volatile 读）

var dict = new ConcurrentDictionary<string, int>();

// 原子操作
dict.TryAdd("key", 1);                                      // 不存在则添加
dict.TryUpdate("key", 2, 1);                                // 旧值=1 才更新为 2
dict.AddOrUpdate("key", 1, (k, old) => old + 1);           // 存在+1，不存在设1
int val = dict.GetOrAdd("key", 42);                         // 存在返回，不存在添加

// 批量操作（非原子，但安全）
foreach (var kv in dict) { }  // 遍历快照，不抛异常
```

| 操作 | 线程安全性 |
|------|------|
| TryAdd / TryGetValue / TryRemove / TryUpdate | 原子操作 |
| GetOrAdd / AddOrUpdate | 原子操作（valueFactory 可能多次调用但只有一个结果生效） |
| foreach | 安全（遍历快照） |
| Count | 近似值（快照） |

#### ConcurrentQueue / ConcurrentStack / ConcurrentBag

```
ConcurrentQueue<T>：
  • lock-free 的 Michael-Scott 队列（单向链表+原子操作）
  • Enqueue / TryDequeue 无锁
  • 对应 C++: boost::lockfree::queue / moodycamel::ConcurrentQueue

ConcurrentStack<T>：
  • lock-free 的 Treiber Stack（单向链表+CAS）
  • Push / TryPop 无锁
  • 对应 C++: boost::lockfree::stack

ConcurrentBag<T>：
  • 线程局部存储 + 窃取（work-stealing）
  • 同一线程 Add/Take 快，多线程间 Take 走窃取
  • 无序

BlockingCollection<T>：
  • 生产者-消费者模式
  • 底层可包装任何 IProducerConsumerCollection（默认 ConcurrentQueue）
  • 支持阻塞 Take、有界容量（BoundedCapacity）
```

```csharp
// 生产者-消费者
var queue = new BlockingCollection<int>(boundedCapacity: 10);

// 生产者
Task.Run(() => {
    for (int i = 0; i < 100; i++)
        queue.Add(i);  // 满时阻塞
    queue.CompleteAdding();
});

// 消费者
foreach (var item in queue.GetConsumingEnumerable()) {
    Process(item);  // 完成时自动退出
}
```

---

### 24.9 选型决策树

```
需要存储什么？
│
├── 需要 Key-Value 映射
│   ├── 需要排序 → SortedDictionary（红黑树）/ SortedList（排序数组）
│   ├── 需要线程安全 → ConcurrentDictionary
│   └── 不需要排序 → Dictionary（O(1)，首选）
│
├── 只需要顺序列表
│   ├── 需要索引访问 → List<T>（99% 场景用这个）
│   ├── 频繁中间插入/删除 → LinkedList<T>（实际很少用）
│   ├── FIFO → Queue<T>（环形数组）
│   └── LIFO → Stack<T>
│
├── 只需要值（去重判断）
│   ├── 不需要排序 → HashSet<T>
│   └── 需要排序 → SortedSet<T>
│
└── 线程安全
    ├── 键值对 → ConcurrentDictionary
    ├── 队列 → ConcurrentQueue（无锁）/ BlockingCollection（阻塞）
    ├── 栈 → ConcurrentStack
    └── 无序包 → ConcurrentBag（work-stealing）
```

---

### 24.10 性能对比速查表

```
操作          List   Dict   HashSet  Queue  Stack  LL   SortedDict  SortedList
─────────────────────────────────────────────────────────────────────────────
索引访问      O(1)   O(1)*  -        -      -      -    -           O(1)
Add/Enqueue   O(1)†  O(1)†  O(1)†   O(1)†  O(1)†  O(1) O(log n)    O(n)
Insert(i)     O(n)   -      -        -      -      O(1)^ O(log n)    O(n)
Remove        O(n)   O(1)†  O(1)†   O(1)   O(1)   O(1)^ O(log n)    O(n)
Contains      O(n)   O(1)†  O(1)†   O(n)   O(n)   O(n)  O(log n)    O(log n)
内存开销      低     中等    中等     低     低     高    高          低

* 通过 Key
† 均摊（扩容时 O(n)）
^ 给定节点 O(1)，给定值 O(n)
```

| 扩容因子 | C# | C++ (GCC) |
|------|------|------|
| List / vector | 2x | 2x (MSVC) / 1.5x (GCC) |
| Dictionary / unordered_map | ~0.72 阈值 | 1.0 (max_load_factor) |
| Queue | 2x | N/A (deque 分段) |
| Stack | 2x | N/A (deque 分段) |
| HashSet / unordered_set | ~0.72 阈值 | 1.0 (max_load_factor) |

---

### 24.11 API 方法对照表（★★★★★）

#### List\<T\> vs std::vector\<T\>

| 操作 | C# List\<T\> | C++ std::vector\<T\> |
|------|------|------|
| 尾部添加 | `Add(item)` | `push_back(item)` / `emplace_back(args...)` |
| 尾部移除 | `RemoveAt(Count-1)` | `pop_back()` |
| 指定位置插入 | `Insert(index, item)` | `insert(it, item)` / `emplace(it, args...)` |
| 指定位置删除 | `RemoveAt(index)` | `erase(it)` |
| 按值删除（首个） | `Remove(item)` | `erase(find(...))` |
| 按值删除（全部） | `RemoveAll(predicate)` | `erase(remove_if(...))` |
| 访问元素 | `list[i]` | `vec[i]` / `vec.at(i)` (带边界检查) |
| 首元素 | `list[0]` | `vec.front()` |
| 尾元素 | `list[^1]` / `list[Count-1]` | `vec.back()` |
| 元素数量 | `Count` (属性) | `size()` |
| 容量 | `Capacity` (属性) | `capacity()` |
| 预分配 | `new List<T>(capacity)` / `list.Capacity = n` | `vec.reserve(n)` |
| 缩容 | `TrimExcess()` / `Capacity = Count` | `shrink_to_fit()` |
| 清空 | `Clear()` | `clear()` |
| 判空 | `Count == 0` | `empty()` |
| 是否包含 | `Contains(item)` | `find(...) != end()` |
| 查找索引 | `IndexOf(item)` / `FindIndex(pred)` | `find(...) - begin()` / `find_if(...)` |
| 查找元素 | `Find(pred)` | `*find_if(...)` |
| 排序 | `Sort()` / `Sort(comparer)` | `sort(begin(), end())` / `sort(..., comp)` |
| 反转 | `Reverse()` | `reverse(begin(), end())` |
| 转数组 | `ToArray()` | `vec.data()` (返回指针) |
| 批量添加 | `AddRange(collection)` | `insert(end(), first, last)` |
| 遍历 | `foreach / ForEach(action)` | range-for / `for_each(...)` |
| 切片 | `Slice(start, length)` (.NET 8+) | `span(vec.data()+s, len)` |

```csharp
// C#
var list = new List<int> { 1, 2, 3 };
list.Add(4);
list.Remove(2);           // 移除值为 2 的第一个元素
list.RemoveAll(x => x < 0);
bool has = list.Contains(3);
int idx = list.IndexOf(3);
list.Sort((a, b) => b.CompareTo(a));  // 降序
```

```cpp
// C++
std::vector<int> vec = {1, 2, 3};
vec.push_back(4);
vec.erase(std::find(vec.begin(), vec.end(), 2));  // 移除值为 2
vec.erase(std::remove_if(vec.begin(), vec.end(), [](int x) { return x < 0; }), vec.end());
bool has = std::find(vec.begin(), vec.end(), 3) != vec.end();
int idx = std::find(vec.begin(), vec.end(), 3) - vec.begin();
std::sort(vec.begin(), vec.end(), std::greater<int>());  // 降序
```

---

#### Dictionary\<K,V\> vs std::unordered_map\<K,V\>

| 操作 | C# Dictionary\<K,V\> | C++ std::unordered_map\<K,V\> |
|------|------|------|
| 添加/覆盖 | `dict[key] = value` | `map[key] = value` |
| 安全添加 | `TryAdd(key, value)` → bool | `map.emplace(key, value)` / `insert({k,v})` |
| 获取值 | `dict[key]` (不存在抛异常) | `map[key]` (不存在 → 插入默认值!) |
| 安全获取 | `TryGetValue(key, out val)` → bool | `find(key) → it` / `at(key)` (不存在抛异常) |
| 获取或默认 | `GetValueOrDefault(key)` / `GetValueOrDefault(key, default)` | `find → it != end ? it->second : default` |
| 获取或添加 | `GetOrAdd(key, factory)` (ConcurrentDict) | `try_emplace` (C++17) |
| 更新 | `dict[key] = newValue` | `map[key] = newValue` / `insert_or_assign` (C++17) |
| 条件更新 | `TryUpdate(key, newVal, oldVal)` (ConcurrentDict) | `find → if match → assign` |
| 删除 | `Remove(key)` → bool / `Remove(key, out val)` | `erase(key)` → size_t |
| 条件删除 | `Remove(key) where ...` (手动) | `erase(it)` |
| 清空 | `Clear()` | `clear()` |
| 数量 | `Count` (属性) | `size()` |
| 判空 | `Count == 0` | `empty()` |
| 是否含 Key | `ContainsKey(key)` | `find(key) != end()` / `contains(key)` (C++20) |
| 是否含 Value | `ContainsValue(value)` | 需遍历 (无对应) |
| 所有 Key | `Keys` (属性) | 需遍历提取 |
| 所有 Value | `Values` (属性) | 需遍历提取 |
| 遍历 | `foreach(var kv in dict)` | `for(auto& [k,v] : map)` (C++17) |
| 批量容量预分配 | `new Dictionary(n)` / `EnsureCapacity(n)` | `reserve(n)` |
| 扩容阈值因子 | ~0.72 (只读) | `max_load_factor(f)` 可设置 |

```csharp
// C# —— 安全获取模式
if (dict.TryGetValue(key, out var val)) {
    Console.WriteLine(val);
}
var v = dict.GetValueOrDefault(key, -1);
```

```cpp
// C++ —— 查找模式
auto it = map.find(key);
if (it != map.end()) {
    std::cout << it->second;
}
// 警告：map[key] 如果 key 不存在会插入默认值！不是只读操作！
```

---

#### HashSet\<T\> vs std::unordered_set\<T\>

| 操作 | C# HashSet\<T\> | C++ std::unordered_set\<T\> |
|------|------|------|
| 添加 | `Add(item)` → bool | `insert(item)` → pair<it, bool> |
| 删除 | `Remove(item)` → bool | `erase(item)` → size_t |
| 是否包含 | `Contains(item)` → bool | `find(item) != end()` / `contains(item)` (C++20) |
| 清空 | `Clear()` | `clear()` |
| 数量 | `Count` | `size()` |
| 集合运算 | `UnionWith` / `IntersectWith` / `ExceptWith` | `set_union` / `set_intersection` / `set_difference` (算法，非成员) |
| 子集判断 | `IsSubsetOf` / `IsSupersetOf` / `Overlaps` | `includes(...)` (算法) |
| 批量添加 | `UnionWith(collection)` | `insert(begin, end)` |

---

#### Queue / Stack / LinkedList

| 操作 | C# Queue\<T\> | C++ std::queue\<T\> |
|------|------|------|
| 入队 | `Enqueue(item)` | `push(item)` / `emplace(args...)` |
| 出队 | `TryDequeue(out val)` → bool / `Dequeue()` | `pop()` (不返回值!) / `front()` + `pop()` |
| 窥视队首 | `TryPeek(out val)` / `Peek()` | `front()` |
| 清空 | `Clear()` | `queue = {}` / `while(!empty()) pop()` |

| 操作 | C# Stack\<T\> | C++ std::stack\<T\> |
|------|------|------|
| 入栈 | `Push(item)` | `push(item)` / `emplace(args...)` |
| 出栈 | `TryPop(out val)` / `Pop()` | `pop()` (不返回值!) / `top()` + `pop()` |
| 窥视栈顶 | `TryPeek(out val)` / `Peek()` | `top()` |

| 操作 | C# LinkedList\<T\> | C++ std::list\<T\> |
|------|------|------|
| 头部添加 | `AddFirst(item)` | `push_front(item)` / `emplace_front(args...)` |
| 尾部添加 | `AddLast(item)` | `push_back(item)` / `emplace_back(args...)` |
| 头部移除 | `RemoveFirst()` | `pop_front()` |
| 尾部移除 | `RemoveLast()` | `pop_back()` |
| 在某节点前插入 | `AddBefore(node, item)` | `insert(it, item)` |
| 在某节点后插入 | `AddAfter(node, item)` | `insert(next(it), item)` |
| 删除某节点 | `Remove(node)` | `erase(it)` |
| 查找 | `Find(value)` → node | `find(begin, end, value)` → it |

---

#### 关键 API 差异速记

```
C# 的 "Try" 模式 vs C++ 的 find 模式：
  C#:  dict.TryGetValue(key, out val)  → 一步完成，无异常
  C++:  auto it = map.find(key); if (it != map.end()) val = it->second;

C# 泛型集合用属性 vs C++ 用方法：
  C#:  list.Count    dict.Keys     set.Count
  C++:  vec.size()   需遍历提取      set.size()

C# Stack/Pop 返回值 vs C++ pop 不返回值：
  C#:  var x = stack.Pop();    // 弹出 + 返回
  C++:  auto x = stack.top(); stack.pop();  // 两步！

C++ map[key] 的陷阱 vs C# dict[key]：
  C++:  map[key] 如果 key 不存在 → 插入默认值！不是只读！
  C#:   dict[key] 如果 key 不存在 → KeyNotFoundException

C# foreach 安全 vs C++ 迭代器野指针：
  C#:  foreach 中修改集合 → InvalidOperationException（安全！）
  C++:  push_back 后迭代器可能失效 → 未定义行为（危险！）
```

---

### 24.12 集合底层原理强化：从内存布局到一次操作

前面的内容已经给出了各容器的结构和复杂度，这里补一条更适合面试展开的主线：C# 集合的性能，通常由四件事共同决定——底层存储是否连续、一次操作会触发多少次对象复制、是否发生托管堆分配，以及比较器/哈希函数是否高效。

#### 24.12.1 数组是所有泛型集合的基础

托管数组是 CLR 认识的一等对象，数组对象通常包含对象头、长度字段和连续的元素区域。数组本身在托管堆上分配，但元素不一定都是对象引用：

```csharp
int[] values = new int[3];       // 堆上连续存放 3 个 int
Player[] players = new Player[3]; // 堆上连续存放 3 个 Player 引用
Player[] objects = new Player[3];  // 元素是引用，Player 实例分散在堆上
```

对于 `int[]`、`Vector3[]` 等值类型数组，元素数据直接内嵌在数组中；对于 `class[]` 或 `string[]`，数组里存的是引用，引用指向的对象不保证连续。因此“数组连续”只保证数组元素区域连续，不代表引用指向的所有对象也连续。

这也是 `List<T>` 对值类型和引用类型行为不同的原因：`List<int>` 扩容时复制的是整数数据，`List<Player>` 扩容时复制的是对象引用，原来的 `Player` 实例不会被复制。

数组长度创建后不可改变。所有可扩容的线性集合，本质上都通过“重新分配一个更大的数组，再复制有效元素”模拟扩容；所以容量规划会直接影响 GC 分配和复制成本。

#### 24.12.2 List<T> 的一次 Add 和一次 Remove

`List<T>.Add` 的典型路径可以抽象为：

```text
Add(item)
  ├─ _size < _items.Length
  │    └─ _items[_size] = item; _size++
  └─ 容量不足
       ├─ 计算新容量
       ├─ new T[newCapacity]
       ├─ Array.Copy 复制已有元素
       └─ 替换 _items 后写入新元素
```

因此 `Add` 的均摊复杂度是 `O(1)`，但扩容那一次是 `O(n)`，并且会产生新的数组对象。可以预估数量时使用：

```csharp
var items = new List<Item>(expectedCount);
```

`RemoveAt(index)` 通常不会移动整个数组，而是把 `index` 后面的元素整体向前移动一格，再将最后一个有效位置清为 `default(T)`。清零对引用类型很重要：它会解除数组槽位对旧对象的引用，使对象有机会被 GC 回收；对值类型则只是写入默认值。

`Clear()` 只清空有效元素并将 `_size` 设为 0，通常保留底层数组；它适合复用同一个列表，不等于释放容量。`TrimExcess()` 或设置 `Capacity` 才可能重新分配更小的数组，但收缩本身也会产生复制成本。

#### 24.12.3 Dictionary 的完整查找链路

以常见 .NET 实现模型为例，`Dictionary<TKey, TValue>` 通过桶数组和 Entry 数组建立一条“索引链表”：

```text
key
 ↓
EqualityComparer<TKey>.GetHashCode(key)
 ↓
根据桶数量计算 bucketIndex
 ↓
buckets[bucketIndex] 得到 Entry 索引
 ↓
比较 hashCode，再调用 Equals
 ↓
通过 Entry.next 继续检查冲突节点
```

`Entry` 通常包含：

```csharp
struct Entry
{
    public uint hashCode;    // 某些旧版实现使用 int，具体类型属于版本实现细节
    public int next;
    public TKey key;
    public TValue value;
}
```

这里的 `next` 是数组下标，不是对象指针。数组下标链有两个重要效果：Entry 可以连续存储，减少每个节点单独分配造成的 GC 压力；冲突链遍历时也更容易命中 CPU Cache。删除元素时，Entry 通常不会整体搬移，而是加入空闲链表，后续插入优先复用空闲位置。

扩容时不能只把 `entries` 数组复制到新数组，因为桶数量变化会改变哈希值到桶下标的映射，`buckets` 和 `next` 都需要重建。扩容阶段的成本是 `O(n)`，正常查找、添加和删除的平均复杂度才是 `O(1)`。

不同 .NET 版本可能采用不同的容量序列、桶数量策略和哈希随机化细节，因此不要在业务代码中依赖“容量一定是质数”或“遍历一定是插入顺序”等实现结论。应依赖公开 API 的语义；如果讨论源码，则必须注明目标 .NET 版本。

#### 24.12.4 HashSet<T> 为什么可以看成没有 Value 的 Dictionary

`HashSet<T>` 使用与字典相同的问题分解：哈希值、桶定位、冲突链、空闲 Entry 复用和扩容。区别是 Entry 只需要保存哈希值、next 和 key，不需要 value 字段。

```text
HashSet<T>.Add(value)
  ├─ 计算 hash
  ├─ 在冲突链中查找 Equals(value)
  ├─ 已存在：返回 false
  └─ 不存在：写入 Entry，返回 true
```

所以 `HashSet<T>` 的“唯一性”不是靠排序，而是靠 `GetHashCode` 先定位、`Equals` 再确认。只要元素放入集合后参与哈希或相等判断的字段发生变化，就可能出现“集合里有这个对象，但 Contains 返回 false”的逻辑问题。可变对象作为字典 Key 或 HashSet 元素时应特别谨慎。

#### 24.12.5 GetHashCode 和 Equals 必须满足的契约

自定义键类型必须遵守：

```text
a.Equals(b) == true  =>  a.GetHashCode() == b.GetHashCode()
```

反过来不成立：哈希值相同只表示发生了哈希碰撞，仍然必须继续调用 `Equals` 判断是否真的相等。

错误示例：

```csharp
sealed class PlayerKey
{
    public int Id;
    public string Region = "";

    public override bool Equals(object? obj)
        => obj is PlayerKey other && Id == other.Id && Region == other.Region;

    // 错误：Equals 比较了 Region，但 hash 只使用 Id
    public override int GetHashCode() => Id;
}
```

这个例子不一定立即失效，因为相等对象仍然会得到相同哈希，但会产生更多碰撞。更危险的是只重写 `Equals` 不重写 `GetHashCode`，或者对象插入集合后改变参与比较的字段，这会破坏查找行为。推荐使用不可变 Key，并让 `Equals` 与 `GetHashCode` 使用完全一致的字段。

#### 24.12.6 Queue<T>、Stack<T> 和 PriorityQueue<TElement,TPriority>

`Queue<T>` 使用环形数组避免出队后搬移全部元素：`head` 指向下一个出队位置，`tail` 指向下一个入队位置，`size` 区分空和满。下标回绕可以通过取模或等价的边界判断实现。扩容时才会把逻辑顺序重新整理到新数组的起点。

`Stack<T>` 更简单：底层数组的 `[0, _size)` 是有效区间，Push 写入 `_size`，Pop 先递减 `_size` 再读取末尾元素，并清理槽位。两者的正常入队/入栈和出队/出栈都是 `O(1)` 均摊复杂度。

`.NET 6+` 的 `PriorityQueue<TElement,TPriority>` 是四叉堆（quaternary min-heap）模型，而不是普通的先进先出队列。底层数组按堆序排列：父节点的优先级不大于子节点，最小优先级位于根部。对于下标 `i`，常见四叉堆关系可以抽象为：

```text
parent(i) = (i - 1) / 4
children(i) = 4 * i + 1 ... 4 * i + 4
```

`Enqueue` 将元素放在尾部并向上调整，`Dequeue` 移除根元素、把末尾元素放到根部并向下调整；两者都是 `O(log n)`，查看最小优先级元素是 `O(1)`。优先队列只保证堆顶最优，不保证整体遍历有序。

#### 24.12.7 LinkedList<T> 的真实成本

`LinkedList<T>` 的每个 `LinkedListNode<T>` 都是独立的托管对象，通常需要保存：

```text
对象头 + Value + Next 引用 + Previous 引用 + 所属 LinkedList 引用
```

因此它虽然在“已知节点位置”时插入删除为 `O(1)`，但会付出：

- 每个节点一次对象分配；
- 额外的对象头和引用字段；
- 节点分散导致的 Cache Miss；
- 按值查找仍然是 `O(n)`。

这解释了为什么很多“中间删除”的场景中 `List<T>` 反而更快：`List<T>` 需要移动一段连续内存，但移动操作高度连续、容易被 CPU 和运行时优化；`LinkedList<T>` 省掉了移动，却增加了指针跳转和分配成本。

#### 24.12.8 SortedDictionary、SortedList 和排序的代价

`SortedDictionary<TKey,TValue>` 适合动态增删，底层通常是平衡树，查找、插入和删除为 `O(log n)`，但每个节点有引用字段和独立分配成本。

`SortedList<TKey,TValue>` 通常维护两个按 Key 对齐的数组：`TKey[] keys` 和 `TValue[] values`。查找使用二分查找为 `O(log n)`，按索引访问为 `O(1)`，但插入和删除需要移动后续数组元素，为 `O(n)`。因此它适合“初始化后主要读取”的配置表，不适合频繁修改的动态索引。

#### 24.12.9 foreach 为什么修改集合会抛异常

许多集合内部维护 `_version`。创建枚举器时，枚举器保存当前版本；每次 `MoveNext()` 时检查集合版本是否变化：

```csharp
if (_version != enumeratorVersion)
    throw new InvalidOperationException("Collection was modified");
```

这不是线程同步机制，也不保证并发访问安全，只是一种 fail-fast 检测，帮助尽早暴露“遍历过程中改变集合”的错误。`Dictionary`、`List` 等集合的具体枚举顺序也不应当作为业务契约。

如果需要在遍历时删除，可以使用集合提供的安全 API，或先记录待删除元素，再在遍历结束后删除：

```csharp
for (int i = list.Count - 1; i >= 0; i--)
{
    if (ShouldRemove(list[i]))
        list.RemoveAt(i);
}
```

倒序删除可以避免删除一个元素后，尚未检查的元素整体左移导致索引跳过。

#### 24.12.10 ArrayPool<T>：复用数组，降低 GC 压力

`ArrayPool<T>` 不是一种新的逻辑数据结构，而是数组对象的复用池。调用 `Rent` 得到的数组长度可能大于请求长度，使用时必须单独记录有效长度；使用结束后调用 `Return`，让数组回到池中：

```csharp
byte[] buffer = ArrayPool<byte>.Shared.Rent(4096);
try
{
    int length = ReadData(buffer);
    Process(buffer.AsSpan(0, length));
}
finally
{
    ArrayPool<byte>.Shared.Return(buffer, clearArray: true);
}
```

池化数组的所有权必须明确：归还后不能继续使用，不能把它的引用保存到异步任务或长期对象中。对于包含敏感数据的数组，应使用 `clearArray: true`，否则旧数据可能被下一次租借者看到；代价是归还时增加清零开销。

#### 24.12.11 线程安全集合的底层判断

普通 `List<T>`、`Dictionary<TKey,TValue>` 的成员方法通常不提供并发写入保护。`ConcurrentDictionary`、`ConcurrentQueue` 等集合通过分段锁、CAS、无锁队列算法或线程本地结构降低竞争，但“线程安全”只保证单个公开操作的并发语义，不会自动把多步业务逻辑合并为一个原子事务。

例如：

```csharp
if (!dict.ContainsKey(key))
    dict[key] = CreateValue();
```

即使 `dict` 是 `ConcurrentDictionary`，这两步之间仍可能被其他线程插入相同 Key。应使用 `GetOrAdd`、`TryAdd`、`AddOrUpdate` 等原子 API；但要注意 `valueFactory` 可能被多个线程竞争调用，工厂方法本身不应依赖“只执行一次”。

#### 24.12.12 一分钟回答：C# 数据结构底层

面试回答可以按以下顺序展开：

1. 先说底层存储：连续数组、桶数组 + Entry、环形数组、链表节点、平衡树还是堆数组；
2. 再说一次核心操作如何定位和修改数据；
3. 说明平均、均摊和最坏复杂度，不把均摊 `O(1)` 说成任何情况下都是 `O(1)`；
4. 补充扩容、复制、GC 分配、装箱和缓存局部性；
5. 最后说版本号、线程安全、Key 可变性和 .NET 版本差异。

例如回答 `Dictionary<TKey,TValue>`：它通常由 `buckets` 和连续的 `entries` 组成；查找先计算 Key 的哈希值定位桶，再沿 `next` 索引链比较哈希值和 `Equals`；冲突使最坏复杂度退化为 `O(n)`，扩容时必须重建桶和冲突链；删除的 Entry 通常进入空闲链表等待复用；实际性能还取决于哈希函数、比较器、容量规划和 Key 是否可变。

---

## 第二十五章 String

### 25.1 为什么 string 不可变？

string 是 immutable 的设计选择，目的是线程安全、安全共享、哈希缓存。每次"修改"字符串实际创建了新对象，原字符串不变。

为什么设计为不可变？① 线程安全（不可变对象天然线程安全）；② 字符串池（相同内容复用同一份内存）；③ 安全性（不会出现被调方修改字符串）；④ 哈希缓存（GetHashCode 只需计算一次并缓存）。

### 25.2 StringBuilder

StringBuilder 维护可变的 char[] 缓冲区，拼接字符串时直接在缓冲区内操作，不产生中间对象。适合大量拼接（循环 100+ 次）。

```csharp
// ❌ 循环拼接——每次创建新对象
string s = "";
for (int i = 0; i < 10000; i++) s += i.ToString();  // 10000 个临时 string！

// ✅ StringBuilder——零临时对象
var sb = new StringBuilder();
for (int i = 0; i < 10000; i++) sb.Append(i);
string result = sb.ToString();
```

内部原理：`StringBuilder` 有 `char[] m_ChunkChars`（当前 chunk）+ `StringBuilder m_ChunkPrevious`（前一个 chunk 链表）。当前 chunk 容量不足 → 分配新 chunk → 旧 chunk 链接。ToString() 时才把所有 chunk 合成一个 string。

### 25.3 字符串池（Intern Pool）

CLR 维护的哈希表，将相同内容的字符串字面量复用为同一个堆对象。编译期字面量自动入池，运行时可用 `string.Intern()` 手动入池。

**慎用 Intern**：✅ 减少重复字符串内存；❌ Intern 的字符串永远不会被 GC 回收；❌ `string.Intern()` 有锁，频繁调用有性能开销。

### 25.4 Equals vs ReferenceEquals

string 重写了 Equals → **值比较**（字符逐个比较）；ReferenceEquals → **引用比较**（指针地址）。直接 `==` 对 string 是值比较（被重载为 Equals）。

### 25.5 性能陷阱

```csharp
// ❌ 陷阱1：循环拼接
for (int i = 0; i < 100; i++) result += data[i];

// ❌ 陷阱2：ToLower/ToUpper 比较（分配新 string）
if (name.ToLower() == "admin") { }

// ✅ 使用 StringComparison 零分配
if (name.Equals("admin", StringComparison.OrdinalIgnoreCase)) { }

// ❌ Unity Update 中字符串操作 → 每帧 GC
void Update() { label.text = "Score: " + score; }

// ✅ 只在变化时更新
if (_lastScore != score) { _lastScore = score; label.text = $"Score: {score}"; }
```

---

## 第二十六章 Unity 高频八股

### 26.1 GetComponent 为什么慢？

GetComponent 在 GameObject 的组件链表上做线性搜索 + 类型比较（字符串/类型），O(n)。频繁调用时累计开销显著。

```csharp
// ❌ 每帧搜索
void Update() { var rb = GetComponent<Rigidbody>(); }

// ✅ 启动时缓存
private Rigidbody _rb;
void Start() { _rb = GetComponent<Rigidbody>(); }
void Update() { _rb.AddForce(...); }
```

### 26.2 Update 为什么不要太多？

每个 MonoBehaviour.Update() 都是引擎的 C++ → C# 跨域调用（Native → Managed）。MonoBehaviour 太多 → 海量跨域调用开销。用少量管理器统一 Update 替代大量独立 Update。

```csharp
// ✅ 一个 UpdateManager 统一 Tick
class Enemy : MonoBehaviour { public void Tick() { } }
class UpdateManager : MonoBehaviour {
    List<Enemy> enemies;
    void Update() { foreach (var e in enemies) e.Tick(); }  // C++→C# 只调 1 次
}
```

### 26.3 协程原理

Unity 协程 = IEnumerator + yield return。引擎每帧检查所有协程的 yield 条件是否满足 → 满足则 MoveNext → 继续执行。不能取代多线程（始终在主线程执行），适合"分帧"处理。

| yield | 含义 |
|------|------|
| `yield return null` | 等一帧 |
| `yield return new WaitForSeconds(n)` | 等 n 秒 |
| `yield return new WaitForEndOfFrame()` | 等帧末尾 |
| `yield return new WaitUntil(() => bool)` | 等条件为真 |
| `yield return StartCoroutine(...)` | 等子协程完成 |

### 26.4 Invoke 原理

Invoke 在内部维护一个计时器列表，每帧检查是否到达延时，到达则通过反射调用对应方法。开销：字符串查找方法名 + 反射调用。不如直接协程或手动计时。

### 26.5 Addressables vs Resources vs AssetBundle

| | Resources | AssetBundle | Addressables |
|------|------|------|------|
| **加载方式** | 同步 Resources.Load | 异步 | 异步 |
| **内存管理** | 全在内存 | 手动管理 | 引用计数自动管理 |
| **热更新** | ❌ | ✅ | ✅ |
| **推荐** | ❌ 已淘汰 | 大项目可用 | ✅ 现代最佳方案 |

### 26.6 GC Alloc 在 Unity 中的表现

Unity 老版本 Mono 是 Boehm GC（非分代），Stop-the-World 时卡顿明显（几十~几百 ms）。新版 IL2CPP 使用增量 GC。避免在 Update 中分配任何堆内存。

零 GC 编程习惯：对象池复用子弹/特效 → StringBuilder 复用（Clear → Append）→ 缓存 WaitForSeconds 实例 → List\<T\> 而非 ArrayList → CompareTag 代替 tag == "xxx"。

### 26.7 DrawCall 优化

DrawCall = CPU 向 GPU 发起的一次绘制指令。减少方法：

1. **Static Batching**：静态物体合并网格
2. **Dynamic Batching**：小网格（<300顶点）运行时合并
3. **GPU Instancing**：同一材质多实例
4. **SRP Batcher**：批量设置材质属性（不是合批网格，而是减少每个 DrawCall 的 CPU 设置开销）
5. **合并网格 / Mesh Baker**

### 26.8 SRP Batcher

不是合批网格，而是**批量设置材质属性**。零 DrawCall 减少，但大幅减少每个 DrawCall 的 CPU 设置开销。要求：兼容 SRP 的 Shader、材质属性需在 CBUFFER 中。

### 26.9 Dynamic Batching

运行时将使用同一材质的小网格（< 300 顶点，< 900 顶点属性）合并为一个 DrawCall。全自动，但顶点数限制严。使用不同材质或多 Pass Shader 不参与。

---

# 第四部分：面试专题

## 面试真题一：Unity 高频面试题 100 道（精选核心）

### C# 基础（20题核心要点）

1. **值类型和引用类型**：值类型存值本身，struct 在栈（局部）；引用类型存地址，对象在堆。
2. **装箱和拆箱**：装箱=值类型→object，堆分配+复制值；拆箱=object→值类型。频繁装箱导致 GC 压力。
3. **struct 和 class**：struct 值类型/栈/不可继承；class 引用类型/堆/可继承。
4. **ref 和 out**：ref 调用前必须初始化；out 方法内必须赋值。
5. **string 为什么不可变**：线程安全+安全共享+字符串池+哈希缓存。
6. **StringBuilder 原理**：维护可变的 char[] 缓冲区，零临时对象。
7. **delegate 和 event**：event 是对 delegate 的包装，外部只能+=/-=。
8. **List\<T\> 扩容**：内部 T[]，2 倍扩容。Add 均摊 O(1)。
9. **Dictionary 底层**：哈希表=bucket[]+Entry[]。GetHashCode→%定位桶→拉链法。
10. **foreach 原理**：GetEnumerator→while(MoveNext)→Current。yield return 生成状态机。
11. **async/await**：编译器生成状态机，await 处暂停→注册回调→Task完成→继续。
12. **Task vs Thread**：Task 基于线程池，支持返回值/async；Thread 是原始 OS 线程。
13. **lock 本质**：Monitor.Enter/Exit 的语法糖，基于 sync block。
14. **IDisposable 模式**：确定性释放，using 语句保证 Dispose()。
15. **反射性能开销**：按名称查找+参数打包拆包。优化：缓存/表达式树编译。
16. **Attribute**：代码的元数据标签，运行时反射获取。
17. **协变逆变**：out T 子→父（IEnumerable）；in T 父→子（Action）。
18. **yield return**：编译器生成 IEnumerator 状态机，每个 yield return 暂停。
19. **LINQ 延迟执行**：定义时不执行，遍历时才执行。ToList 立即执行。
20. **using 三种用法**：引用命名空间；自动 Dispose；C# 8+ using 声明。

### Unity 基础（20题核心要点）

21. **MonoBehaviour 生命周期**：Awake→OnEnable→Start→FixedUpdate→Update→LateUpdate→OnDisable→OnDestroy
22. **MonoBehaviour 为什么不能 new**：需要绑定 GameObject，由引擎管理生命周期。
23. **Update/FixedUpdate/LateUpdate**：Update 每帧；FixedUpdate 固定物理步进；LateUpdate 在 Update 之后。
24. **GetComponent 优化**：组件链表线性搜索 O(n)，Start 中缓存。
25. **协程原理**：IEnumerator+yield return，引擎每帧检查 yield 条件。
26. **ScriptableObject**：数据容器，独立 .asset 文件，不占每帧开销。
27. **Prefab**：GameObject+组件的模板，可实例化同步。
28. **Unity 单例**：static Instance + Awake + DontDestroyOnLoad。
29. **GameObject.Find**：遍历整个场景 O(n)，少用。
30. **协程 vs Invoke**：协程直接调用；Invoke 字符串+反射，开销大。
31. **Destroy vs DestroyImmediate**：Destroy 帧结束后销毁；DestroyImmediate 立即销毁。
32. **position vs localPosition**：世界坐标 vs 相对父节点本地坐标。
33. **Animator vs Animation**：Mecanim+状态机（主流）vs 旧版 Legacy。
34. **OnEnable vs Awake**：Awake 创建时调一次；OnEnable 每次 active 变为 true 时调。
35. **Resources 目录**：无论是否引用都打进包，全量加载到内存，不推荐。
36. **AssetBundle**：资源包，动态加载/卸载，手动管理。
37. **Addressables**：引用计数自动管理，异步加载，现代最佳方案。
38. **Physics.Raycast**：Physics.Raycast(起点, 方向, out hit, 距离, layerMask)
39. **LayerMask**：32位位掩码，每位代表一个 Layer。
40. **Time.deltaTime**：上一帧耗时，用于帧率无关计算。

### 性能优化（20题核心要点）

41. **DrawCall 减少**：Static/Dynamic Batching、GPU Instancing、SRP Batcher、网格合并。
42. **GC 优化**：避免 Update 中 new、对象池、StringBuilder、缓存 GetComponent。
43. **对象池**：预创建→使用时取出→用完放回。减少 Instantiate/Destroy。
44. **LOD**：根据距离切换不同精度模型。
45. **Occlusion Culling**：剔除被遮挡物体不渲染。
46. **MipMap**：纹理多级渐远层级，远处用低分辨率。
47. **Profiler**：Window→Analysis→Profiler，查看 CPU/GPU/内存/Rendering。
48. **合批条件**：同材质、同 Shader Pass。Static Batching 标记 Static；Dynamic Batching<300 顶点。
49. **SRP Batcher**：批量设置材质属性而非合批网格。
50. **Overdraw**：同一像素被多次绘制，减少透明 UI 叠加。
51. **Update 不要太多**：每个 Update 是 C++→C# 跨域调用。
52. **FindObjectOfType vs 单例**：FindObjectOfType O(n)；单例 O(1)。优先单例。
53. **字符串优化**：StringBuilder、避免循环拼接、CompareTag 代替 tag==""。
54. **Mesh.CombineMeshes**：运行时合并多个网格。
55. **Quality Settings 优化**：降低纹理分辨率、关闭阴影、减少 Pixel Light Count。

### 架构与设计 — ECS

ECS（Entity Component System）：数据导向设计。Entity=ID，Component=纯数据，System=逻辑。极致性能。与传统 OOP 的 GameObject-Component 不同，ECS 追求内存连续性（cache-friendly）和并行处理（Job System + Burst）。

---

## 面试真题二：腾讯客户端一面

### 面试节奏（通常 45~60 分钟）

| 环节 | 时长 | 内容 |
|------|------|------|
| 自我介绍 + 项目 | 10min | 选最熟悉项目深讲 |
| C# 基础 | 15min | 值类型/引用类型、GC、委托事件、泛型 |
| Unity 基础 | 15min | 生命周期、协程、性能优化 |
| 算法题 | 15min | 1~2 道中等难度 |

### C# 高频 8 题

1. **值类型和引用类型**：值类型存值本身，赋值复制整个数据；引用类型存地址，赋值复制引用。
2. **GC 原理和分代回收**：Mark-Sweep-Compact 三阶段。Gen0 最频繁，Gen1 缓冲，Gen2 长期存活。
3. **委托和事件**：event 是对 delegate 的封装——外部只能 +=/-=。
4. **Dictionary 底层**：哈希表 = buckets[] + entries[]。拉链法处理碰撞。扩容阈值 ~0.72。
5. **async/await 原理**：编译器生成状态机。死锁：UI线程.Result阻塞+await要回UI线程。
6. **List\<T\> 扩容**：内部 T[]，2倍扩容。Add 均摊 O(1)。预分配容量。
7. **反射性能开销和优化**：按名称查找+参数打包。优化：缓存+Delegate.CreateDelegate+表达式树。
8. **IDisposable 和 Finalizer**：IDisposable 确定释放（using）；Finalizer 不确定释放（GC调），对象多活一轮。

### Unity 高频 8 题

9. **MonoBehaviour 生命周期**：Awake→OnEnable→Start→FixedUpdate→Update→LateUpdate→OnDisable→OnDestroy
10. **MonoBehaviour 不能 new**：需要绑定 GameObject，引擎管理生命周期。
11. **GetComponent 优化**：线性搜索 O(n)，Start/Awake 中缓存；Inspector 拖入最快。
12. **协程原理**：IEnumerator+yield return。主线程执行，不是多线程。
13. **DrawCall 减少**：Static/Dynamic Batching、GPU Instancing、SRP Batcher、网格合并。
14. **Addressables vs Resources**：引用计数自动管理、异步加载。Resources 全量在内存，不推荐。
15. **IL2CPP 原理**：IL→C++→本机代码。iOS 必须 AOT。性能 1.5-2x，不支持动态生成代码。
16. **Update 不要太多**：每个 Update 是 C++→C# 跨域调用。管理器统一 Update。

### 算法题典型

常见题型：反转链表、两数之和（Dictionary O(n)）、有效的括号（Stack）、合并两个有序数组、二叉树层序遍历（Queue BFS）、最长公共前缀、字符串转整数 (atoi)、环形链表检测（快慢指针）。

答题框架：重复题目确认边界 → 说思路（数据结构+复杂度）→ 写代码 → 自测 → 主动说复杂度。

### 项目经验应答

准备：项目架构（设计模式、模块划分）、技术难点（性能优化用数据说话）、团队协作（Git 工作流）、自我反思（可改进之处）。

### 反问

- "项目组目前用的 Unity 版本和技术栈？"
- "客户端开发的日常技术挑战是什么？"
- "新人的成长路径是怎样的？"

---

## 面试真题三：米哈游客户端

### 面试特点

偏引擎底层、渲染管线、图形学基础、C++/C# 深度。相比其他公司更"硬核"。

### C# 深度 7 题

1. **CLR 泛型实现**：引用类型共享同一份 JIT 机器码；值类型各有独立机器码。C++ 模板编译期代码生成，C# 泛型运行时特化。
2. **struct 什么时候在堆上**：class 的字段、装箱、数组元素、闭包捕获、静态字段。
3. **string Intern 和内存泄漏风险**：Intern 的字符串永不被 GC 回收，过度 Intern 会撑大池且有锁竞争。
4. **yield return 状态机**：编译器生成 IEnumerator 实现类，switch(state) 按状态跳转。
5. **await vs yield 状态机区别**：await 是 IAsyncStateMachine 处理 Task 回调；yield 是 IEnumerator 处理数据迭代。await 涉及 SynchronizationContext 和回调注册。
6. **readonly struct 和 ref struct**：readonly struct 消除防御性复制；ref struct 保证只在栈上（Span\<T\>），禁止上堆。
7. **表达式树优化反射**：Expression.Lambda+Compile 生成强类型委托，接近 new 的速度。

### 渲染与图形学 5 题（米哈游核心竞争力）

8. **渲染管线主要阶段**：应用程序阶段 → 几何处理（顶点着色器→曲面细分→几何着色器→裁剪→屏幕映射）→ 光栅化 → 像素处理（片元着色器→逐片元操作）→ FrameBuffer。
9. **Forward vs Deferred Rendering**：Forward O(物体×光源) 适合少光源；Deferred 先渲染 G-Buffer，再屏幕空间光照 O(物体+光源)，适合多光源，但 MSAA 支持差。
10. **PBR 核心思想**：基于物理的渲染：能量守恒 + BRDF。核心参数：Albedo、Metallic、Roughness、Normal。微表面模型（Cook-Torrance）。
11. **阴影实现**：Shadow Map：从光源渲染深度贴图 → 相机渲染时比对深度。问题：Shadow Acne、Peter Panning。解决：Bias、CSM（级联阴影映射）。
12. **图形学基础公式**：向量点乘=夹角cos=投影长度；叉乘=法向量；兰伯特漫反射 `diffuse = lightColor * albedo * max(0, dot(N, L))`；Blinn-Phong = 漫反射+高光（`pow(dot(N,H), shininess)`）+环境光。

### 算法题（可能偏难）

LRU Cache、岛屿数量（DFS/BFS）、最长回文子串（中心扩散）、接雨水（双指针/单调栈）、二叉树最近公共祖先、编辑距离（DP）、合并 K 个有序链表（优先队列）、A* 寻路。

### 反问

- "项目组用的渲染管线是 Built-in 还是 URP/HDRP，有自定义改动吗？"
- "卡通渲染（NPR）方面有什么特别的技术挑战？"

---

## 面试真题四：莉莉丝客户端

### 面试特点

偏向实际项目经验、架构设计、性能优化落地。相比米哈游更"工程化"，少一些图形学。

### C# 高频 5 题

1. **GC 原理和 Unity 优化实战**：Mark-Sweep-Compact。Gen0/Gen1/Gen2。优化：Update 不 new、对象池、StringBuilder、缓存引用。
2. **委托事件内存泄漏**：订阅者未取消订阅（+=后没 -=）。解决：OnEnable+=、OnDisable-=。
3. **async/await 在 Unity 中**：用 UniTask 替代原生 Task：支持 Unity 生命周期、零 GC、可 await Unity API。
4. **泛型在 IL2CPP 下的限制**：泛型虚方法需 IL2CPP 提前生成桥接代码。不确定的泛型实例化可能 MissingMethodException。
5. **深拷贝方式**：手动赋值（最快）、MemberwiseClone（浅拷贝）、序列化/反序列化、反射递归（慢）。

### Unity 实战 5 题

6. **性能优化（用数据说话）**：问题分析 → 措施 → 结果。如 DrawCall 200→50，帧率 25→60 FPS。
7. **UI 性能优化**：Canvas 分层、同一 Atlas 合批、图集（Sprite Atlas）、避免每帧修改 Layout、用 TextMeshPro。
8. **资源管理 / Addressables**：引用计数管理，分组策略。远程更新：Catalog + .bundle 上传 CDN。
9. **热更新方案**：Lua（XLua/ToLua）成熟但互调有开销；ILRuntime C# 热更但解释执行慢；HybridCLR 纯 C# AOT+Interpreter 混合。
10. **网络同步**：帧同步（RTS/FPS竞技）锁步；状态同步（MMO/RPG）权威服务器推送。

### 架构设计

- 分层架构：展示层（View/UI）→ 逻辑层（Manager/System）→ 数据层（Model/Config）
- 模块化：asmdef 拆分，按功能分 Assembly
- 事件系统：全局 EventBus + 模块内事件，全局事件仅用于跨模块通信

### 项目经验（莉莉丝重点）

准备 2~3 个最能体现贡献的项目故事：项目背景 → 角色和贡献 → 最大挑战 → 解决方案 → 量化结果。

---

## 面试真题五：Unity 性能优化

### 优化金字塔

```
         ┌──────────┐
         │  GPU 优化 │ ← DrawCall、Shader、Overdraw
         ├──────────┤
         │  CPU 优化 │ ← Update、GC、算法
         ├──────────┤
         │  内存优化 │ ← 纹理压缩、资源卸载
         ├──────────┤
         │  资源优化 │ ← AssetBundle、Addressables
         ├──────────┤
         │  架构设计 │ ← 对象池、ECS、模块化
         └──────────┘
```

### CPU 优化

**减少 Update 操作**：Start/Awake 缓存引用、管理器统一 Update、降低 Update 频率、事件驱动代替轮询。

**GC 优化（零 GC 编码）**：对象池（子弹、特效、敌人）、StringBuilder 复用（Clear → Append）、缓存 WaitForSeconds 实例、List\<T\> 而非 ArrayList、用 int 枚举替代 string 枚举、CompareTag 代替 tag == "xxx"。

**对象池实现**：
```csharp
public class ObjectPool<T> where T : Component {
    private Queue<T> _pool = new();
    private T _prefab;
    public T Get() => _pool.Count > 0 ? _pool.Dequeue() : Instantiate(_prefab);
    public void Return(T obj) { obj.gameObject.SetActive(false); _pool.Enqueue(obj); }
}
```

**数学运算优化**：sqrMagnitude 代替 magnitude（省 sqrt）、Vector3.Distance → (a-b).sqrMagnitude、缓存 sin/cos 查找表、标记 `[MethodImpl(MethodImplOptions.AggressiveInlining)]`。

### GPU 优化

**DrawCall 优化**：
| 手段 | 说明 |
|------|------|
| Static Batching | 标记 Static，编辑器合并网格 |
| Dynamic Batching | 自动，<300顶点同一材质 |
| GPU Instancing | 同一 Mesh+材质，GPU 侧批量绘制 |
| SRP Batcher | URP/HDRP，批量设置材质属性 |
| Atlas 合图 | 多张小图合成一张 |
| Mesh Combine | 运行时合并网格 |
| LOD | 远处用低模 |
| Occlusion Culling | 遮挡不可见物体不渲染 |

目标：移动端 < 100 DrawCall，PC < 1000 DrawCall。

**Shader 优化**：减少纹理采样次数、移动端避免复杂 per-pixel 计算、用 half/float 精度、避免透明叠加（Overdraw）、Alpha Test 比 Alpha Blend 性能好。

**纹理优化**：压缩格式 ASTC（移动端首选）/ ETC2（Android）/ DXT（PC）；MipMap 远处自动低分辨率；关闭 Read/Write Enabled（双倍内存）。

### 内存优化

- Resources 目录尽量不用（全量打进包 + 常驻内存）
- Addressables 引用计数，无人引用则自动卸载
- 纹理内存 = 宽×高×格式字节数（1024×1024×RGBA32=4MB → ASTC 6×6→0.67MB）

### Profiler 使用

Window → Analysis → Profiler：
- CPU Usage：脚本耗时、GC.Alloc（目标=0）
- GPU Usage：DrawCall、填充率
- Rendering：Batches（=DrawCall）、SetPass Calls
- Memory：纹理、网格、Animation 占用

**Frame Debugger**：逐 DrawCall 查看每帧绘制了什么、合批是否生效、哪个 DrawCall 打破了合批。

### 代码层面优化清单

| 问题 | 方案 |
|------|------|
| GetComponent 每帧调用 | Start 中缓存 |
| GameObject.Find | 引用/单例 |
| string 每帧拼接 | StringBuilder 复用 |
| 频繁 Instantiate/Destroy | 对象池 |
| LINQ 在 Update 中 | 初始处理，循环用 for |
| SendMessage | 直接调用方法 |
| Camera.main | 每帧调 GetComponent。缓存！ |
| foreach 装箱 | 用泛型集合 |
| OnGUI | 不用。用 uGUI |
| Resources.Load 同步卡顿 | 异步加载/Addressables |

### 移动端特殊优化

目标帧率 30fps（省电）/ 60fps（流畅）；`Application.targetFrameRate = 30 / 60`；QualitySettings 降级；移动端 Shader 精度 half > float；Android IL2CPP + ARM64；iOS Xcode 优化等级 -O3。

---

## 面试真题六：C# 高频八股 50 题（一分钟回答模板）

### 类型系统（10题）

1. **值类型 vs 引用类型**：值类型存值本身，在栈（局部）；引用类型存地址，对象在堆。
2. **装箱和拆箱**：装箱=值类型→object，堆分配+复制值，GC压力。泛型是最好方案。
3. **struct 和 class 怎么选**：struct：小而不可变（<16字节参考），不需要继承。class：复杂对象，需要继承和引用语义。
4. **string 为什么不可变**：线程安全+安全共享+字符串池+哈希缓存。循环拼接用 StringBuilder。
5. **Nullable\<T\> 本质**：`Nullable<T> where T : struct` 的 struct。内部 `bool hasValue` + `T value`。
6. **readonly struct 解决什么**：消除防御性复制——in 参数传递时编译器无需复制整份。
7. **ref struct 限制**：只能在栈上（Span\<T\>）。不能是 class 字段、不能装箱、不能存数组。
8. **record**：值相等语义的不可变数据载体。比较属性值而非引用。with 表达式创建副本。
9. **var vs dynamic**：var 编译时推断；dynamic 运行时解析，有 DLR 开销。
10. **const vs readonly**：const 编译时常量（直接嵌入 IL）；readonly 运行时常量（构造器中赋值）。

### 面向对象（8题）

11. **面向对象四大特性**：封装（private+property）、继承（单继承+多接口）、多态（virtual+override）、抽象（abstract/interface）。
12. **virtual vs abstract**：virtual 有默认实现，可选重写；abstract 无实现，强制子类重写。
13. **override vs new**：override=运行时动态绑定（多态）；new=编译时静态绑定（隐藏基类方法）。
14. **interface vs abstract class**：接口=can-do，多实现，无字段；抽象类=is-a，单继承，有字段和构造。
15. **sealed 作用**：sealed class 不可继承；sealed override 禁止进一步重写。可做去虚拟化优化。
16. **依赖倒转（DIP）**：依赖抽象不依赖具体。构造函数注入接口，通过 DI 容器管理依赖。
17. **里氏替换（LSP）**：子类应该能完全替代父类。不要修改父类方法的行为语义。
18. **组合优于继承**：持有其他类实例（has-a）而非继承（is-a）。可运行时组合/切换行为。

### 委托与事件（5题）

19. **delegate 本质**：类型安全的函数指针，继承自 MulticastDelegate→Delegate。核心字段：_target、_methodPtr。
20. **event 比 delegate 安全**：外部只能 +=/-=，不能 =（清空）也不能 Invoke（伪造触发）。
21. **Action/Func/Predicate**：Action=无返回值；Func=有返回值；Predicate=返回 bool。
22. **Lambda 闭包原理**：捕获外部变量→编译器创建 DisplayClass，变量提升为字段。闭包延长变量生命周期。
23. **事件忘记取消订阅**：订阅者被 Destroy，但事件仍持有其引用→无法 GC→内存泄漏。OnEnable+=、OnDisable-=。

### 集合（5题）

24. **List\<T\> 扩容**：内部 T[]。容量不足→2倍扩容→Array.Copy。默认 0→4→8→16...
25. **Dictionary 底层**：哈希表=buckets[]+entries[]。GetHashCode→%buckets.Length→定位桶→拉链法。扩容阈值~0.72。
26. **HashSet 原理**：只有 Key 没有 Value 的 Dictionary。存储唯一值，O(1) 增删查。
27. **Queue/Stack 底层**：Queue 环形数组；Stack 数组。都是 2 倍扩容。
28. **ConcurrentDictionary**：线程安全字典，细粒度锁（lock per bucket），原子操作。

### 泛型（3题）

29. **泛型为什么快**：避免装箱拆箱+编译时类型安全。List\<int\> 内部就是 int[]，零装箱。
30. **C# 泛型 vs C++ 模板**：C++ 编译期代码生成（二进制膨胀）；C# 运行时特化（引用类型共享，值类型各一份）。
31. **协变(out)逆变(in)**：out T 子→父（IEnumerable）；in T 父→子（Action）。

### 异步与线程（5题）

32. **async/await 原理**：编译器生成 IAsyncStateMachine 状态机。await 处暂停→注册回调→Task完成→MoveNext 继续。
33. **Task vs Thread**：Task 基于线程池，轻量级，支持返回值/async。Thread 原始 OS 线程，~1MB 栈。
34. **死锁经典场景**：UI 线程 . Result 阻塞 + await 结束后要回 UI 线程 → 死锁。解决：全部 await 到底。
35. **lock 本质**：Monitor.Enter/Exit 的语法糖。锁专用 object，不锁 this/typeof/string。
36. **async void 危险**：无法 await，异常无法捕获。除了 UI 事件处理器，永远不用 async void。

### LINQ 与迭代（4题）

37. **LINQ 延迟执行**：Where/Select 定义时不执行，遍历时才执行。每次遍历重新执行。
38. **IEnumerable vs IEnumerator**：IEnumerable 提供迭代器（可被 foreach）；IEnumerator 是迭代器本身（游标）。
39. **yield return 原理**：编译器生成 IEnumerator 状态机，switch(state) 按状态跳转。
40. **foreach vs for 性能**：数组上 for 快 5-15%。foreach 对数组有编译器优化。热路径才考虑 for。

### GC 与内存（5题）

41. **GC 三阶段**：Mark（标记可达）→ Sweep（回收不可达）→ Compact（压缩消除碎片）。
42. **LOH**：≥85000 字节进 LOH。Gen2 回收时处理，默认不压缩→碎片风险。
43. **Finalizer vs IDisposable**：Finalizer 不确定调用，对象多活一轮。IDisposable 确定调用，using 自动调。
44. **using 两种写法**：`using(var x = new Resource()){...}`；`using var x = new Resource();`（C# 8+）。
45. **GC.Collect**：一般不该调。手动调用打断分代策略，让短命对象错误晋升到高代。

### 反射与特性（3题）

46. **反射原理和性能**：运行时查询 CLR Metadata。Invoke 慢在参数打包拆包+安全检查。优化：缓存+Delegate.CreateDelegate+表达式树。
47. **Attribute**：继承 Attribute，标记于类型/方法。反射 GetCustomAttribute 获取。
48. **动态创建对象**：new ~1ns → Activator ~100ns → ConstructorInfo.Invoke ~200ns → Expression.Compile() ~2ns。

### 其他（2题）

49. **throw vs throw ex**：throw 保留原始堆栈跟踪；throw ex 重置堆栈。始终用 throw。
50. **C# 版本重要特性**：
    - C# 6: ?. (null 条件)、nameof、字符串插值
    - C# 7: out var、ref return、ValueTuple
    - C# 8: nullable reference types、switch 表达式、using 声明
    - C# 9: record、init-only
    - C# 10: file-scoped namespace、record struct
    - C# 11/12: raw string literals、primary constructors、collection expressions

---

> 本文档整合自 26 个专题 + 6 套面试真题，涵盖 C# 基础、CLR 原理、Unity 实战、性能优化、各厂面试风格。可作为面试前系统复习的完整参考手册。

---

# 第五部分：面试高频追问与深入专题

## 专题一：LINQ 与 Lambda 表达式底层原理（★★★★★）

### 1.1 Lambda 表达式的底层实现

Lambda 表达式本质上是 C# 编译器提供的**语法糖**。编译器在编译阶段会根据 Lambda 的目标类型（Delegate 还是 Expression），将其编译为不同的形态：

**1. 普通委托 → 静态/实例方法**

当 Lambda 赋值给 `Func<T>` 或 `Action<T>` 时，编译器会在当前类中隐式生成一个带有随机命名（如 `<Main>b__0_0`）的私有方法，并创建一个指向该方法的委托实例。若 Lambda 没有引用外部变量，编译器还会缓存该委托实例以避免重复分配内存。

```csharp
// 源码
Func<int, int> square = x => x * x;

// 编译器生成（简化）
[CompilerGenerated]
private static int <Main>b__0_0(int x) => x * x;
Func<int, int> square = new Func<int, int>(<Main>b__0_0);
// 委托实例被缓存，多次调用不重复分配
```

**2. 闭包（Closure）→ DisplayClass**

若 Lambda 内部使用了外部函数的局部变量，编译器会额外生成一个隐藏的泛型类（如 `<>c__DisplayClass0_0`）。捕获的局部变量会被提升为该类的公共字段，Lambda 逻辑变为该类的成员方法。这确保了局部变量在超出原函数作用域后依然存活在堆中。

```csharp
// 源码
int threshold = 100;
var query = list.Where(x => x > threshold);

// 编译器生成（简化）
[CompilerGenerated]
private sealed class <>c__DisplayClass0_0 {
    public int threshold;  // 局部变量提升为字段
    internal bool <M>b__0(int x) => x > threshold;
}

var displayClass = new <>c__DisplayClass0_0();
displayClass.threshold = 100;
var query = Enumerable.Where(list, new Func<int, bool>(displayClass.<M>b__0));
```

**3. 表达式树（Expression Tree）**

当 Lambda 赋值给 `Expression<Func<...>>` 时，编译器不会生成可执行的 CIL 字节码，而是将代码结构解析为一棵**抽象语法树（AST）**。树的节点代表代码中的运算符、变量名、方法调用等元素（如 `BinaryExpression`、`ParameterExpression`）。

```csharp
Expression<Func<int, bool>> expr = x => x > 100;
// 编译器生成表达式树：
// Expression.Lambda<Func<int, bool>>(
//     Expression.GreaterThan(
//         Expression.Parameter(typeof(int), "x"),  // x
//         Expression.Constant(100)                   // 100
//     ),
//     new ParameterExpression[] { x }
// )
```

### 1.2 LINQ 的底层实现

LINQ 结合了扩展方法、泛型、迭代器以及表达式树，在底层根据操作对象的不同分为两类：

**1. 语法降级（Syntactic Sugar Translation）**

LINQ 查询语法（如 `from x in list where x > 0 select x`）在编译期被直接翻译为方法链语法（`list.Where(x => x > 0).Select(x => x)`）。

**2. LINQ to Objects（基于 IEnumerable\<T\>）**

- 定义于 `System.Linq.Enumerable` 扩展方法中。
- **延迟执行（Deferred Execution）**：通过 C# 的 `yield return` 实现。编译器会将包含 `yield return` 的扩展方法改写为一个实现了 `IEnumerator<T>` 的状态机类。只有在显式遍历（如 `foreach`、`.ToList()`）时，状态机才开始一步步求值，实现按需计算与链式组合。
- 每个 LINQ 操作符返回一个新的迭代器，形成**迭代器链**：`Where迭代器 → Select迭代器 → Take迭代器`，从外向内逐层拉取数据。

**3. LINQ to Provider / EF Core（基于 IQueryable\<T\>）**

- 定义于 `System.Linq.Queryable` 扩展方法中，方法接收的参数是 `Expression<Func<...>>`。
- 当调用 `.Where()` 或 `.Select()` 时，系统不会执行计算，而是将传入的表达式树拼接成一棵更宏大的语法树。当最终触发遍历时，底层的 Query Provider（如 EF Core 的 SQL 翻译器）遍历解析这棵语法树，将其转换为目标数据库原生语句（如 SQL），发送给数据库运行。

**两类 LINQ 执行机制对比**：

| 维度 | LINQ to Objects (IEnumerable\<T\>) | LINQ to Provider (IQueryable\<T\>) |
|------|------|------|
| 参数类型 | 委托 `Func<T, bool>` | 表达式树 `Expression<Func<T, bool>>` |
| 底层生成 | 字节码 + yield 状态机 | 内存树状数据结构（AST） |
| 执行位置 | 本地 CPU / 内存 | 远程服务（如 SQL Server、MongoDB） |
| 核心机制 | 迭代器按需拉取数据 | Provider 解析 AST 并翻译为目标查询语言 |

### 1.3 关键结论

- Lambda 表达式 = 匿名方法的语法糖，编译器决定生成委托还是表达式树
- 闭包 = Lambda + 捕获的外部变量，编译器生成 DisplayClass，变量提升为堆字段
- LINQ 查询表达式 = 编译期翻译为方法链语法
- IEnumerable LINQ = 迭代器链 + 延迟执行
- IQueryable LINQ = 表达式树 + Provider 翻译为远程查询

---

## 专题二：Unity Update 方式详解

### 2.1 三种 Update 回调函数

Unity 在主循环（Main Loop）中提供了三种主要的 Update 回调函数，触发时机与用途各不相同：

| 回调 | 执行频率 | 主要用途 |
|------|------|------|
| **Update()** | 每渲染帧一次，频率取决于 FPS | 玩家输入、非物理逻辑、计时器、平滑插值 |
| **FixedUpdate()** | 固定时间间隔（默认 0.02s / 50Hz） | 刚体（Rigidbody）操作、力计算、物理移动 |
| **LateUpdate()** | Update 全部执行完后触发 | 摄像机跟随、程序化动画修正 |

### 2.2 详细执行机制

**Update()**：
- 每渲染一帧执行一次，执行频率完全取决于当前游戏的帧率（FPS）
- 由于每帧间隔时间不固定，需搭配 `Time.deltaTime` 保证逻辑与帧率无关
- 一帧内可能不执行 FixedUpdate、执行 1 次 FixedUpdate、或执行多次 FixedUpdate（取决于帧率高低）

**FixedUpdate()**：
- 按固定的时间间隔执行（可在 Time Settings 中修改 Fixed Timestep）
- 所有涉及 Rigidbody 和物理引擎（PhysX）的代码必须放在此处，以确保物理模拟的确定性与稳定性
- 物理碰撞检测也在 FixedUpdate 阶段处理

**LateUpdate()**：
- 在当前帧所有脚本的 Update() 执行完毕后统一触发
- 典型用途：摄像机跟随——确保目标对象在 Update 中移动完成后，摄像机再计算最终位置，避免镜头抖动

### 2.3 一帧内的完整执行顺序

```
Awake → OnEnable → Start
  ↓
┌─ 物理循环（可能执行 0~N 次）──┐
│ FixedUpdate                    │
│ 物理模拟 + 碰撞检测              │
│ OnTriggerXXX / OnCollisionXXX  │
└────────────────────────────────┘
  ↓
Update（所有脚本）
  ↓
LateUpdate（所有脚本）
  ↓
渲染管线（Render Pipeline）
  ↓
OnDisable → OnDestroy
```

---

## 专题三：物理碰撞与穿模分析

### 3.1 碰撞实现原理

Unity 的物理系统（PhysX/Box2D）每帧在 FixedUpdate 阶段处理碰撞，过程分为两阶段：

1. **粗阶段（Broad-Phase）**：通过空间划分（如 AABB 树）快速过滤掉距离较远、不可能碰撞的物体。
2. **细阶段（Narrow-Phase）**：对相近的碰撞体进行精准图元相交测试（Mesh/Box/Sphere/Capsule）。若检测到交集，物理求解器（Solver）会计算碰撞点、冲量，并通过排斥力将重叠的刚体分离。

### 3.2 速度过快导致穿模（Tunneling）

**根本原因**：物理引擎默认采用**离散碰撞检测（Discrete Collision Detection）**。引擎仅在离散的时间点（t₁, t₂, t₃...）采样物体的空间位置。

- 若物体速度极快或碰撞体厚度极薄，在 t₁ 时刻物体位于墙前，在下一个物理采样点 t₂ 时，物体已直接穿越到了墙后。
- 由于在 t₁ 和 t₂ 两个离散时刻，碰撞体均未发生重叠，物理引擎便判定未发生碰撞，从而造成"穿模"。

```
t₁: 子弹 ● →        ║墙║
t₂:        ║墙║  ● 子弹（已穿过！）
碰撞体从未重叠 → 无碰撞事件！
```

### 3.3 解决方案

| 方案 | 说明 |
|------|------|
| **连续碰撞检测（CCD）** | 将高速物体的 Rigidbody Collision Detection 改为 Continuous 或 Continuous Speculative，利用扫掠/预测算法检测移动路径上的碰撞 |
| **预测射线检测（Raycast/SphereCast）** | 在移动前向移动方向发射射线，提前预判并拦截超高速物体（如子弹） |
| **增大碰撞体厚度** | 增大碰撞体尺寸，使其更难被"跳过" |
| **降低 Fixed Timestep** | 增加物理采样密度（如从 0.02 降到 0.01），但会提高 CPU 开销 |

---

## 专题四：事件系统大规模优化

### 4.1 事件系统实现思路

Unity 中的事件系统通常基于**观察者模式（Observer Pattern）**或**发布-订阅模式（Pub/Sub）**：

- 底层利用 C# 的 `delegate` / `event` 或自定义 EventBroker。
- 发布者（Publisher）维护一个委托链表，订阅者（Subscriber）将自己的回调方法注册（`+=`）到该事件中。
- 当事件触发时，发布者遍历链表同步调用所有已注册的回调。

### 4.2 接收对象过多导致掉帧的处理方案

若同步触发大量订阅者的回调，主线程单帧开销会瞬间爆表。优化思路如下：

| 优化策略 | 方案说明 |
|------|------|
| **时间切片（Time-Slicing）** | 将事件推入队列，不要在同一帧同步广播。利用协程或 Async/Await 控制每帧仅处理一部分队列元素（例如限制每帧事件处理耗时不超过 2ms） |
| **空间/兴趣过滤（Spatial Partitioning）** | 引入网格（Grid）或八叉树。广播时仅通知处于事件源有效视野/影响半径内的对象，而非全局广播 |
| **分频道/精准订阅** | 避免全局大杂烩事件，按类型细化订阅频道；或使用枚举/ID 进行二次过滤 |
| **轻量化回调与多线程处理** | 回调函数内部严禁使用 `GetComponent`、`Instantiate` 或复杂的 LINQ 操作；仅做数据标记，将密集计算转移至 C# Job System 配合 Burst 编译器并行处理 |

```csharp
// 时间切片示例：分帧处理事件队列
public class EventQueue : MonoBehaviour {
    private Queue<Action> _pendingEvents = new();
    
    public void Enqueue(Action evt) => _pendingEvents.Enqueue(evt);
    
    IEnumerator ProcessEvents() {
        var stopwatch = new System.Diagnostics.Stopwatch();
        while (_pendingEvents.Count > 0) {
            stopwatch.Restart();
            while (stopwatch.ElapsedMilliseconds < 2 && _pendingEvents.Count > 0) {
                _pendingEvents.Dequeue()?.Invoke();
            }
            yield return null; // 每 2ms 让出一帧
        }
    }
}
```

---

## 专题五：Unity 动画系统（Mecanim）

### 5.1 核心架构组成

Mecanim 是 Unity 的核心动画架构，基于**有限状态机（FSM）**与数据驱动设计：

| 组件 | 说明 |
|------|------|
| **AnimationClip** | 基础动画资源文件（走、跑、跳等动画数据） |
| **Avatar** | 骨骼映射抽象层，将不同模型的骨骼统一映射，实现动画重定向（Animation Retargeting） |
| **Animator Controller** | 动画状态机资产，包含状态节点、参数（Float/Int/Bool/Trigger）及状态切换条件（Transitions） |
| **Animator** | 挂载在 GameObject 上的控制组件，负责驱动状态机 |

### 5.2 动画混合树（Blend Tree）

Blend Tree 用于根据连续参数平滑融合多个 AnimationClip，避免繁琐的状态机连线：

**1D Blend Tree**：基于单一参数混合。例如基于 Speed 参数在"站立 → 走路 → 跑步"之间平滑过渡。

```
Speed = 0    → 站立动画 100%
Speed = 2.5  → 站立 50% + 走路 50%
Speed = 5    → 走路 50% + 跑步 50%
Speed = 10   → 跑步动画 100%
```

**2D Blend Tree**：基于两个参数混合。例如基于 Horizontal 和 Vertical 两个输入值，平滑融合前、后、左、右及斜向的 8 方向移动动画。

```
          Vertical=1 (前)
              │
  Horizontal=-1 ─┼─ Horizontal=1
              │
          Vertical=-1 (后)
```

### 5.3 动画分层与混合（Layers & Layer Blending）

- **Layer（动画层）**：允许同时运行多个状态机。例如，Layer 0 控制下半身移动，Layer 1 控制上半身开火。
- **Avatar Mask（骨骼遮罩）**：指定某个动画层仅作用于角色的部分骨骼（如仅作用于胸部以上）。
- **Blending Mode**：
  - **Override（覆盖）**：高层动画权重高时直接覆盖底层动作。
  - **Additive（叠加）**：高层动画作为增量叠加到底层动作上（如在跑步动画的基础上叠加"受击晃动"或"呼吸"效果）。

### 5.4 PlayableGraph

PlayableGraph 是 Unity 提供的底层动画控制 API，允许程序化控制动画混合、过渡和分层。相比 Animator Controller，PlayableGraph 提供：
- 更灵活的动画混合控制
- 运行时动态创建动画状态
- 自定义混合逻辑
- 更高的性能（减少 Animator Controller 开销）

```csharp
// PlayableGraph 基本用法
var playableGraph = PlayableGraph.Create("MyGraph");
var animationOutput = AnimationPlayableOutput.Create(playableGraph, "Output", GetComponent<Animator>());
var clipPlayable = AnimationClipPlayable.Create(playableGraph, animationClip);
animationOutput.SetSourcePlayable(clipPlayable);
playableGraph.Play();
```

### 5.5 Motion Matching 动画方案

Motion Matching 是新一代动画方案，不同于传统状态机：
- 每帧从动画数据库中搜索与当前角色状态（速度、方向、姿势）最匹配的动画帧
- 无需手动构建状态机和过渡条件
- 动画效果更自然流畅，但需要大量动画数据支持
- Unity 2023+ 提供了内置 Motion Matching 支持
