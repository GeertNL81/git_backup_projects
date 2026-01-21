# Backup Commands Reference

## GitWatch Commands

### Service Management
```bash
# Check gitwatch status
sudo systemctl status gitwatch.service

# Start gitwatch
sudo systemctl start gitwatch.service

# Stop gitwatch
sudo systemctl stop gitwatch.service

# Restart gitwatch
sudo systemctl restart gitwatch.service

# View gitwatch logs
sudo journalctl -u gitwatch.service -f

# View recent gitwatch logs
sudo journalctl -u gitwatch.service --since "1 hour ago"
```

### Git Commands (for /data/projects)
```bash
# Check git status
sudo git -C /data/projects status

# View commit history
sudo git -C /data/projects log --oneline -20

# View last commit details
sudo git -C /data/projects show

# Manually push changes
sudo git -C /data/projects push

# Pull changes from remote
sudo git -C /data/projects pull
```

---

## BorgBackup Commands

### Backup Operations
```bash
# Run backup manually
sudo /usr/local/bin/borg-backup-etc

# Create a backup with custom name
sudo borg create /data/backups/etc-backup::my-backup-name /etc
```

### View Backups
```bash
# List all backups
sudo borg list /data/backups/etc-backup

# List contents of a specific backup
sudo borg list /data/backups/etc-backup::ARCHIVE_NAME

# Show backup info
sudo borg info /data/backups/etc-backup

# Show specific archive info
sudo borg info /data/backups/etc-backup::ARCHIVE_NAME
```

### Restore Files
```bash
# Restore entire backup to current directory
sudo borg extract /data/backups/etc-backup::ARCHIVE_NAME

# Restore specific file/folder
sudo borg extract /data/backups/etc-backup::ARCHIVE_NAME etc/hosts

# Restore to specific location
cd /tmp && sudo borg extract /data/backups/etc-backup::ARCHIVE_NAME

# Preview what would be extracted (dry run)
sudo borg extract --dry-run --list /data/backups/etc-backup::ARCHIVE_NAME
```

### Compare and Diff
```bash
# Compare two backups
sudo borg diff /data/backups/etc-backup::ARCHIVE1 ARCHIVE2

# Show changes since last backup
sudo borg diff /data/backups/etc-backup::ARCHIVE_NAME --last 1
```

### Maintenance
```bash
# Check repository integrity
sudo borg check /data/backups/etc-backup

# Compact repository (free up space)
sudo borg compact /data/backups/etc-backup

# Delete a specific backup
sudo borg delete /data/backups/etc-backup::ARCHIVE_NAME

# Prune old backups (keep 7 daily, 4 weekly, 6 monthly)
sudo borg prune --keep-daily=7 --keep-weekly=4 --keep-monthly=6 /data/backups/etc-backup
```

### Timer Management
```bash
# Check backup timer status
sudo systemctl status borg-backup-etc.timer

# List all timers (see next backup time)
sudo systemctl list-timers borg-backup-etc.timer

# Start timer
sudo systemctl start borg-backup-etc.timer

# Stop timer
sudo systemctl stop borg-backup-etc.timer

# Run backup service manually
sudo systemctl start borg-backup-etc.service

# View backup logs
sudo journalctl -u borg-backup-etc.service
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Check gitwatch | `sudo systemctl status gitwatch` |
| Check borg timer | `sudo systemctl list-timers borg*` |
| List backups | `sudo borg list /data/backups/etc-backup` |
| Run backup now | `sudo /usr/local/bin/borg-backup-etc` |
| Restore file | `sudo borg extract /data/backups/etc-backup::ARCHIVE etc/filename` |

---

## File Locations

- **GitWatch service:** `/etc/systemd/system/gitwatch.service`
- **Borg backup script:** `/usr/local/bin/borg-backup-etc`
- **Borg service:** `/etc/systemd/system/borg-backup-etc.service`
- **Borg timer:** `/etc/systemd/system/borg-backup-etc.timer`
- **Borg repository:** `/data/backups/etc-backup`
- **Watched directory:** `/data/projects`
