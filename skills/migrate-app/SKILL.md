---
name: migrate-app
description: Guide moving an existing web app onto workswith.ai - one built on Lovable (or Loveable), deployed on Vercel or Netlify, or vibe-coded locally (React, Vite, Next.js, Supabase, Firebase and the like). Use when the person wants to migrate, move, port, bring over, import or rebuild an app "on workswith", "as a workswith app", or "into Studio", or asks whether their app can run there. Assesses the app, picks the shape it can take, maps its backend onto Types, Actions, Workflows and connections, then converts it to a repository app built with git and the workswith CLI.
---

# Migrate an app to workswith.ai

Read the `workswith-ai` skill first: its rules and flows hold here too. This skill adds one thing they don't
cover. The app's code is built with **git and the workswith CLI**, not over MCP. The MCP server is still how
you read the Ontology and how you delegate platform changes.

The migration runs in five phases, and each ends with something the person sees or decides. Never skip
ahead to rewriting code before phase 3 is agreed.

## Two shapes an app can take

| Shape | What it is | Fits when |
|---|---|---|
| **Repository app** (the default) | A static front end in a git repository. workswith.ai builds every pushed branch, serves the output on the app's own address and hands it the signed-in viewer. The page reaches data only through `@workswith/sdk`. | The app builds to static files, or can be made to: React/Vue/Svelte on Vite, Next.js with static export, an older Lovable app. Its backend can become Types, Actions and Workflows. |
| **Hosted elsewhere** | The app stays where it is (Lovable, Vercel, a server of the person's) and calls workswith.ai with the SDK's sign-in and bearer transport. Its approval covers what it reads and the origins it runs at, not its code. | It renders on the server and must keep doing so (a new Lovable app on TanStack Start, Next.js with server components or API routes), or it needs what the platform doesn't run: its own database, realtime sockets, heavy compute. |

What decides it:

- **Hosted elsewhere is read-only today.** It may read Types, but it can't invoke Actions or answer
  Workflows. An app that must write through workswith.ai has to become a repository app.
- **A repository app runs no server code.** A root folder named `api/`, `functions/`, `server/` or
  `supabase/functions/` is refused on push. The build must leave static files only. The page's CSP allows
  its own origin and pinned CDN scripts and nothing else, so calls to Supabase, Firebase or any other API
  are blocked.
- **A repository app has no sign-in code.** The platform hands the page its viewer, so Supabase Auth,
  Clerk, NextAuth or Firebase Auth are removed, not ported.
- **Inlining the app into a one-file Studio app is not offered.** A one-file app is written in Studio.

## Phase 1: Inventory

**Preflight.** Check that the CLI and SDK are installed: `npx --no-install workswith --help` or `npm view workswith`,
and `npm view @workswith/sdk`. If either is missing, the migration can be planned (phases 1 to 3) but
not shipped. Say so now, before anything is created in the org.

Find the app's code first: the working folder, a path the person gives, or a clone of its GitHub
repository. Then read the source's reference for what to look for:

- Lovable: `references/lovable.md`
- Vercel, Netlify, Next.js: `references/vercel.md`
- Local or anything else: `references/local.md`

Write down, from the code rather than from guesses:

1. **Stack and build**: framework, router, bundler, package manager and lockfile, the build command,
   the output folder, and whether anything renders on the server.
2. **Data**: every table, collection or entity, with its columns. Look in migrations, `schema.sql`,
   generated `types.ts`, Prisma or Drizzle schemas, and the fetch and query calls themselves.
3. **Writes and logic**: every insert, update, delete, RPC, API route, server action, edge function,
   cron job and webhook, and what each one does.
4. **Auth and roles**: who signs in, what roles exist, any row-level security or ownership checks.
5. **Secrets and third parties**: every `.env` variable, key, vendor SDK (Stripe, Resend, OpenAI…) and
   outbound URL.
6. **Files and realtime**: storage buckets and uploads, subscriptions, websockets, polling.
7. **External assets**: fonts, stylesheets, scripts, images and icons loaded from another host (Google
   Fonts, Unsplash, a storage bucket's URLs). A repository app serves images and fonts from its own origin
   only. A script or stylesheet from elsewhere must come from an allowed CDN, pinned to an exact version,
   with `integrity`. The build's output is limited to 25 MiB, 2,000 files and 5 MiB per file.

Scan for committed secrets now (`.env*`, keys in source). Tell the person about any you find. They must
not be pushed to the app's repository, and the person should rotate them.

## Phase 2: Map onto the organisation

Read what the org already keeps: `list_types`, `describe_type`, `list_actions` and `list_workflows`. Then
map each item in the inventory with `references/backend-mapping.md`:

- each entity to an **existing Type** when one fits (say which Fields match which columns), else to a
  **new Type** to propose;
- each write to an **Action**: an existing one, or a new one to propose;
- multi-step, scheduled or long-running logic to a **Workflow** behind an Action;
- each secret and vendor call to a **connection**;
- each role check to **who may see and act**, set on the Types and Actions, not in the page.

List what has **no answer** on the platform. If any of it is essential, the app should be hosted
elsewhere, or the person drops that feature.

## Phase 3: Plan, and the person decides

Present one short plan:

- the shape, and why;
- the mapping table (entity → Type, write → Action, secret → connection), marking each as existing or
  new;
- what is removed (server code, sign-in, the old backend's client) and what has no answer;
- where existing data comes from, if any, and that importing it is a separate step the person
  approves;
- the app key: lower-case letters, digits and single underscores, starting with a letter, at most 40
  characters (`customer_list`).

Ask about each real decision: which Type to reuse, whether to drop a feature, what to name the app. Don't
start phase 4 until the person agrees to the plan.

## Phase 4: Prepare the platform

Delegate the platform changes as one job with `call_system_agent`: the new Types and Fields, the Actions
and their Workflows, the connections, and the app itself. Give it the plan's mapping table, the app key
and its title, and say which Types the app reads. Ask for:

- a **repository app** (in Studio: *New app*, kind *Advanced*), which makes its repository on the org's
  git host with a `workswith.json` and a starter; or
- an app **hosted elsewhere** with its origins (exact `https://` origins, for example
  `https://myapp.lovable.app`) and its redirect URLs.

If the system agent can't make the app, the person makes it in Studio and gives you its key. Ontology
changes wait for approval in workswith.ai, so say what waits and for whom. A branch the Ontology draft
carries may name what the draft adds; its preview runs through the draft, and it is approved with the
draft. Ask the system agent to have the draft carry the app's branch once you have pushed it.

## Phase 5: Convert and ship

**Repository app.** Call `find_skill` for `workswith-apps` and read it with `get_skill`. It's the org's
own guide to the CLI steps (`login`, `clone`, `types`, push a branch, `status`, `send`) and the SDK's
verbs. Follow it, not a remembered version. Then:

1. Clone the app's repository with `npx workswith clone <key>`. Work there, not in the old repository, so
   the app's existing hosting is untouched.
2. Make a branch. Copy the front end in: pages, components, styles, assets. Keep the starter's
   `workswith.json` and adjust `build`, `entry`, `spa`, `reads.types`, `invokes` and `answers` to the plan.
   The app's `package.json` replaces the starter's, but keep `@workswith/sdk` in it. Delete the old
   lockfile, run `npm install`, and commit the new one.
3. Remove what can't run: server folders, server-side rendering, the old backend's client and its
   environment variables, and sign-in screens and guards. The person signed in is `ww.page.viewer`
   (`id`, `key`, `name`, and the `scopes` they belong to), so use it wherever the app read its own
   session. Self-host the external assets: install fonts with npm (`@fontsource/*`) and copy images
   into the repository.
4. Replace each data call with the SDK. Put them behind one small module (`src/data.ts`), so the
   components change little. Reads become `ww.records("<type>").list/read`, and writes become
   `ww.actions("<action>").run`. Handle a `WwError` by showing its `message`.
5. Run `npx workswith types` and type the module against `workswith.d.ts`. Build it locally
   (`npm run build`) before pushing, and fix what fails.
6. Push the branch. `npx workswith status` shows the build and the version's dry preview at `/v/<n>`.
   Share the preview with the person, then iterate on the branch.
7. Run `npx workswith send` only when the person asks. Someone who signs for the app's owner approves it
   in workswith.ai. Approval moves `main`, and the app goes live.

**Hosted elsewhere.** Add `@workswith/sdk`, sign in with its `signIn({org, app, clientId, redirectUri})`
(the client id is the app's id, shown on its page in Studio), and replace reads with the client's verbs.
Writes stay on the app's own backend for now. Once a version is approved, a viewer's first sign-in asks
for their consent.

Don't hand-roll git credentials. If the CLI's login fails, stop and say so.

## Rules

- **Never push `main`.** It is refused, and only approval moves it.
- **Never touch the old app's hosting, repository or data** without the person's say-so: no deploys,
  deletes, branch protection changes or exports.
- **Never put a secret in the app's repository.** A secret belongs in a connection, and the page never
  holds one.
- **Don't promise parity you haven't mapped.** Name every feature that is removed or waits on an
  approval.
- **The org's skills win.** If `find_skill` returns a migration or app skill of the org's own, follow it
  where it differs from this one.
