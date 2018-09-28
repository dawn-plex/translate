> * 原文地址：[structuring projects and naming components in react](https://hackernoon.com/structuring-projects-and-naming-components-in-react-1261b6e18d76)
> * 原文作者：[Vinicius Dacal](https://hackernoon.com/@viniciusdacal)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/structuring-projects-and-naming-components-in-react.md](https://github.com/dawn-teams/translate/blob/master/articles/structuring-projects-and-naming-components-in-react.md)
> * 译者：[也树](https://github.com/xdlrt)
> * 校对者：[]()

# Structuring projects and naming components in React

As React is just a lib, it doesn’t dictate rules about how you should organize and structure your projects. This is nice, because it gives us freedom to try different approaches and adapt the ones that better fit for us. On the other hand, this could cause some confusion for devs that are starting in React world.

In this post, I’m going to show some approaches that I have been using for a while and have been scaling very well. These approaches doesn’t re-create the wheel, they just put together and refine what we have on the market.

> Remember: Nothing here is written in rocks! You can take only the approaches you think that make sense and adapt/change it to fit in your context.

## Folder Structure

One of the questions I often see is regarding to how structure files and folders. In this post, we are considering you have a minimum structure, as the one created with `create-react-app`.

The `create-react-app` generates a basic project for us, containing in its root, the files: `.gitignore`, `package.json`, `README.md`, `yarn.lock`

It also generates the folders: `public` and `src`11. The last one is where we keep our source code.

Take a look the image below, with the described structure:

![structure](https://img.alicdn.com/tfs/TB1GhuvdZfpK1RjSZFOXXa6nFXa-400-479.png)

In this post, we are going to focus on src folder. Everything that is outside that, will remain intact.

## Containers and Components

It’s likely that you already seen the separation between Containers and Presentation Components in the some project’s root folder. I mean, inside src, you have a folder named components and another folder named containers:

```
src
├─ components 
└─ containers
```

However, this kind of approach has some issues, as you can see below:

- **Subjectives rules** - You don’t have clear rules about what is a Container and what is a Presentational Component. The difference between each other can be subjective and when you are in a team, it will be hard to have all the developers agreeing and judging equally this matter.
- **It doesn’t consider the dynamism of the components** - Even when you decide a component fits in one of the specific types, it’s easy to get it changed along the project lifetime, becoming from another type and forcing you to move it from `components` to `containers` folder and vice-versa.
- **Allow two components with the same name** - Components should have declarative and unique names in the application, to avoid confusion about the responsibility of each one. However, the above approach opens a breach to having two components with the same name, one being a container and other being a presentational.
- **Productivity loss** - You have to constantly navigate between containers and components folders, even when working in a single feature. Because it’s common that a single feature has components of the two types.
There’s also a variation of that approach, that keeps this separation, but inside the modules.

Imagine that inside your application, you have the module User. Inside it, you would have two folders to separate your components:

```
src
└─ User
  ├─ components
  └─ containers
```

The above approach, minimize the problem about navigating between distant folders in the project tree. However, it adds a lot of noise. Depending on how many modules your application has, you would end up with dozens of containers and components folders.

For those reasons, when we are talking about organizing folders and files, it’s irrelevant to split our components by the concept of presentational vs container. That said, we are going to keep all our components inside the `components` folder, except the screens.

> Even being irrelevant to separate them in folders, it’s important to know the conceptual difference between one and other. If you still have questions about this subject, I suggest you to read the article: [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0).

## Separating and grouping the code

Inside the `components` folder, we group the files by module/feature.

In a CRUD of user, we would have only the module User. So, our structure would be as the following:

```
src
└─ components
  └─ User
    ├─ Form.jsx
    └─ List.jsx
```

When a component is composed by more than one file, we put this component and its files in a folder with the same name. Eg: Let’s say you have a Form.css containing the Form.jsx’s styles. In this case, your structure would be like that:

```
src
└─ components
  └─ User
    ├─ Form
    │ ├─ Form.jsx
    │ └─ Form.css
    └─ List.jsx
```

> The test files stay with the file that is being tested. In the above case, the test for Form.jsx would stay in its same folder and would be named as Form.spec.jsx

## UI Components

Beyond separating the components by modules, we also include a `UI` folder inside `src/components`, to keep all our generic components in it.

UI Components are components generic enough to not belong to a module. They are components that you could keep in an open source lib, because they don’t have any business logic from the specific application. Examples of those components are: Buttons, Inputs, Checkboxes, Selects, Modals, Data display elements, etc…

# Naming components

Above we saw how to structure folders and separate our components by modules. However, there’s still a question: How to name them?

> When we talk about naming a component, it’s regarding to the name we give to the class or to the const that defines the component:
```
class MyComponent extends Component {
}
const MyComponent () => {};
```


As mentioned before, the name we give to the components, should be clear and unique in the application, in order to make them being easier to find and to avoid possible confusions.

A component’s name is very handy when we need to debug using tools as React Dev Tools, and when run time errors happen in the application. The error always come with the component name where it happened.

To name the components, we follow the pattern path-based-component-naming, which consists in naming the component accordingly to its relative path to the folders `components` or to `src`, in the case you are outside `components` folder. Basically, a component that is located at: `components/User/List.jsx` would be named as `UserList`.

When the file is inside a folder with same name, we don’t need to repeat the name. That said, `components/User/Form/Form.jsx`, would be named as `UserForm` and not as `UserFormForm`.

The above pattern has some benefits we can see below:

## Facilitate the search for the file inside the project

If you editor has support to fuzzy search, just searching for the name `UserForm` would make you find the correct file:

![](https://img.alicdn.com/tfs/TB1Y7Kwd5rpK1RjSZFhXXXSdXXa-1600-455.png)

If you want to search the file on the folder tree, you can easily find it just orienting yourself by the component name:

![](https://img.alicdn.com/tfs/TB15F9vdYvpK1RjSZPiXXbmwXXa-400-528.png)

## Avoid repeating names on the imports

Following the pattern, you will always name the file accordingly to its context. Considering the above form, we know it is a user form, but as we are already inside the User folder, we don’t need to repeat that word in the component file name. So, we name it only as Form.jsx

When I started working with React, I used to put the full name in the file. However, that makes you repeat a name a lot of times and the importation path becomes too big. Take a look at the difference between the approaches:

```js
import ScreensUserForm from './screens/User/UserForm';
// vs
import ScreensUserForm from './screens/User/Form';
```

In the above example, you can’t see the advantage from one approach to other. But the application growing a little, it’s possible to see the difference. Take a look in the example below, considering a component from the project that I work on:

```js
import MediaPlanViewChannel from '/MediaPlan/MediaPlanView/MediaPlanViewChannel.jsx';
// vs
import MediaPlanViewChannel from './MediaPlan/View/Channel';
```

Now imagine this multiplied by 5 or 10 times in a single file.

For that reasons, we always name the file accordingly to its context and the component accordingly to its relative location to components or src folder.

## Screens

Screens, as the name already describes, would be the screens that we have in the application.

In a CRUD of users, we would have a screen for the user list, a screen for create a new user and a screen for edit an existent user.

A screen is where you use components to compose a page to the application. Ideally, the screen would’t contain any logic and would be a functional component.

We keep the screens in a separated folder in the root of src, because they will be grouped accordingly to the route definition and not by modules:

```
src
├─ components 
└─ screens
  └─ User
    ├─ Form.jsx
    └─ List.jsx
```

Considering that the project is using react-router, we keep the file Root.jsx inside the screens folder, and we define in it all the application routes.

The code for Root.jsx would be similar to this:

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

Notice that we put all the screens inside a folder with the same name of the route, `user/ -> User/`. Try to keep a folder for each parent route, and group the sub-routes in it. In this case, we created the folder User and we keep the screens List and screen Form in it. This pattern will help you find easily which screen is rendering each route, by just taking a look at the url.

A single screen could be used to render two different routes, as we did above with the routes for creating and editing an user.

You may notice that all the components contain the Screen as a prefix in it’s name. When the component is located outside the components folder, we should name it accordingly to its relative path to src folder. A component located at `src/screens/User/List.jsx` should be named as ScreensUserList.

With the Root.jsx created, our structure would be the following:

```
src
├─ components 
└─ screens
  ├─ User
  │ ├─ Form.jsx
  │ └─ List.jsx
  └─ Root.jsx
```

> Don’t forget to import Root.jsx inside index.js to be the application root component.

In case you still have a doubt about how a screen should look like, take a look at the example below, for what would be the screen for the user form.

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

Finally, our application would be structured like that:

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

## Recapping

- Presentational and Container components are kept at src/components
- Group components by module/feature.
- Keep generic components inside src/components/UI
- Keep screens simple, with minimum structure and code.
- Group screens accordingly to route definition. For a route /user/list we would have a screen located at /src/screens/User/List.jsx.
- Components are named accordingly to its relative path to components or src. Given that, a component located at src/components/User/List.jsx would be named as UserList. A component located at src/screens/User/List would be named as ScreensUserList.
- Components that are in a folder with same name, don’t repeat the name in the component. Considering that, a component located at src/components/User/List/List.jsx would be named as UserList and NOT as UserListList.

## Conclusion

The above tips cover only a piece from organization and structure of a project. I tried to keep it only about React and leave Redux for a future post.

How about you? Do you have some approach that you would like to share with us? Write an answer below, I would love to read that!

Did you enjoy the read? Help us spread the word by giving a like and sharing

Don’t forget to follow me, to receive notifications about future posts!