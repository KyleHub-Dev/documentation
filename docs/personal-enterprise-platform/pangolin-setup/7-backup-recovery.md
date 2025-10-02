---
title: Backup & Recovery
sidebar_position: 8
tags:
  - Backup
  - Recovery
  - Data Protection
  - Disaster Recovery
---

## 1. Backup and Recovery

### 1.1 Backup Pangolin Configuration

Create a backup script for Pangolin data:

```bash
# Create backup script
cat <<'EOF' > /opt/pangolin/backup.sh
#!/bin/bash
BACKUP_DIR="/opt/pangolin/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop containers
cd /opt/pangolin
docker compose down

# Backup data directory
tar -czf "$BACKUP_DIR/pangolin-backup-$TIMESTAMP.tar.gz" data/

# Restart containers
docker compose up -d

# Remove backups older than 30 days
find "$BACKUP_DIR" -name "pangolin-backup-*.tar.gz" -mtime +30 -delete

echo "Backup completed: pangolin-backup-$TIMESTAMP.tar.gz"
EOF

chmod +x /opt/pangolin/backup.sh

# Schedule daily backups at 2 AM
(crontab -l 2>/dev/null; echo "0 2 * * * /opt/pangolin/backup.sh >> /var/log/pangolin-backup.log 2>&1") | crontab -
```

### 1.2 Restore from Backup

To restore Pangolin from a backup:

```bash
cd /opt/pangolin
docker compose down
rm -rf data/
tar -xzf backups/pangolin-backup-YYYYMMDD_HHMMSS.tar.gz
docker compose up -d
```