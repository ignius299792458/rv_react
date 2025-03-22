# **State**  

State is a **core concept in React** that allows `components to store and manage dynamic data internally`. Unlike props, **state is mutable** and can change over time. When state updates, **React triggers a re-render**, updating the UI efficiently.  

---

# **1. What is State?**  
**State is a `JavaScript object`** that stores **dynamic data** and **`controls the behavior of a component`**.  

## 💡 **Key Characteristics:**  
✔ **Mutable** → Unlike props, state can be modified.  
✔ **Triggers Re-Renders** → When state changes, React updates the UI.  
✔ **Scoped to the Component** → Each component manages its own state.  
✔ **Can be Passed as Props** → A component’s state can be passed down to child components.  

---

# **2. Functional Component with `useState()`**
The `useState()` **hook** allows function components to have state.

### **🔹 Example: Counter with `useState()`**
```tsx
import React, { useState } from "react";

const Counter: React.FC = () => {
  const [count, setCount] = useState<number>(0); // State initialized with 0

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};

export default Counter;
```

---

### **🔍 What Happens Internally?**
1. **React Calls `useState()`**  
   - `useState(0)` **creates an internal state variable**.
   - React **stores the state in memory** and returns:
     ```js
     [0, function setCount(newValue) {...}]
     ```

2. **State Change with `setCount()`**
   - Clicking the button calls:
     ```js
     setCount(count + 1);
     ```
   - This **does not mutate the existing state** but **creates a new state value**.

3. **Reconciliation & Re-Rendering**
   - React **compares the old Virtual DOM with the new one**.
   - It updates **only the changed parts** of the UI.

---

# **3. Class Component with State**
Before hooks, **class components** used `this.state`.

### **🔹 Example: Counter with Class Component**
```tsx
import React, { Component } from "react";

type CounterState = {
  count: number;
};

class Counter extends Component<{}, CounterState> {
  constructor(props: {}) {
    super(props);
    this.state = { count: 0 };
  }

  increment = () => {
    this.setState((prevState) => ({ count: prevState.count + 1 }));
  };

  render() {
    return (
      <div>
        <h1>Count: {this.state.count}</h1>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}

export default Counter;
```

---

### **🔍 What Happens Internally?**
1. **React Instantiates the Class**
   ```js
   const instance = new Counter();
   instance.state = { count: 0 };
   ```
   
2. **Calling `setState()`**
   ```js
   this.setState((prevState) => ({ count: prevState.count + 1 }));
   ```
   - React **merges the new state** with the old state.
   - A **new Virtual DOM is created**.

3. **Reconciliation & DOM Updates**
   - React **detects changes** in the Virtual DOM.
   - **Only modified elements are updated** in the real DOM.

---

# **`4. Updating State Correctly`**
State updates are **`asynchronous`**, so always use **`the callback form`** for updates that depend on the previous state.

### **🔹 Example: Incorrect vs Correct State Updates**
```tsx
const Counter: React.FC = () => {
  const [count, setCount] = useState<number>(0);

  // ❌ Incorrect: Directly using count (can cause stale state issues)
  const incrementWrong = () => {
    setCount(count + 1);
    setCount(count + 1); // This won't work as expected
  };

  // ✅ Correct: Using function form to get latest state
  const incrementCorrect = () => {
    setCount((prevCount) => prevCount + 1);
    setCount((prevCount) => prevCount + 1); // Correctly increments twice
  };

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={incrementWrong}>Increment (Wrong)</button>
      <button onClick={incrementCorrect}>Increment (Correct)</button>
    </div>
  );
};
```

---

# **5. Handling Complex State with Objects**
### **🔹 Example: State with Objects**
```tsx
type User = {
  name: string;
  age: number;
};

const UserProfile: React.FC = () => {
  const [user, setUser] = useState<User>({ name: "Ignius", age: 25 });

  const updateAge = () => {
    setUser((prevUser) => ({ ...prevUser, age: prevUser.age + 1 }));
  };

  return (
    <div>
      <h1>{user.name} is {user.age} years old.</h1>
      <button onClick={updateAge}>Increase Age</button>
    </div>
  );
};
```

### **🔍 What Happens?**
- `setUser((prevUser) => ({ ...prevUser, age: prevUser.age + 1 }));`
  - **Creates a new object** (React requires immutability).
  - Merges new state with the previous state.

---

# **6. `useState()` with Arrays**
### **🔹 Example: Adding Items to an Array**
```tsx
const TodoList: React.FC = () => {
  const [todos, setTodos] = useState<string[]>([]);

  const addTodo = () => {
    setTodos((prevTodos) => [...prevTodos, `Task ${prevTodos.length + 1}`]);
  };

  return (
    <div>
      <ul>
        {todos.map((todo, index) => (
          <li key={index}>{todo}</li>
        ))}
      </ul>
      <button onClick={addTodo}>Add Todo</button>
    </div>
  );
};
```
- **Why `[...prevTodos, newTodo]`?**
  - React requires state updates to be **immutable** (no direct mutation).

---

# **7. `useState()` vs `useReducer()` (For Complex State)**
For **simple state**, `useState()` is fine.  
For **complex state logic**, `useReducer()` is better.

### **🔹 Example: Using `useReducer()` Instead of `useState()`**
```tsx
import React, { useReducer } from "react";

type CounterState = { count: number };
type Action = { type: "increment" } | { type: "decrement" };

const reducer = (state: CounterState, action: Action): CounterState => {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    default:
      return state;
  }
};

const Counter: React.FC = () => {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <div>
      <h1>Count: {state.count}</h1>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
    </div>
  );
};

export default Counter;
```
✅ **`useReducer()` is great for managing complex state logic.**

---

# **Final Thoughts**
✔ **State is mutable and managed inside the component.**  
✔ **React triggers re-renders when state changes.**  
✔ **Use callback form (`setState(prev => newState)`) for updates that depend on previous state.**  
✔ **Use `useReducer()` for complex state management.**  

---