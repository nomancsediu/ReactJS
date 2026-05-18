# Container and Presentational Components

When I first started building React apps, every component fetched data, managed state, and rendered UI all at once. It worked for small apps, but as things grew, components became impossible to test and reuse.

Then I learned the container and presentational pattern, and it changed how I think about components.

## The Idea

Split your components into two types:

- **Container components** (smart): handle data fetching, state management, and business logic
- **Presentational components** (dumb): receive data via props and just render UI

```jsx
// Container: smart, handles logic
function UserListContainer() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/users')
      .then(res => res.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <p>Loading...</p>;

  return <UserList users={users} />;
}
```

```jsx
// Presentational: dumb, just renders
function UserList({ users }) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name} - {user.email}</li>
      ))}
    </ul>
  );
}
```

The container knows about APIs and state. The presentational component only knows about props and HTML.

## Why This Matters

**Reusability:** You can use `UserList` with any data source. Hardcoded data, mock data, or data from a different API. The list does not care where users come from.

**Testability:** Testing `UserList` is easy. Just pass props and check the output. No mocks needed for fetch or state.

**Separation of concerns:** Each component has one job. The container fetches. The list renders.

## More Examples

```jsx
// Container
function DashboardContainer() {
  const [stats, setStats] = useState(null);

  useEffect(() => {
    fetchDashboardStats().then(setStats);
  }, []);

  return stats ? <Dashboard stats={stats} /> : <LoadingSkeleton />;
}

// Presentational
function Dashboard({ stats }) {
  return (
    <div className="grid grid-cols-3 gap-4">
      <StatCard title="Users" value={stats.userCount} />
      <StatCard title="Revenue" value={stats.revenue} />
      <StatCard title="Orders" value={stats.orderCount} />
    </div>
  );
}
```

## Is This Still Relevant?

With hooks, the line has blurred. Custom hooks can extract the "smart" logic, and components can stay focused on rendering. The core idea remains: separate data logic from UI.

```jsx
// Modern approach with custom hooks
function useUsers() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/users')
      .then(res => res.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []);

  return { users, loading };
}

// Component is cleaner but still uses the hook
function UserListPage() {
  const { users, loading } = useUsers();
  if (loading) return <p>Loading...</p>;
  return <UserList users={users} />;
}
```

Whether you use containers or hooks, the principle is the same: keep logic and presentation separate.
