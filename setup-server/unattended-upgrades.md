# Unattended Upgrades Step-By-Step Setup & Best Practices

Unattended upgrades automatically install security updates on your server, keeping it secure without manual intervention. This guide covers installation, configuration, and monitoring.

---

## 1. Install Unattended Upgrades

```bash
sudo apt update
sudo apt install -y unattended-upgrades apt-listchanges
```

---

## 2. Enable Unattended Upgrades

```bash
sudo dpkg-reconfigure -plow unattended-upgrades
```

Select **"Yes"** when prompted to enable automatic updates.

---

## 3. Configure Unattended Upgrades

Edit the configuration file:

```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

### Recommended Configuration

Uncomment and modify these lines:

```conf
// Automatically upgrade security updates
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
    "${distro_id}ESM:${distro_codename}-infra-security";
};

// Remove unused kernel packages and dependencies
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";

// Automatically reboot if required (optional - use with caution)
Unattended-Upgrade::Automatic-Reboot "false";

// If automatic reboot is enabled, set a specific time
// Unattended-Upgrade::Automatic-Reboot-Time "02:00";

// Email notifications (optional - requires mail setup)
// Unattended-Upgrade::Mail "your-email@example.com";
// Unattended-Upgrade::MailReport "on-change";
```

**Important Notes:**
- Set `Automatic-Reboot` to `"true"` only if you're comfortable with automatic reboots
- Reboots only happen when kernel updates require them
- Consider setting `Automatic-Reboot-Time` to off-peak hours (e.g., "02:00")

---

## 4. Configure Update Frequency

Edit the auto-update configuration:

```bash
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```

### Recommended Settings

```conf
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

**What these do:**
- `Update-Package-Lists "1"` - Check for updates daily
- `Download-Upgradeable-Packages "1"` - Download updates daily
- `AutocleanInterval "7"` - Clean up old packages weekly
- `Unattended-Upgrade "1"` - Install security updates daily

---

## 5. Test the Configuration

### Dry Run (Test Without Installing)

```bash
sudo unattended-upgrades --dry-run --debug
```

This shows what would be upgraded without actually upgrading.

### Manual Run (Force Upgrade Now)

```bash
sudo unattended-upgrades --debug
```

---

## 6. Monitor Unattended Upgrades

### Check Logs

```bash
# View recent upgrade activity
sudo cat /var/log/unattended-upgrades/unattended-upgrades.log

# View detailed debug log
sudo cat /var/log/unattended-upgrades/unattended-upgrades-dpkg.log

# Monitor in real-time
sudo tail -f /var/log/unattended-upgrades/unattended-upgrades.log
```

### Check Last Run Status

```bash
sudo systemctl status unattended-upgrades
```

### Check for Pending Updates

```bash
sudo apt update
sudo apt list --upgradable
```

---

## 7. Configure Email Notifications (Optional)

### Install Mail Utilities

```bash
sudo apt install -y mailutils postfix
```

### Enable Email Notifications

Edit the configuration:

```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Uncomment and configure:

```conf
Unattended-Upgrade::Mail "your-email@example.com";
Unattended-Upgrade::MailReport "on-change";
```

**MailReport options:**
- `"always"` - Send email after every run
- `"only-on-error"` - Send only when errors occur
- `"on-change"` - Send only when packages are upgraded

---

## 8. Handle Automatic Reboots

### Check if Reboot is Required

```bash
[ -f /var/run/reboot-required ] && cat /var/run/reboot-required
```

### List Packages Requiring Reboot

```bash
[ -f /var/run/reboot-required.pkgs ] && cat /var/run/reboot-required.pkgs
```

### Enable Automatic Reboot (Use with Caution)

Edit configuration:

```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Set:

```conf
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "02:00";
Unattended-Upgrade::Automatic-Reboot-WithUsers "false";
```

**Note:** Only enable automatic reboots if your services can tolerate brief downtime.

---

## Troubleshooting

### Check if Service is Running

```bash
sudo systemctl status apt-daily.timer
sudo systemctl status apt-daily-upgrade.timer
```

### View Timer Schedule

```bash
systemctl list-timers apt-daily*
```

### Manually Trigger Update

```bash
sudo systemctl start apt-daily-upgrade.service
```

### Disable Unattended Upgrades (If Needed)

```bash
sudo dpkg-reconfigure -plow unattended-upgrades
# Select "No"
```

Or disable the service:

```bash
sudo systemctl stop unattended-upgrades
sudo systemctl disable unattended-upgrades
```

---

## Common Issues

**Updates not running:**
- Check timer status: `systemctl status apt-daily.timer`
- Check logs: `sudo cat /var/log/unattended-upgrades/unattended-upgrades.log`
- Verify configuration: `sudo unattended-upgrades --dry-run --debug`

**Disk space issues:**
- Clean old packages: `sudo apt autoremove && sudo apt clean`
- Check disk usage: `df -h`
- Increase `AutocleanInterval` in `/etc/apt/apt.conf.d/20auto-upgrades`

**Packages held back:**
- Some packages require manual intervention
- Check with: `sudo apt list --upgradable`
- Upgrade manually: `sudo apt full-upgrade`

---

## Additional Tips

- **Monitor disk space** - Automatic updates can fill disk over time
- **Test in staging** - Try configuration in a test environment first
- **Keep manual control** - Consider disabling automatic reboots for critical servers
- **Review logs regularly** - Check for failed updates or errors
- **Whitelist critical packages** - Prevent specific packages from auto-updating if needed
- **Combine with monitoring** - Use with server monitoring tools for alerts
- For more options, see [`man unattended-upgrades`](https://manpages.ubuntu.com/manpages/latest/en/man8/unattended-upgrades.8.html)

---

## Security Best Practices

✅ **Enable security updates only** - Don't auto-install all updates  
✅ **Remove unused packages** - Reduce attack surface  
✅ **Monitor logs** - Check for failed updates  
✅ **Test reboot behavior** - Ensure services restart properly  
✅ **Keep manual oversight** - Review major updates before applying  
✅ **Backup before enabling** - Ensure you can recover if needed  

---

## Useful Commands Reference

```bash
# Check configuration
sudo unattended-upgrades --dry-run

# Force immediate upgrade
sudo unattended-upgrades

# View upgrade history
sudo cat /var/log/apt/history.log

# Check pending security updates
sudo apt update && sudo apt list --upgradable | grep security

# Disable specific package from auto-upgrade
sudo apt-mark hold package-name

# Re-enable auto-upgrade for package
sudo apt-mark unhold package-name

# List held packages
sudo apt-mark showhold
```

