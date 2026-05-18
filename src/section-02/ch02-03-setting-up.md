# Setting Up a React Project

Setting up a React project used to be complicated. You needed Webpack, Babel, a dev server, and a bunch of configuration. Today, tools handle all of that for you. Let me show you the easiest ways to get started.

## Vite (Recommended)

Vite is fast and simple. It is my go-to for new projects. Here is how to create one:

```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

That is it. You have a running React app in seconds. Vite starts the dev server instantly and updates the browser as you save files.

## Create React App

This was the standard for years. It still works, but the React team no longer recommends it for new projects:

```bash
npx create-react-app my-app
cd my-app
npm start
```

It is slower to start and slower to rebuild than Vite. If you have an existing CRA project, it still runs fine. Just avoid it for new ones.

## Project Structure

After creating a project with Vite, here is what you typically see:

```
my-app/
  public/
    vite.svg
  src/
    App.jsx        // Main component
    App.css        // Styles for App
    main.jsx       // Entry point
    index.css      // Global styles
  index.html       // HTML template
  package.json
  vite.config.js
```

The important file is `main.jsx`. This is where React attaches to the page:

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

It finds the `<div id="root">` in `index.html` and renders your `App` component inside it.

## StrictMode

You will see `<React.StrictMode>` wrapping the app in development. It runs your components twice to help find bugs. It does not affect production. Keep it on during development. It catches problems early.

## Dev Server

Run `npm run dev` and open the local URL shown in your terminal. The dev server provides:

- **Hot Module Replacement** - changes appear without a full page refresh
- **Error overlay** - syntax and runtime errors display in the browser
- **Fast rebuilds** - Vite only rebuilds what changed

## Customizing the Setup

The default template is minimal on purpose. As your project grows, you will add:

- A `components/` folder for reusable UI pieces
- A `hooks/` folder for custom hooks
- A `pages/` folder if using routing
- A `utils/` folder for helper functions
- A `services/` folder for API calls

Start simple. Add structure when you need it. There is no prize for over-engineering on day one.
