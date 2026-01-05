# Monthly Server Maintenance Playbook

> A simple checklist for maintaining Hetzner servers (with Coolify), Google Cloud Run, and Cloudflare.

---

## Table of Contents

1. [Hetzner Servers](#part-1-hetzner-servers)
2. [Coolify Dashboard](#part-2-coolify-dashboard)
3. [Google Cloud Run](#part-3-google-cloud-run)
4. [Cloudflare](#part-4-cloudflare)
5. [Backups & Snapshots](#part-5-backups--snapshots)
6. [Monthly Log Template](#monthly-log-template)
7. [Quick Reference Commands](#quick-reference-commands)

---

## Part 1: Hetzner Servers

> Repeat this section for each server.

### 1.1 System Updates

```bash
# SSH into your server
ssh root@your-server-ip

# Update package lists and upgrade
sudo apt update && sudo apt upgrade -y

# Check if reboot is needed
cat /var/run/reboot-required
# If file exists, plan a reboot
```

### 1.2 Disk Space Check

```bash
# Check disk usage
df -h

# If disk is >80% full, find large files
du -sh /* | sort -hr | head -10
```

**Action needed if:** Disk usage exceeds 80%

### 1.3 Memory & CPU Check

```bash
# Quick system overview
htop
# or
free -h && uptime
```

**Look for:** Consistently high memory/CPU usage

### 1.4 Temporary Files Cleanup

```bash
# Clear temporary files
sudo rm -rf /tmp/*

# Clear user cache
sudo rm -rf ~/.cache/*
```

**Note:** Safe to run monthly. These files are meant to be temporary.

### 1.5 Remove Unused Packages & Dependencies

```bash
# Remove unneeded dependencies
sudo apt autoremove -y

# Delete old package archives
sudo apt autoclean

# Remove all downloaded package files
sudo apt clean
```

**What each command does:**
- `autoremove` - Removes packages that were installed as dependencies but are no longer needed
- `autoclean` - Deletes old versions of downloaded package files
- `clean` - Removes all downloaded package files from cache

### 1.6 Clean Old Logs

```bash
# Check current log size
sudo journalctl --disk-usage

# Keep logs for only 7 days
sudo journalctl --vacuum-time=7d

# Or limit log size to 100MB
sudo journalctl --vacuum-size=100M
```

**Optional - Clear all logs (use with caution):**

```bash
sudo rm -rf /var/log/*.log
sudo rm -rf /var/log/*.gz
```

**Warning:** Only clear logs if you don't need them for debugging. Consider keeping at least 7 days of logs.

### 1.7 Find and Remove Large Files

```bash
# Find files over 1GB
sudo find / -type f -size +1G 2>/dev/null

# Find files over 500MB
sudo find / -type f -size +500M 2>/dev/null

# Find files over 100MB (more results)
sudo find / -type f -size +100M 2>/dev/null
```

**To delete a large file:**

```bash
# Only after confirming the file is not needed!
rm -rf /path/to/large/file
```

**Warning:** Always verify what the file is before deleting. Never delete system files or database files.

### 1.8 Remove Old Snap Packages (If Using Snap)

```bash
# Check snap disk usage
sudo du -sh /var/lib/snapd/snaps

# List installed snaps
snap list --all

# Remove disabled/old snap versions
sudo snap list --all | awk '/disabled/{print $1, $3}' | while read snapname revision; do
    sudo snap remove "$snapname" --revision="$revision"
done
```

**Note:** Snap keeps old versions for rollback. This removes old versions while keeping the current one.

### 1.9 Docker Cleanup (Important for Coolify)

```bash
# Check Docker disk usage before cleanup
docker system df

# Remove unused Docker resources
docker system prune -a --volumes

# Check Docker disk usage after cleanup
docker system df
```

**Note:** This removes unused images, containers, and volumes. Make sure no important stopped containers exist.

---

---

## Part 2: Coolify Dashboard

### 2.1 Check Application Health

- [ ] Log into Coolify dashboard
- [ ] Review each project's status (green = healthy)
- [ ] Check for any failed deployments in the last month

### 2.2 Update Coolify

- [ ] Check if Coolify update is available (shown in dashboard)
- [ ] If yes, click update (pick a low-traffic time)

### 2.3 Review Resource Usage

- [ ] Check CPU/Memory graphs for each application
- [ ] Note any apps that consistently use too many resources

---

## Part 3: Google Cloud Run

### 3.1 Quick Health Check

- [ ] Go to Google Cloud Console → Cloud Run
- [ ] Check each service shows "✓" healthy status
- [ ] Review "Metrics" tab for any error spikes

### 3.2 Review Logs

- [ ] Click on each service → Logs
- [ ] Filter for "Error" or "Warning"
- [ ] Note any recurring issues

### 3.3 Cost Check

- [ ] Go to Billing → Reports
- [ ] Compare this month vs last month
- [ ] Investigate any unexpected increases

---

## Part 4: Cloudflare

### 4.1 Security Check

- [ ] Log into Cloudflare dashboard
- [ ] Go to Security → Overview
- [ ] Review any blocked threats
- [ ] Check SSL certificates are valid (not expiring soon)

### 4.2 DNS Review

- [ ] Go to DNS → Records
- [ ] Verify all records point to correct IPs
- [ ] Remove any old/unused records

---

## Part 5: Backups & Snapshots

### 5.1 Hetzner Automatic Backups (Recommended)

**How to Enable:**

1. Go to [Hetzner Cloud Console](https://console.hetzner.cloud)
2. Select your server
3. Click **Backups** in the left menu
4. Click **Enable Backups**

**What You Get:**

- Automatic daily backups
- Hetzner keeps the last 7 backups
- One-click restore if something goes wrong

**Cost:** ~20% of server cost (e.g., €4 server = €0.80/month for backups)

### 5.2 Monthly Backup Verification

- [ ] Go to Hetzner Console → Server → Backups
- [ ] Verify recent backup dates are showing
- [ ] Check backup size is reasonable

### 5.3 Manual Snapshots (Before Major Changes)

**When to create manual snapshots:**

- Before major system updates
- Before installing new software
- Before Coolify updates

**How to create:**

1. Go to Hetzner Cloud Console
2. Select your server
3. Click **Snapshots** → **Create Snapshot**
4. Name it: `servername-YYYY-MM-DD-reason`

### 5.4 Database Backups

- [ ] Verify your database backup solution is running
- [ ] Test: Can you access a recent backup file?
- [ ] Check backup storage location has space

---

## Monthly Log Template

> Copy this section for each month's maintenance.

```markdown
## Maintenance Log - [Month Year]

**Date completed:** ___________
**Performed by:** ___________

---

### Server 1
- **Name/IP:** _________
- [ ] System updated
- [ ] Disk usage: ____% 
- [ ] Temp files cleaned
- [ ] Unused packages removed
- [ ] Old logs cleaned
- [ ] Large files checked
- [ ] Docker cleaned
- [ ] Backups verified
- **Notes:**

---

### Server 2
- **Name/IP:** _________
- [ ] System updated
- [ ] Disk usage: ____%
- [ ] Temp files cleaned
- [ ] Unused packages removed
- [ ] Old logs cleaned
- [ ] Large files checked
- [ ] Docker cleaned
- [ ] Backups verified
- **Notes:**

---

### Coolify
- [ ] All apps healthy
- [ ] Coolify updated (if available)
- [ ] Resource usage reviewed
- **Notes:**

---

### Google Cloud Run
- [ ] All services healthy
- [ ] Logs reviewed for errors
- [ ] Costs reviewed: $____
- **Notes:**

---

### Cloudflare
- [ ] SSL certificates valid
- [ ] DNS records verified
- [ ] Security overview checked
- **Notes:**

---

### Backups & Snapshots
- [ ] Hetzner backups verified
- [ ] Manual snapshot created (if needed)
- [ ] Database backups verified
- **Notes:**

---

### Issues Found

| Issue | Severity | Action Taken | Status |
|-------|----------|--------------|--------|
|       |          |              |        |
|       |          |              |        |

---

### Next Month Reminders

1. 
2. 
3. 

```

---

## Quick Reference Commands

### Server Basics

```bash
# Connect to server
ssh root@SERVER_IP

# Check disk space
df -h

# Check memory
free -h

# Live system monitor
htop

# Server uptime
uptime

# Check if reboot needed
cat /var/run/reboot-required

# Clear temporary files
sudo rm -rf /tmp/*

# Clear user cache
sudo rm -rf ~/.cache/*
```

### Cleanup Commands

```bash
# Remove unused packages
sudo apt autoremove -y
sudo apt autoclean
sudo apt clean

# Clean old logs (keep 7 days)
sudo journalctl --vacuum-time=7d

# Find large files (over 1GB)
sudo find / -type f -size +1G 2>/dev/null

# Remove old snap versions
sudo snap list --all | awk '/disabled/{print $1, $3}' | while read snapname revision; do
    sudo snap remove "$snapname" --revision="$revision"
done
```

### Docker / Coolify

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Check Docker disk usage
docker system df

# Clean up unused resources (careful!)
docker system prune -a --volumes

# View container logs
docker logs CONTAINER_NAME

# View container logs (follow mode)
docker logs -f CONTAINER_NAME

# Restart a container
docker restart CONTAINER_NAME
```

### System Logs

```bash
# View system logs
journalctl -xe

# View recent logs
journalctl --since "1 hour ago"

# View Docker daemon logs
journalctl -u docker
```

### Network

```bash
# Check open ports
ss -tulpn

# Check firewall status
ufw status

# Test connectivity
ping google.com
```

---

## Time Estimate

| Task | Estimated Time |
|------|----------------|
| Hetzner servers (both) | 20-30 min |
| Coolify dashboard | 5-10 min |
| Google Cloud Run | 5-10 min |
| Cloudflare | 5 min |
| Backups verification | 5 min |
| **Total** | **~50-60 min** |

---

## Maintenance Schedule

| Frequency | Task |
|-----------|------|
| Weekly | Quick health check (Coolify dashboard) |
| Monthly | Full maintenance (this playbook) |
| Before updates | Create manual snapshot |
| Quarterly | Review and clean unused resources |

---

## Emergency Contacts & Links

| Service | Dashboard URL |
|---------|---------------|
| Hetzner | https://console.hetzner.cloud |
| Coolify | https://your-coolify-url |
| Google Cloud | https://console.cloud.google.com |
| Cloudflare | https://dash.cloudflare.com |

---

## Version History

| Date | Version | Changes |
|------|---------|---------|
| YYYY-MM-DD | 1.0 | Initial playbook |
|  |  |  |

---

*Last updated: January 2025*