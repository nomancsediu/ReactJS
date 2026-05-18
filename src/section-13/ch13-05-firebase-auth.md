# Firebase Authentication

When I want to add authentication quickly without building a backend, Firebase Auth is my go-to. It handles user management, password resets, email verification, and social logins out of the box. The setup is surprisingly simple.

## Firebase Setup

First, install the Firebase SDK and initialize it in your app:

```bash
npm install firebase
```

```js
// src/lib/firebase.js
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';

const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_FIREBASE_MESSAGING_SENDER_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID,
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
```

Never hardcode these values. Always use environment variables with the `VITE_` prefix.

## Email and Password Login

Firebase makes email/password auth very straightforward:

```jsx
import { useState } from 'react';
import { createUserWithEmailAndPassword, signInWithEmailAndPassword } from 'firebase/auth';
import { auth } from '../lib/firebase';

function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleRegister = async (e) => {
    e.preventDefault();
    try {
      await createUserWithEmailAndPassword(auth, email, password);
    } catch (error) {
      console.error(error.message);
    }
  };

  const handleLogin = async (e) => {
    e.preventDefault();
    try {
      await signInWithEmailAndPassword(auth, email, password);
    } catch (error) {
      console.error(error.message);
    }
  };

  return (
    <form onSubmit={handleLogin}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button type="submit">Log In</button>
      <button type="button" onClick={handleRegister}>Register</button>
    </form>
  );
}
```

## Google Sign-In

Adding Google sign-in is just a few lines:

```jsx
import { GoogleAuthProvider, signInWithPopup } from 'firebase/auth';
import { auth } from '../lib/firebase';

function GoogleLoginButton() {
  const handleGoogleLogin = async () => {
    const provider = new GoogleAuthProvider();
    try {
      const result = await signInWithPopup(auth, provider);
      const user = result.user;
      console.log('Logged in as:', user.displayName);
    } catch (error) {
      console.error(error.message);
    }
  };

  return (
    <button onClick={handleGoogleLogin}>
      Sign in with Google
    </button>
  );
}
```

## Auth State Observer

The most powerful feature of Firebase Auth is the `onAuthStateChanged` observer. It listens for changes in auth state and fires whenever the user logs in, logs out, or when the token refreshes automatically.

```jsx
import { onAuthStateChanged } from 'firebase/auth';
import { auth } from '../lib/firebase';

// In your AuthProvider component
useEffect(() => {
  const unsubscribe = onAuthStateChanged(auth, (user) => {
    if (user) {
      setUser(user);
    } else {
      setUser(null);
    }
    setLoading(false);
  });

  return () => unsubscribe();
}, []);
```

This observer handles token refresh for you. Firebase automatically refreshes the ID token about 5 minutes before it expires. That is one less thing to worry about.
