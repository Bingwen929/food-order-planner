# Food Planner

Food Planner is a web app that plans a group pizza order for an event. You give it
the headcount, where the food should go, when it should arrive, your total budget and
any pizza preferences. It turns that into a plain-language request, runs it through an
n8n workflow that compares pizza options on delivery apps, and shows the options back
on a results page where you can keep asking follow-up questions.

## How it works

1. The landing page collects the event details and validates them with a zod schema
   (`src/lib/event-plan.ts`).
2. On submit, the app navigates to `/results` with the plan in the URL search params.
3. The results page builds a single natural-language message from the plan, for example:
   "I'm planning pizza for an event with 25 people. Delivery address: ... Total budget: $300."
4. A TanStack Start server function (`src/lib/planner.functions.ts`) posts that message to the
   published n8n chat workflow through the Lovable connector gateway, keeping one session id
   per visit.
5. The workflow's markdown reply is parsed (`src/lib/planner-response.ts`) and rendered with
   its bold section headings separated out. Follow-up messages reuse the same session.

## Form fields

| Field | Required | Notes |
| --- | --- | --- |
| Guests | Yes | Whole number, 1 to 2000 |
| Pizza preferences | No | Free text, up to 200 characters |
| Date and time | Yes | Today or later, food arrives by this time |
| Delivery address | Yes | Address or ZIP code |
| Total budget | Yes | More than $0, shown as a per-person figure while typing |

## Routes

- `/` landing page with the event form
- `/results` planner reply, with a follow-up chat box that stays in the same n8n session

## Stack

- TanStack Start with TanStack Router and TanStack Query, React 19, Vite
- Tailwind CSS v4 with shadcn/ui components (Radix primitives)
- zod for form and response validation
- Vitest and Testing Library for tests
- n8n workflow `AI Food Order Optimizer` (exported in `AI Food Order Optimizer.json`)

## Getting started

Install dependencies. The lockfile is a bun lockfile, so bun is the intended package manager,
and npm or pnpm work too.

```sh
bun install
bun run dev
```

Vite prints the local URL when the dev server starts.

### Environment variables

The planner only works when both keys are set:

```sh
LOVABLE_API_KEY=...   # Lovable connector gateway token
N8N_API_KEY=...       # n8n connection API key
```

Without them the results page returns "The planner isn't connected right now."

The workflow itself must be published in n8n. If it is switched off, the app reports that the
workflow needs to be turned on.

## Scripts

| Command | Purpose |
| --- | --- |
| `bun run dev` | Start the dev server |
| `bun run build` | Production build |
| `bun run preview` | Preview the build |
| `bun run test` | Run the Vitest suite once |
| `bun run test:watch` | Run tests in watch mode |
| `bun run lint` | ESLint |
| `bun run format` | Prettier |

## Project layout

```
src/
  routes/index.tsx            event form and landing page
  routes/results.tsx          planner reply and follow-up chat
  components/planner/         form fields, stepper, searching state, reply renderer
  components/ui/              shadcn/ui component set
  lib/event-plan.ts           zod schema and request message builder
  lib/planner.functions.ts    server function that calls the n8n webhook
  lib/planner-response.ts     response parsing, including demo-mode detection
  test/                       routing, schema and response parsing tests
AI Food Order Optimizer.json  exported n8n workflow
roadmap.md                    work completed so far
.lovable/                     Lovable project metadata and product north star
```

## Notes

- Recommendations come from live web search inside the workflow, so prices and availability are
  estimates, not confirmed quotes.
- Replies that start with a `DEMO MODE:` marker are flagged as fictional in the UI and the marker
  is stripped from the rendered text.
- The app was scaffolded and is maintained with [Lovable](https://lovable.dev). Editing the
  Lovable project keeps this repository in sync.
