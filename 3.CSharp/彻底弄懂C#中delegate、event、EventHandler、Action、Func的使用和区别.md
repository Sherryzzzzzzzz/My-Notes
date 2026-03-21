# 彻底弄懂C#中delegate、event、EventHandler、Action、Func的使用和区别

### 【目录】

### 1 [委托](https://zhida.zhihu.com/search?content_id=225909793&content_type=Article&match_order=1&q=委托&zhida_source=entity)

### 2 [事件](https://zhida.zhihu.com/search?content_id=225909793&content_type=Article&match_order=1&q=事件&zhida_source=entity)-概念的引出

### 3 事件-关于异常

### 4 事件-关于异步

### 5 委托-Func与[Action](https://zhida.zhihu.com/search?content_id=225909793&content_type=Article&match_order=1&q=Action&zhida_source=entity)



### 1 委托

在.NET中定义“委托”需要用到delegate关键字，它是存有对某个方法的引用的一种引用类型变量，类似于 C 或 C++ 中函数的指针。“委托”主要有两大作用：（1）将方法当作参数传递

（2）方法的一种多态（类似于一个方法模板，可以匹配很多个方法）下面，给出一个展现了上述两大作用的委托代码示例：

```text
        //定义一个委托
        public delegate int MyDelegate(int x, int y);

        //与委托匹配的一个方法
        public static int Add(int a, int b)
        {
            return a + b;
        }

        //与委托匹配的另一个方法
        public static int Reduce(int a, int b)
        {
            return a - b;
        }

        //示例：将委托/方法当参数传递
        public static int Test(MyDelegate MD)
        {
            return MD(10, 20);
        }

        static void Main(string[] args)
        {
            int a, b, x, y;

            MyDelegate md;

            //将委托指向Add这个方法，并进行相关操作
            md = Add;
            a = md(1, 2);
            b = Test(md);

            //再将委托指向Reduce这个方法，并进行相关操作
            md = Reduce;
            x = md(7, 2);
            y = Test(md);

            Console.WriteLine($"1+2={a},10+20={b},7-2={x},10-20={y}");
            Console.ReadLine();
        }
```

执行以上程序，输出结果如下：

*1+2=3,10+20=30,7-2=5,10-20=-10*

委托也可以使用+=/-=来实现[“发布/订阅”模式](https://zhida.zhihu.com/search?content_id=225909793&content_type=Article&match_order=1&q=“发布%2F订阅”模式&zhida_source=entity)，示例代码如下：

```text
        //定义一个委托
        public delegate void MyDelegate1(int x);

        public static void Method1(int a)
        {
            Console.WriteLine($"a={a}");
        }

        public static void Method2(int b)
        {
            Console.WriteLine($"b={b}");
        }

        static void Main(string[] args)
        {
            MyDelegate1 md = null;
            md += Method1;
            md += Method2;
            md(35);

            Console.ReadLine();
        }
```

以上程序输出如下：

*a=35*

*b=35*

但是委托有一个弊端，它可以使用“=”将所有已经订阅的取消，只保留=后的这一个订阅。

为了解决这个弊端，事件event应运而生。

### 2 事件-概念的引出

事件event是一种特殊的委托，它只能+=，-=，不能直接用=。event在定义类中（发布者）是可以直接=的，但是在其他类中（订阅者）就只能+= -=了，**也就是说发布者发布一个事件后，订阅者针对他只能进行自身的订阅和取消。**下面是定义一个事件的代码：

```text
        //定义一个委托
        public delegate void MyDelegate1(int x);
        //定义一个事件
        public event MyDelegate1 emd;
```

经过长久的经验积累后，人们发现，绝大多数事件的定义，是用public delegate void XXX(object sender, EventArgs e);这样一个委托原型进行定义的，是一件重复性的工作，于是，[EventHandler](https://zhida.zhihu.com/search?content_id=225909793&content_type=Article&match_order=1&q=EventHandler&zhida_source=entity)应运而生。它的出现就是为了避免这种重复性工作，并建议尽量使用该类型作为事件的原型。

```text
//@sender: 引发事件的对象
//@e: 传递的参数
public delegate void EventHandler(object sender, EventArgs e);

//使用
public event EventHandler emd;
```

下面给出一个使用事件的具体示例：

```text
        public class Demo
        {
            public event EventHandler emd;
            public void RaiseEvent()
            {
                emd(this, EventArgs.Empty);
            }
        }

        static void Main(string[] args)
        {
            var instance = new Demo();
            instance.emd += (sender, arg) =>
            {
                Console.WriteLine("执行事件1!");
            };

            instance.emd += (sender, arg) =>
            {
                Console.WriteLine("执行事件2!");
            };

            instance.RaiseEvent();

            Console.ReadLine();
        }
```

这里我们先定义一个Demo类，其内部有个事件是emd，我们给他开放了一个接口RaiseEvent，如果谁敢调用它，那么，它就触发报警事件emd。

这里模拟了2个订阅者，分别处理报警事件emd。程序执行结果如下：执行事件1!执行事件2!同时，我们也可以看出：**事件是按照+=的订阅先后顺序执行的。**

### 3 事件-关于异常

现在，我们在第一个订阅者中加入异常，如下：

```text
    instance.emd += (sender, arg) =>
    {
        Console.WriteLine("执行事件1!");
        throw new Exception("执行事件1,错误");
    };
```

执行后发现，第1个订阅者事件触发抛出异常后，第2个订阅者的事件没有执行。

**可见，如果你想让所有订阅者都执行处理的话，那每个订阅者必须在订阅程序内自己处理好异常，不能抛出来！**



### 4 事件-关于异步

如果事件的订阅者中有一个是“异步”处理，又会是什么情况？

下面我们把第1个订阅者改为异步处理，代码如下：

```text
    instance.emd += async (sender, arg) =>
    {
        Console.WriteLine("执行事件1!");
        await Task.Delay(1000);
        Console.WriteLine("执行事件1!完毕");
    };
```

执行后输出如下：

*执行事件1!*

*执行事件2!*

*执行事件1!完毕*

可见，异步的事件处理没有阻塞进程，很好的起到了异步方法的作用。



### 5 委托-Func与Action

本文最开始探讨委托，然后直接顺到了事件的相关话题上。其实，关于委托还有一个重点话题漏掉了，那就是Func与Action。

在委托delegate出现了很久以后，微软的.NET设计者们终于领悟到，其实所有的委托定义都可以归纳并简化成只用Func与Action这两个语法糖来表示。其中，Func代表有返回值的委托，Action代表无返回值的委托。有了它们两，我们以后就不再需要用关键字delegate来定义委托了。

同时，若再用lambda表达式取代被委托指向的具体方法，**则整个委托的“定义+赋值”两步将大大简化**（lambda表达式本来也是方法定义的一种简化形式）。

`Func` 是 .NET 提供的一个 **泛型委托类型**，可以表示 **有返回值** 的方法。
 它的定义大致可以理解为：

```
public delegate TResult Func<T1, T2, ..., TResult>(T1 arg1, T2 arg2, ...);
```

也就是说：

- 参数个数可以是 0～16 个；
- 最后一个类型参数始终表示 **返回值类型**；
- 前面的类型参数表示 **输入参数类型**。

下面，把最开始委托章节中关于加减法的程序代码，用Func与lambda表达式进行简化改造，改造后的代码如下：

```text
        //示例：将委托/方法当参数传递
        public static int Test(Func<int, int, int> MD)
        {
            return MD(10, 20);
        }

        static void Main(string[] args)
        {
            int a, b, x, y;

            Func<int, int, int> md;

            //将委托指向加法，并进行相关操作
            md = (t, v) => t + v;
            a = md(1, 2);
            b = Test(md);

            //再将委托指向减法，并进行相关操作
            md = (t, v) => t - v;
            x = md(7, 2);
            y = Test(md);

            Console.WriteLine($"1+2={a},10+20={b},7-2={x},10-20={y}");
            Console.ReadLine();
        }
```