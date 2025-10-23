# VPS Setup Guides & Best Practices

A comprehensive collection of step-by-step guides for setting up and securing a VPS server. Each guide follows best practices and includes troubleshooting tips based on real-world experience.

---

## 🚀 Quick Start

Follow these guides in order for a complete secure server setup:

1. **[Server Setup](setup-server/setup-server.md)** - Initial server configuration
2. **[SSH Setup](setup-ssh/setup.md)** - Secure SSH access with key-based authentication
3. **[UFW Firewall](setup-ufw/setup.md)** - Configure firewall rules
4. **[Fail2ban](setup-fail2ban/setup.md)** - Intrusion prevention and attack mitigation

---

## 📚 Core Security Guides

### [Server Setup](setup-server/)
Initial server configuration and maintenance:
- System updates and package management
- Timezone configuration
- [Disable IPv6](setup-server/disable-ipv6.md) (optional)

### [SSH Setup](setup-ssh/)
Secure SSH access configuration:
- Create non-root user with sudo privileges
- SSH key-based authentication (ED25519)
- Custom SSH port configuration (port 20220)
- Disable root login and password authentication
- SSH hardening with fail2ban compatibility
- **Includes**: Ready-to-use [ssh.conf](setup-ssh/ssh.conf)

### [UFW Firewall](setup-ufw/)
Uncomplicated Firewall setup:
- Default deny incoming, allow outgoing policy
- Custom port rules and rate limiting
- IPv6 configuration options
- Real-time monitoring and logging

### [Fail2ban](setup-fail2ban/)
Intrusion prevention system:
- SSH brute-force protection (custom filters)
- UFW attack detection (port scan, SYN/UDP floods)
- **Critical**: SSH port protection to prevent lockouts
- Database SSH tunnel compatibility (Postico, TablePlus)
- **Includes**: 
  - 6 custom filters in [filter.d/](setup-fail2ban/filter.d/)
  - Jail configurations: [sshd.conf](setup-fail2ban/sshd.conf), [ufw.conf](setup-fail2ban/ufw.conf)
  - SSH protection: [jail.d/](setup-fail2ban/jail.d/)

---

## 🛠️ Additional Modules

### [Node.js](modules/node/)
Node.js installation and version management

### [Docker](modules/docker/)
Docker and container setup

### [PostgreSQL](modules/postgresql/)
PostgreSQL database configuration

### [NATS](modules/nats/)
NATS messaging system setup

---

## 🔒 Security Highlights

- ✅ Custom SSH port (20220) to reduce attack surface
- ✅ Key-based authentication only (no passwords)
- ✅ Fail2ban with SSH port protection (prevents lockouts)
- ✅ UFW firewall with rate limiting
- ✅ Database client SSH tunnel compatibility
- ✅ Root login disabled (KVM access only)

---

## 🤝 Contributing

These guides are based on production deployments and real-world troubleshooting. Feel free to suggest improvements or report issues.
