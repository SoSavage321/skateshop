# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Stack

Next.js 14 (App Router) · TypeScript · Drizzle ORM (PostgreSQL) · Clerk auth · Stripe · Uploadthing · Upstash Redis · Contentlayer (MDX blog) · shadcn/ui + Tailwind CSS · pnpm

## Commands

```bash
pnpm dev           # start dev server
pnpm build         # contentlayer build && next build
pnpm lint          # next lint
pnpm lint:fix      # auto-fix lint
pnpm typecheck     # contentlayer build && tsc --noEmit
pnpm format:write  # prettier write (ts, tsx, mdx)
pnpm check         # lint + typecheck + format:check (all-in-one CI check)

# DB (all require a .env file; dotenv-cli injects it automatically via "dotenv" prefix)
pnpm db:generate   # generate drizzle migrations
pnpm db:push       # push schema directly (no migration file)
pnpm db:migrate    # run migrations via src/db/migrate.ts
pnpm db:seed       # seed via src/db/seed.ts
pnpm db:studio     # open drizzle studio

pnpm email:dev     # preview emails on port 3001
pnpm stripe:listen # forward Stripe webhooks to localhost:3000
```

> There are **no tests** in this project — no test runner is configured.

## Critical Patterns

### ID Generation
Always use `generateId()` from [`src/lib/id.ts`](src/lib/id.ts) for new DB records. It produces prefixed nanoid strings (e.g. `str_abc123`). Prefixes are defined in the `prefixes` map in that file — add new entity types there.

### Server Actions return shape
All server actions in [`src/lib/actions/`](src/lib/actions/) return `{ data, error }` — never throw to the caller. Use `getErrorMessage()` from [`src/lib/handle-error.ts`](src/lib/handle-error.ts) inside `catch` blocks.

### Query files are server-only
All files under [`src/lib/queries/`](src/lib/queries/) import `"server-only"` at the top. They use `unstable_cache` for read queries and `unstable_noStore` for mutation-adjacent reads. Do not call them from client components.

### DB schema conventions
- Spread `lifecycleDates` from [`src/db/schema/utils.ts`](src/db/schema/utils.ts) into every table (`createdAt` / `updatedAt`).
- Export `type Foo = typeof foos.$inferSelect` and `type NewFoo = typeof foos.$inferInsert` at the bottom of each schema file.
- All schema files are re-exported through [`src/db/schema/index.ts`](src/db/schema/index.ts).
- Use `takeFirst` / `takeFirstOrThrow` from [`src/db/utils.ts`](src/db/utils.ts) instead of `[0]` indexing on query results.

### Environment variables
Env is validated at startup via `@t3-oss/env-nextjs` in [`src/env.js`](src/env.js) (`.js` extension, not `.ts`). Always import env as `import { env } from "@/env.js"` — the `.js` extension is required even in TypeScript files. Skip validation with `SKIP_ENV_VALIDATION=1`. *(Proven: [`src/env.js`](src/env.js) lines 1–76; [`src/db/index.ts`](src/db/index.ts) line 1 shows the `.js` import in practice.)*

### Type imports
ESLint enforces `import type { ... }` (inline style). Use `import { type Foo, bar }` not `import type { Foo }` as a separate statement.

## Code Style

- **Prettier**: no semicolons, double quotes, 2-space indent, LF line endings, trailing commas (ES5).
- **Import order** (enforced by `@ianvs/prettier-plugin-sort-imports`): react → next → third-party → `@/types` → `@/config` → `@/lib` → `@/hooks` → `@/components/ui` → `@/components` → `@/styles` → `@/app` → relative.
- **Tailwind**: use `cn()` from `@/lib/utils` for conditional classes. Tailwind ESLint plugin validates class names; `cva` and `cn` are registered as callees.
- **`@total-typescript/ts-reset`** is applied globally via [`reset.d.ts`](reset.d.ts) — `noUncheckedIndexedAccess` is enabled in tsconfig.
- Path alias `@/*` maps to `src/*`.

## Architecture Notes

- Route groups: `(auth)`, `(checkout)`, `(dashboard)`, `(lobby)` — purely organizational, no shared layout coupling implied by name. **Only `/dashboard(.*)`** is actually protected by Clerk middleware ([`src/middleware.ts`](src/middleware.ts) lines 3, 9); all other routes are public unless a layout adds its own guard.
- DB client uses the `postgres` npm driver (not `pg`) via `drizzle-orm/postgres-js` — `pg` is a listed dependency but Drizzle does not use it. *(Proven: [`src/db/index.ts`](src/db/index.ts) lines 2–3.)*
- Drizzle schema entry point is `src/db/schema/index.ts`; dialect is `postgresql`; migrations output to `./drizzle/`. *(Proven: [`drizzle.config.ts`](drizzle.config.ts) lines 4–10.)*
- All `db:*` scripts prefix with `dotenv` (dotenv-cli) to inject `.env`; running `drizzle-kit` directly without it will fail. *(Proven: [`package.json`](package.json) lines 18–24.)*
- `next.config.js` sets `eslint.ignoreDuringBuilds: true` and `typescript.ignoreBuildErrors: true` — CI validation must be done via `pnpm check` separately.
- Contentlayer processes MDX from [`src/content/`](src/content/) and outputs to `.contentlayer/generated` (gitignored). Run `pnpm build` or `pnpm typecheck` to regenerate types after editing MDX documents.

## Validation

Scripts confirmed present in [`package.json`](package.json):

| Script | Command |
|---|---|
| `dev` | `next dev` |
| `build` | `contentlayer build && next build` |
| `lint` / `lint:fix` | `next lint` |
| `typecheck` | `contentlayer build && tsc --noEmit` |
| `format:write` / `format:check` | prettier |
| `check` | lint + typecheck + format:check |
| `db:generate/push/migrate/seed/studio` | drizzle-kit / tsx via dotenv-cli |
| `email:dev` | react-email dev server on port 3001 |
| `stripe:listen` | stripe CLI (must be installed separately) |
| `clean` | rimraf node_modules, dist, .next, etc. |
| `shadcn:add` | pnpm dlx shadcn-ui@latest add |

**No test script exists.** There is no `test`, `test:unit`, or `test:e2e` entry in `package.json` and no test runner (Jest, Vitest, Playwright) is installed.
