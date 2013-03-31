# 跳出惯性思维，逆推法从 `map` 出发实现Python的 `range` 函数

Python的 `range` 函数，可以方便的生成包含递增数字的列表，比如

```Python
range(1, 5)
```
得到 `[1, 2, 3, 4]` ，这篇文章中我们用逆推法，一步步分析怎么用JavaScript实现类似 `range` 函数的思路。

首先看一下Underscore.js 1.4.4中的版本：

```JavaScript
// Generate an integer Array containing an arithmetic progression. A port of
// the native Python `range()` function. See
// [the Python documentation](http://docs.python.org/library/functions.html#range).
_.range = function(start, stop, step) {
  if (arguments.length <= 1) {
    stop = start || 0;
    start = 0;
  }
  step = arguments[2] || 1;

  var len = Math.max(Math.ceil((stop - start) / step), 0);
  var idx = 0;
  var range = new Array(len);

  while(idx < len) {
    range[idx++] = start;
    start += step;
  }

  return range;
};
```

这个版本的思路也很简单，就是给新数组一个一个的赋值。有没有更好的办法呢？

---

提高JavaScript在计算上的性能，有两种思路：

1. 使用数据结构与算法来优化性能
2. 使用源生方法，在语言层面来提高性能（JavaScript引擎比如V8由C++实现，源生方法已编译优化成机器语言或别的中间语言）

这里我们讨论第二种方法，在源生方法中寻找实现的方式。

---

`Array.prototype.map` 方法返回一个新数组，其元素由传给 `map` 的函数（以下简称函数 f ）的返回值组成。

换句话说，`map` 可以用指定的值产生一个新数组，而新数组的每个元素值的产生，可利用的信息包括原数组对应元素的值和 **索引**。

或许有很多人惯性思维更多关注的是值，其实 **拿索引做值** 在这个场景里大有用处：

**我们要产生一个数字递增的列表，而数组索引不就是一串递增的数字吗！只要在函数 f 中返回它的的第二个参数，也就是索引，最后 `map` 得到的就是一个元素值为递增数字的数组。**

比如：

```JavaScript
arrayWithLength5.map(function (val, i) {
  return i;
});
// [0, 1, 2, 3, 4]
```

---

**此时问题转化成怎么得到怎么得到一个指定长度的数组（只要有索引就行，不管元素值是什么）**，这一步看起来简单，但有一个陷阱。首先，我这有两种方法：

```JavaScript
(function (len) {
  // way 1:
  var arr1 = new Array(len);
  alert(1 in arr1); // false

  // way 2:
  var arr2 = [];
  arr2.length = len;
  alert(1 in arr2); // false
})(3);
```

其中[`new Array` 比修改 `length` 快](http://jsperf.com/create-sparse-array)。


但这样得到的都是sparse array（详见 *JavaScript: The Definitive Guide* 7.3 Sparse Arrays），没有索引属性（JavaScript中数组实际也是对象，索引其实是这个对象的属性），在这个例子中就是缺少了 `arr1['0'], arr1['1']` 这些属性，这和属性值为 `undefined` 不同（但spars array还是有 `length` 属性的）。

这里有两种方式可以将sparse array转换为dense array:

```JavaScript
(function (len) {
  // way 1:
  var arr2 = new Array(len + 1).join('a').split('');
  alert(arr2); // output: a, a, a

  // way 2:
  var arr1 = Array.apply(null, new Array(len));
  alert((0 in arr1) + ' ' + (1 in arr1) + ' ' + (2 in arr1)); // output: true true true. Awesome!
})(3);
```

其中[`Array.apply` 比先 `join` 后 `split` 快](http://jsperf.com/sparse-array-to-dense-array)。

---

P.S. 其实 `Array.apply` 还能把伪数组转换成数组：

```JavaScript
(function () {
  alert( Object.prototype.toString.call(arguments) ); // [object Arguments]
  alert( Object.prototype.toString.call( Array.apply(null, arguments) ) ); // [object Array]
})(1, 'foo', 'bar')
```

[对于把伪数组 `arguments` 转换为数组，`Array.apply` 是最快或比较快的一种方式](http://jsperf.com/converting-arguments-to-an-array/14)

---

到了这步， 已经可以基本实现支持 `range` 函数前两个参数的版本了：

```JavaScript
(function range (start, end) {
  return Array.apply(null, new Array( Math.max(end - start, 0) ))
              .map(function (val, i) { return i + start });
})(5, 15);
// [5, 6, 7, 8, 9, 10, 11, 12, 13, 14]
```

稍加修饰，也可像Underscore.js那样支持第三个参数：

```JavaScript
(function range (start, end, step) {
  return Array.apply(null, new Array( Math.max(Math.ceil( (end - start) / step ), 0) ))
              .map(function (val, i) { return i * step + start; });
})(5, 15, 2);
// [5, 7, 9, 11, 13]
```

加上参数缺省的支持：参数 `start` 缺省则为0, `step` 缺省则为1：

```JavaScript
function range (start, end, step) {
  if (arguments.length <= 1) {
    end = start || 0;
    start = 0;
  }
  step = step || 1;
  return Array.apply(null, new Array( Math.max(Math.ceil( (end - start) / step ), 0) ))
              .map(function (val, i) { return i * step + start; });
};
range(5, 15); // [5, 6, 7, 8, 9, 10, 11, 12, 13, 14]
range(5); // [0, 1, 2, 3, 4]
```

---

比较这最后的版本和Underscore.js 1.4.4中的版本：

我们的版本的核心计算部分主要是用源生方法实现，相比于Underscore.js 1.4.4中用一层循环一个一个赋值的方式，我们的版本看起来挺酷的，而且源生的方法肯定比较快。

再次跳出惯性思维！测试之后发现，[用最简单的while循环居然压倒性的胜过用源生方法的版本](http://jsperf.com/simulate-range-of-python/2)！Where amazing happen!

仔细分析一下，我觉得最有可能的原因就是：传给 `map` 方法的闭包（这里说闭包，是想强调作用域链，[A closure is a combination of a code block and data of a context in which this code block is created](http://dmitrysoshnikov.com/ecmascript/chapter-6-closures/#closure)）的执行代码中，有两个标识符 `step` 和 `start` 需要沿着作用域链向上查找到 `range` 的上下文的 [`activation object`](http://dmitrysoshnikov.com/ecmascript/chapter-2-variable-object/#variable-object-in-function-context) 当中，Zakas的 *High Performance JavaScript* Chapter 2 有解释过查找作用域链是比较耗性能的，这个例子算是体现了他的破坏力的吧，不但让源生方法的优势当然无存，还被压倒性的胜过。怪不得Underscore.js里创建了很多局部变量以尽量避免查找作用域链：

```JavaScript
// Save bytes in the minified (but not gzipped) version:
var ArrayProto = Array.prototype, ObjProto = Object.prototype, FuncProto = Function.prototype;

// Create quick reference variables for speed access to core prototypes.
var push             = ArrayProto.push,
    slice            = ArrayProto.slice,
    concat           = ArrayProto.concat,
    toString         = ObjProto.toString,
    hasOwnProperty   = ObjProto.hasOwnProperty;
```

如果你对查找作用域链的性能损耗还抱有疑虑，[请看这个比较测试](http://jsperf.com/original-foreach-with-scope-chain-searching-vs-while-lo)

---

P.S. 

做了这几个测试，看来IE9的硬件加速对计算能力的提升还是挺明显的，一在IE9下跑起来，我机器的风扇就呼呼响（本人的机器，内存总共3.42GB可用，CPU 2.67GHz）。

而Chrome25的V8估计是对JavaScript程序员常用的操作做过特别的优化，比如将 `arguments` 转换为数组。
