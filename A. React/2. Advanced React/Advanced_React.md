# 2. **Advanced React**

1. Context API & Compound Components
2. Custom Hooks (for reusability and separation of concerns)
3. Performance Optimization (useMemo, useCallback, React Profiler)
4. Error Boundaries & Suspense
5. React Server Components (RSC)

---

# React Engine and Babel

React relies on Babel to transpile JSX and modern JavaScript syntax (such as async/await, class properties) into code that browsers can understand. Babel is also used to transform JSX into `React.createElement` calls, which are the underlying JavaScript constructs that React uses.

- **React Engine**: The React engine is responsible for rendering components and managing updates to the Virtual DOM. It uses an optimized diffing algorithm to reconcile differences between the Virtual DOM and the real DOM efficiently.

In combination with Babel, React optimizes your code, allowing you to write modern JavaScript syntax and JSX, which is then transformed into optimized JavaScript that browsers can execute.

---
