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
