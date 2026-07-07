# C# Dictionary 底层原理详解（从冲突到 Rehash）

## 一、Dictionary 的本质

`Dictionary<TKey, TValue>` 本质是一个：

> **基于哈希表（Hash Table）的键值对存储结构**

它的核心目标是：

- 插入：O(1)
- 查找：O(1)

但这个 O(1) 是**平均情况**，背后依赖两个关键机制：

1. 哈希函数（GetHashCode）
2. 冲突处理（链地址法）

------

# 二、整体结构（非常关键）

Dictionary 内部不是一个数组，而是两层结构：

```
int[] buckets;
Entry[] entries;
```

------

## 1. buckets（桶数组）

- 存储的是 **entries 的索引**
- 每个位置代表一个“槽位”

例如：

```
index:   0   1   2   3
value:  -1  -1   2  -1
```

表示：

```
bucket[2] = 2
```

------

## 2. entries（数据存储）

```
struct Entry
{
    int hashCode;
    int next;
    TKey key;
    TValue value;
}
```

关键字段：

- `hashCode`：哈希值
- `next`：**下一个冲突节点的索引**
- `key/value`：数据

------

# 三、哈希冲突是什么

## 定义

> **不同的 key 映射到了同一个 bucket**

即：

```
hash(key1) % N == hash(key2) % N
但 key1 ≠ key2
```

------

## 为什么一定会发生冲突

因为：

> key 的数量通常 > bucket 数量（鸽巢原理）

------

# 四、链地址法（解决冲突的核心）

## 核心思想

> 一个 bucket 可以挂多个元素，用“链表”串起来

------

## 实现方式（重点）

不是传统链表，而是：

> **用数组 + index 模拟链表**

------

## 示例

```
bucket[2] = 1
```

entries：

```
index | key | next
------------------
0     | A   | -1
1     | B   | 2
2     | C   | 0
```

链结构：

```
bucket[2]
   ↓
   B → C → A → null
```

------

## next 的含义

```
next = 下一个节点在 entries 数组中的索引
```

例如：

```
next = 2 → 下一个是 entries[2]
next = -1 → 链结束
```

------

# 五、查找流程（必须掌握）

查找一个 key：

```
int i = buckets[bucketIndex];

while (i >= 0)
{
    if (entries[i].key.Equals(key))
        return entries[i].value;

    i = entries[i].next;
}
```

------

## 执行逻辑

1. 计算 hash
2. 定位 bucket
3. 遍历链表（通过 next）

------

## 复杂度

- 平均：O(1)
- 冲突严重：O(n)

------

# 六、插入流程

插入一个元素：

1. 计算 hash
2. 找到 bucket
3. 遍历链（检查是否已存在）
4. 插入链头

------

## 链头插入（关键）

```
原：
bucket[2] → A → B

插入 C：
C.next = A
bucket[2] = C
```

结果：

```
bucket[2] → C → A → B
```

------

# 七、为什么使用“数组 + index”而不是指针

## 优势

### 1. 连续内存（cache-friendly）

```
entries[0], entries[1], entries[2]
```

------

### 2. 减少 GC 压力

不需要频繁：

```
new Node()
```

------

### 3. 更高性能

相比传统链表：

- 更少跳转
- 更好缓存命中

------

# 八、冲突的本质总结

冲突发生在三层：

------

## 1. 数学层

```
hash(key1) % N == hash(key2) % N
```

------

## 2. 数据结构层

```
多个 key 落在同一个 bucket
```

------

## 3. 运行时层

```
查找需要遍历链表
```

------

# 九、扩容（Resize）为什么必须 Rehash

这是最核心问题之一。

------

## 1. bucket 的计算公式

```
bucketIndex = hash % capacity
```

------

## 2. 扩容前后对比

### 扩容前（capacity = 4）

```
hash = 6 → 6 % 4 = 2
```

------

### 扩容后（capacity = 8）

```
hash = 6 → 6 % 8 = 6
```

------

## 3. 问题

如果直接复制：

```
旧 bucket[2] → 新 bucket[2]
```

但查找时：

```
查找 → 6 % 8 = 6 → 去 bucket[6]
```

👉 找不到数据（逻辑错误）

------

## 结论

> **capacity 改变 → 映射函数改变 → 所有元素位置失效**

------

## 4. 正确做法（rehash）

```
for each entry:
    newBucket = hash % newCapacity
    插入新位置
```

------

## 5. 链也必须重建

扩容后：

- bucket 变化
- next 链全部改变

------

# 十、扩容的意义

## 降低冲突

### 扩容前

```
mod 4：
6 → 2
10 → 2
14 → 2
```

------

### 扩容后

```
mod 8：
6  → 6
10 → 2
14 → 6
```

------

👉 冲突减少，性能提升

------

# 十一、复杂度分析

| 操作 | 平均 | 最坏 |
| ---- | ---- | ---- |
| 查找 | O(1) | O(n) |
| 插入 | O(1) | O(n) |
| 扩容 | O(n) | O(n) |

------

# 十二、性能关键点（工程视角）

## 1. hash 函数质量

差的 hash：

```
return 1;
```

👉 所有元素落一个 bucket → O(n)

------

## 2. capacity 设置

避免频繁扩容：

```
new Dictionary<int, T>(capacity)
```

------

## 3. load factor

```
load = count / capacity
```

越大：

```
冲突越严重
```

------

# 十三、与 C++ unordered_map 对比

| 特性      | C# Dictionary     | C++ unordered_map |
| --------- | ----------------- | ----------------- |
| 冲突处理  | 链地址法（index） | 链地址法（指针）  |
| 内存布局  | 连续              | 分散              |
| cache友好 | 高                | 低                |
| GC影响    | 有                | 无                |
| 扩容      | rehash            | rehash            |

------

# 十四、核心总结（必须记住）

### 1

> Dictionary = 数组 + 哈希 + 链表（index模拟）

------

### 2

> 冲突 = 多个 key 映射到同一个 bucket

------

### 3

> next = 下一个节点在数组中的索引

------

### 4

> 查找 = bucket 定位 + 链表遍历

------

### 5（最重要）

> **位置由 hash % capacity 决定，capacity 一变，所有元素位置必须重算（rehash）**

------

如果你下一步要继续深入，我建议你重点看：

- Dictionary 源码中的 `Resize()` 实现
- `GetHashCode()` 如何影响性能
- 在游戏开发中什么时候不用 Dictionary（这个比原理更重要）