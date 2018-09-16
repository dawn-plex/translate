> * 原文地址：[How to build an RPC based API with node.js](https://scotch.io/@alloys/how-to-build-an-rpc-based-api-with-nodejs)
> * 原文作者：[Alloys Mila](https://scotch.io/@alloys)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/How-to-build-an-rpc-based-api-with-nodejs.md](https://github.com/dawn-teams/translate/blob/master/articles/How-to-build-an-rpc-based-api-with-nodejs.md)
> * 译者：[牧曈](https://github.com/jeasonstudio)
> * 校对者：[也树](https://github.com/xdlrt)，[灵沼](https://github.com/su-dan)
---

# 如何使用 NodeJS 构建基于 RPC 的 API 系统

API 在它存在的很长时间内都不断地侵蚀着我们的开发工作。无论是构建仅供其他微服务访问的微服务还是构建对外暴露的服务，你都需要开发 API。

目前，大多数 API 都基于 REST 规范，REST 规范通俗易懂，并且建立在 HTTP 协议之上。 但是在很大程度上，REST 可能并不适合你。许多公司比如 Uber，facebook，Google，netflix 等都构建了自己的服务间内部通信协议，这里的关键问题在于何时做，而不是应不应该做。

假设你想使用传统的 RPC 方式，但是你仍然想通过 http 格式传递 json 数据，这时要怎么通过 node.js 来实现呢？请继续阅读本文。

**阅读本教程前应确保以下两点**

- 你至少应该具备 Node.js 的实战经验
- 为了获得 ES6 支持，需要安装 Node.js `v4.0.0` 以上版本。

## 设计原则

在本教程中，我们将为 API 设置如下两个约束：
- 保持简单（没有外部包装和复杂的操作）
- API 和接口文档，应该一同编写

## 现在开始

本教程的完整源代码可以在 [Github](https://github.com.mofax/noderpc) 上找到，因此你可以 clone 下来方便查看。
首先，我们需要首先定义类型以及将对它们进行操作的方法（这些将是通过 API 调用的相同方法）。

创建一个新目录，并在新目录中创建两个文件，`types.js` 和 `methods.js`。 如果你正在使用 linux 或 mac 终端，可以键入以下命令。

```bash
mkdir noderpc && cd noderpc
touch types.js methods.js
```

在 `types.js` 文件中，输入以下内容。

```javascript
'use strict';

let types = {
    user: {
        description:'the details of the user',
        props: {
            name:['string', 'required'],
            age: ['number'],
            email: ['string', 'required'],
            password: ['string', 'required']
        }
    },
    task: {
        description:'a task entered by the user to do at a later time',
        props: {
            userid: ['number', 'required'],
            content: ['string', 'require'],
            expire: ['date', 'required']
        }
    }
}

module.exports = types;
```

乍一看很简单，用一个 `key-value` 对象来保存我们的类型，`key` 是类型的名称，`value` 是它的定义。该定义包括描述（是一段可读文本，主要用于生成文档），在 props 中描述了各个属性，这样设计主要用于文档生成和验证，最后通过 `module.exports` 暴露出来。

在 `methods.js` 有以下内容。

```javascript
'use strict';

let db = require('./db');

let methods = {
    createUser: {
        description: `creates a new user, and returns the details of the new user`,
        params: ['user:the user object'],
        returns: ['user'],
        exec(userObj) {
            return new Promise((resolve) => {
                if (typeof (userObj) !== 'object') {
                    throw new Error('was expecting an object!');
                }
                // you would usually do some validations here
                // and check for required fields

                // attach an id the save to db
                let _userObj = JSON.parse(JSON.stringify(userObj));
                _userObj.id = (Math.random() * 10000000) | 0; // binary or, converts the number into a 32 bit integer
                resolve(db.users.save(userObj));
            });
        }
    },

    fetchUser: {
        description: `fetches the user of the given id`,
        params: ['id:the id of the user were looking for'],
        returns: ['user'],
        exec(userObj) {
            return new Promise((resolve) => {
                if (typeof (userObj) !== 'object') {
                    throw new Error('was expecting an object!');
                }
                // you would usually do some validations here
                // and check for required fields

                // fetch
                resolve(db.users.fetch(userObj.id) || {});
            });
        }
    },

     fetchAllUsers: {
        released:false;
        description: `fetches the entire list of users`,
        params: [],
        returns: ['userscollection'],
        exec() {
            return new Promise((resolve) => {
                // fetch
                resolve(db.users.fetchAll() || {});
            });
        }
    },

};

module.exports = methods;
```

可以看到，它和类型模块的设计非常类似，但主要区别在于每个方法定义中都包含一个名为 `exec` 的函数，它返回一个 `Promise`。 这个函数暴露了这个方法的功能，虽然其他属性也暴露给了用户，但这必须通过 API 抽象。

#### db.js

我们的 API 需要在某处存储数据，但是在本教程中，我们不希望通过不必要的 `npm install` 使教程复杂化，我们创建一个非常简单、原生的内存中键值存储，因为它的数据结构由你自己设计，所以你可以随时改变数据的存储方式。

在 `db.js` 中包含以下内容。

```javascript
'use strict';

let users = {};
let tasks = {};

// we are saving everything inmemory for now
let db = {
    users: proc(users),
    tasks: proc(tasks)
}

function clone(obj) {
    // a simple way to deep clone an object in javascript
    return JSON.parse(JSON.stringify(obj));
}

// a generalised function to handle CRUD operations
function proc(container) {
    return {
        save(obj) {
            // in JS, objects are passed by reference
            // so to avoid interfering with the original data
            // we deep clone the object, to get our own reference
            let _obj = clone(obj);

            if (!_obj.id) {
                // assign a random number as ID if none exists
                _obj.id = (Math.random() * 10000000) | 0;
            }

            container[_obj.id.toString()] = _obj;
            return clone(_obj);
        },
        fetch(id) {
            // deep clone this so that nobody modifies the db by mistake from outside
            return clone(container[id.toString()]);
        },
        fetchAll() {
            let _bunch = [];
            for (let item in container) {
                _bunch.push(clone(container[item]));
            }
            return _bunch;
        },
        unset(id) {
            delete container[id];
        }
    }
}

module.exports = db;
```

其中比较重要是 `proc` 函数。通过获取一个对象，并将其包装在一个带有一组函数的闭包中，方便在该对象上添加，编辑和删除值。如果你对闭包不够了解，应该提前阅读关于 `JavaScript` 闭包的内容。

所以，我们现在基本上已经完成了程序功能，我们可以存储和检索数据，并且可以实现对这些数据进行操作，我们现在需要做的是通过网络公开这个功能。 因此，最后一部分是实现 HTTP 服务。

这是我们大多数人希望使用express的地方，但我们不希望这样，所以我们将使用随节点一起提供的http模块，并围绕它实现一个非常简单的路由表。

正如预期的那样，我们继续创建 `server.js` 文件。在这个文件中我们把所有内容关联在一起，如下所示。

```javascript
'use strict';

let http = require('http');
let url = require('url');
let methods = require('./methods');
let types = require('./types');

let server = http.createServer(requestListener);
const PORT = process.env.PORT || 9090;
```

文件的开头部分引入我们所需要的内容，使用 `http.createServer` 来创建一个 HTTP 服务。`requestListener` 是一个回调函数，我们稍后定义它。 并且我们确定下来服务器将侦听的端口。

在这段代码之后我们来定义路由表，它规定了我们的应用程序将响应的不同 URL 路径。

```javascript
// we'll use a very very very simple routing mechanism
// don't do something like this in production, ok technically you can...
// probably could even be faster than using a routing library :-D

let routes = {
    // this is the rpc endpoint
    // every operation request will come through here
    '/rpc': function (body) {
        return new Promise((resolve, reject) => {
            if (!body) {
                throw new (`rpc request was expecting some data...!`);
            }
            let _json = JSON.parse(body); // might throw error
            let keys = Object.keys(_json);
            let promiseArr = [];

            for (let key of keys) {
                if (methods[key] && typeof (methods[key].exec) === 'function') {
                    let execPromise = methods[key].exec.call(null, _json[key]);
                    if (!(execPromise instanceof Promise)) {
                        throw new Error(`exec on ${key} did not return a promise`);
                    }
                    promiseArr.push(execPromise);
                } else {
                    let execPromise = Promise.resolve({
                        error: 'method not defined'
                    })
                    promiseArr.push(execPromise);
                }
            }

            Promise.all(promiseArr).then(iter => {
                console.log(iter);
                let response = {};
                iter.forEach((val, index) => {
                    response[keys[index]] = val;
                });

                resolve(response);
            }).catch(err => {
                reject(err);
            });
        });
    },

    // this is our docs endpoint
    // through this the clients should know
    // what methods and datatypes are available
    '/describe': function () {
        // load the type descriptions
        return new Promise(resolve => {
            let type = {};
            let method = {};

            // set types
            type = types;

            //set methods
            for(let m in methods) {
                let _m = JSON.parse(JSON.stringify(methods[m]));
                method[m] = _m;
            }

            resolve({
                types: type,
                methods: method
            });
        });
    }
};
```

这是整个程序中非常重要的一部分，因为它提供了实际的接口。 我们有一组 endpoint，每个 endpoint 都对应一个处理函数，在路径匹配时被调用。根据设计原则每个处理函数都必须返回一个 Promise。

RPC endpoint 获取一个包含请求内容的 json 对象，然后将每个请求解析为 `methods.js` 文件中的对应方法，调用该方法的 `exec` 函数，并将结果返回，或者抛出错误。

describe endpoint 扫描方法和类型的描述，并将该信息返回给调用者。让使用 API 的开发者能够轻松地知道如何使用它。

现在让我们添加我们之前讨论过的函数 `requestListener`，然后就可以启动服务。

```javascript
// request Listener
// this is what we'll feed into http.createServer
function requestListener(request, response) {
    let reqUrl = `http://${request.headers.host}${request.url}`;
    let parseUrl = url.parse(reqUrl, true);
    let pathname = parseUrl.pathname;

    // we're doing everything json
    response.setHeader('Content-Type', 'application/json');

    // buffer for incoming data
    let buf = null;

    // listen for incoming data
    request.on('data', data => {
        if (buf === null) {
            buf = data;
        } else {
            buf = buf + data;
        }
    });

    // on end proceed with compute
    request.on('end', () => {
        let body = buf !== null ? buf.toString() : null;

        if (routes[pathname]) {
            let compute = routes[pathname].call(null, body);

            if (!(compute instanceof Promise)) {
                // we're kinda expecting compute to be a promise
                // so if it isn't, just avoid it

                response.statusCode = 500;
                response.end('oops! server error!');
                console.warn(`whatever I got from rpc wasn't a Promise!`);
            } else {
                compute.then(res => {
                    response.end(JSON.stringify(res))
                }).catch(err => {
                    console.error(err);
                    response.statusCode = 500;
                    response.end('oops! server error!');
                });
            }

        } else {
            response.statusCode = 404;
            response.end(`oops! ${pathname} not found here`)
        }
    })
}

// now we can start up the server
server.listen(PORT);
```

每当有新请求时调用此函数并等待拿到数据，之后查看路径，并根据路径匹配到路由表上的对应处理方法。然后使用 `server.listen` 启动服务。

现在我们可以在目录下运行 `node server.js` 来启动服务，然后使用 postman 或你熟悉的 API 调试工具，向 `http://localhost{PORT}/rpc` 发送请求，请求体中包含以下 JSON 内容。

```json
{
    "createUser": {
        "name":"alloys mila",
        "age":24
    }
}
```

server 将会根据你提交的请求创建一个新用户并返回响应结果。一个基于 RPC、文档完善的 API 系统已经搭建完成了。

注意，我们尚未对本教程接口进行任何参数验证，你在调用测试的时候必须手动保证数据正确性。