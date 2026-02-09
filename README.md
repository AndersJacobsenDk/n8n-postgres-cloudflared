# homeflow-lab: n8n + Cloudflare Tunnel (local-first) + Postgres

This repo contains a minimal, local-first setup for running **n8n** on a home server/NAS with:
- **Postgres** for reliable persistence
- **Cloudflare Tunnel (cloudflared)** to expose a stable public HTTPS URL **without port-forwarding**
- Bind-mounted storage for backups and portability

The original goal is to support read-only integrations (e.g., pulling public Reddit posts into a private workflow) while keeping automation logic and data local.

---

## Architecture (high level)

Traffic flow for a public URL like `https://n8n.example.com`:

1. Your browser connects to **Cloudflare** (TLS terminated at Cloudflare).
2. Cloudflare forwards the request through an **outbound-only** tunnel to `cloudflared` running on your NAS.
3. `cloudflared` forwards to the internal n8n service at `http://n8n:5678`.
4. n8n responds back through the same path.

Key properties:
- No inbound ports opened on your router.
- Your NAS public IP is not exposed via DNS.
- If `cloudflared` stops, the public URL stops working (no fallback path to your home network).

---

## Prerequisites

- Docker + Docker Compose (or Portainer Stacks)
- A domain managed in Cloudflare (or added to Cloudflare)
- Cloudflare Zero Trust enabled
- A Cloudflare Tunnel created (you will paste the tunnel token)

---

## Quick start

### 1) Create local folders for persistence

Update the paths in `docker-compose.yml` if needed. The compose uses bind mounts:

- `/local/docker/n8n/postgres_data`
- `/local/docker/n8n/n8n_data`

Make sure these directories exist and are writable by Docker.

### 2) Configure secrets and URLs

Replace placeholders in the compose file:

- `TUNNEL_TOKEN: <insert token>`
- `POSTGRES_PASSWORD: <insert password>`
- `DB_POSTGRESDB_PASSWORD: <insert password>`
- `N8N_ENCRYPTION_KEY: <insert encryptionkey>`
- `WEBHOOK_URL: <insert webhookurl>`

**Notes**
- `N8N_ENCRYPTION_KEY` should be **32+ characters** and kept stable. If you change it later, previously saved credentials may become unreadable.
- `WEBHOOK_URL` should be the public URL of your n8n instance, e.g.:
  - `https://n8n.example.com/`

### 3) Deploy with Docker Compose

```bash
docker compose up -d
