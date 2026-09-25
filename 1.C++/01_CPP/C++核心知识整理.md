# C++ 核心知识整理

> 整合自 C++ 笔记，按 STL 容器与核心特性分类组织。

---

# 目录

- [01 Vector](#01-vector)
- [02 List](#02-list)
- [03 Deque](#03-deque)
- [04 Map](#04-map)
- [05 UnorderedMap](#05-unorderedmap)
- [06 String](#06-string)
- [07 SmartPointer](#07-smartpointer)
- [08 Template](#08-template)
- [09 面向对象三大特性](#09-面向对象三大特性)
- [10 VirtualFunction](#10-virtualfunction)
- [11 ObjectModel](#11-objectmodel)
- [12 内存基础](#12-内存基础)
- [13 指针详解](#13-指针详解)
- [14 const 关键字](#14-const-关键字)
- [15 类型转换](#15-类型转换)
- [16 sizeof 和 strlen](#16-sizeof-和-strlen)
- [17 static 关键字](#17-static-关键字)

---

# 01 Vector

> C++ 中最常用的容器，没有之一。面试中被问到的概率也是最高的。

## 01 为什么会有 Vector

在 C 语言中，数组是我们最常用的数据结构。但是数组有一个致命的缺陷：**大小必须在编译期确定**。

```cpp
int arr[10];  // 只能存 10 个元素，多了溢出，少了浪费
```

这就带来了几个问题：不知道实际运行时会有多少数据，只能按"最大可能"来分配浪费内存；一旦分配好大小就不能变了；需要手动 malloc/free，容易出错。

于是 C++ 提供了一个"**能自动扩容的数组**"—— `std::vector`。

## 02 Vector 解决了什么问题

一句话总结：**vector = 动态数组，自动管理内存，大小可在运行时改变。**

| 问题 | C 数组 | Vector |
|------|--------|--------|
| 大小 | 编译期固定 | 运行时动态增长 |
| 内存管理 | 手动 malloc/free | 自动管理（RAII） |
| 边界安全 | 无检查，容易越界 | `at()` 有边界检查 |
| 元素访问 | `arr[i]` | `vec[i]`（连续内存，O(1)） |
| 尾部操作 | 不支持 | `push_back` / `pop_back` O(1) 均摊 |

## 03 底层实现（三个指针）

Vector 的底层实现非常简单，就靠**三个指针**：

```cpp
template<typename T>
class vector {
private:
    T* _start;            // 指向第一个元素
    T* _finish;           // 指向最后一个元素的下一个位置
    T* _end_of_storage;   // 指向已分配内存的末尾
};
```

通过这三个指针，各种操作极其简单：

```cpp
size_t size()     const { return _finish - _start; }        // 元素个数
size_t capacity() const { return _end_of_storage - _start; } // 容量
bool   empty()    const { return _start == _finish; }        // 是否为空
```

## 04 内存布局

Vector 保证**连续内存**，这是它最大的优势。

### 关键：两层内存——对象在栈，数据在堆

```
                    栈区                          堆区
             ┌──────────────┐          ┌──────────────────────┐
vec 对象:    │   _start ●────┼─────────→│ [1][2][3][4][5]...   │ ← 连续数组
             │   _finish ●───┼────┐     └──────────────────────┘
             │   _end ●──────┼──┐ │
             └──────────────┘  │ │
                sizeof=24字节   │ │
              (三个指针×8)      │ │
                               └─┼── finish 指向最后一个元素的下一个位置
                                 └── end_of_storage 指向已分配内存末尾
```

| 存储位置 | 内容 | 生命周期 |
|---------|------|---------|
| **栈** | vector 对象本身（三个指针，24字节） | 离开作用域自动销毁 |
| **堆** | 实际元素数组 | 随 vector 析构函数释放 |

```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};

// vec 对象在栈上(三个指针24字节)，数据 [1][2][3][4][5] 在堆上
// sizeof(vec) = 24（三个指针），不是元素大小之和

// sizeof 对比
int arr[10];
std::vector<int> vec2(10);
sizeof(arr);    // 40（10个int × 4，数组本身在栈）
sizeof(vec2);   // 24（三个指针，对象在栈，数据在堆）
```

> **⚠️ 常见误区：** `sizeof(vec)` 返回的是 vector 对象本身的大小（三个指针），不是元素的总大小。要获取元素总大小用 `vec.size() * sizeof(T)`。

## 05 push_back()

在尾部添加元素。空间不够时触发扩容。

```cpp
void push_back(const T& value) {
    if (_finish == _end_of_storage) {
        _grow();  // _grow内部：new 新堆内存 → 拷贝 → delete 旧堆内存
    }
    new (_finish) T(value);  // placement new（在已分配的堆内存上构造）
    ++_finish;
}
```

**扩容 = 在堆上换一块更大的地：**
1. 在**堆**上 `new` 分配一块更大的新内存
2. 把旧堆内存上的元素拷贝/移动到新堆内存
3. `delete` 释放旧堆内存
4. 更新三个指针指向新堆内存

**均摊时间复杂度：O(1)**。

**左值 vs 右值版本：**

```cpp
void push_back(const T& value);   // 左值：拷贝
void push_back(T&& value);        // 右值：移动

vec.push_back(x);             // x 是左值 → 拷贝
vec.push_back(42);            // 42 是右值 → 移动
vec.push_back(std::move(x));  // 显式移动
```

## 06 reserve()

`reserve(n)` 预分配容量，但**不改变 size**，不构造元素。

```cpp
std::vector<int> vec;
vec.reserve(1000);   // 预分配 1000 个元素的空间
// size 仍为 0，capacity >= 1000

for (int i = 0; i < 1000; i++) {
    vec.push_back(i);  // 这 1000 次不会触发任何扩容！
}
```

**什么时候用 reserve？**
1. 知道大概要存多少元素：避免多次扩容拷贝
2. 需要指针/迭代器稳定性：提前 reserve 防止扩容导致失效

**常见误区：**
```cpp
vec.reserve(10);
vec[5] = 42;  // 未定义行为！size 还是 0，元素没构造！
```

## 07 resize()

`resize(n)` 改变 size，可能需要扩容或析构元素。

```cpp
std::vector<int> vec = {1, 2, 3};  // size=3

vec.resize(5);      // size=5, 新增元素用默认值填充 → {1,2,3,0,0}
vec.resize(5, 42);  // size=5, 新增元素用 42 填充 → {1,2,3,42,42}
vec.resize(2);      // size=2, 尾部元素被销毁 → {1,2}
```

| 操作 | size | capacity | 元素是否构造 |
|------|------|----------|------------|
| `reserve(n)` | 不变 | ≥n | 不构造 |
| `resize(n)` | 变为 n | 可能扩容 | 构造/析构 |

## 08 emplace_back()

C++11 引入，**直接在 vector 内部原地构造对象**，避免临时对象的创建和拷贝。

```cpp
struct Person {
    std::string name;
    int age;
    Person(const std::string& n, int a) : name(n), age(a) {
        std::cout << "构造: " << name << std::endl;
    }
    Person(const Person& other) : name(other.name), age(other.age) {
        std::cout << "拷贝: " << name << std::endl;
    }
};

std::vector<Person> vec;

// push_back：先构造临时对象，再拷贝到 vector → 两次构造
vec.push_back(Person("Alice", 25));  // 输出：构造 → 拷贝

// emplace_back：直接在 vector 内存中构造 → 一次构造
vec.emplace_back("Bob", 30);         // 输出：构造
```

**实现（使用完美转发）：**

```cpp
template<typename... Args>
void emplace_back(Args&&... args) {
    if (_finish == _end_of_storage) { _grow(); }
    new (_finish) T(std::forward<Args>(args)...);
    ++_finish;
}
```

## 09 erase()

删除指定位置的元素。**O(N)**（后面的元素要向前移动）。

```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};

vec.erase(vec.begin() + 2);  // {1, 2, 4, 5}
vec.erase(vec.begin() + 1, vec.begin() + 3);  // 删除范围
```

**经典陷阱——遍历删除：**

```cpp
// 错误写法！
for (auto it = vec.begin(); it != vec.end(); ++it) {
    if (*it % 2 == 0) vec.erase(it);  // it 失效！
}

// 正确写法1：使用 erase 返回的新迭代器
for (auto it = vec.begin(); it != vec.end(); ) {
    if (*it % 2 == 0) it = vec.erase(it);
    else ++it;
}

// 正确写法2：erase-remove 惯用法
vec.erase(std::remove_if(vec.begin(), vec.end(),
    [](int x) { return x % 2 == 0; }), vec.end());

// 正确写法3：C++20
std::erase_if(vec, [](int x) { return x % 2 == 0; });
```

## 10 iterator

Vector 的迭代器本质上就是**指针**（在大多数实现中）：

```cpp
template<typename T>
class vector {
public:
    using iterator        = T*;         // 迭代器就是裸指针
    using const_iterator  = const T*;

    iterator begin() { return _start; }
    iterator end()   { return _finish; }
};
```

**为什么用指针就行？** 因为元素是连续存储的，指针天然满足：`++` 移到下一个元素、`*` 解引用、`==` 比较。

这里体现了 **using 的设计哲学**：

- **对内**：`iterator` 就是 `T*`（实现细节）
- **对外**：使用者用 `vector<int>::iterator`，不关心底层
- **好处**：如果底层换了（比如换了分配器），外部代码不用改

这是 C++ 中 using 的三个维度：

| 维度 | 作用 |
|------|------|
| 语法 | 更现代的 typedef：`using iterator = T*;` |
| 设计 | 隐藏底层实现：外部用 `vector::iterator` 而不是 `T*` |
| 架构 | 标准化接口契约：STL 要求所有容器定义 `value_type`、`iterator` 等 |

[整合自 Vector/using 的终极总结.md]

## 11 为什么 iterator 会失效

### 扩容导致失效

```cpp
std::vector<int> vec = {1, 2, 3};
auto it = vec.begin();
vec.push_back(4);  // 触发扩容 → it 变成悬空指针！
// 原因：扩容过程是 分配新内存 → 拷贝 → 释放旧内存
```

### 插入/删除导致失效

```cpp
// 中间插入 → 插入点之后所有元素后移 → 迭代器失效
vec.insert(vec.begin(), 0);  // 在开头插入，所有迭代器失效

// 删除 → 删除点之后所有元素前移 → 迭代器失效
vec.erase(vec.begin());  // 之后所有迭代器失效
```

### 总结

| 操作 | 失效范围 |
|------|---------|
| `push_back`（触发扩容） | 全部 |
| `insert`（触发扩容） | 全部 |
| `insert`（不扩容） | 插入点之后 |
| `erase` | 删除点及之后 |
| `pop_back` | 仅尾后迭代器 |
| `clear` | 全部 |
| `reserve`（触发扩容） | 全部 |

**防止方法：** 提前 `reserve`；使用 `erase` 返回的迭代器；能用索引时用索引。

## 12 Cache Friendly

Vector 是**对 CPU 缓存最友好的数据结构**。

```
CPU 按缓存行（Cache Line，64字节）读取：
  读取 vec[0] → 同时把 vec[1], vec[2]... 加载进缓存
  紧接着访问 vec[1] → 已在缓存中 → Cache Hit！
```

**遍历性能对比：**

```cpp
// vector：连续内存 → 每次访问大概率 Cache Hit
std::vector<int> vec(1000000);
for (int& x : vec) x *= 2;

// list：节点分散 → 每次访问大概率 Cache Miss
std::list<int> lst(1000000);
for (int& x : lst) x *= 2;
// vector 版本通常比 list 快 10-100 倍！
```

**预取 + 分支预测：**

```
Vector 遍历：    [1][2][3][4][5][6][7][8]...  CPU 硬件预取器完美预取
List 遍历：      [1]→[2]→[3]→[4]→...          每次跳转不可预测
```

[整合自 Vector/C++性能优化——likely和unlikely分支预测.md]

C++20 之前用编译器内置函数做分支预测提示：

```cpp
#define likely(x)   __builtin_expect(!!(x), 1)
#define unlikely(x) __builtin_expect(!!(x), 0)
```

C++20 成为正式关键字：

```cpp
if (likely(vec.size() > 0)) { /* 大概率走这里 */ }
switch(i) {
    [[likely]] case 1: handle1(); break;
    [[unlikely]] case 99: handle_error(); break;
}
```

## 13 STL 源码

### 简化实现

```cpp
template <class T, class Alloc = std::allocator<T>>
class vector {
public:
    using value_type      = T;
    using iterator        = T*;
    using const_iterator  = const T*;
    using reference       = T&;
    using size_type       = size_t;

protected:
    iterator start;           // 第一个元素
    iterator finish;          // 尾后
    iterator end_of_storage;  // 容量末尾

public:
    vector() : start(nullptr), finish(nullptr), end_of_storage(nullptr) {}

    explicit vector(size_type n) {
        start = allocate(n);
        finish = start;
        end_of_storage = start + n;
        for (; finish != end_of_storage; ++finish) construct(finish);
    }

    ~vector() {
        destroy(start, finish);
        deallocate(start, end_of_storage - start);
    }

    iterator begin() { return start; }
    iterator end()   { return finish; }
    size_type size() const { return finish - start; }
    size_type capacity() const { return end_of_storage - start; }
    reference operator[](size_type n) { return *(start + n); }

    void push_back(const T& value) {
        if (finish == end_of_storage) _grow();
        construct(finish, value);
        ++finish;
    }

    void pop_back() { --finish; destroy(finish); }

    iterator erase(iterator pos) {
        if (pos + 1 != finish)
            std::copy(pos + 1, finish, pos);
        --finish;
        destroy(finish);
        return pos;
    }

protected:
    void _grow() {
        size_type old_size = size();
        size_type new_cap = old_size ? old_size * 2 : 1;  // 2倍扩容
        iterator new_start = allocate(new_cap);
        iterator new_finish = new_start;
        for (iterator it = start; it != finish; ++it, ++new_finish)
            construct(new_finish, std::move_if_noexcept(*it));
        destroy(start, finish);
        deallocate(start, end_of_storage - start);
        start = new_start;
        finish = new_finish;
        end_of_storage = new_start + new_cap;
    }
};
```

### 扩容因子

- **MSVC / GCC / Clang**：都使用 **2 倍**扩容
- 理论上 1.5 倍有个优势：可以重用之前释放的内存（新旧内存可以拼接），但 2 倍实现简单且扩容次数最少

### std::initializer_list

[整合自 Vector/C++ stdinitializer_list 详解.md]

`std::initializer_list<T>` 是 C++11 引入，用于支持 `{}` 列表初始化。内部只有两个成员：指向临时数组的指针 + 大小。

**关键特性：** 只读（元素是 const）、浅拷贝（复制不复制元素）、生命周期有限（临时数组）。

```cpp
// vector 的 initializer_list 构造函数
vector(std::initializer_list<T> init) { /* 逐元素拷贝 */ }

std::vector<int> vec = {1, 2, 3, 4, 5};  // 调用 initializer_list 构造
```

### STL 内存分配器（allocator）

[整合自 深入剖析STL内存分配器allocator及其萃取器.md]

allocator 将 new/delete 的两步合一开始拆分：

```
new    = allocate() + construct()    （分配 + 构造）
delete = destroy()  + deallocate()   （析构 + 释放）
```

**为什么要分离？** 容器可以预分配大块内存（reserve），稍后逐个构造；避免不必要的构造/析构开销。

**rebind 机制：** `std::list<int>` 传入 `allocator<int>`，但内部需要分配节点 `_List_node<int>`。rebind 就是把 `allocator<int>` 变成 `allocator<_List_node<int>>` 的机制。

```
allocator 四步走：分配 → 构造 → 析构 → 释放
rebind 换模具：同一个分配器，换一种类型
traits 查户口：编译期检查是否存在某个成员（SFINAE）
```

## 14 UE TArray

UE 的 `TArray` 是 `std::vector` 的游戏引擎版本：

| 特性 | std::vector | UE TArray |
|------|------------|-----------|
| 扩容策略 | 2x | 小数组2x，大数组~1.25x |
| 内存分配 | std::allocator | FMemory（UE自定义） |
| 序列化 | 不支持 | 内置（FArchive） |
| 网络复制 | 不支持 | 内置 Replication |
| GC | 无关 | 配合 UE GC 系统 |

```cpp
// UE TArray 常用API
TArray<int32> Arr;
Arr.Add(42);           // push_back
Arr.Emplace(42);       // emplace_back
Arr.Pop();             // pop_back
Arr.RemoveAt(0);       // erase
Arr.Empty();           // clear
Arr.Reserve(100);      // reserve
```

## 15 Unity NativeArray

Unity DOTS 中的 `NativeArray<T>`：

| 特性 | std::vector | Unity NativeArray |
|------|------------|-------------------|
| 扩容 | 自动扩容 | **固定大小**，不支持扩容 |
| 内存 | 托管堆 | 非托管内存（手动释放） |
| 线程安全 | 不安全 | Safety Handle 机制 |
| Job System | 无关 | 内置配合 |

```csharp
NativeArray<int> array = new NativeArray<int>(100, Allocator.TempJob);
array.Dispose();  // 必须手动释放！
```

为什么不支持扩容？因为 Unity DOTS 追求极致性能和确定性，固定大小保证内存布局确定，配合 Burst Compiler 优化。

如果需要类似 vector 的动态扩容，Unity 提供了 `NativeList<T>`。

## 16 面试题

**Q1: push_back 时间复杂度？** → 均摊 O(1)。

**Q2: 为什么 2 倍扩容？** → 扩容次数和内存利用率的平衡，2倍扩容次数最少。

**Q3: reserve vs resize？** → reserve 只改 capacity；resize 改 size，会构造/析构元素。

**Q4: emplace_back vs push_back？** → emplace_back 原地构造，省临时对象；push_back 先构造再拷贝/移动。

**Q5: 遍历删除的正确写法？** → `it = vec.erase(it)` 或 erase-remove 或 C++20 `std::erase_if`。

**Q6: 什么时候迭代器失效？** → 扩容（全部失效）、插入（插入点之后）、删除（删除点及之后）。

**Q7: vector\<bool\> 有什么问题？** → 特化为 bit 存储，返回代理对象而不是 `bool&`，不能取地址，不符合容器语义。替代：`vector<char>` 或 `deque<bool>`。

**Q8: 如何减少内存占用？** → `shrink_to_fit()` 或 swap 技巧。

**Q9: 中间插入的开销？** → O(N)，所有后续元素需要后移。

**Q10: 为什么遍历比 list 快？** → Cache 友好 + 预取 + 无指针跳转，实测快 10-100 倍。

---

# 02 List

> 双向链表。与 vector 形成鲜明对比：插入删除 O(1)，遍历 O(N) 且 Cache 不友好。

## 为什么会有 List

Vector 虽然好用，但在**中间插入/删除**时很慢——所有后续元素都要移动（O(N)）。List（`std::list`）用链表的方式解决了这个问题。

## 双向链表

C++ `std::list` 是**双向循环链表**。每个节点在**堆**上独立分配。

```cpp
template<typename T>
struct _List_node {
    _List_node* _M_next;  // 指向下一个节点
    _List_node* _M_prev;  // 指向前一个节点
    T           _M_data;  // 存储的数据
};
// 每个节点通过 new 在堆上分配，彼此内存不连续
```

### 存储布局

```
        栈区                       堆区（节点分散）
   ┌─────────────┐
   │ list 对象    │         ┌──────┐     ┌──────┐     ┌──────┐
   │  sentinel ●──┼────────→│ prev │←───│ prev │←───│ prev │
   │  (哨兵节点)   │         │ next │───→│ next │───→│ next │──→ sentinel (循环)
   │  size=3     │         │ data │    │ data │    │ data │
   └─────────────┘         └──────┘    └──────┘    └──────┘
   sizeof(list)=24~32字节      ↑            ↑            ↑
   对象在栈                    每次new分配一个节点(堆)，彼此地址不连续
```

| 存储位置 | 内容 | 生命周期 |
|---------|------|---------|
| **栈** | list 对象本身（哨兵节点+size，约 24~32 字节） | 离开作用域自动销毁 |
| **堆** | 每个链表节点（独立 new） | list 析构时逐个 delete

## push_front / push_back

```cpp
std::list<int> lst;

lst.push_back(1);   // 尾部插入
lst.push_back(2);
lst.push_front(0);  // 头部插入
// lst = {0, 1, 2}
```

两者的时间复杂度都是 **O(1)**——只需要修改相邻节点的指针。

## erase

```cpp
std::list<int> lst = {1, 2, 3, 4, 5};
auto it = std::find(lst.begin(), lst.end(), 3);
lst.erase(it);  // O(1)！只修改前后节点的指针
// lst = {1, 2, 4, 5}
```

与 vector 的关键区别：**list 的 erase 是 O(1)**，因为不需要移动其他元素。只需修改被删除节点前后节点的 `next`/`prev` 指针。

## iterator

List 的迭代器不是指针，而是一个**包装了节点的类**：

```cpp
template<typename T>
struct _List_iterator {
    _List_node<T>* _M_node;  // 指向链表节点

    T& operator*()  const { return _M_node->_M_data; }
    T* operator->() const { return &_M_node->_M_data; }

    // ++it：移动到下一个节点
    _List_iterator& operator++() {
        _M_node = _M_node->_M_next;
        return *this;
    }
    // --it：移动到上一个节点
    _List_iterator& operator--() {
        _M_node = _M_node->_M_prev;
        return *this;
    }
};
```

## 为什么不会失效

这是 list 相比 vector 最大的优势：**除了被删除的元素，其他迭代器都不会失效**。

```
删除 B 前：
    [A] ⇄ [B] ⇄ [C] ⇄ [D]
     ↑     ↑     ↑     ↑
    itA   itB   itC   itD

删除 B 后：
    [A] ⇄ [C] ⇄ [D]
     ↑     ↑     ↑
    itA   itC   itD     ← itA, itC, itD 仍然有效！
```

原因很简单：链表节点是独立分配的，删除一个节点不影响其他节点的地址。而 vector 删除元素后，后面的元素全部向前移动，导致迭代器指向了错误的位置。

## Cache Miss

List 最大的性能问题——**每个节点在内存中分散存储**：

```
Vector：    [1][2][3][4][5][6][7][8]  ← 连续，Cache 友好
List：      [1]→[2]→[3]→[4]→...       ← 分散，每次访问都可能 Cache Miss
              ↓    ↓    ↓    ↓
            地址各不相同，CPU 预取器无法预测
```

遍历 100 万个元素的 list，可能比遍历同样大小的 vector **慢 10-100 倍**。

## 侵入式链表（Intrusive List）

### 先看 std::list 的问题

std::list 是这样存储数据的：

```cpp
std::list<int> lst = {10, 20};
```

```
内存布局：
             堆上分配的节点                        你的数据
    +----------------------------+         +------+
    | prev | next | data_ptr ●---+--------→|  10  |
    +----------------------------+         +------+
    | prev | next | data_ptr ●---+--------→|  20  |
    +----------------------------+         +------+

每次插入 = new 一个节点（含指针 + 指针）+ 你的数据本身
→ 两次堆分配！而且数据和节点分离，Cache 不友好
```

### 侵入式链表的思路

**反过来：把链表的"钩子"（prev/next 指针）直接塞进你的数据结构里。**

```c
// 先定义一个通用的"钩子"——只有 prev 和 next 两个指针
struct list_head {
    struct list_head *next, *prev;
};

// 你的数据里嵌入这个钩子
struct MyData {
    int id;
    char name[32];
    struct list_head node;  // ← "钩子"直接长在你的数据里面
};
```

### 对比两种方式

```
std::list:                              侵入式:
                                         
  节点(独立分配)      你的数据             你的数据（里面自带钩子）
  ┌──────────┐      ┌──────┐            ┌──────────────────┐
  │prev,next │      │      │            │ id = 1           │
  │data_ptr ●──────→│ data │            │ name = "Alice"   │
  └──────────┘      └──────┘            │ list_head {      │
     ↑ 每次插入                              │   next ●──┐    │
     └── new 一次 ──→ 两次分配！            │   prev    │    │
                                         └───────────┼────┘
                                                      │
你的数据 和 链表的"钩子"                              │
    是分开的两块内存                                 │
                                         ┌───────────┼────┐
                                         │ id = 2           │
                                         │ name = "Bob"     │
                                         │ list_head {      │
                                         │   next ●──┐      │
                                         │   prev ●──┘      │
                                         └───────────┼──────┘
                                                      ↓
你的数据 和 链表的"钩子"
    是同一块内存 → 零额外分配！
```

### 最关键的问题：怎么从"钩子"找到你的数据？

假设我拿到了一个 `list_head*` 指针，怎么得到包含它的 `MyData`？

```
MyData 对象的内存：
┌──────────────────────────┐ 低地址
│        id = 1            │  ← MyData 的起始地址
│        name = "Alice"    │
│  list_head:              │
│     next ●─────────┐     │
│     prev ●──┐      │     │
└──────────────┼──────┼────┘
               │      │
               │      │
如果我知道 list_head 在 MyData 里的偏移量是 40 字节，
那么：MyData* ptr = (MyData*)((char*)list_head_ptr - 40);

这 40 就是 container_of 宏算出来的！
```

这就是 Linux 内核的 `container_of` 宏做的事：

```c
// 简化版 container_of
#define container_of(ptr, type, member) \
    ((type*)((char*)(ptr) - offsetof(type, member)))

// 用法：
struct list_head *pos = ...;  // 我手里只有一个钩子
struct MyData *data = container_of(pos, struct MyData, node);
//  data 现在指向完整的 MyData 结构体！
```

### 完整示例：用侵入式链表遍历

```c
// 假设已有一个链表头 head，链了几个 MyData

struct list_head *pos;
struct MyData *entry;

// 遍历链表
list_for_each(pos, &head) {
    // pos 只是 list_head*，怎么拿数据？
    entry = container_of(pos, struct MyData, node);
    printf("id=%d, name=%s\n", entry->id, entry->name);
}
```

### 一句话总结

| | std::list（非侵入式）| 侵入式链表 |
|---|---|---|
| 节点和数据 | **分开**两块内存 | **同一块**内存 |
| 每次插入 | 分配节点 + 存数据 | 数据已经有了，零额外分配 |
| container_of | 不需要 | **必须**（从钩子反推数据地址） |
| 一个对象 | 可在多个 list 中 | 每个 list_head 只能在一个链表中 |

**侵入式 = 钩子长在数据身上，数据自带"被链接"的能力。**


## UE TLinkedList

UE 提供了两种链表实现：

```cpp
// 1. TLinkedList：侵入式（类似 Linux）
//    节点继承自 TLinkedList<T>
class UMyObject : public TLinkedList<UMyObject> { ... };

// 2. TDoubleLinkedList：独立节点式（类似 std::list）
TDoubleLinkedList<int32> List;
List.AddHead(1);
List.AddTail(2);
```

UE 倾向于使用侵入式链表，因为它避免了额外的堆分配，对游戏性能更友好。

## LRU（Least Recently Used）缓存

List 的经典应用场景——LRU 缓存：

```cpp
class LRUCache {
    int _capacity;
    std::list<std::pair<int, int>> _list;  // (key, value)
    std::unordered_map<int, std::list<std::pair<int,int>>::iterator> _map;

    int get(int key) {
        auto it = _map.find(key);
        if (it == _map.end()) return -1;
        // 移到链表头部（最近使用）
        _list.splice(_list.begin(), _list, it->second);
        return it->second->second;
    }

    void put(int key, int value) {
        if (_map.count(key)) {
            _list.erase(_map[key]);
        } else if (_list.size() >= _capacity) {
            // 淘汰最久未使用（链表尾部）
            _map.erase(_list.back().first);
            _list.pop_back();
        }
        _list.push_front({key, value});
        _map[key] = _list.begin();
    }
};
```

LRU 之所以用 list 而不用 vector，是因为需要频繁地把元素移到头部——list 的 `splice` 是 O(1)，vector 需要 O(N) 移动。

## 为什么游戏很少使用

游戏引擎（如 UE）很少直接使用 `std::list`，原因：

1. **Cache 不友好**：游戏每帧遍历大量对象，list 的 Cache Miss 代价太高
2. **内存碎片**：每个节点独立分配，导致大量小内存分配
3. **内存开销**：每个节点有 2 个指针（64位下 16 字节），对小对象来说开销太大
4. **分配开销**：每次插入都触发一次堆分配（`new`）

替代方案：
- 大多数场景用 **TArray**（vector）
- 需要频繁中间插入删除时，考虑用**索引数组 + 空闲链表**（对象池模式）
- 少量场景用侵入式链表（`TLinkedList`）

---

# 03 Deque

> 双端队列。头尾插入都 O(1)，但内存不是完全连续的。

## 为什么不是连续内存

如果 deque 像 vector 一样用一整块连续内存，那么在头部插入就需要把所有元素后移——O(N)。为了支持头尾都 O(1) 插入，deque 采用了一种分段连续的设计。

## map + block（中控器 + 缓冲区）

Deque 的核心设计：**一个指针数组（map）+ 多个等大小的数据块（buffer）**。全部在**堆**上。

```
            栈区                      堆区
       ┌──────────┐
       │deque对象  │         ┌─────┬─────┬─────┬─────┐
       │ map指针 ●─┼────────→│ B0  │ B1  │ B2  │ B3  │ ← map（指针数组，堆）
       │ start    │         └──┼───┴──┼───┴──┼───┴──┼───┘
       │ finish   │            │      │      │      │
       └──────────┘            v      v      v      v
                          ┌──────┐┌──────┐┌──────┐┌──────┐
                          │buffer││buffer││buffer││buffer│ ← 数据块（堆）
                          │ [0]  ││ [0]  ││ [0]  ││ [0]  │
                          │ [1]  ││ [1]  ││ [1]  ││ [1]  │
                          └──────┘└──────┘└──────┘└──────┘
```

| 存储位置 | 内容 | 生命周期 |
|---------|------|---------|
| **栈** | deque 对象本身（map指针 + 两个迭代器，约 80 字节） | 离开作用域自动销毁 |
| **堆** | map 指针数组 + 所有 buffer 数据块 | deque 析构时释放 |

GCC 实现中，每个 buffer 大小通常为 `max(512 / sizeof(T), 1)` 个元素。

```cpp
template<typename T>
class deque {
protected:
    using map_pointer = T**;
    map_pointer _M_map;        // 中控器（指针数组）
    size_t      _M_map_size;   // map 的大小
    iterator    _M_start;      // 第一个元素的位置
    iterator    _M_finish;     // 最后一个元素的下一个位置
};

// deque 的迭代器需要4个成员
template<typename T>
struct _Deque_iterator {
    T*    _M_cur;     // 当前 buffer 中的当前位置
    T*    _M_first;   // 当前 buffer 的起始
    T*    _M_last;    // 当前 buffer 的末尾
    T**   _M_node;    // 指向 map 中当前 buffer 的指针
};
```

## push_back / push_front

```cpp
// push_back：在尾部 buffer 追加
// 如果尾部 buffer 满了 → 在 map 末尾新增一个 buffer
void push_back(const T& value) {
    if (_M_finish._M_cur + 1 == _M_finish._M_last) {
        // 当前 buffer 满了，分配新 buffer
        _M_reserve_map_at_back();
        *(_M_finish._M_node + 1) = _M_allocate_node();
    }
    construct(_M_finish._M_cur, value);
    ++_M_finish;
}

// push_front：在头部 buffer 前插入
// 如果头部 buffer 满了 → 在 map 开头新增一个 buffer
```

两端插入都是 **O(1)**。

## iterator

Deque 的迭代器是**随机访问迭代器**（支持 `it + n`），但 `+` 操作比 vector 复杂：

```cpp
// 典型的 operator+=
_Deque_iterator& operator+=(difference_type n) {
    difference_type offset = n + (_M_cur - _M_first);
    if (offset >= 0 && offset < buffer_size()) {
        _M_cur += n;  // 在同一个 buffer 内
    } else {
        // 跨 buffer：计算跳到 map 中第几个 buffer
        difference_type node_offset = offset / buffer_size();
        _M_node += node_offset;
        _M_first = *_M_node;
        _M_last = _M_first + buffer_size();
        _M_cur = _M_first + (offset % buffer_size());
    }
    return *this;
}
```

因为需要跨 buffer 计算，deque 的随机访问比 vector 稍慢。

## 扩容

Deque 的扩容不是重新分配所有内存，而是**扩展 map**：

```
扩容前：
    map: [B0][B1][B2]           ← 3个buffer, map满了
扩容后：
    map: [  ][  ][B0][B1][B2][  ][  ]  ← 分配更大的map，拷贝指针，中间留空
```

这样**已有元素不需要移动**——deque 的扩容比 vector 更高效（不需要拷贝所有元素），这也是 deque 不提供 `reserve()` 的原因之一。

## 为什么 queue 默认用 deque

```cpp
template<typename T, typename Container = std::deque<T>>
class queue { ... };
```

`std::queue` 和 `std::stack` 默认都以 deque 为底层容器，而不是 vector。原因：

1. **vector 不支持 `pop_front`**：queue 需要从头部弹出，vector 没有 `pop_front`
2. **deque 头尾操作都是 O(1)**：符合 queue/stack 的需求
3. **扩容不拷贝元素**：deque 扩容只需分配新 buffer，vector 要拷贝所有元素
4. **内存利用率合理**：不像 list 每个元素有指针开销

---

# 04 Map

> 有序的 key-value 容器，底层是红黑树。查找、插入、删除都是 O(log N)。

## 存储位置

```
            栈区                       堆区（红黑树节点，new分配）
       ┌──────────┐
       │ map 对象  │           ┌─────────────────────────┐
       │  root ●───┼──────────→│     红黑树节点(根)       │
       │  size=4  │           │  parent/left/right ●    │
       └──────────┘           │  key-value 数据          │
    sizeof(map)≈48字节        └──────┬─────────┬────────┘
      对象在栈                       │         │
                              ┌──────┘    ┌───┴──────┐
                              v           v           v
                            节点(左)    节点(右)    ...所有节点都在堆
```

| 存储位置 | 内容 | 生命周期 |
|---------|------|---------|
| **栈** | map 对象本身（root指针+size等，约 48 字节） | 离开作用域自动销毁 |
| **堆** | 每个红黑树节点（通过 allocator 在堆上分配） | map 析构时递归释放所有节点 |

## 先别管红黑树——你什么时候会用到 map？

```cpp
// 场景1：按分数排名，自动排序
std::map<int, string> scores;
scores[95] = "Alice";
scores[87] = "Bob";
scores[92] = "Charlie";
// 遍历时自动从小到大：87→92→95
for (auto& [score, name] : scores) {
    cout << name << ": " << score << endl;
}
// Bob: 87
// Charlie: 92
// Alice: 95

// 场景2：范围查询——"分数在 80~90 之间的人"
auto lower = scores.lower_bound(80);  // 第一个 ≥80
auto upper = scores.upper_bound(90);  // 第一个  >90
for (auto it = lower; it != upper; ++it) { ... }
// 只遍历符合条件的，不用遍历全部！

// 场景3：检查是否存在
if (scores.count("Alice")) { ... }  // O(log N)，不用遍历
```

**map 的核心价值：保持 key 有序 → 支持范围查询和高效查找。**

unordered_map 更快（O(1)）但**无序**，无法做 `lower_bound` 这种操作。

---

## 从零开始：为什么需要红黑树？

### 第一步：最简单的想法——排序数组

```
        key:   [23]  [45]  [67]  [89]
        value: [Alice][Bob][Charlie][Dave]
        index:   0     1     2     3

查找 "67" → 二分查找 O(log N) ✓ 很快
插入 "50" → 所有后面的元素后移 O(N) ✗ 太慢！
```

数组：查找快，插入慢。

### 第二步：换用链表？

```
        [23]→[45]→[67]→[89]
        插入 "50"：改两个指针 O(1) ✓
        查找 "67"：必须从头遍历  O(N) ✗
```

链表：插入快，查找慢。

### 第三步：二叉搜索树（BST）——鱼和熊掌都想得

```
规则：左子树 < 根 < 右子树

              [67]
             /    \
          [45]    [89]
          /  \    /  \
        [23][50][72][95]

查找 50：67>50→左边, 45<50→右边, 找到！3步,O(log N) ✓
插入 50：比较3次就找到位置,改指针 O(log N) ✓
```

**BST 解决了数组和链表各自的缺点！**

### 但是……BST 有个致命弱点

```cpp
// 如果按顺序插入：1, 2, 3, 4, 5, 6
```

```
所谓的"树"退化成了链表：
       [1]
         \
         [2]
           \
           [3]
             \
             [4]
               \
               [5]
                 \
                 [6]

查找 6：要走 6 步！O(N)！之前说的 O(log N) 完全白搭了！
```

**BST 的形状取决于插入顺序。最坏情况：一条链，退化到 O(N)。**

### 第四步：给 BST 上"保险"——自平衡

怎么防止退化？给树加一些**规则**，一旦违反就**调整**。

调整的手段就两种：
1. **变色**：把节点从红变黑、黑变红
2. **旋转**：改变树的父子关系，但保持 BST 性质

这就是**红黑树**——一种会自己调整的 BST。它保证：

> **树不会太高——最坏情况下，高度也不超过 2×log(N)。N=100万时，高度最多约 40 层。**

---

## 红黑树：给树上个"保险"

### 先把 BST 当作一棵普通的树来看

```
一个节点长这样：
        ┌──────┐
        │ key  │
        │color │  ← 每个节点要么红 🔴 要么黑 ⚫
        ├──────┤
        │ left │────→ 左孩子（比它小）
        │ right│────→ 右孩子（比它大）
        │parent│────→ 父节点
        └──────┘
```

### 五条"家规"（性质）

```
① 每个节点要么红，要么黑。             —— 废话，但必须说

② 根节点是黑色。⚫                     —— 老大必须是黑的

③ 叶子节点（NIL/空节点）是黑色。⚫      —— 树的"地面"是黑的

④ 红的两个孩子必须是黑的。            —— 🔴不能生🔴，避免"红红"
   （换句话说：红色节点不能相邻）

⑤ 从任意节点出发，到所有后代的叶子，
    经过的黑色节点数量相同。           —— 每条路径的黑节点数一致
```

### 这五条规则怎么保证"树不太高"？

用图来看：

```
性质⑤：每条路径黑节点一样多  → 最短路径 = 全黑
性质④：红的下面必须是黑的    → 最长路径 = 红黑红黑交替

最短路径（全黑）：
    ⚫ → ⚫ → ⚫          3 个黑节点
    |    |    |
    ⚫   ⚫   ⚫

最长路径（红黑交替）：
    ⚫ → 🔴 → ⚫ → 🔴 → ⚫  也是 3 个黑节点，中间夹最多 2 个红
    |    |    |    |    |
    ⚫   🔴   ⚫   🔴   ⚫

结论：最长路径 ≤ 最短路径 × 2

一条全黑，一条红黑交替 → 最长不超过最短的 2 倍
→ 树不可能"一边高一边矮"得离谱
→ 高度 ≈ O(log N)
```

### 用具体的图感受一下

```
这是一棵合法的红黑树（7个节点）：

                    ⚫ 13
                   /     \
              🔴 8        🔴 17
              /   \       /   \
          ⚫ 1     ⚫ 11  ⚫ 15  ⚫ 25
          /  \    /  \   /  \   /  \
        NIL NIL NIL NIL NIL NIL ⚫ 22 NIL
                                   /  \
                                NIL  🔴 27

验证规则：
① ✓ 每个都是红或黑
② ✓ 根 13 是黑
③ ✓ 叶子都是 NIL（黑）
④ ✓ 红的两个孩子都是黑（8的俩孩子1和11都是黑，17的俩孩子15和25都是黑）
⑤ ✓ 随便找一条路径数黑节点：13→8→1→NIL = 3个黑(13,8,1)
                     13→17→25→22→NIL = 3个黑(13,17,25)（22不是黑，是红不算）
   每条路径都是 3 个黑节点 ✓
```

**这棵树的查找深度最多 5 层（13→17→25→22→27），7个节点的全黑路径 3 层，最长路径 5 层 ≤ 3×2。**

---

## 插入：新来一个节点，怎么放？

### 第一步：和新节点是什么颜色？

```
规则：新节点默认涂红色 🔴

为什么？
- 如果涂黑色 → 违反规则⑤（这条路多了一个黑节点）→ 修复范围大
- 如果涂红色 → 只可能违反规则④（"红红"）→ 修复范围小
```

### 第二步：按 BST 规则找到位置，挂上去

```cpp
// 向这个树插入 10：
//         ⚫ 13
//        /     \
//     🔴 8     🔴 17
//     /  \     /  \
//  ⚫ 1  ⚫ 11 ⚫ 15 ⚫ 25

// BST 查找过程：
// 10 < 13 → 往左
// 10 > 8  → 往右
// 10 < 11 → 往左，发现是 NIL → 挂在这

// 插入后：
//         ⚫ 13
//        /     \
//     🔴 8     🔴 17
//     /  \     /  \
//  ⚫ 1  ⚫ 11 ⚫ 15 ⚫ 25
//        /
//     🔴 10    ← 新来的！红的！
```

### 第三步：检查有没有违反规则

```
🔴 10 的爸爸 🔴 8 也是红的 → 违反规则④（红红）！
需要修复！
```

### 第四步：分情况处理

关键看**叔叔**（爸爸的兄弟）是什么颜色：

```
         ⚫ 13（爷爷）
        /     \
     🔴 8     🔴 17（叔叔）
     /  \
  ⚫ 1  🔴 11（爸爸）
       /
    🔴 10（新来的，我！）

叔叔 17 是 🔴 红色 → Case 1：变色！
```

**Case 1：叔叔是红的 → 变色就行，不用旋转**

```
变色操作：
  爸爸 8：  红 → 黑 ⚫
  叔叔 17： 红 → 黑 ⚫
  爷爷 13： 黑 → 红 🔴

变色后：
         🔴 13
        /     \
     ⚫ 8     ⚫ 17
     /  \
  ⚫ 1  🔴 11
       /
    🔴 10

现在 13 的爸爸是谁？13是根，没爸爸。
规则②：根必须是黑的 → 13 变回黑

最终结果：
         ⚫ 13
        /     \
     ⚫ 8     ⚫ 17
     /  \
  ⚫ 1  🔴 11
       /
    🔴 10

检查所有规则：✓ 全部通过！
```

### 用动画视角：Case 2 + Case 3（需要旋转）

插入 3 到下面这棵树：

```
         ⚫ 13
        /     \
     🔴 8     ⚫ 17
     /  \
  ⚫ 1  ⚫ 11   ← 叔叔 11 是 ⚫ 黑色！
  (爸爸)
    \
     🔴 3      ← 新来的，在爸爸的右边
```

因为叔叔是黑的，不能用 Case 1。而且新节点在爸爸的**右边**：

**Case 2：新节点在右边，而且叔叔是黑的 → 先对爸爸左旋，变成 Case 3**

```
左旋前：                      左旋后：
    🔴 8                        🔴 3
    /  \                        /  \
 ⚫ 1   ⚫ 11                   🔴 8 ⚫ 11
    \                          /
     🔴 3 ← 我和爸爸互换了！  ⚫ 1
                              \
                               🔴 3(原来的爸爸) ← 现在在左边了
```

**Case 3：新节点在左边，叔叔是黑的 → 爸爸变黑，爷爷变红，对爷爷右旋，一步到位**

```
旋转前：          旋转后：
    ⚫ 13               ⚫ 8
    /  \               /  \
 🔴 8  ⚫ 17         ⚫ 3  🔴 13
  /  \               /     /  \
⚫ 3 ⚫ 11          ⚫ 1   ⚫ 11 ⚫ 17
/
⚫ 1

完成！检查：
- 根 8 是黑的 ✓
- 没有相邻红色 ✓
- 每条路径黑节点数相同 ✓
```

### 插入总结

```
新节点 🔴 → 挂 BST 位置
          ↓
       爸爸是黑的？→ 完美！什么也不用做 ✓
          ↓
       爸爸是红的？→ 违反规则④！
          ↓
       看叔叔：
     🔴 红的   → Case 1：爸叔变黑，爷变红，往上继续检查
     ⚫ 黑的 → 我在右边？Case 2：旋转变 Case 3
             → 我在左边？Case 3：爸变黑爷变红，旋转一步到位

最多旋转 2 次！这就是红黑树比 AVL 快的原因。
```

---

## 为什么不用 AVL 树？

| | AVL 树 | 红黑树 |
|---|---|---|
| 平衡要求 | 严格：左右子树高度差 ≤ 1 | 宽松：最长 ≤ 最短 × 2 |
| 插入后 | **可能需要 O(log N) 次旋转** | **最多 2 次旋转** |
| 删除后 | 可能需要 O(log N) 次旋转 | 最多 3 次旋转 |
| 查找 | 稍快（树更矮）| 稍慢（树稍高）|
| 适用场景 | **读多写少**（查数据库索引）| **读写频繁**（map 就是这样的）|

红黑树是工程上的权衡：**牺牲一点查找精度，大幅减少旋转次数。**

---

## 用 map 的时候你不需要关心红黑树

```cpp
std::map<int, string> m;

// 插入
m[1] = "one";
m.insert({2, "two"});
m.emplace(3, "three");

// 查找
if (auto it = m.find(2); it != m.end()) {
    cout << it->second;  // "two"
}

// 删除
m.erase(1);

// 范围查询
auto start = m.lower_bound(10);   // 第一个 key ≥ 10
auto end   = m.upper_bound(50);   // 第一个 key  > 50

// 遍历（按 key 升序）
for (auto& [k, v] : m) { ... }
```

你只要知道：**这些操作都是 O(log N)，背后有一棵会自动保持平衡的红黑树在默默工作。**

---

## UE TMap

UE 的 `TMap` **不是**红黑树，是**哈希表**（≈ `std::unordered_map`）：

```cpp
TMap<FString, int32> Map;
Map.Add(TEXT("Key"), 42);
```

UE 选哈希表的原因：游戏里"按名字查到对象"比"按范围遍历"频繁得多，O(1) 更重要。有序需求用单独的 `TSortedMap`。

---

# 05 UnorderedMap

> 无序关联容器，底层是哈希表。平均 O(1) 查找/插入/删除。

## 存储位置

```
            栈区                      堆区（bucket数组+节点链表）
       ┌──────────────┐
       │unordered_map │        ┌───┬───┬───┬───┬───┬───┬───┬───┐
       │  buckets ●───┼───────→│ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ ← bucket数组(堆)
       └──────────────┘        └─┬─┴───┴───┴─┬─┴───┴───┴───┴───┘
       sizeof≈56字节              │            │
      对象在栈                    v            v
                            ┌─────────┐  ┌─────────┐
                            │ K1  V1  │  │ K4  V4  │ ← 节点(堆)
                            │ next ●──│─→│ next ●──│─→...
                            └─────────┘  └─────────┘
```

| 存储位置 | 内容 | 生命周期 |
|---------|------|---------|
| **栈** | unordered_map 对象本身（约 56 字节） | 离开作用域自动销毁 |
| **堆** | bucket 数组 + 所有哈希节点（拉链法） | 析构时释放 |

哈希表的核心思想：**通过哈希函数将 key 映射到数组索引，实现 O(1) 访问**。

```cpp
// 最简单的哈希表
template<typename K, typename V>
class SimpleHashMap {
    vector<list<pair<K, V>>> buckets;  // bucket 数组，每个bucket是一个链表

    V& operator[](const K& key) {
        size_t idx = hash(key) % buckets.size();
        auto& bucket = buckets[idx];
        for (auto& [k, v] : bucket) {
            if (k == key) return v;
        }
        bucket.emplace_back(key, V{});
        return bucket.back().second;
    }
};
```

## Bucket

Bucket（桶）是哈希表的基本单位。每个 bucket 可以存储 0 个或多个元素（当发生哈希冲突时）。

```cpp
std::unordered_map<int, std::string> map;
map.bucket_count();      // bucket 总数
map.bucket(key);         // key 对应的 bucket 索引
map.bucket_size(idx);    // 第 idx 个 bucket 中的元素数
```

## Hash

哈希函数的好坏直接影响性能：

```cpp
// 好的哈希函数：均匀分布
size_t good_hash(const string& s) {
    size_t h = 0;
    for (char c : s) {
        h = h * 31 + c;  // 经典的多项式哈希
    }
    return h;
}

// 坏的哈希函数：聚集
size_t bad_hash(const string& s) {
    return s.length();  // 所有同长度的字符串冲突！
}
```

C++ 标准库提供了常见类型的 `std::hash` 特化：

```cpp
std::hash<int>{} (42);           // 整数哈希
std::hash<std::string>{} ("hi"); // 字符串哈希
// 自定义类型需要自己特化 std::hash
```

## 拉链法（Separate Chaining）

C++ `std::unordered_map` 使用的冲突解决方法。每个 bucket 是一个链表（或类似结构），冲突的元素追加到链表中。

```
Bucket 0: [K1,V1] → [K3,V3]  (K1 和 K3 哈希到同一个 bucket)
Bucket 1: (空)
Bucket 2: [K2,V2]
Bucket 3: (空)
Bucket 4: [K4,V4] → [K5,V5] → [K6,V6]
```

查找过程：hash(key) → 定位 bucket → 在 bucket 的链表中顺序查找。

**负载因子（Load Factor）** = 元素数 / bucket 数。当超过 `max_load_factor`（默认 1.0）时触发 rehash。

## 开放寻址（Open Addressing）

另一种冲突解决方法，不用链表，冲突时找下一个空位。C++ 的 `std::unordered_map` 不用这种方法，但一些高性能哈希表（如 `absl::flat_hash_map`）使用。

```
插入 K2，hash=2，但 bucket[2] 已被占 → 检查 bucket[3] → bucket[4] → ...
                                                    ↑ 线性探测
```

**三种探测方式：**
- 线性探测：`h(k) + i`（简单但容易聚集）
- 二次探测：`h(k) + i²`（减少聚集）
- 双重哈希：`h1(k) + i * h2(k)`（最佳分布）

## rehash

当负载因子超限时，需要扩容并重新哈希所有元素：

```
rehash前: 8个bucket, 7个元素
rehash后: 16个bucket, 7个元素 → 每个元素重新 hash(key) % 16
```

rehash 是 **O(N)** 的操作，类似 vector 的扩容。

```cpp
map.rehash(100);  // 手动触发 rehash，设置 bucket 数
map.reserve(100); // 预分配（类似 vector::reserve）
```

## iterator

UnorderedMap 的迭代器是**前向迭代器**（只能 ++，不能 --），按 bucket 顺序遍历。**迭代器在 rehash 时会失效**（类似 vector 扩容），但插入元素不会失效（除非触发 rehash）。

## 为什么 O(1)

理想情况下（好的哈希函数 + 合适的负载因子）：
- 哈希到 bucket：O(1)
- 在 bucket 内查找：链表长度 ≈ 负载因子 ≈ 1，O(1)
- 总：**O(1) 均摊**

最坏情况（所有元素哈希到同一个 bucket）：**退化到 O(N)**。这就是为什么哈希函数质量很重要。

## Dictionary（C# 对比）

C# 的 `Dictionary<K, V>` 等价于 `std::unordered_map<K, V>`：

```csharp
Dictionary<int, string> dict = new Dictionary<int, string>();
dict.Add(1, "one");
dict[2] = "two";
// 底层也是哈希表 + 拉链法
```

C# Dictionary 的 Int32 key 有特殊优化——直接作为哈希值，避免了哈希函数调用。

[整合自 3.CSharp/2.集合/17.Dictionary常规使用.md、18.Dictionary基本概念.md、19.哈希表.md]

**C# Dictionary 核心原理：** 底层是数组 + 链表。通过哈希函数将 key 映射到 bucket 索引，冲突用拉链法解决。当元素数超过 buckets 的 `LoadFactor` 时自动扩容（扩容到最近的质数）。

---

# 06 String

> `std::string` 是最常用的字符串类。看似简单，底层有很多值得了解的优化。

## string 的本质

```cpp
// std::string ≈ std::basic_string<char>
template<typename CharT>
class basic_string {
    CharT*  _M_p;        // 指向字符数据
    size_t  _M_size;     // 字符串长度
    // 可能还有 capacity 等信息
};
```

GCC 的实现中，string 对象本身通常是 **32 字节**（64位）。

## SSO（Small String Optimization，短字符串优化）

绝大多数字符串都很短。如果每次都去堆上分配，开销太大。SSO 就是：**短字符串直接存储在 string 对象内部（栈上），不分配堆内存**。

```cpp
// GCC std::string 的内部结构（简化）
class string {
    union {
        char   _M_local_buf[16];  // 本地缓冲区（对象内，随对象在栈上）
        char*  _M_allocated_ptr;  // 指向堆内存
    };
    size_t _M_size;
    size_t _M_capacity;  // 15 表示本地存储（SSO），>15 表示堆存储
};
```

### 两种存储模式（取决于字符串长度）

```
短字符串 (< 16 字节) → 纯栈
    ┌────────────────────────────┐
    │ string 对象（栈上）         │
    │ ┌────────────────────────┐ │
    │ │ "Hello\0"              │ │ ← 数据直接在对象内！无堆分配
    │ │ size=5, capacity=15    │ │
    │ └────────────────────────┘ │
    └────────────────────────────┘
    sizeof(string) = 32字节（都在栈上）

长字符串 (≥ 16 字节) → 栈 + 堆
    ┌──────────────────────────┐      ┌──────────────────────┐
    │ string 对象（栈上）       │      │ 堆内存               │
    │ ptr ●────────────────────┼─────→│ "Very long string.." │
    │ size=100, capacity=128   │      └──────────────────────┘
    └──────────────────────────┘
    栈上32字节                    堆上128字节
```

| 存储位置 | 内容 | 生命周期 |
|---------|------|---------|
| **栈** | string 对象本身（32字节，含 SSO 缓冲区） | 离开作用域自动销毁 |
| **堆**（仅长字符串） | 字符数据 | string 析构时 delete[] |

**不同编译器的 SSO 阈值：**
- GCC (libstdc++)：15 字符
- MSVC：15 字符
- Clang (libc++)：22 字符

## COW（Copy-On-Write，写时复制）

老的 GCC 实现（GCC 4.x 之前）使用 COW：拷贝 string 时不立即复制数据，而是共享同一块内存，只有修改时才复制。**C++11 后已废弃**，原因：多线程下 COW 的引用计数需要原子操作，反而比直接拷贝更慢；且不符合 `operator[]` 的 const 语义。

## 常用操作和复杂度

| 操作 | 复杂度 | 说明 |
|------|--------|------|
| `s[i]` | O(1) | 随机访问 |
| `s + t` | O(N+M) | 拼接，需要分配新内存 |
| `s.find(t)` | O(N*M) | 朴素查找，部分实现用 Boyer-Moore |
| `s.substr(pos, n)` | O(N) | 需要拷贝子串 |
| `s == t` | O(N) | 先比较长度，再逐字符比较 |
| `s.append(t)` | O(M) | 可能触发扩容 |
| `std::to_string(n)` | O(log N) | 数字转字符串 |

**C++17 引入 `std::string_view`**：不拥有数据的字符串视图，传递/查找/比较等操作零拷贝。

```cpp
std::string_view sv = "Hello";  // 不拷贝，只是指针+长度
sv.substr(1, 3);  // 不拷贝，只调整指针
// 注意：string_view 不保证 '\0' 结尾，传给 C API 要小心
```

## 与 C 字符串的互操作

```cpp
// C++ → C
std::string s = "hello";
const char* c_str = s.c_str();  // 保证 '\0' 结尾
const char* data  = s.data();   // C++11 前不保证 '\0', C++11 后保证

// C → C++
const char* c = "world";
std::string s1(c);              // 构造
std::string s2(c, 3);           // 只取前3个字符 "wor"
```

## 常见面试题

**Q: `s.c_str()` 返回的指针什么时候失效？**

→ 任何修改 string 的操作（`append`、`operator=`、`resize` 等）都可能导致失效（类似 vector 扩容）。

**Q: `std::string a = "abc"` 和 `std::string a("abc")` 的区别？**

→ 没区别。两者都调用构造函数，都是深拷贝。`=` 在这里是拷贝初始化，不是赋值。

**Q: 如何高效拼接大量字符串？**

→ 使用 `std::ostringstream` 或在 C++20 用 `std::format`。或者用 `reserve` 预分配，然后逐个 `append`。

---

# 07 SmartPointer

> C++ 智能指针是 RAII 思想的体现——把资源的生命周期和对象的生命周期绑定。

[整合自 3.进阶/7.智能指针.md、8.深入理解sharedptr.md、9.深入理解weakptr.md、Vector/现代C++智能指针详解.md]

## 为什么需要智能指针

C++ 中手动管理内存的传统方式很容易出错：

- **野指针**：未初始化或已释放的指针
- **空悬指针**：指向已释放内存的指针
- **内存泄漏**：忘记释放动态分配的内存
- **双重释放**：同一个指针被 delete 两次

智能指针的解决方案：**RAII（Resource Acquisition Is Initialization）**——资源获取即初始化。

```cpp
// 传统方式：容易忘记 delete
int* p = new int(10);
// ... 很多代码 ...
delete p;  // 忘了？内存泄漏！

// RAII方式
{
    std::unique_ptr<int> p(new int(10));
    // ... 很多代码 ...
}  // 离开作用域，自动调用析构 → 自动 delete
```

## std::unique_ptr

**独占所有权**的智能指针。不能拷贝，只能移动。

### 存储位置

```
unique_ptr<T>:
    栈上的 unique_ptr 对象（8字节，就一个裸指针）
          │
          ▼
    堆上的 T 对象（通过 new 分配）
```

```cpp
std::unique_ptr<int> p1(new int(10));  // p1在栈，*p1在堆
std::unique_ptr<int> p2 = std::make_unique<int>(20);  // C++14推荐
// p2 离开作用域 → 自动 delete 堆上的 int → 无泄漏
```

| 存储位置 | 内容 | 生命周期 |
|---------|------|---------|
| **栈** | unique_ptr 对象本身（8字节，一个裸指针） | 离开作用域自动销毁 |
| **堆** | 被管理的对象 | unique_ptr 析构时自动 delete |

**内部实现：** 禁用拷贝构造和拷贝赋值。

```cpp
unique_ptr(const unique_ptr&) = delete;
unique_ptr& operator=(const unique_ptr&) = delete;
```

**大小：** 默认删除器下和一个裸指针一样大（8字节）。如果使用函数指针作为删除器，大小变为裸指针的两倍（16字节）。

### 自定义删除器

```cpp
// 删除器是函数指针 → sizeof 变为 16 字节（多存一个函数指针）
std::unique_ptr<FILE, decltype(&fclose)> file(fopen("test.txt", "r"), fclose);

// 删除器是无状态 lambda → sizeof 仍为 8 字节（空对象优化）
auto deleter = [](FILE* f) { fclose(f); };
std::unique_ptr<FILE, decltype(deleter)> file2(fopen("test.txt", "r"), deleter);
```

[整合自 3.进阶/7.智能指针.md 中关于 unique_ptr 的内容]

## std::shared_ptr

**共享所有权**的智能指针，通过**引用计数**管理生命周期。

### 存储位置（三层结构）

```
shared_ptr<T>:
    栈(shared_ptr对象)                 堆(T对象)              堆(控制块)
    ┌───────────────┐              ┌─────────────┐      ┌──────────────────┐
    │ ptr ──────────┼─────────────→│     T       │      │ strong_count: 2  │
    │ ctrl ●────────┼──┐           └─────────────┘      │ weak_count:   0  │
    └───────────────┘  │                                │ deleter          │
     sizeof = 16字节    │                                └──────────────────┘
     (两个指针×8)      └──────────────────────────────→   (make_shared时和T
                                                          对象一起分配)
```

| 存储位置 | 内容 | 生命周期 |
|---------|------|---------|
| **栈** | shared_ptr 对象本身（16字节，两个指针） | 离开作用域自动销毁 |
| **堆-数据** | 被管理的 T 对象 | 引用计数归零时 delete |
| **堆-控制块** | 强/弱引用计数 + 删除器 + 分配器 | 和数据一起或独立回收 |

> **⚠️ `new shared_ptr` 时控制块和数据分两次分配；`make_shared` 一次分配合并两者，更高效。**

```cpp
std::shared_ptr<int> p1 = std::make_shared<int>(42);  // 推荐
std::shared_ptr<int> p2 = p1;  // 引用计数 +1

p1.use_count();  // 2
p2.reset();      // p2 释放，引用计数 -1
p1.use_count();  // 1
// p1 离开作用域，引用计数归零 → delete
```

**底层结构（两个指针）：**

```cpp
// shared_ptr 内部有两个指针
element_type*     _M_ptr;         // 指向管理的数据
__shared_count    _M_refcount;    // 指向控制块

// 控制块包含：
// - 强引用计数（shared_ptr 的数量）
// - 弱引用计数（weak_ptr 的数量）
// - 删除器（Deleter）
// - 分配器（Allocator）
```

**大小：** `sizeof(shared_ptr<T>)` = 裸指针的 2 倍（64位下 16 字节）。

**引用计数为什么在堆上？** 因为多个 shared_ptr 需要共享同一个引用计数。如果是对象内的成员，拷贝时无法共享。

### 循环引用问题

```cpp
class A { public: std::shared_ptr<B> b_ptr; };
class B { public: std::shared_ptr<A> a_ptr; };

auto a = std::make_shared<A>();
auto b = std::make_shared<B>();
a->b_ptr = b;
b->a_ptr = a;
// a 和 b 离开作用域 → 引用计数永远不为0 → 内存泄漏！
```

**解决方案：** 把其中一个改为 `std::weak_ptr`。

### enable_shared_from_this

[整合自 3.进阶/8.深入理解sharedptr.md]

当需要在类的成员函数中获取指向自己的 shared_ptr 时，**不能直接用 `this` 构造新的 shared_ptr**：

```cpp
class MyClass : public std::enable_shared_from_this<MyClass> {
public:
    std::shared_ptr<MyClass> getShared() {
        return shared_from_this();  // 正确！
        // return std::shared_ptr<MyClass>(this);  // 错误！会导致双重释放
    }
};
```

**原理：** `enable_shared_from_this` 内部维护一个 `weak_ptr`，当对象被 `shared_ptr` 管理时会自动设置。`shared_from_this()` 从这个 `weak_ptr` 提升得到共享同一引用计数的 `shared_ptr`。

[整合自 3.进阶/8.深入理解sharedptr.md 中手写 shared_ptr 的内容]

### 手写简化版 shared_ptr

```cpp
template <typename T>
class SimpleSharedPtr {
public:
    explicit SimpleSharedPtr(T* ptr = nullptr)
        : ptr_(ptr), count_(ptr ? new size_t(1) : nullptr) {}

    SimpleSharedPtr(const SimpleSharedPtr& other)
        : ptr_(other.ptr_), count_(other.count_) {
        if (count_) ++(*count_);
    }

    SimpleSharedPtr& operator=(const SimpleSharedPtr& other) {
        if (this != &other) {
            release();
            ptr_ = other.ptr_;
            count_ = other.count_;
            if (count_) ++(*count_);
        }
        return *this;
    }

    ~SimpleSharedPtr() { release(); }

    T& operator*()  const { return *ptr_; }
    T* operator->() const { return ptr_; }
    size_t use_count() const { return count_ ? *count_ : 0; }

private:
    void release() {
        if (count_ && --(*count_) == 0) {
            delete ptr_;
            delete count_;
        }
    }
    T* ptr_;
    size_t* count_;
};
```

### make_shared vs new shared_ptr

```cpp
// make_shared：一次分配（对象 + 控制块一起）
auto p1 = std::make_shared<int>(42);

// new + shared_ptr：两次分配（对象、控制块分开）
std::shared_ptr<int> p2(new int(42));

// make_shared 更好：更高效（少一次分配）、异常安全
```

### 线程安全性

- **引用计数的更新是线程安全的**（原子操作）
- **修改 shared_ptr 指向的数据不是线程安全的**（需要加锁）
- **多个线程同时修改同一个 shared_ptr 对象本身（如赋值）不是线程安全的**

```cpp
// 线程安全：拷贝 shared_ptr（只操作引用计数）
std::shared_ptr<int> p2 = p1;  // OK，原子操作

// 线程不安全：修改指向的数据
(*p1)++;  // 需要加锁！

// 线程不安全：同时给 shared_ptr 赋值
p1 = std::make_shared<int>(42);  // 多线程下需要加锁！
```

## std::weak_ptr

[整合自 3.进阶/9.深入理解weakptr.md]

**弱引用**——指向 shared_ptr 管理的对象，但不增加引用计数。

```cpp
std::shared_ptr<int> sp = std::make_shared<int>(42);
std::weak_ptr<int> wp = sp;  // 不影响 sp 的引用计数

// 使用前必须 lock()
if (auto locked = wp.lock()) {  // lock() 返回 shared_ptr
    std::cout << *locked << std::endl;
}

// 检查是否过期
if (!wp.expired()) {
    // 注意：多线程下 expired() 和 lock() 之间对象可能被释放
    // 正确用法：
    auto sp = wp.lock();
    if (sp) { /* 使用 sp */ }
}
```

**主要用途：**
1. **打破循环引用**：在观察者模式等场景中避免 shared_ptr 循环引用
2. **缓存**：缓存对象但不阻止其释放——不用的对象自动析构，需要时尝试获取
3. **观察者**：观察 shared_ptr 对象的生命周期，不拥有所有权

[整合自 3.进阶/9.深入理解weakptr.md 中资源所有权的内容]

> **核心理解：** weak_ptr 并不拥有对象。它像一个"房地产中介"——中介不拥有房子，但在你想买的时候可以帮你联系房主。`lock()` 就是问房主"房子还在吗？"的过程。

## 三种智能指针存储位置和大小对比

| 特性 | unique_ptr | shared_ptr | weak_ptr |
|------|-----------|------------|----------|
| 所有权 | 独占 | 共享 | 无（弱引用） |
| 拷贝 | 禁止 | 引用计数+1 | 允许，不影响计数 |
| 对象本身在哪 | 栈（或作为成员） | 栈（或作为成员） | 栈（或作为成员） |
| 管理的数据在哪 | 堆 | 堆 | 不拥有数据 |
| 控制块在哪 | 无 | 堆 | 堆（和shared_ptr共享） |
| 大小（64位） | 8字节（默认删除器） | 16字节（两个指针） | 16字节（两个指针） |
| 适用场景 | 明确唯一所有者 | 共享所有权 | 打破循环/缓存 |

[整合自 Vector/现代C++智能指针详解.md 中的性能对比]

**性能数据（开启优化）：** unique_ptr ≈ new/delete；shared_ptr 比 new 慢约 2-3 倍（原子引用计数开销）。

---

# 08 Template

> 模板就是编译期的代码生成器。`T` 在编译前不存在，只是占位符。

[整合自 2.面向对象/1.上部.md 中模板相关的内容]

## 模板的本质

很多人觉得 `T` 是一种"万能类型"。实际上 **`T` 在编译之前根本不存在，它只是编译器的一个占位符**。

```cpp
template<typename T>
T Add(T a, T b) { return a + b; }

// 当你写：
Add(1, 2);     // 编译器生成 int Add(int, int)
Add(1.5, 2.3); // 编译器生成 double Add(double, double)

// 编译后，`T` 已经消失了
// 只有 int Add(int,int) 和 double Add(double,double)
```

模板和 virtual 恰好是两种完全不同的思想：
- **virtual**：运行时决定调用哪个函数（动态多态）
- **template**：编译时生成对应的代码（静态多态）

## 模板实例化（Template Instantiation）

```cpp
// 你写的：
template<typename T>
void Print(T value) { cout << value; }

Print(1);       // → 编译器生成 void Print(int value)
Print("abc");   // → 编译器生成 void Print(const char* value)
Print(3.14);    // → 编译器生成 void Print(double value)
```

编译器实际上偷偷生成了三个独立的函数。CPU 根本不知道"template"是什么。

## 为什么模板没有运行时开销？

因为生成工作全部发生在**编译阶段**。`vector<int>` 编译后实际上变成了类似 `class vector_int { ... };` 的具体类。CPU 看到的是真正的类，没有间接调用，所以零开销。

## 为什么模板代码都写在 .h？

模板是在**编译时实例化**的。编译 `main.cpp` 的时候，编译器必须看到模板的完整代码，否则无法生成 `int Add(int,int)`。

```cpp
// Add.h
template<typename T>
T Add(T a, T b);  // 只有声明

// Add.cpp
template<typename T>
T Add(T a, T b) { return a + b; }  // 实现在这里

// main.cpp
Add(1, 2);  // 编译错误！main.cpp 编译时看不到 Add 的实现
```

## 模板 vs 虚函数

| 模板 | 虚函数 |
|------|--------|
| 编译时生成代码 | 运行时查 vtable |
| 没有运行时开销 | 有一次虚函数调用开销 |
| 类型必须在编译期确定 | 类型可以运行时决定 |
| 静态多态 | 动态多态 |
| 可能导致代码膨胀（Code Bloat）| 二进制更小 |

## 为什么模板这么强大？

模板不仅能替换**类型**，还能替换**值**：

```cpp
template<int N>
class Array {
    int data[N];
};

Array<10> a1;   // 编译器生成 int data[10]
Array<100> a2;  // 编译器生成 int data[100]
```

所以模板其实更像：**一种编译时代码生成语言**。

## 和 C# 泛型的区别

- **C++ 模板**：编译时为每种类型生成独立代码，更快但可能导致代码膨胀
- **C# 泛型**：运行时 CLR 仍知道"这是泛型"，很多类型共享一份 IL，生成的机器码更少

## 为什么成员模板函数不能是 virtual？

```cpp
class Animal {
    template<typename T>
    virtual void make_sound() { }  // 编译错误！
};
```

原因：C++ 的编译与链接模型是"分离"的。
- 模板：要等到所有编译单元都编译完才知道有多少个实例
- 虚函数：编译类时就必须确定虚函数表的大小和布局

一个函数不能同时是"编译完才知道有多少个"（模板）和"编译类时必须确定"（虚函数）。

---

# 09 面向对象三大特性

> 封装、继承、多态——C++ 面向对象的三个核心。面试必问。

[整合自 2.面向对象/1.上部.md]

## 一、封装（Encapsulation）

### 是什么？

**把数据和操作数据的方法捆在一起，对外隐藏内部细节，只暴露必要的接口。**

```cpp
class BankAccount {
private:           // ← 内部细节，外面看不到
    double balance;

public:            // ← 对外接口
    void Deposit(double amount) {
        if (amount > 0) balance += amount;
    }
    double GetBalance() const {
        return balance;
    }
};

BankAccount acc;
acc.balance = 999;     // ❌ 编译错误！private 不能直接访问
acc.Deposit(100);      // ✓ 通过公开接口操作
```

### 怎么实现？——访问修饰符

| 修饰符 | 类内部 | 派生类 | 外部 |
|--------|--------|--------|------|
| **public** | ✓ | ✓ | ✓ |
| **protected** | ✓ | ✓ | ✗ |
| **private** | ✓ | ✗ | ✗ |

### 封装解决了什么问题？

```
没有封装：
  外部代码可以直接改 balance → 可能改成负数 → 数据被破坏

有封装：
  外部只能通过 Deposit() 改 → Deposit() 里可以校验 → 数据安全
```

> **封装 = 给你的数据装一扇门。想进来？走我提供的门（public 方法），别翻窗（直接访问 private）。**

---

## 二、继承（Inheritance）

### 是什么？

**一个类（派生类）从另一个类（基类）那里获得属性和方法，避免重复写代码。**

```cpp
// 基类：定义"共性"
class Animal {
protected:
    string name;
public:
    void SetName(const string& n) { name = n; }   // 所有动物都有
    virtual void Speak() { cout << "???" << endl; }
};

// 派生类：只需要写"个性"
class Dog : public Animal {
public:
    void Speak() override { cout << name << "：汪汪" << endl; }
    void WagTail()      { cout << "摇尾巴" << endl; }  // Dog 特有
};

class Cat : public Animal {
public:
    void Speak() override { cout << name << "：喵喵" << endl; }
};

Dog dog;
dog.SetName("旺财");   // 继承来的，不用自己写
dog.Speak();           // 旺财：汪汪
```

### 继承了什么，没继承什么？

| | 是否继承 | 说明 |
|------|---------|------|
| public/protected 成员 | ✓ 继承 | 派生类可以直接用 |
| private 成员 | ✓ 被继承了但**不能直接访问** | 只能通过基类的 public/protected 方法间接访问 |
| 构造函数 / 析构函数 | ✗ 不继承 | 每个类有自己的 |
| 赋值运算符 | ✗ 不继承 | 编译器生成或自己写 |
| 友元关系 | ✗ 不继承 | 朋友的朋友不是我的朋友 |

### 三种继承方式

```cpp
class Dog : public Animal    { ... };  // 最常用：基类的 public→public, protected→protected
class Dog : protected Animal { ... };  // 基类的 public→protected, protected→protected
class Dog : private Animal   { ... };  // 基类的 public→private, protected→private
```

**99% 的情况用 `public` 继承。** `protected` 和 `private` 继承极少用到。

---

## 三、多态（Polymorphism）

### 是什么？

**同一个接口，不同对象表现不同的行为。**

```cpp
Animal* animals[] = { new Dog(), new Cat(), new Dog() };
for (Animal* a : animals) {
    a->Speak();  // 同一行代码！
}
// 输出：汪汪、喵喵、汪汪  ← 不同的对象，不同的行为
```

### 怎么实现？

虚函数（virtual）+ 基类指针/引用 + 派生类重写。详见下一节 [#10 VirtualFunction](#10-virtualfunction)。

### 三种C++多态

| 类型 | 机制 | 决定时机 | 例子 |
|------|------|---------|------|
| **函数重载** | 函数名相同，参数不同 | 编译时 | `void f(int)` / `void f(float)` |
| **模板** | 代码生成 | 编译时 | `template<typename T> T Add(T,T)` |
| **虚函数** | vtable + vptr | **运行时** | `virtual void Speak()` |

---

# 10 VirtualFunction

> 虚函数是 C++ 运行时多态的基石。理解 vtable 和 vptr 就理解了虚函数的全部。

## 什么是多态？

多态（Polymorphism）不是"有很多函数"，而是：

> **同一条调用语句，在运行时根据对象的真实类型执行不同的函数。**

```cpp
class Animal {
public:
    virtual void Speak() { cout << "Animal\n"; }
};
class Dog : public Animal {
public:
    void Speak() override { cout << "Dog\n"; }
};

Animal* p = new Dog();
p->Speak();  // 输出 "Dog" ← 变量类型是 Animal*，但执行 Dog::Speak()
```

多态 = 统一接口，不同实现。

## 继承一定要虚函数吗？

**不一定。** 继承可以完全不写虚函数，只是"复用代码"。

```cpp
// 没有虚函数的继承——纯复用代码
class Base {
public:
    void Print() { cout << "Base" << endl; }  // 普通函数，没有 virtual
};
class Derived : public Base {
public:
    void Print() { cout << "Derived" << endl; }  // 不是重写，是隐藏
};

Derived d;
d.Print();            // "Derived"（调用自己的）
d.Base::Print();      // "Base"（显式调基类的）

Base* p = &d;
p->Print();           // "Base"！没有多态，编译器只看 p 的类型是 Base*
```

| 场景 | 需要虚函数吗 | 例子 |
|------|------------|------|
| 只想复用基类的代码 | ❌ 不需要 | 继承一个工具类的方法 |
| 想通过基类指针调子类方法 | ✅ 需要 | `Animal* p = new Dog(); p->Speak();` |
| 基类本身就是完整可用的类 | ❌ 不需要 | `class MyWidget : public Widget` 可能不加虚函数 |
| 希望派生类必须实现某个方法 | ✅ 需要，而且用**纯虚函数** | `virtual void Draw() = 0;` |

> **继承 ≠ 虚函数。继承是"代码复用"，virtual 是"运行时多态"。两者独立。你完全可以写一个没有任何虚函数的继承。**

## 纯虚函数是什么？

**纯虚函数 = 只有声明没有实现的虚函数，强制派生类必须重写它。**

```cpp
class Shape {
public:
    virtual void Draw() = 0;  // ← "= 0" 就是纯虚函数
    //                      ↑ 没有函数体！只是一个"规定"
};

class Circle : public Shape {
public:
    void Draw() override { cout << "画圆形" << endl; }  // 必须写，否则 Circle 也是抽象类
};

class Square : public Shape {
public:
    void Draw() override { cout << "画方形" << endl; }
};
```

**三个关键后果：**

| 后果 | 说明 |
|------|------|
| **包含纯虚函数的类 = 抽象类** | 不能 `new Shape()`，编译直接报错 |
| **派生类必须实现纯虚函数** | 不实现？那派生类也是抽象类 |
| **作为"接口"使用** | C++ 没有 interface 关键字，纯虚函数就充当接口 |

```cpp
Shape s;           // ❌ 编译错误！抽象类不能实例化
Shape* p = new Circle();  // ✅ 可以用指针/引用
p->Draw();         // "画圆形"
```

**和普通虚函数的区别：**

| | 普通虚函数 | 纯虚函数 |
|---|---|---|
| 写法 | `virtual void f() { ... }` | `virtual void f() = 0;` |
| 有实现吗 | 有默认实现 | 没有（子类必须写） |
| 能实例化吗 | 能 `new Base()` | ❌ 抽象类不能 `new` |
| 子类必须重写 | 可选（不重写就用基类的） | **必须** |

> **普通虚函数 = "你可以重写，不写的话用我的默认版本"。纯虚函数 = "你必须重写，我这只有规矩没有实现"。**

## 没有 virtual 的情况（静态绑定）

```cpp
class Animal {
public:
    void Speak() { cout << "Animal\n"; }  // 没有 virtual
};
class Dog : public Animal {
public:
    void Speak() { cout << "Dog\n"; }
};

Animal* p = new Dog();
p->Speak();  // 输出 "Animal"！
```

为什么？因为 `Speak()` 不是虚函数。编译器在编译 `p->Speak()` 时，只知道 p 的类型是 `Animal*`，于是直接把这次调用绑定到 `Animal::Speak()`。这就是**静态绑定（Static Binding）**。

## virtual 做了什么？

`virtual` 告诉编译器：**这个函数不要在编译期决定调用哪个实现，在运行时决定。**

## 虚函数表（vtable）

只要一个类里有虚函数，编译器就会为这个类生成一张**虚函数表（Virtual Table）**。

### vtable 存在哪里？

```
程序地址空间布局（简化）：

    ┌─────────────┐ 高地址
    │    栈       │ ← 局部变量、函数参数
    ├─────────────┤
    │    堆       │ ← new/malloc 分配的区域
    ├─────────────┤
    │  .bss       │ ← 未初始化全局变量（全0）
    ├─────────────┤
    │  .data      │ ← 已初始化全局/静态变量
    ├─────────────┤
    │  .rodata    │ ← 只读数据：字符串常量、虚函数表 ⚫ vtable 在这！
    ├─────────────┤
    │  .text      │ ← 代码段：函数指令、成员函数代码
    └─────────────┘ 低地址
```

> **vtable 存在 `.rodata`（只读数据段），编译时就已确定，程序加载时随 .rodata 一起映射到内存。整个类共享一张表，只有一份。**

### vptr 存在哪里？

> **vptr 在对象内部，通常在对象的起始位置。对象在哪，vptr 就在哪。**
> - 对象在**栈**上 → vptr 在栈上
> - 对象在**堆**上（new）→ vptr 在堆上
> - 对象在**静态区**（全局/static）→ vptr 在静态区

```cpp
Dog stackDog;           // 对象在栈 → vptr在栈，指向 .rodata 中的 Dog vtable
Dog* heapDog = new Dog(); // 对象在堆 → vptr在堆，指向 .rodata 中的 Dog vtable
static Dog globalDog;     // 对象在静态区 → vptr在静态区，指向 .rodata 中的 Dog vtable
```

### vtable 里面有什么？

```cpp
class Animal {
public:
    virtual void Speak();
    virtual void Eat();
};
```

```
Animal vtable
+----------------------+
| Animal::Speak        |  ← 函数地址
+----------------------+
| Animal::Eat          |
+----------------------+
```

```cpp
class Dog : public Animal {
public:
    void Speak() override;  // 重写 Speak
    // Eat 不重写
};
```

```
Dog vtable
+----------------------+
| Dog::Speak           |  ← 替换为 Dog 的实现
+----------------------+
| Animal::Eat          |  ← 沿用 Animal 的
+----------------------+
```

子类只是把对应位置的函数地址替换掉了。

## vptr（虚函数表指针）

每个有虚函数的对象内部都有一个**隐藏成员——vptr**。它存放在**对象内存的第一个位置**（通常）。

```
Dog对象的内存布局（栈/堆/静态区均可，取决于创建方式）：
+----------------------+
| vptr ────────────────┼────→ Dog vtable（在.rodata只读数据段）
+----------------------+      +------------------+
| Animal::age          |      | Dog::Speak       |
+----------------------+      +------------------+
| Dog::weight          |      | Animal::Eat      |
+----------------------+      +------------------+
                              | type_info(RTTI)  |
                              +------------------+

vptr的指向变化（构造过程中）：
  Animal构造期间 → vptr → Animal vtable
  Dog构造期间    → vptr → Dog vtable    ← 构造完成后指向最终vtable
```

**总结——vtable 和 vptr 的关键区别：**

| | vtable（虚函数表） | vptr（虚函数表指针） |
|---|---|---|
| **属于** | 类（所有对象共享一份） | 对象（每个对象一个） |
| **存储位置** | `.rodata` 只读数据段 | **对象内部**（通常在对象第一个成员位置） |
| **何时确定** | 编译时 | 对象构造时（随构造阶段变化） |
| **生命周期** | 程序加载到结束 | 随对象的生命周期 |
| **大小** | 每个虚函数一个槽（8字节/槽） | 8 字节（64位） |
| **内容** | 虚函数地址数组 | 指向 vtable 的指针 |

## 虚函数调用的完整过程

```cpp
Animal* p = new Dog();
p->Speak();
```

CPU 实际执行的流程：

```
1. 读取对象中的 vptr
       ↓
2. 根据 vptr 找到 Dog vtable
       ↓
3. 在 vtable 中定位 Speak 的索引（编译时已确定是第0项）
       ↓
4. 读取函数地址 → Dog::Speak()
       ↓
5. 跳转执行
```

比普通函数只多了一次**查表 + 间接跳转**。

## 为什么必须用指针或引用？

```cpp
Dog dog;
Animal animal = dog;  // 对象切片（Object Slicing）！
animal.Speak();       // 调用 Animal::Speak()，不是多态
```

赋值时 Dog 新增的成员（包括属于 Dog 的 vptr 相关信息）被切除。只有指针和引用不复制对象。

```
Animal* p = &dog;   // ✓ 指针指向原对象，vptr 仍是 Dog 的
Animal& ref = dog;  // ✓ 引用是别名，vptr 仍是 Dog 的
Animal a = dog;     // ✗ 切片！a 是全新的 Animal 对象
```

## 构造函数能不能是虚函数？

**不能。语法上就不允许。** `virtual` 不能和构造函数一起用：

```cpp
class Animal {
public:
    virtual Animal() { }  // ❌ 编译错误！构造函数不能是虚函数
};
```

**两个原因：**

**原因一（语法层面）：** 构造函数负责初始化对象。每个类有自己的构造函数，派生类构造时必须调用基类构造函数——这个调用链是**编译时就确定的**，不需要运行时决定。

```
构造 Dog 对象时：
  编译器明确知道：先调 Animal()，再调 Dog()
  这不需要"多态"——因为你知道你在构造什么类型。
```

**原因二（vptr 机制层面）：** 虚函数依赖 vptr 查 vtable。但 vptr 是在**构造函数执行期间**才被设置的：

```
1. 进入 Animal() 之前 → 编译器设置 vptr → Animal vtable
2. Animal() 执行完毕
3. 进入 Dog() 之前    → 编译器设置 vptr → Dog vtable
4. Dog() 执行完毕

如果在第1步之前就要用 vptr 来决定调用哪个构造函数？
→ vptr 还没设置！→ 鸡生蛋蛋生鸡的问题。
```

> **简单记：虚函数 = "不知道对象是谁，运行时查表决定"。构造函数 = "明确在创建谁，编译时就确定"。两者语义矛盾。**

---

## 构造函数里调用虚函数为什么不会多态？

```cpp
class Animal {
public:
    Animal() { Speak(); }  // 构造函数里调用虚函数
    virtual void Speak() { cout << "Animal\n"; }
};
class Dog : public Animal {
public:
    void Speak() override { cout << "Dog\n"; }
};

Dog dog;  // 输出 "Animal" 而不是 "Dog"！
```

构造顺序：

```
1. 申请对象内存
2. 构造 Animal → vptr 指向 Animal vtable
3. Animal 构造中调用 Speak() → 查 vptr → Animal::Speak()
4. Animal 构造完成
5. 构造 Dog → vptr 改为指向 Dog vtable
6. Dog 构造完成
```

在基类构造函数执行期间，vptr 指向基类的 vtable，虚函数调用不会发生向下的动态绑定。这是 C++ 标准的规定——防止访问尚未初始化的子类成员。

## 析构函数为什么要声明为 virtual？

```cpp
class Base {
public:
    ~Base() { cout << "Base\n"; }  // 不是虚函数
};
class Derived : public Base {
    int* data = new int[100];
public:
    ~Derived() { delete[] data; }  // data 不会被释放！
};

Base* p = new Derived();
delete p;  
// 输出：只有 "Base"
// 内存泄漏！Derived::~Derived() 没有执行，data 没被释放
```

如果析构函数不是 virtual，`delete p` 只调用 `Base::~Base()`。改成 `virtual ~Base()` 后：

```
delete p → 查 vptr → ~Derived() → ~Base() → 释放内存
```

输出：Derived 析构 → Base 析构。

## 析构函数一定要是 virtual 吗？所有类都要？

**不是所有类都要。只有一种情况必须写：**

> **当这个类会被当作基类继承，并且可能通过基类指针去 delete 派生类对象时。**

### 什么时候必须写？

```cpp
// ✅ 必须写 virtual 的场景
class Animal {
public:
    virtual ~Animal() = default;  // 必须写！
};

Animal* p = CreateAnimal();  // 工厂函数，返回可能是 Dog/Cat/Bird
delete p;  // 如果不写 virtual → 派生类析构不执行 → 内存泄漏
```

判断标准很简单——**只要你的类有任何一个虚函数，析构就该写 virtual**：

```cpp
class Base {
public:
    virtual void Process();  // 有一个虚函数 → 说明这个类设计了多态
    // ~Base() 也应该 virtual！哪怕析构函数体是空的也要写
};
```

### 什么时候不需要写？

```cpp
// ❌ 不需要写 virtual 的场景

// 1. 不会被继承的数据类
class Vector3 {
public:
    float x, y, z;
    // ~Vector3() 普通析构就行
};

// 2. 值语义的类（靠值传递、拷贝的）
class Color { uint8_t r, g, b, a; };
class Rect  { float x, y, w, h; };

// 3. final 类（C++11）
class MyFinalClass final {
    // 明确标记 final → 不会被继承 → 不需要 virtual
};
```

### 为什么默认不是 virtual？

| 不写 virtual | 写 virtual |
|-------------|-----------|
| sizeof(Vector3) = 12 字节 | sizeof(Vector3) = 24 字节（多了 vptr + 对齐） |
| 不用额外内存 | 每个对象多 8 字节（64位） |

C++ 零成本抽象原则：**不用，就不付出代价。** 如果编译器默认所有析构都是 virtual，100万个 Vector3 就多占 ~16MB 内存——完全没必要。

### 判断口诀

```
是不是基类？——不是 → 不写
有虚函数吗？—— 有 → 必须写（你已经付出了 vptr 的代价）
会被继承吗？—— 会 → 写
不确定？     —— 有虚函数就写，没有就不写
```

## 面试核心答案

> **"为什么基类析构函数要声明为虚函数？"**

> 因为在多态场景下，我们通常通过基类指针管理派生类对象。如果基类析构函数不是虚函数，`delete Base*` 时只会调用基类析构函数，派生类析构函数不会执行，导致派生类持有的资源无法释放。将基类析构函数声明为 `virtual` 后，析构过程会发生动态绑定，先执行派生类析构函数再执行基类析构函数，从而保证整个对象被正确析构。

---

# 11 ObjectModel

> C++ 对象在内存中到底长什么样？vtable、vptr、成员变量、成员函数如何排列？

[整合自 2.面向对象/1.上部.md 中对象模型、构造析构顺序、深拷贝浅拷贝等内容]

## C++ 对象的内存布局

### 对象存在哪里，取决于你怎么创建它

```cpp
class Point {
    float _x;
    virtual ~Point();
    virtual void print();
};

Point stackObj;                     // 整个对象（含vptr）在栈上
Point* heapObj = new Point();       // 整个对象（含vptr）在堆上
static Point staticObj;             // 整个对象（含vptr）在 .data 静态存储区

// delete heapObj 后 → 堆上的对象被销毁，vptr 自然消失
// stackObj 离开作用域 → 栈上对象自动销毁
// staticObj → 程序结束时销毁
```

### 对象内存 = 非静态成员 + vptr（如果有虚函数）

来自《深度探索C++对象模型》的经典图示：

```
Point 对象（栈/堆/静态区均可，取决于你怎么创建）
+----------------------+
| vptr ────────────────┼──→ Point vtable（在 .rodata 只读数据段）
+----------------------+      +--------------------------+
| _x (float)           |      | type_info (RTTI)         |
+----------------------+      +--------------------------+
                              | Point::~Point()          |
                              +--------------------------+
                              | Point::print()           |
                              +--------------------------+
```

### 对象各部分存储位置总结

| 组成部分 | 存储位置 | 生命周期 | 说明 |
|---------|---------|---------|------|
| **非静态成员变量** | 随对象 | 随对象 | 对象在哪就在哪 |
| **vptr** | 随对象（通常第一个成员） | 随对象，构造时设置 | 8字节，指向 .rodata 的vtable |
| **vtable** | `.rodata` 只读数据段 | 程序加载到结束 | 整个类一份，编译时生成 |
| **静态成员变量** | `.data` / `.bss` 全局静态区 | 程序加载到结束 | 不属于任何对象 |
| **成员函数代码** | `.text` 代码段 | 程序加载到结束 | 不占对象空间 |
| **RTTI (type_info)** | `.rodata`，紧邻vtable | 程序加载到结束 | `typeid` 和 `dynamic_cast` 依赖它 |

### RTTI 是什么？

**RTTI = Run-Time Type Information（运行时类型信息）**。

一句话：**程序运行时，能知道"这个指针实际指向的是什么类型"。**

```
为什么需要它？

  Animal* p = 某个函数返回的指针;
  // p 实际是 Animal？Dog？Cat？
  // 编译时不知道，运行时怎么判断？

  → RTTI 就是干这个的。
```

**RTTI 存了什么？** 类型的名字、继承关系等信息，以 `type_info` 结构体的形式存在 `.rodata`，紧挨着 vtable。

**RTTI 存在哪？** 看这张图：

```
Dog vtable（在 .rodata 只读数据段）
+--------------------------+
| Dog::Speak               | ← 虚函数地址
+--------------------------+
| Animal::Eat              |
+--------------------------+
| type_info for "Dog"      | ← RTTI 在这！vtable 的前一个槽位（通常）
|   类名: "Dog"            |    包含类名、父类信息等
|   父类: Animal           |
+--------------------------+
```

**RTTI 怎么用？** 两个方式：

```cpp
// 方式1：typeid —— 获取类型信息
Animal* p = new Dog();
cout << typeid(*p).name();  // 输出 "Dog"（或 mangled 后的名字）
if (typeid(*p) == typeid(Dog)) { ... }

// 方式2：dynamic_cast —— 安全向下转型（背后依赖 RTTI）
Dog* d = dynamic_cast<Dog*>(p);   // p 真是 Dog → 成功
Cat* c = dynamic_cast<Cat*>(p);   // p 不是 Cat → 返回 nullptr
```

**RTTI 的开销和限制：**

| 条件 | 说明 |
|------|------|
| **必须有虚函数** | 没有 vtable 的类就没有 RTTI（`dynamic_cast` 对无虚函数的类直接编译报错） |
| **占用空间** | vtable 旁多一个 `type_info` 结构体（几字节到几十字节） |
| **typeid 有开销** | 需要查 vtable 取 type_info（一次间接寻址） |
| **dynamic_cast 有开销** | 需要遍历继承链比较 type_info（比 static_cast 慢不少） |
| **可以关闭** | GCC 用 `-fno-rtti` 关闭，UE 默认关闭（游戏追求性能） |

> **简单记：RTTI = 对象的"身份证"。vtable 管"调用哪个函数"，RTTI 管"你到底是哪个类型"。**

## sizeof 空类

```cpp
class Empty {};

sizeof(Empty);  // = 1（大多数编译器）
```

为什么是 1 而不是 0？**C++ 标准要求每个对象都有独一无二的内存地址。** 1 字节确保不同实例有不同的地址。

```cpp
Empty a, b;
&a != &b;  // 必须成立！所以至少 1 字节
```

**存储位置取决于创建方式：**
```cpp
Empty stackObj;              // 1字节在栈
Empty* heapObj = new Empty;  // 1字节在堆
static Empty staticObj;      // 1字节在.bss
```

如果空类做基类，派生类中不需要额外的字节（**空基类优化 EBO**）。

[整合自 2.面向对象/1.上部.md 中 sizeof 空类、1.基础/1.sizeof关键字.md 的内容]

**sizeof 计算要点：**
1. 指针大小固定：32位=4字节，64位=8字节
2. 数组做参数退化为指针
3. struct 要考虑字节对齐
4. 字符串数组要算末尾的 `\0`

## 类对象的初始化和析构顺序

### 构造顺序

1. **虚基类**（virtual base）构造函数（按声明顺序）
2. **普通基类**构造函数（按声明顺序）
3. **成员变量**初始化（**按声明顺序，不是初始化列表顺序！**）
4. **自身构造函数体**执行

```cpp
class MyClass : public virtual Base3, public Base1, public virtual Base2 {
    int num1;
    int num2;
    Base base;   // 成员对象
};

// 构造顺序：
// Base3() → Base2() → Base1() → num1 → num2 → base() → MyClass()
```

**注意：** 成员变量的初始化顺序只与声明顺序有关，与初始化列表顺序无关！

### 析构顺序

**与构造顺序完全相反。**

## 深拷贝与浅拷贝

[整合自 2.面向对象/1.上部.md 中深拷贝浅拷贝部分]

### 浅拷贝（Shallow Copy）

只复制成员的值（包括指针的值），不复制指针指向的内容。编译器默认生成的拷贝构造函数就是浅拷贝。

```
浅拷贝：
obj1: data → [Hello]     obj2: data → [Hello]
                              ↗ 两个指针指向同一块内存！
```

### 深拷贝（Deep Copy）

不仅复制成员的值，还复制指针指向的内容。需要**自己实现**拷贝构造函数和赋值运算符。

```cpp
class MyClass {
    char* data;
public:
    // 深拷贝的拷贝构造函数
    MyClass(const MyClass& other) {
        data = new char[strlen(other.data) + 1];
        strcpy(data, other.data);
    }

    // 深拷贝的赋值运算符
    MyClass& operator=(const MyClass& other) {
        if (this == &other) return *this;   // 自赋值检查
        delete[] data;                       // 释放旧资源
        data = new char[strlen(other.data) + 1];
        strcpy(data, other.data);
        return *this;
    }

    ~MyClass() { delete[] data; }
};
```

### 何时需要深拷贝？

当类有**指向动态分配资源的指针**时（如 `char*`、`int*` 等），不实现深拷贝会导致：
- 两个对象共享同一块内存
- 一个对象析构后，另一个对象的指针变成悬空指针
- 或者同一块内存被释放两次（double free）

## 重载、重写、隐藏的区别

[整合自 2.面向对象/1.上部.md]

| 特性 | 重载（Overload） | 重写（Override） | 隐藏（Hide） |
|------|-----------------|-----------------|-------------|
| 作用域 | 同一作用域 | 父子类之间 | 父子类之间 |
| 函数名 | 相同 | 相同 | 相同 |
| 参数 | **不同** | **相同** | 可以相同可以不同 |
| virtual | 不需要 | **基类必须有** | 不重要 |
| 关系 | 同一类中 | 覆盖基类函数 | 屏蔽基类同名函数 |

## this 指针

this 是成员函数的隐式形参，指向当前对象。编译阶段由编译器自动添加到参数列表中。

```cpp
class MyClass {
    int value;
public:
    // 你写的：
    void setValue(int v) { value = v; }

    // 编译器实际生成：
    void setValue(MyClass* const this, int v) { this->value = v; }
};
```

**static 函数没有 this 指针**，因此不能访问非静态成员变量。

## 析构函数可以抛异常吗？

语法上允许，但**实践中强烈不推荐**。

原因：
1. 析构函数常被自动调用（如容器析构），异常难以捕获
2. 如果同时有另一个异常在传播，析构函数再抛异常 → `std::terminate()`
3. 容器析构时某个元素析构抛异常 → 后续元素无法正确析构

**处理方式：** 在析构函数中 try-catch 吞下异常，或提供单独的 close 方法让用户处理。

## 类成员访问权限

[整合自 2.面向对象/1.上部.md]

| 访问修饰符 | 类内部 | 派生类 | 外部 |
|-----------|--------|--------|------|
| public | ✓ | ✓ | ✓ |
| protected | ✓ | ✓ | ✗ |
| private | ✓ | ✗ | ✗ |

---

# 12 内存基础

> 编程的本质是操控数据，而数据存放在内存中。理解内存是理解一切的前提。

[整合自 3.进阶/1.内存是什么.md、2.内存分布.md]

## 内存本质

计算机内存是一块用于存储数据的空间，由一系列连续的存储单元组成。8 个 bit 为一组命名为 **byte**，每个 byte 有一个唯一的编号——**内存地址**。

- 32 位系统可寻址范围：2^32 = 4GB
- 64 位系统可寻址范围：2^64（远大于实际物理内存）

## 变量的本质

当你写下一个变量定义时，实际上是向内存申请了一块空间：

```cpp
int a = 999;      // 申请 4 字节，存 999 的补码
char c = 'c';     // 申请 1 字节
```

**大端 vs 小端：**
- 大端：高位字节放在低地址
- 小端：低位字节放在低地址（x86 架构）

## 内存分区

一道面试高频题：**"C++ 内存分为哪几个区？"**

标准答案五个区：**栈、堆、全局区、常量区、代码区**。

### 一张图记住五区

```
高地址  ┌──────────────────────────────┐
       │           栈  Stack           │  ← "临时工"
       │  局部变量、函数参数、返回地址    │     进来干活，干完就走
       │   int a;  float x;  char buf[] │     编译器自动分配/回收
       │         ↓ 向下增长              │     几 MB，别放太大东西
       ├──────────────────────────────┤
       │           堆  Heap            │  ← "仓库"
       │  new / malloc 动态分配         │     想存什么存什么，想存多久存多久
       │  vector的元素  list的节点       │     但得自己收拾（delete/free）
       │         ↑ 向上增长              │     空间大（受物理内存限制）
       ├──────────────────────────────┤
       │       全局区  静态区           │  ← "永久居民"
       │  ┌────────┐ ┌────────┐       │
       │  │ .data  │ │ .bss   │       │     .data: int x = 5; (有初值)
       │  │已初始化 │ │未初始化 │       │     .bss:  int y;     (自动=0)
       │  │全局变量 │ │全局变量 │       │     程序启动就存在，程序结束才走
       │  │静态变量 │ │静态变量 │       │     static 变量/全局变量都在这
       │  └────────┘ └────────┘       │
       ├──────────────────────────────┤
       │         常量区  .rodata        │  ← "石碑"
       │  "hello world" 字符串字面量    │     刻上去就不能改了
       │  虚函数表(vtable)  RTTI       │     程序加载时刻好，只读
       ├──────────────────────────────┤
       │         代码区  .text          │  ← "说明书"
       │  编译后的二进制机器指令         │     你写的所有函数编译后放这
       │  所有成员函数、全局函数         │     只读，防止被意外改写
低地址  └──────────────────────────────┘
```

### 五个区，五个比喻

| 区域 | 比喻 | 存什么 | 谁管理 | 生命周期 |
|------|------|--------|--------|---------|
| **栈** | 🏃 临时工 | 局部变量、函数参数、返回地址 | 编译器自动 | 离开 `{}` 就释放 |
| **堆** | 📦 仓库 | `new`/`malloc` 的动态内存 | **你**（程序员） | delete/free 时释放 |
| **全局区** | 🏠 永久居民 | 全局变量、static 变量 | 系统 | 程序启动→结束 |
| **常量区** | 🪦 石碑 | 字符串字面量、vtable、RTTI | 系统 | 程序加载→结束，只读 |
| **代码区** | 📖 说明书 | 编译后的函数指令 | 系统 | 程序加载→结束，只读 |

### 你的代码去了哪里？

```cpp
int globalVar = 42;           // → 全局区 .data（有初值）
static int staticVar = 0;     // → 全局区 .bss（初值为0，优化到这）
const char* str = "hello";    // str → 栈，"hello" → 常量区 .rodata

class Animal {
public:
    virtual void Speak() {}    // 函数代码 → 代码区 .text
    // Animal 的 vtable      → 常量区 .rodata
};

void func() {
    int a = 10;                // → 栈
    int* p = new int(99);      // p → 栈，*p → 堆
    static int count = 0;      // → 全局区 .data
    string s = "hi";           // s对象 → 栈，短字符串数据也在栈(SSO)
}
```

### 创建方式 → 存储位置 → 生命周期 速查

```cpp
// ========== 栈（自动管理） ==========
int a = 10;                    // 栈。生命周期：离开所在 {} 作用域
std::vector<int> vec;          // vec对象在栈，数据在堆。离开作用域析构

// ========== 堆（手动/智能指针管理） ==========
int* p = new int(42);          // *p 在堆，p 在栈。必须 delete
int* arr = (int*)malloc(100);  // 100字节在堆，必须 free
auto sp = std::make_unique<int>(42); // 对象在堆，离开作用域自动delete
//          ↑ unique_ptr对象本身在栈 ↑

// ========== 静态区（程序级生命周期） ==========
static int count = 0;          // .data。程序启动→结束
int globalVar = 42;            // .data。程序启动→结束
static int globalZero;         // .bss。 程序启动→结束（自动初始化为0）

// ========== 常量区（只读） ==========
const char* s = "hello";       // s在栈，"hello"在.rodata。程序加载→结束
// vtable                  →   // .rodata。编译时生成，程序加载→结束
```

### 栈（Stack）和堆（Heap）各自存什么

**一句话总结：**

| 栈 | 堆 |
|----|----|
| 编译器自动管理的、大小/生命周期在编译期确定的、临时的 | 运行时动态分配的、编译期算不出大小的、需要跨函数存活的 |

**栈——存什么：**

| 类别 | 举例 | 特点 |
|------|------|------|
| 局部变量 | `int a = 10;` `float x;` | 进 `{}` 创建，出 `{}` 销毁 |
| 函数参数 | `void f(int a)` 里的 `a` | 函数调用时压栈，返回时弹出 |
| 返回地址 | 函数执行完该跳回哪一行 | 编译器自动保存 |
| 容器对象本身 | `vector<int> v;` 那个 v（三个指针） | v 在栈，元素在堆 |
| 智能指针本身 | `unique_ptr<T> p;` 那个 p | p 在栈，管理的 T 在堆 |
| SSO 短字符串 | `string s = "hi";` | 数据在对象的内部缓冲区（栈） |

栈的特点：**快**（移动栈顶指针就分配好了）、**自动回收**、**空间有限**（通常几 MB）。

**堆——存什么：**

| 类别 | 举例 | 为什么必须在堆 |
|------|------|---------------|
| 动态数组数据 | `vector` 的元素、`string` 的长字符 | 大小运行时才确定 |
| 链表/树节点 | `list`、`map` 的节点 | 数量不确定，每个节点独立存活 |
| new/malloc 出来的 | `new int[1000]`、`malloc(1024)` | 程序员控制生命周期 |
| shared_ptr 管理的数据 | 数据对象 + 控制块 | 需要多个指针共享 |
| 大对象/大数组 | MB 级的纹理、缓冲区 | **栈放不下**（栈只有几 MB） |
| 多态对象 | `Animal* p = new Dog();` | 类型运行时才确定，需地址稳定 |
| 跨函数存活的对象 | 函数内创建 return 后还用的 | 栈上的会随函数返回销毁 |

堆的特点：**慢**（需要系统调用分配）、**手动管理**（或智能指针）、**空间大**（受物理内存限制）。

**一张图记住：**

```
"框"本身 → 栈           "框里的内容" → 堆
═══════════════         ═══════════════════
int x = 5;              new int(42);
int arr[10];            vector<int>(1000);  // 元素
Point p;                new Point();
unique_ptr<T> up;       *up 管理的对象
string s = "hello";     s[SSO放不下时] 堆上
list<int> lst;          链表节点
map<int,string> m;      红黑树节点
```

> **记忆口诀：编译期能确定大小的在栈，算不出的在堆；"容器壳"在栈，"壳里装的东西"在堆。**

## malloc 和 free

[整合自 3.进阶/10.malloc-free.md]

malloc/free 是 C 语言中用于动态内存分配的函数，底层通过 `brk/sbrk`（小内存）和 `mmap`（大内存，>128KB）系统调用实现。

### malloc 内存池设计

为了减少系统调用开销，malloc 使用内存池：先申请一大块内存，分成不同大小的 chunk，通过 bins 数组（128 个双向链表）管理：

```
小内存 (< 512B) → small bins  (每个 bin 存放相同大小的 chunk)
大内存 (> 512B) → large bins  (每个 bin 存放一个范围大小的 chunk)
释放的 chunk    → unsorted bin (缓冲区，加速分配)
极小内存 (< 64B)→ fast bins    (不合并，快速复用)
```

### 关键机制

- **chunk 结构**：malloc 分配的空间前后有控制信息（prev_size、size、标志位 P/M/A）
- **P 标志位**：前一个 chunk 是否在使用中（0=空闲，prev_size 有效）
- **free 时合并**：相邻空闲 chunk 合并，避免碎片
- **延迟分配**：malloc 只分配虚拟内存，**第一次访问时才触发缺页中断分配物理内存**

## malloc vs new

[整合自 3.进阶/11.mallocnew.md]

| 特性 | malloc / free | new / delete |
|------|--------------|-------------|
| 本质 | C 库函数 | C++ 运算符 |
| 存储位置 | **堆** | **堆**（或 placement new 指定位置） |
| 分配内存 | 只分配原始内存 | 分配内存 + 调用构造函数 |
| 返回值 | `void*`（需强制转换） | 具体类型指针 |
| 大小 | 手动计算 `sizeof` | 编译器自动计算 |
| 失败时 | 返回 NULL | 抛出 `bad_alloc` 异常 |
| 数组 | `malloc(sizeof(T)*n)` | `new T[n]` / `delete[]` |
| 扩容 | `realloc`（可能原地扩或分配新堆内存） | 无直接支持 |
| 释放 | `free` | `delete` / `delete[]`（先析构再释放） |

**new 的三个步骤：** `operator new` 分配内存 → 调用构造函数 → 返回指针
**delete 的两个步骤：** 调用析构函数 → `operator delete` 释放内存

## 内存泄漏

[整合自 3.进阶/12.内存泄漏.md]

**什么是内存泄漏？** 分配了动态内存但没有释放，长期运行最终无内存可用。

**检测方法：**
- Valgrind（Linux）：`valgrind --leak-check=yes ./program`
- AddressSanitizer：编译加 `-fsanitize=address`
- Visual Studio CRT：`_CrtDumpMemoryLeaks()`

**避免方法：** 使用智能指针、RAII、确保 new/delete 配对。

## 野指针与空悬指针

[整合自 3.进阶/13.野指针.md]

| 类型 | 成因 | 示例 |
|------|------|------|
| **野指针** | 未初始化 | `int* p; *p = 5;` |
| **空悬指针** | 指向已释放内存 | `delete p; *p = 5;` |

**预防：** 初始化为 `nullptr`；释放后置 `nullptr`；使用前检查。

## 常见内存错误

[整合自 3.进阶/14.内存错误.md]

1. **间接引用坏指针**：`scanf("%d", val)` 而不是 `scanf("%d", &val)`
2. **读未初始化内存**：假设堆内存被初始化为零（实际不会）
3. **栈缓冲区溢出**：`gets(buf)` 不检查输入大小
4. **误解指针运算**：`p += sizeof(int)` 实际跳过了 `sizeof(int)` 个元素
5. **引用不存在的变量**：返回局部变量的地址/引用
6. **内存泄漏**：忘记释放已分配的内存

---

# 13 指针详解

> 指针是 C/C++ 的灵魂。理解指针的关键在于理解内存。

[整合自 3.进阶/5.深入理解指针.md、3.进阶/4.快速搞懂C++指针声明.md、3.进阶/3.指针和引用的区别.md]

## 指针的本质

指针就是一个**存储内存地址的变量**。

```cpp
int a = 10;
int* pa = &a;  // pa 存储的是 a 的地址

// pa 本身也是一个变量，也有自己的地址：
int** ppa = &pa;  // ppa 存储的是 pa 的地址（二级指针）
```

### 指针本身的存储位置 vs 指向内容的存储位置

```cpp
// 情况1：指针在栈，指向栈
int a = 10;          // a 在栈
int* p1 = &a;        // p1 在栈，指向栈上的 a

// 情况2：指针在栈，指向堆
int* p2 = new int(42);  // p2 在栈，*p2 在堆

// 情况3：指针在静态区，指向堆
static int* p3 = new int(100);  // p3 在.data，*p3 在堆

// 情况4：指针在栈，指向静态区
static int s = 5;
int* p4 = &s;          // p4 在栈，指向.data区的 s

// 情况5：指针在堆（二级指针的中间层可以在堆）
int** pp = new int*(new int(42));  // pp在栈, *pp在堆, **pp在堆
```

**关键：指针本身和它指向的内容是两个独立的概念，可以分别在栈/堆/静态区。**

## 为什么需要指针？

1. **跨函数修改变量**：值传递只是副本，指针传递可以修改原变量
2. **动态内存管理**：new/malloc 返回的是指针
3. **避免大对象拷贝**：传指针只拷贝 8 字节
4. **多态**：通过基类指针指向派生类对象

## 解引用和指针类型

```cpp
int a = 10;
int* pa = &a;
*pa = 20;  // 解引用：通过地址访问变量

// 指针类型的作用：告诉编译器该取几个字节、如何解释
// int*    → 取 4 字节，按 int 解释
// char*   → 取 1 字节，按 char 解释
// double* → 取 8 字节，按 double 解释
```

**关键理解：**

```cpp
float f = 1.0f;
short c = *(short*)&f;  // 取 f 的前 2 字节，按 short 解释 → 不是 1！

short c2 = 1;
float f2 = *(float*)&c2;  // 取 4 字节（short 只有 2 字节）→ 越界访问！
```

位模式没有改变，变的只是**解释这些位的方式**。

## 多级指针

多级指针只是逻辑概念。实际上所有内存都只存两种东西：**实际内容** 或 **其他变量的地址**。

```
快递柜类比：
  07号柜：书（实际内容）          ← 变量
  05号柜：纸条写"书在07号"        ← 一级指针
  03号柜：纸条写"纸条在05号"      ← 二级指针
```

## 数组与指针

```cpp
int arr[10] = {1, 2, 3};
int* p = arr;

// 以下等价：
arr[0]  == *arr     == p[0]   == *p
arr[1]  == *(arr+1) == p[1]   == *(p+1)
// arr[i] = *(arr + i) = address + sizeof(element) * offset
```

**但数组名 ≠ 指针：**

```cpp
sizeof(arr);  // 40（整个数组的大小）
sizeof(p);    // 8（指针本身的大小）

// arr 不能改变指向
// arr = new_addr;  // 编译错误！
p = new_addr;       // 可以
```

## 数组做参数退化为指针

```cpp
void func(char array[]) {
    sizeof(array);  // = 8（退化为 char*）！
}

// 如果非要传数组大小，用模板：
template<typename T, size_t N>
void func(const T (&arr)[N]) {
    sizeof(arr);  // 正确！编译期推导出 N
}
```

[整合自 1.基础/3.数组做参数退化为指针.md]

## 指针声明阅读（右左法则）

从变量名开始，先向右看，再向左看。括号改变优先级。

```cpp
int* (*p(int))[3];
// p 是一个函数（参数 int）
//    返回一个指针
//      指向一个数组 [3]
//        数组元素是 int*
```

[整合自 3.进阶/4.快速搞懂C++指针声明.md]

**建议：复杂声明用 typedef 拆分。**

## void 指针

`void*` 是通用指针，可以存放任意类型的地址。但不能解引用（编译器不知道取几个字节）。

```cpp
void* pv = &a;
*pv = 10;  // 编译错误！void 没有大小信息
// 必须先转换回具体类型：
*(int*)pv = 10;  // OK
```

## 指针和引用的区别

[整合自 3.进阶/3.指针和引用的区别.md]

| 特性 | 指针 | 引用 |
|------|------|------|
| 本质 | 存储地址的变量 | 别名（语法糖） |
| 可为空 | `nullptr` | 必须绑定变量 |
| 可重绑 | 可以 | 不可以 |
| 解引用 | 需要 `*` | 直接使用 |
| 底层实现 | 指针 | **编译器当作 const 指针** |

**从汇编角度看，引用和指针完全一样。** 引用是 C++ 的语法糖——编译器自动完成取地址和解引用。

### 什么叫指针可以"重绑"？

**重绑 = 让指针指向另一个不同的变量。** 就是这个：

```cpp
int a = 10;
int b = 20;

int* p = &a;   // p 指向 a
p = &b;        // p 改为指向 b  ← 这就是"重绑"
```

**指针可以，引用不行：**

```cpp
int a = 10;
int b = 20;

int& ref = a;   // ref 是 a 的别名
ref = b;        // 这不是重绑！这是把 b 的值赋给 a！
// ref 还是引用 a，a 的值变成了 20

// 为什么叫"重绑"？
// 指针 = 一根可以重新系的绳子，想系谁系谁
// 引用 = 纹身，刻上去就改不了了
```

> **指针 = 存储地址的变量，地址本来就是可以改的值。引用 = 别名，从一而终。**

## 值传递、指针传递、引用传递

[整合自 3.进阶/6.传递.md]

```cpp
void byValue(int a)    { a = 5; }   // 拷贝，不影响实参
void byPointer(int* a) { *a = 5; }  // 拷贝指针，通过解引用修改
void byRef(int& a)     { a = 5; }   // 引用，直接修改

int x = 10;
byValue(x);    // x = 10（不变）
byPointer(&x); // x = 5
byRef(x);      // x = 5
```

**本质：一切参数传递都是拷贝栈上的值。** 区别在于拷贝的是值本身，还是地址的值。

---

# 14 const 关键字

[整合自 1.基础/4.const.md]

## const 用法汇总

| 用法 | 含义 | 示例 |
|------|------|------|
| `const int a` | 只读变量 | `const int a = 10;` |
| `const int* p` | 指向只读变量的指针 | `*p = 5;` ❌ `p = &b;` ✓ |
| `int* const p` | 只读指针 | `*p = 5;` ✓ `p = &b;` ❌ |
| `const int* const p` | 只读指针指向只读变量 | 都不能改 |
| `const int& r` | 常量引用 | `r = 5;` ❌ |
| `void func() const` | 成员函数不修改对象 | const 对象可调用 |

## const_cast 陷阱

```cpp
const int a = 10;
const int* p = &a;
int* q = const_cast<int*>(p);
*q = 20;
cout << a;  // 输出 10！编译器优化，直接替换为常量
```

编译器可能做了优化，代码中的 `a` 直接被替换为常量值。**不要用 const_cast 去掉 const 限制。**

---

# 15 类型转换

[整合自 1.基础/5.类型转换.md]

C++ 四大类型转换：

| 转换 | 用途 | 运行时检查 |
|------|------|-----------|
| `static_cast` | 基本类型/父子类指针转换 | 无（编译期） |
| `dynamic_cast` | 安全的向下转型 | **有**（RTTI） |
| `const_cast` | 去掉/添加 const | 无 |
| `reinterpret_cast` | 重新解释位模式（最危险）| 无 |

## static_cast

```cpp
double d = static_cast<double>(42);  // 基本类型转换
Derived* d = static_cast<Derived*>(base_ptr);  // 不检查，自己保证安全
```

## dynamic_cast

```cpp
Animal* animal = new Dog();
Dog* dog = dynamic_cast<Dog*>(animal);  // 安全！运行时检查
if (dog) { dog->bark(); }

Cat* cat = dynamic_cast<Cat*>(animal);  // 返回 nullptr（animal 实际是 Dog）
```

**要求基类有虚函数**（需要 vtable 中的 RTTI 信息做运行时类型检查）。

## const_cast

```cpp
const int a = 42;
int* p = const_cast<int*>(&a);  // 去掉 const
// 不推荐！可能导致未定义行为
```

## reinterpret_cast

```cpp
int a = 42;
char* p = reinterpret_cast<char*>(&a);  // 重新解释位模式
// 最危险！不检查，仅重新解释底层比特
```

---

# 16 sizeof 和 strlen

[整合自 1.基础/2.sizeof和strlen.md]

| 特性 | sizeof | strlen |
|------|--------|--------|
| 本质 | 编译期**运算符** | 运行时**函数** |
| 参数 | 类型或变量 | `const char*` |
| 计算 | 类型所占字节数 | 到 `\0` 为止的字符数 |
| 包含 `\0` | 包含 | 不包含 |

```cpp
char str[] = "Hello World";
sizeof(str);  // 12（11个字符 + \0）
strlen(str);  // 11（不包含 \0）

const char* p = str;
sizeof(p);    // 8（指针大小）
strlen(p);    // 11
```

# 17 static 关键字

> static 在 C++ 中有三种完全不同的用途。理解它们的关键是搞清楚**存储位置**和**作用域**的变化。

## 三种 static，一张图区分

```
                    static 修饰什么？
                   /       |       \
                  /        |        \
           局部变量      全局变量     类成员
              │            │           │
              ▼            ▼           ▼
         "只初始化一次"  "仅本文件可见"  "属于类，不属于对象"
         存在全局区      存在全局区     "所有对象共享一份"
         生命周期=程序   生命周期=程序   存在全局区
```

## 一、static 修饰局部变量——持久化

```cpp
void counter() {
    static int count = 0;  // 只初始化一次！后续调用跳过这行
    count++;
    cout << count << endl;
}

counter();  // 1
counter();  // 2（count 的值"记住"了！）
counter();  // 3
```

| | 普通局部变量 | static 局部变量 |
|---|---|---|
| **存储位置** | 栈 | **全局区** (.data) |
| **生命周期** | 离开 `{}` 销毁 | **程序启动→结束** |
| **初始化次数** | 每次进入函数都初始化 | **只初始化一次** |
| **作用域** | 函数内部可见 | 函数内部可见（不变） |

> **作用域没变（只能在本函数内访问），但生命周期变了（从"跟函数走"变成"跟程序走"）。**

## 二、static 修饰全局变量/函数——隐藏

```cpp
// file_a.cpp
static int internalVar = 100;     // 只能在本 .cpp 内访问
static void helper() { ... }      // 只能在本 .cpp 内调用

// file_b.cpp
extern int internalVar;  // ❌ 链接错误！找不到
helper();                // ❌ 链接错误！找不到
```

| | 普通全局变量 | static 全局变量 |
|---|---|---|
| **存储位置** | 全局区 | 全局区（相同） |
| **生命周期** | 程序启动→结束 | 程序启动→结束（相同） |
| **作用域** | **整个程序**（其他 .cpp 用 extern 可见） | **仅本文件**（.cpp 内部） |

> **生命周期没变，但可见范围变了——从"全程序可见"变成"仅本文件可见"。这就是"隐藏"。**

## 三、static 修饰类成员——共享

这是面试问得最多的。

### 3.1 static 成员变量：属于类，不属于对象

```cpp
class Player {
public:
    static int totalCount;  // 声明：属于 Player 这个类
    //     ↑ 注意：这里只是声明，不是定义！必须类外定义
    string name;
};

// 必须在类外定义（分配内存）
int Player::totalCount = 0;  // 存储在 .data 全局区
```

**存储位置和生命周期：**

```
Player 对象1（栈）          Player 对象2（堆）
┌──────────────┐          ┌──────────────┐
│ name         │          │ name         │
└──────────────┘          └──────────────┘
     没有 totalCount！        没有 totalCount！

                        全局区 (.data)
                   ┌──────────────────┐
                   │ Player::totalCount│ ← 只有一份，所有对象共享
                   │       = 5         │    程序启动时分配，程序结束时回收
                   └──────────────────┘
```

> **static 成员变量不在任何对象内部，它在全局区（.data/.bss），整个类只有一份，所有对象共享。**

### 3.2 怎么访问 static 成员变量？

```cpp
// 方式1：通过类名（推荐，因为属于类）
Player::totalCount = 10;

// 方式2：通过对象（也行，但不推荐——容易让人误会它在对象里）
Player p1;
p1.totalCount = 10;  // 实际上修改的还是 Player::totalCount

// 方式3：通过指针
Player* p2 = new Player();
p2->totalCount = 10;  // 同上
// 即使 p2 是 nullptr，p2->totalCount 在某些编译器上也能工作！
// 因为 static 成员不经过对象内存，编译器直接替换为 Player::totalCount
```

> **不管用哪种方式访问，改的都是全局区那一份。**

### 3.3 static 成员方法和普通成员方法的区别

```cpp
class Player {
public:
    string name;

    // 普通成员方法 → 有 this 指针
    void SetName(const string& n) {
        name = n;             // ✓ 访问成员变量（其实是 this->name）
        totalCount++;          // ✓ 访问静态成员（不需要 this）
    }

    // static 成员方法 → 没有 this 指针
    static int GetTotalCount() {
        return totalCount;    // ✓ 访问静态成员
        // name = "test";     // ❌ 编译错误！没有 this，不知道是谁的 name
    }

    // 想访问实例成员？只能手动传对象
    static void PrintName(Player& p) {
        cout << p.name;       // ✓ 通过参数间接访问
    }

private:
    static int totalCount;
};
```

| | 普通成员方法 | static 成员方法 |
|---|---|---|
| **this 指针** | 有（隐式传入当前对象地址） | **没有** |
| **访问实例成员** | ✓ 可以（通过 this） | ❌ 不能直接访问 |
| **访问 static 成员** | ✓ 可以 | ✓ 可以 |
| **调用方式** | 必须通过对象 `obj.method()` | 可通过类名 `Player::method()` |
| **存储位置** | 代码段 (.text) | 代码段 (.text)（相同） |

### 通过实例化对象间接访问实例成员

```cpp
class Player {
public:
    static void DoSomething() {
        // 在 static 方法里 new 一个对象，就能访问它的实例成员了
        Player temp;
        temp.name = "test";   // ✓ 访问 temp 的实例成员
        cout << temp.name;    // ✓
    }
private:
    string name;
};
```

> **static 方法里不是"不能访问实例成员"，而是"没有隐式的 this，不知道访问的是谁"。你自己 new 一个对象出来，明确告诉它是哪个对象，就可以访问。**

## static 总结口诀

```
局部 static：   生命周期变"永久"，作用域不变
全局 static：   作用域变"本文件"，生命周期不变
类 static：     从"属于对象"变成"属于类"，存在全局区，所有对象共享
静态方法：      没有 this，只能直接访问静态成员
```
> - 《STL 源码剖析》（侯捷）
> - 《深度探索C++对象模型》（Lippman）
> - 《Effective C++》（Scott Meyers）
> - 《CSAPP》（深入理解计算机系统）
> - SGI STL / GCC libstdc++ 源码
> - Unreal Engine 源码
> - Unity DOTS 文档
