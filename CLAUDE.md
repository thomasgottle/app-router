# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A hands-on walkthrough of the official Next.js [App Router course](https://nextjs.org/learn/dashboard-app) — the "Acme" invoices dashboard. It is a **teaching starter, not a finished app**: the course adds routes, data flows, and features chapter by chapter. Deliberately unfinished code is expected here; do not "fix" it into completeness unless asked.

Current state (Chapter 2-ish): only `/` , `/seed`, and `/query` routes exist. `app/lib/data.ts` and every component under `app/ui/` are pre-written by the course but **not yet wired up**.

## Commands

Package manager is **pnpm** (`pnpm-lock.yaml`, `pnpm-workspace.yaml`).

```bash
pnpm install         # bcrypt/sharp need native builds — approved via onlyBuiltDependencies
pnpm dev             # next dev --turbopack
pnpm build
pnpm start
npx tsc --noEmit     # typecheck (tsconfig is noEmit; there is no `pnpm lint` script)
```

There are no tests and no linter configured. Next 16 has no `next lint`; don't invent those scripts.

**Typechecks and builds currently fail by design.** Seven `app/ui/` files import `@/app/ui/fonts`, which the course creates in Chapter 3. Components also link to `/dashboard/*` routes that don't exist yet. Treat those as expected until the corresponding chapter; only report *new* errors.

## Database setup

`app/lib/data.ts` and `app/seed/route.ts` each open their own connection: `postgres(process.env.POSTGRES_URL!, { ssl: 'require' })`. There is no committed `.env` (gitignored) — a `POSTGRES_URL` must be set locally or the app throws at request time.

- Seed by hitting `GET /seed` in the browser once. It creates `users`, `customers`, `invoices`, `revenue` and inserts `app/lib/placeholder-data.ts` with `ON CONFLICT DO NOTHING`, so it's idempotent.
- `app/query/route.ts` is a scratch endpoint for a course exercise; it's meant to be uncommented and later deleted.

## Architecture

Everything lives under `app/` with the `@/*` path alias mapping to the repo root (`@/app/lib/...`).

- **`app/lib/definitions.ts`** — hand-written row types (the course's stand-in for ORM-generated types). Query helpers pass them as the `postgres` generic: ``sql<InvoicesTable[]>`...` ``.
- **`app/lib/data.ts`** — every SQL query, one exported async function per view. Server-side only; called directly from Server Components. Interpolated values go through tagged-template parameters, so `${...}` inside a `sql` template is safe — never build query strings by concatenation. Each function catches, logs `Database Error:`, and rethrows a user-facing `Error`.
- **Money is stored in cents** as `INT`. `formatCurrency` in `app/lib/utils.ts` divides by 100 at the display boundary; `fetchInvoiceById` divides by 100 so edit forms show dollars. Anything writing back to the DB must multiply by 100.
- **`app/ui/`** — presentational components grouped by feature (`dashboard/`, `invoices/`, `customers/`) plus shared ones at the top level. Server Components by default; only `search.tsx`, `invoices/pagination.tsx`, and `invoices/edit-form.tsx` carry `'use client'`. Keep the client boundary that narrow — push interactivity into leaves rather than marking a page as client.
- **`app/ui/skeletons.tsx`** — loading skeletons matching each dashboard component, used with `loading.tsx` / `<Suspense>` in later chapters. The `shimmer` keyframe they rely on is defined in `tailwind.config.ts`.
- **Pagination** uses a fixed `ITEMS_PER_PAGE = 6` in `data.ts`; `generatePagination` in `utils.ts` produces the ellipsis-aware page list.

## Styling

Tailwind v3 with `tailwind.config.ts` + `postcss.config.js` (not the v4 CSS-first setup). `app/ui/global.css` holds the `@tailwind` directives and number-input resets. Custom `blue-400/500/600` and a `grid-cols-13` extension live in the config. `clsx` is the convention for conditional classes; `@tailwindcss/forms` is enabled.

Note `global.css` is not yet imported in `app/layout.tsx` — the course does that in the CSS chapter.

## Dependencies present but unused so far

`next-auth@5.0.0-beta.25`, `bcrypt`, `zod`, and `use-debounce` are installed for later chapters (auth, Server Action validation, debounced search). There is no `auth.ts`, `middleware.ts`, or `app/lib/actions.ts` yet — create them where the course places them.
