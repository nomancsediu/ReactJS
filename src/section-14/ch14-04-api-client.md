# Creating an API Client with Axios

Raw `fetch` works fine for simple projects. But once your app grows, you end up repeating the same headers, error handling, and token logic in every call. That is when I switch to axios and build a proper API client.

## Why Axios?

Axios gives you interceptors, automatic JSON parsing, request cancellation, and better error objects. It is not magic, but it saves a lot of boilerplate.

```bash
npm install axios
```

## Creating the Instance

I create one configured axios instance and use it everywhere:

```js
// src/lib/apiClient.js
import axios from 'axios';
import { tokenManager } from './tokenManager';

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL || '/api',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
  withCredentials: true,
});

export default apiClient;
```

## Request Interceptor for Tokens

Every request needs the authorization header. Instead of adding it manually, I use a request interceptor:

```js
apiClient.interceptors.request.use(
  (config) => {
    const token = tokenManager.getToken();
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);
```

Now every request automatically includes the token. No more forgetting to add the header.

## Response Interceptor for Token Refresh

The most useful interceptor I have built handles automatic token refresh. When a request fails with 401, the interceptor silently refreshes the token and retries the request:

```js
let isRefreshing = false;
let failedQueue = [];

apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    // Skip refresh for login/refresh endpoints
    if (error.response?.status !== 401 || originalRequest._retry) {
      return Promise.reject(error);
    }

    if (isRefreshing) {
      // Queue this request until refresh completes
      return new Promise((resolve, reject) => {
        failedQueue.push({ resolve, reject });
      }).then((token) => {
        originalRequest.headers.Authorization = `Bearer ${token}`;
        return apiClient(originalRequest);
      });
    }

    originalRequest._retry = true;
    isRefreshing = true;

    try {
      const response = await axios.post('/api/auth/refresh', {}, {
        withCredentials: true,
      });
      const { accessToken } = response.data;
      tokenManager.setToken(accessToken);

      // Retry all queued requests
      failedQueue.forEach(({ resolve }) => resolve(accessToken));
      failedQueue = [];

      originalRequest.headers.Authorization = `Bearer ${accessToken}`;
      return apiClient(originalRequest);
    } catch (refreshError) {
      failedQueue.forEach(({ reject }) => reject(refreshError));
      failedQueue = [];
      tokenManager.clearToken();
      window.location.href = '/login';
      return Promise.reject(refreshError);
    } finally {
      isRefreshing = false;
    }
  }
);
```

This pattern handles the case where multiple requests fail at the same time. Only one refresh call happens, and all queued requests are retried after the new token arrives.

## Using the Client

Components stay clean and focused:

```jsx
import { useEffect, useState } from 'react';
import apiClient from '../lib/apiClient';

function ProductList() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    apiClient.get('/products')
      .then((res) => setProducts(res.data))
      .catch((err) => console.error(err));
  }, []);

  return (
    <ul>
      {products.map((p) => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}
```

The API client handles tokens, refresh, and errors. The component just calls the endpoint and renders the data.
