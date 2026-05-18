# State Overview

Before picking a state management tool, you need to understand what kinds of state exist in a React app. I used to treat all state the same, and it caused problems.

## Local vs Global State

**Local state** belongs to a single component. Other components do not need it.

```jsx
function SearchBar() {
  const [query, setQuery] = useState('');

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

Only `SearchBar` cares about the query. No need for a global store.

**Global state** is shared across multiple components that are far apart in the component tree.

```jsx
// Both of these need the current user
<Header>
  <UserAvatar /> {/* needs user */}
</Header>
<Dashboard>
  <WelcomeMessage /> {/* also needs user */}
  <Settings /> {/* also needs user */}
</Dashboard>
```

Passing `user` as props through every layer is painful. This is where global state helps.

## State Categories

I think of state in four categories:

**UI State:** Is this modal open? Which tab is active? Is the sidebar collapsed?

```jsx
const [isOpen, setIsOpen] = useState(false);
const [activeTab, setActiveTab] = useState('profile');
```

This is almost always local. Keep it in the component that uses it.

**Server State:** Data fetched from an API. User profiles, product lists, blog posts.

```jsx
const [users, setUsers] = useState([]);
const [loading, setLoading] = useState(true);
```

Server state is tricky because it can become stale. Tools like TanStack Query specialize in this.

**Form State:** Input values, validation errors, submission status.

```jsx
const [email, setEmail] = useState('');
const [errors, setErrors] = useState({});
```

Usually local, but complex forms benefit from dedicated form libraries.

**App State:** Authentication, theme, locale, shopping cart.

```jsx
const [user, setUser] = useState(null);
const [theme, setTheme] = useState('light');
const [cart, setCart] = useState([]);
```

Often global because many components need access.

## When State Gets Complex

State management becomes a problem when:

- You pass the same prop through five or more levels (prop drilling)
- Multiple components need the same data and it falls out of sync
- State updates are complex with many interdependent values
- You need to persist state across page navigation

```jsx
// Prop drilling nightmare
<App user={user}>
  <Layout user={user}>
    <Sidebar user={user}>
      <Profile user={user} />
    </Sidebar>
  </Layout>
</App>
```

## Choosing the Right Level

My rule of thumb: keep state as local as possible. Only lift it up or make it global when you have a clear reason.

Start with `useState`. If logic gets complex, try `useReducer`. If multiple distant components need the data, try Context. If the app is large with many state interactions, consider Redux or Zustand.

Do not reach for the big guns until you actually need them. Simple is almost always better.
