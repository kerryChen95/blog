突然想到的一种模式，在JS中实现private成员和protect成员。

跟天生有语言层面支持的Java相比，有一点还算可以接受的瑕疵：

* 需要在实例上有一个ID，即示例代码中的`this._privateId`;
* 在类的外部可以访问到protect成员（可以通过添加特殊的前缀，在代码规范层面约定不要在类的外部访问protect成员），当然你有可能也不需要protect成员，只要private和public成员就够了。
* 销毁实例的时候，需要手动调用`destructor`方法来销毁private属性，以免内存泄漏；

```javascript
var _ = require('underscore')

var Klass = function () {
  // @member {string} _privateId
  // @protect
  // can accessed or overrided by sub-class, use prefix `_` to notify that
  // should not be accessed outside of class `Klass`
  this._privateId = _.uniqueId('privateId')
  var privateAttr = privateAttrMap[this._privateId] = {
    // @member {string} privateAttrA
    // @private
    // can only accessed inside of class `Klass`
    privateAttrA: 'aaa',
    // @member {string} privateAttrB
    // @private
    privateAttrB: 'bbb',
  }
}

var privateAttrMap = {};

Klass.prototype = {
  constructor: Klass,

  destructor: function () {
    privateAttrMap[this._privateId] = null
  },

  publicMethod: function () {
    this._protectMethod()
    privateMethodA.call(this)
  },

  // can accessed or overrided by sub-class
  _protectMethod: function () {},
}

// @content {Klass} this
// can only accessed inside of class `Klass`
var privateMethodA = function () {
  this._protectMethod()
}

module.exports = Klass
```
