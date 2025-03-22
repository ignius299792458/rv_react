# **Props**  

Props (**properties**) allow React components to **`pass data from parent to child`**. They are **`immutable`** (read-only) inside the child component.

---

## **1. What Happens When Props Are Passed?**
When a parent component passes props to a child:
1. **The Parent Component Executes** → Calls the Child Component.
2. **React Creates a Virtual DOM Representation of the Child**.
3. **The Child Receives the Props as a Function Argument (for Function Components) or as `this.props` (for Class Components)**.
4. **If Props Change, React Triggers Reconciliation**.

---

## **2. Functional Components with Props**
### **Example**
```jsx
function Greeting(props) {
  return <h1>Hello, {props.name}!</h1>;
}

function App() {
  return <Greeting name="Ignius" />;
}

export default App;
```

---

### **What Happens Internally?**
#### **Step 1: JSX Gets Transformed**
The JSX:
```jsx
<Greeting name="Ignius" />
```
is **converted by Babel** into:
```js
React.createElement(Greeting, { name: "Ignius" }, null);
```

#### **Step 2: Virtual DOM Representation**
React **creates a Virtual DOM object**:
```json
{
  "type": "Greeting",
  "props": {
    "name": "Ignius"
  }
}
```

#### **Step 3: Component Execution**
React **calls the function**:
```js
Greeting({ name: "Ignius" });
```
which returns:
```js
React.createElement("h1", null, "Hello, Ignius!");
```

#### **Step 4: Real DOM Update**
React **diffs the new Virtual DOM with the old one** and updates only the changed parts.

---

## **3. Class Components with Props**
Before hooks, class components used `this.props`.

### **Example**
```jsx
import React, { Component } from "react";

class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

function App() {
  return <Greeting name="Ignius" />;
}

export default App;
```

---

### **What Happens Internally?**
#### **Step 1: React Creates an Instance**
React **instantiates the class**:
```js
const instance = new Greeting({ name: "Ignius" });
```
#### **Step 2: Calls `render()`**
React **calls**:
```js
instance.render();
```
which returns:
```js
React.createElement("h1", null, "Hello, Ignius!");
```

#### **Step 3: React Updates the Virtual DOM**
```json
{
  "type": "h1",
  "props": { "children": "Hello, Ignius!" }
}
```

---

## **4. Props Are Immutable**
Props **cannot be changed** inside a child component.

### ❌ **Bad Example (Modifying Props)**
```jsx
function Greeting(props) {
  props.name = "New Name"; // ❌ This will throw an error
  return <h1>Hello, {props.name}!</h1>;
}
```
**Why?**  
- React **treats props as immutable data**.
- Modifying props **breaks component reusability**.
- This **violates React’s declarative approach**.

---

## **5. Default Props & Prop Types**
### **Default Props (If No Value is Passed)**
```jsx
function Greeting(props) {
  return <h1>Hello, {props.name}!</h1>;
}

Greeting.defaultProps = {
  name: "Guest",
};
```
If `<Greeting />` is used without a `name` prop, `"Guest"` is displayed.

---

### **Prop Type Validation**
To prevent **wrong prop types**, we use `prop-types`:
```jsx
import PropTypes from "prop-types";

function Greeting({ name, age }) {
  return (
    <h1>
      Hello, {name}! You are {age} years old.
    </h1>
  );
}

Greeting.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
};
```
If `name` is missing or not a string, React logs a warning.

---

## **6. Props with Arrays & Objects**
Props can **pass complex data** like arrays and objects.

### **Passing an Array**
```jsx
function List({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  );
}

function App() {
  return <List items={["Apple", "Banana", "Cherry"]} />;
}
```

---

### **Passing an Object**
```jsx
function UserProfile({ user }) {
  return (
    <h1>
      {user.name} is {user.age} years old.
    </h1>
  );
}

function App() {
  const userData = { name: "Ignius", age: 25 };
  return <UserProfile user={userData} />;
}
```
**Internally**, React receives `props` as:
```json
{
  "user": { "name": "Ignius", "age": 25 }
}
```

---

## **7. Props with `useEffect()` (Machine-Level Behavior)**
When a prop changes, React **triggers a re-render**.

```jsx
import { useState, useEffect } from "react";

function DisplayMessage({ message }) {
  useEffect(() => {
    console.log("Message updated:", message);
  }, [message]); // Runs when `message` changes

  return <h1>{message}</h1>;
}

function App() {
  const [msg, setMsg] = useState("Hello!");

  return (
    <div>
      <DisplayMessage message={msg} />
      <button onClick={() => setMsg("New Message!")}>Update</button>
    </div>
  );
}
```

### **What Happens Internally?**
1. `useEffect()` runs **when `message` changes**.
2. React **calls `DisplayMessage()` again** with new props.
3. **Reconciliation** checks if the Virtual DOM changed.
4. **Updates only the changed parts** in the real DOM.

---

## **8. Performance Optimization: `React.memo()`**
`React.memo()` prevents unnecessary re-renders.

```jsx
const Greeting = React.memo(({ name }) => {
  console.log("Rendered!");
  return <h1>Hello, {name}!</h1>;
});

function App() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <Greeting name="Ignius" />
      <button onClick={() => setCount(count + 1)}>Click Me</button>
    </div>
  );
}
```
Since `name="Ignius"` never changes, `Greeting` **won’t re-render** when clicking the button.

---

## **Final Thoughts**
- **Props are immutable** → Prevents accidental modification.
- **Props trigger re-renders** when changed.
- **Default Props** ensure values exist.
- **PropTypes** enforce type safety.
- **React.memo()** prevents unnecessary re-renders.

---