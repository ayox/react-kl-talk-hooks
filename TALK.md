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

## useRef 
a hook to hold a mutable value thorughout renders. It doesn't change when component re-render. 

Yes! The useRef() Hook isn’t just for DOM refs. The “ref” object is a generic container whose current property is mutable and can hold any value, similar to an instance property on a class.

useRef does not accept a special function overload like useState. Instead, you can write your own function that creates and sets it lazily:



### examples 

``` js
import { useEffect, useRef } from 'react';

/**
 * Tracks previous state of a value.
 *
 * @param value Props, state or any other calculated value.
 * @returns Value from the previous render of the enclosing component.
 *
 * @example
 * function Component() {
 *   const [count, setCount] = useState(0);
 *   const prevCount = usePrevious(count);
 *   // ...
 *   return `Now: ${count}, before: ${prevCount}`;
 * }
 */
export default function usePrevious<T>(value: T): T | undefined {
  // Source: https://reactjs.org/docs/hooks-faq.html#how-to-get-the-previous-props-or-state
  const ref = useRef<T>();
  useEffect(() => {
    ref.current = value;
  });
  return ref.current;
}
```
```js
* @example
 * function Component() {
 *   const ref = useRef<HTMLElement>(null);
 *   const [width, height] = useSize(ref);
 *   // ...
 *   return <ElementToObserve ref={ref} />;
 * }
 */
export default function useSize(
  ref: React.RefObject<HTMLElement>,
  ResizeObserverOverride?: typeof ResizeObserver,
): readonly [number, number] {
  const [size, setSize] = useState<readonly [number, number]>([0, 0]);

  useEffect(() => {
    const ResizeObserver = ResizeObserverOverride || window.ResizeObserver;
    if (!ResizeObserver || !ref.current) return undefined;

    const observer = new ResizeObserver(([entry]) => {
      const { width, height } = entry.contentRect;
      setSize([width, height]);
    });
    observer.observe(ref.current);

    return (): void => {
      observer.disconnect();
    };
  }, [ResizeObserverOverride, ref]);

  return size;
} 
```
https://github.com/kripod/react-hooks/blob/9b791e63f18209546deb16a08ef7c15b3ad65842/packages/state-hooks/src/useUndoable.ts 

https://github.com/kripod/react-hooks/blob/ffcbf42af0b6ca8faa5deff56ba47f7ce51d14aa/packages/state-hooks/src/useTimeline.ts


## useCallback 
referential equality

Since javascript compares equality by reference

Function are just objects with ability to call. 

A strong use-case here to avoid child component re-renders.

The useCallback Hook lets you keep the same callback reference between re-renders so that shouldComponentUpdate continues to work:
Pass an inline callback and an array of dependencies. useCallback will return a memoized version of the callback that only changes if one of the dependencies has changed. This is useful when passing callbacks to optimized child components that rely on reference equality to prevent unnecessary renders (e.g. shouldComponentUpdate).

useCallback(fn, deps) is equivalent to useMemo(() => fn, deps).
```js


function Foo({bar, baz}) {
  const options = {bar, baz}
  React.useEffect(() => {
    buzz(options)
  }, [options]) // we want this to re-run if bar or baz change
  return <div>foobar</div>
}
function Blub() {
  return <Foo bar="bar value" baz={3} />
}
```

The reason this is problematic is because useEffect is going to do a referential equality check on options between every render, and thanks to the way JavaScript works, options will be new every time so when React tests whether options changed between renders it'll always evaluate to true, meaning the useEffect callback will be called after every render rather than only when bar and baz change.

There are two things we can do to fix this:
```js
// option 1
function Foo({bar, baz}) {
  React.useEffect(() => {
    const options = {bar, baz}
    buzz(options)
  }, [bar, baz]) // we want this to re-run if bar or baz change
  return <div>foobar</div>
}
```

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

a use case where useCallback should be used instead of useRef : 
https://codesandbox.io/s/818zzk8m78

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

## useImperativeHandle
useImperativeHandle customizes the instance value that is exposed to parent components when using ref


useImperativeHandle is very similar, but it lets you do two things:

- It gives you control over the value that is returned. Instead of returning the instance element, you explicitly state what the return value will be (see snippet below).
- It allows you to replace native functions (such as blur, focus, etc) with functions of your own, thus allowing side-effects to the normal behavior, or a different behavior altogether. Though, you can call the function whatever you like.

### Examples 

```js 
const MyInput = React.forwardRef((props, ref) => {
  const [val, setVal] = React.useState('');
  const inputRef = React.useRef();

  React.useImperativeHandle(ref, () => ({
    blur: () => {
      document.title = val;
      inputRef.current.blur();
    }
  }));

  return (
    <input
      ref={inputRef}
      val={val}
      onChange={e => setVal(e.target.value)}
      {...props}
    />
  );
});

const App = () => {
  const ref = React.useRef(null);
  const onBlur = () => {
    console.log(ref.current); // Only contains one property!
    ref.current.blur();
  };

  return <MyInput ref={ref} onBlur={onBlur} />;
};

ReactDOM.render(<App />, document.getElementById("app"));
```

## useLayoutEffect

https://stackoverflow.com/questions/57005663/when-to-use-useimperativehandle-uselayouteffect-and-usedebugvalue


## useDebugValue
Sometimes you might want to debug certain values or properties, but doing so might require expensive operations which might impact performance.

useDebugValue is only called when the React DevTools are open and the related hook is inspected, preventing any impact on performance.

useDebugValue can be used to display a label for custom hooks in React DevTools.



## Pitfalls 


Each Render Has Its Own Event Handlers
So far so good. What about event handlers?

Look at this example. It shows an alert with the count after three seconds:
```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}
```
Let’s say I do this sequence of steps:

Increment the counter to 3
Press “Show alert”
Increment it to 5 before the timeout fires
Counter demo

What do you expect the alert to show? Will it show 5 — which is the counter state at the time of the alert? Or will it show 3 — the state when I clicked?

``` jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setTimeout(() => {
      console.log(`You clicked ${count} times`);
    }, 3000);
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```