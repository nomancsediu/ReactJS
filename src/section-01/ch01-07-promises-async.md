# Promises and Async/Await

React apps fetch data from APIs. That means dealing with asynchronous code. Promises and async/await are how JavaScript handles things that take time. I found async code confusing at first, so let me break it down simply.

## What Is a Promise?

A promise represents a value that will be available later. It has three states:

- **Pending** - still waiting
- **Fulfilled** - got the result
- **Rejected** - something went wrong

```js
const fetchData = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({ name: "Alex" }) // success
      // or reject(new Error("Something went wrong")) // failure
    }, 1000)
  })
}
```

## .then() and .catch()

The traditional way to handle promises:

```js
fetchData()
  .then(data => {
    console.log(data.name) // "Alex"
    return data.name.toUpperCase()
  })
  .then(upper => {
    console.log(upper) // "ALEX"
  })
  .catch(error => {
    console.error(error)
  })
```

Chaining `.then()` works but gets messy with complex logic.

## async/await

This is the modern way. It looks like synchronous code but works asynchronously:

```js
async function loadUser() {
  try {
    const data = await fetchData()
    console.log(data.name) // "Alex"
  } catch (error) {
    console.error(error)
  }
}
```

`async` marks a function as asynchronous. `await` pauses execution until the promise resolves. `try/catch` handles errors instead of `.catch()`.

## Promise.all

When you have multiple promises and want to wait for all of them:

```js
async function loadDashboard() {
  const [users, posts, comments] = await Promise.all([
    fetchUsers(),
    fetchPosts(),
    fetchComments()
  ])
  // All three are done, proceed together
}
```

`Promise.all` runs them in parallel and resolves when all finish. If any one fails, the whole thing fails. Use this when requests do not depend on each other.

## In React

Fetching data in a component looks like this:

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)

  useEffect(() => {
    async function loadUser() {
      try {
        const response = await fetch(`/api/users/${userId}`)
        const data = await response.json()
        setUser(data)
      } catch (err) {
        setError(err.message)
      } finally {
        setLoading(false)
      }
    }
    loadUser()
  }, [userId])

  if (loading) return <p>Loading...</p>
  if (error) return <p>Error: {error}</p>
  return <h1>{user.name}</h1>
}
```

The pattern of loading, error, and data states is something you will write over and over. Get comfortable with async/await because almost every React app needs it.
