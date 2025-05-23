# UFW Firewall Setup Guide

This guide provides step-by-step instructions for setting up and configuring the Uncomplicated Firewall (UFW) on a Linux system.

## Install UFW

```bash
sudo apt install -y ufw
```

## Reset UFW rules

```bash
sudo ufw reset
```

## Disable IPv6 in UFW

```bash
# Allow opening UFW config file
sudo chmod 666 "/etc/default/ufw"

# Change IPV6=yes to IPV6=no in the file
# Then verify the change
grep ^IPV6 /etc/default/ufw

# Restore original permissions
sudo chmod 644 "/etc/default/ufw"
```

## Configure default UFW policies

```bash
sudo ufw default allow outgoing comment 'allow all outgoing traffic'
sudo ufw default deny incoming comment 'deny all incoming traffic'
```

## Configure specific port rules

```bash
sudo ufw limit 22/tcp comment "Rate limit SSH connections"
sudo ufw deny 80/tcp comment "HTTP"
sudo ufw allow 443/tcp comment "HTTPS"
```

## Enable and configure UFW

```bash
sudo ufw enable
sudo ufw reload
sudo ufw logging on
```

## Verify UFW status

```bash
sudo ufw status verbose
```

## Monitoring and troubleshooting commands

### View real-time firewall logs

```bash
sudo tail -f "/var/log/ufw.log"
```

### View complete firewall log history

```bash
sudo cat "/var/log/ufw.log"
```

### View fail2ban banned IPs

```bash
sudo fail2ban-client banned
```

### Monitor authentication logs

```bash
sudo tail -f "/var/log/auth.log"
```

### View system journal logs

```bash
sudo journalctl -n 100
sudo journalctl -f
```

### List successful SSH key authentications

```bash
sudo grep "Accepted publickey" "/var/log/auth.log" | awk '{print $7, $9}'
```