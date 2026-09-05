---
description: The Vitest suite guarding the VitePress config, theme and anchor gate
paths:
  - tests/**/*.ts
  - scripts/**/*.mjs
  - vitest.config.ts
  - docs/.vitepress/config.ts
  - docs/.vitepress/theme/index.ts
---

# Repo tests

`npm test` (`vitest run` over `tests/**/*.test.ts`) is a gate, not a formality:
both jobs in `.github/workflows/docs.yml` run it *before* `npm run docs:build`.

```bash
npm test               # the whole suite
npm run test:watch     # vitest in watch mode
npm run test:coverage  # v8 coverage over docs/.vitepress/**/*.ts
npm run typecheck      # vue-tsc --noEmit over docs/.vitepress and tests/
```

## Editing `docs/.vitepress/config.ts` breaks tests on purpose

`tests/config.test.ts` (asserts on the source text) and
`tests/config.import.test.ts` (asserts on the imported object) both pin config
invariants — `ignoreDeadLinks: false`, `cleanUrls: false`, the `srcExclude` of
`docs/old/`, and one `it()` per sidebar section. Adding or renaming a section
means updating the matching assertion; never relax `ignoreDeadLinks` to make a
test pass. `tests/theme.test.ts` and `tests/theme.enhanceApp.test.ts` pin
`docs/.vitepress/theme/index.ts` the same way (imports, `Layout`, `enhanceApp`,
`satisfies Theme`).

## The anchor gate is tested by running it

`tests/anchor-gate.test.ts` spawns the real `scripts/check-anchor-links.mjs`
against synthetic fixture sites and asserts both verdicts — a clean fixture
exits 0, one broken anchor exits 1 and names the href. The fixtures drive it
through its flags:

```bash
node scripts/check-anchor-links.mjs --dist=<dir> --base=</prefix/>
```

Without `--base` the script reads the real base from `docs/.vitepress/config.ts`.
Keep both flags working — this suite is what proves the gate can still fail.
