# CLAUDE.md - Next.js App Router Project Conventions

## Project Structure

- Use Next.js App Router (`app/` directory) — do not use Pages Router
- Directory conventions:
  - `app/` - Routes and pages
  - `components/` - Reusable components, organized by feature (`components/ui/`, `components/layout/`)
  - `lib/` - Utility functions, API clients, database operations
  - `types/` - Global type definitions
  - `public/` - Static assets

## Server / Client Components

- All components are Server Components by default — only add `'use client'` when interaction, browser APIs, or hooks are needed
- `'use client'` goes on the first line of the file; everything in the module tree below it becomes client-side
- In Server Components, `await` async data directly — no need for `useEffect` + `useState`
- Extract interactive logic into small Client Components embedded within Server Components to minimize the client bundle

## Data Fetching

- In Server Components, call the database or internal APIs directly without going through HTTP
- When using `fetch()`, leverage Next.js built-in caching: `fetch(url, { next: { revalidate: 3600 } })`
- For real-time data, use `{ cache: 'no-store' }` or set `export const dynamic = 'force-dynamic'` in the layout/page
- Mutations use Server Actions (`'use server'` functions) paired with `revalidatePath()` / `revalidateTag()` to refresh cache
- Form submissions prefer `<form action={serverAction}>`; supplement on the client side with `useFormStatus` and `useActionState`

## API Routes

- API routes live under `app/api/`, exporting named functions like `GET`, `POST`
- Use `NextRequest` / `NextResponse` types
- Consistent response format: `{ data, error, message }`
- Auth middleware goes in `middleware.ts` (project root), matching via `config.matcher`

## Styling

- Use Tailwind CSS — do not write custom CSS files unless absolutely necessary
- Apply Tailwind utility classes directly in JSX for component styles
- For complex style combinations, use `clsx` or `cn` (a `twMerge + clsx` wrapper in `lib/utils.ts`)
- Responsive breakpoint order: `sm:` > `md:` > `lg:` > `xl:` — mobile-first approach

## TypeScript Conventions

- Enable strict mode — `any` is forbidden
- Page props types use Next.js built-in generics: `type Props = { params: Promise<{ slug: string }>; searchParams: Promise<Record<string, string>> }`
- Server Action parameters must be validated with Zod — never trust client input

## Common Commands

- `pnpm dev` - Start development server (http://localhost:3000)
- `pnpm build` - Build production artifacts
- `pnpm lint` - Run `next lint`
- `pnpm typecheck` - Run `tsc --noEmit`
- `pnpm test` - Run Vitest tests
