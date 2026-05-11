# Foundations — Complete Mastery Guide

---

# PART 1: LINUX FUNDAMENTALS

---

## 1. Overview

Linux is not just an operating system — it is the backbone of virtually every production system on the planet. Servers, containers, cloud VMs, Kubernetes nodes, CI runners — all Linux. As a DevOps engineer, Linux is your primary operating environment. You don't interact with it through a GUI. You command it through a terminal, and it responds precisely to what you ask — no more, no less.

Understanding Linux at a deep level means understanding how processes are born and die, how the kernel manages memory and I/O, how files are structured, how networking is handled, and how permissions protect the system. Every tool in the DevOps stack — Docker, Kubernetes, Nginx, systemd — is built on top of Linux primitives.

---

## 2. Core Concepts

### The Kernel and User Space

Linux has two distinct worlds:

- **Kernel space**: Where the kernel runs with full hardware access. Manages CPU scheduling, memory, devices, file systems, networking.
- **User space**: Where all applications and your shell run. They cannot access hardware directly — they go through the kernel via **system calls** (syscalls).

When you run `ls`, bash calls the kernel's `getdents64` syscall to read directory entries. When a process reads a file, it calls `read()`. Everything is a syscall underneath.

```
User Space
  └── Shell / Applications / Tools
          │
          ▼  (system calls: read, write, fork, exec, mmap...)
Kernel Space
  └── Process Scheduler, Memory Manager, VFS, Network Stack, Device Drivers
          │
          ▼
Hardware (CPU, RAM, Disk, NIC)
```

### The Filesystem Hierarchy

Linux follows the **Filesystem Hierarchy Standard (FHS)**. Every path means something:

```
/           Root — the beginning of everything
/bin        Essential user binaries (ls, cp, mv) — needed before /usr mounts
/sbin       System binaries for root (iptables, fdisk)
/etc        Configuration files for the system and services
/var        Variable data — logs (/var/log), spools, runtime state
/tmp        Temporary files, cleared on reboot
/proc       Virtual filesystem — kernel exposes process and system info here
/sys        Virtual filesystem — kernel exposes hardware/driver info here
/dev        Device files (block and character devices)
/home       User home directories
/root       Root user's home
/usr        User programs, libraries, docs (most installed software lands here)
/usr/bin    Non-essential user binaries (most commands you type)
/usr/lib    Libraries for /usr/bin programs
/opt        Optional third-party software packages
/mnt        Temporary mount point for filesystems
/media      Auto-mounted removable media (USB, CD)
/boot       Kernel images, initramfs, bootloader config
/run        Runtime data since last boot (PIDs, sockets) — tmpfs
```

**Why this matters in production**: When a disk fills up on `/var` due to log overflow, your application still works but can't write logs. When `/tmp` fills up, applications that write temp files crash. Knowing what lives where tells you exactly where to look.

### Everything is a File

In Linux, almost everything is represented as a file:
- Regular files: your code, configs
- Directories: files that list other files
- Devices: `/dev/sda`, `/dev/null`, `/dev/random`
- Sockets: Unix domain sockets for IPC
- Pipes: FIFOs
- Symlinks: pointers to other files
- `/proc` entries: live kernel data as files

This is powerful because tools like `cat`, `echo`, `read` work on almost everything. You can read CPU info with `cat /proc/cpuinfo`. You can discard output by redirecting to `/dev/null`.

### Process Model

Every running program is a **process**. Every process has:
- A **PID** (Process ID)
- A **PPID** (Parent Process ID)
- An **owner UID/GID**
- An **environment** (key=value pairs)
- **File descriptors** (open files, sockets, pipes)
- A **working directory**
- **Memory mappings** (stack, heap, code, shared libs)

Processes are created via the `fork()` syscall — a process clones itself. Then `exec()` replaces the cloned process's memory with a new program. This fork-exec model is fundamental to how shells work.

```
PID 1 (init/systemd)
  └── PID 500 (sshd)
        └── PID 1200 (bash)
              └── PID 1350 (python app.py)
```

If a parent dies, its children become orphans and are re-parented to PID 1. If a child dies but parent hasn't called `wait()`, it becomes a **zombie** (Z state).

### File Descriptors

Every process gets three standard file descriptors:
- **0**: stdin (standard input)
- **1**: stdout (standard output)
- **2**: stderr (standard error)

When you open a file, you get FD 3, then 4, etc. Redirection in bash manipulates these:

```bash
command > file        # stdout (FD 1) → file
command 2> file       # stderr (FD 2) → file
command 2>&1          # stderr → wherever stdout goes
command &> file       # both stdout and stderr → file
command < input       # stdin (FD 0) ← input file
```

### Permissions Model

Linux uses **Discretionary Access Control (DAC)**:

```
-rwxr-xr--  1  owner  group  size  date  filename
│├──┤├──┤├─┤
│ │   │   └── others (everyone else): r-- = read only
│ │   └────── group: r-x = read and execute
│ └────────── owner: rwx = read, write, execute
└──────────── file type: - = regular, d = dir, l = symlink, c = char dev, b = block dev
```

Permission bits in octal:
- read = 4
- write = 2
- execute = 1

So `755` = owner: rwx (7), group: r-x (5), others: r-x (5)

**Special permission bits**:
- **setuid (s)**: When set on executable, runs as the file owner, not the caller. `passwd` uses this to write `/etc/shadow` as root.
- **setgid (s)**: Runs as the group, or on directories — files created inside inherit the group.
- **sticky bit (t)**: On directories, only the file owner can delete their files. `/tmp` has this.

**ACLs** (Access Control Lists) extend beyond owner/group/others. You can grant specific users access:

```bash
setfacl -m u:deploy:rx /var/app
getfacl /var/app
```

---

## 3. Internal Working

### The Boot Process

```
BIOS/UEFI
  └── POST (Power-On Self Test)
        └── Locates bootable disk
              └── Loads bootloader (GRUB2) from MBR/EFI partition
                    └── GRUB loads kernel image + initramfs into RAM
                          └── Kernel initializes:
                                - Decompresses itself
                                - Initializes memory (paging, NUMA)
                                - Initializes CPU, drivers
                                - Mounts initramfs as temporary root
                                - Runs /init (or systemd) from initramfs
                                └── Pivots to real root filesystem
                                      └── Starts PID 1 (systemd)
                                            └── systemd starts all services
```

**initramfs** is a temporary RAM-based root filesystem that contains enough drivers and tools to mount the real root filesystem.

### systemd

Modern Linux uses **systemd** as PID 1. It manages:
- Service lifecycle (start, stop, restart, reload)
- Boot ordering and dependencies
- Socket activation
- Timers (cron replacement)
- Logging (journald)
- cgroups assignment

Units are the fundamental building blocks:

```
service    — a daemon or oneshot process
socket     — a socket that activates a service
timer      — a scheduled trigger
target     — a group of units (like runlevels)
mount      — a filesystem mount point
device     — a kernel device
```

### The Scheduler

The Linux kernel scheduler decides which process runs on which CPU core and for how long. It uses the **Completely Fair Scheduler (CFS)**. Processes are ranked by their **virtual runtime** — the one that has run the least gets priority. This ensures fairness.

**Nice values**: Range from -20 (highest priority) to +19 (lowest). Only root can set negative nice.

**cgroups (Control Groups)**: Allow you to limit and account for resource usage (CPU, memory, I/O, network) per group of processes. This is the foundation of containers.

### The Virtual Memory System

Every process sees its own virtual address space. The kernel maps virtual addresses to physical RAM pages. This means:
- Processes are isolated (can't touch each other's memory)
- Processes can use more memory than physically exists (swap)
- Memory-mapped files share physical pages between processes

**Page faults**: When a process accesses a virtual address with no physical mapping, the CPU triggers a page fault. The kernel then either allocates a page (minor fault) or reads it from disk (major fault — expensive).

### The VFS (Virtual Filesystem Switch)

Linux abstracts all filesystems through VFS. Whether you're reading ext4, XFS, tmpfs, NFS, or procfs — your application calls `read()` the same way. VFS dispatches to the appropriate filesystem driver.

Common production filesystems:
- **ext4**: Default, battle-tested, good for general use
- **XFS**: Better for large files, parallel I/O, used heavily in RHEL
- **Btrfs**: Copy-on-write, snapshots, checksums
- **tmpfs**: RAM-based, ephemeral
- **overlayfs**: Union mount — the foundation of Docker layers

---

## 4. Real-World Usage

### User and Group Management

```bash
# Create user with home dir and specific shell
useradd -m -s /bin/bash -G sudo,docker appuser

# Set password
passwd appuser

# Lock/unlock user
usermod -L appuser   # lock
usermod -U appuser   # unlock

# Switch user
su - appuser         # full login shell
sudo -u appuser cmd  # run as appuser

# View current user info
id
whoami

# View all users
cat /etc/passwd   # username:x:UID:GID:comment:home:shell
cat /etc/shadow   # hashed passwords (root only)
cat /etc/group    # groups and members
```

### Process Management

```bash
# View processes
ps aux                    # all processes, BSD format
ps -ef                    # all processes, full format
ps -eo pid,ppid,user,cmd  # custom columns

# Process tree
pstree -p

# Real-time monitoring
top
htop                      # better UI
atop                      # disk and network too

# Find process by name
pgrep nginx
pidof nginx

# Send signals
kill -SIGTERM 1234    # graceful shutdown
kill -SIGKILL 1234    # force kill (can't be caught)
kill -SIGHUP 1234     # reload config (for many daemons)
killall nginx
pkill -f "python app"

# Job control
command &             # run in background
jobs                  # list background jobs
fg %1                 # bring job 1 to foreground
bg %1                 # resume job 1 in background
nohup command &       # run immune to HUP signal
```

### File Operations

```bash
# Find files
find /var/log -name "*.log" -mtime -7           # modified in last 7 days
find / -perm -4000 -type f 2>/dev/null          # setuid files
find /etc -size +1M                              # files over 1MB
find . -type f -exec chmod 644 {} \;            # chmod all files

# File content
cat file                    # print file
less file                   # paginate
head -n 50 file             # first 50 lines
tail -n 100 file            # last 100 lines
tail -f /var/log/app.log    # follow log in real time
tail -f -n 0 file           # only new content

# Text processing
grep "ERROR" app.log
grep -r "pattern" /etc/      # recursive
grep -v "DEBUG"              # invert match
grep -E "ERR|WARN"           # extended regex

# Sort, unique, count
sort file
sort -k2 -n file             # sort by 2nd column numerically
uniq -c file                 # count duplicates
sort file | uniq -c | sort -rn   # frequency analysis

# Column manipulation
cut -d: -f1 /etc/passwd      # first field, colon delimiter
awk '{print $1, $3}' file    # print columns 1 and 3
awk -F: '{print $1}' /etc/passwd

# Sed — stream editor
sed 's/foo/bar/g' file               # replace
sed -i 's/foo/bar/g' file            # in-place replace
sed '/^#/d' file                     # delete comment lines
sed -n '10,20p' file                 # print lines 10-20

# wc — word count
wc -l file     # line count
wc -c file     # byte count
```

### Disk and Storage

```bash
# Disk usage
df -h                       # filesystem usage
df -ih                      # inode usage (important!)
du -sh /var/log             # directory size
du -sh /* | sort -rh        # largest top-level dirs

# Disk I/O
iostat -x 1                 # extended I/O stats every 1s
iotop                       # per-process I/O
lsblk                       # block devices tree
blkid                       # UUIDs and filesystem types
fdisk -l                    # partition table

# Mount/unmount
mount /dev/sdb1 /mnt/data
umount /mnt/data
# Persistent mounts in /etc/fstab
# UUID=xxx  /data  ext4  defaults  0 2
```

### Networking from Linux Perspective

```bash
# Interface info
ip addr show
ip link show
ip route show

# Network connections
ss -tulnp         # listening TCP/UDP sockets with process
ss -s             # summary stats
netstat -tulnp    # older equivalent

# Test connectivity
ping -c 4 8.8.8.8
traceroute 8.8.8.8
mtr 8.8.8.8           # continuous traceroute

# DNS
dig google.com
dig @8.8.8.8 google.com
nslookup google.com

# HTTP testing
curl -v https://example.com
curl -I https://example.com       # headers only
wget https://example.com/file.tar.gz

# Firewall
iptables -L -n -v          # list all rules
iptables -L INPUT -n       # INPUT chain
ufw status verbose         # Ubuntu UFW wrapper
```

### systemd Service Management

```bash
# Service control
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx           # reload config without restart
systemctl status nginx
systemctl enable nginx           # start on boot
systemctl disable nginx
systemctl is-active nginx
systemctl is-enabled nginx

# View logs for a service
journalctl -u nginx
journalctl -u nginx -f          # follow
journalctl -u nginx --since "1 hour ago"
journalctl -u nginx -n 100      # last 100 lines

# List all units
systemctl list-units
systemctl list-unit-files

# System state
systemctl get-default           # default target
systemctl isolate multi-user.target

# Write a custom service unit
cat /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Application
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/server
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal
Environment=PORT=8080
EnvironmentFile=/etc/myapp/env

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload    # after editing unit files
```

---

## 5. Architecture & Production Perspective

### Resource Limits

Linux allows you to set limits on resource usage per user or process:

```bash
# View current limits
ulimit -a

# Set open file limit (crucial for high-connection services)
ulimit -n 65536

# Permanent limits via /etc/security/limits.conf
# appuser soft nofile 65536
# appuser hard nofile 65536
```

In production, services like Nginx, Redis, or database connections can hit the default open file limit (1024). Always tune this.

### /proc — The Window Into the Kernel

```bash
cat /proc/cpuinfo          # CPU details
cat /proc/meminfo          # Memory details
cat /proc/loadavg          # Load average (1m, 5m, 15m) + running/total procs
cat /proc/net/tcp          # TCP connections in hex
cat /proc/sys/net/core/somaxconn   # max socket backlog

# Per-process info
ls /proc/1234/             # PID directory
cat /proc/1234/status      # process state
cat /proc/1234/maps        # memory mappings
ls -la /proc/1234/fd/      # open file descriptors
cat /proc/1234/environ     # environment variables
```

### sysctl — Kernel Parameters

```bash
# View all kernel parameters
sysctl -a

# View specific
sysctl net.core.somaxconn
sysctl vm.swappiness

# Set temporarily
sysctl -w net.core.somaxconn=65535

# Permanently (survives reboot)
echo "net.core.somaxconn=65535" >> /etc/sysctl.conf
sysctl -p   # reload
```

Critical production sysctl settings:

```
net.core.somaxconn = 65535          # socket backlog queue
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535   # ephemeral port range
vm.swappiness = 10                  # prefer RAM over swap (10 for servers, 1 for DBs)
vm.overcommit_memory = 1            # allow overcommit (needed by Redis, Go)
net.ipv4.tcp_tw_reuse = 1          # reuse TIME_WAIT sockets
```

---

## 6. Commands / Configs / Code Examples

### Useful One-Liners Engineers Use Daily

```bash
# Who is logged in?
who
w
last -n 20     # last 20 logins

# What's using all the disk?
du -sh /var/log/* | sort -rh | head -20

# What process is using port 8080?
ss -tlnp sport = :8080
lsof -i :8080

# Watch a command output every 2 seconds
watch -n 2 "ss -s"

# Count lines matching pattern in logs
grep -c "ERROR" /var/log/app.log

# Follow multiple log files
tail -f /var/log/nginx/access.log /var/log/nginx/error.log

# Find largest files in /var
find /var -type f -printf '%s %p\n' | sort -rn | head -20

# Check if service is listening
nc -zv localhost 6379       # test Redis port

# CPU and memory of a process
ps -p 1234 -o %cpu,%mem,cmd

# Recursive grep with line numbers
grep -rn "DB_PASSWORD" /etc/

# Archive and compress a directory
tar czf backup.tar.gz /opt/myapp/

# Extract
tar xzf backup.tar.gz -C /opt/

# Sync directories
rsync -avz --delete /src/ user@host:/dst/

# Secure copy
scp -r /local/path user@host:/remote/path

# Create a large test file (for disk tests)
dd if=/dev/zero of=testfile bs=1M count=1024

# Measure disk write speed
dd if=/dev/zero of=/tmp/testfile bs=1G count=1 oflag=direct

# strace a process (syscall tracing)
strace -p 1234 -f -e trace=network

# lsof - list open files
lsof -p 1234             # files opened by PID
lsof -u appuser          # files opened by user
lsof /var/log/app.log    # who has this file open
```

---

## 7. Debugging & Troubleshooting

### High CPU

```bash
top              # which PID is top?
ps aux --sort=-%cpu | head
perf top         # which kernel function?
pidstat 1        # per-process CPU stats

# Check if it's kernel or user
top → look at 'us' vs 'sy' vs 'wa' vs 'si'
# us = user space CPU
# sy = kernel/system CPU
# wa = I/O wait (high = disk bottleneck)
# si = software interrupts (high = network overload)
```

### High Memory

```bash
free -h
cat /proc/meminfo

# Find memory hogs
ps aux --sort=-%mem | head -10

# OOM Killer logs
dmesg | grep -i "oom"
journalctl -k | grep oom
```

### Disk Full

```bash
df -h       # which filesystem?
du -sh /*   # what's using it?
lsof | grep deleted   # deleted but still-open files (common cause!)
```

The classic production gotcha: a log file is deleted, but the process still has it open. The disk space isn't freed until the file descriptor is closed. Fix:

```bash
# Option 1: restart the process
# Option 2: truncate the file via its FD
lsof | grep deleted
# Get the PID and FD number
> /proc/<PID>/fd/<FD>   # truncate it to zero bytes
```

### Process Won't Start

```bash
# Check logs
journalctl -u myapp -n 50

# Check return code
systemctl status myapp -l    # -l for full output

# Try running manually as the service user
sudo -u appuser /opt/myapp/bin/server

# Check port already in use
ss -tlnp | grep 8080

# Check file permissions
ls -la /opt/myapp/bin/server

# Check SELinux/AppArmor
dmesg | grep denied
aa-status    # AppArmor
getenforce   # SELinux
```

---

## 8. Best Practices

- **Never run applications as root.** Create dedicated service users.
- **Use sudo with specific commands**, not blanket root access. Configure via `/etc/sudoers`.
- **Set resource limits** for all production services (file descriptors, memory, CPU).
- **Log everything to journald or a centralized log system**, never to random files.
- **Use systemd** for service management — don't write custom init scripts.
- **Enable and configure a firewall** even within a VPC — defense in depth.
- **Audit setuid/setgid binaries** regularly: `find / -perm -4000 -o -perm -2000`.
- **Use tmpfs for /tmp** on servers where temp data should never hit disk.
- **Set swappiness appropriately** — 10 for most servers, 1 for Redis/databases.
- **Rotate logs** — configure logrotate for any log file a service writes.
- **Kernel updates** — apply security patches, but test first. Use live patching (ksplice/kpatch) for critical systems.

---

## 9. Common Mistakes

- **Not understanding file descriptor limits.** A Node.js or Go service with many connections hits 1024 FDs and starts failing mysteriously.
- **Ignoring inode exhaustion.** `df -h` shows space free, but `df -ih` shows 0 inodes. Many small files (like mail queues or temp files) can exhaust inodes.
- **Wildcard expansion in production.** `rm -rf /opt/myapp/` followed by a space and then something else is catastrophic.
- **Not checking return codes.** Shell scripts that continue even after a critical command fails.
- **Misunderstanding symlinks.** A symlink's permissions are irrelevant — the target's permissions apply.
- **Forgetting timezone configuration.** Server logs in UTC but application logs in local time makes correlation hell. Use UTC everywhere.
- **Shared mutable `/tmp`.** Race conditions and security issues from multiple services using predictable temp file names.

---

## 10. Senior Engineer Mental Models

**"Everything is a file descriptor"** — When something behaves strangely with I/O, always check what file descriptors are open, what limits are in place, and what's on the other end.

**"Signals are the process communication protocol"** — SIGTERM for graceful, SIGKILL as last resort, SIGHUP for reload, SIGUSR1/SIGUSR2 for application-defined actions.

**"The filesystem is your API to the kernel"** — `/proc`, `/sys`, and `/dev` are not magic. They're the kernel's published interface.

**"Permissions are about trust boundaries"** — Every setuid binary, every sudo rule, every world-writable directory is a potential privilege escalation. Treat them as attack surface.

**"When debugging, start with the simplest explanation"** — Is the disk full? Is the port in use? Is the process running as the wrong user? Before writing scripts, check the basics.

---

## 11. Interview-Level Knowledge

- Explain the difference between a process, thread, and goroutine at the kernel level.
- What is a zombie process and how do you clean it up without killing the parent?
- Explain what happens between pressing Enter and seeing output from a command.
- How does `sudo` work internally (setuid + PAM + sudoers)?
- What is the difference between soft and hard links?
- Why can't you delete a file that's currently open by a process?
- Explain the `/proc/sys/vm/overcommit_memory` options and when you'd set each.
- What is copy-on-write memory and how does it relate to `fork()`?
- Explain inotify and how tools like `tail -f` work without polling.

---

## 12. Advanced Insights

- **cgroups v2** is the unified hierarchy replacing v1. Kubernetes uses it for container resource enforcement. Understanding cgroups means understanding how containers are isolated.
- **eBPF** (Extended Berkeley Packet Filter) is revolutionizing Linux observability. Tools like `bpftrace`, `cilium`, and `Pixie` use eBPF to trace kernel activity with zero overhead and no code changes. This is the future of AIOps and observability.
- **Linux namespaces** (pid, net, mnt, uts, ipc, user, cgroup) are the foundation of containers. Docker/Kubernetes just use unshare() and clone() with namespace flags.
- **NUMA (Non-Uniform Memory Access)** matters on multi-socket servers. Memory access time depends on which CPU socket is accessing which DIMM. `numactl` pins processes to NUMA nodes.
- **Huge pages** reduce TLB pressure for large memory applications (databases, JVMs). Transparent Huge Pages (THP) can cause latency spikes — often disabled for Redis/MySQL in production.

---

## 13. Summary

Linux is the substrate everything runs on. As a DevOps engineer, you need to be fluent in:
- Process model, signals, job control
- Filesystem hierarchy and operations
- Permissions, users, and groups
- systemd for service management
- The /proc and /sys interfaces
- Networking tools (ss, ip, iptables)
- Text processing (grep, awk, sed, cut)
- Kernel tuning via sysctl
- Debugging methodology (strace, lsof, journalctl)

Linux mastery is not memorizing commands. It's understanding the model: everything is a file, every program is a process, every process is owned by a user, and the kernel mediates everything via syscalls.

---

---

# PART 2: BASH SCRIPTING

---

## 1. Overview

Bash (Bourne Again Shell) is the glue of the DevOps world. CI/CD pipelines, Docker entrypoints, Kubernetes init containers, deployment scripts, health checks, automation — all bash. Even if you write Python for complex tooling, you still need bash for bootstrapping, one-liners, and system-level glue.

Bash is not a "simple scripting language" — it has significant gotchas that can silently break production deployments, introduce security vulnerabilities, or cause data loss. Senior engineers write bash that is robust, readable, and defensively coded.

---

## 2. Core Concepts

### The Shell's Job

The shell's job is to:
1. Parse your command line
2. Expand tokens (variables, globs, brace expansion, command substitution)
3. Handle redirections
4. Fork a process and exec the command
5. Wait for the result
6. Return the exit code

Understanding the expansion order prevents bugs:

```
Brace expansion:      {a,b}c → ac bc
Tilde expansion:      ~ → $HOME
Variable expansion:   $VAR or ${VAR}
Command substitution: $(command) or `command`
Arithmetic:           $((expr))
Word splitting:       split on IFS after expansion
Pathname expansion:   glob: *, ?, [...]
```

### Exit Codes

Every command returns an exit code: 0 = success, non-zero = failure.

```bash
ls /exists    # exit code 0
ls /nope      # exit code 2
echo $?       # print last exit code
```

This is critical. A script without exit code checking is a script that silently fails.

### Variables

```bash
# Assignment — no spaces around =
NAME="production"
COUNT=42

# Expansion
echo $NAME
echo ${NAME}          # preferred — unambiguous
echo "${NAME}_server" # concatenation

# Read-only
readonly API_KEY="secret123"

# Unset
unset NAME

# Default values
echo ${NAME:-"default"}          # use default if unset/empty
echo ${NAME:="default"}          # assign default if unset/empty
echo ${NAME:?"error: NAME unset"} # error if unset/empty
echo ${NAME:+"alternate"}        # use alternate if set

# String operations
STR="hello world"
echo ${#STR}              # length: 11
echo ${STR:0:5}           # substring: hello
echo ${STR/hello/goodbye} # replace first
echo ${STR//l/L}          # replace all
echo ${STR^^}             # uppercase
echo ${STR,,}             # lowercase

# Path manipulation
FILE="/var/log/app.log"
echo ${FILE%.*}     # strip shortest suffix match: /var/log/app
echo ${FILE%%.*}    # strip longest suffix match: /var/log/app
echo ${FILE##*/}    # strip longest prefix match: app.log
echo ${FILE%/*}     # dirname: /var/log
```

### Arrays

```bash
# Indexed array
SERVERS=("web1" "web2" "db1")
echo ${SERVERS[0]}        # web1
echo ${SERVERS[@]}        # all elements
echo ${#SERVERS[@]}       # count
SERVERS+=("cache1")       # append

# Iterate
for s in "${SERVERS[@]}"; do
    echo "Server: $s"
done

# Associative array (bash 4+)
declare -A CONFIG
CONFIG[host]="localhost"
CONFIG[port]="5432"
echo ${CONFIG[host]}
echo ${!CONFIG[@]}    # all keys
echo ${CONFIG[@]}     # all values
```

---

## 3. Internal Working

When you type a command, bash:
1. Reads the line
2. Tokenizes it
3. Performs all expansions in order
4. Forks itself (fork())
5. The child process exec()s the command
6. The parent waits
7. Exit code is captured in `$?`

For built-in commands (cd, echo, export, source), no fork is needed — bash handles them directly.

**Source vs Execute**: When you run `./script.sh`, it forks a new bash process. When you `source script.sh` or `. script.sh`, it runs in the current shell's context. This is why `source` is used to load environment variables — a child process can't modify its parent's environment.

---

## 4. Real-World Usage

### Script Structure

Every production bash script should start like this:

```bash
#!/usr/bin/env bash
# ^^^ Use env to find bash — more portable than /bin/bash

# Script metadata
# Purpose: Deploy application to target environment
# Usage: ./deploy.sh [environment] [version]
# Dependencies: kubectl, helm, aws-cli

set -euo pipefail
# -e: exit on any error (non-zero exit code)
# -u: exit on undefined variable usage
# -o pipefail: exit if any command in a pipe fails
# Together: the holy trinity of safe bash scripting

IFS=$'\n\t'
# Set Internal Field Separator to newline and tab only
# Prevents word splitting on spaces in filenames

# Optional: trace mode (uncomment for debugging)
# set -x   # print each command before executing
```

### Functions

```bash
# Define functions
log_info() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [INFO] $*"
}

log_error() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [ERROR] $*" >&2
}

die() {
    log_error "$*"
    exit 1
}

# Usage
log_info "Starting deployment"
die "Failed to connect to database"

# Functions with return values
get_pod_count() {
    local namespace="$1"
    kubectl get pods -n "$namespace" --no-headers | wc -l
}

count=$(get_pod_count "production")
```

### Conditionals

```bash
# File tests
if [[ -f /etc/config.yaml ]]; then
    echo "File exists"
fi

if [[ -d /opt/app ]]; then
    echo "Directory exists"
fi

if [[ -r /etc/secret ]]; then  # readable
    ...
fi

if [[ -x /usr/bin/kubectl ]]; then  # executable
    ...
fi

# String tests
if [[ -z "$VAR" ]]; then    # empty
if [[ -n "$VAR" ]]; then    # not empty
if [[ "$VAR" == "prod" ]]; then
if [[ "$VAR" != "dev" ]]; then
if [[ "$VAR" =~ ^[0-9]+$ ]]; then    # regex match

# Numeric tests
if [[ $COUNT -eq 0 ]]; then
if [[ $COUNT -gt 10 ]]; then
if [[ $COUNT -le 100 ]]; then

# Logical operators
if [[ -n "$A" && -n "$B" ]]; then    # AND
if [[ -z "$A" || -z "$B" ]]; then   # OR
if [[ ! -f /tmp/lock ]]; then        # NOT

# [[ vs [ vs test
# Use [[ ]] always in bash — it's safer, supports regex, no word splitting issues
# [ ] is POSIX sh compatible but more error-prone
# test is equivalent to [ ]
```

### Loops

```bash
# For loop
for i in 1 2 3 4 5; do
    echo "Item: $i"
done

# C-style for
for ((i=0; i<10; i++)); do
    echo "$i"
done

# Iterate over array
for server in "${SERVERS[@]}"; do
    ssh "$server" "uptime"
done

# While loop
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/hosts

# Process each file
find /var/log -name "*.log" | while IFS= read -r file; do
    echo "Processing: $file"
done

# Until loop
until curl -sf http://localhost:8080/health; do
    echo "Waiting for service..."
    sleep 5
done

# Break and continue
for i in {1..10}; do
    [[ $i -eq 5 ]] && continue   # skip 5
    [[ $i -eq 8 ]] && break      # stop at 8
    echo "$i"
done
```

### Input Handling

```bash
#!/usr/bin/env bash
set -euo pipefail

# Positional parameters
SCRIPT_NAME="$0"     # script name
ARG1="${1:-}"        # first arg, empty if not provided
ARG2="${2:-}"

# Require arguments
if [[ $# -lt 2 ]]; then
    echo "Usage: $0 <environment> <version>"
    exit 1
fi

ENV="$1"
VERSION="$2"

# Validate input
case "$ENV" in
    production|staging|dev)
        ;;
    *)
        die "Invalid environment: $ENV. Must be production, staging, or dev."
        ;;
esac

# Named args with getopts
usage() {
    echo "Usage: $0 [-e environment] [-v version] [-d]"
    exit 1
}

DEBUG=false

while getopts "e:v:dh" opt; do
    case $opt in
        e) ENV="$OPTARG" ;;
        v) VERSION="$OPTARG" ;;
        d) DEBUG=true ;;
        h) usage ;;
        *) usage ;;
    esac
done
```

### Error Handling

```bash
#!/usr/bin/env bash
set -euo pipefail

# Trap for cleanup on exit
TMPDIR=$(mktemp -d)
trap 'rm -rf "$TMPDIR"' EXIT

# Trap errors with context
trap 'echo "Error on line $LINENO: $BASH_COMMAND" >&2' ERR

# Handle specific exit codes
if ! kubectl apply -f deployment.yaml; then
    log_error "kubectl apply failed"
    # Rollback
    kubectl rollout undo deployment/myapp
    exit 1
fi

# Retry logic
retry() {
    local max_attempts="$1"
    local delay="$2"
    shift 2
    local attempt=1

    until "$@"; do
        if (( attempt >= max_attempts )); then
            log_error "Command failed after $max_attempts attempts: $*"
            return 1
        fi
        log_info "Attempt $attempt failed. Retrying in ${delay}s..."
        sleep "$delay"
        ((attempt++))
    done
}

retry 5 10 curl -sf http://myservice/health
```

---

## 5. Architecture & Production Perspective

### Deployment Script Pattern

```bash
#!/usr/bin/env bash
set -euo pipefail

# ============================================================
# Production Deployment Script
# ============================================================

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly TIMESTAMP=$(date +%Y%m%d_%H%M%S)
readonly LOG_FILE="/var/log/deployments/deploy_${TIMESTAMP}.log"

# Redirect all output to log file AND terminal
exec > >(tee -a "$LOG_FILE") 2>&1

log_info() { echo "[${TIMESTAMP}] [INFO]  $*"; }
log_error() { echo "[${TIMESTAMP}] [ERROR] $*" >&2; }
die() { log_error "$*"; exit 1; }

check_prerequisites() {
    local required_tools=("kubectl" "helm" "aws")
    for tool in "${required_tools[@]}"; do
        command -v "$tool" &>/dev/null || die "Required tool not found: $tool"
    done
}

validate_environment() {
    local env="$1"
    [[ "$env" =~ ^(production|staging|dev)$ ]] || die "Invalid env: $env"
}

deploy() {
    local env="$1"
    local version="$2"

    log_info "Deploying version $version to $env"

    # Set kubeconfig
    export KUBECONFIG="/etc/kubernetes/${env}.kubeconfig"

    # Helm upgrade
    helm upgrade --install myapp ./charts/myapp \
        --namespace "$env" \
        --set image.tag="$version" \
        --set environment="$env" \
        --wait \
        --timeout 5m \
        --atomic    # rollback on failure

    log_info "Deployment complete"
}

main() {
    [[ $# -ne 2 ]] && { echo "Usage: $0 <env> <version>"; exit 1; }

    local env="$1"
    local version="$2"

    check_prerequisites
    validate_environment "$env"
    deploy "$env" "$version"
}

main "$@"
```

---

## 6. Commands / Configs / Code Examples

### Text Processing in Scripts

```bash
# Parse CSV
while IFS=',' read -r host port service; do
    echo "Checking $service at $host:$port"
    nc -z "$host" "$port" || log_error "$service is down!"
done < services.csv

# Extract values from JSON (with jq)
VERSION=$(curl -s https://api.example.com/version | jq -r '.version')

# Multi-line string
SQL=$(cat <<'EOF'
SELECT id, name
FROM users
WHERE active = true
ORDER BY created_at DESC;
EOF
)

# Heredoc to file
cat > /etc/myapp/config.yaml <<EOF
database:
  host: ${DB_HOST}
  port: ${DB_PORT}
  name: ${DB_NAME}
environment: ${ENV}
EOF
```

### Parallel Execution

```bash
# Run commands in parallel
for server in "${SERVERS[@]}"; do
    ssh "$server" "apt-get update -y" &
done
wait   # wait for all background jobs

# With GNU parallel
parallel ssh {} "apt-get update -y" ::: "${SERVERS[@]}"

# Parallel with job limit
for i in {1..100}; do
    process_item "$i" &
    # Limit to 10 parallel jobs
    (( (i % 10) == 0 )) && wait
done
wait
```

### Lock Files (Preventing Concurrent Runs)

```bash
LOCKFILE="/tmp/myapp.lock"

if ! mkdir "$LOCKFILE" 2>/dev/null; then
    echo "Script already running. Lock: $LOCKFILE"
    exit 1
fi

trap "rmdir '$LOCKFILE'" EXIT
```

Using `mkdir` for locks is atomic — unlike `touch`.

---

## 7. Debugging & Troubleshooting

```bash
# Trace execution
set -x   # enable
set +x   # disable

# Selective tracing
{
    set -x
    critical_function
    set +x
} 2>&1 | grep "^+"

# Check what a script does without running it
bash -n script.sh    # syntax check only

# Verbose mode
bash -v script.sh    # print each line before executing
bash -x script.sh    # print each command after expansion

# Debug specific lines
PS4='+(${BASH_SOURCE}:${LINENO}): ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'
# Makes trace output include file:line:function

# Print all variables
declare -p
declare -p VAR_NAME

# Check IFS
echo "${IFS@Q}"   # q quotes — shows special chars

# Test regexp
[[ "hello123" =~ ^[a-z]+[0-9]+$ ]] && echo "match" || echo "no match"
```

---

## 8. Best Practices

- **Always use `set -euo pipefail`** at the top of every script.
- **Quote all variable expansions**: `"$VAR"`, not `$VAR`. Unquoted variables split on spaces.
- **Use `[[ ]]` instead of `[ ]`** for conditionals.
- **Use `$(command)` instead of backticks** for command substitution.
- **Use `local` in functions** to prevent variable leakage.
- **Use `readonly` for constants**.
- **Always validate input** before using it in commands.
- **Use `trap` for cleanup** on EXIT, ERR, INT, TERM.
- **Log with timestamps** and send errors to stderr.
- **Test scripts with `shellcheck`** before deploying.

---

## 9. Common Mistakes

- **Not quoting variables**: `rm -rf $DIR` where `$DIR` is empty becomes `rm -rf /`.
- **Ignoring exit codes**: `grep "ERROR" log.txt; echo "done"` — grep returns 1 if no match, but script continues.
- **Pipe subshell variable scope**: Variables set inside `cmd | while read` are in a subshell and lost after the loop.
- **`set -e` doesn't catch everything**: `VAR=$(cmd)` doesn't trigger `set -e` on failure in some bash versions.
- **Globs that expand to nothing**: `for f in *.txt` with no .txt files gives you the literal string `*.txt`.
- **Arithmetic overflow**: Bash integers are 64-bit but be careful with large numbers.

---

## 10. Senior Engineer Mental Models

**"Bash is text manipulation with side effects."** The mental model is: commands produce text, you transform it, you pipe it somewhere. When something breaks, trace the text flow.

**"Always assume the worst input."** Empty variables, spaces in filenames, newlines in output, zero results from grep.

**"Scripts are not programs — they're automation glue."** For complex logic, use Python. For system glue, file operations, and one-time tasks, bash is ideal.

**"Side effects are the entire point."** Unlike pure functions, bash scripts create files, make network calls, modify systems. Every side effect must be reversible or you need a rollback plan.

---

## 11. Interview-Level Knowledge

- What does `2>&1` mean, and what's the difference from `&>`?
- Why does `set -e` not always trigger on failed subshell commands?
- Explain the difference between `source` and executing a script.
- What is word splitting and how does quoting prevent it?
- How do you handle arrays with spaces in bash?
- What is `$$`, `$!`, `$?`, `$#`, `$@`, `$*`, and `$0`?
- When would you use `xargs` vs a `for` loop?

---

## 12. Advanced Insights

- **Bash 5** added many features: associative arrays, `@Q` quoting, `EPOCHSECONDS`, `EPOCHREALTIME`.
- **ShellCheck** (shellcheck.net) is a static analyzer for shell scripts. Use it in CI.
- **Bats** (Bash Automated Testing System) allows unit testing bash functions.
- **`envsubst`** is a great tool for template substitution in configs using env vars.
- **Process substitution** `<(cmd)` creates a virtual file from command output:
  ```bash
  diff <(ssh host1 cat /etc/passwd) <(ssh host2 cat /etc/passwd)
  ```

---

## 13. Summary

Bash scripting mastery means: safe defaults (`set -euo pipefail`), defensive quoting, proper error handling with traps, structured functions, input validation, and knowing when bash is the right tool vs when to reach for Python. Production bash scripts should be treated as code: version controlled, tested, reviewed, and documented.

---

---

# PART 3: PYTHON (FOR DEVOPS)

---

## 1. Overview

Python in DevOps is not about data science or web frameworks — it's about automation, tooling, API interaction, configuration management, and infrastructure scripting. Python handles what bash can't: complex logic, data structures, error handling, testing, HTTP clients, JSON parsing, and more.

The key libraries for a DevOps engineer: `requests`, `boto3`, `subprocess`, `pathlib`, `os`, `sys`, `logging`, `argparse`, `yaml`, `json`, `paramiko`, `kubernetes`, `docker`.

---

## 2. Core Concepts

### Python Execution Model

Python is an **interpreted** language. The interpreter compiles source to **bytecode** (.pyc files), then the **Python Virtual Machine (PVM)** executes the bytecode.

```
source.py → CPython compiler → bytecode (.pyc) → PVM executes
```

The **GIL (Global Interpreter Lock)** in CPython means only one thread executes Python bytecode at a time. For CPU-bound parallelism, use `multiprocessing`. For I/O-bound (network, file) parallelism, use `threading` or `asyncio`.

### Key Data Types

```python
# Strings — immutable
s = "hello"
s.upper()           # HELLO
s.strip()           # remove whitespace
s.split(",")        # list
",".join(["a","b"]) # "a,b"
f"Value: {s!r}"     # f-string with repr
s.startswith("he")
s.endswith("lo")

# Lists — mutable, ordered
lst = [1, 2, 3, 4, 5]
lst.append(6)
lst.extend([7, 8])
lst.insert(0, 0)
lst.pop()           # removes last
lst[1:3]            # slicing: [2, 3]
lst[::-1]           # reverse
sorted(lst)
lst.sort(key=lambda x: -x)

# Dicts — mutable, ordered (Python 3.7+)
d = {"host": "localhost", "port": 5432}
d.get("host", "default")     # safe get with default
d.keys(), d.values(), d.items()
d.update({"timeout": 30})
{k: v for k, v in d.items() if v}  # dict comprehension

# Sets
s = {1, 2, 3}
s.add(4)
s1 & s2    # intersection
s1 | s2    # union
s1 - s2    # difference

# Tuples — immutable
t = (1, 2, 3)
a, b, c = t    # unpacking
```

### Functions

```python
# Default arguments
def deploy(env, version, dry_run=False, timeout=300):
    ...

# *args and **kwargs
def log(*args, level="INFO", **kwargs):
    print(f"[{level}]", *args, kwargs)

# Type hints (crucial for production code)
from typing import Optional, List, Dict, Tuple

def get_pods(namespace: str, label: Optional[str] = None) -> List[Dict]:
    ...

# Decorators
import functools
import time

def retry(max_attempts=3, delay=1):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts:
                        raise
                    print(f"Attempt {attempt} failed: {e}. Retrying...")
                    time.sleep(delay * attempt)
        return wrapper
    return decorator

@retry(max_attempts=5, delay=2)
def call_api(url):
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    return response.json()
```

### Error Handling

```python
# Specific exception handling
try:
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    data = response.json()
except requests.exceptions.ConnectionError as e:
    logger.error(f"Cannot connect to {url}: {e}")
    raise SystemExit(1)
except requests.exceptions.Timeout:
    logger.error(f"Request to {url} timed out")
    raise
except requests.exceptions.HTTPError as e:
    logger.error(f"HTTP error: {e.response.status_code}")
    raise
except Exception as e:
    logger.exception(f"Unexpected error: {e}")  # logs traceback
    raise
finally:
    # Always runs
    cleanup()
```

---

## 3. Real-World DevOps Python Patterns

### Running System Commands

```python
import subprocess
import shlex

def run_command(cmd: str | list, check=True, capture=True, timeout=60) -> subprocess.CompletedProcess:
    """
    Run a shell command safely.
    - Accepts string or list
    - Returns CompletedProcess with stdout/stderr
    - Raises CalledProcessError on failure if check=True
    """
    if isinstance(cmd, str):
        # Use shlex.split for safe tokenization
        cmd = shlex.split(cmd)
    
    result = subprocess.run(
        cmd,
        capture_output=capture,
        text=True,          # string output, not bytes
        check=check,        # raise on non-zero exit
        timeout=timeout
    )
    return result

# Usage
result = run_command("kubectl get pods -n production -o json")
pods = json.loads(result.stdout)

# Don't do this — shell injection risk
# subprocess.run(f"kubectl delete pod {pod_name}", shell=True)
# Do this instead:
subprocess.run(["kubectl", "delete", "pod", pod_name], check=True)
```

### Working with Files and Paths

```python
from pathlib import Path
import yaml
import json

# Pathlib — the modern way
config_dir = Path("/etc/myapp")
config_file = config_dir / "config.yaml"

# Read
if config_file.exists():
    config = yaml.safe_load(config_file.read_text())

# Write
(config_dir / "output.json").write_text(json.dumps(data, indent=2))

# Iterate
for log_file in Path("/var/log").glob("*.log"):
    process_log(log_file)

# Create directories
Path("/opt/app/data").mkdir(parents=True, exist_ok=True)

# Temp files
import tempfile
with tempfile.NamedTemporaryFile(mode='w', suffix='.yaml', delete=False) as f:
    yaml.dump(config, f)
    temp_path = f.name
```

### HTTP and API Interaction

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

def create_session(retries=3, backoff=0.5) -> requests.Session:
    """Create a requests session with retry and timeout."""
    session = requests.Session()
    
    retry_strategy = Retry(
        total=retries,
        backoff_factor=backoff,
        status_forcelist=[429, 500, 502, 503, 504],
        allowed_methods=["GET", "POST"]
    )
    
    adapter = HTTPAdapter(max_retries=retry_strategy)
    session.mount("https://", adapter)
    session.mount("http://", adapter)
    
    return session

# Usage
session = create_session()
response = session.get(
    "https://api.example.com/deployments",
    headers={"Authorization": f"Bearer {token}"},
    timeout=(5, 30)   # (connect_timeout, read_timeout)
)
response.raise_for_status()
```

### Logging

```python
import logging
import sys
from datetime import datetime

def setup_logging(level: str = "INFO") -> logging.Logger:
    logger = logging.getLogger("myapp")
    logger.setLevel(getattr(logging, level.upper()))

    handler = logging.StreamHandler(sys.stdout)
    formatter = logging.Formatter(
        fmt='%(asctime)s [%(levelname)s] %(name)s: %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    
    return logger

logger = setup_logging("DEBUG")
logger.info("Deployment started")
logger.error("Failed: %s", error_message)  # use % style for lazy formatting
logger.exception("Unexpected error")        # includes traceback automatically
```

### Environment and Configuration Management

```python
import os
from dataclasses import dataclass

@dataclass
class Config:
    db_host: str
    db_port: int
    db_name: str
    debug: bool
    log_level: str
    
    @classmethod
    def from_env(cls) -> "Config":
        return cls(
            db_host=os.environ["DB_HOST"],     # raise KeyError if missing
            db_port=int(os.environ.get("DB_PORT", "5432")),
            db_name=os.environ["DB_NAME"],
            debug=os.environ.get("DEBUG", "false").lower() == "true",
            log_level=os.environ.get("LOG_LEVEL", "INFO"),
        )

config = Config.from_env()
```

### AWS with boto3

```python
import boto3
from botocore.exceptions import ClientError

def get_secret(secret_name: str, region: str = "us-east-1") -> dict:
    """Retrieve secret from AWS Secrets Manager."""
    client = boto3.client("secretsmanager", region_name=region)
    
    try:
        response = client.get_secret_value(SecretId=secret_name)
        secret_string = response["SecretString"]
        return json.loads(secret_string)
    except ClientError as e:
        error_code = e.response["Error"]["Code"]
        if error_code == "ResourceNotFoundException":
            raise ValueError(f"Secret {secret_name} not found")
        raise

def list_ec2_instances(tag_name: str, tag_value: str) -> list:
    ec2 = boto3.resource("ec2")
    instances = ec2.instances.filter(
        Filters=[
            {"Name": f"tag:{tag_name}", "Values": [tag_value]},
            {"Name": "instance-state-name", "Values": ["running"]}
        ]
    )
    return [
        {
            "id": i.id,
            "type": i.instance_type,
            "private_ip": i.private_ip_address,
            "state": i.state["Name"]
        }
        for i in instances
    ]
```

### CLI Tools with argparse

```python
#!/usr/bin/env python3
import argparse
import sys

def main():
    parser = argparse.ArgumentParser(
        description="Deploy application to environment",
        formatter_class=argparse.RawDescriptionHelpFormatter
    )
    
    parser.add_argument("environment", choices=["dev", "staging", "production"])
    parser.add_argument("version", help="Docker image tag to deploy")
    parser.add_argument("--dry-run", action="store_true", help="Don't actually deploy")
    parser.add_argument("--timeout", type=int, default=300, help="Deployment timeout")
    parser.add_argument("--replicas", type=int, default=None)
    parser.add_argument("-v", "--verbose", action="store_true")
    
    args = parser.parse_args()
    
    if args.verbose:
        logging.getLogger().setLevel(logging.DEBUG)
    
    deploy(args.environment, args.version, dry_run=args.dry_run)

if __name__ == "__main__":
    main()
```

---

## 4. Architecture & Production Perspective

### Virtual Environments

```bash
# Create venv
python3 -m venv .venv

# Activate
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Freeze exact versions
pip freeze > requirements.txt

# Preferred: use pip-tools for dependency management
pip install pip-tools
# requirements.in (high-level deps)
pip-compile requirements.in    # generates requirements.txt with pinned versions
pip-sync requirements.txt      # installs exactly what's in the lock file
```

### Packaging Scripts for Production

```toml
# pyproject.toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.backends.legacy:build"

[project]
name = "deploy-tool"
version = "1.0.0"
requires-python = ">=3.10"
dependencies = [
    "requests>=2.28.0",
    "boto3>=1.26.0",
    "kubernetes>=26.1.0",
    "pyyaml>=6.0",
    "click>=8.1.0",
]

[project.scripts]
deploy = "deploy_tool.cli:main"
```

---

## 5. Debugging & Troubleshooting

```python
# pdb — Python debugger
import pdb; pdb.set_trace()  # breakpoint

# Python 3.7+
breakpoint()

# Remote debugging with debugpy (VS Code)
import debugpy
debugpy.listen(5678)
debugpy.wait_for_client()

# Profile CPU
python -m cProfile -o output.prof script.py
python -m pstats output.prof

# Profile memory
pip install memory-profiler
@profile
def my_function():
    ...

# Examine running process
pip install pyspy
py-spy top --pid 1234
py-spy dump --pid 1234    # stack trace without stopping
```

---

## 6. Best Practices

- **Use type hints** everywhere. They document intent and enable static analysis.
- **Use `pathlib`** not `os.path` for file operations.
- **Never use `shell=True`** in subprocess unless absolutely necessary — it's a security risk.
- **Use `requests.Session`** with retry logic for all HTTP calls.
- **Handle secrets via environment variables**, never hardcode.
- **Use `logging`** not `print` for any code that goes to production.
- **Write tests** — even DevOps scripts should have pytest tests.
- **Use `black` for formatting**, `flake8` or `ruff` for linting, `mypy` for type checking.
- **Use dataclasses or Pydantic** for structured config and data models.

---

## 7. Common Mistakes

- **Mutable default arguments**: `def f(lst=[])` — the list is shared across all calls. Use `None` default.
- **Catching broad exceptions**: `except Exception` hides bugs. Catch specific exceptions.
- **`os.system()` instead of `subprocess`**: `os.system` gives you no control over output.
- **Not closing file handles**: Always use `with open()` context managers.
- **Blocking the event loop**: Using `requests` in async code — use `aiohttp` instead.
- **Integer division confusion**: `1/2 == 0.5` in Python 3, `1//2 == 0` for integer division.
- **Dictionary mutation during iteration**: Causes `RuntimeError`. Iterate over `list(d.items())`.

---

## 8. Senior Engineer Mental Models

**"Python is the right tool when bash becomes unreadable."** The threshold is roughly: conditional logic + data structures + error handling = use Python.

**"Configuration is code."** Treat your Python scripts as production software: version control, CI, tests, dependencies pinned.

**"The subprocess module is your bridge to the OS."** Everything you can do in bash, you can do in Python via subprocess — but with proper error handling and structured output.

---

## 9. Interview-Level Knowledge

- Explain the GIL and when it matters.
- What's the difference between `deepcopy` and `copy`?
- How do Python decorators work at the bytecode level?
- What is a generator and how does `yield` work?
- Explain `*args`, `**kwargs`, and keyword-only arguments.
- What is `__slots__` and when would you use it?
- How does Python's memory management work (reference counting + cyclic GC)?

---

## 10. Summary

Python for DevOps is about clean automation: subprocess for system commands, requests for APIs, boto3 for AWS, pathlib for files, logging for observability, and argparse for CLIs. Write production-quality code: typed, tested, logged, and exception-aware.

---

---

# PART 4: GIT

---

## 1. Overview

Git is a **distributed version control system**. Every repository is a complete history. There is no central server requirement — but in practice, teams use GitHub, GitLab, or Bitbucket as a shared remote.

In DevOps, Git is the trigger for everything: a push to `main` starts a CI pipeline, a merge to `release` triggers a deployment. Git isn't just version control — it's the entry point to your entire automation system.

---

## 2. Core Concepts

### The Git Object Model

Git stores everything as **content-addressed objects** in `.git/objects/`. There are four types:

- **blob**: File contents (no filename, just content)
- **tree**: Directory structure (maps names to blobs and other trees)
- **commit**: Points to a tree, has author/message/date, points to parent commit(s)
- **tag**: Named pointer to a commit (annotated tags)

Every object is identified by its **SHA-1 hash** (40 hex chars). The hash is computed from the content — change any bit and you get a different hash. This is what makes Git tamper-evident.

```
commit a1b2c3d
  │
  ├── tree e4f5g6h
  │     ├── blob: README.md (sha: 111...)
  │     ├── blob: app.py    (sha: 222...)
  │     └── tree: src/
  │           └── blob: main.py (sha: 333...)
  │
  └── parent: 9z8y7x6w (previous commit)
```

### The Three Areas

```
Working Directory    ──── git add ────►    Staging Area (Index)    ──── git commit ────►    Repository
(your files)                               (.git/index)                                    (.git/objects)

◄──── git checkout ──────────────────────────────────────────────────────────────────────
◄──── git restore --staged ──────────────
◄──── git restore ────
```

- **Working directory**: Your actual files on disk
- **Staging area (index)**: Snapshot of what will go into the next commit
- **Repository**: The commit history and object store

### Refs and HEAD

A **ref** is a named pointer to a commit SHA:

```
refs/heads/main         → commit SHA  (branches)
refs/remotes/origin/main → commit SHA (remote tracking)
refs/tags/v1.0.0        → tag object or commit SHA

HEAD → refs/heads/main → commit SHA   (current branch)
HEAD → commit SHA                      (detached HEAD state)
```

**Detached HEAD** means you've checked out a specific commit, not a branch. New commits won't be tracked by any branch.

---

## 3. Internal Working

### How `git commit` Works

1. Git hashes all staged file contents → creates blob objects
2. Creates tree objects representing the directory structure
3. Creates a commit object with: tree hash, parent commit hash, author, timestamp, message
4. Updates the current branch ref to point to the new commit

### How `git merge` Works

**Fast-forward merge**: If the target branch is directly ahead of the current branch — just move the pointer forward. No merge commit.

**Three-way merge**: Git finds the common ancestor, compares the two branch tips against it, and combines changes. A merge commit is created with two parents.

**Conflict**: When both branches changed the same lines since the common ancestor. Git marks the conflict in the file and requires manual resolution.

### How `git rebase` Works

Rebase replays commits from one branch on top of another. Each original commit is re-applied (creating new commits with new SHAs):

```
Before rebase:
  A - B - C (main)
       \
        D - E (feature)

After: git rebase main (from feature branch):
  A - B - C (main)
               \
                D' - E' (feature)
```

D' and E' are new commits — same changes but different parent, different SHA.

**Golden rule**: Never rebase commits that have been pushed to a shared remote. Rebase rewrites history, causing divergence for anyone who has the old commits.

---

## 4. Real-World Usage

### Everyday Commands

```bash
# Initialize and clone
git init
git clone https://github.com/org/repo.git
git clone --depth 1 repo.git       # shallow clone (no history)
git clone -b main repo.git         # clone specific branch

# Status and inspection
git status
git status -s                      # short format
git diff                           # working dir vs index
git diff --staged                  # index vs last commit
git diff main feature              # between branches
git log --oneline --graph --all    # visual history
git log --oneline -20              # last 20 commits
git show abc1234                   # show a specific commit

# Staging
git add file.py
git add -p                         # interactive staging (hunks)
git add -u                         # add all tracked changed files
git add .                          # add everything (careful!)

# Committing
git commit -m "feat: add retry logic to API client"
git commit --amend                 # modify last commit (before push)
git commit --amend --no-edit      # amend without changing message

# Branching
git branch                         # list local branches
git branch -a                      # list all including remote
git branch feature/new-thing       # create branch
git checkout feature/new-thing     # switch (old style)
git switch feature/new-thing       # switch (new style)
git switch -c feature/new-thing    # create and switch
git branch -d feature/done         # delete (safe)
git branch -D feature/abandon      # force delete

# Remote operations
git fetch origin                   # fetch all remote changes (no merge)
git fetch origin main              # fetch specific branch
git pull origin main               # fetch + merge
git pull --rebase origin main      # fetch + rebase
git push origin feature            # push branch
git push -u origin feature         # push and set upstream
git push origin --delete old-branch  # delete remote branch
git push --force-with-lease        # safer force push (checks remote state)
```

### Stashing

```bash
git stash                          # save dirty state
git stash push -m "WIP: fixing bug"
git stash list                     # show all stashes
git stash pop                      # apply and remove top stash
git stash apply stash@{2}          # apply specific stash
git stash drop stash@{0}           # discard stash
git stash branch feature/wip       # create branch from stash
```

### Undoing Things

```bash
# Undo working directory changes
git restore file.py                # discard uncommitted changes (permanent!)
git checkout -- file.py            # old syntax

# Unstage
git restore --staged file.py

# Undo last commit (keep changes staged)
git reset --soft HEAD~1

# Undo last commit (keep changes unstaged)
git reset --mixed HEAD~1

# Undo last commit (discard all changes)
git reset --hard HEAD~1            # DANGER: changes are lost

# Safe undo (creates new commit that reverses a previous one)
git revert abc1234                 # use this on shared branches
git revert HEAD~3..HEAD            # revert range of commits

# Find lost commits
git reflog                         # every HEAD movement
git checkout abc1234               # restore to that point
```

### Tagging

```bash
git tag v1.0.0                           # lightweight tag
git tag -a v1.0.0 -m "Release 1.0.0"    # annotated tag (preferred)
git tag -a v1.0.0 abc1234               # tag specific commit
git push origin v1.0.0                  # push specific tag
git push origin --tags                  # push all tags
git tag -d v1.0.0                       # delete local tag
git push origin --delete v1.0.0         # delete remote tag
```

---

## 5. Architecture & Production Perspective

### Branching Strategies

**Git Flow**: Long-lived `main` and `develop` branches, feature branches off `develop`, release branches, hotfix branches. Good for software with formal releases. Overhead-heavy.

**Trunk-Based Development**: Everyone commits to `main` (or short-lived feature branches <1-2 days). Feature flags for incomplete features. Requires strong CI. Best for continuous delivery. **Preferred in DevOps culture.**

**GitHub Flow**: Main branch is always deployable. Feature branches, PRs, merge to main, deploy. Simple and works well for web applications.

For DevOps infrastructure code (Terraform, Kubernetes manifests), trunk-based development with environment branches or ArgoCD/Flux-based GitOps is common.

### Git Hooks for Automation

```bash
# .git/hooks/ — client-side hooks
pre-commit          # before commit (run linters, formatters)
commit-msg          # validate commit message format
pre-push            # before push (run tests)
post-checkout       # after checkout (update dependencies)

# Example: pre-commit hook
cat .git/hooks/pre-commit
```

```bash
#!/bin/bash
set -euo pipefail

echo "Running pre-commit checks..."

# Lint
shellcheck scripts/*.sh
black --check .
flake8 .

# Security scan
git diff --cached --name-only | xargs grep -l "password\|secret\|api_key" \
  && echo "Potential secret detected in staged files!" && exit 1

echo "Pre-commit checks passed."
```

Use the **pre-commit** framework instead of managing hooks manually:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: check-yaml
      - id: detect-private-key
      - id: trailing-whitespace
  - repo: https://github.com/psf/black
    rev: 23.1.0
    hooks:
      - id: black
  - repo: https://github.com/shellcheck-py/shellcheck-py
    rev: v0.9.0.2
    hooks:
      - id: shellcheck
```

### `.gitignore`

```gitignore
# Python
__pycache__/
*.pyc
.venv/
dist/
*.egg-info/

# Terraform
.terraform/
*.tfstate
*.tfstate.backup
*.tfvars     # may contain secrets — use .tfvars.example

# Secrets — NEVER commit these
.env
*.pem
*.key
*_rsa
credentials.json

# OS
.DS_Store
Thumbs.db

# Editor
.idea/
.vscode/
```

### `.gitattributes`

```gitattributes
# Normalize line endings
* text=auto
*.sh text eol=lf
*.bat text eol=crlf

# Diff behavior
*.png binary
Dockerfile linguist-language=Dockerfile
```

---

## 6. Advanced Git Operations

```bash
# Cherry-pick specific commits
git cherry-pick abc1234          # apply commit to current branch
git cherry-pick abc1234..def5678  # range (exclusive..inclusive)

# Interactive rebase — rewrite history
git rebase -i HEAD~5             # edit last 5 commits
# Commands: pick, reword, edit, squash, fixup, drop

# Bisect — binary search for bug-introducing commit
git bisect start
git bisect bad HEAD              # current is broken
git bisect good v1.0.0           # known good point
# Git checks out middle commit
git bisect good/bad              # test and tell git result
git bisect reset                 # done

# Grep through history
git log -S "function_name"       # commits that add/remove this string
git log -G "regex pattern"       # commits where diff matches regex
git log --all --full-history -- "**/config.yaml"   # history of deleted file

# Sparse checkout (only checkout subset of files)
git sparse-checkout init --cone
git sparse-checkout set src/     # only checkout src/

# Worktrees (multiple working dirs, one repo)
git worktree add ../hotfix hotfix/urgent-bug
```

---

## 7. Debugging & Troubleshooting

```bash
# Who changed this line?
git blame file.py
git blame -L 10,20 file.py     # only lines 10-20

# What changed in this file between versions?
git log --oneline -- path/to/file
git diff v1.0.0..v1.1.0 -- path/to/file

# Recover deleted file
git log --all --full-history -- deleted_file.py
git checkout <commit>^ -- deleted_file.py

# Fix last commit (didn't push yet)
git add forgotten_file.py
git commit --amend --no-edit

# Remove file from all history (nuclear option)
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch passwords.txt' \
  --prune-empty --tag-name-filter cat -- --all
# Or use BFG Repo Cleaner (faster)

# Uncommit but keep changes
git reset HEAD~1

# What would git push do?
git log origin/main..HEAD --oneline
```

---

## 8. Best Practices

- **Commit messages follow Conventional Commits**: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`. This enables automated changelog generation.
- **Small, focused commits**: One logical change per commit. Makes `git bisect` and review easier.
- **Never force push to shared branches**: Use `--force-with-lease` if you must, and only on your own branches.
- **Sign commits**: `git config commit.gpgsign true`. Proves authorship in high-security environments.
- **Never commit secrets**: Use git-secrets, gitleaks, or truffleHog in pre-commit and CI.
- **Use `.gitignore` religiously**: Especially for credentials, build artifacts, and editor files.
- **Prefer `git revert` over `git reset`** on shared branches. Revert creates history; reset erases it.
- **Tag releases**: Every production deployment should correspond to a git tag.

---

## 9. Common Mistakes

- **Committing secrets or credentials.** Even if you delete them in the next commit, they remain in history. Use `git filter-branch` or BFG to scrub, then rotate the secret immediately.
- **Rebasing public branches.** Anyone who pulled the old commits will have a diverged history.
- **Giant commits with "misc fixes."** Makes review impossible, bisect useless.
- **Not using `.gitignore`** and committing `node_modules/` or `.terraform/`.
- **Merge conflicts in lock files.** Resolve by regenerating: `npm install` or `pip-compile`.
- **Force pushing to `main`** on shared repos — catastrophic.

---

## 10. Senior Engineer Mental Models

**"Git history is documentation."** A well-maintained commit log tells the story of why the code is the way it is. Bad commit messages rob future engineers of context.

**"The working tree is always throwaway."** Anything not committed can be lost. Commit early and often; clean up with interactive rebase before pushing.

**"Branches are cheap — use them."** A branch is just 41 bytes (a SHA reference). Create branches freely for experiments, features, fixes.

**"Git is append-only."** Even `git reset --hard` doesn't delete objects — they linger for 90 days and are reachable via reflog. This is your safety net.

---

## 11. Interview-Level Knowledge

- What is the difference between `merge` and `rebase`? When do you use each?
- Explain how `git reset --soft`, `--mixed`, and `--hard` differ.
- What is a detached HEAD and how do you recover from it?
- How does `git stash` work internally?
- What does `git pull --rebase` do differently from `git pull`?
- How would you find which commit introduced a bug?
- What is a merge conflict at the object level?

---

## 12. Summary

Git is not just version control — it's the foundation of your entire automation pipeline. Master the object model, the three-area mental model, branching strategies, and history rewriting tools. Write meaningful commit messages, protect shared branches, and treat secrets as the highest-risk category of content to keep out of repos.

---

---

# PART 5: NETWORKING BASICS

---

## 1. Overview

Networking is the circulatory system of distributed systems. Without networking knowledge, you cannot debug connection failures, configure load balancers, understand DNS propagation, tune TCP for performance, or design secure infrastructure. Every service you run communicates over a network — you need to understand the medium.

---

## 2. Core Concepts

### The OSI Model

The OSI model provides a conceptual framework. As a DevOps engineer, you mostly work with layers 3-7:

```
Layer 7 — Application   HTTP, HTTPS, DNS, SMTP, SSH, gRPC
Layer 6 — Presentation  TLS/SSL, encoding, compression
Layer 5 — Session       Session management (TCP connections, cookies)
Layer 4 — Transport     TCP, UDP — ports, reliability, flow control
Layer 3 — Network       IP — routing, addressing, subnets
Layer 2 — Data Link     Ethernet, MAC addresses, switches, VLANs
Layer 1 — Physical      Cables, fiber, wireless signals
```

When debugging, you work top-down: "Is the app responding? Is the TCP connection establishing? Can we reach the IP? Is DNS resolving?"

### IP Addressing

**IPv4**: 32-bit address, written as four octets: `192.168.1.100`

**CIDR Notation**: `192.168.1.0/24` means the first 24 bits are the network address, leaving 8 bits (256 addresses) for hosts.

```
192.168.1.0/24
├── Network address:   192.168.1.0    (not usable)
├── Broadcast address: 192.168.1.255  (not usable)
├── Usable hosts:      192.168.1.1 to 192.168.1.254 (254 hosts)
└── Subnet mask:       255.255.255.0
```

**Key CIDR sizes for production:**

```
/32   — single host (1 IP)                            Security group rules, route to one host
/31   — 2 IPs (point-to-point link)
/30   — 4 IPs, 2 usable
/29   — 8 IPs, 6 usable
/28   — 16 IPs, 14 usable
/27   — 32 IPs, 30 usable
/26   — 64 IPs, 62 usable
/25   — 128 IPs, 126 usable
/24   — 256 IPs, 254 usable                          Typical subnet
/22   — 1024 IPs, 1022 usable
/20   — 4096 IPs
/16   — 65536 IPs                                    VPC range
/8    — 16 million IPs
```

**Private IP ranges (RFC 1918)** — not routable on the public internet:
- `10.0.0.0/8` (10.x.x.x)
- `172.16.0.0/12` (172.16.x.x to 172.31.x.x)
- `192.168.0.0/16` (192.168.x.x)

**Loopback**: `127.0.0.0/8` — `127.0.0.1` is localhost.
**Link-local**: `169.254.0.0/16` — auto-assigned when DHCP fails. You'll see this on AWS EC2 instances accessing instance metadata.

### TCP vs UDP

**TCP (Transmission Control Protocol)**:
- Connection-oriented (3-way handshake before data)
- Reliable: acknowledgment, retransmission, ordering
- Flow control and congestion control
- Overhead: ~20 byte header + connection state
- Use when: correctness matters (HTTP, SSH, databases, file transfer)

**UDP (User Datagram Protocol)**:
- Connectionless — send and forget
- No reliability guarantees
- Very low overhead
- Use when: speed > reliability (DNS, DHCP, video streaming, games, metrics)

**TCP Three-Way Handshake**:

```
Client                    Server
  │                          │
  │──── SYN ────────────────►│
  │                          │
  │◄─── SYN-ACK ─────────────│
  │                          │
  │──── ACK ────────────────►│
  │                          │
  │     [Connection ESTABLISHED]
  │                          │
  │◄═══ Data Exchange ══════►│
  │                          │
  │──── FIN ────────────────►│
  │◄─── FIN-ACK ─────────────│
  │──── ACK ────────────────►│
  │                          │
  [Connection CLOSED — TIME_WAIT on client]
```

**TIME_WAIT**: A closed TCP connection stays in TIME_WAIT for 2*MSL (Maximum Segment Lifetime, typically 60s) to handle delayed packets. Under high connection rates, you can exhaust ports. Tuned with `net.ipv4.tcp_tw_reuse`.

### DNS

DNS (Domain Name System) translates names to IPs. It's a hierarchical, distributed database.

```
DNS Resolution for app.example.com:

Client
  │
  ├─► Checks /etc/hosts (first)
  ├─► Checks local DNS cache
  ├─► Queries Recursive Resolver (your ISP or 8.8.8.8)
  │     │
  │     ├─► Root DNS servers (knows .com nameservers)
  │     ├─► .com TLD nameservers (knows example.com nameservers)
  │     └─► example.com nameservers (knows app.example.com IP)
  │
  └─► Returns: 93.184.216.34 (cached per TTL)
```

**Record types:**

```
A        name → IPv4 address      example.com → 1.2.3.4
AAAA     name → IPv6 address
CNAME    name → another name      www → example.com (canonical alias)
MX       mail server for domain
NS       authoritative nameserver for domain
TXT      arbitrary text (SPF, DKIM, domain verification)
SOA      Start of Authority (zone metadata)
PTR      IP → name (reverse DNS)
SRV      service location (_http._tcp.example.com → host:port)
```

**TTL (Time To Live)**: How long resolvers cache the record. Low TTL (30s) for quick failover. High TTL (3600s) for performance.

**DNS in Kubernetes**: CoreDNS provides in-cluster DNS. Services get `service.namespace.svc.cluster.local`. This is how service discovery works.

### Ports and Sockets

A **socket** is the combination of IP + Port + Protocol. A connection is uniquely identified by the **5-tuple**: source IP, source port, destination IP, destination port, protocol.

```
Common well-known ports:
22    SSH
25    SMTP
53    DNS (TCP + UDP)
80    HTTP
443   HTTPS
3306  MySQL
5432  PostgreSQL
6379  Redis
9200  Elasticsearch
9090  Prometheus
2181  Zookeeper
9092  Kafka
2379  etcd (Kubernetes)
10250 kubelet API
```

Ephemeral ports (client-side): 32768-60999 on Linux (configurable via `ip_local_port_range`).

### Routing

Routing determines where to send packets. The OS maintains a **routing table**:

```bash
ip route show
# Example output:
# default via 10.0.1.1 dev eth0
# 10.0.0.0/8 via 10.0.1.1 dev eth0
# 10.0.1.0/24 dev eth0 proto kernel scope link

# Longest prefix match wins:
# Packet to 10.0.1.50:
#   - matches 10.0.0.0/8 (8-bit match)
#   - matches 10.0.1.0/24 (24-bit match — more specific, wins)
#   → send directly on eth0 (on-link)
```

**Default gateway**: The router for packets with no more specific route. Usually your cloud provider's router.

**NAT (Network Address Translation)**: Private IPs can't route on the internet. NAT rewrites source IPs on outbound packets (source NAT / masquerade) and reverses for inbound replies. Every cloud VPC uses this.

---

## 3. Real-World Usage

### Network Interface Management

```bash
# View interfaces
ip addr show
ip link show

# Add/remove IP
ip addr add 192.168.1.100/24 dev eth0
ip addr del 192.168.1.100/24 dev eth0

# Bring interface up/down
ip link set eth0 up
ip link set eth0 down

# View routing table
ip route show
ip route get 8.8.8.8      # which route would be used?

# Add route
ip route add 10.0.0.0/8 via 10.0.1.1
ip route del 10.0.0.0/8

# ARP table (IP → MAC mappings)
arp -n
ip neigh show
```

### Connectivity Testing

```bash
# Ping (ICMP)
ping -c 5 8.8.8.8
ping -c 5 -W 1 host      # 1s timeout per packet

# Trace route
traceroute 8.8.8.8
tracepath 8.8.8.8         # doesn't need root
mtr 8.8.8.8               # real-time traceroute

# Port connectivity
nc -zv host 443           # TCP connect test
nc -zvu host 53           # UDP test
telnet host 22            # old school TCP test

# Latency measurement
hping3 -S -p 80 host      # TCP ping
curl -o /dev/null -s -w "Connect: %{time_connect}s\nTTFB: %{time_starttransfer}s\nTotal: %{time_total}s\n" https://example.com

# Bandwidth test
iperf3 -s                  # on server
iperf3 -c server_ip        # on client
```

### DNS Troubleshooting

```bash
# Basic lookup
dig example.com
dig A example.com
dig AAAA example.com
dig MX example.com
dig TXT example.com

# Query specific DNS server
dig @8.8.8.8 example.com
dig @10.96.0.10 myservice.default.svc.cluster.local   # K8s CoreDNS

# Trace DNS resolution
dig +trace example.com

# Reverse lookup
dig -x 1.2.3.4
host 1.2.3.4

# Check local resolver config
cat /etc/resolv.conf
cat /etc/nsswitch.conf    # resolution order (files, dns, etc.)

# Flush DNS cache
systemd-resolve --flush-caches
# or
/etc/init.d/nscd restart
```

### Packet Capture

```bash
# tcpdump — captures packets
tcpdump -i eth0                        # all traffic
tcpdump -i eth0 port 80                # HTTP only
tcpdump -i eth0 host 1.2.3.4          # to/from specific IP
tcpdump -i eth0 "tcp[tcpflags] & tcp-syn != 0"  # SYN packets only
tcpdump -i eth0 -w capture.pcap        # write to file
tcpdump -r capture.pcap                # read from file

# Wireshark (GUI, runs on pcap files)
# Extremely useful for protocol analysis

# View current connections
ss -tulnp          # listening
ss -tnp            # established TCP connections
ss -s              # statistics summary
ss -tnp dst 1.2.3.4   # connections to specific IP
```

### iptables / nftables

```bash
# List rules
iptables -L -n -v             # all chains
iptables -L INPUT -n -v        # INPUT chain only
iptables -t nat -L -n -v       # NAT table

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow specific port
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Drop everything else
iptables -A INPUT -j DROP

# Masquerade (NAT) for outbound
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Save rules
iptables-save > /etc/iptables/rules.v4
iptables-restore < /etc/iptables/rules.v4
```

---

## 4. Architecture & Production Perspective

### VPC Network Design

In AWS (same principles in GCP/Azure):

```
VPC: 10.0.0.0/16

Public Subnets (have route to Internet Gateway):
  AZ-a: 10.0.1.0/24  (NAT Gateway lives here)
  AZ-b: 10.0.2.0/24
  AZ-c: 10.0.3.0/24

Private Subnets (route to NAT Gateway for outbound):
  AZ-a: 10.0.11.0/24  (Application tier)
  AZ-b: 10.0.12.0/24
  AZ-c: 10.0.13.0/24

Database Subnets (no direct outbound internet):
  AZ-a: 10.0.21.0/24
  AZ-b: 10.0.22.0/24
  AZ-c: 10.0.23.0/24

Internet Gateway — for public subnets
NAT Gateway (in each AZ for HA) — for private subnets outbound
VPC Endpoints — for AWS service access without internet (S3, ECR)
```

**Security Groups**: Stateful firewall at the instance/ENI level. You define inbound rules; matching outbound is automatically allowed.

**NACLs**: Stateless firewall at the subnet level. Both inbound AND outbound must be explicitly allowed.

### Load Balancer Types

**Layer 4 (TCP/UDP)**: Routes based on IP + port. Fastest, protocol-agnostic. AWS NLB. Used for high-throughput, low-latency scenarios.

**Layer 7 (HTTP/HTTPS)**: Routes based on URL path, headers, host. Supports SSL termination, sticky sessions, HTTP health checks. AWS ALB. Used for most web applications.

### Network Performance Tuning

```bash
# Increase socket receive/send buffers
sysctl -w net.core.rmem_max=134217728
sysctl -w net.core.wmem_max=134217728
sysctl -w net.ipv4.tcp_rmem="4096 87380 67108864"
sysctl -w net.ipv4.tcp_wmem="4096 65536 67108864"

# Enable TCP BBR (better congestion control, Linux 4.9+)
sysctl -w net.core.default_qdisc=fq
sysctl -w net.ipv4.tcp_congestion_control=bbr

# Increase listen backlog
sysctl -w net.core.somaxconn=65535
sysctl -w net.ipv4.tcp_max_syn_backlog=65535
```

---

## 5. Debugging & Troubleshooting

### Systematic Approach

```
Can't reach service at host:port
│
├── DNS resolves? → dig host
│     No → DNS problem (CoreDNS? /etc/resolv.conf? VPC DNS?)
│
├── Can ping the IP? → ping IP
│     No → Routing or firewall (iptables, Security Group, NACL)
│
├── TCP port open? → nc -zv IP port
│     No → Service not listening? Firewall blocking? 
│           → ss -tlnp on destination host
│
├── HTTP response? → curl -v http://host:port
│     No → Application problem
│     Connection refused → nothing listening
│     Connection reset → firewall/proxy issue
│     Timeout → network black hole (packet going somewhere wrong)
│
└── TLS handshake? → curl -v https://host
      Certificate error → cert mismatch or expired
      Handshake timeout → TLS version/cipher mismatch
```

---

## 6. Best Practices

- **Design for failure**: Assume any link can fail. Use multiple availability zones, health checks, and failover.
- **Keep subnets right-sized**: Too small = can't grow. Too large = wasted space and sloppy security boundaries.
- **Use VPC Endpoints** for AWS services — keeps traffic off the internet and reduces NAT Gateway costs.
- **Limit Security Group rules**: Don't allow 0.0.0.0/0 on anything except ALB port 80/443.
- **DNS TTLs**: Set low TTL before planned failovers. Set high TTL normally for performance.
- **Monitor connection table state**: High TIME_WAIT or CLOSE_WAIT counts indicate connection management issues.
- **Use TCP keepalives** for long-lived connections to detect dead peers.

---

## 7. Common Mistakes

- **Forgetting about TIME_WAIT** exhausting ephemeral ports under high connection rates.
- **Relying on DNS without caching considerations** — DNS TTL is not always respected by applications.
- **Overlapping CIDR blocks** when setting up VPC peering or VPNs.
- **Not testing both inbound AND outbound** when debugging connectivity (NACLs are stateless).
- **Assuming latency = bandwidth** — they're different dimensions. A satellite connection has high bandwidth but terrible latency.
- **Not accounting for DNS propagation time** when making DNS changes — TTL matters.

---

## 8. Senior Engineer Mental Models

**"Network is about state machines."** TCP connections have states (SYN_SENT, ESTABLISHED, TIME_WAIT). Understanding states explains most TCP problems.

**"When in doubt, packet capture."** `tcpdump` is the ultimate source of truth. If packets aren't arriving, nothing at the application level matters.

**"Routing is longest-prefix match."** When two routes match, the more specific one wins. This is how you can have a default route and still override specific subnets.

**"Security groups are deny-by-default."** If you haven't explicitly allowed it, it's blocked. This is the right mental model for zero-trust networking.

---

## 9. Interview-Level Knowledge

- Explain TCP's three-way handshake and four-way close.
- What is TIME_WAIT and why does it exist?
- How does NAT work, and why can't internal IPs reach the internet directly?
- What is the difference between a Security Group and an NACL in AWS?
- Explain how DNS resolution works from a browser hitting a URL to getting an IP.
- What is BGP and why does it matter for internet routing?
- How does a packet from an EC2 instance reach the internet?

---

## 10. Summary

Networking mastery means understanding: IP addressing and CIDR, TCP/UDP behavior and state, DNS resolution, routing tables and longest-prefix match, NAT and firewall rules, and packet-level debugging. Every production problem eventually touches the network — you need to be able to diagnose at every layer.

---

---

# PART 6: HTTP/HTTPS

---

## 1. Overview

HTTP is the application protocol that powers the web and most service-to-service communication. REST APIs, webhooks, health checks, Kubernetes API calls — all HTTP. HTTPS adds TLS for encryption and authentication. As a DevOps engineer, you'll debug HTTP failures constantly, configure HTTPS termination, manage certificates, tune timeouts, and understand caching behavior.

---

## 2. Core Concepts

### HTTP Request/Response Structure

```
HTTP Request:
─────────────────────────────────────
METHOD /path?query=value HTTP/1.1      ← Request line
Host: example.com                      ← Headers (key: value)
Authorization: Bearer eyJhbGc...
Content-Type: application/json
Content-Length: 45
Accept: application/json, */*

{"key": "value", "action": "deploy"}  ← Body (optional)
─────────────────────────────────────

HTTP Response:
─────────────────────────────────────
HTTP/1.1 200 OK                        ← Status line
Content-Type: application/json
Content-Length: 123
Cache-Control: no-cache
X-Request-ID: a3f9b2c1

{"status": "success", "id": "abc123"} ← Body
─────────────────────────────────────
```

### HTTP Methods

```
GET     — Retrieve resource. Safe, idempotent. No body.
POST    — Create resource or trigger action. Not idempotent.
PUT     — Replace resource entirely. Idempotent.
PATCH   — Partial update. Not necessarily idempotent.
DELETE  — Remove resource. Idempotent.
HEAD    — GET but response body. Used for metadata/health checks.
OPTIONS — What methods does this endpoint support? Used in CORS.
```

**Idempotent**: Making the same request multiple times has the same effect as making it once. Critical for retry logic — only retry idempotent methods safely.

### HTTP Status Codes

```
1xx — Informational
  100 Continue
  101 Switching Protocols (WebSocket upgrade)

2xx — Success
  200 OK
  201 Created (POST that created a resource)
  202 Accepted (async operation started)
  204 No Content (success, no body — common for DELETE)

3xx — Redirection
  301 Moved Permanently (update bookmarks — GET changes to GET)
  302 Found (temporary redirect)
  304 Not Modified (use your cached copy — ETag/Last-Modified)
  307 Temporary Redirect (method preserved)
  308 Permanent Redirect (method preserved)

4xx — Client Error
  400 Bad Request (malformed syntax)
  401 Unauthorized (not authenticated — needs credentials)
  403 Forbidden (authenticated but not authorized)
  404 Not Found
  405 Method Not Allowed
  408 Request Timeout
  409 Conflict (e.g., resource already exists)
  410 Gone (permanently removed)
  422 Unprocessable Entity (valid syntax, invalid semantics)
  429 Too Many Requests (rate limited)

5xx — Server Error
  500 Internal Server Error
  502 Bad Gateway (proxy got invalid response from upstream)
  503 Service Unavailable (overloaded or maintenance)
  504 Gateway Timeout (proxy timeout waiting for upstream)
```

**502 vs 504**: 502 = upstream responded with garbage. 504 = upstream took too long.

### HTTP Versions

**HTTP/1.1** (1997): Persistent connections, pipelining (rarely used), chunked transfer encoding. One request per TCP connection at a time (without pipelining).

**HTTP/2** (2015): Binary framing, multiplexing (multiple requests per connection), header compression (HPACK), server push. Major performance improvement.

**HTTP/3** (2022): HTTP/2 over QUIC (UDP-based). Eliminates TCP head-of-line blocking. Faster connection setup with 0-RTT. Growing adoption.

### Headers Deep Dive

```
# Authentication
Authorization: Bearer <jwt>
Authorization: Basic <base64(user:pass)>
Authorization: AWS4-HMAC-SHA256 ...

# Content Negotiation
Content-Type: application/json; charset=utf-8
Accept: application/json, text/plain, */*
Accept-Encoding: gzip, deflate, br

# Caching
Cache-Control: max-age=3600, public
Cache-Control: no-store, no-cache, must-revalidate
ETag: "abc123"                         # version token
If-None-Match: "abc123"                # conditional GET
Last-Modified: Wed, 01 Jan 2025 00:00:00 GMT
If-Modified-Since: Wed, 01 Jan 2025 00:00:00 GMT

# Security
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY

# CORS
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Max-Age: 86400

# Request tracking
X-Request-ID: abc123-def456           # trace individual requests
X-Correlation-ID: xyz789              # trace across services
X-Forwarded-For: 1.2.3.4             # original client IP (behind proxy)
X-Real-IP: 1.2.3.4                   # original client IP (Nginx)
X-Forwarded-Proto: https             # original protocol (behind TLS terminator)

# Rate limiting (common patterns)
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 750
X-RateLimit-Reset: 1690000000
Retry-After: 60
```

---

## 3. TLS/HTTPS

### How TLS Works

TLS (Transport Layer Security) provides:
- **Encryption**: Traffic is encrypted in transit
- **Authentication**: Server proves its identity via certificate
- **Integrity**: Data cannot be tampered with

**TLS 1.3 Handshake** (simplified):

```
Client                                Server
  │                                      │
  │──── ClientHello ───────────────────►│
  │     (TLS version, cipher suites,     │
  │      client random, SNI)             │
  │                                      │
  │◄─── ServerHello ─────────────────────│
  │◄─── Certificate ─────────────────────│  (server's public cert)
  │◄─── CertificateVerify ───────────────│  (signature proving key possession)
  │◄─── Finished ────────────────────────│
  │                                      │
  │   [Client validates certificate]     │
  │   [Both derive session keys]         │
  │                                      │
  │──── Finished ──────────────────────►│
  │                                      │
  │◄═══════ Encrypted Application Data ═►│
```

**SNI (Server Name Indication)**: Sent in ClientHello, tells the server which certificate to present. This is how one IP can serve multiple HTTPS domains (virtual hosting).

### Certificates

A certificate contains:
- **Subject**: Who this certificate is for (domain name)
- **Issuer**: Who signed it (Certificate Authority)
- **Public key**: The server's public key
- **Validity period**: Not Before, Not After
- **Subject Alternative Names (SANs)**: Additional domains/IPs covered
- **Signature**: CA's digital signature validating the cert

**Certificate chain**:
```
Root CA (trusted by browsers/OS)
  └── Intermediate CA (signed by Root CA)
        └── Server Certificate (signed by Intermediate CA)
```

**DV vs OV vs EV**:
- Domain Validation (DV): Just proves domain control. Instant. Let's Encrypt.
- Organization Validation (OV): Validates organization exists. Moderate vetting.
- Extended Validation (EV): Rigorous org vetting. Green bar in old browsers.

### Certificate Management

```bash
# View certificate
openssl x509 -in cert.pem -noout -text
openssl x509 -in cert.pem -noout -dates
openssl x509 -in cert.pem -noout -subject -issuer

# Check cert on live server
openssl s_client -connect example.com:443 -servername example.com
echo | openssl s_client -connect host:443 2>/dev/null | openssl x509 -noout -dates

# Check cert expiry in script
EXPIRY=$(echo | openssl s_client -connect "$HOST:443" -servername "$HOST" 2>/dev/null \
  | openssl x509 -noout -enddate 2>/dev/null | cut -d= -f2)
echo "Expires: $EXPIRY"

# Generate self-signed cert
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

# Generate CSR for CA signing
openssl req -newkey rsa:2048 -keyout server.key -out server.csr -nodes

# Verify cert chain
openssl verify -CAfile chain.pem server.crt

# Check if private key matches certificate
openssl rsa -noout -modulus -in server.key | md5sum
openssl x509 -noout -modulus -in server.crt | md5sum
# They must match!
```

### Let's Encrypt / ACME

```bash
# Install certbot
apt install certbot python3-certbot-nginx

# Issue cert (Nginx plugin)
certbot --nginx -d example.com -d www.example.com

# Issue cert (standalone — temporarily starts its own server)
certbot certonly --standalone -d example.com

# DNS challenge (for wildcard certs)
certbot certonly --manual --preferred-challenges dns -d "*.example.com"

# Auto-renewal (certbot installs a systemd timer)
systemctl status certbot.timer

# Certificate storage
ls /etc/letsencrypt/live/example.com/
# cert.pem      — server certificate
# chain.pem     — CA intermediate chain
# fullchain.pem — cert.pem + chain.pem (use this in Nginx)
# privkey.pem   — private key
```

---

## 4. Real-World HTTP Patterns

### Nginx HTTP Configuration

```nginx
# /etc/nginx/sites-available/myapp

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    # TLS
    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;

    # Security headers
    add_header Strict-Transport-Security "max-age=63072000" always;
    add_header X-Content-Type-Options    "nosniff" always;
    add_header X-Frame-Options           "DENY" always;

    # Proxy to application
    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host             $host;
        proxy_set_header X-Real-IP        $remote_addr;
        proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeouts
        proxy_connect_timeout 5s;
        proxy_send_timeout    60s;
        proxy_read_timeout    60s;

        # Buffering
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
    }

    # Health check endpoint
    location /health {
        access_log off;
        return 200 "OK\n";
        add_header Content-Type text/plain;
    }

    # Static files with caching
    location /static/ {
        root /var/www/myapp;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### curl for Production Debugging

```bash
# Full verbose request/response
curl -v https://api.example.com/health

# Timing breakdown
curl -w "\nDNS:%{time_namelookup}s Connect:%{time_connect}s TLS:%{time_appconnect}s TTFB:%{time_starttransfer}s Total:%{time_total}s\n" \
  -o /dev/null -s https://example.com

# Follow redirects
curl -L https://example.com

# Send JSON
curl -X POST https://api.example.com/deploy \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"env": "production", "version": "1.2.3"}'

# Test with specific TLS version
curl --tlsv1.3 https://example.com

# Test with client certificate
curl --cert client.pem --key client.key https://api.example.com

# Test without cert validation (debugging only, never production)
curl -k https://self-signed.example.com

# Send headers only (HEAD request)
curl -I https://example.com

# Test HTTP/2
curl --http2 -v https://example.com

# Save response with headers
curl -D headers.txt -o response.json https://api.example.com
```

---

## 5. Architecture & Production Perspective

### HTTPS Termination

In production, HTTPS is typically terminated at:
1. **Load balancer** (ALB, NLB, GCP LB): Cert managed by AWS ACM / GCP Certificate Manager. Traffic to backends is HTTP (within VPC — trusted network).
2. **Ingress controller** (Nginx Ingress, Traefik): cert-manager + Let's Encrypt for Kubernetes.
3. **Service mesh** (Istio, Linkerd): mTLS between all services, cert termination at sidecar.

### mTLS (Mutual TLS)

In regular TLS, only the server proves its identity. In mTLS, **both** client and server present certificates. Used for:
- Service-to-service authentication in microservices
- API security (banking, healthcare)
- VPN alternatives

```nginx
# Nginx mTLS config
ssl_client_certificate /etc/ssl/ca.pem;
ssl_verify_client on;
# Now Nginx rejects requests without a valid client cert
```

### HTTP Health Checks

Every production service needs a health endpoint:

```
/health or /healthz  — liveness: is the process alive?
/ready or /readyz    — readiness: is it ready to serve traffic?
/metrics             — Prometheus metrics
```

Kubernetes uses these:
- **Liveness probe**: Fails → container restart
- **Readiness probe**: Fails → removed from service endpoints (no traffic)
- **Startup probe**: For slow-starting apps — disables liveness until started

---

## 6. Debugging & Troubleshooting

```
HTTP request fails
│
├── DNS? → dig api.example.com
├── TCP connect? → nc -zv api.example.com 443
├── TLS? → openssl s_client -connect api.example.com:443
│     → Check cert expiry, chain, CN/SAN match
├── HTTP response? → curl -v https://api.example.com
│     → 502? → Upstream unreachable or broken
│     → 503? → Upstream overloaded
│     → 504? → Upstream timeout
│     → 401? → Auth token missing/invalid/expired
│     → 403? → Insufficient permissions
│     → 429? → Rate limited, need backoff
└── CORS? → Check Access-Control-Allow-Origin header
            → Run OPTIONS preflight manually
```

---

## 7. Best Practices

- **Always use HTTPS** — even internally. mTLS in service meshes.
- **Set appropriate timeouts** on all HTTP clients — connect timeout (5s) and read timeout (30-60s).
- **Use exponential backoff with jitter** for retries — never retry 429 without respecting `Retry-After`.
- **Include `X-Request-ID`** in every request for distributed tracing.
- **Monitor certificate expiry** — set alerts at 30 days. Use automation (cert-manager, Let's Encrypt).
- **Use HTTP/2** wherever possible — massive performance improvement for multiple concurrent requests.
- **Respect `Cache-Control` headers** — proper caching reduces load dramatically.
- **Log 4xx and 5xx separately** — 4xx is client behavior, 5xx is your problem.

---

## 8. Common Mistakes

- **Not handling TLS certificate expiry** — services fail at midnight and you have an outage.
- **Trusting `X-Forwarded-For` from clients** without ensuring only your load balancer can set it.
- **Not setting HTTP timeouts** — connections hang indefinitely, exhausting thread pools.
- **Using HTTP for internal traffic** thinking the internal network is safe — it isn't.
- **Ignoring HTTP/2** — significant performance left on the table.
- **302 vs 301** — 301 is cached by browsers forever (hard to undo). Use 302 for temporary redirects.

---

## 9. Interview-Level Knowledge

- What is the difference between 401 and 403?
- How does TLS certificate validation work step by step?
- What is SNI and why is it needed?
- Explain CORS and the preflight request.
- What is HSTS and how does preloading work?
- What is the difference between `Cache-Control: no-cache` and `no-store`?
- How does HTTP/2 multiplexing improve performance over HTTP/1.1?

---

---

# PART 7: APIs

---

## 1. Overview

APIs (Application Programming Interfaces) are how services communicate. In DevOps, you interact with APIs constantly: the Kubernetes API server, AWS APIs, GitHub API, monitoring systems. You also build internal tools that expose or consume APIs. Understanding API design, authentication patterns, pagination, error handling, and versioning is essential.

---

## 2. Core Concepts

### REST API Design

REST (Representational State Transfer) is an architectural style, not a protocol. RESTful APIs:
- Use HTTP methods semantically (GET, POST, PUT, PATCH, DELETE)
- Use URIs to identify resources
- Are stateless — every request has all the info needed
- Return standard status codes

```
Resource-based URLs (nouns, not verbs):
  GET    /api/v1/deployments           → list all deployments
  POST   /api/v1/deployments           → create deployment
  GET    /api/v1/deployments/{id}      → get specific deployment
  PUT    /api/v1/deployments/{id}      → replace deployment
  PATCH  /api/v1/deployments/{id}      → partial update
  DELETE /api/v1/deployments/{id}      → delete deployment

  GET    /api/v1/deployments/{id}/pods → list pods for deployment

Wrong (RPC-style, not RESTful):
  POST /api/v1/createDeployment
  GET  /api/v1/getDeploymentById
  POST /api/v1/deleteDeployment
```

### API Authentication

**API Keys**: Simple token in header or query string. Easy but no expiry or granularity.

```bash
curl -H "X-API-Key: your-api-key" https://api.example.com/data
curl "https://api.example.com/data?api_key=xxx"  # less secure
```

**HTTP Basic Auth**: base64(username:password) in Authorization header. Only over HTTPS.

```bash
curl -u username:password https://api.example.com
# Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

**Bearer Token / JWT**: Signed token carrying claims. No session state on server.

```bash
curl -H "Authorization: Bearer eyJhbGciOiJSUzI1NiJ9..." https://api.example.com
```

**OAuth 2.0**: Authorization framework for delegated access. Flows:
- **Client Credentials**: Machine-to-machine (M2M). Service authenticates with client ID + secret, gets access token.
- **Authorization Code**: User-facing. User logs in, app gets code, exchanges for token.
- **Device Flow**: For devices with limited input.

```
Client Credentials Flow:
Client ──POST /token──► Auth Server
  client_id=xxx
  client_secret=yyy
  grant_type=client_credentials

Auth Server ──► Client: access_token (JWT), expires_in

Client ──API Request with Bearer token──► Resource Server
```

**HMAC Signatures**: Request is signed with a shared secret. Used by AWS, Stripe, GitHub webhooks.

```python
import hmac, hashlib, base64

def sign_request(body: bytes, secret: str) -> str:
    signature = hmac.new(secret.encode(), body, hashlib.sha256).digest()
    return base64.b64encode(signature).decode()

# Verify webhook from GitHub
def verify_github_webhook(payload: bytes, signature: str, secret: str) -> bool:
    expected = "sha256=" + hmac.new(secret.encode(), payload, hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected, signature)
```

### Rate Limiting

APIs limit requests to prevent abuse. Common headers:

```
X-RateLimit-Limit: 5000        # max requests per window
X-RateLimit-Remaining: 4999    # remaining in window
X-RateLimit-Reset: 1690000000  # Unix timestamp when window resets
Retry-After: 60                # seconds to wait (on 429)
```

Handling rate limits in Python:

```python
import time

def api_call_with_retry(url, headers, max_retries=5):
    for attempt in range(max_retries):
        response = requests.get(url, headers=headers)
        
        if response.status_code == 429:
            retry_after = int(response.headers.get('Retry-After', 60))
            print(f"Rate limited. Waiting {retry_after}s...")
            time.sleep(retry_after)
            continue
        
        response.raise_for_status()
        return response.json()
    
    raise Exception("Max retries exceeded")
```

### Pagination

APIs don't return millions of records at once. Common patterns:

**Offset pagination**:
```
GET /api/events?page=2&per_page=100
GET /api/events?offset=200&limit=100
```

**Cursor pagination** (preferred for large datasets):
```
GET /api/events?cursor=eyJpZCI6MTAwfQ==&limit=100
Response: {"data": [...], "next_cursor": "eyJpZCI6MjAwfQ==", "has_more": true}
```

Cursor pagination is stable — offset pagination breaks when items are added/removed.

**Link header pagination** (GitHub uses this):
```
Link: <https://api.github.com/repos/org/repo/commits?page=2>; rel="next",
      <https://api.github.com/repos/org/repo/commits?page=10>; rel="last"
```

```python
def paginate(url, headers):
    """Collect all pages of a GitHub-style paginated API."""
    results = []
    while url:
        response = requests.get(url, headers=headers)
        response.raise_for_status()
        results.extend(response.json())
        
        # Parse Link header
        link = response.headers.get('Link', '')
        next_url = None
        for part in link.split(','):
            if 'rel="next"' in part:
                next_url = part.split(';')[0].strip().strip('<>')
        url = next_url
    
    return results
```

---

## 3. Real-World API Interaction

### Kubernetes API

Everything in Kubernetes is an API call. `kubectl` is just a Kubernetes API client.

```bash
# API server URL
kubectl cluster-info

# Direct API call
APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
TOKEN=$(kubectl get secret -n kube-system $(kubectl get sa default -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' | base64 -d)

curl -H "Authorization: Bearer $TOKEN" --cacert /etc/kubernetes/pki/ca.crt \
  "$APISERVER/api/v1/namespaces/default/pods"

# From within a pod (service account auto-mounted)
curl -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
  --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
  https://kubernetes.default.svc/api/v1/namespaces/default/pods
```

### GitHub API

```bash
# List repos
curl -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/orgs/myorg/repos

# Create issue
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title": "Bug: service down", "body": "Error in production", "labels": ["bug","production"]}' \
  https://api.github.com/repos/org/repo/issues

# Get workflow runs
curl -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/org/repo/actions/runs?status=failure&per_page=5"
```

### AWS SDK / CLI

```bash
# AWS CLI is just HTTP calls to AWS APIs
aws ec2 describe-instances --filters "Name=tag:Environment,Values=production"

# With pagination
aws ec2 describe-instances --output json --query 'Reservations[*].Instances[*].[InstanceId,State.Name,Tags[?Key==`Name`].Value|[0]]'

# Paginate in CLI
aws s3api list-objects-v2 --bucket mybucket --query 'Contents[*].Key' --output text
```

---

## 4. Architecture & Production Perspective

### API Versioning

```
URL versioning (most common):
  /api/v1/resources
  /api/v2/resources

Header versioning:
  Accept: application/vnd.myapi.v2+json
  API-Version: 2024-01-01

Query string:
  /api/resources?version=2
```

Kubernetes uses URL versioning with stability levels:
- `v1` — stable
- `v1beta1` — beta
- `v1alpha1` — alpha

### API Gateway Pattern

In microservices, an API Gateway sits in front:
```
Client
  └─► API Gateway (authentication, rate limiting, routing, SSL termination)
          ├─► /api/users  → User Service
          ├─► /api/orders → Order Service
          └─► /api/items  → Inventory Service
```

AWS API Gateway, Kong, Nginx, Traefik, Envoy all serve this role.

### OpenAPI / Swagger

API specification standard. Document once, generate clients and tests.

```yaml
# openapi.yaml
openapi: 3.0.0
info:
  title: Deploy API
  version: 1.0.0
paths:
  /deployments:
    post:
      summary: Create deployment
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/DeploymentRequest'
      responses:
        '201':
          description: Deployment created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Deployment'
```

---

## 5. Debugging & Troubleshooting

```bash
# Debug an API response completely
curl -v -w "\n\nHTTP Status: %{http_code}\nTime: %{time_total}s\n" \
  -H "Authorization: Bearer $TOKEN" \
  https://api.example.com/resource

# Check if token is valid JWT
echo $TOKEN | cut -d. -f2 | base64 -d 2>/dev/null | python3 -m json.tool

# Test CORS
curl -H "Origin: https://app.example.com" \
     -H "Access-Control-Request-Method: POST" \
     -H "Access-Control-Request-Headers: Authorization" \
     -X OPTIONS \
     https://api.example.com/resource

# Test rate limits
for i in {1..10}; do
    curl -s -o /dev/null -w "%{http_code}\n" https://api.example.com/test
done
```

---

## 6. Best Practices

- **Always version your APIs**. Breaking changes in unversioned APIs are silent disasters.
- **Return proper status codes** — don't return 200 for errors.
- **Use consistent error response format**:
  ```json
  {"error": "validation_failed", "message": "Field 'name' is required", "request_id": "abc123"}
  ```
- **Include request IDs** for tracing across systems.
- **Implement idempotency keys** for non-idempotent operations that might be retried.
- **Document with OpenAPI** — it's the contract between services.
- **Rate limit from the start** — not as an afterthought.
- **Validate all input on the server** — never trust client data.

---

## 7. Interview-Level Knowledge

- What is the difference between REST and RPC?
- How does OAuth 2.0 Client Credentials flow work?
- Why is cursor pagination preferred over offset pagination for large datasets?
- What is an idempotency key and when do you need one?
- How do you handle token refresh automatically in an HTTP client?
- What is gRPC and when would you use it over REST?

---

---

# PART 8: YAML / JSON

---

## 1. Overview

YAML and JSON are the **lingua franca of configuration and data exchange** in DevOps. Kubernetes manifests, Docker Compose, Ansible playbooks, Helm charts, GitHub Actions workflows, CI configs — all YAML. API requests, responses, Terraform state, AWS CloudFormation — JSON. You must be able to read, write, validate, and programmatically process both formats with complete fluency.

---

## 2. JSON

### Structure

JSON has six value types: string, number, boolean, null, object ({}), array ([]).

```json
{
  "name": "production-cluster",
  "version": 3,
  "enabled": true,
  "metadata": null,
  "tags": ["kubernetes", "aws", "production"],
  "config": {
    "replicas": 3,
    "resources": {
      "cpu": "500m",
      "memory": "512Mi"
    }
  },
  "environments": [
    {"name": "us-east-1", "primary": true},
    {"name": "eu-west-1", "primary": false}
  ]
}
```

### Rules

- Keys must be **double-quoted strings**
- Strings must be **double-quoted**
- No trailing commas
- No comments
- Numbers: integers or floats, no quotes
- Booleans: lowercase `true`, `false`
- Null: lowercase `null`

### jq — JSON Processor

`jq` is the essential tool for processing JSON in shell scripts:

```bash
# Basic queries
echo '{"name": "prod", "replicas": 3}' | jq '.name'          # "prod"
echo '{"name": "prod", "replicas": 3}' | jq -r '.name'       # prod (raw, no quotes)
echo '{"items": [1,2,3]}' | jq '.items[]'                     # iterate array
echo '{"items": [1,2,3]}' | jq '.items[0]'                   # first element

# Real examples
kubectl get pods -o json | jq '.items[].metadata.name'
kubectl get pods -o json | jq '.items[] | {name: .metadata.name, status: .status.phase}'
kubectl get nodes -o json | jq '.items[] | select(.status.conditions[] | select(.type=="Ready" and .status=="True")) | .metadata.name'

# Transform
cat data.json | jq '[.items[] | {id: .id, name: .metadata.name}]'

# Filter
cat instances.json | jq '.Reservations[].Instances[] | select(.State.Name == "running") | .InstanceId'

# Count
kubectl get pods -o json | jq '.items | length'

# Conditional
echo '{"status": 200}' | jq 'if .status == 200 then "ok" else "error" end'

# Update/add
echo '{"a": 1}' | jq '. + {"b": 2}'           # {"a": 1, "b": 2}
echo '{"a": 1}' | jq '.a = 42'                 # {"a": 42}
echo '{"a": 1}' | jq 'del(.a)'                 # {}

# Combine with shell
REPLICAS=$(kubectl get deployment myapp -o json | jq '.spec.replicas')
echo "Current replicas: $REPLICAS"
```

---

## 3. YAML

### Structure

YAML is a superset of JSON — all valid JSON is valid YAML. YAML uses indentation (spaces only, no tabs) for structure.

```yaml
# This is a comment

# Scalar types
name: production          # string (no quotes needed unless special chars)
replicas: 3               # integer
version: 1.5              # float
enabled: true             # boolean
value: null               # null
quoted: "hello world"     # explicit string quote
multiword: 'it''s here'  # single quotes, escape with ''

# Multi-line strings
literal: |                # literal block: preserves newlines
  Line one
  Line two
  Line three

folded: >                 # folded block: newlines become spaces
  This is a long
  single line string
  
# Sequences (lists)
servers:
  - web1
  - web2
  - db1

# Or inline
servers: [web1, web2, db1]

# Mappings (dictionaries)
database:
  host: localhost
  port: 5432
  name: mydb

# Or inline
database: {host: localhost, port: 5432}

# Nested
services:
  - name: api
    port: 8080
    replicas: 3
    env:
      - name: LOG_LEVEL
        value: INFO
      - name: DB_HOST
        valueFrom:
          secretKeyRef:
            name: db-secret
            key: host

# Anchors and aliases (YAML-only feature)
defaults: &defaults
  replicas: 3
  resources:
    cpu: 500m
    memory: 512Mi

staging:
  <<: *defaults           # merge anchor
  environment: staging

production:
  <<: *defaults           # merge anchor
  replicas: 10            # override
  environment: production
```

### YAML Gotchas

```yaml
# Strings that look like other types
version: "3.8"     # must quote version numbers or YAML parses as float
port: "8080"       # quote if you want string, not number
enabled: "true"    # quote if you want string, not boolean
country: "NO"      # "NO" without quotes is boolean false in YAML 1.1!
yes: "yes"         # "yes" without quotes is boolean true!

# The Norway Problem (YAML 1.1 vs 1.2)
# In YAML 1.1 (used by many tools including older PyYAML):
# yes, no, true, false, on, off → booleans
# In YAML 1.2: only true/false are booleans

# Colons in strings — must quote
description: "host: localhost"   # colon in value needs quotes
url: "http://example.com"        # colon after protocol needs quotes

# Indentation matters — spaces only
services:
  api:         # 2 spaces
    port: 8080 # 4 spaces
```

### Processing YAML

**Python with PyYAML**:

```python
import yaml

# Load YAML (safe_load prevents code execution from malicious YAML)
with open('config.yaml') as f:
    config = yaml.safe_load(f)

# Load multiple documents
with open('k8s-manifests.yaml') as f:
    docs = list(yaml.safe_load_all(f))   # --- separated docs

# Dump YAML
data = {"name": "production", "replicas": 3}
print(yaml.dump(data, default_flow_style=False, sort_keys=False))

# Write to file
with open('output.yaml', 'w') as f:
    yaml.dump(data, f, default_flow_style=False)
```

**yq — YAML Processor** (like jq but for YAML):

```bash
# Install
snap install yq    # or brew install yq

# Read values
yq '.metadata.name' deployment.yaml

# Update in-place
yq -i '.spec.replicas = 5' deployment.yaml

# Multi-document YAML
yq '.metadata.name' *.yaml

# Convert YAML to JSON
yq -o json config.yaml

# Convert JSON to YAML
yq -P config.json

# Merge files
yq '. * load("overrides.yaml")' base.yaml
```

---

## 4. Kubernetes YAML

The most common YAML you'll write in DevOps:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
  labels:
    app: myapp
    version: "1.2.3"
  annotations:
    deployment.kubernetes.io/revision: "5"
    prometheus.io/scrape: "true"
    prometheus.io/port: "9090"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: myapp
        version: "1.2.3"
    spec:
      serviceAccountName: myapp-sa
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
      containers:
        - name: myapp
          image: registry.example.com/myapp:1.2.3
          ports:
            - containerPort: 8080
              protocol: TCP
          env:
            - name: LOG_LEVEL
              value: INFO
            - name: DB_HOST
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: host
          envFrom:
            - configMapRef:
                name: myapp-config
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /readyz
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
          volumeMounts:
            - name: config-vol
              mountPath: /etc/myapp
              readOnly: true
      volumes:
        - name: config-vol
          configMap:
            name: myapp-config
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: myapp
```

---

## 5. Validation Tools

```bash
# JSON validation
echo '{"a": 1}' | python3 -m json.tool   # pretty print and validate
jq empty file.json && echo "Valid"        # returns exit code 0 if valid

# YAML validation
python3 -c "import yaml,sys; yaml.safe_load(sys.stdin)" < config.yaml

# Kubernetes manifest validation
kubectl apply --dry-run=client -f deployment.yaml
kubectl apply --dry-run=server -f deployment.yaml   # validates against real API
kubeval deployment.yaml                              # offline schema validation
kubeconform deployment.yaml                         # faster alternative to kubeval

# JSON Schema validation
pip install jsonschema
python3 -c "
import json, jsonschema
schema = json.load(open('schema.json'))
data = json.load(open('data.json'))
jsonschema.validate(data, schema)
print('Valid!')
"
```

---

## 6. Best Practices

- **Always use `yaml.safe_load()`** in Python — never `yaml.load()` without Loader.
- **Quote strings that look like booleans or numbers** in YAML (especially `"true"`, `"yes"`, `"no"`, `"null"`, version numbers).
- **Use multi-document YAML** (`---` separator) for Kubernetes manifests when grouping related resources.
- **Validate before applying** — use dry-run or kubeval in CI.
- **Don't use YAML anchors for secrets** — don't accidentally share secret values across environments.
- **Use `jq` and `yq` in scripts** instead of parsing with `grep`/`sed`.
- **Format JSON** with `jq .` for readability when debugging.

---

## 7. Common Mistakes

- **Tabs in YAML** — YAML strictly forbids tabs for indentation. Your editor may insert them silently.
- **The Norway Problem** — unquoted `NO` being parsed as boolean `false`. Always quote country codes.
- **Forgetting `---` separator** for multi-document YAML files.
- **Integer vs string confusion** — `port: 8080` is int, `port: "8080"` is string. Some tools care.
- **YAML anchors across files** — anchors only work within a single file.
- **JSON trailing commas** — invalid JSON, but some "JSON5" tools allow it. Pure JSON parsers reject it.

---

## 8. Senior Engineer Mental Models

**"YAML is configuration, not code — but treat it like code."** Version control it, review it, lint it, test it.

**"jq is the awk of JSON."** Learn it deeply. It pays dividends every single day in DevOps work.

**"Validate early, fail fast."** A bad YAML config that makes it to production is far worse than one caught in CI. Run `kubeval` or `--dry-run` in every pipeline.

**"YAML's implicit typing is a footgun."** When in doubt, quote it.

---

## 9. Interview-Level Knowledge

- What is the difference between YAML block style and flow style?
- What is a YAML anchor and merge key?
- How does jq handle filtering vs transformation?
- Why is `yaml.safe_load()` important?
- What is JSON Schema and how does it relate to OpenAPI?
- Explain the difference between `|` and `>` in YAML multi-line strings.

---

---

# FOUNDATIONS — COMPLETE SUMMARY

Here's how all eight foundation topics connect in your daily DevOps reality:

```
A developer pushes code to GitHub [Git]
  │
  └─► GitHub webhook fires HTTP POST [HTTP/HTTPS, APIs]
        │
        └─► CI runner receives it — a Linux process [Linux]
              │
              └─► Bash script orchestrates the pipeline [Bash]
                    │
                    ├─► Python script validates config files [Python]
                    │     └─► Reads YAML/JSON manifests [YAML/JSON]
                    │
                    ├─► Docker build + push
                    │
                    └─► kubectl apply — HTTP to K8s API [Networking, APIs, HTTP]
                          └─► Resources created in cluster [Linux, Networking]
                                └─► Pods communicate via DNS [Networking]
                                └─► Services expose HTTPS endpoints [HTTP/HTTPS]
```

Every layer reinforces the others. Linux is the foundation. Bash scripts the glue. Python handles complexity. Git tracks everything. Networking moves the bits. HTTP is the protocol. APIs are the interface. YAML/JSON is the language of configuration.

Master these eight topics and you have the substrate for everything else — Docker, Kubernetes, Terraform, CI/CD, observability — all of it sits on top of this foundation.