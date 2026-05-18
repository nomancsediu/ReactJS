# Axios

Axios is a popular HTTP client that works in both the browser and Node.js. It has been my go-to for a long time because it handles a lot of the annoying stuff that fetch makes you do manually.

## Why Axios Over Fetch?

Here is what convinced me to switch:

- Automatic JSON parsing (no need to call `.json()`)
- Throws on HTTP errors by default (no manual `response.ok` check)
- Request and response interceptors
- Base URL configuration
- Request timeout support
- Automatic request serialization

## Installing Axios

```bash
npm install axios
```

## Basic Setup with Base URL

I always create an instance with a base URL so I do not repeat myself:

```jsx
import axios from "axios";

const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 10000,
  headers: {
    "Content-Type": "application/json",
  },
});

export default api;
```

## Making Requests

```jsx
// GET
const response = await api.get("/users");
console.log(response.data);

// GET with params
const response = await api.get("/users", {
  params: { page: 1, limit: 10 },
});

// POST
const response = await api.post("/users", { name: "John", email: "john@test.com" });

// PUT
const response = await api.put("/users/1", { name: "Jane" });

// DELETE
await api.delete("/users/1");
```

Notice how `response.data` gives you the parsed JSON directly. No `.json()` call needed.

## Interceptors

Interceptors let you run code before every request or after every response. I use them mostly for adding auth tokens:

```jsx
// Request interceptor
api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor for error handling
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Token expired, redirect to login
      window.location.href = "/login";
    }
    return Promise.reject(error);
  }
);
```

## Error Handling

Axios errors have more useful information than fetch errors:

```jsx
try {
  const response = await api.get("/users");
} catch (error) {
  if (axios.isAxiosError(error)) {
    console.log("Status:", error.response?.status);
    console.log("Message:", error.response?.data?.message);
    console.log("URL:", error.config?.url);
  } else {
    console.log("Unexpected error:", error);
  }
}
```

## Fetch vs Axios

| Feature | Fetch | Axios |
|---------|-------|-------|
| JSON parsing | Manual `.json()` | Automatic |
| Error on HTTP errors | No | Yes |
| Interceptors | No | Yes |
| Timeout | No built-in | Built-in |
| Cancel requests | AbortController | CancelToken |
| Browser support | All modern | All + Node.js |

I use Axios for most projects because the developer experience is smoother. But fetch is perfectly fine for simple apps.
