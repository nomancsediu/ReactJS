# Deploying to Netlify

Netlify is another popular platform for deploying React apps. It has a slightly different feature set than Vercel, and some things it actually does better.

## Quick Deploy

The fastest way is drag and drop:

1. Run `npm run build` locally
2. Go to app.netlify.com
3. Drag your `dist/` or `build/` folder onto the page
4. Your site is live

This is great for quick prototypes. For real projects, use Git integration.

## Git Integration

1. Push your code to GitHub
2. Log in to Netlify and click "Add new site," then "Import an existing project"
3. Select your repository
4. Configure the build settings:
   - Build command: `npm run build`
   - Publish directory: `dist` (Vite) or `build` (CRA)
5. Click "Deploy site"

Netlify rebuilds and redeploys every time you push to your main branch.

## Redirects and Rewrites

React apps use client-side routing, so you need to redirect all routes to `index.html`. Create a `public/_redirects` file:

```
/*    /index.html   200
```

This tells Netlify: for any path, serve `index.html` with a 200 status. Without this, refreshing a page like `/about` returns a 404.

For Next.js apps, Netlify handles this automatically with its Next.js plugin.

## Environment Variables

Set them in the Netlify dashboard under Site settings, Environment variables:

```bash
# Or use the CLI
npm i -g netlify-cli
netlify env:set VITE_API_URL https://api.example.com
```

## Netlify Forms

This is a feature Vercel does not have built in. Netlify can handle form submissions without any backend code:

```html
<form name="contact" method="POST" data-netlify="true">
  <input name="name" type="text" />
  <input name="email" type="email" />
  <button type="submit">Send</button>
</form>
```

In a React app, add a hidden input:

```jsx
<form name="contact" method="POST" data-netlify="true">
  <input type="hidden" name="form-name" value="contact" />
  <input name="name" required />
  <input name="email" type="email" required />
  <button type="submit">Send</button>
</form>
```

Submissions appear in the Netlify dashboard under Forms. You can also get email notifications.

## Netlify Functions

Add serverless functions in a `netlify/functions/` folder:

```js
// netlify/functions/hello.js
export default async () => {
  return new Response(JSON.stringify({ message: "Hello!" }), {
    headers: { "Content-Type": "application/json" },
  });
};
```

This creates an API endpoint at `/.netlify/functions/hello`.

## Deploy Previews

Like Vercel, Netlify creates deploy previews for pull requests. Each PR gets a unique URL where you can test changes before merging.

I use Netlify when I need form handling or when a project is already set up there. It is a solid platform with a generous free tier and great documentation.
