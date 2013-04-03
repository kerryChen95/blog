# 你也许不知道的函数声明与函数表达式

先给出两者的一些特性的诠释，后面再详细分析。

## 函数声明(Function Declaration)

> 根据ECMAScript规范，函数声明是指这样的函数：
> 1. 必须有一个函数名；
> 2. 会影响所处上下文的变量对象。也就是会在该变量对象上添加一个同名属性，属性值为所创建的函数对象的引用。
> 3. 在进入上下文阶段创建函数对象。也就是在代码执行阶段之前；
> 4. 要么直接写在程序的全局上下文中，要么直接写在另一个函数的函数体中。也就是不能嵌套在代码块中；
> 5. 以如下syntax表示：

```JavaScript
function name([param[, param[, ... param]]]) {
  statements
}
```

## 函数表达式(Function Expression)

> 根据ECMAScript规范，函数表达式是指这样的函数：
> 1. 函数名可以省略；
> 2. 不会影响所处上下文的变量对象。也就是不会在该变量对象上添加属性。
> 3. 在代码执行阶段创建函数对象；
> 4. 只能写在表达式的位置；
> 5. 以如下syntax表示：

```JavaScript
function [name]([param] [, param] [..., param]) {
   statements
}
```

## 从解释器的角度看

以上是两者的一些特征的描述，从解释器的角度来看：

函数声明是一个 [`function statement`](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Statements/function)，是一个语句。

而函数表达式则是 [an expression in which the `function` operator defines a function](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Operators/function)，是由 `function` 操作符及其操作数构成的表达式。

## 在实际中要注意的问题和区别

让我们一条条阐述两者的区别在实际中的体现，以及各自的特征导致的在实际中需要注意的问题。

### 1. 函数名

函数声明中的函数名没太多好说的。

---

至于函数表达式中的函数名，最大的用处是可以在自身的函数体内通过这个函数名访问到自身的函数对象，常用来递归调用，避免 `arguments.callee` 这种方式（ECMAScript5的 `strict mode` 下不支持 `arguments.callee`）。

```JavaScript
var factorial = function _f (num) {
  if (num === 1) return num;
  return num * _f(num - 1);
}
factorial(3); // return: 6
```

但它不可以在自身的函数体外被访问到。为什么在外面访问不到，在里面又能访问到？原因详见下一小节“对变量对象的影响”。

### 2. 对变量对象的影响

函数声明会给变量对象添加一个同名属性，属性值为所创建的函数对象的引用。

所以可以在函数声明语句所在的作用域内访问到该函数对象。

---

函数表达式不会给所处上下文的变量对象添加属性。所以在它自身的函数体外，不能通过函数表达式中的函数名访问到所创建的函数对象，我们必须把改函数对象的引用赋值给一个变量，或一个对象的属性，才能访问到它。

```JavaScript
var factorial = function _f (num) {
  if (num === 1) return 1;
  return num * _f(num - 1);
}
alert(typeof _f); // output: undefined
```

那么为什么能在内部访问到函数表达式的所创建的函数对象的函数名呢:

在代码执行阶段，当解释器遇到一个具名的函数，它的算法描述如下：

1. 在创建函数对象之前，先给作用域链的前端添加一个特殊的变量对象；
2. 创建函数对象，这时函数对象上的内部属性 `[[Scope]]` 会拷贝当前上下文的作用域链（作用域链的数据结构可以看做元素为对象的数组，而执行的是对数组元素的浅拷贝），而该作用域链的前端是刚才添加的特殊的变量对象；
3. 像函数声明那样，给当前作用域链中前端的变量对象添加唯一的一个属性，属性名为该函数对象的函数名，值为该函数对象的引用。因为是对数组元素的浅拷贝，所以添加的属性也反映在 `[[Scope]]` 中；
4. 然后从上下文中的作用域链的前端移除那个特殊的变量对象。因为并不是对数组的浅拷贝，所以不会反应在 `[[Scope]]` 中。

这样，具名函数表达式所创建的函数对象的内部属性 `[[Scope]]` 所拥有的作用域链的前端就多了一个特殊的变量对象。所以在被调用的时候，就能通过这个变量对象中保存的引用访问到自身了。

另外顺带一提，在被调用的时候，会把 `[[Scope]]` 上的作用域链又以同样的方式浅拷贝的新创建的上下文中，并添加新上下文的变量对象到新上下文的作用域链的前端。所以不是搜索 `[[Scope]]` 上的作用域链哦，而是新上下文的作用域链。详细说明，请等待我将要发布的系列文章。

### 3. 创建函数对象的时期

函数声明在进入上下文阶段创建函数对象，也就是在代码执行阶段之前。

为了避免误解，这里所说的上下文，是指函数声明语句所在的函数体代码块或全局上下文，而不是所声明的函数被调用时进入的上下文。

这一特性导致了经典的提升(hoisting)现象：在代码执行阶段，执行到声明语句之前，就可以访问并使用其函数对象（调用，修改该函数对象的属性，等等）。因为在执行阶段之前，解释器就已经使用了函数声明语句提供的信息（函数名，形参列表，函数体中的语句）创建了函数对象，并把引用保存在变量对象的一个同名属性上。我想这也解释了为什么专门用 `function` 关键字开头表示函数声明语句，因为这语句比较特殊，它不在执行阶段得到解释，而是在进入上下文阶段。

例子如下：

```JavaScript
(function () {
  alert(typeof foo); // output: function
  function foo () {
    alert(arguments.length);
  }
})();
```

---

P.S.

类似的， `var` 关键字开头，表示声明变量，也在进入上下文阶段得到解释器解释，即给变量对象添加同名属性，但给变量的赋值是在执行阶段解释的。这就解释了为什么在 `var` 之前可以访问所声明的变量，但访问返回的值是 `undefined` 。

另外，一个函数的形参也是在进入这个函数的上下文阶段得到解释器解释，即给变量对象添加同名属性，但与变量声明不同，这时候就给形参赋值了。

根据ECMAScript3，形参、函数声明、变量声明在进入上下文阶段的解释顺序和命名冲突的处理如下：

1. 先为每一个形参（如果有的话）在变量对象上添加一个同名属性，值为实参或 `undefined` 
2. 再为每一个函数声明在变量对象上添加一个同名属性，值为所创建的函数对象的引用，如果已有同名属性存在，可以覆盖该属性的值和attribute
3. 然后为每一个变量声明在变量对象上添加一个同名属性，值为 `undefined` ，如果已有同名属性存在，不影响原有属性。

```JavaScript
(function (foo, bar) {
  function foo () {}
  var foo;
  var bar;
  alert('foo\'s type: ' + typeof foo); // output: function
  alert('bar\'s type: ' + typeof bar); // output: number
  // test in Chrome26, Firefox19, IE9
})(1, 2);
```

---

函数表达式在代码执行阶段创建函数对象，所以函数表达式不会出现hoisting现象。这是实践中使用两者时需要注意的区别。

```JavaScript
(function () {
  alert(typeof foo); // output: undefined
  var foo = function _foo () {};
  alert(typeof foo); // output: function
})();
```

以上这个例子能证明“函数表达式所创建的函数对象不会出现hoisting现象”吗？

感兴趣可以思考一下。

因为变量 `foo` 用 `var` 关键字声明，它会hoisting，而赋值是在执行阶段进行的，所以，无论这个函数表达式创建函数对象是在进入上下文阶段还是代码执行阶段，第一个 `alert` 都会输出 `undefined` ，第二个 `alert` 也都会输出 `function` 。所以，其实不能证明。

要想证明，本人还没想到很好的方法，有哪位大牛想到，欢迎指教。

### 4. 在程序源码中的位置

函数声明，要么直接在程序的全局上下文中，要么直接在另一个函数的函数体中。也就是不能嵌套在代码块中；

```JavaScript
if (true) {
  // 没有直接在全局上下文中，
  // 在一个会执行的代码块中。
  function func1 () {
    alert(arguments);
  }
}
if (false) {
  // 没有直接在全局上下文中，
  // 在一个不会执行的代码块中。
  function func2 () {
    alert(arguments);
  }
}
alert('func1 create property on variable object: ' + (typeof func1 === 'function')); // `true` in Chrome 26, IE 9, Firefox 19
alert('func2: ' + (typeof func2 === 'function')); // `false` in Firefox 19; true in Chrome 26, IE 9

(function () {
  if (true) {
    // 没有直接在函数体中，
    // 在一个会执行的代码块中
    function func3 () {
      alert(arguments);
    }
  }
  if (false) {
    // 没有直接在函数体中，
    // 在一个不会执行的代码块中
    function func4 () {
      alert(arguments);
    }
  }
  alert('func3: ' + (typeof func3 === 'function')); // `true` in Chrome 26, IE 9, Firefox 19
  alert('func4: ' + (typeof func4 === 'function')); // `false` in Firefox 19; true in Chrome 26, IE 9
})();
```

如上，以非标准的方式声明函数（也就是嵌套在代码块中），在不同的实现中有不同的表现：在Chrome26的V8和IE9的Chackra中，会影响变量对象；在Firefox19的SpiderMonkey中，不会影响变量对象。所以为了保证代码的一致性，请直接在全局上下文，或直接在函数体中声明函数。

---

函数表达式只能写在表达式的位置；

```JavaScript
// 在分组操作符(grouping operator)中只能是表达式
(function foo() {});
 
// 逗号操作符的操作数只能是表达式
1, function baz() {};
```

这里要注意的是，表达式不能以 `{` 开头，因为这样会无法跟代码块区分开；表达式也不能以 `function` 关键字开头，因为这样会无法跟函数声明区分开；

或许你因为习惯，觉得表达式不以 `{` 开头是理所当然的。但或许你没有想到过，这跟函数表达式用圆括号包起来是相同的原因：

学习过编译原理的同学，就知道词法分析器对字符流进行词法分析得到符号流，然后语法分析器对符号流进行语法分析得到语法树，这时语法树的特定结构具有特定的语义。

扯远了，继续讲函数表达式。看例子：

```JavaScript
{function func () {}}, 2
```

报错："SyntaxError: Unexpected token ,"。因为逗号操作符前应该是表达式，而不是代码块。改成如下：

```JavaScript
(function func () {}), 2
alert(typeof func);
```

输出： `undefined` 。未影响变量对象，说明这是一个函数表达式，所以不能以 `{` 开头。

再看一个经典例子：

```JavaScript
function func2 (a) { return a }()
```

报错："SyntaxError: Unexpected token )"。这里报错跟 `function func2 (a) { return a }` 无关。分析如下：

 `function` 关键字开头表示函数声明，到 `}` 处函数声明语句结束（JavaScript可以无分号结束语句）。
所以 `function func2 (a) { return a }` 不被解释为函数表达式，
进而 **第二个 `()` 不被解释为函数调用，而是分组操作符(grouping operator)。
而报错则是因为分组操作符在语法上要求有操作数在里面** （操作符总是需要有操作数的）。修改如下：

```JavaScript
function func2 (a) { return a }(1)
alert(typeof func2)
```

给分组操作符加上操作数就不会报错了。输出：`function` 。影响了变量对象，说明这是一个函数声明语句。

但是改成这样不能达到自调用的目的。要想自调用，就需要把 `function func2 (a) { return a }` 解释为函数表达式，然后用 `()` 调用就行了。这其中要注意的地方有两点：

1. 不能以 `function` 关键字开头，否则会解释为函数声明
2. 操作符的操作数总是表达式

所以，只要把 `function func2 (a) { return a }` 放在操作符的操作数的位置，并且不以 `function` 关键字开头，就会被解释为表达式了。于是就出现了经典的如下写法：

```JavaScript
(function func2 (a) { return a })(1);
alert(typeof func2);
```

输出： `undefined` 。与之前同理，说明这是一个函数表达式。

有趣的是，如下写法也会解释为函数表达式：

```JavaScript
void function (a) { return a } (2)
// 返回 `undefined`

!function (a) { return a } (2)
// false

1 + function func () {}
// 返回 `"1function func() {}"`

1 - function func () {}
// 返回 `NaN`
1 - function func () { return 1 }()
// 返回 `0`
```
