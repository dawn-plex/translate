> * 原文地址：[Let’s explore objects in JavaScript](https://medium.freecodecamp.org/lets-explore-objects-in-javascript-4a4ad76af798)
> * 原文作者：[Cristi Salcescu](https://medium.freecodecamp.org/@cristisalcescu)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/Let’s-explore-objects-in-JavaScript.md](https://github.com/dawn-teams/translate/blob/master/articles/Let’s-explore-objects-in-JavaScript.md)
> * 译者：[灵沼](https://github.com/su-dan)
> * 校对者：[也树](https://github.com/xdlrt)[眠云](https://github.com/JeromeYangtao)

---
# 一起探讨 JavaScript 的对象

对象是多个属性的动态集合，它有一个链接着原型的隐藏属性（注：`__proto__`）。

一个属性拥有一个 key 和一个 value 。

## 属性的 key
属性的 key 是一个唯一的字符串。

访问属性有两种方式：点表示法和括号表示法。当使用点表示法，属性的 key 必须是有效的标识符。

```
let obj = {
  message : "A message"
}
obj.message //"A message"
obj["message"] //"A message"
```

访问一个不存在的属性不会抛出错误，但是会返回 `undefined`。

```
obj.otherProperty //undefined
```
当使用括号表示法，属性的 key 不要求是有效的标识符 —— 可以是任意值。

```
let french = {};
french["thank you very much"] = "merci beaucoup";

french["thank you very much"]; //"merci beaucoup"
```
当属性的key是一个非字符串的值，会用toString()方法（如果可用的话）把它转换为字符串。

```
let obj = {};
//Number
obj[1] = "Number 1";
obj[1] === obj["1"]; //true
//Object
let number1 = {
  toString : function() { return "1"; }
}
obj[number1] === obj["1"]; //true
```

在上面的示例中，对象 `number1` 被用作一个 key 。它会被转换为字符串，转换结果 “1” 被用作属性的 key 。

## 属性的值
属性的值可以是任意的基础数据类型，对象，或函数。

## 对象作为值
对象可以嵌套在其他对象里。[看下面这个例子](https://jsfiddle.net/cristi_salcescu/m0a65e2g/)：

```
let book = {
  title : "The Good Parts",
  author : {
    firstName : "Douglas",
    lastName : "Crockford"
  }
}
book.author.firstName; //"Douglas"
```

通过这种方式，我们就可以创建一个命名空间:

```
let app = {};
app.authorService = { getAuthors : function() {} };
app.bookService = { getBooks : function() {} };
```

## 函数作为值
当一个函数被作为属性值，通常成为一个方法。在方法中，`this` 关键字代表着当前的对象。

`this` ，会根据函数的调用方式有不同的值。了解更多关于`this` 丢失上下文的问题，可以查看[当"this"丢失上下文时应该怎么办](https://medium.freecodecamp.org/what-to-do-when-this-loses-context-f09664af076f)。

## 动态性
对象本质上就是动态的。可以任意添加删除属性。

```
let obj = {};
obj.message = "This is a message"; //add new property
obj.otherMessage = "A new message"; //add new property
delete obj.otherMessage; //delete property
```

## Map
我们可以把对象当做一个 Map。Map 的 key 就是对象的属性。

访问一个 key 不需要去扫描所有属性。访问的时间复杂度是 o(1)。

## 原型
对象有一个链接着原型对象的“隐藏”属性 `__proto__`，对象是从这个原型对象中继承属性的。

举个例子，使用对象字面量创建的对象有一个指向 `Object.prototype` 的链接:

```
var obj = {};
obj.__proto__ === Object.prototype; //true
```

### 原型链
原型对象有它自己的原型。当一个属性被访问的时候并且不包含在当前对象中，JavaScript会沿着原型链向下查找直到找到被访问的属性，或者到达 `null` 为止。

### 只读
原型只用于读取值。对象进行更改时，只会作用到当前对象，不会影响对象的原型；就算原型上有同名的属性，也是如此。

### 空对象
正如我们看到的，空对象 `{}` 并不是真正意义上的空，因为它包含着指向 `Object.prototype` 的链接。为了创建一个真正的空对象，我们可以使用 `Object.create(null)` 。它会创建一个没有任何属性的对象。这通常用来创建一个Map。

## 原始值和包装对象
在允许访问属性这一点上，JavaScript 把原始值描述为对象。当然了，原始值并不是对象。

```
(1.23).toFixed(1); //"1.2"
"text".toUpperCase(); //"TEXT"
true.toString(); //"true"
```

为了允许访问原始值的属性， JavaScript 创造了一个包装对象，然后销毁它。JavaScript引擎对创建包装和销毁包装对象的过程做了优化。

数值、字符串和布尔值都有等效的包装对象。跟别是：`Number`、`String `、`Boolean`。

`null` 和 `undefined` 原始值没有相应的包装对象并且不提供任何方法。

## 内置原型
Numbers 继承自`Number.prototype`，`Number.prototype`继承自`Object.prototype`。

```
var no = 1;
no.__proto__ === Number.prototype; //true
no.__proto__.__proto__ === Object.prototype; //true
```
Strings 继承自 `String.prototype`。Booleans 继承自 `Boolean.prototype`

函数都是对象，继承自 `Function.prototype` 。函数拥有 `bind()`、`apply()` 和 `call()` 等方法。

所有对象、函数和原始值（除了 `null` 和 `undefined` ）都从 `Object.prototype` 继承属性。他们都有 `toString()` 方法。

## 使用 polyfill 扩充内置对象
JavaScript 可以轻松地使用新功能扩充内置对象。

polyfill 就是一个代码片段，用于在不支持某功能的浏览器中实现该功能。

### 实用工具
举个例子，这个为 `Object.assign()` 写的[polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign#Polyfill)，如果它不可用，那么就在 `Object` 上添加一个新方法。

为 `Array.from()` 写了类似的[polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from#Polyfill)，如果它不可用，就在 `Array` 上添加一个新方法。

### 原型
新的方法可以被添加到原型。

举个例子，`String.prototype.trim()` [polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/Trim#Polyfill)让所有的字符串都能使用 `trim()` 方法。

```
let text = "   A text  ";
text.trim(); //"A text"
```

`Array.prototype.find()` [polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find#Polyfill)让所有的数组都能使用`find()`方法。[polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/findIndex#polyfill)也是同样的。

```
let arr = ["A", "B", "C", "D", "E"];
arr.indexOf("C"); //2
```

## 单一继承
`Object.create()` 用特定的原型对象创建一个新对象。它用来做单一继承。[思考下面的例子](https://jsfiddle.net/cristi_salcescu/zu79cebu/):

```
let bookPrototype = {
  getFullTitle : function(){
    return this.title + " by " + this.author;
  }
}
let book = Object.create(bookPrototype);
book.title = "JavaScript: The Good Parts";
book.author = "Douglas Crockford";
book.getFullTitle();//JavaScript: The Good Parts by Douglas Crockford
```
## 多重继承
`Object.assign()` 从一个或多个对象拷贝属性到目标对象。它用来做多重继承。[看下面的例子](https://jsfiddle.net/cristi_salcescu/ghqsb9a3/):

```
let authorDataService = { getAuthors : function() {} };
let bookDataService = { getBooks : function() {} };
let userDataService = { getUsers : function() {} };
let dataService = Object.assign({},
 authorDataService,
 bookDataService,
 userDataService
);
dataService.getAuthors();
dataService.getBooks();
dataService.getUsers();
```

## 不可变对象
`Object.freeze()` 冻结一个对象。属性不能被添加、删除、更改。对象会变成不可变的。

```
"use strict";
let book = Object.freeze({
  title : "Functional-Light JavaScript",
  author : "Kyle Simpson"
});
book.title = "Other title";//Cannot assign to read only property 'title'
```
`Object.freeze()` 实行浅冻结。要深冻结，需要递归冻结对象的每一个属性。

## 拷贝
`Object.assign()` 被用作拷贝对象。

```
let book = Object.freeze({
  title : "JavaScript Allongé",
  author : "Reginald Braithwaite"
});
let clone = Object.assign({}, book);
```

`Object.assign()` 执行浅拷贝，不是深拷贝。它拷贝对象的第一层属性。嵌套的对象会在原始对象和副本对象之间共享。

## 对象字面量
对象字面量提供一种简单、优雅的方式创建对象。

```
let timer = {
  fn : null,
  start : function(callback) { this.fn = callback; },
  stop : function() {},
}
```
但是，这种语法有一些缺点。所有的属性都是公共的，方法能够被重定义，并且不能在新实例中使用相同的方法。

```
timer.fn;//null 
timer.start = function() { console.log("New implementation"); }
```

## Object.create()

`Object.create()` 和 `Object.freeze()` 一起能够解决最后两个问题。

首先，我要使用所有方法创建一个冻结原型 `timerPrototype` ，然后创建对象去继承它。

```
let timerPrototype = Object.freeze({
  start : function() {},
  stop : function() {}
});
let timer = Object.create(timerPrototype);
timer.__proto__ === timerPrototype; //true
```

当原型被冻结，继承它的对象不能够更改其中的属性。现在，`start()` 和 `stop()` 方法不能被重新定义。

```
"use strict";
timer.start = function() { console.log("New implementation"); } //Cannot assign to read only property 'start' of object
```

`Object.create(timerPrototype)` 可以用来使用相同的原型构建更多对象。

## 构造函数
最初，JavaScript 语言提出构造函数作为这些的语法糖。[看下面的代码](https://jsfiddle.net/cristi_salcescu/az35x2qs/):

```
function Timer(callback){
  this.fn = callback;
}
Timer.prototype = {
  start : function() {},
  stop : function() {}
}
function getTodos() {}
let timer = new Timer(getTodos);
```

所有的以 `function` 关键字定义的函数都可以作为构造函数。构造函数使用功能 `new` 调用。新对象将原型设定为 `FunctionConstructor.prototype`。

```
let timer = new Timer();
timer.__proto__ === Timer.prototype;
```

同样地，我们需要冻结原型来防止方法被重定义。

```
Timer.prototype = Object.freeze({
  start : function() {},
  stop : function() {}
});
```

### new 操作符
当执行 `new Timer()` 时，它与函数 `newTimer()` 作用相同:

```
function newTimer(){
  let newObj = Object.create(Timer.prototype);
  let returnObj = Timer.call(newObj, arguments);
  if(returnObj) return returnObj;
    
  return newObj;
}
```
使用 `Timer.prototype` 作为原型，创造了一个新对象。然后执行 `Timer` 函数并为新对象设置属性字段。

## 类
ES2015为这一切带来了更好的语法糖。[看下面的例子](https://jsfiddle.net/cristi_salcescu/aLg8t632/):

```
class Timer{
  constructor(callback){
    this.fn = callback;
  }
  
  start() {}
  stop() {}  
}
Object.freeze(Timer.prototype);
```

使用 class 构建的对象将原型设置为 `ClassName.prototype` 。在使用类创建对象时，必须使用 `new` 操作符。

```
let timer= new Timer();
timer.__proto__ === Timer.prototype;
```

class 语法不会冻结原型，所以我们需要在之后进行操作。

```
Object.freeze(Timer.prototype);
```

## 基于原型的继承
在 JavaScript 中，对象继承自对象。

构造函数和类都是用来创建原型对象的所有方法的语法糖。然后它创建一个继承自原型对象的新对象，并为新对象设置数据字段
基于原型的继承具有保护记忆的好处。原型只创建一次并且由所有的实例使用。

### 没有封装
基于原型的继承模式没有私有性。所有对象的属性都是公有的。

`Object.keys()` 返回一个包含所有属性键的数组。它可以用来迭代对象的所有属性。

```
function logProperty(name){
  console.log(name); //property name
  console.log(obj[name]); //property value
}
Object.keys(obj).forEach(logProperty);
```

模拟的私有模式包含使用 `_` 来标记私有属性，这样其他人会避免使用他们：

```
class Timer{
  constructor(callback){
    this._fn = callback;
    this._timerId = 0;
  }
}
```

## 工厂模式
JavaScript 提供一种使用工厂模式创建封装对象的新方式。

```
function TodoStore(callback){
    let fn = callback;
    
    function start() {},
    function stop() {}
    
    return Object.freeze({
       start,
       stop
    });
}
```

`fn` 变量是私有的。只有 `start()` 和 `stop()` 方法是公有的。`start()` 和 `stop()` 方法不能被外界改变。这里没有使用 `this` ，所以没有 `this` 丢失上下文的问题。

对象字面量依然用于返回对象，但是这次它只包含函数。更重要的是，这些函数是共享相同私有状态的闭包。 `Object.freeze()` 被用来冻结公有 API。

Timer 对象的完整实现，请看[具有封装功能的实用JavaScript对象](https://medium.freecodecamp.org/here-are-some-practical-javascript-objects-that-have-encapsulation-fc4c1a79c655).

## 结论
JavaScript 像对象一样处理原始值、对象和函数。

对象本质上是动态的，可以用作Map。

对象继承自其他对象。构造函数和类是创建从其他原型对象继承的对象的语法糖。

`Object.create()` 可以用来单一继承，`Object.assign()` 用来多重继承。

工厂函数可以构建封装对象。

有关 JavaScript 功能的更多信息，请看：

[Discover the power of first class functions](https://medium.freecodecamp.org/discover-the-power-of-first-class-functions-fd0d7b599b69)

[How point-free composition will make you a better functional programmer](https://medium.freecodecamp.org/how-point-free-composition-will-make-you-a-better-functional-programmer-33dcb910303a)

[Here are a few function decorators you can write from scratch](https://medium.freecodecamp.org/here-are-a-few-function-decorators-you-can-write-from-scratch-488549fe8f86)

[Why you should give the Closure function another chance](https://medium.freecodecamp.org/why-you-should-give-the-closure-function-another-chance-31253e44cfa0)

[Make your code easier to read with Functional Programming](https://medium.freecodecamp.org/make-your-code-easier-to-read-with-functional-programming-94fb8cc69f9d)
