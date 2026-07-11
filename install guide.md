# n8n Local Installation Guide (Docker & npm)

A step-by-step guide for installing n8n on your own machine — free, self-hosted, no cloud account required.

---

## Before You Start: What is n8n?

n8n is a workflow automation tool (like Zapier, but self-hostable). Running it locally means:
- **Free forever** for core features (Community Edition, Sustainable Use License)
- **No trial limits** — no signup required to use it
- Your data stays on your machine, not on a third-party server
- Enterprise features (SSO, advanced permissions) require a paid license, but you won't need those for personal/team automation use

You have two installation options: **Docker** (recommended) or **npm**. Both are covered below.

---

## Option 1: Install with Docker (Recommended)

Docker keeps n8n isolated in its own container — no conflicts with other software on your system, and easy to update or remove later.

### Step 1 — Install Docker Desktop

1. Download Docker Desktop from [docker.com](https://www.docker.com/products/docker-desktop/)
2. Install it like any normal application
3. Launch Docker Desktop and make sure it's running (check the whale icon in your system tray)

### Step 2 — Choose where your data will live

You can either:
- Let Docker manage storage automatically (a **named volume**), or
- Pick a **specific folder** on your PC or an external drive (a **bind mount**) — recommended if you want easy backups or want data on a separate drive

**Option A: Named volume (Docker-managed)**
```powershell
docker volume create n8n_data
docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

**Option B: Bind mount to a specific folder (recommended)**

Replace `C:\n8n-data` with any folder path you want (can be on an external drive too, e.g. `F:\n8n`):

```powershell
docker run -d --name n8n -p 5678:5678 -v C:\n8n-data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

> ⚠️ **If using an external drive:** Go to Docker Desktop → Settings → Resources → File Sharing, and add that drive/folder to the shared list, then Apply & Restart. Also consider locking the drive letter in Windows Disk Management so it doesn't change on reconnect.

### Step 3 — Access n8n

Open your browser and go to:
```
http://localhost:5678
```

You'll be prompted to create your local **owner account** (name, email, password) — this stays on your machine.

### Step 4 — Managing the container

```powershell
docker ps              # check if n8n is running
docker stop n8n         # stop it
docker start n8n         # start it again
docker logs n8n           # view logs (useful for troubleshooting)
```

---

## Option 2: Install with npm

Good for quick local testing, or if you prefer not to install Docker.

### Step 1 — Install Node.js

Download and install **Node.js (version 20.19–24.x)** from [nodejs.org](https://nodejs.org).

### Step 2 — Install n8n globally

Open your terminal and run:
```powershell
npm install n8n -g
```

### Step 3 — Start n8n

```powershell
n8n
```

### Step 4 — Access n8n

Open your browser and go to:
```
http://localhost:5678
```

> **Note:** With npm, n8n's data is stored in a default local folder (usually `~/.n8n` on your user profile) unless you configure a custom path via environment variables. Docker's bind mount approach gives you more direct control over data location.

---

## Docker vs npm — Which Should You Use?

| | Docker | npm |
|---|---|---|
| Isolation from other apps | Yes | No — shares your Node.js environment |
| Easy updates | Pull new image, restart | Manual `npm update` and version management |
| Easy to move to another machine/server | Yes — same setup works anywhere | Less portable |
| Good for production | ✅ Yes | ❌ Not recommended |
| Good for quick local testing | Fine, but slightly heavier | ✅ Yes, simpler for one-off testing |
| Official recommendation | ✅ For anything beyond casual testing | Development/testing only |

---

## Production Recommendation: Use Docker

For any real/production deployment, **Docker is the clear choice** — this is also n8n's own official guidance. Reasons:

1. **Isolation & consistency** — no dependency conflicts, predictable behavior across machines
2. **Easier updates** — pull the latest image and restart, instead of managing Node.js versions manually
3. **Scalability** — production setups often need **queue mode** (separate worker containers + Redis) for handling higher workflow volume, which is far easier to configure with Docker
4. **Portability** — the same Docker Compose file can move from your laptop to a VPS or cloud server with minimal changes
5. **Better database support** — production deployments should use **PostgreSQL** instead of the default SQLite (better for concurrent workflow executions), and this is straightforward to add as another container in Docker Compose

### Recommended Production Stack (Docker Compose)

For anything beyond personal experimentation, don't use a single `docker run` command — use **Docker Compose** to manage:
- n8n container
- PostgreSQL database container (replacing default SQLite)
- Reverse proxy (Caddy or Nginx) for HTTPS — required if you're receiving webhooks from external services (e.g. WhatsApp, Meta, Stripe, etc.)
- Persistent volumes for data

n8n maintains official example configurations in their `n8n-hosting` GitHub repository — use those as a starting template.

---

## Quick Troubleshooting

| Problem | Likely Cause |
|---|---|
| `localhost:5678` not loading | Container not running — check `docker ps` |
| Data disappears after restart | Volume/bind mount not set correctly |
| External drive folder empty after running | Drive not shared in Docker Desktop settings |
| Container fails to start after PC restart | External drive not connected before Docker starts |
| Port 5678 already in use | Another app or n8n instance using that port — stop it or change port mapping (`-p 5679:5678`) |

---

## Summary

- **For testing/development:** npm is quick and simple
- **For anything real (production, team use, external webhooks):** Docker — ideally with Docker Compose, PostgreSQL, and HTTPS via reverse proxy
- **Free tier is genuinely free** as long as you self-host and use core (non-enterprise) features — no trial period, no expiration
