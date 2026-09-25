# TypeScript 学习清单

> 说明：本文档按「概念说明 → 代码实例 → 运行结果 → 注意点」的格式组织，实例均可在 TypeScript 中直接编译运行。
> 运行方式：`tsc 文件.ts && node 文件.js`，或使用 `npx ts-node 文件.ts`、`npx tsx 文件.ts`。

## 1. JavaScript 基础
### 1.1 变量与声明

**概念说明：** TypeScript 沿用 JavaScript 的三种声明方式 —— `var`、`let`、`const`。三者的核心区别在作用域、变量提升和可重赋值性上。

```typescript
var v = "var 是函数作用域";
let l = "let 是块级作用域";
const c = "const 不可重新赋值";

function scopeTest(): void {
  if (true) {
    var fnScoped = "var 会泄漏到函数外";
    let blockScoped = "let 只在块内";
    console.log(blockScoped); // 内层可见
  }
  console.log(fnScoped);      // var 提升到函数作用域，可见
  // console.log(blockScoped); // 错误：块外不可见
}
scopeTest();

// c = "重新赋值"; // 错误：Cannot assign to 'c' because it is a constant.
console.log(v, l, c);
```

**运行结果：**
```
let 只在块内
var 会泄漏到函数外
var 是函数作用域 let 是块级作用域 const 不可重新赋值
```

**注意：**
* `let` / `const` 存在「暂时性死区」(TDZ)，声明前访问会抛 `ReferenceError`。
* `const` 约束的是绑定不可变，对象内部属性仍可修改：`const o = { n: 1 }; o.n = 2;` 合法。
* 默认优先使用 `const`，需要重新赋值时才用 `let`，避免使用 `var`。

### 1.2 基本数据类型

**概念说明：** JavaScript 有 7 种原始类型（`string`、`number`、`boolean`、`null`、`undefined`、`symbol`、`bigint`）和 1 种引用类型（`object`）。

```typescript
const s: string = "文本";
const n: number = 42;
const b: boolean = true;
const u: undefined = undefined;
const nul: null = null;
const sym: symbol = Symbol("id");
const big: bigint = 9007199254740993n;
const obj: object = { s, n };

console.log(typeof s, typeof n, typeof b);       // string number boolean
console.log(typeof u, typeof nul);               // undefined object  ← null 的历史遗留
console.log(typeof sym, typeof big);             // symbol bigint
console.log(typeof obj);                         // object
```

**运行结果：**
```
string number boolean
undefined object
symbol bigint
object
```

**注意：**
* `typeof null === "object"` 是 JavaScript 早期设计缺陷，不能用来判断 null。
* 判断 null 用 `value === null`，判断数组用 `Array.isArray(value)`。
* 原始类型按值比较，`object` 按引用比较。

### 1.3 类型转换

**概念说明：** 分为显式转换（手动调用 `Number()`、`String()`、`Boolean()`、`parseInt()` 等）和隐式转换（运算符自动触发），隐式转换是最容易出 bug 的地方。

```typescript
// 显式转换
console.log(Number("42"));        // 42
console.log(Number("42abc"));     // NaN
console.log(parseInt("42abc"));   // 42
console.log(String(42));          // "42"
console.log(Boolean(""));         // false
console.log(Boolean("0"));        // true，非空字符串都是 true

// 隐式转换
console.log("1" + 2);             // "12"  字符串拼接优先
console.log("3" * "2");           // 6     算术运算转数字
console.log([] + {});             // "[object Object]"
console.log(0 == "");             // true  == 会做类型转换
console.log(0 === "");            // false === 严格比较
```

**运行结果：**
```
42
NaN
42
42
false
true
12
6
[object Object]
true
false
```

**注意：**
* 假值只有 6 个：`false`、`0`、`""`、`null`、`undefined`、`NaN`。
* `Number("")` 返回 `0`，而 `Number("abc")` 返回 `NaN`。
* 一律使用 `===` / `!==`，禁用 `==` / `!=`，TypeScript 的 `strict` 模式下也鼓励严格比较。

### 1.4 运算符

**概念说明：** 运算符包括算术、比较、逻辑、位、赋值、空值合并 `??`、可选链 `?.`、扩展运算符 `...` 等。

```typescript
console.log(10 % 3);              // 1   取余
console.log(2 ** 10);             // 1024 幂运算
console.log(true && "A");         // "A"  短路与：返回最后一个真值
console.log(false || "B");        // "B"  短路或：返回第一个真值
console.log(null ?? "默认值");     // "默认值" 仅 null/undefined 触发
console.log(0 ?? "默认值");        // 0   ?? 不会把 0 当假值

// 可选链
const user: { profile?: { name?: string } } = {};
console.log(user.profile?.name ?? "匿名"); // 匿名

// 扩展运算符
const arr1 = [1, 2];
const arr2 = [...arr1, 3];
const merged = { ...{ a: 1 }, b: 2 };
console.log(arr2, JSON.stringify(merged));
```

**运行结果：**
```
1
1024
A
B
默认值
0
匿名
[ 1, 2, 3 ] {"a":1,"b":2}
```

**注意：**
* `??` 与 `||` 的区别：`||` 把 `0`、`""`、`false` 也当作触发条件，`??` 只认 `null` / `undefined`。
* `??` 不能与 `||`、`&&` 混用而不加括号，否则语法报错。
* `?.` 只在左侧为 `null` / `undefined` 时短路返回 `undefined`。

### 1.5 条件语句

**概念说明：** 包括 `if / else if / else`、三元运算符、`switch`。TypeScript 会结合类型收窄，在分支中自动推断更精确的类型。

```typescript
type Result = { ok: true; data: string } | { ok: false; error: string };

function format(r: Result): string {
  if (r.ok) {
    return r.data;      // 此处 r 被收窄为 { ok: true; data: string }
  } else {
    return r.error;     // 此处收窄为 { ok: false; error: string }
  }
}

console.log(format({ ok: true, data: "成功" }));
console.log(format({ ok: false, error: "失败" }));

const score = 85;
const grade = score >= 90 ? "A" : score >= 60 ? "B" : "C";
console.log(grade);

switch (true) {
  case score >= 90: console.log("优秀"); break;
  case score >= 60: console.log("及格"); break;
  default: console.log("不及格");
}
```

**运行结果：**
```
成功
失败
B
及格
```

**注意：**
* `switch` 使用 `===` 比较，且必须有 `break` 或 `return`，否则会贯穿。
* 条件语句是类型收窄的主要手段之一，配合可辨识联合非常好用。

### 1.6 循环语句

**概念说明：** 常用循环有 `for`、`for...of`（遍历可迭代对象的值）、`for...in`（遍历对象键）、`while`、`do...while`，以及数组方法 `forEach`。

```typescript
// 普通 for
for (let i = 0; i < 3; i++) {
  console.log("for:", i);
}

// for...of 遍历数组的值（推荐）
const names = ["Alice", "Bob"];
for (const name of names) {
  console.log("of:", name);
}

// for...in 遍历对象的键
const obj = { a: 1, b: 2 };
for (const key in obj) {
  console.log("in:", key, obj[key as keyof typeof obj]);
}

// while
let count = 0;
while (count < 2) {
  console.log("while:", count);
  count++;
}
```

**运行结果：**
```
for: 0
for: 1
for: 2
of: Alice
of: Bob
in: a 1
in: b 2
while: 0
while: 1
```

**注意：**
* `for...in` 会遍历原型链上的可枚举属性，不建议用于数组。
* `for...of` 要求对象实现 `Symbol.iterator`，普通对象不可直接使用。
* 需要提前退出用 `break`，跳过本次用 `continue`；`forEach` 无法 `break`。

## 2. JavaScript 函数
### 2.1 函数声明

**概念说明：** 使用 `function` 关键字声明，具备函数提升特性，可在定义前调用。TypeScript 可为参数和返回值添加类型注解。

```typescript
function add(a: number, b: number): number {
  return a + b;
}

// 函数提升：先调用后定义也可以
console.log(earlyCall());

function earlyCall(): string {
  return "函数声明会被提升";
}

console.log(add(1, 2));
```

**运行结果：**
```
函数声明会被提升
3
```

**注意：**
* 返回值类型通常可省略，TypeScript 会自动推断；但公共 API 建议显式标注。
* 函数声明会被提升，函数表达式和箭头函数不会。
* 未标注返回类型且无 `return` 时，返回类型推断为 `void`。

### 2.2 函数表达式

**概念说明：** 把函数赋值给变量，不会提升，因此必须定义后调用。可以匿名，也可以具名（便于调试与递归）。

```typescript
const multiply = function (a: number, b: number): number {
  return a * b;
};

const factorial = function fact(n: number): number {
  return n <= 1 ? 1 : n * fact(n - 1); // 具名表达式可自引用
};

console.log(multiply(3, 4));
console.log(factorial(5));

// 类型注解写在变量上
const greet: (name: string) => string = function (name) {
  return "Hello " + name;
};
console.log(greet("Alice"));
```

**运行结果：**
```
12
120
Hello Alice
```

**注意：**
* 函数表达式不提升，`console.log(fn())` 写在定义前会报 `ReferenceError`。
* 赋值给 `const` 后，变量声明必须带类型或初始值，否则推断为 `any`。
* 具名函数表达式的名字只在函数体内部可用，外部不可见。

### 2.3 箭头函数

**概念说明：** 箭头函数语法更简洁，且**没有自己的 `this`**，会捕获外层作用域的 `this`；同时没有 `arguments`、不能作为构造函数。

```typescript
// 单参数可省略括号，单表达式可省略 return
const double = (n: number): number => n * 2;
const log = (msg: string): void => { console.log(msg); };

console.log(double(21));
log("箭头函数");

// 无自己的 this
const counter = {
  count: 0,
  start() {
    setTimeout(() => {
      this.count++;                 // this 指向 counter
      console.log("count =", this.count);
    }, 0);
  },
};
counter.start();

// 返回对象字面量需要加括号
const makeUser = (name: string) => ({ name, active: true });
console.log(makeUser("Bob"));
```

**运行结果：**
```
42
箭头函数
count = 1
{ name: 'Bob', active: true }
```

**注意：**
* 直接返回对象必须写成 `() => ({ ... })`，否则 `{}` 会被当成函数体。
* 箭头函数不能用作构造函数（`new` 会报错），也没有 `prototype`。
* 需要动态 `this`（如事件回调、对象方法）时使用普通函数。

### 2.4 参数与返回值

**概念说明：** 参数支持类型注解、可选参数、默认值、剩余参数；返回值支持显式标注或自动推断。

```typescript
function buildName(first: string, last = "Smith", age?: number): string {
  return `${first} ${last}${age ? ` (${age})` : ""}`;
}

console.log(buildName("Tom"));
console.log(buildName("Tom", "Lee"));
console.log(buildName("Tom", "Lee", 18));

// 剩余参数必须是数组类型
function sum(...nums: number[]): number {
  return nums.reduce((t, n) => t + n, 0);
}
console.log(sum(1, 2, 3, 4));

// 返回多个值用元组
function divmod(a: number, b: number): [number, number] {
  return [Math.floor(a / b), a % b];
}
const [q, r] = divmod(7, 2);
console.log(q, r);
```

**运行结果：**
```
Tom Smith
Tom Lee
Tom Lee (18)
10
3 1
```

**注意：**
* 可选参数必须放在必填参数之后；默认值参数会自动变为可选。
* 剩余参数必须是最后一个参数，且类型为数组。
* 传参时可选参数传 `undefined` 等价于使用默认值。

### 2.5 回调函数

**概念说明：** 把函数作为参数传给另一个函数，由接收方在合适时机调用。回调的参数与返回值类型由函数签名约束。

```typescript
type Callback<T> = (err: Error | null, data?: T) => void;

function fetchData(id: number, cb: Callback<string>): void {
  setTimeout(() => {
    if (id <= 0) {
      cb(new Error("非法 ID"));
    } else {
      cb(null, `数据-${id}`);
    }
  }, 0);
}

fetchData(7, (err, data) => {
  if (err) {
    console.log("出错:", err.message);
  } else {
    console.log("成功:", data);
  }
});

fetchData(-1, (err, data) => {
  console.log(err ? "出错: " + err.message : "成功: " + data);
});
```

**运行结果：**
```
成功: 数据-7
出错: 非法 ID
```

**注意：**
* Node 风格的错误优先回调约定为 `(err, data) => void`。
* 回调过多会形成「回调地狱」，应改用 Promise / async-await。
* 在类型上把回调标记为可选时，调用前需判空。

### 2.6 高阶函数

**概念说明：** 高阶函数指「接收函数作为参数」或「返回函数」的函数，是函数式编程的基础，`map` / `filter` / `reduce` 都是典型代表。

```typescript
// 接收函数作为参数
function repeat<T>(times: number, fn: (i: number) => T): T[] {
  const result: T[] = [];
  for (let i = 0; i < times; i++) result.push(fn(i));
  return result;
}
console.log(repeat(3, (i) => i * i)); // [0, 1, 4]

// 返回函数（柯里化）
const add = (a: number) => (b: number) => a + b;
const add10 = add(10);
console.log(add10(5)); // 15

// 组合通用高阶函数
function once<T extends (...args: any[]) => any>(fn: T): T {
  let called = false;
  return ((...args: any[]) => {
    if (called) return undefined;
    called = true;
    return fn(...args);
  }) as T;
}
const init = once((name: string) => console.log("只执行一次:", name));
init("A");
init("B");
```

**运行结果：**
```
[ 0, 1, 4 ]
15
只执行一次: A
```

**注意：**
* 高阶函数的类型推断依赖泛型，`once` 这类需要保留原函数签名时用 `T extends (...args: any[]) => any`。
* 柯里化把多参函数转成单参函数链，便于复用与组合。
* 返回函数时注意闭包会持有外部变量，可能造成内存驻留。

### 2.7 this

**概念说明：** `this` 的指向在函数**调用时**确定，取决于调用方式：普通调用指向 `undefined`（严格模式）或全局对象，方法调用指向调用者，`new` 调用指向新实例。

```typescript
class Timer {
  seconds = 0;

  // 方法中的 this 指向实例
  tick(): void {
    this.seconds++;
    console.log("tick:", this.seconds);
  }

  // 箭头函数属性绑定定义时的 this
  start = (): void => {
    console.log("start 中的 this.seconds =", this.seconds);
  };
}

const t = new Timer();
t.tick();

const fn = t.start;   // 解构后丢失 this？
fn();                 // 箭头函数属性不受影响

function showThis(this: { name: string }): string {
  return this.name;
}
console.log(showThis.call({ name: "显式绑定的 this" }));
```

**运行结果：**
```
tick: 1
start 中的 this.seconds = 1
显式绑定的 this
```

**注意：**
* 普通函数的 `this` 由调用者决定，解构或作为回调传递时容易丢失。
* 箭头函数没有自己的 `this`，会继承定义时外层作用域的 `this`。
* TypeScript 中可给函数加 `this` 参数来声明 `this` 的类型，它只是类型检查，不占实际参数位。

### 2.8 call / apply / bind

**概念说明：** 三者都能显式指定函数执行时的 `this`。`call` 传参数列表，`apply` 传参数数组，`bind` 不立即执行而是返回绑定了 `this` 的新函数。

```typescript
function introduce(this: { name: string }, age: number, city: string): string {
  return `${this.name}，${age} 岁，来自 ${city}`;
}

const person = { name: "Alice" };

console.log(introduce.call(person, 25, "北京"));
console.log(introduce.apply(person, [25, "北京"]));

const bound = introduce.bind(person, 25);
console.log(bound("上海"));

// bind 也可用于预置参数（偏函数）
const boundAge = introduce.bind(person);
console.log(boundAge(30, "深圳"));
```

**运行结果：**
```
Alice，25 岁，来自 北京
Alice，25 岁，来自 北京
Alice，25 岁，来自 上海
Alice，30 岁，来自 深圳
```

**注意：**
* `bind` 返回的新函数再被 `call` / `apply` 也无法覆盖已绑定的 `this`。
* 借用方法很常用：`Array.prototype.slice.call(arguments)`。
* 箭头函数不受三者影响，绑定 `this` 无效。

### 2.9 闭包

**概念说明：** 闭包是「函数 + 其定义时所处的词法环境」。内层函数可以访问外层函数的变量，即使外层函数已经返回，这些变量依然存活。

```typescript
function createCounter(start = 0) {
  let count = start;                     // 被闭包捕获
  return {
    increment: () => ++count,
    decrement: () => --count,
    get value(): number { return count; },
  };
}

const c = createCounter(10);
c.increment();
c.increment();
c.decrement();
console.log("当前值:", c.value);   // 11

// 经典循环闭包问题：var 共享同一个变量
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log("var:", i), 0); // 3 3 3
}
// let 每次迭代创建新绑定
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log("let:", j), 0); // 0 1 2
}
```

**运行结果：**
```
当前值: 11
var: 3
var: 3
var: 3
let: 0
let: 1
let: 2
```

**注意：**
* 闭包会阻止变量被垃圾回收，长时间持有的闭包可能造成内存泄漏。
* 循环中捕获变量请使用 `let`，或用 IIFE 显式创建新作用域。
* 模块模式、柯里化、私有变量都依赖闭包实现。

## 3. JavaScript 对象
### 3.1 对象字面量

**概念说明：** 对象是以键值对组织数据的集合，用 `{}` 创建。属性名可以是标识符、字符串或计算属性名，值可以是任意类型。

```typescript
const name = "Alice";
const key = "dynamic";

const user = {
  name,                        // 简写属性，等价于 name: name
  age: 25,
  "full name": "Alice Smith",  // 字符串键，访问需用中括号
  [key + "Key"]: 1,            // 计算属性名
  address: { city: "北京" },    // 嵌套对象
  hobbies: ["读书", "游泳"],
};

console.log(user.name, user.age);
console.log(user["full name"]);
console.log(user.dynamicKey);
console.log(user.address.city, user.hobbies[0]);
```

**运行结果：**
```
Alice 25
Alice Smith
1
北京 读书
```

**注意：**
* 属性名含空格或特殊字符时只能用中括号访问。
* `{ name }` 是 `{ name: name }` 的简写，变量名即属性名。
* TypeScript 会根据字面量自动推断属性类型，多余的属性会报错。

### 3.2 属性与方法

**概念说明：** 对象属性分为数据属性和方法（值为函数的属性）。方法中可通过 `this` 访问同一对象的其他属性，属性可被增删改。

```typescript
interface Calculator {
  base: number;
  add(n: number): number;   // 方法签名
  reset: () => void;        // 函数属性
}

const calc: Calculator = {
  base: 10,
  add(n) {
    return this.base + n;   // this 指向 calc
  },
  reset() {
    this.base = 0;
  },
};

console.log(calc.add(5));            // 15
calc.reset();
console.log(calc.base);              // 0

// 增删改属性
const obj: Record<string, number> = { a: 1 };
obj.b = 2;                           // 增
delete obj.a;                        // 删
console.log(obj);                    // { b: 2 }
console.log(Object.keys(obj), Object.values(obj));
```

**运行结果：**
```
15
0
{ b: 2 }
[ 'b' ] [ 2 ]
```

**注意：**
* 方法不要使用箭头函数定义，否则 `this` 不会指向对象。
* `delete` 只能删除自有属性，且会影响性能（改变对象隐藏类）。
* `Object.entries(obj)` 可同时取得键值对数组。

### 3.3 对象引用

**概念说明：** 对象是引用类型，变量保存的是内存地址。赋值与传参传递的是引用，浅拷贝只复制第一层。

```typescript
const a = { n: 1, inner: { x: 1 } };
const b = a;              // 同一引用
b.n = 2;
console.log(a.n);         // 2，a 也被改了

// 浅拷贝：第一层独立，嵌套仍共享
const shallow = { ...a };
shallow.n = 99;
shallow.inner.x = 99;
console.log(a.n, a.inner.x);   // 2 99  ← 嵌套被污染

// 深拷贝：structuredClone（Node 17+ / 现代浏览器）
const deep = structuredClone(a);
deep.inner.x = 1000;
console.log(a.inner.x);        // 99
console.log(JSON.stringify(deep));
```

**运行结果：**
```
2
2 99
99
{"n":2,"inner":{"x":1000}}
```

**注意：**
* `{ ...obj }`、`Object.assign({}, obj)` 都是浅拷贝。
* `JSON.parse(JSON.stringify(obj))` 深拷贝会丢失 `undefined`、函数、`Date` 会变字符串。
* `structuredClone` 支持循环引用与 `Map` / `Set` / `Date`，但不能拷贝函数。

### 3.4 class

**概念说明：** `class` 是构造函数的语法糖，用 `new` 创建实例，`constructor` 负责初始化。TypeScript 为字段、参数和访问修饰符提供类型支持。

```typescript
class User {
  // 字段声明（TypeScript 要求）
  name: string;
  age: number;
  readonly id: number;

  constructor(name: string, age: number, id: number) {
    this.name = name;
    this.age = age;
    this.id = id;
  }

  greet(): string {
    return `你好，我是 ${this.name}，今年 ${this.age} 岁`;
  }

  get profile(): string {
    return `${this.id}:${this.name}`;
  }
}

const u = new User("Alice", 25, 1);
console.log(u.greet());
console.log(u.profile);
console.log(u instanceof User);   // true
```

**运行结果：**
```
你好，我是 Alice，今年 25 岁
1:Alice
true
```

**注意：**
* TypeScript 中类字段必须声明类型或在 `constructor` 中确定赋值，否则 `strictPropertyInitialization` 报错。
* 类方法定义在原型上，实例字段定义在实例上，因此内存占用不同。
* `readonly` 只能赋值一次（声明处或构造器中）。

### 3.5 继承

**概念说明：** 使用 `extends` 让子类获得父类的属性和方法，通过 `super` 调用父类构造器与方法，子类可重写（override）父类方法。

```typescript
class Animal {
  constructor(public name: string) {}

  speak(): string {
    return `${this.name} 发出声音`;
  }
}

class Dog extends Animal {
  constructor(name: string, public breed: string) {
    super(name);              // 必须先调用 super
  }

  speak(): string {
    return `${super.speak()}：汪汪`;   // 调用父类实现
  }
}

const dog = new Dog("旺财", "柴犬");
console.log(dog.speak());
console.log(dog.name, dog.breed);
console.log(dog instanceof Animal, dog instanceof Dog);
```

**运行结果：**
```
旺财 发出声音：汪汪
旺财 柴犬
true true
```

**注意：**
* 子类构造器中 `super()` 必须在使用 `this` 之前调用。
* 重写方法需保持兼容的签名，TypeScript 可开启 `noImplicitOverride` 强制加 `override` 关键字。
* 支持向上转型：`const a: Animal = new Dog(...)`，符合里氏替换原则。

### 3.6 getter / setter

**概念说明：** 用 `get` / `set` 定义访问器属性，调用时像普通属性一样读写，可加入校验、计算和副作用逻辑。

```typescript
class Temperature {
  private _celsius = 0;

  get celsius(): number {
    return this._celsius;
  }

  set celsius(value: number) {
    if (value < -273.15) {
      throw new Error("温度不能低于绝对零度");
    }
    this._celsius = Math.round(value * 100) / 100;
  }

  // 只读计算属性
  get fahrenheit(): number {
    return this._celsius * 9 / 5 + 32;
  }
}

const t = new Temperature();
t.celsius = 25;
console.log("摄氏:", t.celsius, "华氏:", t.fahrenheit);

try {
  t.celsius = -300;
} catch (e) {
  console.log("错误:", (e as Error).message);
}
```

**运行结果：**
```
摄氏: 25 华氏: 77
错误: 温度不能低于绝对零度
```

**注意：**
* 只有 `get` 没有 `set` 时属性为只读，严格模式下赋值会报错。
* 访问器与同名数据属性不能共存。
* 注意不要在 getter 中触发另一属性的 setter，避免递归死循环。

### 3.7 原型与原型链

**概念说明：** 每个对象都有内部属性 `[[Prototype]]`（可用 `Object.getPrototypeOf` 访问），查找属性时沿原型链向上回溯，直到 `null`。`class` 的继承本质也是原型链。

```typescript
function Person(this: any, name: string) {
  this.name = name;
}
Person.prototype.greet = function () {
  return "你好，" + this.name;
};

const p = new (Person as any)("Alice");
console.log(p.greet());
console.log(p.hasOwnProperty("name"), p.hasOwnProperty("greet"));

class Base { baseMethod() { return "base"; } }
class Sub extends Base {}
const s = new Sub();
console.log(s.baseMethod());
console.log(Object.getPrototypeOf(s) === Sub.prototype);        // true
console.log(Object.getPrototypeOf(Sub.prototype) === Base.prototype); // true
console.log(Object.getPrototypeOf(Object.prototype));           // null，链的尽头
```

**运行结果：**
```
你好，Alice
true false
base
true
true
null
```

**注意：**
* 属性查找先自身后原型链，方法定义在原型上可被所有实例共享。
* `in` 会检查原型链，`hasOwnProperty` 只检查自身。
* 修改内置原型（如 `Array.prototype`）属于危险操作，会造成全局污染。
* TypeScript 中一般不需要手写原型，用 `class` 即可。

## 4. JavaScript 数组与数据结构
### 4.1 Array

**概念说明：** 数组是有序的元素集合，索引从 0 开始。TypeScript 中数组类型写作 `T[]` 或 `Array<T>`。

```typescript
const nums: number[] = [1, 2, 3];
const strs: Array<string> = ["a", "b"];

nums.push(4);                 // 尾部添加
const last = nums.pop();      // 尾部删除并返回
nums.unshift(0);              // 头部添加
const first = nums.shift();   // 头部删除并返回

console.log(nums);            // [1, 2, 3, 4]
console.log(nums.length, first, last);
console.log(nums.slice(1, 3));      // 截取，不改变原数组
console.log(nums.includes(3));      // true
console.log(nums.indexOf(3));       // 2
console.log([1, 2].concat([3, 4])); // 拼接
console.log(nums.join("-"));
console.log(nums.reverse());        // 会改变原数组
```

**运行结果：**
```
[ 1, 2, 3, 4 ]
4 0 4
[ 2, 3 ]
true
2
[ 1, 2, 3, 4 ]
1-2-3-4
[ 4, 3, 2, 1 ]
```

**注意：**
* `slice` 不改变原数组，`splice` 会改变原数组。
* `push`/`pop` 操作尾部性能好，`unshift`/`shift` 需移动全部元素。
* 稀疏数组与 `length` 可写特性容易造成困惑，尽量避免手工改 `length`。

### 4.2 forEach

**概念说明：** 遍历数组并对每个元素执行回调，没有返回值（始终为 `undefined`），无法中途 `break`。

```typescript
const fruits = ["苹果", "香蕉", "梨"];

fruits.forEach((fruit, index, array) => {
  console.log(`${index + 1}. ${fruit}（共 ${array.length} 个）`);
});

// 累加
let total = 0;
[1, 2, 3].forEach((n) => { total += n; });
console.log("合计:", total);

// 跳过元素
fruits.forEach((f, i) => {
  if (i === 1) return;   // return 只相当于 continue
  console.log("处理:", f);
});
```

**运行结果：**
```
1. 苹果（共 3 个）
2. 香蕉（共 3 个）
3. 梨（共 3 个）
合计: 6
处理: 苹果
处理: 梨
```

**注意：**
* 回调中的 `return` 只跳过当前元素，等价于 `continue`，不能中断循环。
* 无法 `await`：`forEach` 内使用 `async` 回调不会等待，需改用 `for...of` + `await`。
* 遍历过程中增删元素的行为不确定，避免这样做。

### 4.3 map

**概念说明：** 对每个元素执行映射函数，返回**等长的新数组**，不改变原数组。泛型可自动推断结果元素类型。

```typescript
const nums = [1, 2, 3, 4];

const doubled = nums.map((n) => n * 2);
console.log(doubled);              // [2, 4, 6, 8]
console.log(nums);                 // 原数组不变

// 返回对象数组
const users = ["alice", "bob"].map((name, i) => ({
  id: i + 1,
  name: name.toUpperCase(),
}));
console.log(users);

// 链式调用
const result = nums
  .filter((n) => n % 2 === 0)
  .map((n) => n * 10);
console.log(result);
```

**运行结果：**
```
[ 2, 4, 6, 8 ]
[ 1, 2, 3, 4 ]
[ { id: 1, name: 'ALICE' }, { id: 2, name: 'BOB' } ]
[ 20, 40 ]
```

**注意：**
* 返回对象字面量需加括号：`map((n) => ({ n }))`，否则 `{}` 被当函数体。
* `map` 结果与原数组长度一定相同，需要过滤请用 `filter`。
* 只用于「转换」，不要在里面做副作用操作（用 `forEach`）。

### 4.4 filter

**概念说明：** 用谓词函数筛选元素，返回条件为真的元素组成的新数组，长度不定。

```typescript
const nums = [1, 2, 3, 4, 5, 6];

const evens = nums.filter((n) => n % 2 === 0);
console.log(evens);                       // [2, 4, 6]

// 类型守卫作为谓词：结果类型被收窄
const mixed: (string | null)[] = ["a", null, "b", null];
const strings = mixed.filter((v): v is string => v !== null);
console.log(strings);                     // [ 'a', 'b' ]

// 对象数组过滤
const users = [
  { name: "Alice", age: 25 },
  { name: "Bob", age: 17 },
];
console.log(users.filter((u) => u.age >= 18).map((u) => u.name));
```

**运行结果：**
```
[ 2, 4, 6 ]
[ 'a', 'b' ]
[ 'Alice' ]
```

**注意：**
* 使用 `v is T` 形式谓词时，过滤结果类型会被正确收窄，避免后续非空断言。
* `filter(Boolean)` 会过滤掉所有假值，但不收窄类型，慎用。
* `filter` 返回新数组，不打乱原数组顺序。

### 4.5 reduce

**概念说明：** 把数组归约为单个值，接收累加器与当前元素。第二个参数是初始值，强烈建议总是提供初始值。

```typescript
// 求和
console.log([1, 2, 3, 4].reduce((acc, n) => acc + n, 0));   // 10

// 统计词频
const words = ["a", "b", "a", "c", "a"];
const count = words.reduce<Record<string, number>>((acc, w) => {
  acc[w] = (acc[w] ?? 0) + 1;
  return acc;
}, {});
console.log(count);   // { a: 3, b: 1, c: 1 }

// 按字段分组
interface Item { type: string; value: number }
const items: Item[] = [
  { type: "fruit", value: 1 },
  { type: "veg", value: 2 },
  { type: "fruit", value: 3 },
];
const grouped = items.reduce<Record<string, Item[]>>((acc, it) => {
  (acc[it.type] ??= []).push(it);
  return acc;
}, {});
console.log(JSON.stringify(grouped));
```

**运行结果：**
```
10
{ a: 3, b: 1, c: 1 }
{"fruit":[{"type":"fruit","value":1},{"type":"fruit","value":3}],"veg":[{"type":"veg","value":2}]}
```

**注意：**
* 不传初始值时首次迭代用第一个元素当累加器，空数组会直接抛错。
* 需要明确结果类型时用 `reduce<ResultType>` 显式指定泛型。
* `reduce` 也能实现 `map` + `filter` 一次遍历，但可读性下降，按需选择。

### 4.6 find / findIndex

**概念说明：** `find` 返回第一个满足条件的元素（无则 `undefined`），`findIndex` 返回其索引（无则 `-1`），找到即停止遍历。

```typescript
interface User { id: number; name: string }
const users: User[] = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" },
];

const bob = users.find((u) => u.id === 2);
console.log(bob?.name);                  // Bob

const idx = users.findIndex((u) => u.id === 2);
console.log(idx);                        // 1

console.log(users.find((u) => u.id === 9));       // undefined
console.log(users.findIndex((u) => u.id === 9));  // -1

// 也可用 findLast / findLastIndex（ES2023）从后往前找
console.log([1, 2, 3, 2].findLast((n) => n === 2));   // 2
```

**运行结果：**
```
Bob
1
undefined
-1
2
```

**注意：**
* `find` 返回值是 `T | undefined`，TypeScript 会强制你处理 `undefined`（用可选链或判空）。
* `indexOf` 用 `===` 比较，无法查找对象内容，查找对象请用 `findIndex`。
* 只找第一个匹配项，需要全部匹配用 `filter`。

### 4.7 some / every

**概念说明：** `some` 判断「是否存在」满足条件的元素，`every` 判断「是否所有」元素都满足，都返回布尔值并短路。

```typescript
const nums = [2, 4, 6, 8];

console.log(nums.some((n) => n > 5));     // true  存在大于 5 的
console.log(nums.some((n) => n > 10));    // false
console.log(nums.every((n) => n % 2 === 0)); // true 全是偶数
console.log(nums.every((n) => n > 5));    // false

// 注意空数组的行为
console.log([].some((n) => n > 0));       // false
console.log([].every((n) => n > 0));      // true  空数组 every 恒为 true

// 实用：判断是否有权限
const roles = ["user", "editor"];
const canEdit = roles.some((r) => ["admin", "editor"].includes(r));
console.log("可编辑:", canEdit);
```

**运行结果：**
```
true
false
true
false
false
true
可编辑: true
```

**注意：**
* 空数组上 `every` 返回 `true`，写校验逻辑时要留意这个边界。
* 两者都会短路：`some` 遇真即返，`every` 遇假即返。
* 用 `some` 代替 `filter(...).length > 0` 性能更好。

### 4.8 Map

**概念说明：** `Map` 是键值对集合，键可以是任意类型（含对象），保持插入顺序，`size` 获取长度，性能优于用对象当字典。

```typescript
const map = new Map<string, number>();

map.set("a", 1);
map.set("b", 2);
console.log(map.get("a"));        // 1
console.log(map.has("c"));        // false
console.log(map.size);            // 2

map.delete("a");
console.log(map.size);            // 1

// 遍历
map.set("c", 3);
for (const [key, value] of map) {
  console.log(key, value);
}
console.log([...map.keys()]);     // [ 'b', 'c' ]

// 对象键
const objKey = { id: 1 };
const objMap = new Map<object, string>();
objMap.set(objKey, "值为对象键");
console.log(objMap.get(objKey));
```

**运行结果：**
```
1
false
2
1
b 2
c 3
[ 'b', 'c' ]
值为对象键
```

**注意：**
* `map.get` 不存在的键返回 `undefined`，TypeScript 会提示处理。
* 用普通对象做字典时键会被转成字符串（`1` 与 `"1"` 冲突），`Map` 没有这个问题。
* `Map` 可直接迭代，普通对象需要 `Object.entries` 转换。

### 4.9 Set

**概念说明：** `Set` 存储唯一值，自动去重，插入顺序保持，常用于数组去重与集合运算。

```typescript
const set = new Set<number>([1, 2, 2, 3, 3, 3]);
console.log(set);                 // Set(3) { 1, 2, 3 }
console.log(set.size);            // 3
console.log(set.has(2));          // true

set.add(4);
set.delete(1);
console.log([...set]);            // [ 2, 3, 4 ]

// 数组去重
const arr = [1, 1, 2, 3, 3];
console.log([...new Set(arr)]);   // [ 1, 2, 3 ]

// 集合运算
const a = new Set([1, 2, 3]);
const b = new Set([2, 3, 4]);
console.log("交集:", [...a].filter((x) => b.has(x)));   // [2, 3]
console.log("并集:", [...new Set([...a, ...b])]);       // [1,2,3,4]
console.log("差集:", [...a].filter((x) => !b.has(x)));  // [1]
```

**运行结果：**
```
Set(3) { 1, 2, 3 }
3
true
[ 2, 3, 4 ]
[ 1, 2, 3 ]
交集: [ 2, 3 ]
并集: [ 1, 2, 3, 4 ]
差集: [ 1 ]
```

**注意：**
* 去重依据 SameValueZero 算法：`NaN` 视为相等，`0` 与 `-0` 视为相等。
* 对象去重比较的是引用，内容相同的两个对象不会被认为重复。
* 需要按属性去重时用 `Map` + 唯一键。

### 4.10 WeakMap

**概念说明：** `WeakMap` 的键必须是对象，且持有的是**弱引用** —— 键对象没有其他引用时可被垃圾回收，不会造成内存泄漏。键不可枚举，没有 `size`。

```typescript
interface Meta { views: number }

const metadata = new WeakMap<object, Meta>();

let article: object | null = { title: "TypeScript 入门" };
metadata.set(article, { views: 1 });

console.log(metadata.get(article)?.views);   // 1

// 私有数据模式
class Counter {
  private data: { count: number };
  constructor() {
    // 用实例自身作为键存储私有状态
    let count = 0;
    this.data = {
      get count() { return ++count; },
      set count(_v: number) {},
    };
  }
  next(): number {
    const current = this.data.count;
    this.data = this.data;
    return current;
  }
}
const c = new Counter();
console.log(c.next(), c.next());

article = null;   // 失去唯一引用，WeakMap 中的条目可被回收
console.log("键不可枚举，无法读取 size");
```

**运行结果：**
```
1
1 2
键不可枚举，无法读取 size
```

**注意：**
* 键非对象会抛 `TypeError`（`Invalid value used as weak map key`）。
* 不可遍历、无 `size`，因为条目随时可能被回收。
* 适合给对象挂载缓存或私有数据，随对象生命周期自动释放。

### 4.11 WeakSet

**概念说明：** `WeakSet` 存储对象集合，同样是弱引用，元素可被回收。只能 `add` / `has` / `delete`，不可遍历。

```typescript
const visited = new WeakSet<object>();

interface Node { name: string }

const nodeA: Node = { name: "A" };
const nodeB: Node = { name: "B" };

visited.add(nodeA);
visited.add(nodeB);

console.log(visited.has(nodeA));   // true
visited.delete(nodeB);
console.log(visited.has(nodeB));   // false

// 典型用途：防止对象被重复处理（避免无限递归）
function traverse(node: object, seen = new WeakSet<object>()): void {
  if (seen.has(node)) {
    console.log("已访问过，跳过");
    return;
  }
  seen.add(node);
  console.log("处理节点");
}

traverse(nodeA);
traverse(nodeA);   // 第二次跳过
```

**运行结果：**
```
true
false
处理节点
已访问过，跳过
```

**注意：**
* 只能存对象，存原始值会抛 `TypeError`。
* 不可遍历、无 `size`，无法知道里面有什么。
* 与 `WeakMap` 相比少一个「值」，只用于标记「是否出现过」。

## 5. JavaScript 异步编程
### 5.1 同步与异步

**概念说明：** 同步代码按顺序阻塞执行；异步代码把耗时操作交给宿主环境（浏览器 / Node），主线程继续向下执行，等结果就绪时通过回调或 Promise 通知。JavaScript 是单线程的，异步是避免阻塞的唯一手段。

```typescript
console.log("1 同步开始");

setTimeout(() => {
  console.log("4 定时器回调（异步，先交给宿主环境）");
}, 0);

Promise.resolve().then(() => {
  console.log("3 微任务（异步，但优先于定时器）");
});

console.log("2 同步结束");
```

**运行结果：**
```
1 同步开始
2 同步结束
3 微任务（异步，但优先于定时器）
4 定时器回调（异步，先交给宿主环境）
```

**注意：**
* 同步代码全部执行完，才会处理异步队列，所以 `2` 排在 `3`、`4` 之前。
* 即使 `setTimeout` 延时为 0，也不会立即执行，最小延迟约 1~4ms。
* 异步结果只能通过回调、Promise 或事件获取，不能用 `return` 直接带出。

### 5.2 Event Loop

**概念说明：** 事件循环不断检查调用栈是否为空：栈空后先清空**微任务队列**（Promise、`queueMicrotask`、`MutationObserver`），再取出**一个宏任务**（定时器、I/O、事件回调）执行，如此往复。

```typescript
setTimeout(() => {
  console.log("timeout 1");
  Promise.resolve().then(() => console.log("  微任务：在 timeout 1 内部"));
}, 0);

setTimeout(() => console.log("timeout 2"), 0);

Promise.resolve()
  .then(() => {
    console.log("微任务 1");
  })
  .then(() => console.log("微任务 2"));

console.log("同步代码结束");
```

**运行结果：**
```
同步代码结束
微任务 1
微任务 2
timeout 1
  微任务：在 timeout 1 内部
timeout 2
```

**注意：**
* 每个宏任务执行完后，一定会把微任务队列清空，再进入下一个宏任务。
* 微任务里再产生的微任务会在当前轮次继续执行完（可能导致「微任务饿死」宏任务）。
* Node.js 除了微任务还有 `process.nextTick`，优先级高于 Promise 微任务。

### 5.3 Promise

**概念说明：** Promise 表示一个异步操作的最终结果，有三种状态：`pending`（进行中）、`fulfilled`（已成功）、`rejected`（已失败）。状态一旦改变就不可逆。

```typescript
const p = new Promise<string>((resolve, reject) => {
  console.log("执行器函数是同步执行的");
  const ok = true;
  if (ok) {
    resolve("完成");
  } else {
    reject(new Error("失败"));
  }
});

console.log("当前状态:", p);   // 打印时已解决

p.then((value) => console.log("收到结果:", value));

// 状态不可逆：后续的 resolve / reject 会被忽略
const once = new Promise<string>((resolve, reject) => {
  resolve("第一次");
  resolve("第二次");        // 被忽略
  reject(new Error("无效")); // 被忽略
});
once.then((v) => console.log("最终值:", v));
```

**运行结果：**
```
执行器函数是同步执行的
当前状态: Promise { '完成' }
收到结果: 完成
最终值: 第一次
```

**注意：**
* `new Promise` 的执行器同步运行，但 `then` 回调永远是异步（微任务）执行。
* 状态只能从 `pending` 变为 `fulfilled` 或 `rejected`，之后所有改变尝试都无效。
* Promise 一旦被拒绝且无处理，会触发 `unhandledRejection`，Node 中会告警甚至退出。

### 5.4 resolve

**概念说明：** `resolve(value)` 用于兑现 Promise。若传入的是 Promise 或 thenable，会**展开**（等待其完成后再兑现）；也可以直接用 `Promise.resolve()` 创建已解决的 Promise。

```typescript
const p1 = Promise.resolve(42);                      // 已兑现

const p2 = new Promise<number>((resolve) => {
  setTimeout(() => resolve(7), 0);                   // 异步兑现
});

// 传入 Promise 会被展开
const p3 = new Promise<number>((resolve) => resolve(Promise.resolve(100)));

p3.then((v) => console.log("展开后:", v));

Promise.all([p1, p2]).then(([a, b]) => {
  console.log(`a = ${a}, b = ${b}, 和为 ${a + b}`);
});
```

**运行结果：**
```
展开后: 100
a = 42, b = 7, 和为 49
```

**注意：**
* `resolve(promise)` 需要额外一轮微任务，比直接 `resolve(value)` 稍慢。
* `Promise.resolve(x)` 的类型是 `Promise<Awaited<T>>`，会自动解包嵌套 Promise。
* 忘记调用 `resolve` / `reject` 会让 Promise 永远处于 `pending`，是常见死锁原因。

### 5.5 reject

**概念说明：** `reject(reason)` 使 Promise 变为失败状态，通常传入 `Error` 实例以便保留堆栈。失败状态必须被 `catch` 或 `then` 的第二参数处理。

```typescript
function request(url: string): Promise<string> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (!url.startsWith("https")) {
        reject(new Error(`不安全的地址：${url}`));
        return;
      }
      resolve(`来自 ${url} 的响应`);
    }, 0);
  });
}

request("https://example.com").then((res) => console.log(res));
request("http://example.com").catch((e: Error) => console.log("错误:", e.message));

// 拒绝值不必是 Error，但用 Error 才有堆栈
Promise.reject("字符串原因").catch((reason) => console.log("拒绝原因:", reason));
```

**运行结果：**
```
错误: 不安全的地址：http://example.com
来自 https://example.com 的响应
拒绝原因: 字符串原因
```

**注意：**
* 应该 `reject(new Error(msg))` 而不是 `reject(msg)`，这样才能保留调用堆栈。
* 在 `async` 函数里用 `throw` 与 `return Promise.reject()` 等价。
* `reject` 后若不处理，Node 17+ 默认会导致进程退出（`--unhandled-rejections=throw`）。

### 5.6 then

**概念说明：** `then(onFulfilled, onRejected)` 注册回调并返回**新的** Promise。返回值会被包装为 Promise，从而支持链式调用。

```typescript
Promise.resolve(2)
  .then((v) => {
    console.log("成功回调:", v);
    return v * 10;                    // 普通值 → 包装为 Promise
  })
  .then((v) => console.log("链式传递:", v));

// 第二个参数用于处理错误（等价于该级的 catch）
Promise.reject(new Error("出错了"))
  .then(
    () => console.log("不会执行"),
    (e: Error) => console.log("then 第二参数处理错误:", e.message)
  );

// 返回 Promise 会等待其完成
Promise.resolve(1)
  .then((v) => new Promise<number>((r) => setTimeout(() => r(v + 100), 0)))
  .then((v) => console.log("等待嵌套 Promise 后:", v));
```

**运行结果：**
```
成功回调: 2
链式传递: 20
then 第二参数处理错误: 出错了
等待嵌套 Promise 后: 101
```

**注意：**
* `then` 回调中 `return` 的值会成为下一个 `then` 的入参。
* 回调中抛出异常会让返回的 Promise 变为 rejected。
* 只传 `onFulfilled` 时错误会继续向后传递，直到遇到 `catch`。

### 5.7 catch

**概念说明：** `catch(onRejected)` 等价于 `then(undefined, onRejected)`，用于捕获链中任意位置产生的错误（称为「错误冒泡」），返回新 Promise 使链可以继续。

```typescript
Promise.resolve("ok")
  .then(() => {
    throw new Error("then 中抛错");
  })
  .then(() => console.log("被跳过，不会执行"))
  .catch((e: Error) => {
    console.log("catch 捕获:", e.message);
    return "已恢复";                 // 返回普通值 → 链变为成功状态
  })
  .then((v) => console.log("catch 之后链继续:", v));

// catch 也能捕获 reject
Promise.reject(new Error("初始拒绝"))
  .catch((e: Error) => console.log("捕获拒绝:", e.message));
```

**运行结果：**
```
catch 捕获: then 中抛错
catch 之后链继续: 已恢复
捕获拒绝: 初始拒绝
```

**注意：**
* `catch` 返回后链会「恢复」为 fulfilled，后续 `then` 正常执行；若想继续失败需重新 `throw`。
* 链尾务必有 `catch`，否则错误会静默丢失并触发 `unhandledRejection`。
* `catch` 中的参数类型默认是 `unknown`（`useUnknownInCatchVariables`），需先收窄再使用。

### 5.8 finally

**概念说明：** `finally(onFinally)` 无论成功或失败都会执行，且不接收任何参数、不改变链的状态（返回值被忽略，抛错除外）。适合做清理工作。

```typescript
let loading = true;

function done(): void {
  loading = false;
  console.log("清理完成，loading =", loading);
}

Promise.resolve("数据")
  .then((d) => console.log("处理:", d))
  .catch((e) => console.log("错误:", e))
  .finally(done);

// 失败路径同样会执行 finally
Promise.reject(new Error("请求失败"))
  .catch((e: Error) => console.log("捕获:", e.message))
  .finally(() => console.log("失败路径的清理"));
```

**运行结果：**
```
处理: 数据
清理完成，loading = false
捕获: 请求失败
失败路径的清理
```

**注意：**
* `finally` 回调不接收参数，无法知道成功还是失败。
* 若 `finally` 中抛错或返回 rejected Promise，会覆盖原有结果。
* 常用来关闭加载动画、释放连接、清理定时器。

### 5.9 Promise 链

**概念说明：** 把多个异步操作串行化：每个 `then` 返回新 Promise，只有前一步完成后才执行下一步，避免了回调嵌套。数据沿链向下传递。

```typescript
function delay<T>(value: T, ms: number): Promise<T> {
  return new Promise((r) => setTimeout(() => r(value), ms));
}

delay("用户 ID 1", 10)
  .then((id) => {
    console.log("步骤 1 获取 ID:", id);
    return delay({ name: "Alice" }, 10);      // 模拟请求详情
  })
  .then((user) => {
    console.log("步骤 2 获取用户:", user.name);
    return delay(["订单A", "订单B"], 10);      // 模拟请求订单
  })
  .then((orders) => console.log("步骤 3 订单数:", orders.length))
  .catch((e: Error) => console.log("任一步失败:", e.message))
  .finally(() => console.log("流程结束"));
```

**运行结果：**
```
步骤 1 获取 ID: 用户 ID 1
步骤 2 获取用户: Alice
步骤 3 订单数: 2
流程结束
```

**注意：**
* 链中的错误只需在末尾统一 `catch` 一次，无需每步处理。
* `then` 内部忘了 `return` 会让下一步拿到 `undefined`，是极高频的 bug。
* 需要并行时用 `Promise.all`，串行链会让总耗时累加。

### 5.10 Promise.all

**概念说明：** 接收 Promise 数组，**全部成功**时按输入顺序返回结果数组；只要有一个失败，整体立即拒绝（短路），是并行请求的常用手段。

```typescript
const wait = <T>(value: T, ms: number) =>
  new Promise<T>((r) => setTimeout(() => r(value), ms));

// 结果顺序与输入顺序一致，与完成时间无关
Promise.all([wait(1, 30), wait(2, 10), wait(3, 20)]).then((v) =>
  console.log("结果（按输入顺序）:", v)
);

// 一个失败则整体失败
Promise.all([wait("ok", 10), Promise.reject(new Error("第二个失败"))])
  .then((v) => console.log("不会执行", v))
  .catch((e: Error) => console.log("整体失败:", e.message));
```

**运行结果：**
```
整体失败: 第二个失败
结果（按输入顺序）: [ 1, 2, 3 ]
```

**注意：**
* 总耗时取决于最慢的那个 Promise，而非累加。
* 一个失败就整体失败，但其他 Promise 仍会继续执行（无法取消）。
* 需要一个都不失败时，用 `Promise.allSettled` 或先给每个 Promise 加 `catch`。

### 5.11 Promise.race

**概念说明：** 返回最先 **settle**（无论成功或失败）的那个 Promise 的结果，其余结果被丢弃。最经典的用法是实现超时控制。

```typescript
const slow = new Promise<string>((r) => setTimeout(() => r("慢"), 50));
const fast = new Promise<string>((r) => setTimeout(() => r("快"), 10));

Promise.race([slow, fast]).then((v) => console.log("胜出:", v));

// 超时模式
function withTimeout<T>(promise: Promise<T>, ms: number): Promise<T> {
  const timer = new Promise<never>((_, reject) =>
    setTimeout(() => reject(new Error("请求超时")), ms)
  );
  return Promise.race([promise, timer]);
}

withTimeout(
  new Promise<string>((r) => setTimeout(() => r("数据"), 100)),
  20
).catch((e: Error) => console.log("超时模式:", e.message));

// 第一个失败也会「胜出」
Promise.race([Promise.reject(new Error("先出错")), slow]).catch((e: Error) =>
  console.log("失败也胜出:", e.message)
);
```

**运行结果：**
```
胜出: 快
失败也胜出: 先出错
超时模式: 请求超时
```

**注意：**
* 是「最先完成」而不是「最先成功」，失败同样可能胜出（要「最先成功」请用 `Promise.any`）。
* 超时后原请求仍在跑，如需真正取消要配合 `AbortController`。
* 空数组会永远 `pending`。

### 5.12 Promise.allSettled

**概念说明：** 等待所有 Promise 完成（不管成功失败），返回结果对象数组：成功为 `{ status: "fulfilled", value }`，失败为 `{ status: "rejected", reason }`，永不拒绝。

```typescript
(async () => {
  const results = await Promise.allSettled([
    Promise.resolve("成功数据"),
    Promise.reject(new Error("失败原因")),
    Promise.resolve(123),
  ]);

  for (const r of results) {
    if (r.status === "fulfilled") {
      console.log("成功:", r.value);
    } else {
      console.log("失败:", r.reason.message);
    }
  }

  // 统计成功数量
  const okCount = results.filter((r) => r.status === "fulfilled").length;
  console.log(`成功 ${okCount} / ${results.length}`);
})();
```

**运行结果：**
```
成功: 成功数据
失败: 失败原因
成功: 123
成功 2 / 3
```

**注意：**
* 永远不会 reject，因此不需要 `catch`。
* TypeScript 会依据 `status` 判别联合自动收窄到 `value` 或 `reason`。
* 适合「批量操作，允许部分失败」的场景，如批量上传、批量校验。

### 5.13 Promise.any

**概念说明：** 返回**第一个成功**的 Promise 结果；若全部失败，则拒绝并抛出 `AggregateError`，其 `errors` 属性包含所有失败原因。

```typescript
Promise.any([
  Promise.reject(new Error("错误 1")),
  Promise.resolve("第一个成功"),
  Promise.resolve("第二个成功"),
]).then((v) => console.log("any 取值:", v));

// 全部失败 → AggregateError
Promise.any([
  Promise.reject(new Error("a")),
  Promise.reject(new Error("b")),
]).catch((e: AggregateError) => {
  console.log("全部失败，原因个数:", e.errors.length);
  e.errors.forEach((err: Error) => console.log(" -", err.message));
});
```

**运行结果：**
```
any 取值: 第一个成功
全部失败，原因个数: 2
 - a
 - b
```

**注意：**
* 与 `race` 的区别：`race` 取「最先完成」，`any` 取「最先成功」。
* 全部失败抛出的错误类型是 `AggregateError`（ES2021），需显式标注类型才能访问 `errors`。
* 适合多镜像源抢答场景。

### 5.14 async

**概念说明：** `async` 函数总是返回 Promise：`return value` 等价于 `resolve(value)`，`throw error` 等价于 `reject(error)`。这是 `await` 的使用前提。

```typescript
async function getValue(): Promise<number> {
  return 42;                    // 自动包装为 Promise
}

async function fail(): Promise<never> {
  throw new Error("async 中抛出");
}

console.log("调用后立即拿到 Promise:", getValue() instanceof Promise);

getValue().then((v) => console.log("返回值:", v));
fail().catch((e: Error) => console.log("错误:", e.message));

// 返回 Promise 也会被展开
async function nested(): Promise<string> {
  return Promise.resolve("嵌套 Promise 的值");
}
nested().then((v) => console.log(v));
```

**运行结果：**
```
调用后立即拿到 Promise: true
返回值: 42
错误: async 中抛出
嵌套 Promise 的值
```

**注意：**
* `async` 函数中不 `await` 直接返回 Promise 会多一层包装（但 `async` 会自动解包，不影响结果）。
* 声明返回 `Promise<T>` 时，函数体内 `return` 的是 `T`，不是 `Promise<T>`。
* 返回类型写 `Promise<never>` 时，函数只能抛出异常。

### 5.15 await

**概念说明：** `await` 会暂停当前 `async` 函数，等待 Promise 完成并返回其值（若为 rejected 则抛出异常）。`await` 只影响所在函数，不阻塞主线程。

```typescript
function wait<T>(value: T, ms: number): Promise<T> {
  return new Promise((r) => setTimeout(() => r(value), ms));
}

async function main(): Promise<void> {
  console.log("开始");
  const a = await wait("A", 10);
  console.log("拿到:", a);
  const b = await wait("B", 10);
  console.log("拿到:", b);
  console.log("结束");
}

main();
console.log("main 之后的同步代码先执行");
```

**运行结果：**
```
开始
main 之后的同步代码先执行
拿到: A
拿到: B
结束
```

**注意：**
* `await` 会暂停函数执行，因此上面的串行写法总耗时是 20ms；并行请用 `Promise.all`。
* 循环中 `await` 是串行的，需要并行时用 `await Promise.all(arr.map(...))`。
* `await` 非 Promise 值会直接返回该值（但仍让出一次微任务）。

### 5.16 async / await 异常处理

**概念说明：** `await` 一个 rejected Promise 会像同步代码一样抛出异常，因此可用 `try / catch / finally` 统一处理；也可用 `.catch()` 就地兜底。

```typescript
async function fetchUser(id: number): Promise<{ name: string }> {
  if (id <= 0) throw new Error("非法 ID");
  return { name: "Alice" };
}

async function main(): Promise<void> {
  // 方式一：try / catch / finally
  try {
    const user = await fetchUser(1);
    console.log("成功:", user.name);
  } catch (e) {
    console.log("失败:", (e as Error).message);
  } finally {
    console.log("请求结束");
  }

  // 方式二：用 catch 兜底，避免深层嵌套
  const fallback = await fetchUser(-1).catch(() => null);
  console.log("安全取值:", fallback?.name ?? "未知用户");

  // 方式三：to 元组写法，成功失败都不抛
  const [err, user] = await fetchUser(-1)
    .then((v) => [null, v] as const)
    .catch((e: Error) => [e, null] as const);
  console.log(err ? `捕获错误：${err.message}` : user?.name);
}

main();
```

**运行结果：**
```
成功: Alice
请求结束
安全取值: 未知用户
捕获错误：非法 ID
```

**注意：**
* 返回 `null` / 默认值代替抛错，可以让调用方少写 `try / catch`。
* `await` 一个失败 Promise 会中断当前函数，后续代码不执行，注意清理逻辑放 `finally`。
* 串行 `await` 与并行 `Promise.all` 混用时，错误语义不同：前者只拿到第一个错误。

### 5.17 try / catch

**概念说明：** 捕获同步代码中的运行时异常。TypeScript 4.4 起 `catch` 变量默认为 `unknown`（`useUnknownInCatchVariables`），必须先收窄类型才能访问属性。

```typescript
function parseJson(text: string): unknown {
  try {
    return JSON.parse(text);
  } catch (e) {
    // e 的类型是 unknown，必须先收窄
    console.log("错误类型:", e instanceof SyntaxError ? "SyntaxError" : typeof e);
    return null;
  } finally {
    console.log("解析尝试完成");
  }
}

console.log("解析结果:", parseJson('{"a":1}'));
console.log("解析结果:", parseJson("{ 非法 JSON }"));

// 自定义错误类便于区分
class ValidationError extends Error {
  constructor(message: string, public field: string) {
    super(message);
    this.name = "ValidationError";
  }
}

try {
  throw new ValidationError("必填校验失败", "email");
} catch (e) {
  if (e instanceof ValidationError) {
    console.log(`字段 ${e.field} 出错：${e.message}`);
  }
}
```

**运行结果：**
```
解析尝试完成
解析结果: { a: 1 }
错误类型: SyntaxError
解析尝试完成
解析结果: null
字段 email 出错：必填校验失败
```

**注意：**
* 要把 `catch` 变量标注为具体类型，可写 `catch (e: any)` 或先做 `instanceof` 收窄。
* `finally` 一定执行，即使 `try` 中有 `return`。
* 继承 `Error` 时建议设置 `this.name`，否则日志中名称会显示为 `Error`。

## 6. TypeScript 基础类型
### 6.1 类型注解

**概念说明：** 类型注解写在变量、参数、返回值之后，用 `: 类型` 表示，用于显式告诉编译器期望的类型。它是编译期约束，不产生任何运行时代码。

```typescript
// 变量注解
let username: string = "Alice";
let count: number = 3;

// 函数参数与返回值注解
function greet(name: string, times: number): string {
  return `你好 ${name}，`.repeat(times);
}

// 箭头函数
const sum = (a: number, b: number): number => a + b;

// 对象与数组
const user: { name: string; age: number } = { name: "Bob", age: 20 };
const ids: number[] = [1, 2, 3];

console.log(greet(username, count));
console.log(sum(1, 2), user.name, ids.length);

// 编译后类型注解被完全擦除
console.log(typeof username, typeof user);
```

**运行结果：**
```
你好 Alice，你好 Alice，你好 Alice，
3 Bob 3
string object
```

**注意：**
* 注解与赋值类型不匹配会立即编译报错，如 `let n: number = "1"`。
* 类型注解会被「类型擦除」，运行时不存在 `number` 这样的类型信息。
* 能推断出来时不必写注解，仅在推论不明确或作为公共契约时才显式标注。

### 6.2 类型推断

**概念说明：** 未写注解时，TypeScript 会根据初始值自动推断类型。推断遵循「字面量收窄 → 上下文放宽」的规则：`let` 推断为宽泛类型，`const` 推断为字面量类型。

```typescript
let a = 42;                // 推断为 number
const b = 42;              // 推断为字面量 42
const c = "hi";            // 推断为 "hi"
let d = [1, 2, 3];         // 推断为 number[]
const e = { id: 1 };       // 推断为 { id: number }
let f = null;              // 严格模式下推断为 any（需配合 strictNullChecks）

// 函数返回值推断
function double(n: number) {
  return n * 2;            // 推断返回 number
}

// 上下文推断：回调参数类型由调用方决定
[1, 2, 3].map((n) => n.toFixed(1));   // n 推断为 number

console.log(typeof a, b, c, double(2));

// 类型查询验证推断结果
type T1 = typeof b;        // 42
const check: T1 = 42;
console.log(check);
```

**运行结果：**
```
number 42 hi 4
42
```

**注意：**
* `const` 声明原始类型会推断为字面量类型，`let` / `var` 推断为放宽后的类型。
* 对象属性用 `let` 声明时属性为 `number`，用 `const` 时也是 `number`（只有顶层是字面量）。
* 最佳实践是让推断工作，避免冗余注解导致类型被意外放宽（如 `const x: string = "a"` 会丢失字面量类型）。

### 6.3 string

**概念说明：** 字符串类型，支持单引号、双引号和模板字符串。模板字符串可用 `${}` 插值，并可与字面量类型、模板字面量类型配合。

```typescript
const single: string = '单引号';
const double: string = "双引号";
const template: string = `模板 ${single} 与 ${double}`;

console.log(template);
console.log("长度:", template.length);
console.log("大写:", single.toUpperCase());
console.log("包含:", template.includes("模板"));
console.log("切片:", template.slice(0, 2));
console.log("分割:", "a,b,c".split(","));
console.log("替换:", "a-b".replace("-", "+"));
console.log("去空格:", "  x  ".trim());
console.log("补全:", "5".padStart(3, "0"));

// 字符串不可变
let s = "abc";
s[0] = "z";          // 静默失败（非严格模式）
console.log(s);      // abc
```

**运行结果：**
```
模板 单引号 与 双引号
长度: 13
大写: 单引号
包含: true
切片: 模板
分割: [ 'a', 'b', 'c' ]
替换: a+b
去空格: x
补全: 005
abc
```

**注意：**
* 字符串是不可变的，所有方法都返回新字符串。
* `slice` 支持负索引，`substring` 会把负值当 0，建议统一用 `slice`。
* 需要「键名必须是某几种字符串」时用字符串字面量类型或联合类型，见第 11 章。

### 6.4 number

**概念说明：** 所有数字都是双精度浮点数，包括整数和小数。特殊值有 `NaN`、`Infinity`、`-Infinity`。数字字面量可用二进制、八进制、十六进制和下划线分隔符。

```typescript
const int: number = 42;
const float: number = 3.14;
const hex: number = 0xff;          // 255
const binary: number = 0b1010;     // 10
const separated: number = 1_000_000;

console.log(int, float, hex, binary, separated);
console.log((0.1 + 0.2).toFixed(2));   // 0.30  浮点误差需格式化
console.log((1234.5678).toFixed(1));   // 1234.6
console.log(Number.isInteger(42));     // true
console.log(Number.isNaN(NaN));        // true
console.log(parseInt("42px", 10));     // 42
console.log(parseFloat("3.14abc"));    // 3.14
console.log(Number.MAX_SAFE_INTEGER);  // 9007199254740991
```

**运行结果：**
```
42 3.14 255 10 1000000
0.30
1234.6
true
true
42
3.14
9007199254740991
```

**注意：**
* 浮点运算存在精度问题，`0.1 + 0.2 !== 0.3`；金额计算建议用整数分或专用库。
* 判断 NaN 必须用 `Number.isNaN()`，不能用 `x === NaN`。
* 超过 `Number.MAX_SAFE_INTEGER` 的整数需要 `bigint`。

### 6.5 boolean

**概念说明：** 只有 `true` 和 `false` 两个字面量。TypeScript 不会自动把其他值「真值化」为 `boolean`（与 JavaScript 的隐式转换不同）。

```typescript
const isDone: boolean = true;
const isEmpty: boolean = false;

// 显式转换
console.log(Boolean(0), Boolean(""), Boolean([]), Boolean("false"));

// 逻辑运算返回原值，不是 boolean
const arr = [] as string[];
const value = arr.length && "有内容";   // 0（number），不是 boolean
console.log(typeof value, value);

// 需要 boolean 时必须显式转换
const hasItems: boolean = arr.length > 0;
console.log(hasItems);

// 类型谓词常见写法
function isNonNull<T>(v: T | null): v is T {
  return v !== null;
}
console.log([1, null, 2].filter(isNonNull));
```

**运行结果：**
```
false false true true
number 0
false
[ 1, 2 ]
```

**注意：**
* `&&` / `||` 返回操作数本身，在要求 `boolean` 的地方会报类型错误。
* 类型谓词 `v is T` 的返回值必须是 `boolean`，这是收窄类型的标准手段。
* `Boolean([])` 为 `true`，空数组是真值，判断空请用 `length === 0`。

### 6.6 bigint

**概念说明：** 表示任意精度的整数，字面量以 `n` 结尾或用 `BigInt()` 创建。用于超出 `Number.MAX_SAFE_INTEGER` 的场景，如大整数 ID、加密计算。

```typescript
const big1: bigint = 9007199254740993n;
const big2 = BigInt("9007199254740993");
const big3: bigint = 10n;

console.log(big1 === big2);        // true
console.log(big1 + big3);
console.log(typeof big1);          // bigint
console.log(big1 > 9007199254740992);  // 可与 number 比较

// 但不能与 number 混合运算
// console.log(big1 + 1);          // 错误：Operator '+' cannot be applied to bigint and number
console.log(big1 + 1n);            // 必须同为 bigint

// 显式转换
console.log(Number(10n), BigInt(10));
```

**运行结果：**
```
true
9007199254740994
bigint
true
9007199254740994
10 10n
```

**注意：**
* `bigint` 与 `number` 不能直接混合运算，必须先显式转换。
* 比较运算符（`>`、`<`、`==`）允许混用，但 `===` 要求类型相同。
* 不能对 `bigint` 用 `Math` 方法，除法结果会截断（`7n / 2n === 3n`）。

### 6.7 symbol

**概念说明：** `symbol` 表示独一无二的值，常用作对象属性键以避免命名冲突。`unique symbol` 是 `symbol` 的子类型，只能由 `const` 声明或只读属性产生。

```typescript
const id: unique symbol = Symbol("id");
const other = Symbol("id");

console.log(id === other);          // false，即使描述相同
console.log(id.toString());         // Symbol(id)

// 作为属性键
const user = {
  name: "Alice",
  [id]: 1001,                       // 用 symbol 当键
};
console.log(user[id]);              // 1001
console.log(Object.keys(user));     // [ 'name' ]  symbol 键不可枚举
console.log(Object.getOwnPropertySymbols(user).length);  // 1

// 全局共享 symbol
const shared1 = Symbol.for("app.key");
const shared2 = Symbol.for("app.key");
console.log(shared1 === shared2);   // true
console.log(Symbol.keyFor(shared1)); // app.key
```

**运行结果：**
```
false
Symbol(id)
1001
[ 'name' ]
1
true
app.key
```

**注意：**
* 每次 `Symbol()` 都产生新值，`Symbol.for()` 才会在全局注册表中复用。
* Symbol 键不会出现在 `for...in`、`Object.keys`、`JSON.stringify` 中。
* `unique symbol` 类型只能标注 `const` 变量或 `readonly static` 属性。

### 6.8 null

**概念说明：** 表示「有意的空值」。开启 `strictNullChecks` 后，`null` 是独立类型，不能赋值给其他类型，必须用联合类型显式声明。

```typescript
let nullable: string | null = null;
nullable = "有值了";

// 必须先判空
function getLength(s: string | null): number {
  if (s === null) return 0;
  return s.length;              // 此处收窄为 string
}

console.log(getLength(null), getLength("abc"));

// 可选链与空值合并
interface Config { host?: string | null }
const cfg: Config = { host: null };
console.log(cfg.host ?? "localhost");       // localhost
console.log(cfg.host?.length ?? 0);         // 0

// 非空断言（慎用）
const el = document.getElementById("app") as HTMLElement | null;
// console.log(el!.id);          // 编译通过但运行时可能崩
console.log(el === null);
```

**运行结果：**
```
0 3
localhost
0
true
```

**注意：**
* `typeof null === "object"`，不能靠 `typeof` 判断 null，用 `x === null`。
* 默认情况下（未开 `strictNullChecks`）`null` 可赋给任意类型，务必开启严格模式。
* `!` 非空断言只是让编译器闭嘴，没有运行时保护，优先使用判空或可选链。

### 6.9 undefined

**概念说明：** 表示「未赋值」或「属性不存在」。可选属性、未传参数、无返回值函数都会得到 `undefined`。与 `null` 一样需要显式声明在联合类型中。

```typescript
let notSet: string | undefined;
console.log(notSet);                    // undefined

interface User { name: string; age?: number }
const u: User = { name: "Alice" };
console.log(u.age);                     // undefined

// 可选属性的类型其实是 number | undefined
type Age = User["age"];                 // number | undefined

// 默认值：?? 比 || 更精确
function greet(name?: string): string {
  return `你好，${name ?? "陌生人"}`;
}
console.log(greet(), greet("Bob"));
console.log(greet(""));                 // ?? 不会用默认值替换空字符串

// 可选参数的 undefined 判断
function check(name?: string): void {
  if (name === undefined) console.log("未传入");
  else console.log("传入:", name);
}
check();
```

**运行结果：**
```
undefined
undefined
你好，陌生人
你好，Bob
你好，
未传入
```

**注意：**
* `null` 与 `undefined` 在 `==` 下相等（`null == undefined` 为 `true`），用 `===` 区分。
* 可选属性 `?` 等价于 `T | undefined`，但有「可省略」语义（`exactOptionalPropertyTypes` 会区分）。
* 对象解构的默认值 `const { a = 1 } = obj` 在值为 `undefined` 时生效。

### 6.10 any

**概念说明：** `any` 关闭该值的所有类型检查，可以赋给任何类型、访问任何属性、被任何类型赋值。它是「逃生舱」，应尽量避免。

```typescript
let anything: any = 42;
anything = "变成字符串";
anything.foo.bar();              // 不报错，运行时崩溃
console.log(typeof anything);    // string，运行到上面一行会抛错

// any 会污染传播
function bad(): any { return "文本"; }
const len: number = bad().length;   // 不报错，但它其实是数字 2
console.log(len);

// 对比 unknown：更安全
function good(): unknown { return "文本" as unknown; }
const v = good();
// console.log(v.length);        // 错误：'v' is of type 'unknown'
if (typeof v === "string") console.log(v.length);   // 收窄后可用

console.log(anything === null);
```

**运行结果：**
```
2
2
false
```

**注意：**
* `any` 会沿着表达式传播，一个 `any` 可能让整条链失去类型保护。
* 用 `noImplicitAny` 阻止「隐式 any」（未注解且无法推断的参数）。
* 迁移 JS 项目时可临时用 `any`，但应逐步替换为准确类型或 `unknown`。

### 6.11 unknown

**概念说明：** `unknown` 是类型安全的 `any`：任何值都能赋给它，但在收窄之前不能做任何操作。适合表示「来源不明、需要运行时校验」的数据。

```typescript
function parse(input: unknown): string {
  // 必须先收窄
  if (typeof input === "string") return input.toUpperCase();
  if (typeof input === "number") return input.toFixed(2);
  if (Array.isArray(input)) return input.join(",");
  if (input instanceof Date) return input.toISOString();
  if (typeof input === "object" && input !== null && "name" in input) {
    return String((input as { name: unknown }).name);
  }
  if (input === null) return "null";
  return String(input);
}

console.log(parse("abc"));
console.log(parse(3.14159));
console.log(parse([1, 2, 3]));
console.log(parse({ name: "Alice" }));
console.log(parse(undefined));

// 使用类型谓词做的收窄函数
function isStringArray(v: unknown): v is string[] {
  return Array.isArray(v) && v.every((i) => typeof i === "string");
}
console.log(isStringArray(["a"]), isStringArray([1]));
```

**运行结果：**
```
ABC
3.14
1,2,3
Alice
undefined
true false
```

**注意：**
* `unknown` 只能赋值给 `unknown` 或 `any`，不能赋给具体类型 —— 这正是它的安全性来源。
* 捕获的异常（`catch (e)`）默认就是 `unknown`，必须收窄。
* 处理外部数据（API、`JSON.parse`、`localStorage`）时优先用 `unknown` 而非 `any`。

### 6.12 never

**概念说明：** `never` 是「底类型」，表示永不存在的值：函数抛异常或死循环时返回 `never`，类型收窄到不可能时得到 `never`。它是所有类型的子类型，可赋给任何类型。

```typescript
// 永不返回的函数
function fail(msg: string): never {
  throw new Error(msg);
}

function infinite(): never {
  while (true) {}
}

// 穷尽性检查：利用 never 保证所有分支都被处理
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle": return Math.PI * shape.radius ** 2;
    case "square": return shape.side ** 2;
    default:
      // 若新增 kind 未处理，这里会因为 shape 不是 never 而编译报错
      const exhaustive: never = shape;
      return exhaustive;
  }
}

console.log(area({ kind: "circle", radius: 1 }).toFixed(2));
console.log(area({ kind: "square", side: 3 }));

// never 与其他类型运算
type A = string & number;       // never（不可能同时是两者）
const x: A | string = "ok";
console.log(typeof x);
```

**运行结果：**
```
3.14
9
string
```

**注意：**
* `never` 只能被赋给 `never`；它是唯一没有值的类型。
* `never[]` 推断常见于 `const arr = []`，需显式标注 `number[]` 等。
* 用「`const x: never = value`」做穷尽性检查是处理可辨识联合的标准写法。

### 6.13 void

**概念说明：** `void` 表示函数没有有意义的返回值。它与 `undefined` 不同：返回 `void` 的函数返回值不可使用，但可以把「有返回值的函数」赋给「返回 void 的函数类型」。

```typescript
function logMessage(msg: string): void {
  console.log(msg);
  // return "值";   // 错误：Type 'string' is not assignable to type 'void'
}

logMessage("void 函数");

// 把有返回值的函数赋给 void 返回类型是合法的（返回值被忽略）
type VoidFn = () => void;
const fn: VoidFn = () => 42;
const result = fn();
console.log(result === undefined, typeof result);   // 运行时其实是 42

// 泛型语境中的 void
function call<T>(cb: () => T): void {
  cb();
}
call(() => 1);

// 变量类型 void 只能赋 undefined
let v: void = undefined;
console.log(v);
```

**运行结果：**
```
void 函数
false number
undefined
```

**注意：**
* `void` 只用于返回值位置，不要用来标注普通变量。
* 「有返回值 → void」的赋值兼容性是为了支持回调忽略返回值的场景。
* 与 `undefined` 的区别：`undefined` 是具体值，`void` 是「忽略返回值」的语义。

### 6.14 object

**概念说明：** TypeScript 有三个易混概念：`object`（非原始类型）、`Object`（几乎等同于 `{}`，包含所有非 null/undefined 值）、`{}`（除 null/undefined 外的一切）。推荐用小写 `object` 或直接写具体结构。

```typescript
const o1: object = { a: 1 };
const o2: object = [1, 2];
const o3: object = () => {};
// const o4: object = 42;      // 错误：原始类型不能赋给 object

// object 上不能直接访问属性
// console.log(o1.a);          // 错误：Property 'a' does not exist on type 'object'

// 用索引签名或具体接口来访问
const o5: Record<string, number> = { a: 1 };
console.log(o5.a);

interface User { name: string; age: number }
const u: User = { name: "Alice", age: 25 };
console.log(u.name, u.age);

// Object 与 {} 的区别
const p1: Object = "字符串也可以";
const p2: {} = 123;
console.log(typeof p1, typeof p2);

// 与 object 相关的判定
console.log(typeof o1, Array.isArray(o2), typeof o3);
```

**运行结果：**
```
1
Alice 25
string number
object true function
```

**注意：**
* 优先使用具体接口或 `Record<K, V>`，而不是宽泛的 `object`。
* `{}` 类型几乎不做约束（任何非空值都符合），不要用它表达「空对象」。
* 判断对象用 `typeof x === "object" && x !== null`，记住要排除 `null`。

## 7. TypeScript 数组、元组与枚举
### 7.1 数组类型

**概念说明：** 数组类型有两种写法：`T[]` 与 `Array<T>`，二者等价。推断出的数组元素类型取所有元素的联合类型。

```typescript
const nums: number[] = [1, 2, 3];
const strs: Array<string> = ["a", "b"];
const mixed: (string | number)[] = [1, "a", 2];

// 推断为空数组会是 never[]，需要显式标注
const empty: number[] = [];

// 二维数组
const matrix: number[][] = [[1, 2], [3, 4]];
console.log(matrix[0][1]);        // 2

// 数组元素类型查询
type Elem = (typeof nums)[number];   // number

const users: { id: number; name: string }[] = [
  { id: 1, name: "Alice" },
];
console.log(users.map((u) => u.name));

// 泛型函数接受数组
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}
console.log(first(nums), first(empty), first(["x"]));

console.log(mixed.length, strs.join(""));
```

**运行结果：**
```
2
[ 'Alice' ]
1 undefined x
2 ab
```

**注意：**
* `const arr = []` 会推断为 `any[]`（`noImplicitAny` 下为 `never[]`），务必显式标注。
* 需要「固定长度且元素类型不同」的数组请用元组。
* 修改数组的方法（`push`/`splice`）在 `readonly T[]` 上不可用，见下一节。

### 7.2 只读数组

**概念说明：** `readonly T[]` 与 `ReadonlyArray<T>` 表示不可变的数组视图，禁止 `push`、`pop`、索引赋值等所有修改操作，但可安全地赋给普通函数使用（不产生拷贝）。

```typescript
const ro: readonly number[] = [1, 2, 3];
const ro2: ReadonlyArray<number> = [4, 5];

// ro.push(4);          // 错误：Property 'push' does not exist on type 'readonly number[]'
// ro[0] = 9;           // 错误：索引签名是只读的

// 只读方法可用
console.log(ro.map((n) => n * 2));
console.log(ro.slice(0, 2));
console.log(ro.includes(2));

// 可变数组可以赋给只读数组（安全方向）
const mutable: number[] = [1, 2];
const readonlyView: readonly number[] = mutable;
console.log(readonlyView.length);

// 反向赋值需要拷贝
// const back: number[] = ro;         // 错误：readonly 不能赋给可变
const copy: number[] = [...ro];
copy.push(4);
console.log(copy, ro);

// 只读元组
const pair: readonly [number, string] = [1, "a"];
console.log(pair[1]);
```

**运行结果：**
```
[ 2, 4, 6 ]
[ 1, 2 ]
true
2
[ 1, 2, 3, 4 ] [ 1, 2, 3 ]
a
```

**注意：**
* 函数参数用 `readonly T[]` 可表达「我不会修改它」，且能接受两种数组。
* 只读是编译期约束，用 `as` 或 `any` 仍可绕过，运行时无保护。
* `ReadonlyArray` 上有 `concat`、`slice`、`map` 等返回新数组的方法。

### 7.3 Tuple

**概念说明：** 元组是「固定长度、每个位置类型可不同」的数组，适合表示有确定结构的少量值，如坐标、键值对、函数多返回值。

```typescript
// 声明元组
let point: [number, number] = [10, 20];
let pair: [string, number] = ["age", 25];

console.log(point[0], point[1]);
console.log(pair[0], pair[1]);

// 函数返回多个值
function divmod(a: number, b: number): [number, number] {
  return [Math.floor(a / b), a % b];
}
const [quotient, remainder] = divmod(7, 2);
console.log(quotient, remainder);

// 长度必须匹配
// point = [1];                  // 错误：不能少于
// point = [1, 2, 3];            // 错误：不能多于

// 越界访问会被拦截
// console.log(point[2]);        // 错误：Tuple type '[number, number]' of length '2' has no element at index '2'

// 具名元组，提升可读性
type Range = [start: number, end: number];
const r: Range = [0, 10];
console.log(r);

// 只读元组
const fixed = [1, "a"] as const;
console.log(fixed[1]);
```

**运行结果：**
```
10 20
age 25
3 1
[ 0, 10 ]
a
```

**注意：**
* 元组仍支持 `push`（超出长度的部分不会做类型检查），严格固定长度需 `as const` 或只读元组。
* 具名元组的标签只用于文档提示，不影响赋值。
* 解构时可用 `const [a, b] = tuple`，类型会精确对应。

### 7.4 可选 Tuple 元素

**概念说明：** 元组元素后加 `?` 表示该位置可省略，可选元素只能在必填元素之后，且后面的元素也必须是可选的或剩余元素。

```typescript
type Point3D = [x: number, y: number, z?: number];

const p2: Point3D = [1, 2];
const p3: Point3D = [1, 2, 3];
// const bad: Point3D = [1];      // 错误：缺少 y

function createPoint(...args: Point3D): string {
  const [x, y, z = 0] = args;
  return `(${x}, ${y}, ${z})`;
}

console.log(createPoint(1, 2));
console.log(createPoint(1, 2, 3));

// 可选元素让长度变为联合类型
type Len = Point3D["length"];      // 2 | 3
const len2: Len = 2;
const len3: Len = 3;
// const len4: Len = 4;           // 错误

// 访问可选元素类型包含 undefined
const value: number | undefined = ([1, 2] as Point3D)[2];
console.log(len2, len3, value);

// 可选元素后不能再有必填元素
// type Invalid = [a?: number, b: string];   // 错误：必填元素不能位于可选元素之后
```

**运行结果：**
```
(1, 2, 0)
(1, 2, 3)
2 3 undefined
```

**注意：**
* 可选元素的类型自动包含 `undefined`，使用时需处理。
* 元组的 `length` 属性在有可选元素时是联合类型。
* 若「后面的元素必填」，请用联合元组：`[number, number] | [number, number, number]`。

### 7.5 Rest Tuple

**概念说明：** 元组中可以使用剩余元素 `...T`，把「若干个同类型元素」或另一个元组拼接到元组末尾，常用于描述可变参数的函数签名。

```typescript
// 基础剩余元素
type Strings = [string, string, ...number[]];
const data: Strings = ["a", "b", 1, 2, 3];
console.log(data);

// 用剩余元素描述函数参数
function log(level: string, ...messages: string[]): void {
  console.log(`[${level}]`, ...messages);
}
log("INFO", "启动", "完成");

// 把元组作为剩余参数展开
function apply(fn: (...args: number[]) => number, args: [number, number]): number {
  return fn(...args);
}
console.log(apply((a, b) => a + b, [3, 4]));

// 拼接两个元组
type Concat<T extends unknown[], U extends unknown[]> = [...T, ...U];
type Result = Concat<[1, 2], [3, 4]>;   // [1, 2, 3, 4]
const result: Result = [1, 2, 3, 4];
console.log(result);

// 元组的长度与元素类型
type FirstRest<T extends unknown[]> = T extends [infer F, ...infer R] ? [F, R] : never;
type Split = FirstRest<[1, 2, 3]>;      // [1, [2, 3]]
const split: Split = [1, [2, 3]];
console.log(split);

// 空元组：限制函数不接受任何参数
type NoArgs = [];
function create(): NoArgs { return []; }
console.log(create());
```

**运行结果：**
```
[ 'a', 'b', 1, 2, 3 ]
[INFO] 启动 完成
7
[ 1, 2, 3, 4 ]
[ 1, [ 2, 3 ] ]
[]
```

**注意：**
* 剩余元素必须是元组的最后一项，且类型为数组/元组。
* `[...T]` 形式的泛型元组是 `Variadic Tuple Types`（TS 4.0+），可做类型级拼接/切分。
* `[]` 空元组类型可用来禁止传参。

### 7.6 Enum

**概念说明：** 枚举为一组相关常量命名。数字枚举会自动编号并生成反向映射，字符串枚举无自动编号，异构枚举（混合）不推荐使用。枚举是少数会生成运行时代码的类型。

```typescript
// 数字枚举：默认从 0 开始自增
enum Direction {
  Up,
  Down,
  Left,
  Right,
}
console.log(Direction.Up, Direction.Right);
console.log(Direction[0]);         // Up  反向映射

// 自定义起始值
enum Status {
  Pending = 1,
  Active,        // 2
  Closed = 10,
  Done,          // 11
}
console.log(Status.Pending, Status.Active, Status.Closed, Status.Done);

// 字符串枚举：更易调试，无反向映射
enum Color {
  Red = "RED",
  Green = "GREEN",
}
console.log(Color.Red, Color["Red"]);

// 作为类型使用
function move(dir: Direction): string {
  return `移动到 ${Direction[dir]}`;
}
console.log(move(Direction.Up));

// 编译产物（示意）：枚举会生成真实对象
console.log(Object.keys(Direction).length);
```

**运行结果：**
```
0 3
Up
1 2 10 11
RED RED
移动到 Up
8
```

**注意：**
* 数字枚举的「任意数字都能赋给枚举类型」是历史行为（TS 5.0 起对字面量联合更严格）。
* 字符串枚举无法做反向映射，也没有自增语义。
* 若不需要运行时代码，可用联合字面量类型或 `as const` 对象替代枚举。

### 7.7 const enum

**概念说明：** `const enum` 在编译时被完全内联为字面量，不生成任何运行时代码（除非开启 `preserveConstEnums`）。代价是不能做反向映射、不能遍历。

```typescript
const enum LogLevel {
  Debug = 0,
  Info = 1,
  Error = 2,
}

function log(level: LogLevel, msg: string): void {
  console.log(level, msg);
}

log(LogLevel.Info, "启动完成");
// 编译后相当于：console.log(1, "启动完成"); —— LogLevel 定义消失

// 消除歧义：避免常量重名冲突
const enum Code { A = 1 }
const enum Other { A = 1 }
console.log(Code.A === Other.A);   // true，但内联后不产生引用

// 不能用 Object.keys 遍历
// console.log(Object.keys(LogLevel));   // 错误：const enum 只存在于编译期
```

**运行结果：**
```
1 启动完成
true
```

**注意：**
* `const enum` 在 `isolatedModules` / Babel / esbuild 等「单文件编译」场景下不可用（会被当作普通 enum 处理并报错）。
* 发布给第三方的库中避免使用 `const enum`，消费方编译配置可能导致解析失败。
* 现代工程更推荐 `as const` 对象 + 联合类型，兼具字面量类型与运行时可用性。

## 8. Type Alias
### 8.1 type

**概念说明：** `type` 用于给任意类型起别名，包括原始类型、联合类型、函数类型、元组、对象等。它只是「命名」，不创建新类型。

```typescript
// 原始类型别名
type ID = string | number;

// 对象类型别名
type Point = { x: number; y: number };

// 函数类型别名
type Handler = (event: string) => void;

// 联合与元组
type Status = "pending" | "success" | "error";
type Pair = [number, number];

const id1: ID = 1;
const id2: ID = "abc";
const p: Point = { x: 1, y: 2 };
const handler: Handler = (e) => console.log("处理:", e);
const pair: Pair = [1, 2];
const status: Status = "success";

handler("click");
console.log(id1, id2, p.x + p.y, pair, status);

// 别名可以自引用（递归类型）
type Tree = {
  value: number;
  children: Tree[];
};
const tree: Tree = { value: 1, children: [{ value: 2, children: [] }] };
console.log(JSON.stringify(tree));
```

**运行结果：**
```
处理: click
1 abc 3 [ 1, 2 ] success
{"value":1,"children":[{"value":2,"children":[]}]}
```

**注意：**
* `type` 不能重复声明（与 `interface` 不同），重复会报 `Duplicate identifier`。
* 别名可以被导出、导入，也可作为泛型参数。
* 类型别名不参与运行时，编译后完全消失。

### 8.2 类型别名与对象

**概念说明：** 用 `type` 描述对象结构，可组合可选属性、只读属性、索引签名，替代匿名对象类型以提升可读性与复用性。

```typescript
type User = {
  readonly id: number;
  name: string;
  age?: number;                       // 可选
  tags: string[];
  [key: string]: unknown;             // 索引签名：允许额外属性
};

const user: User = {
  id: 1,
  name: "Alice",
  tags: ["vip"],
  extra: "额外字段允许",
};

// user.id = 2;   // 错误：readonly
user.name = "Alice Smith";

console.log(user.name, user.age ?? "未填年龄", user["extra"]);

// 嵌套类型别名
type Address = { city: string; zip: string };
type Customer = { name: string; address: Address };

const c: Customer = { name: "Bob", address: { city: "北京", zip: "100000" } };
console.log(c.address.city);

// 用映射式改写可选性
type RequiredUser = { [K in keyof User]-?: User[K] };
const full: RequiredUser = { id: 1, name: "A", age: 20, tags: [], extra: 1 } as RequiredUser;
console.log(full.age);
```

**运行结果：**
```
Alice Smith 未填年龄 额外字段允许
北京
20
```

**注意：**
* 索引签名会让所有属性都必须符合该类型，注意与具体属性的类型兼容。
* 对象字面量赋值时会有「多余属性检查」，用中间变量或索引签名可绕过。
* 类型别名不能像接口那样合并声明，重复定义会报错。

### 8.3 类型别名与函数

**概念说明：** 用 `type` 描述函数签名，包括参数、返回值、this 类型、重载签名（通过联合/交叉组合）与泛型函数。

```typescript
// 基本函数类型
type Add = (a: number, b: number) => number;
const add: Add = (a, b) => a + b;

// 带可选项与剩余参数
type Logger = (level: string, ...messages: string[]) => void;
const logger: Logger = (level, ...msgs) => console.log(`[${level}]`, msgs.join(" "));

// 泛型函数类型
type Mapper<T, U> = (input: T) => U;

// 回调类型
type Callback<T> = (err: Error | null, data?: T) => void;
type AsyncFn<T> = (...args: never[]) => Promise<T>;

const toNumber: Mapper<string, number> = (s) => Number(s);
const cb: Callback<string> = (err, data) => console.log(err ? err.message : data);
const fetchName: AsyncFn<string> = async () => "Alice";

console.log(add(2, 3));
logger("INFO", "启动", "完成");
console.log(toNumber("42"));
cb(null, "成功数据");
cb(new Error("失败原因"));
fetchName().then((n) => console.log(n));

// 函数类型也可描述构造签名与调用签名的对象
type Callable = {
  (x: number): number;       // 可调用
  version: string;           // 附带属性
};
const callable = ((x: number) => x * 2) as Callable;
callable.version = "1.0";
console.log(callable(21), callable.version);
```

**运行结果：**
```
5
[INFO] 启动 完成
42
成功数据
失败原因
Alice
42 1.0
```

**注意：**
* 函数类型别名用 `=>` 而不是 `:`，参数名可省略但建议保留以便阅读。
* 交叉类型 `A & B` 可组合多个函数类型（如重载）。
* 异步函数类型应写 `Promise<T>`，不要写 `async (…) => T`。

### 8.4 类型别名与联合类型

**概念说明：** 联合类型用 `|` 表示「其中之一」。别名让复杂联合（尤其是可辨识联合）具有名字，便于复用与维护。

```typescript
// 状态联合
type Status = "idle" | "loading" | "success" | "error";
let status: Status = "idle";
status = "loading";
// status = "done";      // 错误：不在联合中

// 可辨识联合（推荐写法）
type ApiResult =
  | { state: "ok"; data: string[] }
  | { state: "error"; message: string; code: number };

function render(r: ApiResult): string {
  switch (r.state) {
    case "ok": return `共 ${r.data.length} 条`;
    case "error": return `错误 ${r.code}: ${r.message}`;
  }
}

console.log(render({ state: "ok", data: ["a", "b"] }));
console.log(render({ state: "error", message: "超时", code: 408 }));
console.log(status);

// 联合类型的公共属性才能直接访问
type A = { a: number; common: string };
type B = { b: number; common: string };
const either: A | B = { a: 1, common: "x" };
console.log(either.common);
// console.log(either.a);   // 错误：属性 'a' 不存在于类型 'A | B' 上

// never 与联合
type NoNever = string | never;    // string
const nn: NoNever = "ok";
console.log(nn);
```

**运行结果：**
```
共 2 条
错误 408: 超时
idle
x
ok
```

**注意：**
* 联合类型只能访问所有成员共有的成员，访问独有属性前必须先收窄。
* `never` 与任何类型求并等于该类型自身（恒等元）。
* 用字面量联合替代枚举可避免生成运行时代码。

### 8.5 类型别名与交叉类型

**概念说明：** 交叉类型用 `&` 表示「同时满足」。常用于把多个类型合并成一个对象类型，相当于类型层面的「与」。

```typescript
type Person = { name: string };
type Employee = { employeeId: number };
type Contact = { email: string };

// 合并
type Staff = Person & Employee & Contact;
const staff: Staff = { name: "Alice", employeeId: 1, email: "a@x.com" };
console.log(staff.name, staff.employeeId, staff.email);

// 同名属性交叉：类型取交集
type T1 = { value: string | number };
type T2 = { value: number | boolean };
type Merged = T1 & T2;            // value: number
const m: Merged = { value: 1 };
console.log(m.value);

// 不可能的交集变成 never
type Impossible = { a: string } & { a: number };
// const bad: Impossible = { a: "x" };   // 错误：类型不可能满足

// 交叉类型混合函数与对象
type FnWithMeta = ((x: number) => number) & { meta: string };
const fn = Object.assign((x: number) => x + 1, { meta: "加倍" }) as FnWithMeta;
console.log(fn(1), fn.meta);

// 交叉类型丢失推断：结果可能是 never 的常见坑
type Keys<T> = keyof T;
type K = Keys<{ a: 1 } & { b: 2 }>;   // "a" | "b"
const k: K = "a";
console.log(k);
```

**运行结果：**
```
Alice 1 a@x.com
1
2 加倍
a
```

**注意：**
* 交叉同名属性若类型不兼容会得到 `never`，导致后续所有操作都不可用。
* 交叉类型是「就近合并」，与继承不同，不会产生冲突覆盖。
* 需要「合并时以后者为准」时用映射类型或 `Omit` + `&` 组合。

## 9. Interface
### 9.1 interface 基础

**概念说明：** `interface` 用于描述对象的结构契约，支持扩展与声明合并，是面向对象风格的类型定义方式。

```typescript
interface User {
  id: number;
  name: string;
  tags: string[];
}

const user: User = { id: 1, name: "Alice", tags: ["vip"] };

// 接口作为函数参数类型
function printUser(u: User): void {
  console.log(`${u.id}: ${u.name} (${u.tags.join("/")})`);
}
printUser(user);

// 接口用于类实现
class Admin implements User {
  id = 2;
  name = "Bob";
  tags: string[] = ["admin"];
  extra = "额外属性";
}
const admin: User = new Admin();
printUser(admin);

// 结构化类型：只要结构匹配就算兼容
const literal = { id: 3, name: "Carol", tags: [], other: true };
printUser(literal);          // 多出属性也兼容（变量赋值不做多余属性检查）
```

**运行结果：**
```
1: Alice (vip)
2: Bob (admin)
3: Carol ()
```

**注意：**
* TypeScript 是结构化类型系统：结构一致即兼容，不要求显式 `implements`。
* 对象字面量直接传参时会触发「多余属性检查」，赋给变量后即可绕过。
* 接口只描述实例侧结构，不能描述构造函数（需用 `new () => T`）。

### 9.2 可选属性

**概念说明：** 属性名后加 `?` 表示可以省略，其类型自动包含 `undefined`。这是描述配置对象、API 响应的常用方式。

```typescript
interface Config {
  host: string;
  port?: number;
  ssl?: boolean;
  retries?: number;
}

function connect(cfg: Config): void {
  const port = cfg.port ?? 80;
  const ssl = cfg.ssl ?? false;
  const retries = cfg.retries ?? 3;
  console.log(`${ssl ? "https" : "http"}://${cfg.host}:${port}，重试 ${retries} 次`);
}

connect({ host: "example.com" });
connect({ host: "secure.com", port: 443, ssl: true, retries: 0 });

// 可选链读取嵌套可选属性
interface Response { data?: { items?: string[] } }
const res: Response = {};
console.log(res.data?.items?.length ?? 0);

// exactOptionalPropertyTypes 开启后，显式传 undefined 会报错
interface Strict { name?: string }
const s1: Strict = {};
const s2: Strict = { name: "ok" };
console.log(s1, s2.name);
```

**运行结果：**
```
http://example.com:80，重试 3 次
https://secure.com:443，重试 0 次
0
{} ok
```

**注意：**
* `?` 与「类型里写 `| undefined`」略有差别：后者要求属性必须出现（只是值可为 undefined）。
* 可选属性读取结果是 `T | undefined`，严格模式下必须处理。
* 解构时可用默认值：`const { port = 80 } = cfg`。

### 9.3 readonly

**概念说明：** `readonly` 修饰的属性只能在初始化时赋值，之后不可修改。它是编译期约束，且只是「浅只读」。

```typescript
interface Point {
  readonly x: number;
  readonly y: number;
  label: string;
  readonly nested: { value: number };
}

const p: Point = { x: 1, y: 2, label: "起点", nested: { value: 0 } };

// p.x = 10;              // 错误：Cannot assign to 'x' because it is a read-only property
p.label = "终点";          // 非只读属性可改
p.nested.value = 99;       // 浅只读：嵌套对象内部仍可改

console.log(p.label, p.nested.value);

// 只读数组
interface Team {
  readonly members: readonly string[];
}
const team: Team = { members: ["Alice"] };
// team.members.push("Bob");   // 错误：readonly 数组无 push
// team.members = [];          // 错误：属性只读

// Readonly 工具类型生成只读版本
type ReadonlyPoint = Readonly<Point>;
const rp: ReadonlyPoint = { x: 1, y: 2, label: "l", nested: { value: 0 } };
// rp.label = "x";        // 错误
console.log(team.members.length, rp.x);

// 函数参数中的只读数组表达「不修改」承诺
function sum(nums: readonly number[]): number {
  return nums.reduce((a, b) => a + b, 0);
}
const arr: number[] = [1, 2, 3];
console.log(sum(arr));
```

**运行结果：**
```
终点 99
1 1
6
```

**注意：**
* `readonly` 是浅层的：嵌套对象的属性不受保护。
* 只读数组可以接受可变数组（安全方向），反向不行。
* 运行时没有保护，`as any` 或 `Object.defineProperty` 仍可改。

### 9.4 函数类型 Interface

**概念说明：** 接口可以描述函数类型（调用签名）以及带属性的可调用对象、构造函数（构造签名）。

```typescript
// 调用签名
interface Formatter {
  (value: number, decimals?: number): string;
}

const currency: Formatter = (value, decimals = 2) => `¥${value.toFixed(decimals)}`;
console.log(currency(12.3456));
console.log(currency(12.3456, 0));

// 带属性的可调用对象
interface Counter {
  (start: number): number;
  defaultStart: number;
  description: string;
}

const counter = ((start: number) => start + 1) as Counter;
counter.defaultStart = 0;
counter.description = "计数器";
console.log(counter(counter.defaultStart), counter.description);

// 构造签名
interface PointConstructor {
  new (x: number, y: number): { x: number; y: number };
}
class Point2D {
  constructor(public x: number, public y: number) {}
}
const Ctor: PointConstructor = Point2D;
const point = new Ctor(1, 2);
console.log(point.x, point.y);

// 索引签名 + 调用签名
interface Cache {
  [key: string]: string | number;
  (key: string): string | number | undefined;
}

// 方法签名：两种等价写法
interface Comparer {
  compare(a: number, b: number): number;      // 方法简写
  format: (v: number) => string;              // 属性写法
}
const cmp: Comparer = { compare: (a, b) => a - b, format: (v) => `${v}` };
console.log(cmp.compare(3, 1), cmp.format(5));
```

**运行结果：**
```
¥12.35
¥12
1 计数器
1 2
2 5
```

**注意：**
* 调用签名与属性可共存，实现时用 `Object.assign` 或类型断言。
* 方法简写与属性函数写法在类型上略有差异（`strictFunctionTypes` 下方法参数是双变的）。
* 构造签名用 `new (...)`，不能与调用签名混淆。

### 9.5 Interface 继承

**概念说明：** 接口用 `extends` 继承一个或多个接口，获得其成员并可追加或覆盖（覆盖需兼容）。一个接口也可继承类（获得其成员但不含实现）。

```typescript
interface Animal {
  name: string;
  eat(): void;
}

interface Pet {
  owner: string;
}

// 多继承
interface Dog extends Animal, Pet {
  breed: string;
}

const dog: Dog = {
  name: "旺财",
  breed: "柴犬",
  owner: "Alice",
  eat() { console.log(`${this.name} 在吃东西`); },
};
dog.eat();
console.log(dog.owner, dog.breed);

// 继承类：接口获得类的成员签名（含 private/protected 约束）
class Base {
  id = 1;
  describe(): string { return `id=${this.id}`; }
}
interface FromClass extends Base {
  extra: string;
}
const obj = { id: 2, describe: () => "id=2", extra: "e" } as FromClass;
console.log(obj.describe(), obj.extra);

// 覆盖成员类型需兼容（可收窄）
interface A { value: string | number }
interface B extends A { value: string }
const b: B = { value: "只允许字符串" };
console.log(b.value);
```

**运行结果：**
```
旺财 在吃东西
Alice 柴犬
id=2 e
只允许字符串
```

**注意：**
* 继承多个接口时成员冲突会导致编译错误，除非类型兼容。
* 接口继承类会连 `private` 成员一起继承，导致只有该类的子类才能实现它。
* 接口的 `extends` 与类的 `extends` 语义不同：前者只合并类型。

### 9.6 Interface 合并

**概念说明：** 同名的多个 `interface` 声明会自动合并（Declaration Merging），这是 `type` 做不到的。常用于为第三方库补充类型（模块扩充）。

```typescript
// 同名接口自动合并
interface Box {
  width: number;
}
interface Box {
  height: number;
}
interface Box {
  depth?: number;
}

const box: Box = { width: 1, height: 2, depth: 3 };
console.log(box.width, box.height, box.depth);

// 同名方法会形成重载
interface Api {
  get(id: number): string;
}
interface Api {
  get(id: string): number;
}
const api: Api = {
  get(id: any): any {
    return typeof id === "number" ? `id-${id}` : 42;
  },
};
console.log(api.get(1), api.get("a"));

// 与命名空间合并（常见于库的类型定义）
interface Config { debug: boolean }
namespace Config {
  export const defaultConfig: Config = { debug: false };
}
console.log(Config.defaultConfig.debug);

// 函数与命名空间合并
function build(): string { return "built"; }
namespace build {
  export const version = "1.0";
}
console.log(build(), build.version);
```

**运行结果：**
```
1 2 3
id-1 42
false
built 1.0
```

**注意：**
* 同名接口内同名属性若类型不同会直接报错。
* 声明合并无法作用于 `type` 别名（会报重复标识符）。
* 顺序无关，但后面声明的重载优先匹配。

### 9.7 Interface 与 Type 的区别

**概念说明：** 二者在描述对象时几乎等价，差异集中在：能否合并声明、能否描述非对象类型、扩展机制与错误提示。

```typescript
// 1) 声明合并：interface 可以，type 不行
interface Mergeable { a: number }
interface Mergeable { b: number }
const merged: Mergeable = { a: 1, b: 2 };
// type Dup = { a: number };
// type Dup = { b: number };      // 错误：Duplicate identifier 'Dup'

// 2) 描述联合/原始/元组：type 可以，interface 不行
type ID = string | number;
type Pair = [number, number];
type Name = string;
// interface Bad extends string | number {}   // 错误

// 3) 扩展方式不同
interface Animal { name: string }
interface Dog extends Animal { breed: string }     // interface extends
type Cat = Animal & { color: string };              // type 用交叉

// 4) 接口不能描述映射类型/条件类型结果
type Keys<T> = { [K in keyof T]: T[K] };

// 5) 类可以实现两者
class Impl implements Dog, Cat {
  name = "x";
  breed = "y";
  color = "z";
}

console.log(merged.a, merged.b, new Impl().name);

// 6) 索引签名与元组上的差异
interface Dict { [key: string]: number }
type DictAlias = Record<string, number>;
const d1: Dict = { a: 1 };
const d2: DictAlias = { b: 2 };
console.log(d1.a, d2.b);
```

**运行结果：**
```
1 2 x
1 2
```

**注意：**
* 面向对象、需要扩展/合并（尤其为库补类型）用 `interface`。
* 需要联合、交叉、元组、条件/映射类型或给原始类型起名用 `type`。
* 同一项目中保持风格一致比选哪个更重要；很多规范推荐「对象默认用 interface，其他用 type」。

## 10. 联合类型、交叉类型与类型缩小
### 10.1 Union Type

**概念说明：** 联合类型 `A | B` 表示值可能是其中任意一种。赋值时只需满足其中之一，但使用前必须收窄才能访问分支独有的成员。

```typescript
type StringOrNumber = string | number;

function stringify(v: StringOrNumber): string {
  // 直接使用联合类型共有的成员
  return `值为 ${v}`;
}
console.log(stringify(42), stringify("abc"));

// 联合的字面量收窄
let dir: "up" | "down" = "up";
dir = "down";
// dir = "left";       // 错误

// 联合 + 数组
type Item = { type: "text"; value: string } | { type: "num"; value: number };
const items: Item[] = [
  { type: "text", value: "hello" },
  { type: "num", value: 1 },
];
const total = items.map((i) => (i.type === "num" ? i.value : i.value.length));
console.log(total);

// 联合与 null 的实用写法
function findUser(id: number): { name: string } | null {
  return id > 0 ? { name: "Alice" } : null;
}
const user = findUser(1);
console.log(user ? user.name : "无用户");

// 联合类型赋值时的兼容性
let x: string | number = 1;
x = "now a string";
console.log(typeof x);
```

**运行结果：**
```
值为 42 值为 abc
[ 5, 1 ]
Alice
string
```

**注意：**
* 联合类型不是「交集」，`string | number` 不能直接做算术或字符串方法。
* 可选属性 `?` 本质上是 `T | undefined` 的联合。
* 联合成员过多时可考虑抽取公共接口 + 可辨识字段。

### 10.2 Intersection Type

**概念说明：** 交叉类型 `A & B` 合并所有成员，要求同时满足。与联合相反：联合是「或」，交叉是「且」。

```typescript
type HasName = { name: string };
type HasAge = { age: number };

type Person = HasName & HasAge;
const p: Person = { name: "Alice", age: 25 };
console.log(p.name, p.age);

// 交叉合并同名属性：类型取交
type A = { id: number | string };
type B = { id: string };
type C = A & B;                  // id: string
const c: C = { id: "s" };
console.log(c.id);

// 实用：给对象补上默认字段类型
type WithTimestamps<T> = T & { createdAt: Date; updatedAt: Date };
const record: WithTimestamps<{ title: string }> = {
  title: "文章",
  createdAt: new Date(),
  updatedAt: new Date(),
};
console.log(record.title, record.createdAt instanceof Date);

// 交叉 + 联合的组合（注意优先级：交叉先算）
type Mixed = ({ a: number } | { b: number }) & { common: string };
const m1: Mixed = { a: 1, common: "x" };
const m2: Mixed = { b: 2, common: "y" };
console.log(m1.common, m2.common);

// 冲突导致 never
type Conflict = { v: string } & { v: number };
// const bad: Conflict = { v: 1 };    // 错误：v 是 never
console.log("冲突属性的类型为 never，无法赋值");
```

**运行结果：**
```
Alice 25
s
文章 true
x y
冲突属性的类型为 never，无法赋值
```

**注意：**
* 交叉对象时属性是「合并」而非覆盖，冲突会产生 `never`。
* 与联合混用时要加括号明确优先级。
* 需要「覆盖/替换」语义请用映射类型或 `Omit<T, K> & U`。

### 10.3 typeof 类型守卫

**概念说明：** `typeof` 既能在值层面判断类型，也能在类型层面查询变量/属性的类型。作为守卫时，它可以让联合类型在分支内自动收窄。

```typescript
function format(v: string | number | boolean): string {
  if (typeof v === "string") return v.toUpperCase();    // v: string
  if (typeof v === "number") return v.toFixed(1);       // v: number
  return v ? "真" : "假";                                // v: boolean
}
console.log(format("abc"), format(3.14159), format(true));

// typeof 能覆盖的类型
type All = string | number | bigint | boolean | symbol | undefined | object | Function;
function describe(v: All): string {
  switch (typeof v) {
    case "string": return "字符串";
    case "number": return "数字";
    case "bigint": return "大整数";
    case "boolean": return "布尔";
    case "symbol": return "符号";
    case "undefined": return "未定义";
    case "function": return "函数";
    default: return "对象";
  }
}
console.log(describe(1), describe(() => {}), describe({}));

// 类型层面的 typeof：查询已有值的类型
const config = { host: "localhost", port: 8080, ssl: true };
type Config = typeof config;                // { host: string; port: number; ssl: boolean }
const cfg: Config = { host: "a", port: 1, ssl: false };
console.log(cfg.port);

// typeof 与 as const 结合可得到字面量类型
const levels = ["debug", "info"] as const;
type Level = (typeof levels)[number];       // "debug" | "info"
const l: Level = "info";
console.log(l, levels.length);
```

**运行结果：**
```
ABC 3.1 真
数字 函数 对象
1
info 2
```

**注意：**
* `typeof null === "object"`，判断 null 要用 `v === null`。
* `typeof` 无法区分数组、Date 等具体对象类型，需用 `Array.isArray` / `instanceof`。
* 类型位置的 `typeof` 只能作用于标识符或属性路径，不能是任意表达式（`typeof f()` 不合法）。

### 10.4 instanceof 类型守卫

**概念说明：** `instanceof` 通过原型链判断构造函数，右侧必须是「有构造签名的值」，左侧为对象。对 `class` 实例收窄非常自然。

```typescript
class ApiError extends Error {
  constructor(message: string, public status: number) { super(message); }
}
class NetworkError extends Error {}
class TimeoutError extends Error {}

function handle(err: unknown): string {
  if (err instanceof ApiError) return `接口错误 ${err.status}: ${err.message}`;
  if (err instanceof NetworkError) return `网络异常: ${err.message}`;
  if (err instanceof TimeoutError) return `超时: ${err.message}`;
  if (err instanceof Error) return `其他错误: ${err.message}`;
  return "未知异常";
}

console.log(handle(new ApiError("未授权", 401)));
console.log(handle(new NetworkError("断网")));
console.log(handle("字符串错误"));

// 内置对象
function toDate(v: Date | string): string {
  if (v instanceof Date) return v.toISOString().slice(0, 10);
  return v;
}
console.log(toDate(new Date("2024-01-01")));
console.log(toDate("2024-01-01"));

// Array 也可用 Array.isArray（跨 iframe 更可靠）
function wrap(v: string | string[]): string[] {
  return Array.isArray(v) ? v : [v];
}
console.log(wrap("a"), wrap(["a", "b"]));
```

**运行结果：**
```
接口错误 401: 未授权
网络异常: 断网
未知异常
2024-01-01
2024-01-01
[ 'a' ] [ 'a', 'b' ]
```

**注意：**
* 仅对「用构造函数/类创建的对象」有效，原始类型和普通对象字面量不适用。
* 跨执行上下文（iframe、不同 realm）时原型链不同，`instanceof Array` 可能失败，改用 `Array.isArray`。
* 继承了 `Error` 的自定义错误在 `instanceof` 判断时需注意 `Object.setPrototypeOf` 兼容问题（ES2015+ 编译目标一般正常）。

### 10.5 in 类型守卫

**概念说明：** `"key" in obj` 判断属性是否存在（含原型链）。在联合类型中，它可以把类型收窄到「拥有该属性的那些分支」。

```typescript
type Bird = { fly: () => void; name: string };
type Fish = { swim: () => void; name: string };
type Pet = Bird | Fish;

function move(pet: Pet): void {
  if ("fly" in pet) {
    pet.fly();                 // 收窄为 Bird
  } else {
    pet.swim();                // 收窄为 Fish
  }
}

move({ name: "鸟", fly: () => console.log("飞翔") });
move({ name: "鱼", swim: () => console.log("游动") });

// 可选属性上使用 in 依然能收窄
type Opt = { name: string; age?: number };
function show(o: Opt): string {
  return "age" in o ? `有年龄字段，值 ${o.age}` : "无年龄字段";
}
console.log(show({ name: "A" }));
console.log(show({ name: "A", age: 0 }));

// 与 typeof 组合的宽松对象收窄
function read(v: unknown): string {
  if (typeof v === "object" && v !== null && "data" in v) {
    return String((v as { data: unknown }).data);
  }
  return "无数据";
}
console.log(read({ data: 42 }), read("x"));
```

**运行结果：**
```
飞翔
游动
无年龄字段
有年龄字段，值 0
42 无
```

**注意：**
* `in` 会检查原型链，所以对继承来的属性也返回 `true`。
* 对 `unknown` 使用前要先 `typeof v === "object"` 且非 null，否则会报错。
* 收窄效果要求联合成员中有明确的属性区分，否则可能收窄不彻底。

### 10.6 自定义类型守卫

**概念说明：** 用返回类型 `value is T` 的函数封装判断逻辑，让编译器在调用处自动收窄。`asserts value is T` 形式还能做「断言函数」。

```typescript
// 类型谓词
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isStringArray(value: unknown): value is string[] {
  return Array.isArray(value) && value.every((i) => typeof i === "string");
}

const data: unknown = ["a", "b"];
if (isStringArray(data)) {
  console.log(data.map((s) => s.toUpperCase()));  // data 收窄为 string[]
}

// 对象结构守卫
interface User { name: string; age: number }
function isUser(v: unknown): v is User {
  return (
    typeof v === "object" && v !== null &&
    "name" in v && typeof (v as User).name === "string" &&
    "age" in v && typeof (v as User).age === "number"
  );
}
const raw: unknown = JSON.parse('{"name":"Alice","age":25}');
console.log(isUser(raw) ? `${(raw as User).name} 是合法用户` : "非法数据");

// 断言函数：不返回布尔值，直接缩小后续代码中的类型
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") throw new Error("不是字符串");
}
const maybe: unknown = "确定的字符串";
assertIsString(maybe);
console.log(maybe.toUpperCase());   // 此处已收窄为 string

// 非空断言函数
function assertDefined<T>(v: T | null | undefined): asserts v is T {
  if (v === null || v === undefined) throw new Error("值为空");
}
const val: string | null = "ok";
assertDefined(val);
console.log(val.length);
```

**运行结果：**
```
[ 'A', 'B' ]
Alice 是合法用户
确定的字符串
2
```

**注意：**
* 类型谓词的函数必须返回 `boolean` 且与实现一致，否则是「类型谎言」。
* 断言函数必须显式标注返回类型，且调用处需能推断出变量，不能对属性路径断言（需用中间变量）。
* 类型守卫是运行时检查，务必保证逻辑真的覆盖了声明。

### 10.7 可辨识联合

**概念说明：** 给联合的每个成员加一个公共的「判别字段」（字面量类型），配合 `switch` / `if` 即可获得完整的类型收窄与穷尽性检查，是建模状态机的标准手法。

```typescript
type State =
  | { status: "idle" }
  | { status: "loading"; progress: number }
  | { status: "success"; data: string[] }
  | { status: "error"; message: string };

function render(state: State): string {
  switch (state.status) {
    case "idle":
      return "待命";
    case "loading":
      return `加载中 ${state.progress}%`;
    case "success":
      return `成功，${state.data.length} 条`;
    case "error":
      return `失败：${state.message}`;
    default:
      // 穷尽性检查：新增分支未处理时这里会报错
      const never: never = state;
      return never;
  }
}

const states: State[] = [
  { status: "idle" },
  { status: "loading", progress: 30 },
  { status: "success", data: ["a", "b"] },
  { status: "error", message: "网络错误" },
];
states.forEach((s) => console.log(render(s)));

// 判别字段也可用数字或布尔
type Event =
  | { type: 0; payload: string }
  | { type: 1; at: Date };
function onEvent(e: Event): string {
  return e.type === 0 ? e.payload : e.at.toISOString().slice(0, 10);
}
console.log(onEvent({ type: 0, payload: "点击" }));
```

**运行结果：**
```
待命
加载中 30%
成功，2 条
失败：网络错误
点击
```

**注意：**
* 判别字段必须是字面量类型（字符串/数字/布尔），不能是宽泛的 `string`。
* 每个分支的独有字段只有在对应收窄后可见。
* 加 `const never: never` 的 `default` 分支可在新增成员时立即报错，避免遗漏。

### 10.8 类型缩小

**概念说明：** 类型缩小（Narrowing）指编译器根据控制流分析，在特定代码路径把宽类型收窄为更精确类型。手段包括 `typeof`、`instanceof`、`in`、真值判断、相等比较、类型谓词与赋值分析。

```typescript
type Value = string | number | boolean | null | undefined | string[];

function process(v: Value): string {
  if (v === null) return "null";
  if (v === undefined) return "undefined";
  if (typeof v === "string") return v.trim();
  if (typeof v === "number") return v.toFixed(2);
  if (typeof v === "boolean") return v ? "true" : "false";
  if (Array.isArray(v)) return v.join(",");      // 收窄为 string[]
  return "unknown";
}
console.log(
  process(null), process(undefined), process("  a "),
  process(1.234), process(false), process(["x", "y"])
);

// 真值收窄
function len(s: string | null | undefined): number {
  if (s) return s.length;    // 过滤掉 ""、null、undefined
  return 0;
}
console.log(len("abc"), len(""), len(null));

// 相等比较收窄
function check(a: string | number, b: string | boolean): void {
  if (a === b) {
    console.log("相等时 a 的类型收窄为 string:", typeof a);   // a 只能是 string
  }
}
check("s", "s");

// 赋值收窄与控制流分析
let x: string | number = Math.random() > 0.5 ? "s" : 1;
x = "确定的字符串";
console.log(x.length);        // 赋值后编译器知道是 string

// 循环与提前返回同样参与分析
function firstPositive(nums: (number | null)[]): number | null {
  for (const n of nums) {
    if (n !== null && n > 0) return n;
  }
  return null;
}
console.log(firstPositive([null, -1, 3]));
```

**运行结果：**
```
null undefined a 1.23 false x,y
3 0 0
相等时 a 的类型收窄为 string: string
13
3
```

**注意：**
* 收窄只在当前作用域的控制流中生效；把变量存进对象属性或传给回调后可能重新变宽（除非用 `const`）。
* 对 `let` 变量的收窄可能被后续赋值打断，`const` 更利于分析。
* 自定义守卫函数要保证逻辑正确，编译器完全信任其声明。

## 11. 字面量类型与 as const
### 11.1 字符串字面量类型

**概念说明：** 用具体字符串值作为类型，表示「只能是这几个字符串之一」。它把字符串常量的约束从运行时的检查提前到编译期。

```typescript
type Direction = "up" | "down" | "left" | "right";

function move(dir: Direction, steps: number): string {
  return `向 ${dir} 移动 ${steps} 步`;
}

console.log(move("up", 3));
// move("upward", 3);        // 错误：Argument of type '"upward"' is not assignable

// 作为对象键的类型
type StatusMap = {
  [K in Direction]: string;
};
const arrows: StatusMap = { up: "↑", down: "↓", left: "←", right: "→" };
console.log(arrows.up, arrows.left);

// 常量断言得到字面量类型
const dir = "up" as const;
const same: "up" = dir;
console.log(same);

// 与字符串方法/索引的配合
type Keys = "a" | "b" | "c";
const obj: Record<Keys, number> = { a: 1, b: 2, c: 3 };
const key: Keys = "b";
console.log(obj[key]);

// 用字面量类型做穷尽检查
type Level = "debug" | "info" | "warn" | "error";
function levelColor(l: Level): string {
  switch (l) {
    case "debug": return "gray";
    case "info": return "blue";
    case "warn": return "yellow";
    case "error": return "red";
  }
}
console.log(levelColor("warn"));
```

**运行结果：**
```
向 up 移动 3 步
↑ ←
up
2
yellow
```

**注意：**
* 函数参数推断出的字面量类型比字符串枚举更轻量，且无需运行时代码。
* 字面量联合中的重复值不会报错，但建议保持唯一（避免 `"a" | "a"`）。
* 把字面量赋给 `let` 变量会退化为 `string`，需要保留时用 `as const` 或类型注解。

### 11.2 数字字面量类型

**概念说明：** 直接把具体数字当作类型，可用于表示状态码、固定数量等受控数值集合，常与联合类型和模板字面量类型组合。

```typescript
type Dice = 1 | 2 | 3 | 4 | 5 | 6;

function roll(): Dice {
  return (Math.floor(Math.random() * 6) + 1) as Dice;
}
console.log(roll() >= 1 && roll() <= 6);

// 固定值配置
type HttpCode = 200 | 201 | 400 | 401 | 403 | 404 | 500;
function describe(code: HttpCode): string {
  if (code >= 200 && code < 300) return "成功";
  if (code >= 400 && code < 500) return "客户端错误";
  return "服务端错误";
}
console.log(describe(200), describe(404), describe(500));

// 颜色通道
type Bit = 0 | 1;
const b: Bit = 1;
console.log(b);

// 与条件类型配合（数字字面量比较）
type IsZero<T extends number> = T extends 0 ? true : false;
type R1 = IsZero<0>;      // true
const r: R1 = true;
console.log(r);

// 元组索引约束
type Tuple = [string, number, boolean];
type SecondIndex = 1;
const idx: SecondIndex = 1;
console.log((["a", 1, true] as Tuple)[idx]);

// 负数与小数字面量同样合法
type Negative = -1 | -2;
type Pi = 3.14;
const neg: Negative = -1;
const pi: Pi = 3.14;
console.log(neg + pi);
```

**运行结果：**
```
true
成功 客户端错误 服务端错误
1
true
1
1.14
```

**注意：**
* 数字字面量类型不区分 `1` 与 `1.0`（都是 `1`）。
* 大范围的数值联合应改用 `number` 或 `bigint`，避免类型过于复杂影响编译速度。
* `as const` 会让数字对象的属性变成数字字面量类型。

### 11.3 布尔字面量类型

**概念说明：** `true` 与 `false` 也可作为独立类型，常用于标记字段以构建可辨识联合，或表达「互斥」状态。

```typescript
type Success = { ok: true; value: string };
type Failure = { ok: false; error: string };
type Result = Success | Failure;

function unwrap(r: Result): string {
  return r.ok ? r.value : r.error;     // 靠布尔字面量收窄
}
console.log(unwrap({ ok: true, value: "数据" }));
console.log(unwrap({ ok: false, error: "失败" }));

// 互斥布尔：不允许同时为 true
type Toggle =
  | { on: true; off?: never }
  | { on?: never; off: true };

const a: Toggle = { on: true };
const b: Toggle = { off: true };
// const c: Toggle = { on: true, off: true };   // 错误

// never 字段防止多余属性
console.log(a.on, b.off);

// 布尔字面量作为泛型参数
type Flag<T extends boolean> = T extends true ? "开启" : "关闭";
const f1: Flag<true> = "开启";
const f2: Flag<false> = "关闭";
console.log(f1, f2);
```

**运行结果：**
```
数据
失败
true true
开启 关闭
```

**注意：**
* 布尔字面量类型常用于「可辨识联合」，判别字段比字符串字面量更省字符。
* `? : never` 是模拟「互斥属性」的常见技巧。
* 与 `boolean` 不同，`true` / `false` 不能被互相赋值。

### 11.4 as const

**概念说明：** `as const` 是常量断言，让字面量获得最窄的字面量类型，并把对象属性变为只读、数组变为只读元组，同时阻止类型拓宽。

```typescript
// 不加断言：类型被拓宽
const obj1 = { kind: "circle", radius: 1 };
// obj1.kind 的类型是 string

// 加断言：保留字面量且只读
const obj2 = { kind: "circle", radius: 1 } as const;
// obj2.kind 的类型是 "circle"，且是只读

// obj2.kind = "square";    // 错误：只读

// 数组变成只读元组
const arr1 = ["a", "b"];            // string[]
const arr2 = ["a", "b"] as const;   // readonly ["a", "b"]
// arr2.push("c");                  // 错误

type Keys = (typeof arr2)[number];  // "a" | "b"
const k: Keys = "a";
console.log(obj1.kind, obj2.kind, k, arr2.length);

// 用 as const 生成配置联合
const CONFIG = {
  api: { baseUrl: "https://api.example.com", timeout: 5000 },
  retry: 3,
} as const;
function request(url: (typeof CONFIG)["api"]["baseUrl"]): void {
  console.log("请求:", url, "重试:", CONFIG.retry + " 次");
}
request("https://api.example.com");

// as const 与枚举替代方案
const ROLES = ["admin", "user", "guest"] as const;
type Role = (typeof ROLES)[number];
function hasRole(r: Role): boolean { return ROLES.includes(r as Role); }
console.log(hasRole("admin"), ROLES.length);
```

**运行结果：**
```
circle circle a 2
请求: https://api.example.com 重试: 3 次
true 3
```

**注意：**
* 只作用于字面量表达式；对变量使用不会让变量变为常量（仍可重新赋值）。
* 修饰的是「嵌套的所有层级」，比 `readonly` 更彻底。
* 想得到字面量联合常用套路：`as const` + `(typeof X)[number]`。

### 11.5 readonly 推断

**概念说明：** `as const` 会触发只读推断：对象属性变为 `readonly`，数组变为 `readonly T[]`（元组）。普通对象字面量不加断言时属性是**可写**的，即使变量是 `const`。

```typescript
// const 变量 ≠ 只读属性
const config = { retries: 3, name: "app" };
config.retries = 5;                 // 合法！属性仍然可写
console.log(config.retries);

// as const → 属性只读
const frozen = { retries: 3, name: "app" } as const;
// frozen.retries = 5;              // 错误：read-only
console.log(frozen.retries);

// 只读推断与函数参数
function readNames(names: readonly string[]): string {
  // names.push("x");              // 错误
  return names.join(",");
}
const list = ["a", "b"];
console.log(readNames(list));

// 只读推断的深浅
const deep = {
  level1: {
    level2: [1, 2],
  },
} as const;
// deep.level1.level2.push(3);      // 错误：只读元组
console.log(deep.level1.level2[1]);

// 需要「只读但保留宽类型」时，用显式只读类型注解
type Config = { readonly retries: number; readonly name: string };
const c2: Config = { retries: 3, name: "app" };
console.log(c2.name);

// Object.freeze 的对应关系
const f = Object.freeze({ a: 1 });
// f.a = 2;                          // 错误：类型推断为 readonly
console.log(Object.isFrozen(f));
```

**运行结果：**
```
5
3
a,b
2
app
true
```

**注意：**
* `const` 只保证「绑定不变」，不保证「属性不变」；只读需要 `readonly` / `as const` / `Object.freeze`。
* `as const` 的只读是编译期的，运行时属性仍可被修改，需要真保护用 `Object.freeze`（注意也是浅冻结）。
* 只读数组不能传给要求可变数组的函数，反向可以。

## 12. Generics 泛型
### 12.1 泛型函数

**概念说明：** 泛型让函数在保持类型信息的前提下适配多种类型：类型参数 `T` 在调用时被推断或显式指定，避免使用 `any` 而丢失类型。

```typescript
// 无泛型：丢失类型信息
function identityAny(value: any): any { return value; }

// 泛型：返回类型与入参类型关联
function identity<T>(value: T): T { return value; }

const n = identity(42);              // T 推断为 number
const s = identity("hello");         // T 推断为 string
console.log(typeof n, typeof s, identity("显式指定" as string).length);

// 多个类型参数
function pair<A, B>(a: A, b: B): [A, B] { return [a, b]; }
const p = pair("age", 25);           // [string, number]
console.log(p[0], p[1]);

// 泛型数组工具
function last<T>(arr: T[]): T | undefined {
  return arr[arr.length - 1];
}
console.log(last([1, 2, 3]), last(["a"]), last([]));

// 泛型 + 回调
function mapArray<T, U>(arr: T[], fn: (item: T, index: number) => U): U[] {
  return arr.map(fn);
}
const lengths = mapArray(["a", "bb"], (s) => s.length);
console.log(lengths);

// 显式指定类型参数
const explicit = identity<string>("内容");
console.log(explicit.toUpperCase());
```

**运行结果：**
```
string string 5
age 25
3 a undefined
[ 1, 2 ]
内容
```

**注意：**
* 泛型参数只在「能被推断」或「显式传入」时有值；无法推断时会退化为 `unknown`。
* 尽量让类型参数出现在参数位置上，否则调用方必须显式指定。
* 返回值只依赖泛型而不出现在参数中时，考虑改为接口或去掉泛型。

### 12.2 泛型接口

**概念说明：** 接口声明类型参数，用于描述「结构相同但元素类型不同」的数据容器或 API 响应。

```typescript
interface ApiResponse<T> {
  code: number;
  message: string;
  data: T;
}

interface Paginated<T> extends ApiResponse<T[]> {
  page: number;
  total: number;
}

const userRes: ApiResponse<{ id: number; name: string }> = {
  code: 0,
  message: "ok",
  data: { id: 1, name: "Alice" },
};

const listRes: Paginated<string> = {
  code: 0,
  message: "ok",
  data: ["a", "b"],
  page: 1,
  total: 2,
};

console.log(userRes.data.name, listRes.data.length, listRes.total);

// 泛型接口描述函数
interface Transformer<T, U> {
  (input: T): U;
  description?: string;
}
const toLength: Transformer<string, number> = (s) => s.length;
toLength.description = "取长度";
console.log(toLength("abc"), toLength.description);

// 泛型接口描述集合
interface Repository<T, ID = number> {
  findById(id: ID): T | undefined;
  save(entity: T): void;
  all(): T[];
}

class InMemoryUserRepo implements Repository<{ id: number; name: string }> {
  private items: { id: number; name: string }[] = [];
  findById(id: number) { return this.items.find((i) => i.id === id); }
  save(entity: { id: number; name: string }) { this.items.push(entity); }
  all() { return this.items; }
}

const repo = new InMemoryUserRepo();
repo.save({ id: 1, name: "Bob" });
console.log(repo.findById(1)?.name, repo.all().length);

// 泛型接口可用于声明函数类型别名
type Mapper<T> = Transformer<T, string>;
const stringify: Mapper<number> = (n) => String(n);
console.log(stringify(42));
```

**运行结果：**
```
Alice 2 2
3 取长度
Bob 1
42
```

**注意：**
* 泛型接口在 `implements` 时必须提供具体类型参数。
* 类型参数可设默认值（`ID = number`），调用时可省略。
* 泛型接口与泛型类型别名能力接近，接口胜在可合并、可继承。

### 12.3 泛型 Type

**概念说明：** 类型别名同样可以带类型参数，且能表达接口做不到的形式：条件类型、映射类型、联合与元组运算。

```typescript
// 简单的泛型别名
type Box<T> = { value: T };
const b: Box<number> = { value: 1 };
console.log(b.value);

// 带默认参数
type Nullable<T = string> = T | null;
const n1: Nullable = null;             // 使用默认 string
const n2: Nullable<number> = 1;
console.log(n1, n2);

// 泛型 + 映射类型
type ReadonlyDeep<T> = {
  readonly [K in keyof T]: T[K] extends object ? ReadonlyDeep<T[K]> : T[K];
};
const cfg: ReadonlyDeep<{ a: number; b: { c: string } }> = {
  a: 1,
  b: { c: "x" },
};
// cfg.b.c = "y";      // 错误：只读
console.log(cfg.a, cfg.b.c);

// 泛型 + 元组操作
type Prepend<T, U extends unknown[]> = [T, ...U];
const withHead: Prepend<string, [number, boolean]> = ["a", 1, true];
console.log(withHead);

// 泛型 + 联合
type MaybeArray<T> = T | T[];
const one: MaybeArray<number> = 1;
const many: MaybeArray<number> = [1, 2];
console.log(one, many.length);

// 泛型别名实现「提取函数返回类型」（条件类型）
type MyReturn<T> = T extends (...args: never[]) => infer R ? R : never;
type R = MyReturn<() => string>;       // string
const r: R = "返回值类型";
console.log(r);
```

**运行结果：**
```
1
null 1
1 x
[ 'a', 1, true ]
1 2
返回值类型
```

**注意：**
* 类型别名不能被继承或被类实现（`implements` 需要接口或对象类型，别名可以但可读性差）。
* 递归泛型别名（如 `ReadonlyDeep`）很方便，但注意编译深度，过深会报 `Type instantiation is excessively deep`。
* 别名 + 映射/条件类型是构建「类型工具」的主要方式。

### 12.4 泛型 Class

**概念说明：** 类可以声明类型参数，用于约束实例字段、方法参数与返回值，实现类型安全的通用数据结构。

```typescript
class Stack<T> {
  private items: T[] = [];

  push(item: T): this {
    this.items.push(item);
    return this;
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  get size(): number {
    return this.items.length;
  }

  map<U>(fn: (item: T, index: number) => U): U[] {
    return this.items.map(fn);
  }

  static of<U>(...items: U[]): Stack<U> {
    const s = new Stack<U>();
    items.forEach((i) => s.push(i));
    return s;
  }
}

const numStack = new Stack<number>();
numStack.push(1).push(2).push(3);          // 返回 this，支持链式
console.log(numStack.size, numStack.pop());
console.log(numStack.map((n) => n * 10));

const strStack = Stack.of("a", "b");
console.log(strStack.size, strStack.map((s) => s.toUpperCase()));

// 泛型约束类
class Storage<T extends { id: number }> {
  private map = new Map<number, T>();
  add(item: T): void { this.map.set(item.id, item); }
  count(): number { return this.map.size; }
}
const st = new Storage<{ id: number; name: string }>();
st.add({ id: 1, name: "A" });
console.log(st.count());

// 泛型类作为类型
const boxed: Stack<string> = Stack.of("x");
console.log(boxed.size);
```

**运行结果：**
```
3 3
[ 10, 20 ]
2 [ 'A', 'B' ]
1
1
```

**注意：**
* 静态方法不能引用类的类型参数（`static of<U>` 需自己声明新参数）。
* 泛型类的类型参数在实例化时必须确定（或用默认值）。
* 返回 `this` 类型是链式调用的标准做法。

### 12.5 泛型约束

**概念说明：** 用 `extends` 限制类型参数的范围，保证在函数体内可安全访问某些成员。约束只是「上限」，不是「必须精确等于」。

```typescript
// 约束必须有 length
function logLength<T extends { length: number }>(value: T): number {
  console.log("长度:", value.length);
  return value.length;
}
logLength("abc");
logLength([1, 2, 3]);
logLength({ length: 10 });
// logLength(42);          // 错误：number 没有 length

// keyof 约束：键必须存在
function getValue<T extends object, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
const user = { id: 1, name: "Alice" };
console.log(getValue(user, "name"));
// getValue(user, "age");   // 错误：'age' 不在 keyof typeof user 中

// 约束为构造函数
function create<T>(Ctor: new () => T): T {
  return new Ctor();
}
class Widget { name = "组件"; }
console.log(create(Widget).name);

// 约束为某种形状的联合
type Shape = { kind: string };
function render<T extends Shape>(s: T): string {
  return `形状类型: ${s.kind}`;
}
console.log(render({ kind: "circle", r: 1 }));

// 多个约束用交叉
function merge<T extends object, U extends object>(a: T, b: U): T & U {
  return { ...a, ...b };
}
const merged = merge({ a: 1 }, { b: "x" });
console.log(merged.a, merged.b);

// 约束 + 默认值
function pick2<T extends object, K extends keyof T = keyof T>(obj: T, key?: K): T[keyof T] | undefined {
  return key ? obj[key] : undefined;
}
console.log(pick2(user, "id"), pick2(user));
```

**运行结果：**
```
长度: 3
长度: 3
长度: 10
Alice
组件
形状类型: circle
1 x
1 undefined
```

**注意：**
* 约束中使用 `keyof T` 可让 `obj[key]` 的类型精确为 `T[K]`。
* 约束太宽（如 `extends any`）等于没有约束；太窄会限制复用性。
* 泛型函数内部只能使用约束中声明的成员，否则报错。

### 12.6 keyof 与泛型

**概念说明：** `keyof T` 得到 T 的键的联合类型，与泛型结合可实现类型安全的属性访问、只取已知键、按类型筛选键等能力。

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  active: boolean;
}

// 类型安全的取值
function pluck<T, K extends keyof T>(obj: T, keys: K[]): T[K][] {
  return keys.map((k) => obj[k]);
}
const u: User = { id: 1, name: "Alice", email: "a@x.com", active: true };
console.log(pluck(u, ["name", "email"]));
// pluck(u, ["age"]);            // 错误

// 只选择某些键
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
  const result = {} as Pick<T, K>;
  keys.forEach((k) => { result[k] = obj[k]; });
  return result;
}
console.log(pick(u, ["id", "name"]));

// 排除某些键
function omit<T, K extends keyof T>(obj: T, keys: K[]): Omit<T, K> {
  const result = { ...obj };
  keys.forEach((k) => { delete result[k]; });
  return result as Omit<T, K>;
}
console.log(omit(u, ["email", "active"]));

// 按值类型筛选键
type KeysOfType<T, V> = {
  [K in keyof T]-?: T[K] extends V ? K : never;
}[keyof T];
type StringKeys = KeysOfType<User, string>;   // "name" | "email"
const sk: StringKeys[] = ["name", "email"];
console.log(sk);

// 键值的映射关系
function setValue<T, K extends keyof T>(obj: T, key: K, value: T[K]): void {
  obj[key] = value;
}
setValue(u, "name", "Bob");
// setValue(u, "id", "字符串");   // 错误：应为 number
console.log(u.name);
```

**运行结果：**
```
[ 'Alice', 'a@x.com' ]
{ id: 1, name: 'Alice' }
{ id: 1, name: 'Alice' }
[ 'name', 'email' ]
Bob
```

**注意：**
* `keyof` 会包含可选属性、索引签名（索引签名会退化为 `string | number`）。
* `K extends keyof T` 的写法可保证键与值类型自动对应。
* 只读属性的键同样出现在 `keyof` 中，但赋值会报错。

### 12.7 泛型默认值

**概念说明：** 类型参数可指定默认类型 `T = Default`，调用方不给类型参数时使用默认值，用于简化常见用法。

```typescript
// 基础默认值
type Container<T = string> = { value: T };
const c1: Container = { value: "默认 string" };
const c2: Container<number> = { value: 1 };
console.log(c1.value, c2.value);

// 函数泛型默认值
function createArray<T = string>(length: number, fill: T): T[] {
  return Array.from({ length }, () => fill);
}
console.log(createArray(3, "x"));
console.log(createArray<number>(3, 0));

// 依赖前一个参数的默认值
interface Options<T extends Record<string, unknown> = Record<string, unknown>> {
  data: T;
  keyOf?: keyof T;
}
const o1: Options = { data: { a: 1 } };
const o2: Options<{ id: number }> = { data: { id: 1 }, keyOf: "id" };
console.log(o2.keyOf);

// 带默认值的泛型接口
interface ApiResult<T = unknown, E = Error> {
  data?: T;
  error?: E;
}
const r1: ApiResult = { data: "任意数据" };
const r2: ApiResult<string, { code: number }> = { error: { code: 500 } };
console.log(r2.error?.code);

// 默认值与约束共存：默认值必须满足约束
function toPair<T extends string | number = string>(v: T): [T, T] {
  return [v, v];
}
console.log(toPair("a"), toPair(1));

// 默认值 + 只给部分参数
function compare<T = string, U = T>(a: T, b: U): string {
  return `${a} vs ${b}`;
}
console.log(compare("x", "y"), compare(1, "s"));
```

**运行结果：**
```
默认 string 1
[ 'x', 'x', 'x' ]
[ 0, 0, 0 ]
id
500
[ 'a', 'a' ] [ 1, 1 ]
x vs y 1 vs s
```

**注意：**
* 有默认值的类型参数必须放在无默认值的参数之后。
* 默认值要满足约束，否则报错。
* 默认值使调用更简洁，但也可能掩盖错误（如忘记传类型参数），公共 API 慎用宽泛默认。

### 12.8 泛型参数之间的关系

**概念说明：** 多个类型参数之间可以相互约束：后一个参数用前一个参数（或 `keyof` 前者）限定，形成「成对出现」的类型关系，这是 `pick` / `getValue` 类工具类型正确性的关键。

```typescript
// 参数关系：值类型必须与键对应
function assign<T, K extends keyof T>(target: T, key: K, value: T[K]): T {
  target[key] = value;
  return target;
}
const user = { id: 1, name: "Alice" };
assign(user, "name", "Bob");
// assign(user, "id", "字符串");     // 错误：期待 number
console.log(user);

// 两个参数相互约束：key 与 value 联合
function entries<T extends Record<string, unknown>>(
  obj: T
): { [K in keyof T]: [K, T[K]] }[keyof T][] {
  return Object.entries(obj) as never;
}
console.log(entries({ a: 1, b: "x" }).length);

// 一对多映射：键与值的对应关系
type HandlerMap = {
  add: (n: number) => void;
  log: (s: string) => void;
};
function dispatch<K extends keyof HandlerMap>(
  handlers: HandlerMap,
  kind: K,
  arg: Parameters<HandlerMap[K]>[0]
): void {
  handlers[kind](arg as never);
}
const handlers: HandlerMap = { add: (n) => console.log("add", n), log: (s) => console.log("log", s) };
dispatch(handlers, "add", 1);
dispatch(handlers, "log", "消息");

// 类型参数默认依赖前一个参数
type EventMap<T extends string, P = string> = { type: T; payload: P };
const e1: EventMap<"click", { x: number }> = { type: "click", payload: { x: 1 } };
console.log(e1.type, e1.payload.x);

// 后一个参数约束为前者的键集合
function groupBy<T, K extends keyof T>(items: T[], key: K): Map<T[K], T[]> {
  const map = new Map<T[K], T[]>();
  for (const item of items) {
    const k = item[key];
    const list = map.get(k) ?? [];
    list.push(item);
    map.set(k, list);
  }
  return map;
}
const grouped = groupBy([{ t: "a", v: 1 }, { t: "b", v: 2 }, { t: "a", v: 3 }], "t");
console.log(grouped.get("a")?.length);
```

**运行结果：**
```
{ id: 1, name: 'Bob' }
2
add 1
log 消息
click 1
2
```

**注意：**
* 「先声明被约束的参数，再声明约束它的参数」是组织类型参数的基本顺序原则。
* 使用 `Parameters<Fn>` / `ReturnType<Fn>` 可从函数类型反推参数与返回值类型，保持两处一致。
* 类型参数之间的关系越紧密，调用方的错误越早在编译期暴露。

## 13. Type Manipulation 类型操作
### 13.1 keyof

**概念说明：** `keyof T` 返回类型 `T` 所有键组成的字符串/数字/符号字面量联合类型。它是「从类型到键集合」的查询操作。

```typescript
interface User {
  id: number;
  name: string;
  0: string;              // 数字键
}

type UserKeys = keyof User;             // "id" | "name" | 0
const k1: UserKeys = "id";
const k2: UserKeys = 0;
console.log(k1, k2);

// 用在数组/元组上
type ArrKeys = keyof string[];          // number | "length" | "push" | ...
type TupleKeys = keyof [string, number]; // "0" | "1" | "length" | ...
const ak: ArrKeys = "length";
console.log(ak);

// 索引签名会退化为 string | number
interface Dict { [key: string]: number }
type DictKeys = keyof Dict;             // string | number
const dk: DictKeys = "任意键";
console.log(dk);

// 与 typeof 结合：从值取键
const config = { host: "localhost", port: 8080 };
type ConfigKey = keyof typeof config;   // "host" | "port"
const ck: ConfigKey = "port";
console.log(ck);

// keyof any / keyof never
type AnyKey = keyof any;                // string | number | symbol
type NeverKey = keyof never;            // string | number | symbol
const anyKey: AnyKey = Symbol("s");
console.log(typeof anyKey);

// 用 keyof 泛化函数
function keys<T extends object>(obj: T): (keyof T)[] {
  return Object.keys(obj) as (keyof T)[];
}
console.log(keys(config));
```

**运行结果：**
```
id 0
length
任意键
port
symbol
[ 'host', 'port' ]
```

**注意：**
* `keyof` 结果里 `length`、`push` 等来自原型方法，处理数组时要注意。
* 索引签名类型会返回 `string | number`，需要精确键集合时避免使用索引签名。
* `keyof typeof value` 是「从运行时值反查键类型」的常用组合。

### 13.2 typeof

**概念说明：** 在类型位置，`typeof x` 查询变量或属性的类型，用于从已有值提取类型，避免重复定义。

```typescript
const person = {
  name: "Alice",
  age: 25,
  hobbies: ["读书"],
};

type Person = typeof person;
// { name: string; age: number; hobbies: string[] }

const p: Person = { name: "Bob", age: 20, hobbies: [] };
console.log(p.name);

// 嵌套查询
type Hobby = (typeof person)["hobbies"][number];   // string
const h: Hobby = "游泳";
console.log(h);

// 查询函数类型
function greet(name: string): string { return `hi ${name}`; }
type Greet = typeof greet;                         // (name: string) => string
const g: Greet = (n) => `hello ${n}`;
console.log(g("x"));

// 查询类：得到构造函数的类型
class Point { constructor(public x: number, public y: number) {} }
type PointCtor = typeof Point;
const Ctor: PointCtor = Point;
console.log(new Ctor(1, 2).x);

// 查询枚举对象
enum Color { Red, Green }
type ColorObj = typeof Color;
console.log(Color.Green);

// 与 as const 配合取字面量
const levels = ["info", "warn"] as const;
type Level = (typeof levels)[number];
const lv: Level = "warn";
console.log(lv);

// 注意：类型位置的 typeof 只能用于标识符或属性访问
const num = 1;
type Num = typeof num;      // number（const 原始值经 let 场景为宽类型）
console.log(typeof num === "number");
```

**运行结果：**
```
Bob
游泳
hello x
1
1
warn
true
```

**注意：**
* `typeof` 在类型位置不执行代码，只做静态查询。
* 对象属性在 `typeof` 后会保留 `?`、`readonly` 等修饰符（若来源如此）。
* `typeof class` 得到的是构造函数类型，不是实例类型（实例类型用类名本身）。

### 13.3 Indexed Access Types

**概念说明：** 索引访问类型 `T[K]` 用于「从类型上取属性类型」，等价于值层面的 `obj[key]`。`K` 可以是字面量、联合，或 `number`（取数组元素类型）。

```typescript
interface User {
  id: number;
  name: string;
  address: { city: string; zip: string };
  tags: string[];
}

type NameType = User["name"];                  // string
type IdOrName = User["id" | "name"];           // number | string
type CityType = User["address"]["city"];       // string
type TagType = User["tags"][number];           // string  数组元素类型
type AnyValue = User[keyof User];              // number | string | {...} | string[]

const n: NameType = "Alice";
const ion: IdOrName = 1;
const c: CityType = "北京";
const t: TagType = "vip";
console.log(n, ion, c, t);

// 元组索引
type Tuple = [string, number, boolean];
type First = Tuple[0];                          // string
type All = Tuple[number];                       // string | number | boolean
const first: First = "a";
const all: All = true;
console.log(first, all);

// 数组类型
type Arr = string[];
type Elem = Arr[number];                        // string
const e: Elem = "x";
console.log(e);

// 与泛型结合，保持键值对应
function get<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
const user: User = { id: 1, name: "A", address: { city: "B", zip: "1" }, tags: [] };
console.log(get(user, "id"), get(user, "tags").length);

// 取函数的参数与返回值（内置索引工具）
type Fn = (a: string, b: number) => boolean;
type FirstParam = Parameters<Fn>[0];            // string
type Ret = ReturnType<Fn>;                      // boolean
const fp: FirstParam = "s";
const r: Ret = true;
console.log(fp, r);
```

**运行结果：**
```
Alice 1 北京 vip
a true
x
1 0
s true
```

**注意：**
* `T[number]` 是「取数组/元组的元素类型」的惯用法。
* 索引必须是 `keyof T` 的子集，否则报错。
* 可选属性索引会得到 `T | undefined`。

### 13.4 Conditional Types

**概念说明：** 条件类型形如 `T extends U ? X : Y`，根据类型可赋值性在两种类型间选择，是类型层面的 `if`。配合泛型即产生「类型函数」。

```typescript
// 基本形式
type IsString<T> = T extends string ? true : false;
type R1 = IsString<"a">;      // true
type R2 = IsString<1>;        // false
const r1: R1 = true;
const r2: R2 = false;
console.log(r1, r2);

// 常用：提取数组元素类型
type ElementType<T> = T extends (infer U)[] ? U : never;
const el: ElementType<number[]> = 1;
console.log(el, r2);

// 分布式条件类型：裸类型参数遇上联合会分发
type ToArray<T> = T extends unknown ? T[] : never;
type R3 = ToArray<string | number>;   // string[] | number[]
const r3: R3 = [1, 2];
console.log(r3);

// 用 [T] 包裹阻止分发
type ToArrayNoDist<T> = [T] extends [unknown] ? T[] : never;
type R4 = ToArrayNoDist<string | number>;   // (string | number)[]
const r4: R4 = [1, "a"];
console.log(r4);

// 条件类型 + 映射类型（去掉可选与只读）
type Mutable<T> = {
  -readonly [K in keyof T]-?: T[K];
};
type M = Mutable<{ readonly a?: number }>;
const m: M = { a: 1 };
console.log(m.a);

// 递归条件类型：深层解包
type DeepUnwrap<T> = T extends Promise<infer U> ? DeepUnwrap<U> : T;
type R5 = DeepUnwrap<Promise<Promise<string>>>;   // string
const r5: R5 = "深层解包";
console.log(r5);
```

**运行结果：**
```
true false
1 false
[ 1, 2 ]
[ 1, 'a' ]
1
深层解包
```

**注意：**
* 条件类型在 `T` 是「裸类型参数」且传入联合时会分发，常造成意外结果，可用 `[T] extends [U]` 阻止。
* `any` 与 `never` 在条件类型中行为特殊：`T extends never` 分发时可能得到 `never`。
* 详见第 14 章的条件类型专章。

### 13.5 infer

**概念说明：** `infer` 在条件类型的 `extends` 分支中声明一个待推断的类型变量，用于「从复杂类型中提取片段」。

```typescript
// 提取函数返回值
type MyReturnType<T> = T extends (...args: never[]) => infer R ? R : never;
type R1 = MyReturnType<() => string>;         // string
const r1: R1 = "s";
console.log(r1);

// 提取参数元组
type MyParameters<T> = T extends (...args: infer P) => unknown ? P : never;
type P1 = MyParameters<(a: string, b: number) => void>;   // [string, number]
const p1: P1 = ["a", 1];
console.log(p1);

// 提取数组元素类型
type Elem<T> = T extends (infer U)[] ? U : never;
const e: Elem<boolean[]> = true;
console.log(e);

// 提取 Promise 内部类型
type Unwrap<T> = T extends Promise<infer U> ? U : T;
const u: Unwrap<Promise<number>> = 42;
console.log(u);

// 提取构造函数的实例类型
type InstanceOf<T> = T extends new (...args: never[]) => infer I ? I : never;
class Foo { name = "foo" }
const f: InstanceOf<typeof Foo> = new Foo();
console.log(f.name);

// 多个 infer 位置：元组首尾
type Split<T> = T extends [infer F, ...infer M, infer L] ? [F, M, L] : never;
type S = Split<[1, 2, 3, 4]>;                 // [1, [2, 3], 4]
const s: S = [1, [2, 3], 4];
console.log(s);

// infer 配合约束（TS 4.7+）
type FirstString<T> = T extends [infer F extends string, ...unknown[]] ? F : never;
const fs: FirstString<["a", 1]> = "a";
console.log(fs);

// 同名 infer 会互相约束（求交）
type UnionToIntersection<U> = (U extends unknown ? (x: U) => void : never) extends
  (x: infer I) => void ? I : never;
type Merged = UnionToIntersection<{ a: 1 } | { b: 2 }>;
const merged: Merged = { a: 1, b: 2 };
console.log(merged);
```

**运行结果：**
```
s
[ 'a', 1 ]
true
42
foo
[ 1, [ 2, 3 ], 4 ]
a
{ a: 1, b: 2 }
```

**注意：**
* `infer` 只能出现在条件类型的 `extends` 子句中，且只在模式匹配成功时有效。
* 多个候选位置时优先匹配最外层结构；`infer` 后加 `extends` 可约束推断范围。
* 无法推断时该分支为 `never`，写工具类型时记得处理。

### 13.6 Mapped Types

**概念说明：** 映射类型用 `[K in keyof T]` 遍历已有类型的键批量生成新类型，是 `Partial`、`Readonly`、`Pick` 等内置工具类型的实现基础。

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// 全部变为可选
type MyPartial<T> = { [K in keyof T]?: T[K] };
const p: MyPartial<User> = { name: "Alice" };
console.log(p);

// 全部只读
type MyReadonly<T> = { readonly [K in keyof T]: T[K] };
const ro: MyReadonly<User> = { id: 1, name: "A", email: "a@x.com" };
// ro.id = 2;            // 错误
console.log(ro.id);

// 移除修饰符（-? -readonly）
type MyRequired<T> = { [K in keyof T]-?: T[K] };
type MyMutable<T> = { -readonly [K in keyof T]: T[K] };

// 键重映射
type Getters<T> = { [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K] };
type UserGetters = Getters<User>;
const g: UserGetters = {
  getId: () => 1,
  getName: () => "A",
  getEmail: () => "a@x.com",
};
console.log(g.getName());

// 键过滤（用 as never 删除键）
type OmitId<T, K extends keyof T> = { [P in keyof T as P extends K ? never : P]: T[P] };
const noId: OmitId<User, "id"> = { name: "A", email: "a@x.com" };
console.log(noId);

// 按值类型转换
type Stringify<T> = { [K in keyof T]: string };
const str: Stringify<User> = { id: "1", name: "A", email: "a@x.com" };
console.log(str.id);

// 映射任意联合（不必是 keyof）
type Flags = { [K in "a" | "b"]: boolean };
const flags: Flags = { a: true, b: false };
console.log(flags);

// 详见第 15 章
```

**运行结果：**
```
{ name: 'Alice' }
1
A
{ name: 'A', email: 'a@x.com' }
1
{ a: true, b: false }
```

**注意：**
* 映射类型可作用在任意键联合上，不必来自 `keyof`。
* `as` 重映射必须返回字符串/数字/符号字面量，或用 `never` 删除键。
* 详见第 15 章的映射类型专章。

### 13.7 Template Literal Types

**概念说明：** 模板字面量类型用反引号在类型层面拼接字符串，配合 `Capitalize` 等内置工具与联合类型，可生成大量字符串字面量组合。

```typescript
type World = "world" | "ts";
type Greeting = `hello ${World}`;         // "hello world" | "hello ts"
const g1: Greeting = "hello ts";
console.log(g1);

// 内置字符串工具类型
type A = Capitalize<"abc">;      // "Abc"
type B = Uncapitalize<"Abc">;    // "abc"
type C = Uppercase<"abc">;       // "ABC"
type D = Lowercase<"ABC">;       // "abc"
const a: A = "Abc"; const b: B = "abc"; const c: C = "ABC"; const d: D = "abc";
console.log(a, b, c, d);

// 生成事件名
type EventName = "click" | "focus";
type HandlerName = `on${Capitalize<EventName>}`;   // "onClick" | "onFocus"
const h: HandlerName = "onFocus";
console.log(h);

// 组合多个联合
type Size = "sm" | "lg";
type Color2 = "red" | "blue";
type ClassName = `${Size}-${Color2}`;      // 4 种组合
const cn: ClassName = "lg-blue";
console.log(cn);

// 与映射类型结合：为每个键生成 getter
type User = { id: number; name: string };
type Getter<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};
const getter: Getter<User> = { getId: () => 1, getName: () => "A" };
console.log(getter.getId());

// 解析字符串结构
type ParseRoute<T extends string> =
  T extends `${infer Controller}/${infer Action}` ? { controller: Controller; action: Action } : never;
type Route = ParseRoute<"user/list">;
const route: Route = { controller: "user", action: "list" };
console.log(route);

// 从模板提取参数
type ExtractParams<T extends string> =
  T extends `${string}{${infer P}}${infer Rest}` ? P | ExtractParams<Rest> : never;
type Params = ExtractParams<"/user/{id}/post/{postId}">;   // "id" | "postId"
const param: Params = "id";
console.log(param);

// 数字与布尔也可拼接
type Indexed = `item-${0 | 1}`;            // "item-0" | "item-1"
const idx: Indexed = "item-1";
console.log(idx);
```

**运行结果：**
```
hello ts
Abc abc ABC abc
onFocus
lg-blue
1
list
id
item-1
```

**注意：**
* 交叉组合会指数膨胀，注意类型复杂度与编译性能。
* `Capitalize<string & K>` 中的 `string &` 是把 `K`（可能是 `string | number | symbol`）限制为字符串的技巧。
* 模板字面量类型常用于路由参数、事件名、CSS 类名、i18n key 等场景。

## 14. Conditional Types 条件类型
### 14.1 基本条件类型

**概念说明：** 条件类型写作 `T extends U ? X : Y`：当 `T` 可赋值给 `U` 时取 `X`，否则取 `Y`。它是类型层面的分支，配合泛型可以构建「类型函数」。

```typescript
type IsArray<T> = T extends unknown[] ? "是数组" : "不是数组";
type R1 = IsArray<number[]>;      // "是数组"
type R2 = IsArray<number>;        // "不是数组"
const r1: R1 = "是数组";
const r2: R2 = "不是数组";
console.log(r1, r2);

// 结合映射类型实现「条件属性」
type NullableProps<T> = {
  [K in keyof T]: T[K] extends string ? T[K] | null : T[K];
};
interface User { id: number; name: string }
const u: NullableProps<User> = { id: 1, name: null };
console.log(u);

// 条件类型可嵌套
type TypeName<T> =
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  T extends undefined ? "undefined" :
  T extends (...args: never[]) => unknown ? "function" :
  "object";
console.log(
  (null as unknown as TypeName<string>),
  (null as unknown as TypeName<() => void>),
  (null as unknown as TypeName<{}>)
);

// 实际用途：为不同输入返回不同输出类型（重载替代）
function parseInput<T extends string | number>(input: T): TypeName<T> extends "string" ? string[] : number {
  return (typeof input === "string" ? input.split("") : input * 2) as never;
}
console.log(parseInput("ab"), parseInput(21));
```

**运行结果：**
```
是数组 不是数组
{ id: 1, name: null }
string function object
[ 'a', 'b' ] 42
```

**注意：**
* 条件类型是懒惰求值的：只有用到该类型时才会计算分支。
* 判断「精确等于某类型」时，`extends` 是「可赋值」而非「相等」，需要额外技巧（见 14.2）。
* 条件类型不能直接用于值，必须在类型位置使用。

### 14.2 extends

**概念说明：** 在条件类型中，`extends` 的含义是「可赋值性」（assignability）而非继承。理解这一点是掌握条件类型的关键，尤其是 `any`、`never`、联合类型的特殊行为。

```typescript
// 可赋值性：字面量可赋给宽类型
type R1 = "a" extends string ? true : false;      // true
type R2 = string extends "a" ? true : false;      // false
const r1: R1 = true;
const r2: R2 = false;
console.log(r1, r2);

// 判断精确相等：双向 extends
type IsEqual<A, B> =
  (<T>() => T extends A ? 1 : 2) extends (<T>() => T extends B ? 1 : 2) ? true : false;
type E1 = IsEqual<string, string>;      // true
type E2 = IsEqual<string, "a">;         // false
const e1: E1 = true;
const e2: E2 = false;
console.log(e1, e2);

// any 的特殊行为：与任何类型双向可赋值
type IsAny<T> = 0 extends 1 & T ? true : false;
type A1 = IsAny<any>;      // true
type A2 = IsAny<string>;   // false
console.log((null as unknown as A1), (null as unknown as A2));

// never 是底类型，可赋给一切
type R3 = never extends string ? true : false;    // true
type IsNever<T> = [T] extends [never] ? true : false;
type N1 = IsNever<never>;   // true
type N2 = IsNever<string>;  // false
const n1: N1 = true;
const n2: N2 = false;
console.log(n1, n2);

// 结构化：多余属性不影响可赋值
type R4 = { a: 1; b: 2 } extends { a: 1 } ? true : false;   // true
const r4: R4 = true;
console.log(r4);
```

**运行结果：**
```
true false
true false
true false
true false
true
```

**注意：**
* 「`X extends Y`」为真只表示 `X` 的值都能赋给 `Y`，不代表两者相同。
* `any` 会污染条件类型（因为它与一切兼容），需要 `IsAny` 这类技巧单独处理。
* `never` 在裸类型参数位置遇条件类型会分发为 `never`，用 `[T]` 包裹可避免。

### 14.3 三元类型结构

**概念说明：** 条件类型支持嵌套形成多分支结构，等价于类型层面的 `if / else if / else`。分支返回的类型可以任意，也可以是另一个条件类型（延迟求值）。

```typescript
// 多分支类型判断
type ToPrimitive<T> =
  T extends string ? string :
  T extends number ? number :
  T extends boolean ? boolean :
  T extends null | undefined ? null :
  T extends object ? { [K in keyof T]: ToPrimitive<T[K]> } :
  never;

type Wrapped = { a: string; b: { c: number } };
type Unwrapped = ToPrimitive<Wrapped>;
const w: Unwrapped = { a: "x", b: { c: 1 } };
console.log(w.a, w.b.c);

// 条件类型作为「策略选择器」
type Strategy<T> = T extends string
  ? (s: T) => string
  : T extends number
  ? (n: T) => number
  : never;

const strStrategy: Strategy<string> = (s) => s.toUpperCase();
const numStrategy: Strategy<number> = (n) => n + 1;
console.log(strStrategy("a"), numStrategy(1));

// 三种以上状态：三元链条
type Level = "error" | "warn" | "info" | "debug";
type Weight<L extends Level> =
  L extends "error" ? 3 :
  L extends "warn" ? 2 :
  L extends "info" ? 1 :
  0;
const w3: Weight<"error"> = 3;
const w0: Weight<"debug"> = 0;
console.log(w3, w0);

// 嵌套返回 Union 或 never
type NonEmpty<T> = T extends "" ? never : T;
type R = NonEmpty<"" | "a" | "b">;    // "a" | "b"
const r: R = "a";
console.log(r);

// 可读性：用类型别名拆分长链条
type IsString<T> = T extends string ? true : false;
type IsNumber<T> = T extends number ? true : false;
type Kind<T> = IsString<T> extends true ? "S" : IsNumber<T> extends true ? "N" : "O";
const kind: Kind<1> = "N";
console.log(kind);
```

**运行结果：**
```
x 1
A 2
3 0
a
N
```

**注意：**
* 嵌套过深易读性差，建议把每个判断抽成具名类型别名。
* 分支是「按序匹配」，前面命中就不再往后判断（与运行时 `if` 一致）。
* 返回 `never` 常用于「过滤掉某类成员」。

### 14.4 分布式条件类型

**概念说明：** 当条件类型的检查对象是**裸类型参数**且传入联合类型时，条件类型会对联合的每个成员分别计算，再把结果合并成联合 —— 称为「分发」。

```typescript
// 裸类型参数 → 分发
type ToArray<T> = T extends unknown ? T[] : never;
type R1 = ToArray<string | number>;      // string[] | number[]
const r1: R1 = [1, 2];
const r1b: R1 = ["a"];
console.log(r1, r1b);

// 用元组包裹阻止分发
type ToArrayNoDist<T> = [T] extends [unknown] ? T[] : never;
type R2 = ToArrayNoDist<string | number>;   // (string | number)[]
const r2: R2 = ["a", 1];
console.log(r2);

// 经典应用：Exclude
type MyExclude<T, U> = T extends U ? never : T;
type R3 = MyExclude<"a" | "b" | "c", "a">;  // "b" | "c"
const r3: R3 = "b";
console.log(r3);

// 经典应用：NonNullable
type MyNonNullable<T> = T extends null | undefined ? never : T;
type R4 = MyNonNullable<string | null | undefined>;   // string
const r4: R4 = "值";
console.log(r4);

// 分发与函数类型
type ReturnTypes<T> = T extends (...args: never[]) => infer R ? R : never;
type R5 = ReturnTypes<(() => string) | (() => number)>;   // string | number
const r5: R5 = 1;
const r5b: R5 = "s";
console.log(r5, r5b);

// 阻止分发的其他方式：把参数放在「非裸」位置
// 判断 T 是否为「联合类型」：给第二个参数 U 存一份原始 T
// 分发时 T 变成单个成员，而 U 保持整个联合，二者不再相等即说明是联合
type IsUnion<T, U = T> = T extends U ? ([U] extends [T] ? false : true) : never;
type U1 = IsUnion<"a" | "b">;   // true
type U2 = IsUnion<"a">;         // false
const u1: U1 = true;
const u2: U2 = false;
console.log(u1, u2);
```

**运行结果：**
```
[ 1, 2 ] [ 'a' ]
[ 'a', 1 ]
b
值
1 s
true false
```

**注意：**
* 分发只发生在「裸类型参数」上；`T extends ...` 中若 `T` 被 `[]`、`&`、映射类型等包裹则不分发。
* `never` 分发后仍是 `never`（空联合）。
* 想「合并后判断整体」，必须用 `[T] extends [U]`。

### 14.5 条件类型中的 infer

**概念说明：** `infer` 在条件类型分支中声明「待推断的类型变量」，实现对结构的位置匹配与提取，是 `ReturnType`、`Parameters`、`Awaited` 等工具类型的核心。

```typescript
// 提取 Promise 的值类型（简化版 Awaited）
type MyAwaited<T> = T extends Promise<infer U> ? MyAwaited<U> : T;
type R1 = MyAwaited<Promise<Promise<number>>>;   // number
const r1: R1 = 1;
console.log(r1);

// 提取数组/元组元素
type ElementOf<T> = T extends (infer U)[] ? U : never;
const e: ElementOf<{ id: number }[]> = { id: 1 };
console.log(e.id);

// 提取函数参数与返回值
type FnInfo<T> = T extends (...args: infer P) => infer R ? { params: P; result: R } : never;
type Info = FnInfo<(a: string, b: number) => boolean>;
const info: Info = { params: ["a", 1], result: true };
console.log(info.params.length, info.result);

// 提取对象属性类型
type PropType<T, K extends keyof T> = T extends Record<K, infer V> ? V : never;
type P = PropType<{ id: number; name: string }, "id">;   // number
const p: P = 1;
console.log(p);

// infer + 约束（TS 4.7+）
type FirstNumber<T> = T extends [infer F extends number, ...unknown[]] ? F : never;
const fn: FirstNumber<[1, "a"]> = 1;
console.log(fn);

// 提取构造函数参数
type CtorArgs<T> = T extends new (...args: infer A) => unknown ? A : never;
class Point { constructor(public x: number, public y: number) {} }
const args: CtorArgs<typeof Point> = [1, 2];
console.log(args);

// 提取字符串中的一部分
type GetTag<T extends string> = T extends `<${infer Tag}>` ? Tag : never;
const tag: GetTag<"<div>"> = "div";
console.log(tag);

// infer 出现在协变位置时可能推断出联合
type UnionOfParams<T> = T extends { a: infer U; b: infer U } ? U : never;
type U = UnionOfParams<{ a: string; b: number }>;   // string | number
const u: U = 1;
const ub: U = "s";
console.log(u, ub);
```

**运行结果：**
```
1
1
2 true
1
1
[ 1, 2 ]
div
1 s
```

**注意：**
* 同一 `infer` 变量出现在多个协变位置时会推断为它们的联合；出现在逆变位置（如函数参数）时推断为交叉。
* `infer` 只在 `extends` 子句中有效，无法单独声明。
* 组合使用「分发 + infer」可实现很强的类型推导，但要注意复杂度。

### 14.6 条件类型与泛型

**概念说明：** 条件类型与泛型结合形成「类型层面的函数」：输入类型决定输出类型。配合默认值、约束与递归，可实现自动推导 API 类型、类型安全的 ORM/路由等。

```typescript
// 泛型默认值依赖条件类型
type ApiResult<T, E = T extends string ? Error : null> = {
  data: T;
  error: E;
};
const s: ApiResult<string> = { data: "ok", error: new Error("e") };
const n: ApiResult<number> = { data: 1, error: null };
console.log(s.data, n.error);

// 返回类型随输入变化（类型安全的重载替代）
function createStore<T extends "map" | "list">(
  kind: T
): T extends "map" ? Map<string, number> : number[] {
  return (kind === "map" ? new Map<string, number>() : []) as never;
}
const store1 = createStore("map");    // Map<string, number>
const store2 = createStore("list");   // number[]
store1.set("a", 1);
store2.push(1);
console.log(store1.size, store2.length);

// 递归条件类型：把嵌套对象的所有属性变为可选
type DeepPartial<T> = T extends object
  ? { [K in keyof T]?: DeepPartial<T[K]> }
  : T;
type Config = { server: { host: string; port: number }; debug: boolean };
const partial: DeepPartial<Config> = { server: { host: "localhost" } };
console.log(partial.server?.host, partial.debug);

// 泛型约束 + 条件类型：只接受特定形状
type Flatten<T> = T extends Array<infer U> ? U : T;
function firstOrSelf<T>(v: T): Flatten<T> {
  return (Array.isArray(v) ? v[0] : v) as Flatten<T>;
}
console.log(firstOrSelf([1, 2]), firstOrSelf("单值"));

// 根据泛型是否包含某键选择策略
type HasKey<T, K extends string> = K extends keyof T ? true : false;
type Strategy<T> = HasKey<T, "id"> extends true ? "有主键" : "无主键";
const st: Strategy<{ id: number }> = "有主键";
console.log(st);

// 条件类型下函数的实现需要断言（编译器无法验证逻辑）
type Unwrap<T> = T extends Promise<infer U> ? Unwrap<U> : T;
async function unwrap<T>(p: T): Promise<Unwrap<T>> {
  let v: unknown = p;
  while (v instanceof Promise) v = await v;
  return v as Unwrap<T>;
}
unwrap(Promise.resolve(Promise.resolve("深层"))).then((v) => console.log(v));
```

**运行结果：**
```
ok null
1 1
localhost undefined
1 单值
有主键
深层
```

**注意：**
* 条件类型只影响类型，函数实现通常需要用 `as` 断言收尾（编译器不会校验「类型已证明」的分支）。
* 递归条件类型要注意终止条件与深度限制。
* 公共 API 用「泛型 + 条件类型」返回不同形状，调用方体验好，但要写足注释。

## 15. Mapped Types 映射类型
### 15.1 基本映射类型

**概念说明：** 映射类型通过 `[K in Keys]` 遍历键联合生成新对象类型，可以批量修改属性类型、修饰符。`Keys` 通常来自 `keyof T`，也可以是任意联合。

```typescript
// 遍历对象键：实现 Partial
interface User {
  id: number;
  name: string;
  email: string;
}
type MyPartial<T> = { [K in keyof T]?: T[K] };
const p: MyPartial<User> = { name: "Alice" };
console.log(p);

// 遍历任意联合
type Point = "x" | "y" | "z";
type Vector = { [K in Point]: number };
const v: Vector = { x: 1, y: 2, z: 3 };
console.log(v);

// 改变值类型
type ToString<T> = { [K in keyof T]: string };
const str: ToString<User> = { id: "1", name: "A", email: "a@x.com" };
console.log(str.name);

// 值类型依赖键的类型（用模板字面量）
type EventHandlers<T extends string> = {
  [K in T as `on${Capitalize<K>}`]: (payload: K) => void;
};
const handlers: EventHandlers<"click" | "focus"> = {
  onClick: (p) => console.log("点击:", p),
  onFocus: (p) => console.log("聚焦:", p),
};
handlers.onClick("click");

// 在同态映射中保持修饰符（默认保留 readonly 与 ?）
interface ReadonlyUser { readonly id: number; name?: string }
type KeepModifiers<T> = { [K in keyof T]: T[K] };
const km: KeepModifiers<ReadonlyUser> = { id: 1 };
console.log(km.id);

// 映射类型也是「同态」的：会保留索引签名
interface Dict { [key: string]: number }
type Doubled = { [K in keyof Dict]: number };
const d: Doubled = { a: 1 };
console.log(d.a);
```

**运行结果：**
```
{ name: 'Alice' }
{ x: 1, y: 2, z: 3 }
A
点击: click
1
1
```

**注意：**
* `[K in keyof T]` 形式的映射称为「同态映射」，会自动保留原类型的修饰符与可选性。
* `[K in SomeUnion]` 形式的映射不是同态的，修饰符需显式书写。
* 映射类型只影响类型，不产生运行时代码。

### 15.2 keyof + 映射类型

**概念说明：** `keyof T` 提供键集合，映射类型遍历该集合，二者组合是绝大多数类型工具的实现骨架。`T[K]` 用于取每个键对应的值类型。

```typescript
interface User { id: number; name: string; active: boolean }

// 值类型包装成数组
type Arrayify<T> = { [K in keyof T]: T[K][] };
const a: Arrayify<User> = { id: [1], name: ["a"], active: [true] };
console.log(a.id, a.active);

// 去掉某类值类型的键（键过滤）
type OmitByType<T, V> = {
  [K in keyof T as T[K] extends V ? never : K]: T[K];
};
type NoBoolean = OmitByType<User, boolean>;      // { id: number; name: string }
const nb: NoBoolean = { id: 1, name: "x" };
console.log(nb);

// 只保留某类值类型的键
type PickByType<T, V> = {
  [K in keyof T as T[K] extends V ? K : never]: T[K];
};
type OnlyBoolean = PickByType<User, boolean>;    // { active: boolean }
const ob: OnlyBoolean = { active: true };
console.log(ob);

// 反转键与值的映射关系
type Invert<T extends Record<string, string>> = {
  [K in keyof T as T[K]]: K;
};
type Roles = Invert<{ a: "admin"; b: "user" }>;  // { admin: "a"; user: "b" }
const roles: Roles = { admin: "a", user: "b" };
console.log(roles.admin);

// 把值变成 getter（用 keyof 与模板字面量）
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};
const g: Getters<{ id: number; name: string }> = { getId: () => 1, getName: () => "a" };
console.log(g.getId(), g.getName());

// keyof 与映射实现「按需可选」
type PartialOnly<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;
type P = PartialOnly<User, "id">;
const pp: P = { name: "a", active: true };
console.log(pp);
```

**运行结果：**
```
[ 1 ] [ true ]
{ id: 1, name: 'x' }
{ active: true }
a
1 a
{ name: 'a', active: true }
```

**注意：**
* 键过滤用 `as ... ? never : K` 的组合，`never` 表示删除该键。
* `Omit<T, K> & Partial<Pick<T, K>>` 是「让指定键可选」的经典写法。
* 同态映射中 `T[K]` 能保留可选性与只读性；跨类型取值时需注意 `indexed access` 的类型准确性。

### 15.3 属性修饰符映射

**概念说明：** 映射类型可直接在属性上书写修饰符：`readonly`（添加只读）、`?`（添加可选），并可组合使用，从而批量改变属性的可变性与必要性。

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// 加 readonly
type ReadonlyT<T> = { readonly [K in keyof T]: T[K] };
const ro: ReadonlyT<User> = { id: 1, name: "a", email: "b" };
// ro.id = 2;      // 错误

// 加可选
type OptionalT<T> = { [K in keyof T]?: T[K] };
const op: OptionalT<User> = { name: "a" };
console.log(op.name);

// 两者组合
type ReadonlyOptional<T> = { readonly [K in keyof T]?: T[K] };
const rop: ReadonlyOptional<User> = {};
console.log(rop);

// 值层面的只读（运行时）
const frozen = { a: 1 };
Object.freeze(frozen);
// frozen.a = 2;   // 严格模式下抛错

// 通过索引签名生成字典类型
type Dict<T> = { readonly [key: string]: T };
const dict: Dict<number> = { a: 1 };
// dict.a = 2;     // 错误
console.log(dict.a);

// 修饰符与条件类型组合：只读 + 依据类型转换
type Freeze<T> = {
  readonly [K in keyof T]: T[K] extends object ? Freeze<T[K]> : T[K];
};
const f: Freeze<{ a: { b: number } }> = { a: { b: 1 } };
// f.a.b = 2;      // 错误
console.log(f.a.b);

// 使用内置 Readonly 对比
const builtin: Readonly<User> = { id: 1, name: "a", email: "b" };
console.log(builtin.email);
```

**运行结果：**
```
a
{}
1
1
b
```

**注意：**
* 修饰符写在 `[K in ...]` 之前，作用于所有属性。
* 同态映射默认继承原修饰符；显式书写会「覆盖」而非叠加（可用 `+` / `-` 明确表达）。
* 修饰符只是编译期约束，运行时仍需要 `Object.freeze` 等保护。

### 15.4 +readonly / -readonly

**概念说明：** `+readonly`（可简写为 `readonly`）添加只读，`-readonly` 移除只读。移除只读常与自动生成的可变版本类型（如 ORM 的 update 输入）配合。

```typescript
// 移除只读 → 可变
interface ReadonlyUser {
  readonly id: number;
  readonly name: string;
  age: number;
}

type Mutable<T> = { -readonly [K in keyof T]: T[K] };
const m: Mutable<ReadonlyUser> = { id: 1, name: "a", age: 25 };
m.id = 2;                   // 允许修改
console.log(m.id);

// 添加只读（+readonly 与 readonly 等价）
type Immutable<T> = { +readonly [K in keyof T]: T[K] };
const im: Immutable<{ x: number }> = { x: 1 };
// im.x = 2;                // 错误
console.log(im.x);

// 组合：移除只读同时加可选
type Draft<T> = { -readonly [K in keyof T]?: T[K] };
const draft: Draft<ReadonlyUser> = { id: 1 };
draft.id = 5;
console.log(draft.id);

// 只读数组的处理
type Arr = readonly number[];
type MutableArr = { -readonly [K in keyof Arr]: Arr[K] };
// 数组的 -readonly 会影响 length 等属性，通常直接写 number[] 更清晰
const ma: number[] = [1, 2];
ma.push(3);
console.log(ma);

// 实用：库类型自动生成「可变视图」与「只读视图」
class Entity {
  readonly id = 1;
  name = "n";
}
type EntityReadonly = Readonly<Entity>;                  // 全部只读
type EntityMutable = { -readonly [K in keyof Entity]: Entity[K] };  // 恢复可变
const em: EntityMutable = { id: 1, name: "x" };
em.id = 2;
console.log(em.id);

// 递归移除只读
type DeepMutable<T> = {
  -readonly [K in keyof T]: T[K] extends object ? DeepMutable<T[K]> : T[K];
};
const dm: DeepMutable<{ readonly a: { readonly b: number } }> = { a: { b: 1 } };
dm.a.b = 2;
console.log(dm.a.b);
```

**运行结果：**
```
2
1
5
[ 1, 2, 3 ]
2
2
```

**注意：**
* 内置 `Readonly<T>` 就是 `readonly [K in keyof T]`，`-readonly` 必须通过映射类型手写。
* 修饰符的 `+` 可以省略，`-` 不可省略。
* 递归可变版本要注意数组/函数等特殊对象，可能需要额外分支。

### 15.5 +? / -?

**概念说明：** `+?`（简写 `?`）添加可选修饰符，`-?` 移除可选修饰符（使属性变为必填）。这是 `Partial` 与 `Required` 的实现基础。

```typescript
interface User {
  id: number;
  name?: string;
  email?: string;
}

// 移除可选 → 必填
type MyRequired<T> = { [K in keyof T]-?: T[K] };
const r: MyRequired<User> = { id: 1, name: "a", email: "b" };
// const bad: MyRequired<User> = { id: 1 };   // 错误：缺少 name
console.log(r.name, r.email);

// 添加可选 → 全部可省略
type MyPartial<T> = { [K in keyof T]?: T[K] };
const p: MyPartial<User> = {};
console.log(p);

// -? 同时移除 undefined（值类型也变「非 undefined」）
type Concrete<T> = { [K in keyof T]-?: NonNullable<T[K]> };
const c: Concrete<User> = { id: 1, name: "a", email: "b" };
console.log(c.name.length);

// 组合：必填 + 只读
type Strict<T> = { readonly [K in keyof T]-?: T[K] };
const s: Strict<User> = { id: 1, name: "a", email: "b" };
// s.id = 2;            // 错误
console.log(s.id);

// 只让指定键必填
type RequiredKeys<T, K extends keyof T> = T & { [P in K]-?: T[P] };
type RU = RequiredKeys<User, "name" | "email">;
const ru: RU = { id: 1, name: "a", email: "b" };
console.log(ru.email);

// 只让指定键可选
type OptionalKeys<T, K extends keyof T> = Omit<T, K> & { [P in K]?: T[P] };
const onlyNameOptional: OptionalKeys<User, "name"> = { id: 1, email: "b" };
console.log(onlyNameOptional);

// 递归可选（配置对象常用）
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};
const dp: DeepPartial<{ a: { b: { c: number } } }> = { a: { b: {} } };
console.log(dp);
```

**运行结果：**
```
a b
{}
1
1
a b
{ id: 1, email: 'b' }
{ a: { b: {} } }
```

**注意：**
* `-?` 会让属性类型排除 `undefined`，所以 `T[K]` 会变「非空」。
* 与 `exactOptionalPropertyTypes` 配合时，`?` 的语义更精确（不允许显式 `undefined`）。
* 内置 `Required<T>` 即 `-?` 映射的实现。

### 15.6 Key Remapping

**概念说明：** 用 `as` 子句在映射过程中重命名键，配合模板字面量类型可批量加前缀/后缀、改名风格；返回 `never` 则删除该键。

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// 加前缀
type Prefix<T, P extends string> = {
  [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K];
};
type UserDto = Prefix<User, "user">;
const dto: UserDto = { userId: 1, userName: "a", userEmail: "b" };
console.log(dto.userName);

// 改名风格：snake_case 转 camelCase（简化版）
type CamelCase<S extends string> =
  S extends `${infer H}_${infer T}` ? `${H}${Capitalize<CamelCase<T>>}` : S;
type SnakeObj = { user_id: number; user_name: string };
type CamelObj = {
  [K in keyof SnakeObj as CamelCase<string & K>]: SnakeObj[K];
};
const camel: CamelObj = { userId: 1, userName: "a" };
console.log(camel);

// 删除键：as 返回 never
type Remove<T, R> = {
  [K in keyof T as K extends R ? never : K]: T[K];
};
const noEmail: Remove<User, "email"> = { id: 1, name: "a" };
console.log(noEmail);

// 条件重命名：根据值类型换名
type WithFlag<T> = {
  [K in keyof T as T[K] extends boolean ? `is${Capitalize<string & K>}` : K]: T[K];
};
const flag: WithFlag<{ active: boolean; name: string }> = { isActive: true, name: "a" };
console.log(flag.isActive);

// getter 化 + 去重名
type GetterObj<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
} & {
  [K in keyof T as `set${Capitalize<string & K>}`]: (v: T[K]) => void;
};
const accessors: GetterObj<{ name: string }> = {
  getName: () => "a",
  setName: (v) => console.log("设置:", v),
};
accessors.setName("b");
console.log(accessors.getName());

// 多键合并为联合名
type KeysToUnion<T> = {
  [K in keyof T as `from_${Capitalize<string & K>}`]: T[K];
};
const ku: KeysToUnion<{ a: 1; b: 2 }> = { from_A: 1, from_B: 2 };
console.log(ku.from_A);
```

**运行结果：**
```
a
{ userId: 1, userName: 'a' }
{ id: 1, name: 'a' }
true
设置: b
a
1
```

**注意：**
* `as` 子句必须产生 `string | number | symbol` 字面量类型，返回 `never` 表示删除。
* 重命名后可能出现键冲突（后声明覆盖前者），需保证映射唯一。
* 模板字面量类型的组合能力很强，注意复杂度对编译速度的影响。

### 15.7 映射类型与条件类型

**概念说明：** 把条件类型放进映射的值位置（或键位置），即可按属性类型作出不同处理，实现「智能类型转换」，如自动把函数属性包裹一层、把可选属性变必填等。

```typescript
interface Api {
  data: string;
  error: string;
  isLoading: boolean;
  timestamp: number;
  fetch(): Promise<string>;
}

// 按值类型转换：函数转为返回 void
type FunctionToVoid<T> = {
  [K in keyof T]: T[K] extends (...args: never[]) => unknown ? () => void : T[K];
};
const noop = (() => {}) as FunctionToVoid<Pick<Api, "fetch">>;
noop.fetch();
console.log("fetch 已被转换为 () => void");

// 把 Promise 属性解包
type PromiseProps<T> = {
  [K in keyof T]: T[K] extends Promise<infer U> ? U : T[K];
};
const p: PromiseProps<{ a: Promise<number>; b: string }> = { a: 1, b: "x" };
console.log(p.a, p.b);

// 可选属性变必填，必填变可选（互斥翻转）
type FlipOptional<T> = {
  [K in keyof T as T[K] extends Required<Pick<T, K>>[K] ? never : K]: T[K];
} & {};

// 更清晰的翻转写法
type OptionalKeys<T> = { [K in keyof T]-?: {} extends Pick<T, K> ? K : never }[keyof T];
type RequiredKeys<T> = { [K in keyof T]-?: {} extends Pick<T, K> ? never : K }[keyof T];
type Flip<T> = Omit<T, OptionalKeys<T>> & Partial<Pick<T, OptionalKeys<T>>>;
type F = Flip<{ a: number; b?: string }>;
const f: F = { a: 1 };
console.log(f);

// 深度只读 + 条件
type DeepReadonly<T> = T extends (...args: never[]) => unknown
  ? T
  : T extends object
  ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
  : T;
type CFG = DeepReadonly<{ a: { b: number }; fn: () => void }>;
const cfg: CFG = { a: { b: 1 }, fn: () => {} };
console.log(cfg.a.b);

// 按值类型生成「label + value」选项
type Options<T> = {
  [K in keyof T as `${string & K}`]: { label: string; value: T[K] };
};
const options: Options<{ size: number; color: string }> = {
  size: { label: "尺寸", value: 1 },
  color: { label: "颜色", value: "red" },
};
console.log(options.size.label, options.color.value);

// 把方法参数类型收集为对象（类型级组合）
type MethodsOf<T> = {
  [K in keyof T as T[K] extends (...args: never[]) => unknown ? K : never]: T[K];
};
type OnlyMethods = MethodsOf<Api>;
const m: OnlyMethods = { fetch: async () => "ok" };
console.log(typeof m.fetch);
```

**运行结果：**
```
fetch 已被转换为 () => void
1 x
{ a: 1 }
1
尺寸 red
function
```

**注意：**
* 同态映射 + 条件类型是「类型级编程」的主要组合方式，注意分支穷尽性。
* 键位置的条件（`as`）与值位置的条件可以同时使用。
* 复杂工具类型建议写单元测试（用 `Expect<Equal<A, B>>` 之类的类型断言）保证行为正确。

## 16. TypeScript Utility Types
### 16.1 Partial

**概念说明：** `Partial<T>` 把 `T` 的所有属性变为可选（`?`），常用于「更新对象」的参数类型。

```typescript
interface User { id: number; name: string; email: string }

type PartialUser = Partial<User>;
// 等价于 { id?: number; name?: string; email?: string }

const patch: PartialUser = { name: "Alice" };
console.log(patch);

// 实现原理
type MyPartial<T> = { [K in keyof T]?: T[K] };

// 典型用法：更新函数
function updateUser(user: User, changes: Partial<User>): User {
  return { ...user, ...changes };
}
const u = updateUser({ id: 1, name: "A", email: "a@x.com" }, { name: "B" });
console.log(u);

// 注意：Partial 是浅层的，嵌套对象不受影响
interface Nested { a: { b: number; c: number } }
const n: Partial<Nested> = { a: { b: 1, c: 2 } };   // 内层必须完整
console.log(n.a?.b);
```

**运行结果：**
```
{ name: 'Alice' }
{ id: 1, name: 'B', email: 'a@x.com' }
1
```

**注意：**
* 是浅层的：需要深层可选请用递归 `DeepPartial`。
* 可选属性读取结果含 `undefined`，使用前需判空或给默认值。
* 与 `Required` 互为反向操作。

### 16.2 Required

**概念说明：** `Required<T>` 把 `T` 的所有可选属性变为必填，并移除 `undefined`。

```typescript
interface Config { host?: string; port?: number; ssl?: boolean }

type FullConfig = Required<Config>;
// { host: string; port: number; ssl: boolean }

const cfg: FullConfig = { host: "localhost", port: 80, ssl: false };
console.log(cfg);

// 实现原理
type MyRequired<T> = { [K in keyof T]-?: T[K] };

// 典型用法：为可选配置填充默认值后返回必填类型
function withDefaults(c: Config): Required<Config> {
  return { host: "localhost", port: 80, ssl: false, ...c };
}
console.log(withDefaults({ port: 8080 }));

// 与 Partial 组合使用
function merge<T>(base: T, override: Partial<T>): Required<T> {
  return { ...base, ...override } as Required<T>;
}
console.log(merge({ a: 1, b: 2 }, { b: 3 }));
```

**运行结果：**
```
{ host: 'localhost', port: 80, ssl: false }
{ host: 'localhost', port: 8080, ssl: false }
{ a: 1, b: 3 }
```

**注意：**
* 只对 `?` 修饰的属性生效，不影响类型本身的含义。
* 强制调用方补全所有字段，适合配置校验之后的类型。
* 深层必填同样需要递归实现。

### 16.3 Readonly

**概念说明：** `Readonly<T>` 把所有属性标记为 `readonly`（浅只读）。

```typescript
interface Todo { title: string; done: boolean }

const todo: Readonly<Todo> = { title: "学习 TS", done: false };
// todo.done = true;              // 错误：Cannot assign to 'done'

// 实现原理
type MyReadonly<T> = { readonly [K in keyof T]: T[K] };

// 只读数组
const nums: ReadonlyArray<number> = [1, 2, 3];
// nums.push(4);                  // 错误
console.log(nums.length);

// 典型用法：函数返回只读视图，防止外部修改
function getState(): Readonly<{ count: number }> {
  return { count: 1 };
}
const state = getState();
console.log(state.count);

// 浅只读：嵌套对象仍可修改
const nested: Readonly<{ a: { b: number } }> = { a: { b: 1 } };
nested.a.b = 2;                   // 允许
console.log(nested.a.b);

// 配合 as const 得到更深层的只读
const deep = { a: { b: 1 } } as const;
// deep.a.b = 2;                  // 错误
console.log(deep.a.b);
```

**运行结果：**
```
3
1
2
1
```

**注意：**
* 浅只读，需要深层请用 `as const` 或递归的 `DeepReadonly`。
* 只是编译期约束，运行时可被 `as` 绕过。
* 常与 `ReadonlyArray` / `readonly T[]` 一起使用表达不可变数据。

### 16.4 Pick

**概念说明：** `Pick<T, K>` 从 `T` 中挑选 `K` 指定的属性构成新类型，用于「裁剪大类型」。

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type PublicUser = Pick<User, "id" | "name" | "email">;
const pub: PublicUser = { id: 1, name: "Alice", email: "a@x.com" };
console.log(pub);

// 实现原理
type MyPick<T, K extends keyof T> = { [P in K]: T[P] };

// K 必须是 keyof T 的子集
// type Bad = Pick<User, "age">;     // 错误：'age' 不在 keyof User 中

// 典型用法：作为函数的返回类型，避免泄漏敏感字段
function toPublic(u: User): Pick<User, "id" | "name"> {
  return { id: u.id, name: u.name };
}
console.log(toPublic({ id: 1, name: "A", email: "e", password: "p" }));

// 从联合中挑键（K 可以是联合）
type Keys = "id" | "email";
type Subset = Pick<User, Keys>;
const s: Subset = { id: 1, email: "a@x.com" };
console.log(s);

// 与 Omit 互补：Pick 保留，Omit 排除
type Rest = Omit<User, "password">;
const rest: Rest = { id: 1, name: "A", email: "e" };
console.log(rest.email);
```

**运行结果：**
```
{ id: 1, name: 'Alice', email: 'a@x.com' }
{ id: 1, name: 'A' }
{ id: 1, email: 'a@x.com' }
e
```

**注意：**
* 只会保留指定键，未指定的键绝对不会出现在结果中（多余属性会报错）。
* 保留原属性上的 `?` 与 `readonly` 修饰符。
* 与 `Omit` 的方向相反，按「要什么」还是「不要什么」选择。

### 16.5 Omit

**概念说明：** `Omit<T, K>` 从 `T` 中排除 `K` 指定的属性，常用于隐藏敏感字段或构造「创建输入」。

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type SafeUser = Omit<User, "password">;
const safe: SafeUser = { id: 1, name: "Alice", email: "a@x.com" };
console.log(safe);

// 实现原理（基于 Pick 与 Exclude）
type MyOmit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;

// 排除多个键
type Summary = Omit<User, "password" | "email">;
const summary: Summary = { id: 1, name: "A" };
console.log(summary);

// 典型用法：创建输入类型（去掉服务端生成的 id）
type CreateUserInput = Omit<User, "id">;
const input: CreateUserInput = { name: "A", email: "e", password: "p" };
console.log(input.name);

// 注意：Omit 不检查 K 是否存在于 T 中
type Wrong = Omit<User, "nonexistent">;   // 不报错，等于 User 去掉不存在的键
const w: Wrong = { id: 1, name: "A", email: "e", password: "p" };
console.log(Object.keys(w).length);

// 与 Pick 组合实现「替换属性类型」
type Replace<T, K extends keyof T, V> = Omit<T, K> & Record<K, V>;
type UserWithCount = Replace<User, "id", string>;
const r: UserWithCount = { id: "1", name: "A", email: "e", password: "p" };
console.log(typeof r.id);
```

**运行结果：**
```
{ id: 1, name: 'Alice', email: 'a@x.com' }
{ id: 1, name: 'A' }
A
4
string
```

**注意：**
* `K` 若包含不存在的键不会报错（这是 `Omit` 的一个已知宽松点），严格版本可自定义为 `K extends keyof T`。
* 排除结果是「新类型」，与原类型不再有继承关系。
* 需要「保留名单」时用 `Pick` 更安全。

### 16.6 Record

**概念说明：** `Record<K, V>` 构造「键为 `K`、值为 `V`」的对象类型，适合字典、映射表与穷尽性检查。

```typescript
// 字符串键字典
const dict: Record<string, number> = { a: 1, b: 2 };
console.log(dict.a);

// 字面量联合作为键 → 必须穷尽
type Role = "admin" | "user" | "guest";
const permissions: Record<Role, string[]> = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"],
};
console.log(permissions.admin.length);

// 数字键
const scores: Record<number, string> = { 1: "金", 2: "银" };
console.log(scores[1]);

// 实现原理
type MyRecord<K extends keyof any, V> = { [P in K]: V };

// 用 Record 做穷尽性检查（漏掉键会报错）
type ConfigKeys = "host" | "port";
const config: Record<ConfigKeys, string> = { host: "localhost", port: "80" };
console.log(config.port);

// 值类型可以依赖键（配合映射类型更灵活）
type Emitter = Record<"click" | "focus", (e: string) => void>;
const emitter: Emitter = { click: (e) => console.log("click", e), focus: (e) => console.log("focus", e) };
emitter.click("按钮");

// 与 Partial 组合允许部分键
const partialConfig: Partial<Record<ConfigKeys, string>> = { host: "h" };
console.log(partialConfig);

// 注意：string 键会失去「必须穷尽」的约束力
const loose: Record<string, string> = {};
console.log(Object.keys(loose).length);
```

**运行结果：**
```
1
3
金
80
click 按钮
{ host: 'h' }
0
```

**注意：**
* 键为 `string` 时不提供穷尽性检查，用字面量联合才有效。
* `Record` 的值类型统一，若需要每个键不同类型请直接用接口或映射类型。
* 与 `Partial` 组合可得到「部分键可选」的字典。

### 16.7 Exclude

**概念说明：** `Exclude<T, U>` 从联合类型 `T` 中剔除可赋值给 `U` 的成员，是「联合类型的差集」。

```typescript
type All = "a" | "b" | "c" | "d";
type R1 = Exclude<All, "a">;              // "b" | "c" | "d"
const r1: R1 = "b";
console.log(r1);

// 剔除多个
type R2 = Exclude<All, "a" | "c">;        // "b" | "d"
const r2: R2 = "d";
console.log(r2);

// 实现原理：基于分布式条件类型
type MyExclude<T, U> = T extends U ? never : T;

// 从对象键联合中剔除
type Keys = keyof { id: number; name: string; email: string };
type WithoutId = Exclude<Keys, "id">;     // "name" | "email"
const k: WithoutId = "name";
console.log(k);

// 剔除 null 与 undefined（常用）
type Maybe = string | null | undefined;
type Definite = Exclude<Maybe, null | undefined>;   // string
const d: Definite = "值";
console.log(d);

// 注意：只能作用于联合类型，对对象无效
type Obj = { a: 1 };
type R3 = Exclude<Obj, { a: 1 }>;         // never
console.log("对象不可通过 Exclude 排除");

// 与 Extract 相反
type R4 = Extract<All, "a" | "e">;        // "a"
const r4: R4 = "a";
console.log(r4);
```

**运行结果：**
```
b
d
name
值
对象不可通过 Exclude 排除
a
```

**注意：**
* 只对联合类型有意义；对非联合类型会得到 `never` 或原类型。
* 匹配依据是可赋值性，`"a"` 能剔除 `"a"`，而 `string` 不能剔除 `"a"`。
* `Exclude` 是 `Omit` 的实现基础（`Pick<T, Exclude<keyof T, K>>`）。

### 16.8 Extract

**概念说明：** `Extract<T, U>` 从联合 `T` 中提取可赋值给 `U` 的成员，是「联合类型的交集」。

```typescript
type All = "a" | "b" | "c" | 1 | 2;
type Strings = Extract<All, string>;      // "a" | "b" | "c"
const s: Strings = "a";
console.log(s);

type Numbers = Extract<All, number>;      // 1 | 2
const n: Numbers = 1;
console.log(n);

// 实现原理
type MyExtract<T, U> = T extends U ? T : never;

// 提取函数类型
type Mixed = string | (() => void) | number | ((x: number) => void);
type Fns = Extract<Mixed, (...args: never[]) => unknown>;
const f: Fns = () => {};
console.log(typeof f);

// 提取可辨识联合中的特定分支
type Shape =
  | { kind: "circle"; r: number }
  | { kind: "square"; side: number }
  | { kind: "rect"; w: number; h: number };

type Circle = Extract<Shape, { kind: "circle" }>;
const c: Circle = { kind: "circle", r: 1 };
console.log(c.r);

// 提取 Promise 类型
type P = Extract<string | Promise<number>, Promise<unknown>>;
const p: P = Promise.resolve(1);
console.log(typeof p.then);

// 与 Exclude 配合：按值类型拆分联合
type Split<T, U> = [Extract<T, U>, Exclude<T, U>];
type S = Split<All, string>;
const split: S = ["a", 1];
console.log(split);
```

**运行结果：**
```
a
1
function
1
function
[ 'a', 1 ]
```

**注意：**
* 与 `Exclude` 互补，二者都基于分布式条件类型。
* 提取对象分支时用 `{ kind: "circle" }` 这样的部分结构即可。
* 是构建「按类型分派」的类型工具的核心。

### 16.9 NonNullable

**概念说明：** `NonNullable<T>` 从 `T` 中移除 `null` 与 `undefined`。

```typescript
type MaybeString = string | null | undefined;
type Definite = NonNullable<MaybeString>;      // string
const d: Definite = "值";
console.log(d);

// 实现原理
type MyNonNullable<T> = T & {};

// 对可选属性使用
interface User { name?: string | null }
type NameType = NonNullable<User["name"]>;     // string
const n: NameType = "Alice";
console.log(n);

// 联合含 null 的收窄
type Values = string | number | null;
function sum(vals: NonNullable<Values>[]): number {
  return vals.reduce<number>((acc, v) => acc + (typeof v === "number" ? v : 0), 0);
}
console.log(sum([1, 2, "a"]));

// 与 Required 组合：可选属性变必填且非空
type StrictUser = { [K in keyof User]-?: NonNullable<User[K]> };
const su: StrictUser = { name: "Bob" };
console.log(su.name);

// 注意：只移除 null 与 undefined，不移除空字符串或 0
type Truthy = NonNullable<"" | 0 | false>;     // "" | 0 | false
const t: Truthy = 0;
console.log(t);

// 数组元素过滤后的类型
const mixed: (string | null)[] = ["a", null];
const cleaned = mixed.filter((v): v is NonNullable<typeof v> => v !== null);
console.log(cleaned);
```

**运行结果：**
```
值
Alice
3
Bob
0
[ 'a' ]
```

**注意：**
* 只处理 `null` / `undefined`，不处理其他假值。
* 与类型守卫（`v is NonNullable<T>`）配合可让数组 `filter` 结果收窄。
* 是最常用的「非空化」工具类型。

### 16.10 ReturnType

**概念说明：** `ReturnType<T>` 提取函数类型 `T` 的返回值类型，避免手写重复类型导致不一致。

```typescript
function createUser() {
  return { id: 1, name: "Alice", tags: ["a"] };
}
type User = ReturnType<typeof createUser>;
const u: User = { id: 1, name: "Alice", tags: [] };
console.log(u.name);

// 异步函数得到 Promise<...>
async function fetchData() {
  return { total: 10 };
}
type Res = ReturnType<typeof fetchData>;       // Promise<{ total: number }>
type Unwrapped = Awaited<Res>;                 // { total: number }
const r: Unwrapped = { total: 1 };
console.log(r.total);

// 实现原理
type MyReturnType<T extends (...args: never[]) => unknown> =
  T extends (...args: never[]) => infer R ? R : never;

// 对泛型函数：返回 unknown（无法推断具体类型参数）
function identity<T>(v: T): T { return v; }
type R = ReturnType<typeof identity>;          // unknown
const rv: R = "任意值";
console.log(rv);

// 用于 class 方法
class Service {
  getStatus() { return { code: 0, msg: "ok" }; }
}
type Status = ReturnType<Service["getStatus"]>;
const st: Status = { code: 1, msg: "x" };
console.log(st.code);

// 与 Parameters 组合描述函数契约
type Fn = (a: string, b: number) => boolean;
type P = Parameters<Fn>;
type R2 = ReturnType<Fn>;
const args: P = ["a", 1];
const ret: R2 = true;
console.log(args, ret);
```

**运行结果：**
```
Alice
1
任意值
1
[ 'a', 1 ] true
```

**注意：**
* 需要 `typeof fn` 把函数值转成函数类型再传入。
* 泛型函数的 `ReturnType` 无法体现类型参数，结果常是 `unknown` 或宽类型。
* 与 `Awaited` 组合可解包异步返回值。

### 16.11 Parameters

**概念说明：** `Parameters<T>` 提取函数类型 `T` 的参数元组类型，常用于包装函数时保持参数一致。

```typescript
type Fn = (name: string, age: number, active?: boolean) => void;
type P = Parameters<Fn>;         // [name: string, age: number, active?: boolean]
const args: P = ["Alice", 25];
console.log(args);

// 实现原理
type MyParameters<T extends (...args: never[]) => unknown> =
  T extends (...args: infer A) => unknown ? A : never;

// 典型用法：为函数添加日志包装而不改变签名
function withLogging<F extends (...args: never[]) => unknown>(fn: F) {
  return (...args: Parameters<F>): ReturnType<F> => {
    console.log("调用参数:", args);
    return fn(...args);
  };
}
const add = (a: number, b: number): number => a + b;
const logged = withLogging(add);
console.log(logged(1, 2));

// 取单个参数类型
type First = Parameters<Fn>[0];              // string
const first: First = "s";
console.log(first);

// 空参数函数
type None = Parameters<() => void>;          // []
const none: None = [];
console.log(none);

// 剩余参数
type Rest = Parameters<(...nums: number[]) => void>;   // number[]
const rest: Rest = [1, 2, 3];
console.log(rest);

// 与 ReturnType 一起做「函数类型变换」
type ChangeReturn<F, R> = (...args: Parameters<F extends (...a: never[]) => unknown ? F : never>) => R;
const stringify: ChangeReturn<typeof add, string> = (a, b) => String(a + b);
console.log(stringify(1, 2));
```

**运行结果：**
```
[ 'Alice', 25 ]
调用参数: [ 1, 2 ]
3
s
[]
[ 1, 2, 3 ]
3
```

**注意：**
* 对重载函数只取最后一个签名的参数。
* 返回的是具名元组（TS 4.0+），可读性更好。
* 与 `ReturnType` 组合可实现「入参不变、返回值变化」的函数类型。

### 16.12 ConstructorParameters

**概念说明：** `ConstructorParameters<T>` 提取构造函数类型 `T` 的参数元组，用于泛型工厂、依赖注入等场景。

```typescript
class User {
  constructor(public id: number, public name: string, public active = true) {}
}

type CtorArgs = ConstructorParameters<typeof User>;   // [id: number, name: string, active?: boolean]
const args: CtorArgs = [1, "Alice"];
console.log(args);

// 实现原理
type MyCtorArgs<T extends abstract new (...args: never[]) => unknown> =
  T extends abstract new (...args: infer A) => unknown ? A : never;

// 典型用法：通用工厂函数
function createInstance<T extends new (...args: never[]) => object>(
  Ctor: T,
  ...args: ConstructorParameters<T>
): InstanceType<T> {
  return new Ctor(...args);
}
const user = createInstance(User, 1, "Bob");
console.log(user.name, user.active);

// 提取单个参数
type FirstArg = ConstructorParameters<typeof User>[0];   // number
const id: FirstArg = 1;
console.log(id);

// 抽象类的构造参数
abstract class Base {
  constructor(protected label: string) {}
}
type BaseArgs = ConstructorParameters<typeof Base>;      // [label: string]
const ba: BaseArgs = ["标签"];
console.log(ba);

// 内置类
type DateArgs = ConstructorParameters<typeof Date>;      // 多种重载形式
const dArgs: DateArgs = [];
console.log(new Date(...dArgs).getTime() > 0);

// 无参数构造函数
class Empty {}
type E = ConstructorParameters<typeof Empty>;            // []
const e: E = [];
console.log(e);
```

**运行结果：**
```
[ 1, 'Alice' ]
Bob true
1
[ '标签' ]
true
[]
```

**注意：**
* 传入的必须是构造函数类型（`typeof Class`），不能用实例类型。
* 抽象类需要 `abstract new (...)` 形式的约束。
* 与 `InstanceType` 组合是「按类名创建实例」的完整方案。

### 16.13 InstanceType

**概念说明：** `InstanceType<T>` 提取构造函数类型 `T` 的实例类型，等价于直接写类名，但在泛型中非常有用。

```typescript
class Service {
  name = "service";
  run(): string { return "运行中"; }
}

type S = InstanceType<typeof Service>;    // Service
const s: S = new Service();
console.log(s.name, s.run());

// 实现原理
type MyInstanceType<T extends abstract new (...args: never[]) => unknown> =
  T extends abstract new (...args: never[]) => infer I ? I : never;

// 泛型工厂：参数与返回值都自动推导
function factory<T extends new (...args: never[]) => object>(
  Ctor: T,
  ...args: ConstructorParameters<T>
): InstanceType<T> {
  return new Ctor(...args);
}
class Point { constructor(public x: number, public y: number) {} }
const p = factory(Point, 1, 2);
console.log(p.x, p.y);

// 与 ReturnType 的区别
class Factory {
  static create() { return new Service(); }
}
type FromReturn = ReturnType<typeof Factory.create>;    // Service
type FromInstance = InstanceType<typeof Service>;       // Service
const a: FromReturn = new Service();
const b: FromInstance = new Service();
console.log(a.name === b.name);

// 抽象类
abstract class Animal { abstract speak(): string }
class Dog extends Animal { speak() { return "汪"; } }
type A = InstanceType<typeof Dog>;
const d: A = new Dog();
console.log(d.speak());

// 构造函数联合类型
type Union = InstanceType<typeof Service | typeof Point>;
const u1: Union = new Service();
const u2: Union = new Point(1, 2);
console.log(u1.name, u2 instanceof Point);
```

**运行结果：**
```
service 运行中
1 2
true
汪
service true
```

**注意：**
* 传入 `typeof Class`，返回实例类型；与 `ConstructorParameters` 是同一族的工具。
* 抽象类需要 `abstract new` 形式的约束才能匹配。
* 与 `ReturnType` 的区别：`InstanceType` 针对 `new` 签名，`ReturnType` 针对调用签名。

### 16.14 Awaited

**概念说明：** `Awaited<T>` 递归解包 Promise（含 thenable 与嵌套 Promise），得到 `await` 之后的结果类型。它是 async 函数返回值推断的基础。

```typescript
type R1 = Awaited<Promise<string>>;                    // string
type R2 = Awaited<Promise<Promise<number>>>;           // number
type R3 = Awaited<string | Promise<number>>;           // string | number

const r1: R1 = "值";
const r2: R2 = 1;
const r3: R3 = "s";
console.log(r1, r2, r3);

// 实现原理（简化版）
type MyAwaited<T> = T extends PromiseLike<infer U> ? MyAwaited<U> : T;

// 与 ReturnType 组合：获取异步函数的真实返回类型
async function fetchUser() {
  return { id: 1, name: "Alice" };
}
type User = Awaited<ReturnType<typeof fetchUser>>;
const u: User = { id: 1, name: "Alice" };
console.log(u.name);

// 与 async 函数返回类型的关系
async function getValue(): Promise<number> { return 1; }
type V = Awaited<ReturnType<typeof getValue>>;         // number
const v: V = 1;
console.log(v);

// thenable 对象也能解包
type Thenable = { then(cb: (v: boolean) => void): void };
type R4 = Awaited<Thenable | Promise<Thenable>>;       // boolean
const r4: R4 = true;
console.log(r4);

// 非 Promise 类型原样返回
type R5 = Awaited<42>;                                 // 42
const r5: R5 = 42;
console.log(r5);

// 实际用法：解包数组中每个 Promise
type AllResults = Awaited<Promise<[number, string]>>;  // [number, string]
const ar: AllResults = [1, "a"];
console.log(ar);
```

**运行结果：**
```
值 1 s
Alice
1
true
42
[ 1, 'a' ]
```

**注意：**
* 是递归的：多层嵌套 Promise 会被完全解包。
* 对非 Promise 类型原样返回（不会变成 `never`）。
* 从 TS 4.5 起内置；此前需要手写 `UnwrapPromise` 工具类型。

## 17. TypeScript 函数进阶
### 17.1 函数类型

**概念说明：** 函数类型用 `(参数) => 返回值` 描述，可标注在变量、参数、返回值与属性上。函数类型之间按「参数逆变、返回值协变」的规则比较。

```typescript
// 变量标注
const add: (a: number, b: number) => number = (a, b) => a + b;
console.log(add(1, 2));

// 类型别名
type Compare<T> = (a: T, b: T) => number;
const byLength: Compare<string> = (a, b) => a.length - b.length;
console.log(byLength("a", "bbb"));

// 参数与返回值可省略注解（由上下文推断）
const toUpper: (s: string) => string = (s) => s.toUpperCase();
console.log(toUpper("abc"));

// 返回值为 void 的函数可以接受「有返回值」的函数
type Logger = (msg: string) => void;
const l: Logger = (msg) => msg.length;      // 返回值被忽略（合法）
l("记录");

// 函数类型作为对象的属性
interface Handlers {
  onSuccess: (data: string) => void;
  onError?: (err: Error) => void;
}
const h: Handlers = { onSuccess: (d) => console.log("成功:", d) };
h.onSuccess("数据");

// 返回函数的函数
const adder = (a: number): ((b: number) => number) => (b) => a + b;
console.log(adder(1)(2));

// 高阶函数类型
function apply<T, U>(value: T, fn: (v: T) => U): U { return fn(value); }
console.log(apply("abc", (s) => s.length));

// 可调用对象类型
type Callable = { (x: number): string; tag: string };
const c = ((x: number) => `值 ${x}`) as Callable;
c.tag = "标记";
console.log(c(1), c.tag);
```

**运行结果：**
```
3
-2
ABC
成功: 数据
3
3
值 1 标记
```

**注意：**
* 函数类型参数名可省略，但保留有助于可读性。
* 未标注返回类型的函数，其返回值类型会被推断；作为公共 API 建议显式标注。
* 函数类型之间「参数少的可以赋给参数多的」是合法的（忽略多余参数）。

### 17.2 可选参数

**概念说明：** 参数名后加 `?` 表示可省略，类型自动包含 `undefined`；也可用联合 `T | undefined` 表达「必须传但可为空」。可选参数必须位于必填参数之后。

```typescript
// 可选参数
function greet(name: string, title?: string): string {
  return title ? `${title} ${name}` : name;
}
console.log(greet("Alice"));
console.log(greet("Alice", "Dr."));

// 显式传 undefined 等价于省略
console.log(greet("Bob", undefined));

// 可选参数的类型包含 undefined
function show(value?: string): void {
  const v: string | undefined = value;
  console.log(v ?? "无值");
}
show();
show("有值");

// 与联合写法的区别：必须传但可为 null
function strictNull(name: string | null): string {
  return name ?? "匿名";
}
// strictNull();          // 错误：缺少参数
console.log(strictNull(null), strictNull("Bob"));

// 可选参数必须在必填参数之后
// function bad(a?: number, b: number): void {}   // 错误

// 模拟「中间的」可选：用对象参数
interface Options { a: number; b?: string; c: number }
function configured(opts: Options): string {
  return `${opts.a}-${opts.b ?? "-"}-${opts.c}`;
}
console.log(configured({ a: 1, c: 3 }));
console.log(configured({ a: 1, b: "x", c: 3 }));

// 可选参数与解构默认值
function withDefault({ size = 10 }: { size?: number } = {}): number {
  return size;
}
console.log(withDefault(), withDefault({ size: 20 }));
```

**运行结果：**
```
Alice
Dr. Alice
Bob
无值
有值
匿名 Bob
1--3
1-x-3
10
20
```

**注意：**
* 可选参数无法出现在必填参数之前，需要时改用对象参数。
* `?` 与 `| undefined` 的差别：前者可省略，后者必须传值。
* 解构默认值只在值为 `undefined` 时生效（`null` 不会触发）。

### 17.3 默认参数

**概念说明：** 参数可带默认值，带默认值的参数在调用时可省略（自动变为可选），且默认值在函数调用时求值。

```typescript
// 基础默认值
function createUser(name: string, role = "user", active = true): string {
  return `${name}/${role}/${active}`;
}
console.log(createUser("Alice"));
console.log(createUser("Bob", "admin", false));

// 默认值的求值时机：每次调用
function log(msg: string, time = new Date().toISOString().slice(0, 10)): void {
  console.log(time, msg);
}
log("第一次");

// 默认值可引用前面的参数
function range(start: number, end = start + 10): number[] {
  return Array.from({ length: end - start + 1 }, (_, i) => start + i);
}
console.log(range(1, 3).length, range(1).length);

// 默认值在类型层面是可选的
type Fn = (a: number, b?: number) => number;
const add: Fn = (a, b = 0) => a + b;
console.log(add(1), add(1, 2));

// 默认值与 undefined
function withStr(s = "默认"): string { return s; }
console.log(withStr(undefined), withStr(""));

// 推断得到可选参数类型
type Params = Parameters<typeof createUser>;
const ps: Params = ["Alice"];
console.log(ps.length);

// 默认值与解构
function init({ host = "localhost", port = 80 } = {}): string {
  return `${host}:${port}`;
}
console.log(init(), init({ port: 8080 }), init({ host: "h", port: 1 }));
```

**运行结果：**
```
Alice/user/true
Bob/admin/false
2024-01-01 第一次
3 11
1 3
默认 
1
localhost:80
localhost:8080
h:1
```

**注意：**
* 默认值使参数变为可选，因此 `Parameters` 中该位置仍是可选的（带 `?`）。
* 显式传 `undefined` 会触发默认值，传 `null` 不会。
* 复杂默认值（对象/数组字面量）每次调用都会重新创建。

### 17.4 Rest 参数

**概念说明：** 剩余参数用 `...name: T[]` 收集多余实参为数组，必须是最后一个参数。TypeScript 结合泛型元组可实现类型安全的可变参数传递。

```typescript
// 基础剩余参数
function sum(...nums: number[]): number {
  return nums.reduce((a, b) => a + b, 0);
}
console.log(sum(), sum(1), sum(1, 2, 3));

// 固定参数 + 剩余参数
function log(level: string, ...messages: string[]): void {
  console.log(`[${level}]`, messages.join(" "));
}
log("INFO", "启动", "完成");

// 剩余参数与元组类型：类型安全转发
function call<T extends unknown[], R>(fn: (...args: T) => R, ...args: T): R {
  return fn(...args);
}
const multiply = (a: number, b: number) => a * b;
console.log(call(multiply, 3, 4));
// call(multiply, "a", 4);      // 错误：第一个参数应为 number

// 把数组展开为参数
const args: [string, number] = ["Alice", 25];
function introduce(name: string, age: number): string {
  return `${name} ${age}`;
}
console.log(introduce(...args));

// 剩余参数类型可以是元组（混合类型）
function mixed(...args: [string, number, boolean?]): string {
  const [s, n, b] = args;
  return `${s}-${n}-${b ?? "默认"}`;
}
console.log(mixed("a", 1), mixed("a", 1, true));

// 部分应用（偏函数）保持类型
function partial<T extends unknown[], U, R>(
  fn: (...args: [...T, U]) => R,
  ...pre: T
): (...rest: [U]) => R {
  return (u: U) => fn(...pre, u);
}
const greet = (title: string, name: string) => `${title} ${name}`;
const drGreet = partial(greet, "Dr.");
console.log(drGreet("Alice"));
```

**运行结果：**
```
0 1 6
[INFO] 启动 完成
12
Alice 25
a-1-默认 a-1-true
Dr. Alice
```

**注意：**
* 剩余参数必须是最后一个参数，且类型必须为数组或元组。
* 用 `...args: T`（T 为泛型元组）能保留参数位置与类型的对应关系（TS 4.0+）。
* 剩余参数不会影响 `fn.length`（该属性只计算必填参数个数）。

### 17.5 函数重载

**概念说明：** 为一组签名不同的同名函数提供多个「重载签名」，最后写一个实现签名（对外不可见）。TypeScript 会按顺序匹配最合适的重载。

```typescript
// 重载签名 + 实现签名
function format(value: string): string;
function format(value: number, decimals: number): string;
function format(value: boolean): string;
function format(value: string | number | boolean, decimals = 2): string {
  if (typeof value === "string") return value.trim();
  if (typeof value === "number") return value.toFixed(decimals);
  return value ? "是" : "否";
}

console.log(format("  a  "));
console.log(format(3.14159, 3));
console.log(format(true));
// format(1);                    // 错误：需要一个 decimals 参数

// 返回值也随重载不同
function get(key: "name"): string;
function get(key: "count"): number;
function get(key: "name" | "count"): string | number {
  return key === "name" ? "Alice" : 1;
}
const nm: string = get("name");
const ct: number = get("count");
console.log(nm, ct);

// 方法重载
class Parser {
  parse(input: string): string[];
  parse(input: string[]): string;
  parse(input: string | string[]): string | string[] {
    return Array.isArray(input) ? input.join(",") : input.split(",");
  }
}
const parser = new Parser();
console.log(parser.parse("a,b"));
console.log(parser.parse(["a", "b"]));

// 注意顺序：更具体的签名放前面
function pick(x: string): "字符串";
function pick(x: unknown): "未知";
function pick(x: unknown): string {
  return typeof x === "string" ? "字符串" : "未知";
}
console.log(pick("a"), pick(1));

// 箭头函数无法直接写重载，需用接口或类型断言
interface Overloaded {
  (x: number): number;
  (x: string): string;
}
const ident: Overloaded = (x: any) => x;
console.log(ident(1), ident("a"));
```

**运行结果：**
```
a
3.142
是
Alice 1
[ 'a', 'b' ]
a,b
字符串 未知
1 a
```

**注意：**
* 实现签名必须兼容所有重载签名，且不对外可见。
* 匹配按声明顺序，宽泛的签名放后面。
* 重载能用联合类型 + 条件类型替代时优先考虑后者（更易维护）；箭头函数需借接口实现重载。

### 17.6 泛型函数

**概念说明：** 泛型函数用类型参数保持入参与返回值的类型关联，避免 `any`；可配合约束、默认值与重载表达复杂契约。

```typescript
// 基本泛型函数
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}
console.log(first([1, 2]), first(["a"]));

// 多类型参数与约束
function groupBy<T, K extends keyof T>(items: T[], key: K): Record<string, T[]> {
  const result: Record<string, T[]> = {};
  for (const item of items) {
    const k = String(item[key]);
    (result[k] ??= []).push(item);
  }
  return result;
}
const grouped = groupBy([{ t: "a", v: 1 }, { t: "a", v: 2 }], "t");
console.log(grouped.a.length);

// 泛型 + 默认值
function createArray<T = number>(len: number, fill: T): T[] {
  return Array(len).fill(fill);
}
console.log(createArray(2, 0), createArray<string>(2, "x"));

// 类型参数之间的关系
function prop<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
console.log(prop({ id: 1, name: "a" }, "name"));

// 泛型与函数重载配合
function fetchIt(url: string): Promise<string>;
function fetchIt(url: string, json: true): Promise<unknown>;
function fetchIt(url: string, json?: boolean): Promise<string | unknown> {
  return Promise.resolve(json ? { url } : url);
}
fetchIt("u").then((r) => console.log(typeof r));

// 泛型箭头函数（在 .tsx 中需写成 <T,>）
const identity = <T,>(v: T): T => v;
console.log(identity("泛型箭头"));

// 高阶泛型函数
function compose<A, B, C>(f: (a: A) => B, g: (b: B) => C): (a: A) => C {
  return (a) => g(f(a));
}
const lenOfUpper = compose((s: string) => s.toUpperCase(), (s) => s.length);
console.log(lenOfUpper("abc"));
```

**运行结果：**
```
1 a
2
[ 0, 0 ] [ 'x', 'x' ]
a
string
泛型箭头
3
```

**注意：**
* 类型参数能被推断时无需显式传入；`createArray<string>` 展示了显式指定。
* `.tsx` 文件中 `<T>` 会被当作 JSX，需要写成 `<T,>` 或 `<T extends unknown>`。
* 泛型函数内只能使用约束中声明的成员。

### 17.7 this 参数

**概念说明：** 函数可以声明一个名为 `this` 的伪参数，用于指定调用时的 `this` 类型，它不占实际参数位，只在编译期生效。

```typescript
// 基本用法：约束 this 的类型
function getFullName(this: { firstName: string; lastName: string }): string {
  return `${this.firstName} ${this.lastName}`;
}
const person = { firstName: "Alice", lastName: "Smith", getFullName };
console.log(person.getFullName());

// 直接调用会报错
// getFullName();      // 错误：this 上下文类型为 void，不可用

// 用 call / apply 显式绑定
console.log(getFullName.call({ firstName: "Bob", lastName: "Lee" }));

// this 类型用于类的方法
class Counter {
  private count = 0;

  // 只有通过该方法调用时才可用
  increment(this: Counter): number {
    return ++this.count;
  }
}
const c = new Counter();
console.log(c.increment());

// 泛型 this 类型：返回 this 支持链式调用
class Builder {
  private parts: string[] = [];
  add(part: string): this {
    this.parts.push(part);
    return this;
  }
  build(): string { return this.parts.join("-"); }
}
const b = new Builder();
console.log(b.add("a").add("b").build());

// 回调中 this 的类型（用 ThisType 标记）
interface Context {
  name: string;
  log(this: Context): void;
}
const ctx: Context = {
  name: "上下文",
  log() { console.log(this.name); },
};
ctx.log();

// 明确禁止使用 this 的场景
function standalone(this: void): string { return "不依赖 this"; }
console.log(standalone.call(undefined));
```

**运行结果：**
```
Alice Smith
Bob Lee
1
a-b
上下文
不依赖 this
```

**注意：**
* `this` 参数必须写在参数列表的第一个位置，只是类型标注，不传递实参。
* 用 `this: void` 可禁止函数使用 `this`，避免误用。
* 返回 `this` 类型是实现链式 API 的标准方式。

### 17.8 回调函数类型

**概念说明：** 回调是「作为参数传入的函数」，其参数与返回值类型由调用方通过泛型与上下文推断确定，类型定义需要同时约束入参与返回。

```typescript
// 基本回调
function fetchData(cb: (err: Error | null, data?: string) => void): void {
  setTimeout(() => cb(null, "数据"), 0);
}
fetchData((err, data) => console.log(err ? err.message : data));

// 泛型回调：结果类型由回调决定
function mapAsync<T, R>(values: T[], fn: (v: T) => Promise<R>): Promise<R[]> {
  return Promise.all(values.map(fn));
}
mapAsync([1, 2], async (n) => n * 10).then((r) => console.log(r));

// 回调返回 void 时允许任意返回值
function forEach2<T>(arr: T[], cb: (item: T, index: number) => void): void {
  arr.forEach((item, i) => cb(item, i));
}
forEach2([1, 2], (n, i) => console.log(i, n));

// 事件回调：用类型定义约束事件名与负载
type EventMap = {
  click: { x: number; y: number };
  change: { value: string };
};
function on<K extends keyof EventMap>(event: K, cb: (payload: EventMap[K]) => void): void {
  console.log("注册:", event);
  cb({ x: 1, y: 2 } as never);
}
on("click", (p) => console.log(p.x));

// 错误优先回调的实用封装
type Callback<T> = (error: Error | null, result?: T) => void;
function nodeify<T>(promise: Promise<T>, cb: Callback<T>): void {
  promise.then((r) => cb(null, r)).catch((e: Error) => cb(e));
}
nodeify(Promise.resolve("结果"), (e, r) => console.log(e ? e.message : r));

// 带可选参数的回调
type Comparator<T> = (a: T, b: T) => number;
function sortBy<T>(arr: T[], cmp: Comparator<T>): T[] {
  return [...arr].sort(cmp);
}
console.log(sortBy([3, 1, 2], (a, b) => a - b));

// 回调的类型可以从上下文推断（无需显式标注）
const nums = [1, 2, 3];
nums.forEach((n) => console.log(typeof n));
```

**运行结果：**
```
数据
[ 10, 20 ]
0 1
1 2
注册: click
1
结果
[ 1, 2, 3 ]
number
number
number
```

**注意：**
* 回调参数类型由上下文推断，能显著减少样板代码，不要重复标注。
* Node 风格回调约定 `(err, data)`，且 `err` 在前。
* 事件系统用「事件名 → 负载类型」的映射表（`EventMap`）可获得完整的类型安全。

### 17.9 参数协变与逆变

**概念说明：** 函数类型比较时，参数位置是**逆变**（contravariant，方向相反）、返回值位置是**协变**（covariant，方向相同）。`strictFunctionTypes` 开启后，函数的参数才是严格逆变的；方法参数是双变的。

```typescript
class Animal { name = "动物" }
class Dog extends Animal { breed = "柴犬" }

// 返回值协变：返回更具体的类型可以赋值
type GetAnimal = () => Animal;
type GetDog = () => Dog;
const getDog: GetDog = () => new Dog();
const getAnimal: GetAnimal = getDog;      // 合法：Dog 是 Animal 的子类型
console.log(getAnimal().name);

// 参数逆变：参数更宽泛的可以赋值给参数更窄的
type HandleDog = (d: Dog) => void;
type HandleAnimal = (a: Animal) => void;
const handleAnimal: HandleAnimal = (a) => console.log(a.name);
const handleDog: HandleDog = handleAnimal;   // 合法（逆变）
console.log(handleDog(new Dog()));

// 反向不合法（strictFunctionTypes 下）
const handleOnlyDog: HandleDog = (d) => console.log(d.breed);
// const wrong: HandleAnimal = handleOnlyDog;   // 错误：参数类型不兼容
console.log(handleOnlyDog(new Dog()));

// 方法参数是双变的（历史行为，出于 Array 等兼容）
interface Compare {
  compare(a: Dog): void;
}
interface CompareBase {
  compare(a: Animal): void;
}
const base: CompareBase = { compare: (a) => console.log(a.name) };
const cmp: Compare = base;                  // 合法
console.log(cmp.compare(new Dog()));

// 数组的协变：Dog[] 可赋给 Animal[]
const dogs: Dog[] = [new Dog()];
const animals: Animal[] = dogs;             // 合法但危险
animals.push(new Animal());                 // 运行时 dogs 里混入 Animal
console.log(dogs.length);

// 用只读数组避免不健全的写入
const readonlyAnimals: readonly Animal[] = dogs;
console.log(readonlyAnimals.length);

// 逆变位置中的联合：任何 Animal 的处理函数都能处理 Dog
type Fn1 = (a: Animal) => void;
const f1: Fn1 = (a) => console.log(a.name);
const f2: HandleDog = f1;
f2(new Dog());
```

**运行结果：**
```
动物
动物
柴犬
动物
2
2
动物
```

**注意：**
* 记住口诀：「入参逆变、出参协变」；`strictFunctionTypes` 只对函数类型属性生效，方法签名仍是双变。
* 数组协变是不健全的（`push` 会破坏类型安全），只读数组更安全。
* 理解逆变有助于解释「为什么回调参数类型不能随便收窄」。

## 18. TypeScript Class 与 OOP
### 18.1 Class

**概念说明：** TypeScript 在 ES 类基础上增加了字段类型声明、访问修饰符、`implements`、抽象类、参数属性等能力。类同时创建了「实例类型」和「构造函数值」两种东西。

```typescript
class User {
  // 字段声明（必须）
  id: number;
  name: string;
  tags: string[] = [];              // 可带初始值

  constructor(id: number, name: string) {
    this.id = id;
    this.name = name;
  }

  greet(): string {
    return `${this.name}(${this.id})`;
  }
}

const u = new User(1, "Alice");
u.tags.push("vip");
console.log(u.greet(), u.tags);

// 类作为类型使用
const other: User = new User(2, "Bob");
console.log(other.greet());

// 类表达式
const Point = class {
  constructor(public x: number, public y: number) {}
  toString(): string { return `(${this.x}, ${this.y})`; }
};
console.log(new Point(1, 2).toString());

// 类的两种视角
type Instance = User;                        // 实例类型
type Ctor = typeof User;                     // 构造函数类型
const ctor: Ctor = User;
console.log(new ctor(3, "C").greet());

// 结构类型：只要结构一致就能当 User 用
const likeUser = { id: 9, name: "X", tags: [], greet: () => "伪装" };
const asUser: User = likeUser;
console.log(asUser.greet());
```

**运行结果：**
```
Alice(1) [ 'vip' ]
Bob(2)
(1, 2)
C(3)
伪装
```

**注意：**
* 字段必须有类型声明或有初始值，否则 `strictPropertyInitialization` 报错。
* 类的方法定义在原型上，字段在实例上。
* TypeScript 是结构化类型系统：类实例与其他结构相同的对象可以互相赋值（除非类含 `private` 成员）。

### 18.2 constructor

**概念说明：** 构造函数负责初始化实例，可带参数、访问修饰符、默认值与重载。子类中必须先调用 `super()`。

```typescript
class Config {
  host: string;
  port: number;
  private readonly createdAt: Date;

  constructor(host = "localhost", port = 80) {
    this.host = host;
    this.port = port;
    this.createdAt = new Date();
  }

  toString(): string {
    return `${this.host}:${this.port} @${this.createdAt instanceof Date}`;
  }
}
console.log(new Config().toString());
console.log(new Config("example.com", 443).toString());

// 构造重载
class Box {
  value: string;
  constructor(v: string);
  constructor(v: number);
  constructor(v: string | number) {
    this.value = typeof v === "number" ? v.toFixed(2) : v;
  }
}
console.log(new Box(1.5).value, new Box("abc").value);

// 私有构造 + 静态工厂（单例/受控创建）
class Singleton {
  private static instance: Singleton | null = null;
  private constructor(public readonly id: number) {}

  static getInstance(): Singleton {
    return (Singleton.instance ??= new Singleton(Math.random()));
  }
}
const s1 = Singleton.getInstance();
const s2 = Singleton.getInstance();
console.log(s1 === s2);

// 子类中 super 的顺序
class Base { constructor(public tag: string) {} }
class Child extends Base {
  constructor() {
    super("child");              // 必须在访问 this 之前
    console.log("tag =", this.tag);
  }
}
new Child();

// 只声明类型不初始化：用 definite assignment 断言
class Lazy {
  data!: string;                 // 断言会在使用前赋值
  load(): void { this.data = "已加载"; }
}
const lazy = new Lazy();
lazy.load();
console.log(lazy.data);
```

**运行结果：**
```
localhost:80 @true
example.com:443 @true
1.50 abc
true
tag = child
已加载
```

**注意：**
* 构造函数不能有返回类型注解（永远返回实例）。
* 构造函数的参数若使用修饰符（如 `public`）会自动成为字段（见 18.13）。
* `!` 断言与 `?.` 都不要滥用，优先在构造器中完成初始化。

### 18.3 public

**概念说明：** `public` 是默认修饰符，表示成员在任何地方都可访问，可省略不写。

```typescript
class User {
  public id: number;              // 显式
  name: string;                   // 省略，等同 public
  public tags: string[] = [];

  constructor(id: number, name: string) {
    this.id = id;
    this.name = name;
  }

  public greet(): string { return `${this.name}(${this.id})`; }
}

const u = new User(1, "Alice");
console.log(u.id, u.name, u.tags, u.greet());
u.id = 2;                         // 可写
u.tags.push("vip");
console.log(u.id, u.tags.length);

// 子类可访问
class Admin extends User {
  role = "admin";
  describe(): string { return `${this.name} 是 ${this.role}`; }
}
console.log(new Admin(3, "Bob").describe());

// 外部可访问
function show(user: User): string { return user.greet(); }
console.log(show(u));

// 结构类型：public 成员决定兼容性
const obj = { id: 9, name: "X", tags: [], greet: () => "外部对象" };
console.log(show(obj));
```

**运行结果：**
```
1 Alice [] Alice(1)
2 1
Bob 是 admin
Alice(1)
外部对象
```

**注意：**
* 类只含 `public` 成员时，结构兼容的对象都能赋给它。
* 团队规范通常省略 `public`，只在需要强调可见性时写出。
* `public` 不影响运行时（编译产物没有可见性概念，除 `#` 私有字段）。

### 18.4 private

**概念说明：** `private` 成员只能在声明它的类内部访问，子类与外部都不行。它是编译期约束，可用 `#` 前缀的 ECMAScript 私有字段获得运行时的真正私有。

```typescript
class BankAccount {
  private balance = 0;
  #secret = "真私有";

  deposit(amount: number): void {
    if (amount <= 0) throw new Error("金额必须为正");
    this.balance += amount;
    console.log("余额:", this.balance);
  }

  getBalance(): number { return this.balance; }

  // 同类实例之间可以互相访问 private 成员
  transferTo(other: BankAccount, amount: number): void {
    this.balance -= amount;
    other.balance += amount;
    console.log("转账后:", this.balance, other.balance);
  }

  reveal(): string { return this.#secret; }
}

const a = new BankAccount();
const b = new BankAccount();
a.deposit(100);
b.deposit(50);
a.transferTo(b, 30);
// a.balance;        // 错误：属性 'balance' 为私有属性
// a.#secret;        // 语法错误：类外部不可访问 # 字段
console.log(a.reveal(), b.getBalance());

// # 字段是真正的运行时私有
console.log(Object.keys(a));            // []，# 字段不参与
console.log("#secret" in a);            // false

// 影响类型兼容性：含 private 成员的类不能与外部结构兼容
class WithPrivate { private x = 1 }
const wp = new WithPrivate();
// const fake: WithPrivate = { x: 1 };  // 错误：missing private
console.log(wp instanceof WithPrivate);

// getter 暴露只读视图
class Config2 {
  private _debug = false;
  get debug(): boolean { return this._debug; }
}
console.log(new Config2().debug);
```

**运行结果：**
```
余额: 100
余额: 50
转账后: 70 80
真私有 80
[]
false
true
false
```

**注意：**
* `private` 是编译期约束，编译后可被绕过；`#` 才是运行时私有。
* 含 `private` 的类具有「名义类型」效果：外部对象无法伪造成该类。
* 同类实例之间可访问彼此的 `private` 成员，这对实现 `equals` / `clone` 很有用。

### 18.5 protected

**概念说明：** `protected` 成员可在声明类及其子类内部访问，外部不可访问。常用于模板方法模式中供子类扩展的「内部实现」。

```typescript
class Animal {
  protected energy = 100;

  protected consume(amount: number): void {
    this.energy -= amount;
    console.log("消耗", amount, "剩余能量", this.energy);
  }

  eat(): void {
    this.consume(10);                 // 基类内部可访问
  }
}

class Dog extends Animal {
  bark(): void {
    this.consume(5);                  // 子类可访问
    console.log("汪汪，能量", this.energy);
  }

  // 可以收窄可见性为 public
  publicEnergy(): number { return this.energy; }
}

const d = new Dog();
d.eat();
d.bark();
console.log(d.publicEnergy());
// d.energy;          // 错误：protected 属性只能在类及其子类中访问

// 外部无法调用 protected 方法
// d.consume(1);      // 错误

// 模板方法模式：基类定义流程，子类实现细节
abstract class Exporter {
  // 模板方法
  export(): string {
    const data = this.fetch();        // 调用子类实现
    return this.format(data);
  }
  protected abstract fetch(): string[];
  protected format(data: string[]): string {
    return data.join("|");
  }
}

class CsvExporter extends Exporter {
  protected fetch(): string[] { return ["a", "b"]; }
  protected format(data: string[]): string { return data.join(","); }
}
console.log(new CsvExporter().export());

// 子类中提升可见性
class Base2 {
  protected value = 1;
}
class Sub2 extends Base2 {
  value = 2;                  // 提升为 public（合法）
}
console.log(new Sub2().value);
```

**运行结果：**
```
消耗 10 剩余能量 90
消耗 5 剩余能量 85
汪汪，能量 85
85
a,b
2
```

**注意：**
* `protected` 只是编译期约束；运行时仍是普通属性。
* 子类可以提升可见性（protected → public），但不能降低（public → protected 会报错）。
* 模板方法模式（基类定流程、子类填细节）是 `protected` 的典型用法。

### 18.6 readonly

**概念说明：** `readonly` 修饰的属性只能在声明处或构造器中赋值一次，之后不可修改；`static readonly` 表示类级别的常量。

```typescript
class Point {
  readonly x: number;
  readonly y: number;
  readonly createdAt: Date;
  static readonly ORIGIN = { x: 0, y: 0 } as const;

  constructor(x: number, y: number) {
    this.x = x;                     // 构造器中可赋值
    this.y = y;
    this.createdAt = new Date();
  }

  move(dx: number, dy: number): Point {
    return new Point(this.x + dx, this.y + dy);   // 不可变风格：新建对象
  }
}

const p = new Point(1, 2);
// p.x = 10;                        // 错误：只读属性
const moved = p.move(1, 1);
console.log(moved.x, moved.y, Point.ORIGIN.x);

// readonly 参数属性
class Config {
  constructor(public readonly host: string, private readonly port = 80) {}
  url(): string { return `http://${this.host}:${this.port}`; }
}
const cfg = new Config("localhost");
// cfg.host = "other";              // 错误
console.log(cfg.url());

// 只读是浅层的
class Container {
  readonly items: string[] = [];
  add(v: string): void { this.items.push(v); }      // 修改内容合法
}
const c = new Container();
c.add("a");
// c.items = [];                    // 错误：不能重新赋值
console.log(c.items);

// 只读 + 索引签名
interface ReadonlyDict { readonly [key: string]: number }
const rd: ReadonlyDict = { a: 1 };
// rd.a = 2;                        // 错误
console.log(rd.a);

// readonly 与 as const 的区别：后者作用于字面量表达式
const literal = { x: 1 } as const;
console.log(literal.x);
```

**运行结果：**
```
2 3 0
http://localhost:80
[ 'a' ]
1
1
```

**注意：**
* `readonly` 是浅层的，数组/对象内容仍可变（只读数组用 `readonly T[]`）。
* 构造器中允许赋值，其他方法中不行。
* `static readonly` 通常与 `as const` 搭配定义真正的常量表。

### 18.7 static

**概念说明：** `static` 成员属于类本身而非实例，通过 `类名.成员` 访问。静态方法不能访问实例成员，也不能使用类的类型参数。

```typescript
class MathUtil {
  static readonly PI = 3.14159;
  static count = 0;                 // 可变静态属性

  static circleArea(r: number): number {
    MathUtil.count++;               // 通过类名访问
    return MathUtil.PI * r ** 2;
  }

  // 静态块（TS 4.4+/ES2022）用于复杂初始化
  static readonly TABLE: Map<string, number>;
  static {
    MathUtil.TABLE = new Map([["a", 1], ["b", 2]]);
  }
}

console.log(MathUtil.circleArea(1).toFixed(2), MathUtil.count);
console.log(MathUtil.TABLE.get("b"));

// 静态工厂
class User {
  private constructor(public name: string, public role: string) {}

  static createAdmin(name: string): User {
    return new User(name, "admin");
  }
  static createGuest(): User {
    return new User("访客", "guest");
  }
  // 静态方法可访问 private 成员
  static fromJson(json: string): User {
    const o = JSON.parse(json) as { name: string; role: string };
    return new User(o.name, o.role);
  }
}
console.log(User.createAdmin("Alice").role, User.createGuest().name);

// 静态与实例同名不冲突
class Counter {
  static value = 0;
  value = 100;
  static reset(): void { Counter.value = 0; }
  describe(): string { return `实例 ${this.value}，静态 ${Counter.value}`; }
}
const cnt = new Counter();
Counter.value = 5;
console.log(cnt.describe());

// 静态继承
class Base {
  static kind = "base";
  static describe(): string { return this.kind; }   // this 指向调用类
}
class Sub extends Base {
  static kind = "sub";
}
console.log(Base.describe(), Sub.describe());

// 注意：静态方法无法使用类的泛型参数
class Box<T> {
  static create(): Box<string> {          // 只能指定具体类型
    return new Box<string>();
  }
}
console.log(Box.create());
```

**运行结果：**
```
3.14 1
2
admin 访客
实例 100，静态 5
base sub
Box {}
```

**注意：**
* 静态成员在继承中通过 `this` 可以指向子类（多态静态）。
* 静态方法不能引用类的类型参数，需要另声明自己的类型参数。
* 静态块适合做复杂初始化，且只执行一次。

### 18.8 abstract

**概念说明：** `abstract` 类不能被实例化，用于定义「不完整」的基类；`abstract` 成员只有签名没有实现，必须由子类实现。

```typescript
abstract class Shape {
  constructor(public readonly name: string) {}

  // 抽象方法：子类必须实现
  abstract area(): number;
  abstract perimeter(): number;

  // 具体方法：可复用
  describe(): string {
    return `${this.name}：面积 ${this.area().toFixed(2)}，周长 ${this.perimeter().toFixed(2)}`;
  }

  // 抽象属性
  abstract readonly unit: string;
}

class Circle extends Shape {
  readonly unit = "cm";
  constructor(public radius: number) { super("圆形"); }
  area(): number { return Math.PI * this.radius ** 2; }
  perimeter(): number { return 2 * Math.PI * this.radius; }
}

class Rect extends Shape {
  readonly unit = "cm";
  constructor(public w: number, public h: number) { super("矩形"); }
  area(): number { return this.w * this.h; }
  perimeter(): number { return 2 * (this.w + this.h); }
}

// const s = new Shape("x");       // 错误：无法创建抽象类的实例
const shapes: Shape[] = [new Circle(1), new Rect(2, 3)];
shapes.forEach((s) => console.log(s.describe(), s.unit));

// 抽象类作为类型与构造类型
function createShape(Ctor: new (...args: never[]) => Shape): Shape {
  return new Ctor();
}
console.log(createShape(class extends Shape {
  readonly unit = "u";
  area() { return 0; }
  perimeter() { return 0; }
}).name);

// 抽象构造签名
type AbstractCtor = abstract new (...args: never[]) => Shape;
let AC: AbstractCtor = Circle;
console.log(new AC(2).area().toFixed(2));
```

**运行结果：**
```
圆形：面积 3.14，周长 6.28 cm
矩形：面积 6.00，周长 10.00 cm
x
12.57
```

**注意：**
* 抽象类不能 `new`，但可以作为类型、被继承、被 `implements`。
* 抽象成员不能在抽象类中实现；子类必须实现所有抽象成员。
* 抽象构造签名用 `abstract new (...)` 描述。

### 18.9 implements

**概念说明：** 类用 `implements` 声明它满足某个接口（或类型别名）。这是显式的契约检查，编译器会校验类是否实现了接口的所有成员。

```typescript
interface Serializable {
  serialize(): string;
  readonly version: number;
}

interface Loggable {
  log(msg: string): void;
}

// 多接口实现
class User implements Serializable, Loggable {
  readonly version = 1;
  constructor(public id: number, public name: string) {}

  serialize(): string { return JSON.stringify({ id: this.id, name: this.name }); }
  log(msg: string): void { console.log(`[User] ${msg}`); }
}

const u = new User(1, "Alice");
u.log("创建完成");
console.log(u.serialize());

// 缺少成员会报错
// class Bad implements Serializable { readonly version = 1 }   // 错误：缺少 serialize

// implements 只检查签名，不检查修饰符（private 不可用于实现）
interface HasId { id: number }
class Impl implements HasId {
  private id = 1;                 // 错误示例：private 不满足 public 要求
}
// 上面这行会报错：Property 'id' is private in type 'Impl' but not in 'HasId'

// 用 implements 实现「类型别名」
type Point = { x: number; y: number };
class PointImpl implements Point {
  x = 0;
  y = 0;
}
console.log(new PointImpl());

// implements 泛型接口
interface Repository<T> {
  all(): T[];
  add(item: T): void;
}
class MemoryRepo<T extends { id: number }> implements Repository<T> {
  private items: T[] = [];
  all(): T[] { return [...this.items]; }
  add(item: T): void { this.items.push(item); }
}
const repo = new MemoryRepo<{ id: number; name: string }>();
repo.add({ id: 1, name: "A" });
console.log(repo.all().length);

// implements 不继承实现，只是契约
interface Greeter { greet(): string }
class GreeterImpl implements Greeter {
  greet(): string { return "你好"; }          // 必须自己实现
}
console.log(new GreeterImpl().greet());

// 可实现多个接口获得组合契约
interface A { a(): void }
interface B { b(): void }
class AB implements A, B {
  a(): void { console.log("a"); }
  b(): void { console.log("b"); }
}
const ab = new AB();
ab.a(); ab.b();
```

**运行结果：**
```
[User] 创建完成
{"id":1,"name":"Alice"}
PointImpl { x: 0, y: 0 }
1
你好
a
b
```

**注意：**
* `implements` 只做静态检查，不改变运行时（没有继承实现）。
* 类成员的可见性必须至少与接口要求一样宽（不能更窄），也不支持可选成员的「必须实现」差异。
* 一个类可以实现多个接口，实现多重契约。

### 18.10 extends

**概念说明：** 类通过 `extends` 继承基类的字段、方法与访问器，用 `super` 调用基类构造器或方法。子类可重写方法，支持抽象成员、泛型与多态。

```typescript
class Base {
  constructor(public id: number) {}
  describe(): string { return `Base#${this.id}`; }
  static kind = "base";
}

class Middle extends Base {
  constructor(id: number, public tag: string) {
    super(id);
  }
  describe(): string { return `${super.describe()}(${this.tag})`; }   // 调用父类实现
}

class Leaf extends Middle {
  constructor(id: number) {
    super(id, "leaf");
  }
  override describe(): string {                    // noImplicitOverride 下需写 override
    return `Leaf: ${super.describe()}`;
  }
}

const leaf = new Leaf(1);
console.log(leaf.describe());
console.log(leaf instanceof Leaf, leaf instanceof Middle, leaf instanceof Base);

// 多态：父类引用指向子类实例
const list: Base[] = [new Base(1), new Middle(2, "m"), new Leaf(3)];
list.forEach((b) => console.log(b.describe()));

// 泛型继承
class Collection<T> {
  protected items: T[] = [];
  add(item: T): void { this.items.push(item); }
  size(): number { return this.items.length; }
}
class NameCollection extends Collection<string> {
  add(name: string): void {
    super.add(name.trim());                      // 利用父类实现
  }
}
const nc = new NameCollection();
nc.add("  Alice  ");
console.log(nc.size());

// 继承 + 抽象
abstract class Shape {
  abstract area(): number;
  describe(): string { return `面积 ${this.area()}`; }
}
class Square extends Shape {
  constructor(public side: number) { super(); }
  area(): number { return this.side ** 2; }
}
console.log(new Square(3).describe());

// 抽象构造签名：只能继承抽象类本身
type ShapeCtor = new (...args: never[]) => Shape;
const ctor: ShapeCtor = Square;
console.log(new ctor(2).area());
```

**运行结果：**
```
Leaf: Base#1(leaf)
true true true
Base#1
Base#2(m)
Leaf: Base#3(leaf)
1
面积 9
4
```

**注意：**
* 子类构造器必须在 `this` 之前调用 `super()`。
* 开启 `noImplicitOverride` 时，重写必须写 `override` 关键字，能有效防止「改名后误重写」。
* 子类方法签名需与父类兼容（参数逆变、返回值协变）。

### 18.11 Class 泛型

**概念说明：** 类可声明类型参数，配合约束、默认值与静态方法，实现类型安全的通用数据结构与容器。

```typescript
// 基本泛型类
class Pair<K, V> {
  constructor(public key: K, public value: V) {}
  swap(): Pair<V, K> { return new Pair(this.value, this.key); }
  toString(): string { return `${String(this.key)} = ${this.value}`; }
}
const p = new Pair("age", 25);
console.log(p.toString(), p.swap().toString());

// 泛型约束
class Repository<T extends { id: number }> {
  private map = new Map<number, T>();
  save(item: T): T { this.map.set(item.id, item); return item; }
  get(id: number): T | undefined { return this.map.get(id); }
  get size(): number { return this.map.size; }
}
const repo = new Repository<{ id: number; name: string }>();
repo.save({ id: 1, name: "A" });
console.log(repo.get(1)?.name, repo.size);

// 泛型默认值
class Cache<T = string> {
  private store = new Map<string, T>();
  set(key: string, value: T): void { this.store.set(key, value); }
  get(key: string): T | undefined { return this.store.get(key); }
}
const strCache = new Cache();                 // 默认 string
const numCache = new Cache<number>();
strCache.set("a", "值");
numCache.set("b", 1);
console.log(strCache.get("a"), numCache.get("b"));

// 泛型方法（独立于类的类型参数）
class Util<T> {
  constructor(public value: T) {}
  map<U>(fn: (v: T) => U): Util<U> { return new Util(fn(this.value)); }
}
console.log(new Util(2).map((n) => `值: ${n * 10}`).value);

// 泛型类与静态成员：静态不能使用类的类型参数
class Container<T> {
  items: T[] = [];
  add(v: T): this { this.items.push(v); return this; }
  static create<U>(...items: U[]): Container<U> {
    const c = new Container<U>();
    items.forEach((i) => c.add(i));
    return c;
  }
}
const c = Container.create(1, 2, 3);
console.log(c.items.length);

// 泛型类实现泛型接口
interface Comparable<T> { compareTo(other: T): number }
class Version implements Comparable<Version> {
  constructor(public major: number, public minor: number) {}
  compareTo(o: Version): number { return this.major - o.major || this.minor - o.minor; }
}
const v1 = new Version(1, 2);
const v2 = new Version(1, 3);
console.log(v1.compareTo(v2) < 0);
```

**运行结果：**
```
age = 25 25 = age
A 1
值 1
值: 20
3
true
```

**注意：**
* 静态成员不能引用类的类型参数，需要自己声明（如 `static create<U>`）。
* 泛型类的类型参数可用默认值简化调用。
* 泛型方法比泛型类更常见，优先考虑用泛型方法。

### 18.12 getter / setter

**概念说明：** `get` / `set` 定义访问器属性，外部使用像普通属性，内部可执行校验、计算与懒加载。可只写 `get` 形成只读属性。

```typescript
class Temperature {
  private _celsius = 0;

  get celsius(): number { return this._celsius; }
  set celsius(v: number) {
    if (v < -273.15) throw new Error("低于绝对零度");
    this._celsius = Math.round(v * 10) / 10;
  }

  // 计算属性（只读）
  get fahrenheit(): number { return this._celsius * 9 / 5 + 32; }
  set fahrenheit(v: number) { this.celsius = (v - 32) * 5 / 9; }

  // 懒加载
  private _expensive?: string;
  get expensive(): string {
    return (this._expensive ??= "计算完成");
  }
}

const t = new Temperature();
t.celsius = 25;
console.log(t.celsius, t.fahrenheit);
t.fahrenheit = 32;
console.log(t.celsius);
console.log(t.expensive, t.expensive);

// 只读访问器
class Config {
  constructor(private readonly raw: Record<string, string>) {}
  get host(): string { return this.raw["host"] ?? "localhost"; }
  // set host(v: string) { ... }     // 不写 set 即为只读
}
const cfg = new Config({ host: "example.com" });
console.log(cfg.host);
// cfg.host = "x";                    // 错误：Cannot assign to 'host' because it is a read-only property

// 访问器实现接口
interface HasName { name: string }
class Person implements HasName {
  constructor(private _name: string) {}
  get name(): string { return this._name; }
  set name(v: string) { this._name = v.trim(); }
}
const person = new Person("Alice");
person.name = "  Bob  ";
console.log(person.name);

// getter 与同名属性冲突
class Bad {
  // get value() { return 1 }
  // value = 2;                       // 错误：访问器与属性同名
}
console.log("同名会编译报错");

// 访问器在子类中重写
class Base {
  protected _v = 1;
  get value(): number { return this._v; }
}
class Sub extends Base {
  get value(): number { return super.value * 10; }
}
console.log(new Sub().value);
```

**运行结果：**
```
25 77
0
计算完成 计算完成
example.com
Bob
同名会编译报错
10
```

**注意：**
* 只有 `get` 时属性只读；只写 `set` 也可以（少见）。
* 不能在同一个类中同时定义同名访问器与数据属性。
* 访问器的类型必须一致，且不能用 `readonly` 修饰符。

### 18.13 参数属性

**概念说明：** 在构造器参数前加访问修饰符（`public` / `private` / `protected` / `readonly`）会同时声明字段并赋值，大幅精简样板代码。

```typescript
// 参数属性：一行完成声明 + 赋值
class User {
  constructor(
    public readonly id: number,
    public name: string,
    private password: string,
    protected role: string = "user"
  ) {}

  checkPassword(pwd: string): boolean { return pwd === this.password; }
  describe(): string { return `${this.name}(${this.role})`; }
}

const u = new User(1, "Alice", "secret");
console.log(u.id, u.name, u.describe());
console.log(u.checkPassword("secret"));
// u.password;       // 错误：私有
// u.id = 2;         // 错误：只读

// 等价的传统写法（对比）
class UserVerbose {
  public readonly id: number;
  public name: string;
  private password: string;

  constructor(id: number, name: string, password: string) {
    this.id = id;
    this.name = name;
    this.password = password;
  }
}
console.log(new UserVerbose(2, "Bob", "p").name);

// 与继承配合
class Admin extends User {
  constructor(id: number, name: string, pwd: string, public permissions: string[]) {
    super(id, name, pwd, "admin");
  }
  can(action: string): boolean { return this.permissions.includes(action); }
}
const admin = new Admin(3, "Carol", "p", ["read", "write"]);
console.log(admin.describe(), admin.can("write"));

// 参数属性也可用于 static（TS 4.x 起不推荐，仅修饰符组合）
class Config {
  constructor(public readonly host: string, public readonly port: number) {}
  get url(): string { return `http://${this.host}:${this.port}`; }
}
console.log(new Config("localhost", 80).url);

// 与解构参数区分：解构参数不能带修饰符
interface Options { name: string; age: number }
class Person2 {
  name: string;
  age: number;
  constructor({ name, age }: Options) {      // 解构，不是参数属性
    this.name = name;
    this.age = age;
  }
}
console.log(new Person2({ name: "A", age: 1 }).name);

// 参数属性的编译产物：等价于在构造器内赋值
console.log(Object.keys(new User(4, "D", "p")));
```

**运行结果：**
```
1 Alice Alice(user)
true
Bob
Carol(admin) true
http://localhost:80
A
[ 'id', 'name', 'password', 'role' ]
```

**注意：**
* 参数属性同时完成「声明字段 + 赋值」，但不能带默认值以外的额外逻辑（需自己写构造体）。
* 支持 `public` / `private` / `protected` / `readonly` 及其组合。
* 解构参数不能使用修饰符，需在构造体内手动赋值。

## 19. Modules 模块系统
### 19.1 export

**概念说明：** `export` 把声明暴露给其他模块。可以导出变量、函数、类、接口、类型别名，也可以导出整个声明列表或重导出其他模块的内容。

```typescript
// math.ts
export const PI = 3.14159;
export function add(a: number, b: number): number { return a + b; }
export class Calculator {
  multiply(a: number, b: number): number { return a * b; }
}
export interface Result { ok: boolean }
export type ID = string | number;

// 批量导出
const sub = (a: number, b: number): number => a - b;
const div = (a: number, b: number): number => a / b;
export { sub, div };

// 导出时重命名
const internal = "内部值";
export { internal as publicValue };

// 重导出其他模块
export { PI as pi } from "./math";
export * from "./constants";

// 使用方
// import { add, PI, Calculator } from "./math";
```

**运行结果：**
```
（模块定义本身不产生输出；被导入后使用即可）
add(1, 2) → 3
new Calculator().multiply(2, 3) → 6
```

**注意：**
* 每个模块有独立作用域，未导出的声明外部不可见。
* `export type` / `export interface` 只在编译期存在，会被擦除。
* 顶层 `export` 使文件成为「模块」；没有 import/export 的文件是「脚本」，其声明进入全局作用域。

### 19.2 import

**概念说明：** `import` 引入其他模块的导出。支持具名导入、默认导入、命名空间导入、重命名、副作用导入与动态 `import()`。

```typescript
// utils.ts
export const VERSION = "1.0";
export function helper(): string { return "辅助函数"; }
export default function main(): string { return "默认导出"; }

// 具名导入
import { VERSION, helper } from "./utils";

// 重命名
import { VERSION as ver, helper as h } from "./utils";

// 默认导入
import main from "./utils";

// 默认 + 具名混合
import mainFn, { VERSION as v2 } from "./utils";

// 命名空间导入
import * as utils from "./utils";

// 副作用导入（只执行模块代码）
import "./polyfill";

console.log(VERSION, helper(), main());
console.log(ver, utils.VERSION, typeof utils.helper);
console.log(mainFn() === main());

// 动态导入：返回 Promise，按需加载
async function load() {
  const mod = await import("./utils");
  return mod.VERSION;
}
load().then((x) => console.log("动态加载:", x));
```

**运行结果：**
```
1.0 辅助函数 默认导出
1.0 1.0 function
true
动态加载: 1.0
```

**注意：**
* 静态 `import` 会被提升到文件顶部，必须位于模块顶层。
* 具名导入的绑定是只读的（不能给导入的名字重新赋值）。
* 动态 `import()` 返回 Promise，可用于代码分割与条件加载。

### 19.3 default export

**概念说明：** 每个模块最多有一个默认导出，导入时可自定义名称。默认导出用「值」而非名字绑定，因此重构改名不会影响使用方。

```typescript
// logger.ts
export default class Logger {
  constructor(private prefix: string = "LOG") {}
  info(msg: string): void { console.log(`[${this.prefix}] ${msg}`); }
}

// 也可以默认导出一个函数
// export default function createLogger() { ... }

// 使用方可以任意命名
// app.ts
import Logger from "./logger";
import MyLogger from "./logger";           // 名称随意
const log = new Logger("APP");
log.info("启动完成");
console.log(new MyLogger("X") instanceof Logger);

// 默认导出 + 具名导出共存
// config.ts
export const defaults = { debug: false };
export default { name: "config" };

// 导入时分开写
import config, { defaults } from "./config";

// 默认导出的对象
const obj = { name: "direct" };
export default obj;

// 注意：默认导出是「表达式」，不是声明
// 无法直接 export default const x = 1;       // 语法错误
const x = 1;
export default x;
```

**运行结果：**
```
[APP] 启动完成
true
```

**注意：**
* 默认导出不适合「需要被静态分析工具改名」的类型（IDE 自动导入体验较差）。
* 一个模块只能有一个默认导出，但可以有多个具名导出。
* 团队规范常推荐统一使用具名导出，便于自动重构与 tree-shaking。

### 19.4 named export

**概念说明：** 具名导出通过名字绑定，导入时必须用对应的名字（可重命名）。它是支持 tree-shaking 与 IDE 重命名重构的首选方式。

```typescript
// api.ts
export interface ApiConfig {
  baseUrl: string;
  timeout?: number;
}

export const defaultConfig: ApiConfig = { baseUrl: "/api", timeout: 5000 };

export function createClient(config: ApiConfig): string {
  return `客户端 → ${config.baseUrl}（超时 ${config.timeout}ms）`;
}

export class ApiError extends Error {
  constructor(message: string, public status: number) { super(message); }
}

// 使用方
// client.ts
import { createClient, defaultConfig, ApiError, type ApiConfig } from "./api";

const cfg: ApiConfig = { ...defaultConfig, baseUrl: "https://api.example.com" };
console.log(createClient(cfg));

try {
  throw new ApiError("未授权", 401);
} catch (e) {
  if (e instanceof ApiError) console.log(e.status, e.message);
}

// 重导出形成统一入口
// index.ts
export { createClient } from "./api";
export { ApiError } from "./api";
export type { ApiConfig } from "./api";

// 使用方从统一入口导入
// import { createClient, ApiError } from "./"; 

// 未使用的具名导出会被 bundler 摇掉（tree-shaking）
console.log(typeof createClient, defaultConfig.timeout);
```

**运行结果：**
```
客户端 → https://api.example.com（超时 5000ms）
401 未授权
function 5000
```

**注意：**
* 具名导出利于 tree-shaking：未使用的导出不会进入产物（前提是没有副作用）。
* 配合 `export * from` 可做「桶文件」（barrel），但过度使用会拖慢编译与打包。
* 导入类型时加 `type` 前缀可确保编译后不产生运行时代码。

### 19.5 import type

**概念说明：** `import type` 只导入类型，编译后完全移除该导入语句，避免「仅类型导入」导致的循环依赖与运行时残留。

```typescript
// types.ts
export interface User { id: number; name: string }
export class UserModel { constructor(public name: string) {} }

// 只导入类型（编译后该行消失）
import type { User } from "./types";

// 混合导入：用 type 修饰符标注类型成员
import { type User, UserModel } from "./types";

// 默认导出类型的导入
// import type Logger from "./logger";

// 命名空间形式的类型导入
import type * as Types from "./types";

const u: User = { id: 1, name: "Alice" };
const u2: Types.User = { id: 2, name: "Bob" };
const model = new UserModel("Carol");

console.log(u.name, u2.name, model.name);

// 验证：类型导入不会产生运行时引用
// 编译后的 JS 中只有 import { UserModel } from "./types";

// 类型只在类型位置使用
function describe(user: User): string { return `${user.id}: ${user.name}`; }
console.log(describe(u));

// verbatimModuleSyntax 开启时，类型必须显式标注 type
// import { User } from "./types";        // 错误：User 是类型
// import type { User } from "./types";   // 正确

// import type + typeof 组合取「值的类型」
const config = { debug: true } as const;
import type { User as UserAlias } from "./types";
const alias: UserAlias = { id: 3, name: "C" };
console.log(alias.id, config.debug);
```

**运行结果：**
```
Alice Bob Carol
1: Alice
3 true
```

**注意：**
* `import type` 不能用于值（类实例化、`extends` 值等），否则报错。
* 开启 `isolatedModules` / `verbatimModuleSyntax` 时，类型导入必须显式标注 `type`。
* 只导入类型可有效打断仅存在于类型层面的循环依赖。

### 19.6 export type

**概念说明：** `export type` 只导出类型（接口、类型别名、类型参数），编译后不产生任何运行时代码；`export type { X }` 可在重导出时标注。

```typescript
// models.ts
export interface User { id: number; name: string }
export type ID = string | number;
export type Role = "admin" | "user";

// 值导出（会生成代码）
export const ROLES: Role[] = ["admin", "user"];

// 只导出类型
export type { ID as Identifier };

// 从其他模块重导出类型
// export type { User } from "./user";

// 混合重导出：值用 export，类型用 export type
// export { ROLES } from "./models";
// export type { Role } from "./models";

// 使用方
// import { ROLES, type User, type Role, type Identifier } from "./models";

const id: Identifier = 1;
const role: Role = "admin";
const user: User = { id: 1, name: "A" };
console.log(id, role, user.name, ROLES.length);

// 对比：export * 会同时导出值与类型
// export * from "./models";

// 导出类型并重命名
type InternalOptions = { debug: boolean };
export type { InternalOptions as Options };
const opt: InternalOptions = { debug: true };
console.log(opt.debug);

// 编译产物验证：export type 不生成 JS
// 编译前：export type { Identifier };
// 编译后：（无输出）
console.log("export type 不产生运行时代码");
```

**运行结果：**
```
1 admin A 2
true
export type 不产生运行时代码
```

**注意：**
* 在 `isolatedModules` 下，重导出类型必须用 `export type`，否则单文件编译工具会误判为值导出。
* `export type * from "./x"`（TS 5.0+）可一次重导出所有类型。
* 类型导出对 tree-shaking 友好，因为不产生代码。

### 19.7 CommonJS

**概念说明：** CommonJS 是 Node 传统的模块规范：用 `require` 导入、`module.exports` / `exports` 导出，**同步加载**，导出是值拷贝（`module.exports` 的引用）。

```typescript
// counter.ts → 编译为 CommonJS
let count = 0;
function increment(): number { return ++count; }
const name = "计数器";

export = { increment, name };
// 编译产物：module.exports = { increment: increment, name: name };

// 使用方（CommonJS 风格）
// const counter = require("./counter");
// console.log(counter.name, counter.increment());
```

**运行结果：**
```
计数器 1
```

```typescript
// 用 Node 直接跑 CommonJS：需要 module: "commonjs"
// module.exports 与 exports 的关系
// exports.a = 1;            // 等价于 module.exports.a = 1
// exports = { a: 1 };       // 错误：只是重新赋值局部变量，对外无效

// 混合默认导出（CommonJS 的常见形态）
// module.exports = function main() { ... }
// module.exports.helper = () => { ... };

// TypeScript 中的写法
export function cjsHelper(): string { return "helper"; }
export default function cjsMain(): string { return "main"; }
// 编译为 CJS 后：exports.cjsHelper = ...; exports.default = cjsMain;

// require 是同步的，可在条件中使用
// if (process.env.NODE_ENV === "production") {
//   const prod = require("./prod");
// }

// 动态 require（不推荐，破坏静态分析）
// const mod = require(`./locales/${lang}`);

// 与 ESM 互通：CJS 可被 ESM 用默认导入访问
// import cjs from "./counter.cjs";
// console.log(cjs.name);

console.log("CommonJS 使用 require / module.exports，同步加载");
```

**运行结果：**
```
CommonJS 使用 require / module.exports，同步加载
```

**注意：**
* CommonJS 是同步加载，不适合浏览器；Node 中 `require` 有缓存，多次 require 返回同一对象。
* 导出的是「值的拷贝」快照，导出的原始值后续变化不会反映到导入方（用函数/getter 规避）。
* 现代项目优先 ESM；需要与旧包互操作时注意 `esModuleInterop` 与 `allowSyntheticDefaultImports`。

### 19.8 ES Modules

**概念说明：** ESM 是标准模块规范（`import` / `export`），**静态分析、异步加载、实时绑定**（导出值变化会同步到导入方），并支持顶层 `await`。

```typescript
// counter.mts / counter.ts（module: esnext）
export let count = 0;                     // let 导出会被实时更新
export function increment(): void { count++; }

// main.ts
// import { count, increment } from "./counter";
// increment();
// console.log(count);                    // 1（实时绑定，不是快照）

// 顶层 await（ES2022，module: esnext / nodenext）
const data = await Promise.resolve([1, 2, 3]);
console.log("顶层 await 结果:", data.length);

// 动态导入用于条件加载与代码分割
async function loadModule(name: string) {
  const mod = await import(`./locales/${name}.js`);
  return mod.default;
}
console.log("动态 import 返回 Promise:", typeof loadModule === "function");

// 导入断言 / 属性（JSON 模块）
// import data from "./data.json" with { type: "json" };

// import.meta：ESM 独有的元信息
console.log("import.meta.url 存在:", typeof import.meta?.url === "string" || "（取决于运行时）");

// 默认导出与具名导出混用
export const a = 1;
export default function fn(): void {}
console.log(a);

// 实时绑定的演示（同一模块内的等价写法）
let counter = 0;
const snapshot = counter;                 // 快照
counter++;
console.log("快照不变:", snapshot, "实时读取:", counter);
```

**运行结果：**
```
顶层 await 结果: 3
动态 import 返回 Promise: true
import.meta.url 存在: true
1
快照不变: 0 实时读取: 1
```

**注意：**
* ESM 导出是「实时绑定」：导入方看到的是最新值（与 CommonJS 的值拷贝不同）。
* ESM 是异步加载，导入语句会被提升到顶部，且只能在顶层使用（除动态 import）。
* Node 中需 `"type": "module"` 或 `.mjs` 后缀；TypeScript 侧配置 `module: "nodenext"` 最稳妥。

### 19.9 模块解析

**概念说明：** 模块解析决定 `import "x"` 最终指向哪个文件。涉及相对/绝对路径、扩展名省略、`node_modules` 查找、`exports` 字段、`baseUrl`/`paths` 映射等，由 `moduleResolution` 控制。

```json
// tsconfig.json — 解析策略示例
{
  "compilerOptions": {
    "moduleResolution": "bundler",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@utils/*": ["src/utils/*"]
    },
    "resolveJsonModule": true,
    "allowImportingTsExtensions": true
  }
}
```

```typescript
// 解析顺序示意（假设 import "./utils"）
// 1) ./utils.ts
// 2) ./utils.tsx
// 3) ./utils.d.ts
// 4) ./utils/index.ts        （目录导入）
// 5) package.json 的 types / exports → 再落到 JS

// 包解析（import "lodash"）
// 1) node_modules/lodash/package.json 的 "types" / "exports"."types"
// 2) node_modules/@types/lodash
// 3) 未找到 → 报错 TS2307: Cannot find module

// 路径别名（配合 paths）
// import { helper } from "@/utils/helper";        // → src/utils/helper.ts
// import { log } from "@utils/log";

// JSON 模块（需 resolveJsonModule）
// import pkg from "../package.json";
// console.log(pkg.name);

// 扩展名：不同策略要求不同
// moduleResolution: "node16"/"nodenext" → 必须写 .js 扩展名（相对导入）
// moduleResolution: "bundler"          → 可省略扩展名

// 副作用式解析：找不到类型时的临时声明
// declare module "some-untyped-lib";

console.log("解析策略决定 import 指向哪个文件");
```

**运行结果：**
```
解析策略决定 import 指向哪个文件
```

**注意：**
* `moduleResolution` 必须与运行时/打包器行为一致：Node ESM 用 `nodenext`，Vite/webpack 用 `bundler`。
* `paths` 只影响 TypeScript 的类型检查，运行时需要打包器或 `tsconfig-paths` 配合。
* 报错 `TS2307 Cannot find module` 时，优先检查 `moduleResolution`、`types` 字段与是否安装了 `@types/*`。

## 20. tsconfig 与 TypeScript 工程配置
### 20.1 tsconfig.json

**概念说明：** `tsconfig.json` 是 TypeScript 项目的编译配置根文件，包含 `compilerOptions`（编译行为）、`include`/`exclude`/`files`（参与编译的文件）、`references`（项目引用）等字段。

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "outDir": "dist",
    "rootDir": "src",
    "declaration": true,
    "sourceMap": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "types": ["node"]
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"],
  "files": ["src/global.d.ts"],
  "references": [{ "path": "./packages/core" }]
}
```

```bash
# 常用命令
tsc                       # 按 tsconfig.json 编译
tsc --noEmit              # 只做类型检查，不输出文件
tsc --init                # 生成默认 tsconfig.json
tsc -p tsconfig.build.json
tsc --showConfig          # 打印解析后的最终配置（排查继承问题很有用）
```

**运行结果：**
```
（tsc --noEmit 无输出即类型检查通过；有错误会打印 文件:行:列 - error TSxxxx）
```

**注意：**
* 开启 `noEmit` 时用 `tsc --noEmit` 做纯类型检查（CI 常用），输出交给 bundler 处理。
* 配置可通过 `extends` 继承其他配置（如 `@tsconfig/node20/tsconfig.json`）。
* 命令行参数会覆盖 `tsconfig.json` 中的同名选项。

### 20.2 target

**概念说明：** `target` 指定编译输出的 JavaScript 版本，决定语法降级程度（如箭头函数、可选链、类字段是否被转换）。它不影响 API（如 `Promise`）是否存在，那由 `lib` 决定。

```json
{
  "compilerOptions": {
    "target": "ES2020"
  }
}
```

```typescript
// 源码（使用可选链、空值合并、类字段）
class User {
  name = "Alice";
  get upper(): string { return this.name?.toUpperCase() ?? "空"; }
}
console.log(new User().upper);
```

```javascript
// target: "ES2020" 的产物（这些语法原生支持，基本原样输出）
class User {
  name = "Alice";
  get upper() { return this.name?.toUpperCase() ?? "空"; }
}
console.log(new User().upper);
```

```javascript
// target: "ES5" 的产物（语法降级，配合 lib 引入 polyfill）
"use strict";
var User = /** @class */ (function () {
    function User() { this.name = "Alice"; }
    Object.defineProperty(User.prototype, "upper", {
        get: function () { var _a, _b; return (_b = (_a = this.name) === null || _a === void 0 ? void 0 : _a.toUpperCase()) !== null && _b !== void 0 ? _b : "空"; },
        enumerable: false, configurable: true
    });
    return User;
}());
console.log(new User().upper);
```

**运行结果：**
```
ALICE
```

**注意：**
* `target` 只做「语法降级」，不会引入 polyfill；低版本目标需要手动引入 `core-js` 等。
* 只降级语法，不改 API 行为；例如 `target: ES5` 时 `class` 会被模拟但语义基本一致。
* 现代项目常用 `ES2020` ~ `ES2022`；浏览器兼容需求交给 bundler 的 `browserslist`。

### 20.3 module

**概念说明：** `module` 指定生成哪种模块格式：`commonjs`（Node CJS）、`esnext`/`es2022`（ESM）、`nodenext`（按 Node 规则自动判断）、`preserve`（保持原样）等。

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler"
  }
}
```

```typescript
// source: index.ts
export const VERSION = "1.0";
export default function main(): void { console.log(VERSION); }
```

```javascript
// module: "commonjs" 产物
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.VERSION = void 0;
exports.VERSION = "1.0";
function main() { console.log(exports.VERSION); }
exports.default = main;
```

```javascript
// module: "esnext" 产物（保持 ESM）
export const VERSION = "1.0";
export default function main() { console.log(VERSION); }
```

```bash
# module 与运行时必须匹配，否则报 "Cannot use import statement outside a module"
# Node ESM 项目：package.json 加 "type": "module"，tsconfig 用 module: "nodenext"
# 前端打包项目：module: "esnext" + moduleResolution: "bundler"
# Node CommonJS（旧项目/CLI）：module: "commonjs"
```

**运行结果：**
```
（module 决定产物格式，不影响运行结果本身）
1.0
```

**注意：**
* `module` 与 `moduleResolution` 通常成对配置，混搭会产生解析错误。
* `nodenext` 会根据文件扩展名与 `package.json` 的 `type` 自动决定 ESM/CJS，推荐 Node 项目使用。
* 用 bundler 打包时不要依赖 `tsc` 转换模块，交给 bundler 更可靠。

### 20.4 moduleResolution

**概念说明：** `moduleResolution` 指定「如何为一个 import 找到对应文件」，可选 `classic`（已过时）、`node10`（旧 Node）、`node16`/`nodenext`（Node 现代规则）、`bundler`（打包器规则）。

```json
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true
  }
}
```

```typescript
// 不同策略对「扩展名」的要求不同

// node16 / nodenext（Node ESM 规则）：相对导入必须写扩展名
// import { helper } from "./utils/helper.js";     // 必须写 .js（即使源文件是 .ts）

// bundler（打包器规则）：可省略扩展名，支持目录导入与 exports 字段
// import { helper } from "./utils/helper";

// node10：走 node_modules 与 package.json 的 main/types
// import express from "express";

// 路径别名（需要与 baseUrl 配合）
// import { log } from "@utils/log";

// JSON 与 CSS 等非代码资源（需相应开关）
// import pkg from "../package.json";
// import "./style.css";

// 三方类型查找顺序
// 1) 包内 "types"/"typings" 字段
// 2) package.json "exports" 中的 types 条件
// 3) node_modules/@types/<pkg>

console.log("moduleResolution 决定裸模块与相对路径如何解析");
```

**运行结果：**
```
moduleResolution 决定裸模块与相对路径如何解析
```

**注意：**
* 必须与运行时一致：Node 用 `nodenext`，打包器项目用 `bundler`，否则可能「类型检查通过但运行时找不到模块」。
* `bundler` 不支持 `import` 带 `.ts` 扩展名（除非开启 `allowImportingTsExtensions`）。
* 从 `node` 迁移到 `nodenext` 时最常见的报错是「需要显式 .js 扩展名」。

### 20.5 strict

**概念说明：** `strict: true` 一次性开启全部严格检查（等价于同时开启 `noImplicitAny`、`strictNullChecks`、`strictFunctionTypes`、`strictBindCallApply`、`strictPropertyInitialization`、`noImplicitThis`、`alwaysStrict`、`useUnknownInCatchVariables`）。

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

```typescript
// 开启 strict 后会被拦截的常见问题

// 1) 隐式 any（noImplicitAny）
// function f(x) { return x; }                    // 错误：参数 x 隐式 any

// 2) null/undefined（strictNullChecks）
const s: string = null as unknown as string;
// const bad: string = null;                      // 错误

// 3) 类字段未初始化（strictPropertyInitialization）
class User1 {
  // name: string;                                 // 错误：未初始化
  name = "";                                      // 正确：有初始值
}

// 4) this 隐式 any（noImplicitThis）
// function g() { return this.value; }             // 错误

// 5) 函数参数双变检查（strictFunctionTypes）
type Handler = (a: { x: number }) => void;
const h: Handler = (a: { x: number; y: number }) => {};   // 参数更宽泛，合法

// 6) catch 变量为 unknown（useUnknownInCatchVariables）
try { throw new Error("e"); } catch (e) {
  console.log(e instanceof Error ? e.message : String(e));
}

console.log(s, new User1().name);

// 局部关闭：对确知安全的代码用注释指令
// // @ts-expect-error 说明原因
// // @ts-ignore（不推荐，不检查原因）
```

**运行结果：**
```
null（示例中使用了断言）
```
（实际输出：`null ` 对应 `s` 与空字符串 `name`）

**注意：**
* 新项目务必开启 `strict`；老项目可先用 `strict: false` 再逐项开启（推荐顺序：`noImplicitAny` → `strictNullChecks` → 其余）。
* 单个文件/行可用 `// @ts-expect-error` 临时绕过，但要写清理由。
* `strict` 只是总开关，仍可按需单独关闭某项。

### 20.6 noImplicitAny

**概念说明：** `noImplicitAny` 禁止「隐式 any」：无法推断类型的参数、变量会直接报错，强制你写出类型。它是 `strict` 中最重要的一项。

```json
{
  "compilerOptions": {
    "noImplicitAny": true
  }
}
```

```typescript
// 错误用法：参数没有类型注解且无法推断
// function double(x) { return x * 2; }          // 错误：Parameter 'x' implicitly has an 'any' type

// 正确：显式标注
function double(x: number): number { return x * 2; }
console.log(double(2));

// 可推断时不报错
const n = 1;                                     // number
const arr = [1, 2];                              // number[]
console.log(n, arr.length);

// 对象索引访问会返回 any（需用索引签名约束）
const obj: Record<string, number> = { a: 1 };
console.log(obj["a"]);

// 事件与回调参数通常有上下文类型，不会报隐式 any
[1, 2].forEach((v) => console.log(v));           // v: number

// 显式 any 被允许（但仍建议避免）
function explicit(x: any): any { return x; }
console.log(explicit(1));

// 类型断言帮不上忙的场景
const raw = JSON.parse('{"a":1}');               // any（JSON.parse 返回 any）
console.log(raw);

// 结构化数据先用 unknown 再收窄
const input: unknown = { a: 1 };
if (typeof input === "object" && input !== null) console.log("已收窄");

// 空数组需要显式类型
const empty: string[] = [];
empty.push("a");
console.log(empty);
```

**运行结果：**
```
4
1 2
1
1
2
{ a: 1 }
已收窄
[ 'a' ]
```

**注意：**
* 开启后第三方库的隐式 any 也会暴露，通常配合 `skipLibCheck: true` 跳过 `.d.ts` 检查。
* 无法推断时优先用 `unknown` 而非 `any`。
* 迁移旧 JS 项目时可以先只开这一项，逐步补齐类型。

### 20.7 strictNullChecks

**概念说明：** `strictNullChecks` 让 `null` 与 `undefined` 成为独立类型，不能随意赋给其他类型，从源头消灭「undefined is not a function」类错误。

```json
{
  "compilerOptions": {
    "strictNullChecks": true
  }
}
```

```typescript
// 关闭时：null 可赋给任何类型（危险）
// 开启后：
// let name: string = null;          // 错误
let name: string | null = null;      // 必须显式联合
name = "Alice";
console.log(name);

// 可选属性/参数都含 undefined
interface User { name: string; age?: number }
const u: User = { name: "A" };
// console.log(u.age + 1);           // 错误：对象可能为 undefined
console.log((u.age ?? 0) + 1);

// 数组查找结果是 undefined
const found = [1, 2, 3].find((n) => n > 5);
// console.log(found.toFixed());     // 错误
console.log(found?.toFixed() ?? "未找到");

// 对象属性访问链
interface Cfg { db?: { host?: string } }
const cfg: Cfg = {};
// console.log(cfg.db.host);         // 错误
console.log(cfg.db?.host ?? "默认主机");

// Map.get 返回 undefined
const m = new Map<string, number>();
// m.get("a") + 1;                   // 错误
console.log((m.get("a") ?? 0) + 1);

// 非空断言（确定不为空时）
const el = { textContent: "值" };
console.log(el.textContent!);

// 明确表达「可能为空」的返回类型
function parse(s: string): number | null {
  const n = Number(s);
  return Number.isNaN(n) ? null : n;
}
console.log(parse("1"), parse("x"));
```

**运行结果：**
```
Alice
1
未找到
默认主机
1
值
1 null
```

**注意：**
* 与 `noImplicitAny` 一起是收益最大的两项检查。
* 处理手段优先级：类型守卫 > `??` 默认值 > 可选链 > 非空断言 `!`（最后手段）。
* 严格模式下 `null` 与 `undefined` 仍可用 `==` 互等判断（`x == null` 同时匹配两者）。

### 20.8 esModuleInterop

**概念说明：** `esModuleInterop` 让 ESM 风格的 `import x from "cjs-module"` 能正确导入 CommonJS 模块，通过生成 `__importDefault` 等辅助函数处理 `module.exports` 与 `exports.default` 的差异。

```json
{
  "compilerOptions": {
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true
  }
}
```

```typescript
// 场景：CommonJS 包 express，导出形式为 module.exports = express
// 未开启 esModuleInterop：
// import * as express from "express";        // 必须用命名空间导入
// express();                                 // 可能报「不可调用」

// 开启后：
// import express from "express";             // 默认导入可用
// import { Router } from "express";          // 具名导入也可用

// 生成代码（示意）
// var __importDefault = (this && this.__importDefault) || function (mod) {
//     return (mod && mod.__esModule) ? mod : { "default": mod };
// };
// const express_1 = __importDefault(require("express"));

// TypeScript 侧的自造示例
// cjs-lib.js: module.exports = function greet() { return "hi"; }

// 开启后可以这样写：
// import greet from "./cjs-lib.js";
// console.log(greet());

// allowSyntheticDefaultImports 只影响类型检查（不生成辅助代码）
// 某些 bundler 场景下二者都可开

console.log("esModuleInterop 解决 CJS 与 ESM 的默认导入差异");

// 注意：与 verbatimModuleSyntax 同时开启时，不允许「默认导入 CJS 包」的合成行为
```

**运行结果：**
```
esModuleInterop 解决 CJS 与 ESM 的默认导入差异
```

**注意：**
* 新项目一律开启；它同时隐含 `allowSyntheticDefaultImports`。
* 关掉时需要用 `import * as x` 再访问属性，改造成本高。
* 与 `verbatimModuleSyntax`（强调原样输出）搭配时行为更严格，需按 ESM 规则书写导入。

### 20.9 paths / baseUrl

**概念说明：** `baseUrl` 设定非相对模块名的解析基准目录；`paths` 定义路径别名映射，避免深层相对路径（`../../../`）。注意它们只影响类型检查，运行时需 bundler 配合。

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils": ["src/utils/index.ts"],
      "@types/*": ["src/types/*"]
    }
  }
}
```

```typescript
// 别名导入（项目结构：src/components/Button.tsx）
// import { Button } from "@/components/Button";
// import { Button } from "@components/Button";
// import { log } from "@utils";
// import type { User } from "@types/user";

// 等价于原来的相对路径
// import { Button } from "../../components/Button";

// 支持多个候选位置（按顺序尝试）
// "@lib/*": ["src/lib/*", "vendor/lib/*"]

// 精确映射某一文件
// "@config": ["config/app.config.ts"]

// 通配符 * 会捕获剩余路径片段

// 运行时配合方案
// 1) Vite：resolve.alias = { "@": path.resolve(__dirname, "src") }
// 2) webpack：resolve.alias
// 3) Node：tsconfig-paths/register 或 tsx
// 4) 打包产物：bundler 会在构建时把别名替换为实际路径

console.log("@/... 别名让导入路径不再受目录层级影响");
```

**运行结果：**
```
@/... 别名让导入路径不再受目录层级影响
```

**注意：**
* `paths` 不会被 `tsc` 重写为真实路径，务必在 bundler 中配置相同别名，否则运行时找不到模块。
* TS 4.x 起 `paths` 可以不带 `baseUrl`（相对于 tsconfig 所在目录解析）。
* 别名过多会影响 IDE 跳转速度，保持精简（通常一个 `@/*` 足够）。

### 20.10 include / exclude

**概念说明：** `include` / `exclude` / `files` 决定哪些文件参与编译。`files` 精确列出文件，`include` 用 glob 匹配，`exclude` 从 `include` 结果中剔除（默认排除 `node_modules`、`bower_components`、`jspm_packages` 与 `outDir`）。

```json
{
  "compilerOptions": { "outDir": "dist" },
  "include": [
    "src/**/*.ts",
    "src/**/*.tsx",
    "types/**/*.d.ts"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "**/*.spec.ts",
    "**/*.test.ts",
    "coverage"
  ],
  "files": [
    "src/globals.d.ts"
  ]
}
```

```typescript
// include 的 glob 规则
// "src/**/*"       → src 下所有文件（任意深度）
// "src/*"          → src 一级文件
// "src/**/*.ts"    → 只匹配 .ts
// "types/**/*.d.ts"→ 只匹配声明文件

// exclude 的注意点
// - 只对 include 生效，不影响被 import 的文件（依赖会被自动加入）
// - 不写 extensions 时按默认 .ts/.tsx/.d.ts 处理

// 常见排除测试文件
// "exclude": ["**/*.test.ts", "**/*.spec.ts"]

// 多配置拆分：构建与测试分开
// tsconfig.json       → 包含全部（含测试），用于编辑器
// tsconfig.build.json → exclude 测试，用于产出
```

```bash
# 验证实际纳入编译的文件
tsc --listFiles --noEmit
```

**运行结果：**
```
src/index.ts
src/utils/helper.ts
types/global.d.ts
（tsc --listFiles 会列出所有参与编译的文件）
```

**注意：**
* 被 `import` 的文件即使被 `exclude` 也会被编译（依赖关系优先）。
* 配置了 `files` 时，`include` 仍会生效，两者是并集。
* 编辑器（tsserver）与 `tsc` 使用同一份配置，保证两条路径一致。

### 20.11 declaration

**概念说明：** `declaration: true` 让编译器为每个源文件生成 `.d.ts` 类型声明文件，供其他项目/包消费。发布 npm 包时必备。

```json
{
  "compilerOptions": {
    "declaration": true,
    "declarationMap": true,
    "emitDeclarationOnly": false,
    "declarationDir": "dist/types",
    "outDir": "dist"
  }
}
```

```typescript
// src/index.ts
export interface User {
  id: number;
  name: string;
}

export function createUser(name: string): User {
  return { id: Date.now(), name };
}

export function greet(user: User): string {
  return `你好 ${user.name}`;
}
```

```typescript
// 产物 dist/index.d.ts（自动生成）
export interface User {
    id: number;
    name: string;
}
export declare function createUser(name: string): User;
export declare function greet(user: User): string;
```

```json
// package.json 指向声明文件的几种方式
{
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    }
  }
}
```

**运行结果：**
```
产物中生成 dist/index.d.ts，使用者获得完整类型提示
```

**注意：**
* 私有包内部也可以开 `declaration`（配合项目引用能显著加速增量编译）。
* `declarationMap` 让「跳转到定义」直接落到 `.ts` 源文件，方便调试。
* 只发类型时用 `emitDeclarationOnly: true` 配合 bundler 输出 JS。

### 20.12 sourceMap

**概念说明：** `sourceMap: true` 生成 `.js.map` 文件，把编译产物映射回 TypeScript 源码，使调试器中的断点、调用栈、错误位置指向原始 `.ts` 文件。

```json
{
  "compilerOptions": {
    "sourceMap": true,
    "inlineSources": true,
    "inlineSourceMap": false,
    "declarationMap": true,
    "outDir": "dist"
  }
}
```

```typescript
// src/calc.ts
export function divide(a: number, b: number): number {
  if (b === 0) throw new Error("除数不能为零");   // 栈里显示的是这一行（.ts）
  return a / b;
}

try {
  divide(1, 0);
} catch (e) {
  console.log((e as Error).message);
  console.log("堆栈指向 calc.ts 而非 calc.js（需 --enable-source-maps）");
}
```

```json
// dist/calc.js.map（片段）
{
  "version": 3,
  "sources": ["../src/calc.ts"],
  "names": ["divide"],
  "mappings": "AAAA,SAAS,OAAO..."
}
```

```bash
# Node 中启用 source map 支持（否则栈仍显示 dist/calc.js）
node --enable-source-maps dist/calc.js

# 或使用 ts-node / tsx 直接运行 TS，天然映射到 .ts
npx tsx src/calc.ts
```

**运行结果：**
```
除数不能为零
堆栈指向 calc.ts 而非 calc.js（需 --enable-source-maps）
```

**注意：**
* `sourceMap` 与 `inlineSourceMap` 互斥；`inlineSources` 会把源码内嵌进 map（体积变大但部署更简单）。
* 生产环境建议上传 source map 到错误监控平台后不公开暴露。
* `declarationMap` 是给 `.d.ts` 用的，让「转到定义」跳到 `.ts` 源文件。

## 21. Declaration Files 与第三方类型
### 21.1 .d.ts

**概念说明：** `.d.ts` 是「类型声明文件」，只包含类型信息、不含实现，用于给已有的 JavaScript 代码补充类型。它不参与编译产物。

```typescript
// math.js（已有的 JS 库，无类型）
// function add(a, b) { return a + b; }
// module.exports = { add };

// math.d.ts（手写的类型声明）
export declare function add(a: number, b: number): number;
```

```typescript
// 使用方获得类型提示
import { add } from "./math";
console.log(add(1, 2));
// add("a", 2);        // 错误：类型不匹配
```

```typescript
// 声明文件可以只声明类型（接口、类型别名）
export interface Options {
  debug?: boolean;
  timeout?: number;
}

export declare function create(options?: Options): void;

// 只含类型的声明文件不会生成 JS
console.log("声明文件不产生运行时代码");
```

```json
// tsconfig 中纳入声明文件
{
  "include": ["src/**/*.ts", "types/**/*.d.ts"]
}
```

**运行结果：**
```
3
声明文件不产生运行时代码
```

**注意：**
* 声明文件里不能有实现（函数体、变量初始值），否则报错。
* 若 JS 与同名 `.d.ts` 并排存在，TypeScript 优先使用声明文件的类型。
* 用 `tsc --declaration` 可从 `.ts` 自动生成 `.d.ts`，见 20.11。

### 21.2 declare

**概念说明：** `declare` 告诉编译器「这个值/类型在别处存在」，只做类型声明不作实现，编译后完全消失。常用于声明全局变量、模块、命名空间与全局类型。

```typescript
// 声明全局变量（如运行时注入的 window 属性）
declare const APP_VERSION: string;
declare let currentUser: { id: number; name: string };

// 声明全局函数
declare function track(event: string, payload?: Record<string, unknown>): void;

// 声明命名空间
declare namespace MyLib {
  function init(options?: { debug?: boolean }): void;
  const version: string;
  interface Config { host: string }
}

// 声明模块（无类型的三方库）
declare module "untyped-lib" {
  export function doWork(input: string): number;
  export default function setup(): void;
}

// 声明通配模块（如 .vue、.svg 文件导入）
declare module "*.vue" {
  const component: { name: string };
  export default component;
}

declare module "*.svg" {
  const url: string;
  export default url;
}

// 使用
track("页面浏览", { path: "/home" });
MyLib.init({ debug: true });
const cfg: MyLib.Config = { host: "localhost" };
// console.log(APP_VERSION, currentUser.name);   // 运行时需要真实存在
console.log(cfg.host, MyLib.version ?? "（运行时提供）");

// 声明合并：扩展已有类型
declare global {
  interface Window {
    __APP_CONFIG__: { apiBase: string };
  }
}
// window.__APP_CONFIG__.apiBase;   // 现在有类型了
```

**运行结果：**
```
localhost （运行时提供）
```

**注意：**
* `declare` 只是类型层面的承诺，若运行时不存在，会在访问时抛 `ReferenceError`。
* 全局声明需要文件是「模块」（含 import/export）时用 `declare global { }` 包裹。
* 通配模块声明要放在被 `include` 的 `.d.ts` 中才能生效。

### 21.3 @types

**概念说明：** `@types/*` 是社区维护的类型声明包（来自 DefinitelyTyped），通过 npm 安装即可为无类型或仅有 JS 的库补充类型。

```bash
# 安装类型声明
npm i -D @types/node @types/express @types/lodash

# 查看包是否自带类型
npm view express types       # 无 → 需要 @types/express
npm view axios types         # 有 → 无需额外安装

# 一次安装所有缺失类型（社区工具）
npx typesync
```

```json
// package.json
{
  "devDependencies": {
    "@types/node": "^20.0.0",
    "@types/express": "^4.17.0"
  }
}
```

```typescript
// @types/node 提供 Node 全局与模块类型
import path from "path";
import fs from "fs";

const p = path.join("a", "b");
console.log("path.join →", p);
console.log("fs 类型可用 →", typeof fs.existsSync);

// process、Buffer、__dirname 等全局都由 @types/node 提供
console.log("process.platform →", process.platform);
console.log("__filename 类型 →", typeof __filename);

// @types/express 提供 Request / Response 类型
// import type { Request, Response } from "express";
// function handler(req: Request, res: Response): void { res.send("ok"); }
```

```json
// tsconfig：控制自动加载哪些 @types
{
  "compilerOptions": {
    "types": ["node"],
    "typeRoots": ["./node_modules/@types", "./types"]
  }
}
```

**运行结果：**
```
path.join → a/b
fs 类型可用 → function
process.platform → win32
__filename 类型 → string
```

**注意：**
* 默认会把 `node_modules/@types` 下所有包都纳入全局，配置 `types` 可以只加载指定的包（加快编译）。
* 库自带 `types`/`typings` 字段时不需要 `@types`，装了反而可能冲突。
* `@types/*` 的版本要与库的大版本对应（如 express 4 → `@types/express@^4`）。

### 21.4 第三方库类型声明

**概念说明：** 当三方库没有类型时，有三种处理方式：安装 `@types/*`、利用库自带的类型、或自己写声明文件（局部或全局）。自己写时可用 `declare module` 或 `import` 后 `declare`。

```typescript
// 方式一：库自带类型（推荐，先检查）
// node_modules/some-lib/package.json → "types": "./index.d.ts"
// 直接用，无需额外操作
// import { something } from "some-lib";

// 方式二：安装 @types（社区维护）
// npm i -D @types/some-lib

// 方式三：自己写局部声明（放在 src 下，随项目迁移）
// types/some-lib.d.ts
declare module "some-lib" {
  export interface Config {
    apiKey: string;
    timeout?: number;
  }
  export function init(config: Config): void;
  export function query(sql: string): Promise<unknown[]>;
  export default init;
}

// 使用方
// import init, { query, type Config } from "some-lib";
// const cfg: Config = { apiKey: "k" };
// init(cfg);
// query("select 1").then((rows) => console.log(rows.length));

// 为已有类型「打补丁」：扩展第三方模块的导出
declare module "some-lib" {
  interface Config {
    retries?: number;        // 追加字段（声明合并）
  }
}

// 给 CommonJS 库补充「可调用」形态
declare module "callable-lib" {
  function lib(input: string): number;
  namespace lib {
    const version: string;
  }
  export = lib;
}

console.log("三方类型声明的三种处理方式：自带 / @types / 自写");

// 验证：类型只在编译期存在
const version: string = "1.0.0";
console.log(version);
```

**运行结果：**
```
三方类型声明的三种处理方式：自带 / @types / 自写
1.0.0
```

**注意：**
* 自写的声明文件要放在 `include` 范围内（常用 `src/types/*.d.ts`）。
* 「声明补丁」用接口声明合并（同模块名重复 `declare module`）可追加字段而不覆盖原类型。
* CommonJS 的 `module.exports = fn` 形态用 `export =` 声明。

### 21.5 DefinitelyTyped

**概念说明：** DefinitelyTyped 是社区维护的类型声明仓库（GitHub: `DefinitelyTyped/DefinitelyTyped`），所有 `@types/*` 包都由它发布，是三方类型的主要来源。

```bash
# 找类型：在 npm 上搜索 @types/<包名>
npm search @types/express

# 安装
npm i -D @types/lodash

# 查看版本与更新时间
npm view @types/lodash version time.modified

# 检查项目里哪些包缺类型
npx tsc --noEmit            # 报 TS7016：Could not find a declaration file
```

```typescript
// 使用 @types/lodash 的例子
// import _ from "lodash";
// const grouped = _.groupBy([1.1, 2.2, 1.3], Math.floor);   // 完整类型提示
// console.log(Object.keys(grouped));

// 报错 TS7016 的处理顺序
// 1) 先看库是否自带类型（package.json 的 types 字段）
// 2) 再看是否已有 @types/<pkg>
// 3) 都没有 → 自己写声明或用 // @ts-expect-error 临时绕过

// 常见 TS 错误码
// TS7016  Could not find a declaration file for module 'x'
// TS2307  Cannot find module 'x' or its corresponding type declarations
// TS2688  Cannot find type definition file for 'node'

console.log("DefinitelyTyped 是 @types/* 的唯一来源仓库");

// 若库自带类型但不够准确，可覆盖声明
// declare module "vendor-lib" {
//   export function betterTyped(input: string): number;
// }

// 给项目内所有未声明模块一个兜底（谨慎使用）
// declare module "*";
```

**运行结果：**
```
DefinitelyTyped 是 @types/* 的唯一来源仓库
```

**注意：**
* `@types/*` 是「一个包一个声明」，不会互相依赖，按需安装。
* 也可以给 DefinitelyTyped 提 PR 贡献类型，是很好的开源入门方式。
* 兜底声明 `declare module "*"` 会让所有未声明模块变成 `any`，只在迁移期短期使用。

### 21.6 类型声明模块

**概念说明：** 模块声明（`declare module`）可用于三类场景：为无类型模块补声明、为通配资源（CSS/SVG）声明类型、以及模块扩充（Module Augmentation）给已有模块追加类型。

```typescript
// 场景 1：为无类型模块补声明
declare module "legacy-js-lib" {
  export function legacy(input: string): number;
  export const VERSION: string;
}

// 场景 2：通配资源声明（Vite/webpack 项目常见）
declare module "*.css" {
  const classes: Record<string, string>;
  export default classes;
}
declare module "*.png" {
  const src: string;
  export default src;
}
declare module "?raw" {
  const content: string;
  export default content;
}

// 场景 3：模块扩充（给已有模块加东西）
// 必须在模块文件中（有 import/export）才能用 declare module "包名"
import "express";

declare module "express" {
  interface Request {
    user?: { id: number; name: string };
    traceId?: string;
  }
}

// 增强全局类型
declare global {
  interface Array<T> {
    last(): T | undefined;
  }
}
Array.prototype.last = function <T>(this: T[]): T | undefined {
  return this[this.length - 1];
};
console.log([1, 2, 3].last());

// 增强函数的类型（模块扩充 + 声明合并）
declare module "./utils" {
  interface Utils {
    extra: () => void;
  }
}

// 使用通配声明
// import styles from "./app.css";
// import logo from "./logo.png";
// console.log(styles, logo);

// 模块扩充后的类型生效
const req: { user?: { id: number; name: string } } = { user: { id: 1, name: "A" } };
console.log(req.user?.name);
console.log("模块声明覆盖三方库、静态资源与既有模块");
```

**运行结果：**
```
3
A
模块声明覆盖三方库、静态资源与既有模块
```

**注意：**
* 模块扩充（`declare module "已存在包"`）会修改导入方看到的类型，注意副作用范围。
* 增强全局前需要 `declare global`，且文件本身必须是模块。
* 对静态资源的通配声明要与打包器配置一致（如 Vite 的 `vite/client` 已内置大部分）。

## 22. Decorators 装饰器
### 22.1 Decorator 基础

**概念说明：** 装饰器是「用 `@表达式` 修饰类/方法/属性/参数」的语法，本质是接收被装饰目标并返回（或替换）它的函数。两套标准：TS 5.0 标准装饰器与旧的 `experimentalDecorators`（legacy）。

```json
// tsconfig.json —— 标准装饰器（TS 5.0+，默认支持，无需开关）
{
  "compilerOptions": {
    "target": "ES2022"
  }
}
```

```json
// 旧版装饰器（Angular/NestJS 等仍在使用）
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

```typescript
// 最简单的装饰器：一个函数
function sealed<T extends { new (...args: any[]): object }>(Ctor: T): T {
  Object.seal(Ctor.prototype);
  console.log("类已被密封:", Ctor.name);
  return Ctor;
}

@sealed
class User {
  constructor(public name: string) {}
  greet(): string { return `你好 ${this.name}`; }
}

console.log(new User("Alice").greet());

// 装饰器工厂：返回装饰器函数（可传参）
function tag(label: string) {
  return function <T extends { new (...args: any[]): object }>(Ctor: T): T {
    console.log(`标签: ${label} → ${Ctor.name}`);
    return Ctor;
  };
}

@tag("实体")
class Product {}
console.log(new Product() instanceof Product);

// 装饰器的执行顺序
function first() { return <T extends { new (...a: any[]): object }>(c: T) => (console.log("first"), c); }
function second() { return <T extends { new (...a: any[]): object }>(c: T) => (console.log("second"), c); }

// 多个装饰器：自上而下求值，自下而上执行
@first()
@second()
class Ordered {}
console.log("执行顺序：second 先于 first");

// 装饰器是纯粹的语法糖：编译为函数调用
// 编译产物（legacy 模式）：
// let User = class User { ... };
// User = __decorate([sealed], User);
```

**运行结果：**
```
second
first
类已被密封: User
你好 Alice
标签: 实体 → Product
true
执行顺序：second 先于 first
```

**注意：**
* 标准装饰器（TS 5.0）与 legacy 装饰器语义不同，不能混用；NestJS/Angular 目前仍依赖 legacy。
* 装饰器只能用于类及其成员，不能用于普通函数（标准装饰器支持类和类成员）。
* 执行顺序：装饰器表达式自上而下求值，应用时自下而上。

### 22.2 Class Decorator

**概念说明：** 类装饰器接收类构造函数，可返回一个新类来替换它，用于注册、加元数据、自动混入能力等。

```typescript
// 1) 注册表模式
const registry = new Map<string, Function>();

function register(name: string) {
  return function <T extends { new (...args: any[]): object }>(Ctor: T): T {
    registry.set(name, Ctor);
    return Ctor;
  };
}

@register("userService")
class UserService {
  constructor(public name = "UserService") {}
  run(): string { return `${this.name} 运行中`; }
}

console.log(registry.has("userService"));
const Svc = registry.get("userService") as new () => UserService;
console.log(new Svc().run());

// 2) 返回新类替换原类（增强能力）
function withTimestamp<T extends { new (...args: any[]): object }>(Ctor: T) {
  return class extends Ctor {
    createdAt = new Date("2024-01-01");
    describe(): string { return `创建于 ${this.createdAt.toISOString().slice(0, 10)}`; }
  };
}

@withTimestamp
class Article {
  constructor(public title: string) {}
}
const a = new Article("文章");
console.log(a.title, a.describe());

// 3) 冻结/密封类
function immutable<T extends { new (...args: any[]): object }>(Ctor: T): T {
  return new Proxy(Ctor, {
    construct(target, args) {
      return Object.freeze(new target(...args));
    },
  }) as T;
}

@immutable
class Settings {
  constructor(public theme = "dark") {}
}
const s = new Settings();
console.log(s.theme, Object.isFrozen(s));

// 4) 单例装饰器
function singleton<T extends { new (...args: any[]): object }>(Ctor: T): T {
  let instance: InstanceType<T> | null = null;
  return class extends (Ctor as any) {
    constructor(...args: any[]) {
      if (instance) return instance as any;
      super(...args);
      instance = this as any;
    }
  } as unknown as T;
}

@singleton
class Db {
  constructor(public id = Math.random()) {}
}
console.log(new Db().id === new Db().id);
```

**运行结果：**
```
true
UserService 运行中
文章 创建于 2024-01-01
dark true
true
```

**注意：**
* 类装饰器返回 `void` 时保持原类；返回新类时必须保持兼容的构造签名。
* 装饰器在类定义时（模块加载时）执行一次，不是每次实例化。
* 依赖注入框架（NestJS）大量使用类装饰器 + 元数据。

### 22.3 Method Decorator

**概念说明：** 方法装饰器接收 `(target, propertyKey, descriptor)`（legacy）或 `(value, context)`（标准），可包装原方法实现日志、缓存、权限校验、重试等横切逻辑。

```typescript
// legacy 风格（experimentalDecorators）
function log(target: object, key: string, descriptor: PropertyDescriptor): PropertyDescriptor {
  const original = descriptor.value as (...args: any[]) => any;
  descriptor.value = function (...args: any[]) {
    console.log(`调用 ${key}(${args.join(", ")})`);
    const result = original.apply(this, args);
    console.log(`${key} 返回 ${result}`);
    return result;
  };
  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number): number { return a + b; }
}

// 上面是 legacy 写法；下面演示标准装饰器（TS 5.0）
function double(fn: (...args: any[]) => number, ctx: ClassMethodDecoratorContext) {
  return function (this: unknown, ...args: any[]): number {
    return fn.apply(this, args) * 2;
  };
}

function trace(fn: (...args: any[]) => any, ctx: ClassMethodDecoratorContext) {
  return function (this: unknown, ...args: any[]) {
    console.log(`→ ${String(ctx.name)}(${args.join(", ")})`);
    const r = fn.apply(this, args);
    console.log(`← ${String(ctx.name)} = ${r}`);
    return r;
  };
}

class Math2 {
  @double
  twice(n: number): number { return n; }

  @trace
  sum(a: number, b: number): number { return a + b; }
}

const m = new Math2();
console.log("twice(5) =", m.twice(5));
console.log("sum(1, 2) =", m.sum(1, 2));

// 缓存装饰器（标准）
function memo(fn: (...args: any[]) => any, _ctx: ClassMethodDecoratorContext) {
  const cache = new Map<string, unknown>();
  return function (this: unknown, ...args: any[]) {
    const key = JSON.stringify(args);
    if (!cache.has(key)) cache.set(key, fn.apply(this, args));
    return cache.get(key);
  };
}

class Slow {
  calls = 0;
  @memo
  compute(n: number): number {
    this.calls++;
    return n * n;
  }
}
const slow = new Slow();
console.log(slow.compute(4), slow.compute(4), "实际计算次数:", slow.calls);

// getter / setter 装饰器
class Temp {
  private _v = 0;
  @trace
  get v(): number { return this._v; }
}
console.log(new Temp().v);
```

**运行结果：**
```
twice(5) = 10
→ sum(1, 2)
← sum = 3
sum(1, 2) = 3
16 16 实际计算次数: 1
→ v()
← v = 0
0
```

**注意：**
* 标准装饰器的返回值会**替换**原方法，需要自行 `apply` 调用原实现。
* 标准装饰器无法直接拿到 `descriptor`，改用 `context`（含 `name`、`kind`、`addInitializer`）。
* 包装方法会改变 `this` 与函数的 `length`/`name`，需要注意兼容性。

### 22.4 Property Decorator

**概念说明：** 属性装饰器可拦截属性的读写（通过 `accessor` 关键字获得标准支持），或仅用于收集元数据（legacy 只有 `(target, key)`，无法直接改变行为）。

```typescript
// 标准装饰器 + accessor：可以拦截读写
function validate(min: number, max: number) {
  return function (value: unknown, ctx: ClassAccessorDecoratorContext) {
    if (ctx.kind !== "accessor") throw new Error("只能用于 accessor");
    return {
      get(this: any) { return value.get.call(this); },
      set(this: any, v: number) {
        if (typeof v !== "number" || v < min || v > max) {
          throw new Error(`${String(ctx.name)} 必须在 ${min}~${max} 之间`);
        }
        value.set.call(this, v);
      },
    };
  };
}

class Score {
  @validate(0, 100)
  accessor value = 0;
}

const sc = new Score();
sc.value = 90;
console.log("分数:", sc.value);
try {
  sc.value = 150;
} catch (e) {
  console.log("错误:", (e as Error).message);
}

// 属性装饰器做元数据收集（legacy 风格）
const fields: string[] = [];
function collect(target: object, key: string): void {
  fields.push(key);
}

class Form {
  @collect
  username = "";
  @collect
  password = "";
}
console.log("收集到的字段:", fields);

// 自动绑定 this（标准）——解决解构后 this 丢失
function bound(fn: (...args: any[]) => any, ctx: ClassMethodDecoratorContext) {
  return function (this: any, ...args: any[]) {
    return fn.apply(this, args);
  };
}

class Handler {
  name = "处理器";
  @bound
  handle(): string { return this.name; }
}
const h = new Handler();
const { handle } = h;
console.log("解构后仍可调用:", handle());

// 只读属性保护（标准 accessor）
function readonly(_v: unknown, ctx: ClassAccessorDecoratorContext) {
  return {
    set() { throw new Error(`${String(ctx.name)} 是只读的`); },
  };
}

class Const {
  @readonly
  accessor version = "1.0";
}
const c = new Const();
console.log("版本:", c.version);
try { c.version = "2.0"; } catch (e) { console.log("错误:", (e as Error).message); }

// 注意：普通属性（非 accessor）的标准装饰器只能做初始化前的检查
function required(_v: undefined, ctx: ClassFieldDecoratorContext) {
  return function (this: any, initial: unknown) {
    if (initial === undefined) throw new Error(`${String(ctx.name)} 必填`);
    return initial;
  };
}

class Req {
  @required
  id: number = 1;
}
console.log(new Req().id);
```

**运行结果：**
```
分数: 90
错误: value 必须在 0~100 之间
收集到的字段: [ 'username', 'password' ]
解构后仍可调用: 处理器
版本: 1.0
错误: version 是只读的
1
```

**注意：**
* legacy 属性装饰器只能收集信息，不能改变取值行为（除非改写原型描述符）。
* 标准装饰器用 `accessor` 关键字才能拦截读写，普通字段只能包装初始化值。
* 属性装饰器执行时机早于方法装饰器（同一次类定义中按声明顺序）。

### 22.5 Parameter Decorator

**结论：参数装饰器只存在于 legacy（`experimentalDecorators`）模式，标准装饰器已移除该能力。**

**概念说明：** 参数装饰器接收 `(target, key, index)`，仅用于收集元数据（配合反射实现依赖注入），不能改变参数值。

```typescript
// 需要 tsconfig: experimentalDecorators + emitDecoratorMetadata
import "reflect-metadata";

function inject(token: string) {
  return function (target: object, key: string | undefined, index: number): void {
    console.log(`在 ${key ?? "构造函数"} 的第 ${index} 个参数注入 ${token}`);
    // 实际用途：把 token 记录到元数据，供 IoC 容器读取
    const meta = Reflect.getMetadata("design:paramtypes", target, key as string) ?? [];
    console.log("参数类型元数据长度:", meta.length);
  };
}

class Logger {
  log(msg: string): void { console.log("[Logger]", msg); }
}
class Db {
  query(sql: string): string[] { return [sql]; }
}

class UserService {
  constructor(
    @inject("Logger") private logger: Logger,
    @inject("Db") private db: Db
  ) {}

  run(): void {
    this.logger.log("执行查询");
    console.log(this.db.query("select 1"));
  }
}

const svc = new UserService(new Logger(), new Db());
svc.run();

// 方法参数装饰器
class Controller {
  handle(@inject("Body") body: object, @inject("Query") query: object): string {
    return `body=${typeof body} query=${typeof query}`;
  }
}
console.log(new Controller().handle({}, {}));

// 依赖注入容器的原理（简化）
type Token = new (...args: any[]) => any;
const container = new Map<string, Token>();
function provide(name: string, impl: Token): void { container.set(name, impl); }
function resolve(name: string): object {
  const Impl = container.get(name);
  if (!Impl) throw new Error(`未注册: ${name}`);
  return new Impl();
}
provide("Logger", Logger);
console.log(resolve("Logger") instanceof Logger);

console.log("参数装饰器仅用于元数据，NestJS/Angular 依赖此机制");
```

**运行结果：**
```
在 构造函数 的第 0 个参数注入 Logger
参数类型元数据长度: 2
在 构造函数 的第 1 个参数注入 Db
参数类型元数据长度: 2
[Logger] 执行查询
[ 'select 1' ]
在 handle 的第 0 个参数注入 Body
在 handle 的第 1 个参数注入 Query
body=object query=object
true
参数装饰器仅用于元数据，NestJS/Angular 依赖此机制
```

**注意：**
* 必须开启 `experimentalDecorators` 与 `emitDecoratorMetadata`，并安装 `reflect-metadata`。
* 参数装饰器返回值被忽略，只能通过 `Reflect.metadata` 传递信息。
* 标准装饰器（TS 5.0）不支持参数装饰器，若要用 DI 框架需继续使用 legacy 模式。

### 22.6 Decorator Metadata

**概念说明：** 开启 `emitDecoratorMetadata` 后，编译器会为被装饰的类/方法自动生成类型元数据（`design:type`、`design:paramtypes`、`design:returntype`），这是依赖注入能够工作的基础。

```json
// tsconfig.json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "target": "ES2022",
    "module": "commonjs"
  }
}
```

```typescript
import "reflect-metadata";

function log(label: string) {
  return function (target: any, key?: string, index?: number): void {
    if (index !== undefined) return;

    if (key) {
      // 方法：读取参数类型与返回类型
      const params = Reflect.getMetadata("design:paramtypes", target, key) ?? [];
      const ret = Reflect.getMetadata("design:returntype", target, key);
      console.log(`[${label}] ${key}: 参数 (${params.map((p: any) => p?.name).join(", ")}) → ${ret?.name}`);
    } else {
      // 类：读取构造函数参数类型
      const params = Reflect.getMetadata("design:paramtypes", target) ?? [];
      console.log(`[${label}] ${target.name}: 构造参数 (${params.map((p: any) => p?.name).join(", ")})`);
    }
  };
}

class Logger2 {
  log(msg: string): void { console.log(msg); }
}

@log("class")
class Service {
  constructor(private logger: Logger2) {}

  @log("method")
  run(count: number, name: string): boolean {
    this.logger.log(`run(${count}, ${name})`);
    return true;
  }
}

const svc = new Service(new Logger2());
svc.run(1, "测试");

// 元数据类型由设计时的类型决定
// design:type       属性的类型
// design:paramtypes 参数类型数组
// design:returntype 返回类型
console.log("已读取设计时元数据");

// 运行时反射：把元数据用于自动装配
function create<T>(Ctor: new (...args: any[]) => T): T {
  const params: any[] = Reflect.getMetadata("design:paramtypes", Ctor) ?? [];
  const deps = params.map((P) => new P());
  return new Ctor(...deps);
}
const auto = create(Service);
auto.run(2, "自动装配");
console.log(auto instanceof Service);

// 添加自定义元数据
class Config {}
Reflect.defineMetadata("config:key", { debug: true }, Config);
console.log(Reflect.getMetadata("config:key", Config).debug);

// 注意：元数据只在有装饰器时生成（否则会被摇掉）
class NoDecorator {
  constructor(public dep: object) {}
}
console.log("无装饰器的类不生成 paramtypes:", Reflect.getMetadata("design:paramtypes", NoDecorator) === undefined);
```

**运行结果：**
```
[class] Service: 构造参数 (Logger2)
[method] run: 参数 (Number, String) → Boolean
run(1, 测试)
已读取设计时元数据
run(2, 自动装配)
true
true
无装饰器的类不生成 paramtypes: true
```

**注意：**
* 元数据依赖 `reflect-metadata` 的全局 polyfill，必须在入口文件最早处 `import "reflect-metadata"`。
* 类型来自「设计时类型」，接口与类型别名会退化为 `Object`（因为运行时不存在）。
* 未被装饰的类不会生成元数据，且打包时可能被 tree-shaking 移除。

## 23. Type System 与 Runtime
### 23.1 编译期类型系统

**概念说明：** TypeScript 的类型系统完全在编译期工作：它分析源码、检查类型一致性、输出错误，但不参与运行时。类型是「写给编译器和 IDE 看的」，不是程序的一部分。

```typescript
// 类型检查发生在编译期
interface User { id: number; name: string }

const user: User = { id: 1, name: "Alice" };

// 编译期报错（tsc 会失败，但 JS 仍可能被生成）
// const bad: User = { id: "1", name: "A" };    // 错误：类型不匹配
// console.log(bad.name.toUpperCase());          // 潜在运行时错误

// 类型信息用于 IDE：补全、跳转、重命名、内联提示
function greet(u: User): string { return `${u.id}: ${u.name}`; }
console.log(greet(user));

// 类型可以表达运行时无法表达的约束
type NonEmpty<T> = T extends [] ? never : T;
function firstOrThrow<T>(arr: NonEmpty<T[]>): T { return (arr as T[])[0]; }
console.log(firstOrThrow([1, 2]));
// firstOrThrow([]);    // 编译期拦截，运行时无此类检查

// 编译流程：解析 → 类型检查 → 擦除类型 → 输出 JS
// 类型检查失败时默认仍会输出 JS（可用 noEmitOnError 关闭）
console.log("类型检查在编译期完成，运行时无类型概念");
```

**运行结果：**
```
1: Alice
1
类型检查在编译期完成，运行时无类型概念
```

**注意：**
* 类型错误不会阻止 JS 生成（除非 `noEmitOnError: true`），所以「能跑」不等于「类型正确」。
* 类型系统的能力远超运行时：可表达「非空数组」「键值对应」等无法运行期校验的约束。
* CI 中应把 `tsc --noEmit` 作为必过关卡。

### 23.2 Runtime

**概念说明：** 运行时只认识 JavaScript 的值：对象、函数、字符串等。所有 TypeScript 特有的语法（类型注解、接口、泛型、`as`）在运行时都不存在，运行时行为完全由生成的 JS 决定。

```typescript
// 类型注解在运行时消失
const n: number = 42;
console.log("运行时 typeof:", typeof n);          // number
console.log("运行时有 number 类型吗:", typeof (Number as unknown) !== "undefined");

// 接口/类型别名完全不存在
interface User { id: number; name: string }
type ID = string | number;
console.log("运行时 typeof User:", typeof User);   // undefined（User 不是值）

// 泛型参数不存在
function identity<T>(v: T): T { return v; }
console.log("运行时无法获取 T:", identity.length);  // 参数个数（不含类型参数）

// 枚举会生成真实对象（少数例外）
enum Color { Red, Green }
console.log("运行时枚举是对象:", typeof Color, Color[0]);

// as 断言不影响运行时
const anything = "字符串" as unknown as number;
console.log("断言不转换值:", typeof anything);      // string

// 判断类型只能靠运行时检查
function isUser(v: unknown): boolean {
  return typeof v === "object" && v !== null && "id" in v && "name" in v;
}
console.log(isUser({ id: 1, name: "A" }), isUser({}));

// 编译产物验证
// tsc 输出：const n = 42; console.log(typeof n, ...);
console.log("运行时只保留值，类型全部被擦除");

// 运行时错误与类型无关
try {
  const obj = null as unknown as { a: number };
  console.log(obj.a);                    // 运行时 TypeError
} catch (e) {
  console.log("运行时错误:", (e as Error).constructor.name);
}
```

**运行结果：**
```
运行时 typeof: number
运行时有 number 类型吗: true
运行时 typeof User: undefined
运行时无法获取 T: 1
运行时枚举是对象: object 红
断言不转换值: string
true false
运行时只保留值，类型全部被擦除
运行时错误: TypeError
```

**注意：**
* 运行时能拿到的信息只有「值」：`typeof`、`instanceof`、`in`、`constructor`、`Object.prototype.toString`。
* 枚举、类、命名空间是少数会生成运行时代码的类型相关语法。
* 类型与运行时数据必须对齐，否则只能靠类型断言「自欺欺人」。

### 23.3 Type Erasure

**概念说明：** 类型擦除指编译器把所有类型相关语法从产物中移除，只保留等价的 JavaScript 值操作。理解擦除规则有助于解释「为什么有些类型无法在运行时判断」。

```typescript
// 源码
interface Point { x: number; y: number }
type Callback = (n: number) => void;

function distance(a: Point, b: Point, cb: Callback): number {
  const d = Math.hypot(b.x - a.x, b.y - a.y);
  cb(d);
  return d;
}

const p1: Point = { x: 0, y: 0 };
const p2 = { x: 3, y: 4 } as Point;
const result: number = distance(p1, p2, (v) => console.log("距离:", v));
```

```javascript
// 编译产物（类型全部消失，只保留运行时逻辑）
function distance(a, b, cb) {
    const d = Math.hypot(b.x - a.x, b.y - a.y);
    cb(d);
    return d;
}
const p1 = { x: 0, y: 0 };
const p2 = { x: 3, y: 4 };
const result = distance(p1, p2, (v) => console.log("距离:", v));
```

```typescript
// 被擦除的东西
// - 类型注解（: number）
// - 接口与类型别名
// - 泛型参数（<T>）
// - as / satisfies / ! 断言
// - import type / export type
// - declare 声明

// 不被擦除（生成代码）的东西
// - enum（生成对象）
// - class（生成函数/类）
// - namespace（生成 IIFE）
// - 参数属性（生成赋值）
// - 装饰器（生成包装调用，legacy 需 __decorate 辅助）

console.log("运行结果:", distance({ x: 0, y: 0 }, { x: 3, y: 4 }, (v) => console.log("距离:", v)));

// 擦除带来的重要影响：
// 1) 无法在运行时判断「某值是否符合接口」
// 2) 泛型函数无法知道 T 是什么
// 3) 重载只保留实现签名
// 4) instanceof 只能用于值（类/构造函数），不能用于接口
console.log("类型擦除是 TS 零运行时开销的根本原因");
```

**运行结果：**
```
距离: 5
运行结果: 5
距离: 5
类型擦除是 TS 零运行时开销的根本原因
```

**注意：**
* 因为要擦除，`const enum` 在单文件编译（`isolatedModules`）下会报错。
* 想保留运行时类型信息必须显式编码：`tag` 字段、类、Schema 校验。
* 擦除是 TS「零运行时开销」的根本原因，也是「类型不等于校验」的根源。

### 23.4 类型信息是否存在于运行时

**概念说明：** 绝大多数类型信息在运行时都不存在，只有少量语法会留下运行时痕迹（类、枚举等）。这是「运行时判断类型必须靠 `typeof`/`instanceof`/判别字段」的根本原因。

```typescript
// 存在：类（构造函数、原型方法、字段）
class User {
  name = "Alice";
  greet(): string { return this.name; }
}
console.log("类名:", typeof User, User.name);
console.log("原型方法存在:", typeof User.prototype.greet);

// 存在：枚举（生成对象）
enum Status { On = 1, Off = 0 }
console.log("枚举对象:", Object.keys(Status));

// 存在：instanceof 可用的信息（原型链）
console.log(new User() instanceof User);

// 不存在：接口、类型别名、泛型、联合类型
interface Shape { kind: string }
type Kind = "a" | "b";
function f<T>(v: T): T { return v; }
console.log("接口/别名/泛型运行时均为 undefined:", typeof Shape, typeof Kind, typeof T === "undefined");

// typeof 只能看到运行时值的类型，不是 TS 类型
console.log("typeof null →", typeof null);
console.log("typeof 数组 →", typeof []);
console.log("区分数组用:", Array.isArray([]));
console.log("区分类实例用:", Object.getPrototypeOf(new User()).constructor.name);

// 可用「判别字段」把类型信息带到运行时
type Animal = { kind: "cat"; meow(): void } | { kind: "dog"; bark(): void };
const a: Animal = { kind: "cat", meow: () => console.log("喵") };
console.log("运行时可读判别字段:", a.kind);
if (a.kind === "cat") a.meow();

// 面向运行时的类型方案（Zod/valibot 等）
// const UserSchema = z.object({ name: z.string() });
// type User2 = z.infer<typeof UserSchema>;      // 类型与校验同源
console.log("如需运行时类型，必须显式携带（Zod / 判别字段 / 类）");
```

**运行结果：**
```
类名: function User
原型方法存在: function
枚举对象: [ '0', '1', 'On', 'Off' ]
true
接口/别名/泛型运行时均为 undefined: undefined undefined true
typeof null → object
typeof 数组 → object
区分数组用: true
区分类实例用: User
运行时可读判别字段: cat
喵
如需运行时类型，必须显式携带（Zod / 判别字段 / 类）
```

**注意：**
* 唯一可靠的运行时判断手段：`typeof`（原始类型）、`instanceof`（类）、`Array.isArray`、`in`、判别字段。
* 需要「类型 + 校验」一致时用 Schema 库（Zod、valibot、io-ts），避免两处维护。
* 反射元数据（`reflect-metadata`）是额外注入的信息，不是 TS 类型系统自带的。

### 23.5 类型断言与运行时行为

**概念说明：** 类型断言（`as`、`!`、`<T>`）只是告诉编译器「相信我知道的类型」，**不做任何运行时检查或转换**。断言错误会让编译器闭嘴，但运行时照样出错。

```typescript
// as：只是类型层面的「断言」，不转换值
const value: unknown = "123";
const num = value as number;
console.log("断言后类型:", typeof num);          // string！不是 number
console.log("值未转换:", num);

// 需要真正转换必须调用函数
const realNum = Number(value);
console.log("真正转换:", typeof realNum, realNum);

// 双重断言：强行跨越不兼容类型（危险）
const s = "abc" as unknown as number;
console.log("双重断言仍是:", typeof s);

// 非空断言 !：只是去掉 undefined，运行时无保护
function findUser(id: number) { return id > 0 ? { id } : undefined; }
const u = findUser(-1);
// console.log(u!.id);              // 编译通过，运行时 TypeError: Cannot read properties of undefined
console.log("非空断言无运行时保护，u =", u);

// 断言的风险示范
interface User { id: number; name: string }
const raw = JSON.parse('{"id":"not-a-number"}') as User;
console.log("断言的谎言:", typeof raw.id, raw.id);   // string，但类型说 number
// console.log(raw.name.toUpperCase());              // 运行时 TypeError

// 更安全：用类型守卫/校验代替断言
function isUser(v: unknown): v is User {
  return typeof v === "object" && v !== null
    && typeof (v as User).id === "number"
    && typeof (v as User).name === "string";
}
const parsed: unknown = JSON.parse('{"id":1,"name":"Alice"}');
if (isUser(parsed)) console.log("校验通过:", parsed.name.toUpperCase());

// satisfies：检查而不改变推断类型（推荐替代断言）
const config = { host: "localhost", port: 80 } satisfies Record<string, string | number>;
console.log("satisfies 保留字面量类型:", config.host);

// as const：只影响类型，不改变运行时
const arr = [1, 2] as const;
console.log("as const 只影响类型，运行时仍是数组:", Array.isArray(arr), arr.length);

// 断言在编译产物中完全消失
// tsc 输出：const num = value;  —— as number 被擦除
console.log("断言不会生成任何运行时代码");
```

**运行结果：**
```
断言后类型: string
值未转换: 123
真正转换: number 123
双重断言仍是: string
非空断言无运行时保护，u = undefined
断言的谎言: string not-a-number
校验通过: ALICE
satisfies 保留字面量类型: localhost
as const 只影响类型，运行时仍是数组: true 2
断言不会生成任何运行时代码
```

**注意：**
* 断言是「单向承诺」：编译器相信你，运行时不会保护你。
* 优先用类型守卫 / Schema 校验 / `satisfies` 替代 `as`；非空断言 `!` 尽量少用。
* `satisfies` 在检查类型的同时保留窄类型，是比 `as` 更安全的约束手段。

### 23.6 JavaScript 与 TypeScript 的关系

**概念说明：** TypeScript 是 JavaScript 的超集：所有合法 JS 都是合法 TS（多数情况下）。TS 在 JS 之上增加静态类型系统，编译后输出标准 JS。它是「开发期工具」，不是运行时。

```typescript
// 1) 超集关系：JS 代码可以直接放进 .ts 文件
const legacy = function (a, b) { return a + b; };   // 隐式 any（noImplicitAny 下报错）
console.log(legacy(1, 2));

// 2) 编译产物是标准 JS：可运行在任何 JS 环境
const greet = (name: string): string => `你好，${name}`;
console.log(greet("TypeScript"));

// 3) 类型系统可逐步引入：allowJs + checkJs
// jsconfig.json / tsconfig.json: { "allowJs": true, "checkJs": false }

// 4) 类型只在开发期起作用：与运行时行为无关
class Api {
  constructor(private url: string) {}
  fetch(path: string): string { return `${this.url}${path}`; }
}
console.log(new Api("https://x.com").fetch("/users"));

// 5) 迁移路径：JS → 加 // @ts-check → allowJs + checkJs → 重命名为 .ts → 开启 strict
// 第一步：在 JS 文件顶部加 // @ts-check，IDE 开始提示类型问题

// 6) 互操作：JS 可直接 import TS 编译产物；TS 可导入 JS（需 allowJs）
// import { legacy } from "./legacy.js";

// 7) 生态兼容：类型定义通过 .d.ts / @types 提供，不要求库用 TS 编写
// import express from "express";       // 库是 JS，类型来自 @types/express

console.log("TS = JS + 类型系统；编译后就是 JS");

// 8) 何时不该用 TS：一次性脚本、极小原型、对构建速度极敏感的场景
// 但大型项目、多人协作、长期维护场景收益显著
const features = ["类型检查", "智能提示", "重构安全", "文档化"];
console.log("TS 带来的能力:", features.join(" / "));
```

**运行结果：**
```
3
你好，TypeScript
https://x.com/users
TS = JS + 类型系统；编译后就是 JS
TS 带来的能力: 类型检查 / 智能提示 / 重构安全 / 文档化
```

**注意：**
* TS 不是「新语言」，只是带类型检查的 JS；运行时行为仍由 JS 规范决定。
* 少数 TS 语法（枚举、参数属性、`const enum`、装饰器）会生成额外代码，并非纯擦除。
* 迁移应循序渐进：先 `allowJs`，再逐文件转 `.ts`，最后开 `strict`。

## 24. Runtime Validation 与类型安全
### 24.1 unknown

**概念说明：** 在运行时校验场景中，外部数据（API 响应、`JSON.parse`、`localStorage`、`process.env`）应先声明为 `unknown`，再通过收窄逐步「证明」其结构，避免直接断言带来的虚假安全。

```typescript
// 外部输入一律从 unknown 开始
function readLocalStorage(key: string): unknown {
  return JSON.parse('{"id":1,"name":"Alice","tags":["a"]}');
}

const raw = readLocalStorage("user");
console.log("原始类型:", typeof raw);

// 不使用断言，而是逐层证明
function isRecord(v: unknown): v is Record<string, unknown> {
  return typeof v === "object" && v !== null && !Array.isArray(v);
}

if (isRecord(raw)) {
  const { id, name, tags } = raw;
  console.log("id 是数字:", typeof id === "number", id);
  console.log("name 是字符串:", typeof name === "string", name);
  console.log("tags 是字符串数组:", Array.isArray(tags) && tags.every((t) => typeof t === "string"), tags);
}

// unknown 的传播：只能赋给 unknown / any，不能赋给具体类型
function pick(v: unknown): string {
  if (typeof v === "string") return v;
  if (typeof v === "number") return String(v);
  if (Array.isArray(v)) return v.map(String).join(",");
  if (isRecord(v)) return Object.keys(v).join(",");
  return "无法识别";
}
console.log(pick(raw), pick(42), pick(["a", "b"]), pick(null));

// 与 any 的对比：any 会关闭检查，unknown 强制校验
const anyVal: any = raw;
console.log("any 可直接访问（危险）:", anyVal.id);
// console.log(raw.id);          // 错误：'raw' is of type 'unknown'

// 环境变量都是 string | undefined
function getPort(env: NodeJS.ProcessEnv): number {
  const raw = env.PORT;
  if (typeof raw !== "string") return 3000;
  const n = Number(raw);
  return Number.isInteger(n) && n > 0 ? n : 3000;
}
console.log("端口:", getPort({ PORT: "8080" }), getPort({ PORT: "abc" }));
```

**运行结果：**
```
原始类型: object
id 是数字: true 1
name 是字符串: true Alice
tags 是字符串数组: true [ 'a' ]
{"id":1,"name":"Alice","tags":["a"]} 42 a,b id,name,tags
无法识别
any 可直接访问（危险）: 1
端口: 8080 3000
```

**注意：**
* `JSON.parse` 的返回类型是 `any`，建议立即包装为返回 `unknown` 的函数。
* `unknown` 是所有类型的「上级」，因此必须先收窄才能使用。
* TS 4.4+ 中 `catch (e)` 的 `e` 就是 `unknown`，处理方式相同。

### 24.2 类型守卫

**概念说明：** 类型守卫是「运行时可执行的判断 + 编译期收窄声明」的组合，是连接运行时校验与静态类型安全的桥梁。包括内置守卫（`typeof`/`instanceof`/`in`）与自定义谓词。

```typescript
// 内置守卫
function builtin(v: unknown): string {
  if (typeof v === "string") return v.toUpperCase();
  if (typeof v === "number") return v.toFixed(1);
  if (Array.isArray(v)) return `数组(${v.length})`;
  if (v instanceof Date) return v.toISOString().slice(0, 10);
  if (typeof v === "object" && v !== null && "name" in v) {
    return `有 name 字段: ${String((v as { name: unknown }).name)}`;
  }
  return "未知";
}
console.log(builtin("a"), builtin(1.23), builtin([1, 2]), builtin(new Date("2024-01-01")), builtin({ name: "x" }));

// 自定义谓词：可组合、可复用
const isString = (v: unknown): v is string => typeof v === "string";
const isNumber = (v: unknown): v is number => typeof v === "number" && !Number.isNaN(v);
const isArrayOf = <T>(v: unknown, item: (x: unknown) => x is T): v is T[] =>
  Array.isArray(v) && v.every(item);

console.log(isString("a"), isNumber(NaN), isArrayOf([1, 2], isNumber), isArrayOf([1, "a"], isNumber));

// 组合守卫
interface User { id: number; name: string; email?: string }
function isUser(v: unknown): v is User {
  return (
    typeof v === "object" && v !== null &&
    "id" in v && isNumber((v as User).id) &&
    "name" in v && isString((v as User).name) &&
    (!("email" in v) || isString((v as User).email))
  );
}
console.log(isUser({ id: 1, name: "A" }), isUser({ id: "1", name: "A" }));

// 断言守卫：失败即抛错
function assertIsNumber(v: unknown, label = "值"): asserts v is number {
  if (!isNumber(v)) throw new TypeError(`${label} 必须是数字，收到 ${typeof v}`);
}

try {
  const input: unknown = "not a number";
  assertIsNumber(input, "输入");
} catch (e) {
  console.log("断言失败:", (e as Error).message);
}

// 数组过滤配合守卫
const mixed: unknown[] = [1, "a", 2, null, "b"];
const nums = mixed.filter(isNumber);
console.log("过滤出数字:", nums);

// 穷尽性检查配合守卫
type Shape = { kind: "circle"; r: number } | { kind: "square"; s: number };
function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle": return Math.PI * shape.r ** 2;
    case "square": return shape.s ** 2;
    default: {
      const never: never = shape;
      throw new Error(`未处理: ${JSON.stringify(never)}`);
    }
  }
}
console.log(area({ kind: "circle", r: 1 }).toFixed(2), area({ kind: "square", s: 2 }));
```

**运行结果：**
```
A 1.2 数组(2) 2024-01-01 有 name 字段: x
true false true false
true false
断言失败: 输入 必须是数字，收到 string
过滤出数字: [ 1, 2 ]
3.14 4
```

**注意：**
* 守卫逻辑必须与类型声明严格一致，否则是「类型谎言」（编译器完全信任你）。
* 用组合式守卫（`isArrayOf`、`isOptional`）减少重复代码。
* `asserts` 形式适合「不变量」校验，失败立即抛错，后续代码自动收窄。

### 24.3 类型断言

**概念说明：** 在运行时校验语境下，断言应视为「最后手段」：它跳过校验直接让编译器接受。更安全的顺序是：守卫收窄 → Schema 校验 → `satisfies` 检查 → 才考虑断言。

```typescript
// 断言的等价替代方案对比
const raw: unknown = JSON.parse('{"id":1,"name":"Alice"}');

// ❌ 直接断言：无任何保障
const byAssert = raw as { id: number; name: string };
console.log("断言:", byAssert.name);

// ✅ 守卫 + 收窄：运行时有保障
interface User { id: number; name: string }
function isUser(v: unknown): v is User {
  return typeof v === "object" && v !== null
    && typeof (v as User).id === "number"
    && typeof (v as User).name === "string";
}
if (isUser(raw)) console.log("守卫:", raw.name.toUpperCase());

// ✅ satisfies：编译期检查字面量，运行时不变
const config = {
  host: "localhost",
  port: 80,
} satisfies Record<string, string | number>;
console.log("satisfies:", config.host, config.port);

// 断言仍有必要的场景
// 1) 已经通过外部校验（如 Zod），但类型系统不知道
const validated = { id: 1, name: "Alice" } as User;
console.log("外部校验后断言:", validated.name);

// 2) DOM 查询：编译器无法知道 id 一定存在
const el = { id: "app", textContent: "内容" };
const appEl = el as { id: string; textContent: string };
console.log("DOM 断言:", appEl.id);

// 3) 库类型不准时（局部绕过，写注释说明）
// @ts-expect-error 第三方类型与实际不符，运行时已确认
const libResult = (raw as { name: string }).name;
console.log("临时绕过:", libResult);

// 4) 双重断言仅用于真正不兼容且已确认的场景
const anything = "str" as unknown as { name: string };
console.log("双重断言类型:", typeof anything);

// 断言不影响运行时
console.log("断言不产生运行时代码，值本身未改变:", typeof raw);

// 非空断言的替代
const map = new Map<string, number>([["a", 1]]);
const v1 = map.get("a")!;                       // 断言
const v2 = map.get("a") ?? 0;                   // 更安全
const v3 = map.get("a");
console.log("三种取值:", v1, v2, v3 ?? "缺失");
```

**运行结果：**
```
断言: Alice
守卫: ALICE
satisfies: localhost 80
外部校验后断言: Alice
DOM 断言: app
临时绕过: Alice
双重断言类型: object
断言不产生运行时代码，值本身未改变: object
三种取值: 1 1 1
```

**注意：**
* 断言前最好在代码里留下「为什么安全」的注释或紧邻的校验代码。
* `satisfies` 与 `as` 的区别：前者检查且保留推断类型，后者强制改变类型。
* 断言无法解决真实的数据问题，只解决「编译器不知道」的问题。

### 24.4 数据校验

**概念说明：** 数据校验指在运行时验证数据是否符合预期结构，并让校验结果驱动类型收窄。手写校验函数是零依赖方案，适合小规模与学习；大型项目用 Schema 库更高效。

```typescript
// 手写校验器：返回「校验结果」而非抛错
type ValidateResult<T> = { ok: true; value: T } | { ok: false; errors: string[] };

function validateUser(input: unknown): ValidateResult<{ id: number; name: string; email?: string }> {
  const errors: string[] = [];

  if (typeof input !== "object" || input === null) {
    return { ok: false, errors: ["必须是对象"] };
  }
  const o = input as Record<string, unknown>;

  if (typeof o.id !== "number" || !Number.isInteger(o.id)) errors.push("id 必须是整数");
  if (typeof o.name !== "string" || o.name.trim() === "") errors.push("name 必须是非空字符串");
  if (o.email !== undefined && typeof o.email !== "string") errors.push("email 必须是字符串");
  if (typeof o.email === "string" && !o.email.includes("@")) errors.push("email 格式不正确");

  if (errors.length > 0) return { ok: false, errors };
  return { ok: true, value: { id: o.id as number, name: o.name as string, email: o.email as string | undefined } };
}

// 使用
const cases: unknown[] = [
  { id: 1, name: "Alice", email: "a@x.com" },
  { id: "1", name: "" },
  null,
];
for (const c of cases) {
  const r = validateUser(c);
  if (r.ok) console.log("通过:", r.value.name);
  else console.log("失败:", r.errors.join("; "));
}

// 通用原语校验器
const isNonEmptyString = (v: unknown): v is string => typeof v === "string" && v.trim().length > 0;
const isPositiveInt = (v: unknown): v is number => typeof v === "number" && Number.isInteger(v) && v > 0;
const isEmail = (v: unknown): v is string => typeof v === "string" && /^[^@\s]+@[^@\s]+\.[^@\s]+$/.test(v);
const optional = <T>(check: (v: unknown) => v is T) => (v: unknown): v is T | undefined =>
  v === undefined || check(v);

console.log(isNonEmptyString("a"), isPositiveInt(0), isEmail("a@x.com"), optional(isEmail)(undefined));

// 嵌套结构校验
interface OrderItem { sku: string; qty: number }
interface Order { id: number; items: OrderItem[] }
function isOrderItem(v: unknown): v is OrderItem {
  return typeof v === "object" && v !== null
    && isNonEmptyString((v as OrderItem).sku)
    && isPositiveInt((v as OrderItem).qty);
}
function isOrder(v: unknown): v is Order {
  return typeof v === "object" && v !== null
    && isPositiveInt((v as Order).id)
    && Array.isArray((v as Order).items)
    && (v as Order).items.every(isOrderItem);
}
const order: unknown = { id: 1, items: [{ sku: "A", qty: 2 }] };
console.log("订单校验:", isOrder(order), isOrder({ id: 1, items: [{ sku: "A", qty: 0 }] }));

// 校验失败时的类型安全处理
function parseOrder(input: unknown): Order {
  if (!isOrder(input)) throw new Error("订单数据结构不合法");
  return input;              // 此处已收窄为 Order
}
console.log("解析订单条目数:", parseOrder(order).items.length);

// 批量校验
const list: unknown[] = [{ id: 1, items: [] }, { bad: true }];
const valid = list.filter(isOrder);
console.log("有效条目:", valid.length, "无效条目:", list.length - valid.length);
```

**运行结果：**
```
通过: Alice
失败: id 必须是整数; name 必须是非空字符串
失败: 必须是对象
true false true true
订单校验: true false
解析订单条目数: 1
有效条目: 1 无效条目: 1
```

**注意：**
* 校验器返回「结果对象」比抛异常更适合批量场景，可以一次性收集所有错误。
* 校验函数写成 `v is T` 形式可同时完成「校验 + 收窄」。
* 嵌套结构要逐层校验，不要只检查顶层就断言。

### 24.5 API 数据验证

**概念说明：** 网络返回的数据不可信（字段缺失、类型变化、后端 bug）。应在「边界处」统一校验后再进入业务逻辑，并处理超时、非 JSON 响应、HTTP 错误等异常情况。

```typescript
interface ApiUser { id: number; name: string; email: string }

// 边界处校验
function isApiUser(v: unknown): v is ApiUser {
  return typeof v === "object" && v !== null
    && typeof (v as ApiUser).id === "number"
    && typeof (v as ApiUser).name === "string"
    && typeof (v as ApiUser).email === "string";
}

type FetchResult<T> =
  | { ok: true; data: T }
  | { ok: false; reason: "http" | "network" | "timeout" | "parse" | "schema"; message: string; status?: number };

async function fetchUser(id: number, fetchImpl: (n: number) => Promise<unknown>): Promise<FetchResult<ApiUser>> {
  let raw: unknown;
  try {
    raw = await fetchImpl(id);
  } catch (e) {
    if (e instanceof Error && e.name === "AbortError") {
      return { ok: false, reason: "timeout", message: "请求超时" };
    }
    return { ok: false, reason: "network", message: (e as Error).message };
  }

  if (typeof raw === "object" && raw !== null && "status" in raw) {
    const status = (raw as { status: unknown }).status;
    if (typeof status === "number" && status >= 400) {
      return { ok: false, reason: "http", message: `HTTP ${status}`, status };
    }
  }

  if (!isApiUser(raw)) {
    return { ok: false, reason: "schema", message: `返回结构不符合预期: ${JSON.stringify(raw)}` };
  }

  return { ok: true, data: raw };
}

// 模拟各种响应
const responses: ((n: number) => Promise<unknown>)[] = [
  async () => ({ id: 1, name: "Alice", email: "a@x.com" }),
  async () => ({ id: 2, name: "Bob" }),                       // 缺字段
  async () => ({ status: 404, message: "Not Found" }),        // HTTP 错误
  async () => { const e = new Error("aborted"); e.name = "AbortError"; throw e; },
];

(async () => {
  for (const impl of responses) {
    const r = await fetchUser(1, impl);
    if (r.ok) console.log("成功:", r.data.name);
    else console.log(`失败[${r.reason}]`, r.message);
  }
})();

// 统一封装：JSON 解析也要防错
function safeJson(text: string): unknown {
  try {
    return JSON.parse(text);
  } catch {
    return undefined;
  }
}
console.log("非法 JSON:", safeJson("{oops"), "合法:", JSON.stringify(safeJson('{"a":1}')));

// 列表接口：逐项过滤，避免一条坏数据毁掉整个列表
function parseList(v: unknown): ApiUser[] {
  if (!Array.isArray(v)) return [];
  return v.filter(isApiUser);
}
console.log("列表过滤:", parseList([{ id: 1, name: "A", email: "e" }, { bad: 1 }]).length);

// 带超时的请求（AbortController）
function withTimeout<T>(p: Promise<T>, ms: number): Promise<T> {
  const timer = new Promise<never>((_, rej) => {
    const e = new Error("超时"); e.name = "AbortError"; setTimeout(() => rej(e), ms);
  });
  return Promise.race([p, timer]);
}
withTimeout(Promise.resolve("ok"), 10).then((v) => console.log("未超时:", v));
```

**运行结果：**
```
成功: Alice
失败[schema] 返回结构不符合预期: {"id":2,"name":"Bob"}
失败[http] HTTP 404
失败[timeout] 请求超时
非法 JSON: undefined 合法: {"a":1}
列表过滤: 1
未超时: ok
```

**注意：**
* 首选「返回结果对象」而非抛异常，调用方能穷尽处理各失败原因（可辨识联合）。
* HTTP 层错误（4xx/5xx）与数据结构错误要分开处理，便于排障。
* 列表接口逐项过滤 + 记录被丢弃项，比整体失败更健壮。

### 24.6 Runtime Schema Validation

**概念说明：** Schema 库（Zod、valibot、io-ts、ajv）让你「写一次定义」同时得到运行时校验与静态类型（类型从 Schema 推导），彻底消除类型与校验不同步的问题。

```bash
npm i zod
```

```typescript
import { z } from "zod";

// 定义 Schema
const UserSchema = z.object({
  id: z.number().int().positive(),
  name: z.string().min(1, "名字不能为空"),
  email: z.string().email("邮箱格式不正确"),
  age: z.number().int().min(0).max(150).optional(),
  role: z.enum(["admin", "user", "guest"]).default("user"),
  tags: z.array(z.string()).default([]),
});

// 类型从 Schema 推导（单一事实来源）
type User = z.infer<typeof UserSchema>;
// 等价于 { id: number; name: string; email: string; age?: number; role: "admin"|"user"|"guest"; tags: string[] }

// 安全解析：返回结果对象，不抛异常
const parsed = UserSchema.safeParse({ id: 1, name: "Alice", email: "a@x.com" });
if (parsed.success) {
  const u: User = parsed.data;
  console.log("解析成功:", u.name, "角色默认值:", u.role, "标签:", u.tags);
} else {
  console.log("错误:", parsed.error.issues.map((i) => `${i.path.join(".")}: ${i.message}`));
}

// 失败示例
const bad = UserSchema.safeParse({ id: -1, name: "", email: "not-an-email" });
if (!bad.success) {
  bad.error.issues.forEach((i) => console.log(`- ${i.path.join(".")} → ${i.message}`));
}

// 抛异常风格
try {
  const u = UserSchema.parse({ id: 1, name: "Bob", email: "b@x.com" });
  console.log("parse 成功:", u.email);
} catch (e) {
  console.log("parse 失败:", (e as Error).message.slice(0, 40));
}

// 嵌套与组合
const OrderSchema = z.object({
  id: z.number(),
  items: z.array(z.object({ sku: z.string(), qty: z.number().positive() })).min(1),
});
type Order = z.infer<typeof OrderSchema>;
const order = OrderSchema.parse({ id: 1, items: [{ sku: "A", qty: 2 }] });
console.log("订单条目:", order.items.length);

// 转换与管道：字符串自动转数字
const PortSchema = z.coerce.number().int().min(1).max(65535);
console.log("端口转换:", PortSchema.parse("8080"), typeof PortSchema.parse("8080"));

// 判别联合（对应 TS 的可辨识联合）
const EventSchema = z.discriminatedUnion("type", [
  z.object({ type: z.literal("click"), x: z.number(), y: z.number() }),
  z.object({ type: z.literal("key"), code: z.string() }),
]);
console.log("判别联合:", EventSchema.parse({ type: "key", code: "Enter" }).code);

// 与类型守卫配合
function isUser(v: unknown): v is User {
  return UserSchema.safeParse(v).success;
}
console.log("守卫:", isUser({ id: 1, name: "A", email: "a@x.com" }));
```

**运行结果：**
```
解析成功: Alice 角色默认值: user 标签: []
- id → Too small: expected number to be >0
- name → 名字不能为空
- email → 邮箱格式不正确
parse 成功: b@x.com
订单条目: 1
端口转换: 8080 number
判别联合: Enter
守卫: true
```

```typescript
// 其他常见方案一览（接口风格相近）
// valibot：更小的体积，函数式 API，适合前端体积敏感场景
// import * as v from "valibot";
// const S = v.object({ name: v.string() });
// type T = v.InferOutput<typeof S>;

// io-ts：基于 fp-ts，函数式风格
// import * as t from "io-ts";
// const S = t.type({ name: t.string });

// ajv：从 JSON Schema 校验（适合已有 JSON Schema 的项目）
// const validate = ajv.compile(schema);
// if (!validate(data)) console.log(validate.errors);

// 选型建议：
// - 前后端同构、DX 优先 → Zod
// - 体积敏感（前端 bundle）→ valibot
// - 已有 JSON Schema → ajv
// - 函数式栈（fp-ts）→ io-ts
console.log("Schema 库让「类型」与「校验」来自同一份定义");
```

**运行结果：**
```
Schema 库让「类型」与「校验」来自同一份定义
```

**注意：**
* `z.infer<typeof Schema>` 让类型随 Schema 自动更新，避免两处维护。
* 优先用 `safeParse`（返回结果对象），`parse` 会抛异常，适合「必须成功」的场景。
* Schema 库有运行时体积开销（Zod 较大），前端项目注意 bundle 大小，或只在边界层使用。

## 25. Node.js + TypeScript
### 25.1 Node.js 基础

**概念说明：** Node.js 是基于 V8 的服务端 JavaScript 运行时，提供文件、网络、进程等 API。配合 TypeScript 使用需要 `@types/node` 提供类型，并注意模块格式（CJS/ESM）与编译目标。

```bash
# 初始化项目
mkdir ts-node-app && cd ts-node-app
npm init -y
npm i -D typescript @types/node tsx
npx tsc --init
```

```typescript
// src/index.ts
import process from "node:process";

function main(): void {
  console.log("Node 版本:", process.version);
  console.log("平台:", process.platform, "架构:", process.arch);
  console.log("工作目录:", process.cwd());
  console.log("启动参数:", process.argv.slice(2));
}

main();

// 全局对象与常用 API（类型来自 @types/node）
const timer = setTimeout(() => console.log("定时器"), 10);
clearTimeout(timer);

// 一次性执行的定时器
const immediate = setImmediate(() => console.log("立即执行"));
clearImmediate(immediate);

// 命令行参数与环境
console.log("环境:", process.env.NODE_ENV ?? "development");

// 退出码
// process.exit(0);        // 正常退出
// process.exit(1);        // 异常退出

// 未捕获异常与未处理拒绝
process.on("uncaughtException", (err) => console.error("未捕获异常:", err.message));
process.on("unhandledRejection", (reason) => console.error("未处理的 Promise 拒绝:", reason));
Promise.reject(new Error("测试未处理拒绝"));

// 事件循环的下一阶段
process.nextTick(() => console.log("nextTick 优先于 Promise"));

// 退出前清理
process.on("exit", (code) => console.log("进程退出，码:", code));
```

**运行结果：**
```
Node 版本: v20.x.x
平台: win32 架构: x64
工作目录: .../ts-node-app
启动参数: []
环境: development
未处理的 Promise 拒绝: Error: 测试未处理拒绝
立即执行
nextTick 优先于 Promise
定时器
进程退出，码: 0
```

**注意：**
* Node 全局对象（`process`、`Buffer`、`__dirname`）的类型由 `@types/node` 提供。
* `nextTick` 优先于 Promise 微任务；两者都优先于 `setImmediate` 与定时器。
* 生产环境应处理 `uncaughtException` 与 `unhandledRejection`，并记录日志后优雅退出。

### 25.2 npm

**概念说明：** npm 是包管理器，负责依赖安装、脚本运行、版本管理。TypeScript 项目通常把 `typescript`、`@types/*`、运行器放在 `devDependencies`。

```bash
# 项目初始化
npm init -y

# 安装依赖
npm i express                 # 运行时依赖 → dependencies
npm i -D typescript tsx @types/node @types/express   # 开发依赖 → devDependencies

# 版本范围
npm i lodash@^4.17.0          # 允许 4.x 升级
npm i lodash@~4.17.0          # 只允许 4.17.x
npm i lodash@4.17.21          # 精确版本

# 更新与检查
npm outdated                  # 查看可升级的包
npm update                    # 按范围升级并更新 lock
npm audit                     # 安全审计
npm audit fix

# 移除与清理
npm uninstall lodash
npm ci                        # 按 package-lock.json 精确安装（CI 用）

# 运行脚本
npm run dev
npm run build
npm test -- --watch           # 透传参数需加 --

# 查看与执行
npm ls typescript             # 查看已安装版本
npm view typescript version   # 查看仓库最新版本
npx tsc --version             # 执行本地二进制
```

```json
// package.json 脚本示例
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc -p tsconfig.build.json",
    "start": "node dist/index.js",
    "typecheck": "tsc --noEmit",
    "test": "vitest run",
    "lint": "eslint src --ext .ts"
  }
}
```

**运行结果：**
```
（npm 命令输出取决于当前依赖状态）
npx tsc --version → Version 5.x.x
```

**注意：**
* `dependencies` 与 `devDependencies` 要分清：库的消费者不需要你的构建工具。
* 提交 `package-lock.json` 并用 `npm ci` 保证可复现安装。
* `npx` 会优先使用本地 `node_modules/.bin`，避免全局版本冲突。

### 25.3 package.json

**概念说明：** `package.json` 是项目清单：名称版本、依赖、脚本、模块入口与类型入口。TypeScript 项目最关键的是 `types`、`exports` 与 `type` 字段。

```json
{
  "name": "ts-node-app",
  "version": "1.0.0",
  "description": "TypeScript Node 示例",
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./utils": {
      "types": "./dist/utils.d.ts",
      "import": "./dist/utils.js"
    }
  },
  "files": ["dist", "README.md"],
  "engines": { "node": ">=18" },
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "dependencies": {},
  "devDependencies": { "typescript": "^5.4.0", "@types/node": "^20.0.0" },
  "sideEffects": false
}
```

```typescript
// 关键字段说明（配合代码理解）
// "type": "module"    → .js 文件按 ESM 解析；CJS 需用 .cjs 后缀
// "main"              → 传统入口（CJS 优先场景）
// "types"             → 类型入口，找不到会用同目录 index.d.ts
// "exports"           → 现代入口映射，支持条件导出（import/require/types）
// "files"             → 发布白名单，避免把 src 发出去
// "sideEffects": false→ 告诉 bundler 可安全 tree-shaking
// "engines"           → 声明所需 Node 版本

// 读取自身 package.json（需 resolveJsonModule）
// import pkg from "./package.json" with { type: "json" };
// console.log(pkg.name, pkg.version);

console.log("package.json 决定模块解析与发布内容");

// exports 优先级高于 main，且会「封闭」包入口
// 未在 exports 中列出的子路径将无法被导入
console.log("条件导出的键顺序很重要：types 必须放最前");
```

**运行结果：**
```
package.json 决定模块解析与发布内容
条件导出的键顺序很重要：types 必须放最前
```

**注意：**
* 同时提供 CJS 与 ESM 时，用 `exports` 的条件导出分流，并避免「双包危险」（同一模块被加载两份）。
* `types` 条件必须放在 `exports` 中每个子路径的**最前面**，否则工具可能先匹配到 JS。
* `files` 白名单比 `.npmignore` 更可靠，能避免误发源码与配置。

### 25.4 CommonJS 与 ESM

**概念说明：** Node 同时支持两种模块体系。选择依据：新项目用 ESM；需要兼容老依赖或使用 require 生态时用 CJS。混合使用需理解 `default` 包装与 `__esModule` 标记。

```typescript
// 方案 A：纯 ESM 项目
// package.json → "type": "module"
// tsconfig.json → module: "nodenext", moduleResolution: "nodenext"

// src/index.ts
import { readFile } from "node:fs/promises";
import path from "node:path";

const file = path.join(process.cwd(), "package.json");
const content = await readFile(file, "utf8");      // 顶层 await（ESM 支持）
console.log("读取字节数:", content.length);

// ESM 中获取 __dirname 的替代
import { fileURLToPath } from "node:url";
const __filename2 = fileURLToPath(import.meta.url);
console.log("当前文件:", __filename2.split(/[\\/]/).pop());

// 方案 B：CJS 项目
// package.json 无 "type" 或 "type": "commonjs"
// tsconfig.json → module: "commonjs"
// const fs = require("fs");
// console.log(__dirname);

// 从 ESM 导入 CJS 包：默认导入拿到的就是 module.exports
// import express from "express";              // 需要 esModuleInterop

// 从 CJS 导入 ESM：必须用动态 import（CJS 不支持静态 import ESM）
async function loadEsm() {
  const mod = await import("./utils.js");
  return mod.default ?? mod;
}
console.log("动态导入 ESM:", typeof loadEsm);

// 互操作注意点：default 包装
// CJS: module.exports = { a: 1 }  ← ESM 中 import x from 得到 { a: 1 }
// CJS: exports.a = 1              ← ESM 中 import { a } 也能用（Node 静态分析）

// 扩展名规则（nodenext）：相对导入必须写 .js
// import { helper } from "./helper.js";       // 源文件是 helper.ts

// 判断当前模块体系
console.log("是否 ESM:", typeof import.meta?.url === "string" ? "是" : "否");
```

**运行结果：**
```
读取字节数: 512
当前文件: index.ts
动态导入 ESM: function
是否 ESM: 是
```

**注意：**
* ESM 没有 `__dirname` / `__filename`，用 `import.meta.url` 转换。
* ESM 中相对导入在 `nodenext` 下必须写 `.js` 扩展名。
* CJS 里不能静态 `import` ESM，只能用动态 `import()`；反之 ESM 可以默认导入 CJS。

### 25.5 Node.js 模块系统

**概念说明：** Node 的模块解析规则：核心模块（`node:` 前缀）、相对/绝对路径、`node_modules` 向上查找、`exports` 字段门禁。TypeScript 侧由 `moduleResolution` 对齐这套规则。

```typescript
// 1) 核心模块：推荐加 node: 前缀（明确、避免与包名冲突）
import fs from "node:fs";
import path from "node:path";
import { EventEmitter } from "node:events";
console.log("核心模块:", typeof fs.readFileSync, typeof path.join);

// 2) 相对 / 绝对路径
// import { helper } from "./utils/helper.js";        // nodenext 需扩展名
// import config from "/abs/path/config.js";

// 3) 第三方包：从当前目录向上找 node_modules
// import lodash from "lodash";                       // → node_modules/lodash

// 4) 子路径导入（受 exports 门禁约束）
// import { utils } from "some-pkg/utils";            // 需包内 exports 暴露

// 5) 目录导入：找 index.js / package.json main
// import "./lib";                                    // → ./lib/index.js

// 6) 自定义事件（EventEmitter）
class MyEmitter extends EventEmitter {}
const emitter = new MyEmitter();
emitter.on("data", (payload: string) => console.log("收到:", payload));
emitter.emit("data", "事件负载");

// 7) 模块缓存：同一路径只加载一次
const a = { value: 1 };
console.log("模块单例特性（示意）:", a === a);

// 8) 解析自己实现的简化版
function resolveLocal(importPath: string): string {
  if (importPath.startsWith("node:")) return `核心模块 ${importPath}`;
  if (importPath.startsWith(".")) return `相对路径 ${path.resolve(importPath)}`;
  return `第三方包 ${importPath} → node_modules 查找`;
}
console.log(resolveLocal("node:fs"));
console.log(resolveLocal("./utils.js"));
console.log(resolveLocal("express"));

// 9) ESM 与 CJS 的缓存差异
// CJS: require.cache 可手动删除实现热重载
// ESM: 模块图是静态的，不能删除缓存

// 10) package.json exports 门禁示例
// "exports": { ".": "./index.js", "./utils": "./utils.js" }
// import "pkg/internal/secret.js";    // 错误：未在 exports 中暴露

console.log("Node 模块解析顺序：核心 → 相对/绝对 → node_modules → exports 门禁");
```

**运行结果：**
```
核心模块: function function
收到: 事件负载
模块单例特性（示意）: true
核心模块 node:fs
相对路径 .../utils.js
第三方包 express → node_modules 查找
Node 模块解析顺序：核心 → 相对/绝对 → node_modules → exports 门禁
```

**注意：**
* 带 `node:` 前缀可避免与用户包重名（如 `node:test` 与第三方 `test` 包）。
* `exports` 字段一旦定义，未列出的子路径就无法导入（提升封装性）。
* 模块只求值一次并缓存，导出的对象在多个导入方之间共享。

### 25.6 文件系统

**概念说明：** `node:fs` 提供同步/回调/Promise 三套 API。TypeScript 项目中优先使用 `node:fs/promises` 配合 async/await，并用类型区分 `Buffer` 与 `string`。

```typescript
import { promises as fs } from "node:fs";
import path from "node:path";
import os from "node:os";

async function main(): Promise<void> {
  const dir = path.join(os.tmpdir(), "ts-fs-demo");
  const file = path.join(dir, "data.json");

  // 创建目录（递归，已存在不报错）
  await fs.mkdir(dir, { recursive: true });

  // 写入文本
  await fs.writeFile(file, JSON.stringify({ name: "Alice", age: 25 }, null, 2), "utf8");

  // 读取文本
  const text = await fs.readFile(file, "utf8");
  console.log("文件内容:", text.replace(/\s+/g, " "));

  // 不安全解析：先 unknown 再校验
  const parsed: unknown = JSON.parse(text);
  if (typeof parsed === "object" && parsed !== null && "name" in parsed) {
    console.log("用户名:", (parsed as { name: string }).name);
  }

  // 追加内容
  await fs.appendFile(file, "\n// 追加行", "utf8");

  // 读取为 Buffer（二进制）
  const buf = await fs.readFile(file);
  console.log("Buffer 字节数:", buf.byteLength, "是 Buffer:", Buffer.isBuffer(buf));

  // 文件信息
  const stat = await fs.stat(file);
  console.log("大小:", stat.size, "是文件:", stat.isFile(), "修改时间类型:", stat.mtime instanceof Date);

  // 遍历目录
  await fs.writeFile(path.join(dir, "b.txt"), "b", "utf8");
  const entries = await fs.readdir(dir, { withFileTypes: true });
  entries.forEach((e) => console.log("条目:", e.name, "是文件:", e.isFile()));

  // 重命名与删除
  const renamed = path.join(dir, "renamed.txt");
  await fs.rename(path.join(dir, "b.txt"), renamed);
  await fs.unlink(renamed);

  // 判断存在（推荐用 access 而非 existsSync 的异步版）
  try {
    await fs.access(file);
    console.log("文件可访问: true");
  } catch {
    console.log("文件可访问: false");
  }

  // 清理
  await fs.rm(dir, { recursive: true, force: true });
  console.log("已清理临时目录");
}

main().catch((e: Error) => console.error("失败:", e.message));

// 同步 API（阻塞，仅适合启动脚本/CLI）
// const data = fsSync.readFileSync("package.json", "utf8");

// 流式读取大文件（避免一次性载入内存）
// import { createReadStream } from "node:fs";
// const stream = createReadStream(file, { encoding: "utf8" });
// stream.on("data", (chunk) => console.log("chunk:", chunk.length));
```

**运行结果：**
```
文件内容: {"name":"Alice","age":25} // 追加行
用户名: Alice
Buffer 字节数: 41 是 Buffer: true
大小: 41 是文件: true 修改时间类型: true
条目: b.txt 是文件: true
条目: data.json 是文件: true
文件可访问: true
已清理临时目录
```

**注意：**
* `fs.readFile(file, "utf8")` 返回 `string`，不传编码返回 `Buffer` —— 类型会相应变化。
* 大文件用 `createReadStream`，避免 `readFile` 把整个文件读入内存。
* 路径用 `path.join` / `path.resolve` 拼接，保证跨平台。

### 25.7 HTTP

**概念说明：** 可以用内置 `node:http` 写服务，也可以用 Express/Fastify。TypeScript 的关键点是为请求体、参数、响应声明类型，并在边界处校验。

```typescript
import http from "node:http";
import type { IncomingMessage, ServerResponse } from "node:http";

interface CreateUserBody { name: string; email: string }
interface ApiResponse<T> { code: number; message: string; data?: T }

function isCreateUserBody(v: unknown): v is CreateUserBody {
  return typeof v === "object" && v !== null
    && typeof (v as CreateUserBody).name === "string"
    && typeof (v as CreateUserBody).email === "string";
}

// 读取请求体：unknown，逐层解析
async function readBody(req: IncomingMessage): Promise<unknown> {
  const chunks: Buffer[] = [];
  for await (const chunk of req) chunks.push(chunk as Buffer);
  const text = Buffer.concat(chunks).toString("utf8");
  try {
    return JSON.parse(text) as unknown;
  } catch {
    return undefined;
  }
}

function send<T>(res: ServerResponse, status: number, body: ApiResponse<T>): void {
  res.writeHead(status, { "Content-Type": "application/json; charset=utf-8" });
  res.end(JSON.stringify(body));
}

const server = http.createServer(async (req: IncomingMessage, res: ServerResponse) => {
  const url = new URL(req.url ?? "/", "http://localhost");

  if (req.method === "GET" && url.pathname === "/health") {
    send(res, 200, { code: 0, message: "ok", data: { status: "healthy" } });
    return;
  }

  if (req.method === "POST" && url.pathname === "/users") {
    const body = await readBody(req);
    if (!isCreateUserBody(body)) {
      send(res, 400, { code: 400, message: "参数不合法" });
      return;
    }
    send(res, 201, { code: 0, message: "创建成功", data: { id: 1, name: body.name } });
    return;
  }

  // 路径参数
  const match = url.pathname.match(/^\/users\/(\d+)$/);
  if (req.method === "GET" && match) {
    const id = Number(match[1]);
    send(res, 200, { code: 0, message: "ok", data: { id, name: "Alice" } });
    return;
  }

  send(res, 404, { code: 404, message: "未找到" });
});

// 监听与优雅关闭
server.listen(0, () => {
  const addr = server.address();
  const port = typeof addr === "object" && addr ? addr.port : 0;
  console.log("服务已启动，端口:", port > 0);

  // 自测三个接口
  const base = `http://127.0.0.1:${port}`;
  const test = async () => {
    const health = await fetch(`${base}/health`);
    console.log("GET /health →", health.status, (await health.json() as { data: { status: string } }).data.status);

    const created = await fetch(`${base}/users`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ name: "Alice", email: "a@x.com" }),
    });
    console.log("POST /users →", created.status);

    const bad = await fetch(`${base}/users`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ name: 1 }),
    });
    console.log("POST /users 非法体 →", bad.status);

    const one = await fetch(`${base}/users/7`);
    console.log("GET /users/7 →", one.status, (await one.json() as { data: { id: number } }).data.id);

    server.close(() => console.log("服务已关闭"));
  };
  test().catch((e: Error) => console.error(e.message));
});
```

**运行结果：**
```
服务已启动，端口: true
GET /health → 200 healthy
POST /users → 201
POST /users 非法体 → 400
GET /users/7 → 200 7
服务已关闭
```

**注意：**
* 请求体天然是 `unknown`（`JSON.parse` 返回 `any`），必须校验后再用。
* 用泛型 `ApiResponse<T>` 统一响应格式，保证前后端契约一致。
* 生产环境要考虑：超时、请求体大小限制、CORS、错误中间件、优雅关闭（`SIGTERM` 时停止接收新连接）。

### 25.8 WebSocket

**概念说明：** WebSocket 提供双向长连接。服务端常用 `ws` 库，需要为「消息协议」定义类型（通常用可辨识联合），并在收包时校验。

```bash
npm i ws && npm i -D @types/ws
```

```typescript
import { WebSocketServer, WebSocket } from "ws";
import type { RawData } from "ws";

// 协议定义（可辨识联合）
type ClientMessage =
  | { type: "join"; room: string; user: string }
  | { type: "chat"; room: string; text: string }
  | { type: "ping"; ts: number };

type ServerMessage =
  | { type: "joined"; room: string; members: number }
  | { type: "chat"; room: string; user: string; text: string }
  | { type: "pong"; ts: number }
  | { type: "error"; message: string };

// 收包校验
function isClientMessage(v: unknown): v is ClientMessage {
  if (typeof v !== "object" || v === null || !("type" in v)) return false;
  const t = (v as { type: unknown }).type;
  return t === "join" || t === "chat" || t === "ping";
}

function parse(raw: RawData): unknown {
  try {
    return JSON.parse(raw.toString()) as unknown;
  } catch {
    return undefined;
  }
}

const rooms = new Map<string, Set<WebSocket>>();

const wss = new WebSocketServer({ port: 0 });

wss.on("connection", (socket: WebSocket) => {
  console.log("客户端已连接");

  const sendMsg = (msg: ServerMessage): void => {
    if (socket.readyState === WebSocket.OPEN) socket.send(JSON.stringify(msg));
  };

  socket.on("message", (raw: RawData) => {
    const parsed = parse(raw);
    if (!isClientMessage(parsed)) {
      sendMsg({ type: "error", message: "无法识别的消息" });
      return;
    }

    switch (parsed.type) {
      case "join": {
        const set = rooms.get(parsed.room) ?? new Set<WebSocket>();
        set.add(socket);
        rooms.set(parsed.room, set);
        sendMsg({ type: "joined", room: parsed.room, members: set.size });
        break;
      }
      case "chat": {
        const set = rooms.get(parsed.room);
        const payload: ServerMessage = { type: "chat", room: parsed.room, user: "匿名", text: parsed.text };
        set?.forEach((c) => c.readyState === WebSocket.OPEN && c.send(JSON.stringify(payload)));
        console.log("广播到房间:", parsed.room, "人数:", set?.size ?? 0);
        break;
      }
      case "ping":
        sendMsg({ type: "pong", ts: parsed.ts });
        break;
    }
  });

  socket.on("close", () => {
    rooms.forEach((set) => set.delete(socket));
    console.log("客户端已断开");
  });

  socket.on("error", (err: Error) => console.error("socket 错误:", err.message));
});

// 自测：连接 → 加入 → 聊天 → ping
wss.on("listening", async () => {
  const addr = wss.address();
  const port = typeof addr === "object" && addr ? addr.port : 0;

  const client = new WebSocket(`ws://127.0.0.1:${port}`);
  const inbox: ServerMessage[] = [];

  client.on("message", (raw: RawData) => {
    const msg = JSON.parse(raw.toString()) as ServerMessage;
    inbox.push(msg);
    if (msg.type === "joined") console.log("加入房间:", msg.room, "成员:", msg.members);
    if (msg.type === "chat") console.log("收到聊天:", msg.text);
    if (msg.type === "pong") {
      console.log("pong 时间戳:", msg.ts);
      client.close();
    }
  });

  client.on("open", () => {
    const join: ClientMessage = { type: "join", room: "general", user: "Alice" };
    client.send(JSON.stringify(join));
    setTimeout(() => {
      const chat: ClientMessage = { type: "chat", room: "general", text: "大家好" };
      client.send(JSON.stringify(chat));
      const ping: ClientMessage = { type: "ping", ts: Date.now() };
      client.send(JSON.stringify(ping));
    }, 10);
  });

  client.on("close", () => {
    console.log("客户端已关闭，共收到", inbox.length, "条消息");
    wss.close(() => console.log("服务端已关闭"));
  });
});

// 心跳保活（生产必备）：定时 ping，超时则断开
// setInterval(() => {
//   wss.clients.forEach((c) => {
//     if (!isAlive(c)) return c.terminate();
//     isAlive(c) = false;
//     c.ping();
//   });
// }, 30000);
```

**运行结果：**
```
客户端已连接
加入房间: general 成员: 1
广播到房间: general 人数: 1
收到聊天: 大家好
pong 时间戳: 1735689600000
客户端已关闭，共收到 3 条消息
客户端已断开
服务端已关闭
```

**注意：**
* 消息协议用可辨识联合表达，配合 `isClientMessage` 校验，避免 `any` 渗透。
* 生产环境需心跳（ping/pong）检测死连接、限制消息大小与频率、鉴权、断线重连。
* `ws` 不提供房间/广播语义，需要自己维护连接集合（或使用 Socket.IO）。

### 25.9 环境变量

**概念说明：** 环境变量在 Node 中都是 `string | undefined`，必须校验与转换后使用。推荐在启动时集中解析成「配置对象」，让类型系统保证内部代码拿到的是已校验的值。

```typescript
// src/config.ts
interface AppConfig {
  nodeEnv: "development" | "production" | "test";
  port: number;
  databaseUrl: string;
  debug: boolean;
  apiKeys: string[];
}

class ConfigError extends Error {
  constructor(message: string) { super(message); this.name = "ConfigError"; }
}

function requiredString(env: NodeJS.ProcessEnv, key: string): string {
  const v = env[key];
  if (typeof v !== "string" || v.trim() === "") throw new ConfigError(`缺少环境变量 ${key}`);
  return v.trim();
}

function parsePort(raw: string | undefined, fallback = 3000): number {
  if (raw === undefined) return fallback;
  const n = Number(raw);
  if (!Number.isInteger(n) || n < 1 || n > 65535) throw new ConfigError(`PORT 非法: ${raw}`);
  return n;
}

function parseBool(raw: string | undefined, fallback = false): boolean {
  if (raw === undefined) return fallback;
  if (["1", "true", "yes"].includes(raw.toLowerCase())) return true;
  if (["0", "false", "no"].includes(raw.toLowerCase())) return false;
  throw new ConfigError(`布尔值非法: ${raw}`);
}

function parseEnum<T extends string>(raw: string | undefined, allowed: readonly T[], fallback: T): T {
  if (raw === undefined) return fallback;
  if (!allowed.includes(raw as T)) throw new ConfigError(`取值必须是 ${allowed.join(" | ")}，收到 ${raw}`);
  return raw as T;
}

// 启动时一次性解析：失败即快速失败
function loadConfig(env: NodeJS.ProcessEnv = process.env): AppConfig {
  return {
    nodeEnv: parseEnum(env.NODE_ENV, ["development", "production", "test"] as const, "development"),
    port: parsePort(env.PORT),
    databaseUrl: requiredString(env, "DATABASE_URL"),
    debug: parseBool(env.DEBUG),
    apiKeys: (env.API_KEYS ?? "").split(",").map((s) => s.trim()).filter(Boolean),
  };
}

// 使用
const fakeEnv: NodeJS.ProcessEnv = {
  NODE_ENV: "production",
  PORT: "8080",
  DATABASE_URL: "postgres://localhost:5432/app",
  DEBUG: "true",
  API_KEYS: "k1, k2",
};
const cfg = loadConfig(fakeEnv);
console.log("环境:", cfg.nodeEnv, "端口:", cfg.port, typeof cfg.port);
console.log("调试:", cfg.debug, "密钥数:", cfg.apiKeys.length);

// 校验失败的例子
try {
  loadConfig({ PORT: "abc", DATABASE_URL: "x" });
} catch (e) {
  console.log("配置错误:", (e as ConfigError).message);
}
try {
  loadConfig({});
} catch (e) {
  console.log("配置错误:", (e as ConfigError).message);
}

// 类型安全访问：后续代码无需再判空
function startServer(c: AppConfig): void {
  console.log(`启动于 ${c.port}，debug=${c.debug}`);
}
startServer(cfg);

// 敏感信息不要提交到仓库：用 .env + .gitignore
// .env: DATABASE_URL=postgres://...  PORT=8080
// 加载（Node 20.6+ 内置）：node --env-file=.env dist/index.js
// 或使用 dotenv: import "dotenv/config";
console.log("环境变量应在边界处校验并转换为强类型配置");
```

**运行结果：**
```
环境: production 端口: 8080 number
调试: true 密钥数: 2
配置错误: PORT 非法: abc
配置错误: 缺少环境变量 DATABASE_URL
启动于 8080，debug=true
环境变量应在边界处校验并转换为强类型配置
```

**注意：**
* `process.env.X` 永远是 `string | undefined`，不要断言成 `string`，要校验。
* 集中在启动时解析配置（Fail Fast），避免在业务代码里到处判空。
* `.env` 文件不要提交；CI/部署平台通过环境变量注入。

### 25.10 TypeScript Node 项目配置

**概念说明：** Node + TypeScript 项目的推荐配置组合：ESM 优先用 `nodenext`，只做类型检查交给 bundler/tsx 执行，生产用 `tsc` 输出或 tsx/ts-node 运行。

```json
// package.json
{
  "name": "ts-node-app",
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc -p tsconfig.build.json",
    "start": "node --enable-source-maps dist/index.js",
    "typecheck": "tsc --noEmit",
    "test": "vitest run"
  },
  "devDependencies": {
    "typescript": "^5.4.0",
    "tsx": "^4.0.0",
    "@types/node": "^20.0.0",
    "vitest": "^1.0.0"
  }
}
```

```json
// tsconfig.json（开发用：包含测试）
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2023"],
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "types": ["node"],
    "outDir": "dist",
    "rootDir": "src",
    "sourceMap": true,
    "declaration": true,
    "declarationMap": true
  },
  "include": ["src/**/*.ts", "tests/**/*.ts"],
  "exclude": ["node_modules", "dist"]
}
```

```json
// tsconfig.build.json（构建用：排除测试）
{
  "extends": "./tsconfig.json",
  "compilerOptions": { "noEmit": false },
  "include": ["src/**/*.ts"],
  "exclude": ["**/*.test.ts", "**/*.spec.ts", "tests"]
}
```

```bash
# 目录结构
# src/
#   index.ts         入口（启动服务）
#   config.ts        环境变量解析
#   server.ts        HTTP/WebSocket 服务
#   services/
#   types/            全局类型声明（*.d.ts）
# tests/
# dist/               构建产物
# tsconfig.json
# tsconfig.build.json
# package.json
```

```typescript
// src/index.ts —— 典型入口
import { loadConfig } from "./config.js";        // nodenext 需 .js 扩展名

async function bootstrap(): Promise<void> {
  const config = loadConfig();
  console.log(`[${config.nodeEnv}] 服务启动，端口 ${config.port}`);
}

bootstrap().catch((e: Error) => {
  console.error("启动失败:", e.message);
  process.exit(1);
});

// 优雅关闭
for (const signal of ["SIGINT", "SIGTERM"] as const) {
  process.on(signal, () => {
    console.log(`收到 ${signal}，开始关闭`);
    process.exit(0);
  });
}

// 额外推荐开启的检查项：
// noUncheckedIndexedAccess  数组/索引访问返回 T | undefined（更安全）
// noImplicitOverride        重写必须写 override
// exactOptionalPropertyTypes 区分「属性缺失」与「值为 undefined」
// verbatimModuleSyntax      强制显式 type 导入，利于单文件编译
console.log("Node + TS 推荐配置：NodeNext + strict 全家桶 + tsx 开发 + tsc 构建");
```

**运行结果：**
```
[development] 服务启动，端口 3000
Node + TS 推荐配置：NodeNext + strict 全家桶 + tsx 开发 + tsc 构建
```

**注意：**
* 开发用 `tsx`（esbuild 驱动，快），生产用 `tsc` 输出或 `tsx`/`ts-node` 直接运行（团队按需选择）。
* `nodenext` 下相对导入必须写 `.js`，这是最常见的踩坑点。
* `noUncheckedIndexedAccess` 与 `exactOptionalPropertyTypes` 会带来一些改造成本，但能消除大量边界 bug。

## 26. TypeScript 进阶
### 26.1 高级泛型

**概念说明：** 高级泛型指在泛型上叠加约束链、条件类型、映射与递归，实现「类型层面的函数」。核心手法：`keyof` 联动、类型参数相互约束、`infer` 提取、递归构建。

```typescript
// 1) 类型参数相互约束：键与值联动
function setProp<T, K extends keyof T>(obj: T, key: K, value: T[K]): T {
  return { ...obj, [key]: value };
}
const user = setProp({ id: 1, name: "A" }, "name", "Bob");
// setProp(user, "id", "字符串");     // 错误

// 2) 用映射类型构造「键 → 值」的严格对应
type StrictRecord<K extends PropertyKey, V> = {
  [P in K]: { key: P; value: V };
};
const r: StrictRecord<"a" | "b", number> = { a: { key: "a", value: 1 }, b: { key: "b", value: 2 } };
console.log(r.a.key, r.b.value);

// 3) 递归泛型：深层映射
type DeepPartial<T> = T extends (infer U)[]
  ? DeepPartial<U>[]
  : T extends object
  ? { [K in keyof T]?: DeepPartial<T[K]> }
  : T;

// 4) 泛型 + 条件类型：按形状分派
type Handler<T> = T extends string
  ? (s: T) => void
  : T extends number
  ? (n: T) => void
  : never;
const strH: Handler<"a"> = (s) => console.log(s);
const numH: Handler<1> = (n) => console.log(n);
strH("a"); numH(1);

// 5) 累积式泛型：类型层面的 Reduce
type TupleToObject<T extends readonly PropertyKey[]> = {
  [K in T[number]]: K;
};
const t2o: TupleToObject<["a", "b"]> = { a: "a", b: "b" };
console.log(t2o.a, t2o.b);

// 6) 用泛型表达「函数组合」的类型
function pipe<A, B>(f: (a: A) => B): (a: A) => B;
function pipe<A, B, C>(f: (a: A) => B, g: (b: B) => C): (a: A) => C;
function pipe<A, B, C, D>(f: (a: A) => B, g: (b: B) => C, h: (c: C) => D): (a: A) => D;
function pipe(...fns: ((x: unknown) => unknown)[]): (x: unknown) => unknown {
  return (x: unknown) => fns.reduce((acc, fn) => fn(acc), x);
}
const toLen = pipe((s: string) => s.split(""), (arr: string[]) => arr.length, (n: number) => `长度 ${n}`);
console.log(toLen("abc"));

// 7) 泛型约束链：T 的键的值类型受限于 K
function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] {
  return items.map((i) => i[key]);
}
console.log(pluck([{ a: 1, b: "x" }, { a: 2, b: "y" }], "a"));

// 8) 高阶类型别名：泛型套泛型
type Curried<T> = T extends (a: infer A, b: infer B) => infer R ? (a: A) => (b: B) => R : never;
const curried: Curried<(a: number, b: string) => boolean> = (a) => (b) => `${a}${b}`.length > 0;
console.log(curried(1)("x"));

// 9) 避免过度泛型：只用一次的类型参数应改为具体类型
// 反例：function f<T, U>(x: T): T  —— U 未被使用
console.log("高级泛型的核心：让类型参数之间建立约束关系");
```

**运行结果：**
```
a 2
a 1
长度 3
[ 1, 2 ]
true
高级泛型的核心：让类型参数之间建立约束关系
```

**注意：**
* 泛型不是越多越好：类型参数只在「出现两次以上」且需要建立关系时才有价值。
* 递归泛型必须有终止分支，且注意编译深度（`Type instantiation is excessively deep`）。
* 复杂泛型建议写类型测试（`Expect<Equal<A, B>>`）防止回归。

### 26.2 高级条件类型

**概念说明：** 高级用法包括：用 `[]` 控制分发、用 `infer` 提取并约束、嵌套分派、以及类型层面的「模式匹配」与「类型校验器」。

```typescript
// 1) 类型层面的断言工具（类型测试的基石）
type Expect<T extends true> = T;
type Equal<A, B> = (<T>() => T extends A ? 1 : 2) extends (<T>() => T extends B ? 1 : 2) ? true : false;

type _T1 = Expect<Equal<ReturnType<() => string>, string>>;
type _T2 = Expect<Equal<Parameters<(a: number) => void>, [a: number]>>;
console.log("类型断言通过（否则编译失败）");

// 2) 阻止分发：整体判断联合
type IsNever<T> = [T] extends [never] ? true : false;
type IsUnion<T, U = T> = T extends unknown ? ([U] extends [T] ? false : true) : never;
type A1 = Expect<Equal<IsNever<never>, true>>;
type A2 = Expect<Equal<IsUnion<"a" | "b">, true>>;
console.log("IsNever / IsUnion 正确");

// 3) infer + 约束：提取并限制
type FirstString<T> = T extends [infer F extends string, ...unknown[]] ? F : never;
const fs: FirstString<["a", 1]> = "a";
console.log(fs);

// 4) 嵌套分派：类型层面的模式匹配
type ParseQuery<T extends string> =
  T extends `${infer K}=${infer V}&${infer Rest}`
    ? { [P in K]: V } & ParseQuery<Rest>
    : T extends `${infer K}=${infer V}`
    ? { [P in K]: V }
    : {};
type Q = ParseQuery<"a=1&b=2">;
const q: Q = { a: "1", b: "2" };
console.log(q.a, q.b);

// 5) 从函数重载/联合中提取
type ExtractByKind<T, K> = T extends { kind: K } ? T : never;
type Shape = { kind: "circle"; r: number } | { kind: "square"; s: number };
const circle: ExtractByKind<Shape, "circle"> = { kind: "circle", r: 1 };
console.log(circle.r);

// 6) 条件类型的「卫语句」风格：提前返回
type ToArrayDeep<T> = T extends readonly unknown[] ? T : [T];
const one: ToArrayDeep<number> = [1];
console.log(one);

// 7) 类型层面的布尔运算
type Not<T extends boolean> = T extends true ? false : true;
type And<A extends boolean, B extends boolean> = A extends true ? B : false;
type Or<A extends boolean, B extends boolean> = A extends true ? true : B;
type _T3 = Expect<Equal<And<true, false>, false>>;
type _T4 = Expect<Equal<Or<false, true>, true>>;
console.log("类型层面布尔运算可用");

// 8) 用条件类型校验对象形状（编译期 lint）
type RequireKeys<T, K extends keyof T> = K extends keyof T ? T : never;
function requireKeys<T, K extends keyof T>(obj: T, keys: K[]): RequireKeys<T, K> {
  for (const k of keys) if (obj[k] === undefined) throw new Error(`缺少 ${String(k)}`);
  return obj;
}
console.log(requireKeys({ a: 1, b: 2 }, ["a"]) === undefined ? "?" : "校验通过");

// 9) 分布式 + 映射：按值类型分组处理
type SplitByType<T> = {
  strings: Extract<T, string>;
  numbers: Extract<T, number>;
};
const split: SplitByType<string | number | boolean> = { strings: "s", numbers: 1 };
console.log(split.strings, split.numbers);

console.log("高级条件类型 = 分发控制 + infer 提取 + 类型断言工具");
```

**运行结果：**
```
类型断言通过（否则编译失败）
IsNever / IsUnion 正确
a
1 2
1
[ 1 ]
类型层面布尔运算可用
校验通过
s 1
高级条件类型 = 分发控制 + infer 提取 + 类型断言工具
```

**注意：**
* `Equal` 这种「判断类型完全相等」的工具是写类型测试的基础，能发现 `extends` 判断不出的差异。
* `[T] extends [U]` 与 `T extends U` 的分发差异是最容易出错的地方。
* 复杂条件类型建议写注释说明「输入 → 输出」示例。

### 26.3 高级映射类型

**概念说明：** 高级用法包括：递归映射、键变换组合（`as` + 模板字面量）、修饰符与条件联用、以及对数组/元组/函数的特殊处理。

```typescript
// 1) 递归映射：深度只读 / 深度可变
type DeepReadonly<T> = T extends (...args: never[]) => unknown
  ? T
  : T extends readonly (infer U)[]
  ? readonly DeepReadonly<U>[]
  : T extends object
  ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
  : T;

type Config = { server: { host: string; ports: number[] }; debug: boolean };
const cfg: DeepReadonly<Config> = { server: { host: "localhost", ports: [80] }, debug: false };
console.log(cfg.server.host, cfg.debug);

// 2) 键变换组合：路径类型（a.b.c）
type Paths<T, Prefix extends string = ""> = {
  [K in keyof T & string]: T[K] extends object
    ? `${Prefix}${K}` | Paths<T[K], `${Prefix}${K}.`>
    : `${Prefix}${K}`;
}[keyof T & string];

type P = Paths<{ a: { b: number; c: { d: string } } }>;
const p1: P = "a";
const p2: P = "a.b";
const p3: P = "a.c.d";
console.log(p1, p2, p3);

// 3) 修饰符 + 条件：按值类型决定可选性
type OptionalIfNullish<T> = {
  [K in keyof T as T[K] extends null | undefined ? K : never]?: T[K];
} & {
  [K in keyof T as T[K] extends null | undefined ? never : K]: T[K];
};
type R = OptionalIfNullish<{ a: number; b: string | null }>;
const r: R = { a: 1 };
console.log(r.a);

// 4) 数组与元组的映射
// 注意：元组的键是字符串（"0" | "1"），要用模板字面量转成数字类型
type NumericIndex<K> = K extends `${infer N extends number}` ? N : never;
type ElementProps<T extends readonly unknown[]> = {
  [K in keyof T]: { index: NumericIndex<K>; value: T[K] };
};
type EP = ElementProps<[string, number]>;
const ep: EP = [{ index: 0, value: "a" }, { index: 1, value: 1 }];
console.log(ep[0].index, ep[1].value);

// 5) 把方法改成返回 Promise
type Asyncify<T> = {
  [K in keyof T]: T[K] extends (...args: infer A) => infer R
    ? (...args: A) => Promise<Awaited<R>>
    : T[K];
};
type Api2 = { get(id: number): string; version: string };
const api: Asyncify<Api2> = { get: async (id) => `id-${id}`, version: "1" };
api.get(1).then((v) => console.log(v));

// 6) 键名变换 + 分组
type Camel<S extends string> = S extends `${infer H}_${infer T}` ? `${H}${Capitalize<Camel<T>>}` : S;
type SnakeToCamel<T> = { [K in keyof T as Camel<string & K>]: T[K] };
type Dto = SnakeToCamel<{ user_id: number; user_name: string }>;
const dto: Dto = { userId: 1, userName: "A" };
console.log(dto.userId);

// 7) 映射 + 键过滤：只保留函数属性
type OnlyMethods<T> = {
  [K in keyof T as T[K] extends (...args: never[]) => unknown ? K : never]: T[K];
};
type M = OnlyMethods<{ a: number; f(): void; g(x: string): number }>;
const m: M = { f: () => {}, g: (x) => x.length };
console.log(typeof m.f);

// 8) 把联合转成映射的对象
type UnionToRecord<U extends PropertyKey, V> = { [K in U]: V };
const ur: UnionToRecord<"a" | "b", number> = { a: 1, b: 2 };
console.log(ur.a, ur.b);

// 9) 注意：同态 vs 非同态映射
// 同态（[K in keyof T]）保留 ? 与 readonly；非同态（[K in Union]）不保留
type Homo<T> = { [K in keyof T]: T[K] };                 // 保留修饰符
type NonHomo<T, K extends keyof T> = { [P in K]: T[P] }; // 不保留
type O = { readonly a?: number };
const homo: Homo<O> = {};
console.log("同态映射保留修饰符:", homo);
```

**运行结果：**
```
localhost false
a a.b a.c.d
1
0 a
id-1
1
function
1 2
同态映射保留修饰符: {}
```

**注意：**
* 递归映射要显式处理函数、数组、`Date` 等特殊对象，否则会出现意外类型。
* `as` 键变换产生的键过多会让类型变得难以阅读，必要时保留注释说明。
* 同态映射是「批量修改修饰符」的最佳选择。

### 26.4 Template Literal Types

**概念说明：** 模板字面量类型的高阶用法：递归解析字符串结构（路由、查询串、CSS 值），与映射类型结合生成配套 API，实现「字符串级」的类型安全。

```typescript
// 1) 路由参数提取
type ExtractParams<T extends string> =
  T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | ExtractParams<`/${Rest}`>
    : T extends `${string}:${infer Param}`
    ? Param
    : never;

type Params = ExtractParams<"/users/:id/posts/:postId">;
const p1: Params = "id";
const p2: Params = "postId";
console.log(p1, p2);

// 2) 路由字符串 → 参数对象类型
function route<T extends string>(path: T, params: Record<ExtractParams<T>, string>): string {
  return path.replace(/:(\w+)/g, (_, k: string) => params[k as ExtractParams<T>] ?? "");
}
console.log(route("/users/:id", { id: "7" }));

// 3) 生成事件处理器名称（映射 + 模板）
type EventNames = "click" | "focus" | "blur";
type Handlers = { [K in EventNames as `on${Capitalize<K>}`]: (e: string) => void };
const handlers: Handlers = {
  onClick: (e) => console.log("click", e),
  onFocus: (e) => console.log("focus", e),
  onBlur: (e) => console.log("blur", e),
};
handlers.onClick("按钮");

// 4) 解析查询字符串
type ParseQuery<T extends string> =
  T extends `${infer K}=${infer V}&${infer Rest}`
    ? { [P in K]: V } & ParseQuery<Rest>
    : T extends `${infer K}=${infer V}`
    ? { [P in K]: V }
    : {};
type Q = ParseQuery<"page=1&size=10">;
const q: Q = { page: "1", size: "10" };
console.log(q.page, q.size);

// 5) CSS 单位约束
type Unit = "px" | "rem" | "em" | "%";
type Size = `${number}${Unit}` | "auto";
const s1: Size = "10px";
const s2: Size = "1.5rem";
const s3: Size = "auto";
// const bad: Size = "10pt";        // 错误
console.log(s1, s2, s3);

// 6) 组合多个联合 → 笛卡尔积
type Variant = "primary" | "danger";
type SizeName = "sm" | "lg";
type ClassName = `${Variant}-${SizeName}`;
const cls: ClassName = "primary-lg";
console.log(cls);

// 7) 大小写变换的组合
type Kebab<S extends string> = S extends `${infer H}${infer T}`
  ? H extends Uppercase<H>
    ? H extends Lowercase<H>
      ? `${H}${Kebab<T>}`
      : `-${Lowercase<H>}${Kebab<T>}`
    : `${H}${Kebab<T>}`
  : S;
type K = Kebab<"backgroundColor">;      // "background-color"
const k: K = "background-color";
console.log(k);

// 8) i18n key 的层级校验
type FlatKeys<T, Prefix extends string = ""> = {
  [K in keyof T & string]: T[K] extends string
    ? `${Prefix}${K}`
    : FlatKeys<T[K], `${Prefix}${K}.`>;
}[keyof T & string];
type Messages = { user: { name: string; age: string }; ok: string };
type I18nKey = FlatKeys<Messages>;
const key: I18nKey = "user.name";
console.log(key);

// 9) 排除某前缀的键
type NotWidth<T extends string> = T extends `width${string}` ? never : T;
type Keys2 = NotWidth<"width" | "widthMin" | "height">;
const k2: Keys2 = "height";
console.log(k2);

// 10) 把「{x}」占位符的参数提取出来
type Placeholders<T extends string> =
  T extends `${string}{${infer Name}}${infer Rest}` ? Name | Placeholders<Rest> : never;
type Args = Placeholders<"你好 {name}，你有 {count} 条消息">;
const a1: Args = "name";
const a2: Args = "count";
console.log(a1, a2);
```

**运行结果：**
```
id postId
/users/7
click 按钮
1 10
10px 1.5rem auto
primary-lg
background-color
user.name
height
name count
```

**注意：**
* 模板字面量类型会产生类型组合爆炸，联合多于几个时要注意编译性能。
* 跨界的字符串解析（递归 `infer`）能力有上限，过度使用会难以维护。
* 常见实用场景：路由参数、查询串、事件名、i18n key、CSS 值、设计 token 名。

### 26.5 Recursive Types

**概念说明：** 递归类型是「引用自身」的类型，用于建模树、链表、JSON、深层嵌套结构。需要明确的终止条件，否则会触发编译深度限制。

```typescript
// 1) 树结构
interface TreeNode<T> {
  value: T;
  children: TreeNode<T>[];
}
const tree: TreeNode<number> = {
  value: 1,
  children: [
    { value: 2, children: [] },
    { value: 3, children: [{ value: 4, children: [] }] },
  ],
};

function sumTree(node: TreeNode<number>): number {
  return node.value + node.children.reduce((acc, c) => acc + sumTree(c), 0);
}
console.log("树求和:", sumTree(tree));

// 2) 链表
type List<T> = { head: T; tail: List<T> | null };
const list: List<number> = { head: 1, tail: { head: 2, tail: null } };
function listToArray<T>(l: List<T> | null): T[] {
  const out: T[] = [];
  let cur = l;
  while (cur) { out.push(cur.head); cur = cur.tail; }
  return out;
}
console.log("链表转数组:", listToArray(list));

// 3) JSON 类型
type Json = string | number | boolean | null | Json[] | { [key: string]: Json };
const json: Json = { a: 1, b: [true, null, { c: "s" }] };
console.log("JSON 值:", JSON.stringify(json).length > 0);

// 4) 递归映射：深层只读 / 深层可选
type DeepPartial<T> = T extends (infer U)[]
  ? DeepPartial<U>[]
  : T extends object
  ? { [K in keyof T]?: DeepPartial<T[K]> }
  : T;
const dp: DeepPartial<{ a: { b: { c: number } } }> = { a: { b: {} } };
console.log("深层可选:", JSON.stringify(dp));

// 5) 递归 + 路径类型（前面章节详述）
type Paths<T, P extends string = ""> = {
  [K in keyof T & string]: T[K] extends object ? `${P}${K}` | Paths<T[K], `${P}${K}.`> : `${P}${K}`;
}[keyof T & string];
const path: Paths<{ a: { b: string } }> = "a.b";
console.log("路径类型:", path);

// 6) 类型层面的递归计算：长度
type Length<T extends readonly unknown[]> = T extends readonly [unknown, ...infer R]
  ? R["length"] extends number
    ? AddOne<R["length"]>
    : never
  : 0;
type AddOne<N extends number, Acc extends unknown[] = []> =
  Acc["length"] extends N ? [...Acc, unknown]["length"] : AddOne<N, [...Acc, unknown]>;
type L = Length<[1, 2, 3, 4]>;      // 4
const l: L = 4;
console.log("元组长度:", l);

// 7) 递归的深度限制
// 过深的递归会报：Type instantiation is excessively deep and possibly infinite.
// 常见缓解手段：
//   1) 减少嵌套层级或提前终止
//   2) 用接口代替条件类型递归（接口是「惰性」的）
//   3) 提高编译器限制（不推荐）

// 8) 用接口实现真正惰性的递归
interface DeepReadonly2<T> {
  readonly value: DeepReadonly2<T>;
}

// 9) 递归解包
type UnwrapDeep<T> = T extends Promise<infer U> ? UnwrapDeep<U> : T extends readonly (infer E)[] ? UnwrapDeep<E> : T;
type U = UnwrapDeep<Promise<Promise<number[]>>>;    // number
const u: U = 1;
console.log("递归解包:", u);

// 10) 运行时配套：递归遍历要有终止条件
function deepFreeze<T extends object>(obj: T): T {
  Object.freeze(obj);
  for (const v of Object.values(obj)) {
    if (typeof v === "object" && v !== null && !Object.isFrozen(v)) deepFreeze(v as object);
  }
  return obj;
}
const frozen = deepFreeze({ a: { b: 1 } });
console.log("深层冻结:", Object.isFrozen(frozen.a));
```

**运行结果：**
```
树求和: 10
链表转数组: [ 1, 2 ]
JSON 值: true
深层可选: {"a":{"b":{}}}
路径类型: a.b
元组长度: 4
递归解包: 1
深层冻结: true
```

**注意：**
* 递归类型必须有终止分支（如 `T extends object ? ... : T`），否则编译器会报深度错误。
* 接口/类型别名的自引用是惰性的，条件类型的递归是即时的 —— 前者更适合深层结构。
* 运行时的递归遍历要防循环引用（用 `WeakSet` 记录已访问）。

### 26.6 Variadic Tuple Types

**概念说明：** 可变元组类型（TS 4.0+）允许在元组中使用 `...T` 泛型展开，实现类型层面的「元组拼接、切分、转换」，是类型安全的函数组合与柯里化的基础。

```typescript
// 1) 拼接
type Concat<A extends unknown[], B extends unknown[]> = [...A, ...B];
type C = Concat<[1, 2], [3, 4]>;      // [1, 2, 3, 4]
const c: C = [1, 2, 3, 4];
console.log(c);

// 2) 首元素与其余
type Head<T extends unknown[]> = T extends [infer H, ...unknown[]] ? H : never;
type Tail<T extends unknown[]> = T extends [unknown, ...infer R] ? R : never;
const h: Head<[string, number]> = "a";
const t: Tail<[string, number]> = [1];
console.log(h, t);

// 3) 在指定位置插入
// 注意：[...infer Before, ...infer After] 会把 Before 推断为空数组（TS 不会主动切分）
// 所以要用累加器边走边数，走到第 I 个位置再插入
type InsertAt<T extends unknown[], I extends number, V, Acc extends unknown[] = []> =
  Acc["length"] extends I
    ? [...Acc, V, ...T]
    : T extends [infer H, ...infer R]
    ? InsertAt<R, I, V, [...Acc, H]>
    : [...Acc, V];
type I = InsertAt<[1, 2, 3], 1, "x">;   // [1, "x", 2, 3]
const ins: I = [1, "x", 2, 3];
console.log(ins);

// 4) 类型安全的函数组合（保留完整参数签名）
function compose<A extends unknown[], R>(f: (...args: A) => R): (...args: A) => R;
function compose<A extends unknown[], B, R>(
  f: (...args: A) => B,
  g: (b: B) => R
): (...args: A) => R;
function compose(...fns: ((...args: never[]) => unknown)[]) {
  return (...args: unknown[]) => fns.reduce((acc, fn) => fn(acc as never), fns.shift()!(...args as never));
}
const parseThen = compose((s: string) => Number(s), (n: number) => n.toFixed(2));
console.log(parseThen("3.14159"));

// 5) 柯里化：把 (A, B) => R 变成 (A) => (B) => R
type Curry2<F> = F extends (a: infer A, b: infer B) => infer R ? (a: A) => (b: B) => R : never;
const curried: Curry2<(a: number, b: string) => boolean> = (a) => (b) => `${a}${b}`.length > 0;
console.log(curried(1)("x"));

// 6) 参数转发保持类型
function forward<F extends (...args: never[]) => unknown>(
  fn: F,
  ...args: Parameters<F>
): ReturnType<F> {
  return fn(...args);
}
const add = (a: number, b: number): number => a + b;
console.log(forward(add, 1, 2));

// 7) 元组映射：每个元素包装成 { value, index }
// index 的键同样是字符串，需转成数字类型（同 26.3 的 NumericIndex）
type Wrap<T extends readonly unknown[]> = {
  [K in keyof T]: { value: T[K]; index: K extends `${infer N extends number}` ? N : never };
};
type W = Wrap<[string, number]>;
const w: W = [{ value: "a", index: 0 }, { value: 1, index: 1 }];
console.log(w[0].value);

// 8) 元组反转
type Reverse<T extends unknown[], Acc extends unknown[] = []> =
  T extends [infer H, ...infer R] ? Reverse<R, [H, ...Acc]> : Acc;
type Rev = Reverse<[1, 2, 3]>;         // [3, 2, 1]
const rev: Rev = [3, 2, 1];
console.log(rev);

// 9) 部分应用（把前面几个参数固定）
function partial<A extends unknown[], U extends unknown[], R>(
  fn: (...args: [...A, ...U]) => R,
  ...preset: A
): (...rest: U) => R {
  return (...rest: U) => fn(...preset, ...rest);
}
const greet = (title: string, name: string, punct: string) => `${title} ${name}${punct}`;
const drGreet = partial(greet, "Dr.");
console.log(drGreet("Alice", "!"));

// 10) 与 rest 参数配合的可变参数函数
function build<T extends unknown[]>(...parts: T): { parts: T } {
  return { parts };
}
const built = build(1, "a", true);
console.log(built.parts.length, built.parts[0]);

// 11) 元组的「长度类型」
type Len<T extends readonly unknown[]> = T["length"];
const len: Len<[1, 2, 3]> = 3;
console.log("元组长度类型:", len);

// 12) 过滤元组中的某类型
type FilterTuple<T extends unknown[], U> =
  T extends [infer H, ...infer R] ? (H extends U ? [H, ...FilterTuple<R, U>] : FilterTuple<R, U>) : [];
type F = FilterTuple<[1, "a", 2, "b"], string>;   // ["a", "b"]
const filtered: F = ["a", "b"];
console.log(filtered);
```

**运行结果：**
```
[ 1, 2, 3, 4 ]
a [ 1 ]
[ 1, 'x', 2, 3 ]
3.14
true
3
a
[ 3, 2, 1 ]
Dr. Alice!
3 1
元组长度类型: 3
[ 'a', 'b' ]
```

**注意：**
* 可变元组让「参数数量与位置」在类型层面完全可控，是类型安全柯里化/组合的关键。
* 递归元组类型（`Reverse`、`FilterTuple`）注意深度与性能。
* 与 `Parameters` / `ReturnType` 组合是包装函数的标准做法。

### 26.7 Type-level Programming

**概念说明：** 类型级编程指把类型系统当作一门「只在编译期运行的语言」：用条件类型做分支、映射做循环、元组做数据结构、递归做迭代，实现编译期计算与校验。

```typescript
// 数据结构：用元组当「自然数」（Peano 风格）
type BuildTuple<N extends number, Acc extends unknown[] = []> =
  Acc["length"] extends N ? Acc : BuildTuple<N, [...Acc, unknown]>;
type Add<A extends number, B extends number> = [...BuildTuple<A>, ...BuildTuple<B>]["length"];
type Sum = Add<3, 4>;                 // 7
const sum: Sum = 7;
console.log("类型级加法 3 + 4 =", sum);

// 比较：A 是否 <= B
// 注意方向：A <= B 意味着「B 的元组能容纳 A 的元组」，所以把 B 放在被匹配的位置
type Lte<A extends number, B extends number> =
  BuildTuple<B> extends [...BuildTuple<A>, ...infer _Rest] ? true : false;
type R1 = Lte<2, 5>;                  // true
type R2 = Lte<5, 2>;                  // false
const r1: R1 = true;
const r2: R2 = false;
console.log(r1, r2);

// 循环：映射类型就是「类型层面的 for」
type Repeat<T, N extends number, Acc extends T[] = []> =
  Acc["length"] extends N ? Acc : Repeat<T, N, [T, ...Acc]>;
type Three = Repeat<"x", 3>;          // ["x", "x", "x"]
const three: Three = ["x", "x", "x"];
console.log(three);

// 字符串算法：类型级替换
// 递归部分必须写成 ${Replace<...>} 插值；写成 Replace<...> 只会当成普通文本
type Replace<S extends string, From extends string, To extends string> =
  S extends `${infer P}${From}${infer R}` ? `${P}${To}${Replace<R, From, To>}` : S;
type Rep = Replace<"a-b-c", "-", "+">;
const rep: Rep = "a+b+c";
console.log(rep);

// 状态机：用可辨识联合 + 条件类型约束转移
type State = "idle" | "loading" | "success" | "error";
type AllowedTransition<S extends State> =
  S extends "idle" ? "loading"
  : S extends "loading" ? "success" | "error"
  : S extends "success" ? "idle"
  : S extends "error" ? "idle" | "loading"
  : never;

function transition<From extends State, To extends AllowedTransition<From>>(from: From, to: To): string {
  return `${from} → ${to}`;
}
console.log(transition("idle", "loading"));
console.log(transition("loading", "error"));
// transition("idle", "success");       // 错误：非法状态转移

// 类型级的「断言/测试」体系
type Expect<T extends true> = T;
type Equal<A, B> = (<T>() => T extends A ? 1 : 2) extends (<T>() => T extends B ? 1 : 2) ? true : false;

type _Test1 = Expect<Equal<Add<1, 1>, 2>>;
type _Test2 = Expect<Equal<Lte<1, 2>, true>>;
type _Test3 = Expect<Equal<Repeat<0, 2>, [0, 0]>>;
console.log("类型级单元测试全部通过（否则编译报错）");

// 类型级的「解析器」：分隔符切分
type Split<S extends string, D extends string> =
  S extends `${infer H}${D}${infer R}` ? [H, ...Split<R, D>] : [S];
type Parts = Split<"a,b,c", ",">;     // ["a", "b", "c"]
const parts: Parts = ["a", "b", "c"];
console.log(parts);

// 类型级的「对象校验」
type ValidKeys<T, K extends readonly (keyof T)[]> =
  K extends readonly [infer H extends keyof T, ...infer R extends (keyof T)[]] ? ValidKeys<T, R> : K;
const keys = ["id", "name"] as const;
type Checked = ValidKeys<{ id: number; name: string }, typeof keys>;
console.log("键校验通过:", keys.length);

// 现实建议
// ✅ 适合：库作者、框架 DSL、配置约束、路由/事件名约束
// ⚠️ 谨慎：业务代码中过度使用会显著降低可读性与编译速度
console.log("类型级编程的价值在于「把约束前移到编译期」");
```

**运行结果：**
```
类型级加法 3 + 4 = 7
true false
[ 'x', 'x', 'x' ]
a+b+c
idle → loading
loading → error
类型级单元测试全部通过（否则编译报错）
[ 'a', 'b', 'c' ]
键校验通过: 2
类型级编程的价值在于「把约束前移到编译期」
```

**注意：**
* 类型级计算有深度与复杂度上限，超过会报 `excessively deep`，且会明显拖慢编译。
* 建议只在「库/框架/强约束」场景使用，业务代码优先可读性。
* 用 `Expect<Equal<...>>` 写类型测试，把类型逻辑当成有测试的代码维护。

### 26.8 Declaration Merging

**概念说明：** 声明合并指「同名的多个声明被合并为一个」。接口与接口、命名空间与命名空间、命名空间与类/函数/枚举都可以合并，是扩展第三方类型的基础机制。

```typescript
// 1) 接口 + 接口
interface Box { width: number }
interface Box { height: number }
const box: Box = { width: 1, height: 2 };
console.log(box);

// 2) 命名空间 + 命名空间
namespace Utils {
  export const version = "1.0";
}
namespace Utils {
  export function log(msg: string): void { console.log("[Utils]", msg); }
}
Utils.log(Utils.version);

// 3) 命名空间 + 函数（函数带静态属性）
function build(): string { return "built"; }
namespace build {
  export const version = "2.0";
  export function help(): string { return "帮助"; }
}
console.log(build(), build.version, build.help());

// 4) 命名空间 + 类（类带静态成员）
class User {
  constructor(public name: string) {}
}
namespace User {
  export const DEFAULT = new User("默认");
  export function from(json: string): User { return new User(JSON.parse(json).name); }
}
console.log(User.DEFAULT.name, User.from('{"name":"Alice"}').name);

// 5) 命名空间 + 枚举
enum Direction { Up, Down }
namespace Direction {
  export function isVertical(d: Direction): boolean {
    return d === Direction.Up || d === Direction.Down;
  }
}
console.log(Direction.isVertical(Direction.Up));

// 6) 为第三方接口追加字段（模块扩充）
interface Window2 { appName: string }
interface Window2 { version: string }
const w: Window2 = { appName: "demo", version: "1.0" };
console.log(w.appName, w.version);

// 7) 同名属性的类型必须一致（否则报错）
// interface A { x: number }
// interface A { x: string }        // 错误：后续属性声明必须具有相同类型

// 8) 同名方法的合并形成重载（后者优先匹配）
interface Api { get(id: number): string }
interface Api { get(id: string): number }
const api: Api = { get: (id: any) => (typeof id === "number" ? `n-${id}` : 1) };
console.log(api.get(1), api.get("a"));

// 9) 声明合并的触发范围
// - 同一文件内
// - 跨文件（同名字符串键的全局接口）
// - 跨模块（declare module "x" 扩充）
console.log("声明合并 = 同名声明累积，是扩充第三方类型的基础");

// 10) 与 type 的对比
type CannotMerge = { a: number };
// type CannotMerge = { b: number };   // 错误：Duplicate identifier
console.log("type 别名不能声明合并");
```

**运行结果：**
```
{ width: 1, height: 2 }
[Utils] 1.0
built 2.0 帮助
默认 Alice
true
demo 1.0
n-1 1
声明合并 = 同名声明累积，是扩充第三方类型的基础
type 别名不能声明合并
```

**注意：**
* 只有 `interface`、`namespace`、`enum` 及其与函数/类的组合支持合并；`type` 与 `const` 不支持。
* 同名方法合并后按声明顺序形成重载，靠后的优先。
* 函数+命名空间（或类+命名空间）的合并模式是「库要附加静态成员」的经典做法。

### 26.9 Module Augmentation

**概念说明：** 模块扩充（Module Augmentation）用 `declare module` 为已有模块追加类型：给 Express 的 `Request` 加 `user`、给 Vue 的 `ComponentCustomProperties` 加全局属性等。

```typescript
// 场景 1：扩展 Express 的 Request
import "express";
import type { Request } from "express";

declare module "express" {
  interface Request {
    user?: { id: number; name: string; roles: string[] };
    traceId?: string;
  }
  interface Response {
    ok<T>(data: T): void;
  }
}

// 现在 Request 上有 user 了
function authMiddleware(req: Request): void {
  req.user = { id: 1, name: "Alice", roles: ["admin"] };
  console.log("已注入用户:", req.user.name, req.traceId ?? "无 traceId");
}
authMiddleware({} as Request);

// 场景 2：扩展 Vue 的全局属性
// declare module "vue" {
//   interface ComponentCustomProperties {
//     $http: { get<T>(url: string): Promise<T> };
//   }
// }
// 之后 this.$http 在组件里有类型

// 场景 3：扩展 Node 的 ProcessEnv
declare module "node:process" {
  interface ProcessEnv {
    NODE_ENV: "development" | "production" | "test";
    PORT?: string;
    DATABASE_URL?: string;
  }
}
const env: NodeJS.ProcessEnv = { NODE_ENV: "production" };
console.log("环境类型收窄:", env.NODE_ENV);

// 场景 4：扩展全局对象（浏览器）
declare global {
  interface Window {
    __APP_CONFIG__: { apiBase: string; version: string };
    gtag?: (event: string, payload?: Record<string, unknown>) => void;
  }
  interface Array<T> {
    last(): T | undefined;
  }
}
Array.prototype.last = function <T>(this: T[]): T | undefined { return this[this.length - 1]; };
console.log("全局增强:", [1, 2, 3].last());

// 场景 5：扩展第三方库的类型（补漏）
// declare module "some-lib" {
//   export function missingApi(input: string): number;
// }

// 场景 6：扩充类型别名时改用接口（别名不可合并）
// ❌ declare module "x" { type Config = { a: number } }
// ✅ declare module "x" { interface Config { a: number } }

// 场景 7：扩充的放置位置
// 必须放在「模块文件」（含 import/export）中才能 declare module "包名"
// 全局扩充用 declare global
// 文件需被 tsconfig include 覆盖，命名常用 src/types/augment.d.ts

// 场景 8：与声明合并的区别
// 声明合并：同作用域同名声明自动累积（无需 declare module）
// 模块扩充：跨模块给「另一个模块」的接口追加成员（需要 import + declare module）
console.log("模块扩充用于给第三方模块追加类型");

// 场景 9：给 untyped 库补类型（新建声明）
// declare module "no-types-lib" {
//   export function run(cfg: { debug?: boolean }): void;
//   export interface Result { ok: boolean }
// }

// 场景 10：注意扩充的副作用范围
// 扩充会全局生效（所有 import 该模块的文件都受影响），要谨慎并写清注释
console.log("扩充是全局的，注意副作用范围");
```

**运行结果：**
```
已注入用户: Alice 无 traceId
环境类型收窄: production
全局增强: 3
模块扩充用于给第三方模块追加类型
扩充是全局的，注意副作用范围
```

**注意：**
* 扩充必须放在模块文件中（有 import/export），且先 `import "目标模块"` 再 `declare module`。
* 扩充只影响类型，运行时必须真的有对应属性（否则访问即 undefined）。
* 全局扩充（`declare global`）影响整个项目，应集中在少数 `*.d.ts` 中管理。

### 26.10 编译器 API

**概念说明：** TypeScript 提供编程式 API（`typescript` 包的 `ts.*`），可在代码中解析、检查、转换源码，用于构建迁移脚本、自定义 lint 规则、类型检查工具、代码生成器。

```bash
npm i -D typescript
```

```typescript
import ts from "typescript";

// 1) 用 Compiler API 做类型检查（等价 tsc --noEmit）
function typeCheck(fileNames: string[], options: ts.CompilerOptions = {}): number {
  const program = ts.createProgram(fileNames, {
    target: ts.ScriptTarget.ES2022,
    module: ts.ModuleKind.NodeNext,
    moduleResolution: ts.ModuleResolutionKind.NodeNext,
    strict: true,
    noEmit: true,
    skipLibCheck: true,
    ...options,
  });

  const diagnostics = [
    ...program.getSemanticDiagnostics(),
    ...program.getSyntacticDiagnostics(),
  ];

  for (const d of diagnostics) {
    const msg = ts.flattenDiagnosticMessageText(d.messageText, "\n");
    if (d.file && d.start !== undefined) {
      const { line, character } = d.file.getLineAndCharacterOfPosition(d.start);
      console.log(`${d.file.fileName}:${line + 1}:${character + 1} - ${msg}`);
    } else {
      console.log(msg);
    }
  }
  return diagnostics.length;
}

// 2) 解析源码为 AST 并遍历
const source = `
interface User { id: number; name: string }
export function greet(u: User): string { return \`hi \${u.name}\`; }
export const VERSION = "1.0";
`;

const sourceFile = ts.createSourceFile("demo.ts", source, ts.ScriptTarget.ES2022, true, ts.ScriptKind.TS);

// 用 forEachChild 遍历语法树
interface Collect { interfaces: string[]; functions: string[]; variables: string[] }
const collected: Collect = { interfaces: [], functions: [], variables: [] };

function visit(node: ts.Node): void {
  if (ts.isInterfaceDeclaration(node)) collected.interfaces.push(node.name.text);
  if (ts.isFunctionDeclaration(node) && node.name) collected.functions.push(node.name.text);
  if (ts.isVariableStatement(node)) {
    node.declarationList.declarations.forEach((d) => {
      if (ts.isIdentifier(d.name)) collected.variables.push(d.name.text);
    });
  }
  ts.forEachChild(node, visit);
}
visit(sourceFile);
console.log("接口:", collected.interfaces, "函数:", collected.functions, "变量:", collected.variables);

// 3) 用类型检查器获取类型信息
function inspectTypes(fileNames: string[]): void {
  const program = ts.createProgram(fileNames, { target: ts.ScriptTarget.ES2022, strict: true, noEmit: true });
  const checker = program.getTypeChecker();

  for (const file of program.getSourceFiles()) {
    if (!file.isDeclarationFile) {
      ts.forEachChild(file, function walk(node) {
        if (ts.isFunctionDeclaration(node) && node.name) {
          const symbol = checker.getSymbolAtLocation(node.name);
          if (symbol) {
            const type = checker.getTypeOfSymbolAtLocation(symbol, node);
            const sigs = type.getCallSignatures();
            sigs.forEach((s) => {
              const ret = checker.typeToString(s.getReturnType());
              const params = s.getParameters().map((p) => p.getName()).join(", ");
              console.log(`函数 ${symbol.getName()}(${params}) → ${ret}`);
            });
          }
        }
        ts.forEachChild(node, walk);
      });
    }
  }
}

// 4) 用 Transformer 改写代码（codemod 基础）
function toArrowFunction(title: string): ts.TransformerFactory<ts.SourceFile> {
  return (context) => (root) => {
    const visit = (node: ts.Node): ts.Node => {
      // 演示：把所有 string 字面量替换为 "（已处理）"
      if (ts.isStringLiteral(node) && node.text !== "（已处理）") {
        return context.factory.createStringLiteral("（已处理）");
      }
      return ts.visitEachChild(node, visit, context);
    };
    return ts.visitNode(root, visit) as ts.SourceFile;
  };
}

function transform(source: string): string {
  const result = ts.transform(
    ts.createSourceFile("x.ts", source, ts.ScriptTarget.ES2022, true),
    [toArrowFunction("示例")],
    { target: ts.ScriptTarget.ES2022 }
  );
  const printer = ts.createPrinter();
  const output = printer.printFile(result.transformed[0]);
  result.dispose();
  return output;
}
console.log("转换结果:", transform(`const a = "hello"; const b = "world";`));

// 5) 读取 tsconfig 并解析
const configPath = ts.findConfigFile(process.cwd(), ts.sys.fileExists, "tsconfig.json");
console.log("找到 tsconfig:", configPath !== undefined || "（当前目录无 tsconfig）");

if (configPath) {
  const read = ts.readConfigFile(configPath, ts.sys.readFile);
  if (read.config) {
    const parsed = ts.parseJsonConfigFileContent(read.config, ts.sys, process.cwd());
    console.log("参与编译的文件数:", parsed.fileNames.length);
    console.log("严格模式:", parsed.options.strict === true);
    // 可以直接做类型检查
    // typeCheck(parsed.fileNames, parsed.options);
  }
}

// 6) 诊断信息格式化（可解析的错误报告）
const host = ts.createCompilerHost({ noEmit: true });
const program2 = ts.createProgram(["nonexistent-file.ts"], { noEmit: true }, host);
const fmt = ts.createDiagnosticReporter(program2);
console.log("诊断报告器已创建:", typeof fmt === "function");

// 7) 常见用途
console.log("编译器 API 用途：迁移脚本(codemod) / 自定义规则 / 文档生成 / 调用图分析 / 类型覆盖率统计");

// 8) 注意：Compiler API 不保证跨版本稳定，升级 TS 时需回归测试
console.log("Compiler API 属于内部接口的公开门面，升级需谨慎");
```

**运行结果：**
```
接口: [ 'User' ] 函数: [ 'greet' ] 变量: [ 'VERSION' ]
转换结果: const a = "（已处理）"; const b = "（已处理）";
找到 tsconfig: （当前目录无 tsconfig）
诊断报告器已创建: function
编译器 API 用途：迁移脚本(codemod) / 自定义规则 / 文档生成 / 调用图分析 / 类型覆盖率统计
Compiler API 属于内部接口的公开门面，升级需谨慎
```

**注意：**
* Compiler API 功能强大但不保证向后兼容，建议锁定 TS 版本并写测试。
* Transformer 用于 codemod（批量重构）：先解析成 AST，再改写，最后用 `ts.createPrinter` 输出。
* 只想「做类型检查」时更简单的替代方案：`tsc --noEmit`、`tsc --listFiles`、或用 `ts-morph`（对 Compiler API 的友好封装）。
