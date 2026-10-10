# Deploy ATEMS — Step-by-step

**Run ATEMS fully on the server (192.0.2.22).** See **RUN_ON_SERVER.md** for the primary flow.

**Use Nginx as reverse proxy** — See **NGINX_DEPLOYMENT.md** for the full deployment flow.

---

## Prerequisites

- DNS: **atems.example.com** points to the server IP (or use IP for testing).
- SSH access to the server as **ansible** (or adjust paths in Nginx + systemd).

---

## Option A: Deploy from your Mint machine (build + sync)

1. **Build and sync** (from ATEMS repo root):
   ```bash
   cd /home/operator/ATEMS
   chmod +x scripts/deploy-to-server.sh
   ./scripts/deploy-to-server.sh operator@192.0.2.22
   ```
   This builds the frontend (if present), rsyncs to `~/atems/` on the server, and prints the next steps.
   It stops with a non-zero exit code if `npm` is missing, the frontend build fails, or `rsync` fails. Set `SKIP_FRONTEND_BUILD=1` to deploy without building the frontend.

2. **On the server** — SSH in and follow the one-time steps in Option B from step 3 onward.

---

## Option B: Deploy entirely on the server (clone + setup there)

1. **SSH to the server:**
   ```bash
   ssh operator@192.0.2.22
   ```

2. **Follow RUN_ON_SERVER.md** for the full one-time setup (clone, venv, .env, db, frontend, systemd, Nginx).

---

## After first deploy — Updating

**If you use Option A (sync from Mint):**
- Run again: `./scripts/deploy-to-server.sh operator@192.0.2.22`
- On server: `sudo systemctl restart atems`

**If you use Option B (git on server):**
```bash
ssh operator@192.0.2.22
cd ~/atems
git pull
source .venv/bin/activate
pip install -r requirements.txt
export FLASK_APP=atems:create_app
flask db upgrade
./scripts/build_on_server.sh --restart
```

---

## Quick checklist (Nginx path)

| Step               | Command / action |
|--------------------|------------------|
| Sync to server     | `./scripts/deploy-to-server.sh operator@192.0.2.22` (from ATEMS root) |
| One-time: .env     | Copy .env.example to .env, set SECRET_KEY, DB URI, DEBUG=False, ADMIN_PASSWORD |
| One-time: DB       | On server: `flask db upgrade` |
| One-time: systemd  | Copy `atems.service` → /etc/systemd/system/, daemon-reload, enable, start |
| One-time: Nginx    | Copy `web-sites-server/nginx-atems.conf` → /etc/nginx/sites-available/atems, link to sites-enabled, test, reload |
| One-time: HTTPS    | Run `sudo certbot --nginx -d atems.example.com` |
| One-time: DNS      | Cloudflare A record atems.example.com → 192.0.2.22 |
| Update             | `sudo systemctl restart atems` |
