
## 0. 快速回顾  
  
### 这篇讲什么  
这篇主要解决两个问题：  
  
1. 普通函数的 `this` 到底看谁  
2. 箭头函数为什么不受 `call` 影响  
  
---  
  
### 先看这 3 条  
- 普通函数的 `this` 看**最终调用方式**  
- 箭头函数**没有自己的 `this`**，它用的是定义时外层的 `this`  
- `call / apply / bind` **不能修改箭头函数的 `this`**  
  
---  
  
### 我上次最容易错的点  
- 把“谁返回了函数”误当成“谁调用了函数”  
- 以为“箭头函数优先级高于 `call`”  
  
---  
  
### 30 秒自测  
#### 题 1  

const a = obj.fn;  
a();

#### 题 2

const a = obj.fn();  
a.call({ name: "小王" });

---

## 1. 核心规则

### 规则 1：普通函数看调用方式

#### 结论

普通函数的 `this` 不是定义时决定的，而是调用时决定的。

#### 本质

你不要看它“写在哪”，要看它“最后怎么执行”。

#### 一句话判断法

普通函数：谁调用它，this 就指向谁

#### 典型形式

obj.fn()        // this -> obj  
fn()            // this -> 全局对象（非严格）/ undefined（严格）  
fn.call(x)      // this -> x  
fn.apply(x)     // this -> x  
fn.bind(x)()    // this -> x

---

### 规则 2：箭头函数没有自己的 this

#### 结论

箭头函数不会在调用时重新绑定 `this`。

#### 本质

箭头函数会直接使用“定义它时”外层作用域中的 `this`。

#### 一句话判断法

箭头函数：不看谁调用，只看它出生时外层的 this

---

### 规则 3：call 改不了箭头函数的 this

#### 结论

`call / apply / bind` 只能改普通函数的 `this`，改不了箭头函数的 `this`。

#### 本质

不是箭头函数“优先级更高”，而是箭头函数根本没有自己的 `this` 可改。

#### 一句话判断法

call 能改普通函数，改不了箭头函数

---

## 2. 关键易错点

### 易错点 1：把“返回者”当成“调用者”

#### 错误说法

这个函数是 obj 返回的，所以 this 还是 obj

#### 正确说法

函数从谁那里返回出来不重要，关键看它最后怎么调用

#### 为什么容易错

因为你看到它“来自 obj”，就会下意识觉得它“还属于 obj”。  
但这是两回事：

返回者 ≠ 调用者

---

### 易错点 2：以为“箭头函数优先级高于 call”

#### 错误说法

箭头函数的优先级高于 call

#### 正确说法

箭头函数没有自己的 this，所以 call 改不了它的 this

#### 为什么容易错

因为现象上看，写了 `.call(...)` 却没生效。  
但本质不是“优先级打架”，而是 `call` 根本没有可修改的目标。

---

### 易错点 3：把“调用返回值”说成“还在调用原函数”

#### 错误说法

a 还是在调用 fn

#### 正确说法

obj.fn() 先执行，然后返回一个新函数；a() 调用的是返回出来的那个函数

#### 为什么容易错

因为脑子里把这三个动作混成一团了：

调用 fn  
返回新函数  
调用新函数

---

## 3. 典型例题

### 例题 1：普通函数返回后独立调用

const name = "全局";  
  
const obj = {  
  name: "阳哥",  
  say: function () {  
    function inner() {  
      console.log(this.name);  
    }  
    return inner;  
  }  
};  
  
const f = obj.say();  
f();

#### 输出

全局

#### 原因

obj.say() -> say 的 this = obj  
say 返回 inner  
f = inner  
f() -> 普通函数独立调用  
=> this = 全局对象（非严格模式）

---

### 例题 2：返回箭头函数后再调用

const name = "全局";  
  
const obj = {  
  name: "阳哥",  
  say: function () {  
    return () => {  
      console.log(this.name);  
    };  
  }  
};  
  
const f = obj.say();  
f();

#### 输出

阳哥

#### 原因

obj.say() -> say 的 this = obj  
箭头函数在 say 执行时被创建  
=> 捕获 say 当时的 this，也就是 obj  
后面 f() 执行时，不会重新绑定 this  
=> 输出 阳哥

---

### 例题 3：普通函数里返回普通函数，再返回箭头函数

const name = "全局";  
  
const obj = {  
  name: "阳哥",  
  say: function () {  
    return function () {  
      return () => {  
        console.log(this.name);  
      };  
    };  
  }  
};  
  
const f1 = obj.say();  
const f2 = f1();  
f2();

#### 输出

全局

#### 原因

obj.say() -> say 的 this = obj  
say 返回普通函数  
f1() -> 普通函数独立调用  
=> f1 这一层的 this = 全局对象（非严格模式）  
箭头函数在 f1 执行时创建  
=> 捕获 f1 当时的 this，也就是全局对象  
f2() 执行时继续用这个 this  
=> 输出 全局

---

### 例题 4：箭头函数配合 call

const name = "全局";  
  
const obj = {  
  name: "阳哥",  
  say: function () {  
    return (() => {  
      console.log(this.name);  
    }).call({ name: "小王" });  
  }  
};  
  
obj.say();

#### 输出

阳哥

#### 原因

obj.say() -> say 的 this = obj  
箭头函数在 say 内部创建  
=> 捕获 this = obj  
后面 .call({ name: "小王" }) 只能触发执行  
不能修改箭头函数的 this  
=> 输出 阳哥

---

## 4. 判断流程

第 1 步：先看当前执行的是普通函数还是箭头函数  
  
如果是普通函数：  
  看最终调用点  
  - obj.fn() ?  
  - fn() ?  
  - fn.call(x) ?  
  - fn.bind(x)() ?  
  
如果是箭头函数：  
  不看谁调用它  
  直接看它定义时外层 this 是谁

### 极简口诀

普通函数看调用  
箭头函数看出生

---

## 5. 一句话总结

普通函数的 `this` 是“调用时决定”的，箭头函数的 `this` 是“创建时锁定”的。

---

## 6. 小练习

### 练习 1

const name = "全局";  
  
const obj = {  
  name: "阳哥",  
  fn: function () {  
    console.log(this.name);  
  }  
};  
  
const a = obj.fn;  
a();

---

### 练习 2

const name = "全局";  
  
const obj = {  
  name: "阳哥",  
  fn: function () {  
    return () => {  
      console.log(this.name);  
    };  
  }  
};  
  
const a = obj.fn();  
a();

---

### 练习 3

const name = "全局";  
  
const obj = {  
  name: "阳哥",  
  fn: function () {  
    return function () {  
      console.log(this.name);  
    };  
  }  
};  
  
const a = obj.fn();  
a.call({ name: "小王" });

---

### 练习 4

const name = "全局";  
  
const obj = {  
  name: "阳哥",  
  fn: function () {  
    return () => {  
      console.log(this.name);  
    };  
  }  
};  
  
const a = obj.fn();  
a.call({ name: "小王" });