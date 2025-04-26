---
title: Javascript
---

### for...in

以任意序迭代一个对象的可枚举属性。

    for (variable in object) {...}

每次迭代，一个不同的属性名将会赋予 `variable`。

### Array 迭代和 for...in

数组索引仅是可枚举的属性名，其他方面和别的普通对象属性没有什么区别。for...in 并不能保证返回的是按一定顺序的索引，但是它会返回所有的可枚举属性，包括非整数名称的和继承的。

因为迭代的顺序是依赖于执行环境的，所以数组遍历不一定按次序访问元素。 因此当迭代那些访问次序重要的 arrays 时用整数索引去进行 for 循环 (或者使用 `Array.prototype.forEach()` 或 for...of 循环) 。

如果只要考虑对象本身的属性，而不是它的原型，那么使用 `getOwnPropertyNames()` 或执行  `hasOwnProperty()` 来确定某属性是否是对象本身的属性。

### Object.keys()

Object.keys() 方法会返回一个由给定对象的自身可枚举属性(字符串)组成的数组，数组中属性名的排列顺序和使用 for...in 循环遍历该对象时返回的顺序一致 （两者的主要区别是 一个 for...in 循环还会枚举其原型链上的属性）。

### Array.prototype.forEach()

forEach() 方法对数组的每个元素执行一次提供的函数。

forEach 方法按升序为数组中含有效值的每一项执行一次 callback 函数，那些已删除（使用 delete 方法等情况）或者未初始化的项将被跳过（但不包括那些值为 undefined 的项）（例如在稀疏数组上）。

### Object.prototype.hasOwnProperty()

所有继承了 Object 的对象都会继承到 hasOwnProperty 方法。这个方法可以用来检测一个对象是否含有特定的自身属性；和 in 运算符不同，该方法会忽略掉那些从原型链上继承到的属性。

### Object.getOwnPropertyNames()

Object.getOwnPropertyNames 返回一个数组，该数组对元素是 obj 自身拥有的枚举或不可枚举属性名称字符串。 数组中枚举属性的顺序与通过 for...in 循环（或 Object.keys）迭代该对象属性时一致。 数组中不可枚举属性的顺序未定义。


### this

#### 函数作为函数的参数

只要一个函数作为另一个函数的参数传递进去，那么里面的函数执行的时候就会丢失它的 context，变成一个匿名函数。

    let image = {
      name: 'Jian Li',
      transform () {
        console.log(this.name)
      }
    }

    function test (func) {
      func()
    }

    image.transform() // print 'Jian Li'

    test(image.transform) // print undefined

Express 里的中间件函数也是如此[stackoverflow]。

[stackoverflow]: https://stackoverflow.com/questions/18505383/how-to-use-this-context-in-middleware
