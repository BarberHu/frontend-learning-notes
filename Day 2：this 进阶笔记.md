
## 今天的目标  
搞清楚下面这几件事：  
  
1. 普通函数的 `this` 到底看什么  
2. 箭头函数的 `this` 和普通函数有什么本质区别  
3. `call` 能做什么，不能做什么  
4. 为什么“函数写在对象里”不等于 “this 就是这个对象”  
  
---  
  
## 一、先记最核心的两句话  
  
### 1. 普通函数  
普通函数的 `this` **看调用方式**。  
  
### 2. 箭头函数  
箭头函数的 `this` **看定义位置**，会继承外层作用域的 `this`。  
  
---  
  
## 二、普通函数的 this  
  
### 1）对象调用：`obj.fn()`  
  
Const obj = {  
  Name: '阳哥',  
  Say: function () {  
    Console.Log (this. Name);  
  }  
};  
  
Obj.Say ();

输出：

阳哥

原因：

- 调用方式是 `obj.say()`
- 所以 `say` 里的 `this === obj`

---

### 2）直接调用：`fn()`

Const obj = {  
  Name: '阳哥',  
  Say: function () {  
    Function inner () {  
      Console.Log (this. Name);  
    }  
  
    inner();  
  }  
};  
  
Obj.Say ();

输出：

Undefined

原因：

- `inner` 是普通函数
- 调用方式是 `inner()`，属于直接调用
- 不是 `某个对象.inner()`
- 所以 `this` 不会指向 `obj`

---

### 3）函数嵌套不会自动继承 this

很多初学者会误以为：

> `inner` 写在 `say` 里面，所以 `inner` 的 `this` 也应该是 `obj`

这是错的。

`this` 不是看函数写在哪，  
而是看函数**这次怎么调用**。

---

## 三、把普通函数挂到别的对象上再调用

Const obj = {  
  Name: '阳哥',  
  Say: function () {  
    Function inner () {  
      Console.Log (this. Name);  
    }  
  
    const another = {  
      name: '小王',  
      inner: inner  
    };  
  
    another.inner();  
  }  
};  
  
Obj.Say ();

输出：

小王

原因：

- `inner` 虽然最初是普通函数
- 但这次调用方式是 `another.inner()`
- 所以执行时 `this === another`
- 因此输出 `another.name`，也就是 `小王`

---

## 四、普通函数的 `call`

### 1）`call` 是什么

`call` 是函数对象自带的方法，用来：

> **立刻调用函数，并手动指定这次执行时的 `this`**

基本写法：

函数.Call (指定的 this, 参数 1, 参数 2, ...)

---

### 2）示例

Function say () {  
  Console.Log (this. Name);  
}  
  
Const obj = { name: '阳哥' };  
  
Say.Call (obj);

输出：

阳哥

原因：

- `call(obj)` 强制把这次执行时的 `this` 指定为 `obj`

---

## 五、箭头函数的 this

### 1）箭头函数没有自己的 this

它不会像普通函数那样在调用时重新决定 `this`，  
而是会直接继承定义时外层作用域的 `this`。

---

### 2）示例

Const obj = {  
  Name: '阳哥',  
  Say: function () {  
    Const inner = () => {  
      Console.Log (this. Name);  
    };  
  
    inner();  
  }  
};  
  
Obj.Say ();

输出：

阳哥

原因：

- `obj.say()` 执行时，`say` 里的 `this === obj`
- `inner` 是箭头函数
- 箭头函数会继承定义时外层的 `this`
- 所以 `inner` 里的 `this` 也是 `obj`

---

## 六、箭头函数和 `call`

### 结论

`call` 可以改普通函数的 `this`，  
**但改不了箭头函数的 `this`**。

---

### 示例

Const obj = {  
  Name: '阳哥',  
  Say: function () {  
    Const inner = () => {  
      Console.Log (this. Name);  
    };  
  
    const another = {  
      name: '小王'  
    };  
  
    inner.call(another);  
  }  
};  
  
Obj.Say ();

输出：

阳哥

原因：

- `inner` 是箭头函数
- 它的 `this` 在定义时就已经继承了 `say` 里的 `this`
- 而 `say` 里的 `this === obj`
- 所以 `inner` 里的 `this` 已经固定为 `obj`
- `call(another)` 对箭头函数无效
- 最终还是输出 `阳哥`

---

## 七、今天最关键的对比表

|情况|this 指向谁|
|---|---|
| `obj.fn()` | `obj` |
| `fn()` |不是 `obj`，常见表现为 `undefined` |
| `fn.call(x)` | `x` |
|箭头函数 `() => {}` |继承定义时外层的 `this` |
| `箭头函数.call(x)` |依然不变，还是定义时那个 `this` |

---

## 八、经典练习题总结

Const obj = {  
  Name: '阳哥',  
  Say: function () {  
    Console.Log ('A', this. Name);  
  
    function inner1() {  
      console.log('B', this.name);  
    }  
  
    const inner2 = () => {  
      console.log('C', this.name);  
    };  
  
    const another = {  
      name: '小王'  
    };  
  
    inner1();  
    inner1.call(another);  
    inner2();  
    inner2.call(another);  
  }  
};  
  
Obj.Say ();

输出：

A 阳哥  
B undefined  
C 小王  
D 阳哥  
E 阳哥

更准确地说，对应关系是：

- `A` -> `A 阳哥`
- `inner1()` -> `B undefined`
- `inner1.call(another)` -> `C 小王`
- `inner2()` -> `D 阳哥`
- `inner2.call(another)` -> `E 阳哥`

---

## 九、每一项为什么

### A：`console.log('A', this.name)`

输出：

A 阳哥

原因：

- 调用方式是 `obj.say()`
- 所以 `say` 里的 `this === obj`

---

### B：`inner1()`

输出：

B undefined

原因：

- `inner1` 是普通函数
- 调用方式是直接调用
- 所以 `this` 不会指向 `obj`

---

### C：`inner1.call(another)`

输出：

C 小王

原因：

- `inner1` 是普通函数
- `call(another)` 强制把 `this` 绑定到 `another`

---

### D：`inner2()`

输出：

D 阳哥

原因：

- `inner2` 是箭头函数
- 它继承定义时外层 `say` 的 `this`
- `say` 的 `this === obj`

---

### E：`inner2.call(another)`

输出：

E 阳哥

原因：

- `inner2` 是箭头函数
- 箭头函数的 `this` 不能被 `call` 修改
- 所以仍然是定义时继承到的 `obj`

---

## 十、今天最容易错的点

### 错误 1：函数写在对象里，就默认 this 是对象

错。

应该是：

> 只有当函数通过 `对象.方法()` 调用时，普通函数的 `this` 才会指向这个对象。

---

### 错误 2：嵌套函数会自动继承外层 this

错。

应该是：

> 普通函数不会因为写在另一个函数里面，就自动继承外层的 `this`。

---

### 错误 3：`call` 什么函数都能改

错。

应该是：

> `call` 能改普通函数的 `this`，改不了箭头函数的 `this`。

---

## 十一、今天的最终口诀

普通函数：看调用  
箭头函数：看定义  
Call 能改普通函数  
Call 改不了箭头函数

---

## 十二、今天的学习成果

今天我已经能判断：

1. `obj.say()` 中普通函数的 `this`
2. `inner()` 这种直接调用时的 `this`
3. `another.inner()` 这种重新挂载后的 `this`
4. 箭头函数会继承外层 `this`
5. `call` 可以改普通函数，但改不了箭头函数

---

## 十三、明天的衔接点

下一步会继续看：

Const fn = obj. Say;  
Fn ();

为什么把函数“拿出来”之后，`this` 就丢了。

这个会把 `this`、函数引用、函数传递这几件事真正串起来。
