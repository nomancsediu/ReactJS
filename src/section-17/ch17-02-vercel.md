# Deploying to Vercel

Vercel is the company behind Next.js, and it shows. Deploying a React or Next.js app to Vercel is almost too easy.

## Git Integration

The simplest way to deploy is through Git:

1. Push your code to GitHub, GitLab, or Bitbucket
2. Go to vercel.com and sign in with your Git provider
3. Click "Import Project" and select your repository
4. Vercel detects your framework and configures the build automatically
5. Click "Deploy"

That is really it. Your app is live in about a minute.

## Using the CLI

If you prefer the command line:

```bash
# Install the CLI
npm i -g vercel

# Deploy from your project directory
vercel
```

The CLI walks you through the setup. On subsequent deploys, just run `vercel` again and it deploys your latest changes.

For production deploys:

```bash
vercel --prod
```

## Environment Variables

Set environment variables in the Vercel dashboard under Settings, Environment Variables. You can assign different values to each environment:

- **Production**: Live site
- **Preview**: Pull request previews
- **Development**: Local development

```bash
# Or use the CLI
vercel env add NEXT_PUBLIC_API_URL
vercel env add DATABASE_URL
```

Variables prefixed with `NEXT_PUBLIC_` are exposed to the browser. Everything else stays on the server.

## Preview Deployments

This is one of my favorite features. Every pull request gets a unique preview URL:

```
my-app-git-feature-login-myteam.vercel.app
```

Your team can review the actual running app before merging. If the PR introduces a bug, you catch it before it reaches production.

## Custom Domains

Add your own domain in the Vercel dashboard:

1. Go to Settings, Domains
2. Add your domain
3. Update your DNS records as instructed
4. Vercel handles SSL automatically

## Automatic Optimizations

Vercel adds optimizations you get for free:

- **Edge caching** for static assets
- **Image optimization** through Next.js Image
- **Function bundling** for API routes
- **Analytics** (optional, paid)
- **Speed Insights** (optional, paid)

## vercel.json Configuration

You can customize behavior with a `vercel.json` file:

```json
{
  "rewrites": [
    { "source": "/api/:path*", "destination": "/api/:path*" }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "DENY" }
      ]
    }
  ]
}
```

Vercel is my go-to for React deployments. It is fast, reliable, and the free tier is generous enough for most personal projects. The preview deployments alone make it worth using.
