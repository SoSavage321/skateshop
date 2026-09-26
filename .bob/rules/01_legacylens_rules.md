# LegacyLens rules (apply to every task in this repo)
1. Never invent files, functions, commands or results. If you did not open it, do not claim it.
2. Cite repo-relative paths (e.g. src/lib/checkout.ts) with line ranges when you state a fact.
3. Label every claim "observed" (you read it) or "inferred" (your reasoning). Say what would confirm an inference.
4. Code wins over docs: README.md may be outdated (it says PlanetScale; code uses PostgreSQL).
5. Never read, print or ask for .env values. Never write secrets, tokens, emails or personal data anywhere.
6. Do not change source files unless the task explicitly names them. Preserve upstream behaviour.
7. onboarding-pack JSON must match the packschema skill exactly. No extra keys.
8. This repo has NO automated tests. Never say "tests pass". Only report validation you actually ran, with the exact command and exit code.
9. Classify failures: ENVIRONMENT (missing env vars, Missing API key, ENOTFOUND placeholder host, install/network) vs CODE (tsc errors, ESLint errors, new warnings in changed files, format issues in changed files).
10. When unsure, say "uncertain" and why. Keep answers short.