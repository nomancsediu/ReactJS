# Fullstack Project Structure

How you organize your full-stack project matters more than you think. I started with React in one folder and Express in another, and it was chaos. Different repos, no shared types, and deployment was a nightmare. Over time, I found structures that actually work.

## Monorepo vs Separate Repos

**Monorepo** means frontend and backend live in the same repository. **Separate repos** means each has its own repository.

| Aspect | Monorepo | Separate Repos |
|--------|----------|----------------|
| Shared code | Easy | Difficult |
| Deployment | Can be coupled | Independent |
| Team scaling | Simpler | More flexible |
| CI/CD | Single pipeline | Multiple pipelines |

I prefer monorepos for most projects. They are simpler to set up and maintain, especially for small teams.

## Monorepo Structure

Here is the structure I use:

```
my-app/
  client/              # React frontend
    src/
      components/
      hooks/
      lib/
      pages/
      styles/
    public/
    package.json
    vite.config.js
  server/              # Express backend
    src/
      routes/
      controllers/
      models/
      middleware/
    package.json
  shared/              # Shared code between client and server
    types/
      api.ts
      models.ts
    validators/
    package.json
  package.json         # Root package.json with workspaces
```

## Shared Types

One of the biggest advantages of a monorepo is sharing TypeScript types between frontend and backend. No more mismatched API contracts:

```ts
// shared/types/api.ts
export interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

export interface ApiResponse<T> {
  data: T;
  message: string;
}

export interface LoginRequest {
  email: string;
  password: string;
}

export interface LoginResponse {
  user: User;
  accessToken: string;
}
```

Use these types on both sides:

```ts
// server/src/controllers/auth.ts
import type { LoginRequest, LoginResponse } from '../../../shared/types/api';

// client/src/lib/api.ts
import type { LoginRequest, LoginResponse } from '../../../shared/types/api';
```

## Workspace Configuration

With npm or bun workspaces, you can manage dependencies from the root:

```json
// Root package.json
{
  "name": "my-app",
  "private": true,
  "workspaces": ["client", "server", "shared"]
}
```

Now you can run scripts from the root:

```json
{
  "scripts": {
    "dev": "concurrently \"npm run dev --workspace=client\" \"npm run dev --workspace=server\"",
    "build": "npm run build --workspace=client && npm run build --workspace=server"
  }
}
```

## Separate Repos (When to Choose)

Sometimes separate repos make more sense. If your frontend and backend teams work independently, or if they deploy on completely different schedules, separate repos give you more flexibility. Just be aware that you lose easy type sharing and need another way to keep API contracts in sync (like OpenAPI specs or a shared npm package).

My advice: start with a monorepo. You can always split later if needed. But going from separate repos to a monorepo is much harder than the other way around.
