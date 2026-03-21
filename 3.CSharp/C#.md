**一、基础知识**

**1、[值类型](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=值类型&zhida_source=entity)与[引用类型](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=引用类型&zhida_source=entity)**

值类型：struct、enum、int、float、char、bool、decimal

引用类型：class、delegate、interface、array、object、string

**2、[装箱](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=装箱&zhida_source=entity)与[拆箱](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=拆箱&zhida_source=entity)**

装箱：把值类型转换成引用类型

拆箱：把引用类型转换成值类型

装箱：对值类型在堆中分配一个对象实例，并将该值复制到新的对象中。

（1）第一步：新分配托管堆内存(大小为值类型实例大小加上一个方法表指针。

（2）第二步：将值类型的实例字段拷贝到新分配的内存中。

（3）第三步：返回托管堆中新分配对象的地址。这个地址就是一个指向对象的引用了。

拆箱：检查对象实例，确保它是给定值类型的一个装箱值。将该值从实例复制到值类型变量中。

在装箱时是不需要显式的类型转换的，不过拆箱需要显式的类型转换。

**3、堆和栈**

存放在栈中时要管存储顺序，保持着先进后出的原则，他是一片连续的内存域，有系统自动分配和维护；

堆是无序的，他是一片不连续的内存域，有用户自己来控制和释放，如果用户自己不释放的话，当内存达到一定的特定值时，通过[垃圾回收器](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=垃圾回收器&zhida_source=entity)(GC)来回收。

栈内存无需我们管理，也不受GC管理。当栈顶元素使用完毕，立马释放。而堆则需要GC清理。

使用引用类型的时候，一般是对指针进行的操作而非引用类型对象本身。但是值类型则操作其本身。

**4、GC（Garbage Collection）**

当程序需要更多的堆空间时，GC需要进行垃圾清理工作，暂停所有线程，找出所有无被引用的对象，进行清理，并通知栈中的指针重新指向地址排序后的对象。

GC只能处理[托管内存](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=托管内存&zhida_source=entity)资源的释放，对于非托管资源则不能使用GC进行回收，必须由程序员手动回收，例如FileStream或SqlConnection需要调用Dispose进行资源的回收。

**5、CLR（Common Language Runtime）**

[公共语言运行库](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=公共语言运行库&zhida_source=entity)，负责资源管理（包括内存分配、程序及加载、异常处理、[线程同步](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=线程同步&zhida_source=entity)、垃圾回收等），并保证应用和底层操作系统的分离。

- 为了取代Java,   2000年的时候微软牵头，联合惠普和intel向国际标准组织提交了一份技术标准规范：[通用语言基础架构](https://zhida.zhihu.com/search?content_id=686570574&content_type=Answer&match_order=1&q=通用语言基础架构&zhida_source=entity)(CLI   ： Common Language Infrastructure )。
- .NET就是微软实现的符合CLI规范的框架， .NET是以[通用语言运行时](https://zhida.zhihu.com/search?content_id=686570574&content_type=Answer&match_order=1&q=通用语言运行时&zhida_source=entity)(CLR: Common Language Runtime)为基础的[虚拟执行系统](https://zhida.zhihu.com/search?content_id=686570574&content_type=Answer&match_order=1&q=虚拟执行系统&zhida_source=entity)(VES: Virtual  Execution System)，除此之外，.NET还包括很多类库。
- C#是微软发布的，符合CLI规范的面向对象的编程语言，C#不能像C/C++那样可以直接编译成机器码，C#先编译成符合CLI规范的[中间语言](https://zhida.zhihu.com/search?content_id=686570574&content_type=Answer&match_order=1&q=中间语言&zhida_source=entity)(IL  : Intermediate Language)，IL和其它资源（如位图、字符串等）一起打包到程序集(dll, exe)中。 当程序集被执行时，先加载到CLR, CLR通过实时编译(JIT: Just In Time)把IL转换成机器码，来让CPU执行。
- 除了C#外，VB及F#也是微软发布的编程语言，他们只是语法有差别。其它的都和C#一样，被编译成IL,  再在CLR上执行被编译成机器码。

**6、[静态构造函数](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=静态构造函数&zhida_source=entity)**

最先被执行的构造函数，且在一个类里只允许有一个无参的静态构造函数

执行顺序：静态变量>静态构造函数>实例变量>实例构造函数

1. [静态构造函数](https://zhida.zhihu.com/search?content_id=165706229&content_type=Article&match_order=3&q=静态构造函数&zhida_source=entity)没有修饰符修饰(public,private),因为静态构造函数不是我们程序员调用的，是由.net 框架在合适的时机调用的。
2. 静态构造函数没有参数，因为框架不可能知道我们需要在函数中添加什么参数，所以规定不能使用参数。
3. 静态构造函数前面必须是static 关键字。如果不加这个关键字，那就是普通的构造函数了。
4. **静态构造函数的调用时机，是在类被实例化或者静态成员被调用的时候进行调用，并且是由.net框架来调用静态构造函数来初始化静态成员变量。**
5. 一个类中只能有一个静态构造函数。
6. 无参数的静态构造函数和无参数的构造函数是可以并存的。
7. **静态构造函数只会被执行一次**。
8. 就像如果没有在类中写构造函数，那么框架会为我们生成一个构造函数，那么**如果我们在类中定义了静态变量，但是又没有定义静态构造函数，那么框架也会帮助我们来生成一个静态构造函数来让框架自身来调用。**

**7、文件I/O**

通过流的方式对文件进行读写操作

（1）FileStream

（2）[StreamReader](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=StreamReader&zhida_source=entity)/[StreamWriter](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=StreamWriter&zhida_source=entity)

**8、[序列化](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=序列化&zhida_source=entity)与[反序列化](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=反序列化&zhida_source=entity)**

序列化：将对象状态转换为可保持或传输的格式的过程。将对象实例的字段及类的名称转换成字节流，然后把字节流写入数据流

反序列化：将流转换为对象。

这两个过程结合起来，可以轻松地存储和传输数据。

**9、线程同步**

（1）方法一：阻塞（调用Sleep()或Join()）

（2）方法二：加互斥锁lock

（3）方法三：信号和句柄（AutoResetEvent/ManualResetEvent，调用Set()和WaitOne()）

![img](https://picx.zhimg.com/80/v2-ed69bae80647ca24756442bbb95a3ee1_720w.webp?source=1def8aca)

有时可能有这样的需求，A脚本中的属性实例化可能需要用到B脚本中的属性，所以在A脚本属性实例化时，必须保证B脚本已经被实例化完毕。当然我们可以通过先挂在A脚本再挂载B脚本来实现。但在实际开发中，用到的脚本中多，很难去记住各个脚本挂载的先后顺序。所以Unity提供了Script Execution Order配置项，来配置多个脚本的执行顺序。

**10、[抽象类](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=抽象类&zhida_source=entity)abstract class与[接口](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=接口&zhida_source=entity)interface的异同**

相同点：

（1）都可以被继承

（2）都不能被实例化

（3）都可以包含方法的声明

不同点：

（1）抽象类被子类继承；接口被类实现

（2）抽象类只能被单个类继承；接口可继承接口，并可多继承接口

（3）抽象基类可以定义字段、属性、方法实现；接口只能定义属性、索引器、事件、和方法声明，不能包含字段

（4）抽象类可以做方法声明，也可做方法实现；接口只能做方法声明

（5）**具体派生类**必须**覆盖(override)**抽象基类的**抽象方法**；派生类必须实现接口的所有方法

（6）抽象类是一个不完整的类，需要进一步细化；接口是一个行为规范

（7）抽象类中的虚方法或抽象方法必须用public修饰；接口中的所有成员默认为public，不能有private修饰符

**11、类class与[结构体](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=结构体&zhida_source=entity)struct的异同**

Class属于引用类型，是分配在内存的堆上的；

Struct属于值类型，是分配在内存的栈上的；不能从另外一个结构或者类继承，本身也不能被继承；没有默认的构造函数，但是可以添加构造函数；可以不使用new 初始化

**12、[using关键字](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=using关键字&zhida_source=entity)的使用场景**

（1）作为指令：用于导入其他命名空间中定义的类型或为命名空间创建别名

（2）作为语句：用于定义一个范围，在此范围的末尾将释放对象

在C++中，`using`关键字是一种语法糖，可以用来简化代码和提高可读性。`using`有多种用法，下面分别介绍。

```
1、using namespace
```

`using namespace`可以将一个命名空间中的所有名称导入到当前作用域中，从而可以直接使用该命名空间中的所有名称，而不必使用作用域解析运算符(::)。

以下是一个使用`using namespace`的示例代码：

```text
#include <iostream>

namespace MyNamespace {
    int x = 42;
}

int main() {
    using namespace MyNamespace;
    std::cout << x << std::endl;
    return 0;
}
 
```

在这个例子中，我们使用了`using namespace MyNamespace`语句将`MyNamespace`命名空间中的所有名称导入到当前作用域中，从而可以直接使用`x`变量，而不必使用`MyNamespace::x`。

需要注意的是，`using namespace`可能会导致命名冲突和名称重定义，因此应谨慎使用。

```
2、using
```

`using`还可以用来定义别名(alias)，例如：

```text
#include <iostream>

using myint = int;

int main() {
    myint x = 42;
    std::cout << x << std::endl;
    return 0;
}
```

在这个例子中，我们使用`using myint = int`语句将`int`类型定义为`myint`类型的别名，从而可以使用`myint`来代替`int`类型。

```
3、using typename
```

在模板编程中，`using typename`可以用来指定模板类型别名(type alias)。

以下是一个使用`using typename`的示例代码：

```text
#include <iostream>
#include <vector>

template<typename T>
using MyVector = std::vector<T>;

int main() {
    MyVector<int> v{1, 2, 3};
    for (auto x : v) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
    return 0;
}
```

在这个例子中，我们使用`using MyVector = std::vector<T>`语句定义了一个名为`MyVector`的模板类型别名，从而可以使用`MyVector<int>`来代替`std::vector<int>`类型。

```
4、using function
```

`using function`可以将一个函数定义为另一个函数的别名。例如：

```text
#include <iostream>

void foo() {
    std::cout << "Hello, world!" << std::endl;
}

using bar = void (*)();

int main() {
    bar b = foo;
    b();
    return 0;
}
```

在这个例子中，我们使用`using bar = void (*)()`语句将函数指针类型定义为`bar`类型的别名，然后将函数`foo()`赋值给`bar`类型的指针变量`b`，最后调用`b()`执行函数`foo()`。

```
5、using template
```

`using template`可以用来定义模板类型别名，例如：

```text
#include <iostream>

template<typename T>
using MyPointer = T*;

int main() {
    MyPointer<int> p = new int(42);
    std::cout << *p << std::endl;
    delete p;
    return 0;
}
```

在这个例子中，我们使用`using MyPointer = T*`语句定义了一个名为`MyPointer`的模板类型别名，从而可以使用`MyPointer<int>`来代替`int*`类型。

```
6、using enum
```

在C++11中，新增了`using enum`语法，用来为枚举类型定义别名，例如：

```text
#include <iostream>

enum class Color {
    RED,
    GREEN,
    BLUE
};

using ColorType = Color;

int main() {
    ColorType c = Color::RED;

    switch (c) {
        case Color::RED:
            std::cout << "Red" << std::endl;
            break;
        case Color::GREEN:
            std::cout << "Green" << std::endl;
            break;
        case Color::BLUE:
            std::cout << "Blue" << std::endl;
            break;
    }

    return 0;
}
```

在这个例子中，我们使用`using ColorType = Color`语句为枚举类型`Color`定义了一个别名`ColorType`，从而可以使用`ColorType`来代替`Color`类型。同时，我们也可以使用`Color::RED`等枚举常量。

```
7、using namespace std
```

最后，需要注意的是，在C++标准库中，经常会使用`using namespace std`语句将`std`命名空间中的所有名称导入到当前作用域中，以便直接使用标准库中的函数、类型和常量等。但是，在实际开发中，不建议在头文件中使用`using namespace std`语句，因为可能会导致名称冲突和代码可读性降低等问题。建议在源文件中使用`using namespace std`语句。



8、`using`别名模板实现函数模板类型推导

```text
#include <iostream>
template <typename T>
using Invoke = typename T::type;

template <typename T>
struct add_pointer {
    using type = T*;
};

template <typename T>
Invoke<add_pointer<T>> foo(T t) {
    return new typename add_pointer<T>::type(t);
}

int main() {
    int x = 42;
    int* p = foo(x);
    std::cout << *p << std::endl;
    delete p;
    return 0;
}
```

在这个例子中，我们使用`using`别名模板`Invoke`定义了一个别名，用于从一个类型模板中提取类型。然后，我们定义了一个结构体模板`add_pointer`，用于将类型添加一个指针。最后，我们定义了一个函数模板`foo`，使用`Invoke`别名模板实现函数模板类型推导，从而可以自动推导出返回值类型。在`main`函数中，我们调用`foo`函数创建一个指向`int`类型的指针，并输出指针所指向的值。

9、`using`别名模板和可变模板参数实现tuple

```text
#include <iostream>

template <typename... Ts>
struct Tuple {
    template <std::size_t N>
    using Type = Invoke<std::tuple_element<N, std::tuple<Ts...>>>;

    template <std::size_t N>
    using ConstType = Invoke<std::tuple_element<N, std::tuple<const Ts...>>>;

    std::tuple<Ts...> data;

    template <typename... Us>
    Tuple(Us&&... args) : data(std::forward<Us>(args)...) {}

    template <std::size_t N>
    Type<N>& get() {
        return std::get<N>(data);
    }

    template <std::size_t N>
    const ConstType<N>& get() const {
        return std::get<N>(data);
    }
};

int main() {
    Tuple<int, std::string> t(42, "hello");
    std::cout << t.get<0>() << ", " << t.get<1>() << std::endl;
    return 0;
}
 
```

在这个例子中，我们使用`using`别名模板实现了一个类模板`Tuple`，用于表示一个元组。我们定义了`Type`和`ConstType`两个别名模板，用于获取元组中第`N`个元素的类型和常量类型。同时，我们使用可变模板参数和`std::forward`实现了一个构造函数，用于构造一个元组对象。最后，我们定义了`get`函数，用于获取元组中的某个元素，并使用`std::get`函数实现。在`main`函数中，我们创建了一个`Tuple<int, std::string>`类型的对象`t`，并输出其中的元素。

**13、[new关键字](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=new关键字&zhida_source=entity)的使用场景**

（1）实例化对象

（2）隐藏父类方法

**14、[委托与事件](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=委托与事件&zhida_source=entity)**

委托可以把一个方法作为参数传入另一个方法，可以理解为指向一个函数的引用；

事件是一种特殊的委托。

**15、重载(overload)与重写(override)的区别**

重载：是方法的名称相同，参数或参数类型不同；重载是面向过程的概念。

重写：是对基类中的虚方法进行重写。重写是面向对象的概念。

**16、return执行顺序**

try{} 里有一个return语句，那么finally{} 里的code在return前执行。

**17、switch(expression)**

其中expression支持任何数据类型，包括null。

**18、[反射](https://zhida.zhihu.com/search?content_id=100690316&content_type=Article&match_order=1&q=反射&zhida_source=entity)Reflection**

动态获取程序集信息。

## 一、[反射](https://zhida.zhihu.com/search?content_id=178590540&content_type=Article&match_order=1&q=反射&zhida_source=entity)(Reflection)

### 1、什么是反射

- 是.NET Framework 的功能，并不是C#语言特有的功能

- 简单理解：

- - 给我一个对象，我能**在不用new操作符的情况下**，也不知道是什么静态类型的情况下，创建一个同类型的对象
  - 还能访问这个对象的各个成员

- 注意：

- - 这相当于**进一步解耦（**如果用new操作符，意味着类型依赖）
  - 反射是托管语言（C#，Java）和原生语言（C，C++）的最大区别之一
  - **单元测试、[依赖注入](https://zhida.zhihu.com/search?content_id=178590540&content_type=Article&match_order=1&q=依赖注入&zhida_source=entity)、泛型编程都是基于反射机制的**
  - 一般情况下，我们在使用反射时并不能感觉到，因为大多数我们使用的都是封装好的反射

### 2、为什么需要反射

- **原因**：

- - 很多时候程序的逻辑并不是在写的时候就能确定，有时需要用户交互时才确定
  - 此时程序已经属于运行状态（Dynamic，动态）
  - 如果要程序员在静态（static）编写时，去枚举用户可能做的操作，会让程序变得十分臃肿，可读性、可维护性都很烂，并且枚举用户可能做的操作这件事是很难实现的
  - 这是我们需要的这种：**以不变应万变**的能力，就是反射机制

### 3、反射优缺点

- **优点**

- - 1、反射提高了程序的灵活性和扩展性。
  - 2、降低耦合性，提高自适应能力。
  - 3、它允许程序创建和控制任何类的对象，无需提前硬编码目标类。

- **缺点**

- - 1、性能问题：反射是去内存中动态拿到对象/类型描述，再用这些描述去创建对象，这个过程是对性能有影响的（使用反射基本上是一种解释操作，用于字段和方法接入时要远慢于直接代码。因此反射机制主要应用在对灵活性和拓展性要求很高的系统框架上，普通程序不建议使用。）
  - 2、使用反射会模糊程序内部逻辑；程序员希望在源代码中看到程序的逻辑，反射却绕过了源代码的技术，因而会带来维护的问题，反射代码比相应的直接代码更复杂。

## 二、依赖注入（DI:DependencyInjection）

### 1、区分DIP（依赖反转原则）和DI（依赖注入）

- 依赖反转原则是一个概念/设计原则
- 依赖注入是在依赖反转概念基础上，结合接口、反射机制形成的应用

### 2、什么是注入

- 最重要的一点：**Container（容器）**

- - 把各种类型、接口放到容器中，就是“注册”

  - - 注册类型的时候还可以设置，创建对象时是每次创建都是一个新对象，还是创建一个单例模式（每次要对象都给你同一个实例）

  - 后边需要创建实例的时候，向容器“要实例”即可

### 3、具体用法

- 参考例1，part2

### 4、注意

- 依赖注入是封装好的反射的一个重要的功能
- 容器也称为[Service Provider](https://zhida.zhihu.com/search?content_id=178590540&content_type=Article&match_order=1&q=Service+Provider&zhida_source=entity)

**//一些补充**

- - 一般情况下，主体程序都会发布包含程序开发接口（API）的程序开发包（SDK）

  - 使用SDK中的API开发插件就会比较容易、高效

  - API里不一定都是接口，也有可能是一组方法，一组类，一组接口

  - 关于“**有依赖注入为什么还需要API的原因**”

  - - 依赖注入的高自由度意味着错误率提高，例如调方法时大小写写错，就无法找到对应的方法再成功调用了。
    - 开发插件过程中，为了避免自由度过高导致的错误，我们需要有一定的约束，就是SDK中的API

## 三、示例汇总

### 例1：反射用途1：依赖注入

[Part1]**反射的原理**

在接口隔离eg1的基础上进行学习

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
using System.Text;
using System.Threading.Tasks;

namespace eg1Part1
{
        class Program
        {
            static void Main(string[] args)
            {
            ITank tank = new MediumTank();
            //=================分割线===========================
            //分割线以下不再用静态类型，而是动态的获取类信息并实例化
            var t = tank.GetType();
            object o = Activator.CreateInstance(t);
            MethodInfo fireMi = t.GetMethod("Fire");
            MethodInfo runMi = t.GetMethod("Run");
            fireMi.Invoke(o,null);
            runMi.Invoke(o,null);
            }
        }

        class Driver
        {
            private IVehicle _vehicle;
            public Driver(IVehicle vehicle)
            {
                _vehicle = vehicle;
            }

            public void Drive()
            {
                _vehicle.Run();
            }

        }
        interface IVehicle
        {
            void Run();
        }

        interface IWeapon
        {
            void Fire();
        }

        class Car : IVehicle
        {
            public void Run()
            {
                Console.WriteLine("Car is running !");
            }
        }
        class Truck : IVehicle
        {
            public void Run()
            {
                Console.WriteLine("Truck is running !");
            }
        }

        interface ITank : IVehicle, IWeapon
        { }

        class LightTank : ITank
        {
            public void Fire()
            {
                Console.WriteLine("[Light]Boom!");
            }

            public void Run()
            {
                Console.WriteLine("[Light]KA!");
            }
        }

        class MediumTank : ITank
        {
            public void Fire()
            {
                Console.WriteLine("[Medium]Boom!Boom!");
            }

            public void Run()
            {
                Console.WriteLine("[Medium]KA!KA!");
            }
        }

        class HeavyTank : ITank
        {
            public void Fire()
            {
                Console.WriteLine("[Heavy]Boom!Boom!Boom!");
            }

            public void Run()
            {
                Console.WriteLine("[Heavy]KA!KA!KA!");
            }
        }

}
```

[Part2]**依赖注入**

- 需要借助依赖注入框架

- - 项目右键---Manage Nuget---搜素DependencyInjection

![img](https://pica.zhimg.com/v2-ffdf13ced0db36771bde3f32243dfa74_1440w.jpg)

- 引用命名空间Microsoft.Extensions.DependencyInjection
- **基本用法**

```csharp
                  //#####基础用法#####
            var sc = new ServiceCollection();
            sc.AddScoped(typeof(ITank),typeof(HeavyTank));
            //上边括号中的含义：接口，类实现了此接口的类
	   //Typeof代表:动态的拿到这个类的信息
            var sp = sc.BuildServiceProvider();
            //====================分割线===============================
            //分割线以上为注册（把东西放进容器），ServiceCollection就是Container
            //分割线以下是从容器中“要对象”（只要能够看到Service Provider的地方都可以这么用）
            ITank tank = sp.GetService<ITank>();
            tank.Fire();
            tank.Run();
		
			注：具体类、接口代码略，已经放了很多遍了
```

- - - 这里的一个好处：当要把HeavyTank改为其他型号的Tank时，只需要改一处（如果是以前的写法的话，有多少个new操作符就要改多少处）

- **进阶用法**

```csharp
            //#####进阶用法#####
            var sc = new ServiceCollection();
            sc.AddScoped(typeof(ITank), typeof(HeavyTank));
            sc.AddScoped(typeof(IVehicle), typeof(Truck));
            sc.AddScoped<Driver>();
            var sp = sc.BuildServiceProvider();
            //==================分割线===================================
            var driver = sp.GetService<Driver>();
            driver.Drive();
			
		注：具体接口、类的代码同略			 	
```

- - Container里继续塞东西进去

  - - 把IVehicle接口类型的Car类型塞进去
    - 把Driver塞进去

  - 我们甚至可以直接和Service Provider“要Driver”

  - - 以前例子的写法是如何创建Driver的实例的：

    - - 先创建驾驶的是什么对象
      - 再把对象传给构造函数里

    - 这里的情况：

    - - 因为注册时，已经把IVehicle、Driver塞进去了
      - 当创建（要）实例的时候，就会去找IVehicle类型的实例，找到直接塞给它的构造器
      - 也就是说，直接一步就搞定了（sp.GetService<Driver>（））

###  例2：反射的用途2：实现更松的耦合（以不变应万变）

- 一般用在插件式编程

- - **插件：不和主程序一起编译，但能和主程序一起工作**

- 需求：

- - 婴儿车生产厂商，婴儿车上有一个数字面板，面板上有不同按钮

  - - 第一排是动物，第二排是数字，按指定的按钮就会发出对应的声音
    - 默认只有几种动物，允许第三方开发插件，添加多种动物，通过USB 插入U盘的方式扩展
    - 主体程序从Animal文件夹中加载插件，调用里边的Voice方法（能接受一个整数参数，表示叫几次）

- 代码：

**主程序：(用来加载插件)**

```csharp
 using System;
using System.IO;
using System.Collections.Generic;
using System.Runtime.Loader;

namespace BabyStroller.app
{
    class Program
    {
        static void Main(string[] args)
        {
            //Console.WriteLine(Environment.CurrentDirectory);  
            //先输出这个路径，找到它，在此处新建一个Animal文件夹用来放插件（dll文件）
            var folder = Path.Combine(Environment.CurrentDirectory, "Animals");
            var files = Directory.GetFiles(folder);
            var animalTypes = new List<Type>();
            foreach (var file in files)
            {
                var assembly = AssemblyLoadContext.Default.LoadFromAssemblyPath(file);
                var types = assembly.GetTypes();
                foreach (var t in types)
                {
                    if (t.GetMethod("Voice") != null)
                    {
                        animalTypes.Add(t);
                    }
                }
            }

            while (true)
            {
                for (int i = 0; i < animalTypes.Count; i++)
                {
                    Console.WriteLine($"{i+1}.{animalTypes[i].Name}");
                }

                Console.WriteLine("========================================");
                Console.WriteLine("Please input an animal's sequence number:");

                int index = int.Parse(Console.ReadLine());
                if (index>animalTypes.Count || index<1)
                {
                    Console.WriteLine("No such an animal,Please try again");
                    continue;
                }

                Console.WriteLine("How many times?");
                int times = int.Parse(Console.ReadLine());
                var t = animalTypes[index - 1];
                var m = t.GetMethod("Voice");
                var o = Activator.CreateInstance(t);
                m.Invoke(o, new object[] { times });
                Console.WriteLine("--------------------------------------------------------------------");
                Console.WriteLine("--------------------------------------------------------------------");

            }
        }
    }
}
```



**插件（库）**

- - - 创建两个类库（.NET CORE）

    - - 库中写4个类，分别代表四种动物，发出对应的声音

![img](https://pic1.zhimg.com/v2-1ccc74b42aba27d8a525596a9ca6b0e8_1440w.jpg)

- - - 两个类库写完之后，Build Solution，找到这个Solution编译后得到的两个dll文件，复制到主程序的插件文件中

![img](https://pic3.zhimg.com/v2-a0a13546825eda6d71742ceee06a4d68_1440w.jpg)




**厂商准备SDK的情况**

- **主体的.app代码如下**

- **.sdk代码中为IAnimal接口和Unfinished标签**

- - //小插曲：在写上边例子的过程中，类库不小心把Voice方法打错了，导致了程序能运行，但是读取不到dll文件里的内容，所以，SDK的标准化还是很有必要的

- 准备一个IAnimal接口，里边有一个Voice方法，所有开发动物类的厂商都得实现这个方法

- **这样就会有以下好处**

- - 这样就可以保证：有Voice方法，
  - 还可以直接在对象创建好后直接转化为接口类型对象（不用Invoke这样的形式调用，而是直接用Voice方法）
  - 对于没有开发完的类，可以直接过滤掉（用attribute）

- 创建sdk的代码（库）

- - 编译完以dll的形式给第三方开发插件，不会给源码
  - 包括一个类、一个接口

![img](https://pic2.zhimg.com/v2-5a2fcd707ba38d4d6ad7dbb3d7329863_1440w.jpg)

![img](https://pic4.zhimg.com/v2-49cb85c7a8cdc6b24c12baf406a243a5_1440w.jpg)

- 写好之后，Build Solution，就能得到dll文件了

![img](https://picx.zhimg.com/v2-56320097acc955de04076b06b934095b_1440w.jpg)

- - 模仿厂商自己使用开发插件的情形

  - - 因为是第一方，直接在项目中给.APP添加.SDK的Reference就行

  - 模仿第三方开发插件的情形

  - - 项目中的两个库都要引用

    - ADD Reference---Browse----找到刚生成的dll文件

    - 每个类加上IAnimal接口即可

    - 对于没有开发完的。用"[Unfinished]"标注即可（**特性**）

    - - 这里我们将Cat用Unfinished标注，理想结果应该显示三个动物，不显示Cat

  - //补充：**Attribute，特性**，其作用：使用反射时，通过反射拿到一个方法/类时，看看它有没有被某个Attribute所修饰，再决定是否调用/保留这个方法/类

  - Build一下

- 再去主体程序更新

- - 将两个类库的dll复制到Animal文件夹中（SDK的dll文件不用复制，因为第一方直接引用了）
  - 不需要之前的方法，只需要判断类型是否是IAnimal接口的实现类型，且没有被[Unfinished]修饰即可
  - 代码如下

```csharp
using System;
using System.IO;
using System.Collections.Generic;
using System.Runtime.Loader;
using System.Linq;
using BabyStroller.sdk;

namespace BabyStroller.app
{
    class Program
    {
        static void Main(string[] args)
        {
            //Console.WriteLine(Environment.CurrentDirectory);  
            //先可以先输出这个路径，找到它然后建一个Animal文件夹
            var folder = Path.Combine(Environment.CurrentDirectory, "Animals");
            var files = Directory.GetFiles(folder);
            var animalTypes = new List<Type>();
            foreach (var file in files)
            {
                var assembly = AssemblyLoadContext.Default.LoadFromAssemblyPath(file);
                var types = assembly.GetTypes();
                foreach (var t in types)
                {
                    //有了SDK就不用以下的操作了
                    //if (t.GetMethod("Voice") != null)
                    //{
                    //    animalTypes.Add(t);
                    //}
                    if (t.GetInterfaces().Contains(typeof(IAnimal))) ;
                    {
                        var isUnfinished = t.GetCustomAttributes(false).Any(a => a.GetType() == typeof(UnfinishedAttribute));
                        if (isUnfinished)
                        {
                            continue;
                        }
                        animalTypes.Add(t);
                    }
                }
            }

            while (true)
            {
                for (int i = 0; i < animalTypes.Count; i++)
                {
                    Console.WriteLine($"{i+1}.{animalTypes[i].Name}");
                }

                Console.WriteLine("========================================");
                Console.WriteLine("Please input an animal's sequence number:");

                int index = int.Parse(Console.ReadLine());
                if (index>animalTypes.Count || index<1)
                {
                    Console.WriteLine("No such an animal,Please try again");
                    continue;
                }

                Console.WriteLine("How many times?");
                int times = int.Parse(Console.ReadLine());
                var t = animalTypes[index - 1];
                var m = t.GetMethod("Voice");
                var o = Activator.CreateInstance(t);
                //m.Invoke(o, new object[] { times });
                //改为下边的写法
                var a = o as IAnimal;
                a.Voice(times);



                Console.WriteLine("--------------------------------------------------------------------");
                Console.WriteLine("--------------------------------------------------------------------");

            }
        }
    }
}
```

- - **结果**
  - 把有Unfinished标签的动物Cat过滤掉了

![img](https://pic3.zhimg.com/v2-e6b24c364b7950d88cf34275c5307112_1440w.jpg)

**19、property与attribute的区别**

property是属性，用于存取类的字段(get;set)；

attribute是特性，用来标识类，方法等的附加性质。

**20、访问修饰符**

（1）public 公有访问，不受任何限制。

（2）private 私有访问，只限于本类成员访问。

（3）protected 保护访问，只限于本类和子类访问。

（4）internal 内部访问，只限于当前程序集内访问。

**21、static关键字的应用**

对类有意义的字段和方法使用static关键字修饰，称为静态成员，通过类名加访问操作符“.”进行访问; 对类的实例有意义的字段和方法不加static关键字，称为非静态成员或实例成员。

注: 静态字段在内存中只有一个拷贝，非静态字段则是在每个实例对象中拥有一个拷贝。而方法无论是否为静态，在内存中只会有一份拷贝，区别只是通过类名来访问还是通过实例名来访问。

**22、文件编码格式**

阶段一：ASCII

阶段二：ANSI（本地化） 如：GBK、GB2312

阶段三：UNICODE（国际化） 如：UTF-8

**23、值传递与引用传递**

值传递时，系统首先为被调用方法的形参分配内存空间，并将实参的值按位置一一对应地复制给形参，此后，被调用方法中形参值得任何改变都不会影响到相应的实参；

引用传递时，系统不是将实参本身的值复制后传递给形参，而是将其引用值（即地址值）传递给形参，因此，形参所引用的该地址上的变量与传递的实参相同，方法体内相应形参值得任何改变都将影响到作为引用传递的实参。

简而言之，按值传递不是值参数是值类型，而是指形参变量会复制实参变量，也就是会在栈上多创建一个相同的变量。而按引用传递则不会。可以通过 ref 和 out 来决定参数是否按照引用传递。

**24、参数传递 ref 与 out 的区别**

（1）ref指定的参数在函数调用时必须先初始化，而out不用

（2）out指定的参数在进入函数时会清空自己，因此必须在函数内部进行初始化赋值操作，而ref不用

总结：ref可以把值传到方法里，也可以把值传到方法外；out只可以把值传到方法外

注意：string作为特殊的引用类型，其操作是与值类型看齐的，若要将方法内对形参赋值后的结果传递出来，需要加上ref或out关键字。

## 【ref和out】



**【为什么要使用ref和out】**

![img](https://pic3.zhimg.com/v2-1ed117781a8fad64e60fb2e0ed3fa18a_1440w.jpg)



 如果我们想要通过函数改变一个[值类型](https://zhida.zhihu.com/search?content_id=191190477&content_type=Article&match_order=1&q=值类型&zhida_source=entity)变量的值，这样写是没有办法改变的。因为我们知道值类型在进行值传递时，是在[栈空间](https://zhida.zhihu.com/search?content_id=191190477&content_type=Article&match_order=1&q=栈空间&zhida_source=entity)中重新开辟了空间，将内容拷贝到新空间。

![img](https://picx.zhimg.com/v2-f2614c2cb6596de436614067b8d65555_1440w.jpg)

 这就是使用**ref和out**的原因，他们的**作用是一样的**，**可以按地址传递对象，在函数内部改变后将改变原来参数的值**。



**【ref】**

**函数参数的修饰符**

**当传入的值类型参数在内部修改时**

**外部的值会发生变化**

![img](https://pic1.zhimg.com/v2-b18c2837dd4e8e57472351c7bd8705cc_1440w.jpg)

**外部的a变成了3**



**【out】**

**函数参数的修饰符**

**当传入的值类型参数在内部修改时**

**外部的值会发生变化**

![img](https://pic4.zhimg.com/v2-27d6cf1bfd00311f3916ca029a4fc55b_1440w.jpg)

**外部的a变成了3**



**【ref和out的区别】**

![img](https://picx.zhimg.com/v2-eb6c8231c4edfd3285ca1eb84a65af4d_1440w.jpg)

 通过上面的两段代码，你肯定会一脸懵，看起来ref和out一模一样，都是**参数前面的修饰符**，都是**传递引用地址**可以在内部改变后，外部也变。那它们有什么区别呢？



**区别一**

**ref传入的变量必须初始化**

**out不用**

![img](https://pic4.zhimg.com/v2-310f1cd031ff81fc40a5d919e0b44a45_1440w.jpg)



**区别二**

**out传入的参数必须在内部赋值**

**ref不用**

![img](https://pic4.zhimg.com/v2-be6af8c6171f020a2cfb9abc6120a137_1440w.jpg)





**【ref和out对[引用类型](https://zhida.zhihu.com/search?content_id=191190477&content_type=Article&match_order=1&q=引用类型&zhida_source=entity)同样有效】**

![img](https://pic3.zhimg.com/v2-016d41877e4d32c6796bb262fa5cf914_1440w.jpg)

 很多人可能会觉得这时候因为引用类型赋值时是传递的地址，那这时候第一个元素应该是3才对啊。我们来画图分析一下！

![img](https://picx.zhimg.com/v2-3676ee81d9fa945c196b43bfcbcb79dd_1440w.jpg)

 也就是，**引用类型**的数组**new了一个**新数组，就**意味着在[堆内存](https://zhida.zhihu.com/search?content_id=191190477&content_type=Article&match_order=1&q=堆内存&zhida_source=entity)中重新开辟了内存空间**，当前**变量指向的地址也会随之改变**。所以当改变了array后并没有影响到外部的arr数组。



**加上ref或者out**

![img](https://pica.zhimg.com/v2-efc5626c72e46033d2a25bc1cd335e96_1440w.jpg)

 我们从打印结果中明显的看到，外部的arr被修改了，所以我们说ref和out对于引用类型的对象来说，也是管用的！我们来图解一下！

![img](https://pic2.zhimg.com/v2-a12b64b13b1b21296cc3b9909b2d0d41_1440w.jpg)



## 【总结】 

**ref和out**



相同点

**函数参数的修饰符**

**传递对象的引用地址**

**让参数在函数内部修改后**

**外部也改变**



不同点

◇**初始化不同**

**ref必须初始化**

**out可以不初始化**

◇**内部赋值不同**

**ref可以不在内部赋值**

**out必须在内部赋值**



注意

**值类型和引用类型**

**都受影响**

**25、浅克隆与深克隆（浅拷贝与深拷贝）**

**（1）浅克隆**

在浅克隆中，如果原型对象的成员变量是值类型，将复制一份给克隆对象；如果原型对象的成员变量是引用类型，则将引用对象的地址复制一份给克隆对象，也就是说原型对象和克隆对象的成员变量指向相同的内存地址。简单来说，在浅克隆中，当对象被复制时只复制它本身和其中包含的值类型的成员变量，而引用类型的成员对象并没有复制，如图：

![img](https://picx.zhimg.com/v2-594faf4d1b70c85a8b72eee0dc55c8a3_1440w.jpg)



通过实现ICloneable接口的Clone()方法，并调用MemberwiseClone()方法来实现浅克隆

![img](https://pic2.zhimg.com/v2-47f6b10245948abc055f5c73aaab0f29_1440w.jpg)

**（2）深克隆**

在深克隆中，无论原型对象的成员变量是值类型还是引用类型，都将复制一份给克隆对象，深克隆将原型对象的所有引用对象也复制一份给克隆对象。简单来说，在深克隆中，除了对象本身被复制外，对象所包含的所有成员变量也将复制，如图：

![img](https://pic2.zhimg.com/v2-a391b561fa7c736abdd3f625a82d3cf3_1440w.jpg)

在C#语言中，如果需要实现深克隆，可以通过**序列化(Serialization)**等方式来实现。序列化就是将对象写到流的过程，写到流中的对象是原有对象的一个拷贝，而原对象仍然存在于内存中。通过序列化实现的拷贝不仅可以复制对象本身，而且可以复制其引用的成员对象，因此通过序列化将对象写到一个流中，再从流里将其读出来，可以实现深克隆。需要注意的是能够实现序列化的对象其类必须实现Serializable接口，否则无法实现序列化操作。

![img](https://picx.zhimg.com/v2-27444c1bee4923ea4a56c3440efa50e9_1440w.jpg)

**二、数据库**

**1、数据库操作的相关类**

特定类：Connection，Command，CommandBuilder，DataAdapter，DataReader，Parameter，Transaction

共享类：DataSet，DataTable，DataRow，DataColumn，DataRealtion，Constraint，DataColumnMapping，DataTableMapping

（1）**Connection**：开启程序与数据库之间的连接。

（2）**Command**：对数据库发送一些指令。例如增删改查等指令，以及调用存在数据库中的存储过程等。

（3）**DataAdapter**：主要在数据源及DataSet 之间执行传输工作，通过Command 下达命令后，将取得的数据放进DataSet对象中。

（4）**DataSet**：这个对象可视为一个暂存区(Cache),可以把数据库中所查询到的数据保存起来，甚至可以将整个数据库显示出来，DataSet是放在内存中的。

备注：将DataAdapter对象当做DataSet 对象与数据源间传输数据的桥梁。DataSet包含若干DataTable、DataTableTable包含若干DataRow。

（5）**DataReader**：一笔向下循序的读取数据源中的数据。

总结：[http://ADO.NET](https://link.zhihu.com/?target=http%3A//ADO.NET) 使用Connection对象来连接数据库，使用Command或DataAdapter对象来执行SQL语句，并将执行的结果返回给DataReader或DataAdapter，然后再使用取得的DataReader或DataAdapter对象操作数据结果。

**2、事务**

**3、索引**

**4、视图**

**5、存储过程**

**三、数据结构（常用的排序算法）**

**1、冒泡排序**

（1）原理

（2）实现代码

**2、快速排序**

（1）原理

（2）实现代码

**四、设计模式**

**1、单例模式**

在软件系统中，经常有这样一些特殊的类，必须保证它们在系统中只存在一个实例，才能确保它们的逻辑正确性、以及良好的效率。保证一个类仅有一个实例，并提供一个该实例的全局访问点。

（1）经典模式--单线程

对于线程来说不安全；但在单线程中已满足要求。

![img](https://picx.zhimg.com/v2-bec678b8ab83fa1090bf12783abcf929_1440w.jpg)

（2）懒汉模式--多线程

多线程安全；线程不是每次都加锁，节省了性能开销。

![img](https://pic1.zhimg.com/v2-b82bcc3af92c6981561a9dce5a446758_1440w.jpg)

（3）饿汉模式--多线程

利用静态变量去实现单例，由CLR保证，在程序第一次使用该类之前被调用，而且只调用一次；饿汉模式中的静态变量是随着类加载时被完成初始化的，静态代码块也会随着类的加载一块执行，哪怕你一直没有调用。

![img](https://pic3.zhimg.com/v2-8737773daac41e12c754add9171e49fe_1440w.jpg)

**五、软件开发流程**

**需求分析 --> 概要设计 --> 详细设计 --> 编码 --> 测试 --> 交付 --> 验收 --> 维护**

**六、其他（了解）**

**1、.NET Core 与 .NET Framework 的区别**

.NET Core 就是 .NET Framework 的开源且跨平台版本。但微软毕竟不能维护两个不同的分支，一个跑在Windows上，一个跑在Linux（Unix Like）系统上，所以微软抽象出来一个标准库.NET Standard Library，.NET Core 与 .NET Framework 都必须实现标准库的API ，就这样.NET Core、.NET Framework、Xamarin成了三兄弟，分别为不同的平台服务。

![img](https://pic4.zhimg.com/v2-865f80588bad9ca27ec25ee4294a7bd1_1440w.jpg)

# **const 和 readonly 有什么区别**

const 关键字用来声明编译时常量，readonly 用来声明运行时常量。他们都可以标识一个常量，主要有以下区别:

1. 初始化位置不同：const 必须在申明的同时进行赋值，readonly 可以在声明时赋值，也可以在构造方法里赋值。
2. 修饰对象不同：const 既可以修饰类字段，也可以修饰局部变量；readonly 只能修饰类字段。
3. const 是编译时常量，在编译时就能确定该值，并且在编译时会被内联到代码中；readonly 是运行时常量，在运行时确定该值。
4. const 默认是静态的；而 readonly 如果设置为静态需要显示声明：public readonly static int PORT = 10086;
5. 支持的类型不同，const 只能修饰基元类型或值为 null 的其他引用类型；而 readonly 可以是任何类型。

```language-csharp
public class ConstantsAndReadonly
{
    // const 声明编译时常量
    const int Id = 1001; // 必须在声明时初始化

    // readonly 声明运行时常量
    readonly int Port; // 可以在声明时或构造方法中初始化

    public ConstantsAndReadonly()
    {
        // 给 readonly 字段赋值
        Port = 10086;
    }

    // const 用作局部变量
    void ConstLocalVariable()
    {
        const double Pi = 3.14159;
        Console.WriteLine("Value of Pi: " + Pi);
    }

    // readonly 用作静态字段
    public readonly static int DefaultTimeout = 5000; // 静态 readonly 需要显示声明

    // const 修饰基元类型
    const int MaxRetries = 3;

    // const 修饰值为null的引用类型（虽然不常见）
    const string OptionalUrl = null;

    // readonly 修饰任意类型
    readonly Dictionary<string, int> LookupTable;

    public ConstantsAndReadonly()
    {
        // 初始化复杂的 readonly 字段
        LookupTable = new Dictionary<string, int>
        {
            { "One", 1 },
            { "Two", 2 }
        };
    }
}

class Program
{
    static void Main(string[] args)
    {
        ConstantsAndReadonly myObject = new ConstantsAndReadonly();

        // 访问 const 常量
        Console.WriteLine("Const Id: " + ConstantsAndReadonly.Id);

        // 访问 readonly 常量
        Console.WriteLine("Readonly Port: " + myObject.Port);

        // 访问 readonly 静态常量
        Console.WriteLine("Readonly static DefaultTimeout: " + ConstantsAndReadonly.DefaultTimeout);

        // 访问 const 修饰的复杂类型
        myObject.ConstLocalVariable();

        // 访问 readonly 修饰的复杂类型
        foreach (var pair in myObject.LookupTable)
        {
            Console.WriteLine("Key: " + pair.Key + ", Value: " + pair.Value);
        }
    }
}
```

作者：架构师
链接：https://www.zhihu.com/question/434520453/answer/1625267213
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



**字段的使用**

**1.关于字段**

a.字段又称为：“成员变量”，一般在类的内部做数据交互使用。

b.字段命名规范：camel命名法（首单词字母小写）。

**2.通俗的理解**：

私有化：字段就好比我们的个人财产，仅供个人使用，所以一般是[private修饰](https://zhida.zhihu.com/search?content_id=325578668&content_type=Answer&match_order=1&q=private修饰&zhida_source=entity)。

添加标准：根据程序的功能需求，具体来添加需要的不同类型的字段

**属性**

**1.属性的使用**

作用：在[面向对象设计](https://zhida.zhihu.com/search?content_id=325578668&content_type=Answer&match_order=1&q=面向对象设计&zhida_source=entity)中主要使用属性描述对象的静态特征。

要求：一般采用Pascal[命名法](https://zhida.zhihu.com/search?content_id=325578668&content_type=Answer&match_order=2&q=命名法&zhida_source=entity)（首字母大写），数据类型要和对应的字段要一致。

**2.属性的理解**

a.属性其实就是外界访问私有字段的入口，属性本身不保存任何数据，在对属性赋值和读取的时候其实就是操作的对应[私有字段](https://zhida.zhihu.com/search?content_id=325578668&content_type=Answer&match_order=2&q=私有字段&zhida_source=entity)。



![img](https://pic1.zhimg.com/80/v2-3297c9d1fb0ea5b1c1a0e5e6c4ad019c_720w.webp?source=1def8aca)



b.属性本质其实就是一个方法，通过get和set方法来操作对应的字段，通过反编译工具我们可以看出，如图：

![img](https://picx.zhimg.com/80/v2-739d820ae10c91b887aa47875cc54581_720w.webp?source=1def8aca)

**3.属性的作用**

A.避免出现非法数据，例如一个年龄属性，正常逻辑是年龄是不可以出现负数的，如果用户对年龄进行了负数的赋值，我们通过属性的set方法加入判断逻辑，从而排除这种非法数据。

示例：



![img](https://pic1.zhimg.com/80/v2-47a06c5057ce5e0ce9877a5c5f72aff7_720w.webp?source=1def8aca)



B.限定属性只读或者只写，例如有些属性为了保护数据，我们只能读取，而不能赋值。具体使用的话，我们根据需要，屏蔽set或get方法达到只读和只写。

示例：



![img](https://pic1.zhimg.com/80/v2-82c030b2eb56fa2c5840a6d2a655a97e_720w.webp?source=1def8aca)



C．没有对应私有字段的属性，例如根据生日属性获取到年龄。

示例：

![img](https://picx.zhimg.com/80/v2-c97d41eca665a45c6f665d4226af1962_720w.webp?source=1def8aca)

**字段与属性比较**

**字段（[成员变量](https://zhida.zhihu.com/search?content_id=325578668&content_type=Answer&match_order=2&q=成员变量&zhida_source=entity)）**

a.字段主要是为类的内部做数据交互使用，字段一般是private。

b.字段可读可写。

c.当字段需要为外部提供数据的时候，请将字段封装为属性，而不是使用公有字段([public修饰符](https://zhida.zhihu.com/search?content_id=325578668&content_type=Answer&match_order=1&q=public修饰符&zhida_source=entity))，这是[面向对象思想](https://zhida.zhihu.com/search?content_id=325578668&content_type=Answer&match_order=1&q=面向对象思想&zhida_source=entity)所提倡的。

**属性（方法）**

a.属性一般是向外提供数据，主要用来描述对象的静态特征，所以，属性一般是public。

b.属性具备get和set方法，可以在方法里加入逻辑[处理数据](https://zhida.zhihu.com/search?content_id=325578668&content_type=Answer&match_order=1&q=处理数据&zhida_source=entity)，灵活拓展使用。

**自动属性**

**1.属性编写新语法（自动属性：.NET3.0及其后续版本）**

示例：

![img](https://pica.zhimg.com/80/v2-66412f38ec771e8563d45d8baa22ad6b_720w.webp?source=1def8aca)



在C#中，基元类型（Primitive Types）是指由语言规范直接支持的简单数据类型。这些类型通常具有固定的内存大小和简单的操作。C#中的基元类型包括：

1. **整型**:
   - byte：无符号的8位整数（范围0-255）。
   - sbyte：有符号的8位整数（范围-128到127）。
   - short（或 int16）：有符号的16位整数。
   - ushort（或 uint16）：无符号的16位整数。
   - int（或 int32）：有符号的32位整数。
   - uint（或 uint32）：无符号的32位整数。
   - long（或 int64）：有符号的64位整数。
   - ulong（或 uint64）：无符号的64位整数。

1. **浮点型**:
   - float：32位单精度浮点数。
   - double：64位双精度浮点数。
   - decimal：128位高精度浮点数，适用于需要高精度的金融计算。

1. **字符型**:
   - char：16位Unicode字符。

1. **布尔型**:
   - bool：表示逻辑值 true 或 false。

1. **指针型**（在安全上下文中使用，例如unsafe代码块）:
   - void*：指向未知数据类型的指针。
   - bool*、char*、int* 等：指向特定类型的指针。

1. **无类型（空类型，****void****）**:
   - void：表示没有值或类型的方法或属性。

# **哪些类型可以定义为常量？常量 const 有什么风险？**

基元类型和值为 null 的引用类型可以定义为常量，常量不支持跨程序集版本更新，常量值更新后，所有使用了该常量值的代码都必须重新编译。

因为对于常量来说，对值的引用是直接内联常量值，而非引用值。相当于直接在代码中将值写死。

常量的值必须在编译时确定，简单说就是在定义时就设置好对应的值，后续的运行中对应的参数值将不会改变。

常量只能用于简单的类型，因为常量值是要被编译然后保存到程序集的元数据中，因此只支持基元类型，如：int、char、bool、double等。

常量在使用时，是把常量的值内联到 IL 代码中的，常量类似于一个占位符 ，在编译时被替换掉了。也正是这个特点导致了常量有一个风险，就是不支持跨程序集版本更新。

**枚举的本质**

接着上述的 const 说，枚举 enum 也有类似的问题，其根源和 const 一样。

下面是一个简单的枚举定义，对应的 IL 代码和 const 的定义是一样的。

枚举的成员定义和常量的定义一样，因此枚举其实本质上相当是一个常量集合。



字段（Fields）和属性（Properties）在C#中都是存储数据的方式，但它们之间存在一些关键的异同点。以下是字段和属性的对比，以及C#代码示例：

# **相同点：**

- **数据存储**：字段和属性都可以用于存储数据内容。

# **不同点：**

1. **访问权限**：字段通常作为类的私有成员（private或protected），而属性可以具有不同的访问修饰符，包括公共访问（public）。
2. **封装性**：属性提供了封装，允许在不暴露实现细节的情况下访问和修改数据。
3. **业务逻辑**：属性的get和set访问器中可以编写代码，例如验证逻辑，而字段则不能。
4. **重写和隐藏**：属性可以被重写（override）和重新声明（new）。
5. **数据类型**：属性可以返回和设置任何类型的值，而字段的数据类型必须在声明时指定

```language-csharp
public class Person
{
    // 私有字段
    private string _name;

    // 公共属性，提供对字段_name的受控访问
    public string Name
    {
        get { return _name; }
        set
        {
            if (string.IsNullOrEmpty(value))
            {
                throw new ArgumentException("Name cannot be empty.");
            }
            _name = value;
        }
    }

    // 公共字段，直接访问
    public int Age;
}

class Program
{
    static void Main()
    {
        Person person = new Person();

        // 使用属性设置和获取值
        person.Name = "Alice"; // 调用属性的set访问器
        Console.WriteLine(person.Name); // 调用属性的get访问器

        // 直接设置字段的值
        person.Age = 30;

        // 尝试设置无效的名称，将触发异常
        try
        {
            person.Name = "";
        }
        catch (ArgumentException e)
        {
            Console.WriteLine(e.Message);
        }
    }
}
```

# **静态成员和非静态成员的区别？**

1. 静态变量使用 static 修饰符进行声明，静态成员在类加载的时候就被加载，并通过类型对象进行访问。
2. 不带有 static 修饰符声明的变量称为 非静态变量，在对象被实例化时创建，通过对象进行访问。
3. 一个类的所有实例的同一个静态变量都是同一个值，同一个类的不同实例的同一非静态变量可以是不同的值。
4. 静态函数的实现里不能使用非静态成员，如非静态变量、非静态函数等。

# **自动属性有什么风险？**

自动属性的私有字段是由编译器命名的，后期不宜随意修改，比如在序列化中会导致字段值丢失。

# **特性是什么？如何使用**

Attribute 特性是关联了一个目标对象的一段配置信息，存储在dll内的元数据。它本身没什么意义，可以通过反射在运行时获取配置的特性信息。

特性可以用来标记代码，以便在编译时或运行时进行特定的处理，例如：

- 标记控件以用于窗体设计。
- 指定序列化和反序列化的行为。
- 定义自定义属性以存储额外信息。

# **如何使用特性：**

1. **定义特性**：创建一个类继承自 System.Attribute。
2. **应用特性**：使用特性时，通常使用 AttributeTargets 枚举指定特性可以应用于哪些类型的代码元素。
3. **获取特性**：通过反射读取特性信息。

```language-csharp
using System;
using System.Reflection;

// 定义一个自定义特性
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, Inherited = false)]
public class MyCustomAttribute : Attribute
{
    public string Description { get; set; }

    public MyCustomAttribute(string description)
    {
        Description = description;
    }
}

// 将特性应用到类和方法
[MyCustomAttribute("This is a demo class.")]
public class DemoClass
{
    [MyCustomAttribute("This method demonstrates the use of attributes.")]
    public void DemoMethod()
    {
        Console.WriteLine("DemoMethod is called.");
    }
}

class Program
{
    static void Main(string[] args)
    {
        // 通过反射获取特性
        Type type = typeof(DemoClass);
        MyCustomAttribute classAttribute = type.GetCustomAttribute<MyCustomAttribute>();
        Console.WriteLine("Class Description: " + classAttribute.Description);

        MethodInfo methodInfo = type.GetMethod("DemoMethod");
        MyCustomAttribute methodAttribute = methodInfo.GetCustomAttribute<MyCustomAttribute>();
        Console.WriteLine("Method Description: " + methodAttribute.Description);

        // 调用带有特性的方法
        DemoClass demo = new DemoClass();
        demo.DemoMethod();
    }
}
```

# **解释：**

- 定义了一个名为 MyCustomAttribute 的自定义特性，它有一个 Description 属性和一个构造函数。
- 特性 MyCustomAttribute 被应用于 DemoClass 类和 DemoMethod 方法。
- 在 Main 方法中，使用反射获取 DemoClass 类和 DemoMethod 方法的自定义特性，并打印出它们的描述。
- 然后创建 DemoClass 类的实例，并调用 DemoMethod 方法。

# **下面的代码输出什么结果？为什么？**

```language-csharp
List<Action> actions = new List<Action>(5);
for (int i = 0; i < actions.Length; i++)
{
	actions.Add(() => Console.WriteLine(i));
}

actions.ForEach(act => act());
```

输出： 5， 5， 5， 5， 5

原因：

1. Action 是一个委托类型，委托类型最后会被编译成一个静态类型，变量 i 在这个作用域中被 Action 捕获到，并存储到 Action 实例的公共字段中，最终委托执行时实际运行的是委托类型中的 Invoke() 方法。
2. 这里创建了5个 Action 委托的实例对象，每个实例对象中都持有对于同一个变量 i 的引用，即最后5个对象中存储的值是指向同一个变量。
3. actions 是在循环结束后执行的，此时变量 i 的值随着循环执行结束已经变成了 5。

# **C# 中的委托是什么？事件是不是一种委托？**

委托类似于 C 或者 C++ 中的函数指针，允许将方法作为参数进行传递。

委托在编译后会生成一个类，类中会存放委托声明时捕获到的一些参数，并最终在 Invoke() 时执行真实的方法调用。

事件可以理解为一种特殊的委托，事件内部也是基于委托来实现的。

# **委托与事件的区别？**

委托 delegate 和事件 event 是 C# 中的两个关键字，在最终编译后都会生成一个具体的类型，用于表示对方法的引用，可以指向一个或者多个方法，并通过调用这些方法来执行特定的操作。当调用多播委托时，所有被引用的方法会被依次调用。

主要区别在于：

1. 定义方式：委托是一种类型，可以单独定义。可以使用 delegate 关键字定义一个委托类型，并创建委托变量来引用方法。而事件是一种特殊的委托，通常用来表示对象或组件的状态变化。事件通过 event 关键字定义。
2. 委托可以直接调用、添加或删除方法引用。但事件通常提供了更严格的访问控制。事件只能在声明它的类内部进行添加和删除操作（+= 和 -=），但在类外部只能通过 += 和 -= 操作符来订阅或取消订阅事件。
3. 发布订阅模式：事件通常用于实现发布-订阅模式。在这种模式中，一个对象（发布者）会通过事件通知其他对象（订阅者）状态的变化。订阅者可以通过订阅发布者的事件来响应这些变化。
4. 使用限制：由于事件只能通过声明它的类内部进行添加和删除操作，因此它提供了更好的封装性和数据保护性。委托则没有这样的限制，可以在任何地方直接调用。

```language-csharp
var p = new Person();
p.greetAction += (name) => Console.WriteLine($"早上好,{name}");
p.greetAction += (name) => Console.WriteLine($"中午好,{name}");
p.greetAction += (name) => Console.WriteLine($"晚上好,{name}");
p.Greet("elias");

Console.ReadKey();

public class Person
{
    public event Action<string>? greetAction;

    public void Greet(string name)
    {
        greetAction?.Invoke(name);
    }
}
```

这里定义了一个事件，并对事件添加了多个委托方法

这段C#代码演示了如何使用事件（）来允许多个方法或lambda表达式订阅同一个行为，当事件触发时，所有订阅的方法或表达式都会被调用。这里是逐步解释：

1. var p = new Person(); - 创建了Person类的一个新实例，并将其存储在变量p中。
2. p.greetAction += (name) => Console.WriteLine($"早上好,{name}"); - 这行代码为Person实例的greetAction事件添加了一个lambda表达式作为事件处理器。当greetAction事件被触发时，这个lambda表达式会被执行，打印出"早上好,"加上传入的名字。
3. 类似地，接下来的两行代码也为greetAction事件添加了另外两个lambda表达式，分别打印不同的问候语。
4. p.Greet("elias"); - 调用Person实例的Greet方法，并传入字符串"elias"作为参数。这个方法会触发greetAction事件，导致所有订阅该事件的lambda表达式被调用，并传入name参数。
5. Console.ReadKey(); - 这行代码让程序暂停，等待用户按下一个键，以便查看控制台输出的结果。

# **Person类定义：**

- public event Action? greetAction; - 定义了一个名为greetAction的事件，它的委托类型是Action，意味着这个事件可以有零个或多个接受一个string类型参数的无返回值方法（或lambda表达式）作为其订阅者。? 表示这个事件可以为null。
- public void Greet(string name) - 这是一个公共方法，当调用时，会检查greetAction事件是否有订阅者。如果有，使用Invoke方法触发事件，并传入name参数。?.Invoke(name) 是 C# 8.0 引入的 null 条件操作符的简写形式，意味着如果greetAction不为null，则调用它，否则不执行任何操作。

# **输出结果：**

当执行时，由于有三个订阅者（三个lambda表达式）都订阅了事件，它们将依次被调用，并分别打印出：

这样的设计模式允许程序在不同的时间点或不同的上下文中响应相同的行为，是一种灵活的事件驱动编程方式。



**委托和事件**

简单来说，委托类似与 C 语言 或者 C++ 中的函数指针，允许将方法作为参数进行传递。

1. C# 中的委托都继承自 System.Delegate 类型。
2. 委托类型的声明和方法签名类似，有返回值和参数。
3. 委托是一种可以封装命名（或匿名）方法的引用类型，可以把方法当作指针传递，但是委托是面向对象、类型安全的。

委托的本质——一个静态类。

.NET 中没有函数指针，方法也不可能传递，委托之所以可以像一个普通引用类型一样传递，那是因为它本质上就是一个类型。

下面代码是一个简单的自定义委托类型：

```language-csharp
//=========================================================================
// 定义委托类型
public delegate int MathOperation(int x, int y);

// 定义委托指向的方法
static int Add(int x, int y) { return x + y; }
static int Subtract(int x, int y) { return x - y; }

class Program
{
    static void Main()
    {
        // 实例化委托
        MathOperation operation = Add;

        // 调用委托，相当于调用方法
        int result = operation(5, 3);
        Console.WriteLine("Result: " + result); // 输出结果

        // 改变委托引用的方法
        operation = Subtract;
        result = operation(5, 3);
        Console.WriteLine("Result: " + result); // 输出结果
    }
}
//=========================================================================
//委托作为参数传递
static void PerformOperation(MathOperation op, int a, int b)
{
    int result = op(a, b);
    Console.WriteLine("Performed operation: " + result);
}

// 在Main中调用
PerformOperation(Add, 5, 3);
//=========================================================================
//委托链式调用
MathOperation复合操作 = Add;
复合操作 += Subtract; // 链式委托

int 复合结果 = 复合操作(10, 5); // 先执行Add，再执行Subtract
Console.WriteLine("Compound Result: " + 复合结果);
//=========================================================================
// 使用匿名委托直接在声明时创建和初始化
MathOperation anonymousOperation = (int x, int y) => x * y;
int product = anonymousOperation(5, 3);
Console.WriteLine("Product: " + product);
//=========================================================================
// Lambda表达式创建委托
MathOperation lambdaOperation = (x, y) => x / y;
int divisionResult = lambdaOperation(10, 3);
Console.WriteLine("Division Result: " + divisionResult);
//=========================================================================
//委托与事件
public class Calculator
{
    public event MathOperation Calculation;

    public void Calculate(int a, int b)
    {
        if (Calculation != null)
        {
            int result = Calculation(a, b);
            Console.WriteLine("Calculation Result: " + result);
        }
    }
}
//=========================================================================
//多播委托
calc.Calculation += Subtract; // 订阅另一个事件处理程序
calc.Calculate(5, 3); // 触发所有订阅的事件处理程序
// 在Main中使用
Calculator calc = new Calculator();
calc.Calculation += Add;
calc.Calculate(5, 3);
//=========================================================================
解释：
定义委托：首先定义一个委托类型，指定可以接受的参数类型和返回类型。
实例化委托：创建委托的实例，并将其指向一个具体的方法。
调用委托：通过委托实例调用方法，就像直接调用该方法一样。
委托作为参数：可以将委托作为参数传递给其他方法，实现策略模式。
委托链式调用：可以创建委托链，一个委托可以链接到另一个委托，实现多个方法的连续调用。
匿名委托：使用匿名委托可以不定义委托类型的名称，直接创建委托实例。
Lambda表达式：Lambda表达式是一种简写的委托声明方式，可以快速创建委托实例。
委托与事件：委托经常用于实现事件，事件是一种特殊的委托，允许多个方法订阅同一个事件。
多播事件：事件可以有多个订阅者，当事件被引发时，所有订阅者的方法都会被调用。
```



```language-csharp
1. 定义和使用事件
public class MessagePublisher
{
    // 定义事件
    public event EventHandler OnMessageReceived;

    // 引发事件
    public void SendMessage(string message)
    {
        Console.WriteLine("Message sent: " + message);
        OnMessageReceived?.Invoke(this, EventArgs.Empty);
    }
}

class Program
{
    static void Main()
    {
        MessagePublisher publisher = new MessagePublisher();

        // 订阅事件
        publisher.OnMessageReceived += (sender, e) =>
        {
            Console.WriteLine("Message received event triggered.");
        };

        // 引发事件
        publisher.SendMessage("Hello, World!");
    }
}
2. 使用强类型事件
public class Calculator
{
    public event EventHandler<CalculationEventArgs> Calculated;

    protected virtual void OnCalculated(double result)
    {
        Calculated?.Invoke(this, new CalculationEventArgs(result));
    }
}

public class CalculationEventArgs : EventArgs
{
    public double Result { get; }

    public CalculationEventArgs(double result)
    {
        Result = result;
    }
}
3. 取消事件订阅
// 假设有一个事件处理方法
void OnMessageReceived(object sender, EventArgs e)
{
    Console.WriteLine("Event received.");
}

// 订阅事件
publisher.OnMessageReceived += OnMessageReceived;

// 取消订阅事件
publisher.OnMessageReceived -= OnMessageReceived;
4. 使用事件的线程安全
lock (_syncRoot) // _syncRoot 是一个用于同步的字段
{
    eventHandler += value; // 安全地添加事件处理器
}
5. 引发事件时考虑空值
// 使用 null 条件运算符确保在事件未被订阅时不引发异常
eventHandler?.Invoke(sender, e);
6. 使用自定义事件参数
public class CustomEventArgs : EventArgs
{
    public string CustomData { get; set; }
}

// 引发带有自定义数据的事件
public void RaiseCustomEvent(string data)
{
    CustomEvent?.Invoke(this, new CustomEventArgs { CustomData = data });
}
解释：
定义事件：在类中使用 event 关键字定义事件，通常是一个返回 void 的委托类型。
引发事件：在适当的时机使用 ?.Invoke() 语法来引发事件，确保在没有订阅者时不会抛出异常。
订阅事件：使用 += 运算符将方法或lambda表达式订阅到事件上。
取消订阅事件：使用 -= 运算符从事件中移除特定的方法或lambda表达式。
强类型事件：使用自定义的 EventArgs 派生类来传递事件数据，提供比 EventArgs.Empty 更丰富的信息。
线程安全：在多线程环境中，使用 lock 语句确保事件订阅和取消订阅的线程安全。
自定义事件参数：创建自定义的 EventArgs 类来传递额外的数据给事件订阅者。
事件是C#中实现观察者模式的关键机制，允许对象在发生感兴趣的事情时通知多个订阅者。通过使
```

[事件](https://zhida.zhihu.com/search?content_id=735801655&content_type=Answer&match_order=1&q=事件&zhida_source=entity)本质上是一种特殊用法的委托，它限制了外部对委托的直接操作，提供了更好的封装性和安全性。

委托本质上是一个**类**（继承自`System.MulticastDelegate`）

事件是**对委托的封装**，本质上是一个**受限的委托字段**+**两个访问器方法（add/remove）**

| 特性     | 委托                 | 事件                                                         |
| -------- | -------------------- | ------------------------------------------------------------ |
| 本质     | 类型                 | 成员                                                         |
| 调用权限 | 任何有委托引用的代码 | 只能在声明类中调用                                           |
| 外部访问 | 可赋值(=)和调用      | 只能+=/-=                                                    |
| 用途     | 通用回调机制         | 实现[观察者模式](https://zhida.zhihu.com/search?content_id=735801655&content_type=Answer&match_order=1&q=观察者模式&zhida_source=entity) |
| 封装性   | 低                   | 高                                                           |
| 多播     | 支持                 | 支持                                                         |

- 如果需要公开回调机制给外部，使用**事件**更安全
- 如果仅在类内部使用回调，委托可能更简单
- 事件实际上是委托的封装器，提供了一层保护

在委托delegate出现了很久以后，微软的.NET设计者们终于领悟到，其实所有的委托定义都可以归纳并简化成只用Func与Action这两个语法糖来表示。其中，Func代表有返回值的委托，Action代表无返回值的委托。有了它们两，我们以后就不再需要用关键字delegate来定义委托了。

# Lambda表达式

[Lambda表达式](https://zhida.zhihu.com/search?content_id=145815636&content_type=Article&match_order=1&q=Lambda表达式&zhida_source=entity)（Lambda Expression）是C#中一种特殊语法，它的引入，使得匿名方法更加简单易用，最直接的是在方法体内调用代码或者为[委托](https://zhida.zhihu.com/search?content_id=145815636&content_type=Article&match_order=1&q=委托&zhida_source=entity)传入方法体的形式与过程变得更加优雅。

Lambda表达式实际上是一种匿名函数，在Lambda表达式中可以包含语句以及运算等操作。并且可用于创建委托或表达式目录树类型，支持带有可绑定到委托或[表达式树](https://zhida.zhihu.com/search?content_id=145815636&content_type=Article&match_order=1&q=表达式树&zhida_source=entity)的输入参数的内联表达式。使用Lambda表达式可大大减少代码量，使得代码更加的优美、简洁，更有可观性。

它的语法如下：

**(Input Param)=>Expression**

主要的结构就是 参数 =>(goes to) 代码块