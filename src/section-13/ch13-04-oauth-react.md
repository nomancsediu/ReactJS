# OAuth in React (Google and GitHub)

OAuth was intimidating to me at first. All the redirects, tokens, and callbacks felt overwhelming. But once I built it step by step, I realized it is just a fancy way of saying: "Let another website verify who I am."

## The Popup Flow

My preferred way to implement OAuth in React is the popup flow. Instead of redirecting the whole page away from your app, you open a small popup window. When the user finishes authenticating with Google or GitHub, the popup closes and your app receives the token.

```jsx
function OAuthLogin() {
  const handleGoogleLogin = () => {
    // Open popup to your backend OAuth endpoint
    const width = 500;
    const height = 600;
    const left = window.screen.width / 2 - width / 2;
    const top = window.screen.height / 2 - height / 2;

    const popup = window.open(
      '/api/auth/google',
      'Google Login',
      `width=${width},height=${height},left=${left},top=${top}`
    );

    // Listen for message from popup
    window.addEventListener('message', (event) => {
      if (event.data.type === 'OAUTH_SUCCESS') {
        const { token, user } = event.data;
        // Save token and update auth state
        tokenManager.setToken(token);
        setUser(user);
        popup.close();
      }
    });
  };

  return (
    <button onClick={handleGoogleLogin}>
      Sign in with Google
    </button>
  );
}
```

## Handling the Callback

On the backend, when Google or GitHub redirects back to your app, you need a callback route. This route exchanges the authorization code for user data and sends it back to the popup.

```js
// Backend callback route (Express example)
app.get('/api/auth/google/callback', async (req, res) => {
  const { code } = req.query;

  // Exchange code for tokens with Google
  const tokens = await exchangeCodeForTokens(code);

  // Get user info from Google
  const userInfo = await getGoogleUserInfo(tokens.access_token);

  // Find or create user in your database
  const user = await findOrCreateUser(userInfo);

  // Generate your own JWT
  const jwt = generateToken(user);

  // Send token back to the popup window
  res.send(`
    <script>
      window.opener.postMessage({
        type: 'OAUTH_SUCCESS',
        token: '${jwt}',
        user: ${JSON.stringify(user)}
      }, '*');
      window.close();
    </script>
  `);
});
```

## GitHub OAuth

The flow for GitHub is almost identical. You just use GitHub endpoints instead of Google:

```jsx
function GitHubLogin() {
  const handleGitHubLogin = () => {
    const popup = window.open(
      '/api/auth/github',
      'GitHub Login',
      'width=500,height=600'
    );

    window.addEventListener('message', handleOAuthMessage);

    return () => {
      window.removeEventListener('message', handleOAuthMessage);
    };
  };

  return (
    <button onClick={handleGitHubLogin}>
      Sign in with GitHub
    </button>
  );
}
```

## Token Exchange Security

A few things I always keep in mind: use the `state` parameter to prevent CSRF attacks, validate the `redirect_uri` on the server, and never expose your client secret on the frontend. The token exchange must happen on the backend. The frontend should only ever receive your own JWT, never the raw OAuth tokens from the provider.
