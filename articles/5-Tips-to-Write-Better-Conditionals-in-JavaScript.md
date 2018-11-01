

>- 原文地址：[5 Tips to Write Better Conditionals in JavaScript](https://scotch.io/bar-talk/5-tips-to-write-better-conditionals-in-javascript#toc-3-use-default-function-parameters-and-destructuring)
>- 原文作者：[ecelyn Yeen](https://scotch.io/@jecelyn)[(@jecelynyeen)](https://twitter.com/jecelynyeen)
>- 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
>- 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/5-Tips-to-Write-Better-Conditionals-in-JavaScript.md](https://github.com/dawn-teams/translate/blob/master/articles/5-Tips-to-Write-Better-Conditionals-in-JavaScript.md)
>- 译者：[眠云(杨涛)](https://github.com/JeromeYangtao)
>- 校对者：

---

# 5条在JavaScript中写出更好的条件语句的建议

在用JavaScript工作时，我们会处理许多条件语句，这里有5条让你写出更好/干净的条件语句的建议。



1.多重判断时使用Array.includes

2.更少的嵌套，尽早Return

3.使用默认参数和解构

4.倾向于Map/Object Literal而不是Switch语句

5.对 所有/部分 判断使用Array.every & Array.some

6.总结



### 1.多重判断时使用Array.includes

让我们看一下下面这个例子:



```js
// condition
function test(fruit) {
  if (fruit == 'apple' || fruit == 'strawberry') {
    console.log('red');
  }
}
```



第一眼，上面这个例子看起来没问题。如果我们有更多名字叫`cherry`和`cranberries`的red fruits呢？我们准备用更多的`||`拓展语句吗？

我们可以用 `Array.includes` [(Array.includes)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes)重写条件语句。



```js
function test(fruit) {
  // extract conditions to array
  const redFruits = ['apple', 'strawberry', 'cherry', 'cranberries'];

  if (redFruits.includes(fruit)) {
    console.log('red');
  }
}
```



我们把`red fruits`(判断条件)提取到一个数组。这样一来，代码看起来更整洁。



### 2.更少的嵌套，尽早Return



让我们扩大上一个例子让它包含两个条件。



- 如果没有传入fruit，抛出错误
- 接受quantity参数，并且在quantity大于10时打印出来



```js
function test(fruit, quantity) {
  const redFruits = ['apple', 'strawberry', 'cherry', 'cranberries'];

  // 条件 1: fruit 必须有值
  if (fruit) {
    // 条件 2: 必须是red的
    if (redFruits.includes(fruit)) {
      console.log('red');

      // 条件 3: quantity大于10
      if (quantity > 10) {
        console.log('big quantity');
      }
    }
  } else {
    throw new Error('No fruit!');
  }
}

// 测试结果
test(null); // error: No fruits
test('apple'); // print: red
test('apple', 20); // print: red, big quantity
```



在上面的代码, 我们有:

- 1个 if/else 语句筛选出无效的语句
- 3层if嵌套语句 (条件 1, 2 & 3)

我个人遵循的规则一般是在发现无效条件时，**尽早Return**。



```js
/_ 当发现无效语句时，尽早Return _/

function test(fruit, quantity) {
  const redFruits = ['apple', 'strawberry', 'cherry', 'cranberries'];

  // 条件 1: throw error early
  if (!fruit) throw new Error('No fruit!');

  // 条件 2: must be red
  if (redFruits.includes(fruit)) {
    console.log('red');

    // 条件 3: must be big quantity
    if (quantity > 10) {
      console.log('big quantity');
    }
  }
}

```



这样一来，我们少了一层嵌套语句。这种编码风格非常好，尤其是当你有很长的if语句的时候(想象你需要滚动到最底层才知道还有else语句，这并不酷)

我们可以通过 倒置判断条件 & 尽早return 进一步减少if嵌套。看下面我们是怎么处理判断 条件2 的: 



```js
/_ 当发现无效语句时，尽早Return _/

function test(fruit, quantity) {
  const redFruits = ['apple', 'strawberry', 'cherry', 'cranberries'];

  if (!fruit) throw new Error('No fruit!'); // 条件 1: throw error early
  if (!redFruits.includes(fruit)) return; // 条件 2: stop when fruit is not red

  console.log('red');

  // 条件 3: must be big quantity
  if (quantity > 10) {
    console.log('big quantity');
  }
}
```



通过倒置判断条件2，我们的代码避免了嵌套语句。这个技巧在我们有很长的逻辑时是非常有用的，而且我们希望在条件不满足时停止进一步处理。

而且这么做并不困难。问问自己，这个版本(没有嵌套)是不是比之前的(两层条件嵌套)更好，可读性更高？

但对于我，我会保留先前的版本(包含两层嵌套)。这是因为:

- 代码比较短且直接，包含if嵌套的更清晰
- 倒置判断条件可能加大思考的过程(增加认知载荷)

因此，应当**尽力减少嵌套和尽早return，但不要过度**。这是关于这个话题的一篇文章&StackOverflow的讨论，如果你感兴趣的话。

- [Avoid Else, Return Early](http://blog.timoxley.com/post/47041269194/avoid-else-return-early) by Tim Oxley
- [StackOverflow discussion](https://softwareengineering.stackexchange.com/questions/18454/should-i-return-from-a-function-early-or-use-an-if-statement) on if/else coding style



### 3.使用默认参数和解构

我猜下面的代码你可能会熟悉，在JavaScript中我们总是需要检查 `null` / `undefined`的值和指定默认值:



```js
function test(fruit, quantity) {
  if (!fruit) return;
  const q = quantity || 1; // if quantity not provided, default to one

  console.log(`We have ${q} ${fruit}!`);
}

//test results
test('banana'); // We have 1 banana!
test('apple', 2); // We have 2 apple!
```



实际上，我们可以通过声明 默认函数参数 来消除变量q。

```js
function test(fruit, quantity = 1) { // if quantity not provided, default to one
  if (!fruit) return;
  console.log(`We have ${quantity} ${fruit}!`);
}

//test results
test('banana'); // We have 1 banana!
test('apple', 2); // We have 2 apple!
```



这更加直观，不是吗？声明每个有自己的[默认参数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)的参数.

例如，我们也能给``fruit``分配默认值:`function test(fruit = 'unknown', quantity = 1)`。

如果``fruit``是一个object会怎么样？我们能分配一个默认参数吗？



```js
function test(fruit) { 
  // printing fruit name if value provided
  if (fruit && fruit.name)  {
    console.log (fruit.name);
  } else {
    console.log('unknown');
  }
}

//test results
test(undefined); // unknown
test({ }); // unknown
test({ name: 'apple', color: 'red' }); // apple
```



看上面这个例子，我们想打印fruit name，如果它可访问的话。否则我们将打印unknown。我们可以通过 默认参数&解构 避免判断条件``fruit && fruit.name``



```js
// destructing - get name property only
// assign default empty object {}
function test({name} = {}) {
  console.log (name || 'unknown');
}

//test results
test(undefined); // unknown
test({ }); // unknown
test({ name: 'apple', color: 'red' }); // apple
```



由于我们只需要``name``属性，我们可以用`{name}`解构出参数，然后我们就能使用变量``name``代替`fruit.name`。

我们也能声明空对象`{}`作为默认值。如果我们不这么做，你将得到一个错误当执行到`test(undefined)` - `Cannot destructure property name of 'undefined' or 'null'`时。因为在undefined中没有``name``属性。

如果你不介意使用第三方库，这有一些方式减少null的检查:

- 使用 [Lodash get](https://lodash.com/docs/4.17.10#get)函数
- 使用Facebook开源的[idx](https://github.com/facebookincubator/idx)库(with Babeljs)

这是一个使用Lodash的例子:



```js
// Include lodash library, you will get _
function test(fruit) {
  console.log(__.get(fruit, 'name', 'unknown'); // get property name, if not available, assign default value 'unknown'
}

//test results
test(undefined); // unknown
test({ }); // unknown
test({ name: 'apple', color: 'red' }); // apple
```



你可以在[这](http://jsbin.com/bopovajiye/edit?js,console)运行demo代码。除此之外，如果你是函数式编程的粉丝，你可能选择使用 [Lodash fp](https://github.com/lodash/lodash/wiki/FP-Guide)，Lodash的函数式版本(方法变更为``get``或者``getOr``)。



### 4.倾向于Map/Object Literal而不是Switch语句

让我们看下面这个例子，我们想根据color打印出fruits:



```js
function test(color) {
  // use switch case to find fruits in color
  switch (color) {
    case 'red':
      return ['apple', 'strawberry'];
    case 'yellow':
      return ['banana', 'pineapple'];
    case 'purple':
      return ['grape', 'plum'];
    default:
      return [];
  }
}

//test results
test(null); // []
test('yellow'); // ['banana', 'pineapple']
```



上面的代码看起来没有错误，但是我找到了一些累赘。用object literal实现相同的结果，语法看起来更简洁:



```js
// use object literal to find fruits in color
  const fruitColor = {
    red: ['apple', 'strawberry'],
    yellow: ['banana', 'pineapple'],
    purple: ['grape', 'plum']
  };

function test(color) {
  return fruitColor[color] || [];
}
```



或者你可能使用 [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)实现相同的结果:



```js
// use Map to find fruits in color
  const fruitColor = new Map()
    .set('red', ['apple', 'strawberry'])
    .set('yellow', ['banana', 'pineapple'])
    .set('purple', ['grape', 'plum']);

function test(color) {
  return fruitColor.get(color) || [];
}
```



 [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)是一种在ES2015之后实现的对象类型，允许你存储key value值。

我们是否应当禁止switch语句的使用？不要限制你自己。个人来说，我尽可能的使用object literal，但我并不严格遵守它，而是使用对当前场景有意义的。

Todd Motto有一篇关于switch语句vs object literal更深入的文章，你可以在[这](https://toddmotto.com/deprecating-the-switch-statement-for-object-literals/)阅读

### TL;DR; 重构语法

在上面的例子，我们能够用`Array.filter` 重构我们的代码，实现相同的效果。

```js
 const fruits = [
    { name: 'apple', color: 'red' }, 
    { name: 'strawberry', color: 'red' }, 
    { name: 'banana', color: 'yellow' }, 
    { name: 'pineapple', color: 'yellow' }, 
    { name: 'grape', color: 'purple' }, 
    { name: 'plum', color: 'purple' }
];

function test(color) {
  // use Array filter to find fruits in color

  return fruits.filter(f => f.color == color);
}
```



有着不止一种方法实现相同的结果，我们已经展示了4种了。代码真有趣!



### 5.对 所有/部分 判断使用Array.every & Array.some

这最后一个建议更多是关于利用新的(可能也不新)JavaScript Array方法来减少代码行数。看下面的代码，我们想要检查是否所有水果都是红色:



```js
const fruits = [
    { name: 'apple', color: 'red' },
    { name: 'banana', color: 'yellow' },
    { name: 'grape', color: 'purple' }
  ];

function test() {
  let isAllRed = true;

  // condition: all fruits must be red
  for (let f of fruits) {
    if (!isAllRed) break;
    isAllRed = (f.color == 'red');
  }

  console.log(isAllRed); // false
}
```



代码那么长！我们可以通过 `Array.every`减少代码行数:



```js
const fruits = [
    { name: 'apple', color: 'red' },
    { name: 'banana', color: 'yellow' },
    { name: 'grape', color: 'purple' }
  ];

function test() {
  // condition: short way, all fruits must be red
  const isAllRed = fruits.every(f => f.color == 'red');

  console.log(isAllRed); // false
}
```



现在更简洁了，不是吗？相同的方式，如果我们想测试是否有水果是红色的，我们可以使用 `Array.some` 一行代码实现。



```js
const fruits = [
    { name: 'apple', color: 'red' },
    { name: 'banana', color: 'yellow' },
    { name: 'grape', color: 'purple' }
];

function test() {
  // condition: if any fruit is red
  const isAnyRed = fruits.some(f => f.color == 'red');

  console.log(isAnyRed); // true
}
```



### 6.总结

让我们一起生产更多可读性高的代码。我希望你能从这篇文章学到东西。

这就是所有的内容。编码快乐！





