# Linux Fundamentals: A Complete Beginner's Guide

---

## Table of Contents

1. [Core Linux Fundamentals](#1-core-linux-fundamentals)
2. [Process & System Management](#2-process--system-management)
3. [File System & Storage](#3-file-system--storage)
4. [Networking](#4-networking)
5. [Package Management](#5-package-management)
6. [Shell Scripting](#6-shell-scripting)
7. [Job Scheduling](#7-job-scheduling)
8. [Logs & Debugging](#8-logs--debugging)
9. [Security Basics](#9-security-basics)
10. [Process Isolation & Performance](#10-process-isolation--performance)
11. [Advanced Text Processing](#11-advanced-text-processing)
12. [Remote Systems & Server Handling](#12-remote-systems--server-handling)

---

# 1. Core Linux Fundamentals

---

## 1.1 File System Hierarchy

### Definition

The Linux file system hierarchy is a standardized tree structure that defines where files and directories are stored on the system. It is governed by the **Filesystem Hierarchy Standard (FHS)**.

### Explanation / Working

Unlike Windows, which uses drive letters (C:\, D:\), Linux uses a single root directory `/` from which all other directories branch. Every file and device on the system is accessible from this root.

### Why It Matters

Understanding the hierarchy allows you to navigate the system, locate configuration files, logs, binaries, and user data without confusion.

### Key Directories

| Directory | Purpose |
|-----------|---------|
| `/` | Root of the entire file system |
| `/bin` | Essential user binaries (e.g., `ls`, `cp`) |
| `/sbin` | System administration binaries (e.g., `fsck`, `reboot`) |
| `/etc` | System-wide configuration files |
| `/var` | Variable data: logs, spools, mail |
| `/home` | Home directories for regular users |
| `/root` | Home directory for the root user |
| `/tmp` | Temporary files (cleared on reboot) |
| `/usr` | User programs and libraries |
| `/lib` | Shared libraries required by binaries |
| `/dev` | Device files (disks, terminals, etc.) |
| `/proc` | Virtual filesystem exposing kernel/process info |
| `/sys` | Virtual filesystem for kernel and hardware info |
| `/mnt` | Temporary mount points |
| `/media` | Auto-mounted removable media |
| `/opt` | Optional/third-party software |
| `/boot` | Bootloader and kernel files |

### Practical Examples

```bash
# Navigate to the root directory
cd /

# List contents of /etc
ls /etc

# View a configuration file
cat /etc/hostname

# View logs directory
ls /var/log
```

### Common Mistakes / Notes

- Never delete files from `/bin`, `/sbin`, `/lib`, or `/boot` — this will break the system.
- `/tmp` is cleared on reboot. Do not store important data there.
- `/proc` and `/sys` are virtual — they do not consume disk space.

---

## 1.2 Everything Is a File

### Definition

In Linux, almost every system resource is represented as a file — including hardware devices, network sockets, processes, and inter-process communication channels.

### Explanation / Working

Linux uses a unified interface to interact with hardware and software through the file abstraction layer. This means tools like `cat`, `echo`, and `read` can interact with devices just as they do with regular files.

### Why It Matters

This design philosophy enables consistent, scriptable access to system resources using standard file operations.

### Types of Files in Linux

| Type | Symbol | Description |
|------|--------|-------------|
| Regular file | `-` | Text, binary, executables |
| Directory | `d` | Container for files |
| Symbolic link | `l` | Pointer to another file |
| Block device | `b` | Hard drives, USB drives |
| Character device | `c` | Keyboards, serial ports |
| Socket | `s` | Network/IPC communication |
| Named pipe | `p` | Inter-process communication |

### Practical Examples

```bash
# Check file type
file /dev/sda
file /etc/passwd

# Read CPU info from virtual file
cat /proc/cpuinfo

# Read memory info
cat /proc/meminfo

# Write to a terminal device
echo "Hello" > /dev/pts/0

# Identify file types with ls -l
ls -l /dev/ | head -20
```

### Common Mistakes / Notes

- `/dev/null` is a special file that discards all data written to it. It is often used to suppress output.
- Symbolic links can become broken if the target is deleted.

---

## 1.3 Permissions

### Definition

Linux file permissions control who can read, write, or execute a file or directory. Permissions are assigned to three categories: **owner**, **group**, and **others**.

### Explanation / Working

Each file has a permission string of 10 characters:

```
-rwxr-xr--
```

| Position | Meaning |
|----------|---------|
| 1st character | File type (`-` = file, `d` = directory, `l` = link) |
| 2nd–4th | Owner permissions (`rwx`) |
| 5th–7th | Group permissions (`r-x`) |
| 8th–10th | Others permissions (`r--`) |

Permission characters:
- `r` = read (4)
- `w` = write (2)
- `x` = execute (1)
- `-` = permission not granted

### Why It Matters

Permissions are fundamental to Linux security. Misconfigured permissions can allow unauthorized access or prevent legitimate processes from functioning.

### Commands / Syntax

```bash
# View permissions
ls -l filename

# Change permissions (symbolic)
chmod u+x filename        # Add execute for owner
chmod g-w filename        # Remove write from group
chmod o=r filename        # Set others to read only

# Change permissions (numeric/octal)
chmod 755 filename        # rwxr-xr-x
chmod 644 filename        # rw-r--r--
chmod 600 filename        # rw-------

# Change ownership
chown user filename
chown user:group filename

# Change group only
chgrp groupname filename

# Recursive permission change
chmod -R 755 /var/www/
chown -R www-data:www-data /var/www/
```

### Octal Permission Table

| Octal | Binary | Permission |
|-------|--------|------------|
| 7 | 111 | rwx |
| 6 | 110 | rw- |
| 5 | 101 | r-x |
| 4 | 100 | r-- |
| 3 | 011 | -wx |
| 2 | 010 | -w- |
| 1 | 001 | --x |
| 0 | 000 | --- |

### Practical Examples

```bash
# Make a script executable
chmod +x deploy.sh

# Set strict permissions on SSH key
chmod 600 ~/.ssh/id_rsa

# Set directory permissions for a web server
chmod 755 /var/www/html

# View permissions in detail
ls -la /etc/passwd
```

### Common Mistakes / Notes

- Setting `chmod 777` (rwxrwxrwx) on sensitive files is a critical security risk.
- Directories need the execute bit (`x`) set for users to enter them.
- Use `umask` to set default permissions for newly created files.

```bash
# View current umask
umask

# Set default umask
umask 022
```

---

## 1.4 Users & Groups

### Definition

Linux is a multi-user operating system. Each user has a unique identity, and users can belong to one or more groups which define their access rights.

### Explanation / Working

- **UID (User ID)**: Numeric identifier for a user. Root is always UID 0.
- **GID (Group ID)**: Numeric identifier for a group.
- User information is stored in `/etc/passwd`.
- Password hashes are stored in `/etc/shadow`.
- Group information is stored in `/etc/group`.

### Why It Matters

User and group management is essential for access control — ensuring users can only access what they are permitted to.

### Commands / Syntax

```bash
# Add a new user
useradd username
useradd -m -s /bin/bash username    # With home directory and shell

# Set or change password
passwd username

# Delete a user
userdel username
userdel -r username                 # Also remove home directory

# Modify a user
usermod -aG groupname username      # Add to group
usermod -s /bin/bash username       # Change shell
usermod -L username                 # Lock account
usermod -U username                 # Unlock account

# Add a group
groupadd groupname

# Delete a group
groupdel groupname

# View current user
whoami

# View user ID and group memberships
id
id username

# Switch user
su - username

# Execute as superuser
sudo command

# View all users
cat /etc/passwd

# View all groups
cat /etc/group

# View groups for current user
groups
```

### Practical Examples

```bash
# Create a user with home dir and bash shell
useradd -m -s /bin/bash john
passwd john

# Add john to the sudo group
usermod -aG sudo john

# Verify group membership
id john

# Switch to user john
su - john
```

### Common Mistakes / Notes

- Always use `useradd -m` to create a home directory; without `-m`, the home directory is not created.
- Use `usermod -aG` (append + group) instead of `usermod -G`, which replaces all existing group memberships.
- The root user (UID 0) bypasses all permission checks.

---

## 1.5 Environment Variables

### Definition

Environment variables are named values stored in the shell's environment that influence the behavior of processes and programs.

### Explanation / Working

When a process is launched, it inherits a copy of its parent's environment variables. Programs use these variables to configure their behavior without hardcoded values.

### Why It Matters

Environment variables control critical settings such as the executable search path (`PATH`), the current user (`USER`), the home directory (`HOME`), and shell behavior.

### Commands / Syntax

```bash
# View all environment variables
env
printenv

# View a specific variable
echo $HOME
echo $PATH
printenv PATH

# Set a variable (current session only)
MYVAR="hello"
export MYVAR

# Set a variable permanently (add to ~/.bashrc or ~/.bash_profile)
echo 'export MYVAR="hello"' >> ~/.bashrc
source ~/.bashrc

# Unset a variable
unset MYVAR

# Run a command with a temporary variable
MYVAR="test" command
```

### Common Environment Variables

| Variable | Description |
|----------|-------------|
| `PATH` | Directories searched for commands |
| `HOME` | Current user's home directory |
| `USER` | Current logged-in user |
| `SHELL` | Path to the current shell |
| `PWD` | Current working directory |
| `EDITOR` | Default text editor |
| `LANG` | System language/locale |
| `TERM` | Terminal type |

### Practical Examples

```bash
# Add a custom directory to PATH
export PATH=$PATH:/opt/myapp/bin

# Persist PATH change
echo 'export PATH=$PATH:/opt/myapp/bin' >> ~/.bashrc
source ~/.bashrc

# View PATH entries clearly
echo $PATH | tr ':' '\n'
```

### Common Mistakes / Notes

- Changes made with `export` are only effective for the current session.
- Use `~/.bashrc` for interactive shells and `~/.bash_profile` or `~/.profile` for login shells.
- Incorrect `PATH` modifications can make commands unavailable.

---

## 1.6 Core Commands

### Navigation

```bash
pwd                     # Print current directory
cd /path/to/dir         # Change directory
cd ~                    # Go to home directory
cd ..                   # Go up one level
cd -                    # Go to previous directory
ls                      # List files
ls -l                   # Long listing format
ls -la                  # Include hidden files
ls -lh                  # Human-readable file sizes
```

### File Operations

```bash
# Copy
cp source destination
cp -r sourcedir destdir        # Recursive (directories)
cp -p file dest                # Preserve permissions and timestamps

# Move / Rename
mv source destination
mv oldname newname

# Remove
rm filename
rm -r directoryname            # Recursive deletion
rm -f filename                 # Force (no prompt)
rm -rf directoryname           # Force recursive (use with caution)

# Create
touch filename                 # Create empty file or update timestamp
mkdir dirname                  # Create directory
mkdir -p a/b/c                 # Create nested directories
```

### Searching

```bash
# find: search by file properties
find /path -name "filename"
find /home -name "*.txt"
find / -type f -size +100M             # Files larger than 100MB
find /etc -mtime -7                    # Modified in last 7 days
find / -user john -type f              # Files owned by john
find /tmp -perm 777                    # Files with permission 777

# locate: fast search using a database
locate filename
updatedb                               # Update locate database (requires root)
```

### Viewing File Contents

```bash
cat filename                           # Print entire file
cat -n filename                        # With line numbers
less filename                          # Paginated view (q to quit)
more filename                          # Simpler paginated view
head filename                          # First 10 lines
head -n 20 filename                    # First 20 lines
tail filename                          # Last 10 lines
tail -n 50 filename                    # Last 50 lines
tail -f filename                       # Follow file in real time (logs)
```

### Text Processing

```bash
# grep: search for patterns
grep "pattern" filename
grep -i "pattern" filename             # Case-insensitive
grep -r "pattern" /path/              # Recursive
grep -n "pattern" filename             # Show line numbers
grep -v "pattern" filename             # Invert match (exclude)
grep -c "pattern" filename             # Count matches
grep -E "regex" filename               # Extended regex

# awk: column-based text processing
awk '{print $1}' filename              # Print first column
awk -F: '{print $1}' /etc/passwd       # Use colon as delimiter
awk '{sum += $1} END {print sum}' f    # Sum column

# sed: stream editor
sed 's/old/new/' filename              # Replace first occurrence
sed 's/old/new/g' filename             # Replace all occurrences
sed -i 's/old/new/g' filename          # Edit file in place
sed -n '5,10p' filename                # Print lines 5–10
sed '/pattern/d' filename              # Delete lines matching pattern
```

### Mini Cheat Sheet

| Command | Description |
|---------|-------------|
| `pwd` | Print working directory |
| `cd /path` | Change directory |
| `ls -la` | List all files with details |
| `cp -r src dst` | Copy directory |
| `mv src dst` | Move or rename |
| `rm -rf dir` | Delete directory recursively |
| `find / -name f` | Find file by name |
| `grep -r "x" .` | Search recursively |
| `cat file` | Display file content |
| `tail -f log` | Follow log file |

---

# 2. Process & System Management

---

## 2.1 Process Lifecycle

### Definition

A process is a running instance of a program. Every process in Linux is assigned a unique **PID (Process ID)** and is created by forking from a parent process.

### Explanation / Working

- The kernel initializes the first process: `init` or `systemd` (PID 1).
- All other processes are descendants of PID 1.
- The lifecycle: **Created → Running → Waiting → Stopped → Zombie → Terminated**

### Process States

| State | Symbol | Description |
|-------|--------|-------------|
| Running | R | Actively executing or in run queue |
| Sleeping | S | Waiting for an event (interruptible) |
| Sleeping | D | Uninterruptible sleep (e.g., I/O wait) |
| Stopped | T | Stopped by signal (e.g., SIGSTOP) |
| Zombie | Z | Terminated but parent hasn't read exit status |

### Practical Examples

```bash
# View running processes
ps aux

# View process tree
pstree

# Get PID of a process by name
pgrep nginx
pidof nginx
```

---

## 2.2 Foreground vs Background Jobs

### Definition

- **Foreground**: A process that occupies the terminal and blocks input until it completes.
- **Background**: A process that runs independently of the terminal.

### Commands / Syntax

```bash
# Run a command in the background
command &

# List background jobs
jobs

# Bring background job to foreground
fg %1

# Send foreground job to background
# Press Ctrl+Z to suspend, then:
bg %1

# Run a command that persists after logout
nohup command &

# Detach completely
disown %1
```

### Practical Examples

```bash
# Run a long backup script in background
./backup.sh &

# Check job status
jobs -l

# Bring job 1 to foreground
fg %1

# Prevent job from stopping on terminal close
nohup python3 server.py > server.log 2>&1 &
```

---

## 2.3 Signals

### Definition

Signals are software interrupts sent to processes to notify them of events. They can be sent by the kernel, other processes, or users.

### Common Signals

| Signal | Number | Description |
|--------|--------|-------------|
| SIGHUP | 1 | Hangup — reload config |
| SIGINT | 2 | Interrupt (Ctrl+C) |
| SIGQUIT | 3 | Quit with core dump |
| SIGKILL | 9 | Force kill (cannot be caught) |
| SIGTERM | 15 | Graceful termination (default) |
| SIGSTOP | 19 | Pause process (cannot be caught) |
| SIGCONT | 18 | Resume paused process |

### Commands / Syntax

```bash
# Send SIGTERM (graceful)
kill PID
kill -15 PID
kill -SIGTERM PID

# Force kill (SIGKILL)
kill -9 PID

# Kill by name
killall nginx
pkill -9 nginx

# Send specific signal
kill -1 PID          # SIGHUP (reload)
kill -SIGSTOP PID    # Pause
kill -SIGCONT PID    # Resume
```

### Practical Examples

```bash
# Gracefully stop a web server
kill -15 $(pgrep nginx)

# Force kill an unresponsive process
kill -9 1234

# Reload configuration without downtime
kill -HUP $(pgrep sshd)
```

---

## 2.4 System Monitoring

### Commands / Syntax

```bash
# ps: process snapshot
ps aux                         # All processes, detailed
ps -ef                         # Full-format listing
ps aux | grep nginx            # Filter by name
ps -p PID                      # Specific PID

# top: real-time monitoring
top                            # Interactive display
# Inside top: q=quit, k=kill, r=renice, M=sort by memory, P=sort by CPU

# htop: enhanced top (may need installation)
htop

# uptime
uptime                         # Shows load averages for 1, 5, 15 minutes

# free: memory usage
free -h                        # Human-readable
free -m                        # In megabytes

# vmstat: virtual memory statistics
vmstat 2 5                     # Report every 2 seconds, 5 times
```

### Understanding Load Average

Load average shows the number of processes waiting for CPU. Rule of thumb:

- Load average = number of CPU cores → system is fully utilized
- Load average > cores → system is overloaded

```bash
# Check number of CPU cores
nproc
lscpu
```

---

## 2.5 systemd

### Definition

`systemd` is the default init system and service manager for most modern Linux distributions. It manages the boot process, services (daemons), logging, and more.

### Key Concepts

- **Unit**: A configuration file describing a service, socket, mount, or timer.
- **Service**: A background daemon managed by systemd.
- **Target**: A group of units (similar to runlevels).

### Commands / Syntax

```bash
# Start/stop/restart a service
systemctl start servicename
systemctl stop servicename
systemctl restart servicename
systemctl reload servicename          # Reload config without full restart

# Enable/disable at boot
systemctl enable servicename
systemctl disable servicename

# Check service status
systemctl status servicename

# View all running services
systemctl list-units --type=service

# View all failed services
systemctl --failed

# View service logs
journalctl -u servicename
journalctl -u servicename -f          # Follow logs
journalctl -u servicename --since "1 hour ago"

# System targets (runlevels)
systemctl get-default                 # View current target
systemctl set-default multi-user.target

# Reboot / shutdown
systemctl reboot
systemctl poweroff
systemctl halt
```

### Practical Examples

```bash
# Enable and start nginx
systemctl enable nginx
systemctl start nginx
systemctl status nginx

# View last 100 lines of SSH logs
journalctl -u sshd -n 100

# View logs since specific time
journalctl -u nginx --since "2024-01-01" --until "2024-01-02"

# View kernel messages from boot
journalctl -k
```

### Common Mistakes / Notes

- `enable` does not start a service immediately — use both `enable` and `start`.
- `reload` is safer than `restart` for production services as it avoids downtime.
- Use `journalctl` instead of manually reading `/var/log/` for systemd-managed services.

---

# 3. File System & Storage

---

## 3.1 Disk Usage

### Commands / Syntax

```bash
# df: disk space usage
df -h                          # Human-readable, all filesystems
df -h /                        # Specific mount point
df -i                          # Inode usage

# du: directory/file sizes
du -h filename
du -sh directory               # Summary, human-readable
du -sh /*                      # Top-level directory sizes
du -ah /var/log | sort -rh | head -20   # Largest files in /var/log
```

### Practical Examples

```bash
# Find what's consuming space in /var
du -sh /var/* | sort -rh | head -10

# Check disk usage on all mounted filesystems
df -h

# Find top 10 largest files
find / -type f -exec du -h {} + 2>/dev/null | sort -rh | head -10
```

---

## 3.2 Mounting

### Definition

Mounting is the process of making a storage device or partition accessible at a specific directory (called the **mount point**) within the file system hierarchy.

### Commands / Syntax

```bash
# Mount a device
mount /dev/sdb1 /mnt/usb

# Mount with filesystem type
mount -t ext4 /dev/sdb1 /mnt/data

# Mount an ISO file
mount -o loop image.iso /mnt/iso

# Unmount
umount /mnt/usb
umount /dev/sdb1

# View currently mounted filesystems
mount
cat /proc/mounts

# Persistent mounts: add to /etc/fstab
# Format: device  mountpoint  fstype  options  dump  pass
# Example:
# /dev/sdb1  /data  ext4  defaults  0  2
```

### Common Mistakes / Notes

- Always unmount (`umount`) before physically removing storage devices to avoid data corruption.
- Entries in `/etc/fstab` persist across reboots.
- Use `blkid` to find device UUIDs for use in `/etc/fstab`:

```bash
blkid /dev/sdb1
```

---

## 3.3 File Systems

### Overview

A file system defines how data is organized and stored on a disk partition.

| File System | Use Case |
|------------|---------|
| ext4 | Default on Debian/Ubuntu — stable, journaling |
| xfs | Default on RHEL/CentOS — high performance, large files |
| btrfs | Snapshots, checksums, compression |
| vfat/FAT32 | USB drives, cross-platform compatibility |
| tmpfs | RAM-based filesystem (temporary storage) |

### Practical Examples

```bash
# Format a partition as ext4
mkfs.ext4 /dev/sdb1

# Format as xfs
mkfs.xfs /dev/sdb1

# Check filesystem integrity
fsck /dev/sdb1              # Run only on unmounted partitions

# Show filesystem type
lsblk -f
file -s /dev/sda1
```

---

## 3.4 Inodes

### Definition

An **inode** (index node) is a data structure on disk that stores metadata about a file — permissions, ownership, timestamps, size, and pointers to data blocks — but **not** the filename.

### Why It Matters

Every file consumes one inode. A system can run out of inodes before running out of disk space, causing errors even with available storage.

### Practical Examples

```bash
# View inode usage
df -i

# View inode number of a file
ls -li filename

# Find files by inode number
find / -inum 1234567
```

### Common Mistakes / Notes

- Running out of inodes is common on servers with millions of small files (e.g., mail servers, cache directories).
- Deleting large files does not free disk space if another process still holds the file descriptor open.

---

# 4. Networking

---

## 4.1 Core Networking Concepts

### Key Terms

| Term | Description |
|------|-------------|
| IP Address | Unique numeric identifier for a device on a network |
| Subnet Mask | Defines the network/host portions of an IP address |
| Gateway | Router connecting local network to the internet |
| DNS | Domain Name System — translates names to IP addresses |
| Port | Numeric endpoint for a specific service (0–65535) |
| TCP | Reliable, connection-oriented protocol |
| UDP | Faster, connectionless protocol — no delivery guarantee |

### localhost vs 0.0.0.0

- **127.0.0.1 / localhost**: Loopback address — refers to the machine itself. Services bound here are only accessible locally.
- **0.0.0.0**: Binds to all available network interfaces — accessible from any network.

### Common Ports

| Port | Service |
|------|---------|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 21 | FTP |
| 25 | SMTP |
| 53 | DNS |
| 3306 | MySQL |
| 5432 | PostgreSQL |

---

## 4.2 Networking Tools

### Commands / Syntax

```bash
# ping: test connectivity
ping hostname_or_ip
ping -c 4 google.com           # Send only 4 packets
ping -i 0.5 google.com         # Interval 0.5 seconds

# curl: HTTP requests and data transfer
curl https://example.com
curl -I https://example.com    # Headers only
curl -o file.html https://example.com    # Save to file
curl -u user:pass https://api.example.com
curl -X POST -d '{"key":"value"}' -H "Content-Type: application/json" https://api.example.com

# wget: download files
wget https://example.com/file.tar.gz
wget -O output.html https://example.com
wget -r https://example.com    # Recursive download

# netstat: network connections (older tool)
netstat -tuln                  # Listening TCP/UDP ports
netstat -anp                   # All connections with PIDs

# ss: modern replacement for netstat
ss -tuln                       # Listening ports
ss -anp                        # All connections with processes
ss -s                          # Summary statistics

# traceroute: trace network path
traceroute google.com
traceroute -n google.com       # No DNS resolution (faster)

# dig: DNS lookup
dig example.com
dig example.com A              # Query A record
dig example.com MX             # Query mail records
dig @8.8.8.8 example.com       # Use specific DNS server
dig +short example.com         # Concise output

# nslookup: DNS query (simpler)
nslookup example.com
nslookup example.com 8.8.8.8
```

### Practical Examples

```bash
# Check if a remote port is open
nc -zv hostname 22

# View current IP address
ip addr show
ip a

# View routing table
ip route show
route -n

# Check DNS resolution
dig +short google.com

# Test HTTP endpoint
curl -o /dev/null -s -w "%{http_code}" https://api.example.com
```

### Common Mistakes / Notes

- `netstat` is deprecated on many systems. Use `ss` instead.
- `curl` is preferred over `wget` for API testing due to its flexibility.
- `ping` uses ICMP — some firewalls block ICMP, causing false negatives.

---

# 5. Package Management

---

## 5.1 apt (Debian / Ubuntu)

### Definition

`apt` (Advanced Package Tool) manages software packages on Debian-based distributions.

### Commands / Syntax

```bash
# Update package list
apt update

# Upgrade all installed packages
apt upgrade
apt full-upgrade               # Also handles dependency changes

# Install a package
apt install packagename
apt install -y packagename     # Auto-confirm

# Remove a package
apt remove packagename
apt purge packagename          # Remove with config files
apt autoremove                 # Remove unused dependencies

# Search for a package
apt search keyword
apt-cache search keyword

# Show package info
apt show packagename
apt-cache show packagename

# List installed packages
dpkg -l
dpkg -l | grep nginx

# Check if package is installed
dpkg -l packagename
```

---

## 5.2 yum (RHEL / CentOS 7)

```bash
yum update
yum install packagename
yum remove packagename
yum search keyword
yum info packagename
yum list installed
yum clean all
```

---

## 5.3 dnf (RHEL 8+ / Fedora)

```bash
dnf update
dnf install packagename
dnf remove packagename
dnf search keyword
dnf info packagename
dnf list installed
dnf autoremove
dnf clean all
```

### Common Mistakes / Notes

- Always run `apt update` before `apt install` to fetch the latest package list.
- Use `apt purge` instead of `apt remove` when you want to fully clean a package including configuration.
- Avoid mixing package managers (e.g., do not use both `apt` and manual compilation for the same software without caution).

---

# 6. Shell Scripting

---

## 6.1 Bash Basics

### Definition

A shell script is a text file containing a sequence of shell commands. Bash (Bourne Again Shell) is the most commonly used shell for scripting on Linux.

### Structure of a Script

```bash
#!/bin/bash
# This is a comment

echo "Hello, World!"
```

- `#!/bin/bash` — the **shebang** line. It tells the OS which interpreter to use.
- Make executable: `chmod +x script.sh`
- Run: `./script.sh` or `bash script.sh`

---

## 6.2 Variables

```bash
#!/bin/bash

# Declare variable
NAME="Linux"
NUMBER=42

# Use variable
echo "Welcome to $NAME"
echo "The number is ${NUMBER}"

# Read user input
read -p "Enter your name: " USERNAME
echo "Hello, $USERNAME"

# Command substitution
CURRENT_DATE=$(date +%Y-%m-%d)
echo "Today is: $CURRENT_DATE"

# Arithmetic
A=10
B=5
echo $((A + B))
echo $((A * B))
```

---

## 6.3 Conditionals

```bash
#!/bin/bash

FILE="/etc/passwd"

# if / elif / else
if [ -f "$FILE" ]; then
    echo "$FILE exists."
elif [ -d "$FILE" ]; then
    echo "$FILE is a directory."
else
    echo "$FILE does not exist."
fi

# String comparison
NAME="admin"
if [ "$NAME" == "admin" ]; then
    echo "Admin user detected."
fi

# Numeric comparison
COUNT=5
if [ $COUNT -gt 3 ]; then
    echo "Count is greater than 3"
fi
```

### Test Operators

| Operator | Meaning |
|----------|---------|
| `-f file` | True if regular file exists |
| `-d dir` | True if directory exists |
| `-e path` | True if path exists |
| `-r file` | True if file is readable |
| `-w file` | True if file is writable |
| `-x file` | True if file is executable |
| `-z str` | True if string is empty |
| `-n str` | True if string is not empty |
| `str1 == str2` | String equality |
| `str1 != str2` | String inequality |
| `n1 -eq n2` | Numeric equality |
| `n1 -ne n2` | Not equal |
| `n1 -lt n2` | Less than |
| `n1 -gt n2` | Greater than |
| `n1 -le n2` | Less than or equal |
| `n1 -ge n2` | Greater than or equal |

---

## 6.4 Loops

```bash
#!/bin/bash

# for loop
for i in 1 2 3 4 5; do
    echo "Iteration: $i"
done

# for loop with range
for i in {1..10}; do
    echo "Number: $i"
done

# for loop C-style
for ((i=0; i<5; i++)); do
    echo "i = $i"
done

# while loop
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Count: $COUNT"
    ((COUNT++))
done

# loop over files
for FILE in /var/log/*.log; do
    echo "Processing: $FILE"
done

# read lines from a file
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/hosts
```

---

## 6.5 Functions

```bash
#!/bin/bash

# Define function
greet() {
    local NAME=$1
    echo "Hello, $NAME!"
}

# Call function
greet "John"
greet "Alice"

# Function with return value
add() {
    echo $(($1 + $2))
}

RESULT=$(add 5 10)
echo "Result: $RESULT"
```

---

## 6.6 Complete Automation Script Example

This script monitors disk usage and sends an alert if usage exceeds a threshold.

```bash
#!/bin/bash
# disk_monitor.sh - Monitor disk usage and log alerts

THRESHOLD=80
LOG_FILE="/var/log/disk_monitor.log"
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')

check_disk() {
    local MOUNT_POINT=$1
    local USAGE=$(df -h "$MOUNT_POINT" | awk 'NR==2 {gsub(/%/, "", $5); print $5}')

    if [ "$USAGE" -ge "$THRESHOLD" ]; then
        echo "[$TIMESTAMP] WARNING: $MOUNT_POINT is at ${USAGE}% usage" | tee -a "$LOG_FILE"
    else
        echo "[$TIMESTAMP] OK: $MOUNT_POINT is at ${USAGE}% usage"
    fi
}

# Check root and /var
check_disk "/"
check_disk "/var"

echo "[$TIMESTAMP] Disk check complete." >> "$LOG_FILE"
```

```bash
# Make executable and run
chmod +x disk_monitor.sh
./disk_monitor.sh
```

### Common Mistakes / Notes

- Always quote variables (`"$VAR"`) to handle spaces in values correctly.
- Use `local` inside functions to avoid polluting the global scope.
- Check exit codes using `$?` or `&&`/`||` operators.
- Use `set -e` at the top of scripts to exit on any error.
- Use `set -x` for debugging — prints each command before execution.

---

# 7. Job Scheduling

---

## 7.1 cron

### Definition

`cron` is a time-based job scheduler that runs commands at specified intervals automatically in the background.

### Explanation / Working

The `crond` daemon reads `crontab` files and executes scheduled tasks. Each user can have their own crontab, and a system-level crontab exists in `/etc/crontab`.

### Crontab Syntax

```
* * * * * command_to_run
│ │ │ │ │
│ │ │ │ └── Day of week (0–7, 0 and 7 = Sunday)
│ │ │ └──── Month (1–12)
│ │ └────── Day of month (1–31)
│ └──────── Hour (0–23)
└────────── Minute (0–59)
```

### Special Characters

| Character | Meaning |
|-----------|---------|
| `*` | Every (any value) |
| `,` | List of values |
| `-` | Range |
| `/` | Step/interval |

### Commands / Syntax

```bash
# Edit current user's crontab
crontab -e

# View current user's crontab
crontab -l

# Remove current user's crontab
crontab -r

# Edit another user's crontab (root only)
crontab -u username -e
```

### Practical Examples

```bash
# Run every minute
* * * * * /path/to/script.sh

# Run at 2:30 AM every day
30 2 * * * /path/to/backup.sh

# Run every Monday at 8 AM
0 8 * * 1 /path/to/report.sh

# Run at 12:00 PM on the 1st of every month
0 12 1 * * /path/to/billing.sh

# Run every 15 minutes
*/15 * * * * /path/to/check.sh

# Run at 6 AM and 6 PM daily
0 6,18 * * * /path/to/sync.sh

# Run Monday to Friday at 9 AM
0 9 * * 1-5 /path/to/workday.sh
```

### Cron Special Strings

```bash
@reboot     # At system startup
@daily      # Once a day (0 0 * * *)
@weekly     # Once a week (0 0 * * 0)
@monthly    # Once a month (0 0 1 * *)
@yearly     # Once a year (0 0 1 1 *)
@hourly     # Once an hour (0 * * * *)
```

### Practical Crontab Entry

```bash
# Edit crontab
crontab -e

# Add: run disk monitor every hour, log output
0 * * * * /home/user/disk_monitor.sh >> /var/log/disk_check.log 2>&1
```

### Common Mistakes / Notes

- Always use absolute paths in crontab entries — cron has a minimal environment and may not find relative paths.
- Redirect output to a log file using `>> /path/to/logfile 2>&1` to capture both stdout and stderr.
- Test scripts manually before scheduling to ensure they work correctly.
- The cron environment does not load `.bashrc`. Set required environment variables explicitly or source them in the script.

---

# 8. Logs & Debugging

---

## 8.1 Log Locations

### Definition

Linux stores system and application logs in `/var/log/`. Logs are essential for diagnosing system failures, security events, and application errors.

### Common Log Files

| File | Contents |
|------|---------|
| `/var/log/syslog` | General system messages (Debian/Ubuntu) |
| `/var/log/messages` | General system messages (RHEL/CentOS) |
| `/var/log/auth.log` | Authentication, login events (Debian/Ubuntu) |
| `/var/log/secure` | Authentication events (RHEL/CentOS) |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/dmesg` | Kernel ring buffer (boot messages) |
| `/var/log/cron` | Cron job logs |
| `/var/log/mail.log` | Mail server logs |
| `/var/log/nginx/` | Nginx access/error logs |
| `/var/log/apache2/` | Apache access/error logs |
| `/var/log/mysql/` | MySQL logs |

---

## 8.2 Viewing and Searching Logs

### Commands / Syntax

```bash
# View end of a log file
tail /var/log/syslog
tail -n 100 /var/log/syslog

# Follow log in real time
tail -f /var/log/syslog
tail -f /var/log/nginx/access.log

# Search logs
grep "error" /var/log/syslog
grep -i "failed" /var/log/auth.log
grep "192.168.1.100" /var/log/nginx/access.log

# View kernel messages
dmesg
dmesg | tail -50
dmesg | grep -i error

# journalctl (systemd logs)
journalctl                              # All logs
journalctl -n 100                       # Last 100 lines
journalctl -f                           # Follow
journalctl -u nginx                     # Service-specific logs
journalctl -p err                       # Only errors
journalctl --since "2024-01-01"
journalctl --since "1 hour ago"
journalctl -b                           # Logs since last boot
journalctl -b -1                        # Logs from previous boot
journalctl -k                           # Kernel messages only
```

### Practical Examples

```bash
# Find all SSH login failures
grep "Failed password" /var/log/auth.log

# Find all successful sudo uses
grep "sudo" /var/log/auth.log | grep "COMMAND"

# Count 404 errors in nginx logs
grep " 404 " /var/log/nginx/access.log | wc -l

# Real-time log monitoring with filtering
tail -f /var/log/syslog | grep -i "error\|warn\|fail"

# View logs for a crashed service
journalctl -u myservice --since "1 hour ago" -p err
```

### Common Mistakes / Notes

- Log rotation is managed by `logrotate`. Logs in `/var/log` are regularly compressed and rotated.
- On systemd systems, logs are stored in a binary journal. Use `journalctl` to read them properly.
- Always check logs first when a service fails: `systemctl status service` provides recent log entries.

---

# 9. Security Basics

---

## 9.1 SSH (Secure Shell)

### Definition

SSH is a cryptographic network protocol for secure remote access to servers. It encrypts all traffic between client and server.

### Commands / Syntax

```bash
# Connect to remote server
ssh user@hostname
ssh user@192.168.1.10
ssh -p 2222 user@hostname       # Custom port

# Execute a command remotely
ssh user@hostname "ls -la /var/log"

# Copy file to remote server (SCP)
scp localfile user@hostname:/remote/path/
scp -r localdir user@hostname:/remote/path/    # Directory

# Copy file from remote server
scp user@hostname:/remote/file /local/path/

# SSH with key file
ssh -i ~/.ssh/id_rsa user@hostname
```

---

## 9.2 Key-Based Authentication

### Definition

Key-based authentication uses a cryptographic key pair (public and private) instead of a password. It is more secure and enables passwordless logins.

### Setup Process

```bash
# Step 1: Generate key pair on client machine
ssh-keygen -t ed25519 -C "your_email@example.com"
# This creates:
#   ~/.ssh/id_ed25519       (private key - keep secret)
#   ~/.ssh/id_ed25519.pub   (public key - share this)

# Step 2: Copy public key to server
ssh-copy-id user@hostname

# Or manually:
cat ~/.ssh/id_ed25519.pub | ssh user@hostname "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Step 3: Set correct permissions on server
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Step 4: Connect (no password prompt)
ssh user@hostname
```

### SSH Server Configuration

```bash
# Edit SSH server config
vi /etc/ssh/sshd_config

# Recommended security settings:
# PasswordAuthentication no       # Disable password login
# PermitRootLogin no              # Disable root SSH access
# Port 2222                       # Change default port
# AllowUsers john alice           # Restrict to specific users

# Restart SSH after changes
systemctl restart sshd
```

### Common Mistakes / Notes

- Never share your private key (`~/.ssh/id_rsa` or `id_ed25519`).
- Set `PasswordAuthentication no` only after confirming key-based login works.
- Use `ed25519` algorithm for new keys — it is more secure and faster than RSA-2048.

---

## 9.3 Firewall (ufw / iptables)

### ufw (Uncomplicated Firewall) — Ubuntu/Debian

```bash
# Enable/disable firewall
ufw enable
ufw disable

# Check status
ufw status
ufw status verbose

# Allow/deny services
ufw allow ssh
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw deny 3306/tcp

# Allow from specific IP
ufw allow from 192.168.1.100
ufw allow from 192.168.1.0/24 to any port 22

# Delete a rule
ufw delete allow 80/tcp

# Reset all rules
ufw reset
```

### iptables (Low-level, all distributions)

```bash
# View current rules
iptables -L -n -v

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Block an IP
iptables -A INPUT -s 192.168.1.200 -j DROP

# Save rules (Debian/Ubuntu)
iptables-save > /etc/iptables/rules.v4

# Flush all rules
iptables -F
```

### Common Mistakes / Notes

- Always allow SSH before enabling a firewall to avoid locking yourself out.
- `ufw` is a front-end for `iptables` — both can coexist but only one should manage rules.
- Test firewall changes carefully on remote servers.

---

# 10. Process Isolation & Performance

---

## 10.1 cgroups (Control Groups)

### Definition

`cgroups` is a Linux kernel feature that limits, accounts for, and isolates the resource usage (CPU, memory, I/O, network) of a collection of processes.

### Why It Matters

cgroups are the foundation of containerization technologies like Docker and Kubernetes. They prevent a single process from consuming all system resources.

### Basic Usage

```bash
# View cgroup hierarchy (v2)
ls /sys/fs/cgroup/

# View resource usage of a service via systemd (uses cgroups internally)
systemd-cgtop

# View cgroups for a process
cat /proc/PID/cgroup

# Set CPU limit for a service via systemd
# In /etc/systemd/system/myservice.service add:
# [Service]
# CPUQuota=50%
# MemoryLimit=512M

systemctl daemon-reload
systemctl restart myservice
```

---

## 10.2 ulimit

### Definition

`ulimit` sets limits on the resources available to processes started by the shell.

### Commands / Syntax

```bash
# View all limits
ulimit -a

# Set maximum open files
ulimit -n 65536

# Set maximum processes
ulimit -u 1024

# Set core dump size (0 = disable)
ulimit -c 0

# Persistent limits: edit /etc/security/limits.conf
# username  soft  nofile  65536
# username  hard  nofile  65536
# *         soft  nproc   1024
```

### Practical Examples

```bash
# Check open file limit (common for web servers)
ulimit -n

# Increase temporarily
ulimit -n 100000

# Check current limit for a running process
cat /proc/PID/limits
```

---

## 10.3 CPU and Memory Monitoring

```bash
# CPU info
lscpu
cat /proc/cpuinfo
nproc                          # Number of logical CPUs

# Memory info
free -h
cat /proc/meminfo
vmstat -s

# Real-time CPU/memory
top
htop
iostat 2 5                     # CPU and I/O stats every 2 seconds

# Per-process memory
ps aux --sort=-%mem | head -10
cat /proc/PID/status | grep VmRSS

# I/O monitoring
iotop                          # Per-process I/O (may need install)
iostat -x 2                    # Detailed I/O statistics
```

---

# 11. Advanced Text Processing

---

## 11.1 grep — Advanced Usage

```bash
# Extended regex
grep -E "error|warn|fail" /var/log/syslog

# Only print matching part
grep -o "[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+" access.log    # Extract IPs

# Print N lines before/after match
grep -B 3 "error" logfile          # 3 lines before
grep -A 5 "error" logfile          # 5 lines after
grep -C 3 "error" logfile          # 3 lines before and after

# Count occurrences per file
grep -rc "error" /var/log/

# Search compressed files
zgrep "pattern" /var/log/syslog.gz

# Match whole word only
grep -w "fail" logfile
```

---

## 11.2 awk — Advanced Usage

`awk` is a powerful field-based text processor. It processes input line by line and splits each line into fields.

```bash
# Syntax
awk 'pattern { action }' file

# Print specific columns
awk '{print $1, $3}' file              # Columns 1 and 3
awk -F: '{print $1, $3}' /etc/passwd  # With colon delimiter

# Filter rows matching a condition
awk '$3 > 100' data.txt               # Rows where column 3 > 100
awk '/error/ {print $0}' logfile      # Rows containing "error"

# Calculations
awk '{sum += $1} END {print "Total:", sum}' numbers.txt
awk '{count++} END {print "Lines:", count}' file

# Print with formatting
awk '{printf "%-15s %s\n", $1, $2}' file

# Multiple conditions
awk -F: '$3 >= 1000 {print $1, $3}' /etc/passwd  # Non-system users

# Built-in variables
# NR  = current line number
# NF  = number of fields in current line
# FS  = field separator (default: whitespace)
# OFS = output field separator

awk 'NR==5' file                      # Print line 5
awk 'NR>=5 && NR<=10' file            # Print lines 5–10
awk '{print NF}' file                 # Number of fields per line
```

---

## 11.3 sed — Advanced Usage

`sed` is a stream editor for filtering and transforming text.

```bash
# Basic substitution
sed 's/old/new/' file                  # First occurrence per line
sed 's/old/new/g' file                 # All occurrences
sed 's/old/new/2' file                 # Second occurrence only
sed 's/old/new/gi' file                # Case-insensitive, all

# In-place editing
sed -i 's/old/new/g' file              # Edit file directly
sed -i.bak 's/old/new/g' file         # Edit with backup (.bak)

# Delete lines
sed '/pattern/d' file                  # Delete matching lines
sed '5d' file                          # Delete line 5
sed '5,10d' file                       # Delete lines 5–10

# Print specific lines
sed -n '5,10p' file                    # Print lines 5–10
sed -n '/start/,/end/p' file          # Print between patterns

# Insert and append
sed '3i\New line here' file            # Insert before line 3
sed '3a\New line here' file            # Append after line 3

# Multiple commands
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file

# Remove blank lines
sed '/^$/d' file

# Remove leading whitespace
sed 's/^[ \t]*//' file

# Add line numbers
sed '=' file | sed 'N; s/\n/\t/'
```

### Practical Pipeline Example

```bash
# Extract all IP addresses from an nginx log, sort, count unique occurrences
awk '{print $1}' /var/log/nginx/access.log \
    | sort \
    | uniq -c \
    | sort -rn \
    | head -20
```

---

# 12. Remote Systems & Server Handling

---

## 12.1 SSH Advanced Usage

```bash
# SSH config file: ~/.ssh/config
# Simplifies repeated SSH connections

# Example ~/.ssh/config
Host myserver
    HostName 192.168.1.10
    User john
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

Host prod-web
    HostName 10.0.0.5
    User deploy
    IdentityFile ~/.ssh/deploy_key

# Connect using alias
ssh myserver
ssh prod-web

# SSH tunneling (port forwarding)
# Local port 8080 -> remote 80
ssh -L 8080:localhost:80 user@remote_server

# Remote port forwarding
ssh -R 9090:localhost:3000 user@remote_server

# SOCKS proxy
ssh -D 1080 user@remote_server

# SSH multiplexing (reuse connections)
# Add to ~/.ssh/config:
# Host *
#     ControlMaster auto
#     ControlPath ~/.ssh/cm_socket/%r@%h:%p
#     ControlPersist 10m
```

---

## 12.2 File Transfer

```bash
# SCP: secure file copy
scp file.txt user@host:/path/
scp user@host:/path/file.txt ./
scp -r directory/ user@host:/path/
scp -P 2222 file.txt user@host:/path/    # Custom port

# rsync: efficient file synchronization
rsync -avz source/ user@host:/destination/
rsync -avz --delete source/ user@host:/dest/   # Mirror (delete extra files)
rsync -avz --progress source/ dest/             # Show progress
rsync -avz -e "ssh -p 2222" source/ user@host:/dest/   # Custom port

# rsync options:
# -a  archive (recursive, permissions, timestamps, symlinks)
# -v  verbose
# -z  compress
# -n  dry run (preview only)
# --exclude='*.log'  exclude pattern
```

---

## 12.3 Managing Remote Servers

```bash
# Run multiple commands over SSH
ssh user@host "cd /var/www; git pull; systemctl restart nginx"

# Execute a local script on a remote server
ssh user@host 'bash -s' < local_script.sh

# SSH with pseudo-terminal (for interactive commands)
ssh -t user@host "sudo htop"

# Keep session alive (add to ~/.ssh/config)
# ServerAliveInterval 60
# ServerAliveCountMax 3
```

---

## 12.4 Debugging Production Environments

A structured approach to diagnosing issues on production servers.

### Step 1 — Check Service Status

```bash
systemctl status servicename
journalctl -u servicename -n 50 --no-pager
```

### Step 2 — Check Logs

```bash
tail -100 /var/log/nginx/error.log
journalctl -p err --since "1 hour ago"
grep -i "error\|fatal\|critical" /var/log/syslog | tail -50
```

### Step 3 — Check Resources

```bash
top                          # CPU and memory
free -h                      # Memory
df -h                        # Disk space
df -i                        # Inode usage
iostat -x 1 5                # Disk I/O
```

### Step 4 — Check Network

```bash
ss -tuln                     # Listening ports
ss -anp | grep :80           # Connections on port 80
ping -c 4 internal_hostname  # Connectivity
curl -I http://localhost      # HTTP response
```

### Step 5 — Check Processes

```bash
ps aux | grep servicename
pgrep -a nginx
lsof -i :80                  # Processes using port 80
lsof -p PID                  # All files opened by a process
```

### Step 6 — Check System Logs

```bash
dmesg | tail -50             # Kernel messages
journalctl -b -p err         # Errors since last boot
last                         # Recent logins
who                          # Currently logged-in users
```

### Common Production Debugging Commands

```bash
# Check what's listening on a port
ss -tlnp | grep :443

# Find which process is using a file
lsof /path/to/file

# Trace system calls of a running process
strace -p PID

# Check open connections count
ss -s

# Verify DNS resolution from server
dig +short api.example.com
nslookup api.example.com

# Test application response time
curl -o /dev/null -s -w "Time: %{time_total}s\n" https://example.com
```

---

# Quick Reference: Essential Commands

| Category | Command | Description |
|----------|---------|-------------|
| Navigation | `pwd` | Current directory |
| | `cd /path` | Change directory |
| | `ls -la` | List all files |
| Files | `cp -r src dst` | Copy directory |
| | `mv src dst` | Move/rename |
| | `rm -rf dir` | Remove directory |
| | `find / -name file` | Find file |
| Content | `cat file` | View file |
| | `less file` | Paginated view |
| | `tail -f file` | Follow file |
| | `grep -r pattern .` | Search recursively |
| Processes | `ps aux` | All processes |
| | `kill -9 PID` | Force kill |
| | `systemctl status svc` | Service status |
| Disk | `df -h` | Disk usage |
| | `du -sh dir` | Directory size |
| Network | `ss -tuln` | Open ports |
| | `ping host` | Test connectivity |
| | `curl -I url` | HTTP headers |
| Users | `id` | Current user info |
| | `whoami` | Current username |
| | `sudo command` | Run as root |
| Logs | `journalctl -f` | Follow system log |
| | `dmesg \| tail` | Kernel messages |
| Packages | `apt update && apt upgrade` | Update system |
| | `apt install pkg` | Install package |
| SSH | `ssh user@host` | Connect remotely |
| | `scp file user@host:` | Copy to remote |

---

*End of Linux Fundamentals Guide*
