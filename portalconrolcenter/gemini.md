# Project X - Handover Documentation

This document summarizes the final working state of the Project X Homelab Portal.

## Project Overview
A **Homelab Management Portal** built with **Next.js 16 (App Router)** and **Tailwind CSS**. 
The portal centralizes access to host services like **Web SSH**, **Cockpit**, and **n8n**, exposing them securely to the internet via **Tailscale Funnel**.

## Infrastructure & Environment
- **Host OS:** Linux (Fedora/similar)
- **Container Engine:** Podman (rootless) managed via `podman-compose`.
- **Networking:** 
  - **Public Access:** `https://homelab.tail7529d7.ts.net/` via Tailscale Funnel.
  - **SSL Termination:** Handled by Tailscale Funnel; traffic reaches the host as unencrypted HTTP on port 80.
  - **Reverse Proxy:** Traefik v3 (`traefik-infra`).
- **Security:** 
  - **Global Dashboard:** Basic Authentication (User: `geert`, Password: `ResetRoot`).
  - **Internal SSH:** Passwordless ED25519 keys for container-to-host access.
- **Global Settings:** 
  - **Timezone:** `Europe/Amsterdam` (set via `TZ` env in all containers).
  - **n8n Timezone:** `GENERIC_TIMEZONE=Europe/Amsterdam` also set for n8n scheduling.

## Infrastructure Services (managed via podman-compose)
1.  **Traefik Proxy:** `traefik-infra`
2.  **Database:** `project-x-db`
3.  **Dashboard (Portal):** `project-x-web` (Custom Next.js app)
4.  **n8n (Automation):** `project-x-n8n`

## Live Resource Monitor (btop)
- **Service:** Embedded `btop` stream on the dashboard dashboard.
- **Implementation:** `ttyd` running inside `project-x-web` on port 7682, connecting to the host via SSH.
- **Authentication:** Uses ED25519 private key mounted at `/root/.ssh/id_ed25519`.
- **Dashboard UI:** Tile height increased to `h-[800px]` to satisfy `btop` minimum terminal size requirements.

## Key Files & Paths
- **Project Root:** `/data/Podman/project-x`
- **Traefik Dynamic Config:** `/data/Podman/traefik/dynamic/`
- **SSH Keys:** `/data/Podman/project-x/ssh_keys/` (Mounted into web container)
- **Compose Config:** `/data/Podman/project-x/docker-compose.yml`

## Common Commands
- **Restart All:** `cd /data/Podman/project-x && podman-compose up -d --build`
- **View Logs:** `podman logs -f project-x-web`

## Status & Troubleshooting (Updated 2026-01-18)
- **Desktop Removal:** The Webtop container (`project-x-desktop`) and the Garuda VM (`garuda-jumpbox`) have been removed as requested.
- **btop Black Screen fix:** Resolved by enabling passwordless SSH access. Generated ED25519 keys, authorized them on the host, and mounted them into the container with the `:Z` flag to handle SELinux permissions.
- **n8n Auth Loop fix:** Removed Traefik Basic Auth from the n8n route. n8n now uses its own internal authentication to avoid conflicts with the dashboard login.
- **ForwardAuth 500 fix:** Switched `auth.yml` to use the container hostname `http://project-x-web:3000` instead of a hardcoded IP, ensuring authentication remains stable after container recreations.
- **Terminal Size fix:** Increased the `btop` iframe container height to 800px to ensure the interface has enough space to render (minimum 24 rows).
