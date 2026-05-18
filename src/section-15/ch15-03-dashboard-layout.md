# Dashboard Layout

A dashboard needs a consistent layout: a sidebar for navigation, a header with user info, and a main content area. Making it responsive is the tricky part. On desktop, the sidebar stays visible. On mobile, it slides in from the side.

## Layout Component

The layout wraps all dashboard pages:

```jsx
import { useState } from 'react';
import { Outlet } from 'react-router-dom';
import Sidebar from './Sidebar';
import Header from './Header';

function DashboardLayout() {
  const [sidebarOpen, setSidebarOpen] = useState(false);

  return (
    <div className="dashboard-layout">
      <Sidebar
        isOpen={sidebarOpen}
        onClose={() => setSidebarOpen(false)}
      />
      <div className="dashboard-main">
        <Header onMenuClick={() => setSidebarOpen(true)} />
        <main className="dashboard-content">
          <Outlet />
        </main>
      </div>
    </div>
  );
}

export default DashboardLayout;
```

The `Outlet` component from React Router renders the matched child route inside the layout.

## Sidebar Component

```jsx
import { NavLink } from 'react-router-dom';
import { useAuth } from '../../context/AuthContext';

const navItems = [
  { path: '/dashboard', label: 'Dashboard', icon: 'LayoutDashboard' },
  { path: '/dashboard/products', label: 'Products', icon: 'Package' },
  { path: '/dashboard/orders', label: 'Orders', icon: 'ShoppingCart' },
  { path: '/dashboard/settings', label: 'Settings', icon: 'Settings' },
];

function Sidebar({ isOpen, onClose }) {
  const { user } = useAuth();

  return (
    <>
      {/* Mobile overlay */}
      {isOpen && (
        <div
          className="sidebar-overlay"
          onClick={onClose}
        />
      )}

      <aside className={`sidebar ${isOpen ? 'sidebar--open' : ''}`}>
        <div className="sidebar-header">
          <h2>MyApp</h2>
          <button className="sidebar-close" onClick={onClose}>
            Close
          </button>
        </div>

        <nav className="sidebar-nav">
          {navItems.map((item) => (
            <NavLink
              key={item.path}
              to={item.path}
              end={item.path === '/dashboard'}
              className={({ isActive }) =>
                `sidebar-link ${isActive ? 'sidebar-link--active' : ''}`
              }
              onClick={onClose}
            >
              {item.label}
            </NavLink>
          ))}
        </nav>

        <div className="sidebar-footer">
          <span>{user?.name}</span>
        </div>
      </aside>
    </>
  );
}

export default Sidebar;
```

## Header Component

```jsx
import { useAuth } from '../../context/AuthContext';
import { useNavigate } from 'react-router-dom';

function Header({ onMenuClick }) {
  const { user, logout } = useAuth();
  const navigate = useNavigate();

  const handleLogout = async () => {
    await logout();
    navigate('/login');
  };

  return (
    <header className="dashboard-header">
      <button
        className="menu-toggle"
        onClick={onMenuClick}
      >
        Menu
      </button>

      <h1 className="header-title">Product Dashboard</h1>

      <div className="header-actions">
        <span className="user-name">{user?.name}</span>
        <button onClick={handleLogout} className="logout-btn">
          Log Out
        </button>
      </div>
    </header>
  );
}

export default Header;
```

## Responsive CSS

The layout needs to work on both desktop and mobile:

```css
.dashboard-layout {
  display: flex;
  min-height: 100vh;
}

.sidebar {
  width: 250px;
  background: #1f2937;
  color: white;
  display: flex;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  z-index: 40;
  transition: transform 0.3s ease;
}

/* Mobile: sidebar hidden by default */
@media (max-width: 768px) {
  .sidebar {
    transform: translateX(-100%);
  }

  .sidebar--open {
    transform: translateX(0);
  }

  .menu-toggle {
    display: block;
  }
}

/* Desktop: sidebar always visible */
@media (min-width: 769px) {
  .sidebar-overlay {
    display: none;
  }

  .menu-toggle {
    display: none;
  }

  .dashboard-main {
    margin-left: 250px;
  }
}
```

This layout gives us a solid foundation. Every dashboard page renders inside the `Outlet`, and the sidebar and header stay consistent. On mobile, users tap the menu button to open the sidebar as an overlay.
