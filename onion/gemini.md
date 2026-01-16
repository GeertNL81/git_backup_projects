# .onion Hosting Project (Unified)

This project hosts a Tor Hidden Service (Onion Service v3) using a single, unified Podman container.

## Architecture
- **Image:** `debian:stable-slim`
- **Service Name:** `tor-web`
- **Mechanism:** Both **Nginx** and **Tor** run within the same container. Tor routes traffic to Nginx via `127.0.0.1:80`, ensuring the web server is completely isolated from the public internet.

## Directory Structure
- `/data/projects/onion/compose.yaml`: The unified orchestration file.
- `/data/projects/onion/tor/hs_data/`: **CRITICAL** Persistent storage for Tor keys and hostname. 
- `/data/projects/onion/nginx/html/`: Your website content (mounted to `/data/www` in the container).

## Managing the Service
Start/Update:
```bash
cd /data/projects/onion
podman-compose up -d
```

Stop:
```bash
podman-compose down
```

View Logs:
```bash
podman logs -f tor-web
```

## Your .onion Address
Current Address: **mxhtaguhmq75zceipslvu6brq55kuygeqqizgotzp5w77dhsriybi7yd.onion**

To retrieve it manually:
```bash
sudo cat /data/projects/onion/tor/hs_data/hostname
```

## Security & Maintenance
- **Persistence:** The `hs_data` folder contains your identity keys. Backup this folder if you want to keep your .onion address.
- **Permissions:** The container automatically manages permissions for the `tor` user inside, but the host directory uses `:Z` for Podman/SELinux compatibility.
