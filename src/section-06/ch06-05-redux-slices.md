# Redux Slices

Slices are the building blocks of your Redux store. Each slice manages a specific piece of state, like a user slice, a cart slice, or a theme slice. I like to think of them as independent modules that combine into the full store.

## createSlice

The `createSlice` function generates action creators and reducers from a single definition:

```jsx
import { createSlice } from '@reduxjs/toolkit';

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [],
    total: 0,
  },
  reducers: {
    addItem: (state, action) => {
      const existing = state.items.find(i => i.id === action.payload.id);
      if (existing) {
        existing.quantity += 1;
      } else {
        state.items.push({ ...action.payload, quantity: 1 });
      }
      state.total = state.items.reduce(
        (sum, item) => sum + item.price * item.quantity, 0
      );
    },
    removeItem: (state, action) => {
      state.items = state.items.filter(i => i.id !== action.payload);
      state.total = state.items.reduce(
        (sum, item) => sum + item.price * item.quantity, 0
      );
    },
    clearCart: (state) => {
      state.items = [];
      state.total = 0;
    },
  },
});

export const { addItem, removeItem, clearCart } = cartSlice.actions;
export default cartSlice.reducer;
```

Notice I am mutating state directly with `state.items.push()`. Redux Toolkit uses Immer internally, so mutations are safe. Immer creates a new state object under the hood.

## Reducers

Each function in `reducers` handles a specific action type. The function receives the current state and the action:

```jsx
reducers: {
  // No payload needed
  clearCart: (state) => {
    state.items = [];
  },

  // Payload is the value you pass to the action creator
  addItem: (state, action) => {
    state.items.push(action.payload);
  },

  // Payload can be an object
  updateQuantity: (state, action) => {
    const { id, quantity } = action.payload;
    const item = state.items.find(i => i.id === id);
    if (item) item.quantity = quantity;
  },
}
```

## Action Creators

`createSlice` automatically generates action creators. They match the names you define in `reducers`:

```jsx
// These are auto-generated
const { addItem, removeItem, clearCart } = cartSlice.actions;

// Dispatching
dispatch(addItem({ id: 1, name: 'Shirt', price: 29.99 }));
dispatch(removeItem(1));
dispatch(clearCart());
```

The action creator takes one argument, which becomes `action.payload`.

## Selectors

Selectors are functions that extract specific data from the store. I define them alongside the slice:

```jsx
// Simple selector
export const selectCartItems = (state) => state.cart.items;
export const selectCartTotal = (state) => state.cart.total;
export const selectCartItemCount = (state) =>
  state.cart.items.reduce((sum, item) => sum + item.quantity, 0);
```

Use them in components with `useSelector`:

```jsx
function CartSummary() {
  const items = useSelector(selectCartItems);
  const total = useSelector(selectCartTotal);
  const count = useSelector(selectCartItemCount);

  return (
    <div>
      <p>{count} items in cart</p>
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}
```

Selectors keep components clean. If the state shape changes, you only update the selector, not every component.

## Slice Organization

I organize slices in separate files under a store folder:

```
src/
  store/
    index.js       (configureStore)
    cartSlice.js
    userSlice.js
    themeSlice.js
```

Each slice is independent and focused. Combine them in the store:

```jsx
import { configureStore } from '@reduxjs/toolkit';
import cartReducer from './cartSlice';
import userReducer from './userSlice';
import themeReducer from './themeSlice';

export const store = configureStore({
  reducer: {
    cart: cartReducer,
    user: userReducer,
    theme: themeReducer,
  },
});
```

This pattern keeps your store modular and easy to maintain as the app grows.
