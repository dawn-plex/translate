> * 原文地址：[How to build an RPC based API with node.js](https://scotch.io/@alloys/how-to-build-an-rpc-based-api-with-nodejs)
> * 原文作者：[Alloys Mila](https://scotch.io/@alloys)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/How-to-build-an-rpc-based-api-with-nodejs.md](https://github.com/dawn-teams/translate/blob/master/articles/How-to-build-an-rpc-based-api-with-nodejs.md)
> * 译者：[牧曈](https://github.com/jeasonstudio)
> * 校对者：[]()
 ---

# How to build an RPC based API with node.js

APIs are eating the development world, they have been for a while. Whether you are building a microservice for access only by other microservices or you are building something exposed to the world, at some point, you may need to develop an api.

These days, most APIs under development are most likely rest based, rest is simple, easy, and built on top of http, the protocol of the web. It's ever, very likely, that at a significant level of scale, rest might not work well for you, and judging by the fact that many companies, uber, facebook, google, netflix have built their own protocols for internal communication between services, it's likely a matter of when, not if.

Assuming, you want to go old school, with RPC, but you want to stick with our friend, `json over http` how would you go about it with node.js? Well that's what this article is here to discover.

\##Assumptions This tutorial assumes

- You have at least basic knowledge on practical use of Node.js
- You have Node.js > `v4.0.0` installed on your system, for considerable es6 support.

## Design rules

specific to this tutorial, we'll set up a two constraints for our api as follows

- Keep it simple (so no external packages, and complicated operations)
- Self Documenting, the api and it's documentation, should grow together

## So, Let's Begin...

The full source code for this tutorial is available on [Github](https://github.com.mofax/noderpc), so you can clone it just to get things going faster.
Ok, With everything in mind, we need to start by defining our types, and the methods that will operate on them, (these will be the same methods called via the api).

We'll create a new directory, and inside the new directory, create two files, `types.js`and `methods.js`. Type the following commands if you're running a linux or a mac terminal. On windows things might differ, but you can install [mintty](https://www.google.com/search?q=mintty) to access the same commands on a windows terminal.

```bash
mkdir noderpc && cd noderpc
touch types.js methods.js
```

Inside our types.js file, we'll have the following,

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

Just from first sight, it's really simple, we have a key-value container to hold our types, the keys being the name of the type, and the value being it's definition. The definition, includes a description, which is human friendly text, and is mainly there for documentation purposes, then we have it's props, which describes the individual properties, and is designed both for documentation, and for validation. We them expose this container to the world via `module.exports`

Moving on to methods.js, we'll have this

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

You'll notice that, it's almost similar to the types module by design, the major difference however, is that each method definition, includes a function called `exec`that returns a promise. This function is what exposes the functionality of the method, and while the other properties can be exposed to users, this has to be abstracted by the api.

You will, also notice that, at the top of methods.js we have required a module `db.js`that we are yet to create, so let's go ahead and create it.

#### db.js

Our api will need to store data somewhere, but, because for the purposes of this tutorial we don't want to complicate things by unnecessary `npm installs` we are going to create a really simple, and naive, in-memory key value store, of cause because it's behind it's own abstraction, you can alsways change how data is stored behind the scenes.

With that out of the way, let's have this inside `db.js`

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

The important thing here is the `proc` function. (I don't even know why I called it that!). I takes an object, and wraps it inside a closure with a set of functions around it, so that you can add, edit and remove values on that object. You should read about javascript closures if you are not yet conversant with them, they're really cool.

So, we now basically have our application set up, We can store and retrieve data, and can implement functions to operate on this data, what we now need to do is to expose this functionality over the network. As such, the last piece is to implement a http server.

This is the point where most of us would have wished to use express, but we don't want that, so we're going to use the http module that ships with node, and implement a very simple routing table around it.

And, as expected, let's go ahead and create our `server.js` file. This is the file that sort of binds everything together, so we'll go step by step, at the top of the file, we'll have this

```javascript
'use strict';

let http = require('http');
let url = require('url');
let methods = require('./methods');
let types = require('./types');

let server = http.createServer(requestListener);
const PORT = process.env.PORT || 9090;
```

What we are doing at the very beginning of the file is to import all the things we'll need, the we use `http.createServer` to create a http server. The value `requestListener` is a callback function, that we'll define later. Then of course, we have the port on which the server will listen.

Just after this code, let us have our routing table, that defines the different url paths that our application will respond to.

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

This is a very important part of the application, since it sets up the actual interface. We have a set of endpoints, each with with a handler function that is called when that endpoint is hit. Each handler function has to return a Promise, as a design principle.

The RPC endpoint, gets a json object containing the requests, each request is then resolved into a method in the `methods.js` file, the `exec` function on that method is called, and the results passed back. Or, an Error is thrown.

The describe endpoint, scans through the descriptions of both the methods and the types, and returns that information to the caller. Makes it easy for the consumers of your api to always know how to use it, even as it is actively developed.

Now let's add that function `requestListener` that we talked about ealier, then start up the server, and this is it

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

This function is called everytime there is a new request, we wait on the data coming in, after which, we look at the path, and match it to a handler on the routing table. We then start up the server using `server.listen`.

There we have it, now you can run `node server.js` from inside the directory to start up the server, then using postman, or your api tool of choice, send a request to `http://localhost:{PORT}/rpc` containing the following json body

```json
{
    "createUser": {
        "name":"alloys mila",
        "age":24
    }
}
```

The server should create the new user an respond with the results. There you go, a working RPC based, documentable api, to give to your users.

\##Note It's good to note that we have not done any validations on this tutorial, something you must always do on any incoming data.