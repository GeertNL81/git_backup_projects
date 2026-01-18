# Project X - Handover Documentation

This document summarizes the final working state of the Project X Homelab Portal.

## Project Overview
A **Homelab Management Portal** built with **Next.js 16 (App Router)** and **Tailwind CSS**. 
The portal centralizes access to host services like **Web SSH**, **Cockpit**, and **n8n**, exposing them securely to the internet via **Tailscale Funnel**.

## Infrastructure & Environment
- **Host OS:** Linux (Fedora/similar)
- **Container Engine:** Podman (rootless) managed via `podman-compose`.
- **Virtualization:** Libvirt (`qemu:///session`) for user-level VMs.
- **Networking:** 
  - **Public Access:** `https://homelab.tail7529d7.ts.net/` via Tailscale Funnel.
  - **SSL Termination:** Handled by Tailscale Funnel; traffic reaches the host as unencrypted HTTP on port 80.
  - **Reverse Proxy:** Traefik v3 (`traefik-infra`).
- **Security:** 
  - **Global Dashboard:** Basic Authentication (User: `geert`, Password: `ResetRoot`).
  - **Internal SSH:** Passwordless ED25519 keys for container-to-host access.

## Infrastructure Services (managed via podman-compose)
1.  **Traefik Proxy:** `traefik-infra`
2.  **Database:** `project-x-db`
3.  **Dashboard (Portal):** `project-x-web` (Custom Next.js app)
4.  **n8n (Automation):** `project-x-n8n`
5.  **NoVNC Bridge:** `project-x-novnc` (Connects to Garuda VM VNC)

## Virtual Machines (managed via virsh)
1.  **Garuda:** 
    - **Specs:** 4 vCPUs, 4GB RAM, 40GB Disk (`/data/vdi/Garuda.qcow2`).
    - **Access:** Accessible via the "Garuda" button on the dashboard.
    - **VNC Port:** 5900 (bridged via NoVNC).

## Live Resource Monitor (btop)
- **Service:** Embedded `btop` stream on the dashboard.
- **Implementation:** `ttyd` running inside `project-x-web` on port 7682, connecting to the host via SSH.
- **Authentication:** Uses ED25519 private key mounted at `/root/.ssh/id_ed25519`.
- **Dashboard UI:** Tile height increased to `h-[800px]` to satisfy `btop` minimum terminal size requirements.

## Key Files & Paths
- **Project Root:** `/data/Podman/project-x`
- **VM Storage:** `/data/vdi/`
- **SSH Keys:** `/data/Podman/project-x/ssh_keys/` (Mounted into web container)
- **Compose Config:** `/data/Podman/project-x/docker-compose.yml`

## Common Commands
- **Restart All:** `cd /data/Podman/project-x && podman-compose up -d --build`
- **Manage VM:** `virsh --connect qemu:///session {start|stop|destroy} Garuda`

## Status & Troubleshooting (Updated 2026-01-18)
- **Garuda VNC fix:** Successfully resolved the "Loading" and "JavaScript Error" issues. 
    - **Solution:** Switched to the lightweight `vnc_lite.html` interface.
    - **WebSocket Path:** Configured the connection URL with `path=desktop` to match Traefik prefix stripping.
    - **Backend IP:** Pointed the bridge to the explicit host gateway `169.254.1.2:5900`.
- **btop fix:** Enabled passwordless SSH and increased dashboard tile height to 800px.
- **n8n fix:** Removed Traefik auth from n8n route to avoid login loops.
- **ForwardAuth fix:** Switched to container hostname `http://project-x-web:3000` for stability.
