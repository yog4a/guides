# SSH Setup Guide for VPS

This guide provides step-by-step instructions for setting up SSH access on a VPS. It includes creating a new user, configuring SSH keys, and testing the connection.

## Create new user with sudo privileges

```bash
sudo adduser XXXXXXXXXX
sudo usermod -G sudo XXXXXXXXXX
```

## Switch to new user

```bash
su - XXXXXXXXXX
```

## Create SSH folder with proper permissions

```bash
sudo mkdir -p "$HOME/.ssh"
sudo chmod 700 "$HOME/.ssh"
sudo chown -R "$(whoami)":"$(whoami)" "$HOME/.ssh"
sudo ls -ld "$HOME/.ssh"
```

## Create authorized_keys file with proper permissions

```bash
sudo touch "$HOME/.ssh/authorized_keys"
sudo chmod 600 "$HOME/.ssh/authorized_keys"
sudo chown -R "$(whoami)":"$(whoami)" "$HOME/.ssh/authorized_keys"
sudo ls -l "$HOME/.ssh/authorized_keys"
```

## Local: Verify and copy SSH key to server

```bash
# Verify your SSH key locally
cat ~/.ssh/id_ed25519.pub
```

```bash
# Get the command to copy the SSH key to the server
public_ip=$(curl -4 -s ifconfig.me); echo "ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 22 $(whoami)@${public_ip}"
```

## Verify the SSH key on the server

```bash
sudo cat "$HOME/.ssh/authorized_keys"
```

## Backup and configure SSH config

```bash
# Backup SSH config
sudo cp "/etc/ssh/sshd_config" "/etc/ssh/sshd_config.$(date +%F-%T)"
ls -lh /etc/ssh/sshd_config*

# Edit SSH config file
sudo chmod 666 "/etc/ssh/sshd_config"

# Add ssh.conf file to /etc/ssh/sshd_config.d
# Edit AllowUsers XXXXXXXXXX ⚠️⚠️⚠️

# Restore original permissions
sudo chmod 640 "/etc/ssh/sshd_config"
```

## Test SSH configuration

```bash
sudo sshd -t
sudo systemctl status ssh
```

No output = config is valid ✅  
If there are errors = it will print them ❌

## Restart SSH service

```bash
sudo systemctl restart ssh
# OR
sudo systemctl restart sshd
```

## Local: Test SSH connection

```bash
public_ip=$(curl -4 -s ifconfig.me); echo "ssh -i ~/.ssh/id_ed25519 -p 22 $(whoami)@${public_ip}"
# If successful, close the terminal with "exit"
```

## Add SSH config on your local machine

Add the following to your local `~/.ssh/config` file:

```
Host your_vps_ip
  HostName your_vps_ip
  User XXXXXXXXXX
  Port 22
  IdentityFile ~/.ssh/id_ed25519
```
