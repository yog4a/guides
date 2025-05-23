# Install fail2ban
    sudo apt install -y fail2ban

# Display fail2ban status
    sudo fail2ban-client status

# Create custom jail config
    sudo nano "/etc/fail2ban/jail.d/sshd.conf"
    sudo nano "/etc/fail2ban/jail.d/ufw.conf"

# Create custom filter config
    sudo nano "/etc/fail2ban/filter.d/sshd-wrongUser.conf"
    sudo nano "/etc/fail2ban/filter.d/sshd-rootUser.conf"
    sudo nano "/etc/fail2ban/filter.d/ufw-wrongPort.conf"
    sudo nano "/etc/fail2ban/filter.d/ufw-synFlood.conf"
    sudo nano "/etc/fail2ban/filter.d/ufw-udpFlood.conf"
    sudo nano "/etc/fail2ban/filter.d/ufw-portScan.conf"

# Restart fail2ban
    sudo systemctl restart fail2ban
    sudo systemctl status fail2ban --no-pager

# Display fail2ban status
    sudo fail2ban-client status

# Display SSH Jail Status
    sudo fail2ban-client status sshd
    sudo fail2ban-client status ufw

# Display Wrong User Jail Status
    sudo fail2ban-client status sshd-wrongUser
    sudo fail2ban-client status sshd-rootUser
    sudo fail2ban-client status ufw-wrongPort
    sudo fail2ban-client status ufw-synFlood
    sudo fail2ban-client status ufw-udpFlood
    sudo fail2ban-client status ufw-portScan

# Additional commands
    sudo systemctl enable fail2ban
    sudo tail -f /var/log/fail2ban.log
    sudo fail2ban-client banned
    sudo journalctl -n 100
    sudo journalctl -f
