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
