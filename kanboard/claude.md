# Kanboard Project Management Deployment

## Overview

Kanboard is a lightweight, open-source project management tool deployed as part of the Project-X infrastructure. It was chosen as an alternative to Plane for its minimal resource footprint and simplicity, making it ideal for small teams (1-10 people).

## Deployment Information

**Deployment Date**: 2026-01-22
**Deployment Method**: Podman Compose (VDI Infrastructure)
**Status**: Running and Healthy

### Container Details

- **Image**: `docker.io/kanboard/kanboard:latest`
- **Container Name**: `kanboard`
- **Management**: `/data/vdi/podman-compose.yml`
- **Container ID**: `4712f5cdb6fc`
- **Restart Policy**: `always`

### Network & Access

#### Ports
- **HTTP Port**: `8081` (mapped to container port 80)

#### Access URLs

**Local Network**:
- http://project-x:8081
- http://localhost:8081
- http://192.168.2.16:8081

**Tailscale Network** (accessible from any Tailscale-connected device):
- http://project-x.tail7529d7.ts.net:8081
- http://100.108.66.67:8081

### Storage

**Volumes**:
- `vdi_kanboard_data` → `/var/www/app/data` (SQLite database, user data, settings)
- `vdi_kanboard_plugins` → `/var/www/app/plugins` (plugins directory)

**Database**: Built-in SQLite (no external database required)

## Authentication

### Default Credentials

**CRITICAL - Must Change After First Login**:
- Username: `admin`
- Password: `admin`

### First-Time Setup

1. Access Kanboard at http://project-x.tail7529d7.ts.net:8081
2. Log in with admin/admin
3. Navigate to profile settings and change password immediately
4. Create user accounts for team members
5. Configure project boards

## Features

Kanboard provides the following features:

- **Visual Kanban Board**: Drag-and-drop task management
- **Swimlanes**: Organize tasks by category, user, or priority
- **Task Management**: Create, assign, and track tasks
- **Time Tracking**: Log time spent on tasks
- **Subtasks**: Break down complex tasks
- **File Attachments**: Attach files to tasks
- **Comments**: Collaborate on tasks with team comments
- **Custom Columns**: Configure workflow stages
- **Notifications**: Email and web notifications
- **Calendar View**: View tasks by due date
- **Search & Filtering**: Find tasks quickly

## Resource Usage

Kanboard is extremely lightweight compared to alternatives:

- **RAM Usage**: <512MB (vs 4GB+ for Plane)
- **Disk Space**: ~100MB total
- **Container Count**: 1 (vs 10-12 for Plane)
- **Database**: Built-in SQLite (no external DB service needed)

## Management Commands

### View Container Status
```bash
sudo podman ps | grep kanboard
```

### View Logs
```bash
sudo podman logs kanboard
# OR via compose
cd /data/vdi
sudo podman-compose logs -f kanboard
```

### Restart Container
```bash
cd /data/vdi
sudo podman-compose restart kanboard
```

### Stop/Start Container
```bash
cd /data/vdi
sudo podman-compose stop kanboard
sudo podman-compose start kanboard
```

### Access Container Shell
```bash
sudo podman exec -it kanboard sh
```

## Backup & Restore

### Backup Kanboard Data

```bash
# Create timestamped backup
sudo podman run --rm \
  -v vdi_kanboard_data:/data \
  -v /data/backups:/backup \
  alpine tar czf /backup/kanboard_data_$(date +%Y%m%d).tar.gz -C /data .

# Verify backup
ls -lh /data/backups/kanboard_data_*.tar.gz
```

### Restore Kanboard Data

```bash
# Stop Kanboard container
cd /data/vdi
sudo podman-compose stop kanboard

# Restore from backup
sudo podman run --rm \
  -v vdi_kanboard_data:/data \
  -v /data/backups:/backup \
  alpine sh -c "cd /data && tar xzf /backup/kanboard_data_YYYYMMDD.tar.gz"

# Start Kanboard container
sudo podman-compose start kanboard
```

## Configuration Files

### Main Configuration
- **Compose File**: `/data/vdi/podman-compose.yml`
- **Environment**: `/data/vdi/.env`
- **SSH Banner**: `/home/geert/.config/fastfetch/config.jsonc`

### Environment Variables (.env)
```bash
KANBOARD_PORT=8081
TIMEZONE=America/New_York
```

## Integration Possibilities

### GitLab Integration
Kanboard supports GitLab webhooks for issue synchronization:
- Link commits to Kanboard tasks
- Create tasks from GitLab issues
- Bidirectional updates

**Configuration**: Settings → Integrations → GitLab

### n8n Integration
Automate Kanboard workflows using n8n:
- Create tasks from external triggers
- Send notifications on task updates
- Sync with other tools

**API Endpoint**: http://project-x:8081/jsonrpc.php

### API Access
Kanboard provides a JSON-RPC API:
- API Documentation: https://docs.kanboard.org/en/latest/api/
- Endpoint: http://project-x.tail7529d7.ts.net:8081/jsonrpc.php
- Authentication: API Token (generated in user profile)

## Why Kanboard Over Plane?

### Decision Rationale

Kanboard was chosen over Plane for the following reasons:

1. **Resource Efficiency**
   - Plane requires 10-12 containers and 4GB+ RAM
   - Kanboard uses 1 container and <512MB RAM
   - Current infrastructure has limited headroom

2. **Team Size**
   - 3-person startup team
   - Plane is designed for 10+ person teams
   - Kanboard is optimized for 1-10 person teams

3. **Simplicity**
   - Built-in SQLite (no external database setup)
   - Single container deployment
   - Minimal configuration required
   - Fast startup time

4. **Core Features**
   - Visual Kanban board (primary need)
   - Task management with time tracking
   - File attachments and comments
   - Sufficient for startup workflows

5. **Maintenance**
   - Easy updates (single container)
   - Simple backup/restore process
   - Lower operational overhead

### Migration Path to Plane

If the team grows beyond 10 people or needs enterprise features (sprints, roadmaps, advanced reporting), migration to Plane is possible:

1. Export Kanboard tasks to JSON/CSV
2. Deploy Plane following standard documentation
3. Import data into Plane
4. Decommission Kanboard

**Recommended Trigger**: Team reaches 8-10 people or complex project management needs arise

## Troubleshooting

### Container Won't Start

```bash
# Check logs
sudo podman logs kanboard

# Check port conflicts
sudo ss -tlnp | grep 8081

# Verify volumes exist
sudo podman volume ls | grep kanboard
```

### Cannot Access Web Interface

```bash
# Check container health
sudo podman ps | grep kanboard

# Test local access
curl -f http://localhost:8081 -I

# Test Tailscale access
curl -f http://100.108.66.67:8081 -I

# Check firewall
sudo firewall-cmd --list-ports
```

### Lost Admin Password

```bash
# Access container and reset password via SQLite
sudo podman exec -it kanboard sh
cd /var/www/app/data
sqlite3 db.sqlite

# In SQLite prompt:
UPDATE users SET password='$2y$10$...hashed_password...' WHERE username='admin';
.exit
```

**Note**: Generate bcrypt hash online or use PHP to create new password hash.

### Database Corruption

```bash
# Stop container
cd /data/vdi
sudo podman-compose stop kanboard

# Restore from backup
sudo podman run --rm \
  -v vdi_kanboard_data:/data \
  -v /data/backups:/backup \
  alpine sh -c "cd /data && tar xzf /backup/kanboard_data_YYYYMMDD.tar.gz"

# Start container
sudo podman-compose start kanboard
```

## Documentation References

### Official Documentation
- **Kanboard Docs**: https://docs.kanboard.org/
- **User Guide**: https://docs.kanboard.org/en/latest/user_guide/
- **API Reference**: https://docs.kanboard.org/en/latest/api/
- **Docker Hub**: https://hub.docker.com/r/kanboard/kanboard

### Project-X Infrastructure
- **Main Documentation**: `/data/CLAUDE.md`
- **VDI Documentation**: `/data/vdi/DEPLOYMENT.md`
- **Backup Commands**: `/data/projects/BACKUP_COMMANDS.md`

## Best Practices

### Security
1. Change default admin password immediately
2. Create individual user accounts (no shared accounts)
3. Enable HTTPS if exposing to internet
4. Regular backups (daily recommended)
5. Keep container image updated

### Workflow
1. Create projects for each major initiative
2. Use swimlanes for team members or priorities
3. Set due dates for accountability
4. Add time estimates for capacity planning
5. Use comments for team collaboration
6. Attach relevant files directly to tasks

### Maintenance
1. **Weekly**: Check disk space, review logs
2. **Monthly**: Update container image, test backups
3. **Quarterly**: Review user access, audit projects

## Support & Community

- **GitHub**: https://github.com/kanboard/kanboard
- **Forum**: https://kanboard.discourse.group/
- **Chat**: Gitter/Matrix channels
- **Issues**: GitHub issue tracker

---

**Last Updated**: 2026-01-22
**Deployed By**: System Administrator
**Version**: Latest (automatically updated)
**Project Directory**: `/data/projects/kanboard`
