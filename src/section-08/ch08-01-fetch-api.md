# The Fetch API

The Fetch API is built into every modern browser. No libraries needed. It is the simplest way to make HTTP requests from React. Let me show you how to use it for all the common operations.

## Basic GET Request

```jsx
useEffect(() => {
  fetch("https://api.example.com/users")
    .then((response) => {
      if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`);
      }
      return response.json();
    })
    .then((data) => console.log(data))
    .catch((error) => console.error("Fetch failed:", error));
}, []);
```

One thing that tripped me up: `fetch` does not throw on HTTP errors like 404 or 500. It only throws on network failures. That is why you need to check `response.ok` manually.

## POST Request

```jsx
const createUser = async (userData) => {
  const response = await fetch("https://api.example.com/users", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(userData),
  });

  if (!response.ok) {
    throw new Error("Failed to create user");
  }

  return response.json();
};
```

## PUT Request

```jsx
const updateUser = async (id, userData) => {
  const response = await fetch(`https://api.example.com/users/${id}`, {
    method: "PUT",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(userData),
  });

  if (!response.ok) {
    throw new Error("Failed to update user");
  }

  return response.json();
};
```

## DELETE Request

```jsx
const deleteUser = async (id) => {
  const response = await fetch(`https://api.example.com/users/${id}`, {
    method: "DELETE",
  });

  if (!response.ok) {
    throw new Error("Failed to delete user");
  }
};
```

## Adding Headers

Headers are how you send auth tokens, content types, and other metadata:

```jsx
const response = await fetch("https://api.example.com/users", {
  headers: {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${token}`,
    "Accept": "application/json",
  },
});
```

## Error Handling Pattern

Here is the pattern I use for robust error handling with fetch:

```jsx
async function apiFetch(url, options = {}) {
  try {
    const response = await fetch(url, {
      headers: {
        "Content-Type": "application/json",
        ...options.headers,
      },
      ...options,
    });

    if (!response.ok) {
      const errorData = await response.json().catch(() => ({}));
      throw new Error(errorData.message || `Request failed with status ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    console.error("API request failed:", error);
    throw error;
  }
}
```

Fetch is great for simple use cases. For more features like interceptors and automatic transforms, Axios is the next step.
