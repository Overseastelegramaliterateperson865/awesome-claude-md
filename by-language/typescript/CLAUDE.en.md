# CLAUDE.md - TypeScript Project Conventions

## Language & Compilation

- Use TypeScript strict mode — `"strict": true` must be enabled in `tsconfig.json`
- Do not use `any` — use `unknown` with type guards to narrow types; when truly unavoidable, add `// eslint-disable-next-line @typescript-eslint/no-explicit-any` with a comment explaining why
- Use ESM module system — set `"type": "module"` in `package.json`, and `"module": "ESNext"`, `"moduleResolution": "bundler"` in `tsconfig.json`
- Target `"target": "ES2022"` to leverage modern APIs like `structuredClone`, `Array.at()`, etc.

## Code Style & Linting

- Prefer Biome (`biome.json`) for formatting and linting; if the project uses ESLint, pair it with `@typescript-eslint/recommended-type-checked`
- 2-space indentation, no semicolons, single quotes, trailing commas `all`
- File naming: `kebab-case.ts`; type definition files: `*.types.ts`; constant files: `*.constants.ts`

## Import Conventions

- Import order: Node built-in modules > third-party packages > internal project modules > relative path modules, with blank lines between groups
- Use `import type { Foo }` for type-only imports to avoid runtime overhead
- Path alias: use `@/` pointing to `src/`, configured in `tsconfig.json` `paths`

## Type Authoring

- Prefer `interface` for object shapes; use `type` for union types, intersection types, and utility types
- Give generic parameters meaningful names: `TItem` instead of `T`, `TResponse` instead of `R`
- Exported functions must have explicit return type annotations; internal functions may rely on inference
- Replace `enum` with `as const` objects + `typeof` to avoid runtime overhead

## Testing

- Test framework: Vitest, configured in `vitest.config.ts`
- Test files go in `__tests__/` directories or alongside source files as `*.test.ts`
- Run tests: `pnpm test`; run a single file: `pnpm test src/utils/__tests__/foo.test.ts`
- Organize tests with `describe` + `it`; descriptions should be English verb phrases, e.g., `it('returns null when input is empty')`
- Prefer `vi.fn()` and `vi.spyOn()` for mocking — avoid mocking entire modules

## Common Commands

- `pnpm dev` - Start development server
- `pnpm build` - Build production artifacts
- `pnpm lint` - Run lint checks
- `pnpm format` - Format code
- `pnpm typecheck` - Run `tsc --noEmit` type checking
