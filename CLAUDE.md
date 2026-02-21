# Project Vision: Android MCP Server

## What This Repository Is Becoming

This repo (forked from fzer0x/android-control-tool-V2) is being transformed into an **MCP (Model Context Protocol) server** that gives LLMs full-stack Android capabilities. The PyQt6 GUI is NOT the focus — it's just one recipe someone made with the same ingredients we're using to build something fundamentally different.

## The Core Idea

A two-layered architecture where the LLM doesn't call tools directly — it queries a **Python MCP server** that acts as a unified endpoint wired to all underlying tools and functionality.

```
LLM Client (Claude, etc.)
    |  (MCP protocol)
    v
Python MCP Server (FastMCP)
    |  (direct calls)
    v
Core Tool Layer (adb, fastboot, jadx, androguard, expo, pnpm, etc.)
    |
    v
Android Device(s) + Build Outputs
```

## The Two Halves

The power of this server comes from combining two opposite ends of the Android development world that are rarely unified:

### Half 1: Reverse Engineering & Security Analysis
Everything needed to tear an app apart and understand it:
- **Androguard** — static APK analysis, permissions, classes, methods, strings, XREF, CFG
- **JADX** — full decompilation to readable source
- **Xposed hooking** — runtime method interception, SSL pinning bypass, parameter logging
- **Magisk / SuperSU** — root management
- **Boot image extraction** — OTA payload dumping, Magisk patching
- **BusyBox** — on-device utilities (dmesg, netstat, ps, find)
- **Build.prop editing**, SELinux toggling
- **Logcat** — real-time log monitoring and filtering

### Half 2: Full-Stack App Development (T3 Turbo Stack)
Not a generic build system — an opinionated, production-ready app factory built on **create-t3-turbo**:

**The Stack (current as of Feb 2026, repo last updated Feb 17 2026):**

```
Monorepo (Turborepo v2.5.8+ / pnpm 10.19.0 / Node.js ^22.21.0)
├── apps/
│   ├── expo/              — React Native 0.81.5 + Expo SDK 54, Expo Router 6, NativeWind v5
│   ├── nextjs/            — Next.js 16.0.9, React 19.1.4, Tailwind CSS v4
│   └── tanstack-start/    — TanStack Start v1 (rc), Vite 7, Nitro
├── packages/
│   ├── api/               — tRPC v11 routers (auth.ts, post.ts, etc.)
│   ├── auth/              — Better Auth 1.4.0-beta.9 (Drizzle adapter + Expo plugin + OAuth proxy)
│   ├── db/                — Drizzle ORM + PostgreSQL on Supabase (snake_case, edge-ready)
│   ├── ui/                — shadcn-ui components
│   └── validators/        — Zod schemas (shared validation)
├── tooling/
│   ├── eslint/            — @acme/eslint-config
│   ├── prettier/          — @acme/prettier-config
│   ├── tailwind/          — @acme/tailwind-config (Tailwind v4)
│   └── typescript/        — @acme/tsconfig
└── turbo/generators/      — Turborepo codegen templates
```

**Scaffolding:**
```bash
# New project from template
npx create-turbo@latest -e https://github.com/t3-oss/create-t3-turbo

# Add a new package to existing monorepo
pnpm turbo gen init
```

**Key tech decisions:**
- **Better Auth 1.4.0-beta.9** over NextAuth/Auth.js — native Expo support via `@better-auth/expo`, plugin ecosystem, OAuth proxy for preview deploys, Discord as default social provider. Auth schema generated via `pnpm --filter @acme/auth generate`
- **Database — two options depending on the project:**
  - **Drizzle + Supabase** — type-safe Postgres with `drizzle-zod` for schema→validator generation, `drizzle-kit push` for migrations, Drizzle Studio for visual DB management. Good for traditional relational data, edge-ready workloads.
  - **Convex** — reactive backend-as-a-service with real-time sync, serverless functions, automatic caching. Use where real-time matters, or where the reactive model is a better fit than traditional request/response.
- **tRPC v11** with `@trpc/tanstack-react-query` — end-to-end type-safe API
- **React 19.1.4** across everything
- **TanStack Query + TanStack Form** — data fetching and form management
- **NativeWind v5** (preview) — Tailwind CSS v4 in React Native
- **Expo Router 6** — file-based routing, `expo-secure-store` for token persistence
- **pnpm 10.19.0** with **catalog system** — centralized dependency versioning in `pnpm-workspace.yaml`
- **Turborepo** — pipeline tasks: build, dev, format, lint, typecheck, clean, push (db), studio, ui-add
- **TypeScript 5.9.3** across all packages
- **superjson 2.2.3** — serialization for tRPC

**Env vars needed:** `POSTGRES_URL`, `AUTH_SECRET`, `AUTH_DISCORD_ID`, `AUTH_DISCORD_SECRET`, `AUTH_REDIRECT_PROXY_URL`

**Build/deploy tools:**
- **Expo CLI / EAS CLI** — local dev builds, cloud production builds
- **pnpm** — workspace-aware dependency management
- **Turborepo** — parallel builds, caching, task orchestration

### The Bridge: UI Automation / Device Interaction
This isn't a bonus — it's what connects the two halves. The LLM can SEE and INTERACT with the device, which is how live analysis feeds into the build process:
- `adb shell screencap` — capture screen
- `adb shell uiautomator dump` — structured view hierarchy (XML)
- `adb shell input tap/swipe/text` — touch, gesture, type
- **scrcpy** — real-time screen mirroring

**The workflow this enables:** A user wants to modernize an app. The LLM runs it on the device, navigates it, hooks into methods as they're triggered by real UI interaction, captures screens/flows/data patterns — all while the app is live. That analysis directly informs what gets built on the development side. It's not just "analyze a static APK" — it's "use the app, understand it in motion, then build something better."

### Native Android Build Tools (Gradle/SDK)
T3 Turbo is for building NEW modern apps. But not everything needs a full-stack rebuild:
- **Gradle** — compile, build, package existing native Android projects
- **APK signing** — keystores, zipalign, apksigner
- **AAPT2** — resource compilation, manifest processing
- **D8/R8** — dex compilation, code shrinking/obfuscation
- **Android SDK Build Tools** — full native compile pipeline

**When to use which:**
- **T3 Turbo** — building a new modern app from scratch (open-source replacement, new project inspired by analysis)
- **Gradle/SDK** — patching an existing APK, modifying native code, or when only the RE half is in play without a full rebuild

## The Pipeline: Why Both Halves Together Matter

RE is the input. Development is the output. One without the other is incomplete.

**The killer workflows:**

### 1. Abandoned App Resurrection
1. Take an abandoned APK nobody maintains anymore
2. Decompile it — map activities, services, permissions, data flows
3. Understand the architecture — screens, APIs, data storage patterns
4. Scaffold a T3 Turbo project — generate tRPC routes matching discovered APIs
5. Create Drizzle schema matching the data models
6. Build React Native screens in Expo replicating/improving the UX
7. Set up Better Auth matching the original auth flow
8. Deploy to device and validate

### 2. Open-Source Replacements
- Decompile a closed-source app the community depends on
- Understand what it does, build a clean open-source version with modern stack
- Ship it as a maintained, community-owned alternative

### 3. Starter Project Inspiration
- Study how real apps solve problems (decompile → analyze)
- Use those patterns as seeds for new projects (scaffold → build)
- Learn from production code, build on it with modern tooling

### 4. Security Audit → Patched Rebuild
- Full analysis of an app's security posture
- Identify vulnerabilities
- Build a fixed version with the same functionality

## The Ingredient Analogy

Cheese and tomato go on pizza, but also in spaghetti and lasagna. The underlying tools (adb, fastboot, androguard, jadx, expo, tRPC, drizzle, etc.) are the ingredients. The current PyQt6 GUI is one recipe. Our MCP server is a completely different dish using the same ingredients. We don't need to "extract" code from main.py — we reference it to see how someone else combined the ingredients, then build our own thing from scratch.

## Architecture Notes

- **Strip the PyQt6 UI entirely** — we don't need it
- **The existing main.py is reference only** — shows how RE tools are invoked, what flags matter, what edge cases exist
- **Build the MCP server fresh** — Python server using FastMCP
- **RE side:** async wrappers around adb, fastboot, androguard, jadx, xposed, etc.
- **Build side:** orchestration of T3 Turbo scaffolding, pnpm, turborepo, expo builds
- The MCP server manages BOTH halves through a unified interface

## Tools Already In This Repo (Auto-Installers Exist for RE Side)
- Android SDK Platform-Tools (adb, fastboot, aapt) — auto-downloads from Google
- scrcpy — auto-downloads from GitHub releases
- JADX — auto-downloads from GitHub releases
- BusyBox — auto-downloads per device architecture
- payload-dumper-go — on-demand download for OTA extraction

## Tools To Add (Build/Development Side)
- Node.js / pnpm — package management
- Turborepo — monorepo orchestration
- create-t3-turbo template — project scaffolding
- Expo CLI / EAS CLI — React Native builds
- Better Auth — authentication setup
- Drizzle ORM — database schema and migrations
- Supabase — hosted Postgres + edge functions
- Convex — reactive backend (alternative to Drizzle+Supabase where it fits)
- tRPC v11 — type-safe API layer
- NativeWind / Tailwind — styling
- Gradle / Android SDK Build Tools — native Android builds (for patching, not full rebuilds)

## Status
- [ ] MCP server architecture designed
- [ ] Core tool layer abstracted (async wrappers around adb, fastboot, etc.)
- [ ] RE/Security tools exposed as MCP tools
- [ ] T3 Turbo scaffolding and build tools integrated
- [ ] UI Automation tools exposed
- [ ] End-to-end pipeline tested (decompile → understand → scaffold → build → deploy)
