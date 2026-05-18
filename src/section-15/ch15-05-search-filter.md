# Search and Filter

A product list without search is frustrating. Users need to find what they are looking for quickly. In this chapter, we add a search bar, category filters, and keep the URL in sync with the filter state so users can share filtered views.

## Search Bar with Debounce

Typing in a search bar triggers a new API call on every keystroke. That is too many requests. I use a debounce to wait until the user stops typing:

```jsx
import { useState, useEffect } from 'react';

function useDebounce(value, delay = 300) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

function SearchBar({ value, onChange }) {
  return (
    <input
      type="text"
      placeholder="Search products..."
      value={value}
      onChange={(e) => onChange(e.target.value)}
      className="search-input"
    />
  );
}
```

## Filter Dropdowns

Category and stock filters let users narrow down results:

```jsx
function FilterBar({ category, inStock, onCategoryChange, onStockChange }) {
  return (
    <div className="filter-bar">
      <select
        value={category}
        onChange={(e) => onCategoryChange(e.target.value)}
      >
        <option value="">All Categories</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
        <option value="food">Food</option>
      </select>

      <select
        value={inStock}
        onChange={(e) => onStockChange(e.target.value)}
      >
        <option value="">All Stock Status</option>
        <option value="true">In Stock</option>
        <option value="false">Out of Stock</option>
      </select>
    </div>
  );
}
```

## URL Sync with Search Params

Keeping filters in the URL means users can bookmark or share filtered views. I use `useSearchParams` from React Router:

```jsx
import { useSearchParams } from 'react-router-dom';
import apiClient from '../lib/apiClient';

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(false);

  // Read filters from URL
  const search = searchParams.get('search') || '';
  const category = searchParams.get('category') || '';
  const inStock = searchParams.get('inStock') || '';

  // Debounce the search term
  const debouncedSearch = useDebounce(search, 300);

  // Update URL params
  const updateFilter = (key, value) => {
    setSearchParams((prev) => {
      if (value) {
        prev.set(key, value);
      } else {
        prev.delete(key);
      }
      return prev;
    });
  };

  // Fetch products when filters change
  useEffect(() => {
    async function fetchProducts() {
      setLoading(true);
      try {
        const params = new URLSearchParams();
        if (debouncedSearch) params.set('search', debouncedSearch);
        if (category) params.set('category', category);
        if (inStock) params.set('inStock', inStock);

        const res = await apiClient.get(`/products?${params.toString()}`);
        setProducts(res.data);
      } catch (err) {
        console.error(err);
      } finally {
        setLoading(false);
      }
    }

    fetchProducts();
  }, [debouncedSearch, category, inStock]);

  return (
    <div>
      <SearchBar
        value={search}
        onChange={(val) => updateFilter('search', val)}
      />
      <FilterBar
        category={category}
        inStock={inStock}
        onCategoryChange={(val) => updateFilter('category', val)}
        onStockChange={(val) => updateFilter('inStock', val)}
      />

      {loading ? (
        <p>Loading...</p>
      ) : (
        <table>
          <thead>
            <tr>
              <th>Name</th>
              <th>Category</th>
              <th>Price</th>
              <th>Stock</th>
            </tr>
          </thead>
          <tbody>
            {products.map((p) => (
              <tr key={p.id}>
                <td>{p.name}</td>
                <td>{p.category}</td>
                <td>${p.price.toFixed(2)}</td>
                <td>{p.inStock ? 'Yes' : 'No'}</td>
              </tr>
            ))}
          </tbody>
        </table>
      )}
    </div>
  );
}
```

The URL sync pattern is one of my favorites. When a user searches for "laptop" in the electronics category, the URL becomes `/dashboard/products?search=laptop&category=electronics`. They can bookmark it, share it, or use the back button to return to it. That is a much better experience than losing your filters on every navigation.
