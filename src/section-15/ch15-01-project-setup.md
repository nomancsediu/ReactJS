# Project Setup

Every good project starts with a solid setup. I used to rush this part and regret it later when I had to restructure everything. Taking the time to set up the project correctly saves hours down the road.

## Project Structure

We are using a monorepo with three workspaces:

```
product-dashboard/
  client/
    src/
      components/
        ui/           # Reusable UI components
        layout/       # Layout components
        auth/         # Auth-related components
        products/     # Product feature components
      hooks/          # Custom hooks
      lib/            # API client, utilities
      pages/          # Page components
      context/        # React contexts
    public/
    package.json
    vite.config.js
  server/
    src/
      routes/
      controllers/
      models/
      middleware/
    package.json
  shared/
    types/
    package.json
  package.json
```

## Frontend Dependencies

Here are the packages I install for the React frontend:

```bash
npm create vite@latest client -- --template react-ts
cd client
npm install react-router-dom axios react-hot-toast
npm install -D tailwindcss postcss autoprefixer
```

These cover routing, API calls, notifications, and styling. Keep the dependency list small and add more only when needed.

## Backend Dependencies

For the Express backend:

```bash
mkdir server && cd server
npm init -y
npm install express cors dotenv jsonwebtoken bcrypt
npm install -D nodemon @types/express @types/jsonwebtoken
```

## Environment Configuration

Create environment files for both projects:

```bash
# client/.env.development
VITE_API_URL=/api
VITE_APP_TITLE=Product Dashboard

# client/.env.production
VITE_API_URL=https://api.productdashboard.com
VITE_APP_TITLE=Product Dashboard

# server/.env
PORT=3001
JWT_SECRET=your-secret-key
JWT_REFRESH_SECRET=your-refresh-secret
DATABASE_URL=file:./dev.db
```

## Vite Proxy Setup

Connect the frontend to the backend during development:

```js
// client/vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true,
      },
    },
  },
});
```

## Shared Types

Create the shared types that both frontend and backend will use:

```ts
// shared/types/index.ts
export interface User {
  id: string;
  email: string;
  name: string;
  role: 'admin' | 'user';
}

export interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  category: string;
  inStock: boolean;
  createdAt: string;
  updatedAt: string;
}

export interface PaginatedResponse<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
}
```

## Start Scripts

Configure the root `package.json` to run both services:

```json
{
  "name": "product-dashboard",
  "private": true,
  "workspaces": ["client", "server"],
  "scripts": {
    "dev": "concurrently \"npm run dev --workspace=client\" \"npm run dev --workspace=server\"",
    "build": "npm run build --workspace=client && npm run build --workspace=server"
  }
}
```

Install `concurrently` at the root level to run both commands at once. Now `npm run dev` starts everything you need.
