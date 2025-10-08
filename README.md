# shopping-cart-ui

React + Vite shopping cart UI using Tailwind. Products are provided via React Context (ProductContext). Cart state is managed in CartContext. Designed to work with either a backend API (`/api/products` proxied to `http://localhost:8000`) or local/static product data and images in `public/images`.

## Quick facts

- Dev server: http://localhost:5173 (Vite)
- Optional backend API: http://localhost:8000 (proxied by Vite `/api` -> `http://localhost:8000`)
- Images (local): place files in `public/images/` and reference as `/images/<name>.jpg`

## Prerequisites

- Node.js 18+ and npm
- Windows: PowerShell or CMD

## Install & run (Windows)

```powershell
cd c:\Users\HP\Desktop\SANBOXES\shopping-cart-ui
npm install
npm run dev
# open http://localhost:5173
```

## Build & preview

```powershell
npm run build
npm run preview
```

## Project structure (important)

- src/main.jsx — mounts App and wraps with ProductProvider and CartProvider
- src/App.jsx — top-level layout (uses ProductContext via hook or useContext)
- src/context/ProductContext.jsx — provides { products, loading, error } (fetch or static)
- src/context/CartContext.jsx — provides cart actions (add, remove, clear) and cart state
- src/components/
  - Header.jsx — cart dropdown / UI (ensure hooks and returns present)
  - ProductList.jsx — maps products -> ProductCard
  - ProductCard.jsx — product UI + add-to-cart action
- public/images/ — put local product images here
- vite.config.js — proxy `/api` to `http://localhost:8000`

## Using local product data (no backend)

If you do not run a backend, modify ProductContext to return static data. Example:

```js
// filepath: c:\Users\HP\Desktop\SANBOXES\shopping-cart-ui\src\context\ProductContext.jsx
...existing code...
const localProducts = [
  { id: 1, name: "Product 1", price: 9.99, image: "/images/product1.jpg", description: "..." },
  { id: 2, name: "Product 2", price: 19.99, image: "/images/product2.jpg", description: "..." },
];

export function ProductProvider({ children }) {
  const products = localProducts;
  const loading = false;
  const error = null;
  return (
    <ProductContext.Provider value={{ products, loading, error }}>
      {children}
    </ProductContext.Provider>
  );
}
...existing code...
```

- Put images in `public/images/` and reference `/images/<file>`.

## Using a backend API

- Vite proxies `/api/*` to `http://localhost:8000` (see `vite.config.js`).
- Start your backend on port 8000 and expose `/products` returning JSON array of products.
- Test directly: `http://localhost:8000/products` (or use curl/Postman).

## Common problems & fixes

- Error: `Uncaught ReferenceError: loading is not defined`

  - Cause: component uses `loading` but ProductProvider not wrapping App or you accessed `loading` before using context.
  - Fix: Ensure `main.jsx` wraps `<App/>` with `<ProductProvider>` and use the context hook: `const { products, loading, error } = useProducts();`

- Error: `GET /api/products 500` or `ECONNREFUSED 127.0.0.1:8000`

  - Cause: Backend not running or backend route errored.
  - Fix: Start backend on port 8000. Check backend terminal/logs. If you don't have a backend, switch ProductContext to local data (see above).

- Product list empty / not rendering

  - Ensure `products` is an array and `ProductList` calls `products.map(...)` and returns JSX. Example:
    ```jsx
    {
      products.map((p) => <ProductCard key={p.id} product={p} />);
    }
    ```
  - Confirm ProductCard expects `product` prop and uses `product.image`, `product.name`, etc.

- Item not showing in cart after adding

  - Verify CartContext `addToCart` updates cart state and the Header uses the same CartContext.
  - Ensure Header returns JSX (not just defining variables) and includes:
    - `import { useState } from 'react'`
    - `import { useCart } from '../context/CartContext'` (or correct hook)
    - `return ( ...cart dropdown markup... )`
  - Check items added to cart include `id`, `name`, `price`, and `qty` fields.

- Images not loading
  - Place images in `public/images/`. Reference as `/images/<filename>`. Example in product data: `image: "/images/product1.jpg"`.

## Debugging tips

- Frontend network: open DevTools > Network to inspect `/api/products` requests and responses.
- Backend logs: check the terminal where the backend runs for stack traces.
- Frontend console: check for React errors (missing imports, undefined variables).
- Quick backend test: `curl http://localhost:8000/products` or open in browser.

## Useful commands (Windows)

- Start Vite dev server: `npm run dev`
- Start backend (example): `cd backend && npm start`
- Check which process uses port 8000: `netstat -ano | findstr :8000`

## Notes

- Keep context providers in `main.jsx` order consistent (ProductProvider, CartProvider).
- Keep component props consistent: product objects should contain id, name, price, image, description.
- If switching between backend and local data, update ProductContext accordingly.

## License

MIT
