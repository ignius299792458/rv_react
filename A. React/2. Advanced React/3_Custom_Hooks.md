# 3. **Custom Hooks in React: For Reusability and Separation of Concerns**

Custom hooks are an essential feature in React that allow you to extract and reuse stateful logic across multiple components. They help to **avoid code duplication**, **encapsulate logic** in a reusable manner, and **maintain separation of concerns** within your components.

Custom hooks allow you to encapsulate logic like state management, side effects, event listeners, or any other logic that can be reused across multiple components. Since they follow the same rules as React’s built-in hooks (e.g., `useState`, `useEffect`), they allow you to combine multiple pieces of logic without complicating your component code.

## **Why Use Custom Hooks?**

1. **Reusability**: You can reuse logic across components without duplicating code.
2. **Separation of Concerns**: Custom hooks encapsulate complex logic and keep the component code clean.
3. **Performance Optimization**: By isolating logic in hooks, you can optimize your components’ performance by minimizing unnecessary re-renders or side effects.
4. **Testability**: Custom hooks make testing easier because the logic is separated into isolated units.

---

## **Creating a Custom Hook Example: `useLocalStorage`**

Let’s build a **custom hook** called `useLocalStorage` that helps manage data in the browser’s localStorage. This hook will allow components to **read from** and **write to** `localStorage` while providing a fallback value when there is no data.

## **Step-by-Step Breakdown of `useLocalStorage` Hook**

### 1. **Initialization:**

```jsx
import { useState } from 'react';

function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error("Error reading localStorage", error);
      return initialValue;
    }
  });
  
  const setValue = (value) => {
    try {
      setStoredValue(value); // Update the state
      window.localStorage.setItem(key, JSON.stringify(value)); // Save to localStorage
    } catch (error) {
      console.error("Error writing to localStorage", error);
    }
  };

  return [storedValue, setValue]; // Return value and setter function
}
```

### **How the `useLocalStorage` Hook Works:**

- **State Initialization**: 
  - The hook uses `useState` to initialize the `storedValue` based on the value from `localStorage` (if available). If the value doesn't exist in `localStorage`, it defaults to the provided `initialValue`.
  - **`useState(() => {...})`** is a **lazy initialization** technique. It runs the function only once, so you only access `localStorage` the first time the component renders.

- **Setting the Value**: 
  - The `setValue` function updates both the local component state (`storedValue`) and `localStorage` using `localStorage.setItem`.

- **Error Handling**: 
  - The hook has `try-catch` blocks to catch and handle any errors that may occur when interacting with `localStorage`, such as when the data exceeds storage limits or the user’s browser settings prevent localStorage access.

### **Using `useLocalStorage` in a Component:**

Now, let’s use `useLocalStorage` in a component to manage a user's preference (e.g., a dark mode setting) across sessions.

```jsx
const DarkModeToggle = () => {
  const [isDarkMode, setIsDarkMode] = useLocalStorage('darkMode', false);

  const toggleDarkMode = () => {
    setIsDarkMode(!isDarkMode);
  };

  return (
    <div style={{ background: isDarkMode ? 'black' : 'white', color: isDarkMode ? 'white' : 'black' }}>
      <h1>Dark Mode</h1>
      <button onClick={toggleDarkMode}>
        Toggle Dark Mode
      </button>
    </div>
  );
};
```

- **`useLocalStorage('darkMode', false)`**: This initializes the `isDarkMode` state with the value from `localStorage` if available. If not, it defaults to `false` (light mode).
- The component uses a button to toggle the dark mode setting. Whenever the button is clicked, it updates both the local state and `localStorage` using `setIsDarkMode`.

## **Performance Benefits of Custom Hooks**

By using a custom hook like `useLocalStorage`, we separate the logic for accessing and updating `localStorage` from the component, which provides the following performance benefits:

### **1. Avoiding Repetitive Code**
Instead of writing the same `localStorage` logic in multiple components, we encapsulate it in a custom hook that can be reused across the entire application. This makes your code more efficient and maintainable.

### **2. Improved Re-renders**
By isolating the logic into a custom hook, we reduce unnecessary re-renders. In the `useLocalStorage` example, React only triggers a re-render when the value changes, not when the component mounts, because the logic inside the custom hook is kept isolated from the component's render flow.

### **3. Dependency Optimization**
Custom hooks can also improve performance by optimizing dependency arrays in hooks like `useEffect`. By separating the logic into hooks, you reduce the likelihood of passing unnecessary dependencies to effects, thus optimizing when they should run.

---

## **Advanced Example: usePrevious Custom Hook**

Let’s look at another example of a custom hook, `usePrevious`, which stores the previous value of a state or prop. This is useful for comparing the previous and current values within a component.

```jsx
import { useState, useEffect } from 'react';

function usePrevious(value) {
  const [prevValue, setPrevValue] = useState();

  useEffect(() => {
    setPrevValue(value);
  }, [value]);

  return prevValue;
}
```

### **Using `usePrevious`:**

```jsx
const Counter = () => {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  const increment = () => setCount(count + 1);

  return (
    <div>
      <p>Current Count: {count}</p>
      <p>Previous Count: {prevCount}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
};
```

- The `usePrevious` hook stores the previous value of `count`, and it updates whenever the `count` value changes.
- This is useful for scenarios where you need to compare the previous state to the current state (for example, in animations or transitions).

---

## **When to Use Custom Hooks for Performance Optimization:**

1. **State Management**: If you have repetitive state logic in multiple components, encapsulate it in a custom hook to reduce redundancy and make the code more readable and maintainable.
2. **Side Effects**: For side effects that are reused (e.g., fetching data, event listeners), creating a custom hook can help avoid code duplication.
3. **Complex Logic**: For any complex or shared logic that interacts with hooks like `useState`, `useEffect`, or `useReducer`, custom hooks allow you to encapsulate that complexity in a reusable function, improving code maintainability and optimization.

## **Conclusion**

Custom hooks provide a clean, reusable, and optimized way to share logic across components in React. They help encapsulate complex behavior, improve code readability, and optimize performance by reducing unnecessary re-renders or state changes. By isolating stateful logic, you can keep your components simple and focused on the UI while delegating the non-UI logic to custom hooks.
