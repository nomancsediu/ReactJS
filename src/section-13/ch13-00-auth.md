# Authentication in React Apps

If there is one thing that confused me a lot when I started building React apps, it was authentication. I kept asking myself: where do I store the token? How do I protect my pages? What happens when the token expires?

Authentication in React is different from traditional server-rendered apps. In a server-rendered app, the server handles everything. It checks your session cookie on every request. But in a React single-page application (SPA), the browser holds the state. That means you, the developer, need to manage tokens, protect routes, and handle session expiry yourself.

This section is everything I have learned about adding auth to React apps. We will cover the full journey from understanding how authentication works in SPAs to building a complete auth system with context, protected routes, and session management.

Here is what we will explore:

- **How auth works in an SPA** - The flow of tokens, where to store them, and how refresh mechanisms keep users logged in.
- **JWT on the frontend** - Storing access and refresh tokens, dealing with expiry, and automatic refresh.
- **Protected routes** - Building an auth guard component that redirects unauthenticated users to the login page.
- **OAuth in React** - Implementing Google and GitHub login with the popup flow.
- **Firebase Authentication** - Using Firebase for a quick and reliable auth setup.
- **Auth context** - Creating a React context that provides login, logout, and user state throughout your app.
- **Session management** - Persisting sessions, adding "remember me", handling logout, and timing out inactive sessions.

Authentication is not just about logging in. It is about keeping the user safe, keeping their session alive when it should be, and cleaning up when it should not be. I made a lot of mistakes along the way, like storing tokens in the wrong place or forgetting to handle token expiry. I hope this section helps you avoid those same mistakes.

Let us dive in and learn how to build a solid auth system in React.
