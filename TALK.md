# Hooks 

- [Basic Hooks](#basic-hooks)
  - [`useState`](#usestate)
  - [`useEffect`](#useeffect)
  - [`useContext`](#usecontext)
- [Additional Hooks](#additional-hooks)
  - [`useReducer`](#usereducer)
  - [`useCallback`](#usecallback)
  - [`useMemo`](#usememo)
  - [`useRef`](#useref)
  - [`useImperativeHandle`](#useimperativehandle)
  - [`useLayoutEffect`](#uselayouteffect)
  - [`useDebugValue`](#usedebugvalue)
## useState & useEffect 
all are familiar with these hooks, useState are used to create state variables that would trigger a render if value change. useEffect will fire every render except if the deps array values are not changed. 
## useCallback 
The useCallback Hook lets you keep the same callback reference between re-renders so that shouldComponentUpdate continues to work:
Pass an inline callback and an array of dependencies. useCallback will return a memoized version of the callback that only changes if one of the dependencies has changed. This is useful when passing callbacks to optimized child components that rely on reference equality to prevent unnecessary renders (e.g. shouldComponentUpdate).

useCallback(fn, deps) is equivalent to useMemo(() => fn, deps).

### examples of useCallback: 
```js 
function MeasureExample() {
  const [rect, ref] = useClientRect();
  return (
    <>
      <h1 ref={ref}>Hello, world</h1>
      {rect !== null &&
        <h2>The above header is {Math.round(rect.height)}px tall</h2>
      }
    </>
  );
}

function useClientRect() {
  const [rect, setRect] = useState(null);
  const ref = useCallback(node => {
    if (node !== null) {
      setRect(node.getBoundingClientRect());
    }
  }, []);
  return [rect, ref];
}

```


## useMemo

To recap: you should not use ‘useCallback’ and ‘useMemo’ for everything. ‘useMemo’ should be used for big data processing while ‘useCallback’ is a way to add more dependency to your code to avoid useless rendering.

## useReducer 
the formal definiton for useReducer is : 
```js 
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

### useReducer like setState method
First let's write some pseudo code:
```js 
//Somewhere in FUNCTION COMPONENT

const reducer = (prevState, updatedProperty) => ({
  ...prevState,
  ...updatedProperty,
});

const initState = {
  name: 'Bob',
  age: 25,
};

//initialize state with initState
const [state, setState] = useReducer(reducer, initState);

//update name value (like we do in class component!)
setState({ name: 'Alice' });

//updated state:
state = {
  name: 'Alice',
  age: 25,
};
```

This time it worked as expected. We've updated name value and didn't lose age property.

And now full working example:
```js
import React, { useReducer, useEffect } from 'react';

const reducer = (prevState, updatedProperty) => ({
  ...prevState,
  ...updatedProperty,
});

const initState = {
  name: 'Bob',
  age: 25,
  isLoading: true,
};

function App() {
  const [state, setState] = useReducer(reducer, initState);

  const handleOnChange = (e) => setState({ [e.target.name]: e.target.value });

  useEffect(() => {
    setState({ isLoading: false });
  }, []);

  const { name, age, isLoading } = state;

  return(
    <>
      {isLoading ? 'Loading...' : (
        <>
          <input type="text" name="name" value={name} onChange={handleOnChange} />
          <input type="text" name="age" value={age} onChange={handleOnChange} />
        </>
      )}
    </>
  );
}
```

## useContext
as the name suggets, it is used to access a context value within the provider. The way we used context before is like this : import React from "react";
import ReactDOM from "react-dom";
``` js
// Create a Context
const NumberContext = React.createContext();
// It returns an object with 2 values:
// { Provider, Consumer }

function App() {
  // Use the Provider to make a value available to all
  // children and grandchildren
  return (
    <NumberContext.Provider value={42}>
      <div>
        <Display />
      </div>
    </NumberContext.Provider>
  );
}

function Display() {
  // Use the Consumer to grab the value from context
  // Notice this component didn't get any props!
  return (
    <NumberContext.Consumer>
      {value => <div>The answer is {value}.</div>}
    </NumberContext.Consumer>
  );
}

ReactDOM.render(<App />, document.querySelector("#root"));
```

now we can use it like this : 
```js 
// import useContext (or we could write React.useContext)
import React, { useContext } from 'react';

// ...

function Display() {
  const value = useContext(NumberContext);
  return <div>The answer is {value}.</div>;
}

```


```js 
function HeaderBar() {
  return (
    <CurrentUser.Consumer>
      {user =>
        <Notifications.Consumer>
          {notifications =>
            <header>
              Welcome back, {user.name}!
              You have {notifications.length} notifications.
            </header>
          }
      }
    </CurrentUser.Consumer>
  );
}

```

```js 
function HeaderBar() {
  const user = useContext(CurrentUser);
  const notifications = useContext(Notifications);

  return (
    <header>
      Welcome back, {user.name}!
      You have {notifications.length} notifications.
    </header>
  );
}
```


### examples : 
#### How to avoid passing callbacks down? {#how-to-avoid-passing-callbacks-down}

We've found that most people don't enjoy manually passing callbacks through every level of a component tree. Even though it is more explicit, it can feel like a lot of "plumbing".

In large component trees, an alternative we recommend is to pass down a `dispatch` function from [`useReducer`](/docs/hooks-reference.html#usereducer) via context:

```js{4,5}
const TodosDispatch = React.createContext(null);

function TodosApp() {
  // Note: `dispatch` won't change between re-renders
  const [todos, dispatch] = useReducer(todosReducer);

  return (
    <TodosDispatch.Provider value={dispatch}>
      <DeepTree todos={todos} />
    </TodosDispatch.Provider>
  );
}
```

Any child in the tree inside `TodosApp` can use the `dispatch` function to pass actions up to `TodosApp`:

```js{2,3}
function DeepChild(props) {
  // If we want to perform an action, we can get dispatch from context.
  const dispatch = useContext(TodosDispatch);

  function handleClick() {
    dispatch({ type: 'add', text: 'hello' });
  }

  return (
    <button onClick={handleClick}>Add todo</button>
  );
}
```
