> * 原文地址：[A tour of JavaScript timers on the web](https://nolanlawson.com/2018/09/01/a-tour-of-javascript-timers-on-the-web/)
> * 原文作者：[Nolan Lawson](https://nolanlawson.com/about/)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/A-tour-of-JavaScript-timers-on-the-web.md](https://github.com/dawn-teams/translate/blob/master/articles/A-tour-of-JavaScript-timers-on-the-web.md)
> * 译者：[灵沼](https://github.com/su-dan)
> * 校对者：[也树](https://github.com/xdlrt)、[靖鑫](https://github.com/luckyjing)、[眠云](https://github.com/JeromeYangtao)

---

# JavaScript 计时器之旅

突击小测验: JavaScript 各种定时器之间的区别是什么?

* Promises
* setTimeout
* setInterval
* setImmediate
* requestAnimationFrame
* requestIdleCallback

更具体地讲，如果你立刻对这些计时器进行排序，知道他们触发的顺序是什么吗？

如果不能，那你可能并不孤独。我已经写 JavaScript 和做编程许多年，曾经为一家浏览器厂商工作超过两年，直到最近，我才真正了解了这些计时器以及如何使用它们。

在这篇文章中，我将高度概述这些定时器工作方式以及使用它们的时机，并且会一起介绍 Loadash 很有用的 `debounce()` 和 `throttle()` 函数。

## Promises 和 microtasks

让我们先从这里开始，因为它大概是最简单的了。一个 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 回调也被称为 “microtask”，它以与 [MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) 回调相同的频率运行。如果 [queueMicrotask()](https://github.com/whatwg/html/commit/9d7cf125f960e6bb8d9b7c9456595f505f2e9d4b) 没有被规范排除并且进入浏览器领域，它也会有同样的结果。

[我已经写过很多关于 promise 的文章](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)。然而值得一提的是，Promise 有一个很容易被误解的地方是它们不会给浏览器留空闲的时间。那是因为处于异步回调队列中，但是并不意味着浏览器可以进行渲染，或者处理输入，或者做其他我们希望浏览器做的工作。

举个例子，假设我们有一个阻塞主线程1秒钟的函数：

```
function block() {
  var start = Date.now()
  while (Date.now() - start < 1000) { /* wheee */ }
}
```

如果我们用一组 microtasks 来调用这个函数：

```
for (var i = 0; i < 100; i++) {
  Promise.resolve().then(block)
}
```

这将会阻塞浏览器100秒。这与下面的操作一样：

```
for (var i = 0; i < 100; i++) {
  block()
}
```

任何同步任务执行完成后，microtasks 会立即执行。在这两者之间没有空闲做其他工作。所以，如果想把一个运行时间较长的任务分解为 microtasks，是不会如你所愿的。

## setTimeout 和 setInterval

它们是两兄弟：[setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout) 将任务排在 X 毫秒之后运行，而 [setInterval](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setInterval) 每隔 X 毫秒运行一次任务。

由于许多网站比如 confetti 到处乱用 `setTimeout(0)`。为了避免阻塞浏览器主线程，浏览器必须为 `setTimeout(/* ... */, 0)` 添加缓解措施。

这就是[crashmybrowser.com](https://www.crashmybrowser.com/) 中许多技巧不再起作用的原因，比如，在 `setTimeout` 中调用另外两个调用了更多 `setTimeout` 的 `setTimeout`等等。我在 [“Improving input responsiveness in Microsoft Edge”](https://blogs.windows.com/msedgedev/2017/06/01/input-responsiveness-event-loop-microsoft-edge/) 中从边缘部分介绍了其中一些缓解方法。

宽泛地说，`setTimeout(0)` 不是真正的在0毫秒之后执行。通常会在4毫秒内执行。有时会在16毫秒内执行（当 Edge 在充电时会这样）。有时候还会被限制到1秒钟（例子：[when running in a background tab](https://github.com/WICG/interventions/issues/5)）。这些是浏览器必须具备的能力，为了防止不受控制的网页占用 CPU 执行无用的 `setTimeout`。

所以说，`setTimeout` 确实允许浏览器在回调函数被调用之前做一些工作（和 microtasks 不同）。但是，如果你想在回调之前进行输入或是渲染操作，一般来说 `setTimeout` 不是最好的选择，因为它只是偶尔允许在回调之前做其他操作。 现在，有更好的浏览器 API 可以更直接地挂到浏览器渲染系统中。

## setImmediate
在继续介绍使用“更好的浏览器 API ”之前，这里有件事情值得一提。称为[setImmediate](https://developer.mozilla.org/en-US/docs/Web/API/Window/setImmediate) 是因为缺少一个更好的词语...很奇怪。如果在[caniuse.com](https://caniuse.com/#feat=setimmediate)上查找，你会发现只有 Microsoft 浏览器支持它。但是[它也在 node.js 中存在](https://nodejs.org/api/timers.html#timers_setimmediate_callback_args)。这到底是个什么东西？

`setImmediate` 最初是由微软提出来解决上述 `setTimeout` 的问题的。基本上，`setTimeout` 已经被滥用了，`setImmediate(0)` 实际上就是 `setImmediate(0)`，而不是一个被限制在4毫秒的东西。你可以查看 [some discussion about it from Jason Weber back in 2011](https://lists.w3.org/Archives/Public/public-web-perf/2011Jul/0012.html)。

不幸的是，`setImmediate` 只被 IE 和 Edge 采用了。仍在使用的部分原因是它在 IE 浏览器中作用很大，它允许输入事件比如键盘输入和鼠标点击“跳过队列”并在 `setImmediate` 回调之前执行，而 `setTimeout` 在 IE 中就没有这么大魔力。（Edge 最终解决了这个问题，详细说明在上一篇文章中）。

而且，`setImmediate` 存在于 Node 中这一事实意味着许多 “Node-polyfilled” 代码在浏览器中使用它，但是并不真正知道它在做什么。Node 中 `process.nextTick` 和 `setImmediate`的区别令人很困惑，甚至 [Node 的官方文档](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)都说名字应该交换。（然而为了这篇文章的初衷，我会把重心放在浏览器而不是 Node 上，因为我不是一个 Node 专家）。

最低原则：如果你知道你要做什么并且尝试优化 IE 的输入性能，就使用 `setImmediate`。如果不是，就不用麻烦了。（或者只在 Node 中使用）

## requestAnimationFrame

现在，我们有一个最重要的 `setTimeout` 替代品，一个真正挂在浏览器渲染循环中的定时器。顺便说一句，如果你不知道浏览器事件循环机制，我强烈推荐 [Jake Archibald 的这个演讲](https://youtu.be/cCOL7MC4Pl0)。

`requestAnimationFrame` 基本上是这样工作的：它虽然和 `setTimeout` 有点像，但是它会在浏览器下次重绘时调用，而非等待一些无法预测的时间（4毫秒，16毫秒，1秒等）。现在，像 Jake 在他的演讲中指出的一样，这里有一个小问题，在 Safari 、IE 和 Edge 18以下版本的浏览器中，他在样式/布局计算之后执行。但是让我们忽略它，因为这不是一个很重要的细节。

我认为 `requestAnimationFrame` 的使用方式是这样的：无论什么时候，只要我知道我将要修改浏览器的样式或布局——举个例子，改变 CSS 属性或启动一个动画——我就会把它放在 `requestAnimationFrame`（这里缩写为 `rAF`）。这样确保了几件事情：

1. 我不太可能打乱布局，因为所有的DOM的变化都在排队和协调。
2. 我的代码会自然地去适应浏览器的性能特点。举个例子，如果这里有一个配置较低的设备正在试图渲染一些DOM元素，rAF 会自然地从通常的16.7毫秒（在60赫兹的屏幕上）时间间隔慢下来，因此，它不会像运行了大量 setTimeout 或 setInterval 的一样让设备崩溃。

这就是为什么不依赖 CSS 转换或 keyframes 的动画库的原因，比如  [GreenSock](https://greensock.com/) or [React Motion](https://github.com/chenglou/react-motion)，通常会在 rAF 回调中更改。如果一个元素在 `opacity: 0` 和 `opacity: 1` 之间进行动画转换，那么排队等待十亿次回调来对每个可能的中间状态进行处理是没有意义的，包括 `opacity: 0.0000001` 和 `opacity: 0.9999999`。

相反，你最好只使用 `rAF`，让浏览器告诉你在给定的时间段能绘制多少帧，并为特定帧进行计算。这样，较慢的设备自然就会以慢的帧速率结束，较快的设备以快的帧速率结束，如果使用类似 `setTimeout` 这种独立于浏览器绘制速度的 API，上述情况都是不可能出现的。

## requestIdleCallback
`rAF` 可能是 toolkit 中最有用的定时器，但是[`requestIdleCallback`](https://developer.mozilla.org/en-US/docs/Web/API/Background_Tasks_API) 也同样值得一提。[浏览器支持不是很好](https://caniuse.com/#feat=requestidlecallback)，但是有一个 [工作很不错的polyfill](https://www.npmjs.com/package/requestidlecallback)（底层使用了 rAF）。

在很多情况下 `rAF` 类似于 `requestIdleCallback`。（从这开始缩写为 `rIC`）

像 `rAF` 一样，`rIC` 会自然地适应浏览器的性能特征：如果设备过载，`rIC` 可能会延迟。`rIC` 的不同之处在于它会在浏览器空闲状态触发，比如，当浏览器确定它没有其他任务，microtasks 或输入事件要处理的时候，你就自由地做想做的工作。它也会给你一个 "deadline" 来追踪使用的预算值，这是个很不错的特性。

Dan Abramov 在[2018 冰岛 JSConf 上有一个精彩讲话](https://youtu.be/v6iR3Zk4oDY)，在谈话中他展示了如何使用 `rIC`。在谈话中，有一个 webapp 在用户打字的每一次键盘输入的时候会调用 `rIC`，然后它会更新回调中的渲染状态。这很棒，因为一个快速打字的用户会导致 `keydown`/`keyup` 事件非常快地触发，但是你并不希望为每个按键都重新渲染页面。

另一个很好的例子是 Twitter 或 MastoDon 上的“剩余字符计数”指示器。在 [Pinafore](https://pinafore.social/) 中，我使用 `rIC` 进行操作，因为我不真正关心指示符是否针对我每一次输入都重新渲染。如果我快速打字，最好优先考虑输入相应，这样才不会失去流畅感。

![](https://nolanwlawson.files.wordpress.com/2018/09/screenshot-2018-09-01-13-59-21.png)

在 Pinafore 中，输入框下面的小提示条和“剩余字符”提示会随着输入而更新。

我注意到 `rIC` 在 Chrome 中有点瑕疵。在Firefox 中，每当我直觉的认为浏览器是空闲并准备运行一些代码的时候，它就会运行。（在 pollyfill 中也是这样。）不过在 Chrome 的安卓移动模式中，我注意到，每当我触摸滚动的时候，它就会将 `rIC` 延迟几秒钟，即使在我刚触摸完屏幕，浏览器也什么都不会做。（我怀疑我看到的问题是[这个](https://bugs.chromium.org/p/chromium/issues/detail?id=811451).）

**更新**：来自 Chrome 团队的 Alex Russell [通知我](https://toot.cafe/@slightlyoff/100655566963584982)这是一个已知 bug，应该很快就修复！


无论如何，`rIC` 是另一个很好地工具。我倾向于这样想：使用 `rAF` 来进行关键的渲染工作，使用 `rIC` 来进行非关键的渲染工作。

## debounce 和 throttle

这里有两个非浏览器内置的方法，但是它们很有用并值得了解。如果你不熟悉它们，这里有一个[很棒的 CSS 技巧攻略](https://css-tricks.com/debouncing-throttling-explained-examples/)

[`debounce`](https://lodash.com/docs/4.17.10#debounce) 的标准用法是在 [`resize`](https://developer.mozilla.org/en-US/docs/Web/Events/resize)回调中。当用户调整浏览器窗口大小的时候，没必要在每个 `resize` 回调中更新布局，因为触发太频繁了。相反，你可以 `debounce` 几百毫秒，这会保证回调在用户在处理完窗口大小后触发。

[`throttle`](https://lodash.com/docs/4.17.10#throttle)，另一方面，是我使用得更多的方法。举个例子，[`scroll`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onscroll) 事件是一个很棒的使用示例。再说一遍，对于每个 `scroll` 回调都更新一遍视图状态是没有意义的，因为触发频率太高了（频率在不同浏览器，不同输入法之间是不同的）。使用 `throttle` 可以规范这个行为，并确保它只在每 X 毫秒后触发。你可以调整 Loadash 的 `throttle`（或者 `debounce`）方法启动延迟的时机，在结束的时候或者不启动。

相反，我不会在滚动场景中使用 `debounce`，因为我不希望 UI 仅在用户明确停止滚动后才更新。因为这可能会让用户苦恼和困惑，并且试图滚动继续更新 UI 状态（例如在无限滚动列表中）。

我在各种用户输入和一些定时安排的任务中会使用 `throttle`，比如 IndexedDB 清理。也许有一天它会内置到浏览器中。

## 结论

这是我对浏览器中各种定时器的快速了解以及如何使用它们。我可能漏掉了一些，因为这里有一些特殊的特性（[`postMessage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) 或 [`lifecycle events`](https://developers.google.com/web/updates/2018/07/page-lifecycle-api)，还有其他的吗？）。但希望这至少能对我如何看待 JavaScript 中定时器有一个很好地概述。