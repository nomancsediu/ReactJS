# Form Basics in React

Before we get into fancy form libraries, let us understand how forms work natively in React. It is important to know the basics so you appreciate what the libraries do for you later.

## The HTML Form Problem

In plain HTML, a form submits a request and reloads the page. That is how the web originally worked. But in a React single-page app, we do not want full page reloads. We want to handle the submission ourselves with JavaScript.

## Preventing Default Behavior

The first thing you learn about React forms is preventing the default submission:

```jsx
function ContactForm() {
  const handleSubmit = (event) => {
    event.preventDefault();
    console.log("Form submitted!");
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" name="name" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

That `event.preventDefault()` call is critical. Without it, the browser will try to submit the form the traditional way and reload the page. I forgot this so many times when I started.

## Getting Form Values

There are a couple ways to get the values from a form. The simplest is using the form element itself:

```jsx
function LoginForm() {
  const handleSubmit = (event) => {
    event.preventDefault();
    const formData = new FormData(event.target);
    const email = formData.get("email");
    const password = formData.get("password");
    console.log(email, password);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" name="email" />
      <input type="password" name="password" />
      <button type="submit">Log In</button>
    </form>
  );
}
```

Using `FormData` is nice because you do not need any state at all. The DOM holds the values. This is sometimes called an "uncontrolled" approach.

## Button Types Matter

One thing that caught me off guard: if you have a button inside a form without specifying the type, it defaults to `type="submit"`. That means clicking it will trigger form submission. If you want a button that does not submit, you must use `type="button"`:

```jsx
<form onSubmit={handleSubmit}>
  <input name="search" />
  <button type="submit">Search</button>
  <button type="button" onClick={clearForm}>Clear</button>
</form>
```

Understanding these basics sets the foundation for everything else. Next, we look at controlled forms where React state drives the inputs.
