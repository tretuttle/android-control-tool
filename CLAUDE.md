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

**The Stack (current as of Feb 2026):**

```
Monorepo (Turborepo + pnpm)
├── apps/
│   ├── expo/          — React Native + Expo SDK 54, RN 0.81, Expo Router, NativeWind v5
│   └── nextjs/        — Next.js 16, React 19, Tailwind CSS v4 (optional web companion)
├── packages/
│   ├── api/           — tRPC v11 router definitions
│   ├── auth/          — Better Auth (replaced Auth.js/NextAuth — Auth.js is now part of Better Auth)
│   ├── db/            — Drizzle ORM + Supabase (edge-ready, replaced PlanetScale)
│   ├── ui/            — shadcn-ui components
│   ├── validators/    — Zod schemas (shared validation)
│   ├── eslint/        — shared lint presets
│   ├── prettier/      — shared formatting
│   ├── tailwind/      — shared theme/config
│   └── typescript/    — shared tsconfig
```

**Key tech decisions:**
- **Better Auth** over NextAuth/Auth.js — native Expo support, no proxy server needed, plugin ecosystem
- **Drizzle + Supabase** — type-safe DB with edge-ready Postgres
- **tRPC v11** — end-to-end type-safe API between all apps and packages
- **React 19** across everything
- **TanStack Query + TanStack Form** — data fetching and form management
- **NativeWind v5** — Tailwind CSS v4 in React Native
- **Expo Router** — file-based routing for the mobile app
- **pnpm workspaces** — monorepo package management
- **Turborepo** — build orchestration, `pnpm turbo gen init` to scaffold new packages

**Build/deploy tools:**
- **Expo CLI / EAS CLI** — local dev builds, cloud production builds
- **pnpm** — workspace-aware dependency management
- **Turborepo** — parallel builds, caching, task orchestration

### Bonus Half (Strong Addition): UI Automation / Device Interaction
The ability for the LLM to actually SEE and INTERACT with the device:
- `adb shell screencap` — capture screen
- `adb shell uiautomator dump` — structured view hierarchy (XML)
- `adb shell input tap/swipe/text` — touch, gesture, type
- **scrcpy** — real-time screen mirroring
- This enables autonomous app navigation, testing, and verification
- Pairs with BOTH halves: drive UI to trigger hooked methods (RE), verify built apps work (dev)

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
- tRPC v11 — type-safe API layer
- NativeWind / Tailwind — styling

## Status
- [ ] MCP server architecture designed
- [ ] Core tool layer abstracted (async wrappers around adb, fastboot, etc.)
- [ ] RE/Security tools exposed as MCP tools
- [ ] T3 Turbo scaffolding and build tools integrated
- [ ] UI Automation tools exposed
- [ ] End-to-end pipeline tested (decompile → understand → scaffold → build → deploy)
