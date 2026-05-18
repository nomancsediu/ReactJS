# What Is Routing?

Before I understood routing, every web page I visited worked the same way. I clicked a link, the browser made a request to the server, the server sent back a full HTML page, and the browser rendered it from scratch. That is server-side routing.

## Server-Side Routing

Traditional websites use server-side routing. Every URL corresponds to a file on the server.

```
/about.html  -> Server reads about.html  -> Sends full HTML back
/contact.html -> Server reads contact.html -> Sends full HTML back
```

Each navigation triggers a full page reload. The browser goes blank for a moment, then the new page appears. It works, but it is slow and jarring.

## Client-Side Routing

Single-page apps take a different approach. The server sends one HTML file and one JavaScript bundle. After that, JavaScript handles all navigation without talking to the server again.

```
/about  -> JavaScript updates the DOM -> New view appears instantly
/contact -> JavaScript updates the DOM -> New view appears instantly
```

No full reload. No blank flash. The page updates in place.

## The History API

Client-side routing works because of the browser's History API. It lets JavaScript change the URL without reloading the page.

```js
// Change the URL without reloading
window.history.pushState({}, '', '/about');

// Go back
window.history.back();

// Listen for navigation changes
window.addEventListener('popstate', () => {
  console.log('User navigated', window.location.pathname);
});
```

React Router wraps the History API in a nice React-friendly interface. You do not need to use the History API directly, but understanding it helps.

## The Key Difference

| Server Routing | Client Routing |
|---|---|
| Server renders each page | JavaScript renders each page |
| Full page reload on navigation | No reload, DOM updates in place |
| Each URL is a separate file | One HTML file, JavaScript handles URLs |
| Slower transitions | Instant transitions |

## Why This Matters for React

React apps are single-page applications. The server sends your React app once, and then React takes over. Without client-side routing, your app would be stuck on one URL forever.

React Router gives you the best of both worlds: the speed of a single-page app with the familiarity of traditional URL-based navigation. Users can bookmark pages, share links, and use the back button just like they expect.

When I first saw a React app update the URL without reloading, it felt like magic. Behind the scenes, it is just the History API and some clever DOM updates. Now let us set it up ourselves.
