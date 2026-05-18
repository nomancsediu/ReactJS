# CORS From the Frontend Perspective

CORS stands for Cross-Origin Resource Sharing. If you have ever seen an error in your browser console that says "Blocked by CORS policy," you are not alone. It was the first major wall I hit when connecting React to a backend.

## Why CORS Exists

Browsers have a security feature called the Same-Origin Policy. It prevents a website from making requests to a different domain, port, or protocol. This is good for security, but it causes problems when your React app on `localhost:5173` needs to talk to your Express API on `localhost:3001`.

The browser sends a "preflight" OPTIONS request before the actual request. If the server does not respond with the right CORS headers, the browser blocks the request.

## What the Backend Needs to Do

This is a backend configuration, but as a frontend developer, you should understand what needs to happen. The backend must include these headers:

```
Access-Control-Allow-Origin: http://localhost:5173
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
```

If you are using Express, the `cors` middleware handles this:

```js
// Express backend
const cors = require('cors');

app.use(cors({
  origin: 'http://localhost:5173',
  credentials: true,
}));
```

## Vite Proxy Configuration

Instead of dealing with CORS in development, I prefer using Vite's proxy. It makes your React app think the API is on the same origin:

```js
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true,
        // Optional: rewrite the path
        // rewrite: (path) => path.replace(/^\/api/, '')
      },
      // WebSocket proxy if needed
      '/socket.io': {
        target: 'http://localhost:3001',
        ws: true,
      },
    },
  },
});
```

With this setup, all requests to `/api/*` are forwarded to your Express server. The browser sees no cross-origin request, so CORS is never triggered.

## Multiple Proxy Targets

If your backend has multiple services, you can proxy different paths:

```js
server: {
  proxy: {
    '/api': {
      target: 'http://localhost:3001',
      changeOrigin: true,
    },
    '/auth': {
      target: 'http://localhost:3002',
      changeOrigin: true,
    },
    '/uploads': {
      target: 'http://localhost:3003',
      changeOrigin: true,
    },
  },
}
```

## Production CORS

In production, there is no Vite dev server to proxy requests. You have two options:

1. Serve the React build from the Express server (same origin, no CORS needed).
2. Configure the backend to allow your production domain in CORS headers.

I prefer option 1 for simpler deployments. But if your frontend and backend are on different domains, make sure the backend allows your frontend domain specifically, not `*` with credentials.

The key takeaway: use the Vite proxy in development and plan your production CORS strategy early. Do not leave it as an afterthought.
