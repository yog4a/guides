# UFW Firewall Step-By-Step Setup & Best Practices

The Uncomplicated Firewall (UFW) makes managing iptables on Linux systems easy and beginner-friendly. This guide will walk you through installing, configuring, and monitoring UFW for enhanced system security.

---

## 1. Install UFW

Update your package list and install UFW:

```bash
sudo apt update
sudo apt install -y ufw
```

---

## 2. Reset All UFW Rules (Recommended for Fresh Setup)

Reset all existing UFW configuration to the default state (Optional, but highly recommended before reconfiguring):

```bash
sudo ufw reset
```

---

## 3. (Optional) Disable IPv6 in UFW

If your server does not require IPv6 support, you may disable it for simplicity.

**a. Check if IPv6 is enabled in UFW:**

```bash
grep ^IPV6 /etc/default/ufw
```

**b. Temporarily modify UFW config permissions to allow editing:**

```bash
sudo chmod 666 /etc/default/ufw
```

**c. Edit `/etc/default/ufw` and set:**
```
IPV6=no
```

**d. (Optional) Disable system-wide IPv6 (edit `/etc/sysctl.conf`):**
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
```
Then apply sysctl changes:

```bash
sudo sysctl -p
```

**e. Restore original file permissions:**

```bash
sudo chmod 644 /etc/default/ufw
```

---

## 4. Whitelist Localhost (Recommended)

Ensure localhost traffic is always allowed to prevent blocking local services.

**Note:** UFW typically allows localhost by default, but this ensures it explicitly.

```bash
# Allow all traffic on loopback interface
sudo ufw allow from 127.0.0.1 to any
sudo ufw allow from ::1 to any
```

**Alternative:** Add custom rules to `/etc/ufw/before.rules` (copy from `local-whitelist.conf`):

```bash
sudo nano /etc/ufw/before.rules
# Add the rules from local-whitelist.conf after the initial comment block
```

---

## 5. Set Secure Default Firewall Policies

Recommended defaults: Allow all outgoing, deny all incoming.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

---

## 6. Add Custom Firewall Rules

### Example: Secure SSH, Block HTTP, Allow HTTPS

```bash
# Limit SSH brute-force on port 20220 (change to actual SSH port if different)
sudo ufw limit 20220/tcp comment 'Rate limit SSH connections'

# Deny all HTTP (port 80) traffic
sudo ufw deny 80/tcp comment 'Block HTTP'

# Allow HTTPS (port 443) traffic
sudo ufw allow 443/tcp comment 'Allow HTTPS'
```
**Tip:** Adjust ports/rules to fit your server’s requirements.

---

## 7. Enable & Activate the Firewall

```bash
sudo ufw enable     # Activates UFW
sudo ufw reload     # Reloads to apply changes
sudo ufw logging on # Enables logging for monitoring
```

**Note:** Enabling will drop existing SSH sessions if you have not allowed your SSH port.

---

## 8. Verify Status and Rules

Check current UFW status and all rules:

```bash
sudo ufw status verbose
```

---

## 9. Monitor & Troubleshoot UFW Activity

### View Real-time Firewall Logs

```bash
sudo tail -f /var/log/ufw.log
```

### Show Entire UFW Log

```bash
sudo less /var/log/ufw.log
```

### List IPs Banned by fail2ban

```bash
sudo fail2ban-client banned
```

### Monitor Authentication Attempts (for SSH, etc.)

```bash
sudo tail -f /var/log/auth.log
```

### Check Recent SystemD Journal Messages

```bash
sudo journalctl -n 100
sudo journalctl -f
```

### List Successful SSH Logins with Public Keys

```bash
sudo grep "Accepted publickey" /var/log/auth.log | awk '{print $(NF-5), $(NF-3)}'
```
---

## Additional Tips

- **Always test firewall changes in a disconnected or safe environment.**
- **Remember to allow ports needed for remote management before enabling UFW.**
- For more advanced rule syntax and examples, see [`man ufw`](https://manpages.ubuntu.com/manpages/latest/en/man8/ufw.8.html).
