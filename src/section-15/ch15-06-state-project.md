# State Management in the Project

Choosing how to manage state in a full-stack React app can be overwhelming. There are so many options: Redux, Zustand, Context, TanStack Query, and more. I used to overthink this. Now I follow a simple rule: use the simplest tool that gets the job done.

## State Categories

I break state into three categories:

1. **UI state** - Is the sidebar open? Which tab is active? Which modal is showing? This is local and temporary.
2. **App state** - Who is the current user? What are their permissions? This is global and persistent.
3. **Server state** - Product lists, search results, user profiles. This comes from the backend and needs caching.

Each category gets a different tool.

## UI State: useState

For simple UI state, `useState` is all you need:

```jsx
function ProductList() {
  const [showForm, setShowForm] = useState(false);
  const [editingProduct, setEditingProduct] = useState(null);

  return (
    <div>
      <button onClick={() => setShowForm(true)}>Add Product</button>
      {showForm && (
        <ProductForm
          product={editingProduct}
          onClose={() => { setShowForm(false); setEditingProduct(null); }}
        />
      )}
    </div>
  );
}
```

No need for a global store for a modal visibility flag.

## App State: Context + Zustand

For auth state and other global app data, I use a combination. Auth context we already built. For other global state, I like Zustand because it is simpler than Redux:

```js
// src/store/uiStore.js
import { create } from 'zustand';

const useUIStore = create((set) => ({
  sidebarOpen: false,
  activeTab: 'dashboard',
  notifications: [],

  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
  setActiveTab: (tab) => set({ activeTab: tab }),
  addNotification: (notification) =>
    set((state) => ({
      notifications: [...state.notifications, notification],
    })),
  clearNotifications: () => set({ notifications: [] }),
}));

export default useUIStore;
```

Using Zustand in a component:

```jsx
import useUIStore from '../store/uiStore';

function Header() {
  const { sidebarOpen, toggleSidebar, notifications } = useUIStore();

  return (
    <header>
      <button onClick={toggleSidebar}>Menu</button>
      {notifications.length > 0 && (
        <span className="notification-badge">{notifications.length}</span>
      )}
    </header>
  );
}
```

## Server State: TanStack Query

For data from the backend, I use TanStack Query (formerly React Query). It handles caching, refetching, and loading states automatically:

```jsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import apiClient from '../lib/apiClient';

function useProducts(filters = {}) {
  return useQuery({
    queryKey: ['products', filters],
    queryFn: async () => {
      const params = new URLSearchParams(filters);
      const res = await apiClient.get(`/products?${params}`);
      return res.data;
    },
  });
}

function useCreateProduct() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (product) => {
      const res = await apiClient.post('/products', product);
      return res.data;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] });
    },
  });
}
```

Using the hooks in components:

```jsx
function ProductList() {
  const { data: products, isLoading, error } = useProducts();

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error loading products</p>;

  return (
    <ul>
      {products.map((p) => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}
```

## Data Flow Summary

Here is how data flows in the project:

- **UI state** flows locally through `useState` and props.
- **Auth state** flows globally through `AuthContext`.
- **App settings** flow through `Zustand` stores.
- **Server data** flows through `TanStack Query` with automatic caching and refetching.

This separation keeps things clean. Each type of state uses the right tool, and components stay focused on rendering rather than managing data.
