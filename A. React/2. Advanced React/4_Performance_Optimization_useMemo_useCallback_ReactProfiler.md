# 4. **Performance Optimization (`useMemo`, `useCallback`, React Profiler)**

## `useMemo`:

`useMemo` is used to memoize a calculation so that it is only recomputed when its dependencies change. This helps avoid unnecessary recalculations of expensive operations.

```jsx
const expensiveComputation = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

- **When to use:** When you have an expensive calculation or transformation and you want to avoid recalculating it unless the input values change.

## `useCallback`:

`useCallback` memoizes the function itself, which is helpful when passing functions to child components to prevent unnecessary re-renders.

```jsx
const handleClick = useCallback(() => {
  console.log("Clicked");
}, []); // Memoize the function
```

- **When to use:** Use it to optimize function references that get passed down to child components, especially in case of performance issues in child components.

---

## Real-World Example for `useMemo` and `useCallback`

Real-world examples for both `useMemo` and `useCallback`, and then we'll compare `useMemo` with `React.memo`.

### 1. **`useMemo` Example:**

Imagine you're building a product filtering UI where the user can filter a list of products based on price ranges. The list of products is large, so calculating the filtered results can be computationally expensive.

```jsx
import React, { useState, useMemo } from "react";

const ProductList = ({ products }) => {
  const [minPrice, setMinPrice] = useState(0);
  const [maxPrice, setMaxPrice] = useState(1000);

  // useMemo to memoize the filtered products based on price range
  const filteredProducts = useMemo(() => {
    console.log("Filtering products...");
    return products.filter(
      (product) => product.price >= minPrice && product.price <= maxPrice
    );
  }, [minPrice, maxPrice, products]); // Only re-compute when minPrice, maxPrice, or products change

  return (
    <div>
      <h1>Product List</h1>
      <div>
        <label>Min Price: </label>
        <input
          type="number"
          value={minPrice}
          onChange={(e) => setMinPrice(e.target.value)}
        />
      </div>
      <div>
        <label>Max Price: </label>
        <input
          type="number"
          value={maxPrice}
          onChange={(e) => setMaxPrice(e.target.value)}
        />
      </div>
      <ul>
        {filteredProducts.map((product) => (
          <li key={product.id}>
            {product.name} - ${product.price}
          </li>
        ))}
      </ul>
    </div>
  );
};

export default ProductList;
```

**Explanation:**

- **useMemo** is used to **memoize** the filtered list of products. The expensive filtering calculation will only happen when `minPrice`, `maxPrice`, or `products` change. Without `useMemo`, the `filter` method would be called every time the component re-renders (even if the filter inputs have not changed), which could cause performance issues with large datasets.

### 2. **useCallback Example:**

Now, let's take the same `ProductList` example and add a function that needs to be passed to child components, such as a callback to add a product to the cart.

```jsx
import React, { useState, useCallback } from "react";

const ProductList = ({ products, onAddToCart }) => {
  const [minPrice, setMinPrice] = useState(0);
  const [maxPrice, setMaxPrice] = useState(1000);

  const handleAddToCart = useCallback(
    (product) => {
      onAddToCart(product);
    },
    [onAddToCart]
  ); // Re-memoize only if onAddToCart function changes

  const filteredProducts = products.filter(
    (product) => product.price >= minPrice && product.price <= maxPrice
  );

  return (
    <div>
      <h1>Product List</h1>
      <div>
        <label>Min Price: </label>
        <input
          type="number"
          value={minPrice}
          onChange={(e) => setMinPrice(e.target.value)}
        />
      </div>
      <div>
        <label>Max Price: </label>
        <input
          type="number"
          value={maxPrice}
          onChange={(e) => setMaxPrice(e.target.value)}
        />
      </div>
      <ul>
        {filteredProducts.map((product) => (
          <li key={product.id}>
            {product.name} - ${product.price}
            <button onClick={() => handleAddToCart(product)}>
              Add to Cart
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default ProductList;
```

**Explanation:**

- **useCallback** is used to **memoize** the `handleAddToCart` function. The function is only re-created when `onAddToCart` changes. Without `useCallback`, the `handleAddToCart` function would be recreated on every render, even if its dependencies (`onAddToCart`) hadn't changed. This can be especially helpful when passing this callback to deeply nested child components to prevent unnecessary re-renders.

## **Difference Between `useMemo` and `React.memo`:**

#### `useMemo`:

- `useMemo` is a React Hook that is used to memoize **values** (such as the result of a computation) across renders.
- It helps to optimize performance by preventing expensive calculations unless their dependencies change.
- `useMemo` is used inside a component to cache the result of a function (e.g., calculation, filtering, etc.) between renders.

  **Example:**

  ```jsx
  const filteredProducts = useMemo(() => {
    return products.filter(
      (product) => product.price >= minPrice && product.price <= maxPrice
    );
  }, [minPrice, maxPrice, products]);
  ```

#### `React.memo`:

- `React.memo` is a higher-order component (HOC) used to memoize **components**, preventing unnecessary re-renders when the props passed to the component have not changed.
- It's particularly useful for functional components that rely heavily on props but do not always change across renders.

  **Example:**

  ```jsx
  const ProductItem = React.memo(({ product }) => {
    console.log("Rendering ProductItem");
    return (
      <li>
        {product.name} - ${product.price}
      </li>
    );
  });
  ```

  - In this case, `ProductItem` will only re-render if the `product` prop changes. If the parent component re-renders without changing the `product` prop, `React.memo` will prevent the unnecessary re-render of `ProductItem`.

### **Summary of Differences:**

- **`useMemo`** is used to **memoize values** (like computations or transformations) within a component.
- **`React.memo`** is used to **memoize components**, preventing them from re-rendering when their props haven't changed.

Both serve the purpose of optimizing performance but are applied in different contexts:

- Use `useMemo` to optimize expensive calculations within a component.
- Use `React.memo` to prevent unnecessary re-renders of functional components based on props.

Let me know if you need more details or have further questions!

## React Profiler:

React's built-in Profiler API lets you measure the performance of your React components.

```jsx
import { Profiler } from "react";

<Profiler
  id="MyComponent"
  onRender={(id, phase, actualDuration) => {
    console.log(`${id} rendered in ${actualDuration}ms`);
  }}
>
  <MyComponent />
</Profiler>;
```
