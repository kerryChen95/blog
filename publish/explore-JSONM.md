# 探索JSONM的原理

## JSONM简介

JSONM是玉伯在 [《聊聊 JSONP 的 P》](https://github.com/lifesinger/lifesinger.github.com/issues/118) 一文中介绍的一种方案，它可以一定程度上解决JSONP回调函数命名冲突的问题。

对于SeaJS，这个方案的使用方式简单来说就是，像通常加载一个匿名模块那样写，但是指定的URI是JSONP要请求的URI，而回调函数内执行的逻辑是JSONP回调函数内执行的逻辑。

比如SeaJS中：

```JavaScript
seajs.use("http://cross.origin.com/somedata?type=jsonp",
  function dataHandler(data) { // function name will be used in description below
    console.log(data.hello); // => world
  });
```

向一个跨域的支持JSONP的URI发送请求，返回如下：

```JavaScript
define({"hello": "world"});
```

JSONM在各个请求中都使用同一个JSONP回调函数名，从而在一定程度上解决命名冲突的问题，这也是它最神奇的地方。

具体用法请看玉伯的那篇文章。如果没看过，建议先看，再看本文。

## JSONM原理探索

在看玉伯的那篇文章的时候，我就一直好奇：既然各个请求都使用同一个回调函数名，那么是如何把某一次调用跟特定的请求关联在一起的？也就是传给 `define` 的JSON数据是如何传给 `dataHeadler` 回调函数的？

因为不知道别的XMD loader是否实现JSONM，本文中只拿[SeaJS](https://github.com/seajs/seajs)举例，通过分析[SeaJS2.0.0的源码](https://github.com/seajs/seajs/blob/2.0.0/dist/sea-debug.js)探索JSONM的原理。

首先来一窥SeaJS中用作JSONM回调函数的 `define` 函数内部：

```JavaScript
function define(id, deps, factory) {
  // define(factory)
  if (arguments.length === 1) {
    factory = id
    id = undefined
  }

  // Parse dependencies according to the module factory code
  if (!isArray(deps) && isFunction(factory)) {
    deps = parseDependencies(factory.toString())
  }

  var data = { id: id, uri: resolve(id), deps: deps, factory: factory }

  // Try to derive uri in IE6-9 for anonymous modules
  if (!data.uri && doc.attachEvent) {
    var script = getCurrentScript()

    if (script) {
      data.uri = script.src
    }
    else {
      log("Failed to derive: " + factory)

      // NOTE: If the id-deriving methods above is failed, then falls back
      // to use onload event to get the uri
    }
  }

  // Emit `define` event, used in plugin-nocache, seajs node version etc
  emit("define", data)

  data.uri ? save(data.uri, data) :
      // Save information for "saving" work in the script onload event
      anonymousModuleData = data
}
```

显然我们应该关注的是 `arguments.length` 为1的情况，此时传给 `define` 函数的JSON数据的引用保存在 `factory` 形参中。然后运行到 `getCurrentScript()` ，进入这个函数体中。

```JavaScript
function getCurrentScript() {
  if (currentlyAddingScript) {
    return currentlyAddingScript
  }

  // For IE6-9 browsers, the script onload event may not fire right
  // after the the script is evaluated. Kris Zyp found that it
  // could query the script nodes and the one that is in "interactive"
  // mode indicates the current script
  // ref: http://goo.gl/JHfFW
  if (interactiveScript && interactiveScript.readyState === "interactive") {
    return interactiveScript
  }

  var scripts = head.getElementsByTagName("script")

  for (var i = scripts.length - 1; i >= 0; i--) {
    var script = scripts[i]
    if (script.readyState === "interactive") {
      interactiveScript = script
      return interactiveScript
    }
  }
}
```

比想象中要快，到这一步便真像大白了： **原来是通过判断 `script` 标签的 `readyState` 属性是否为 `interactive` 来得知是哪个 `script` 标签中的脚本正在执行，进而得知 `define` 函数是在哪个 `script` 标签中被调用的，然后得知 `script` 标签的URI，也就得知JSONP请求的URI了** 。然后就可以通过这个URI，找到要使用JSON数据的回调函数 `dataHandler`（因为URI和 `dataHandler` 被一起传给了 `seajs.use` 函数，它们的关联可以建立起来）。


## 路边的风景：所谓的“同步”加载

SeaJS率先实现在模块的业务逻辑代码内（也就是作为第三个参数传给 `define` 的那个回调函数内。说其是模块的业务逻辑代码，是相对于SeaJS内部的代码）通过 `require` 函数进行“同步”加载依赖。

我们知道XHR对象是可以同步的进行网络I/O的：`oepn` 方法的第三个参数为 `false`，然后在调用 `send` 方法的时候，会阻塞JavaScript的执行和浏览器的UI渲染（两者都在同一个线程），直到响应完全接收。

第一次看到SeaJS的“同步”加载的时候，惊呼awesome！以为就像XHR对象进行同步的网络I/O那样：这些依赖是在执行到 `require` 函数的调用才同步加载的，也就是说 `require` 是阻塞式的，它直到加载完依赖并解析完，得到了依赖的 `exports` 才返回。

但是，在探索JSONM原理的路上，我们可以在路边发现一些有意思的东西。

`define` 函数内通过调用 `parseDependencies` 函数来获取依赖，给 `parseDependencies` 传入的是 `factory.toString()` 。

`Function.prototype.toString` 方法返回函数的字符串表示，包括 `function` 关键字、参数列表、完整的函数体等，这一字符串是有效的JavaScript代码，可以被 `eval` 函数执行。

`factory` 在这里是定义一个模块时，模块依赖加载完毕所执行的回调函数。那么，这里解析的依赖应该就是在回调函数内部“同步”加载的依赖。那么它是如何获取依赖的呢？

在 `parseDependencies` 函数内部：

```JavaScript
var REQUIRE_RE = /"(?:\\"|[^"])*"|'(?:\\'|[^'])*'|\/\*[\S\s]*?\*\/|\/(?:\\\/|[^/\r\n])+\/(?=[^\/])|\/\/.*|\.\s*require|(?:^|[^$])\brequire\s*\(\s*(["'])(.+?)\1\s*\)/g
var SLASH_RE = /\\\\/g

function parseDependencies(code) {
  var ret = []

  code.replace(SLASH_RE, "")
      .replace(REQUIRE_RE, function(m, m1, m2) {
        if (m2) {
          ret.push(m2)
        }
      })

  return ret
}
```

原来如此，通过正则表达式获取传给 `require` 函数的参数，进而得知依赖的是哪些模块。

在 `define` 函数内， `parseDependencies` 函数的返回值直接赋给了 `deps` ，在对 `deps` 的使用上，跟传给 `define` 函数的第二个参数（也就是预先声明的依赖）的使用没有区别。

这说明，一个模块的依赖，无论是通过 `define` 函数的第二个参数预先声明，还是在模块的业务逻辑代码内通过 `require` 函数进行加载，都是在执行模块的业务逻辑回调函数之前，就加载依赖并获取依赖的 `exports`。而在调用 `require` 函数的时候，不阻塞，不加载，直接获取依赖的 `exports` 。

这样看来，所谓的“同步”加载并不是真正的同步加载，只是语法上像是同步的而已。
