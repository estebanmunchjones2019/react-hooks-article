# State management with React custom hooks

Learn how to use React custom hooks 🪝to manage global state across the app **without** the need of  the`Context API` or libraries like Redux or MobX 🤯.

This is not a boring theory tutorial, it's a hands on one 💪, so we're gonna build this demo app: demo-app-link.

This is the gitHub repo with all the code: code-repo-url

The content of this tutorial is based on this course: [React 16: The Complete Course (incl. React Router 4 &amp; Redux) | Udemy](https://www.udemy.com/course/react-the-complete-guide-incl-redux/?couponCode=D_0322)

👉 Big thanks to Gonzalo Aguirre [@_gonaguirre_](https://twitter.com/_gonaguirre), from Underscope ([Underscope - We deliver world-class mobile apps using React Native](https://underscope.io/)), a React Native company based in South America, for reviewing this tutorial.

Table of contents:

**Intro**:

- What is global state?

- Is Context API the solution?

- What are hooks?

**Understanding custom hooks**:

- Let's build our first hook

- A more advanced hook

- Scope of state

**Building a basic global store**

- how the store should work?rules

- Storing function pointers

- Notifying subscribed components

- Version with `useState`

- Version with `useReducer`.

**Improving the stores**

- Separating reusable code

**Bonus**

- Why not to use the Context API for state managment (explain memo is not for free, slow API).

- Asynchronous code (mimic Redux thunk).?

- Keep an eye on Recoil

## What is global state?

In the React world the UI part of apps are made up of components, which are small units of code that render a view, and all of them are part of what is called a component tree.

Component tree chart here. ❌

What if we'd like to access some piece of data on different parts of the app? We'd be forced to keep state in a parent component that wraps the interested parts of the component tree.

Then, we could pass down the data via props to the interested parts, but that would lead to prop drilling. 

But hang on, what is prop drilling? Is when the same prop is passed through a long chain of components, making it repetitive and difficult to maintain.

Prop drilling chart here ❌

### Is Context API the solution?

There's a solution that is widely adopted in the React community to fix the propr drilling issue, and is the usage of the built in `Context API`, but it presents these 2 downsides:

- It's not meant and optimised for passing down high frequency changing data, like the `isFavourite` boolean property of a product item in an ecommerce app, but was meant for passing down more static things like `theme` variables, `login status`, `language`, and so on.
- After any of the data passed down via props through the Context API changes, all the components wrapped by the Provider that uses useContext will re-render, no matter if they use that specific piece of data or not. That could be patched by using the `useMemo` hook, but using that function is not for free, and will slow the performance of component tree re-rendering cycle and bloat your code.

When I say frequency, I mean a property changing at least twice in the lifecycle of the app. Usually, those changes are triggered by a user input, like the mentioned example of clicking a heart icon on the product card, to mark it is as favourite.

So, to anwser the question `Is Context API the solution?` It might not.

To sum it up, using Context is a way to avoid prop drilling, but it's still keeping state **inside** a React component, that is part of the **component tree**, and that way, the architecture is tied to that limitation of always choosing a parent component to hold that state.

If you'r interested in seeing the usage of the `Context API` and it's limitations, check out this great article: https://kentcdodds.com/blog/how-to-use-react-context-effectively.

## State management solution libraries

This issue of keeping state in React without hitting the problem of prop drilling, has been address initialy by the [Redux](https://redux.js.org/) library, by keeping state **outside components**.

Storing the state of the app outisde the component tree was the thing that made this library the go-to management solution for React, in other words, decoupling the app state and the UI components.

How does Redux work work? A store is created and then components can subscribe to changes to it, and dispatch actions that modify that state. Other libraries need to be added on top of Redux to perform asynchronous operations (or called side effects) before updating the state, like [Redux-Saga](https://redux-saga.js.org/) and [Redux-Thunk](https://github.com/reduxjs/redux-thunk).

There other state management solutions out there as well, like [Zustand]([Zustand Documentation](https://docs.pmnd.rs/zustand/introduction)), [Jotai](https://jotai.org/) , [Recoil](https://recoiljs.org/) (still in beta), [Rematch](https://rematchjs.org/) and [MobX]([README · MobX](https://mobx.js.org/README.html)).

To have an idea of the usage of the mentioned libraries, check out the chart here [@rematch/core vs jotai vs mobx vs react-redux vs recoil vs zustand | npm trends](https://www.npmtrends.com/@rematch/core-vs-jotai-vs-mobx-vs-react-redux-vs-recoil-vs-zustand)

Using libraries could be a great idea, but in this tutorial, you'll learn how to use the built in tools React offers to solve our problems, which are `hooks` in this case, so you can, at the end of the article, have a better knowlegde or React basics.

So, the plan for this tutorial is to use a **React custom hook** to

 👉 **keep state outside of components** 👈 and **manage global state** with them.

## What are hooks?

If you know about hooks, feel free to jump to section ❌

Hooks are **functions** that start with the name `use` and then the name of something else, like `State`, giving the full name  `useState`, as an example.

There are:

- **built in hooks**, like `useState`, and `useEffect`, `useCallback`, etc, that come already built in inside the React library code and we can import them from there.
  They help updating the state and do things when some `state` or `props` change on functional components, among other things.
  `useState`, and` useEffect` made the full switch from class based to functional components possible.
  
  You can have a look at hooks here: https://reactjs.org/docs/hooks-intro.html

- **Custom hooks** that we can use in components and other custom hooks, and they're helpful to move stateful logic/side effects outisde functional components so it can be re-used and at the same time, make the components leaner.

There are certain rules when using these hooks:
![](./images/rules-of-hooks-01.jpg)


![](./images/rules-of-hooks-02.jpg)

In the coming sections, we'll take a look at 2 custom hooks to understand them in depth, all with code examples and repos.

## Let's build our first custom hook

What if we have two components, `Posts.js` and `Widget.js`, that need to show a list of posts from an API with different markup and amount of posts?

Post.js only displays the first 10 posts from the API.

```jsx
// Posts.js

import { useEffect, useState } from "react";
import React from 'react';

const Posts = props => {
    const [apiData, setApiData] = useState([]);

    useEffect(() => {
        const fetchData = async() =>{
          const rawData = await fetch("https://jsonplaceholder.typicode.com/posts");
          const data = await rawData.json();
          setApiData(data);
        }
        fetchData();
    // see if I get an error with empty dependancy array.
      // and if it works wihtout it.
    }, []);

  // then apiData is used in the template
  const posts = apiData.slice(0,9).map(item => {   
    return <li>{item.title}</li> <li>{item.body}</li>;
});
  return (
// React.fragments and title
    // add title here
      <ul>
          {posts}
      </ul>
  );
};

export default Posts;
```

And Widget.js displays just the first 3 posts from the list:

```jsx
// Widget.js

import { useEffect, useState } from "react";
import React from 'react';

const Widget = props => {
    const [apiData, setApiData] = useState([]);

    useEffect(() => {
        const fetchData = async() =>{
          const rawData = await fetch("https://jsonplaceholder.typicode.com/posts");
          const data = await rawData.json();
          setApiData(data);
        }
        fetchData();
    // see if I get an error with empty dependancy array.
      // and if it works wihtout it.
    }, []);

  // then apiData is used in the template to show only 3 posts
  const posts = apiData.slice(0,2).map(item => {   
    return <li>{item.title}</li>;
});
  return (
// React.fragments and title
    // change to <ol>
      <ul>
          {posts}
      </ul>
  );
};

export default Widget;
```

If we look at both components, we see a lot of duplication:

```jsx
// duplicated code
const [apiData, setApiData] = useState([]);

    useEffect(() => {
        const fetchData = async() =>{
          const rawData = await fetch("https://jsonplaceholder.typicode.com/posts");
          const data = await rawData.json();
          setApiData(data);
        }
        fetchData();
    // see if I get an error with empty dependancy array.
      // and if it works wihtout it.
    }, []);
```

 So **we need to outsource that repeated code into an function, outisde the components.**

The logic we need to abstract needs to call an API when the first render of `Posts.js` or `Widget.js` components happened, and then, when the asynchronous call to the API finished, the state of those component needs to get updated.

If we put all the logic into a normal function, like this:

```jsx
// WRONG APPROACH!!

export const duplicatedCode = () => {
const [apiData, setApiData] = useState([]);

    useEffect(() => {
        const fetchData = async() =>{
          const rawData = await fetch("https://jsonplaceholder.typicode.com/posts");
          const data = await rawData.json();
          setApiData(data);
        }
        fetchData();
    // see if I get an error with empty dependancy array.
      // and if it works wihtout it.
    }, []);

}
```

First, we would get an error saying: `error -here` ❌

The problem here is that the piece of logic we want to abstract contains built in hooks, like `useState` and `useEffec`, which are tied to the component lifecycle and state.

**The function has no way to know what's going on inside the components calling this hook (`Posts.js` and `Widget.js`)?** 

Even having those  `useEffect` and `useState` built in hooks inside that logic is not enough.

So, **the problem is the function needs to be more connected to what's going on inside the component.**

And that's what **custom hooks** solve! 🎉

Custom hooks can have any built in hooks inside it, like `useEffect` and `useState` (and also other custom hooks, like `useWhatever`) so every time the component that uses the hook re-renders, `useState` runs inside the the hook, and everytime we set up state inside the hook using `useState`, is the same as setting up state inside the component 🤯
👉 That's our connection problem solved! 👈

It can take you some time around to wrap your head around this idea 🥴, but over time it will become natural 😌.

Here is the custom hook called `useApi` that will solve our code duplication problem:

```jsx
// useApi.js

// import built in React hooks
import { useEffect, useState } from "react";

// this is the custom hook
const useApi = () => {
  const [apiData, setApiData] = useState([]);

  useEffect(() => {
      const fetchData = async() =>{
        const rawData = await fetch("https://jsonplaceholder.typicode.com/posts");
        const data = await rawData.json();
        setApiData(data);
      }
      fetchData();
  }, []);

  return apiData;
}

export default useApi;
```

Here is the code explained step by step:

1. We need to import the built in hooks `useEffect` and `useState` from the React library.

2. Then, we create a function (Yes! custom hooks are functions) BUT, we need to start its name with `use`.

3. We initialize the state as an empty arrayby typing: `const [apiData, setApiData] = useState([]);` and that's the same is initializing state inside the component `Posts.js` or `Widget.js`.

4. Once the component `Posts.js` or `Widget.js` has been rendered, the second time it re-renders, the `useEffect's` callback function is called again, and that's when the API is hit.
   
   As a side note, if we had another `useEffect` inside `Posts.js` for example, that function will be called **at the same time** as the `useEffect` inside `useApi`.

5. We return the `apiData` array, as we wanna use it for displaying some posts in the UI. The good thing is that when the variable `apiData` is updated inside `useApi` hook, that will trigger a re-render on `Posts.js` or `Widget.js` and the updated value of `useApi` will be reflected on the template!

6. Then we export the hook, so we can call it inside components.

💡Useful notes about the `useEffect` dependacy array:

1. the `useEffect` dependancies array doesn't have `setApiData` added to it as it's a built in function of React that never changes (guaranteed by React), so it's the same pointer on re-renders

2. If you have functions as a dependancy, make sure you wrap them with `useCallback` so the they are the same object when the component re-renders, like this:

   ````
   // useCallback approach
   
   const SomeComponent = () => {
   
     // when there are re-renders, someFunction will be the same JS pointer
   	const someFunction = 👉 useCallback(() => {
   		// function body here
   	},[]);
   	
   	useEffect(() => {
   		someFunction();
   		// some other code here
   	}, [👉someFunction])
   }
   ````

   ````
   // workaround
   
   const SomeComponent = () => {
   	useEffect(() => {
   	 // we define the function inside the useEffect hook
   	  const someFunction(){
   	    // function body here
   		}
   		someFunction();
   		// some other code here
   	}, [])
   }
   ````
   
   
   
   

### Let's consume the custom hook

Now that our `useApi` custom hook is ready, let's use it inside our components!

```jsx
// Posts.js

import React from 'react';
import useApi from '../../useApi-hook/useApi';

const Posts = props => {
    const apiData = 👉 useApi();

  const posts = apiData.slice(0,9).map(item => {   
    return <li>{item.title}</li> add body
});
  return (
      <ul>
// add title
          {posts}
      </ul>
  );
};

export default Posts;
```

```jsx
// Widget.js

import { useEffect, useState } from "react";
import React from 'react';

const Widget = props => {

 const apiData = 👉 useApi();

  const posts = apiData.slice(0,2).map(item => {   
    return <li>{item.title}</li>;
});
  return (
// React.fragments and title
      <ul>
          {posts}
      </ul>
  );
};

export default Widget;
```

The component looks much leaner, and many other components can use that custom hook to query for data, that's great!

The only downside is that for every component using the custom hook, a new request to the API is made, but that can be fixed, you'll see the solution by the end of this tutorial. One way to check for this it to check the `Network` tab, and we'll see as many `GET` requests to the API as many components using that hook are shown on the screen.

Another way to check how many times the fetch function was fired, is to add a `debugger`to the hook , like this:

```jsx
debbuger;
const rawData = await fetch("https://jsonplaceholder.typicode.com/posts");
```

and open the developer tools of your browser. We'll get the code stoped as many times as the number of components on the screen using that hook.



## A more complex hook example

There are more things hooks can do:

1. Get some arguments to configure it

2. Return objects, arrays, anything!

💡 Remember: hooks are functions, so they can take arguments and return any type of data

Let's image we have these two components, `App.js` and `NewTask.js` that connect to an API to read and create some tasks respectively:

```jsx
// App.js
// it fetches taks from the backend databse and displays them

import React, { useEffect, useState } from 'react';

import Tasks from './components/Tasks/Tasks';
import NewTask from './components/NewTask/NewTask';

function App() {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  const [tasks, setTasks] = useState([]);

  const fetchTasks = async (taskText) => {
    setIsLoading(true);
    setError(null);
    try {
      const response = await fetch(
        'https://react-http-6b4a6.firebaseio.com/tasks.json'
      );

      if (!response.ok) {
        throw new Error('Request failed!');
      }

      const data = await response.json();

      const loadedTasks = [];

      for (const taskKey in data) {
        loadedTasks.push({ id: taskKey, text: data[taskKey].text });
      }

      setTasks(loadedTasks);
    } catch (err) {
      setError(err.message || 'Something went wrong!');
    }
    setIsLoading(false);
  };

  useEffect(() => {
    fetchTasks();
  }, []);

  const taskAddHandler = (task) => {
    setTasks((prevTasks) => prevTasks.concat(task));
  };

  return (
    <React.Fragment>
      <NewTask onAddTask={taskAddHandler} />
      <Tasks
        items={tasks}
        loading={isLoading}
        error={error}
        onFetch={fetchTasks}
      />
    </React.Fragment>
  );
}

export default App;
```

```jsx
// NewTask.js
// it adds new tasks to the database in the backend
// and updates the App's component state `tasks`

import { useState } from 'react';

import Section from '../UI/Section';
import TaskForm from './TaskForm';

const NewTask = (props) => {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  const enterTaskHandler = async (taskText) => {
    setIsLoading(true);
    setError(null);
    try {
      const response = await fetch(
        'https://react-http-6b4a6.firebaseio.com/tasks.json',
        {
          method: 'POST',
          body: JSON.stringify({ text: taskText }),
          headers: {
            'Content-Type': 'application/json',
          },
        }
      );

      if (!response.ok) {
        throw new Error('Request failed!');
      }

      const data = await response.json();

      const generatedId = data.name; // firebase-specific => "name" contains generated id
      const createdTask = { id: generatedId, text: taskText };

      props.onAddTask(createdTask);
    } catch (err) {
      setError(err.message || 'Something went wrong!');
    }
    setIsLoading(false);
  };

  return (
    <Section>
      <TaskForm onEnterTask={enterTaskHandler} loading={isLoading} />
      {error && <p>{error}</p>}
    </Section>
  );
};

export default NewTask;
```

The code that's common to both components is:

```jsx
 setIsLoading(true);
    setError(null);
    try {
      const response = // GET || POST request here

      if (!response.ok) {
        throw new Error('Request failed!');
      }

      const data = await response.json();
      // do something with the data and update the app `tasks` state

    } catch (err) {
      setError(err.message || 'Something went wrong!');
    }
    setIsLoading(false);
```

The custom hook we can create to move this logic to looks like this:

```jsx
// use-http.js

import { useState, useCallback } from 'react';

const useHttp = () => {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  const sendRequest = useCallback(async (requestConfig, applyData) => {
    setIsLoading(true);
    setError(null);
    try {
      const response = await fetch(requestConfig.url, {
        method: requestConfig.method ? requestConfig.method : 'GET',
        headers: requestConfig.headers ? requestConfig.headers : {},
        body: requestConfig.body ? JSON.stringify(requestConfig.body) : null,
      });

      if (!response.ok) {
        throw new Error('Request failed!');
      }

      const data = await response.json();
      applyData(data);
    } catch (err) {
      setError(err.message || 'Something went wrong!');
    }
    setIsLoading(false);
  }, []);

  return {
    isLoading,
    error,
    sendRequest,
  };
};

export default useHttp;
```

The hook is returning these an object in this case, with 3 keys:

1. `isLoading` of type `boolean`

2. `error` of type `string` or `null`;

3. `sendRequest`, a function that calls the api, that can be called inside the component whenever it's suits it. The function takes two arguments:
   
   a. `requestConfig`, that configures the http call with the appropiate `url`, `method`, `headers` and `body`.
   
   b. A callback function called `applyData`, that can update the UI by changing the state of the app.

You might notice the `useCallback` built in hook here:

```jsx
 const sendRequest = useCallback(//more code here)
```

That is done to make sure the `sendRequest` functions is the same object on every component re-render,and that way just make the useEffect callback function run just once (like when using the good old `ComponentDidMount()` method in class based components).

```jsx
useEffect(() => {
   sendRequest
  }, [sendRequest]);
```

The above useCallback usage is done in case we wanna comply with the dependancy array standards (we could leave it empty, and it would work as well).

Here is how the components use this hook:

```jsx
// App.js

import React, { useEffect, useState } from 'react';

import Tasks from './components/Tasks/Tasks';
import NewTask from './components/NewTask/NewTask';
import useHttp from './hooks/use-http';

function App() {
  const [tasks, setTasks] = useState([]);

   👉 const { isLoading, error, sendRequest: fetchTasks } = useHttp();

  useEffect(() => {
    const transformTasks = (tasksObj) => {
      const loadedTasks = [];
	    // massage the API data
      for (const taskKey in tasksObj) {
        loadedTasks.push({ id: taskKey, text: tasksObj[taskKey].text });
      }
      // change state to reflect changes in the UI
      setTasks(loadedTasks);
    };

    fetchTasks(
      { url: 'https://react-http-6b4a6.firebaseio.com/tasks.json' },
      transformTasks
    );
  }, [fetchTasks]);

  const taskAddHandler = (task) => {
    setTasks((prevTasks) => prevTasks.concat(task));
  };

  return (
    <React.Fragment>
      <NewTask onAddTask={taskAddHandler} />
      <Tasks
        items={tasks}
        loading={isLoading}
        error={error}
        onFetch={fetchTasks}
      />
    </React.Fragment>
  );
}

export default App;
```

Side note: `const { isLoading, error, sendRequest: fetchTasks } = useHttp();` means that `sendRequest` is being renamed to `fetchTasks`.

```jsx
// NewTask.js

import Section from '../UI/Section';
import TaskForm from './TaskForm';
import useHttp from '../../hooks/use-http';

const NewTask = (props) => {
  👉 const { isLoading, error, sendRequest: sendTaskRequest } = useHttp();

  const createTask = (taskText, taskData) => {
    const generatedId = taskData.name; // firebase-specific => "name" contains generated id
    const createdTask = { id: generatedId, text: taskText };

    props.onAddTask(createdTask);
  };

  const enterTaskHandler = async (taskText) => {
    sendTaskRequest(
      {
        url: 'https://react-http-6b4a6.firebaseio.com/tasks.json',
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: { text: taskText },
      },
      createTask.bind(null, taskText)
    );
  };

  return (
    <Section>
      <TaskForm onEnterTask={enterTaskHandler} loading={isLoading} />
      {error && <p>{error}</p>}
    </Section>
  );
};

export default NewTask;
```

There's a tricky part here:

```jsx
 // NewTask.js
 // What a heck is this? 🤔
 const enterTaskHandler = async (taskText) => {
     ...
     createTask.bind(null, taskText);👈❓
     ...
 } 
```

Well, our `applyData` callback function expects only one argument `data`:

```jsx
// use-http.js

const sendRequest = useCallback(async (requestConfig, applyData) => {
    ...
    applyData(data); 👈 //expects just one argument
    ...
}
```

and the function `createTask` we pass as `applyData` takes two arguments `taskText` and  `taskData`

```jsx
// NewTask.js

const createTask = (taskText, taskData) => { 👈 // expects 2 arguments
    props.onAddTask(createdTask);
  };
```

So we've got a problem: how we can possibly pass an extra argument `taskText` to `applyData`??

The `.bind` method lets us pre-configure (not execute) the function, so it takes the extra parameter `taskText` we need.

From [Function.prototype.bind() - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind):
*The **`bind()`** method creates a new function that, when called, has its `this` keyword set to the provided value, with a given sequence of arguments preceding any provided when the new function is called.*

*The `bind()` function creates a new **bound function**, which is an *exotic function object* (a term from ECMAScript 2015) that wraps the original function object. Calling the bound function generally results in the execution of its wrapped function.

```jsx
createTask.bind(null, taskText)
```

The first argument is the context, which is `null`, beause we don't wanna change it, and the first argument is `taskText` which will be the extra argument.

To avoid the usage of this `bind()` method, the other option could be defining `createTask` inside `enterTaskHandler`, like this:

```jsx
// NewTask.js
import Section from '../UI/Section';
import TaskForm from './TaskForm';
import useHttp from '../../hooks/use-http';

const NewTask = (props) => {

 const { isLoading, error, sendRequest: sendTaskRequest } = useHttp();

 const enterTaskHandler = async (taskText) => {
👉 const createTask = (taskData) => {
    // taskText is defined in the scope of enterTaskHandler
    // I can safely use it.
    const generatedId = taskData.name; // firebase-specific => "name" contains generated id
    const createdTask = { id: generatedId, text: taskText };

    props.onAddTask(createdTask);
  };
    sendTaskRequest(
      {
        url: 'https://react-http-6b4a6.firebaseio.com/tasks.json',
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: { text: taskText },
      },
      👉 createTask // this a function pointer
    );
  };
return //some 
JSX here
```

If you've read to this point, congratulations! You have now a solid 💪 foundational knowledge of how hooks 🪝work, so let's move on the next section, where we'll explore what happens with state inside hooks.

There is an important question here:

👉 Is the state inside hooks shared between the components that use it ❓❓❓🤔

Let's find out! 🤓

## Scope of state

The above hooks doesn't return a function that can update the state because the idea was that the components read the state but don't change it, but this case is not always what we want.

Sometimes, we want to have state that can be read and also changed from different parts of the app. To see how hooks manage state, let's build a simple app that displays a count and has a button to increment and decrement it, from 2 different components.

useCounter example here.

When calling the built in `useState` or `useReducer` inside custom hooks, the state stored inside the hook is **different** for every component calling that hook, in other words: **scoped to the component/custom hook** using it.

That isn't very helpful if we want to have the same global scope and many parts of the app can change it.

When not to use a custom hook?

As a side note, custom hooks are not always required to abstract logic from components, as long as that logic doesn't involve any usage of built in hooks. That way, we could outsource the logic into a regular function, usually called helper functions, inside our code base.

## How the store should work?

The requisites of the store are:

1) There is one state that components can access.

2) When one components updates it, the rest should get the update value.

Without further a do, here is the magic formula. Don't worry if you don't understand it at first, we'll see each part in detail with explanations.

Explain each part of the hook step by step, pointing out that the useState and useEffect run at the same as if they were part of the component that called the hook.

We could abstract the API call into a custom hook:

Explain here the construction of the custom hook step by step

The hook will run the useEffect code at the same time the component would do, and the variable got from the useApi() call behaves like a normal variable returned from calling useState(), that's the magic of hooks: reusable code and slimmer components!🎉

Now that you undertand that the built in hooks useState and useEffect behave the same way as they were in the component that called the hook, you're ready for the next step: build the custom hook for the demo app!

## JS objects, the key

functions are objects in JS, and they were stored on the listeners array!

Bonus Pro tip when using useEffect: avoiding hooks hell.

Opinion on React and re-renders has the problem of functions, that they're new objects and that affects effects. Other framworks are more straight forward regarding triggering effects. 

The usage of useCallback to guarantee having a same object on successive re-renders is convoluted.

useMemo can be used to have the same object on re-renders

Solution: use as little external dependancies inside useEffects.

# See Max videos about custom hooks and the store.

(check React version of Max code).

things to explain:

1. usage of built in and custom hooks inside it

2. the return variables, usage of the array for more than one variable, (anything can be return in a function, right?).

3. the parameter pass to it to configure it (initial state, conditionals, etc)

4. Explain that with built in hooks, we're already passing arguments useState(0), useEffect(()=> someCodeHere, []);

5. add parameters to the dependacy useEffect array.

6. setting state inside the custom hook will trigger a re-render of the component that uses the hooks, because is the same as triggerin useState directly inside the component.

7. a function that is passed as useEffect dependancy array can cause infinite loops, as when the component re-renders, the pointer of that function is a new one (even though the functions looks the same in content). Solution use useCallback so the function is the same object on re-renders.

8. So, to call the api and prevent infinite calls of the useEffect (and not leaving the dependancy array empty) a function used there fetchData had to be pass there, so that function had to be wrapped around a useCallback hook, and as the useCallback dependancy array needed to be passed the dependancies, it was too complex to also wrap those dependancies in useCallback (for functions) and useMemo(for the http config object).

9. when passing fetch tasks as parameter to the custom hook, a pointer of the function is passed to it. Objects are copied by pointers.

10. Parameters of useCallback callback function are not considered to be included on the dependancy array, only external things that are not react functions (React guarantees that).

11. Mindblowing: pre-configuring functions with the .bind() method, to pass some extra argument when calling it, even though the code that exectutes is expecting fewer arguments.
