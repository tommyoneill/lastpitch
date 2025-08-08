# Last Pitch – Product Requirements Document (PRD)

## 1. Overview
Last Pitch is a web application that transforms backyard and community wiffle ball tournaments into a professional, shareable experience. It enables organizers to manage registration, team creation, scheduling, scoring, and post-game recaps in a fast, mobile-friendly interface.

**Tech Stack**
- **Framework**: T3 Stack (Next.js + TypeScript + Prisma + tRPC + NextAuth + TailwindCSS)
- **UI Components**: ShadCN UI
- **Deployment**: Vercel
- **Database**: PostgreSQL (Supabase)
- **Hosting**: Multi-tenant, lightweight implementation (single DB with tenant IDs)
- **Storage**: Supabase storage for media uploads

---

## 2. Goals
- Deliver a **fast, intuitive MVP** by prioritizing core features.
- Support **multi-tenant** setup for multiple tournaments with minimal complexity.
- Optimize for **mobile-first use** for both organizers and players.
- Create a **"big league" feel** without slowing development.

---

## 3. Target Users
- **Organizers**: Parents, coaches, or community leaders running the tournament.
- **Players**: Kids and teens participating in the games.
- **Fans**: Friends and family following along online.

---

## 4. Core Features (MVP)

### 4.1 Authentication & Multi-Tenancy
- Email/password login (NextAuth w/ Supabase adapter).
- Organizer creates an **organization/tournament** upon sign-up.
- All data scoped by **tenant ID**.

### 4.2 Tournament Setup
- Create tournament (name, date, location, description).
- Add player registration form (name, age, optional photo).
- Assign players to teams manually or via auto-draft.

### 4.3 Scheduling & Brackets
- Auto-generate pools and brackets (round robin + single elimination).
- Editable match schedule.

### 4.4 Live Scorekeeping
- Mobile-friendly score entry screen.
- Real-time score updates via WebSocket (tRPC subscription).

### 4.5 Game Recaps
- AI-generated game summary (OpenAI API).
- Attach fan-uploaded photos and short videos.

### 4.6 Public View
- Public tournament page (shareable link).
- Live scores, standings, and recaps.

---

## 5. Non-Goals (MVP)
- No payment processing or monetization.
- No advanced stats or analytics.
- No custom domain per tenant (single domain with subpaths).
- No offline mode.

---

## 6. Technical Architecture

### 6.1 Data Model (Initial)
- **User**: id, name, email, role (organizer, player, fan).
- **Tenant**: id, name, owner_id.
- **Tournament**: id, tenant_id, name, date, location, description.
- **Team**: id, tournament_id, name, color.
- **Player**: id, team_id, name, age, photo_url.
- **Game**: id, tournament_id, team1_id, team2_id, scheduled_time, score_team1, score_team2, status.
- **Media**: id, game_id, uploaded_by, type (photo/video), url.
- **Recap**: id, game_id, content (AI-generated).

### 6.2 Multi-Tenant Strategy
- **Single database schema**.
- Every entity tied to a `tenant_id` to isolate data.
- Access control enforced in tRPC routers.

### 6.3 API
- **tRPC** for all API endpoints.
- **Protected procedures** for organizer-only actions.
- **Public procedures** for read-only access.

### 6.4 Hosting
- Vercel for front-end and serverless API routes.
- Supabase for DB, auth, and media storage.

---

## 7. UI/UX Notes
- ShadCN UI for polished, consistent components.
- Dark and light themes.
- Use the Indigo colors from ShadCN
- Mobile-first design with thumb-friendly buttons.
- Organizer dashboard for setup, scheduling, and scoring.
- Public pages optimized for quick sharing.

---

## 8. Performance & Scalability
- Favor **simple queries** and **indexed lookups**.
- Use **ISR** (Incremental Static Regeneration) for public pages.
- WebSocket (tRPC subscription) only for live scoring.
- Optimize images and media uploads (Supabase storage + CDN).

---

## 9. Success Metrics
- MVP launched within **6 weeks**.
- First 5 tournaments successfully run on the platform.
- < 200ms server response times for 95% of API calls.
- > 80% of users return for at least one more tournament.

---

## 10. Milestones
1. **Week 1-2**: Auth, multi-tenant setup, tournament CRUD.
2. **Week 3**: Player registration & team drafting.
3. **Week 4**: Scheduling, bracket generation, live scoring UI.
4. **Week 5**: AI recap integration, media uploads.
5. **Week 6**: Polish, bug fixes, deploy to Vercel.

---

## 11. Risks & Mitigations
- **Scope creep** → Strict MVP feature set.
- **AI cost overruns** → Limit recap generation to 1 per game in MVP.
- **Media abuse** → Implement basic file type/size validation and content moderation.
