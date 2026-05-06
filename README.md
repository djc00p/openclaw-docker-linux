# 🐋 OpenClaw Docker Deployment & Infrastructure Guide

[![ClawHub Skill](https://img.shields.io/badge/ClawHub-Skill-blue)](https://clawhub.ai/djc00p/openclaw-docker-linux) [![Agent Skill](https://img.shields.io/badge/Agent-Skill-blue)](#) [![Platform: Linux](https://img.shields.io/badge/Platform-Linux-green)](#) [![Tech: Docker](https://img.shields.io/badge/Tech-Docker-blue)](#) [![Type: Infrastructure](https://img.shields.io/badge/Type-Infrastructure-orange)](#) [![Security: Secure](https://img.shields.io/badge/Security-Secure-brightgreen)](#) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**The authoritative guide for deploying OpenClaw in a containerized environment on Linux, featuring secure remote access via Tailscale.**

This repository provides the configuration, orchestration, and security protocols required to run a production-ready OpenClaw instance using Docker and Docker Compose. This setup is specifically optimized for Linux (Ubuntu 24.04+) and integrates seamlessly with Tailscale for secure, encrypted remote access without exposing ports to the public internet.

---

## ⚠️ Security & Compliance Mandate

Deploying OpenClaw involves elevated system privileges and sensitive credential management. **All engineers must review the following before proceeding:**

* **Privilege Escalation:** All Docker setup commands require `sudo` permissions. Use the provided `docker-setup.sh` only after auditing the script contents.
* **Credential Exposure:** Mounting host directories (e.s., `~/.config/gh`) into containers exposes host secrets to the container image. **Only mount directories from trusted, verified images.**
* **Network Exposure:** **NEVER** bind port `18789` to `0.0.0.0` (public interface). Always bind to `127.0.0.1` or use the Tailscale `serve` pattern to ensure the API is only accessible via your private Tailnet.
* **Image Integrity:** Use specific version tags (e.g., `ghcr.io/openclaw/openclaw:v1.2.3`) in your `docker-compose.yml`. Avoid the `:latest` tag to prevent breaking changes during container restarts.

---

## 🚀 Deployment Workflow

### 1. Host Environment Preparation

Install the Docker engine via `apt` (avoiding the Snap version to prevent permission conflicts) and configure user permissions.

```bash
# Install Docker and Docker Compose
sudo apt update && sudo apt install docker.io docker-compose -y

# Add your user to the docker group to avoid sudo on every command
sudo usermod -aG docker $USER

# IMPORTANT: You must log out and log back in for group changes to take effect
```

### 2. Initial Onboarding

Use the OpenClaw CLI to initialize the gateway and generate your unique access token.

```bash
# Run the onboarding wizard
docker-compose run --rm openclaw-cli onboard
```

### 3. Configuration & Secrets

After onboarding, create your `docker-compose.yml` and a `.env` file. Use the `.env` file to inject your `ANTHROP_API_KEY` and `OPENCLAW_GATEWAY_TOKEN`.

**Pro-Tip:** Use `openssl` to generate a high-entropy gateway token for your configuration:

```bash
openssl rand -hex 32
```

### 4. Launching the Stack

Once the configuration is complete, launch the services in detached mode.

```bash
# Start the OpenClaw stack
docker-compose up -d

# Verify the container is running
docker-compose ps
```

---

## 🏗️ Architecture & Networking

### Tailscale Remote Access (The "Secure Entry" Pattern)

We recommend running Tailscale on the **host machine**, not inside the container. This allows you to use `tailscale serve` to wrap your local Docker port in a secure, HTTPS-encrypted tunnel.

**To enable remote access via your Tailnet:**

```bash
# Setup Tailscale on the host
sudo apt install tailscale
sudo tailscale up

# Securely expose the OpenClaw port via Tailscale HTTPS
sudo tailscale serve 18789
```

You can now access your agent securely at:
`https://<your-machine-name>.<your-tailnet>.ts.net?token=YOUR_TOKEN`

### Volume Mapping & Pathing

The container uses a "Mirror Mapping" strategy. The host's configuration directory is mounted into the container to ensure persistence.

* **Host Path:** `~/.openclaw/`
* **Container Path:** `/home/node/.openclaw/`

*Note: If you encounter permission errors, ensure the host directory is owned by the Docker user (UID 1000):*
`sudo chown -R 1000:1000 ~/.openclaw`

---

## 🛠️ Operations & Maintenance

| Task | Command |
| :--- | :--- |
| **View Live Logs** | `docker-compose logs -f openclaw` |
| **Execute CLI Command** | `docker-compose run --rm openclaw-cli <COMMAND>` |
| **Approve Telegram Bot** | `docker-compose run --rm openclaw-cli pairing approve telegram <CODE>` |
| **Full System Reset** | `docker-compose down -v` (Warning: Deletes all volumes) |
| **Check System Health** | `openclaw doctor` |

---

**Standardized by the OpenClaw DevOps Team.**
