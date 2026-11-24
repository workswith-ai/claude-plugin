# From a local or vibe-coded app

The app is a folder on the person's machine, often with no deploy, no tests and a backend mixed into the
front end. Work out its shape before anything else.

## Find what it is

- `package.json`: the framework, the scripts (`dev`, `build`), and the dependencies that reveal the
  backend: `express`, `fastify`, `hono`, `@supabase/supabase-js`, `firebase`, `prisma`, `drizzle-orm`,
  `better-sqlite3`, `mongoose`, `openai`, `stripe`.
- **A single `index.html`** with inline scripts and no build: it fits a one-file Studio app better than a
  repository. Tell the person, who can recreate it in Studio. Or give it a minimal Vite setup to make it a
  repository app.
- **A server in the same repository** (`server.js`, `server/`, `backend/`, an Express app serving
  `dist/`): split it. The front end is the app, and each route the server answers becomes an Action, or a
  read through the SDK.
- **Not JavaScript** (a Flask, Django, Rails or Streamlit app rendering HTML): it can't be a repository
  app as is. Either rebuild the UI as a small front end over the SDK (say how many screens), or keep it
  where it runs and treat it as hosted elsewhere.
- **Local data**: SQLite files, JSON files, `localStorage` used as a database, CSVs. These are the
  inventory of entities, and also the existing data the person may want imported.

## Hygiene before the first push

- No lockfile? Run `npm install` to write `package-lock.json`. A build needs one.
- Make sure `npm run build` works locally and writes its files to one output folder.
- Check `.gitignore` covers `node_modules`, the build output and `.env*`.
- Hard-coded keys in source are common in vibe-coded apps. Find them, move what is needed into a connection,
  and tell the person to rotate them.
- Absolute URLs to `localhost:3000/api/...` mark the calls to replace with the SDK.
