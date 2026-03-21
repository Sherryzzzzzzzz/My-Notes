# 深入剖析STL内存分配器allocator及其萃取器

![深入剖析STL内存分配器allocator及其萃取器](https://pic1.zhimg.com/70/v2-3db8987473102bc101e4824ce7145ba3_1440w.image?source=172ae18b&biz_tag=Post)

本期主要讲解C++ STL中的内存分配器`std::allocator`及其特性萃取器`__gnu_cxx::__alloc_traits`。

为防止混淆，规定如下：

- `allocator`：泛指内存分配器，仅仅是一个术语。
- `std::allocator`：是STL实现的内存分配器类`std::allocator`。

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

在`__gnu_cxx::new_allocator`、`std::allocator`中都有一个`rebind`函数，其主要作用：获得类型`_Tp1`的内存分配器`allocator<_Tp1>`。

```cpp
template <typename _Tp1>
    struct rebind
    {
      typedef allocator<_Tp1> other;
    };
```

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

上面的`list`中使用到了用于萃取内存分配器属性的类`__gnu_cxx::__alloc_traits`。

```cpp
__gnu_cxx::__alloc_traits 继承自 std::allocator_traits
std::allocator_traits     继承自 std::__allocator_traits_base
```

类`__allocator_traits_base`，用于获取内存分配器`_Alloc`的属性，这个分配器`_Alloc`不一定是上面所述的`std::allocator`，可以是自定义的。

```cpp
struct __allocator_traits_base
  {
    template <typename _Tp,
              typename _Up,
              typename = void>
    struct __rebind : __replace_first_arg<_Tp, _Up>
    { };  

    // __rebind 特化版本：当分配器 _Tp 有成员函数 rebind 时调用此特化版本
    template <typename _Tp, typename _Up>
    struct __rebind<_Tp,
                    _Up,
                    __void_t<typename _Tp::template rebind<_Up>::other>>
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
    template <template <typename, typename...> class _Template, 
              typename _Up,
              typename _Tp, 
              typename... _Types>
    struct __replace_first_arg<_Template<_Tp, _Types...>, _Up>
    {
      using type = _Template<_Up, _Types...>;
    };
```

**by the way**

在此，补充点模板的一点知识：

1. 模板参数模板

在`__replace_first_arg`类中，使用了一个类模板参数模板`_Template`，这表示模板参数`_Template`本身就是个类模板。

```
cpp template <typename, typename...> class _Template
```

2. `::template`

在`__rebind`函数体中，在`::`后面有个`template`关键字，这是用于告诉编译器 `template`后面的 `<` 不是比较符号，而是模板参数符号。就是类似于`_Tp`前面的`typename`是告诉编译器`::`后面的是类成员函数，而不是`static`函数。

```cpp
using type = typename _Tp::template rebind<_Up>::other; 
```

### __alloc_rebind

全局函数`__alloc_rebind`，是`std::__allocator_traits_base`的wrapper，用于获取为`_Up`类型分配内存的内存分配器`_Alloc<_Up>`

```cpp
template <typename _Alloc, typename _Up>
 using __alloc_rebind = typename __allocator_traits_base::template __rebind<_Alloc, _Up>::type;
```

### std::allocator_traits

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