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

### 6. Gateway Authentication

OpenClaw supports **Token** (default) and **Password** authentication.

- **Token Mode**: Auto-generated during setup. Found in `~/.openclaw/openclaw.json`.
- **Password Mode**: Recommended for shared environments. Configure in `.env`:
  ```env
  OPENCLAW_GATEWAY_AUTH_MODE=password
  OPENCLAW_GATEWAY_PASSWORD=YourSecurePassword
  ```

**Precedence Rules:**

1. CLI Flag (`--auth password --password XYZ`)
2. Environment Variables (`.env`)
3. Config File (`openclaw.json`)

**Verification:**
Check logs on startup for: `[gateway] auth mode: password`.

---

## 🐳 Running with Docker (M2 Optimized)

The repository provides two ways to run OpenClaw in Docker, optimized for Apple Silicon (M2). Both methods automatically load configuration from your `.env` file.

### Choice 1: Development Mode (Hot-Reload)

Best for active coding. It bind-mounts your source code and uses `pnpm gateway:watch`.

```bash
docker compose -f docker-compose.dev.yml up
```

- **Features**: Instant reloads on file save, builds the image locally.
- **Works via**: `env_file: .env` + bind mounts to `/app`.

### Choice 2: Production Mode (Stable)

Best for testing a specific release or running in the background.

```bash
docker compose -f docker-compose.prod.yml up
```

- **Features**: Uses a pre-built static image (`openclaw:local` by default), runs the compiled `dist/index.js`.
- **Works via**: `env_file: .env` (automatically loads all settings from your `.env`).

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
- **Environment**: `.env` (Local overrides)
- **User Config**: `~/.openclaw/openclaw.json`
- **Agent Workspace**: `~/.openclaw/workspace`
