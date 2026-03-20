# SvelteKit Project

## Tech Stack
- Svelte 5 + SvelteKit 2, TypeScript in strict mode
- Svelte 5 Runes reactivity system
- Vitest for unit tests + Playwright for E2E tests

## Project Structure
```
src/
  routes/       # File-system routing: +page.svelte / +layout.svelte / +server.ts
  lib/          # Shared code, accessible via the $lib alias
    components/ # Reusable components
    server/     # Server-only code (database access, secrets, etc.)
  params/       # Route parameter matchers
static/         # Static assets
```

## Svelte 5 Runes Syntax
- `$state(value)` declares reactive state, replacing the old reactive `let` declarations
- `$derived(expr)` computes derived values, replacing `$:` syntax
- `$effect(() => {})` runs side effects, replacing `$:` statement blocks
- `$props()` declares component props, replacing `export let`
- `$bindable()` declares a prop that supports two-way binding
- Do NOT use legacy Svelte 4 syntax (`$:`, `export let`, etc.)

## SvelteKit Routing and Data Loading
- `load` functions in `+page.ts` run on both client and server
- `load` functions in `+page.server.ts` run only on the server and can safely access databases
- Form handling uses `actions` in `+page.server.ts`, enhanced with `use:enhance` for progressive enhancement
- API routes go in `+server.ts` files, exporting `GET/POST/PUT/DELETE` functions

## Development Guidelines
- TypeScript strict mode: enable `strict: true` in `tsconfig.json`
- Component files use `<script lang="ts">` tags
- Place server-only code under `$lib/server/` — SvelteKit prevents client-side imports from this path
- Environment variables: use `$env/static/public` for public values, `$env/static/private` for secrets

## Testing
- Unit tests: `npx vitest run`
- E2E tests: `npx playwright test`
- Component tests use `render` + `screen` from `@testing-library/svelte`
- E2E test files go in the `tests/` directory, organized with `test.describe`

## Common Commands
```bash
npm run dev          # Start dev server
npm run build        # Build for production
npm run preview      # Preview the production build
npm run test:unit    # Run Vitest unit tests
npm run test:e2e     # Run Playwright E2E tests
npm run check        # Run svelte-check for type checking
```

## Common Pitfalls
- `$state` objects are deeply reactive, but reassigning the entire object is needed to trigger a reference change
- Return values from `load` functions are serialized for the client — they cannot contain non-serializable values (functions, class instances, etc.)
- When rendering HTML with `{@html}`, always sanitize the content to prevent XSS
- `goto()` calls the target page's `load` by default; pass `invalidateAll: false` to skip it
