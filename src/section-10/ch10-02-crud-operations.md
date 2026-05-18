# CRUD Operations with React

CRUD stands for Create, Read, Update, Delete. These are the four basic things you do with any API. Let me show you how I handle each one in React.

## Read: Fetching Data

Reading data is the most common operation. Here is a pattern I use for fetching:

```jsx
function PostList() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let cancelled = false;

    fetch("/api/posts")
      .then((res) => res.json())
      .then((data) => {
        if (!cancelled) setPosts(data);
      })
      .finally(() => {
        if (!cancelled) setLoading(false);
      });

    return () => { cancelled = true; };
  }, []);

  if (loading) return <p>Loading...</p>;
  return posts.map((post) => <PostCard key={post.id} post={post} />);
}
```

That `cancelled` flag prevents setting state on an unmounted component.

## Create: Posting Data

Creating resources means sending data to the server:

```jsx
function CreatePost() {
  const [title, setTitle] = useState("");
  const [body, setBody] = useState("");
  const [submitting, setSubmitting] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setSubmitting(true);

    try {
      const res = await fetch("/api/posts", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ title, body }),
      });
      const newPost = await res.json();
      // Add the new post to the list
      setPosts((prev) => [newPost, ...prev]);
      setTitle("");
      setBody("");
    } catch (err) {
      console.error("Failed to create post:", err);
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={title} onChange={(e) => setTitle(e.target.value)} />
      <textarea value={body} onChange={(e) => setBody(e.target.value)} />
      <button disabled={submitting}>
        {submitting ? "Creating..." : "Create Post"}
      </button>
    </form>
  );
}
```

## Update: Editing Data

Updating uses PUT or PATCH:

```jsx
const updatePost = async (id, updates) => {
  const res = await fetch(`/api/posts/${id}`, {
    method: "PATCH",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(updates),
  });
  const updated = await res.json();
  setPosts((prev) =>
    prev.map((p) => (p.id === id ? updated : p))
  );
};
```

## Delete: Removing Data

Deleting is the simplest:

```jsx
const deletePost = async (id) => {
  await fetch(`/api/posts/${id}`, { method: "DELETE" });
  setPosts((prev) => prev.filter((p) => p.id !== id));
};
```

## The CRUD Pattern

Notice the pattern. Every operation follows the same flow:

1. Make the fetch request
2. Wait for the response
3. Update your local state to reflect the change
4. Handle errors

That last step, updating local state after a successful API call, is called **optimistic state management**. You could also refetch the entire list, but updating locally is faster and saves a network request.

These four operations form the backbone of any data-driven React app. Master this pattern and you can build almost anything.
