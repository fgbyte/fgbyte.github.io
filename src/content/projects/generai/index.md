---
title: "GenerAI"
logo: "https://generai-web-staging.fgbyte.workers.dev/favicon.ico"
summary: "A web application designed to empower creators and businesses by generating engaging social media content."
date: "Jul 28 2026"
draft: false
tags:
  - React
  - Tanstack Router
  - Tailwind
  - Drizzle ORM
  - Neon Postgres
demoUrl: https://generai-web-staging.fgbyte.workers.dev
---

# Generai: from full stack developer to Product Engineer — building with AI, live, at $0 cost

I went from shipping features for other people to building my own product with AI agents. It's called **Generai**: you upload an image, and the AI generates captions and ready-to-publish content for Instagram, LinkedIn, X, and more.

100% open source repo, MIT license: **github.com/fgbyte/generai**
Live demo: **generai-web-staging.fgbyte.workers.dev/app**

The entire project — from development through staging — runs on free tiers. **Zero dollars spent.** Cloudflare Workers, Neon serverless Postgres, self-hosted Better Auth: the whole MVP costs nothing to operate. That's as deliberate an architectural decision as any other.

## What it does

1. Upload an image (compressed client-side)
2. AI (vision + LLMs via LangChain) generates captions and a social content pack
3. Auth via email or Google
4. Credits/points system
5. In-app feedback that goes straight to Telegram — no extra DB table, no support dashboard to maintain

## The stack and why

- **Edge / API:** Cloudflare Workers + Hono
- **Web:** Vite + TanStack Router + TanStack Query + Hono RPC
- **Mobile:** Tauri v2 (Rust + Web) — same web codebase running natively on Android
- **Auth:** Better-Auth
- **DB:** Neon (serverless Postgres) + Drizzle ORM
- **AI:** LangChain + vision models + LLMs
- **Infra:** Alchemy on top of Cloudflare
- **Monorepo:** Turborepo

Fully typed end to end, and deliberately structured so agents/LLMs can work comfortably in the codebase — not a minor detail when a large part of the build leaned on AI-assisted development.

### Hono, not Express/Fastify

Express and Fastify assume a Node runtime. Cloudflare Workers is an edge runtime (V8 isolates), without native Node APIs unless you emulate them. Dropping Express in there means fighting the environment constantly. Hono is built from the ground up for edge: zero friction with Workers, and native typed RPC (Hono RPC) between client and server — the piece that later shaped how I structured the monorepo.

### Alchemy, not plain Wrangler

Wrangler has a rough DX: `wrangler.toml`, limited expressiveness, constant friction the moment you go past the simplest use case. Alchemy points straight at Cloudflare with no tunnels or weird intermediate config, and it's infrastructure-as-code in real TypeScript — the same language as the rest of the project, fully typed end to end. Infra feels like part of the code, not a separate file nobody understands six months later. This decision was the actual starting point for everything that followed.

### Monorepo with Turborepo

Once Alchemy was sorted, splitting backend and frontend into separate apps inside the same repo stopped being friction. That unlocked two concrete things:

- **Hono RPC across apps**: types shared directly between `apps/server` and `apps/web`, no generated clients or separately maintained API contracts.
- **Mobile without a rewrite**: with Tauri v2, the same web codebase runs natively on Android. UI and logic already organized under `packages/` get reused instead of duplicated.

Turborepo sits on top of this to handle build caching.

## Structure

```
generai/
├── apps/
│   ├── web/          # React + TanStack Router
│   └── server/       # Hono API
├── packages/
│   ├── auth/         # Better-Auth config
│   ├── db/            # Drizzle schema, queries, migrations
│   ├── mail/           # Postmark
│   ├── infra/           # Alchemy deployment config
│   ├── config/           # Shared TS configs
│   └── env/               # Env var validation (T3 env)
```

## How it was actually built — no filter

- **Real AI-assisted design**: I described the full concept to Stitch (Google), generated near-final screens in Figma, refined them with Auto Layout + Design Tokens, and exported a `DESIGN.md` (Figma-to-Design.md plugin) to feed directly into coding agents.
- **My own scaffolding**: started from my own starter (`modern-web-starter`) — Drizzle + Neon + typed schemas + Better Auth + CI/CD already wired up, so I wasn't reconfiguring the same setup project after project.
- **Rust + Tauri on Android**: getting the same web codebase onto mobile wasn't trivial. It included a 4-hour debugging session on a `babel/generator` resolution bug in Bun — the lesson: overrides in `package.json` save lives.
- **Cross-platform auth**: cookies + WebView in Tauri + Google OAuth on mobile is a combination that breaks assumptions. The final fix was a bearer token passed via hash in the redirect, not cookies.
- **`/generate` endpoint** with vision + LangChain, TanStack Query + Hono RPC with full type safety, deployed to staging, end-to-end flow working: upload photo → caption + social content → saved.

## The result

A real multi-environment setup (dev/staging/production), isolated Workers and databases per stage, CI running, dependency automation with branch protection, everything running at $0 cost, and an architecture that already supports mobile without having to redo anything.

Now comes the part that matters most: iterating on real feedback.

Open source, MIT: **github.com/fgbyte/generai**
