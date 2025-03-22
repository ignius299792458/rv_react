# 1. **Context API**

The Context API is a powerful tool in React for `managing global state without prop-drilling`. It **allows you to `share state` between components at different levels of the component tree**.

- **Creating Context:**

  ```jsx
  const MyContext = React.createContext();
  ```

  The context is created with `React.createContext()`, which provides a

  1. `Provider` component
  2. `Consumer` component

- **Using the Provider:**

  The `Provider` is used to wrap your component tree and pass down the state to child components.

  ```jsx
  <MyContext.Provider value={{ user: "Ignius" }}>
    <ChildComponent />
  </MyContext.Provider>
  ```

- **Consuming Context:**

  To consume the context in a child component, you use the `useContext` hook or the `Consumer` component.

  ```jsx
  const context = useContext(MyContext);
  console.log(context.user); // 'Ignius'
  ```

- **Updating Context:**

  First, create a context with some default values and provide a function to update those values.

    ```jsx
    import React, { createContext, useState } from "react";

    // Create a context
    const MyContext = createContext();

    const MyProvider = ({ children }) => {
    const [user, setUser] = useState("Ignius");

    // Function to update the user
    const updateUser = (newUser) => {
        setUser(newUser);
    };

    return (
        <MyContext.Provider value={{ user, updateUser }}>
        {children}
        </MyContext.Provider>
    );
    };

    export { MyContext, MyProvider };
    ```