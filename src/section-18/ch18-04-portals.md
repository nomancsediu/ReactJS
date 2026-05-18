# Portals

Sometimes you need to render a component outside the normal DOM hierarchy. Modals are the classic example. If a modal is nested deep inside the component tree, it might get clipped by a parent with `overflow: hidden` or appear below other elements because of z-index stacking. Portals solve this.

## createPortal

`createPortal` lets you render children into a DOM node that exists outside the parent component's DOM hierarchy:

```tsx
import { createPortal } from "react-dom";

function Modal({ children, isOpen }: { children: React.ReactNode; isOpen: boolean }) {
  if (!isOpen) return null;

  return createPortal(
    <div className="fixed inset-0 bg-black/50 flex items-center justify-center">
      <div className="bg-white p-6 rounded-lg max-w-md w-full">
        {children}
      </div>
    </div>,
    document.body
  );
}
```

The modal renders into `document.body`, not into the parent component's position in the DOM tree. This means it is never affected by parent styles like `overflow: hidden` or `z-index`.

## Setting Up the Portal Target

You need a DOM element for the portal to render into. The simplest option is `document.body`. But you can also create a dedicated element:

```html
<!-- index.html -->
<body>
  <div id="root"></div>
  <div id="modal-root"></div>
</body>
```

```tsx
function Modal({ children, isOpen }: { children: React.ReactNode; isOpen: boolean }) {
  if (!isOpen) return null;

  const modalRoot = document.getElementById("modal-root");
  if (!modalRoot) return null;

  return createPortal(
    <div className="fixed inset-0 bg-black/50 flex items-center justify-center">
      <div className="bg-white p-6 rounded-lg">{children}</div>
    </div>,
    modalRoot
  );
}
```

## A Complete Modal Pattern

Here is a modal I use in most projects:

```tsx
"use client";

import { createPortal } from "react-dom";
import { useEffect, useState } from "react";

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
}

function Modal({ isOpen, onClose, title, children }: ModalProps) {
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);
  }, []);

  useEffect(() => {
    if (!isOpen) return;

    const handleEsc = (e: KeyboardEvent) => {
      if (e.key === "Escape") onClose();
    };
    document.addEventListener("keydown", handleEsc);
    document.body.style.overflow = "hidden";

    return () => {
      document.removeEventListener("keydown", handleEsc);
      document.body.style.overflow = "";
    };
  }, [isOpen, onClose]);

  if (!mounted || !isOpen) return null;

  return createPortal(
    <div
      className="fixed inset-0 z-50 flex items-center justify-center"
      onClick={onClose}
    >
      <div className="fixed inset-0 bg-black/50" />
      <div
        className="relative bg-white rounded-lg p-6 max-w-lg w-full mx-4"
        onClick={(e) => e.stopPropagation()}
      >
        <div className="flex justify-between items-center mb-4">
          <h2 className="text-xl font-bold">{title}</h2>
          <button onClick={onClose} className="text-gray-500 hover:text-gray-700">
            &times;
          </button>
        </div>
        {children}
      </div>
    </div>,
    document.body
  );
}

// Usage
function App() {
  const [showModal, setShowModal] = useState(false);

  return (
    <div>
      <button onClick={() => setShowModal(true)}>Open Modal</button>
      <Modal isOpen={showModal} onClose={() => setShowModal(false)} title="Confirm">
        <p>Are you sure you want to do this?</p>
        <button onClick={() => setShowModal(false)}>Confirm</button>
      </Modal>
    </div>
  );
}
```

Key details in this pattern:

- Click the backdrop to close
- Press Escape to close
- Lock body scroll when open
- Stop click propagation so clicking inside does not close
- Handle SSR with the `mounted` check

Portals are not just for modals. I have also used them for tooltips, dropdown menus, and toast notifications. Any time a component needs to escape its parent's DOM constraints, portals are the answer.
