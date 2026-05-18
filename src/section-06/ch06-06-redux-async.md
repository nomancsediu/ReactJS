# Redux Async Operations

Real apps do not just update local state. They fetch data from APIs, submit forms, and handle loading and error states. Redux Toolkit makes async operations manageable with `createAsyncThunk`.

## createAsyncThunk

`createAsyncThunk` generates action creators for an async workflow: pending, fulfilled, and rejected.

```jsx
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';

export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async () => {
    const response = await fetch('/api/users');
    if (!response.ok) throw new Error('Failed to fetch users');
    return response.json();
  }
);
```

The first argument is a prefix for the action types. The second is the async function. Whatever it returns becomes the `action.payload` on success.

## Handling Thunk Actions in the Slice

Use `extraReducers` to handle the thunk's generated actions:

```jsx
const usersSlice = createSlice({
  name: 'users',
  initialState: {
    data: [],
    loading: false,
    error: null,
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  },
});

export default usersSlice.reducer;
```

The three cases handle the full lifecycle of the async operation.

## A Complete Example with Error Handling

```jsx
export const createPost = createAsyncThunk(
  'posts/createPost',
  async (postData, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/posts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(postData),
      });

      if (!response.ok) {
        const error = await response.json();
        return rejectWithValue(error.message);
      }

      return response.json();
    } catch (err) {
      return rejectWithValue(err.message);
    }
  }
);

const postsSlice = createSlice({
  name: 'posts',
  initialState: {
    list: [],
    loading: false,
    error: null,
  },
  reducers: {
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(createPost.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(createPost.fulfilled, (state, action) => {
        state.loading = false;
        state.list.unshift(action.payload);
      })
      .addCase(createPost.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload || action.error.message;
      });
  },
});
```

Using `rejectWithValue` lets you control the error payload instead of relying on the default error serialization.

## Using Thunks in Components

```jsx
import { useDispatch, useSelector } from 'react-redux';
import { fetchUsers } from './usersSlice';

function UserList() {
  const dispatch = useDispatch();
  const { data, loading, error } = useSelector((state) => state.users);

  useEffect(() => {
    dispatch(fetchUsers());
  }, [dispatch]);

  if (loading) return <Spinner />;
  if (error) return <p>Error: {error}</p>;

  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## Loading State Pattern

I follow a consistent pattern for all async slices:

- `loading` starts as `false`, becomes `true` on pending, back to `false` on fulfilled or rejected
- `error` is `null` by default, cleared on pending, set on rejected
- `data` holds the successful result

This pattern makes every async slice predictable. You always know what state to check and when.
