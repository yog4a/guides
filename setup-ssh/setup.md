# SSH Step-By-Step Setup & Best Practices

This guide provides comprehensive instructions for setting up secure SSH access on a VPS. It includes creating a new user, configuring SSH keys, hardening SSH configuration, and testing the connection.

---

## 1. Create New User with Sudo Privileges

Create a non-root user for enhanced security:

```bash
sudo adduser XXXXXXXXXX
sudo usermod -aG sudo XXXXXXXXXX
```

**Tip:** Replace `XXXXXXXXXX` with your desired username.

---

## 2. Switch to New User

```bash
su - XXXXXXXXXX
```

---

## 3. Create SSH Directory with Proper Permissions

Set up the `.ssh` directory with secure permissions:

```bash
sudo mkdir -p "$HOME/.ssh"
sudo chmod 700 "$HOME/.ssh"
sudo chown -R "$(whoami)":"$(whoami)" "$HOME/.ssh"
sudo ls -ld "$HOME/.ssh"
```

---

## 4. Create Authorized Keys File

Create and secure the `authorized_keys` file:

```bash
sudo touch "$HOME/.ssh/authorized_keys"
sudo chmod 600 "$HOME/.ssh/authorized_keys"
sudo chown -R "$(whoami)":"$(whoami)" "$HOME/.ssh/authorized_keys"
sudo ls -l "$HOME/.ssh/authorized_keys"
```

---

## 5. Copy SSH Public Key to Server

### On Local Machine: Verify Your SSH Key

```bash
cat ~/.ssh/id_ed25519.pub
```

### Generate Copy Command

Run this on the server to get the command for your local machine:

```bash
public_ip=$(curl -4 -s ifconfig.me); echo "ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 22 $(whoami)@${public_ip}"
```

**Note:** Execute the generated command on your local machine.

---

## 6. Verify SSH Key on Server

Confirm the public key was copied correctly:

```bash
sudo cat "$HOME/.ssh/authorized_keys"
```

---

## 7. Backup and Configure SSH

### Backup Existing SSH Configuration

```bash
sudo cp "/etc/ssh/sshd_config" "/etc/ssh/sshd_config.$(date +%F-%T)"
ls -lh /etc/ssh/sshd_config*
```

### Temporarily Modify Permissions for Editing

```bash
sudo chmod 666 "/etc/ssh/sshd_config"
```

### Configure SSH Settings

**⚠️ Important:** Edit `/etc/ssh/sshd_config` with the content from `ssh.conf`:
- Change SSH port to a custom port (recommended: 1024-65535, e.g., 20220)
- Update `AllowUsers` with your username
- Update `DenyUsers` to restrict root access (allows root only via KVM)

### List Users with Shell Access

```bash
grep -E '/bin/(bash|sh|zsh)$' /etc/passwd | cut -d: -f1
```

Add these users to `AllowUsers` or `DenyUsers` as needed.

### Restore Original Permissions

```bash
sudo chmod 640 "/etc/ssh/sshd_config"
```

---

## 8. Test SSH Configuration

Validate the configuration before restarting:

```bash
sudo sshd -t
```

**Expected:** No output means configuration is valid ✅

Check SSH service status:

```bash
sudo systemctl status ssh
```

---

## 9. Disable SSH Socket (Recommended)

We disable `ssh.socket` for:
- Better compatibility with custom ports (e.g., 20220)
- Improved fail2ban monitoring
- Simpler configuration and troubleshooting

```bash
sudo systemctl disable ssh.socket
sudo systemctl mask ssh.socket
```

---

## 10. Temporarily Disable Security Services During Setup

Prevent lockout during configuration:

```bash
sudo systemctl stop ufw && sudo systemctl disable ufw
sudo systemctl stop fail2ban && sudo systemctl disable fail2ban
```

**Note:** Re-enable these services after confirming SSH access works.

---

## 11. Restart SSH Service

Apply the new configuration:

```bash
sudo systemctl restart ssh
```

Alternative commands if needed:

```bash
sudo systemctl restart sshd
```

Or full reload:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ssh.service
```

---

## 12. Test SSH Connection from Local Machine

### Generate Test Command on Server

```bash
public_ip=$(curl -4 -s ifconfig.me); echo "ssh -vvv -i ~/.ssh/id_ed25519 -p 20220 $(whoami)@${public_ip}"
```

**Note:** Execute the generated command on your local machine. Adjust port if you used a different one.

### Verify Connection

If successful, you should be logged in. Exit with:

```bash
exit
```

---

## 13. Add SSH Config on Local Machine

Add the following to your local `~/.ssh/config` file for easier access:

```
Host your_vps_alias
  HostName your_vps_ip
  User XXXXXXXXXX
  Port 20220
  IdentityFile ~/.ssh/id_ed25519
```

Then connect simply with:

```bash
ssh your_vps_alias
```

---

## Additional Tips

- **Always keep a backup terminal session open** when modifying SSH config to avoid lockout.
- **Test configuration changes** with `sudo sshd -t` before restarting the service.
- **Re-enable firewall and fail2ban** after confirming SSH access works properly.
- **Use strong SSH keys** (ED25519 or RSA 4096-bit minimum).
- For more SSH hardening options, see [`man sshd_config`](https://manpages.ubuntu.com/manpages/latest/en/man5/sshd_config.5.html).

---

## SSH Socket Configuration (Legacy/Deprecated)

If you need to use `ssh.socket` with a custom port, update the configuration:

```bash
sudo systemctl edit --full ssh.socket
```

Edit the `ListenStream` lines:

```
ListenStream=0.0.0.0:20220
ListenStream=[::]:20220
```

The SSH service runs as a standard daemon via `ssh.service`, which is the recommended approach.
