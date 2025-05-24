# SSH Configuration & Troubleshooting Guide

A comprehensive guide for SSH setup, configuration, and troubleshooting common issues that Linux users encounter daily.

---

## Table of Contents

- [What is SSH?](#what-is-ssh)
- [Basic SSH Usage](#basic-ssh-usage)
- [SSH Key Authentication Setup](#ssh-key-authentication-setup)
- [SSH Configuration File](#ssh-configuration-file)
- [Common SSH Errors and Solutions](#common-ssh-errors-and-solutions)
- [SSH Security Best Practices](#ssh-security-best-practices)
- [SSH Agent and Key Management](#ssh-agent-and-key-management)
- [Advanced SSH Features](#advanced-ssh-features)
- [Troubleshooting Checklist](#troubleshooting-checklist)

---

## What is SSH?

SSH (Secure Shell) is a cryptographic network protocol for operating network services securely over an unsecured network. It's commonly used for:

- Remote server access
- File transfers (SCP, SFTP)
- Git repository access (GitHub, GitLab)
- Port forwarding and tunneling

---

## Basic SSH Usage

### Connecting to a Remote Server

```bash
# Basic connection
ssh username@hostname

# Connect with specific port
ssh -p 2222 username@hostname

# Connect with specific key
ssh -i /path/to/private/key username@hostname
```

### Common SSH Commands

```bash
# Copy files to remote server
scp file.txt username@hostname:/path/to/destination/

# Copy files from remote server
scp username@hostname:/path/to/file.txt ./

# Copy directories recursively
scp -r directory/ username@hostname:/path/to/destination/

# SSH with verbose output (for debugging)
ssh -v username@hostname
```

---

## SSH Key Authentication Setup

### Step 1: Generate SSH Key Pair

```bash
# Generate RSA key (recommended: 4096 bits)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Generate Ed25519 key (modern, secure)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Generate with custom filename
ssh-keygen -t rsa -b 4096 -f ~/.ssh/custom_key -C "your_email@example.com"
```

**During key generation:**

- Press Enter for default location (`~/.ssh/id_rsa`)
- Enter a strong passphrase (recommended) or leave empty
- Do NOT overwrite existing keys unless intentional

### Step 2: Copy Public Key to Remote Server

```bash
# Method 1: Using ssh-copy-id (easiest)
ssh-copy-id username@hostname

# Method 2: Using ssh-copy-id with specific key
ssh-copy-id -i ~/.ssh/id_rsa.pub username@hostname

# Method 3: Manual copy
cat ~/.ssh/id_rsa.pub | ssh username@hostname "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# Method 4: Copy and paste manually
cat ~/.ssh/id_rsa.pub
# Copy the output and paste it to ~/.ssh/authorized_keys on the remote server
```

### Step 3: Test Key Authentication

```bash
# Test connection
ssh username@hostname

# Test with verbose output
ssh -v username@hostname
```

---

## SSH Configuration File

Create `~/.ssh/config` to simplify SSH connections:

```bash
# Create or edit SSH config
nano ~/.ssh/config
```

### Basic Configuration Examples

```bash
# Simple host configuration
Host myserver
    HostName example.com
    User myusername
    Port 22
    IdentityFile ~/.ssh/id_rsa

# GitHub configuration
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa
    IdentitiesOnly yes

# Server with custom port and key
Host production
    HostName prod.example.com
    User deploy
    Port 2222
    IdentityFile ~/.ssh/production_key
    StrictHostKeyChecking no

# Multiple servers with same configuration
Host server1 server2 server3
    User admin
    Port 22
    IdentityFile ~/.ssh/admin_key

# Wildcard configuration
Host *.example.com
    User myuser
    IdentityFile ~/.ssh/example_key
```

### Advanced Configuration Options

```bash
Host jumpserver
    HostName jump.example.com
    User admin
    IdentityFile ~/.ssh/jump_key

    # Connection multiplexing (faster subsequent connections)
    ControlMaster auto
    ControlPath ~/.ssh/control-%h-%p-%r
    ControlPersist 300

    # Keep connection alive
    ServerAliveInterval 60
    ServerAliveCountMax 3

    # Compression
    Compression yes

    # X11 forwarding
    ForwardX11 yes
```

### Set Proper Permissions

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/authorized_keys
chmod 400 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```

---

## Common SSH Errors and Solutions

### 1. Connection Refused (Port 22)

**Error:**

```
ssh: connect to host example.com port 22: Connection refused
```

**Solutions:**

```bash
# Check if SSH service is running on remote server
sudo systemctl status ssh
sudo systemctl start ssh
sudo systemctl enable ssh

# Check if SSH is listening on correct port
sudo netstat -tlnp | grep :22
sudo ss -tlnp | grep :22

# Try different port
ssh -p 2222 username@hostname

# Check firewall
sudo ufw status
sudo ufw allow ssh
```

### 2. Permission Denied (Publickey)

**Error:**

```
Permission denied (publickey)
```

**Solutions:**

```bash
# Check if key is added to SSH agent
ssh-add -l

# Add key to SSH agent
ssh-add ~/.ssh/id_rsa

# Verify public key is on remote server
ssh username@hostname "cat ~/.ssh/authorized_keys"

# Check key permissions
ls -la ~/.ssh/
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 400 ~/.ssh/id_rsa

# Debug with verbose output
ssh -v username@hostname
```

### 3. Host Key Verification Failed

**Error:**

```
Host key verification failed
```

**Solutions:**

```bash
# Remove old host key
ssh-keygen -R hostname

# Accept new host key
ssh -o StrictHostKeyChecking=no username@hostname

# Manually verify and add
ssh-keyscan hostname >> ~/.ssh/known_hosts
```

### 4. SSH Agent Issues

**Error:**

```
Could not open a connection to your authentication agent
```

**Solutions:**

```bash
# Start SSH agent
eval $(ssh-agent)

# Or start with specific shell
ssh-agent bash

# Add keys to agent
ssh-add ~/.ssh/id_rsa

# List loaded keys
ssh-add -l

# Remove all keys from agent
ssh-add -D
```

### 5. Too Many Authentication Failures

**Error:**

```
Received disconnect from host: 2: Too many authentication failures
```

**Solutions:**

```bash
# Use specific identity file
ssh -o IdentitiesOnly=yes -i ~/.ssh/specific_key username@hostname

# Or add to SSH config
Host hostname
    IdentitiesOnly yes
    IdentityFile ~/.ssh/specific_key
```

---

## SSH Security Best Practices

### Server-Side Security

```bash
# Edit SSH daemon configuration
sudo nano /etc/ssh/sshd_config

# Recommended security settings:
Port 2222                          # Change default port
PermitRootLogin no                  # Disable root login
PasswordAuthentication no           # Disable password auth
PubkeyAuthentication yes            # Enable key auth
MaxAuthTries 3                      # Limit auth attempts
ClientAliveInterval 300             # Set timeout
ClientAliveCountMax 2
AllowUsers username1 username2      # Limit allowed users
Protocol 2                          # Use SSH protocol 2

# Restart SSH service after changes
sudo systemctl restart ssh
```

### Client-Side Security

```bash
# Use strong key types
ssh-keygen -t ed25519
ssh-keygen -t rsa -b 4096

# Always use passphrases for keys
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Regularly rotate keys
ssh-keygen -t ed25519 -f ~/.ssh/new_key

# Use SSH config for better security
Host *
    HashKnownHosts yes
    VisualHostKey yes
    StrictHostKeyChecking ask
```

---

## SSH Agent and Key Management

### Managing SSH Agent

```bash
# Check if agent is running
echo $SSH_AUTH_SOCK

# Start agent if not running
eval $(ssh-agent)

# Add agent to shell startup
echo 'eval $(ssh-agent)' >> ~/.bashrc

# For zsh users
echo 'eval $(ssh-agent)' >> ~/.zshrc
```

### Key Management

```bash
# List all keys in agent
ssh-add -l

# Add specific key
ssh-add ~/.ssh/id_rsa

# Add key with lifetime (in seconds)
ssh-add -t 3600 ~/.ssh/id_rsa

# Remove specific key
ssh-add -d ~/.ssh/id_rsa

# Remove all keys
ssh-add -D

# List public key fingerprints
ssh-add -L
```

### Multiple SSH Keys for Different Services

```bash
# Create keys for different services
ssh-keygen -t ed25519 -f ~/.ssh/github_key -C "github@email.com"
ssh-keygen -t ed25519 -f ~/.ssh/work_key -C "work@email.com"

# SSH config for multiple keys
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_key
    IdentitiesOnly yes

Host work-server
    HostName work.example.com
    User myuser
    IdentityFile ~/.ssh/work_key
    IdentitiesOnly yes
```

---

## Advanced SSH Features

### SSH Tunneling (Port Forwarding)

```bash
# Local port forwarding (access remote service locally)
ssh -L 8080:localhost:80 username@hostname

# Remote port forwarding (expose local service remotely)
ssh -R 8080:localhost:3000 username@hostname

# Dynamic port forwarding (SOCKS proxy)
ssh -D 8080 username@hostname

# Background tunneling
ssh -fN -L 8080:localhost:80 username@hostname
```

### SSH Jump Hosts (ProxyJump)

```bash
# Connect through jump host
ssh -J jumpuser@jumphost finaluser@finalhost

# SSH config for jump hosts
Host target
    HostName target.internal.com
    User myuser
    ProxyJump jumpuser@jumphost.com
```

### X11 Forwarding

```bash
# Enable X11 forwarding
ssh -X username@hostname

# Trusted X11 forwarding
ssh -Y username@hostname

# SSH config for X11
Host graphical-server
    HostName server.com
    User myuser
    ForwardX11 yes
    ForwardX11Trusted yes
```

---

## Troubleshooting Checklist

When SSH connection fails, check these in order:

### 1. Network Connectivity

```bash
# Test basic connectivity
ping hostname

# Test specific port
telnet hostname 22
nc -zv hostname 22
```

### 2. SSH Service Status

```bash
# On remote server
sudo systemctl status ssh
sudo journalctl -u ssh -f
```

### 3. Firewall Configuration

```bash
# Check firewall rules
sudo ufw status
sudo iptables -L
```

### 4. SSH Configuration

```bash
# Test SSH config syntax
sudo sshd -t

# Check effective configuration
sudo sshd -T
```

### 5. Key and Permission Issues

```bash
# Verify key permissions
ls -la ~/.ssh/

# Check SELinux (if applicable)
restorecon -R ~/.ssh/
```

### 6. Debug with Verbose Output

```bash
# Client-side debugging
ssh -vvv username@hostname

# Server-side debugging
sudo /usr/sbin/sshd -d -p 2222
```

---

## Quick Reference Commands

```bash
# Generate new SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy public key to server
ssh-copy-id username@hostname

# Connect with specific key
ssh -i ~/.ssh/key_name username@hostname

# Debug connection
ssh -v username@hostname

# Remove host from known_hosts
ssh-keygen -R hostname

# Start SSH agent and add key
eval $(ssh-agent) && ssh-add

# Test SSH config
ssh -T git@github.com
```

---

## Common Use Cases

### GitHub/GitLab SSH Setup

```bash
# Generate key for GitHub
ssh-keygen -t ed25519 -f ~/.ssh/github_key -C "your_email@example.com"

# Add to SSH config
echo "Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_key
    IdentitiesOnly yes" >> ~/.ssh/config

# Test connection
ssh -T git@github.com
```

### Multiple Server Management

```bash
# SSH config for multiple servers
Host prod
    HostName production.company.com
    User deploy
    IdentityFile ~/.ssh/production_key

Host staging
    HostName staging.company.com
    User deploy
    IdentityFile ~/.ssh/staging_key

Host dev
    HostName dev.company.com
    User developer
    IdentityFile ~/.ssh/dev_key
```

---

## Getting Help

- Check SSH manual: `man ssh`, `man ssh_config`, `man sshd_config`
- SSH debugging: Use `-v`, `-vv`, or `-vvv` flags
- Server logs: `sudo journalctl -u ssh`
- Community forums: Stack Overflow, Reddit r/linuxquestions

---

_This guide covers the most common SSH scenarios. For specific edge cases or advanced configurations, consult the SSH manual pages or seek community help._
