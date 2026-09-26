# Project Documentation Rules (Non-Obvious Only)

- **`src/env.js` is `.js`, not `.ts`**: This is intentional for ESM compatibility with Next.js. The schema is defined with Zod inside a `.js` file — do not rename it.
- **Two separate "content" systems**: `src/content/` is MDX processed by Contentlayer (blog/docs). `src/components/emails/` is React Email (transactional email). They are unrelated.
- **Route groups are purely organizational**: `(auth)`, `(checkout)`, `(dashboard)`, `(lobby)` do not imply shared auth guards just by name — check each group's `layout.tsx` or middleware for actual protection.
- **`reset.d.ts` in root**: Applies `@total-typescript/ts-reset` globally. This changes array filter behavior and tightens many TS types — effects are project-wide without any import.
- **`pnpm check` is the CI gate**: Running `pnpm build` does NOT check types or lint (both are suppressed in `next.config.js`). The canonical validation command is `pnpm check`.
- **DB scripts need dotenv**: All `db:*` scripts use `dotenv` prefix (dotenv-cli) to inject `.env`. Running `drizzle-kit` directly without dotenv-cli will fail with missing env vars.
- **Stripe webhook local testing**: `pnpm stripe:listen` forwards to `localhost:3000/api/webhooks/stripe` — requires Stripe CLI installed separately.
