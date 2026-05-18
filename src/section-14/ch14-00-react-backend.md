# Connecting React to a Backend

A React app by itself is like a car without an engine. It looks great, but it cannot go anywhere without data. At some point, every React app needs to talk to a backend. That backend might be a REST API, a GraphQL server, or even a serverless function. The React side does not really care, as long as it can make HTTP requests and get data back.

When I first started connecting React apps to backends, I ran into all sorts of problems. CORS errors, wrong base URLs, environment variables not loading, and error handling that was all over the place. This section is everything I wish I knew back then.

We will cover these topics:

- **Connecting to Express** - Setting up API calls, configuring base URLs, and using a proxy during development.
- **CORS configuration** - Understanding CORS from the frontend side and setting up a Vite proxy to avoid headaches.
- **Environment variables** - Using `.env` files properly with Vite, the `VITE_` prefix, and managing different environments.
- **API client** - Creating a reusable axios instance with interceptors for tokens, automatic refresh, and consistent error handling.
- **Error patterns** - Building a global error handling system with toast notifications and React error boundaries.
- **Fullstack structure** - Organizing a full-stack project, whether you use a monorepo or separate repositories.

The goal is not just to make API calls work. It is to make them reliable, maintainable, and easy to debug. A well-structured API layer saves you from hours of confusion later.

Let us start building that solid connection between React and the backend.
