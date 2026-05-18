# Global Error Handling Patterns

Scattered error handling is a mess. I used to write `try/catch` blocks in every component, showing different error messages in different ways. It was inconsistent and hard to maintain. Then I learned to centralize error handling with a global approach.

## Global Error Handler

First, I create an error handler that categorizes errors and shows the right message:

```js
// src/lib/errorHandler.js
import { toast } from 'react-hot-toast';

export function handleApiError(error) {
  if (error.response) {
    // Server responded with an error status
    const { status, data } = error.response;

    switch (status) {
      case 400:
        toast.error(data.message || 'Invalid request');
        break;
      case 401:
        toast.error('Please log in again');
        break;
      case 403:
        toast.error('You do not have permission');
        break;
      case 404:
        toast.error('Resource not found');
        break;
      case 422:
        // Validation errors
        const errors = data.errors || {};
        Object.values(errors).flat().forEach((msg) => {
          toast.error(msg);
        });
        break;
      case 500:
        toast.error('Server error. Please try again later.');
        break;
      default:
        toast.error(data.message || 'Something went wrong');
    }
  } else if (error.request) {
    // No response received
    toast.error('Network error. Check your connection.');
  } else {
    toast.error('An unexpected error occurred');
  }
}
```

## Toast Notifications

I use `react-hot-toast` for notifications. It is simple and looks good:

```bash
npm install react-hot-toast
```

```jsx
import { Toaster } from 'react-hot-toast';

function App() {
  return (
    <>
      <Toaster
        position="top-right"
        toastOptions={{
          duration: 4000,
          style: {
            borderRadius: '8px',
            padding: '12px 16px',
          },
        }}
      />
      <Routes>{/* your routes */}</Routes>
    </>
  );
}
```

## Axios Error Interceptor

Hook the error handler into the axios interceptor so every API error gets handled automatically:

```js
import { handleApiError } from './errorHandler';

apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    // Skip 401 here if handled by refresh interceptor
    if (error.response?.status !== 401) {
      handleApiError(error);
    }
    return Promise.reject(error);
  }
);
```

## React Error Boundaries

API errors are one thing, but what about React rendering errors? A single broken component can crash the entire app. Error boundaries catch these:

```jsx
import { Component } from 'react';

class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    // Log to an error reporting service
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-container">
          <h2>Something went wrong</h2>
          <p>{this.state.error.message}</p>
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}

export default ErrorBoundary;
```

Wrap your routes with the boundary:

```jsx
function App() {
  return (
    <ErrorBoundary>
      <AuthProvider>
        <Routes>
          <Route path="/*" element={<Layout />} />
        </Routes>
      </AuthProvider>
    </ErrorBoundary>
  );
}
```

The result: API errors show toasts, and rendering errors show a fallback UI. The user never sees a blank white screen. That is the kind of reliability I aim for in every project.
