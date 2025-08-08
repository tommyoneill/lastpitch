# LastPitch — T3 + Shadcn app

TypeScript-first web app using the T3 stack with pragmatic defaults.

## Stack

- Next.js 15 (App Router, RSC-first)
- React 19
- tRPC v11 + React Query v5
- Prisma 6 + PostgreSQL
- Auth.js (NextAuth) v5 beta
- Zod for runtime validation
- Tailwind CSS 4 + Shadcn UI primitives

## Prerequisites

- Node.js 20+ (recommended)
- npm 10+ (repo uses `npm@11.x` via `packageManager`)
- Docker or Podman (for local Postgres via `start-database.sh`)

## Quickstart

1) Copy envs and fill them in

```bash
cp .env.example .env
# Required server envs (see src/env.js):
# AUTH_SECRET (required in production)
# AUTH_DISCORD_ID
# AUTH_DISCORD_SECRET
# DATABASE_URL (Postgres)
```

Example `DATABASE_URL` (Postgres):

```
postgresql://postgres:password@localhost:5432/lastpitch
```

2) Start a local Postgres (Docker/Podman)

```bash
./start-database.sh
```

3) Install dependencies

```bash
npm install
```

4) Initialize DB and Prisma Client

```bash
# Create/apply a dev migration (preferred for local dev)
npm run db:generate

# Alternatively, sync schema without migrations
npm run db:push

# Optional: open Prisma Studio
npm run db:studio
```

5) Start the dev server

```bash
npm run dev
```

Visit http://localhost:3000

## Scripts

- `dev`: Next dev with Turbo
- `build`: Next build
- `preview`: Build + start
- `start`: Start production server
- `check`: Lint + typecheck
- `lint` / `lint:fix`: ESLint
- `typecheck`: `tsc --noEmit`
- `format:check` / `format:write`: Prettier
- `db:generate`: `prisma migrate dev` (creates and applies a migration in dev)
- `db:push`: `prisma db push` (pushes schema without creating a migration)
- `db:migrate`: `prisma migrate deploy` (apply migrations in prod)
- `db:studio`: Prisma Studio

## Environment variables

Defined and validated in `src/env.js` via `@t3-oss/env-nextjs`.

- `AUTH_SECRET` (required in production)
- `AUTH_DISCORD_ID`
- `AUTH_DISCORD_SECRET`
- `DATABASE_URL` (Postgres connection string)
- `NODE_ENV`

Auth provider: Discord is configured by default. Add more providers in `src/server/auth/config.ts` (ensure schema fields as needed).

## App architecture (T3 conventions)

- Next.js App Router with Server Components by default; opt in to Client Components only when interactivity/hooks are needed.
- tRPC for all app APIs. Public vs protected procedures are the security boundary. Access `db` via `ctx.db`, session via `ctx.session`.
- Prisma is the single data access layer. Prefer `select` to narrow results.
- Zod validates every procedure input; infer types with `z.infer`.
- React Query manages client-side async state; use sensible `staleTime`/`gcTime` and invalidate on mutations.
- Shadcn UI primitives + Tailwind for styling. Use `cva` for variants; keep components small and accessible.

## Prisma workflow

1. Edit `prisma/schema.prisma`
2. Create/apply a dev migration: `npm run db:generate`
3. Regenerate client (handled by `postinstall`, but safe to rerun): `npx prisma generate`
4. Open Studio if needed: `npm run db:studio`

For production: run `npm run db:migrate` during deploy to apply committed migrations.

## tRPC usage

- Define small routers under `src/server/api/routers/*`. Export them via `src/server/api/root.ts`.
- Use `publicProcedure` for anonymous reads; `protectedProcedure` for user data or writes.
- Access session via `ctx.session` and DB via `ctx.db`. Never accept `userId` from the client.

Client usage (React):

```tsx
import { api } from "~/trpc/react";

function Example() {
  const { data } = api.post.getLatest.useQuery();
  return <pre>{JSON.stringify(data, null, 2)}</pre>;
}
```

RSC usage:

```tsx
import { api, HydrateClient } from "~/trpc/server";

export default async function Page() {
  const post = await api.post.getLatest.fetch();
  return (
    <HydrateClient>
      {/* your client components */}
    </HydrateClient>
  );
}
```

## Auth

- NextAuth v5 (Auth.js) with Prisma adapter.
- Session augmentation adds `user.id`. Access on server via `auth()` and in tRPC via `ctx.session`.
- Default provider: Discord. Set `AUTH_DISCORD_ID` and `AUTH_DISCORD_SECRET` in `.env`.

## Cursor rules (project conventions)

This repo includes Cursor rule sets that encode our conventions:

- `.cursor/rules/t3app.mdc` — Global T3 app rules
- `.cursor/rules/typescript.mdc` — TypeScript conventions
- `.cursor/rules/trpc.mdc` — tRPC usage, context, errors, subscriptions
- `.cursor/rules/prisma.mdc` — Modeling, migrations, query scoping
- `.cursor/rules/shadcn.mdc` — UI composition, styling, accessibility
- `.cursor/rules/auth.mdc` — Auth sessions, providers, roles/tenancy
- `.cursor/rules/react.mdc` — RSC-first, client components, state/data

## Deployment

- Vercel recommended. Set all env vars in the project settings.
- Use a managed Postgres and set `DATABASE_URL`.
- Run `npm run db:migrate` during deploy to apply migrations.

## Troubleshooting

- "Prisma client not found": run `npx prisma generate`.
- "AUTH_SECRET missing" in prod: set `AUTH_SECRET` in envs.
- DB port in use: stop existing Postgres or change port in `DATABASE_URL` and restart `./start-database.sh`.
