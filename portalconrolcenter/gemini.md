# Project X - Handover Documentation

This document summarizes the final working state of the Project X Homelab Portal.

## Project Overview
A **Homelab Management Portal** built with **Next.js 16 (App Router)** and **Tailwind CSS**. 
The portal centralizes access to host services like **Web SSH**, **Cockpit**, **n8n**, and **Host Desktop**, exposing them securely to the internet via **Tailscale Funnel**.

## Infrastructure & Environment
- **Host OS:** Linux (Fedora/similar)
- **Container Engine:** Podman (rootless) managed via `podman-compose` and systemd.
- **Networking:** 
  - **Public Access:** `https://homelab.tail7529d7.ts.net/` via Tailscale Funnel.
  - **SSL Termination:** Handled by Tailscale Funnel; traffic reaches the host as unencrypted HTTP on port 80.
  - **Reverse Proxy:** Traefik v3 (`traefik-infra`) managed by a user-level systemd service. Port 443 is NOT used by Traefik (removed to avoid conflict with Tailscale).
- **Security:** Basic Authentication globally enforced (User: `geert`, Password: `ResetRoot`).
- **Global Settings:** 
  - **Timezone:** `Europe/Amsterdam` (set via `TZ` env in all containers).
  - **n8n Timezone:** `GENERIC_TIMEZONE=Europe/Amsterdam` also set for n8n scheduling.
  - **Hostnames:** Explicitly set to match container names inside all containers.

## Infrastructure Services (managed via systemd --user)
1.  **Traefik Proxy:** `container-traefik-infra.service`
2.  **MCP Hub (Tools):** `container-mcp-infra.service`
3.  **Database:** `container-project-x-db.service`
4.  **Dashboard (Portal):** `container-project-x-web.service`
5.  **n8n (Automation):** `container-project-x-n8n.service`
6.  **NoVNC Bridge:** `container-project-x-novnc.service` (Web interface for VNC)
7.  **Host VNC Server:** `host-cosmic-vnc.service` (Captures host Cosmic Desktop)

## Desktop (NoVNC) Access
- **Service:** Access via the "Desktop" tile on the dashboard.
- **VNC Password:** `password` (configured in `~/.config/wayvnc/config`).
- **Input Limitation:** Currently running with `--disable-input` because the Cosmic Compositor (Wayland) does not yet support the virtual pointer protocol required by `wayvnc` for remote input. It is currently "view-only".

## Application Management
Services are now fully managed by systemd user units for persistence.
- **Restart All:** `systemctl --user restart container-project-x-*`
- **View Logs:** `journalctl --user -u container-project-x-web.service -f`
- **Legacy:** `podman-compose` is no longer used for service management but remains in the project directory for reference.

## Key Files & Paths
- **Project Root:** `/data/Podman/project-x`
- **MCP Hub Root:** `/data/Podman/mcp-hub`
- **Traefik Dynamic Config:** `/data/Podman/traefik/dynamic/cockpit.yml`
- **VNC Configuration:** `~/.config/wayvnc/config`
- **Systemd Units:** `~/.config/systemd/user/container-{traefik,mcp,project-x-*,novnc}.service` and `host-cosmic-vnc.service`.

## MCP Registration Commands
Run these locally to enable tools in Gemini CLI:
```bash
# Postgres
gemini mcp add postgres --scope user podman exec -i -e POSTGRES_URL="postgresql://user:password@project-x-db:5432/project_x" mcp-infra postgres-mcp-server stdio

# Chrome
gemini mcp add chrome --scope user podman exec -i mcp-infra npx chrome-devtools-mcp --browserUrl http://127.0.0.1:9222
```

## Common Commands
- **Restart Entire App:** `cd /data/Podman/project-x && podman-compose up -d --build`
- **Check Funnel Status:** `sudo tailscale funnel status`

## Status & Troubleshooting (Updated 2026-01-15)
- **502 Bad Gateway:** Resolved by removing port 443 binding from Traefik and switching all internal routers to the `web` (port 80) entrypoint.
- **n8n White Background:** Fixed by setting `N8N_PATH=/n8n/` and using Traefik `StripPrefix("/n8n")` middleware. This ensures n8n generates correct internal links while correctly serving static assets from its internal root.
- **Dashboard Desktop Feature:** Added a NoVNC bridge to access the host's Cosmic Desktop. Verified connectivity.
- **MCP Connectivity:** Verified working with standard execution.
- **Traefik ForwardAuth Fix:** Resolved 500 errors during authentication by updating the `forwardAuth` middleware address in `/data/Podman/traefik/dynamic/auth.yml` to use the explicit internal IP (`10.89.1.3`) instead of the container hostname, as Traefik was failing to resolve `project-x-web` across networks.
