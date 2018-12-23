> * 原文地址：[css-animations-with-finite-state-machines](https://medium.com/@DavidKPiano/css-animations-with-finite-state-machines-7d596bb2914a)
> * 原文作者：[David Khourshid](https://medium.com/@DavidKPiano)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/.md](https://github.com/dawn-teams/translate/blob/master/articles/.md)
> * 译者：[也树](https://github.com/xdlrt)
> * 校对者：[灵沼](https://github.com/su-dan)，[照天](https://github.com/zzwzzhao)

# 有限状态机在 CSS 动画中的应用

随着用户界面中可能出现的不同状态和状态间转换的数目的不断增长，样式和动画的管理很快就变得复杂起来。即使是一个简单的登录表单也可以有很多不同的“用户状态流”，并且有许多边界情况需要考虑、

示例：[https://codepen.io/davidkpiano/pen/WKvPBP](https://codepen.io/davidkpiano/pen/WKvPBP)

状态机作为一种很好的编程范式，通过符合直觉和声明式的方式来管理用户界面状态间的过渡。我们已经在 [the Keyframers](https://keyframe.rs/) 中作为一种简化复杂动画和用户交互流的方式大量使用到了状态机。

所以，什么是状态机呢？听起来是很技术向的一个名词，对吗？它实际上可能比你想的要更简单和直观。（不要直接看 [Wikipedia](https://en.wikipedia.org/wiki/Finite-state_machine) 的介绍，相信我）

让我们从动画的角度来探索一下状态机。假设你在编写一个 loading 动画，在任意给定时间，它只能处于以下四个状态之一。

- idle (还未进入 loading 状态)
- loading
- failure
- success

这很容易理解，你的动画不可能既处于 loading 状态又处于 success 状态中。但是，这些状态如何在彼此之间过渡是需要重点考虑的。

![](https://img.alicdn.com/tfs/TB1EwA8wbvpK1RjSZPiXXbmwXXa-398-341.png)

每个箭头告诉我们一个状态是如何通过事件过渡到另一个状态的，并且有些状态是不可能互相转换的。（比如说你不可能从 success 状态到 failure 状态）。每一个箭头代表一个可以落地的动画，或者可以说是一个过渡。CSS 过渡是用来描述一个视觉状态在 CSS 中是如何转换至另一个视觉状态的。

换句话说，只要你在使用 CSS 过渡动画，你就已经在使用状态机的思想，但你可能没有意识到这一点。在不同状态间切换时你可能会使用添加或者移除类名的方式在实现：

```css
.button {
  /* ... button styles ... */
  transition: all 0.3s ease-in-out;
}
.button.is-loading {
  opacity: 0.5;
}
.button.is-loaded {
  opacity: 1;
  background-color: green;
}
```

这样可以正常工作，但是你必须确保 `is-loading` 类名被移除并且 `is-loaded` 类名被添加，因为更有可能出现的情况是类名变成 `.button.is-loading.is-loaded`。这样可能会导致不符合预期的副作用。

一个更好的方式是使用 [data- 属性](https://developer.mozilla.org/en-US/docs/Learn/HTML/Howto/Use_data_attributes)。它们只能展示一个值因此在这种场景下是有用的。当你的用户界面的某部分同时只能在一个状态下时（比如 `loading` 或 `success` 或 `error`），更新 `data-` 属性是更直接的：

```js
const elButton = document.querySelector('.button');
// set to loading
elButton.dataset.state = 'loading';
// set to success
elButton.dataset.state = 'success';
```

这种方式自然地限制在任意给定的时机里你的按钮只存在单个状态。你可以使用 `data-state` 属性来表示不同的按钮状态：

```css
.button[data-state="loading"] {
  opacity: 0.5;
}
.button[data-state="success"] {
  opacity: 1;
  background-color: green;
}
```

## 有限状态机
通常来说，有限状态机由五部分组成：

- 一系列有限的状态（如 idle，loading，success，failure）
- 一系列有限的事件（如 FETCH，ERROR，RESOLVE，RETRY）
- 一个初始状态（如 idle）
- 一系列过渡方式（如 idle 通过 FETCH 事件过渡至 laoding）
- 最终状态

它还有一些规范：

- 一个有限状态机同时只能在一种状态中
- 所有的过渡方式必须是确定的，意味着任意给定的状态和时间，必定会导致相同的预定义的下一个状态。没有意外。

现在，让我们看看我们如何在 HTML 和 CSS 中表示有限状态机。

## 上下文提供状态
有时，你需要根据当前应用（或某个父组件）的状态来决定其它组件的样式。只读的 `data-` 属性同样也可以在这种场景下使用，比如：`data-show`：

```css
.button[data-state="loading"] .text[data-show="loading"] {
  display: inline-block;
}
.button[data-state="loading"] .text[data-show]:not([data-show="loading"]) {
  display: none;
}
```

这是一种用来标记特定的 UI 元素仅仅应该在特定状态下展示的方式。然后再分别地在需要展示的元素上添加 `data-show="..."` 即可。如果你的组件在多个状态下都想显示，你可以像下面这样使用 [空格分割属性选择器](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors)。

```js
<button class="button" data-state="idle">
  <!-- 处于 idle 和 loading 状态时展示下载图标 -->
  <span class="icon" data-show="idle loading"></span>
  <span class="text" data-show="idle">Download</span>
  <span class="text" data-show="loading">Downloading...</span>
  <span class="text" data-show="success">Done!</span>
</button>
```

这是对应的 CSS：

```css
/* ... */
.button[data-state="loading"] [data-show~="loading"] {
  display: inline-block;
}
```

`data-state` 属性可以使用 JavaScript 进行改变：

```js
const elButton = document.querySelector('.button');
function setButtonState(state) {
  // set the data-state attribute on the button
  elButton.dataset.state = state;
}
setButtonState('loading');
// the button's data-state attribute is now "loading"
```

## 动态 data- 属性样式
随着应用的逐渐迭代，将所有的 `data-` 属性规则添加进来会让样式表不断膨胀并且难以维护，因为你在 JavaScript 文件和样式表中都需要维护这些不同的状态。同时因为每个类名和 `data-` 属性添加了不同的权重，也会让权重变得异常复杂。为了减少这些问题带来的影响，我们可以依照以下两条原则使用动态的 `data-active` 属性：

- 当匹配到 `data-show="..."` 属性时，元素应当具有 `data-active` 属性。
- 当没有匹配到 `data-hide="..."` 属性时，元素也应当具有 `data-active` 属性。

下面是在 JavaScrit 实际应用的例子：

```js
const elButton = document.querySelector('.button');
function setButtonState(state) {
  // change data-state attribute
  elButton.dataset.state = state;
  // remove any active data-attributes
  document.querySelectorAll(`[data-active]`).forEach(el => {
    delete el.dataset.active;
  });
  // add active data-attributes to proper elements
  document.querySelectorAll(`[data-show~="${state}"], [data-hide]:not([data-hide~="${state}"])`)
    .forEach(el => {
      el.dataset.active = true;
    });
}
// set button state to 'loading'
setButtonState('loading');
```

现在，我们上面的展示隐藏的样式可以被简化：

```css
.text[data-active] {
  display: inline-block;
}
.text:not([data-active]) {
  display: none;
}
```

## 声明可视化的状态
目前为止，一切都好。但是我们想防止改变状态的函数包含业务逻辑，我们可以创建一个状态机转换函数，包含当前状态和触发事件后转换到的下个状态和返回此状态的逻辑。通过使用 switch 代码块，可能像下面这样：

```js
// ...
function transitionButton(currentState, event) {
  switch (currentState) {
    case 'idle':
      switch (event) {
        case 'FETCH':
          return 'loading';
        default:
          return currentState;
      }
    case 'loading':
      switch (event) {
        case 'ERROR':
          return 'failure';
        case 'RESOLVE':
          return 'success';
        default:
          return currentState;
      }
    case 'failure':
      switch (event) {
        case 'RETRY':
          return 'loading';
        default:
          return currentState;
      }
    case 'success':
      default:
        return currentState;
  }
}
let currentState = 'idle';
function send(event) {
  currentState = transitionButton(currentState, event);
// change data-attributes
  setButtonState(currentState);
}
send('FETCH');
// => button state is now 'loading'
```

Switch 代码块基于事件对状态之间的转换进行编码，我们可以使用对象来简化它：

```js
// ...
const buttonMachine = {
  initial: 'idle',
  states: {
    idle: {
      on: {
        FETCH: 'loading'
      }
    },
    loading: {
      on: {
        ERROR: 'failure',
        RESOLVE: 'success'
      }
    },
    failure: {
      on: {
        RETRY: 'loading'
      }
    },
    success: {}
  }
};
let currentState = buttonMachine.initial;
function transitionButton(currentState, event) {
  return buttonMachine
    .states[currentState]
    .on[event] || currentState; // fallback to current state
}
// ...
// use the same send() function
```

不仅这种方式看起来比 Switch 代码块更干净，同时也是可以 JSON 序列化的。同时我们可以声明式地对状态和事件进行枚举。这就可以让我们将 `buttonMachine` 的代码复制粘贴至可视化工具中，比如[xviz](https://musing-rosalind-2ce8e7.netlify.com/?machine=%7B%22initial%22%3A%22idle%22%2C%22states%22%3A%7B%22idle%22%3A%7B%22on%22%3A%7B%22FETCH%22%3A%22loading%22%7D%7D%2C%22loading%22%3A%7B%22on%22%3A%7B%22ERROR%22%3A%22failure%22%2C%22RESOLVE%22%3A%22success%22%7D%7D%2C%22failure%22%3A%7B%22on%22%3A%7B%22RETRY%22%3A%22loading%22%7D%7D%2C%22success%22%3A%7B%7D%7D%7D)：

![](https://img.alicdn.com/tfs/TB1Bak7wirpK1RjSZFhXXXSdXXa-650-400.png)

## 总结
状态机的模式让应用中状态的处理更简便，并且让 CSS 中的样式过渡更简洁。总结一下，我们介绍了以下的 `data-` 属性：

- `data-state` 表示组件上有限的状态（如 `data-state="loading"`）
- `data-show` 决定了当其中一种状态匹配到 `data-state` 中的状态时元素需要增加 `data-active` 属性。（如 `data-state="idle loading"`）
- `data-hide` 决定了当其中一种状态匹配到 `data-state` 中的状态时元素需要移除 `data-active` 属性。（如 `data-state="success error"`）
- `data-active` 在当前元素 `data-show` 和 `data-hide` 属性匹配到 `data-state` 中的状态时，动态添加至以上元素。

还有以下的编程范式，使用以下属性，通过 JavaScript 对象定义一个状态机：

- `initial` - 状态机的初始状态（如 `idle`）
- `states` - 一个包含过渡方式和状态的 Map
- `on` - 标识了转换至下个状态的事件（如 `FETCH: "loading"`）
- 创建一个 `transition(currentState, event)` 函数，根据当前状态在状态机中查找下一个状态
- 创建一个 `send(event)` 函数，包含以下特点：
  1. 调用 `transition(...)` 方法来决定下一个状态
  2. 设置当前状态为获取到的下一个状态
  3. 执行相应的副作用（在这里是设置合适的 `data-` 属性）

我们同样可以通过调用 `setButtonState(...)` 人工测试想要的状态，这样就可以设置合适的 `data-` 属性和在特定状态下帮助我们开发和 debug 组件。这样可以减少为了到达合适的状态而不得不进行的一整套繁琐的流程。

## 更进一步
如果你想更深地探究状态机（和它延伸出来的概念，“状态表”），可以查阅下面的资源：

[xstate](https://github.com/davidkpiano/xstate) 是一个能够帮助更好地创建和使用状态机和状态图的库，支持嵌套/扁平的状态，行为等等。通过阅读这篇文章，你已经知道如何去使用它了：


```js
import { Machine } from 'xstate';
const buttonMachine = Machine({
  // the same buttonMachine object from earlier
});
let currentState = buttonMachine.initialState;
// => 'idle'
function send(event) {
  currentState = buttonMachine.transition(currentState, event);
// change data-attributes
  setButtonState(currentState);
}
send('FETCH');
// => button state is now 'loading'
```

[The World of Statecharts](https://statecharts.github.io/) 是由 [Erik Mogensen](https://twitter.com/mogsie) 整理的非常棒的资源，可以透彻地解释状态表和如何在用户界面上应用。
[Spectrum Statecharts community](https://spectrum.chat/statecharts) 有许多热心并且乐于助人，同时对 状态机和状态表很有兴趣的开发者。
[Learn State Machines](https://learnstatemachines.com/) 是一个通过构建 Instagram 的应用示例来教你学习状态表基础概念的课程。
[React-Automata](https://github.com/MicheleBertoli/react-automata/) 是 [Michele Bertoli](https://twitter.com/michelebertoli) 开发的使用 xstate 的库，它能够让你在 React 中使用状态表，有很多好处，比如自动生成测试快照。
如果你想了解更多前端用户界面中状态机的好处，可以查看我曾经在 Shop Talk Show 和 [Jon Bellah](https://jonbellah.com/articles/intro-state-machines) 对 [状态机](https://shoptalkshow.com/episodes/327-working-state-machines/) 的讨论。