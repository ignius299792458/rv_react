# React Context + useReducer

`React Context` and `useReducer` are two powerful built-in features in React that, when combined, provide `a robust state management solution for medium to large applications`.

# React Context

## What is React Context?

React Context provides a **way to pass data through the component tree without having to pass props down manually at every level (`without props drilling`)**. It's designed to share data that can be considered "`global`" for a tree or banch of tree of React components.

## When to Use Context

Context is primarily used when:
- Data needs to be accessible by many components at different nesting levels
- Prop drilling becomes excessive and hard to maintain
- You need a lightweight state management solution without external libraries

## Key Context API Components

### 1. `React.createContext`

```jsx
const MyContext = React.createContext(defaultValue);
```

This creates a Context object. When React renders a component that subscribes to this Context, it will read the current context value from the closest matching Provider above it in the tree.

### 2. `Context.Provider`

```jsx
<MyContext.Provider value={/* some value */}>
  {children}
</MyContext.Provider>
```

The Provider component accepts a `value` prop that will be passed to consuming components that are descendants of this Provider.

### 3. `useContext` Hook

```jsx
const value = useContext(MyContext);
```

This hook lets you subscribe to a context within a function component. It returns the current context value for that context.

## Context Implementation Example

```jsx
// Create context
const ThemeContext = React.createContext('light');

// Parent component with Provider
function App() {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <MainContent />
    </ThemeContext.Provider>
  );
}

// Nested child component
function ThemedButton() {
  const { theme, setTheme } = useContext(ThemeContext);
  
  return (
    <button
      style={{ background: theme === 'dark' ? '#333' : '#fff', color: theme === 'dark' ? '#fff' : '#333' }}
      onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
    >
      Toggle Theme
    </button>
  );
}
```

# `useReducer` Hook

## What is useReducer?

`useReducer` is a React Hook that is an alternative to `useState` for complex state logic. It's inspired by Redux and follows the reducer pattern.

## When to Use useReducer

Use `useReducer` when:
- State logic is complex involving multiple sub-values
- The next state depends on the previous state
- You need more predictable state transitions
- You want to centralize state update logic

## useReducer Syntax

```jsx
const [state, dispatch] = useReducer(reducer, initialState, init);
```

- `reducer`: A function that takes the current state and an action, and returns the new state
- `initialState`: The initial state value
- `init` (optional): A function to lazily create the initial state

## Reducer Function

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}
```

## useReducer Example

```jsx
function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
    </>
  );
}
```
---
---

# Combining Context and useReducer

When combined, Context and useReducer provide a powerful state management solution similar to Redux but built into React.

## Implementation Pattern

This pattern provides several benefits:

1. **Centralized State Logic**: All state updates are handled by the reducer
2. **Predictable State Changes**: Actions describe changes clearly
3. **Easier Testing**: Pure reducer functions are simple to test
4. **Dev Tools Support**: Works well with React DevTools
5. **No Prop Drilling**: Components can access state without passing props

## Advanced Patterns and Best Practices

## 1. Multiple Contexts

For large applications, consider splitting your context by domain:

```jsx
<AuthProvider>
  <ThemeProvider>
    <UserProvider>
      <TodoProvider>
        <App />
      </TodoProvider>
    </UserProvider>
  </ThemeProvider>
</AuthProvider>
```

## 2. Memoizing Context Value

To prevent unnecessary re-renders, memoize your context value:

```jsx
function TodoProvider({ children }) {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  
  // Memoize value object to prevent unnecessary re-renders
  const value = useMemo(() => {
    return {
      todos: state.todos,
      loading: state.loading,
      // other state properties and actions
    };
  }, [state]);
  
  return (
    <TodoContext.Provider value={value}>
      {children}
    </TodoContext.Provider>
  );
}
```

## 3. Async Actions with useReducer

For handling asynchronous operations:

```jsx
function todoReducer(state, action) {
  switch (action.type) {
    case 'FETCH_TODOS_START':
      return { ...state, loading: true };
    case 'FETCH_TODOS_SUCCESS':
      return { ...state, loading: false, todos: action.payload };
    case 'FETCH_TODOS_ERROR':
      return { ...state, loading: false, error: action.payload };
    // other cases
  }
}

// In your provider:
async function fetchTodos() {
  dispatch({ type: 'FETCH_TODOS_START' });
  try {
    const response = await api.getTodos();
    dispatch({ type: 'FETCH_TODOS_SUCCESS', payload: response.data });
  } catch (error) {
    dispatch({ type: 'FETCH_TODOS_ERROR', payload: error.message });
  }
}
```

## 4. TypeScript Integration

With TypeScript, you can add strong typing to your context and reducer:

```typescript
// Define action types
type Action = 
  | { type: 'add-todo'; payload: { text: string } }
  | { type: 'toggle-todo'; payload: { id: number } }
  | { type: 'delete-todo'; payload: { id: number } };

// Define state type
interface State {
  todos: Array<{ id: number; text: string; completed: boolean }>;
  loading: boolean;
  error: string | null;
}

// Type the context
interface TodoContextType extends State {
  addTodo: (text: string) => void;
  toggleTodo: (id: number) => void;
  deleteTodo: (id: number) => void;
}

const TodoContext = createContext<TodoContextType | undefined>(undefined);
```

## 5. Performance Considerations

- Use `React.memo` for components that only need to re-render when props change
- Split your context into smaller contexts if different parts of your app need different pieces of state
- Use the `useCallback` hook for functions passed down as props to prevent unnecessary re-renders

## When to Move Beyond Context + useReducer

While Context + useReducer works well for many applications, consider alternatives when:

1. You have very complex state with many levels or branches
2. You need robust middleware support
3. You require time-travel debugging
4. Your app has significant performance demands with large state objects
5. You need strong ecosystem support with many extensions

In those cases, libraries like Redux Toolkit, Zustand, or Recoil might be more appropriate.

## Conclusion

React Context combined with useReducer provides a powerful, built-in state management solution that works well for many applications. It offers:

- Centralized state management without external libraries
- Predictable state updates through the reducer pattern
- Elimination of prop drilling
- Good performance for small to medium applications

By understanding these patterns deeply, you can build maintainable React applications with clean, predictable state management.