# Difference of array in JavaScript and C

Array in JavaScript is actually object:

- automatically maintains indexed values and `length` property.
- its constructor is `Array` which provides some methods to be inherited, e.g. `forEach`.
- may use non-sequence memory since it's an object.

Array in C is different, it always uses sequence memory, at this point, it is real array.

So what it means?

Indexes of an array returned by `new Array(3)` are even not defined, which just means this object (array is object) has no properties '0', '1', '2'. So check like `in` operation returns `false` and `forEach` interator skips these properties.

Diffrent in C, indexes of an array means offset from the address of first element, so you can always access via index, take no account of what's in the memory unit.

P.S.

As for pseudo array in JavaScript, it is also an object:
- but not inherit from `Array.prototype`, which means it has no methods e.g. `forEach`.
- it may has `length` property, but do not automatically maintains it, e.g. `arguments`.
