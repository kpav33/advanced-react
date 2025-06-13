# Chapter 1 – Introduction to Re-renders

- To prevent unnecessary rendering in a component tree, especially in apps with many nested components, you can use the `useMemo` and `useCallback` hooks. These help ensure only the parts of the code affected by a state change are re-rendered.
- **Component Lifecycle Stages:**

  - **Mounting**: React creates the component, initializes its state, runs its hooks, and appends elements to the DOM.
  - **Updating (Re-rendering)**: When state or props change, React reuses the existing component instance to update the DOM. This process is more lightweight than mounting.
  - **Unmounting**: React performs cleanup when a component is removed, such as removing event listeners or canceling timers.

- **Re-renders are triggered by state updates.** State is interactive data that persists through a component’s lifecycle. When it changes, React re-renders the component and re-executes relevant hooks.

- In the React component tree, **everything "down" from where a state update is triggered will re-render**. React does not re-render components “up” the tree.

- To affect a parent component from a child, you must **explicitly pass state update functions down** or pass the component itself as a function (i.e., function-as-a-child/component pattern).

- A component **will not re-render solely due to prop changes**, unless those prop values are tied to state.

- However, if a component is wrapped in `React.memo`, **React will check if props have changed before re-rendering**. If none have changed, React skips the re-render. If any prop changes, normal rendering proceeds.

- Use the **"move state down"** pattern—placing state in a lower-level component—to prevent unnecessary re-renders higher up the tree.

- Be mindful with **custom hooks**: although they abstract state logic, their internal state still resides in the component that uses the hook. So, any state change in the custom hook will cause the component to re-render.

- **Where you place your state matters.** For optimal performance, isolate state to the smallest possible component scope and avoid lifting state unnecessarily.

---

## Key Takeaways

- Re-rendering is how React updates components with new data.
- Without re-renders, there will be no interactivity in our apps.
- State update is the initial source of all re-renders.
- If a component's re-render is triggered, all nested components inside that component will be re-rendered.
- During the normal React re-renders cycle (without the use of memoization), props change doesn't matter: components will rerender even if they don't have any props.
- We can use the pattern known as "moving state down" to prevent unnecessary re-renders in big apps.
- State update in a hook will trigger the re-render of a component that uses this hook, even if the state itself is not used.
- In the case of hooks using other hooks, any state update within that chain of hooks will trigger the re-render of a component that uses the very first hook.

# Chapter 2 – Understanding Elements, Components, and Re-renders

- We can **pass components as props** to other components to render them. This approach allows these components to **avoid re-rendering** during state updates, as they are treated as static props. This pattern is commonly used when rendering children in React.

- **Components** are JavaScript functions that return React **Elements**, which React then converts into actual DOM elements.

- A **React Element** is a plain object that represents a part of the UI. The familiar JSX syntax (`<div>` or `<MyComponent />`) is just syntactic sugar for `React.createElement(...)`. Elements can represent either DOM nodes (like `div`) or custom components.

- During a re-render, React calls the component functions again. Based on the returned values, it constructs a **Virtual DOM tree**, also known as the **Fiber Tree**. In fact, React creates **two trees**:

  - One representing the UI **before** the update.
  - One representing the UI **after** the update.

  React compares these two trees using a process called the **Reconciliation Algorithm** (or diffing), to determine the minimal set of changes required in the real DOM.

- If the element object before and after the update is **exactly the same**, React will **skip re-rendering** the corresponding component and its children. “Exactly the same” is defined by using `Object.is(oldElement, newElement)`. React does **not** perform deep comparisons of objects.

- React supports **nesting composition** using the `children` prop. This pattern mirrors HTML nesting and improves code readability. Passing elements via `children` has the same re-rendering behavior and performance benefits as passing them via regular props.

---

## Key Takeaways

- A Component is just a function that accepts an argument (props) and returns Elements that should be rendered when this Component renders on the screen.
  `const A = () => <B />` is a Component.

- An Element is an object that describes what needs to be rendered on the screen, with the type either a string for DOM elements or a reference to a Component for components.
  `const b = <B />` is an Element.

- Re-render is just React calling the Component's function.

- A component re-renders when its element object changes, as determined by `Object.is` comparison of it before and after re-render.

- When elements are passed as props to a component, and this component triggers a re-render through a state update, elements that are passed as props won't re-render.

- `children` are just props and behave like any other prop when they are passed via JSX nesting syntax:

  ```jsx
  <Parent>
    <Child />
  </Parent>

  // is the same as:
  <Parent children={<Child />} />
  ```

# Chapter 3 - Configuration concerns with elements as props

- We can pass **React elements as props** to components to allow greater customization. Instead of exposing multiple props for styling and configuration, we delegate control to the consumer by letting them pass fully defined elements.

- Using the **element-as-prop pattern**, you signal to the consumer:  
  _“I'm responsible for placing this element in the layout, but you control its implementation.”_

- When conditionally rendering components, you may still declare their element props beforehand. This **does not cause a performance hit**—from React’s perspective, elements are lightweight objects in memory. They are only rendered if actually returned by a component during rendering.

- You can use the `React.cloneElement` function to **clone an element and assign new props** to it. This allows you to apply default configuration while still enabling the consumer to customize the element.

---

## Key Takeaways

- When a component renders another component, the configuration of which is controlled by props, we can pass the entire component element as a prop instead, leaving the configuration concerns to the consumer.

  ```jsx
  const Button = ({ icon }) => {
    return <button>Submit {icon}</button>;
  };

  // large red Error icon
  <Button icon={<Error color="red" size="large" />} />;
  ```

- If a component that has elements as props is rendered
  conditionally, then even if those elements are created outside of
  the condition, they will only be rendered when the conditional
  component is rendered.

  ```jsx
  const App = () => {
    // footer will be rendered only when the dialog itself renders
    // after isDialogOpen is set to "true"
    const footer = <Footer />;
    return isDialogOpen ? <ModalDialog footer={footer} /> : null;
  };
  ```

- If we need to provide default props to the element from props, we
  can use the `cloneElement` function for that.

- This pattern, however, is very fragile. It's too easy to make a
  mistake there, so use it only for very simple cases.

# Chapter 4 - Advanced configuration with render props

- A **render prop** is simply a function that returns a React **Element**. It’s similar to a component, but unlike components (which are called by React), **you call render functions directly** in your code.

- With render props, there’s no need to use `React.cloneElement`. Instead, you can pass an object to the render function and **spread its properties as props** inside the returned element.

- This pattern also allows for **state sharing**, since you can include state values in the object passed to the render function and apply them directly.

- One common approach is to use `children` as a function — often referred to as the **children-as-a-function** or **render props via children** pattern. This enables state sharing between components, but it can lead to **performance issues**, as every state change **triggers a re-render** of everything within the function.

- While this pattern was once common, today **hooks have replaced most use cases for render props**. You'll mainly encounter render props in **older codebases** or **specific scenarios** that require tightly controlled rendering or state-sharing logic.

---

## Key Takeaways

- If a component that has elements as props wants to have control  
   over the props of those elements or pass state to them, you'll need  
   to convert those elements into **render props**.

  ```jsx
  const Button = ({ renderIcon }) => {
    const [someState, setSomeState] = useState()
    const someProps = { ... };
    return <button>Submit {renderIcon(someProps, someState)}
  </button>;
  };

  <Button renderIcon={(props, state) => <IconComponent {...props}
  someProps={state} /> } />
  ```

- `children` can also be used as **render props**, even with JSX  
   nesting syntax.

  ```jsx
  const Parent = ({ children }) => {
    return children(somedata);
  };
  ```

- **Render props** were once the go-to pattern for sharing stateful  
  logic between components **without lifting state up**.

- But **hooks have replaced render props** in 99% of use cases.

- Render props can still be useful today for **specific cases**,  
  especially when stateful logic needs to be attached directly to  
  a DOM element.

# Chapter 5 – Memoization with useMemo, useCallback, and React.memo

- In JavaScript, **primitive values** (like strings, numbers, booleans) are compared by their **value**, while **objects** (including arrays and functions) are compared by their **reference**.

- When you create an object in JavaScript, the variable holds a reference to the memory location, not the actual object. So two objects with the same contents are still **not equal** unless they reference the same object in memory.

- React faces this challenge during **re-renders**. For instance, if you pass a freshly created `submit` function to a dependency array in a `useEffect`, the effect will **re-run on every render**, because the function reference is always new.

- To solve this, we use **memoization hooks** like `useMemo` and `useCallback` to preserve references between renders.

- `useCallback` is used to memoize **functions**. If the dependencies don’t change, the same function reference is returned between renders.

- `useMemo` is used to memoize **values**. You pass a function to `useMemo`, and it **returns the result** of that function. This is helpful for caching expensive computations.

- `useCallback(fn, deps)` is functionally equivalent to `useMemo(() => fn, deps)`.

- **Important distinction**: Neither `useCallback` nor `useMemo` **prevents** the function or value from being recreated if dependencies change.

- **Memoizing props** doesn't prevent a component from re-rendering unless:

  - The component is wrapped in `React.memo`.
  - The prop is used in another hook's dependency array that relies on stable references.

- `React.memo` is a higher-order component used to **memoize a component**. If the component’s parent re-renders, React will **only re-render this component if its props have changed**.

- For `React.memo` to work effectively:

  - The component's props need to be stable — which often requires memoizing them using `useMemo` or `useCallback`.
  - Avoid spreading props from parent components or custom hooks.
  - Avoid passing non-primitive values (like arrays, objects, or functions) unless they're memoized.

- If you're using `React.memo` with the **children prop**, remember that `children` is a **non-primitive value** and also needs to be memoized.

- While `useMemo` is often used for "expensive" calculations, be cautious:

  - Define what “expensive” means in your context.
  - Use **profiling tools** to measure performance impact.
  - Avoid overusing `useMemo` — excessive caching can **slow down initial renders** and add complexity.

- `useMemo` does **nothing** unless the component **actually re-renders**. On first render, React must still execute and **cache** the result, which uses some CPU and memory.

---

## Key Takeaways

- Does all of this mean we shouldn't use memoization? **Not at all.**  
  It can be a very valuable tool in our performance battle. But considering so many caveats and complexities that surround it, I would recommend using **composition-based optimization techniques first**.  
  `React.memo` should be the **last resort** when all other things have failed.

- Remember:

  - React compares **objects/arrays/functions by their reference**, not their value.  
    That comparison happens in **hooks' dependencies** and in **props** of components wrapped in `React.memo`.

  - The **inline function** passed as an argument to either `useMemo` or `useCallback` will be **re-created on every re-render**.

    - `useCallback` memoizes that function itself.
    - `useMemo` memoizes the result of the function’s execution.

  - **Memoizing props on a component makes sense only when**:

    - The component is wrapped in `React.memo`.
    - The component uses those props as **dependencies** in any of its hooks.
    - The component **passes those props down** to other components, and those components meet either of the above conditions.

  - If a component is wrapped in `React.memo` and its re-render is triggered by its parent, then React will **not re-render** this component if its **props haven't changed**. In any other case, re-render will proceed as usual.

  - Memoizing **all props** on a component wrapped in `React.memo` is **harder than it seems**.  
    Avoid passing non-primitive values that are **coming from other props or hooks**.

  - When memoizing props, **remember that `children` is also a non-primitive** value that needs to be memoized.

# Chapter 6 – Deep Dive into Diffing and Reconciliation

- When we create a React element, we are actually creating a JavaScript object. JSX is just syntactic sugar for the `React.createElement()` function. This function returns a description object with a `type` property that points to either:

  - a component,
  - a memoized component, or
  - a string representing an HTML tag.

- If the reference to that object changes between re-renders, and the `type` remains the same (and isn't memoized with `React.memo`), React will re-render the element.

- React transforms our code into DOM elements. For example, a React `input` becomes a standard HTML input element in the DOM. If only an attribute like `placeholder` changes, React updates just that attribute instead of re-creating the whole element. This is for performance optimization.

- This transformation is done by manipulating the **Virtual DOM** — a large JavaScript object that represents the entire component tree with all their props and children.

- DOM elements in the Virtual DOM tree have their `type` set as strings (e.g., `"div"`, `"input"`), which tells React to create corresponding HTML elements. Components have their `type` set to their function definition, allowing React to execute the function and traverse the tree it returns.

- On initial render (mount), React:

  1. Iterates over the tree.
  2. For elements with string `type`, generates DOM nodes.
  3. For elements with function `type`, calls the function and recursively processes its return value.
  4. Finally, appends the generated elements to the document using JavaScript's `appendChild()`.

- During re-renders, React starts traversing the tree from the point of update. It compares the old and new `type` values:

  - If the `type` is the same, the component is marked for update.
  - If the `type` has changed, React unmounts the old component and mounts the new one.

- Defining components inside other components is considered an anti-pattern. In JavaScript, functions are compared by reference. So defining a child component inside a parent function means the reference changes on every render, causing React to unmount and remount that component each time. This is inefficient and can introduce bugs.

- When conditionally rendering the same component (where only attributes change), React doesn't unmount and remount it. It simply re-renders it with the updated props. For example, an input element with a changed `placeholder` will maintain its current value; only the attribute will be updated.

- To force re-mounting instead of re-rendering, we can use arrays and keys.

- When React encounters an array of children, it compares each item between renders. Each item is treated as distinct, allowing React to match components correctly across renders.

- The `key` attribute serves as a unique identifier for React to track elements in dynamic arrays. It ensures consistent identification of components even when their order or presence changes (e.g., reorder, add, or remove operations).

- Key does not improve performance directly; it is used by React solely for identification between renders.

- To prevent unnecessary re-renders, use `React.memo` to memoize components.

- For static arrays that don’t change their order, using the index as a key is acceptable. For dynamic arrays that might be reordered, use a unique value such as an ID as the key.

- Changing the key on an uncontrolled component causes a state reset. This is a useful technique to reset a component’s state to its initial value.

- If two elements share the same key and one is unmounted while the other is mounted, the newly mounted element may inherit the state of the unmounted one. This happens even if the elements are otherwise different.

- React only enforces the use of keys in dynamic arrays. In static arrays, there is no risk of elements being rearranged, so keys are optional.

- When mixing dynamic and static elements, React wraps the dynamic ones into a single array, and that array becomes a single child within the parent’s children array.

---

## Key Takeaways

- React will compare elements between re-renders with elements in the same place in the returned array on any level of hierarchy. The first one with the first one, the second with the second, etc.
- If the type of the element and its position in the array is the same, React will re-render that element. If the type changes at that position, then React will unmount the previous component and mount the new one.
- An array of children will always have the same number of children (if it's not dynamic). Conditional elements (`isSomething ? <A /> : <B />`) will take just one place, even if one of them is `null`.
- If the array is dynamic, then React can't reliably identify those elements between re-renders. So we use the `key` attribute to help it. This is important when the array can change the number of its items or their position between re-renders (re-order, add, remove), and especially important if those elements are wrapped in `React.memo`.
- We can use the key outside of dynamic arrays as well to force React to recognize elements at the same position in the array with the same type as different. Or to force it to recognize elements at different positions with the same type as the same.
- We can also force unmounting of a component with a key if that key changes between re-renders based on some information (like routing). This is sometimes called "state reset".

# Chapter 7 - Higher-order components in modern world

- Higher-order components are a pattern for reusing component logic. An HOC is simply a function that accepts a component as one of its arguments, executes some logic, and returns a new component that renders the original one.

```jsx
// accept a Component as an argument
const withSomeLogic = (Component) => {
  // do something
  // return a component that renders the component from the
  argument;
  return (props) => <Component {...props} />;
};

// just a button
const Button = ({ onClick }) => <button onClick={onClick}>Button</button>;
// same button, but with enhanced functionality
const ButtonWithSomeLogic = withSomeLogic(Button);
```

- The most common use case for HOCs is to **inject props** into components. Before the introduction of Hooks, HOCs were widely used to access context or manage external data subscriptions (like Redux or Firebase listeners).

- In modern React code, HOCs can still be useful, especially for:

  - Enhancing callbacks
  - React lifecycle manipulation
  - Intercepting DOM or keyboard events

```jsx
const withEscapeKey = (Component) => {
  return (props) => {
    useEffect(() => {
      const handleKeyDown = (e) => {
        if (e.key === "Escape") {
          console.log("Escape key pressed");
        }
      };
      window.addEventListener("keydown", handleKeyDown);
      return () => window.removeEventListener("keydown", handleKeyDown);
    }, []);

    return <Component {...props} />;
  };
};

const withUserData = (Component) => {
  return (props) => {
    const [user, setUser] = useState(null);

    useEffect(() => {
      fetch("/api/user")
        .then((res) => res.json())
        .then(setUser);
    }, []);

    return <Component {...props} user={user} />;
  };
};
```

- Whenever multiple components need to share the same logic, HOCs can be a viable option — especially when using Hooks is not suitable or would add unnecessary complexity.

---

## Key Takeaways

- A higher-order component is just a function that accepts a component as an argument and returns a new component. That new component renders the component from the argument.

- We can inject props or additional logic into the components that are wrapped in a higher-order component.

```jsx
// accept a Component as an argument
const withSomeLogic = (Component) => {
  // inject some logic here
  // return a component that renders the component from the
  argument;
  // inject some prop to it
  return (props) => {
    // or inject some logic here
    // can use React hooks here, it's just a component
    return <Component {...props} some="data" />;
  };
};
```

- We can pass additional data to the higher-order component, either through the function’s argument or through props.

# Chapter 8 – React Context and Performance

- While React Context can **cause unnecessary re-renders**, it can also **prevent some** and improve app performance when used thoughtfully.

- When we want to share state across multiple components, a common pattern is to **lift the state up** to a shared parent and pass it down via props. However, this can lead to **prop drilling**, where intermediate components that don't actually use the data are still re-rendered whenever the state changes—this can hurt performance.

- To avoid prop drilling, we can use **React Context** (or similar state-sharing libraries). Context allows us to **"escape" the component tree** and pass values directly to deeply nested components without threading them through props. Only the components that actually **consume** the context value will re-render when the value changes.

- A common architectural pattern is to create a **Controller** component that manages the logic and wraps children using the `children` prop. Inside this component, we create a React **Context** and render a `Context.Provider`, passing the shared state into the `value` prop. Any components rendered within this tree can access the values using the `useContext` hook.

```jsx
import React, { createContext, useState } from "react";

export const ThemeContext = createContext();

export const ThemeController = ({ children }) => {
  const [theme, setTheme] = useState("light");

  const toggleTheme = () =>
    setTheme((prev) => (prev === "light" ? "dark" : "light"));

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};
```

- However, there are **three major caveats** to keep in mind:

  1. **All consumers will re-render** when the `value` on the `Provider` changes, even if they don’t use the specific part that changed.
  2. These re-renders **cannot be prevented with memoization** (at least not easily).
  3. If the **Controller component re-renders** (e.g., due to its parent updating), **all consumer components** will also re-render, even if the context value didn’t change.

- To mitigate this:

  - Use `useMemo` and `useCallback` to **memoize the value passed to the context provider**.
  - This is one of the few scenarios where **memoizing by default is recommended** rather than considered premature optimization.

- Even if a context consumer is only using part of the context value, **any change to the context will re-render all consumers**.

- To solve this, use the **"splitting providers"** technique—create **separate context providers** for different parts of the state, allowing more granular control over which components re-render.

- Using `useReducer` instead of `useState` can also help. With `useReducer`, components **dispatch actions** rather than directly managing state, which can decouple logic and reduce unnecessary re-renders.

- For advanced scenarios, we can use **context selectors**, a technique that mimics `useMemo`-like optimization. This involves using **higher-order components** (HOCs) that memoize the component passed to them, isolating the re-renders.

```jsx
// withSelectedContext.js
import React, { useContext, memo } from "react";

const withSelectedContext = (Context, selector) => (Component) => {
  const Wrapped = (props) => {
    const context = useContext(Context);
    const selected = selector(context);
    return <Component {...props} selected={selected} />;
  };
  return memo(Wrapped);
};

// usage:
const SelectedThemeComponent = withSelectedContext(
  ThemeContext,
  (ctx) => ctx.theme
)(MyComponent);
```

---

## Key Takeaways

- With Context (or any context-like state management library), we can pass data directly from one component to another deep in the render tree without the need to wire it through props.

- Passing data in this way can improve the performance of our apps, as we'll avoid the re-rendering of all components in between.

- Context, however, can be dangerous: all components that use it will re-render if the value in the Context provider changes. This re-render can't be stopped with standard memoization techniques.

- To minimize Context re-renders, we should always memoize the value we pass to the provider.

- We can split the Context provider into multiple providers to further minimize re-renders. Switching from useState to useReducer can help with this.

- Even though we don't have proper selectors for Context, we can imitate their functionality with higher order components and React.memo.

# Chapter 9 – Refs: From Storing Data to Imperative API

- Refs are used to access the actual DOM elements in React (which is rare but occasionally necessary).
- Typical use cases for using native DOM APIs in React include:
  - Manually focusing an input after it renders
  - Detecting clicks outside a component (like closing a popup)
  - Manually scrolling to an element after it appears
  - Calculating element size and position (e.g., tooltips)

```jsx
const inputRef = useRef();

useEffect(() => {
  inputRef.current?.focus();
}, []);

return <input ref={inputRef} />;
```

- While you could use `getElementById`, React provides **Refs** as a more React-friendly way to achieve the same.
- A **Ref** is a **mutable object** that persists between renders.
  - Since it's not state, updating a ref **doesn't trigger re-renders**.
  - To reflect updated ref values in the UI, some other state change must cause a re-render.
- If a **ref value is passed as a prop to a child component**, it behaves like a primitive value. Even if the child re-renders due to its own state change, the ref value remains the same unless the parent re-renders.
- Ref updates are **synchronous**, unlike state updates, which are batched and asynchronous to allow React to optimize diffs and rendering.
- **How to decide between state and ref?**
  - Ask: _Will the value be used for rendering?_ and _Will it be passed as props to other components?_
  - If the answer to both is **no**, then it’s fine to use a **ref**.
- A very common use case is assigning DOM elements to refs.
- A ref is only assigned **after** the element is rendered and its DOM node is created.
  - Before that, `ref.current` is `null`.
  - You should only read or write to `ref.current` inside `useEffect` or in callbacks.
- Refs can be **passed from parent to child**:
  - Declare `useRef` in the parent.
  - Assign it to an element in the child.
  - The parent now has access to that element.

```jsx
// Parent
const inputRef = useRef();
return <Child inputRef={inputRef} />;

// Child
const Child = ({ inputRef }) => {
  return <input ref={inputRef} />;
};
```

- When **passing refs as props**, avoid naming the prop `ref`. Instead, use something like `inputRef`.
  - `ref` is a special prop in React (a leftover from class components).
  - If you really want to use `ref` as the prop name, use `forwardRef`.
    - `forwardRef` lets you pass a ref to a functional component.
    - The component receives the ref as the **second argument**, and you pass it to a DOM element inside.
  - Whether to use a custom prop (like `inputRef`) or `forwardRef` is a matter of preference—they behave the same in the end.

```jsx
const FancyInput = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

const inputRef = useRef();

<FancyInput ref={inputRef} />;
```

- React is **declarative**, but sometimes you need **imperative logic**.
  - For such cases, React provides the `useImperativeHandle` hook.
    - You decide what public methods the component should expose.
    - You also need a ref to attach this handle to.
    - This is useful to expose a custom API, combining state logic and ref values.

```jsx
const FancyInput = forwardRef((props, ref) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
  }));

  return <input ref={inputRef} />;
});

// Parent
const inputRef = useRef();

<FancyInput ref={inputRef} />;
inputRef.current.focus(); // Imperatively focus input
```

- You don’t **have to** use `useImperativeHandle`.
  - Since a ref is just a mutable object, you can assign any value to it manually in a `useEffect`.
- Using refs and imperative methods in React is considered an **escape hatch**.
  - Use them only when necessary.
  - In 99% of cases, the usual **props and callbacks flow** should suffice.

---

## Key takeaways

- A Ref is just a mutable object that can store any value. That value will be preserved between re-renders.
- A Ref's update doesn't trigger re-renders and is synchronous.
- We can assign a Ref to a DOM element via the ref attribute. After that element is rendered, we'll see that element in the ref.current property.
- We can pass Refs as regular props to any component.
- If we want to pass it as the actual ref prop, we need to wrap that component in forwardRef . Otherwise, it won't work on functional components. The second argument of that component will be the ref itself, which we then need to pass down to the desired DOM element.

```jsx
// second argument, next to props, is ref that is injected by
"forwardRef";
const InputField = forwardRef((props, ref) => {
  return <input ref={ref} />;
});
```

- We can hide the implementation details of a component and expose its public API with the useImperativeHandle hook. We'll need to pass a Ref to that component, which will be mutated with the API properties:

```jsx
const InputField = ({ apiRef }) => {
  useImperativeHandle(
    apiRef,
    () => ({
      focus: () => {},
      shake: () => {},
    }),
    []
  );
};
```

- Or, we can always just mutate that Ref manually in the useEffect hook:

```jsx
const InputField = ({ apiRef }) => {
  useEffect(() => {
    apiRef.current = {
      focus: () => {},
      shake: () => {},
    };
  }, [apiRef]);
};
```

# Chapter 10 – Closures in React

- `React.memo` accepts a comparison function that allows fine-grained control over prop comparisons. By default, React compares previous and next props automatically. When a comparison function is provided, React relies on its return value (`false` to re-render, `true` to skip).

```jsx
const HeavyComponentMemo = React.memo(HeavyComponent, (before, after) => {
  return before.title === after.title;
});
```

- In JavaScript, declaring a function creates a **local scope**. Variables declared inside are not visible from the outside.
- A function declared inside another function has its own local scope, but it can also **access variables declared outside** its own scope. This is known as a **closure**.
- A closure is a function that "closes over" its surrounding variables—capturing a snapshot of the environment (variables and values) at the time it was created.
- When we pass a value to a function that returns another function (which uses that value), the returned function holds on to that value via closure. As long as the returned function is referenced somewhere, that "snapshot" remains alive.
- Closures are like taking a picture: once it's taken, it's frozen in time. Any new "pictures" (i.e., new closures) do not alter the previous ones but capture a new scene.
- In React, we use closures constantly:
  - Every callback is a closure.
  - Every hook involves closures.
  - Every function inside a component is a closure, since components themselves are just functions.

```jsx
const Component = () => {
  const [state, setState] = useState();

  const onClick = useCallback(() => {
    // perfectly fine
    console.log(state);
  });

  useEffect(() => {
    // perfectly fine
    console.log(state);
  });
};
```

- Closures remain alive as long as the function that formed them is still referenced. This reference can be stored anywhere, including state or refs.
- A **stale closure** occurs when a function is reused without recreating it, and it references outdated values from its closure. This typically happens when referencing variables from an outer scope that have changed since the closure was created.
  - To avoid stale closures, we can use separate variables to track previous values and compare them explicitly.

```jsx
const cache = {};
let prevValue;

const something = (value) => {
  // check whether the value has changed
  if (!cache.current || value !== prevValue) {
    cache.current = () => {
      console.log(value);
    };
  }

  // refresh it
  prevValue = value;
  return cache.current;
};
```

- `useCallback` creates a closure. If you access state or props inside it, you must include them in the dependency array.
  - The dependency array tells React when to refresh the closure. If it's empty, the closure will become stale over time.
- `useRef` can also lead to stale closures. Since the ref is only initialized once (on mount), its internal function closures won't update automatically.
  - To update the value inside a ref, you need to do it manually, often inside `useEffect`, based on other state or props.
- One way to **escape closure traps** is by using refs:
  - Create a ref and store a function or value in `ref.current`.
  - Ensure that the function stored in `ref.current` is updated on every render (e.g., using `useEffect` without a dependency array).
  - Be cautious when passing `ref.current` to memoized components—it will always appear as a new value on each render, breaking memoization.
  - Instead, you can create a stable function using `useCallback` (with no dependencies) and have it call the up-to-date value inside `ref.current`.

```jsx
const Form = () => {
  const [value, setValue] = useState();

  const ref = useRef();
  useEffect(() => {
    ref.current = () => {
      // will be latest
      console.log(value);
    };
  });

  const onClick = useCallback(() => {
    // will be latest
    ref.current?.();
  }, []);

  return (
    <>
      <input
        type="text"
        value={value}
        onChange={(e) => setValue(e.target.value)}
      />
      <HeavyComponentMemo title="Welcome closures" onClick={onClick} />
    </>
  );
};
```

---

## Key takeaways

- Closures are formed every time a function is created inside another function.
- Since React components are just functions, every function created inside forms a closure, including such hooks as useCallback and useRef.
- When a function that forms a closure is called, all the data around it is "frozen", like a snapshot.
- To update that data, we need to re-create the "closed" function. This is what dependencies of hooks like useCallback allow us to do.
- If we miss a dependency, or don't refresh the closed function assigned to ref.current , the closure becomes "stale".
- We can escape the "stale closure" trap in React by taking advantage of the fact that Ref is a mutable object. We can mutate ref.current outside of the stale closure, and then access it inside. Will be the latest data.

# Chapter 11 – Implementing Advanced Debouncing and Throttling with Refs

- Refs are commonly used to store various timers or timeout IDs, especially when working with `setInterval`, `setTimeout`, or debounce/throttle mechanisms. For instance, we might debounce `onChange` callbacks in form inputs to prevent excessive re-renders.

```jsx
const debounce = (callback, wait) => {
  // initialize the timer
  let timer;

  // lots of code here involving the actual implementation of timer
  // to track the time passed since the last callback call
  const debouncedFunc = () => {
    // checking whether the waiting time has passed
    if (shouldCallCallback(Date.now())) {
      callback();
    } else {
      // if time hasn't passed yet, restart the timer
      timer = startTimer(callback);
    }
  };

  return debouncedFunc;
};
```

- **Debouncing** and **throttling** are techniques for controlling how often a function is allowed to execute within a given time period:
  - **Debounce** delays function execution until a specified time has passed **since the last call**.
  - **Throttle** guarantees the function is called **at most once per interval**, regardless of how often it's triggered.
- For controlled input elements, `onChange` fires on every keystroke. Using debounce, we can delay expensive operations (like API calls) until the user stops typing.
- A debounce function typically:
  - Accepts a callback and a delay interval.
  - Returns a new function that resets a timer every time it's called.
  - Only executes the original callback if the timer completes without being reset.
- Throttle is similar, but instead of resetting the timer, it ensures the callback runs **once per interval**, no matter how often it's called.
- When using controlled inputs, it's better to **debounce only a portion of logic**, rather than delaying `setState`, to avoid stale or lagging UI values.

```jsx
const Input = () => {
  const [value, setValue] = useState("initial");

  // memoize the callback with useCallback
  // we need it since it's a dependency in useMemo below
  const sendRequest = useCallback((value: string) => {
    console.log("Changed value:", value);
  }, []);

  // memoize the debounce call with useMemo
  const debouncedSendRequest = useMemo(() => {
    return debounce(sendRequest, 1000);
  }, [sendRequest]);

  const onChange = (e) => {
    const value = e.target.value;
    setValue(value);
    debouncedSendRequest(value);
  };

  return <input onChange={onChange} value={value} />;
};
```

- Instead of using `useMemo` or `useCallback` to memoize debounced functions, we can use a `useRef` to persist them across renders.
  - However, since `ref.current` does not automatically update, you should update it manually using `useEffect` whenever dependencies change.
  - A challenge here is that **debounced functions get re-created on dependency changes**, so you may need to handle cleanup inside the `useEffect` to cancel any in-progress debounced calls.
  - This pattern works for debouncing, but **not reliably for throttling**, because throttling relies on consistent interval-based calls, which get interrupted by resets.

```jsx
const useDebounce = (callback) => {
  const ref = useRef();

  useEffect(() => {
    ref.current = callback;
  }, [callback]);

  const debouncedCallback = useMemo(() => {
    const func = () => {
      ref.current?.();
    };
    return debounce(func, 1000);
  }, []);

  return debouncedCallback;
};

const Input = () => {
  const [value, setValue] = useState();

  const debouncedRequest = useDebounce(() => {
    // send request to the backend
    // access to the latest state here
    console.log(value);
  });

  const onChange = (e) => {
    const value = e.target.value;
    setValue(value);
    debouncedRequest();
  };

  return <input onChange={onChange} value={value} />;
};
```

---

## Key takeaways

- We use debounce and throttle when we want to skip some function's executions that were fired too often.
- In order for those functions to work properly, they should be called only once in a component's life, usually when it's mounted.
- If we call them in the component's render function directly, the timer inside will be re-created with every re-render, and the functions will not work as expected.
- To fix this, we can memoize those with `useMemo` or through the usage of Refs.
- If we simply memoize them or use Refs "naively", we won't have access to the component's latest data, like state or props. This is happening because a closure is created when we initialize Ref, which freezes values at the time it's created.
- To escape the closure trap, we can leverage the mutable nature of the Ref object and gain access to the latest data by constantly updating the "closed" function in `ref.current` within `useEffect`.

# Chapter 12 – Escaping Flickering UI with `useLayoutEffect`

- Suppose we want to build a navigation component that dynamically adjusts the number of visible links based on the container’s width. To achieve this, we need **accurate measurements** of the container and each link item, which we can obtain using the native `getBoundingClientRect` API.

- To measure the sizes, we use **refs** to access DOM elements and read their dimensions inside a `useEffect`. We then iterate over the items, calculate their total width, and compare it to the container’s width. Based on that, we calculate a **state value** that determines how many items to render.

- The problem with this approach is that it can cause **visible flickers**. This happens because React renders the items and makes them visible before the unnecessary ones are removed. A workaround is to initially **hide** the items (e.g., `opacity: 0`) and only show them once dimensions have been measured and the correct items have been rendered.

```jsx
const Component = ({ items }) => {
  // set the initial value to -1, to indicate that we haven't run the calculations yet
  const [lastVisibleMenuItem, setLastVisibleMenuItem] = useState(-1);

  useEffect(() => {
    const itemIndex = getLastVisibleItem(ref.current);
    // update state with the actual number
    setLastVisibleMenuItem(itemIndex);
  }, [ref]);

  // render everything if it's the first pass and the value is still the default
  if (lastVisibleMenuItem === -1) {
    // render all of them here, same as before
    return "...";
  }

  // show "more" button if the last visible item is not the last one in the array
  const isMoreVisible = lastVisibleMenuItem < items.length - 1;
  // filter out those items which index is more than the last visible
  const filteredItems = items.filter(
    (item, index) => index <= lastVisibleMenuItem
  );

  return (
    <div className="navigation">
      {/*render only visible items*/}
      {filteredItems.map((item) => (
        <a href={item.href}>{item.name}</a>
      ))}

      {/*render "more" conditionally*/}
      {isMoreVisible && <button id="more">...</button>}
    </div>
  );
};
```

- A more robust solution is to use `useLayoutEffect` instead of `useEffect`. The logic remains the same, but `useLayoutEffect` runs **synchronously after all DOM mutations and before the browser repaints**. This prevents the flicker by ensuring that the DOM is fully prepared before the user sees it.

- However, use `useLayoutEffect` **only when necessary**, because it can **negatively impact performance**. Unlike `useEffect`, it blocks painting until it finishes executing, which can delay rendering if the logic inside is heavy.

- In React, browser rendering is referred to as **painting** to distinguish it from React’s own rendering process. The browser doesn’t continuously repaint the screen—it works in frames, similar to a slideshow, aiming for **60 frames per second (FPS)**, or about **16ms per frame**. (The original text said 13ms, but the typical target is ~16.67ms.)

- The browser schedules tasks in a queue and attempts to complete as many as possible within this 16ms window before the next paint.

- Any synchronous JavaScript code (like that in a `<script>` tag) is considered a **task**. If a task takes longer than 16ms, the browser **won’t interrupt it**—it waits for it to finish before painting.

- React avoids long, blocking tasks by **splitting rendering into smaller chunks**, using asynchronous mechanisms like callbacks, event handlers, and `setTimeout`. For example, wrapping logic in `setTimeout` defers it as a separate task that the browser can handle efficiently.

- This task-splitting is part of React’s **concurrent rendering engine**, which tries to keep task durations under 16ms for smoother UI performance.

- `useLayoutEffect` behaves differently—it is executed **synchronously during React updates** as part of the same task. Even if state is updated (which normally causes an async render), React keeps it **synchronous** when `useLayoutEffect` is involved.

- This synchronous behavior is why `useLayoutEffect` can **prevent flickering**, but it may **hinder performance** since React cannot break the work into smaller tasks.

- When working with frameworks like **Next.js** or any other **server-side rendering (SSR)** solutions, `useLayoutEffect` can cause issues. During SSR, React components are rendered on the server using something like `React.renderToString(<App />)` to generate HTML that is sent to the client.

- This initial render on the server means **browser-only APIs** (like `getBoundingClientRect`) won’t be available, causing errors if accessed during the SSR phase.

- To prevent this, we can show a **default UI** on the server and wait to render the actual UI until the client mounts. This can be done using a `shouldRender` state variable that flips to `true` inside a `useEffect`—since `useEffect` only runs on the client, the real layout-sensitive content is deferred until after the browser loads.

```jsx
const Component = () => {
  const [shouldRender, setShouldRender] = useState(false);
  useEffect(() => {
    setShouldRender(true);
  }, []);
  if (!shouldRender) return <SomeNavigationSubstitude />;
  return <Navigation />;
};
```

- Simply checking for the existence of the `window` object is not sufficient, because the initial render on the client must **match the HTML** sent from the server for hydration to succeed.

```jsx
const Component = () => {
  // Detecting SSR by checking whether window is there
  if (typeof window === undefined) return <SomeNavigationSubstitude />;
  return <Navigation />;
};
```

---

## Key Takeaways

- When we calculate the dimensions of elements inside the `useEffect` hook and then hide them or adjust their size, we might see the visual "glitch".
- This is happening because normally `useEffect` is run asynchronously. Asynchronous code is a separate task from the browser's perspective. So it has a chance to paint the state "before" and "after" the change, resulting in the glitch.
- We can prevent this behavior with the `useLayoutEffect` hook. This hook is run synchronously. From the browser's perspective, it will be one large, unbreakable task. So the browser will wait and will not paint anything until the task is complete and the final dimensions are calculated.
- In the SSR environment, useLayoutEffect will not work since React doesn't run `useLayoutEffect` in SSR mode, and the "glitch" will be visible again.
- This can be fixed by opting out of SSR for this specific feature.
