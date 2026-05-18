# How Auth Works in SPA

When I first learned about authentication in single-page apps, I was surprised how different it is from traditional websites. In a traditional website, the server sets a session cookie, and the browser sends it automatically on every request. Done. But in an SPA, things work differently.

## The Typical Auth Flow

Here is what usually happens when a user logs in to a React app:

1. The user enters their email and password.
2. React sends a POST request to the login API.
3. The server validates the credentials and returns a token (usually a JWT).
4. React stores that token somewhere in the browser.
5. On every future API request, React attaches the token in the Authorization header.
6. The server checks the token and either allows or denies the request.

```js
// A basic login flow
async function login(email, password) {
  const response = await fetch('/api/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
  });

  const data = await response.json();

  if (data.token) {
    localStorage.setItem('token', data.token);
  }
}
```

## Token Storage

Where you store the token matters a lot. I learned this the hard way. The three common options are:

- **localStorage** - Persists across page reloads but vulnerable to XSS attacks.
- **sessionStorage** - Cleared when the tab closes, still vulnerable to XSS.
- **HttpOnly cookies** - Set by the server, not accessible via JavaScript. Best for security but harder to configure with CORS.

For most SPAs, a common pattern is storing the access token in memory and the refresh token in an HttpOnly cookie.

## Refresh Mechanism

Access tokens are short-lived (maybe 15 minutes). When they expire, you use a refresh token to get a new one without making the user log in again.

```js
async function fetchWithRefresh(url, options = {}) {
  let response = await fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      Authorization: `Bearer ${getToken()}`,
    },
  });

  if (response.status === 401) {
    // Token expired, try to refresh
    const newToken = await refreshToken();
    response = await fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        Authorization: `Bearer ${newToken}`,
      },
    });
  }

  return response;
}
```

## Security Considerations

I keep these rules in mind now: never store sensitive tokens in localStorage for production apps, always use HTTPS, and set short expiry times on access tokens. The refresh token should be stored in an HttpOnly cookie if possible. Also, always validate tokens on the server side. The frontend is just the messenger.
