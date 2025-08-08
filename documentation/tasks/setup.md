## Framework Setup Tasks

This checklist captures the foundation we should complete before building features. We’ll check items off here as we go.

### Environment & Secrets
- [X] Create `.env` from `.env.example` and set:
  - [X] `AUTH_GOOGLE_ID`
  - [X] `AUTH_GOOGLE_SECRET`
  - [X] `AUTH_SECRET` (required in production)
  - [X] `DATABASE_URL` (Postgres)
- [X] Configure Google OAuth client in Google Cloud Console
  - [X] Authorized JS origin: `http://localhost:3000`
  - [X] Redirect URI: `http://localhost:3000/api/auth/callback/google`
- [X] Ensure `.env.example` includes Google keys and example `DATABASE_URL`
- [X] Verify env validation passes: `npm run dev` starts without env errors

### Database & Prisma
- [X] Setup local database in Docker 
- [X] Run initial migration and generate client: `npm run db:generate`
- [ ] Align Prisma schema with conventions
  - [ ] Prefer `cuid()`/`uuid()` ids over incremental
  - [ ] Ensure timestamps: `createdAt` (default now), `updatedAt` (`@updatedAt`)
  - [ ] Add common indices (ids, foreign keys, `createdAt`)
  - [ ] Remove or refactor the example `Post` model to use `cuid/uuid`
- [ ] Plan multi-tenancy in app models (not auth tables)
  - [ ] Add `tenantId` to multi-tenant entities when introduced
  - [ ] Composite uniques where applicable (e.g., `@@unique([tenantId, slug])`)
  - [ ] Create helper stub `withTenant(where, tenantId)` for routers

### tRPC API
- [X] Confirm server context exposes `db` and `session` (`~/server/api/trpc.ts`)
- [ ] Add tenant derivation placeholder in context (derive from session/request)
- [X] Use `publicProcedure` only for truly public reads
- [X] Use `protectedProcedure` for any user-specific reads/writes
- [ ] Add a basic `health` query for smoke testing

### Scaffold Cleanup (remove demo blog)
- [ ] Remove demo Post feature end-to-end
  - [ ] Delete `src/server/api/routers/post.ts`
  - [ ] Remove `post` from `src/server/api/root.ts`
  - [ ] Delete `src/app/_components/post.tsx`
  - [ ] Remove demo usage from `src/app/page.tsx`
  - [ ] Update `prisma/schema.prisma` to remove `Post` model
  - [ ] Create and apply migration
  - [ ] Update README to remove blog references

### Auth (Auth.js / NextAuth)
- [X] Verify Google sign-in/out flow works end-to-end
- [X] Session contains `user.id` (already added in callbacks)
- [ ] Prepare for roles later (extend `Session` when roles land)
- [ ] Add basic SignIn/SignOut UI using Shadcn Button

### UI Foundation (Shadcn + Tailwind)
- [ ] Initialize Shadcn primitives (Button, Input, Dialog, DropdownMenu, Form)
- [ ] Add `cn` helper and adopt `cva` for variants
- [ ] Apply theme tokens (Indigo palette per PRD)
- [ ] Layout skeleton with navbar (brand, auth actions) and container
- [ ] Accessibility baseline: focus styles, labels, keyboard nav

### Home Page Mockup (Last Pitch)
- [ ] Replace default landing with Last Pitch mockup (RSC)
  - [ ] Navbar: brand + sign in/out (Shadcn Button)
  - [ ] Hero: headline, subcopy, primary CTA (Sign in / Try demo)
  - [ ] “What it enables” section (cards matching PRD bullets)
  - [ ] Footer (links, minimal)
  - [ ] Mobile-first layout and accessible structure

### Data Fetching & State
- [X] RSC: use `~/trpc/server` with `HydrateClient` for server data + hydration
- [X] Client: use `~/trpc/react` hooks + React Query
- [X] Set sensible React Query defaults (staleTime/gcTime) in `createQueryClient`
- [X] Add mutation success invalidations for common entities

### Realtime (for live scoring)
- [ ] Decide transport for live updates (SSE via `httpBatchStreamLink` vs websockets)
- [ ] Scaffold subscription endpoint(s) in tRPC (defer until live scoring feature)

### Storage (future for media uploads)
- [ ] Add Supabase client config and envs (defer until media feature)
- [ ] Define upload constraints (type/size) + basic moderation approach

### Deployment
- [ ] Vercel project setup: add env vars (`AUTH_*`, `DATABASE_URL`, `AUTH_SECRET`)
- [ ] Ensure `npm run db:migrate` runs during deploy
- [ ] Choose managed Postgres and pooling strategy (e.g., Supabase/Neon with pgBouncer)

### DX, QA & CI
- [ ] Ensure `npm run check` (lint + typecheck) passes cleanly
- [ ] Optional: GitHub Actions for lint/typecheck on PRs
- [ ] Optional: Pre-commit hooks (lint-staged) for staged files

### Docs
- [X] Keep `README.md` in sync (Google OAuth, Quickstart, Prisma workflow)
- [ ] Maintain this `setup.md` checklist; check off items as completed
