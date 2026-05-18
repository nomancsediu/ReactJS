# TypeScript with React

I resisted TypeScript for a long time. All those extra types felt like busywork. But once I started using it with React, I wondered how I ever lived without it. The compiler catches bugs I would have spent hours debugging.

## Typing Props

```tsx
interface ButtonProps {
  label: string;
  variant?: "primary" | "secondary";
  onClick: () => void;
  children?: React.ReactNode;
}

function Button({ label, variant = "primary", onClick }: ButtonProps) {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {label}
    </button>
  );
}
```

The `variant` prop is now restricted to only two values. If you pass `variant="danger"`, TypeScript tells you immediately.

## Typing State

Most of the time, TypeScript infers the state type automatically:

```tsx
const [count, setCount] = useState(0); // TypeScript knows this is number
const [name, setName] = useState(""); // TypeScript knows this is string
```

When the initial value is `null`, you need to be explicit:

```tsx
const [user, setUser] = useState<User | null>(null);

interface User {
  id: number;
  name: string;
  email: string;
}
```

## Typing Events

React events have specific types:

```tsx
function SearchForm() {
  const [query, setQuery] = useState("");

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setQuery(e.target.value);
  };

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    console.log(query);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={query} onChange={handleChange} />
      <button type="submit">Search</button>
    </form>
  );
}
```

Common event types:

| Event | Type |
|---|---|
| Input change | `React.ChangeEvent<HTMLInputElement>` |
| Form submit | `React.FormEvent<HTMLFormElement>` |
| Click | `React.MouseEvent<HTMLButtonElement>` |
| Key press | `React.KeyboardEvent<HTMLInputElement>` |

## Typing Custom Hooks

```tsx
function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  const setStoredValue = (newValue: T) => {
    setValue(newValue);
    localStorage.setItem(key, JSON.stringify(newValue));
  };

  return [value, setStoredValue] as const;
}

// Usage: TypeScript knows count is a number
const [count, setCount] = useLocalStorage("count", 0);
```

## Typing Component Children

```tsx
interface LayoutProps {
  children: React.ReactNode;
  title: string;
}

function Layout({ children, title }: LayoutProps) {
  return (
    <div>
      <h1>{title}</h1>
      <main>{children}</main>
    </div>
  );
}
```

## Generics in Components

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item) => (
        <li key={keyExtractor(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}

// Usage
<List
  items={[{ id: "1", name: "Alice" }]}
  renderItem={(user) => <span>{user.name}</span>}
  keyExtractor={(user) => user.id}
/>;
```

TypeScript with React takes a bit of extra typing (pun intended), but the safety and autocompletion you get in return are worth every extra keystroke.
