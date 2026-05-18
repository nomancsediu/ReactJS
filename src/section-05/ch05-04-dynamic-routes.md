# Dynamic Routes

Not every URL is fixed. Sometimes you need a route like `/users/123` where `123` is a variable. That is what dynamic routes are for, and they are essential for building real apps.

## URL Parameters

Define a dynamic segment with a colon:

```jsx
<Route path="/users/:id" element={<UserProfile />} />
```

The `:id` part is a parameter. Any value in that position becomes the `id` parameter.

## Reading Parameters with useParams

Inside the component, use the `useParams` hook to read the parameter:

```jsx
import { useParams } from 'react-router-dom';

function UserProfile() {
  const { id } = useParams();

  return <h1>User ID: {id}</h1>;
}
```

Visiting `/users/42` renders "User ID: 42". Visiting `/users/abc` renders "User ID: abc". The parameter is always a string.

## Multiple Parameters

You can have more than one dynamic segment:

```jsx
<Route path="/users/:userId/posts/:postId" element={<UserPost />} />
```

```jsx
function UserPost() {
  const { userId, postId } = useParams();

  return (
    <div>
      <p>User: {userId}</p>
      <p>Post: {postId}</p>
    </div>
  );
}
```

Visiting `/users/5/posts/42` gives you `userId` of "5" and `postId` of "42".

## Fetching Data with Parameters

The most common use case is fetching data based on the URL:

```jsx
function UserProfile() {
  const { id } = useParams();
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(`/api/users/${id}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      });
  }, [id]);

  if (loading) return <p>Loading...</p>;
  if (!user) return <p>User not found</p>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

Notice `id` in the dependency array. When the URL changes from `/users/1` to `/users/2`, the effect runs again with the new id.

## Wildcard Routes

Use `*` to match anything after a certain path:

```jsx
<Route path="/files/*" element={<FileExplorer />} />
```

```jsx
function FileExplorer() {
  const params = useParams();
  // params["*"] contains everything after /files/

  return <p>Path: {params['*']}</p>;
}
```

Visiting `/files/docs/readme.md` gives you `params["*"]` as "docs/readme.md".

## Optional Parameters

You can make parameters optional with a question mark:

```jsx
<Route path="/products/:category?" element={<Products />} />
```

This matches both `/products` and `/products/electronics`. Check if the parameter exists before using it.

Dynamic routes are how I build almost every detail page in my apps. They turn URLs into data lookups, and that is incredibly powerful.
