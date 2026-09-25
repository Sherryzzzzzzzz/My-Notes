# C# 核心知识整理

> 整合自 C# 笔记，涵盖基础语法、面向对象、集合、泛型、反射、异步、内存与 GC。

---

# 目录

- [01 C#基础与常识](#01-c基础与常识)
- [02 数据类型与内存](#02-数据类型与内存)
- [03 值类型与引用类型](#03-值类型与引用类型)
- [04 字符串](#04-字符串)
- [05 ref与out](#05-ref与out)
- [06 重载与重写](#06-重载与重写)
- [07 static](#07-static)
- [08 委托 delegate](#08-委托-delegate)
- [09 事件 event](#09-事件-event)
- [10 深拷贝与浅拷贝](#10-深拷贝与浅拷贝)
- [11 for与foreach](#11-for与foreach)
- [12 面向对象](#12-面向对象)
- [13 接口](#13-接口)
- [14 结构体与类](#14-结构体与类)
- [15 New关键字](#15-new关键字)
- [16 集合](#16-集合)
- [17 泛型](#17-泛型)
- [18 反射](#18-反射)
- [19 异常](#19-异常)
- [20 异步编程](#20-异步编程)
- [21 内存分布与管理](#21-内存分布与管理)
- [22 垃圾回收 GC](#22-垃圾回收-gc)

---

# 01 C#基础与常识

## .NET 与 Mono

Mono 是 .NET 的开源跨平台实现。.NET Framework 主要面向 Windows，Mono 让 .NET 应用能跑在 Linux、macOS 等平台上。类比：Java 本身不是跨平台，但运行在 JVM 上就跨平台了——Mono 相当于各平台的"虚拟机"。

## C# 语言特性

1. **纯面向对象**：完全支持封装、继承、多态
2. **强类型系统**：编译时捕获类型错误，值类型和引用类型合理设计
3. **自动内存管理**：GC 自动管理内存
4. **现代化特性**：Lambda、LINQ、async/await
5. **跨平台**：Windows、Linux、macOS、Android/iOS

## JIT 与 AOT

```
C# 程序执行流程：
  C# 源码 → C#编译器(csc) → IL(中间语言) → CLR → JIT → 机器码 → CPU
```

| | JIT（即时编译） | AOT（提前编译） |
|---|---|---|
| **时机** | 运行时按需编译 | 运行前全部编译 |
| **优点** | 可动态优化、热点优化 | 无运行时编译开销 |
| **缺点** | 首次调用有编译开销 | 无法利用运行时信息优化 |
| **例子** | 标准 .NET | Unity IL2CPP |

> CLR = .NET 的"操作系统"，负责 GC、JIT、异常、线程、类型安全、反射等所有运行时服务。

---

# 02 数据类型与内存

## C# 数据类型分类

| 类型 | 关键字 | 存在哪 |
|------|--------|--------|
| **值类型** | int, float, bool, char, struct, enum | 栈（局部变量）/ 随对象在堆（字段） |
| **引用类型** | class, interface, delegate, string, array | 对象在堆，引用在栈 |
| **指针类型** | int*, float* (需 unsafe) | 指向任意内存 |

## 基本类型占用字节数

| 类型 | 字节 | 范围 |
|------|------|------|
| bool | 1 | true/false |
| byte/sbyte | 1 | 0~255 / -128~127 |
| char | 2 | Unicode |
| short/ushort | 2 | ±3.2万 / 0~6.5万 |
| int/uint | 4 | ±21亿 / 0~42亿 |
| long/ulong | 8 | ±9×10¹⁸ |
| float | 4 | 单精度 |
| double | 8 | 双精度 |
| decimal | 16 | 高精度金融 |

---

# 03 值类型与引用类型

## 核心区别

```
值类型 ── 直接存数据          引用类型 ── 存引用(地址)
  栈上                        栈上存引用，堆上存对象
  赋值 = 复制整个值            赋值 = 复制引用
  继承自 System.ValueType      继承自 System.Object
```

## 内存分布

```
栈 (Stack)                           堆 (Heap)
┌──────────────┐                ┌──────────────────┐
│ int x = 10   │                │  class 对象       │
│ str ref ●────┼───────────────→│  "Hello"          │
│ p ref ●──────┼──────┐         └──────────────────┘
└──────────────┘      │         ┌──────────────────┐
                      └────────→│  new Person()     │
值类型(局部)直接在栈            └──────────────────┘
引用类型对象在堆
```

## 装箱（Boxing）与拆箱（Unboxing）

**装箱 = 值类型 → object（在堆上包装）**

```csharp
int value = 42;
object obj = value;  // 装箱！堆上分配新对象，复制 42
```

**拆箱 = object → 值类型（从堆上提取）**

```csharp
int unboxed = (int)obj;  // 拆箱！检查类型 + 复制值
```

**装箱的性能代价：** 堆分配 + 复制 → 增加 GC 压力。Unity 中频繁装箱导致卡顿。

**减少装箱的方法：**
- 用泛型集合（`List<int>` 替代 `ArrayList`）
- 重写 `ToString()`、`GetHashCode()`、`Equals()` 避免隐式装箱
- 使用 `yield return null` 而非 `yield return 0`

---

# 04 字符串

## String vs StringBuilder

| | String | StringBuilder |
|---|---|---|
| **可变性** | **不可变**（每次修改创建新对象） | **可变**（内部缓冲区） |
| **性能** | 少量操作 OK，大量拼接慢 | 大量拼接快 |
| **线程安全** | ✓ 不可变天然安全 | ✗ 非线程安全 |

```csharp
// String：每次 += 都产生垃圾
string s = ""; for(int i=0;i<10000;i++) s += "text"; // 慢！

// StringBuilder：内部缓冲区直接追加
var sb = new StringBuilder(); for(int i=0;i<10000;i++) sb.Append("text"); // 快！
```

> 编译器已对 `string +` 做了 StringBuilder 优化，一般场景用 String 即可。

## 常用字符串 API

```csharp
string s = "Hello World";
s.IndexOf('o');        // 4
s.Contains("World");   // true
s.Replace("World","C#"); // "Hello C#"
s.Split(' ');           // ["Hello","World"]
s.Substring(2,4);       // "llo "
s.Trim();               // 去首尾空白
s.ToUpper();            // "HELLO WORLD"
string.Format("Hi {0}!", "C#");  // "Hi C#!"
// C# 6+ 字符串插值
$"Hi {name}!";
// @ 逐字字符串
string path = @"C:\Users\file.txt";
```

---

# 05 ref与out

| | ref | out |
|---|---|---|
| **初始化** | 调用前必须初始化 | 调用前无需初始化 |
| **方法内赋值** | 可不赋值 | **必须赋值** |
| **语义** | "修改现有值"（有进有出） | "输出新值"（只出不进） |

```csharp
void Swap(ref int a, ref int b) { int t=a; a=b; b=t; }
int x=10, y=20;
Swap(ref x, ref y);  // x=20, y=10

void Divide(int a, int b, out int q, out int r) {
    q = a/b; r = a%b;
}
int q, r;
Divide(10, 3, out q, out r);  // q=3, r=1
```

---

# 06 重载与重写

| | 重载（Overload） | 重写（Override） |
|---|---|---|
| **作用域** | 同一类中 | 父子类之间 |
| **方法名** | 相同 | 相同 |
| **参数** | **不同** | **相同** |
| **基类方法** | 不需要 | 必须是 virtual/abstract |
| **时机** | 编译时决定 | **运行时决定**（多态） |

```csharp
// 重载
int Add(int a, int b) { return a+b; }
double Add(double a, double b) { return a+b; }

// 重写
class Animal { public virtual void Speak() {} }
class Dog : Animal { public override void Speak() { "汪汪".Dump(); } }
```

---

# 07 static

## static 字段 vs 非静态字段

| | 静态字段 | 非静态字段 |
|---|---|---|
| **属于** | 类本身 | 实例 |
| **存储** | 静态存储区 | 随对象（堆/栈） |
| **生命周期** | 程序加载→结束 | 随对象 |
| **访问** | `ClassName.field` | `obj.field` |

## 静态构造函数

- 无访问修饰符，无参数，只执行一次
- 在第一次访问类的静态成员时自动调用

## Unity 中 static 的常见用途

```csharp
// 1. 全局状态
public class GameManager {
    public static int score;
    public static bool isGameOver;
}

// 2. 单例模式
public class AudioManager : MonoBehaviour {
    public static AudioManager Instance { get; private set; }
}

// 3. 工具类
public class MathUtils {
    public static float Lerp(float a, float b, float t) => a + (b-a)*t;
}
```

---

# 08 委托 delegate

## 是什么？

**委托 = 类型安全的函数指针。** 引用类型。存储对方法的引用，可作为参数传递。

```csharp
// 声明委托类型
public delegate int MathOp(int a, int b);

// 使用
MathOp op = (a, b) => a + b;
int result = op(3, 5);  // 8
```

## 多播委托

```csharp
MathOp op = Add;
op += Subtract;
op(5, 3);  // 依次调 Add 再调 Subtract。返回最后注册的方法的结果
```

## Action 和 Func——内置委托

```csharp
// Action：无返回值（0~16个参数）
Action<string> print = Console.WriteLine;

// Func：有返回值（最后一个是返回值类型）
Func<int, int, int> add = (a, b) => a + b;
```

> **现代 C# 很少自定义委托，直接用 Action/Func + Lambda。**

## 委托 vs 事件区别

| | 委托 | 事件 |
|---|---|---|
| 外部赋值 | ✅ `=` | ❌ 只能 `+=`/`-=` |
| 外部调用 | ✅ | ❌ 只能在声明类内触发 |
| 适用场景 | 回调、策略 | 发布-订阅 |

**事件 = 加了权限控制的委托。** 防止外部把订阅者清空。

---

# 09 事件 event

## 标准事件模式

```csharp
// 自定义事件参数
public class HealthChangedEventArgs : EventArgs {
    public int OldHealth { get; }
    public int NewHealth { get; }
}

// 发布者
public class Player {
    public event EventHandler<HealthChangedEventArgs> OnHealthChanged;

    public void TakeDamage(int dmg) {
        int old = health;
        health -= dmg;
        OnHealthChanged?.Invoke(this, new HealthChangedEventArgs(old, health));
    }
}

// 订阅者
player.OnHealthChanged += (sender, e) => {
    Console.WriteLine($"HP: {e.OldHealth} → {e.NewHealth}");
};
```

## Unity 中的事件 vs UnityEvent

```csharp
// C# 原生事件：代码侧使用，性能好
public event Action<int> OnScoreChanged;

// UnityEvent：Inspector 可拖拽配置
using UnityEngine.Events;
public UnityEvent<int> OnScoreChanged;
```

## 常见陷阱

1. **忘记取消订阅 → 内存泄漏**：`OnEnable` 中 `+=`，`OnDisable` 中 `-=`
2. **Lambda 隐式捕获 this**：匿名方法会持有 this 引用
3. **安全调用**：始终用 `?.Invoke()` 防 null

---

# 10 深拷贝与浅拷贝

| | 浅拷贝 | 深拷贝 |
|---|---|---|
| **引用类型字段** | 复制引用（共享同一对象） | 递归复制（完全独立） |
| **实现** | `MemberwiseClone()` | 序列化 / 手动 |
| **性能** | 快 | 慢 |
| **场景** | 临时副本 | 需完全隔离 |

```csharp
// 浅拷贝：Skills 共享
var copy = (Person)original.MemberwiseClone();
copy.Skills.Add("新技能");  // ⚠️ original 的 Skills 也被改了！

// 深拷贝（JSON 序列化）
var json = JsonSerializer.Serialize(original);
var deep = JsonSerializer.Deserialize<Person>(json);
```

---

# 11 for与foreach

| | for | foreach |
|---|---|---|
| **数组遍历** | 快 5-15%（直接索引） | 稍慢（迭代器） |
| **List\<T\>** | 快 | 编译器优化后接近 |
| **修改集合** | ✅ 注意索引 | ❌ 禁止，抛异常 |
| **可读性** | 需写索引 | 简洁 |

---

# 12 面向对象

## 四大支柱

| 支柱 | 解决什么 | 怎么实现 |
|------|---------|---------|
| **封装** | 隐藏内部状态 | private + 属性/property |
| **继承** | 代码复用 | `class Dog : Animal`（单继承） |
| **多态** | 统一接口不同行为 | virtual + override |
| **抽象** | 隐藏复杂度 | abstract class / interface |

```csharp
public class Dog : Animal {
    public override void MakeSound() => Console.WriteLine("汪汪");
}
Animal[] zoo = {new Dog(), new Cat()};
foreach(var a in zoo) a.MakeSound();  // 多态！
```

## C# 访问修饰符

| 修饰符 | 访问范围 |
|--------|---------|
| public | 无限制 |
| private | 仅当前类 |
| protected | 当前类 + 派生类 |
| internal | 当前程序集 |
| protected internal | 程序集内 OR 派生类 |

---

# 13 接口

**接口 = 行为契约。** 只定义"必须做什么"，不规定"怎么做"。

```csharp
public interface IShape {
    double Area();
    string Name { get; }
}

public class Circle : IShape {
    public double Radius { get; set; }
    public string Name => "圆形";
    public double Area() => Math.PI * Radius * Radius;
}
```

## 接口 vs 抽象类

| | 接口 | 抽象类 |
|---|---|---|
| 多继承 | ✅ 可实现多个 | ❌ 单继承 |
| 字段 | ❌ | ✅ |
| 构造函数 | ❌ | ✅ |
| 默认实现 | C# 8+ | ✅ |
| 语义 | "能做什么" (can-do) | "是什么" (is-a) |

---

# 14 结构体与类

## 核心区别

| | struct（值类型） | class（引用类型） |
|---|---|---|
| **内存** | 栈（局部）/ 堆（字段中） | 堆 |
| **继承** | ❌（但可实现接口） | ✅ |
| **赋值** | **复制整个值** | **复制引用** |
| **传参** | 传副本 | 传引用 |
| **适用** | 小而不可变数据（Point, Vector3） | 复杂对象、需继承 |

## 关键行为差异

```csharp
// struct：赋值是独立副本
var s1 = new PointStruct { X=1 };
var s2 = s1; s2.X = 100;
Console.WriteLine(s1.X);  // 1（不受影响！）

// class：赋值是共享引用
var c1 = new PointClass { X=1 };
var c2 = c1; c2.X = 100;
Console.WriteLine(c1.X);  // 100（被改了！）
```

## struct 的装箱陷阱

```csharp
var p = new PointStruct { X=1 };
object obj = p;       // ⚠️ 装箱！
IEquatable<PointStruct> eq = p;  // ⚠️ 装箱！
```

> **Unity 例子**：`Vector3`、`Quaternion`、`Color` 是 struct；`GameObject`、`Transform` 是 class。

---

# 15 New关键字

## 三种用途

```csharp
// 1. new运算符：创建对象（托管堆分配 + 调用构造函数 + 返回引用）
var obj = new MyClass();

// 2. new修饰符：隐藏基类成员（非多态）
public new void Method() {}

// 3. new约束：泛型必须有无参构造
public T Create<T>() where T : new() => new T();
```

## new 执行流程

```
1. CLR在托管堆分配内存（根据类型计算大小）
2. 字段初始化为默认值（int=0, 引用=null）
3. 调用构造函数：基类构造 → 当前类构造
4. 返回对象引用（栈上的变量存地址→堆上的对象）
```

---

# 16 集合

## ArrayList vs List\<T\>

| | ArrayList | List\<T\> |
|---|---|---|
| 泛型 | ❌ | ✅ |
| 类型安全 | ❌ | ✅ 编译时检查 |
| 值类型 | 装箱/拆箱 | 直接存储 |
| 推荐 | ❌ 旧代码 | ✅ |

## List\<T\> 底层原理

```csharp
public class List<T> {
    private T[] _items;    // 内部数组
    private int _size;     // 元素数量
    private int _version;  // 版本号（检测迭代中修改）
}
// 扩容：容量不足时 → 分配新数组(2倍) → Array.Copy → 旧数组GC回收
// 默认容量：0→4→8→16→32...
```

| 操作 | 时间复杂度 |
|------|-----------|
| Add | 均摊 O(1) |
| Remove/RemoveAt | O(n) |
| Insert | O(n) |
| Contains/IndexOf | O(n) |
| Sort | O(n log n) |

## Dictionary\<TKey, TValue\>

```csharp
// 内部结构
class Dictionary<TKey, TValue> {
    struct Entry {
        int hashCode;
        int next;      // 碰撞链：下一个Entry索引
        TKey key;
        TValue value;
    }
    int[] _buckets;    // 桶数组，存Entry索引+1（0=空桶）
    Entry[] _entries;  // 所有键值对
}
```

| 操作 | 时间复杂度 | 原理 |
|------|-----------|------|
| Add | O(1) | hash→定位桶→链入 |
| ContainsKey | O(1) | hash→定位桶→遍历碰撞链 |
| ContainsValue | O(n) | 遍历所有Entry |
| operator[] | O(1) | hash直接定位 |

> `GetHashCode` 实现不好 → 所有 key 进同一桶 → 退化为链表 O(n)

## 哈希表原理

```
Key → Hash函数(GetHashCode) → 整数值 → % 数组长度 → 桶索引 → 存Entry

碰撞（不同key同桶）→ 拉链法：桶内维护链表
扩容：元素数 > buckets的75% → 扩容到最近质数 → 重新哈希
```

---

# 17 泛型

## 泛型 vs ArrayList——为什么需要泛型

```csharp
// 问题1：无类型安全
ArrayList list = new ArrayList();
list.Add(100);
list.Add("Hello");
int x = (int)list[1];  // 运行时 InvalidCastException！

// 问题2：值类型装箱
list.Add(42);  // int → object 装箱！GC压力！

// 泛型解决
List<int> list = new List<int>();
list.Add(42);  // 内部 int[]，无装箱，编译时类型安全
```

## C# 泛型 vs C++ 模板

| | C++ Template | C# Generic |
|---|---|---|
| **负责** | 编译器 | CLR + JIT |
| **实例化** | 编译期，每种类型生成新代码 | 运行期，泛型信息保留 |
| **引用类型** | 生成独立代码 | 通常共享同一份机器码 |
| **值类型** | 生成独立代码 | 单独 JIT 生成 |
| **本质** | 编译期代码生成 | 运行时类型系统的一部分 |

---

# 18 反射

**反射 = 运行时获取类型信息并动态操作。** 通过 `System.Reflection` 命名空间。

```csharp
Assembly asm = Assembly.LoadFrom("MyLib.dll");
Type type = asm.GetType("MyNamespace.MyClass");
object obj = Activator.CreateInstance(type);
MethodInfo method = type.GetMethod("MyMethod");
method.Invoke(obj, new object[] { "参数" });
```

## 核心类

| 类 | 作用 |
|----|------|
| Assembly | 程序集入口 |
| Type | 类型信息（typeof/GetType） |
| MethodInfo | 方法信息，动态调用 |
| PropertyInfo | 属性信息 |
| FieldInfo | 字段信息 |

## 反射性能优化

1. 缓存 Type/MethodInfo（避免重复解析）
2. 用 `Delegate.CreateDelegate` 转强类型委托
3. 表达式树编译
4. 非热路径使用

## 应用场景

- 插件系统（动态加载 DLL）
- 依赖注入框架
- ORM 映射（属性→数据库字段）
- Unity Inspector 序列化

---

# 19 异常

## 基本结构

```csharp
try {
    // 可能抛异常的代码
} catch (SpecificException ex) {
    // 特定异常处理
} catch (Exception ex) {
    // 兜底
} finally {
    // 必定执行（清理资源）
}
```

## 经典面试题

### throw vs throw ex

```csharp
catch (Exception ex) {
    throw;       // ✅ 保留原始堆栈跟踪
    throw ex;    // ❌ 重置堆栈，原始位置丢失！
}
```

### when 异常过滤器（C# 6+）

```csharp
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.NotFound) {
    // 只捕获 404
}
```

### AggregateException（并行异常）

```csharp
try { Task.WaitAll(tasks); }
catch (AggregateException aggEx) {
    foreach(var ex in aggEx.Flatten().InnerExceptions) { ... }
}
```

> **try-catch 本身几乎零开销，贵的是抛出异常（堆栈展开）。** 用 `TryParse` 等预检查替代异常。

---

# 20 异步编程

## async/await 核心原理

```
同步：主线程等待 → 阻塞
异步：耗时操作丢给线程池 → 主线程继续 → 完成后回调
```

```csharp
// async/await 状态机
public async Task<GameObject> LoadAsync(string path) {
    var req = Resources.LoadAsync<GameObject>(path);
    await req;  // 编译器生成状态机，await 处"暂停"，完成后"恢复"
    return req.asset as GameObject;
}
```

## ConfigureAwait(false)

```csharp
await SomeAsync().ConfigureAwait(false);
// 不回到原 SynchronizationContext，在任意线程继续
// 库代码应使用，避免死锁
```

## async void 的危险

```csharp
async void Bad() { await Task.Delay(100); throw new Exception(); }
// ❌ 异常无法捕获！调用方不知道何时完成！无法 await！
// ✅ 除事件处理器外，始终返回 Task
```

## 死锁经典场景

```csharp
// ❌ 死锁！UI线程等.Result，await完成后又要回UI线程
var result = DownloadAsync().Result;

// ✅ 一直 await 到底
var result = await DownloadAsync();
```

## TAP 命名规范

- 返回 Task 的方法以 `Async` 结尾
- 接受 `CancellationToken` 作为最后参数
- 返回 `Task`/`Task<T>`/`ValueTask<T>`

---

# 21 内存分布与管理

## C# 内存六大区

```
高地址  ┌──────────────┐
       │     栈        │ ← 值类型数据、引用地址。编译器自动管理，不归GC管
       │   Stack       │   函数结束自动弹出释放。快，容量小
       ├──────────────┤
       │   托管堆      │ ← 引用类型对象。CLR分配，GC回收
       │ Managed Heap  │   连续分配（移动指针），比malloc快
       ├──────────────┤
       │  非托管堆     │ ← C++/malloc 等非托管代码分配，需手动释放
       ├──────────────┤
       │  静态数据区    │ ← static变量、全局数据。程序整个生命周期
       ├──────────────┤
       │  常量数据区    │ ← 字符串常量等。编译时确定，只读
       ├──────────────┤
       │  程序代码区    │ ← 编译后的IL/机器码
低地址  └──────────────┘
```

## 栈 vs 托管堆

| | 栈 | 托管堆 |
|---|---|---|
| **存什么** | 值类型、引用地址 | 引用类型对象 |
| **管理** | 编译器自动（出作用域就释放） | GC 自动（时机不确定） |
| **速度** | 极快（移动栈指针） | 快（移动next指针），GC有开销 |
| **容量** | 小（几MB） | 大 |
| **GC** | 不归GC管 | GC回收+压缩 |

## 托管 vs 非托管资源

| | 托管资源 | 非托管资源 |
|---|---|---|
| **管理** | GC 自动 | 程序员手动 |
| **例子** | 普通对象、数组 | 文件句柄、数据库连接、网络连接 |
| **释放** | 自动 | `IDisposable.Dispose()` / 终结器 |
| **using** | - | `using var fs = File.Open(...)` 自动调Dispose |

---

# 22 垃圾回收 GC

## GC 是什么

CLR 的自动内存管理器。负责：分配托管堆上的对象 → 标记可达对象 → 回收不可达对象 → 压缩碎片。

## 触发时机

- 托管堆内存不足（第0代阈值超限）
- 系统物理内存低
- 显式调用 `GC.Collect()`（不推荐）
- CLR 卸载 AppDomain

## GC 流程

```
1. 暂停所有线程（除触发GC的线程）
2. 标记阶段：从根(静态字段、栈变量、CPU寄存器)出发，标记所有可达对象
3. 回收阶段：不可达对象 → 回收内存
4. 压缩阶段：移动存活对象，消除碎片，更新引用地址
```

## 分代回收

```
第0代 ── 新对象，回收最频繁（如临时变量）
第1代 ── 经历一次GC仍存活。0代和1代之间的缓冲
第2代 ── 长期存活对象（如静态数据）
LOH  ── ≥85000字节的大型对象，逻辑上在第2代
```

**为什么分代？** 绝大多数对象短命→只回收第0代就够了→避免全堆扫描，大幅提升性能。

**回收频率：** 0代最频繁 → 1代 → 2代最少。只有前代不够才会回收后代。

## 三种GC算法

| 算法 | 步骤 | 特点 |
|------|------|------|
| 标记清除 Mark-Sweep | 标记→清除 | 不清除碎片 |
| 复制 Copying | 内存对半分，拷贝存活对象 | 天然无碎片，浪费一半内存 |
| 标记整理 Mark-Compact | 标记→清除→压缩 | 无碎片，.NET主流 |

## 大型对象堆 LOH

- 对象 ≥ 85,000 字节 → 进入 LOH
- LOH 在第2代GC时回收
- **默认不压缩**（移动大对象太贵）
- 导致 LOH 碎片 → 即使有足够内存也可能 OOM

## 强引用 vs 弱引用

| | 强引用 | 弱引用 |
|---|---|---|
| **阻止GC** | ✅ | ❌ |
| **默认** | 是 | 否 |
| **场景** | 核心对象 | 缓存、事件订阅 |
| **访问** | 直接 | `weakRef.TryGetTarget(out obj)` |

```csharp
// 弱引用缓存：内存紧张时GC自动回收，不用手动管理过期
var cache = new Dictionary<string, WeakReference<Bitmap>>();
```

## Finalize vs IDisposable

| | Finalize（析构函数） | IDisposable |
|---|---|---|
| **调用时机** | GC 回收时，不确定 | 手动 `Dispose()`，确定 |
| **语法** | `~MyClass() {}` | `void Dispose()` |
| **推荐** | 少用 | **优先使用** |
| **using** | - | `using var x = ...` 自动调Dispose |

```csharp
// ✅ 推荐：IDisposable + using
public class MyResource : IDisposable {
    public void Dispose() { /* 释放非托管资源 */ }
}
using var resource = new MyResource();  // 离开作用域自动Dispose
```

## Unity GC 注意事项

- 不要在 `Update` 中频繁 `new` 对象（每帧产生垃圾 → GC 卡顿）
- 使用对象池复用对象
- 用 `List<T>` 而非 `ArrayList`（避免装箱）
- 字符串拼接大量时用 `StringBuilder`
- 使用 `UniTask`（第三方，零GC分配）

---

> 📝 **参考资料：** .NET 官方文档、Unity 文档、《Unity 客户端面试宝典》
