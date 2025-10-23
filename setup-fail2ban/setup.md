# Fail2ban Step-By-Step Setup & Best Practices

Fail2ban is an intrusion prevention tool that protects your server from brute-force attacks by monitoring log files and banning IPs that show malicious behavior. This guide covers installation, configuration of SSH and UFW filters, and monitoring.

---

## 1. Install Fail2ban

Update your package list and install fail2ban:

```bash
sudo apt update
sudo apt install -y fail2ban
sudo systemctl enable fail2ban
```

---

## 2. Verify Initial Status

Check that fail2ban is running:

```bash
sudo fail2ban-client status
```

---

## 3. Create Custom Filter Files

Create custom filters to detect specific attack patterns. Copy content from the `filter.d/` directory:

### SSH Filters

```bash
# Filter for wrong username attempts
sudo nano "/etc/fail2ban/filter.d/sshd-wrongUser.conf"
# Copy content from filter.d/sshd-wrongUser.conf
```

```bash
# Filter for root user login attempts
sudo nano "/etc/fail2ban/filter.d/sshd-rootUser.conf"
# Copy content from filter.d/sshd-rootUser.conf
```

### UFW Filters

```bash
# Filter for connections to wrong ports
sudo nano "/etc/fail2ban/filter.d/ufw-wrongPort.conf"
# Copy content from filter.d/ufw-wrongPort.conf
```

```bash
# Filter for port scanning attempts
sudo nano "/etc/fail2ban/filter.d/ufw-portScan.conf"
# Copy content from filter.d/ufw-portScan.conf
```

```bash
# Filter for SYN flood attacks
sudo nano "/etc/fail2ban/filter.d/ufw-synFlood.conf"
# Copy content from filter.d/ufw-synFlood.conf
```

```bash
# Filter for UDP flood attacks
sudo nano "/etc/fail2ban/filter.d/ufw-udpFlood.conf"
# Copy content from filter.d/ufw-udpFlood.conf
```

---

## 4. Test Filters Against Log Files

Verify that filters work correctly before enabling them:

### Test SSH Filters

```bash
sudo fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd-wrongUser.conf
sudo fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd-rootUser.conf
```

### Test UFW Filters

```bash
sudo fail2ban-regex /var/log/ufw.log /etc/fail2ban/filter.d/ufw-wrongPort.conf
sudo fail2ban-regex /var/log/ufw.log /etc/fail2ban/filter.d/ufw-portScan.conf
sudo fail2ban-regex /var/log/ufw.log /etc/fail2ban/filter.d/ufw-synFlood.conf
sudo fail2ban-regex /var/log/ufw.log /etc/fail2ban/filter.d/ufw-udpFlood.conf
```

**Note:** Look for "Lines: X" in the output. If X > 0, the filter matched log entries successfully.

---

## 5. Protect SSH Port from Being Banned (Critical)

**⚠️ Critical:** Prevent fail2ban from accidentally blocking your SSH port and locking you out.

### Create Global SSH Protection

Copy the content from `jail.d/never-ban-ssh.conf`:

```bash
sudo nano "/etc/fail2ban/jail.d/never-ban-ssh.conf"
# Copy content from jail.d/never-ban-ssh.conf
```

**⚠️ Important:** Adjust port 20220 if using a different SSH port.

### Exclude SSH Port from UFW Jails

Copy the content from `jail.d/ignore-ssh-port.conf`:

```bash
sudo nano "/etc/fail2ban/jail.d/ignore-ssh-port.conf"
# Copy content from jail.d/ignore-ssh-port.conf
```

**Why this matters:**
- Database clients (like Postico, TablePlus) often open multiple SSH tunnels
- UFW jails can mistake legitimate connection bursts for attacks
- Without these rules, fail2ban may ban your own IP and lock you out

---

## 6. Create Jail Configurations

Jails define which filters to use and what actions to take when attacks are detected.

### SSH Jail Configuration

```bash
sudo nano "/etc/fail2ban/jail.d/sshd.conf"
# Copy content from sshd.conf
```

**⚠️ Important:** Verify the port is set to your custom SSH port (e.g., 20220).

### UFW Jail Configuration

```bash
sudo nano "/etc/fail2ban/jail.d/ufw.conf"
# Copy content from ufw.conf
```

---

## 7. Restart & Verify Fail2ban

Apply the new configuration:

```bash
sudo systemctl restart fail2ban
sudo systemctl status fail2ban --no-pager
```

---

## 8. Check Active Jails

### View All Active Jails

```bash
sudo fail2ban-client status
```

### Check SSH Jail Status

```bash
sudo fail2ban-client status sshd
sudo fail2ban-client status sshd-wrongUser
sudo fail2ban-client status sshd-rootUser
```

### Check UFW Jail Status

```bash
sudo fail2ban-client status ufw-wrongPort
sudo fail2ban-client status ufw-portScan
sudo fail2ban-client status ufw-synFlood
sudo fail2ban-client status ufw-udpFlood
```

---

## 9. Monitor Fail2ban Activity

### View All Currently Banned IPs

```bash
sudo fail2ban-client banned
```

### Monitor Fail2ban Log in Real-time

```bash
sudo tail -f /var/log/fail2ban.log
```

### View Recent System Journal Entries

```bash
sudo journalctl -n 100
sudo journalctl -f
```

### Monitor Authentication Attempts

```bash
sudo tail -f /var/log/auth.log
```

### View UFW Block Activity

```bash
sudo tail -f /var/log/ufw.log
```

---

## 10. Unban IP Addresses (If Needed)

If you accidentally ban a legitimate IP:

### Unban from Specific Jail

```bash
sudo fail2ban-client set sshd unbanip <IP_ADDRESS>
```

### Unban from All Jails

```bash
sudo fail2ban-client unban <IP_ADDRESS>
```

**Tip:** Replace `<IP_ADDRESS>` with the actual IP address to unban.

---

## Additional Tips

- **Test filters thoroughly** before enabling jails to avoid false positives.
- **Whitelist your own IP** in `/etc/fail2ban/jail.local` using `ignoreip` directive.
- **Monitor logs regularly** to ensure fail2ban is working as expected.
- **Adjust ban times** based on your security requirements (default: 10 minutes).
- **Consider email notifications** for ban events by configuring the `action` parameter.
- **Always protect your SSH port** using the configurations in Step 5 to prevent lockouts.
- **Database SSH tunnels** (Postico, TablePlus, etc.) can trigger false positives - the SSH port protection is essential.
- For more configuration options, see [`man jail.conf`](https://manpages.ubuntu.com/manpages/latest/en/man5/jail.conf.5.html).

---

## Common Issues & Solutions

### SSH Connection Drops or Fails

**Symptoms:**
- SSH service crashes or refuses connections
- Database SSH tunnels (Postico, TablePlus) fail
- Getting locked out after multiple connection attempts

**Root Causes:**
1. Multiple SSH tunnels exceed SSH limits (`MaxSessions`, `MaxStartups`)
2. Fail2ban + UFW mistake connection bursts for attacks and ban your IP
3. `ssh.socket` causing instability (use `ssh.service` instead)

**Solutions:**
1. **Disable ssh.socket** (covered in SSH setup guide):
   ```bash
   sudo systemctl disable ssh.socket
   sudo systemctl mask ssh.socket
   sudo systemctl restart ssh.service
   ```

2. **Adjust SSH limits** in `/etc/ssh/sshd_config`:
   ```
   MaxSessions 10
   MaxStartups 10:30:60
   ```

3. **Implement SSH port protection** (Step 5 above) to prevent fail2ban from banning SSH connections

4. **Verify SSH port is listening**:
   ```bash
   sudo ss -tlnp | grep 20220
   ```

5. **Check if your IP is banned**:
   ```bash
   sudo fail2ban-client banned
   sudo iptables -L -n | grep <YOUR_IP>
   ```

---

## Useful Commands Reference

```bash
# Reload fail2ban configuration without restart
sudo fail2ban-client reload

# Stop a specific jail
sudo fail2ban-client stop <jail_name>

# Start a specific jail
sudo fail2ban-client start <jail_name>

# Get banned IP count for a jail
sudo fail2ban-client status <jail_name> | grep "Currently banned"

# View fail2ban version
fail2ban-client version
```
