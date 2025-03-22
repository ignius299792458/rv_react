# **Hooks 🪝**  

Hooks revolutionized React by **allowing functional components `to manage state and lifecycle events without needing class components`**. To fully understand how they work, we need to look at:  

1️⃣ **How React manages state internally**  
2️⃣ **How hooks interact with the Fiber reconciliation algorithm**  
3️⃣ **How Virtual DOM updates and re-renders with hooks**  

---

# 1. **`How React Manages State Internally`**
React’s state is **not stored in the component itself** but in a **global linked list of hooks** managed internally.  

### **💡 Key Internal Mechanisms**
- React **does not store hook states in the function itself** (functions do not have memory).  
- Instead, it **creates and maintains a linked list of hooks in Fiber's memory**.  
- **Each component instance has its own state** stored in a Fiber node.  

### **🔹 Understanding Fiber (React’s Internal Engine)**
- React **Fiber** is a **work-loop based scheduler** that enables asynchronous rendering.  
- It **divides rendering work into small chunks** and schedules updates efficiently.  
- Each component **has a corresponding Fiber node** that holds hook states.  

### **🔍 Internal Representation of Hooks (Simplified)**
Internally, React manages hooks **like a linked list** attached to the component’s Fiber node:  
```tsx
const hooks = [
  { state: 10, next: Hook2 },  // useState(10)
  { state: 20, next: Hook3 },  // useState(20)
  { state: refObject, next: null } // useRef()
];
```
- Hooks are processed **in the order they are declared** in the function.  
- **Order matters**! If you conditionally call a hook, the list will break, causing errors.  

---

# 2. **`How Hooks Interact with Fiber and Virtual DOM`**
### **💡 Virtual DOM Reconciliation Process with Hooks**
1. React **`calls the component function`** to build a new Virtual DOM tree.  
2. **`React Fiber schedules the updates`** and compares the old and new trees.  
3. Hooks **`retrieve state from the global linked list`** and compute updates.  
4. If state changes, React triggers **`a re-render and reconciliation`**.  

### **🛠 Key Behavior:**
- **`No direct state mutation`**: Hooks always return **new state references**, forcing React to detect changes and trigger re-renders.  
- **`Efficient updates`**: React **batches state updates** inside event handlers for performance.  
- **`Optimized reconciliation`**: React **skips re-renders if states/props haven’t changed** using **Fiber diffing**.  

---

# **🛠 Low-Level Explanation of Key Hooks**
Now, let’s analyze **each important hook** at a machine level.  

## **1️⃣ `useState` - Managing Component State**
### **📌 How `useState` Works Internally**
```tsx
const [count, setCount] = useState(0);
```
- When `useState(0)` runs:
  1. React **allocates a hook object** in Fiber’s memory (linked list).  
  2. The **initial value (0) is stored** in this object.  
  3. The function `setCount(newValue)` **does not mutate state**, instead:  
     - Creates a **new version** of the Fiber node.  
     - React schedules a re-render.  
  4. During the next render, `useState()` **retrieves the stored value** instead of re-initializing it.  

### **🛠 Internal Representation of `useState`**
```tsx
const hooks = [
  { state: 0, next: Hook2 } // First useState()
];
```
- The **linked list persists across renders**, so each `useState` call refers to the correct state.  
- **Calling `setState` schedules a Fiber update**, triggering reconciliation.  

---

## **2️⃣ `useEffect` - Managing Side Effects**
### **📌 How `useEffect` Works Internally**
```tsx
useEffect(() => {
  console.log("Effect runs");
}, [dependency]);
```
1. **During Initial Render**:
   - The effect function is **stored in Fiber memory**.
   - React marks it as **pending execution after commit**.
   - After the component is **painted**, the effect runs.

2. **During Subsequent Renders**:
   - React **compares the dependency array** (`[dependency]`).
   - If **dependencies haven’t changed**, the effect **skips execution**.
   - If they **have changed**, React re-runs the effect.

3. **On Unmount**:
   - The **cleanup function runs** (`return () => {...}`).
   - This is useful for **removing event listeners or cleaning up timers**.

### **🛠 Internal Representation of `useEffect`**
```tsx
const hooks = [
  { state: 0, next: Hook2 }, // useState()
  { effect: cleanupFunction, dependencies: [dependency], next: null } // useEffect()
];
```
- React **checks dependencies and schedules effect execution** after rendering.

---

## **3️⃣ `useRef` - Managing Persistent References**
### **📌 How `useRef` Works Internally**
```tsx
const inputRef = useRef(null);
```
- Unlike `useState`, **`useRef` does not trigger re-renders**.
- It **creates a stable object reference** that persists across renders.
- The `current` property can be **mutated without causing re-renders**.

### **🛠 Internal Representation of `useRef`**
```tsx
const hooks = [
  { state: 0, next: Hook2 }, // useState()
  { ref: { current: null }, next: Hook3 } // useRef()
];
```
- The `ref.current` value is **directly mutable** without Fiber reconciliation.

---

## **4️⃣ `useReducer` - Managing Complex State Logic**
### **📌 How `useReducer` Works Internally**
```tsx
const [state, dispatch] = useReducer(reducer, initialState);
```
- Works like `useState`, but with **an external reducer function**.  
- Instead of directly setting state, we **dispatch actions** to modify state.  

### **🛠 Internal Representation of `useReducer`**
```tsx
const hooks = [
  { state: { count: 0 }, reducer: reducerFunction, next: null } // useReducer()
];
```
- When `dispatch(action)` is called:
  1. React **calls the reducer function** with `(state, action)`.  
  2. If the new state is different, React **schedules an update**.  
  3. **Reconciliation updates the Virtual DOM** with the new state.

---

# **🔹 Summary of Hook Behavior in Fiber**
| Hook       | Stored In | Triggers Re-Render? | Updates Fiber? |
|------------|----------|---------------------|----------------|
| `useState`  | Fiber memory (linked list) | ✅ Yes | ✅ Yes |
| `useEffect` | Fiber side effects queue | ❌ No | ✅ Yes (only on dependency change) |
| `useRef`    | Fiber memory (mutable ref object) | ❌ No | ❌ No |
| `useReducer` | Fiber memory (linked list) | ✅ Yes | ✅ Yes |

---

# **🎯 Key Takeaways**
1️⃣ **Hooks are stored in a linked list in Fiber’s memory**.  
2️⃣ **State updates create new Fiber nodes, triggering Virtual DOM reconciliation**.  
3️⃣ **Hooks must always be called in the same order** (React relies on indexing).  
4️⃣ **Re-rendering is optimized using dependency tracking** (`useEffect`, `useMemo`).  
5️⃣ **React Fiber schedules and batches state updates for performance**.  

---

# **🚀 Working Steps**

1️⃣ **Compare Class Components vs Hooks with Fiber?**  
2️⃣ **Understand concurrent rendering & React 18 updates?**  
3️⃣ **Deep dive into custom hooks and memoization (`useMemo`, `useCallback`)?**  
