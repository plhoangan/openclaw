# Local Development Setup for OpenClaw (macOS M2)

This guide summarizes how to set up and maintain a local development environment for OpenClaw on a Macbook Air M2.

## 📋 Prerequisites

- **Node.js**: ≥ 22.12.0
- **pnpm**: Version 10+ (preferred package manager)
- **Docker**: Optional, for containerized development

---

## 🚀 Native Development Setup

To run OpenClaw directly on your Mac, follow these steps in order:

### 1. Install Dependencies

```bash
pnpm install
```

### 2. Build Assets (Backend & Frontend)

OpenClaw consists of a TypeScript backend and a Vite-based "Control UI". You must build both:

```bash
pnpm ui:build   # Builds the Control UI (React/Vite) into dist/control-ui
pnpm build      # Builds the core TypeScript engine into dist/
```

### 3. Onboarding

Guide the assistant through model setup and storage configuration:

```bash
pnpm openclaw onboard --install-daemon
```

### 4. Running the Gateway

- **Single Run**: `pnpm openclaw gateway`
- **Development Mode (Watch)**: `pnpm gateway:watch` (Auto-reloads on TypeScript changes)

### 5. Force Stop & Reset Processes

If OpenClaw hangs or you need to clear all running instances:

```bash
# Option A: Clean stop (standard)
pnpm openclaw gateway stop

# Option B: Kill everything (for stuck/orphaned processes)
pkill -f openclaw

# Option C: Find and kill by port (the nuclear option)
lsof -i :18789
kill -9 <PID>
```

---

## 🐳 Docker Development (M2 Optimized)

The repository includes a development-specific Docker Compose file optimized for Apple Silicon:

```bash
docker compose -f docker-compose.dev.yml up
```

- **Platform**: Uses `linux/arm64` for maximum performance on M1/M2 chips.
- **Port Mapping**: Gateway defaults to `http://localhost:18789`.
- **Hot-Reload**: Bind-mounts the source code for immediate updates.

---

## 🛠 Troubleshooting Common Issues

### "Control UI assets not found"

If you see this error when accessing the dashboard:

1. Ensure you have run `pnpm ui:build`.
2. Check that `dist/control-ui/index.html` exists.
3. If the error persists, perform a clean build: `pnpm build && pnpm ui:build`.

### Race Condition: "missing dist/entry.js"

When running `pnpm gateway:watch`, you might see an error saying the build output is missing. This is usually a race condition where the runtime tries to start while the compiler is still cleaning the `dist` folder.

**Fix:** Ensure the compiler uses the `--no-clean` flag. The current `scripts/watch-node.mjs` is patched to include this by default. If manually running:

```bash
# Run a clean build once FIRST
pnpm build
# Then run watch mode
pnpm gateway:watch
```

### Conflicts with the Gateway Daemon

If you installed the gateway as a background service during onboarding, it may block ports (18789) when you try to run from source.

- **Stop the daemon**: `pnpm openclaw gateway stop`
- **Check for orphaned processes**: `lsof -i :18789` or `ps aux | grep openclaw`
- **Kill if necessary**: `kill -9 <PID>`

---

## 📂 Key File Locations

- **Source Code**: `src/` (Backend), `ui/src/` (Control UI)
- **Built Files**: `dist/`
- **User Config**: `~/.openclaw/openclaw.json`
- **Agent Workspace**: `~/.openclaw/workspace`
