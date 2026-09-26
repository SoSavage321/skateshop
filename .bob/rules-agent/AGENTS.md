# Project Coding Rules (Non-Obvious Only)

- **ID generation**: Use `generateId(prefix)` from `@/lib/id` — never `Math.random()`, `crypto.randomUUID()`, or raw nanoid. Prefix map lives in `src/lib/id.ts`; add new entity prefixes there.
- **Error handling in actions**: Every server action must `return { data: null, error: getErrorMessage(err) }` inside `catch` — never re-throw or `toast` from server code. Use `showErrorToast` only from client components.
- **DB result access**: Use `takeFirst` / `takeFirstOrThrow` from `@/db/utils` — not array index `[0]`. `noUncheckedIndexedAccess` is on and will error otherwise.
- **Server-only query files**: Files in `src/lib/queries/` must keep `import "server-only"` as first line. Never import them in Client Components.
- **Cache strategy**: Read queries use `unstable_cache` with a `tags` array for targeted revalidation (`revalidateTag`). Mutation-adjacent reads call `unstable_noStore()` at the top of the action.
- **env import**: Always `import { env } from "@/env.js"` — `.js` extension required even in `.ts` files due to ESM resolution.
- **Schema table template**: spread `...lifecycleDates` from `@/db/schema/utils`, export `type Foo` and `type NewFoo` inferred types, re-export from `src/db/schema/index.ts`.
- **Tailwind classes**: only via `cn()` (`@/lib/utils`). For variant components use `cva`. Both are ESLint-validated callees — arbitrary strings outside these will not be sorted/linted.
- **Type imports**: inline style only — `import { type Foo, bar }` not a separate `import type` statement (ESLint `consistent-type-imports` inline fixStyle).
- **Build skips checks**: `eslint.ignoreDuringBuilds` and `typescript.ignoreBuildErrors` are both `true` in `next.config.js`. Always run `pnpm check` to validate before committing.
- **Contentlayer types**: after adding/editing MDX doc types in `contentlayer.config.ts`, run `pnpm typecheck` (or `pnpm build`) to regenerate `.contentlayer/generated` before TypeScript will resolve the new types.
