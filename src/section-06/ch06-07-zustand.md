# Zustand

Zustand is the state management library I wish I had found sooner. It gives you global state with almost no boilerplate. No providers, no reducers, no action types. Just a store and some functions.

## Creating a Store

Use `create` to define a store with state and actions:

```jsx
import { create } from 'zustand';

const useCartStore = create((set) => ({
  items: [],
  total: 0,

  addItem: (product) =>
    set((state) => {
      const existing = state.items.find(i => i.id === product.id);
      if (existing) {
        return {
          items: state.items.map(i =>
            i.id === product.id ? { ...i, quantity: i.quantity + 1 } : i
          ),
          total: state.total + product.price,
        };
      }
      return {
        items: [...state.items, { ...product, quantity: 1 }],
        total: state.total + product.price,
      };
    }),

  removeItem: (id) =>
    set((state) => ({
      items: state.items.filter(i => i.id !== id),
      total: state.items
        .filter(i => i.id !== id)
        .reduce((sum, i) => sum + i.price * i.quantity, 0),
    })),

  clearCart: () => set({ items: [], total: 0 }),
}));
```

That is it. No slice definition, no action creators, no provider wrapper.

## Using the Store in Components

```jsx
function Cart() {
  const items = useCartStore((state) => state.items);
  const total = useCartStore((state) => state.total);

  return (
    <div>
      {items.map(item => (
        <div key={item.id}>
          {item.name} x {item.quantity} = ${item.price * item.quantity}
        </div>
      ))}
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}

function AddToCartButton({ product }) {
  const addItem = useCartStore((state) => state.addItem);

  return (
    <button onClick={() => addItem(product)}>
      Add to Cart
    </button>
  );
}
```

Each component only subscribes to the state it needs. If `items` changes, only `Cart` re-renders. `AddToCartButton` is unaffected because it only subscribes to `addItem`.

## Middleware

Zustand supports middleware for common patterns:

```jsx
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';

const useCartStore = create(
  devtools(
    persist(
      (set) => ({
        items: [],
        addItem: (product) =>
          set((state) => ({
            items: [...state.items, product],
          })),
        clearCart: () => set({ items: [] }),
      }),
      { name: 'cart-storage' }
    )
  )
);
```

- **persist** saves state to localStorage automatically
- **devtools** connects to Redux DevTools for debugging

You can combine them by nesting, as shown above.

## Zustand vs Redux

| Feature | Redux Toolkit | Zustand |
|---|---|---|
| Boilerplate | More (slices, actions, reducers) | Minimal |
| Provider needed | Yes | No |
| DevTools | Built-in | Via middleware |
| Learning curve | Steeper | Gentle |
| Async support | createAsyncThunk | Just write async functions |
| Best for | Large teams, strict patterns | Small to medium apps, simplicity |

## Async in Zustand

No special API needed. Just make your action async:

```jsx
const useUserStore = create((set) => ({
  users: [],
  loading: false,
  error: null,

  fetchUsers: async () => {
    set({ loading: true, error: null });
    try {
      const res = await fetch('/api/users');
      const data = await res.json();
      set({ users: data, loading: false });
    } catch (err) {
      set({ error: err.message, loading: false });
    }
  },
}));
```

Zustand is my go-to for new projects. It handles 90% of what Redux does with 10% of the code. For massive apps with strict team conventions, Redux still has its place. For everything else, Zustand just works.
