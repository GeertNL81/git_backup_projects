# git_backup & projects management

Project for version control and automated backups of the `/data/projects` folder.

## 1. Real-time Version Control (gitwatch)
The `/data/projects` folder is a Git repository monitored by `gitwatch`.
- **Service:** `gitwatch-projects.service`
- **Interval:** 5-minute wait after changes before commit/push.
- **Remote:** [GeertNL81/git_backup_projects](https://github.com/GeertNL81/git_backup_projects)
- **Check Status:** `systemctl status gitwatch-projects.service`

## 2. Automated Backups (BorgBackup)
Daily encrypted and deduplicated backups stored locally.
- **Repository:** `/data/borg_backup`
- **Script:** `/usr/local/bin/borg-backup.sh`
- **Timer:** `borg-backup.timer` (Runs daily)
- **Passphrase:** `ResetRoot`

### Common Commands

#### Check Borg Backups
```bash
export BORG_PASSPHRASE="ResetRoot"
borg list /data/borg_backup
```

#### Mount a Backup for Restore
```bash
sudo mkdir -p /tmp/restore
export BORG_PASSPHRASE="ResetRoot"
sudo -E borg mount /data/borg_backup::<ARCHIVE_NAME> /tmp/restore
# Browse /tmp/restore, then unmount:
sudo borg umount /tmp/restore
```
