# Infrastructure Documentation - Project X Homelab

## Infrastructure & Environment
- **Host OS:** Linux (Fedora/similar)
- **Container Engine:** Podman (rootless) managed via systemd user units.
- **Networking:** 
  - **Public Access:** `https://homelab.tail7529d7.ts.net/` via Tailscale Funnel.
  - **SSL Termination:** Handled by Tailscale Funnel; traffic reaches the host as unencrypted HTTP on port 80.
  - **Reverse Proxy:** Traefik v3 (`traefik-infra`) managed by a user-level systemd service. Port 443 is NOT used by Traefik.
- **Security:** Basic Authentication globally enforced.
- **Global Settings:** 
  - **Timezone:** `Europe/Amsterdam` (set via `TZ` env in all containers).
  - **Hostnames:** Explicitly set to match container names inside all containers.

## Infrastructure Services (managed via systemd --user)
1.  **Traefik Proxy:** `container-traefik-infra.service`
2.  **MCP Hub (Tools):** `container-mcp-infra.service`
3.  **Database:** `container-project-x-db.service`
4.  **n8n (Automation):** `container-project-x-n8n.service`
5.  **NoVNC Bridge:** `container-project-x-novnc.service` (Web interface for VNC)
6.  **Host VNC Server:** `host-cosmic-vnc.service` (Captures host Cosmic Desktop)

## Desktop (NoVNC) Access
- **Service:** Host desktop access via VNC.
- **VNC Password:** `password` (configured in `~/.config/wayvnc/config`).
- **Input Limitation:** Currently "view-only" due to Wayland/Cosmic Compositor virtual pointer limitations.

## Key Files & Paths
- **Project Root:** `/data/Podman/project-x`
- **MCP Hub Root:** `/data/Podman/mcp-hub`
- **Traefik Dynamic Config:** `/data/Podman/traefik/dynamic/cockpit.yml`
- **VNC Configuration:** `~/.config/wayvnc/config`
- **Systemd Units:** `~/.config/systemd/user/container-{traefik,mcp,project-x-*,novnc}.service` and `host-cosmic-vnc.service`.

## Common Commands
- **Check Funnel Status:** `sudo tailscale funnel status`
- **Restart All Services:** `systemctl --user restart container-project-x-* container-traefik-infra.service container-mcp-infra.service`

## Networking & Proxy Troubleshooting
- **502 Bad Gateway:** Resolved by removing port 443 binding from Traefik and switching all internal routers to the `web` (port 80) entrypoint.
- **Traefik ForwardAuth Fix:** Resolved 500 errors by updating `forwardAuth` middleware to use the explicit internal IP (`10.89.1.3`) instead of container hostnames for resolution across networks.
- **n8n Proxying:** Fixed path issues using `N8N_PATH=/n8n/` and Traefik `StripPrefix("/n8n")`.