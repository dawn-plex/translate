> * 原文地址：[A tour of JavaScript timers on the web](https://scotch.io/@alloys/how-to-build-an-rpc-based-api-with-nodejs)
> * 原文作者：[Nolan Lawson](https://github.com/nolanlawson)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：
> * 译者：[灵沼](https://github.com/su-dan)
> * 校对者：，
---

# A tour of JavaScript timers on the web

Pop quiz: what is the difference between these JavaScript timers?

* Promises
* setTimeout
* setInterval
* setImmediate
* requestAnimationFrame
* requestIdleCallback

More specifically, if you queue up all of these timers at once, do you have any idea which order they’ll fire in?

If not, you’re probably not alone. I’ve been doing JavaScript and web programming for years, I’ve worked for a browser vendor for two of those years, and it’s only recently that I really came to understand all these timers and how they play together.

In this post, I’m going to give a high-level overview of how these timers work, and when you might want to use them. I’ll also cover the Lodash functions debounce() and throttle(), because I find them useful as well.

## Promises and microtasks

Let’s get this one out of the way first, because it’s probably the simplest. A [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) callback is also called a “microtask,” and it runs at the same frequency as [MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) callbacks. Assuming [queueMicrotask()](https://github.com/whatwg/html/commit/9d7cf125f960e6bb8d9b7c9456595f505f2e9d4b) ever makes it out of spec-land and into browser-land, it will also be the same thing.

[I’ve already written a lot about promises](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html). One quick misconception about promises that’s worth covering, though, is that they don’t give the browser a chance to breathe. Just because you’re queuing up an asynchronous callback, that doesn’t mean that the browser can render, or process input, or do any of the stuff we want browsers to do.

For example, let’s say we have a function that blocks the main thread for 1 second:

```
function block() {
  var start = Date.now()
  while (Date.now() - start < 1000) { /* wheee */ }
}
```

If we were to queue up a bunch of microtasks to call this function:

```
for (var i = 0; i < 100; i++) {
  Promise.resolve().then(block)
}
```

This would block the browser for about 100 seconds. It’s basically the same as if we had done:

```
for (var i = 0; i < 100; i++) {
  block()
}
```

Microtasks execute immediately after any synchronous execution is complete. There’s no chance to fit in any work between the two. So if you think you can break up a long-running task by separating it into microtasks, then it won’t do what you think it’s doing.

## setTimeout and setInterval

These two are cousins: [setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout) queues a task to run in x number of milliseconds, whereas [setInterval](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setInterval) queues a recurring task to run every x milliseconds.

The thing is… browsers don’t really respect that milliseconds thing. You see, historically, web developers have abused `setTimeout `. A lot. To the point where browsers have had to add mitigations for `setTimeout(/* ... */, 0)` to avoid locking up the browser’s main thread, because a lot of websites tended to throw around `setTimeout(0)` like confetti.

This is the reason that a lot of the tricks in [crashmybrowser.com](https://www.crashmybrowser.com/) don’t work anymore, such as queuing up a `setTimeout` that calls two more `setTimeout`s, which call two more `setTimeout`s, etc. I covered a few of these mitigations from the Edge side of things in [“Improving input responsiveness in Microsoft Edge”](https://blogs.windows.com/msedgedev/2017/06/01/input-responsiveness-event-loop-microsoft-edge/).

Broadly speaking, a `setTimeout(0)` doesn’t really run in zero milliseconds. Usually, it runs in 4. Sometimes, it may run in 16 (this is what Edge does when it’s on battery power, for instance). Sometimes it may be clamped to 1 second (e.g., [when running in a background tab](https://github.com/WICG/interventions/issues/5)). These are the sorts of tricks that browsers have had to invent to prevent runaway web pages from chewing up your CPU doing useless `setTimeout` work.

So that said, `setTimeout` does allow the browser to run some work before the callback fires (unlike microtasks). But if your goal is to allow input or rendering to run before the callback, `setTimeout` is usually not the best choice because it only incidentally allows those things to happen. Nowadays, there are better browser APIs that can hook more directly into the browser’s rendering system.

## setImmediate
Before moving on to those “better browser APIs,” it’s worth mentioning this thing. [setImmediate](https://developer.mozilla.org/en-US/docs/Web/API/Window/setImmediate) is, for lack of a better word … weird. If you look it up on [caniuse.com](https://caniuse.com/#feat=setimmediate), you’ll see that only Microsoft browsers support it. And yet [it also exists in Node.js](https://nodejs.org/api/timers.html#timers_setimmediate_callback_args), and has lots of [“polyfills” on npm](https://www.npmjs.com/search?q=setimmediate). What the heck is this thing?

`setImmediate` was originally proposed by Microsoft to get around the problems with `setTimeout` described above. Basically, `setTimeout` had been abused, and so the thinking was that we can create a new thing to allow `setImmediate(0)` to actually be `setImmediate(0)` and not this funky “clamped to 4ms” thing. You can see [some discussion about it from Jason Weber back in 2011](https://lists.w3.org/Archives/Public/public-web-perf/2011Jul/0012.html).

Unfortunately, `setImmediate` was only ever adopted by IE and Edge. Part of the reason it’s still in use is that it has a sort of superpower in IE, where it allows input events like keyboard and mouseclicks to “jump the queue” and fire before the `setImmediate` callback is executed, whereas IE doesn’t have the same magic for `setTimeout`. (Edge eventually fixed this, as detailed in the previously-mentioned post.)

lso, the fact that `setImmediate` exists in Node means that a lot of “Node-polyfilled” code is using it in the browser without really knowing what it does. It doesn’t help that the differences between Node’s `setImmediate` and `process.nextTick` are very confusing, and even [the official Node docs](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/) say the names should really be reversed. (For the purposes of this blog post though, I’m going to focus on the browser rather than Node because I’m not a Node expert.)

Bottom line: use `setImmediate` if you know what you’re doing and you’re trying to optimize input performance for IE. If not, then just don’t bother. (Or only use it in Node.)

## requestAnimationFrame

Now we get to the most important `setTimeout` replacement, a timer that actually hooks into the browser’s rendering loop. By the way, if you don’t know how the browser event loops works, I strongly recommend [this talk by Jake Archibald](https://youtu.be/cCOL7MC4Pl0). Go watch it, I’ll wait.

Okay, now that you’re back, `requestAnimationFrame` basically works like this: it’s sort of like a `setTimeout`, except instead of waiting for some unpredictable amount of time (4 milliseconds, 16 milliseconds, 1 second, etc.), it executes before the browser’s next style/layout calculation step. Now, as Jake points out in his talk, there is a minor wrinkle in that it actually executes after this step in Safari, IE, and Edge <18, but let's ignore that for now since it's usually not an important detail.

The way I think of `requestAnimationFrame` is this: whenever I want to do some work that I know is going to modify the browser's style or layout – for instance, changing CSS properties or starting up an animation – I stick it in a `requestAnimationFrame` (abbreviated to `rAF` from here on out). This ensures a few things:

1. I'm less likely to layout thrash, because all of the changes to the DOM are being queued up and coordinated.
2. My code will naturally adapt to the performance characteristics of the browser. For instance, if it's a low-cost device that is struggling to render some DOM elements, rAF will naturally slow down from the usual 16.7ms intervals (on 60 Hertz screens) and thus it won't bog down the machine in the same way that running a lot of setTimeouts or setIntervals might.

This is why animation libraries that don't rely on CSS transitions or keyframes, such as [GreenSock](https://greensock.com/) or [React Motion](https://github.com/chenglou/react-motion), will typically make their changes in a rAF callback. If you're animating an element between `opacity: 0` and `opacity: 1`, there's no sense in queuing up a billion callbacks to animate every possible intermediate state, including `opacity: 0.0000001` and `opacity: 0.9999999`.

Instead, you're better off just using `rAF` to let the browser tell you how many frames you're able to paint during a given period of time, and calculate the "tween" for that particular frame. That way, slow devices naturally end up with a slower framerate, and faster devices end up with a faster framerate, which wouldn't necessarily be true if you used something like `setTimeout`, which operates independently of the browser's rendering speed.

## requestIdleCallback
`rAF` is probably the most useful timer in the toolkit, but [`requestIdleCallback`](https://developer.mozilla.org/en-US/docs/Web/API/Background_Tasks_API) is worth talking about as well. The [browser support isn't great](https://caniuse.com/#feat=requestidlecallback) , but there's a[ polyfill that works just fine](https://www.npmjs.com/package/requestidlecallback) (and it uses rAF under the hood).

In many ways `rAF` is similar to `requestIdleCallback`. (I'll abbreviate it to `rIC` from now on. Starting to sound like a pair of troublemakers from West Side Story, huh? "There go Rick and Raff, up to no good!")

Like `rAF`, `rIC` will naturally adapt to the browser's performance characteristics: if the device is under heavy load, `rIC` may be delayed. The difference is that `rIC` fires on the browser "idle" state, i.e. when the browser has decided it doesn't have any tasks, microtasks, or input events to process, and you're free to do some work. It also gives you a "deadline" to track how much of your budget you're using, which is a nice feature.

Dan Abramov has [a good talk from JSConf Iceland 2018](https://youtu.be/v6iR3Zk4oDY) where he shows how you might use `rIC`. In the talk, he has a webapp that calls `rIC` for every keyboard event while the user is typing, and then it updates the rendered state inside of the callback. This is great because a fast typist can cause many `keydown`/`keyup` events to fire very quickly, but you don't necessarily want to update the rendered state of the page for every keypress.

Another good example of this is a “remaining character count” indicator on Twitter or Mastodon. I use `rIC` for this in [Pinafore](https://pinafore.social/), because I don't really care if the indicator updates for every single key that I type. If I'm typing quickly, it's better to prioritize input responsiveness so that I don't lose my sense of flow.

![](https://nolanwlawson.files.wordpress.com/2018/09/screenshot-2018-09-01-13-59-21.png)

In Pinafore, the little horizontal bar and the “characters remaining” indicator update as you type.

One thing I’ve noticed about `rIC`, though, is that it’s a little finicky in Chrome. In Firefox it seems to fire whenever I would, intuitively, think that the browser is “idle” and ready to run some code. (Same goes for the polyfill.) In mobile Chrome for Android, though, I’ve noticed that whenever I scroll with touch scrolling, it might delay `rIC` for several seconds even after I’m done touching the screen and the browser is doing absolutely nothing. (I suspect the issue I’m seeing is [this one](https://bugs.chromium.org/p/chromium/issues/detail?id=811451).)

**Update**: Alex Russell from the Chrome team [informs me](https://toot.cafe/@slightlyoff/100655566963584982) that this is a known issue and should be fixed soon!


In any case, `rIC` is another great tool to add to the tool chest. I tend to think of it this way: use `rAF` for critical rendering work, use `rIC` for non-critical work.

## debounce and throttle

These two functions aren’t built in to the browser, but they’re so useful that they’re worth calling out on their own. If you aren’t familiar with them, there’s [a good breakdown in CSS Tricks](https://css-tricks.com/debouncing-throttling-explained-examples/).

My standard use for [`debounce`](https://lodash.com/docs/4.17.10#debounce) is inside of a [`resize`](https://developer.mozilla.org/en-US/docs/Web/Events/resize) callback. When the user is resizing their browser window, there’s no point in updating the layout for every `resize` callback, because it fires too frequently. Instead, you can `debounce` for a few hundred milliseconds, which will ensure that the callback eventually fires once the user is done fiddling with their window size.

[`throttle`](https://lodash.com/docs/4.17.10#throttle), on the other hand, is something I use much more liberally. For instance, a good use case is inside of a [`scroll`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onscroll) event. Once again, it’s usually senseless to try to update the rendered state of the app for every `scroll` callback, because it fires too frequently (and the frequency can vary from browser to browser and from input method to input method… ugh). Using `throttle` normalizes this behavior, and ensures that it only fires every x number of milliseconds. You can also tweak Lodash’s `throttle` (or `debounce`) function to fire at the start of the delay, at the end, both, or neither.

In contrast, I wouldn’t use `debounce` for the scrolling scenario, because I don’t want the UI to only update after the user has explicitly stopped scrolling. That can get annoying, or even confusing, because the user might get frustrated and try to keep scrolling in order to update the UI state (e.g. in an infinite-scrolling list). `throttle` is better in this case, because it doesn’t wait for the `scroll` event to stop firing.

`throttle` is a function I use all over the place for all kinds of user input, and even for some regularly-scheduled tasks like IndexedDB cleanups. It’s extremely useful. Maybe it should just be baked into the browser some day!

## Conclusion

So that’s my whirlwind tour of the various timer functions available in the browser, and how you might use them. I probably missed a few, because there are certainly some exotic ones out there ([`postMessage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) or [`lifecycle events`](https://developers.google.com/web/updates/2018/07/page-lifecycle-api), anyone?). But hopefully this at least provides a good overview of how I think about JavaScript timers on the web.