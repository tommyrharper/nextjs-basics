# What I did

```
yarn config set ignore-engines true
yarn config set ignore-engines false
```

## Server rendering

2 Options:

1. SSG - Static Site Generation
   - Pre rendering.
   - Generate all the HTML at build time.
   - Simple.
   - Generate a bunch of HTML files.
   - Upload them to a storage bucket or static host.
   - Can be delivered at high performance over a CDN.
     - **Tradeoffs:**
       - Data may become stale.
         - If the data on the server changes, you need to rebuild and re-deploy your site in order for those changes to take effect.
       - Hard to scale to many pages.
         - Slow and difficult to pre-render a million pages.
     - Therefore it is best for sites with data that doesn't change often, and sites that have a low number of pages.
     - E.g. a blog.

2. SSR - Server Side Rendering
  - Content is generated on a server when requested by the user.
  - Ideal when data changes constantly.
  - It ensures whatever user will always get the latest and greatest data.
  - E.g. Ebay - millions of listings that are changing all the time.
    - **Drawbacks:**
      - Slower
      - Inefficient data caching.

3. CSR - Client Side Rendering

4. ISR - Incremental Static Regeneration

# FreeCodeCamp Talk

## No pre-rendering is a plain React.js application.
- Initial load:
  - The App is not rendered and you see a white screen.

THEN as JS loads

- Hydration:
  - React components are initialized and the App becomes interactive.

## Pre-rendering (Using Next.js)

- Initial Load:
  - Pre-rendered HTML is displayed.

THEN as JS loads

- Hydration:
  - React components are initialized and the App becomes interactive.

## Static Generation

- HTML is generated at **build-time** and is reused for each request.

- `next build` builds the app for production.
- The HTML is generated.
- Reused for each request.

### Without Data

- For pages that can be generated without fetching external data at build time.

- `next build`
- Builds the app for production.
- The HTML is generated - no need to fetch external data

### With Data

- For pages that can only be generated after fetching external data at build time.

- `next build`
- Builds the app for production
- Fetches external data
- The HTML is generated after the promise is resolved and the data has been fetched.

```JSX
export default function Home({ products }) {
  return (
    <ul>
      {products.map(product => (
        <li>{product.title}</li>
      ))}
    </ul>
  )
}

export async function getStaticProps(context) {
  const res = await fetch(`https://.../products`)
  const data = await res.json();

  return {
    props: {
      products: data.products
    }
    // This extra prop switches us from SSG with data to ISR
    // Meaning Incremental Static Regeneration
    // Sets a revalidation period based on the cache-control http request header
    revalidate: 60 // seconds
    // First request: Within 60 sec period -> Reuse the same HTML
    // Second request: After 60 secs -> tells application you have stale data now
    // Third request: Subsequent request -> goes and updates the cache with the new data
  }
}
```

## Page Path Depends on External Data

- You can pre-render pages with paths that depend on external data.

- `next build`
- Builds the app for production.
- Fetches external data
- Generates the pages that have the format: `/posts/[id]`

We can do this using the `getStaticPaths` function.

```JavaScript
// pages/posts/[id].js

export async function getStaticPaths() {
  const res = await fetch('https://.../posts');
  const posts = await res.json();

  const paths = posts.map(post => ({
    params: { id: post },
  }));

  return {
    paths,
    // We don't want a fallback
    // This means if one of those ids doesn't exist, we will get a 404
    fallback: false // true, false or blocking
  }
}
```

## Server Side Rendering

- On each request, the data is fetched and the HTML is generated.

- Page request -> Next.js ->
- Fetches external data.
- The HTML is generated.

```JSX
export default function Home({ products }) {
  return (
    <ul>
      {products.map(product => (
        <li>{product.title}</li>
      ))}
    </ul>
  )
}

export async function getServerSideProps(context) {
  const res = await fetch(`https://.../products`)
  const data = await res.json();

  return {
    props: {
      products: data.products
    }
  }
}
```

## Static Generation without Data + Fetch Data on the Client-Side

- You can also pre-render without data and then load the data on the client-side.

- `next build`
- Builds the app for production
- Statically generate parts of the page that do not require external data
- When JS loads on the client side (at request time), fetch external data
- Populate the remaining parts using external data.

```JSX
// SWR is a react hook library for data fetching by vercel
import useSWR from 'swr' // swr.vercel.app

function Profile() {
  const { data, error } = useSWR('/api/user', fetcher)

  if (error) return <div>failed to load</div>
  if (!data) return <div>loading...</div>
  return <div>hello {data.name}!</div>
}
```

## Next.js

- Next.js is a dynamic framework.
- Can do each page differently.