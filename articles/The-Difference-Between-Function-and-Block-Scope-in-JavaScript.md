

>- 原文地址：[The Difference Between Function and Block Scope in JavaScript](https://medium.com/@josephcardillo/the-difference-between-function-and-block-scope-in-javascript-4296b2322abe)
>- 原文作者：[Joseph Cardillo](https://medium.com/@josephcardillo)
>- 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
>- 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/How-to-build-an-rpc-based-api-with-nodejs.md](https://github.com/dawn-teams/translate/blob/master/articles/The-Difference-Between-Function-and-Block-Scope-in-JavaScript.md)
>- 译者：[眠云](https://github.com/JeromeYangtao)
>- 校对者：

---

### JS函数作用域和块级作用域的区别

回到基础的var，let，const变量



![](https://s1.ax1x.com/2018/09/22/iunxXD.jpg)



最近我向一些人提起我想回顾JS基础知识时，他们推荐了Wes Bos的 “ES6 For Everyone” 课程。Wes是一个位好老师和擅长提炼复杂主题的沟通者。



我今天开始研究它来巩固JS基础知识，开始深入了解JS之旅。



### var，let和const变量的区别

在JS中有三种声明变量的方式



```js
var width = 100;
let height = 200;
const key = 'abc123';
```



### var

var的独特之处在于？它能被重复声明和修改。例如:



```js
// 定义变量:
var width = 100;

// 调用变量:
width;
// 它返回:
100
// 重新分配变量并调用:
width = 200;

width;
// 返回:
200
```

var变量是''函数作用域''这意味着什么？这意味着它们只能在它们被创建的函数中使用，如果不是在函数中被创建，它们会变成''全局作用域''



如果var是定义在函数内部，并且我后来尝试在函数外部调用它，它不会生效的。



例如:



```js
function setWidth(){
    var width = 100;
    console.log(width);
}
width;
// 返回:
Uncaught ReferenceError: width is not defined
```



重复一遍，因为var是在函数内定义的，它在函数外部不可用。



### 理解作用域



```js
Example: 
var age = 100;
if (age > 12){
  var dogYears = age * 7;
  console.log(`You are ${dogYears} dog years old!`);
}
```



在上面的例子，控制台里会返回:



```js
100
You are 700 dog years old!
```



但如果我在控制台里调用dogYears会返回:



```js
dogYears;
// returns:
700
```



这发生了什么事？为什么dogYears会在全局作用域？



因为var是全局作用域—并且上面这个例子中var dogYears不是定义在函数内的—而是定义在window，或者说全局作用域



换句话说，var没有被限制在大括号内。它是定义范围的函数。



### 使用let和const的好处

使用let和const的好处是？它们不是函数作用域而是块级作用域



什么是块？块是一组打开和关闭的大括号。



所以在上面的例子中，如果我们把var dogYears改成let dogYears，然后在控制台调用dogYears，它返回“Uncaught ReferenceError: dogYears is not defined”



```js
var age = 100;
if (age > 12){
  let dogYears = age * 7;
  console.log(`You are ${dogYears} dog years old!`);
 }
```



在上面例子中，我们也能用const替换let



### 总结



```
var是函数作用域
let和const是块级作用域

函数作用域是在函数内
块级作用域是在大括号内

```





