# File Permissions & Ownership Guide

A comprehensive guide to understanding and managing Linux file permissions, ownership, and resolving the most common "Permission denied" errors.

---

## Table of Contents

- [Understanding Linux File Permissions](#understanding-linux-file-permissions)
- [Permission Types and Notation](#permission-types-and-notation)
- [Basic Commands](#basic-commands)
- [Common Permission Scenarios](#common-permission-scenarios)
- [Special Permissions](#special-permissions)
- [Troubleshooting Permission Errors](#troubleshooting-permission-errors)
- [Best Practices](#best-practices)
- [Real-World Examples](#real-world-examples)
- [Quick Reference](#quick-reference)

---

## Understanding Linux File Permissions

Every file and directory in Linux has three types of permissions for three categories of users:

### Permission Categories:

- **Owner (u)**: The user who owns the file
- **Group (g)**: Users who belong to the file's group
- **Others (o)**: All other users on the system

### Permission Types:

- **Read (r)**: View file contents or list directory contents
- **Write (w)**: Modify file contents or create/delete files in directory
- **Execute (x)**: Run file as program or access directory

---

## Permission Types and Notation

### Symbolic Notation

```bash
# Example: -rwxr-xr--
# Position: -rwx r-x r--
#           owner group others

- or d    # File type (- = file, d = directory, l = link)
rwx       # Owner permissions (read, write, execute)
r-x       # Group permissions (read, no write, execute)
r--       # Others permissions (read only)
```

### Numeric (Octal) Notation

```bash
# Each permission has a numeric value:
# Read (r) = 4
# Write (w) = 2
# Execute (x) = 1

# Common permissions:
755 = rwxr-xr-x  # Owner: full, Group & Others: read+execute
644 = rw-r--r--  # Owner: read+write, Group & Others: read only
600 = rw-------  # Owner: read+write, Group & Others: no access
700 = rwx------  # Owner: full access, Group & Others: no access
666 = rw-rw-rw-  # Everyone: read+write
777 = rwxrwxrwx  # Everyone: full access (dangerous!)
```

### Directory vs File Permissions

```bash
# For Files:
# r = read file contents
# w = modify file contents
# x = execute file as program

# For Directories:
# r = list directory contents (ls)
# w = create/delete files in directory
# x = access directory (cd into it)
```

---

## Basic Commands

### chmod - Change File Permissions

```bash
# Numeric method
chmod 755 filename          # Set specific permissions
chmod 644 *.txt             # Set permissions for multiple files
chmod -R 755 directory/     # Recursive permission change

# Symbolic method
chmod u+x filename          # Add execute for owner
chmod g-w filename          # Remove write for group
chmod o+r filename          # Add read for others
chmod a+x filename          # Add execute for all (a = all)
chmod u=rwx,g=rx,o=r file   # Set exact permissions

# Multiple operations
chmod u+x,g-w,o+r filename  # Multiple changes at once
```

### chown - Change File Ownership

```bash
# Change owner only
chown newowner filename
chown root /etc/important.conf

# Change owner and group
chown newowner:newgroup filename
chown user:staff document.txt

# Change group only
chown :newgroup filename
chgrp newgroup filename     # Alternative for group only

# Recursive ownership change
chown -R user:group directory/

# Copy ownership from another file
chown --reference=other_file target_file
```

### chgrp - Change Group Ownership

```bash
# Change group
chgrp newgroup filename
chgrp staff *.txt
chgrp -R developers project/
```

---

## Common Permission Scenarios

### 1. Script Won't Execute

**Problem:**

```bash
$ ./script.sh
bash: ./script.sh: Permission denied
```

**Solution:**

```bash
# Check current permissions
ls -l script.sh
# Output: -rw-r--r-- 1 user user 128 Jan 1 12:00 script.sh

# Add execute permission
chmod +x script.sh
# or
chmod 755 script.sh

# Verify
ls -l script.sh
# Output: -rwxr-xr-x 1 user user 128 Jan 1 12:00 script.sh
```

### 2. Can't Access Directory

**Problem:**

```bash
$ cd /some/directory
bash: cd: /some/directory: Permission denied
```

**Solution:**

```bash
# Check directory permissions
ls -ld /some/directory
# Output: drw-r--r-- 2 root root 4096 Jan 1 12:00 /some/directory

# Add execute permission to access directory
sudo chmod +x /some/directory
# or
sudo chmod 755 /some/directory
```

### 3. Can't Create Files in Directory

**Problem:**

```bash
$ touch /some/directory/newfile
touch: cannot touch '/some/directory/newfile': Permission denied
```

**Solution:**

```bash
# Check directory permissions
ls -ld /some/directory
# Need write permission for the directory

# Add write permission
sudo chmod +w /some/directory
# or
sudo chmod 775 /some/directory

# Or change ownership
sudo chown $USER /some/directory
```

### 4. File Owned by Root

**Problem:**

```bash
$ echo "data" > /etc/myconfig
bash: /etc/myconfig: Permission denied
```

**Solution:**

```bash
# Option 1: Use sudo
sudo echo "data" > /etc/myconfig
# or
echo "data" | sudo tee /etc/myconfig

# Option 2: Change ownership (if appropriate)
sudo chown $USER /etc/myconfig
echo "data" > /etc/myconfig

# Option 3: Change permissions (less secure)
sudo chmod 666 /etc/myconfig
echo "data" > /etc/myconfig
```

### 5. SSH Key Permissions

**Problem:**

```bash
$ ssh user@server
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '/home/user/.ssh/id_rsa' are too open.
```

**Solution:**

```bash
# Fix SSH key permissions
chmod 700 ~/.ssh                    # SSH directory
chmod 600 ~/.ssh/id_rsa            # Private key
chmod 644 ~/.ssh/id_rsa.pub        # Public key
chmod 600 ~/.ssh/authorized_keys   # Authorized keys
chmod 600 ~/.ssh/config            # SSH config
```

---

## Special Permissions

### SUID (Set User ID)

```bash
# SUID bit allows file to run with owner's privileges
chmod u+s filename
chmod 4755 filename    # 4 = SUID bit

# Example: passwd command
ls -l /usr/bin/passwd
# Output: -rwsr-xr-x 1 root root 59640 passwd
#          ^
#          SUID bit (s instead of x)
```

### SGID (Set Group ID)

```bash
# SGID on file: runs with group privileges
chmod g+s filename
chmod 2755 filename    # 2 = SGID bit

# SGID on directory: new files inherit directory's group
chmod g+s directory/
chmod 2775 directory/
```

### Sticky Bit

```bash
# Sticky bit on directory: only owner can delete files
chmod +t directory/
chmod 1777 directory/  # 1 = sticky bit

# Example: /tmp directory
ls -ld /tmp
# Output: drwxrwxrwt 8 root root 4096 /tmp
#                  ^
#                  sticky bit (t instead of x)
```

### Combining Special Permissions

```bash
# Multiple special permissions
chmod 6755 filename    # SUID + SGID
chmod 7777 directory/  # SUID + SGID + Sticky (very dangerous!)
```

---

## Troubleshooting Permission Errors

### Common Error Messages and Solutions

#### "Permission denied"

```bash
# Check file permissions
ls -l filename

# Check directory permissions (for file access)
ls -ld /path/to/

# Check ownership
ls -l filename

# Solutions:
chmod +r filename       # Add read permission
chmod +w filename       # Add write permission
chmod +x filename       # Add execute permission
chown $USER filename    # Change ownership
sudo command           # Run with elevated privileges
```

#### "Operation not permitted"

```bash
# Usually ownership or special attribute issue

# Check file attributes
lsattr filename

# Remove immutable attribute
sudo chattr -i filename

# Check if file is on read-only filesystem
mount | grep "$(df . | tail -1 | awk '{print $1}')"
```

#### "Text file busy"

```bash
# File is being executed, can't modify

# Find processes using the file
lsof filename
fuser filename

# Kill processes or wait for them to finish
sudo fuser -k filename
```

### Debugging Permission Issues

```bash
# Check effective user and groups
id
whoami
groups

# Check process permissions
ps aux | grep process_name

# Check filesystem mount options
mount | grep filesystem

# Check SELinux context (if applicable)
ls -Z filename
getenforce

# Check AppArmor status (if applicable)
sudo aa-status
```

---

## Best Practices

### Security Guidelines

```bash
# 1. Principle of least privilege
# Give minimum permissions necessary

# 2. Safe default permissions for files
chmod 644 regular_files     # rw-r--r--
chmod 755 directories      # rwxr-xr-x
chmod 755 executables      # rwxr-xr-x

# 3. Secure permissions for sensitive files
chmod 600 private_keys     # rw-------
chmod 600 config_files     # rw-------
chmod 700 private_dirs     # rwx------

# 4. Never use 777 unless absolutely necessary
# chmod 777 is dangerous - everyone has full access
```

### Common Permission Patterns

```bash
# Web server files
sudo chown -R www-data:www-data /var/www/
sudo chmod -R 755 /var/www/
sudo chmod -R 644 /var/www/html/*.html

# Log files
sudo chown syslog:adm /var/log/myapp.log
sudo chmod 640 /var/log/myapp.log

# Configuration files
sudo chown root:root /etc/myapp.conf
sudo chmod 644 /etc/myapp.conf

# Backup scripts
chmod 700 backup_script.sh
chown root:root backup_script.sh

# Shared directory
sudo mkdir /shared
sudo chown :staff /shared
sudo chmod 2775 /shared    # SGID + group write
```

### umask - Default Permissions

```bash
# Check current umask
umask

# Set umask (subtracts from 777 for dirs, 666 for files)
umask 022    # Results in 755 for dirs, 644 for files
umask 002    # Results in 775 for dirs, 664 for files
umask 077    # Results in 700 for dirs, 600 for files

# Set umask permanently
echo "umask 022" >> ~/.bashrc
echo "umask 022" >> ~/.zshrc
```

---

## Real-World Examples

### Web Development Setup

```bash
# Create project directory with proper permissions
sudo mkdir -p /var/www/myproject
sudo chown -R $USER:www-data /var/www/myproject
sudo chmod -R 755 /var/www/myproject

# Set permissions for uploaded files
sudo chmod -R 644 /var/www/myproject/uploads/
sudo chmod 755 /var/www/myproject/uploads/

# Make scripts executable
chmod +x /var/www/myproject/scripts/*.sh
```

### Docker Container Setup

```bash
# Fix Docker socket permissions
sudo usermod -aG docker $USER
sudo chmod 666 /var/run/docker.sock

# Container volume permissions
sudo chown -R 1000:1000 /docker/volumes/app/
chmod -R 755 /docker/volumes/app/
```

### Git Repository Setup

```bash
# Shared Git repository
sudo mkdir /git/shared-repo.git
cd /git/shared-repo.git
sudo git init --bare --shared=group
sudo chown -R :developers /git/shared-repo.git
sudo chmod -R g+ws /git/shared-repo.git
```

### Database Files

```bash
# MySQL data directory
sudo chown -R mysql:mysql /var/lib/mysql/
sudo chmod -R 750 /var/lib/mysql/

# PostgreSQL data directory
sudo chown -R postgres:postgres /var/lib/postgresql/
sudo chmod -R 700 /var/lib/postgresql/
```

### System Service Files

```bash
# Systemd service files
sudo chown root:root /etc/systemd/system/myservice.service
sudo chmod 644 /etc/systemd/system/myservice.service

# Service executable
sudo chown root:root /usr/local/bin/myservice
sudo chmod 755 /usr/local/bin/myservice

# Service configuration
sudo chown root:myservice /etc/myservice/config.conf
sudo chmod 640 /etc/myservice/config.conf
```

---

## Quick Reference

### Essential Commands

```bash
# View permissions
ls -l filename              # Long listing with permissions
ls -ld directory/           # Directory permissions
stat filename               # Detailed file information

# Change permissions
chmod 755 file              # Numeric method
chmod u+x file              # Symbolic method
chmod -R 644 directory/     # Recursive

# Change ownership
chown user:group file       # Change owner and group
chown user file             # Change owner only
chgrp group file            # Change group only

# Special permissions
chmod u+s file              # Set SUID
chmod g+s file              # Set SGID
chmod +t directory/         # Set sticky bit

# Check current user
whoami                      # Current username
id                          # User ID and groups
groups                      # User's groups
```

### Permission Numbers Quick Reference

```
0 = ---  (no permissions)
1 = --x  (execute only)
2 = -w-  (write only)
3 = -wx  (write and execute)
4 = r--  (read only)
5 = r-x  (read and execute)
6 = rw-  (read and write)
7 = rwx  (full permissions)

Common combinations:
644 = rw-r--r--  (files)
755 = rwxr-xr-x  (directories, executables)
600 = rw-------  (private files)
700 = rwx------  (private directories)
```

### Troubleshooting Checklist

```bash
# 1. Check file permissions
ls -l filename

# 2. Check directory permissions
ls -ld /path/to/directory/

# 3. Check ownership
ls -l filename

# 4. Check your user and groups
id

# 5. Try with sudo
sudo command

# 6. Check file attributes
lsattr filename

# 7. Check processes using file
lsof filename
```

---

## Advanced Topics

### ACLs (Access Control Lists)

```bash
# Install ACL tools
sudo apt install acl

# View ACLs
getfacl filename

# Set ACL permissions
setfacl -m u:username:rwx filename
setfacl -m g:groupname:rx filename

# Remove ACLs
setfacl -x u:username filename
setfacl -b filename    # Remove all ACLs
```

### SELinux Contexts (if applicable)

```bash
# View SELinux context
ls -Z filename

# Change SELinux context
chcon -t httpd_exec_t /path/to/script

# Restore default context
restorecon filename
```

### Finding Files with Specific Permissions

```bash
# Find files with specific permissions
find /path -perm 777                    # Exact permissions
find /path -perm -644                   # At least these permissions
find /path -perm /644                   # Any of these permissions

# Find files with SUID/SGID
find /path -perm -u+s                   # SUID files
find /path -perm -g+s                   # SGID files

# Find world-writable files (security check)
find /path -perm -o+w
```

---

## Getting Help

- Manual pages: `man chmod`, `man chown`, `man chgrp`
- Permission calculator: Online tools for converting between numeric and symbolic
- Test permissions: Use `test` command or `[[ ]]` in scripts
- Community forums: Stack Overflow, Reddit r/linuxquestions

---

_This guide covers the most common file permission scenarios. Understanding permissions is fundamental to Linux security and system administration._
