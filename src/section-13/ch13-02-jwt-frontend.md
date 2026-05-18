# Storing JWT on the Frontend

JWT stands for JSON Web Token. It is a string that contains encoded user data and is signed by the server. When I first saw a JWT, it looked like gibberish. But once I understood how to use it on the frontend, everything clicked.

## What Is Inside a JWT?

A JWT has three parts separated by dots: header, payload, and signature. The payload contains claims like user ID, roles, and expiry time.

```js
// Decoding a JWT payload (no library needed)
function decodeToken(token) {
  const base64Url = token.split('.')[1];
  const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
  const jsonPayload = decodeURIComponent(
    atob(base64)
      .split('')
      .map((c) => '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2))
      .join('')
  );

  return JSON.parse(jsonPayload);
}
```

## Access Tokens vs Refresh Tokens

This is something that confused me early on. Why two tokens?

- **Access token** - Short-lived (5 to 15 minutes). Sent with every API request. Stored in memory.
- **Refresh token** - Long-lived (days or weeks). Used only to get a new access token. Stored in an HttpOnly cookie or secure storage.

Using two tokens means if an attacker steals the access token, it expires quickly. The refresh token stays safer because it is used less often.

## Storing Tokens in React

I like to keep the access token in memory and the refresh token in an HttpOnly cookie. Here is a simple approach:

```js
// Token manager
let accessToken = null;

export const tokenManager = {
  getToken: () => accessToken,
  setToken: (token) => {
    accessToken = token;
  },
  clearToken: () => {
    accessToken = null;
  },
  isExpired: () => {
    if (!accessToken) return true;
    const decoded = decodeToken(accessToken);
    return decoded.exp * 1000 < Date.now();
  },
};
```

## Automatic Refresh

The best pattern I have found is to check token expiry before every request and refresh silently if needed.

```js
async function getValidToken() {
  if (tokenManager.isExpired()) {
    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      credentials: 'include', // sends HttpOnly cookie
    });
    const data = await response.json();
    tokenManager.setToken(data.accessToken);
  }
  return tokenManager.getToken();
}
```

## Token Expiry Handling

I always decode the token and check the `exp` claim. If the token is expired, I refresh it. If the refresh fails, I redirect to login. Never let the user see a broken state where their token is expired but the app still thinks they are logged in.

The key lesson I learned: treat tokens carefully. Store them wisely, refresh them automatically, and always have a plan for when they expire.
