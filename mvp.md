# The Root Cause — MVP Blueprint

Tech stack, database strategy, and feature phasing for the civic intelligence platform

---

## Tech Stack

### Frontend

| Layer     | Choice                  | Reason                          |
|-----------|-------------------------|---------------------------------|
| Framework | Next.js 14 (App Router) | SSR for SEO + fast feed loads   |
| Language  | TypeScript              | Type safety across full stack   |
| Styling   | Tailwind CSS            | Rapid UI, consistent tokens     |
| State     | Zustand + React Query   | Server state + client cache     |
| Auth UI   | NextAuth.js             | Google OAuth out of the box     |
| Animation | Framer Motion           | Vote buttons, transitions       |

### Backend

| Layer     | Choice                  | Reason                          |
|-----------|-------------------------|---------------------------------|
| Runtime   | Node.js (Hono or tRPC)  | Type-safe API, edge-ready       |
| Auth      | Google OAuth 2.0 only   | No email/password MVP           |
| Queue     | BullMQ (Redis-backed)   | Mod queue, notifications        |
| Real-time | Socket.io               | Live voting, streams, VC        |
| Media     | Cloudflare R2 + Stream  | Cheap storage + video CDN       |
| Search    | Meilisearch             | Fast fulltext on submissions    |

---

## Database Strategy — Polyglot Persistence

| Database       | Role              | Description |
|----------------|-------------------|-------------|
| **PostgreSQL** | 🔵 Primary        | Core relational data — users, submissions, clans, petitions, MLA/MP ratings, roles, points (Vikas Coins). Use Prisma ORM with row-level security for multi-role access control. |
| **Redis**      | 🟢 Cache + Queues | 24-hour vote windows, leaderboard sorted sets (ZADD/ZRANGE), session cache, rate limiting counters, BullMQ job queues for mod pipeline and notification dispatch. |
| **MongoDB**    | 🟡 Documents      | Flexible document store for submissions with evolving schema: root cause analysis, solution proposals, debate threads, comments with nested votes, binaural beat preferences, media metadata. |
| **ClickHouse** | 🔴 Analytics      | Analytics and feed algorithm input — event stream of votes, views, shares, time-on-post. Powers reach score, viral coefficient, and personalisation ranking (X-algo style). Phase 2 add-on. |

---

## Feature Phases

### Phase 1 — Core MVP *(Ship first)*

- Google OAuth signup
- State selection at signup
- Political persona (Librandus / Andhbhakt / Neutral)
- Submit problem + root cause
- Upvote / downvote (24h window)
- Leaderboard by 24h score
- New submissions feed
- Admin panel — delete/ban/moderate
- Report button
- 1k downvote → auto mod queue
- Basic content moderation

### Phase 2 — Engagement Layer

- Vikas Coins points system
- Role upgrades via points
- Clan system
- Quests + petitions
- Geotagging + timestamping
- Anonymous voting
- Comment section with nested votes
- Debate creation
- Regional language support
- Show more / pagination
- MLA/MP constituency rating

### Phase 3 — Media + Social

- Meme picker (latest viral)
- Song / sound picking (Instagram-style)
- Chai Ki Tapri voice channel
- Stream + VC (audio level control)
- Binaural beats layer
- Follower count + news edits feed
- Cross-clan content chain
- White Paper / Objectives page
- Debate summary for new joiners

### Phase 4 — Intelligence + Algorithm

- X-algo style ranking (open source)
- Viral reach score (ClickHouse)
- Govt ID verification
- Troll vs genuine classifier
- Basic content media library
- Personalised feed by state + persona
- Upvote explanation prompts
- Mandate creation from proposals

---

## Infrastructure

### Hosting
Vercel (Next.js frontend) + Railway or Render (backend + DBs). DigitalOcean for self-hosted Meilisearch + ClickHouse in Phase 2.

### Media
Cloudflare R2 for images + uploads. Cloudflare Stream for video. Geotagged media metadata stored in MongoDB with lat/long + timestamp fields.

### Mod Pipeline
BullMQ queue: 1k downvote threshold → auto-enqueue → admin review UI → suspend/restore. Shadowban flag in Postgres users table.
