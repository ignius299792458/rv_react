```JSX

// 1. Create types for your actions (optional but recommended)
const ACTIONS = {
  ADD_TODO: 'add-todo',
  TOGGLE_TODO: 'toggle-todo',
  DELETE_TODO: 'delete-todo'
};

// 2. Create the initial state
const initialState = {
  todos: [],
  loading: false,
  error: null
};

// 3. Create the reducer function
function todoReducer(state, action) {
  switch (action.type) {
    case ACTIONS.ADD_TODO:
      return {
        ...state,
        todos: [...state.todos, {
          id: Date.now(),
          text: action.payload.text,
          completed: false
        }]
      };
    case ACTIONS.TOGGLE_TODO:
      return {
        ...state,
        todos: state.todos.map(todo => 
          todo.id === action.payload.id 
            ? { ...todo, completed: !todo.completed } 
            : todo
        )
      };
    case ACTIONS.DELETE_TODO:
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload.id)
      };
    default:
      return state;
  }
}

// 4. Create the Context
import React, { createContext, useContext, useReducer } from 'react';

const TodoContext = createContext();

// 5. Create a Context Provider component
export function TodoProvider({ children }) {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  
  // Optional: Create action creators for better abstraction
  const addTodo = (text) => {
    dispatch({ type: ACTIONS.ADD_TODO, payload: { text } });
  };
  
  const toggleTodo = (id) => {
    dispatch({ type: ACTIONS.TOGGLE_TODO, payload: { id } });
  };
  
  const deleteTodo = (id) => {
    dispatch({ type: ACTIONS.DELETE_TODO, payload: { id } });
  };
  
  // Create a value object with state and actions
  const value = {
    todos: state.todos,
    loading: state.loading,
    error: state.error,
    addTodo,
    toggleTodo,
    deleteTodo
  };
  
  return (
    <TodoContext.Provider value={value}>
      {children}
    </TodoContext.Provider>
  );
}

// 6. Create a custom hook for using this context
export function useTodos() {
  const context = useContext(TodoContext);
  if (context === undefined) {
    throw new Error('useTodos must be used within a TodoProvider');
  }
  return context;
}

// 7. Use in your application
import { TodoProvider, useTodos } from './TodoContext';

function App() {
  return (
    <TodoProvider>
      <TodoList />
      <AddTodoForm />
    </TodoProvider>
  );
}

function TodoList() {
  const { todos, toggleTodo, deleteTodo } = useTodos();
  
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <span 
            style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
            onClick={() => toggleTodo(todo.id)}
          >
            {todo.text}
          </span>
          <button onClick={() => deleteTodo(todo.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}

function AddTodoForm() {
  const { addTodo } = useTodos();
  const [text, setText] = React.useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      addTodo(text);
      setText('');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input 
        value={text} 
        onChange={(e) => setText(e.target.value)} 
        placeholder="Add todo" 
      />
      <button type="submit">Add</button>
    </form>
  );
}

```