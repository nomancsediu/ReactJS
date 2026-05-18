# CRUD Module: Product Management

CRUD stands for Create, Read, Update, Delete. It is the backbone of most web applications. We will build a product management module with a table view, a form for creating and editing products, and a confirmation dialog for deleting.

## Product List with Table

The table view displays all products and provides actions for each row:

```jsx
import { useState, useEffect } from 'react';
import apiClient from '../lib/apiClient';
import ProductForm from './ProductForm';
import DeleteConfirm from './DeleteConfirm';

function ProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [showForm, setShowForm] = useState(false);
  const [editingProduct, setEditingProduct] = useState(null);
  const [deletingProduct, setDeletingProduct] = useState(null);

  const fetchProducts = async () => {
    setLoading(true);
    try {
      const res = await apiClient.get('/products');
      setProducts(res.data);
    } catch (err) {
      console.error(err);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchProducts();
  }, []);

  const handleEdit = (product) => {
    setEditingProduct(product);
    setShowForm(true);
  };

  const handleDelete = async (product) => {
    try {
      await apiClient.delete(`/products/${product.id}`);
      setProducts((prev) => prev.filter((p) => p.id !== product.id));
      setDeletingProduct(null);
    } catch (err) {
      console.error(err);
    }
  };

  const handleFormSave = () => {
    setShowForm(false);
    setEditingProduct(null);
    fetchProducts();
  };

  if (loading) return <p>Loading products...</p>;

  return (
    <div>
      <div className="page-header">
        <h2>Products</h2>
        <button onClick={() => { setEditingProduct(null); setShowForm(true); }}>
          Add Product
        </button>
      </div>

      <table className="product-table">
        <thead>
          <tr>
            <th>Name</th>
            <th>Category</th>
            <th>Price</th>
            <th>In Stock</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {products.map((product) => (
            <tr key={product.id}>
              <td>{product.name}</td>
              <td>{product.category}</td>
              <td>${product.price.toFixed(2)}</td>
              <td>{product.inStock ? 'Yes' : 'No'}</td>
              <td>
                <button onClick={() => handleEdit(product)}>Edit</button>
                <button onClick={() => setDeletingProduct(product)}>Delete</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>

      {showForm && (
        <ProductForm
          product={editingProduct}
          onSave={handleFormSave}
          onClose={() => { setShowForm(false); setEditingProduct(null); }}
        />
      )}

      {deletingProduct && (
        <DeleteConfirm
          product={deletingProduct}
          onConfirm={() => handleDelete(deletingProduct)}
          onCancel={() => setDeletingProduct(null)}
        />
      )}
    </div>
  );
}
```

## Create and Edit Form

One form handles both creating and editing:

```jsx
import { useState, useEffect } from 'react';
import apiClient from '../lib/apiClient';

function ProductForm({ product, onSave, onClose }) {
  const [name, setName] = useState('');
  const [description, setDescription] = useState('');
  const [price, setPrice] = useState('');
  const [category, setCategory] = useState('');
  const [inStock, setInStock] = useState(true);
  const [saving, setSaving] = useState(false);

  const isEditing = !!product;

  useEffect(() => {
    if (product) {
      setName(product.name);
      setDescription(product.description);
      setPrice(product.price.toString());
      setCategory(product.category);
      setInStock(product.inStock);
    }
  }, [product]);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setSaving(true);

    const payload = {
      name,
      description,
      price: parseFloat(price),
      category,
      inStock,
    };

    try {
      if (isEditing) {
        await apiClient.put(`/products/${product.id}`, payload);
      } else {
        await apiClient.post('/products', payload);
      }
      onSave();
    } catch (err) {
      console.error(err);
    } finally {
      setSaving(false);
    }
  };

  return (
    <div className="modal-overlay">
      <div className="modal">
        <h3>{isEditing ? 'Edit Product' : 'Add Product'}</h3>
        <form onSubmit={handleSubmit}>
          <input value={name} onChange={(e) => setName(e.target.value)} placeholder="Name" required />
          <textarea value={description} onChange={(e) => setDescription(e.target.value)} placeholder="Description" />
          <input type="number" value={price} onChange={(e) => setPrice(e.target.value)} placeholder="Price" required />
          <select value={category} onChange={(e) => setCategory(e.target.value)}>
            <option value="">Select category</option>
            <option value="electronics">Electronics</option>
            <option value="clothing">Clothing</option>
            <option value="food">Food</option>
          </select>
          <label>
            <input type="checkbox" checked={inStock} onChange={(e) => setInStock(e.target.checked)} />
            In Stock
          </label>
          <button type="submit" disabled={saving}>
            {saving ? 'Saving...' : 'Save'}
          </button>
          <button type="button" onClick={onClose}>Cancel</button>
        </form>
      </div>
    </div>
  );
}

export default ProductForm;
```

## Delete Confirmation

Never delete without asking:

```jsx
function DeleteConfirm({ product, onConfirm, onCancel }) {
  return (
    <div className="modal-overlay">
      <div className="modal">
        <h3>Delete Product</h3>
        <p>Are you sure you want to delete "{product.name}"? This cannot be undone.</p>
        <div className="modal-actions">
          <button onClick={onCancel}>Cancel</button>
          <button onClick={onConfirm} className="btn-danger">Delete</button>
        </div>
      </div>
    </div>
  );
}
```

This CRUD pattern repeats across most features in a real app. Products, orders, users, they all follow the same structure: list, form, and delete confirmation. Once you build one, the rest come naturally.
