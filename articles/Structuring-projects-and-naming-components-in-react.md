> * 原文地址：[structuring projects and naming components in react](https://hackernoon.com/structuring-projects-and-naming-components-in-react-1261b6e18d76)
> * 原文作者：[Vinicius Dacal](https://hackernoon.com/@viniciusdacal)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/structuring-projects-and-naming-components-in-react.md](https://github.com/dawn-teams/translate/blob/master/articles/structuring-projects-and-naming-components-in-react.md)
> * 译者：[也树](https://github.com/xdlrt)
> * 校对者：[]()

# React 项目结构和组件命名之道

React 作为一个库，不会决定你如何组织项目的结构。这是件好事，因为这样我们有了充分的自由去尝试不同的组织方式并且选取最适合我们的方式。但是从另一个角度讲，这可能会让刚刚上手 React 的开发者产生些许困惑。

我将会在本文为大家展示我已经使用过一段时间并且效果不错的方式，这些方式没有通过重新造轮子来实现，而是通过将社区中的方案组合和提炼得到。

> 注意：这里没有什么是绝对正确的！你可以选择你认为容易理解，并且可以适应或可以改造成适应你的情景的方式。

## 目录结构

我最常见到的一个问题就是如何组织文件和目录结构。在本文中，我们假设你有一个 `create-react-app` 生成的最简单的目录结构。

 `create-react-app` 为我们生成了一个基础的项目，包含根目录还有诸如`.gitignore`, `package.json`, `README.md`, `yarn.lock` 的文件。

它还生成了 `public` 和 `src` 目录，`src` 目录就是我们存放源代码的目录。

看一下下面图片所描述的结构：

![structure](https://img.alicdn.com/tfs/TB1GhuvdZfpK1RjSZFOXXa6nFXa-400-479.png)

本文我们只会关注 `src` 目录，所有在它之外的都会保持不变。

## 容器和组件

我们可以看到在 `src` 目录下有 containers 目录和 components 目录：


```
src
├─ components 
└─ containers
```

但是这个方式会导致下面这些问题：
- **主观的规则** 你不清楚什么是容器而什么是组件，这两者的差异是主观定义的。如果在你的团队里去推行，让所有的开发者能够相同地赞成和评判这两者是非常困难的。
- **没有考虑组件的变化** 即使你决定了一个组件适用于某种特定的种类，在项目的周期内很容易发生变化，最终迫使你把它从 `components` 挪到 `containers` 目录下，反之亦然。
- **允许两个组件使用同一个名字** 组件在应用中的命名应该是声明式且唯一的，从而避免对相同命名组件的职责产生困惑。但上面的方式为两个组件可以拥有相同命名打开了一个缺口，一个可以是容器，另一个可以是展示型组件。
- **效率低下** 即使你在实现一个独立特性时，也不得不经常在 containers 和 components 目录下来回切换，因为一个独立特性有两种不同类型的组件是再正常不过的事情了。

有一种基于这种方式的变种方式，在模块的目录下保持着两个目录的分离。

想象一下在你的应用中有一个 User 模块，在此模块下，你有两个目录去分离你的组件：

```
src
└─ User
  ├─ components
  └─ containers
```

上述方式最小化了在两个遥远目录下不断切换的的问题，但是同样增加了很多烦恼。当你的应用有非常多模块的时候，你最终会可能会创建几十个 `containers` 和 `components` 目录。

所以我们讨论如何组织目录和文件的时候，和组件是否被拆分为展示型和容器型是无关的。也就是说，我们会把所有的组件都放在 `components` 目录下，除了 screens。

> 即使在目录上拆分它们是不必要的，了解它们之间的差异性依然是有必要的。如果你对这个话题还有疑问，建议阅读这篇文章：[Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)。

## 拆分和组合代码

在 `components` 目录下，我们通过模块/特性(module/feature)的结构来组织文件。

在对用户进行增删改查的过程中，我们只会有一个 User 模块。所以我们的目录结构会像下面这样：

```
src
└─ components
  └─ User
    ├─ Form.jsx
    └─ List.jsx
```

每当一个组件会有不止一个文件的时候，我们会将这个组件和它对应的文件放在同一个文件夹下，并且使用同一个名字来命名。举个例子：现在我们有一个 `Form.css` 文件包含了 `Form.jsx` 的样式，这时我们的目录结构会像这样：

```
src
└─ components
  └─ User
    ├─ Form
    │ ├─ Form.jsx
    │ └─ Form.css
    └─ List.jsx
```

> 测试用的文件和被测试的文件放在一起，在上面这个例子中，`Form.jsx` 的测试文件会放在同一个文件夹下并且命名为 `Form.spec.jsx`

## UI 组件

除了通过模块拆分组件，我们还会在 `src/components` 放置一个 `UI` 目录，用于存放所有通用的组件。

UI 组件不属于任何一个模块，需要足够通用。它们应该可以直接放在开源库中，因为它们不包含任何特定应用的业务逻辑。常见的这类组件有：按钮，输入框，复选框，下拉选择，模态框，数据可视化组件等等。

# 组件命名

以上我们了解了如何组织目录结构和如何通过模块来拆分我们的组件，但是还有一个问题：如何命名它们？

这里我们说的是如何命名我们的 class 或者定义组件的常量。

```
class MyComponent extends Component {}
const MyComponent = () => {};
```

组件的命名在应用中应当清晰且唯一，这样可以让它们可以轻松被找到并且避免可能的困惑。

当应用在运行时发生错误或者通过 React 开发者工具调试时，组件的名字是非常方便易用的，因为错误发生的地方往往都伴随着组件的名字。

我们采用基于路径的组件命名方式，即根据相对于 `components` 文件目录的相对路径来命名，如果在此文件夹以外，则使用相对于 `src` 目录的路径。举个例子，组件的路径如果是 `components/User/List.jsx`，那么它就被命名为 `UserList`。

如果文件名和文件目录名相同，我们不需要重复这个名字。也就是说，`components/User/Form/Form.jsx` 会命名为 `UserForm` 而不是 `UserFormForm`。

这样的命名方式有以下几点好处：

## 便于在项目中搜索文件

如果你的编辑器支持模糊搜索，只需要搜索 `UserForm` 就可以让你找到对应的文件：

![](https://img.alicdn.com/tfs/TB1Y7Kwd5rpK1RjSZFhXXXSdXXa-1600-455.png)

如果你想要在目录树中搜索文件，可以很容易地通过组件的名字定位到它：

![](https://img.alicdn.com/tfs/TB15F9vdYvpK1RjSZPiXXbmwXXa-400-528.png)

## 可以避免在引入时重复名称

遵循这种方式，你可以根据组件的上下文环境来命名文件。想一下上面的 form 组件，我们知道它是一个 User 模块下的 form 组件，但是既然我们已经把 form 组件放在了 User 模块的目录下，我们就不需要在 form 组件的文件名上重复 user 这个单词，使用 `Form.jsx` 就可以了。

我最初使用 React 的时候喜欢用完整的名字来命名文件，但是这样会导致相同的部分重复太多次，同时引入时的路径太长。来看看这两种方式的区别：

```js
import ScreensUserForm from './screens/User/UserForm';
// vs
import ScreensUserForm from './screens/User/Form';
```

在上面的例子中，我们看不出来明显的优势。但是应用复杂度上升一点时就能够看到区别了。我们来看看下面这个我实际项目中的例子：

```js
import MediaPlanViewChannel from '/MediaPlan/MediaPlanView/MediaPlanViewChannel.jsx';
// vs
import MediaPlanViewChannel from './MediaPlan/View/Channel';
```

现在想象一下一个文件名中重复五到十次。

出于这样的原因，我们认为根据组件文件的上下文环境以及它的相对路径来命名是更好的方式。

## 页面（Screen）

如果要对一个用户做增删改查的操作，我们需要有用户列表页面，创建新用户的页面以及编辑已有用户的页面。

在应用中，通过使用组件相互组合的结果，就是一个页面。理想状态下，页面应该不包含任何逻辑，而仅仅是一个函数式组件。

我们以 `src` 目录为根目录，将不同页面分散在不同文件夹中。因为它们是根据路由定义而不是模块来划分成组的。

```
src
├─ components 
└─ screens
  └─ User
    ├─ Form.jsx
    └─ List.jsx
```

假设我们项目中在使用 react-router，我们在 screens 目录下放置 Root.jsx 文件，并且在其中定义我们应用所有的路由。

Root.jsx 的代码可能像下面这样：

```js
import React, { Component } from 'react';
import { Router } from 'react-router';
import { Redirect, Route, Switch } from 'react-router-dom';

import ScreensUserForm from './User/Form';
import ScreensUserList from './User/List';

const ScreensRoot = () => (
  <Router>
    <Switch>
      <Route path="/user/list" component={ScreensUserList} />
      <Route path="/user/create" component={ScreensUserForm} />
      <Route path="/user/:id" component={ScreensUserForm} />
    </Switch>
  </Router>
);

export default ScreensRoot;
```

注意我们将所有页面都放在同一个目录下，这个目录以路由名称命名。尝试为每个父级路由建立一个目录，在这个目录中组织子路由。在这个示例中，我们创建了 User 目录并且将 List 页面和 Form 页面放在里面。这种方式使你看一眼 url 就能够轻松定位当前路由渲染的页面。

像上面的例子中的创建和编辑一个用户的路由一样，一个页面可能会被两个不同的路由渲染使用。

你可能注意到了所有的组件都包含 Screen 作为名称的前缀。当组件在组件目录外使用时，我们需要使用它们相对于 src 目录的路径来命名。位于 `src/screens/User/List.jsx` 的组件应该被命名为 ScreensUserList。

包括 Root.jsx 在内，我们的目录结构如下：

```
src
├─ components 
└─ screens
  ├─ User
  │ ├─ Form.jsx
  │ └─ List.jsx
  └─ Root.jsx
```

> 别忘了在 index.js 中引入作为应用根组件的 Root.jsx 。

如果你对一个页面长什么样子还有疑问，看看下面的示例，它就是用户表单的页面。

```js
import React from 'react';
import UserForm from '../../components/User/Form/Form';

const ScreensUserForm = ({ match: { params } }) => (
  <div>
    <h1>
      {`${!params.id ? 'Create' : 'Update'}`} User
    </h1>
    <UserForm id={params.id} />
  </div>
);

export default ScreensUserForm;
```

最终，我们应用的目录结构会像下面这样：

```
src
├─ components 
│  ├─ User
│  │ ├─ Form
│  │ │ ├─ Form.jsx
│  │ │ └─ Form.css
│  │ └─ List.jsx
│  └─ UI 
│
└─ screens
  ├─ User
  │ ├─ Form.jsx
  │ └─ List.jsx
  └─ Root.jsx
```

## 回顾要点

- 展示型和容器组件放在 src/components 目录下
- 通过模块/特性（module/feature）的方式组织组件
- 基础的 UI 组件放在 src/components/UI 目录下
- 保持页面简单，使用最简洁的结构和代码
- 通过路由定义组织页面。对于 /user/list 路由地址来说，我们会有一个页面在 /src/screens/User/List.jsx。
- 组件由相对 components 或 src 的路径命名，就是说，处于 src/components/User/List.jsx 位置的组件会被命名为 UserList。处于 src/screens/User/List.jsx 位置的组件会被命名为 ScreensUserList
- 组件和目录同名时，不要在使用组件的时候重复这个名字。考虑这样一个场景，处于 src/components/User/List/List.jsx 位置的组件会被命名为 UserList 而不是 UserListList。
