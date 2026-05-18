# Redux Toolkit Setup

Redux used to terrify me. So much boilerplate. Action types as strings. Switch statements. Reducers, actions, selectors, middleware. Setting up a simple counter felt like filing taxes.

Then Redux Toolkit came along and made it almost enjoyable. It removes the boilerplate while keeping the power.

## Installing Redux Toolkit

```bash
npm install @reduxjs/toolkit react-redux
```

`@reduxjs/toolkit` includes Redux core plus utilities. `react-redux` provides the React bindings.

## Creating the Store

Use `configureStore` to set up your store:

```jsx
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';

const store = configureStore({
  reducer: {
    counter: counterReducer,
  },
});

export default store;
```

`configureStore` automatically adds Redux DevTools and sets up common middleware. No manual configuration needed.

## The Provider Component

Wrap your app with the Redux Provider so components can access the store:

```jsx
import { Provider } from 'react-redux';
import store from './store';

function App() {
  return (
    <Provider store={store}>
      <Counter />
    </Provider>
  );
}
```

I put the Provider at the very top of my app, above the router:

```jsx
createRoot(document.getElementById('root')).render(
  <Provider store={store}>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </Provider>
);
```

## A Minimal Example

Here is a complete, minimal setup with a counter slice:

```jsx
// store/counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;
```

```jsx
// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';

export const store = configureStore({
  reducer: { counter: counterReducer },
});
```

```jsx
// App.jsx
import { Provider, useSelector, useDispatch } from 'react-redux';
import { increment, decrement, incrementByAmount } from './store/counterSlice';

function Counter() {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => dispatch(increment())}>+1</button>
      <button onClick={() => dispatch(decrement())}>-1</button>
      <button onClick={() => dispatch(incrementByAmount(5))}>+5</button>
    </div>
  );
}
```

## Redux DevTools

One of the best things about Redux is the DevTools extension. Install the Redux DevTools browser extension and you get:

- A log of every dispatched action
- State snapshots at each step
- Time travel debugging
- Action replay

`configureStore` enables DevTools automatically in development. No setup required.

Redux Toolkit takes the pain out of Redux. The setup is minimal, the boilerplate is gone, and you still get all the benefits of a predictable, debuggable state container.
