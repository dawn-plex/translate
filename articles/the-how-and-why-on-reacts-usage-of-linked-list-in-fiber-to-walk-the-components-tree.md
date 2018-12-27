> * 原文地址：[]()
> * 原文作者：[]()
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/.md](https://github.com/dawn-teams/translate/blob/master/articles/.md)
> * 译者：[照天](https://github.com/zzwzzhao)
> * 校对者：[也树](https://github.com/xdlrt)，[靖鑫](https://github.com/luckyjing)

<!-- 正文 -->
# The how and why on React’s usage of linked list in Fiber to walk the component’s tree
### The main algorithm of the work loop in React’s reconciler

![](https://cdn-images-1.medium.com/max/1600/1*d8GcL9UNG0w9n_WW5TCWRw.png)
<p class="figcaption_hack" style="text-align: center;">Work loop representation from [an amazing talk by Lin
Clark](https://www.youtube.com/watch?v=ZCuYPiUIONs) at ReactConf 2017</p>

![](https://cdn-images-1.medium.com/max/1600/1*7CImQFZPe816uJN9FPSWxQ.png)

To educate myself and the community, I spend a lot of time [reverse-engineering
web
technologies](https://blog.angularindepth.com/practical-application-of-reverse-engineering-guidelines-and-principles-784c004bb657)
and writing about my findings. In the last year, I’ve focused mostly on Angular
sources which resulted in the biggest Angular publication on the web —
[Angular-In-Depth](https://blog.angularindepth.com/). **Now the time has come to
dive deep into React**. [Change
detection](https://medium.freecodecamp.org/what-every-front-end-developer-should-know-about-change-detection-in-angular-and-react-508f83f58c6a)
has become the main area of my expertise in Angular, and with some patience and
*a lot of debugging,* I hope to soon achieve that level in React.

In React, the mechanism of change detection is often referred to as
reconciliation or rendering, and Fiber is its newest implementation. Due to the
underlying architecture, it provides capabilities to implement many interesting
features like performing non-blocking rendering, applying updates based on the
priority and pre-rendering content in the background. These features are
referred to as **time-slicing** in the [Concurrent React
philosophy](https://twitter.com/acdlite/status/1056612147432574976).

Besides solving real problems of application developers, **the internal
implementation of these mechanisms has a wide appeal from the engineering
perspective. There’s such a wealth of knowledge in the sources that will help us
grow as developers.**

If you Google “React Fiber” today you’re going to see quite a lot articles in
the search results. All of them, though, except for the [notes by Andrew
Clark](https://github.com/acdlite/react-fiber-architecture) are pretty
high-level explanations. In this article, I’ll refer to this resource and**
provide an elaborate explanation for some particularly important concepts in
Fiber**. Once we’ve finished, you’ll have enough knowledge to understand the
work loop representation from [a very good talk by Lin
Clark](https://www.youtube.com/watch?v=ZCuYPiUIONs) at ReactConf 2017. *That’s
the one talk you need to see.* But it’ll make a lot more sense to you after
you’ve spent a little time in the sources.

[This post opens a series on React’s Fiber
internals.](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e)
I’m about 70% through understanding the internal details of the implementation
and have three more articles on reconciliation and rendering mechanism in the
works.

> I work as a developer advocate at
> [ag-Grid](https://react-grid.ag-grid.com/?utm_source=medium&utm_medium=blog&utm_campaign=reactcustom).
If you’re curious to learn about data grids or looking for the ultimate react
data grid solution, give it a try with the guide “[Get started with React grid
in 5
minutes](http://blog.ag-grid.com/index.php/2018/08/07/get-started-with-react-grid-in-5-minutes/?utm_source=medium&utm_medium=blog&utm_campaign=getstartedreact)”.
I’m happy to answer any questions you may have. [And follow me to stay
tuned!](https://twitter.com/maxim_koretskyi)

Let’s get started!

![](https://cdn-images-1.medium.com/max/1600/1*7CImQFZPe816uJN9FPSWxQ.png)

### Setting the background

Fiber’s architecture has two major phases: reconciliation/render and commit. In
the source code the reconciliation phase is mostly referred to as the “render
phase”. This is the phase when React walks the tree of components and:

* updates state and props,
* calls lifecycle hooks,
* retrieves the children from the component,
* compares them to the previous children,
* and figures out the DOM updates that need to be performed.

**All these activities are referred to as work inside Fiber**. The type of work
that needs to be done depends on the type of the React Element. For example, for
a `Class Component` React needs to instantiate a class, while it doesn't do it
for a `Functional Component`. If interested,
[here](https://github.com/facebook/react/blob/340bfd9393e8173adca5380e6587e1ea1a23cefa/packages/shared/ReactWorkTags.js#L29-L28)
you can see all types of work targets in Fiber. These activities are exactly
what Andrew talks about here:

> When dealing with UIs, the problem is that if **too much work is executed all at
> once**, it can cause animations to drop frames…

Now what about that ‘*all at once’* part? Well, basically, if React is going to
walk the entire tree of components **synchronously** and perform work for each
component, it may run over 16 ms available for an application code to execute
its logic. And this will cause frames to drop causing stuttering visual effects.

So this can be helped?

> Newer browsers (and React Native) implement APIs that help address this exact
> problem…

The new API he talks about is the
[requestIdleCallback](https://developers.google.com/web/updates/2015/08/using-requestidlecallback)
global function that can be used to queue a function to be called during a
browser’s idle periods. Here’s how you would use it by itself:

```js
requestIdleCallback((deadline)=>{
    console.log(deadline.timeRemaining(), deadline.didTimeout)
});
```

If I now open the console and execute the code above, Chrome logs `49.9 false`.
It basically tells me that I have `49.9 ms` to do whatever work I need to do and
I haven’t yet used up all allotted time, otherwise the `deadline.didTimeout`
would be `true`. Keep in mind that `timeRemaining` can change as soon as a
browser gets some work to do, so it should be constantly checked.

> `requestIdleCallback` is actually a bit too restrictive and [is not executed
> often
enough](https://github.com/facebook/react/issues/13206#issuecomment-418923831)
to implement smooth UI rendering, so React team [had to implement their own
version](https://github.com/facebook/react/blob/eeb817785c771362416fd87ea7d2a1a32dde9842/packages/scheduler/src/Scheduler.js#L212-L222).

Now if we put all the activities Reacts performs on a component into the
function `performWork`, and use `requestIdleCallback` to schedule the work, our
code could look like this:

```js
requestIdleCallback((deadline) => {
    // while we have time, perform work for a part of the components tree
    while ((deadline.timeRemaining() > 0 || deadline.didTimeout) && nextComponent) {
        nextComponent = performWork(nextComponent);
    }
});
```

We perform the work on one component and then return the reference to the next
component to process. This would work, if not for one thing. You can’t process
the entire tree of components synchronously, as in the [previous implementation
of the reconciliation
algorithm](https://reactjs.org/docs/codebase-overview.html#stack-reconciler).
And that’s the problem Andrew talks about here:

> in order to use those APIs, you need a way to break rendering work into
> incremental units

So to solve this problem, React had to re-implement the algorithm for walking
the tree **from the synchronous recursive model that relied on the built-in
stack to an asynchronous model with linked list and pointers.** And that’s what
Andrew writes about here:

> If you rely only on the [built-in] call stack, it will keep doing work until the
> stack is empty…Wouldn’t it be great if we could interrupt the call stack at will
and manipulate stack frames manually? That’s the purpose of React Fiber. **Fiber
is re-implementation of the stack, specialized for React components**. You can
think of a single fiber as a virtual stack frame.

And that’s what I’m going to explain now.

![](https://cdn-images-1.medium.com/max/1600/1*7CImQFZPe816uJN9FPSWxQ.png)

#### A word about the stack

I assume you’re all familiar with the notion of a call stack. This is what you
see in your browser’s debugging tools if you pause code at a breakpoint. Here
are a few relevant quotes and diagrams [from
Wikipedia](https://en.wikipedia.org/wiki/Call_stack?fbclid=IwAR06VWEQnwoEawg0NsoR8loBJwIbmPWsXXKqbAuOFBjkawHThK7zlIBsJ_U#Structure):

> In computer science, a **call stack** is a stack data structure that stores
> information about the active subroutines of a computer program… the main reason
for having call stack is **to keep track of the point** to which each active
subroutine should return control when it finishes executing… A **call stack** is
composed of **stack frames**… Each stack frame corresponds to a call to a
subroutine which has not yet terminated with a **return**. For example, if a
subroutine named `DrawLine` is currently running, having been called by a
subroutine `DrawSquare`, the top part of the call stack might be laid out like
in the adjacent picture.

![](https://cdn-images-1.medium.com/max/1600/1*WqEscsoRHItF0p4fZjHfsA.png)


#### Why is the stack relevant to React?

As we defined in the first part of the article, Reacts walks the components tree
during the reconciliation/render phase and performs some work for components.
The previous implementation of the reconciler used the synchronous recursive
model that relied on the built-in stack to walk the tree. [The official doc on
reconciliation](https://reactjs.org/docs/reconciliation.html#recursing-on-children)
describe this process and talk a lot about recursion:

> By default, when recursing on the children of a DOM node, React just iterates
> over both lists of children at the same time and generates a mutation whenever
there’s a difference.

If you think about it, **each recursive call adds a frame to the stack. And it
does so synchronously**. Suppose we have the following tree of components:

![](https://cdn-images-1.medium.com/max/1600/1*TYWa1WAZ9iLip-rwBNPGEQ.png)


Represented as objects with the `render` function. You can think of them as
instances of components:

```js
const a1 = {name: 'a1'};
const b1 = {name: 'b1'};
const b2 = {name: 'b2'};
const b3 = {name: 'b3'};
const c1 = {name: 'c1'};
const c2 = {name: 'c2'};
const d1 = {name: 'd1'};
const d2 = {name: 'd2'};

a1.render = () => [b1, b2, b3];
b1.render = () => [];
b2.render = () => [c1];
b3.render = () => [c2];
c1.render = () => [d1, d2];
c2.render = () => [];
d1.render = () => [];
d2.render = () => [];
```

React needs to iterate the tree and perform work for each component. To
simplify, the work to do is to log the name of the current component and
retrieve its children. Here’s how we do it with recursion.

#### Recursive traversal

The main function that iterates over the tree is called `walk` in the
implementation below:

```js
walk(a1);

function walk(instance) {
    doWork(instance);
    const children = instance.render();
    children.forEach(walk);
}

function doWork(o) {
    console.log(o.name);
}
```

Here’s the output we’re getting:

`a1, b1, b2, c1, d1, d2, b3, c2
`

If you don’t feel confident with recursions, check out [my in-depth article on
recursion](https://medium.freecodecamp.org/learn-recursion-in-10-minutes-e3262ac08a1).

A recursive approach is intuitive and well-suited for walking the trees. But as
we discovered, it has limitations. The biggest one is that we can’t break the
work into incremental units. We can’t pause the work at a particular component
and resume it later. With this approach React just keeps iterating until it
processed all components and the stack is empty.

**So how does React implement the algorithm to walk the tree without recursion?
It uses a singly linked list tree traversal algorithm. It makes it possible to
pause the traversal and stop the stack from growing.**

### Linked list traversal

I was lucky to find the gist of the algorithm outlined by Sebastian Markbåge
[here](https://github.com/facebook/react/issues/7942#issue-182373497). To
implement the algorithm, we need to have a data structure with 3 fields:

* child — reference to the first child
* sibling — reference to the first sibling
* return — reference to the parent

In the context of the new reconciliation algorithm in React, the data structure
with these fields is called Fiber. Under the hood it’s the representation of a
React Element that keeps a queue of work to do. More on that in my next
articles.

The following diagram demonstrates the hierarchy of objects linked through the
linked list and the types of connections between them:

![](https://cdn-images-1.medium.com/max/1600/1*7dsyUaUpKbFG7EoNR9Cu2w.png)

So let’s first define our custom node constructor:

```js
class Node {
    constructor(instance) {
        this.instance = instance;
        this.child = null;
        this.sibling = null;
        this.return = null;
    }
}
```

And the function that takes an array of nodes and links them together. We’re
going to use it to link children returned by the `render` method:

```js
function link(parent, elements) {
    if (elements === null) elements = [];

    parent.child = elements.reduceRight((previous, current) => {
        const node = new Node(current);
        node.return = parent;
        node.sibling = previous;
        return node;
    }, null);

    return parent.child;
}
```

The function iterates over the array of nodes starting from the last one and
links them together in a singly linked list. It returns the reference to the
first sibling in the list. Here is a simple demo of how it works:

```js
const children = [{name: 'b1'}, {name: 'b2'}];
const parent = new Node({name: 'a1'});
const child = link(parent, children);

// the following two statements are true
console.log(child.instance.name === 'b1');
console.log(child.sibling.instance === children[1]);
```

We’ll also implement a helper function that performs some work for a node. In
our case, it’s going to log the name of a component. But besides that it also
retrieves the children of a component and links them together:

```js
function doWork(node) {
    console.log(node.instance.name);
    const children = node.instance.render();
    return link(node, children);
}
```

Okay, now we’re ready to implement the main traversal algorithm. It’s a parent
first, depth-first implementation. Here is the code with useful comments:

```js
function walk(o) {
    let root = o;
    let current = o;

    while (true) {
        // perform work for a node, retrieve & link the children
        let child = doWork(current);

        // if there's a child, set it as the current active node
        if (child) {
            current = child;
            continue;
        }

        // if we've returned to the top, exit the function
        if (current === root) {
            return;
        }

        // keep going up until we find the sibling
        while (!current.sibling) {

            // if we've returned to the top, exit the function
            if (!current.return || current.return === root) {
                return;
            }

            // set the parent as the current active node
            current = current.return;
        }

        // if found, set the sibling as the current active node
        current = current.sibling;
    }
}
```

Although the implementation is not particularly difficult to understand, you may
need to play with it a little to grok it. [Do it
here](https://stackblitz.com/edit/js-tle1wr). The idea is that we keep the
reference to the current node and re-assign it while descending the tree until
we hit the end of the branch. Then we use the `return` pointer to return to the
common parent.

If we now check the call stack with this implementation, here’s what we’re going
to see:

![](https://cdn-images-1.medium.com/max/1600/1*ybVgRoNf-dBxR_OKxn4oKQ.gif)

As you can see, the stack doesn’t grow as we walk down the tree. But if now put
the debugger into the `doWork` function and log node names, we’re going to see
the following:

![](https://cdn-images-1.medium.com/max/1600/1*ErzqXpJt5KkLKxHCn31hmA.gif)

**It looks like a callstack in a browser.** So with this algorithm, we’re
effectively replacing the browser’s implementation of the call stack with our
own implementation. That’s what Andrew describes here:

> Fiber is re-implementation of the stack, specialized for React components. You
> can think of a single fiber as a virtual stack frame.

Since we’re now controlling the stack by keeping the reference to the node that
acts as a top frame:

```js
function walk(o) {
    let root = o;
    let current = o;

    while (true) {
            ...

            current = child;
            ...
            
            current = current.return;
            ...

            current = current.sibling;
    }
}
```

we can stop the traversal at any time and resume to it later. That’s exactly the
condition we wanted to achieve to be able to use the new `requestIdleCallback`
API.

### Work loop in React

[Here’s the
code](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1118)
that implements work loop in React:

```js
function workLoop(isYieldy) {
    if (!isYieldy) {
        // Flush work without yielding
        while (nextUnitOfWork !== null) {
            nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
        }
    } else {
        // Flush asynchronous work until the deadline runs out of time.
        while (nextUnitOfWork !== null && !shouldYield()) {
            nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
        }
    }
}
```

As you can see, it maps nicely to the algorithm I presented above. It keeps the
reference to the current fiber node in the `nextUnitOfWork` variable that acts
as a top frame.

The algorithm can walk the components tree **synchronously **and perform the
work for each fiber node in the tree (nextUnitOfWork). This is usually the case
for so-called interactive updates caused by UI events (click, input etc). Or it
can walk the components tree **asynchronously **checking if there’s time left
after performing work for a Fiber node. The function `shouldYield` returns the
result based on
[deadlineDidExpire](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1806)
and
[deadline](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1809)
variables that are constantly updated as React performs work for a fiber node.

The `peformUnitOfWork` function is described in depth
[here](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e#1a7d).

*****

#### I’m working on the series of in-depth articles that explore the implementation details of Fiber change detection algorithm in React.

### Stay tuned and follow me on [Twitter](https://twitter.com/maxim_koretskyi) and on [Medium](https://medium.com/@maxim.koretskyi), I’ll tweet as soon as it’s ready.

#### Thanks for reading! If you liked this article, hit that clap button below 👏. It means a lot to me and it helps other people see the story.

[<img src="https://cdn-images-1.medium.com/max/1600/1*AqX3Hf4h2icjt8WkIuzv6A.png">](https://react-grid.ag-grid.com/?utm_source=medium&utm_medium=banner&utm_campaign=reactcustom)

<span class="figcaption_hack">React Grid — the fastest and most feature-rich grid component from ag-Grid</span>

---

阿里云翻译小组
