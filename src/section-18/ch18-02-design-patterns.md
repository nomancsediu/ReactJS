# Design Patterns in React

Design patterns are reusable solutions to common problems. I used to think patterns were overkill, but once I started seeing the same problems pop up in every project, I realized patterns save time and reduce bugs.

## Provider Pattern

The provider pattern shares data across many components without prop drilling. Context is the built-in way to do this:

```tsx
const ThemeContext = createContext<{
  theme: string;
  toggle: () => void;
}>({ theme: "light", toggle: () => {} });

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState("light");

  const toggle = () => setTheme((t) => (t === "light" ? "dark" : "light"));

  return (
    <ThemeContext.Provider value={{ theme, toggle }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Usage
function App() {
  return (
    <ThemeProvider>
      <Layout />
    </ThemeProvider>
  );
}

function ThemedButton() {
  const { theme, toggle } = useContext(ThemeContext);
  return (
    <button className={`btn-${theme}`} onClick={toggle}>
      Toggle Theme
    </button>
  );
}
```

## Custom Hook Pattern

Extract complex logic into a custom hook so components stay focused on rendering:

```tsx
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    setLoading(true);
    fetch(url)
      .then((res) => res.json())
      .then((json) => {
        setData(json);
        setLoading(false);
      })
      .catch((err) => {
        setError(err.message);
        setLoading(false);
      });
  }, [url]);

  return { data, loading, error };
}

// Usage
function UserList() {
  const { data, loading, error } = useFetch<User[]>("/api/users");

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <ul>
      {data?.map((user) => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

## Render Props Pattern

Pass a function as a child or prop that returns UI. This gives the consumer control over rendering:

```tsx
interface MouseProps {
  render: (position: { x: number; y: number }) => React.ReactNode;
}

function MouseTracker({ render }: MouseProps) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    window.addEventListener("mousemove", handleMove);
    return () => window.removeEventListener("mousemove", handleMove);
  }, []);

  return render(position);
}

// Usage
<MouseTracker
  render={({ x, y }) => (
    <p>Mouse is at {x}, {y}</p>
  )}
/>
```

## Compound Component Pattern

Create components that work together through a shared context, like HTML select and option:

```tsx
const TabsContext = createContext<{
  activeTab: string;
  setActiveTab: (id: string) => void;
}>({ activeTab: "", setActiveTab: () => {} });

function Tabs({ children, defaultTab }: { children: React.ReactNode; defaultTab: string }) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      {children}
    </TabsContext.Provider>
  );
}

function TabList({ children }: { children: React.ReactNode }) {
  return <div className="flex gap-2">{children}</div>;
}

function Tab({ id, children }: { id: string; children: React.ReactNode }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  return (
    <button
      className={activeTab === id ? "active" : ""}
      onClick={() => setActiveTab(id)}
    >
      {children}
    </button>
  );
}

function TabPanel({ id, children }: { id: string; children: React.ReactNode }) {
  const { activeTab } = useContext(TabsContext);
  if (activeTab !== id) return null;
  return <div className="p-4">{children}</div>;
}

// Usage
<Tabs defaultTab="home">
  <TabList>
    <Tab id="home">Home</Tab>
    <Tab id="about">About</Tab>
  </TabList>
  <TabPanel id="home">Welcome!</TabPanel>
  <TabPanel id="about">About us</TabPanel>
</Tabs>
```

These patterns make your components more flexible and reusable. I do not use all of them in every project, but knowing them means I have the right tool when the problem shows up.
