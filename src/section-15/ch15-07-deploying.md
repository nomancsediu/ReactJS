# Deploying Your Full-Stack App

Deployment used to scare me. There are so many options and it felt like one wrong step would break everything. But once I understood the basics, it became routine. Let me walk you through deploying a React frontend and Express backend.

## Building the Frontend

Before deploying, build the React app for production:

```bash
cd client
npm run build
```

This creates a `dist/` folder with optimized static files. These are the files you deploy.

## Option 1: Serve Frontend from Express

The simplest approach is to serve the React build from Express. No CORS issues, no separate deployment:

```js
// server/src/index.js
import express from 'express';
import path from 'path';
import { fileURLToPath } from 'url';

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const app = express();

// API routes
app.use('/api', apiRoutes);

// Serve React static files
app.use(express.static(path.join(__dirname, '../../client/dist')));

// SPA fallback: serve index.html for all non-API routes
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, '../../client/dist/index.html'));
});

app.listen(3001, () => {
  console.log('Server running on port 3001');
});
```

## Option 2: Deploy Separately

For larger apps, deploy frontend and backend separately:

**Frontend on Vercel or Netlify:**

```bash
# Vercel
cd client
npx vercel

# Or connect your GitHub repo in the Vercel dashboard
```

**Backend on Railway or Render:**

Push your server code to a platform like Railway. It detects Express automatically and runs it.

When deploying separately, configure CORS on the backend to allow your frontend domain:

```js
app.use(cors({
  origin: 'https://your-app.vercel.app',
  credentials: true,
}));
```

## Environment Variables

Never hardcode environment-specific values. Set them in your deployment platform:

**Vercel environment variables:**
```
VITE_API_URL=https://api.yourapp.com
```

**Railway environment variables:**
```
PORT=3001
JWT_SECRET=production-secret-key
DATABASE_URL=postgresql://...
NODE_ENV=production
```

## Domain Setup

If you want a custom domain:

1. Buy a domain from a registrar (Namecheap, Google Domains).
2. Add DNS records pointing to your hosting provider.
3. Configure SSL (most platforms like Vercel and Railway do this automatically).

For a setup where frontend is at `yourapp.com` and backend is at `api.yourapp.com`:

```
# DNS records
A     yourapp.com        -> 76.76.21.21 (Vercel)
CNAME api.yourapp.com    -> yourapp.railway.app
```

## Deployment Checklist

Before going live, I always check these items:

- All environment variables are set on the deployment platform
- CORS is configured for the production domain
- The React build runs without errors (`npm run build` locally first)
- API endpoints return correct responses
- SSL/HTTPS is enabled
- Database migrations have been run
- Error logging is set up (I use a simple error logging middleware)

## Continuous Deployment

The best part of modern deployment is automation. Connect your GitHub repo to Vercel or Railway, and every push to the main branch triggers a new deployment. No manual steps needed.

```yaml
# .github/workflows/deploy.yml (optional GitHub Actions)
name: Deploy
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: cd client && npm ci && npm run build
      - run: cd server && npm ci
      # Add deployment commands for your platform
```

Deployment is not the end. It is the beginning of maintaining a live application. Start simple, automate what you can, and improve as you go.
