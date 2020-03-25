# ECMA Script 6

本文主要介绍ECMA Script 6.0中演变过的新语法，在新版的JavaScript中可支持，Google Chrome（`51.0.2704.84`）中纯脚本测试结果！兼容性测试表参考：

[https://kangax.github.io/compat-table/es6/](https://kangax.github.io/compat-table/es6/)

（\*：后边代码中，如果直接写的`===`则是为了防止使用console.log方法打印该值，而直接表示当前变量的值就是`===`之后的值！）

## 1. Constants

新版的JavaScript中支持常量定义，常量代码如下，和Java一样，如果是Object类型仅仅是这个对象引用不可重新赋值，但对象内容可变更：

```javascript
const PI = 3.23;
```

## 2. Scoping

### 2.1. Block-Scoped Variables

新关键字`let`，仅仅支持块域，看下边代码：

```javascript
for(let i = 0; i < 3; i++ ){
    console.log(i);
}
console.log(i);         // 报错：Uncaught ReferenceError: i is not defined

for(var j = 0; j < 3; j++ ){
    console.log(j);
}
console.log(j);         // 不报错，直接输出：3
```

\*：从上边的测试可以知道，let比原来的var定义变量的范围小，只支持在某一个Block内的变量使用域，而第二个循环为原始JavaScript写法。

### 2.2. Block-Scoped Functions Definition

在ECMA中支持块语句，替换原来的块闭包语法：

_原始语法_

```javascript
(function(){

})();
```

_新语法_

```javascript
{
    function foo(){
        return 1;
    }
    foo() === 1;
}
console.log(foo());     // 1

// 原始语法对比
(function(){
    function test(){
        return 2;
    }
})();
console.log(test());    // Uncaught ReferenceError: test is not defined
```

\*：这里的块域主要涉及的是函数本身的域问题，并不是闭包，所以上边的`console.log(foo())`依然可执行，并且不像最后那句抛出错误信息，而这里的块语法主要是针对函数的定义作用域，也就是在Block内定义Function的功能，上述例子仅仅只是一种对比，下边的例子更加能够说明Block Function的特性——最后需要注意的是这个功能只能在块内适用，这一点一定要对比上述例子的第一个：

```javascript
{
    function foo() {
        return 2;
    }
    {
        function foo() {
            return 1;
        }
        console.log(foo());         // 1
    }
    console.log(foo());             // 2
}
```

## 3.Arrow Functions

### 3.1. Expression Bodies

支持更多的表达式的闭包语法：

```javascript
var events = [1, 2, 3];
console.log(events);

var adds = events.map(v => v + 1);
console.log(adds);
var paris = events.map(v => ({event: v, odd: v + 1}));
console.log(paris);
var nums = events.map((v, i) => v + i);
console.log(nums);
```

看看上边的代码输出，理解一下这些新算法：

```
[1, 2, 3]
[2, 3, 4]
[Object, Object, Object]        // Object = { event: 1, odd: 2 } ...
[1, 3, 5]
```

### 3.2. Statement Bodies

```javascript
var events = [1, 2, 3, 4, 5, 6, 7];
var threes = [];
events.forEach(v => {
    if( v % 3 == 0 ){
        threes.push(v);
    }
});
console.log(threes);
```

上边代码的输出结果：

```
[3, 6]
```

### 3.3. Lexical this

提供了更多当前Object上下文中直接的this操作：

**EC6**

```javascript
this.nums.forEach((v) => {
    if (v % 5 === 0)
        this.fives.push(v)
})
```

**EC5**

```javascript
//  variant 1
var self = this;
this.nums.forEach(function (v) {
    if (v % 5 === 0)
        self.fives.push(v);
});

//  variant 2 (since ECMAScript 5.1 only)
this.nums.forEach(function (v) {
    if (v % 5 === 0)
        this.fives.push(v);
}.bind(this));
```

## 4. Extended Parameter Handling

### 4.1. Default Parameter Values

EC6中支持默认值直接在形参中赋值的语法（类似PHP）

```javascript
function fun(x, y = 7, z = 50){
    return x + y + z;
}

console.log(fun(1));
console.log(fun(1,0));
```

上边代码输出为：

```
58
51
```

### 4.2. Rest Parameter

EC6开始支持可变参数的基础语法：

```javascript
function fun(x, y, ...z){
    return (x + y) * z.length;
}

console.log(fun(1,2,true,"Hello"));         // 输出结果6
```

\*：和Java中的可变参数一样，Rest Parameter只能放到函数形参的最后一个参数位！

### 4.3. Spread Operator

EC6支持一种新的连接两个数组的语法（注意Rest Parameter的实参传递方式），其语法如下：

```javascript
function fun(x, y, ...z){
    return (x + y) * z.length;
}
var params = ["hello",true,7];
var other = [1,2,...params];
console.log(other);
console.log(fun(1,2,...params));

var str = "Hello";
var chars = [...str];
console.log(chars);
```

代码输出如下：

```
[1, 2, "hello", true, 7]
7 9
["H", "e", "l", "l", "o"]
```

## 5.Template Literials

### 5.1. String Interpolation

EC6支持多行的字面量赋值方式，并且支持在String中使用表达式，使用符号为 \`：
```javascript
    var customer = { name: "Lang" };
    var card = { amount: 7, product: "EC6", unitprice: 41.2 }
    message = `Hello ${customer.name},
    want to buy ${card.amount} ${card.product} for
    a total of ${card.amount * card.unitprice} bucks?`

    console.log(message);
```
最终打印结果如下：

```
Hello Lang,
want to buy 7 EC6 for
a total of 288.40000000000003 bucks?
```

### 5.2. Custom Interpolation

EC6除了字符串的的语法支持以外，还支持自定义的部分语法，对比两个规范中的等价语法对比：

_EC6_

    get`http://example.com/foo?bar=${bar + baz}&quux=${quux}`

_EC5_

```
get([ "http://example.com/foo?bar=", "&quux=", "" ],bar + baz, quux);
```

### 5.3. Raw String Access

EC6支持新语法，而这个语法在原始的JavaScript是不支持的！
```javascript
    function quux(strings, ...values){
        console.log(strings[0]);
        console.log(strings[1]);
        console.log(strings.raw[0]);
        console.log(strings.raw[1]);
        console.log(values[0]);
    }

    quux `foo\n${ 42 }bar`;
```
注意这段代码的输出（主要是注意换行符号）：

```
foo

bar
foo\n
bar
42
```

## 6. Extended Literials

### 6.1. Binary & Octal Literal

EC6中开始支持二进制和八进制的字面量了（前缀`0b`和`0o`大小写不敏感）：

```javascript
console.log(0b111110111);       // 503
console.log(0o767);             // 503
```

### 6.2. Unicode String & RegExp Literal

EC6中支持Unicode字符串字面量以及正则表达式的字面量表示：

```javascript
console.log("𠮷".length);
console.log("𠮷".match(/./u));
console.log("𠮷" === "\uD842\uDFB7");
console.log("\u{20BB7}");
console.log("\uD842\uDFB7");
console.log("𠮷".charCodeAt(0));
for (let codepoint of "𠮷") console.log(codepoint);
```

上边的代码输出结果为：

```
2
["𠮷", index: 0, input: "𠮷"]
true
𠮷
𠮷
55362
𠮷
```

## 7.Enhanced Regular Expression

### 7.1. Regular Expression Sticky Matching

EC6介绍（因为对JS这部分语法不熟，就不翻译了）

_Keep the matching position sticky between matches and this way support efficient parsing of arbitrary long input strings, even with an arbitrary number of distinct regular expressions._

```javascript
let parser = (input, match) => {
    for (let pos = 0, lastPos = input.length; pos < lastPos; ) {
        for (let i = 0; i < match.length; i++) {
            match[i].pattern.lastIndex = pos
            let found
            if ((found = match[i].pattern.exec(input)) !== null) {
                match[i].action(found)
                pos = match[i].pattern.lastIndex
                break
            }
        }
    }
}

let report = (match) => {
    console.log(JSON.stringify(match))
}
parser("Foo 1 Bar 7 Baz 42", [
    { pattern: /^Foo\s+(\d+)/y, action: (match) => report(match) },
    { pattern: /^Bar\s+(\d+)/y, action: (match) => report(match) },
    { pattern: /^Baz\s+(\d+)/y, action: (match) => report(match) },
    { pattern: /^\s*/y,         action: (match) => {}            }
])
```

## 8. Enhanced Object Properties

### 8.1. Property Shorthand

对于常用的Object定义可以使用下边方式：

_EC6_

```javascript
obj = { name, email }
```

_EC5_

```javascript
obj = { name:name, email:email }
```

例如下边代码：

```javascript
let name = "Lang";
let email = "lang.yu@hpe.com";
obj = { name, email };
console.log(obj);
```

打印的最终结果如：

```
Object {name: "Lang", email: "lang.yu@hpe.com"}
```

### 8.2. Computed Property Names

EC6支持直接对Object中的name进行运算：

```javascript
function quux(){ return "test"; }
let obj = {
    foo: "lang",
    ["baz" + quux()]: 42
}
console.log(obj);
```

最终打印结果如：

```
Object {foo: "lang", baztest: 42}
```

### 8.3. Method Properties

EC6中开始在Object中支持方法直接属性定义，而且同时支持正则表达式的方法以及生成器方法（再也不依赖function关键字）：

```javascript
obj = {
    foo (a, b) {
    return "foo";
    },
    bar (x, y) {
        return "bar";
    },
    *quux (x, y) {
        return "quux";
    }
}

console.log(obj);
console.log(obj.foo(1,2));
console.log(obj.bar(1,2));
console.log(obj.quux(3,4));
```

输出为：

```
Object {}
foo
bar
quux {[[GeneratorStatus]]: "suspended", [[GeneratorReceiver]]: Object}      // 注意这一行的写法
```

## 9. Destructuring Assignment

### 9.1. Array Matching

EC6中支持更加松散和自由的数组Array赋值操作：

_EC6_

```javascript
var list = [ 1, 2, 3 ]
var [ a, , b ] = list       // a = 1, b = 3
[ b, a ] = [ a, b ]         // a = 3, b = 1
console.log(a);
console.log(b);
```

_EC5_

```javascript
var list = [ 1, 2, 3 ];
var a = list[0], b = list[2];
var tmp = a; a = b; b = tmp;
```

### 9.2. Object Matching, Shorthand Notation

EC6支持更加松散和自由的对象Object赋值操作：

```javascript
function getUser(){
    return {
        name:"lang",
        email:"lang.yu@hpe.com",
        value:"testValue"
    }
}
var { name, email, mobile } = getUser();
console.log(name);
console.log(email);
console.log(mobile);
```

输出信息：

```
lang
lang.yu@hpe.com
undefined
```

### 9.3. Object Matching, Deep Matching

EC6还可以针对对象Object进行深度松散自由赋值：

```javascript
function getUser(){
    var addr = "lang.yu@hpe.com";
    return {
        name:"lang",
        email:{
            addr
        },
        value:"testValue"
    }
}
var { name, email:{ addr:op }, mobile } = getUser();
console.log(name);
console.log(op);
console.log(mobile);
```

输出信息：

```
Object {name: "lang", email: Object, value: "testValue"}
lang
lang.yu@hpe.com
undefined
```

### 9.4. Parameter Context Matching

EC6中Array和Object也支持更加松散和自由的赋值形式：

```javascript
function fun([name,val]){
    console.log(name, val);
}
function gun({name:n, val:v}){
    console.log(n,v);
}
function hun({name,val}){
    console.log(name,val);
}
fun([true,"Hello"]);
gun({name:"lang",val:"Value"});
hun({name:"lang",val:"Value2"});
```

输出结果如：

```
true "Hello"
lang Value
lang Value2
```

### 9.5. Fail-Soft Destructuring

EC6中对Array自由赋值的默认值设置：

```javascript
var list = [ 7, 42 ]
var [ a = 1, b = 2, c = 3, d ] = list
console.log(a);
console.log(b);
console.log(c);
console.log(d);
```

输出结果如：

```
7
42
3
undefined
```

## 10.Modules（Compiler Time Only）

### 10.1.Value Export/Import

EC6中一个很重要的语法，就是支持模块化操作，如果没有全局名空间时，还支持将Module中的内容Export/Import：

```javascript
//  lib/math.js
export function sum (x, y) { return x + y }
export var pi = 3.141593

//  someApp.js
import * as math from "lib/math"
console.log("2π = " + math.sum(math.pi, math.pi))

//  otherApp.js
import { sum, pi } from "lib/math"
console.log("2π = " + sum(pi, pi))
```

\*：需要说明的一点是，import/export两个关键字的使用只能在静态编译中使用，如果要在浏览器中使用必须使用类似webpack这种已经将JS打包成bundle.js的工具，最终才能够在浏览器平台直接使用该语法！

### 10.2.Default & Wildcard

该语法同上，编译期有用，运行时直接使用在浏览器中会报错！

```javascript
//  lib/mathplusplus.js
export * from "lib/math"
export var e = 2.71828182846
export default (x) => Math.exp(x)

//  someApp.js
import exp, { pi, e } from "lib/mathplusplus"
console.log("e^{π} = " + exp(pi))
```

**Static vs. dynamic, or: rules and how to break them**

* For a dynamic language, JavaScript has gotten itself a surprisingly static module system.
* All flavors of import and export are allowed only at toplevel in a module. There are no conditional imports or exports, and you can’t use import in function scope.
* All exported identifiers must be explicitly exported by name in the source code. You can’t programmatically loop through an array and export a bunch of names in a data-driven way.
* Module objects are frozen. There is no way to hack a new feature into a module object, polyfill style.
* All of a module’s dependencies must be loaded, parsed, and linked eagerly, before any module code runs. There’s no syntax for an import that can be loaded lazily, on demand.
* There is no error recovery for import errors. An app may have hundreds of modules in it, and if anything fails to load or link, nothing runs. You can’t import in a try/catch block. \(The upside here is that because the system is so static, webpack can detect those errors for you at compile time.\)
* There is no hook allowing a module to run some code before its dependencies load. This means that modules have no control over how their dependencies are loaded.

## 11. Classes

### 11.1. Class Definition

EC6中开始支持OOP编程

```javascript
class User{
    constructor(id, name, email){
        this.id = id;
        this.name = name;
        this.email = email;
    }
    move(x,y){
        this.x = x;
        this.y = y;
    }
    get(){
        return this.name + this.email;
    }
}

var user = new User("id","lang","lang.yu@hpe.com");
console.log(user.get());
```

### 11.2. Class Inheritance

```javascript
class User{
    constructor(id,name){
        this.name = name;
        this.id = id;
    }
    get(){
        return this.name;
    }
}
class Teacher extends User{
    constructor(id,name,email){
        super(id,name);
        this.email = email;
    }
}
var teacher = new Teacher("id","Lang");
console.log(teacher.get());
```

### 11.3. Class Inheritance, From Expressions

EC6除了支持OOP，还支持混合模式的OOP操作：

```javascript
var aggregation = (baseClass, ...mixins) => {
    let base = class _Combined extends baseClass {
        constructor (...args) {
            super(...args)
            mixins.forEach((mixin) => {
                mixin.prototype.initializer.call(this)
            })
        }
    }
    let copyProps = (target, source) => {
        Object.getOwnPropertyNames(source)
            .concat(Object.getOwnPropertySymbols(source))
            .forEach((prop) => {
                if (prop.match(/^(?:constructor|prototype|arguments|caller|name|bind|call|apply|toString|length)$/))
                    return
                Object.defineProperty(target, prop, Object.getOwnPropertyDescriptor(source, prop))
            })
    }
    mixins.forEach((mixin) => {
        copyProps(base.prototype, mixin.prototype)
        copyProps(base, mixin)
    })
    return base
}

class Colored {
    initializer ()     { this._color = "white" }
    get color ()       { return this._color }
    set color (v)      { this._color = v }
}

class ZCoord {
    initializer ()     { this._z = 0 }
    get z ()           { return this._z }
    set z (v)          { this._z = v }
}

class Shape {
    constructor (x, y) { this._x = x; this._y = y }
    get x ()           { return this._x }
    set x (v)          { this._x = v }
    get y ()           { return this._y }
    set y (v)          { this._y = v }
}

class Rectangle extends aggregation(Shape, Colored, ZCoord) {}

var rect = new Rectangle(7, 42)
rect.z     = 1000
rect.color = "red"
console.log(rect.x, rect.y, rect.z, rect.color)
```

### 11.4. Base Class Access
```javascript
    class Shape {
        toString () {
            return `Shape(${this.id})`
        }
    }
    class Rectangle extends Shape {
        constructor (id, x, y, width, height) {
            super(id, x, y);
        }
        toString () {
            return "Rectangle > " + super.toString()
        }
    }
    class Circle extends Shape {
        constructor (id, x, y, radius) {
            super(id, x, y);
        }
        toString () {
            return "Circle > " + super.toString()
        }
    }
```
### 11.5. Static Members

```javascript
class Rectangle extends Shape {
    static defaultRectangle () {
        return new Rectangle("default", 0, 0, 100, 100)
    }
}
class Circle extends Shape {
    static defaultCircle () {
        return new Circle("default", 0, 0, 100)
    }
}
var defRectangle = Rectangle.defaultRectangle()
var defCircle    = Circle.defaultCircle()
console.log(defRectangle);
console.log(defCircle);
```

### 11.6. Get/Set

```javascript
class Rectangle {
    constructor (width, height) {
        this._width  = width
        this._height = height
    }
    set width  (width)  { this._width = width               }
    get width  ()       { return this._width                }
    set height (height) { this._height = height             }
    get height ()       { return this._height               }
    get area   ()       { return this._width * this._height }
}
var r = new Rectangle(50, 20)
console.log(r.area);
```

## 12.Symbol Type

### 12.1. Symbol Type

```javascript
Symbol("foo") !== Symbol("foo")
const foo = Symbol()
const bar = Symbol()
typeof foo === "symbol"
typeof bar === "symbol"
let obj = {}
obj[foo] = "foo"
obj[bar] = "bar"
JSON.stringify(obj) // {}
Object.keys(obj) // []
Object.getOwnPropertyNames(obj) // []
Object.getOwnPropertySymbols(obj) // [ foo, bar ]
```

### 12.2. Global Symbols

```javascript
Symbol.for("app.foo") === Symbol.for("app.foo")
const foo = Symbol.for("app.foo")
const bar = Symbol.for("app.bar")
Symbol.keyFor(foo) === "app.foo"
Symbol.keyFor(bar) === "app.bar"
typeof foo === "symbol"
typeof bar === "symbol"
let obj = {}
obj[foo] = "foo"
obj[bar] = "bar"
JSON.stringify(obj) // {}
Object.keys(obj) // []
Object.getOwnPropertyNames(obj) // []
Object.getOwnPropertySymbols(obj) // [ foo, bar ]
```

## 13.Iterators

### 13.1. Iterator & For-Of Operator

\*：注意死循环问题

```javascript
let fibonacci = {
    [Symbol.iterator]() {
        let pre = 0, cur = 1
        return {
            next () {
                [ pre, cur ] = [ cur, pre + cur ]
                return { done: false, value: cur }
            }
        }
    }
}

for (let n of fibonacci) {
    if (n > 1000)
        break
    console.log(n)
}
```

## 14.Generators

### 14.1. Generator Function, Iterator Protocol

EC6中最具“魔力”的功能特性，基于迭代协议的生成器：

```javascript
let fibonacci = {
    *[Symbol.iterator]() {
        let pre = 0, cur = 1
        for (;;) {
            [ pre, cur ] = [ cur, pre + cur ]
            yield cur
        }
    }
}

for (let n of fibonacci) {
    if (n > 1000)
        break
    console.log(n)
}
```

生成器函数的定义方式在函数名之前加`*`号，然后在函数执行过程中可调用`yield`的方式将该函数暂停！

### 14.2. Generator Function, Direct Use

```javascript
function* range (start, end, step) {
    while (start < end) {
        yield start
        start += step
    }
}

for (let i of range(0, 10, 2)) {
    console.log(i) // 0, 2, 4, 6, 8
}
```

### 14.3. Generator Matching

```javascript
let fibonacci = function* (numbers) {
    let pre = 0, cur = 1
    while (numbers-- > 0) {
        [ pre, cur ] = [ cur, pre + cur ]
        yield cur
    }
}

for (let n of fibonacci(1000))
    console.log(n)

let numbers = [ ...fibonacci(1000) ]

let [ n1, n2, n3, ...others ] = fibonacci(1000)
```

### 14.4. Generator Control-Flow
```javascript
    //  generic asynchronous control-flow driver
    function async (proc, ...params) {
        var iterator = proc(...params)
        return new Promise((resolve, reject) => {
            let loop = (value) => {
                let result
                try {
                    result = iterator.next(value)
                }
                catch (err) {
                    reject(err)
                }
                if (result.done)
                    resolve(result.value)
                else if (   typeof result.value === "object"
                    && typeof result.value.then === "function")
                    result.value.then((value) => {
                        loop(value)
                    }, (err) => {
                        reject(err)
                    })
                else
                    loop(result.value)
            }
            loop()
        })
    }

    //  application-specific asynchronous builder
    function makeAsync (text, after) {
        return new Promise((resolve, reject) => {
            setTimeout(() => resolve(text), after)
        })
    }

    //  application-specific asynchronous procedure
    async(function* (greeting) {
        let foo = yield makeAsync("foo", 300)
        let bar = yield makeAsync("bar", 200)
        let baz = yield makeAsync("baz", 100)
        return `${greeting} ${foo} ${bar} ${baz}`
    }, "Hello").then((msg) => {
        console.log("RESULT:", msg) // "Hello foo bar baz"
    })
```
### 14.5. Generator Methods

```javascript
class Clz {
    * bar () {
    }
}
let Obj = {
    * foo () {
    }
}
```

## 15. Map/Set & WeakMap/WeakSet

### 15.1. Set Data-Structure

```javascript
let s = new Set()
s.add("hello").add("goodbye").add("hello")
s.size === 2
s.has("hello") === true
for (let key of s.values()) // insertion order
    console.log(key)
```

### 15.2. Map Data-Structure

```javascript
let m = new Map()
m.set("hello", 42)
m.set("key", 34)
m.get("key") === 34
m.size === 2
for (let [ key, val ] of m.entries())
    console.log(key + " = " + val)
```

### 15.3. Map/Set & WeakMap/WeakSet

```javascript
let isMarked     = new WeakSet()
let attachedData = new WeakMap()

class Node {
    constructor (id)   { this.id = id                  }
    mark        ()     { isMarked.add(this)            }
    unmark      ()     { isMarked.delete(this)         }
    marked      ()     { return isMarked.has(this)     }
    set data    (data) { attachedData.set(this, data)  }
    get data    ()     { return attachedData.get(this) }
}

let foo = new Node("foo")

JSON.stringify(foo) === '{"id":"foo"}'
foo.mark()
foo.data = "bar"
foo.data === "bar"
JSON.stringify(foo) === '{"id":"foo"}'

isMarked.has(foo)     === true
attachedData.has(foo) === true
foo = null  /* remove only reference to foo */
attachedData.has(foo) === false
isMarked.has(foo)     === false
```

## 16.Typed Arrays

### 16.1.Typed Arrays

从EC6开始支持字节级别的数据类型用于到网络传输、加密、格式化等操作：

```javascript
//  ES6 class equivalent to the following C structure:
//  struct Example { unsigned long id; char username[16]; float amountDue }
class Example {
    constructor (buffer = new ArrayBuffer(24)) {
        this.buffer = buffer
    }
    set buffer (buffer) {
        this._buffer    = buffer
        this._id        = new Uint32Array (this._buffer,  0,  1)
        this._username  = new Uint8Array  (this._buffer,  4, 16)
        this._amountDue = new Float32Array(this._buffer, 20,  1)
    }
    get buffer ()     { return this._buffer       }
    set id (v)        { this._id[0] = v           }
    get id ()         { return this._id[0]        }
    set username (v)  { this._username[0] = v     }
    get username ()   { return this._username[0]  }
    set amountDue (v) { this._amountDue[0] = v    }
    get amountDue ()  { return this._amountDue[0] }
}

let example = new Example()
example.id = 7
example.username = "John Doe"
example.amountDue = 42.0
```

## 17. New Built-In Methods

### 17.1. Object Property Assignment

对象直接赋值

```javascript
var dst  = { quux: 0 }
var src1 = { foo: 1, bar: 2 }
var src2 = { foo: 3, baz: 4 }
Object.assign(dst, src1, src2)
dst.quux === 0
dst.foo  === 3
dst.bar  === 2
dst.baz  === 4
```

### 17.2. Array Element Finding

数组的查找操作：

```
[ 1, 3, 4, 2 ].find(x => x > 3) // 4
```

\*：注意这个函数，看下边代码的输出：

```javascript
var x = [ 1, 3, 4, 2 ].find(x => x > 3) // 4
var y = [ 1, 3, 4, 2 ].find(x => x > 1) // 3
console.log(x);         // 4
console.log(y);         // 3
```

从输出可以知道返回值就一个元素，不会返回数组！

### 17.3. String Repeating

重复字符串的方法

```
" ".repeat(4 * depth)
"foo".repeat(3)
```

### 17.4. String Searching

常用的String的API，用于进行String之内的搜索：

```javascript
"hello".startsWith("ello", 1) // true
"hello".endsWith("hell", 4)   // true
"hello".includes("ell")       // true
"hello".includes("ell", 1)    // true
"hello".includes("ell", 2)    // false
```

### 17.5. Number Type Checking

数值类型检查

```javascript
Number.isNaN(42) === false
Number.isNaN(NaN) === true

Number.isFinite(Infinity) === false
Number.isFinite(-Infinity) === false
Number.isFinite(NaN) === false
Number.isFinite(123) === true
```

### 17.6. Number Safety Checking

数值安全检查，检查是否越界

```javascript
Number.isSafeInteger(42) === true
Number.isSafeInteger(9007199254740992) === false
```

### 17.7. Number Comparision

数值比较，提供了一些标准数据：

_EC6_

```javascript
console.log(0.1 + 0.2 === 0.3) // false
console.log(Math.abs((0.1 + 0.2) - 0.3) < Number.EPSILON) // true
```

_EC5_

```javascript
console.log(0.1 + 0.2 === 0.3); // false
console.log(Math.abs((0.1 + 0.2) - 0.3) < 2.220446049250313e-16); // true
```

### 17.8. Number Truncation

数值截断操作：

```javascript
console.log(Math.trunc(42.7)) // 42
console.log(Math.trunc( 0.1)) // 0
console.log(Math.trunc(-0.1)) // -0
```

### 17.9. Number Sign Determination

```javascript
console.log(Math.sign(7))   // 1
console.log(Math.sign(0))   // 0
console.log(Math.sign(-0))  // -0
console.log(Math.sign(-7))  // -1
console.log(Math.sign(NaN)) // NaN
```

## 18.Promises

### 18.1. Promise Usage

支持Promise异步调用规范：
```javascript
    function msgAfterTimeout (msg, who, timeout) {
        return new Promise((resolve, reject) => {
            setTimeout(() => resolve(`${msg} Hello ${who}!`), timeout)
        })
    }
    msgAfterTimeout("", "Foo", 100).then((msg) =>
        msgAfterTimeout(msg, "Bar", 200)
    ).then((msg) => {
        console.log(`done after 300ms:${msg}`)
    })
```
### 18.2. Promise Combination

支持Promise规范下的组合功能
```javascript
    function fetchAsync (url, timeout, onData, onError){

    }

    let fetchPromised = (url, timeout) => {
        return new Promise((resolve, reject) => {
            fetchAsync(url, timeout, resolve, reject)
        })
    }
    Promise.all([
        fetchPromised("http://backend/foo.txt", 500),
        fetchPromised("http://backend/bar.txt", 500),
        fetchPromised("http://backend/baz.txt", 500)
    ]).then((data) => {
        let [ foo, bar, baz ] = data
        console.log(`success: foo=${foo} bar=${bar} baz=${baz}`)
    }, (err) => {
        console.log(`error: ${err}`)
    })
```
## 19.Meta-Programming

### 19.1. Proxying

动态代理调用技术
```javascript
    let target = {
        foo: "Welcome, foo"
    }
    let proxy = new Proxy(target, {
        get (receiver, name) {
            return name in receiver ? receiver[name] : `Hello, ${name}`
        }
    })
    proxy.foo   === "Welcome, foo"
    proxy.world === "Hello, world"
```
### 19.2. Reflection

JS中的反射

```javascript
let obj = { a: 1 }
Object.defineProperty(obj, "b", { value: 2 })
obj[Symbol("c")] = 3
Reflect.ownKeys(obj) // [ "a", "b", Symbol(c) ]
```

## 20. Internationalization & Localization

### 20.1. Collation

```javascript
// in German,  "ä" sorts with "a"
// in Swedish, "ä" sorts after "z"
var list = [ "ä", "a", "z" ]
var l10nDE = new Intl.Collator("de")
var l10nSV = new Intl.Collator("sv")
l10nDE.compare("ä", "z") === -1
l10nSV.compare("ä", "z") === +1
console.log(list.sort(l10nDE.compare)) // [ "a", "ä", "z" ]
console.log(list.sort(l10nSV.compare)) // [ "a", "z", "ä" ]
```

### 20.2. Number Formatting

```javascript
var l10nEN = new Intl.NumberFormat("en-US")
var l10nDE = new Intl.NumberFormat("de-DE")
l10nEN.format(1234567.89) === "1,234,567.89"
l10nDE.format(1234567.89) === "1.234.567,89"
```

### 20.3. Currency Formatting

```javascript
var l10nUSD = new Intl.NumberFormat("en-US", { style: "currency", currency: "USD" })
var l10nGBP = new Intl.NumberFormat("en-GB", { style: "currency", currency: "GBP" })
var l10nEUR = new Intl.NumberFormat("de-DE", { style: "currency", currency: "EUR" })
l10nUSD.format(100200300.40) === "$100,200,300.40"
l10nGBP.format(100200300.40) === "£100,200,300.40"
l10nEUR.format(100200300.40) === "100.200.300,40 €"
```

### 20.4. Date/Time Formatting

```javascript
var l10nEN = new Intl.DateTimeFormat("en-US")
var l10nDE = new Intl.DateTimeFormat("de-DE")
l10nEN.format(new Date("2015-01-02")) === "1/2/2015"
l10nDE.format(new Date("2015-01-02")) === "2.1.2015"
```

## Summary

None
