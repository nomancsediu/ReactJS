# Loading States

A blank screen is the worst user experience. When data is loading, you need to show something. I learned this the hard way when my users thought the app was broken because nothing appeared for two seconds.

## The Basic Spinner

The simplest loading state is a spinner:

```jsx
function UserList() {
  const [users, setUsers] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch("/api/users")
      .then((res) => res.json())
      .then(setUsers)
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <Spinner />;
  return <UserCards users={users} />;
}

function Spinner() {
  return (
    <div className="flex justify-center p-8">
      <div className="h-8 w-8 animate-spin rounded-full border-4 border-gray-300 border-t-blue-500" />
    </div>
  );
}
```

## Skeleton Loading

Skeletons are better than spinners because they show the shape of the content that is coming. Users perceive them as faster:

```jsx
function UserListSkeleton() {
  return (
    <div className="space-y-4">
      {Array.from({ length: 5 }).map((_, i) => (
        <div key={i} className="flex items-center gap-3 animate-pulse">
          <div className="h-10 w-10 rounded-full bg-gray-200" />
          <div className="flex-1">
            <div className="h-4 w-32 rounded bg-gray-200 mb-2" />
            <div className="h-3 w-48 rounded bg-gray-200" />
          </div>
        </div>
      ))}
    </div>
  );
}
```

## Progressive Loading

Instead of waiting for everything, load pieces as they arrive:

```jsx
function Dashboard() {
  const [profile, setProfile] = useState(null);
  const [posts, setPosts] = useState(null);

  useEffect(() => {
    fetch("/api/profile").then((r) => r.json()).then(setProfile);
    fetch("/api/posts").then((r) => r.json()).then(setPosts);
  }, []);

  return (
    <div>
      {profile ? <ProfileCard profile={profile} /> : <ProfileSkeleton />}
      {posts ? <PostList posts={posts} /> : <PostListSkeleton />}
    </div>
  );
}
```

Each section loads independently. The profile might appear before the posts, and that is fine. The user sees progress instead of waiting for everything at once.

## React Suspense

Suspense takes a different approach. You declare what you are waiting for, and React handles the rest:

```jsx
import { Suspense } from "react";

function App() {
  return (
    <Suspense fallback={<UserListSkeleton />}>
      <UserList />
    </Suspense>
  );
}
```

With Suspense, your component does not manage its own loading state. Instead, it "suspends" when data is not ready, and the nearest Suspense boundary shows the fallback.

Suspense works best with data fetching libraries like React Query or Relay that integrate with it. For simple `useEffect`-based fetching, the manual approach works great.

## Tips I Follow

- Always show a loading state, never a blank screen
- Skeletons feel faster than spinners
- Load sections independently when possible
- Disable buttons during submission to prevent double clicks

Your users will thank you.
