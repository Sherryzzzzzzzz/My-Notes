# 深入剖析STL内存分配器allocator及其萃取器

![深入剖析STL内存分配器allocator及其萃取器](https://pic1.zhimg.com/70/v2-3db8987473102bc101e4824ce7145ba3_1440w.image?source=172ae18b&biz_tag=Post)

本期主要讲解C++ STL中的内存分配器`std::allocator`及其特性萃取器`__gnu_cxx::__alloc_traits`。

> 💡 **注释：什么是内存分配器？**
>
> 在C++中，我们通常用`new`和`delete`来管理内存。但STL容器（如vector、list）需要更灵活的内存管理方式：
> - 容器需要**分离"分配内存"和"构造对象"**两个步骤（new把这两步合在一起了）
> - 容器需要**为不同类型分配内存**（比如list既要分配元素，又要分配链表节点）
> - 用户可能想用**自定义的内存池**而不是默认的malloc
>
> allocator就是为了解决这些问题而设计的"内存管家"。它把内存操作拆分成4个独立步骤：分配、构造、析构、释放。

> 💡 **注释：什么是traits（萃取器）？**
>
> traits是一种**模板元编程技术**，用于在编译期"提取"类型的各种属性。
>
> 类比理解：traits就像是类型的"体检报告"，可以告诉你：
> - 这个类型有没有某个成员函数？
> - 这个类型的指针是什么类型？
> - 如何从 allocator<int> 得到 allocator<Node>？
>
> allocator_traits就是专门用来提取allocator属性的工具。

为防止混淆，规定如下：

- `allocator`：泛指内存分配器，仅仅是一个术语。
- `std::allocator`：是STL实现的内存分配器类`std::allocator`。

> 📊 **整体结构图：**
>
> ```
> 继承层次：
> ┌─────────────────────────────────────────┐
> │  __gnu_cxx::new_allocator<_Tp>          │ ← 底层实现，封装new/delete
> │     - allocate()   分配内存              │
> │     - construct()  调用构造函数           │
> │     - destroy()    调用析构函数           │
> │     - deallocate() 释放内存              │
> └─────────────────────────────────────────┘
>              ↑ 继承
> ┌─────────────────────────────────────────┐
> │  std::allocator<_Tp>                    │ ← 标准库提供的分配器
> │     (大部分功能继承自new_allocator)       │
> └─────────────────────────────────────────┘
>
> 萃取器层次：
> ┌─────────────────────────────────────────┐
> │  std::__allocator_traits_base           │ ← 基础萃取工具
> │     - __rebind<>  类型转换              │
> └─────────────────────────────────────────┘
>              ↑ 继承
> ┌─────────────────────────────────────────┐
> │  std::allocator_traits<_Alloc>          │ ← 标准萃取器
> └─────────────────────────────────────────┘
>              ↑ 继承
> ┌─────────────────────────────────────────┐
> │  __gnu_cxx::__alloc_traits<_Alloc>      │ ← GNU扩展萃取器
> └─────────────────────────────────────────┘
> ```

### [__gnu_cxx::new_allocator](https://zhida.zhihu.com/search?content_id=166839255&content_type=Article&match_order=1&q=__gnu_cxx%3A%3Anew_allocator&zhida_source=entity)

C++的默认的内存分配器`std::allocator`，继承至`__gnu_cxx::new_allocator`。而 `__gnu_cxx::new_allocator` 主要完成两个任务：

- 分配对象内存、初始化对象
- 析构对象、释放对象内存

`__gnu_cxx::new_allocator` 是个空类，没有成员变量，主要有四种成员函数完成上述任务：

- `allocate` 函数，用于分配内存
- `construct`函数，调用已分配内存对象的构造函数
- `destroy`函数，调用析构函数
- `deallocate`函数，用于释放内存

`__gnu_cxx::new_allocator` 的整体框架大致如下：

```cpp
template <typename _Tp>
  class new_allocator
  {
  public:
    typedef size_t      size_type;
    typedef ptrdiff_t   difference_type;
    typedef _Tp*        pointer;
    typedef const _Tp*  const_pointer;
    typedef _Tp &       reference;
    typedef const _Tp & const_reference;
    typedef _Tp         value_type;

    template <typename _Tp1>
    struct rebind
    {
      typedef new_allocator<_Tp1> other;
    };

    new_allocator() _GLIBCXX_USE_NOEXCEPT {}
    new_allocator(const new_allocator &) noexcept {}
    template <typename _Tp1>
    new_allocator(const new_allocator<_Tp1> &) noexcept {}
    ~new_allocator() noexcept {}

    pointer allocate(size_type __n, const void * = static_cast<const void *>(0));

    void deallocate(pointer __p, size_type);

    size_type max_size() const noexcept;

    template <typename _Up, typename... _Args>
    void construct(_Up *__p, _Args &&...__args) 
                    noexcept(noexcept(::new ((void *)__p)_Up(std::forward<_Args>(__args)...)));
    template <typename _Up>
    void destroy(_Up *__p) noexcept(noexcept(__p->~_Up()));
    //...
  };
```

### allocate

`allocate`函数，用于分配大小为`__n`个字节内存，返回值是分配所得内存的地址。

- 如果待分配内存大小`__n`大于当前进程最大可用内存，那么就会抛出`bad_alloc`异常。
- 再调用`operator new`来分配内存。`operator new`对`malloc`作了一层简单的封装，等效于`malloc`
- 将指向`operator new`的返回值类型转换为此次分配对象`_Tp`的指针类型。

整个过程如下：

```cpp
pointer allocate(size_type __n, const void * = static_cast<const void *>(0))
    {
      if (__n > this->max_size())
        std::__throw_bad_alloc();

#if __cpp_aligned_new
      if (alignof(_Tp) > __STDCPP_DEFAULT_NEW_ALIGNMENT__)
      {
        std::align_val_t __al = std::align_val_t(alignof(_Tp));
        return static_cast<_Tp *>(::operator new(__n * sizeof(_Tp), __al));
      }
#endif
      return static_cast<_Tp *>(::operator new(__n * sizeof(_Tp)));
    }
```

### deallocate

`deallocate`函数，使用`operator delete`来释放地址`__p`指向的内存。

```cpp
void deallocate(pointer __p, size_type)
    {
#if __cpp_aligned_new
      if (alignof(_Tp) > __STDCPP_DEFAULT_NEW_ALIGNMENT__)
      {
        ::operator delete(__p, std::align_val_t(alignof(_Tp)));
        return;
      }
#endif
      ::operator delete(__p);
    }
```

### construct

上面的`allocate`函数相当于`malloc`函数，只是分配`__n`个字节的内存，但是并未对这片内存进行初始化。对`allocate` 函数分配的内存`__p`进行初始化的任务，交给`construct`函数来完成。

`constuct`函数，使用了`new`表达式的另一种形式，叫做`placement new`，使用方式如下注释：

```cpp
template <typename _Up, typename... _Args>
    void construct(_Up *__p, _Args &&...__args) 
                    noexcept(noexcept(::new ((void *)__p)_Up(std::forward<_Args>(__args)...)))
    {
        // 表示在 地址 _p 上调用对象 _Up的构造函数
        // 其中，__args是构造函数的参数
        ::new ((void *)__p) _Up(std::forward<_Args>(__args)...);
    }
```

### destroy

`deallocate`函数，是完成了释放内存，但是在释放内存之前一般需要先调用对象的析构函数，完成相关的资源的释放、关闭操作。因此在`destoy`函数中，直接调用了类型`_Up`的析构函数。

```cpp
template <typename _Up>
    void destroy(_Up *__p) noexcept(noexcept(__p->~_Up()))
    {
      __p->~_Up();
    }
```

### std::allocator

类`std::allocator` 继承`__gnu_cxx::new_allocator`。

```cpp
template<typename _Tp>
using __allocator_base = __gnu_cxx::new_allocator<_Tp>;


  template <typename _Tp>
  class allocator : public __allocator_base<_Tp>
  {
  public:
    typedef size_t      size_type;
    typedef ptrdiff_t   difference_type;
    typedef _Tp*        pointer;
    typedef const _Tp*  const_pointer;
    typedef _Tp&        reference;
    typedef const _Tp&  const_reference;
    typedef _Tp         value_type;

    template <typename _Tp1>
    struct rebind
    {
      typedef allocator<_Tp1> other;
    };

    allocator() noexcept {}
    allocator(const allocator &__a) noexcept : __allocator_base<_Tp>(__a) {}
    allocator &operator=(const allocator &) = default;

    template <typename _Tp1>
    allocator(const allocator<_Tp1> &) _GLIBCXX_NOTHROW
    { }

    ~allocator() _GLIBCXX_NOTHROW {}

    //...
    // Inherit everything else.
  };
```

### rebind

> 💡 **注释：rebind是本文最难理解的概念，请仔细阅读！**
>
> **问题背景**：假设你创建了一个 `std::list<int>`，你传入的分配器是 `allocator<int>`。
>
> 但是！list内部不仅需要存储int，还需要存储**链表节点**，比如：
> ```cpp
> struct _List_node<int> {
>     int data;           // ← 你的int数据
>     _List_node* next;   // ← 指向下一个节点的指针
>     _List_node* prev;   // ← 指向前一个节点的指针
> };
> ```
>
> **问题来了**：你只有 `allocator<int>`，但你需要分配 `_List_node<int>` 类型的内存！怎么办？
>
> **rebind的作用**：把 `allocator<int>` 变成 `allocator<_List_node<int>>`。
>
> **类比理解**：
> - allocator就像是"模具工厂"
> - allocator<int> 是"生产int模具的工厂"
> - rebind就是告诉工厂："请换一个模具，改成生产_List_node<int>的模具"
> - rebind<_List_node<int>>::other 就得到"生产_List_node<int>模具的工厂"

在`__gnu_cxx::new_allocator`、`std::allocator`中都有一个`rebind`函数，其主要作用：获得类型`_Tp1`的内存分配器`allocator<_Tp1>`。

```cpp
template <typename _Tp1>
    struct rebind
    {
      typedef allocator<_Tp1> other;  // ← other就是新类型的分配器
    };
```

> 💡 **注释：rebind的工作原理**
>
> ```cpp
> // 原始分配器
> std::allocator<int> int_allocator;
>
> // 使用rebind获取节点分配器
> using NodeAllocator = std::allocator<int>::rebind<_List_node<int>>::other;
> // NodeAllocator 等价于 std::allocator<_List_node<int>>
>
> // 现在可以分配节点了
> NodeAllocator node_allocator;
> _List_node<int>* node = node_allocator.allocate(1);  // 分配一个节点
> node_allocator.construct(node, ...);                 // 构造节点
> ```

这个函数在容器中被STL中被广泛使用。比如，在`std::list<_Tp, std::allocator<_Tp>>`中，`std::allocator`不仅要为`_Tp`类型的对象分配内存，还要为存储`_Tp`对象的节点`list_node<_Tp>`分配内存。但是`std::list<_Tp, std::allocator<_Tp>>`的类模板参数中只是传入了用于分配`_Tp`类型的内存分配器`std::allocator<_Tp>`，那么怎么获得`list_node<_Tp>`类型的内存分配器呢？

答案就是依靠`rebind`函数：`allocator<_Tp>::rebind<list_node<_Tp>>::other`，获得的就是用于分配`list_node<_Tp>`类型的内存分配器 `allocator<list_node<_Tp>>`。

在`list`中的实现如下：

```cpp
template<typename _Tp, typename _Alloc>
    class _List_base
    {
    protected:
      // 用于分配 _Tp 类型的内存分配器: _Tp_alloc_type
      // _Tp_alloc_type 实际上就是 std::allocator
      typedef typename __gnu_cxx::__alloc_traits<_Alloc>::template rebind<_Tp>::other _Tp_alloc_type;
      // 用于分配 List_node<_Tp> 类型的内存分配器：_Node_alloc_type
      typedef typename _Tp_alloc_traits::template rebind<_List_node<_Tp> >::other _Node_alloc_type;        
      //...
    };

    template<typename _Tp, typename _Alloc = std::allocator<_Tp> >
    class list : protected _List_base<_Tp, _Alloc>
    {
    protected:
      typedef _List_node<_Tp>                _Node;
      //...
    };
```

### std::__allocator_traits_base

> 💡 **注释：为什么需要traits_base？**
>
> 上面的`rebind`直接写在allocator类里面。但问题是：
> - 如果用户自定义了一个allocator，没有写rebind怎么办？
> - 我们需要一种**通用方法**来判断allocator有没有rebind，如果有就用它，没有就自动生成。
>
> `__allocator_traits_base`就是用来解决这个问题的。它使用**SFINAE**技术（Substitution Failure Is Not An Error，替换失败不是错误）来检测allocator是否实现了rebind。

上面的`list`中使用到了用于萃取内存分配器属性的类`__gnu_cxx::__alloc_traits`。

```cpp
__gnu_cxx::__alloc_traits 继承自 std::allocator_traits
std::allocator_traits     继承自 std::__allocator_traits_base
```

类`__allocator_traits_base`，用于获取内存分配器`_Alloc`的属性，这个分配器`_Alloc`不一定是上面所述的`std::allocator`，可以是自定义的。

```cpp
struct __allocator_traits_base
  {
    // 【版本1】普通版本：当分配器没有rebind时使用
    template <typename _Tp,
              typename _Up,
              typename = void>
    struct __rebind : __replace_first_arg<_Tp, _Up>
    { };

    // 【版本2】特化版本：当分配器有rebind成员时使用
    // __rebind 特化版本：当分配器 _Tp 有成员函数 rebind 时调用此特化版本
    template <typename _Tp, typename _Up>
    struct __rebind<_Tp,
                    _Up,
                    __void_t<typename _Tp::template rebind<_Up>::other>>  // ← SFINAE检测点
    {
      using type = typename _Tp::template rebind<_Up>::other;
    };

  protected:
    template <typename _Tp> using __pointer     = typename _Tp::pointer;
    template <typename _Tp> using __c_pointer   = typename _Tp::const_pointer;
    template <typename _Tp> using __v_pointer   = typename _Tp::void_pointer;
    template <typename _Tp> using __cv_pointer  = typename _Tp::const_void_pointer;
    template <typename _Tp> using __pocca = typename _Tp::propagate_on_container_copy_assignment;
    template <typename _Tp> using __pocma = typename _Tp::propagate_on_container_move_assignment;
    template <typename _Tp> using __pocs  = typename _Tp::propagate_on_container_swap;
    template <typename _Tp> using __equal = typename _Tp::is_always_equal;
  };
```

### __rebind

> 💡 **注释：理解SFINAE如何选择版本**
>
> SFINAE的工作原理：
> 1. 编译器会先尝试匹配**特化版本**（版本2）
> 2. 如果 `typename _Tp::template rebind<_Up>::other` 这个表达式**无效**（比如_Tp没有rebind），编译不会报错，而是"替换失败"
> 3. "替换失败不是错误"，编译器会fallback到**普通版本**（版本1）
>
> ```cpp
> // 情况A：std::allocator<int> 有 rebind 成员
> __allocator_traits_base::__rebind<std::allocator<int>, Node<int>>
> // → 匹配特化版本（版本2），因为 rebind<int>::other 存在
> // → 结果：std::allocator<Node<int>>
>
> // 情况B：MyCustomAllocator 没有 rebind 成员
> __allocator_traits_base::__rebind<MyCustomAllocator<int>, Node<int>>
> // → 特化版本替换失败（因为没有rebind）
> // → fallback到普通版本（版本1）
> // → 通过 __replace_first_arg 自动生成 allocator<Node<int>>
> ```
>
> `__void_t`的作用：如果表达式有效，`__void_t<...>`就是void，第三个参数变成void，匹配特化版本。

类`__allocator_traits_base` 中最重要的是成员函数`__rebind`。 `__rebind`的模板参数`_Tp`是分配器类型，根据`_Tp`来实现重载：

- 当传入的内存分配器类型`_Tp`，实现了`rebind`成员函数时，比如上面的`std::allocator`，那么就调用`__rebind`的特化版本：

```cpp
    template <typename _Tp, typename _Up>
    struct __rebind<_Tp,
                    _Up,
                    __void_t<typename _Tp::template rebind<_Up>::other>>
    {
      using type = typename _Tp::template rebind<_Up>::other;
    };
```

以`std::allocator<int>`为例，获取`Node<int>`类型的内存分配器：

```
cpp __allocator_traits_base::__rebind<std::allocator<int>, Node<int>>::type // 等价于 std::allocator<Node<int>>
```

- 当传入的分配器`_Tp`没有实现`rebind`成员函数时，就调用普通`__rebind`版本：

```cpp
 __allocator_traits_base::__rebind<std::allocator<int>, Node<int>>::type  
  // 等价于 
  std::allocator<Node<int>>
```

其中`__replace_first_arg`实现如下。此时，需要自定义一个内存分配器模板`_Template`，

```cpp
 template <typename _Tp, typename _Up>
    struct __replace_first_arg
    { };

    // _Template 是个类模板
    template <template <typename, typename...> class _Template,   // ← 模板参数模板！
              typename _Up,
              typename _Tp,
              typename... _Types>
    struct __replace_first_arg<_Template<_Tp, _Types...>, _Up>
    {
      using type = _Template<_Up, _Types...>;   // ← 把第一个参数_Tp换成_Up
    };
```

> 💡 **注释：理解模板参数模板（Template Template Parameter）**
>
> 这是C++的高级特性：**模板参数本身也是一个模板**。
>
> ```cpp
> // 普通模板参数：
> template<typename T>          // T是一个具体类型，如int、string
>
> // 模板参数模板：
> template<template<typename> class T>   // T本身是一个模板，如vector、allocator
> ```
>
> **__replace_first_arg的工作原理**：
>
> ```cpp
> // 输入：allocator<int> 和 Node<int>
> // 期望输出：allocator<Node<int>>
>
> __replace_first_arg<allocator<int>, Node<int>>
>
> // 匹配过程：
> // allocator<int> 展开为 allocator<_Tp=int, _Types...=(空)>
> // _Template = allocator（这是个模板名）
> // _Tp = int
> // _Up = Node<int>
>
> // 结果：
> // type = allocator<_Up, _Types...> = allocator<Node<int>>
> ```
>
> 简单说：把 `Allocator<OldType>` 变成 `Allocator<NewType>`，模板名不变，只换第一个参数。

**by the way**

在此，补充点模板的一点知识：

> 💡 **注释：两个关键语法理解**

1. 模板参数模板

在`__replace_first_arg`类中，使用了一个类模板参数模板`_Template`，这表示模板参数`_Template`本身就是个类模板。

```cpp
template <typename, typename...> class _Template
```

> 这行代码的意思：
> - `_Template`不是一个具体类型，而是一个**模板的名字**
> - 它必须接受至少一个类型参数（`typename`）和可能有更多参数（`typename...`）
> - 例如：`allocator`、`vector`、`basic_string`都是这样的模板
> - 当你传入`allocator<int>`时，`_Template`匹配到`allocator`这个模板名

2. `::template`

在`__rebind`函数体中，在`::`后面有个`template`关键字，这是用于告诉编译器 `template`后面的 `<` 不是比较符号，而是模板参数符号。就是类似于`_Tp`前面的`typename`是告诉编译器`::`后面的是类成员函数，而不是`static`函数。

```cpp
using type = typename _Tp::template rebind<_Up>::other;
```

> **为什么需要 ::template？**
>
> 编译器看到 `<` 时，不知道这是"小于号"还是"模板参数起始符"。
>
> ```cpp
> // 错误理解：编译器可能认为这是比较操作
> _Tp::rebind < _Up > ::other
> // 翻译为：(_Tp::rebind < _Up) > ::other
> // 这完全是错的！
>
> // 正确理解：这是模板参数
> _Tp::template rebind<_Up>::other
> // template关键字告诉编译器："后面是模板，<是参数列表开始"
> ```
>
> **类比**：
> - `typename`告诉编译器："`::`后面是类型，不是静态成员"
> - `template`告诉编译器："`::`后面是模板，`<`不是比较符号"

### __alloc_rebind

全局函数`__alloc_rebind`，是`std::__allocator_traits_base`的wrapper，用于获取为`_Up`类型分配内存的内存分配器`_Alloc<_Up>`

```cpp
template <typename _Alloc, typename _Up>
 using __alloc_rebind = typename __allocator_traits_base::template __rebind<_Alloc, _Up>::type;
```

### std::allocator_traits

> 💡 **注释：allocator_traits的作用**
>
> `allocator_traits`是一个"包装器"，它把allocator的各种操作统一封装起来。
>
> **为什么需要它？**
> - 用户可能提供自定义的allocator，可能缺少某些功能
> - allocator_traits会**自动补全**缺失的功能
> - STL容器通过traits来操作allocator，而不是直接操作allocator本身
>
> ```cpp
> // 直接用allocator：
> allocator.allocate(10);         // 如果allocator没有这个方法，编译错误
>
> // 通过traits用allocator：
> allocator_traits<Alloc>::allocate(alloc, 10);  // traits会自动处理缺失的方法
> ```

类`std::allocator_traits`，继承于`std::__allocator_traits_base`，用于获取内存分配器`allocator`的各个属性。

```cpp
template <typename _Alloc>
  struct allocator_traits : __allocator_traits_base
  {
    typedef _Alloc allocator_type;                    /// The allocator type
    typedef typename _Alloc::value_type value_type;   /// The allocated type
    //...
  };
```

> 💡 **注释：特化版本的意义**
>
> 当`_Alloc`是`std::allocator`时，编译器会使用这个**特化版本**。
>
> 为什么？因为`std::allocator`是STL默认的分配器，使用最频繁，特化版本可以：
> 1. 简化编译（不需要复杂的traits逻辑）
> 2. 提高编译速度
> 3. 直接给出明确的类型定义

当`_Alloc`是`std::allocator`时，有个特化版本：

```cpp
template <typename _Tp>
  struct allocator_traits<allocator<_Tp>>
  {
    using allocator_type = allocator<_Tp>;  // 分配器类型
    using value_type = _Tp;                 // 待分配内存的对象类型
    using pointer = _Tp *;                 // 对象指针
    using const_pointer = const _Tp *;
    //...  using 
    using is_always_equal = true_type;

    // 使用allocator为_Up分配内存
    template <typename _Up>
    using rebind_alloc = allocator<_Up>;

    template <typename _Up>
    using rebind_traits = allocator_traits<allocator<_Up>>;

    // 下面是 std::allocator<_Tp> 成员函数的 wrapper
    static pointer allocate(allocator_type &__a, size_type __n)
    {
      return __a.allocate(__n);
    }

    static pointer allocate(allocator_type &__a, size_type __n, const_void_pointer __hint)
    {
      return __a.allocate(__n, __hint);
    }

    static void deallocate(allocator_type &__a, pointer __p, size_type __n)
    {
      __a.deallocate(__p, __n);
    }

    template <typename _Up, typename... _Args>
    static void construct(allocator_type &__a, _Up *__p, _Args &&...__args) 
                noexcept(noexcept(__a.construct(__p, std::forward<_Args>(__args)...)))
    {
      __a.construct(__p, std::forward<_Args>(__args)...);
    }

    template <typename _Up>
    static void destroy(allocator_type &__a, _Up *__p) noexcept(noexcept(__a.destroy(__p)))
    {
      __a.destroy(__p);
    }

    static size_type max_size(const allocator_type &__a) noexcept
    {
      return __a.max_size();
    }
  };
```

### **gnu_cxx::**alloc_traits

`__gnu_cxx::__alloc_traits`类，也大都是`std::allocator_traits`的wrapper，

```cpp
template<typename _Alloc, typename = typename _Alloc::value_type>
struct __alloc_traits : std::allocator_traits<_Alloc> { 

    typedef _Alloc                               allocator_type;
    typedef std::allocator_traits<_Alloc>        _Base_type;

    typedef typename _Base_type::value_type      value_type;
    typedef typename _Base_type::pointer         pointer;
    typedef typename _Base_type::const_pointer   const_pointer;
    typedef typename _Base_type::size_type       size_type;
    typedef typename _Base_type::difference_type difference_type;

    typedef value_type &                         reference;
    typedef const value_type&                    const_reference;

    using _Base_type::allocate;
    using _Base_type::construct;
    using _Base_type::deallocate;
    using _Base_type::destroy;
    using _Base_type::max_size;

  private:

    // 当 _Ptr 不是个标准指针，但是 _Ptr 和 value_type* 相同
    // __is_custom_pointer 才是 true，即 _Ptr 是个自定义指针
    // 即 _Ptr 可转换为 pointer
    template <typename _Ptr>
    using __is_custom_pointer 
        = std::__and_<std::is_same<pointer, _Ptr>, std::__not_<std::is_pointer<_Ptr>>>;

  public:
    // overload construct for non-standard pointer types
    // 重载非标准类型的指针，调用构造函数
    template <typename _Ptr, typename... _Args>
    static typename std::enable_if<__is_custom_pointer<_Ptr>::value>::type
    construct(_Alloc &__a, _Ptr __p, _Args &&...__args) noexcept(...) // 省略了noexcept中的表达式
    {
      // 使用分配器 __a , 在地址 __p 调用构造函数
      _Base_type::construct(__a, 
                            std::__to_address(__p),
                            std::forward<_Args>(__args)...);
    }

    // overload destroy for non-standard pointer types
    // 重载非标准类型指针，调用析构函数
    template <typename _Ptr>
    static typename std::enable_if<__is_custom_pointer<_Ptr>::value>::type
    destroy(_Alloc &__a, _Ptr __p) noexcept(...) 
    {
      _Base_type::destroy(__a, std::__to_address(__p));
    }

    /*** 对于标准的指针，会直接调用父类的constuct、destroy ***/

    // wrapper
    template <typename _Tp>
    struct rebind
    {
      typedef typename _Base_type::template rebind_alloc<_Tp> other;
    };
    //...
}
```

总体来说，`__gnu_cxx::__alloc_traits`提供了一个顶层的内存分配器萃取器，可以使用 `_Alloc` 的 `allocate`、 `deallocate`、`construct` 以及 `destroy`等函数来完成对象构造和析构等任务。

而类`std::allocator_traits` 是底层直接获取内存分配器`_Alloc`属性的类，其中`std::allocator_traits`有个特化版本，即使`_Alloc`是`std::allocator`，因为`std::allocator`是STL的容器默认的内存分配器。

如果想将自定义的内存分配器`Custome_Alloc`融入到STL体系中，那么也需要像`std::allocator`一样完成相应的接口设计、以及`rebind`函数。这样，容器就能通过`__gnu_cxx::__alloc_traits<Custome_Alloc>`使用自定义的内存分配器`Custome_Alloc`。

好嘞，到此完成了本期的目标，即讲解完毕C++ STL内存分配器的设计

---

> 📝 **总结：核心知识点回顾**
>
> | 概念 | 作用 | 难点 |
> |------|------|------|
> | `allocator` | 内存管理器，分离分配和构造 | 理解为什么需要分离 |
> | `rebind` | 把`allocator<T>`变成`allocator<U>` | 理解为什么容器需要它 |
> | `traits` | 提取allocator属性，自动补全缺失功能 | 理解SFINAE如何选择版本 |
> | `::template` | 告诉编译器后面是模板参数 | 理解`<`的歧义性 |
> | 模板参数模板 | 模板参数本身是模板 | 理解模板可以作为参数传递 |
>
> **记忆口诀**：
> - **allocator四步走**：分配→构造→析构→释放
> - **rebind换模具**：同一个分配器，换一种类型
> - **traits查户口**：编译期检查有没有某个成员
> - **template关键**：消除`<`的歧义