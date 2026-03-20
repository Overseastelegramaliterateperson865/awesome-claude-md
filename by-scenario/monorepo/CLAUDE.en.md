# Monorepo Project Conventions (Turborepo + pnpm Workspace)

## Workspace Structure

- `apps/` - Deployable applications (web, api, admin, etc.)
- `packages/` - Shared packages (ui, utils, config, tsconfig, etc.)
- `tooling/` - Dev toolchain configuration (eslint-config, prettier-config, etc.)
- Root `pnpm-workspace.yaml` defines all workspace members

## Dependency Management

- Always use `pnpm add` to install dependencies — npm/yarn are forbidden
- Add a dependency to a specific workspace: `pnpm add <pkg> --filter <workspace>`
- Inter-package references use `"@repo/ui": "workspace:*"` — never pin to a fixed version
- Root `package.json` should only contain devDependencies (husky, lint-staged, etc.)

## Build Order & Caching

- Turborepo automatically infers the build topological order based on `dependsOn` in `turbo.json`
- `packages/` build before `apps/` — downstream consumers must rebuild after shared package changes
- Use `turbo run build --filter=<app>...` to build only the target and its dependency chain
- With remote caching enabled, unchanged packages hit the cache and skip builds in CI

## Shared Package Development

- Every `packages/*` must have its own `package.json`, `tsconfig.json`, and `src/index.ts` entry point
- Shared UI components go in `packages/ui`; utility functions in `packages/utils`
- Shared package exports must be explicitly declared in the `exports` field of `package.json`
- Ship type definitions alongside source code — do not maintain separate `.d.ts` files

## Testing Strategy

- Unit tests: each workspace runs independently via `pnpm test --filter=<workspace>`
- Integration tests: placed in the corresponding app directory, may depend on built shared packages
- In CI, use `turbo run test --filter=...[origin/main]` to test only affected packages
- Each shared package's tests must pass independently without depending on other workspaces' runtime state

## Common Commands

- `pnpm dev --filter=web` - Start a single app's development server
- `pnpm build` - Full build (Turborepo auto-parallelizes with caching)
- `pnpm lint` - Full lint check
- `turbo run build --dry` - View the build dependency graph without executing

## Important Notes

- Run `pnpm install` to update the lockfile after adding a new workspace
- Cross-workspace imports via relative paths are forbidden — always reference by package name
- Environment variables are isolated per app; each app maintains its own `.env.example`
- PRs that modify shared packages should automatically trigger tests for all consumers in CI
