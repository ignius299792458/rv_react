``` JSX
// File structure overview:
// - src/
//   - app/
//     - store.js         // Redux store configuration
//     - hooks.js         // Custom typed hooks
//   - features/
//     - users/           // Users feature domain
//       - usersSlice.js  // Redux slice for users
//       - UsersList.js   // React component
//     - posts/           // Posts feature domain
//       - postsSlice.js  // Redux slice for posts
//       - PostsList.js   // React component
//       - AddPostForm.js // Form component
//   - api/
//     - client.js        // API client configuration
//   - index.js           // App entry point

// ------------------------------------------------------
// File: app/store.js
// ------------------------------------------------------
import { configureStore } from '@reduxjs/toolkit';
import usersReducer from '../features/users/usersSlice';
import postsReducer from '../features/posts/postsSlice';
import { apiSlice } from '../api/apiSlice';

export const store = configureStore({
  reducer: {
    users: usersReducer,
    posts: postsReducer,
    [apiSlice.reducerPath]: apiSlice.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(apiSlice.middleware),
  devTools: process.env.NODE_ENV !== 'production',
});

// Infer types from store
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// ------------------------------------------------------
// File: app/hooks.js
// ------------------------------------------------------
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './store';

// Use throughout your app instead of plain useDispatch and useSelector
export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;

// ------------------------------------------------------
// File: api/apiSlice.js
// ------------------------------------------------------
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const apiSlice = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ 
    baseUrl: '/api',
    prepareHeaders: (headers, { getState }) => {
      // Add auth token to headers if it exists
      const token = (getState() as RootState).auth?.token;
      if (token) {
        headers.set('authorization', `Bearer ${token}`);
      }
      return headers;
    }
  }),
  tagTypes: ['Post', 'User'],
  endpoints: (builder) => ({
    // Endpoints will be injected from features
  }),
});

// ------------------------------------------------------
// File: features/users/usersSlice.js
// ------------------------------------------------------
import { 
  createSlice, 
  createAsyncThunk,
  createEntityAdapter,
  createSelector,
  EntityState,
  PayloadAction
} from '@reduxjs/toolkit';
import { client } from '../../api/client';
import { RootState } from '../../app/store';
import { apiSlice } from '../../api/apiSlice';

// Define User type
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
  status: 'active' | 'inactive';
}

// Create entity adapter for normalized state
const usersAdapter = createEntityAdapter<User>({
  // Optional: customize how entities are sorted
  sortComparer: (a, b) => a.name.localeCompare(b.name),
});

// Define initial state with additional fields
interface UsersState extends EntityState<User> {
  status: 'idle' | 'loading' | 'succeeded' | 'failed';
  error: string | null;
  selectedUserId: string | null;
}

const initialState = usersAdapter.getInitialState<Omit<UsersState, keyof EntityState<User>>>({
  status: 'idle',
  error: null,
  selectedUserId: null,
});

// Create async thunk for fetching users
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async (_, { rejectWithValue }) => {
    try {
      const response = await client.get('/users');
      return response.data;
    } catch (err) {
      return rejectWithValue(err.message);
    }
  }
);

// Create slice
const usersSlice = createSlice({
  name: 'users',
  initialState: initialState as UsersState,
  reducers: {
    userAdded: usersAdapter.addOne,
    userUpdated: usersAdapter.updateOne,
    userRemoved: usersAdapter.removeOne,
    allUsersRemoved: usersAdapter.removeAll,
    userSelected(state, action: PayloadAction<string>) {
      state.selectedUserId = action.payload;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.status = 'succeeded';
        // Add users to state array
        usersAdapter.setAll(state, action.payload);
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.payload as string || action.error.message || null;
      });
  },
});

// Export actions
export const {
  userAdded,
  userUpdated,
  userRemoved,
  allUsersRemoved,
  userSelected
} = usersSlice.actions;

// Export reducer
export default usersSlice.reducer;

// Get entity adapter selectors
export const {
  selectAll: selectAllUsers,
  selectById: selectUserById,
  selectIds: selectUserIds,
} = usersAdapter.getSelectors<RootState>((state) => state.users);

// Additional selectors
export const selectUserStatus = (state: RootState) => state.users.status;
export const selectUserError = (state: RootState) => state.users.error;
export const selectSelectedUserId = (state: RootState) => state.users.selectedUserId;

// Create memoized selector
export const selectSelectedUser = createSelector(
  [selectAllUsers, selectSelectedUserId],
  (users, selectedId) => users.find(user => user.id === selectedId) || null
);

// Extended API slice for users
export const extendedApiSlice = apiSlice.injectEndpoints({
  endpoints: (builder) => ({
    getUsers: builder.query<User[], void>({
      query: () => '/users',
      transformResponse: (responseData: User[]) => {
        // Sort by name as needed
        return responseData.slice().sort((a, b) => a.name.localeCompare(b.name));
      },
      providesTags: (result) =>
        result
          ? [
              ...result.map(({ id }) => ({ type: 'User' as const, id })),
              { type: 'User', id: 'LIST' },
            ]
          : [{ type: 'User', id: 'LIST' }],
    }),
    getUserById: builder.query<User, string>({
      query: (id) => `/users/${id}`,
      providesTags: (result, error, id) => [{ type: 'User', id }],
    }),
    addUser: builder.mutation<User, Partial<User>>({
      query: (user) => ({
        url: '/users',
        method: 'POST',
        body: user,
      }),
      invalidatesTags: [{ type: 'User', id: 'LIST' }],
    }),
    updateUser: builder.mutation<User, Partial<User> & Pick<User, 'id'>>({
      query: ({ id, ...patch }) => ({
        url: `/users/${id}`,
        method: 'PATCH',
        body: patch,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'User', id }],
    }),
    deleteUser: builder.mutation<{ success: boolean; id: string }, string>({
      query: (id) => ({
        url: `/users/${id}`,
        method: 'DELETE',
      }),
      invalidatesTags: (result, error, id) => [{ type: 'User', id }],
      // Optimistic update
      async onQueryStarted(id, { dispatch, queryFulfilled }) {
        const patchResult = dispatch(
          extendedApiSlice.util.updateQueryData('getUsers', undefined, (draft) => {
            const index = draft.findIndex(user => user.id === id);
            if (index !== -1) draft.splice(index, 1);
          })
        );
        try {
          await queryFulfilled;
        } catch {
          patchResult.undo();
        }
      },
    }),
  }),
});

// Export generated hooks
export const {
  useGetUsersQuery,
  useGetUserByIdQuery,
  useAddUserMutation,
  useUpdateUserMutation,
  useDeleteUserMutation,
} = extendedApiSlice;

// ------------------------------------------------------
// File: features/posts/postsSlice.js
// ------------------------------------------------------
import {
  createSlice,
  createAsyncThunk,
  createEntityAdapter,
  createSelector,
  EntityState,
  PayloadAction,
} from '@reduxjs/toolkit';
import { client } from '../../api/client';
import { RootState } from '../../app/store';
import { sub } from 'date-fns';

// Define the Post type
interface Post {
  id: string;
  title: string;
  content: string;
  userId: string;
  date: string;
  reactions: {
    thumbsUp: number;
    heart: number;
    rocket: number;
    wow: number;
  };
}

// Create entity adapter for normalized state
const postsAdapter = createEntityAdapter<Post>({
  sortComparer: (a, b) => b.date.localeCompare(a.date),
});

// Define additional state
interface PostsState extends EntityState<Post> {
  status: 'idle' | 'loading' | 'succeeded' | 'failed';
  error: string | null;
}

// Initialize with normalized entity state and additional fields
const initialState = postsAdapter.getInitialState<Omit<PostsState, keyof EntityState<Post>>>({
  status: 'idle',
  error: null,
});

// Thunk for fetching posts
export const fetchPosts = createAsyncThunk(
  'posts/fetchPosts',
  async (_, { rejectWithValue }) => {
    try {
      const response = await client.get('/posts');
      return response.data;
    } catch (err) {
      return rejectWithValue(err.message);
    }
  }
);

// Thunk for adding a new post
export const addNewPost = createAsyncThunk(
  'posts/addNewPost',
  async (initialPost: { title: string; content: string; userId: string }, { rejectWithValue }) => {
    try {
      // Create a new post object
      const post = {
        ...initialPost,
        date: new Date().toISOString(),
        reactions: {
          thumbsUp: 0,
          heart: 0,
          rocket: 0,
          wow: 0,
        },
      };

      const response = await client.post('/posts', post);
      return response.data;
    } catch (err) {
      return rejectWithValue(err.message);
    }
  }
);

// Create the posts slice
const postsSlice = createSlice({
  name: 'posts',
  initialState: initialState as PostsState,
  reducers: {
    postUpdated: {
      reducer(state, action: PayloadAction<Partial<Post> & Pick<Post, 'id'>>) {
        const { id, ...changes } = action.payload;
        postsAdapter.updateOne(state, { id, changes });
      },
      prepare(id: string, title: string, content: string, userId: string) {
        return {
          payload: {
            id,
            title,
            content,
            userId,
          },
        };
      },
    },
    reactionAdded(state, action: PayloadAction<{ postId: string; reaction: keyof Post['reactions'] }>) {
      const { postId, reaction } = action.payload;
      const existingPost = state.entities[postId];
      if (existingPost) {
        existingPost.reactions[reaction]++;
      }
    },
    postDeleted: postsAdapter.removeOne,
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchPosts.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.status = 'succeeded';
        // Update the normalized state with the fetched posts
        postsAdapter.upsertMany(state, action.payload);
      })
      .addCase(fetchPosts.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.payload as string || action.error.message || null;
      })
      .addCase(addNewPost.fulfilled, postsAdapter.addOne);
  },
});

// Export actions
export const { postUpdated, reactionAdded, postDeleted } = postsSlice.actions;

// Export reducer
export default postsSlice.reducer;

// Get the normalized selectors
export const {
  selectAll: selectAllPosts,
  selectById: selectPostById,
  selectIds: selectPostIds,
} = postsAdapter.getSelectors<RootState>((state) => state.posts);

// Additional selectors
export const selectPostsByUser = createSelector(
  [selectAllPosts, (_, userId: string) => userId],
  (posts, userId) => posts.filter((post) => post.userId === userId)
);

export const selectPostStatus = (state: RootState) => state.posts.status;
export const selectPostError = (state: RootState) => state.posts.error;

// ------------------------------------------------------
// File: features/posts/AddPostForm.js
// ------------------------------------------------------
import React, { useState } from 'react';
import { useAppDispatch, useAppSelector } from '../../app/hooks';
import { unwrapResult } from '@reduxjs/toolkit';
import { addNewPost } from './postsSlice';
import { selectAllUsers } from '../users/usersSlice';

export const AddPostForm = () => {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');
  const [userId, setUserId] = useState('');
  const [addRequestStatus, setAddRequestStatus] = useState('idle');

  const dispatch = useAppDispatch();
  const users = useAppSelector(selectAllUsers);

  const canSave = 
    [title, content, userId].every(Boolean) && addRequestStatus === 'idle';

  const onSavePostClicked = async () => {
    if (canSave) {
      try {
        setAddRequestStatus('pending');
        // Dispatch the thunk and get the result
        const resultAction = await dispatch(
          addNewPost({ title, content, userId })
        );
        // Unwrap to get the actual result or throw an error
        unwrapResult(resultAction);
        
        // Reset form
        setTitle('');
        setContent('');
        setUserId('');
      } catch (err) {
        console.error('Failed to save the post: ', err);
      } finally {
        setAddRequestStatus('idle');
      }
    }
  };

  const usersOptions = users.map(user => (
    <option key={user.id} value={user.id}>
      {user.name}
    </option>
  ));

  return (
    <section>
      <h2>Add a New Post</h2>
      <form>
        <label htmlFor="postTitle">Post Title:</label>
        <input
          type="text"
          id="postTitle"
          name="postTitle"
          value={title}
          onChange={e => setTitle(e.target.value)}
        />
        
        <label htmlFor="postContent">Content:</label>
        <textarea
          id="postContent"
          name="postContent"
          value={content}
          onChange={e => setContent(e.target.value)}
        />
        
        <label htmlFor="postAuthor">Author:</label>
        <select
          id="postAuthor"
          value={userId}
          onChange={e => setUserId(e.target.value)}
        >
          <option value=""></option>
          {usersOptions}
        </select>
        
        <button 
          type="button" 
          onClick={onSavePostClicked}
          disabled={!canSave}
        >
          Save Post
        </button>
      </form>
    </section>
  );
};

// ------------------------------------------------------
// File: App.js
// ------------------------------------------------------
import React, { useEffect } from 'react';
import { Routes, Route, Navigate } from 'react-router-dom';
import { useAppDispatch, useAppSelector } from './app/hooks';

import { PostsList } from './features/posts/PostsList';
import { AddPostForm } from './features/posts/AddPostForm';
import { SinglePostPage } from './features/posts/SinglePostPage';
import { EditPostForm } from './features/posts/EditPostForm';
import { UsersList } from './features/users/UsersList';
import { UserPage } from './features/users/UserPage';
import { Navbar } from './components/Navbar';

import { selectPostStatus, fetchPosts } from './features/posts/postsSlice';
import { fetchUsers } from './features/users/usersSlice';

function App() {
  const dispatch = useAppDispatch();
  const postsStatus = useAppSelector(selectPostStatus);

  useEffect(() => {
    if (postsStatus === 'idle') {
      dispatch(fetchPosts());
      dispatch(fetchUsers());
    }
  }, [postsStatus, dispatch]);

  return (
    <>
      <Navbar />
      <div className="container">
        <Routes>
          <Route 
            path="/" 
            element={
              <>
                <AddPostForm />
                <PostsList />
              </>
            } 
          />
          <Route path="/posts/:postId" element={<SinglePostPage />} />
          <Route path="/editPost/:postId" element={<EditPostForm />} />
          <Route path="/users" element={<UsersList />} />
          <Route path="/users/:userId" element={<UserPage />} />
          <Route path="*" element={<Navigate to="/" replace />} />
        </Routes>
      </div>
    </>
  );
}

export default App;

// ------------------------------------------------------
// File: index.js
// ------------------------------------------------------
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import { BrowserRouter } from 'react-router-dom';
import { store } from './app/store';
import App from './App';
import './index.css';

// Mock Service Worker for development
if (process.env.NODE_ENV === 'development') {
  const { worker } = require('./mocks/browser');
  worker.start({
    onUnhandledRequest: 'bypass',
  });
}

ReactDOM.render(
  <React.StrictMode>
    <Provider store={store}>
      <BrowserRouter>
        <App />
      </BrowserRouter>
    </Provider>
  </React.StrictMode>,
  document.getElementById('root')
);

```