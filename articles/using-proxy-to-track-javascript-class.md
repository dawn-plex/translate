> * 原文地址：[Using Proxy to Track Javascript Class](https://medium.com/front-end-weekly/using-proxy-to-track-javascript-class-50a33a6ccb)
> * 原文作者：[Amir Harel](https://medium.com/@amir.harel)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/.md](https://github.com/dawn-teams/translate/blob/master/articles/using-proxy-to-track-javascript-class.md)
> * 译者：[牧曈](https://github.com/jeasonstudio)
> * 校对者：[]()

---

# 使用 Proxy 追踪 JavaScript 类

[Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 对象是 ES6 中一个很酷而且鲜为人知的特性。虽然它已经存在了相当长的一段时间，但我想写这篇文章并解释一下它的功能，且用一个真实的例子来说明如何使用它。

### Proxy 是什么

正如 MDN 网站中定义的那样：

> The **Proxy** object is used to define custom behavior for fundamental operations (e.g. property lookup, assignment, enumeration, function invocation, etc).
> **Proxy** 对象用于定义基本操作的自定义行为（如属性查找，赋值，枚举，函数调用等）。

虽然这几乎总结的很全面了，但每当读到它时，我并不是很清楚它的作用、它有什么帮助。

首先，Proxy 的概念来自元编程(meta-programming)世界。简单来说，元编程就是允许我们使用 `我们编写的应用程序（或核心）代码` 的代码。例如，允许我们将字符串代码评估为可执行代码的臭名昭着的 `eval 函数` 就属于元编程领域。

`Proxy` API 允许我们在对象及其消费实体之间创建某种层，这使得我们能够控制该对象的行为，比如决定如何完成 `get` 和 `set` 的行为，甚至决定有人试图访问对象中未定义的属性时，我们该怎样做。

### Proxy API

```
var p = new Proxy(target, handler);
```

`Proxy` 对象获取 `target` 和 `handler`，以捕获目标对象中的不同行为。以下列出了部分您可以设置的 `陷阱`：

- `has`  — 捕获 `in` 操作符。例如，这将允许您隐藏对象的某些属性。
- `get`  — 捕获 `获取对象中属性的值` 操作。例如，如果此属性不存在，这将允许您返回一些默认值。
- `set` — 捕获 `设置对象中属性的值` 操作。例如，这将允许您验证设置为属性的值，并在值无效时抛出异常。
- `apply`  — 捕获函数调用。例如，这将允许您将所有函数包装在 `try/catch` 代码块中。

这只是一个简单示例，您可以在MDN网站上查看完整列表。

让我们看一个使用 Proxy 进行验证的简单示例：

```
const Car = {
  maker: 'BMW',
  year: '2018,
};
const proxyCar = new Proxy(Car, {
  set(obj, prop, value) {
    if (prop === 'maker' && value.length < 1) {
      throw new Error('Invalid maker');
    }
    if (prop === 'year' && typeof value !== 'number') {
      throw new Error('Invalid year');
    }
    obj[prop] = value;
    return true;
  }
});
proxyCar.maker = ''; // throw exception
proxyCar.year = '1999'; // throw exception
```

可以看到，我们能够对 `正在设置到代理对象中的值` 进行验证。

### 使用 Proxy 调试

为了显示 Proxy 的强大功能，我创建了一个简单的跟踪库，它会跟踪给定 `对象/类` 的以下内容：

- 函数的执行时间
- 每个方法或属性的调用者
- 每个方法或属性的调用次数

它是通过在任何对象、类甚至函数，上调用 `proxyTrack` 函数实现的。

如果您想要知道谁正在更改对象中的值，或者调用函数的时间长度和次数，以及谁会调用它，这可能非常有用。我知道可能有更好的工具可以做到这一点，但我创建这个库只是为了试用 Proxy API。

#### 使用 proxyTrack

首先，我们看看如何使用它：

```
function MyClass() {}

MyClass.prototype = {
    isPrime: function() {
        const num = this.num;
        for(var i = 2; i < num; i++)
            if(num % i === 0) return false;
        return num !== 1 && num !== 0;
    },

    num: null,
};

MyClass.prototype.constructor = MyClass;

const trackedClass = proxyTrack(MyClass);

function start() {
    const my = new trackedClass();
    my.num = 573723653;
    if (!my.isPrime()) {
        return `${my.num} is not prime`;
    }
}

function main() {
    start();
}

main();
```

如果我们执行这段代码，会在控制台看到：

```
MyClass.num is being set by start for the 1 time
MyClass.num is being get by isPrime for the 1 time
MyClass.isPrime was called by start for the 1 time and took 0 mils.
MyClass.num is being get by start for the 2 time
```

`proxyTrack` 传入 2 个参数：第一个是要跟踪的对象/类，第二个是 `option`，如果未传递，将设置为默认 `option`。我们来看看第二个参数：

```
const defaultOptions = {
    trackFunctions: true,
    trackProps: true,
    trackTime: true,
    trackCaller: true,
    trackCount: true,
    stdout: null,
    filter: null,
};
```

可以看到，您可以通过设置适当的标志来控制要跟踪的内容。如果你想控制输出先到其他地方，然后到 `console.log`，你可以将一个函数传递给 `stdout`。

如果传入 `filter` 回调函数，还可以控制输出哪条跟踪消息。您将获得一个包含跟踪数据信息的对象，必须返回 `true` 以保留消息，或者返回 `false` 可以忽略它。

#### 在 React 中使用 proxyTrack

react 组件实际上是类，因此您可以跟踪类以实时检查它。例如：

```
class MyComponent extends Component{...}

export default connect(mapStateToProps)(proxyTrack(MyComponent, {
    trackFunctions: true,
    trackProps: true,
    trackTime: true,
    trackCaller: true,
    trackCount: true,
    filter: (data) => {
        if( data.type === 'get' && data.prop === 'componentDidUpdate') return false;
        return true;
    }
}));
```

如您所见，您可以过滤掉可能与您无关，或者可能会使控制台混乱的消息。

### proxyTrack 的实现

我们来看下 `proxyTrack` 方法是如何实现的。

首先，函数本身：

```
export function proxyTrack(entity, options = defaultOptions) {
    if (typeof entity === 'function') return trackClass(entity, options);
    return trackObject(entity, options);
}
```

这里没什么特别的，我们只是调用相应的功能。

看下 `trackObject`：

```
function trackObject(obj, options = {}) {
    const { trackFunctions, trackProps } = options;

    let resultObj = obj;
    if (trackFunctions) {
        proxyFunctions(resultObj, options);
    }
    if (trackProps) {
        resultObj = new Proxy(resultObj, {
            get: trackPropertyGet(options),
            set: trackPropertySet(options),
        });
    }
    return resultObj;
}
function proxyFunctions(trackedEntity, options) {
    if (typeof trackedEntity === 'function') return;
    Object.getOwnPropertyNames(trackedEntity).forEach((name) => {
        if (typeof trackedEntity[name] === 'function') {
            trackedEntity[name] = new Proxy(trackedEntity[name], {
                apply: trackFunctionCall(options),
            });
        }
    });
}
```

如您所见，如果我们需要跟踪对象的属性，我们使用 `get` 和 `set` 陷阱创建一个代理对象。以下是设置陷阱的代码：

```
function trackPropertySet(options = {}) {
    return function set(target, prop, value, receiver) {
        const { trackCaller, trackCount, stdout, filter } = options;
        const error = trackCaller && new Error();
        const caller = getCaller(error);
        const contextName = target.constructor.name === 'Object' ? '' : `${target.constructor.name}.`;
        const name = `${contextName}${prop}`;
        const hashKey = `set_${name}`;
        if (trackCount) {
            if (!callerMap[hashKey]) {
                callerMap[hashKey] = 1;
            } else {
                callerMap[hashKey]++;
            }
        }
        let output = `${name} is being set`;
        if (trackCaller) {
            output += ` by ${caller.name}`;
        }
        if (trackCount) {
            output += ` for the ${callerMap[hashKey]} time`;
        }
        let canReport = true;
        if (filter) {
            canReport = filter({
                type: 'get',
                prop,
                name,
                caller,
                count: callerMap[hashKey],
                value,
            });
        }
        if (canReport) {
            if (stdout) {
                stdout(output);
            } else {
                console.log(output);
            }
        }
        return Reflect.set(target, prop, value, receiver);
    };
}
```

`trackClass` 函数对我来说更有趣一些：

```
function trackClass(cls, options = {}) {
    cls.prototype = trackObject(cls.prototype, options);
    cls.prototype.constructor = cls;

    return new Proxy(cls, {
        construct(target, args) {
            const obj = new target(...args);
            return new Proxy(obj, {
                get: trackPropertyGet(options),
                set: trackPropertySet(options),
            });
        },
        apply: trackFunctionCall(options),
    });
}
```

在这种情况下，我们想要为 `函数原型` 创建一个代理并为 `构造函数` 创建一个陷阱，因为我们希望能够在类中捕获不是来自原型的属性。

不要忘记，即使您在原型级别定义了属性，一旦为其设置了值，JavaScript 将创建该属性的本地副本，因此这个类的所有其他实例并不会随之更改。这就是为什么仅仅代理原型还不够。

您可以在[这里](https://gist.github.com/mrharel/592df0228cebc017ca413f2f763acc5f)看到完整的代码。