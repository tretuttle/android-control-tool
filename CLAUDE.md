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
Core Tool Layer (adb, fastboot, jadx, androguard, gradle, sdk build tools, etc.)
    |
    v
Android Device(s)
```

## The Two Halves

The power of this server comes from combining two sides of the Android world that are rarely unified:

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

### Half 2: Android Development & Build Tools
Everything needed to create, build, and ship apps:
- **Gradle** — build, compile, package
- **APK signing** — keystores, zipalign, apksigner
- **AAPT2** — resource compilation, manifest processing
- **D8/R8** — dex compilation, code shrinking/obfuscation
- **Android SDK Build Tools** — full compile pipeline
- **Emulator management** — AVD creation, snapshots, headless instances
- **Lint / static analysis** — code quality checks

### Bonus Half (Discussed, Strong Addition): UI Automation
The ability for the LLM to actually SEE and INTERACT with the device:
- `adb shell screencap` — capture screen
- `adb shell uiautomator dump` — structured view hierarchy (XML)
- `adb shell input tap/swipe/text` — touch, gesture, type
- **scrcpy** — real-time screen mirroring
- This enables autonomous app navigation, testing, and verification

## The Pipeline: Why Both Halves Together Matter

RE is the input. Development is the output. One without the other is incomplete.

**The killer workflow — abandoned app resurrection:**
1. Take an abandoned APK nobody maintains anymore
2. Decompile it — map activities, services, permissions, data flows
3. Understand the architecture — screens, APIs, data storage patterns
4. Generate clean, modern, open-source code replicating or improving the functionality
5. Build and sign it into a working APK
6. Deploy to device and validate

This turns "abandoned app revival" from a weeks-long manual RE project into something approachable. But it's not just revival:
- **Open-source replacements** of closed/abandoned apps that the community still needs
- **Inspiration for new projects** — see how someone solved a problem, learn from it, seed something new
- **Security auditing** — full analysis through to patched rebuild
- **Education** — understand real-world app architecture by decompiling and studying

## The Ingredient Analogy

Cheese and tomato go on pizza, but also in spaghetti and lasagna. The underlying tools (adb, fastboot, androguard, jadx, gradle, etc.) are the ingredients. The current PyQt6 GUI is one recipe. Our MCP server is a completely different dish using the same ingredients. We don't need to "extract" code from main.py — we reference it to see how someone else combined the ingredients, then build our own thing from scratch.

## Architecture Notes

- **Strip the PyQt6 UI entirely** — we don't need it
- **Replace QProcess with asyncio.subprocess** — for async command execution
- **Replace QSettings** — use simple config files or environment variables
- **Replace signals/slots** — use standard async patterns
- **The existing main.py is reference only** — shows how tools are invoked, what flags matter, what edge cases exist
- **Build the MCP server fresh** using the same underlying tool binaries

## Tools Already In This Repo (Auto-Installers Exist)
- Android SDK Platform-Tools (adb, fastboot, aapt) — auto-downloads from Google
- scrcpy — auto-downloads from GitHub releases
- JADX — auto-downloads from GitHub releases
- BusyBox — auto-downloads per device architecture
- payload-dumper-go — on-demand download for OTA extraction

## Tools To Add (The Build/Development Side)
- Gradle wrapper / build system integration
- Android SDK Build Tools (aapt2, d8, apksigner, zipalign)
- Keystore generation and management
- Emulator (AVD) management
- Lint and static analysis tooling

## Status
- [ ] MCP server architecture designed
- [ ] Core tool layer abstracted (async wrappers around adb, fastboot, etc.)
- [ ] RE/Security tools exposed as MCP tools
- [ ] Build/Development tools integrated and exposed
- [ ] UI Automation tools exposed
- [ ] End-to-end pipeline tested (decompile -> understand -> rebuild -> deploy)
