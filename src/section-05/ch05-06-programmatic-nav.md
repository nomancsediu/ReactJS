# Programmatic Navigation

Not all navigation happens through links. Sometimes you need to redirect a user after a form submission, when authentication fails, or when a timer expires. That is where programmatic navigation comes in.

## useNavigate Hook

The `useNavigate` hook returns a function you can call to navigate:

```jsx
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();

  const handleSubmit = (e) => {
    e.preventDefault();
    // After successful login, go to dashboard
    navigate('/dashboard');
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" placeholder="Email" />
      <input type="password" placeholder="Password" />
      <button type="submit">Log In</button>
    </form>
  );
}
```

When the user submits the form, they are immediately taken to `/dashboard`.

## Navigate with Replace

By default, navigating adds a new entry to the browser history. The user can press back to return. Sometimes you do not want that.

```jsx
// After login, user should not go back to login page
navigate('/dashboard', { replace: true });
```

The `replace` option replaces the current history entry instead of adding a new one. I always use this after login and logout flows.

## Go Back

You can navigate relative to the current position:

```jsx
function BackButton() {
  const navigate = useNavigate();

  return (
    <button onClick={() => navigate(-1)}>
      Go Back
    </button>
  );
}
```

`navigate(-1)` goes back one step. `navigate(1)` goes forward. `navigate(-2)` goes back two steps. It works just like hitting the browser back button.

## Redirect After Actions

A common pattern is redirecting after a successful action:

```jsx
function CreatePost() {
  const navigate = useNavigate();

  const handleCreate = async (postData) => {
    const response = await fetch('/api/posts', {
      method: 'POST',
      body: JSON.stringify(postData),
    });

    const newPost = await response.json();
    navigate(`/posts/${newPost.id}`);
  };

  return <PostForm onSubmit={handleCreate} />;
}
```

After creating the post, the user lands on the new post's page.

## Navigate with State

You can pass data along with navigation:

```jsx
navigate('/dashboard', { state: { from: 'login' } });
```

Read it in the destination component with `useLocation`:

```jsx
import { useLocation } from 'react-router-dom';

function Dashboard() {
  const location = useLocation();
  const from = location.state?.from;

  return (
    <div>
      {from === 'login' && <p>Welcome back!</p>}
      <DashboardContent />
    </div>
  );
}
```

## The Navigate Component

For declarative redirects, use the `Navigate` component:

```jsx
import { Navigate } from 'react-router-dom';

function OldPage() {
  return <Navigate to="/new-page" replace />;
}
```

This is useful when a component should always redirect, like when a feature has moved.

## Common Patterns

I use programmatic navigation for:

- Redirecting after form submissions
- Sending unauthenticated users to the login page
- Navigating based on user choices
- Going back after cancel actions

`useNavigate` is the tool I reach for whenever navigation needs to happen as a result of something, not from a link click.
