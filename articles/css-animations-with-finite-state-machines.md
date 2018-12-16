> * 原文地址：[css-animations-with-finite-state-machines](https://medium.com/@DavidKPiano/css-animations-with-finite-state-machines-7d596bb2914a)
> * 原文作者：[David Khourshid](https://medium.com/@DavidKPiano)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/.md](https://github.com/dawn-teams/translate/blob/master/articles/.md)
> * 译者：[也树](https://github.com/xdlrt)
> * 校对者：[]()

As the number of different possible states and transitions between states in a user interface grows, managing styles and animations can quickly become complicated. Even a simple login form can have many different “user flows” and edge cases that need to be considered.

示例：[https://codepen.io/davidkpiano/pen/WKvPBP](https://codepen.io/davidkpiano/pen/WKvPBP)

State machines are an excellent pattern for managing state transitions in user interfaces in an intuitive, declarative way. We’ve been using them a lot on [the Keyframers](https://keyframe.rs/) as a way to simplify otherwise complex animations and user flows, like the one above.

So, what is a state machine? Sounds technical, right? It’s actually more simple and intuitive than you might think. (Don’t look at [Wikipedia](https://en.wikipedia.org/wiki/Finite-state_machine) article just yet… trust me.)

Let’s approach this from an animation perspective. Suppose you’re creating a loading animation, which can be in only one of four states at any given time:

- idle (not loading yet)
- loading
- failure
- success

This makes sense — it should be impossible for your animation to be in both the “loading” and “success” states at the same time. But, it’s also important to consider how these states transition to each other.

![](https://img.alicdn.com/tfs/TB1EwA8wbvpK1RjSZPiXXbmwXXa-398-341.png)

Each arrow shows us how one state transitions to another state via events, and how some state transitions should be impossible (that is, you can’t go from the `success` state to the `failure` state). Each one of those arrows is an animation that you can implement, or more practically, a transition. If you’re wondering where the term “CSS transitions” comes from, it’s for describing how one visual “state” in CSS transitions to another visual “state.”

In other words, if you’re using CSS transitions, you’ve been using state machines all along and you didn’t even realize it! However, you were probably toggling between different “states” by adding and removing classes:

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

This may work fine, but you have to make sure that the `is-loading` class is removed and the `is-loaded` class is added, because it's all too possible to have a `.button.is-loading.is-loaded`. This can lead to unintended side-effects.

A better pattern for this is using [data-attributes](https://developer.mozilla.org/en-US/docs/Learn/HTML/Howto/Use_data_attributes). They’re useful because they represent a single value. When a part of your UI can only be in one state at a time (such as `loading` or `success` or `error`), updating a data-attribute is much more straightforward:

```
const elButton = document.querySelector('.button');
// set to loading
elButton.dataset.state = 'loading';
// set to success
elButton.dataset.state = 'success';
```

This naturally enforces that there is a single, finite state that your button can be in at any given time. You can use this `data-state `attribute to represent the different button states:

```css
.button[data-state="loading"] {
  opacity: 0.5;
}
.button[data-state="success"] {
  opacity: 1;
  background-color: green;
}
```

## Finite State Machines
More formally, a finite state machine is made up of five parts:

- A finite set of states (e.g., idle, loading, success, failure)
- A finite set of events (e.g., FETCH, ERROR, RESOLVE, RETRY)
- An initial state (e.g., idle)
- A set of transitions (e.g., idle transitions to loading on the FETCH event)
- Final states

And it has a couple rules:

- A finite state machine can only be in one state at any given time
- All transitions must be deterministic, meaning for any given state and event, it must always go to the same predefined next state. No surprises!
Now let’s look at how we can represent finite states in HTML and CSS.

## Contextual State
Sometimes, you’ll need to style other UI components based on what state the app (or some parent component) is in. “Read-only” data-attributes can also be used for this, such as `data-show`:

```css
.button[data-state="loading"] .text[data-show="loading"] {
  display: inline-block;
}
.button[data-state="loading"] .text[data-show]:not([data-show="loading"]) {
  display: none;
}
```

This is a way to signify that certain UI elements should only be shown in certain states. Then, it’s just a matter of adding [data-show="..."] to the respective elements that should be shown. If you want to handle a component being shown for multiple states, you can use the [space-separated attribute selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors) with HTML like the following:

```js
<button class="button" data-state="idle">
  <!-- Show download icon while in idle or loading states -->
  <span class="icon" data-show="idle loading"></span>
  <span class="text" data-show="idle">Download</span>
  <span class="text" data-show="loading">Downloading...</span>
  <span class="text" data-show="success">Done!</span>
</button>
```

And with the corresponding CSS:

```css
/* ... */
.button[data-state="loading"] [data-show~="loading"] {
  display: inline-block;
}
```

The `data-state` attribute can be modified using JavaScript:

```js
const elButton = document.querySelector('.button');
function setButtonState(state) {
  // set the data-state attribute on the button
  elButton.dataset.state = state;
}
setButtonState('loading');
// the button's data-state attribute is now "loading"
```

## Dynamic Data-Attribute Styles
As your app grows, adding all of these data-attribute rules can make your stylesheet get bigger and harder to maintain, since you have to maintain the different states in both the client JavaScript files and in the stylesheet. It can also make specificity complicated since each class and data-attribute selector adds to the specificity weight. To mitigate this, we can instead use a dynamic `data-active` attribute that follows these two rules:

When the overall state matches a [data-show="..."] state, the element should have the data-active attribute.
When the overall state doesn’t match any [data-hide="..."] state, the element should also have the data-active attribute.
Here’s how this can be implemented in JavaScript:

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

Now, our above show/hide styles can be simplified:

```css
.text[data-active] {
  display: inline-block;
}
.text:not([data-active]) {
  display: none;
}
```

## Declaratively Visualizing States
So far, so good. However, we want to prevent function calls to change state littered throughout our UI business logic. We can create a state machine transition function that contains the logic for what the next state should be given the current state and event, and returns that next state. With a switch-case block, here’s how that might look:

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

The switch-case block codifies the transitions between states based on events. We can simplify this by using objects instead:

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

Not only does this look cleaner than the switch-case code block, but it is also JSON-serializable, and we can declaratively iterate over the states and events. This allows us to copy-paste the `buttonMachine` definition code into a visualization tool, like [xviz](https://musing-rosalind-2ce8e7.netlify.com/?machine=%7B%22initial%22%3A%22idle%22%2C%22states%22%3A%7B%22idle%22%3A%7B%22on%22%3A%7B%22FETCH%22%3A%22loading%22%7D%7D%2C%22loading%22%3A%7B%22on%22%3A%7B%22ERROR%22%3A%22failure%22%2C%22RESOLVE%22%3A%22success%22%7D%7D%2C%22failure%22%3A%7B%22on%22%3A%7B%22RETRY%22%3A%22loading%22%7D%7D%2C%22success%22%3A%7B%7D%7D%7D):

![](https://img.alicdn.com/tfs/TB1Bak7wirpK1RjSZFhXXXSdXXa-650-400.png)

## Summary
The state machine pattern makes it much simpler to handle state transitions in your app, and also makes it cleaner to apply transition styles in your CSS. To summarize, we introduced the following data-attributes:

- `data-state` represents the finite state for the component (e.g., `data-state="loading"`)
- `data-show` dictates that the element should be `data-active` if one of the states matches the overall `data-state` (e.g., `data-show`="idle loading")
- `data-hide` dictates that the element should not be `data-active` if one of the states matches the overall `data-state `(e.g., `data-hide="success error"`)
- `data-active` is dynamically added to the above `data-show` and `data-hide` elements when they are "matched" by the current `data-state`.

And the following code patterns:

- Defining a `machine` definition as a JavaScript object with the following properties:
- `initial` - the initial state of the machine (e.g., `"idle"`)
- `states` - a mapping of states to "transition objects" with the on property:
- `on` - a mapping of events to next states (e.g., `FETCH: "loading"`)
- Creating a `transition(currentState, event)` function that returns the next state by looking it up from the above machine definition
- Creating a `send(event)` function that:
1. calls `transition(...)` to determine the next state
2. sets the `currentState` to that next state
3. executes side effects (sets the proper data-attributes, in this case).

As a bonus, we’re able to visualize our app’s behavior from that machine definition! We can also manually test each state by calling `setButtonState(...)` to the desired state, which will set the proper data-attributes and allow us to develop and debug our components in specific states. This eliminates the frustration of having to "go through the flow" in order to get our app to the proper state.

## Going further
If you want to dive deeper into state machines (and their scalable companion, “statecharts”), check out the below resources:

[xstate](https://github.com/davidkpiano/xstate) is a library I created that facilitates the creation and execution of state machines and statecharts, with support for nested/parallel states, actions, guards, and more. By reading this article, you already know how to use it:
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

[The World of Statecharts](https://statecharts.github.io/) is a fantastic resource by [Erik Mogensen](https://twitter.com/mogsie) that thoroughly explains statecharts and how it’s applicable to user interfaces
[Spectrum Statecharts community](https://spectrum.chat/statecharts) is full of developers who are helpful, passionate, and eager to learn and use state machines and statecharts
[Learn State Machines](https://learnstatemachines.com/) is a course that teaches you the fundamental concepts of statecharts through example — by building an Instagram clone and more!
[React-Automata](https://github.com/MicheleBertoli/react-automata/) is a library by [Michele Bertoli](https://twitter.com/michelebertoli) that uses xstate and allows you use statecharts in React, with many benefits, including automatically generated snapshot tests!
I talked with [Jon Bellah](https://jonbellah.com/articles/intro-state-machines) on Shop Talk Show about Working with [State Machines](https://shoptalkshow.com/episodes/327-working-state-machines/), if you want to learn more about the benefits of using them in front-end UIs.
And finally, I’m working on an interactive statechart visualizer, editor, generative testing and analysis tool for easily creating statecharts for user interfaces. For more info, and to be notified when the beta releases, visit [uistates.com](https://uistates.com/). 🚀