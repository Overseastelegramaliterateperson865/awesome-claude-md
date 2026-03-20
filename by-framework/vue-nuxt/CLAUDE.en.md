# CLAUDE.md - Vue 3 + Nuxt 3 Project Conventions

## Project Overview

- Built on Nuxt 3 + Vue 3
- Package manager: pnpm, Node.js >= 18

## Project Structure

- `pages/` - File-system routing | `components/` - Auto-imported components | `composables/` - Composition functions (`use*.ts`)
- `stores/` - Pinia state management | `server/api/` - Server-side API endpoints | `utils/` - Utility functions
- `plugins/` - Nuxt plugins | `middleware/` - Route middleware | `types/api/` - API response types

## Composition API

- Use `<script setup lang="ts">` exclusively — Options API is forbidden
- Code order within components: props/emits > reactive state > computed > watch > methods > lifecycle hooks
- Prefer `ref` for consistency; define props with `defineProps<{ title: string }>()` paired with `withDefaults`

## Auto-Imports

- Nuxt 3 auto-imports Vue APIs (`ref`, `computed`, `watch`) and exports from `composables/` and `utils/` — no manual imports needed
- Third-party libraries still require explicit `import`; auto-import types are generated in `.nuxt/imports.d.ts`

## Pinia State Management

- Stores go in the `stores/` directory, named `use*Store.ts`, using Setup Store syntax (functional `defineStore`)
- Persistence via `pinia-plugin-persistedstate`, configured in `nuxt.config.ts`

## UI Component Library

- Use Element Plus or Ant Design Vue via Nuxt modules with on-demand imports — full CSS bundles are forbidden
- Theme customization through CSS variable overrides in `assets/styles/variables.scss`
- Form validation uses the component library's built-in validation rules

## Data Fetching

- Use `useFetch` / `useAsyncData` for data fetching — raw `axios` or `fetch` calls are forbidden
- Wrap a unified request interceptor in `composables/useRequest.ts` for token injection and error handling
- Server APIs use `defineEventHandler`; response format: `{ code: number; data: T; message: string }`

## Internationalization & Localization

- Date handling: dayjs with locale support; i18n via `@nuxtjs/i18n`
- Configure `nuxt.config.ts` for production to disable external resource loading if deploying to restricted networks
- WeChat-related features (official accounts, Mini Program webview) are encapsulated in `composables/useWechat.ts`

## TypeScript Conventions

- Enable strict mode — `any` is forbidden
- Component props and emits must use TypeScript type definitions, not runtime declarations

## Common Commands

- `pnpm dev` - Start development server
- `pnpm build` - Build production artifacts
- `pnpm lint` - Run ESLint checks
- `pnpm typecheck` - Run `nuxi typecheck`
