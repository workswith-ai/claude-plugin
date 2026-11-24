# Mapping a backend onto workswith.ai

An app on workswith.ai has no server code of its own. Each backend need maps onto something the
organisation keeps, approves and audits.

| The app has | On workswith.ai | Notes |
|---|---|---|
| A table or collection | A **Type**, its columns as **Fields** | Reuse an existing Type when its Fields cover the columns. Map enums to option Fields, and status columns to a state Field with transitions. |
| A foreign key | A **reference Field** to the other Type | The page reads a referenced Record only if `reads.types` names that Type too. |
| Insert, update or delete | An **Action** on the Type | Actions are the only door to change data. The page calls `ww.actions("<key>").run({record, input, key})`, and the manifest lists the key in `invokes`. |
| Validation or business rules on a write | The Action's **Workflow**, with script or expression Steps | Not in the page, which anyone can change in their browser. |
| An RPC, API route, server action or edge function | An **Action** and its **Workflow** | One per distinct operation. |
| A cron job or scheduled function | A **Workflow** on a schedule trigger | |
| A long process with human steps (approve, review) | A **Workflow** with waits | The page lists waits with `ww.waits.list()` and answers with `ww.waits.answer`; the manifest lists the Workflow in `answers`. |
| A vendor API key or SDK (Stripe, Resend, OpenAI…) | A **connection**, with its calls through Actions | The secret is sealed in the connection, and the page never sees it. |
| An incoming webhook | A connection's inbound webhook and a trigger | Ask the system agent whether one exists for that vendor. |
| An LLM call | An `agent` Step in a Workflow, or a kept agent | |
| Sign-in, sessions, user tables | **Nothing in the app** | The platform signs the viewer in and hands them to the page as `ww.page.viewer` (`id`, `key`, `name`, `scopes`). Drop the auth code. A `profiles` table often maps to the org's people rather than a Type. |
| Roles, row-level security, ownership checks | **Who may see and act**, on the Types and Actions | Each SDK call runs as the viewer and is checked against what they may see and do. The page needs no checks of its own, and hiding a button is cosmetic. |
| Sorting on the server (`order by`) | Sort in the page | Record listings take a filter, a limit and a cursor, but no order. Read every page with the cursor when the full set is needed. |
| File storage, uploads | **Ask the system agent** | There is no general file store for apps. A file Field or a connection to the existing storage may fit. |
| Realtime subscriptions | **No answer yet** | Poll with `list`, or keep the app hosted elsewhere. |
| Its own database it must keep | **No answer**: hosted elsewhere | |
| Heavy compute (image processing, large exports) | **No answer**: hosted elsewhere | |
| Analytics, error tracking scripts | Usually removed | The CSP blocks outside endpoints. App errors are logged by the platform for the app's editors. |

## Existing data

Moving rows into Records is a separate step from moving the app. Put it to the person as its own decision:
which tables, how many rows, and whether old ids must be kept. Hand the job to the system agent (a connection
to the old database, or a one-time import) rather than looping `create_record` over thousands of rows.
