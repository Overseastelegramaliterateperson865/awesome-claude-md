# Full-Stack SaaS Project Conventions

## Project Structure

- `frontend/` - Frontend application (React/Next.js) with its own `package.json`
- `backend/` - Backend service (Node.js/Express or NestJS) with its own `package.json`
- `shared/` - Shared type definitions and constants between frontend and backend (via workspace reference)
- `database/` - Database migrations and seed scripts
- `infra/` - Infrastructure configuration (Docker, Terraform, CI/CD)
- `docker-compose.yml` - Local development environment orchestration

## Frontend-Backend Separation

- Frontend and backend communicate via REST API or tRPC; interface definitions go in `shared/api-types.ts`
- Frontend must never access the database directly or depend on backend internal modules
- Unified API response format: `{ code: number, data: T, message: string }`
- Error codes are shared between frontend and backend, defined in `shared/error-codes.ts`
- Frontend uses `react-query` / `swr` for server state management — raw fetch calls in components are forbidden

## Authentication & Authorization

- Dual-token model: JWT access token (short-lived, 15min) + refresh token (long-lived, 7d)
- Access token goes in the Authorization header; refresh token in an httpOnly cookie
- Every backend route validates permissions via middleware: `requireRole('admin')`
- Multi-tenant isolation: all database queries must include a `tenant_id` condition, injected uniformly at the ORM layer
- OAuth third-party login (Google/GitHub) callback handlers go in `backend/src/auth/providers/`

## Database & Migrations

- Use Prisma or Drizzle as the ORM — schema is documentation
- Every schema change must generate a migration file: `npx prisma migrate dev --name <description>`
- Never manually modify already-committed migration files — only append new migrations
- Table names use snake_case plural form (`users`, `subscription_plans`)
- Every table must include `created_at` and `updated_at` fields; add `deleted_at` for soft deletes
- Seed scripts provide baseline data for development; run with `pnpm db:seed`

## Environment Variable Management

- Each service maintains an `.env.example` listing all required variables (without actual values)
- Sensitive variables (API keys, DB passwords) must never be committed to the repository
- Frontend environment variables are prefixed with `NEXT_PUBLIC_`, exposing only non-sensitive configuration
- Backend validates environment variable completeness at startup using `zod` — missing variables cause immediate exit
- Multi-environment configuration: `.env.development` / `.env.staging` / `.env.production`

## Subscription & Billing

- Integrate Stripe for payment processing; webhook endpoints verify signatures before handling events
- User plan information is cached in the local database, synced via webhooks — never query Stripe in real time
- Feature limits (rate limiting, quotas) are enforced uniformly at the backend middleware layer
- Free-tier limits are defined in configuration files for easy adjustment without code changes

## Deployment

- Frontend deploys to Vercel / Cloudflare Pages; backend deploys to Railway / Fly.io
- Database uses managed services (PlanetScale / Supabase / Neon)
- CI pipeline: lint -> type-check -> test -> build -> deploy
- Staging and production environments have isolated configuration and separate databases
- Run migrations on staging before release to verify no breaking changes

## Common Commands

- `pnpm dev` - Start frontend and backend simultaneously (concurrently)
- `pnpm db:migrate` - Run database migrations; `pnpm db:studio` - Database GUI
- `pnpm test` - Run all tests; `pnpm typecheck` - Type checking
