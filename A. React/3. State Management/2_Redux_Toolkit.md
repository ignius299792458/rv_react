# Redux Toolkit

Redux Toolkit (RTK) is the official, opinionated, batteries-included toolset for efficient Redux development. It simplifies the Redux experience by providing utilities to streamline setup, reduce boilerplate code, and implement Redux best practices automatically.

## Core Concepts and Philosophy

Redux Toolkit was created to address three common concerns with vanilla Redux:
1. "Configuring a Redux store is too complicated"
2. "I have to add a lot of packages to get Redux to do anything useful"
3. "Redux requires too much boilerplate code"

It incorporates best practices, simplifies most Redux tasks, and prevents common mistakes.

## Key Features of Redux Toolkit

### 1. `configureStore`

`configureStore` wraps the Redux `createStore` function with simplified configuration options and good defaults:

```javascript
import { configureStore } from '@reduxjs/toolkit';
import rootReducer from './reducers';

const store = configureStore({
  reducer: rootReducer,
  // Optional configurations:
  middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat(logger),
  devTools: process.env.NODE_ENV !== 'production',
  preloadedState: initialState,
  enhancers: [reduxBatch],
});

export default store;
```

**What it does automatically:**
- Combines your slice reducers into the root reducer
- Adds redux-thunk middleware for async logic
- Enables Redux DevTools Extension
- Sets up middleware to catch common mistakes like mutating state

### 2. `createSlice`

`createSlice` is the cornerstone of Redux Toolkit that generates action creators and action types based on the reducer functions you supply:

**Key advantages of `createSlice`:**
- Generates action creators and action types automatically
- Uses Immer internally to let you write "mutating" update logic that actually produces immutable updates
- Eliminates the need for switch-case statements in reducers
- Simplifies handling complex nested state through "mutative" code

### 3. `createAsyncThunk`

`createAsyncThunk` generates thunk functions for handling async requests with automatic action dispatching:



**What `createAsyncThunk` does:**
- Creates three action types: pending, fulfilled, and rejected
- Automatically dispatches these actions based on the Promise lifecycle
- Handles Promise rejection and serialization automatically
- Provides helpful tools for handling loading states and errors

### 4. `createEntityAdapter`

`createEntityAdapter` provides a standardized way to store normalized data in your Redux store:



**Benefits of entity adapters:**
- Standardized CRUD operations
- Normalized state shape (ids array and entities object)
- Generated selectors for common operations
- Improved performance for large collections

### 5. RTK Query

RTK Query is the newest addition to Redux Toolkit - it's a powerful data fetching and caching tool built into Redux Toolkit:



**Key features of RTK Query:**
- Automatic fetching, caching, and cache invalidation
- Normalized cache and request deduplication
- Loading and error state tracking
- Automatic re-fetching after mutations
- Optimistic updates
- Polling and pagination support
- Auto-generated React hooks
- TypeScript support

# Advanced Redux Toolkit Concepts

### 1. Middleware Configuration

Redux Toolkit provides powerful middleware customization:

```javascript
import { configureStore } from '@reduxjs/toolkit';
import logger from 'redux-logger';
import rootReducer from './reducers';

const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) => 
    getDefaultMiddleware({
      // Customize middleware options
      thunk: {
        extraArgument: { api: myApiService }
      },
      serializableCheck: {
        // Ignore these field paths in the state when checking serialization
        ignoredActions: ['some/action'],
        ignoredPaths: ['items.dates'],
      },
      immutableCheck: {
        // Ignore deep paths in the state when checking for mutations
        ignoredPaths: ['ignoredPath', 'ignoredPath.nestedPath'],
      },
    }).concat(logger),
});

// Using the extra argument in a thunk
const fetchItems = () => async (dispatch, getState, { api }) => {
  const response = await api.getItems();
  dispatch(itemsLoaded(response.data));
};
```

### 2. Redux Toolkit and TypeScript

Redux Toolkit is designed with excellent TypeScript support:





**TypeScript benefits in Redux Toolkit:**
- Type-safe state and action payloads
- Automatic action type inference
- Type-safe selectors
- Typed hooks for useDispatch and useSelector
- Intellisense/autocomplete support

### 3. Performance Optimization Techniques

Redux Toolkit provides several tools for performance optimization:

#### Memoized Selectors with `createSelector`

```javascript
import { createSelector } from '@reduxjs/toolkit';

// Base selectors
const selectItems = state => state.items;
const selectFilter = state => state.filter;

// Memoized selector
export const selectFilteredItems = createSelector(
  [selectItems, selectFilter],
  (items, filter) => {
    // This calculation only reruns when inputs change
    return items.filter(item => {
      if (filter === 'all') return true;
      if (filter === 'active') return !item.completed;
      if (filter === 'completed') return item.completed;
      return true;
    });
  }
);

// In component:
const filteredItems = useSelector(selectFilteredItems);
```

#### Using `entityAdapter.getSelectors` with Memoization

```javascript
// Selector with factory functions to create selectors for specific state
export const {
  selectAll: selectAllPosts,
  selectById: selectPostById,
  selectIds: selectPostIds
} = postsAdapter.getSelectors(state => state.posts);

// Further composition with createSelector
export const selectPostsByUser = createSelector(
  [selectAllPosts, (state, userId) => userId],
  (posts, userId) => posts.filter(post => post.userId === userId)
);
```

### 4. Custom Serializable State Checking

```javascript
import { configureStore, isPlain } from '@reduxjs/toolkit';

// Custom isSerializable function
const isSerializable = (value) => {
  // Allow non-serializable values in specific cases
  if (value instanceof MySpecialClass) return true;
  return isPlain(value);
};

const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        isSerializable,
        // Ignore specific paths
        ignoredActions: ['special/action'],
        ignoredActionPaths: ['meta.arg', 'payload.timestamp'],
        ignoredPaths: ['items.special'],
      },
    }),
});
```

### 5. Dynamic State Injection with Redux Toolkit

For code splitting and lazy loading reducers:

```javascript
import { combineReducers } from '@reduxjs/toolkit';

// Initial static reducers
const staticReducers = {
  users: usersReducer,
  posts: postsReducer,
};

// Setup for dynamic reducer injection
export function configureAppStore(initialState = {}) {
  const rootReducer = createReducer();
  
  const store = configureStore({
    reducer: rootReducer,
    preloadedState: initialState
  });
  
  // Add a dictionary to track lazy-loaded reducers
  store.asyncReducers = {};
  
  // Create a function to add a reducer later
  store.injectReducer = (key, asyncReducer) => {
    store.asyncReducers[key] = asyncReducer;
    store.replaceReducer(createReducer(store.asyncReducers));
    return store;
  };
  
  return store;
}

function createReducer(asyncReducers = {}) {
  return combineReducers({
    ...staticReducers,
    ...asyncReducers
  });
}

// Usage in a component:
// When a component that needs a feature mounts:
const SomeComponent = () => {
  useEffect(() => {
    // Dynamically load a reducer
    import('./featureReducer').then(({ default: reducer }) => {
      store.injectReducer('feature', reducer);
    });
  }, []);
  
  // ...component code
};
```
