# formal parameters share values with `arguments` object

```JavaScript
var objInGlobal = {
  a: 'a',
  b: 'b'
};

(function (baseTypeVar, referTypeVar, notPassedArg) {
  baseTypeVar = 2;
  console.log(baseTypeVar === arguments[0]); // true
  
  console.log(referTypeVar === objInGlobal); // true
  referTypeVar = {
    c: 'c',
    d: 'd'
  };
  console.log(referTypeVar === arguments[1]); // true
  console.log(referTypeVar === objInGlobal); //false
  arguments[1] = {
    e: 'e',
    f: 'f'
  };
  console.log(referTypeVar === arguments[1]); // true
  
  console.log(2 in arguments); // false
  notPassedArg = 1;
  console.log(notPassedArg === arguments[2]); // false
  arguments[2] = 2;
  console.log(notPassedArg === arguments[2]); // false
})(1, objInGlobal);
```
**Vaules of indexes on `arguments` object are shared with values of formal parameters, which means they are always equal even when you override values of indexes on `arguments` or ones of formal parameters.**

**However, for those parameters that no value passed to, values of related indexes on `arguments` object are not shared, since these indexes are not defined on `arguments` when entering function**

From ECMA-262-3, but not JavaScript interpreter implementation.