
## **1. `JSX` (JavaScript XML)**

`JSX` is a `syntax extension for JavaScript` that allows us to write UI elements in a syntax that looks similar to HTML. It is **not** JavaScript or HTML but a syntactic sugar for `React.createElement()`, making code more readable and maintainable.

### **How JSX Works Internally (Machine-Level Explanation)**

When JSX is written, it is not directly understood by the browser. The **Babel compiler** transpiles JSX into vanilla JavaScript (`React.createElement()` calls). This is what happens under the hood:

```jsx
const element = <h1>Hello, Ignius!</h1>;
```

gets compiled into:

```js
const element = React.createElement("h1", null, "Hello, Ignius!");
```

And then `React.createElement()` produces a **React element**, which is a JavaScript object:

```json
{
  "type": "h1",
  "props": {
    "children": "Hello, Ignius!"
  }
}
```

This object is then passed to **ReactDOM.render()**, which efficiently updates the **Virtual DOM** and syncs it with the real DOM.

---

### **Example of JSX**

```jsx
import React from "react";

function App() {
  const name = "Ignius";
  return <h1>Hello, {name}!</h1>; // JSX allows JavaScript expressions inside {}
}

export default App;
```

### **Why JSX?**
- **More readable**: Looks like HTML, making UI structures easy to understand.
- **Compile-time safety**: Errors are caught early.
- **Performance optimizations**: JSX produces optimized `React.createElement()` calls.