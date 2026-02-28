# NewTube — A modern YouTube-style video platform

A production-grade YouTube clone built on **Next.js App Router** with **tRPC end-to-end typesafety**, **Clerk authentication**, **Mux video streaming**, and a **PostgreSQL + Drizzle ORM** data layer. NewTube focuses on a clean modular architecture, secure webhook-driven workflows, and an optimized UI/UX for browsing, uploading, and managing videos.

> Live Demo: https://youtube-clone-eta-lyart-26.vercel.app/  
> Repository: https://github.com/RiyaKaushik-tech/youtube-clone

---

## Key Features

- **Authentication & protected routes** via **Clerk** middleware (`/studio`, `/subscriptions`, `/playlists`, etc.)
- **Video ingest + streaming pipeline** powered by **Mux**
  - Webhook verification and event handling for asset lifecycle updates
  - Playback/thumbnail/preview generation based on Mux playback IDs
- **Uploads & media management** with **UploadThing**
  - Banner image upload + replacement (deletes previous file)
  - Video thumbnail upload + replacement (deletes previous file)
- **Typed API layer** using **tRPC v11 + React Query**
  - Batched client requests (`httpBatchLink`)
  - RSC hydration helpers for server rendering (`@trpc/react-query/rsc`)
- **PostgreSQL persistence** via **Neon serverless driver** and **Drizzle ORM**
- **Background workflows** using **Upstash Workflow / QStash**
  - AI-assisted **SEO title generation** from transcript (Gemini)
  - AI-assisted **description summarization** from transcript (Gemini)
- **Rate limiting plumbing** with **Upstash Ratelimit** (implemented but currently commented in tRPC auth middleware)
- UI components and UX primitives:
  - **Infinite scroll**
  - **Filter carousel**
  - **Responsive modal**
  - Toast notifications via **Sonner**

---

## Technical Architecture Overview

**Runtime model (high level):**

- **Next.js App Router** serves pages and API routes.
- **Clerk** provides authentication and webhook events:
  - `src/middleware.ts` protects sensitive routes.
  - `src/app/api/users/webhook` syncs Clerk user events into the DB.
- **tRPC** provides the application API:
  - `src/app/api/trpc/[trpc]/route.ts` exposes tRPC over fetch adapter.
  - `src/trpc/*` configures context, auth, client/server integration, and hydration.
- **Drizzle + Neon** provide a typed PostgreSQL data layer:
  - Schema lives in `src/db/schema.ts`.
- **Mux** powers video streaming:
  - `src/app/api/videos/webhook` verifies Mux signatures and updates video records as assets progress.
- **UploadThing** handles file uploads (banner/thumbnail) and cleanup of replaced assets.
- **Upstash Workflow (QStash)** runs async AI workflows (title/description) by pulling transcript text from Mux and calling **Gemini**.

---

## Tech Stack

| Category | Libraries / Services |
|---|---|
| **Frontend** | Next.js 15, React 19, TypeScript |
| **Backend** | Next.js Route Handlers (App Router), tRPC (fetch adapter) |
| **State Management** | TanStack React Query (via tRPC) |
| **Database / ORM** | PostgreSQL (via `@neondatabase/serverless`), Drizzle ORM, drizzle-zod |
| **APIs / Integrations** | Mux (streaming + webhooks), UploadThing (uploads), Upstash Redis + Workflow (QStash), Google Gemini API (`@google/generative-ai`) |
| **Authentication** | Clerk (middleware + webhooks), Svix verification |
| **Styling / UI** | Tailwind CSS, Radix UI, class-variance-authority, tailwind-merge, tailwindcss-animate |
| **Tooling** | ESLint, Drizzle Kit (studio), Bun support (bun.lock), ngrok (local webhook dev), superjson |

---

## Folder Structure

```txt
.
├── drizzle/
├── public/
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   ├── (home)/
│   │   ├── (studio)/
│   │   ├── api/
│   │   │   ├── trpc/[trpc]/route.ts
│   │   │   ├── uploadthing/
│   │   │   │   ├── core.ts
│   │   │   │   └── route.ts
│   │   │   ├── users/webhook/route.ts
│   │   │   └── videos/
│   │   │       ├── webhook/route.ts
│   │   │       └── workflows/
│   │   │           ├── title/route.ts
│   │   │           └── description/route.ts
│   │   ├── globals.css
│   │   ├── layout.tsx
│   │   └── favicon.ico
│   ├── components/
│   │   ├── filter-carousel.tsx
│   │   ├── infinite-scroll.tsx
│   │   ├── responsive-modal.tsx
│   │   ├── user-avatar.tsx
│   │   └── ui/
│   ├── constants.ts
│   ├── db/
│   │   ├── index.ts
│   │   └── schema.ts
│   ├── hooks/
│   ├── lib/
│   │   ├── mux.ts
│   │   ├── ratelimit.ts
│   │   ├── redis.ts
│   │   ├── uploadthing.ts
│   │   ├── utils.ts
│   │   └── workflow.ts
│   ├── middleware.ts
│   ├── modules/
│   │   ├── auth/
│   │   ├── categories/
│   │   ├── comments/
│   │   ├── comment-reactions/
│   │   ├── home/
│   │   ├── playlists/
│   │   ├── search/
│   │   ├── studio/
│   │   ├── subscriptions/
│   │   ├── suggestions/
│   │   ├── users/
│   │   ├── videos/
│   │   ├── video-reactions/
│   │   └── video-views/
│   ├── scripts/
│   └── trpc/
│       ├── client.tsx
│       ├── init.ts
│       ├── query-client.ts
│       ├── server.tsx
│       └── routers/
├── drizzle.config.ts
├── next.config.ts
├── tailwind.config.ts
├── eslint.config.mjs
├── tsconfig.json
├── package.json
└── env-example
```

---

## Installation & Setup

### Prerequisites
- Node.js 18+ (or Bun)
- PostgreSQL database (e.g., **Neon**)
- Accounts/keys for **Clerk**, **Mux**, **UploadThing**, **Upstash**, and **Gemini**

### 1) Clone & install
```bash
git clone https://github.com/RiyaKaushik-tech/youtube-clone.git
cd youtube-clone
npm install
```

### 2) Configure environment
Create `.env.local` using the provided `env-example` as a template.

```bash
cp env-example .env.local
```

### 3) Run the app
```bash
npm run dev
# http://localhost:3000
```

### 4) (Optional) Drizzle Studio
```bash
npm run studio
```

### 5) (Optional) Local webhook development
The repo includes an ngrok-based helper:
```bash
npm run dev:all
```

---

## Environment Variables

The project expects `.env.local` (Drizzle config explicitly loads `.env.local`).

> Source: `env-example`

| Variable | Purpose |
|---|---|
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk frontend key |
| `CLERK_SECRET_KEY` | Clerk server key |
| `CLERK_SIGNING_SECRET` | Clerk webhook signing secret (Svix) |
| `NEXT_PUBLIC_CLERK_SIGN_IN_URL` / `NEXT_PUBLIC_CLERK_SIGN_UP_URL` | Auth routes |
| `NEXT_PUBLIC_CLERK_SIGN_IN_FALLBACK_REDIRECT_URL` / `NEXT_PUBLIC_CLERK_SIGN_UP_FALLBACK_REDIRECT_URL` | Redirect behavior |
| `DATABASE_URL` | PostgreSQL connection string (Neon) |
| `UPSTASH_REDIS_REST_URL` / `UPSTASH_REDIS_REST_TOKEN` | Upstash Redis for rate limit & general use |
| `MUX_ACCESS_TOKEN` / `MUX_SECRET_KEY` | Mux API credentials |
| `MUX_WEBHOOK_SECRET` | Mux webhook verification secret |
| `UPLOADTHING_TOKEN` | UploadThing server token |
| `QSTASH_TOKEN` | QStash token (used by Upstash Workflow client) |
| `UPSTASH_WORKFLOW_URL` | Workflow callback/base URL (used for local dev / webhook routing) |
| `QSTASH_CURRENT_SIGNING_KEY` / `QSTASH_NEXT_SIGNING_KEY` | QStash signature validation keys |
| `GEMINI_API_KEY` | Gemini API key used by workflow routes *(required by code)* |
| `OPEN_API_KEY` | Present in template (not referenced in inspected routes) |
| `DEEPSEEK_API_KEY` | Present in template (not referenced in inspected routes) |

---

## Usage Guide

### Authentication
- Sign in/up through Clerk routes.
- Protected routes are enforced by middleware:
  - `/studio(.*)`
  - `/subscriptions`
  - `/feed/subscribed`
  - `/playlists(.*)`

### Upload banner / thumbnail (UploadThing)
- UploadThing routes live at:
  - `GET/POST /api/uploadthing`
- Upload flows enforce authorization and ownership:
  - Banner uploader deletes the previous banner file (if any) and updates the `users` table.
  - Thumbnail uploader requires a `videoId` and validates video ownership before replacing media.

### Video processing pipeline (Mux + webhooks)
- `POST /api/videos/webhook` verifies `mux-signature`.
- Handles Mux events such as `video.asset.created` and `video.asset.ready` to update DB records.
- On asset ready, it:
  - derives Mux thumbnail/preview URLs from playback ID,
  - uploads them into UploadThing via `uploadFilesFromUrl`,
  - stores persisted URLs/keys in the `videos` record.

### AI workflows (Upstash Workflow + Gemini)
- Workflow endpoints:
  - `POST /api/videos/workflows/title`
  - `POST /api/videos/workflows/description`
- Both:
  - fetch transcript text from Mux `stream.mux.com/.../text/...txt`
  - call Gemini (`gemini-2.0-flash:generateContent`)
  - update `videos.title` or `videos.description`

---

## Engineering Highlights

- **End-to-end type safety**: tRPC + superjson transformer keeps client/server contracts aligned.
- **Server Components hydration**: `createHydrationHelpers` enables RSC-friendly data loading patterns.
- **Secure webhook verification**
  - Clerk webhooks verified via **Svix** headers.
  - Mux webhooks verified via `mux.webhooks.verifySignature(...)`.
- **Asset lifecycle correctness**: webhooks drive state changes (asset created → ready), reducing client polling.
- **Media replacement without leaks**: UploadThing `UTApi.deleteFiles(...)` removes previously stored banner/thumbnail keys before writing new ones.
- **Rate limiting ready**: Upstash Ratelimit is implemented (`src/lib/ratelimit.ts`) and wired into tRPC auth flow (currently commented), making it easy to enable request-level throttling.

---

## Performance & Optimization Notes

- **tRPC HTTP batching** via `httpBatchLink` reduces request overhead for chatty UIs.
- **Stable QueryClient per request** (server) and **singleton QueryClient** (browser) avoids unnecessary cache resets and improves perceived performance.
- Next.js `images.remotePatterns` is configured to allow optimized image loading from:
  - `image.mux.com` (Mux thumbnails)
  - `vt38fw71wp.ufs.sh` (UploadThing-hosted assets)

---

## Security Considerations

- **Clerk middleware protection** prevents unauthenticated access to privileged surfaces.
- **Webhook signature verification** is mandatory for both Clerk and Mux routes to prevent spoofing.
- **Ownership checks** in upload middleware ensure only the video owner can replace thumbnails.
- Secrets are expected in `.env.local`; do not expose:
  - `CLERK_SECRET_KEY`, `CLERK_SIGNING_SECRET`
  - `MUX_SECRET_KEY`, `MUX_WEBHOOK_SECRET`
  - `UPLOADTHING_TOKEN`, `QSTASH_TOKEN`

---

## Future Improvements

- Enable and tune **rate limiting** inside `protectedProcedure` (currently commented).
- Add automated DB migrations and seed scripts (Drizzle output directory exists; migration workflow can be expanded).
- Add automated testing (unit/integration) for webhook handlers and tRPC routers.
- Observability: structured logging + tracing around webhook and workflow execution.
- Harden webhook handlers with stricter schema validation for payloads and improved error telemetry.

---

## Author

**Riya Kaushik**

---
