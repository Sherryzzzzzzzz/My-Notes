# C++性能优化——likely和unlikely分支预测

## References

1. [C++之likely和unlikely](https://zhuanlan.zhihu.com/p/357434227)
2. [C++ likely-unlikely-directives](https://medium.com/software-design/likely-unlikely-directives-802c09bd5232)

## 一、背景知识：流水线技术

现代CPU为了提高执行指令执行的吞吐量，使用了流水线技术，它将每条指令分解为多步，让不同指令的各步操作重叠，从而实现若干条指令并行处理。在流水线中，一条指令的生命周期可能包括：

取指：将指令从存储器中读取出来，放入指令缓冲区中。
译码：对取出来的指令进行翻译
执行：知晓了指令内容，便可使用CPU中对应的计算单元执行该指令
访存：将数据从存储器读出，或写入存储器
写回：将指令的执行结果写回到通用寄存器组

流水线技术无法提升CPU执行单条指令的性能，但是可以通过相邻指令的并行化提高整体执行指令的吞吐量。

## 二、分支预测

程序的控制流程基本可分为三种：顺序、分支和循环。对CPU流水线来说，顺序比较好处理，一条路往前趟就行了。但是当程序中有了分支结构之后，CPU无法确切知道到底应该取分支1中的D指令，还是分支二中的E指令。此时CPU会根据指令执行的上下文，猜测那一路分支应该被执行。预测的结果有两个，命中或者命不中。在前一种情况下，CPU流水线正常执行，不会被打断。在后一种情况下，需要CPU丢掉为跳转指令之后的所有指令所做的工作，再开始从正确位置处起始的指令去填充流水线，这会导致很严重的惩罚：大约20-40个时钟周期的浪费，导致程序性能的严重下降。

## 三、likely和unlikely的作用

通过先验概率提示编译器和CPU，提高分支预测准确率，降低预测错误带来的性能损失。

### （1）. 自定义的宏

likely，用于修饰if/else if分支，表示该分支的条件更有可能被满足。而unlikely与之相反。

```c
#define likely(x) __builtin_expect(!!(x), 1) 
#define unlikely(x) __builtin_expect(!!(x), 0)
```

### （2）. 添加likely或unlikely后，汇编代码的差别。

- 无unlikely对应汇编

```c
#include <cstdio>

#define likely(x)       __builtin_expect(!!(x), 1)
#define unlikely(x)     __builtin_expect(!!(x), 0)

int main(int argc, char *argv[])
{
    if (argc > 0) {
        puts ("Positive\n");
    } else
    {
        puts ("Zero or Negative\n");
    }
    return 0;
}
.LC0:
        .string "Positive\n"
.LC1:
        .string "Zero or Negative\n"
main:
        sub     rsp, 8                  # sub rsp,8 — Allocate space on the stack.
        test    edi, edi                # Test the value of argc.
        jle     .L2                     # Jump if less than equal to. This jump is not taken.
        mov     edi, OFFSET FLAT:.LC0   # Copy the pointer to the string "Positive\n" into edi register.
        call    puts                    # Call the puts function.
.L3:
        xor     eax, eax                # Set eaxto 0 (compilers use a self XOR as it is faster than the instruction for setting eax to 0).
        add     rsp, 8                  # Release space on the stack.
        ret                             return from main.
.L2:
        mov     edi, OFFSET FLAT:.LC1   # Copy the pointer to the string "Zero or Negative\n" into edi register.
        call    puts                    # Call the puts function.
        jmp     .L3                     # Unconditional jump to .L3.
```

- 有unlikely对应汇编

```c
#include <cstdio>

#define likely(x)       __builtin_expect(!!(x), 1)
#define unlikely(x)     __builtin_expect(!!(x), 0)

int main(int argc, char *argv[])
{
    if (unlikely(argc > 0)) {
        puts ("Positive\n");
    } else
    {
        puts ("Zero or Negative\n");
    }
    return 0;
}
.LC0:
        .string "Positive\n"
.LC1:
        .string "Zero or Negative\n"
main:
        sub     rsp, 8
        test    edi, edi
        jg      .L6                           # jg .L6 — Jump if greater. This jump is not taken.
        mov     edi, OFFSET FLAT:.LC1
        call    puts
.L3:
        xor     eax, eax
        add     rsp, 8
        ret
.L6:
        mov     edi, OFFSET FLAT:.LC0
        call    puts
        jmp     .L3
```

- 差异总结：添加分支预测后，可以增加分支预测正确性，让更大概率走到的分支对应指令顺序排放在后面，可以减少jmp指令的调用，有助于提高性能。

## 四. C++20开始成为关键字

```c++
int foo(int i) {
    switch(i) {
               case 1: handle1();
                       break;
    [[likely]] case 2: handle2();
                       break;
    }
}
```