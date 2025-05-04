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
