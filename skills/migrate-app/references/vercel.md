# From Vercel, Netlify or Next.js

## Get the code

The code is in the person's Git repository, the one Vercel or Netlify deploys from. Clone it into its own
folder. Don't change the deploy settings or the repository's `main`: the live app depends on them.

## What renders where

- **A static SPA** (Vite, Create React App, Vue, Svelte, Astro with `output: "static"`) is a **repository
  app** once its backend is mapped. Check `vercel.json` / `netlify.toml` for rewrites (an SPA fallback maps
  to `"spa": true`) and redirects. Note any `headers`, since those are not carried over.
- **Next.js**: read `next.config.*` and the routes.
  - `output: "export"` is static, with build output `out`. A Pages Router app without `getServerSideProps`
    and `pages/api/` can be made static by setting it.
  - App Router with server components that fetch data, `"use server"` actions, `route.ts` handlers,
    `middleware.ts`, or `getServerSideProps` render on the server. Either convert (client components that
    fetch through the SDK, `output: "export"`, each server action and route handler an Action), or keep it
    **hosted elsewhere** with reads only.
  - `next/image` with the default loader doesn't work in a static export. Set `images.unoptimized`.
- **Serverless functions**: `api/` at the root (Vercel) or `netlify/functions/`. Each becomes an Action and
  Workflow, and the folder is removed.
- **Cron**: `crons` in `vercel.json`, Netlify scheduled functions. Each becomes a scheduled Workflow.

## The backend

Look for the data layer the functions use: Vercel Postgres, Neon, PlanetScale, Supabase, Firebase, MongoDB,
Prisma (`prisma/schema.prisma`) or Drizzle (`drizzle/`, `schema.ts`). The schema file is the inventory of
tables. Auth is usually NextAuth/Auth.js, Clerk, Supabase Auth or Firebase Auth. All of it is removed on a
repository app.

## Environment

List every variable in the project's `.env.example`, `.env.local` (don't print its values) and the code
(`process.env.*`, `import.meta.env.*`). Sort them into: the old backend's (dropped), a vendor's secret
(a connection), and a public setting (a constant in the code, since a build gets no environment of the
person's).

## Build

The platform builds with `npm run build` (or the command in `workswith.json`) on Node 20, 22 or 24. During
the build, the network reaches the npm registry and nothing else. A build that downloads anything else
(fonts, a schema, private packages) fails, so vendor those files into the repository.
