# From Lovable

## Get the code

Lovable syncs a project to a GitHub repository it creates (Lovable › GitHub › Connect). Ask the person for
that repository and clone it into its own folder. If the project isn't synced yet, ask them to connect GitHub
in Lovable first. You can't read a Lovable project any other way.

## Tell the generation apart

- **Older projects: React on Vite.** `vite.config.ts`, `index.html` at the root, `src/main.tsx`, usually
  React Router, Tailwind and shadcn/ui (`src/components/ui/`). These build to static files: a **repository
  app**.
- **Newer projects: TanStack Start, rendered on the server.** `app.config.ts` or `vite.config.ts` with
  `tanstackStart`, `src/routes/` with server functions (`createServerFn`), deployed to Cloudflare Workers.
  Either keep it **hosted elsewhere** (reads only), or convert it: switch to TanStack Router's client-only
  SPA and turn each server function into an Action. Converting is real work, so say so in the plan.

## Lovable Cloud / Supabase

Most Lovable apps use Lovable Cloud, which runs on Supabase:

- `src/integrations/supabase/client.ts` is the client, and `src/integrations/supabase/types.ts` holds the
  generated table types. **Read `types.ts` for the inventory of tables and columns.**
- `supabase/migrations/*.sql` holds the tables, enums, row-level security policies and database functions.
  Its RLS policies are the app's roles: map them onto who may see and act.
- `supabase/functions/*` holds edge functions. Each becomes an Action and Workflow, and the folder is
  removed (a repository app with it is refused).
- Calls in the code: `supabase.from("x").select/insert/update/delete`, `supabase.rpc(...)`,
  `supabase.functions.invoke(...)`, `supabase.auth.*`, `supabase.storage.*`, `supabase.channel(...)`
  (realtime).

On conversion, delete `src/integrations/supabase/`, the `@supabase/supabase-js` dependency, the auth pages
and route guards, and the `VITE_SUPABASE_*` variables. Route every data call through one `src/data.ts`
over the SDK. Keep the components, which are usually written against hooks and query results, so put the
SDK behind the same hooks (for example inside the existing `useQuery` functions).

## After the move

- **Copy the code into the new app's repository** on the org's git host, as the main skill says. Lovable keeps
  editing its own repository, and its pushes are not built by workswith.ai.
- If the person wants to keep editing in Lovable and have the result land on workswith.ai, that is attaching
  the Lovable repository through workswith.ai's GitHub App. Attaching checks the repository's `main`, so the
  conversion would have to be on `main` already, which breaks the live Lovable app. Raise it with the person and
  delegate it to the system agent. Don't set it up yourself.
- `lovable.dev` badges, `lovable-tagger` in `vite.config.ts`, and Lovable's analytics scripts can go.
