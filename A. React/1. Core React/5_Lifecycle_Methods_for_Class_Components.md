# **Lifecycle Methods in Class Components**  

React class components have **lifecycle methods** that allow developers to **execute code at different stages** of a component's life. These methods are mainly divided into **`Mounting`, `Updating`, and `Unmounting` phases**.

## **🔹 `React Component Lifecycle Phases`**  
1️⃣ **`Mounting`** → When the component is first created and inserted into the DOM.  
2️⃣ **`Updating`** → When the component's state or props change.  
3️⃣ **`Unmounting`** → When the component is removed from the DOM.  
4️⃣ **`Error Handling`** → Catch errors inside the component tree.  

---

# **1️⃣ Mounting Phase (Component Creation)**
When a component is created, React calls these methods in order:  
✔ `constructor()`  
✔ `static getDerivedStateFromProps()`  
✔ `render()`  
✔ `componentDidMount()`

### **🔹 Example of a Class Component with Mounting Lifecycle**
```tsx
import React, { Component } from "react";

type State = { count: number };

class Counter extends Component<{}, State> {
  constructor(props: {}) {
    super(props);
    console.log("1️⃣ constructor() called");
    this.state = { count: 0 };
  }

  static getDerivedStateFromProps(props: {}, state: State) {
    console.log("2️⃣ getDerivedStateFromProps() called");
    return null; // Must return an object to update state, or null to do nothing
  }

  componentDidMount() {
    console.log("4️⃣ componentDidMount() called");
    // Example: Fetching data or setting up subscriptions
  }

  render() {
    console.log("3️⃣ render() called");
    return (
      <div>
        <h1>Count: {this.state.count}</h1>
      </div>
    );
  }
}

export default Counter;
```

### **🔍 Machine-Level Explanation**
1️⃣ **`constructor()`**  
   - Initializes the component with a state and binds methods.  
   - Runs **once** when the component is created.  

2️⃣ **`getDerivedStateFromProps(props, state)`**  
   - Runs **before `render()`** and updates the state based on props.  
   - **Use case:** If state needs to be derived from props.  
   - **⚠️ Rarely used**, as it can make the component hard to debug.  

3️⃣ **`render()`**  
   - Converts JSX to a Virtual DOM representation.  
   - Must be a **pure function** (i.e., no side effects).  

4️⃣ **`componentDidMount()`**  
   - Runs **after the component is mounted** into the real DOM.  
   - Common use case: **Fetching data, subscriptions, event listeners.**  

---

# **2️⃣ Updating Phase (Component Re-Renders)**
A component **updates** when its **state or props change**. The order of method execution is:  
✔ `static getDerivedStateFromProps()`  
✔ `shouldComponentUpdate()`  
✔ `render()`  
✔ `getSnapshotBeforeUpdate()`  
✔ `componentDidUpdate()`

### **🔹 Example of an Updating Lifecycle**
```tsx
import React, { Component } from "react";

type State = { count: number };

class Counter extends Component<{}, State> {
  constructor(props: {}) {
    super(props);
    this.state = { count: 0 };
  }

  static getDerivedStateFromProps(props: {}, state: State) {
    console.log("1️⃣ getDerivedStateFromProps() (Updating)");
    return null;
  }

  shouldComponentUpdate(nextProps: {}, nextState: State) {
    console.log("2️⃣ shouldComponentUpdate() called");
    return true; // If false, component will not re-render
  }

  getSnapshotBeforeUpdate(prevProps: {}, prevState: State) {
    console.log("4️⃣ getSnapshotBeforeUpdate() called");
    return null; // Used for capturing DOM info before update
  }

  componentDidUpdate(prevProps: {}, prevState: State) {
    console.log("5️⃣ componentDidUpdate() called");
  }

  increment = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    console.log("3️⃣ render() (Updating)");
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

### **🔍 Machine-Level Explanation**
1️⃣ **`getDerivedStateFromProps()`** (again)  
   - Runs before every render when props change.  
   - Use only if **state needs to be updated from props.**  

2️⃣ **`shouldComponentUpdate(nextProps, nextState)`**  
   - Runs **before re-rendering** to decide if a re-render is needed.  
   - **Returns `true` to allow re-render, `false` to prevent it.**  
   - Useful for **performance optimizations** (prevent unnecessary renders).  

3️⃣ **`render()`**  
   - Converts JSX to Virtual DOM **again**.  

4️⃣ **`getSnapshotBeforeUpdate(prevProps, prevState)`**  
   - Runs **before React updates the real DOM**.  
   - Used for capturing **scroll positions, animations, or measurements.**  

5️⃣ **`componentDidUpdate(prevProps, prevState)`**  
   - Runs **after the component updates in the DOM**.  
   - **Common use case:** Fetching new data based on updated props.  

---

# **3️⃣ Unmounting Phase (Component Removal)**
When a component is removed from the DOM, only **one method is called**:

✔ `componentWillUnmount()`

### **🔹 Example of `componentWillUnmount()`**
```tsx
import React, { Component } from "react";

class Timer extends Component {
  componentWillUnmount() {
    console.log("componentWillUnmount() called");
    // Cleanup operations like clearing intervals or event listeners
  }

  render() {
    return <h1>Timer Running...</h1>;
  }
}

export default Timer;
```

### **🔍 Machine-Level Explanation**
- **Called when the component is about to be destroyed**.  
- Used to **clear event listeners, close connections, or clean up resources**.  

---

# **4️⃣ Error Handling Phase**
React provides **error boundaries** to catch errors that occur inside the component tree.  

✔ `componentDidCatch(error, info)`  

### **🔹 Example of Error Boundaries**
```tsx
import React, { Component, ErrorInfo } from "react";

class ErrorBoundary extends Component<{ children: React.ReactNode }, { hasError: boolean }> {
  constructor(props: { children: React.ReactNode }) {
    super(props);
    this.state = { hasError: false };
  }

  componentDidCatch(error: Error, info: ErrorInfo) {
    console.log("Error Caught:", error);
    this.setState({ hasError: true });
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong!</h1>;
    }
    return this.props.children;
  }
}

export default ErrorBoundary;
```
- **Wrap components in `ErrorBoundary` to catch errors.**  

---

# **🛠 Summary of Lifecycle Methods**
| Phase          | Method                      | Purpose |
|---------------|---------------------------|----------------|
| **Mounting**  | `constructor()`           | Initialize state |
|               | `getDerivedStateFromProps()` | Sync state with props |
|               | `render()`                 | Render the UI |
|               | `componentDidMount()`      | Fetch data, setup subscriptions |
| **Updating**  | `getDerivedStateFromProps()` | Update state before render |
|               | `shouldComponentUpdate()`  | Optimize rendering |
|               | `render()`                 | Render updated UI |
|               | `getSnapshotBeforeUpdate()` | Capture DOM info before update |
|               | `componentDidUpdate()`     | Perform post-update operations |
| **Unmounting** | `componentWillUnmount()`  | Cleanup (remove listeners, cancel API calls) |
| **Error Handling** | `componentDidCatch()` | Catch errors in component tree |

---
