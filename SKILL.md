---
name: openclaw-docker-setup
description: "Run OpenClaw inside Docker on Linux with Tailscale remote access. Complete setup guide covering installation, configuration, and critical gotchas. Trigger phrases: docker openclaw, openclaw in docker, setup openclaw docker, tailscale openclaw, docker-compose openclaw."
metadata: {"clawdbot":{"emoji":"🐋","requires":{"bins":["docker","docker-compose"]},"env":["ANTHROPIC_API_KEY","OPENCLAW_GATEWAY_TOKEN"],"os":["linux"],"homepage":"https://clawhub.com/djc00p/openclaw-docker-linux"}}
---

# OpenClaw Docker Setup

Run OpenClaw inside Docker on Linux (Ubuntu 24.04+) with Tailscale for remote access.

## Quick Start

1. **Install Docker via APT** (not Snap):
   ```bash
   sudo apt install docker.io docker-compose && \
   sudo usermod -aG docker $USER
   ```
   Then log out and back in — `sudo usermod` doesn't take effect with `newgrp`.

2. **Run onboard** to configure gateway and get your token:
   ```bash
   docker-compose run --rm openclaw-cli onboard
   ```

3. **Create `docker-compose.yml`** using the token from onboard.
   See `references/docker-config.md` for the full template and .env setup.

4. **Start the container:**
   ```bash
   docker-compose up -d
   ```
   Access at `http://localhost:18789?token=YOUR_TOKEN`

## Key Concepts

- **bind: lan vs loopback** — `lan` = accessible from the host via port mapping; `loopback` = locked inside container.
- **Tailscale on host, not container** — Run Tailscale on the Ubuntu host for remote access.
- **One method only** — Docker OR global install, never both (port + config conflicts).
- **Config path mapping** — Host `~/.openclaw/` → Container `/home/node/.openclaw/` (same files, different paths).
- **Docker group login** — `sudo usermod -aG docker` requires full logout/login, not `newgrp`.

## Common Usage

**Generate a secure token:**
```bash
openssl rand -hex 32
```

**View container logs:**
```bash
docker-compose logs -f openclaw
```

**Run CLI commands inside container:**
```bash
docker-compose run --rm openclaw-cli COMMAND_HERE
```

**Fix volume permissions (Linux):**
```bash
sudo chown -R 1000:1000 ~/.openclaw ~/openclaw
```

**Approve Telegram pairing:**
```bash
docker-compose run --rm openclaw-cli pairing approve telegram YOUR_CODE
```

**Access via Tailscale (recommended — HTTPS):**
```bash
sudo tailscale up
./docker-setup.sh tailscale  # Starts tailscale serve on port 18789
```
Then visit `https://YOUR_MACHINE_NAME.YOUR_TAILNET.ts.net?token=YOUR_TOKEN` from any device on your tailnet. Use MagicDNS hostname over raw IP — it's HTTPS by default and more stable.

## References

- `references/docker-config.md` — docker-compose.yml, .env template, permissions, Tailscale, management script
- `references/quickstart.md` — Simple 5-minute setup guide
- `references/docker-setup.sh` — Management script (start/stop/logs/doctor/tailscale/approve_telegram)
- `references/gotchas.md` — Critical mistakes and how to avoid them
- `references/troubleshooting.md` — Common errors and fixes
