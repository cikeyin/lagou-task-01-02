# lagou-task-01-02

函数式编程与 JavaScript 性能优化

##### 一、简答题

###### 1. 描述引用计数的工作原理和优缺点

1.工作原理
引用计数是跟踪每个引用类型值被引用的次数。当一个引用类型值被赋给一个变量时，该值的引用次数加 1；当包含对该值引用的变量取得其他值时，该值的引用次数减 1。当该值的引用次数变为 0 时，可以回收其所占用的内存空间。当垃圾回收期下一次运行时，会对其进行回收。

2.优缺点
(1)优点：内存释放及时，当一个对象引用次数为 0 时其占用的空间马上被释放
(2)缺点：每个对象都需要计数器记录引用次数，耗费空间；无法解决循环引用的问题

###### 2. 描述标记整理算法的工作流程

标记整理算法是对标记清除算法的改进优化。分为两个阶段先进行标记，再进行整理清除。
在标记阶段，从 root 对象开始，遍历所有对象，所有从 root 对象可达的对象被标记为活动对象，不可达对象不进行标记
在清理阶段，先将活动对象整理到内存的一端，解决标记清理碎片化的问题。

###### 3. 描述 V8 中新生代存储区垃圾回收的流程

v8 采用的是分代式垃圾回收机制，将内存分为新生代和老生代两部分内存空间。新生代中内存空间中存放存活时间短的对象。
新生代内存空间采用复制算法。内存空间平均分成 From(使用空间)和 To(闲置空间)两个空间。新对象被分配到 From 空间中，当 From 空间快要被占满时将活动对象复制到 To 空间，然后清空 From 内存空间。此时，调换 From 空间和 To 空间，继续进行内存分配。
当满足两个条件时新生代中的对象晋升到老生代：一个对象第二次经历从 From 空间复制到 To 空间；当一个对象要从 From 空间复制到 To 空间时，To 空间已经使用了超过 25%

###### 4. 描述增量标记算法在何时使用及工作原理

垃圾回收阶段会暂停执行程序，如果垃圾回收耗费时间长，就会造成卡顿，影响用户体验，增量标记就是为了解决这个问题。
增加标记算法不会等 GC 执行完才将控制权交回程序，而是逐步完成垃圾回收，在程序运行中穿插进行，降低 GC 导致的暂停时间。

##### 二、代码题 1

基于一下代码完成下面的四个练习

```javascript
const fp = require("lodash/fp");

// 数据  power:马力；price：价格；stock：库存
const cars = [
    { name: "haval", power: 100, price: 100000, stock: true },
    { name: "honda", power: 200, price: 200000, stock: true },
    { name: "audi", power: 300, price: 300000, stock: false },
    { name: "benz", power: 400, price: 400000, stock: true },
    { name: "ferrari", power: 500, price: 500000, stock: false },
];
```

###### 1. 使用函数组合 fp.flowRight()重新实现下面的函数，获取最后一辆车的 stock 属性值

```javascript
let isLastInStock = function (cars) {
    let lastCar = fp.last(cars);
    return fp.prop("stock", lastCar);
};
```

答：

```javascript
let isLastInStock = fp.flowRight(fp.prop("stock"), fp.last);
console.log(isLastInStock(cars));
```

###### 2. 使用 fp.flowRight()、fp.prop()和 fp.first()获取第一个 car 的 name

答：

```javascript
let isLastInStock = fp.flowRight(fp.prop("name"), fp.first);
console.log(isLastInStock(cars));
```

###### 3. 使用帮助函数\_average 重构 averageDollarValue，使用函数组合的方式实现

```javascript
let _average = function (xs) {
    return fp.reduce(fp.add, 0, xs) / xs.length;
};

let averageDollarValue = function (cars) {
    let prices = fp.map((car) => car.price, cars);
    return _average(prices);
};
```

答：

```javascript
let averageDollarValue = fp.flowRight(
    _average,
    fp.map((car) => car.price)
);
console.log(averageDollarValue(cars));
```

###### 4. 使用 fp.flowRight()写一个 sanitizeNames 函数，返回一个下划线连接的小写字符串，把数组中的 name 转成如下形式： sanitizeNames(["Hello Wold"])=>["hello_world"]

```javascript
let _underscore = fp.replace(/\W+/g, "_"); // 无需改动，并在sanitizeNames中使用它
```

答：

```javascript
let sanitizeNames = fp.flowRight(fp.map(fp.toLower), fp.map(_underscore));
console.log(sanitizeNames(["Hello Wold"]));
```

##### 三、代码题 2

基于以下代码完成下面的四个练习

```javascript
// support.js
class Container {
    static of(value) {
        return new Container(value);
    }
    constructor(value) {
        this._value = value;
    }
    map(fn) {
        return Container.of(fn(this._value));
    }
}

class Maybe {
    static of(x) {
        return new Maybe(x);
    }
    constructor(x) {
        this._value = x;
    }
    isNothing() {
        return this._value === null || this._value === undefined;
    }
    map(fn) {
        return this.isNothing() ? this : Maybe.of(fn(this._value));
    }
}
```

###### 1. 使用 fp.add(x,y) 和 fp.map(f,x)创建一个能让 functor 里的值增加的函数 ex1。

```javascript
const fp = require("lodash/fp")
const {Maybe,Container} = require("./support.js")

let maybe = Maybe.of([5,6,1])
let ex1 = //...你需要实现的位置
```

答：

```javascript
let ex1 = function (functor) {
    return functor.map(fp.map(fp.add(1)));
};
```

###### 2. 实现一个函数 ex2，能够使用 fp.first 获取列表的第一个元素。

```javascript
const fp = require("lodash/fp")
const {Maybe,Container} = require("./support.js")

let xs = Container.of(['a','b','c','d'])
let ex2 = //...你需要实现的位置
```

答：

```javascript
let ex2 = function (functor) {
    return functor.map(fp.first)._value;
};
console.log(ex2(xs));
```

###### 3. 实现一个函数 ex3，使用 safeProp 和 fp.first 找到 user 的名字首字母。

```javascript
const fp = require("lodash/fp")
const {Maybe,Container} = require("./support.js")

let safeProp = fp.curry(function(x,o){
    return Maybe.of(o[x])
})
let user = {id:2,name:'Albert'}
let ex3 = //...你需要实现的位置
```

答：

```javascript
let ex3 = function (data) {
    return fp.first(safeProp("name")(data)._value);
};
console.log(ex3(user));
```

###### 4. 使用 Maybe 重写 ex4，不要有 if 语句。

```javascript
const fp = require("lodash/fp");
const { Maybe, Container } = require("./support.js");

let ex4 = function (n) {
    if (n) {
        return parseInt(n);
    }
};
```

答：

```javascript
let ex4 = (n) => Maybe.of(n).map(parseInt)._value;
console.log(ex4(1.444));
console.log(ex4(0));
console.log(ex4());
```
