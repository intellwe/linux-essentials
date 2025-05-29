# System Services & systemctl Guide

A comprehensive guide to managing Linux system services using systemd and systemctl, including troubleshooting service failures and daemon management.

---

## Table of Contents

- [Introduction to systemd](#introduction-to-systemd)
- [Basic systemctl Commands](#basic-systemctl-commands)
- [Service Status and Information](#service-status-and-information)
- [Managing Service States](#managing-service-states)
- [Common Service Issues](#common-service-issues)
- [Creating Custom Services](#creating-custom-services)
- [Service Dependencies](#service-dependencies)
- [Logs and Monitoring](#logs-and-monitoring)
- [Systemd Timers](#systemd-timers)
- [Boot Process Troubleshooting](#boot-process-troubleshooting)
- [Best Practices](#best-practices)
- [Troubleshooting Checklist](#troubleshooting-checklist)

---

## Introduction to systemd

systemd is the init system and service manager for most modern Linux distributions. It manages:

- **Services**: Background processes and daemons
- **Sockets**: Network and IPC communication endpoints
- **Timers**: Scheduled tasks (cron replacement)
- **Mounts**: Filesystem mount points
- **Devices**: Hardware device management
- **Targets**: System states (runlevels)

### Key Concepts

```bash
# Unit Types
.service    # Services and daemons
.socket     # Network sockets
.timer      # Scheduled tasks
.mount      # Mount points
.target     # Groups of units
.path       # Path-based activation

# Unit Locations
/lib/systemd/system/        # System units (distribution)
/etc/systemd/system/        # System units (admin override)
/run/systemd/system/        # Runtime units
~/.config/systemd/user/     # User units
```

---

## Basic systemctl Commands

### Essential Service Management

```bash
# Start a service
sudo systemctl start service-name

# Stop a service
sudo systemctl stop service-name

# Restart a service
sudo systemctl restart service-name

# Reload service configuration (without restart)
sudo systemctl reload service-name

# Restart or reload if supported
sudo systemctl reload-or-restart service-name

# Enable service (start at boot)
sudo systemctl enable service-name

# Disable service (don't start at boot)
sudo systemctl disable service-name

# Enable and start immediately
sudo systemctl enable --now service-name

# Disable and stop immediately
sudo systemctl disable --now service-name
```

### System Control

```bash
# Reboot system
sudo systemctl reboot

# Shutdown system
sudo systemctl poweroff

# Hibernate system
sudo systemctl hibernate

# Suspend system
sudo systemctl suspend

# Emergency mode
sudo systemctl emergency

# Rescue mode
sudo systemctl rescue
```

---

## Service Status and Information

### Checking Service Status

```bash
# Check service status
systemctl status service-name

# Check if service is active
systemctl is-active service-name

# Check if service is enabled
systemctl is-enabled service-name

# Check if service failed
systemctl is-failed service-name

# List all services
systemctl list-units --type=service

# List only running services
systemctl list-units --type=service --state=running

# List failed services
systemctl list-units --type=service --state=failed

# List all unit files
systemctl list-unit-files

# List enabled services
systemctl list-unit-files --state=enabled
```

### Detailed Service Information

```bash
# Show service properties
systemctl show service-name

# Show specific property
systemctl show service-name --property=MainPID

# Show service dependencies
systemctl list-dependencies service-name

# Show reverse dependencies (what depends on this)
systemctl list-dependencies service-name --reverse

# Show service file content
systemctl cat service-name

# Edit service file
sudo systemctl edit service-name

# Edit full service file
sudo systemctl edit --full service-name
```

---

## Managing Service States

### Service Lifecycle

```bash
# Service states
inactive    # Not running
active      # Running successfully
failed      # Crashed or failed to start
activating  # Starting up
deactivating # Shutting down

# Enable states
enabled     # Will start at boot
disabled    # Will not start at boot
static      # Cannot be enabled/disabled
masked      # Completely disabled
```

### Advanced Service Control

```bash
# Mask service (prevent any activation)
sudo systemctl mask service-name

# Unmask service
sudo systemctl unmask service-name

# Reset failed state
sudo systemctl reset-failed service-name

# Reset all failed services
sudo systemctl reset-failed

# Kill service forcefully
sudo systemctl kill service-name

# Kill with specific signal
sudo systemctl kill --signal=SIGTERM service-name

# Reload systemd configuration
sudo systemctl daemon-reload
```

---

## Common Service Issues

### 1. Service Failed to Start

**Error:**

```
Job for service-name.service failed because the control process exited with error code.
```

**Diagnosis:**

```bash
# Check detailed status
systemctl status service-name -l

# Check recent logs
journalctl -u service-name -n 50

# Check logs since last boot
journalctl -u service-name -b

# Follow logs in real-time
journalctl -u service-name -f

# Check service file
systemctl cat service-name
```

**Common Solutions:**

```bash
# Fix configuration file permissions
sudo chmod 644 /etc/service-name/config.conf
sudo chown root:root /etc/service-name/config.conf

# Check if executable exists and is executable
ls -la /usr/bin/service-executable
sudo chmod +x /usr/bin/service-executable

# Validate configuration syntax
service-name --test-config
service-name --check-config

# Fix SELinux context (if applicable)
sudo restorecon -R /usr/bin/service-executable

# Reset failed state and try again
sudo systemctl reset-failed service-name
sudo systemctl start service-name
```

### 2. Service Starts but Immediately Fails

**Diagnosis:**

```bash
# Check exit codes
systemctl show service-name --property=ExecMainStatus

# Monitor service startup
sudo strace -f -e trace=execve systemctl start service-name

# Check file access
sudo strace -f -e trace=openat systemctl start service-name

# Run service manually to debug
sudo -u service-user /usr/bin/service-executable --debug
```

**Solutions:**

```bash
# Fix missing dependencies
sudo apt install missing-package

# Create missing directories
sudo mkdir -p /var/log/service-name
sudo chown service-user:service-group /var/log/service-name

# Fix user/group issues
sudo useradd -r service-user
sudo usermod -a -G required-group service-user

# Check and fix file permissions
sudo chmod 755 /var/lib/service-name
sudo chown -R service-user:service-group /var/lib/service-name
```

### 3. Service Won't Stop

**Issue:** Service hangs during shutdown.

**Solutions:**

```bash
# Check what's preventing shutdown
systemctl list-jobs

# Kill service forcefully
sudo systemctl kill --signal=SIGKILL service-name

# Set timeout for service
sudo systemctl edit service-name
# Add:
[Service]
TimeoutStopSec=30s

# Check for hung processes
sudo lsof | grep service-name
sudo fuser -v /path/to/service/files

# Force kill related processes
sudo pkill -f service-name
```

### 4. Service Keeps Restarting

**Issue:** Service enters restart loop.

**Diagnosis:**

```bash
# Check restart configuration
systemctl show service-name --property=Restart

# Monitor restart attempts
journalctl -u service-name -f

# Check system load
top
htop
systemctl status
```

**Solutions:**

```bash
# Modify restart policy
sudo systemctl edit service-name
# Add:
[Service]
Restart=on-failure
RestartSec=10s
StartLimitBurst=3
StartLimitInterval=60s

# Fix underlying issue causing crashes
# Check logs for specific error patterns
journalctl -u service-name | grep -i error

# Disable automatic restart temporarily
sudo systemctl edit service-name
# Add:
[Service]
Restart=no
```

### 5. Port Already in Use

**Error:**

```
bind: Address already in use
```

**Solutions:**

```bash
# Find what's using the port
sudo netstat -tlnp | grep :port-number
sudo ss -tlnp | grep :port-number
sudo lsof -i :port-number

# Kill process using the port
sudo kill -9 PID

# Change service port
sudo systemctl edit service-name
# Add configuration to use different port

# Stop conflicting service
sudo systemctl stop conflicting-service
sudo systemctl disable conflicting-service
```

---

## Creating Custom Services

### Basic Service Unit File

```bash
# Create service file
sudo nano /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Application Service
Documentation=https://example.com/docs
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/myapp --config=/etc/myapp/config.conf
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal

# Security settings
NoNewPrivileges=true
PrivateTmp=true
PrivateDevices=true
ProtectHome=true
ProtectSystem=strict
ReadWritePaths=/var/lib/myapp /var/log/myapp

[Install]
WantedBy=multi-user.target
```

### Service Types

```bash
# Type=simple (default)
# Service starts immediately, doesn't fork

# Type=forking
# Service forks and parent exits
[Service]
Type=forking
PIDFile=/var/run/myapp.pid

# Type=oneshot
# Service runs once and exits
[Service]
Type=oneshot
ExecStart=/usr/bin/setup-script

# Type=notify
# Service notifies systemd when ready
[Service]
Type=notify
NotifyAccess=main

# Type=idle
# Service waits until all jobs complete
[Service]
Type=idle
```

### Advanced Service Configuration

```bash
# Environment variables
[Service]
Environment="VAR1=value1"
Environment="VAR2=value2"
EnvironmentFile=/etc/myapp/environment

# Pre/post execution
[Service]
ExecStartPre=/usr/bin/setup-command
ExecStartPost=/usr/bin/post-setup
ExecStopPost=/usr/bin/cleanup

# Resource limits
[Service]
LimitNOFILE=65536
LimitNPROC=4096
MemoryLimit=1G
CPUQuota=50%

# Security hardening
[Service]
DynamicUser=true
PrivateNetwork=true
ProtectKernelTunables=true
ProtectControlGroups=true
RestrictRealtime=true
SystemCallFilter=@system-service
```

### Enabling Custom Service

```bash
# Reload systemd to read new service
sudo systemctl daemon-reload

# Enable and start service
sudo systemctl enable --now myapp.service

# Check status
systemctl status myapp.service

# View logs
journalctl -u myapp.service -f
```

---

## Service Dependencies

### Dependency Types

```bash
# Requires= - Hard dependency (fails if dependency fails)
[Unit]
Requires=network.target

# Wants= - Soft dependency (continues if dependency fails)
[Unit]
Wants=network-online.target

# After= - Start after (ordering)
[Unit]
After=network.target postgresql.service

# Before= - Start before (ordering)
[Unit]
Before=multi-user.target

# Conflicts= - Cannot run together
[Unit]
Conflicts=shutdown.target

# BindsTo= - Stop if dependency stops
[Unit]
BindsTo=docker.service
```

### Common Targets

```bash
# System targets
multi-user.target     # Multi-user mode (runlevel 3)
graphical.target      # Graphical mode (runlevel 5)
rescue.target         # Rescue mode (runlevel 1)
emergency.target      # Emergency mode
network.target        # Network interfaces up
network-online.target # Network fully configured

# Check current target
systemctl get-default

# Set default target
sudo systemctl set-default multi-user.target

# Change target temporarily
sudo systemctl isolate rescue.target
```

---

## Logs and Monitoring

### journalctl Basics

```bash
# View all logs
journalctl

# View logs for specific service
journalctl -u service-name

# Follow logs in real-time
journalctl -u service-name -f

# Show logs since last boot
journalctl -b

# Show logs for specific time
journalctl --since "2024-01-01 10:00:00"
journalctl --since "1 hour ago"
journalctl --until "2024-01-01 15:00:00"

# Show only errors
journalctl -p err

# Show logs in reverse order (newest first)
journalctl -r

# Show specific number of lines
journalctl -n 50

# Output in JSON format
journalctl -o json-pretty
```

### Advanced Log Analysis

```bash
# Search logs
journalctl -u service-name | grep -i error
journalctl --grep="pattern"

# Show logs with context
journalctl -u service-name -C 10

# Filter by priority
journalctl -p crit       # Critical errors only
journalctl -p warning    # Warnings and above

# Multiple units
journalctl -u service1 -u service2

# Kernel messages
journalctl -k

# Boot messages
journalctl -b -1         # Previous boot
journalctl --list-boots  # List all boots

# Disk usage
journalctl --disk-usage

# Clean old logs
sudo journalctl --vacuum-time=2weeks
sudo journalctl --vacuum-size=100M
```

### Log Configuration

```bash
# Configure journald
sudo nano /etc/systemd/journald.conf

# Common settings:
[Journal]
SystemMaxUse=100M        # Max disk usage
MaxRetentionSec=1month   # Keep logs for 1 month
MaxFileSec=1week         # Rotate weekly
Compress=yes             # Compress old logs
Storage=persistent       # Store on disk

# Restart journald after changes
sudo systemctl restart systemd-journald
```

---

## Systemd Timers

### Creating Timer Units

Timer units replace cron jobs in systemd environments.

```bash
# Create service file
sudo nano /etc/systemd/system/backup.service
```

```ini
[Unit]
Description=Daily Backup Job
Documentation=man:backup(8)

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup-script.sh
User=backup
StandardOutput=journal
```

```bash
# Create timer file
sudo nano /etc/systemd/system/backup.timer
```

```ini
[Unit]
Description=Run backup daily
Requires=backup.service

[Timer]
OnCalendar=daily
Persistent=true
RandomizedDelaySec=1800

[Install]
WantedBy=timers.target
```

### Timer Scheduling

```bash
# Time formats
OnCalendar=daily            # Every day at 00:00
OnCalendar=weekly           # Every Monday at 00:00
OnCalendar=monthly          # First day of month at 00:00
OnCalendar=*-*-01 03:00:00  # First day of every month at 3 AM
OnCalendar=Mon *-*-* 09:00:00  # Every Monday at 9 AM
OnCalendar=*:0/15           # Every 15 minutes

# Boot-relative timers
OnBootSec=15min             # 15 minutes after boot
OnStartupSec=30min          # 30 minutes after systemd start

# Unit-relative timers
OnUnitActiveSec=1h          # 1 hour after unit activation
OnUnitInactiveSec=30min     # 30 minutes after unit deactivation
```

### Managing Timers

```bash
# Enable and start timer
sudo systemctl enable --now backup.timer

# List all timers
systemctl list-timers

# Check timer status
systemctl status backup.timer

# Check when timer last ran
systemctl list-timers backup.timer

# Run timer service manually
sudo systemctl start backup.service

# View timer logs
journalctl -u backup.timer
journalctl -u backup.service
```

---

## Boot Process Troubleshooting

### Boot Analysis

```bash
# Analyze boot time
systemd-analyze

# Show service startup times
systemd-analyze blame

# Show critical chain
systemd-analyze critical-chain

# Plot boot process
systemd-analyze plot > boot-analysis.svg

# Show configuration
systemd-analyze dump

# Verify service files
systemd-analyze verify /etc/systemd/system/myservice.service
```

### Emergency Boot Recovery

```bash
# Boot into emergency mode
# Add to kernel command line: systemd.unit=emergency.target

# Boot into rescue mode
# Add to kernel command line: systemd.unit=rescue.target

# Boot with specific target
# Add to kernel command line: systemd.unit=multi-user.target

# Disable problematic service at boot
# Add to kernel command line: systemd.mask=problematic.service

# Debug systemd
# Add to kernel command line: systemd.log_level=debug systemd.log_target=console
```

### Recovery Commands

```bash
# Mount root filesystem read-write
mount -o remount,rw /

# Check filesystem
fsck /dev/sdaX

# Rebuild systemd dependencies
sudo systemctl daemon-reload

# Reset all failed services
sudo systemctl reset-failed

# Emergency shell
sulogin

# Start networking manually
systemctl start network.target
systemctl start NetworkManager
```

---

## Best Practices

### Service Management

```bash
# Always check status after changes
sudo systemctl start service-name
systemctl status service-name

# Use enable --now for new services
sudo systemctl enable --now service-name

# Check dependencies before stopping critical services
systemctl list-dependencies --reverse service-name

# Use reload instead of restart when possible
sudo systemctl reload service-name

# Test configuration before applying
sudo systemctl edit service-name --full
sudo systemctl daemon-reload
systemctl status service-name
```

### Security Considerations

```bash
# Use dedicated users for services
sudo useradd -r -s /bin/false service-user

# Apply principle of least privilege
[Service]
User=service-user
Group=service-group
PrivateTmp=true
ProtectSystem=strict
NoNewPrivileges=true

# Regular security audits
systemctl list-unit-files --state=enabled
journalctl -p warning --since "24 hours ago"

# Monitor failed services
systemctl list-units --state=failed
```

### Performance Optimization

```bash
# Identify slow-starting services
systemd-analyze blame

# Optimize service dependencies
systemd-analyze critical-chain service-name

# Use socket activation for infrequently used services
# Creates .socket unit that starts service on demand

# Disable unnecessary services
sudo systemctl disable unnecessary-service
sudo systemctl mask completely-unused-service

# Use resource limits
[Service]
MemoryLimit=512M
CPUQuota=25%
```

---

## Troubleshooting Checklist

### Service Won't Start

```bash
# 1. Check service status
systemctl status service-name -l

# 2. Check recent logs
journalctl -u service-name -n 50

# 3. Verify service file syntax
systemctl cat service-name
sudo systemd-analyze verify /etc/systemd/system/service-name.service

# 4. Check file permissions
ls -la /etc/systemd/system/service-name.service
ls -la /usr/bin/service-executable

# 5. Check dependencies
systemctl list-dependencies service-name --failed

# 6. Try manual execution
sudo -u service-user /usr/bin/service-executable

# 7. Check system resources
df -h
free -h
systemctl status
```

### Service Fails Intermittently

```bash
# 1. Monitor service continuously
journalctl -u service-name -f

# 2. Check system logs for patterns
journalctl --since "24 hours ago" | grep -i error

# 3. Monitor system resources
top
iotop
nethogs

# 4. Check for memory leaks
systemctl show service-name --property=MemoryCurrent

# 5. Review restart policy
systemctl show service-name --property=Restart

# 6. Check external dependencies
ping dependency-server
telnet dependency-server port
```

### System Boot Issues

```bash
# 1. Analyze boot performance
systemd-analyze
systemd-analyze blame

# 2. Check failed units
systemctl list-units --state=failed

# 3. Check critical chain
systemd-analyze critical-chain

# 4. Review boot logs
journalctl -b

# 5. Check for dependency loops
systemd-analyze dump | grep -A5 -B5 "loop"

# 6. Verify target configuration
systemctl get-default
systemctl list-dependencies default.target
```

---

## Quick Reference

### Essential Commands

```bash
# Service control
sudo systemctl start/stop/restart service-name
sudo systemctl enable/disable service-name
sudo systemctl enable --now service-name

# Status checking
systemctl status service-name
systemctl is-active service-name
systemctl is-enabled service-name

# Listing services
systemctl list-units --type=service
systemctl list-units --state=failed
systemctl list-timers

# Logs
journalctl -u service-name
journalctl -u service-name -f
journalctl -b

# System control
sudo systemctl reboot
sudo systemctl poweroff
sudo systemctl daemon-reload
```

### Common Service Locations

```bash
# System services
/lib/systemd/system/        # Distribution defaults
/etc/systemd/system/        # Administrator overrides
/run/systemd/system/        # Runtime units

# User services
~/.config/systemd/user/     # User-specific services

# Service states
systemctl list-unit-files   # All unit files and states
systemctl list-units        # Active units
```

### Emergency Commands

```bash
# Reset failed services
sudo systemctl reset-failed

# Force kill service
sudo systemctl kill --signal=SIGKILL service-name

# Emergency mode
sudo systemctl emergency

# Rescue mode
sudo systemctl rescue

# Reload systemd
sudo systemctl daemon-reload
```

---

## Advanced Topics

### Socket Activation

```bash
# Create socket unit
sudo nano /etc/systemd/system/myapp.socket
```

```ini
[Unit]
Description=MyApp Socket

[Socket]
ListenStream=8080
Accept=false

[Install]
WantedBy=sockets.target
```

### User Services

```bash
# Enable user lingering (services start without login)
sudo loginctl enable-linger username

# User service commands
systemctl --user start service-name
systemctl --user enable service-name
journalctl --user -u service-name

# User service location
~/.config/systemd/user/service-name.service
```

### Container Integration

```bash
# Podman/Docker with systemd
podman generate systemd container-name > container.service
sudo mv container.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now container.service
```

---

## Getting Help

- Manual pages: `man systemctl`, `man systemd`, `man journalctl`
- systemd documentation: freedesktop.org/software/systemd/man/
- Community forums: Stack Overflow, Reddit r/linuxquestions
- Distribution-specific docs: Ubuntu, RHEL, Arch wikis

---

_This guide covers essential systemd and service management scenarios. Proper service management is crucial for system stability and security._
