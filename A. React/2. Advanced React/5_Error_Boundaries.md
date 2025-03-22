# 5. **Error Boundaries & Suspense**

## Error Boundaries:

`Error boundaries` are React components `that catch JavaScript errors anywhere in their child component tree`, log those errors, and display a fallback UI.

- **Example:**

  ```jsx
  class ErrorBoundary extends React.Component {
    constructor(props) {
      super(props);
      this.state = { hasError: false };
    }

    static getDerivedStateFromError(error) {
      return { hasError: true };
    }

    componentDidCatch(error, info) {
      console.log(error, info);
    }

    render() {
      if (this.state.hasError) {
        return <h1>Something went wrong.</h1>;
      }
      return this.props.children;
    }
  }
  ```

Use the error boundary like this:

```jsx
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>
```

## Example of **`Error Boundaries`** in React

Error Boundaries help to ensure that the app continues to work even if one part of the component tree crashes.

#### 1. **Create an Error Boundary Component**

Error boundaries are typically implemented as class components. Here's a basic implementation:

```jsx
import React, { Component } from 'react';

class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, info: null };
  }

  static getDerivedStateFromError(error) {
    // Update state to show fallback UI
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // Log error and info for debugging purposes
    console.error("Error captured:", error);
    console.error("Error info:", info);
    this.setState({ error, info });
  }

  render() {
    if (this.state.hasError) {
      // Render fallback UI
      return (
        <div>
          <h1>Something went wrong!</h1>
          <p>{this.state.error ? this.state.error.toString() : 'Unknown Error'}</p>
        </div>
      );
    }

    return this.props.children; // Render children if no error
  }
}

export default ErrorBoundary;
```

#### 2. **Using the Error Boundary**

Now, we can use this `ErrorBoundary` component to wrap other components in the application. If any child component inside the `ErrorBoundary` throws an error, the `ErrorBoundary` will catch it and display the fallback UI.

```jsx
import React, { useState } from 'react';
import ErrorBoundary from './ErrorBoundary';

// Example of a component that may throw an error
const BuggyComponent = () => {
  const [count, setCount] = useState(0);

  if (count === 5) {
    // Simulate an error
    throw new Error('I crashed!');
  }

  return (
    <div>
      <h2>Buggy Component</h2>
      <p>Current Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};

const App = () => (
  <div>
    <h1>My React Application</h1>
    {/* Wrap BuggyComponent with ErrorBoundary */}
    <ErrorBoundary>
      <BuggyComponent />
    </ErrorBoundary>
  </div>
);

export default App;
```

#### 3. **How It Works:**

- The `ErrorBoundary` component is a class component that implements two lifecycle methods: `getDerivedStateFromError` and `componentDidCatch`.
  - **`getDerivedStateFromError`**: This is a static method that is called when an error is thrown. It updates the state to indicate that an error occurred, so we can display a fallback UI.
  - **`componentDidCatch`**: This method logs the error details (both the error and info) for debugging purposes. It is called after an error has been thrown.
  
- In the `App` component, the `BuggyComponent` is wrapped with the `ErrorBoundary`. If `BuggyComponent` throws an error (like when the count reaches 5), the `ErrorBoundary` catches it and renders a fallback UI (i.e., "Something went wrong!").

#### 4. **Why Use Error Boundaries?**
- **Preventing App Crashes**: When a component throws an error, it doesn’t crash the entire app. Instead, the error boundary catches it and renders fallback UI.
- **Graceful Error Handling**: You can catch unexpected errors in any part of your app and render a user-friendly message.
- **Logging for Debugging**: `componentDidCatch` provides error details that can be logged or sent to a monitoring service for debugging.

#### 5. **Limitations of Error Boundaries**
- Error boundaries only catch errors in lifecycle methods, render methods, and constructors of class components. They do not catch errors in event handlers or asynchronous code (e.g., `setTimeout`, `Promise`).
- You can use `try/catch` for handling errors in event handlers and asynchronous code.

To handle errors in asynchronous code or event handlers, you can use try-catch blocks or promise error handling like this:

```jsx
const handleClick = async () => {
  try {
    // Simulate an async operation
    throw new Error("Async error");
  } catch (error) {
    console.error("Caught error:", error);
  }
};
```

### Summary:
- **Error boundaries** are essential for catching JavaScript errors in React components and ensuring the app doesn’t crash.
- They allow for fallback UI to be rendered when an error occurs.
- They help maintain a better user experience by logging errors and preventing UI breakage.
