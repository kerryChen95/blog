# Distinguish empty element and `undefined` valued element in JavaScript array

## Way 1: `in` operator, and `for...in` loop
Due to indexes are not even defined for empty element, `in` operator return `false`.

```JavaScript
var arr = new Array(5);
console.log(1 in arr); // false

arr[1] = undefined;
console.log(1 in arr); // true

for (var i in arr) {
  console.log(i);
}
// 1
```
Tested in Chrome 25.

## Way 2: `forEach`, `filter`, `every`, `some` etc. iterator method.
These methods do not invoke the function for indexes at which no element actually exists.

```JavaScript
var arr = new Array(5);
arr.forEach(function () { console.log('Never log this') });
arr[1] = undefined;
arr.forEach(function () { console.log('Only log 1') });
```