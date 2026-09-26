# Project Architecture Rules (Non-Obvious Only)

- **Actions vs Queries separation**: `src/lib/actions/` = mutations (Server Actions with `"use server"`). `src/lib/queries/` = reads (server-only, cached). Never mix — queries must not mutate, actions must not use `unstable_cache`.
- **Single DB client**: `src/db/index.ts` exports one shared `db` instance using `postgres` (not `pg` directly). The `pg` package is a dependency but Drizzle uses the `postgres` driver.
- **Clerk user ID is a string (UUID v4)**: Stored as `varchar(36)` in DB. It is never a number. Do not confuse with store/product IDs which are prefixed nanoid `varchar(30)`.
- **Subscription plan limits are DB columns**: `productLimit`, `tagLimit`, `variantLimit` on the `stores` table are authoritative — not derived from a config map. Enforce limits by querying these columns, not by checking plan name.
- **Content layer must be built before tsc**: `pnpm typecheck` runs `contentlayer build && tsc --noEmit`. Any plan that runs `tsc` alone will fail if contentlayer types are stale.
- **No test infrastructure**: There is no Jest, Vitest, or Playwright setup. Any plan to add tests must also provision the test runner from scratch.
- **Upstash Redis is rate-limiting only**: The Redis client (`src/lib/rate-limit.ts`) is exclusively for `@upstash/ratelimit`. It is not used as a general cache — Next.js `unstable_cache` handles caching.
- **Stripe Connect per store**: Each store has its own `stripeAccountId` (Connect account), not a platform-level Stripe customer. Payments flow through individual store Stripe accounts, not a central one.
