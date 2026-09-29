# Product Manager – React, React Router Data APIs, Valibot and Tailwind CSS

A product manager built with **React 19** and **TypeScript**. You list, create, edit and delete products, and toggle whether each one is available, all against a REST API. Data loading and form submissions go through **React Router's data APIs**, requests are sent with **Axios**, and every response is validated with **Valibot**.

This is **Project 10** of the Udemy course [React de Principiante a Experto](https://www.udemy.com/course/react-de-principiante-a-experto-creando-mas-de-10-aplicaciones/). It is the frontend of a full-stack app whose Node.js API lives in [its own repository](https://github.com/TarekM-7/product-manager-server). The goal of this project is to connect a React client to a real backend, move data loading and mutations into the router, validate data at runtime, and deploy both halves.

![The Productos page under a dark "Administrador de Productos" header: a table of four products with their name and price, an availability button marked Disponible, or No Disponible in red for the keyboard, and blue Editar and red Eliminar buttons on each row, with an "Agregar Producto" button above the table](docs/screenshot.png)

**Live demo:** [the app on Vercel](https://fullstack-project-node-react-typesc.vercel.app/) · [the API docs on Render](https://fullstack-project-node-react-typescript.onrender.com/docs/)

> **Hosting:** the React client runs on [Vercel](https://vercel.com/), and the API with its PostgreSQL database on [Render](https://render.com/), all on free plans. A free service can go to sleep when nobody is using it, so the product list may take a while to show up the first time, and the live demo may stop working at some point.

## Features

- A table with every product, its price in dollars and its availability.
- Create a product with a name and a price. Both fields are required.
- Edit a product in a form filled in with its current data, including its availability.
- Toggle a product's availability with one click, without leaving the page.
- Delete a product after confirming it.
- Every response from the API is validated before it is shown.
- Any route can be reloaded or opened directly, even on Vercel.

## Tech Stack

- [React 19](https://react.dev/)
- [TypeScript](https://www.typescriptlang.org/)
- [Vite](https://vite.dev/)
- [React Router 7](https://reactrouter.com/) as a data router: loaders, actions and fetchers
- [Tailwind CSS 4](https://tailwindcss.com/)
- [Axios](https://axios-http.com/) for HTTP requests
- [Valibot](https://valibot.dev/) to validate the data at runtime
- [ESLint](https://eslint.org/) with `typescript-eslint`
- [Vercel](https://vercel.com/) for hosting

## How It Works

Each route declares a **loader**, which reads data before the page renders, and an **action**, which runs when one of its forms is submitted:

| Route | Loader | Action |
|---|---|---|
| `/` | `getProducts()` | `updateProductAvailability()`, from a `fetcher.Form` |
| `/products/new` | – | `addProduct()` |
| `/products/:id/edit` | `getProductById()` | `updateProduct()` |
| `/products/:id/delete` | – | `deleteProduct()` |

All the routes render inside a shared `Layout` through `<Outlet />`. The delete route has no page of its own: it only exists to run its action.

A form submission never talks to the API directly. It goes through the action, then a service:

```
<Form method="POST"> ──submit──▶ route action                          (in the browser)
                                   ├─ empty field? → return the error → useActionData → <ErrorMessage>
                                   └─ ProductService
                                        ├─ convert the text: +price, toBoolean(availability)
                                        ├─ Valibot checks the shape and the types
                                        └─ Axios → REST API on Render    (over the network)
                                   ▼
                               redirect('/') → the loader runs again → fresh product list
```

The availability button uses a `fetcher.Form` instead of a `<Form>`: it runs the action without navigating away, and React Router reloads the list by itself.

## Project Structure

```
src/
├── components/
│   ├── ErrorMessage.tsx      # Red banner for validation errors
│   ├── ProductDetails.tsx    # Table row, availability toggle and the delete action
│   └── ProductForm.tsx       # Name and price fields, shared by the new and edit pages
├── layouts/
│   └── Layout.tsx            # Header and the <Outlet /> for every page
├── services/
│   └── ProductService.ts     # Axios calls to the API and Valibot validation
├── types/
│   └── index.ts              # Valibot schemas and the Product type derived from them
├── utils/
│   └── index.ts              # Currency formatting and text-to-boolean conversion
├── views/
│   ├── EditProduct.tsx       # Edit page with its loader and action
│   ├── NewProduct.tsx        # Create page with its action
│   └── Products.tsx          # Product list with its loader and action
├── index.css                 # Tailwind CSS import
├── main.tsx                  # Mounts the RouterProvider
└── router.tsx                # Routes with their loaders and actions
vercel.json                   # Sends every route to index.html on Vercel
```

## What I Learned

- **Data routers.** A loader fetches what the page needs before it renders, and `useLoaderData` reads it. An action handles a form submission, and `useActionData` reads what it returned. The components only render.
- **`<Form>` versus `fetcher.Form`.** `<Form>` navigates, like the delete button that redirects home. `fetcher.Form` submits in the background and keeps you on the page, which suits a toggle inside a table.
- **The form's method is not the API's method.** `<Form method="POST">` only tells React Router to run the action; a `GET` would run the loader instead. The HTTP verb that reaches the API is picked by the service, with `axios.put`, `axios.patch` or `axios.delete`.
- **Everything from the browser arrives as text.** Form fields and URL params are always strings, even from an `<input type="number">`. They have to be converted, with `+value` or `toBoolean`, before they are validated.
- **Validating at runtime with Valibot.** TypeScript types disappear at runtime, so a schema checks the real data, and `InferOutput` derives the `Product` type from that same schema. `safeParse` returns the result, while `parse` throws. The course uses Valibot 0.x, so I had to translate its code to version 1, for example `coerce` into `pipe(string(), transform(Number), number())`.
- **Axios versus `fetch`.** Axios converts to and from JSON by itself, and it throws on `4xx` and `5xx` responses, which `fetch` does not.
- **Client and server validation.** Checking the form in the browser helps the user, but only the API's validation protects the data.
- **Environment variables at build time.** Vite only exposes variables with the `VITE_` prefix, and writes their values into the JavaScript when it builds. So `VITE_API_URL` has to be set on Vercel before the build, not after.
- **Deploying a single-page app.** On Vercel, reloading `/products/5/edit` would give a `404`, because that file does not exist. `vercel.json` sends every route without a dot to `index.html`, so React Router can handle it, while real files like `/assets/index.js` are still served as they are.
- **CORS from the client's side.** The API only answers websites whose origin matches its `FRONTEND_URL` exactly, so a trailing slash in either URL is enough to break the app.

## Getting Started

Requirements: [Node.js](https://nodejs.org/) 20 or later and the [API](https://github.com/TarekM-7/product-manager-server) running, locally or deployed.

```bash
# Clone the repository
git clone https://github.com/TarekM-7/product-manager-client.git
cd product-manager-client

# Install dependencies
npm install

# Point the client at the API
# create a .env.local file with:
# VITE_API_URL=http://localhost:4000

# Start the dev server
npm run dev
```

The API has to allow this client: set its `FRONTEND_URL` to `http://localhost:5173`, without a trailing slash.

Other scripts:

```bash
npm run build     # Type-check and build for production
npm run preview   # Preview the production build
npm run lint      # Run ESLint
```

> Note: every `VITE_` variable ends up in the browser bundle, where anyone can read it. Here it is only the public URL of the API, so nothing secret is exposed.

## Acknowledgements

Project idea and design come from the Udemy course linked above. I wrote the implementation while following the course and adapted it as I learned.
