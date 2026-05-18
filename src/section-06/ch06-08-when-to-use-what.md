# When to Use What

After using all these state management tools, I realized the hardest part is not learning them but knowing when to use each one. Here is the decision framework I use.

## The Decision Tree

**1. Is the state used by only one component?**
Use `useState`. End of story.

```jsx
function Modal() {
  const [isOpen, setIsOpen] = useState(false);
  // Only this component cares
}
```

**2. Is the state complex with many related fields and transitions?**
Use `useReducer`. It keeps related logic together and makes transitions predictable.

```jsx
function CheckoutForm() {
  const [state, dispatch] = useReducer(checkoutReducer, initialState);
  // Many interdependent fields and steps
}
```

**3. Is the state needed by a few nearby components?**
Lift state up to the nearest common parent. Props are not evil for small distances.

```jsx
function SearchPage() {
  const [query, setQuery] = useState('');
  return (
    <div>
      <SearchBar query={query} onChange={setQuery} />
      <Results query={query} />
    </div>
  );
}
```

**4. Is the state needed by many distant components?**
Use Context API for simple shared state like theme, auth, or locale.

```jsx
<ThemeProvider>
  <App /> {/* Everything inside can access theme */}
</ThemeProvider>
```

**5. Is the state complex, global, and needs strict patterns?**
Use Redux Toolkit. Best for large teams, complex state interactions, and when you need time-travel debugging.

```jsx
const store = configureStore({
  reducer: { users, posts, cart, ui },
});
```

**6. Is the state global but you want minimal boilerplate?**
Use Zustand. Best for small to medium apps where you want simplicity.

```jsx
const useStore = create((set) => ({
  user: null,
  login: (user) => set({ user }),
  logout: () => set({ user: null }),
}));
```

## Quick Reference Table

| Tool | Scope | Boilerplate | Best For |
|---|---|---|---|
| useState | Local | None | Simple component state |
| useReducer | Local | Low | Complex local logic |
| Context | Shared tree | Low | Theme, auth, locale |
| Redux Toolkit | Global | High | Large apps, strict patterns |
| Zustand | Global | Very low | Simple global state |

## Combinations

You can mix and match. A real app might use:

```jsx
function App() {
  return (
    <Provider store={reduxStore}>      {/* Redux for complex data */}
      <ThemeProvider>                   {/* Context for theme */}
        <AuthProvider>                  {/* Context for auth */}
          <Router>
            <Layout />
          </Router>
        </AuthProvider>
    </ThemeProvider>
    </Provider>
  );
}
```

Individual pages still use `useState` and `useReducer` for local concerns.

## Anti-Patterns I Have Learned

**Do not put everything in global state.** A modal's open/close state belongs locally. A form's input values belong locally. Global state is for data many components share.

**Do not use Context for high-frequency updates.** Every consumer re-renders when the context value changes. A rapidly changing value like mouse position will kill performance.

**Do not choose Redux because it is popular.** Choose it because your app has complex state interactions that benefit from strict patterns and DevTools.

**Do not over-abstract.** If `useState` works, use it. You can always refactor later when needs change.

## My Default Stack

For most projects, I start with:

- `useState` for everything local
- Context for auth and theme
- Zustand if I need a global store
- Upgrade to Redux only if the app outgrows Zustand

Start simple. Add complexity only when the problem demands it. That is the best advice I can give about state management.
