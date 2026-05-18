# Pagination and Infinite Scroll

When you have hundreds or thousands of items, loading them all at once is a bad idea. Your API will be slow, your app will be heavy, and your users will wait forever. I learned this when my first app tried to load 10,000 posts on one page. Not fun.

## Basic Pagination UI

The simplest approach is page-based pagination:

```jsx
function PostList() {
  const [page, setPage] = useState(1);
  const [posts, setPosts] = useState([]);
  const [totalPages, setTotalPages] = useState(1);

  useEffect(() => {
    fetch(`/api/posts?page=${page}&limit=20`)
      .then((res) => res.json())
      .then((data) => {
        setPosts(data.posts);
        setTotalPages(data.totalPages);
      });
  }, [page]);

  return (
    <div>
      {posts.map((post) => <PostCard key={post.id} post={post} />)}
      <nav className="flex gap-2 justify-center mt-4">
        <button
          disabled={page === 1}
          onClick={() => setPage((p) => p - 1)}
        >
          Previous
        </button>
        <span>Page {page} of {totalPages}</span>
        <button
          disabled={page === totalPages}
          onClick={() => setPage((p) => p + 1)}
        >
          Next
        </button>
      </nav>
    </div>
  );
}
```

## Infinite Scroll with Intersection Observer

Infinite scroll loads more items as the user scrolls down. The Intersection Observer API tells you when an element enters the viewport:

```jsx
function InfinitePostList() {
  const [posts, setPosts] = useState([]);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  const loaderRef = useRef(null);

  useEffect(() => {
    fetch(`/api/posts?page=${page}&limit=20`)
      .then((res) => res.json())
      .then((data) => {
        setPosts((prev) => (page === 1 ? data.posts : [...prev, ...data.posts]));
        setHasMore(data.posts.length === 20);
      });
  }, [page]);

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && hasMore) {
          setPage((p) => p + 1);
        }
      },
      { threshold: 1.0 }
    );

    if (loaderRef.current) observer.observe(loaderRef.current);
    return () => observer.disconnect();
  }, [hasMore]);

  return (
    <div>
      {posts.map((post) => <PostCard key={post.id} post={post} />)}
      {hasMore && <div ref={loaderRef} className="py-8 text-center">Loading more...</div>}
      {!hasMore && <p className="py-4 text-center text-gray-500">No more posts</p>}
    </div>
  );
}
```

When the loader div scrolls into view, we load the next page. The `hasMore` flag stops requesting when there is nothing left.

## Cursor-Based Pagination

Page numbers can skip items if data changes between requests. Cursor-based pagination avoids this by using a pointer to the last item:

```jsx
// API returns: { posts, nextCursor: "abc123" }
fetch(`/api/posts?cursor=${cursor}&limit=20`)
```

The cursor is usually an ID or timestamp. It guarantees you never see the same item twice, even if new items are inserted between loads.

Pick page-based for simple apps, cursor-based for real-time or large datasets, and infinite scroll when users browse casually without needing to jump to specific pages.
