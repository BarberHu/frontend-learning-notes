---
aliases:
  - Day 4｜call / apply / bind
---

## 0. 快速回顾  
  
### 这篇讲什么  
这篇主要讲 3 个手动控制 `this` 的方法：  
  
- `call`  
- `apply`  
- `bind`  
  
重点不是死记 API，而是搞清楚三件事：  
  
1. 它们会不会立刻执行  
2. 它们怎么传参数  
3. `bind` 过的函数，后面还能不能再改 `this`  
  
---  
  
### 先看这 3 条  
1. `call` / `apply`：**立刻执行函数**  
2. `bind`：**不立刻执行，而是返回一个绑定好 `this` 的新函数**  
3. **函数一旦被 `bind`，后续 `call / apply / bind` 都改不了它的 `this`**  
  
---  
  
### 我上次最容易错的点  
1. 以为 `bind` 和 `call` 一样，都是“马上执行”  
2. 以为 `bind` 之后，再 `call(obj2)` 可以把 `this` 改成 `obj2`  
3. 只看“打印了什么”，没看“表达式最终返回了什么”  
  
---  
  
### 30 秒自测  
直接口答下面 4 个结论对不对：  
  
1. `call` 会立刻执行函数    
2. `apply` 传参时用数组    
3. `bind` 会立刻执行函数    
4. `bind` 过的函数，后面还能被 `call` 改掉 `this`  
  
答案：  
  
- 1：对  
- 2：对  
- 3：错  
- 4：错  
  
---  
  
## 1. 核心规则  
  
### 1.1 `call`  
作用：**立刻调用函数，并手动指定 `this`**  

 
function say(age) {  
  console.log(this.name, age);  
}  
  
const obj = { name: "阳哥" };  
  
say.call(obj, 20);

输出：

阳哥 20

结论：

- `call` 会立刻执行
- 参数是一个个传

---

### 1.2 `apply`

作用：**立刻调用函数，并手动指定 `this`**  
和 `call` 的区别只在参数形式。

function say(age, city) {  
  console.log(this.name, age, city);  
}  
  
const obj = { name: "阳哥" };  
  
say.apply(obj, [20, "南京"]);

输出：

阳哥 20 南京

结论：

- `apply` 会立刻执行
- 参数要放进数组里传

---

### 1.3 `bind`

作用：**不立刻执行，而是返回一个新的绑定函数**

function say(age) {  
  console.log(this.name, age);  
}  
  
const obj = { name: "阳哥" };  
  
const fn = say.bind(obj, 20);  
fn();

输出：

阳哥 20

结论：

- `bind` 不会立刻执行
- `bind` 返回一个新函数
- 新函数的 `this` 固定为 `obj`
- 参数也可以预先绑定一部分

---

### 1.4 三者最核心区别

call  -> 立刻执行，参数一个个传  
apply -> 立刻执行，参数数组传  
bind  -> 不立刻执行，返回新函数

---

### 1.5 `bind` 的 `this` 不能被后续改掉

这是今天最关键的一条。

const name = "全局";  
  
const obj = {  
  name: "阳哥",  
  say() {  
    console.log(this.name);  
  }  
};  
  
const obj2 = {  
  name: "小王"  
};  
  
const fn = obj.say.bind(obj);  
fn.call(obj2);

输出：

阳哥

原因：

- `fn` 已经是 `bind` 生成的绑定函数
- 它的 `this` 已经固定为 `obj`
- 后面的 `call(obj2)` 改不掉这个 `this`

---

## 2. 关键易错点

### 易错点 1：把 `bind` 当成“立刻执行”

错误理解：

const fn = say.bind(obj);

误以为这里就已经执行了。

正确理解：

- 这里只是返回了一个新函数
- 真正执行要等到后面：

fn();

---

### 易错点 2：以为 `bind` 之后还能被 `call` 改掉

错误理解：

const fn = say.bind(obj);  
fn.call(obj2);

误以为输出 `obj2.name`

正确理解：

- `fn` 已经不是普通函数
- 它是绑定函数
- 后面的 `call / apply / bind` 都改不了它的 `this`

---

### 易错点 3：以为二次 `bind` 能覆盖第一次 `bind`

function say() {  
  console.log(this.name);  
}  
  
const fn1 = say.bind({ name: "阳哥" });  
const fn2 = fn1.bind({ name: "小王" });  
  
fn2();

输出：

阳哥

不是 `小王`。

原因：

- 第一次 `bind` 已经把 `this` 固定住了
- 第二次 `bind` 不能覆盖第一次绑定的 `this`

---

### 易错点 4：只看打印结果，不看返回值

function test() {  
  console.log(this.name);  
}  
  
const f1 = test.bind({ name: "阳哥" });  
const result = f1.call({ name: "小王" });  
console.log(result);

输出：

阳哥  
undefined

这里要分清两层：

1. `f1.call(...)` 执行时打印的是 `阳哥`
2. `result` 的值是 `undefined`

因为：

- `f1` 的 `this` 改不掉
- `test` 本身没有 `return`

---

## 3. 典型例题

### 例题 1：`call`

const obj = {  
  name: "阳哥"  
};  
  
function say() {  
  console.log(this.name);  
}  
  
say.call(obj);

输出：

阳哥

原因：

- `call` 立刻执行
- 并把 `this` 改成 `obj`

---

### 例题 2：`apply`

function add(a, b) {  
  console.log(this.name, a, b);  
}  
  
add.apply({ name: "阳哥" }, [1, 2]);

输出：

阳哥 1 2

原因：

- `apply` 立刻执行
- 参数通过数组传入

---

### 例题 3：`bind`

const obj = {  
  name: "阳哥"  
};  
  
function show(age) {  
  console.log(this.name, age);  
}  
  
const fn = show.bind(obj, 20);  
fn();

输出：

阳哥 20

原因：

- `bind` 返回一个新函数
- `this` 绑定到 `obj`
- `20` 也被预先绑定了

---

### 例题 4：`bind` 后再 `call`

function say() {  
  console.log(this.name);  
}  
  
const fn = say.bind({ name: "阳哥" });  
fn.call({ name: "小王" });

输出：

阳哥

原因：

- `bind` 已经锁死 `this`
- `call` 改不了

---

### 例题 5：返回值问题

function test() {  
  console.log(this.name);  
}  
  
const f1 = test.bind({ name: "阳哥" });  
const f2 = f1.call({ name: "小王" });  
  
console.log("f2 =", f2);

输出：

阳哥  
f2 = undefined

原因：

- 执行时打印 `阳哥`
- `test` 没有返回值，所以 `f2` 是 `undefined`

---

## 4. 判断流程

以后看到 `call / apply / bind`，直接按这个顺序判断：

### 第一步：看是不是立刻执行

fn.call(...)  
fn.apply(...)

这两个是 **立刻执行**

const newFn = fn.bind(...)

这个是 **不执行，只返回新函数**

---

### 第二步：看参数怎么传

- `call(obj, a, b, c)`：一个个传
- `apply(obj, [a, b, c])`：数组传
- `bind(obj, a, b)`：先绑定，后面执行时再补剩余参数

---

### 第三步：看函数是不是已经被 `bind`

如果是普通函数：

- `call / apply` 可以改 `this`

如果已经是绑定函数：

- 后续 `call / apply / bind` 都改不了 `this`

---

### 第四步：别忘了看返回值

- `call / apply` 的返回值：**原函数执行结果**
- `bind` 的返回值：**一个新函数**

所以看到这种代码：

const result = fn.call(...)

一定要追问：

- 原函数有没有 `return`
- `result` 最终是什么

---

## 5. 一句话总结

`call / apply` 是“立刻改 this 并执行”，`bind` 是“先固定 this，返回新函数，之后再执行”；而且函数一旦被 `bind`，后面基本改不动它的 `this`。

---

## 6. 小练习

### 练习 1

写出输出和原因：

function say() {  
  console.log(this.name);  
}  
  
say.call({ name: "阳哥" });

---

### 练习 2

写出输出和原因：

function add(a, b) {  
  console.log(this.name, a, b);  
}  
  
add.apply({ name: "阳哥" }, [3, 4]);

---

### 练习 3

写出输出和原因：

function show() {  
  console.log(this.name);  
}  
  
const fn = show.bind({ name: "阳哥" });  
fn();

---

### 练习 4

写出输出和原因：

function test() {  
  console.log(this.name);  
}  
  
const f1 = test.bind({ name: "阳哥" });  
f1.call({ name: "小王" });

---

### 练习 5

写出输出和原因：

function test() {  
  console.log(this.name);  
}  
  
const f1 = test.bind({ name: "阳哥" });  
const result = f1.call({ name: "小王" });  
  
console.log(result);

---

## 7. 明天衔接

明天切到 **闭包（closure）**。

今天讲的是：

- 函数执行时，`this` 指向谁

明天讲的是：

- 函数创建后，为什么还能“记住”外层变量

也就是从：

函数运行时我是谁

切到：

函数创建后我还能记住谁

这是 JavaScript 另一条主线，后面学：

- 回调
- 防抖节流
- 模块化
- React Hooks

都会反复用到。