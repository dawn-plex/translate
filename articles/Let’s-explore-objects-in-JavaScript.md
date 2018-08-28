> * 原文地址：[Let’s explore objects in JavaScript](https://medium.freecodecamp.org/lets-explore-objects-in-javascript-4a4ad76af798)
> * 原文作者：[Cristi Salcescu](https://medium.freecodecamp.org/@cristisalcescu)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/Let’s-explore-objects-in-JavaScript.md](https://github.com/dawn-teams/translate/blob/master/articles/Let’s-explore-objects-in-JavaScript.md)
> * 译者：灵沼
> * 校对者：

---
# Let’s explore objects in JavaScript

Objects are dynamic collections of properties, with a “hidden” property to the object’s prototype.

A property has a key and a value.

### Property key
The property key is a unique string.

There are two ways to access properties: dot notation and bracket notation. When the dot notation is used, the property key must be a valid identifier.

```
let obj = {
  message : "A message"
}
obj.message //"A message"
obj["message"] //"A message"
```

Accessing a property that doesn’t exist will not throw an error, but will return `undefined` .

```
obj.otherProperty //undefined
```
When the bracket notation is used, the property key does not have to be a valid identifier — it can have any value.

```
let french = {};
french["thank you very much"] = "merci beaucoup";

french["thank you very much"]; //"merci beaucoup"
```
When a non string is used as the property key, it will be converted to a string (via the `toString()` method, when available).

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

In the previous example, the object `number1` is used as a property key. It is then converted to a string, and the result of the conversion ("1" ) is used as the property key.

### Property value
The property value can be a primitive, object, or function.

### Object as value
Objects can be nested inside other objects. [See the next example](https://jsfiddle.net/cristi_salcescu/m0a65e2g/):

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

This way, we can create a namespace:

```
let app = {};
app.authorService = { getAuthors : function() {} };
app.bookService = { getBooks : function() {} };
```

### Function as value
When a function is used as a value for a property, it usually becomes a method. Inside methods, the `this` keyword is used to refer to the current object.

`this`, however, can have many values depending on how the function was invoked. For more on `this` losing context, take a look at [What to do when “this” loses context](https://medium.freecodecamp.org/what-to-do-when-this-loses-context-f09664af076f).

### Dynamic nature
Objects are dynamic by nature. Properties can be added or deleted at any time.

```
let obj = {};
obj.message = "This is a message"; //add new property
obj.otherMessage = "A new message"; //add new property
delete obj.otherMessage; //delete property
```

### Map
We can think of an object as a map. The keys in the map are the names of the object’s properties.

Accessing a key doesn’t require you to scan through all the properties. It’s an O(1) access time.

### Prototype
Objects have a “hidden” link `__proto__` to a prototype object, from which they inherit properties.

For example, objects created with an object literal have a link to the 
`Object.prototype` :

```
var obj = {};
obj.__proto__ === Object.prototype; //true
```

### Prototype chain
The prototype object has a prototype of its own. When a property is accessed and the object does not contain it, JavaScript will look down the prototype objects until it either finds the requested property, or until it reaches `null`.

### Read only
The prototype link is used only for reading values. When a change is made, it is made only on the current object not on its prototype; not even when a property with the same name is already defined on the prototype.

### Empty Object
As we have seen, the empty object `{}` is not really empty as it contains a link to the `Object.prototype` . In order to create a true empty object, we can use `Object.create(null)`. It will create an object with no prototype. Usually this is used to create a map.

### Primitives and Wrapper Objects
JavaScript treats primitives like objects in the sense that it allows access to properties. Primitives, of course, are not objects.

```
(1.23).toFixed(1); //"1.2"
"text".toUpperCase(); //"TEXT"
true.toString(); //"true"
```

In order to allow access to properties on primitives, JavaScript creates an wrapper object and then destroys it. The process of creating and destroying wrapper objects is optimized by the JavaScript engine.

Numbers, strings, and booleans have object equivalent wrappers. These are the `Number`, `String`, and `Boolean` functions.

`null` and `undefined` primitives have no corresponding wrapper objects and provide no methods.

### Built-in Prototypes
Numbers inherit from `Number.prototype`, which inherits from `Object.prototype` .

```
var no = 1;
no.__proto__ === Number.prototype; //true
no.__proto__.__proto__ === Object.prototype; //true
```
Strings inherit from `String.prototype`. Booleans inherit from `Boolean.prototype`. Arrays inherit from `Array.prototype`.

Functions are objects and inherit from `Function.prototype`. They have methods like `bind()`, `apply()` and `call()`.

All objects, functions, and primitives (except `null` and `undefined`) inherit properties from `Object.prototype`. All of them have the `toString()` method, for example.

### Augmenting built-in objects with polyfills
JavaScript makes it easy to augment the built-in objects with new functionalities.

A polyfill is piece of code that implements a feature in browsers that don’t support that feature.

### Utilities
For example, the [polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign#Polyfill) for `Object.assign()` adds a new function on `Object if it’s not available.

The same happens with the `Array.from()` [polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from#Polyfill) which adds a new function on `Array` if it’s not available.

### Prototypes
New methods can be added on prototypes.

For example, the `String.prototype.trim()` [polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/Trim#Polyfill) makes the `trim()` method available on all strings.

```
let text = "   A text  ";
text.trim(); //"A text"
```

The [polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find#Polyfill) for `Array.prototype.find()` makes the `find()` methods available on all arrays. `Array.prototype.findIndex() `[polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/findIndex#polyfill) does the same.

```
let arr = ["A", "B", "C", "D", "E"];
arr.indexOf("C"); //2
```

### Single Inheritance
`Object.create()` creates a new object with a specified prototype object. It is used for single inheritance. [Consider the next example](https://jsfiddle.net/cristi_salcescu/zu79cebu/):

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
### Multiple Inheritance
`Object.assign()` copies properties from one or more objects to a target object. It can be used for multiple inheritance. [Check out the next example](https://jsfiddle.net/cristi_salcescu/ghqsb9a3/):

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

### Immutable Objects
`Object.freeze()` freezes an object. Properties can’t be added, deleted, or changed. The object becomes immutable.

```
"use strict";
let book = Object.freeze({
  title : "Functional-Light JavaScript",
  author : "Kyle Simpson"
});
book.title = "Other title";//Cannot assign to read only property 'title'
```
`Object.freeze()` does a shallow freeze. The nested objects can be changed. For deep freeze, recursively freeze each property of type object.

### Clone
`Object.assign()` can be used to clone objects.

```
let book = Object.freeze({
  title : "JavaScript Allongé",
  author : "Reginald Braithwaite"
});
let clone = Object.assign({}, book);
```

`Object.assign()` does a shallow copy, not a deep copy. It copies the top-level properties. The nested objects are shared between the original and the copy.

### Object literal
Object literal offers a simple, elegant way of creating objects.

```
let timer = {
  fn : null,
  start : function(callback) { this.fn = callback; },
  stop : function() {},
}
```
However, this syntax has a few drawbacks. All properties are public, methods can be redefined, and we can’t use the same methods for new instances.

```
timer.fn;//null 
timer.start = function() { console.log("New implementation"); }
```

### Object.create()

`Object.create()` together with `Object.freeze()` can solve the last two problems.

First, I’ll create a frozen prototype `timerPrototype` with all the methods, then I’ll create and object that inherits from it.

```
let timerPrototype = Object.freeze({
  start : function() {},
  stop : function() {}
});
let timer = Object.create(timerPrototype);
timer.__proto__ === timerPrototype; //true
```

When the prototype is frozen, the objects that inherit from it won’t be able to change the properties defined within. Now, the `start()` and `stop()` methods can’t be redefined.

```
"use strict";
timer.start = function() { console.log("New implementation"); } //Cannot assign to read only property 'start' of object
```

`Object.create(timerPrototype)` can be used to build more objects with the same prototype.

### Function Constructor
Initially, the language proposed the function constructor as a sugar syntax for all this. [Look at this code](https://jsfiddle.net/cristi_salcescu/az35x2qs/):

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

All functions defined with the `function` keyword can be used as function constructors. A function constructor is called with `new`. The new object will have the prototype set to FunctionConstructor.prototype.

```
let timer = new Timer();
timer.__proto__ === Timer.prototype;
```

Again, we need to freeze the prototype to prevent method redefinition.

```
Timer.prototype = Object.freeze({
  start : function() {},
  stop : function() {}
});
```

### new
When `new Timer()` is executed, it does the same as the `newTimer()` function:

```
function newTimer(){
  let newObj = Object.create(Timer.prototype);
  let returnObj = Timer.call(newObj, arguments);
  if(returnObj) return returnObj;
    
  return newObj;
}
```
A new object is created with `Timer.prototype`as its prototype. Then the `Timer` function is run and sets the fields on the new object.

### Class
EcmaScript2015 comes with a much better sugar syntax for all this. [Check out this example](https://jsfiddle.net/cristi_salcescu/aLg8t632/):

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

The object built with class has the prototype set to ClassName.prototype. The `new` operator must be used when creating an object with a class.

```
let timer= new Timer();
timer.__proto__ === Timer.prototype;
```

The class syntax doesn’t freeze the prototype, so we need to do that after.

```
Object.freeze(Timer.prototype);
```

### Prototype-based inheritance
In JavaScript, objects inherit from objects.

Both function constructor and class are a sugar syntax for creating a prototype object with all methods. Then it creates a new object that inherits from that prototype and sets the data fields on the new object.

The prototype inheritance has the benefit of memory conservation. The prototype is created once and used by all instances.

#### No Encapsulation
The prototype-based inheritance patterns have no privacy. All object’s properties are public.

`Object.keys()` returns an array with all property keys. It can be used to iterate over all object properties.

```
function logProperty(name){
  console.log(name); //property name
  console.log(obj[name]); //property value
}
Object.keys(obj).forEach(logProperty);
```

The pretended privacy pattern involves marking private properties with`_ `, so that others will avoid using them:

```
class Timer{
  constructor(callback){
    this._fn = callback;
    this._timerId = 0;
  }
}
```

### Factory Functions
JavaScript offers a new way of creating encapsulated objects using factory functions.

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

The `fn` variable is private. Only the `start()` and `stop()` methods are public. The `start()` and `stop()` methods can’t be modified from outside the object. There is no use of `this`, so there are no `this` losing context problems.

The object literal is still used on the return object, but this time it contains only functions. Even more, these functions are closures sharing the same private state. `Object.freeze()` is used to freeze the public API.

For a full implementation of the Timer object, take a look at [Here are some practical JavaScript objects that have encapsulation](https://medium.freecodecamp.org/here-are-some-practical-javascript-objects-that-have-encapsulation-fc4c1a79c655).

### Conclusion
JavaScript treats primitives, objects, and functions like objects.

Objects are dynamic in nature and can be used as maps.

Objects inherit from other objects. Constructor functions and class are sugar syntax for creating objects that inherit from other prototype objects.

`Object.create()` can be used for single inheritance and `Object.assign()` for multiple inheritance.

Factory functions can build encapsulated objects.

For more on the JavaScript functional side, take a look at:

[Discover the power of first class functions](https://medium.freecodecamp.org/discover-the-power-of-first-class-functions-fd0d7b599b69)

[How point-free composition will make you a better functional programmer](https://medium.freecodecamp.org/how-point-free-composition-will-make-you-a-better-functional-programmer-33dcb910303a)

[Here are a few function decorators you can write from scratch](https://medium.freecodecamp.org/here-are-a-few-function-decorators-you-can-write-from-scratch-488549fe8f86)

[Why you should give the Closure function another chance](https://medium.freecodecamp.org/why-you-should-give-the-closure-function-another-chance-31253e44cfa0)

[Make your code easier to read with Functional Programming](https://medium.freecodecamp.org/make-your-code-easier-to-read-with-functional-programming-94fb8cc69f9d)





