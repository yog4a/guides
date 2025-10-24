# Server Initial Setup & Maintenance

This guide covers essential system setup tasks for a fresh VPS server. Perform these steps before configuring SSH, firewall, and other security measures.

---

## 1. Update System Packages

Update package lists and upgrade all installed packages:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. Remove Unused Packages

Clean up unused packages and cached files:

```bash
sudo apt autoremove -y && sudo apt clean
```

---

## 3. Check for Distribution Upgrades

Simulate a full system upgrade to see what would be changed:

```bash
sudo apt full-upgrade --simulate
```

**Note:** Review the output carefully before running without `--simulate`.

If ready to proceed:

```bash
sudo apt full-upgrade -y
```

---

## 4. Set System Timezone

Configure timezone to UTC (recommended for servers):

```bash
sudo timedatectl set-timezone UTC
sudo timedatectl
```

**Verify timezone is set:**

```bash
timedatectl status
```

**Alternative timezones:**

```bash
# List all available timezones
timedatectl list-timezones

# Set to your local timezone (example)
sudo timedatectl set-timezone America/New_York
```

---

## 5. Configure Automatic Security Updates (Recommended)

For automatic security updates, see [Unattended Upgrades Guide](unattended-upgrades.md).

Quick setup:

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

---

## 6. Set Hostname (Optional)

Set a meaningful hostname for your server:

```bash
# View current hostname
hostname

# Set new hostname
sudo hostnamectl set-hostname your-server-name

# Verify
hostnamectl
```

---

## Additional Tips

- **Always update first** before installing new software
- **Reboot if kernel updated** - Check with `cat /var/run/reboot-required`
- **Monitor disk space** - Use `df -h` to check available space
- **Check system logs** - Use `sudo journalctl -xe` for errors
- **Document your setup** - Keep notes on customizations

---

## Next Steps

After completing initial setup, proceed with:

1. **[SSH Setup](../setup-ssh/setup.md)** - Configure secure SSH access
2. **[UFW Firewall](../setup-ufw/setup.md)** - Set up firewall rules
3. **[Fail2ban](../setup-fail2ban/setup.md)** - Configure intrusion prevention
4. **[Disable IPv6](disable-ipv6.md)** (optional) - If not using IPv6

---

## Useful Commands

```bash
# Check Ubuntu/Debian version
lsb_release -a

# Check system resources
free -h                 # Memory usage
df -h                   # Disk usage
top                     # Process monitor
htop                    # Better process monitor (install first)

# Check for pending updates
sudo apt update
sudo apt list --upgradable

# View system information
uname -a                # Kernel info
hostnamectl             # Hostname and system info
timedatectl             # Time and timezone info

# Check if reboot required
cat /var/run/reboot-required
```
