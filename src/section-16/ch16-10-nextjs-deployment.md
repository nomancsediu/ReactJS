# Next.js Deployment

You have built your Next.js app. Now it is time to put it on the internet. There are several ways to deploy, and I have tried a few of them.

## Vercel (The Easy Way)

Vercel created Next.js, so deploying there is the smoothest experience.

1. Push your code to GitHub
2. Go to vercel.com and import your repo
3. Click deploy

That is it. Vercel detects Next.js automatically and configures everything.

```bash
# Or use the CLI
npm i -g vercel
vercel
```

### Environment Variables

Set them in the Vercel dashboard under Settings, Environment Variables. You can set different values for Production, Preview, and Development.

```bash
# Or use the CLI
vercel env add DATABASE_URL
```

### Preview Deployments

Every pull request gets its own preview URL. This is amazing for testing changes before merging. Your team can review the actual running app, not just the code.

## Self-Hosting with Node.js

Build the app and run it as a Node.js server:

```bash
npm run build
npm run start
```

The `start` command runs a Node.js server that handles SSR, ISR, and API routes. You can deploy this to any VPS or cloud provider.

## Docker

Here is a basic Dockerfile for Next.js:

```dockerfile
FROM node:20-alpine AS base

# Install dependencies
FROM base AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# Build
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Production
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000
CMD ["node", "server.js"]
```

For standalone output, add this to `next.config.ts`:

```ts
const nextConfig: NextConfig = {
  output: "standalone",
};
```

## Docker Compose

```yaml
version: "3.8"
services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
```

## Optimization Tips

- **Enable compression** in your hosting platform
- **Use `next/image`** for automatic image optimization
- **Add `next/font`** for optimized font loading
- **Run `next analyze`** to check bundle size

```bash
# Analyze your bundle
ANALYZE=true npm run build
```

I started with Vercel because it is zero-config, then moved to Docker when I needed more control. Start simple and add complexity only when you need it.
