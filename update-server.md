# System Maintenance Guide

This guide performs essential system maintenance tasks:
1. Updates and upgrades all system packages
2. Removes unused packages and cleans package cache
3. Simulates a full system upgrade
4. Sets system timezone to UTC
5. Temporarily disables security services (ufw and fail2ban) for maintenance

## Update and upgrade system packages

```bash
sudo apt update && sudo apt upgrade -y
```

## Remove unused packages and clean up cached package files

```bash
sudo apt autoremove -y && sudo apt clean
```

## Perform a full upgrade simulation

```bash
sudo apt full-upgrade --simulate
```

## Update timezone to UTC

```bash
sudo timedatectl set-timezone UTC && sudo timedatectl
```

## Disable ufw

```bash
sudo ufw disable
```

## Disable fail2ban

```bash
sudo systemctl stop fail2ban && sudo systemctl disable fail2ban
```
