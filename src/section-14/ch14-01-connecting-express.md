# Connecting to an Express Backend

The first time I tried to call an Express API from React, I got a CORS error. Then I hardcoded `localhost:3001` everywhere and broke everything in production. Let me save you from those mistakes.

## Setting Up API Calls

The simplest way to call an API from React is with `fetch`. But raw `fetch` calls scattered across components become messy fast. I prefer creating a small API layer:

```js
// src/lib/api.js
const BASE_URL = '/api';

async function request(endpoint, options = {}) {
  const url = `${BASE_URL}${endpoint}`;
  const config = {
    headers: {
      'Content-Type': 'application/json',
      ...options.headers,
    },
    ...options,
  };

  const response = await fetch(url, config);

  if (!response.ok) {
    const error = await response.json().catch(() => ({}));
    throw new Error(error.message || 'Something went wrong');
  }

  return response.json();
}

export const api = {
  get: (endpoint) => request(endpoint),
  post: (endpoint, data) => request(endpoint, {
    method: 'POST',
    body: JSON.stringify(data),
  }),
  put: (endpoint, data) => request(endpoint, {
    method: 'PUT',
    body: JSON.stringify(data),
  }),
  delete: (endpoint) => request(endpoint, { method: 'DELETE' }),
};
```

Using relative URLs like `/api` instead of `http://localhost:3001/api` is crucial. Relative URLs work in both development and production.

## Base URL Configuration

For cases where you need different base URLs per environment, use environment variables:

```js
const BASE_URL = import.meta.env.VITE_API_URL || '/api';
```

Then in your `.env` files:

```
# .env.development
VITE_API_URL=http://localhost:3001/api

# .env.production
VITE_API_URL=https://api.myapp.com
```

## Proxy in Development

During development, your React app runs on port 5173 (Vite default) and Express runs on port 3001. The browser blocks requests to a different port due to CORS. The clean solution is a proxy.

In Vite, configure the proxy in `vite.config.js`:

```js
// vite.config.js
export default defineConfig({
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true,
      },
    },
  },
});
```

Now when your React app fetches `/api/users`, Vite forwards it to `http://localhost:3001/api/users`. No CORS issues. No hardcoded URLs.

## Using the API in Components

With the API layer in place, components stay clean:

```jsx
import { useState, useEffect } from 'react';
import { api } from '../lib/api';

function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    api.get('/users')
      .then(setUsers)
      .catch((err) => console.error(err))
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <p>Loading users...</p>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

The component does not need to know about base URLs, headers, or error parsing. The API layer handles all of that. That separation of concern makes the codebase much easier to maintain as it grows.
