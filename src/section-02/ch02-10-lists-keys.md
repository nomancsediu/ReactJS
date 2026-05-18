# Lists and Keys

Rendering lists of data is something you do in almost every React component. Product lists, user lists, menu items, search results, they are all lists. React has a specific way to handle them, and getting the `key` prop wrong causes subtle bugs.

## Rendering Lists with map

Use the `map` array method to transform data into JSX elements:

```jsx
function FruitList() {
  const fruits = ["Apple", "Banana", "Cherry"]

  return (
    <ul>
      {fruits.map(fruit => (
        <li key={fruit}>{fruit}</li>
      ))}
    </ul>
  )
}
```

`map` returns an array of `<li>` elements, and React renders each one.

For objects, you map over the array and access properties:

```jsx
function ProductList({ products }) {
  return (
    <div>
      {products.map(product => (
        <div key={product.id}>
          <h3>{product.name}</h3>
          <p>${product.price}</p>
        </div>
      ))}
    </div>
  )
}
```

## Why Keys Matter

Keys help React identify which items changed, got added, or got removed. They are used during reconciliation to match elements between renders.

Without keys, React re-renders every list item when the list changes. With proper keys, React only updates the items that actually changed.

```jsx
// Imagine this list changes from [A, B, C] to [A, C, D]
// Without keys: React updates all three items (slow)
// With keys: React removes B, keeps A and C, adds D (fast)
```

Keys must be:

- **Unique among siblings** - no two items in the same list should share a key
- **Stable** - the same item should always get the same key between renders

## Using IDs as Keys

The best key is a unique ID from your data:

```jsx
{users.map(user => (
  <UserCard key={user.id} user={user} />
))}
```

Database IDs are perfect. They are unique and stable.

## The Index as Key Problem

Using the array index as a key is tempting but causes bugs when the list can change:

```jsx
// Avoid this when list items can be added, removed, or reordered
{items.map((item, index) => (
  <Item key={index} item={item} />
))}
```

Why is this bad? Consider a list `[A, B, C]` with keys `[0, 1, 2]`. If you add an item at the beginning `[D, A, B, C]`, the keys become `[0, 1, 2, 3]`. React thinks key `0` changed from A to D, key `1` changed from B to A, and so on. Every item gets re-rendered instead of just adding D.

Even worse, if list items have their own state (like input fields), the state gets mixed up because React matches by key, not by content.

## When Index Keys Are OK

If the list is static (never changes) and items have no internal state, index keys work fine. But when in doubt, use a unique ID.

## Keys Do Not Pass as Props

The `key` prop is special. React uses it internally but does not pass it to your component:

```jsx
// key is not available as props.key
<li key={item.id}>{item.name}</li>

// If you need the ID inside the component, pass it separately
<ListItem key={item.id} id={item.id} name={item.name} />
```

Lists and keys are simple once you understand the why. Always use stable, unique IDs as keys, and your list rendering will be fast and bug-free.
