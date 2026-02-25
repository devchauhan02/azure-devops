# 🐧 Complete Linux Guide — In Depth

> Covers: What is Linux → Linux Family → Installation → Shell → Directories → Files & Links → Users & Groups → Permissions → Processes → Memory → CPU → Systemd → Journalctl → Software Management → Network Configuration → SSH → VirtualBox Networking → Linux Networking → DHCP → IP Routing → Troubleshooting → Firewall & NAT
> Every section includes **deep explanations**, **real commands**, **cross-questions**, and **answers**.

---

## Table of Contents

1. [What is Linux](#1-what-is-linux)
2. [Linux Family](#2-linux-family)
3. [CentOS Installation and Basic Configuration](#3-centos-installation-and-basic-configuration)
4. [Basic Shell Usage](#4-basic-shell-usage)
5. [Linux Directories Layout](#5-linux-directories-layout)
6. [Working with Files and Links](#6-working-with-files-and-links)
7. [Linux Users and Groups](#7-linux-users-and-groups)
8. [File Permissions](#8-file-permissions)
9. [Processes Hierarchy](#9-processes-hierarchy)
10. [Memory Diagnostics](#10-memory-diagnostics)
11. [CPU Diagnostics](#11-cpu-diagnostics)
12. [Systemd](#12-systemd)
13. [Using Journalctl](#13-using-journalctl)
14. [Software Management](#14-software-management)
15. [Network Configuration](#15-network-configuration)
16. [SSH Overview and SSH Clients](#16-ssh-overview-and-ssh-clients)
17. [VirtualBox Networking](#17-virtualbox-networking)
18. [Basics of Linux Networking](#18-basics-of-linux-networking)
19. [DHCP in Linux Networking](#19-dhcp-in-linux-networking)
20. [Linux IP Routing](#20-linux-ip-routing)
21. [Linux Network Troubleshooting](#21-linux-network-troubleshooting)
22. [Linux Firewall and NAT](#22-linux-firewall-and-nat)

---

## 1. What is Linux

### Overview

**Linux** is a free, open-source, Unix-like **operating system kernel** created by **Linus Torvalds** in **1991** while he was a student at the University of Helsinki. He released it under the **GNU General Public License (GPL)**, making it freely modifiable and distributable.

The complete operating system is more precisely called **GNU/Linux** — the Linux kernel + GNU tools (bash, gcc, glibc, coreutils) + other software. Richard Stallman's GNU Project provided most of the user-space tools; Torvalds provided the kernel.

### What is a Kernel?

```
+--------------------------+
|   User Applications      |  ← Word processors, browsers, scripts
+--------------------------+
|   System Libraries       |  ← glibc, libssl
+--------------------------+
|      Kernel              |  ← Process/memory/device/filesystem management
+--------------------------+
|      Hardware            |  ← CPU, RAM, Disk, NIC
+--------------------------+
```

The **kernel** is the core of the OS — it manages:
- **Process scheduling** — which process gets CPU time
- **Memory management** — RAM allocation, virtual memory, paging
- **Device drivers** — communicating with hardware
- **File systems** — reading/writing to storage
- **Network stack** — TCP/IP implementation
- **System calls** — interface between user apps and kernel

### Key Characteristics of Linux

| Feature | Description |
|---------|-------------|
| **Open Source** | Source code freely available, modifiable, redistributable |
| **Multi-user** | Multiple users can use the system simultaneously |
| **Multi-tasking** | Runs many processes concurrently |
| **Multi-platform** | Runs on x86, ARM, RISC-V, MIPS, PowerPC, mainframes |
| **Security** | User-based permissions, SELinux, AppArmor, namespaces |
| **Stability** | Servers run for years without rebooting |
| **Free** | No licensing cost (though support may be paid) |
| **Shell** | Powerful command-line interface (bash, zsh, fish) |

### Linux vs Windows vs macOS

| Aspect | Linux | Windows | macOS |
|--------|-------|---------|-------|
| Cost | Free | Paid | Free (hardware cost) |
| Source | Open source | Proprietary | Partially open (Darwin) |
| Market share (server) | ~96% | ~2% | ~2% |
| Package manager | Yes (apt, yum, pacman) | No (winget emerging) | Homebrew (unofficial) |
| Filesystem | ext4, xfs, btrfs | NTFS, FAT32 | APFS, HFS+ |
| Shell | bash/zsh/fish | PowerShell/cmd | zsh |

### Where Linux is Used

- **Web servers:** ~96% of the internet runs on Linux (Apache, Nginx on Linux)
- **Android:** Android kernel is based on Linux
- **Cloud:** AWS, Google Cloud, Azure all run Linux VMs predominantly
- **Supercomputers:** 100% of TOP500 supercomputers run Linux
- **Embedded systems:** Routers, smart TVs, IoT devices
- **Containers:** Docker, Kubernetes run on Linux

### Cross-Questions & Answers

**Q1: Is Linux an operating system or a kernel?**
> **A:** Strictly speaking, Linux is just the **kernel**. The complete operating system is GNU/Linux — combining the Linux kernel with GNU tools, system libraries, and other software. However, in common usage, "Linux" refers to the entire OS (Ubuntu, Fedora, CentOS, etc.).

**Q2: What is the difference between Linux and Unix?**
> **A:** Unix is a proprietary OS developed at Bell Labs in the 1960s–70s. Linux is a **Unix-like** (POSIX-compliant) OS developed independently from scratch — it follows Unix philosophy and design but shares no Unix code. Other Unix descendants include Solaris, AIX, HP-UX, and macOS (based on BSD Unix).

**Q3: What is the Linux kernel version, and how do you check it?**
> **A:** `uname -r` shows the running kernel version (e.g., `5.15.0-91-generic`). Format: `major.minor.patch-distro`. You can also check `/proc/version` for detailed info including the compiler used to build it.

**Q4: What is GPL and why does it matter for Linux?**
> **A:** The GNU General Public License (GPL) is a **copyleft** license. It means: you can use, modify, and distribute Linux, but any distributed modifications must also be open-source under GPL. This prevents companies from taking Linux, modifying it secretly, and selling it without sharing improvements. This is why Android source must be published (though proprietary drivers are allowed).

---

## 2. Linux Family

Linux distributions (**distros**) package the Linux kernel with different tools, package managers, init systems, and philosophies. There are 600+ active distros — most descend from a few major families.

### Major Distribution Families

#### Debian Family
```
Debian (1993)
├── Ubuntu (2004) — Canonical, most popular desktop Linux
│   ├── Linux Mint — Beginner-friendly Ubuntu derivative
│   ├── Pop!_OS — System76, developer/gaming focus
│   ├── elementary OS — macOS-like aesthetics
│   └── Kubuntu/Xubuntu/Lubuntu — Different desktop environments
├── Kali Linux — Penetration testing, security
├── Raspbian/Raspberry Pi OS — ARM, Raspberry Pi
└── MX Linux — Stable, lightweight
```

- **Package manager:** `apt` / `dpkg`
- **Package format:** `.deb`
- **Init system:** systemd (modern), SysV (legacy)
- **Philosophy:** Stability, massive repositories
- **Release cycle:** Debian: slow/stable; Ubuntu: 6-month/LTS

#### Red Hat Family
```
Red Hat Enterprise Linux — RHEL (commercial)
├── CentOS (free RHEL clone — EOL for 8, Stream ongoing)
├── AlmaLinux — CentOS replacement, binary compatible with RHEL
├── Rocky Linux — CentOS replacement, by CentOS founder
├── Fedora — Community, cutting edge, upstream for RHEL
└── Oracle Linux — Oracle's RHEL clone
```

- **Package manager:** `dnf` (modern), `yum` (older)
- **Package format:** `.rpm`
- **Init system:** systemd
- **Philosophy:** Enterprise stability, long support cycles
- **Support:** RHEL — 10 years; AlmaLinux/Rocky — mirrors RHEL

#### SUSE Family
```
openSUSE Leap — Community, stable
openSUSE Tumbleweed — Rolling release
SUSE Linux Enterprise (SLE) — Commercial enterprise
```

- **Package manager:** `zypper` / `rpm`
- **Package format:** `.rpm`
- **Tool:** YaST (graphical system administration)

#### Arch Family
```
Arch Linux — DIY, rolling release, bleeding edge
├── Manjaro — User-friendly Arch derivative
├── EndeavourOS — Close to Arch with easier install
└── Garuda Linux — Gaming, performance-focused
```

- **Package manager:** `pacman`
- **Package format:** `.pkg.tar.zst`
- **Philosophy:** KISS (Keep It Simple, Stupid), rolling release, user-built everything
- **AUR:** Arch User Repository — community-maintained packages

#### Other Notable Distros
- **Gentoo:** Source-based, compile-from-source, extreme customization
- **Slackware:** Oldest maintained distro (1993), minimal, traditionalist
- **Alpine Linux:** Extremely lightweight (~5MB), used in Docker containers
- **NixOS:** Declarative configuration, reproducible builds

### Choosing a Distro

| Use Case | Recommended |
|----------|-------------|
| Beginners | Ubuntu, Linux Mint |
| Enterprise Server | RHEL, AlmaLinux, Rocky |
| Security/Pen Testing | Kali Linux, Parrot OS |
| Developer Workstation | Fedora, Pop!_OS |
| Minimal/Container | Alpine Linux |
| Privacy | Tails, Whonix |
| Rolling/Latest software | Arch, Manjaro |

### Cross-Questions & Answers

**Q1: What is the difference between CentOS, AlmaLinux, and Rocky Linux?**
> **A:** All three are **binary-compatible clones of RHEL** — meaning packages built for RHEL run on them without modification. **CentOS** was the original free RHEL clone but Red Hat shifted it to CentOS Stream (a rolling preview of future RHEL) in 2020, which broke the stable-clone promise. **AlmaLinux** and **Rocky Linux** were created by the community as true stable RHEL replacements. Rocky Linux was created by Gregory Kurtzer, who also co-founded CentOS.

**Q2: What is a rolling release distribution?**
> **A:** A rolling release distro continuously updates packages — there are no major version releases or upgrades. You install once and always have the latest software. Examples: Arch, openSUSE Tumbleweed. **Pros:** Always current. **Cons:** Less stability, occasional breakage. Contrast with **point releases** (Ubuntu 22.04 → 24.04) which are versioned and more stable.

**Q3: What is the difference between `.deb` and `.rpm` packages?**
> **A:** Both are binary package formats but for different ecosystems. `.deb` (Debian format) is used by apt/dpkg on Debian/Ubuntu systems. `.rpm` (Red Hat Package Manager format) is used by dnf/yum/rpm on Red Hat/SUSE systems. They're not directly interchangeable, though tools like `alien` can convert between them (with limitations).

---

## 3. CentOS Installation and Basic Configuration

### Pre-Installation Planning

Before installing, decide:
- **Architecture:** x86_64 (standard), ARM (Raspberry Pi, servers)
- **Installation type:** Minimal (server), GUI (workstation), custom
- **Partitioning:** Scheme for `/`, `/home`, `/var`, `/boot`, swap
- **Network:** Static IP or DHCP during install

### Recommended Partition Scheme (Server)

| Mount Point | Size | Filesystem | Purpose |
|-------------|------|-----------|---------|
| `/boot` | 1 GB | ext4/xfs | Bootloader files, kernels |
| `/boot/efi` | 512 MB | FAT32 | EFI bootloader (UEFI systems) |
| `/` | 20–50 GB | xfs | Root filesystem |
| `/home` | Remaining | xfs | User data |
| `/var` | 10–20 GB | xfs | Logs, databases, spool |
| `swap` | 2× RAM (≤8GB RAM) or 1× RAM (>8GB) | swap | Virtual memory overflow |

### Installation Steps (CentOS/AlmaLinux/Rocky)

1. **Boot from ISO** (USB or optical drive)
2. **Language & Keyboard** selection
3. **Installation Destination** — select disk, configure partitions
4. **Network & Hostname** — configure NIC, set hostname
5. **Software Selection** — Minimal, Server, Workstation
6. **Root Password** — set strong root password
7. **Create User** — create a non-root admin user
8. **Begin Installation** → reboot

### Post-Installation Basic Configuration

#### Update the system
```bash
# RHEL family
dnf update -y

# Debian/Ubuntu
apt update && apt upgrade -y
```

#### Set hostname
```bash
hostnamectl set-hostname myserver.example.com
hostnamectl status
```

#### Set timezone
```bash
timedatectl list-timezones | grep Asia
timedatectl set-timezone Asia/Kolkata
timedatectl status
```

#### Configure NTP (time sync)
```bash
# Install and enable chrony
dnf install chrony -y
systemctl enable --now chronyd
chronyc tracking          # Check sync status
chronyc sources -v        # View time sources
```

#### Configure SELinux
```bash
getenforce                          # Check current mode
setenforce 0                        # Temporarily set permissive
vi /etc/selinux/config              # Permanent: SELINUX=enforcing/permissive/disabled
```

#### Configure firewall
```bash
systemctl enable --now firewalld
firewall-cmd --state
firewall-cmd --permanent --add-service=ssh
firewall-cmd --reload
```

#### SSH configuration
```bash
vi /etc/ssh/sshd_config
# Set: PermitRootLogin no
# Set: PasswordAuthentication no (after setting up key-based auth)
systemctl restart sshd
```

### Cross-Questions & Answers

**Q1: Why create a separate /var partition?**
> **A:** `/var` holds logs, databases, mail spools, and package caches — data that grows unpredictably. If `/var` fills up on a shared root partition, the entire system becomes unusable. A separate partition limits the blast radius — `/var` fills up but `/` remains healthy, so the system keeps running.

**Q2: What is SELinux and should it be disabled?**
> **A:** SELinux (Security-Enhanced Linux) is a mandatory access control (MAC) system — it enforces security policies on every process and file, independent of standard Unix permissions. It's developed by the NSA and Red Hat. **Never disable it in production** — it prevents privilege escalation and lateral movement. Set it to `permissive` temporarily for troubleshooting (it logs but doesn't block), then re-enable after fixing the issue.

**Q3: What is the difference between BIOS and UEFI boot?**
> **A:** **BIOS** (Basic Input/Output System) is the legacy firmware — boots from MBR (Master Boot Record) at start of disk, 16-bit real mode, limited to 2TB disks. **UEFI** (Unified Extensible Firmware Interface) is the modern replacement — boots from GPT (GUID Partition Table), supports >2TB disks, Secure Boot (prevents unauthorized bootloaders), faster boot, graphical interface. Linux systems should use UEFI/GPT for modern hardware.

---

## 4. Basic Shell Usage

### What is the Shell?

The **shell** is a command-line interpreter — it takes user commands, interprets them, and executes them. It's both an interactive interface and a scripting language.

**Common shells:**
- **bash** (Bourne Again Shell) — default on most Linux distros
- **zsh** (Z Shell) — default on macOS, feature-rich, used with Oh-My-Zsh
- **fish** (Friendly Interactive Shell) — beginner-friendly auto-complete
- **sh** (Bourne Shell) — POSIX standard, used in scripts for portability
- **dash** — Lightweight, used as `/bin/sh` on Ubuntu for speed

### The Shell Prompt

```bash
[user@hostname current_directory]$
[root@myserver ~]#

# $ = regular user
# # = root user
# ~ = home directory shortcut
```

### Essential Commands

#### Navigation
```bash
pwd                    # Print Working Directory — where am I?
ls                     # List directory contents
ls -la                 # Long format + hidden files
ls -lh                 # Human-readable sizes
cd /etc                # Change to /etc
cd ~                   # Go to home directory
cd -                   # Go to previous directory
cd ..                  # Go up one level
```

#### File Operations
```bash
touch file.txt         # Create empty file or update timestamp
cat file.txt           # Display file contents
more file.txt          # Page through file (forward only)
less file.txt          # Page through file (forward & backward)
head -n 20 file.txt    # First 20 lines
tail -n 20 file.txt    # Last 20 lines
tail -f /var/log/syslog  # Follow file in real-time

cp source dest         # Copy file
cp -r dir1 dir2        # Copy directory recursively
mv source dest         # Move or rename
rm file.txt            # Remove file
rm -rf directory/      # Remove directory and contents (DANGEROUS)
mkdir newdir           # Create directory
mkdir -p a/b/c         # Create nested directories
```

#### Text Processing
```bash
grep "pattern" file          # Search for pattern in file
grep -r "pattern" /etc/      # Recursive search
grep -i "pattern" file       # Case-insensitive
grep -v "pattern" file       # Invert match (show non-matching)
grep -n "pattern" file       # Show line numbers

awk '{print $1}' file        # Print first field
awk -F: '{print $1}' /etc/passwd  # Use : as delimiter, print first field

sed 's/old/new/g' file       # Replace all occurrences
sed -i 's/old/new/g' file    # In-place replacement

sort file                    # Sort lines alphabetically
sort -n file                 # Sort numerically
sort -r file                 # Reverse sort
uniq file                    # Remove duplicate adjacent lines
sort file | uniq             # Remove all duplicates

wc file                      # Word/line/character count
wc -l file                   # Count lines only

cut -d: -f1 /etc/passwd      # Cut field 1 with : delimiter
```

#### Pipes and Redirection
```bash
# Pipes — pass output of one command to another
ls -l | grep ".txt"
cat /etc/passwd | grep root | cut -d: -f1

# Redirection
command > file         # Redirect stdout to file (overwrite)
command >> file        # Append stdout to file
command 2> error.log   # Redirect stderr to file
command &> all.log     # Redirect both stdout and stderr
command < file         # Use file as stdin

# /dev/null — black hole
command 2>/dev/null    # Discard errors
command > /dev/null 2>&1  # Discard all output
```

#### Useful Utilities
```bash
echo "Hello World"           # Print text
printf "Name: %s\n" "Alice"  # Formatted output
date                         # Current date/time
uptime                       # System uptime and load
whoami                       # Current username
id                           # User ID, group ID
which bash                   # Full path of command
whereis python3              # Find binary, source, man pages
type ls                      # Is it a builtin, alias, or file?
history                      # Command history
!!                           # Repeat last command
!grep                        # Repeat last command starting with 'grep'
Ctrl+R                       # Reverse history search
```

### Shell Variables
```bash
# Set variable (no spaces around =)
NAME="Alice"
COUNT=42

# Access variable
echo $NAME
echo ${NAME}            # Explicit syntax

# Environment variables
export NAME="Alice"     # Make available to child processes
env                     # Show all environment variables
echo $PATH              # Show PATH
echo $HOME              # Home directory
echo $USER              # Current user
echo $SHELL             # Current shell

# Command substitution
TODAY=$(date +%Y-%m-%d)
FILES=$(ls /etc/*.conf | wc -l)
echo "Config files: $FILES"
```

### Basic Shell Scripting
```bash
#!/bin/bash
# Shebang line — tells OS to use bash

# Variables
NAME="World"
echo "Hello, $NAME!"

# Conditionals
if [ $USER == "root" ]; then
    echo "Running as root"
elif [ $USER == "alice" ]; then
    echo "Hello Alice"
else
    echo "Hello $USER"
fi

# Loops
for i in 1 2 3 4 5; do
    echo "Count: $i"
done

for file in /etc/*.conf; do
    echo "Config: $file"
done

# While loop
COUNT=0
while [ $COUNT -lt 5 ]; do
    echo "Count: $COUNT"
    ((COUNT++))
done

# Functions
greet() {
    echo "Hello, $1!"    # $1 = first argument
}
greet "Alice"

# Exit codes
ls /nonexistent 2>/dev/null
echo "Exit code: $?"    # $? = exit code of last command (0=success)
```

### Keyboard Shortcuts
| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Interrupt/kill current process |
| `Ctrl+Z` | Suspend process (send to background) |
| `Ctrl+D` | EOF / logout |
| `Ctrl+L` | Clear screen |
| `Ctrl+A` | Go to beginning of line |
| `Ctrl+E` | Go to end of line |
| `Ctrl+R` | Search command history |
| `Tab` | Auto-complete command or filename |
| `Tab Tab` | Show all completions |

### Cross-Questions & Answers

**Q1: What is the difference between `>` and `>>`?**
> **A:** `>` **overwrites** the file (creates new or truncates existing). `>>` **appends** to the file (adds to end without removing existing content). Example: `echo "line1" > file` creates file with "line1". `echo "line2" >> file` adds "line2" to the file.

**Q2: What does `2>&1` mean?**
> **A:** It redirects **file descriptor 2** (stderr) to wherever **file descriptor 1** (stdout) currently goes. Combined with `> file`, it sends both stdout and stderr to the same file: `command > output.log 2>&1`. The `&1` means "the target of fd 1" — not a file named "1".

**Q3: What is the difference between single quotes and double quotes in bash?**
> **A:** **Double quotes** `"..."` allow variable expansion and command substitution: `"Hello $NAME"` evaluates `$NAME`. **Single quotes** `'...'` treat everything literally — no expansion: `'Hello $NAME'` outputs literally `Hello $NAME`. Use single quotes when you don't want any expansion.

**Q4: What does `chmod +x script.sh` do, and why is `./script.sh` needed?**
> **A:** `chmod +x` adds the execute permission to the file. `./` is needed because the current directory (`.`) is intentionally not in `$PATH` — this is a security measure to prevent accidentally running malicious executables named the same as system commands. `./` explicitly says "run the file in the current directory."

---

## 5. Linux Directories Layout

Linux uses a single unified filesystem tree rooted at `/` — no drive letters like Windows. Everything is mounted under this root, including other drives and network shares.

### The Filesystem Hierarchy Standard (FHS)

```
/
├── bin        → Essential user binaries (ls, cp, cat)
├── sbin       → Essential system binaries (fdisk, iptables, mount)
├── boot       → Bootloader, kernel, initrd
├── dev        → Device files
├── etc        → System configuration files
├── home       → User home directories
│   ├── alice/
│   └── bob/
├── lib        → Shared libraries for /bin and /sbin
├── lib64      → 64-bit libraries
├── media      → Removable media mount points (USB, CD)
├── mnt        → Temporary mount points
├── opt        → Optional/third-party software
├── proc       → Virtual filesystem — kernel/process info
├── root       → Root user's home directory
├── run        → Runtime data (PIDs, sockets) since boot
├── srv        → Service data (web server files)
├── sys        → Virtual filesystem — hardware/kernel info
├── tmp        → Temporary files (cleared on boot)
├── usr        → Secondary hierarchy (user programs)
│   ├── bin/   → Non-essential user commands (vim, python3)
│   ├── sbin/  → Non-essential system admin commands
│   ├── lib/   → Libraries for /usr/bin and /usr/sbin
│   ├── local/ → Locally compiled software
│   └── share/ → Architecture-independent data
└── var        → Variable data (logs, databases, spool)
    ├── log/
    ├── cache/
    ├── spool/
    └── www/
```

### Key Directories Explained

#### /etc — Configuration
```bash
/etc/passwd          # User accounts
/etc/shadow          # Encrypted passwords
/etc/group           # Group definitions
/etc/hosts           # Static hostname resolution
/etc/hostname        # System hostname
/etc/resolv.conf     # DNS resolver config
/etc/fstab           # Filesystem mount table (auto-mount on boot)
/etc/ssh/sshd_config # SSH server configuration
/etc/crontab         # System cron jobs
/etc/sudoers         # Sudo privileges (edit with visudo)
/etc/os-release      # OS identification
/etc/services        # Port-to-service name mappings
/etc/sysctl.conf     # Kernel parameter tuning
```

#### /proc — Process Information (Virtual)
```bash
/proc/cpuinfo        # CPU details
/proc/meminfo        # Memory details
/proc/version        # Kernel version
/proc/uptime         # System uptime
/proc/net/           # Network statistics
/proc/[PID]/         # Per-process directory
/proc/[PID]/cmdline  # Command that started the process
/proc/[PID]/status   # Process status
/proc/[PID]/fd/      # Open file descriptors
/proc/sys/           # Tunable kernel parameters
```

#### /sys — System/Hardware (Virtual)
```bash
/sys/class/net/      # Network interfaces
/sys/block/          # Block devices
/sys/bus/            # Hardware buses (PCI, USB)
/sys/devices/        # Device hierarchy
```

#### /dev — Device Files
```bash
/dev/sda             # First SATA/SCSI hard disk
/dev/sda1            # First partition of sda
/dev/nvme0n1         # First NVMe SSD
/dev/tty             # Current terminal
/dev/null            # Black hole (discard output)
/dev/zero            # Infinite stream of zeros
/dev/random          # Cryptographically secure random
/dev/urandom         # Non-blocking random
/dev/loop0           # Loop device (mount image files)
```

#### /var — Variable Data
```bash
/var/log/syslog      # General system log (Debian)
/var/log/messages    # General system log (RHEL)
/var/log/auth.log    # Authentication log (Debian)
/var/log/secure      # Authentication log (RHEL)
/var/log/nginx/      # Nginx web server logs
/var/log/journal/    # Systemd journal binary logs
/var/cache/          # Application caches
/var/spool/cron/     # User crontabs
/var/www/html/       # Default web server document root
```

### Cross-Questions & Answers

**Q1: What is the difference between /bin and /usr/bin?**
> **A:** Historically, `/bin` contained binaries needed **during boot and single-user mode** before `/usr` was mounted (they could be on separate partitions). `/usr/bin` contained everything else. On modern systems, most distros have merged them — `/bin` is now a symlink to `/usr/bin`. The distinction is largely historical.

**Q2: What is /proc and is it stored on disk?**
> **A:** `/proc` is a **virtual filesystem (procfs)** — it exists entirely in RAM and represents kernel and process information as files. There's nothing on disk. When you read `/proc/cpuinfo`, the kernel generates the content on-the-fly. Writing to `/proc/sys/` files changes live kernel parameters. It's not persistent across reboots.

**Q3: Why does /tmp get cleared on boot?**
> **A:** `/tmp` is for truly temporary files — applications create temp files for current session use. On reboot, the system is starting fresh, so leftover temp files are meaningless and potentially confusing. Many systems mount `/tmp` as `tmpfs` (RAM-backed filesystem) which is cleared automatically. The policy is enforced by `systemd-tmpfiles` or init scripts.

**Q4: What is /etc/fstab and what happens if it's wrong?**
> **A:** `/etc/fstab` (filesystem table) defines which filesystems are mounted where on boot. Format: `<device> <mountpoint> <fstype> <options> <dump> <pass>`. If fstab has errors (wrong UUID, typo in mount point), the system may boot into emergency mode because critical filesystems can't be mounted. Always use `mount -a` to test fstab changes before rebooting.

---

## 6. Working with Files and Links

### File Types in Linux

Linux represents everything as files. The `ls -l` output shows file type as the first character:

| Symbol | Type | Description |
|--------|------|-------------|
| `-` | Regular file | Text, binary, scripts |
| `d` | Directory | Container for files |
| `l` | Symbolic link | Pointer to another file |
| `c` | Character device | Terminal, serial port |
| `b` | Block device | Hard disk, SSD |
| `p` | Named pipe (FIFO) | IPC mechanism |
| `s` | Socket | Network/IPC socket |

```bash
ls -la /dev/
# brw-rw---- sda (block device)
# crw--w---- tty (character device)
# lrwxrwxrwx bin -> usr/bin (symlink)
```

### Working with Files

```bash
# Create files
touch file.txt                    # Empty file
echo "content" > file.txt         # File with content
cat > file.txt << EOF             # Here document
This is line 1
This is line 2
EOF

# View files
file file.txt                     # Determine file type
stat file.txt                     # Detailed metadata (size, timestamps, inode)
xxd file.txt                      # Hex dump
strings binaryfile                # Extract printable strings from binary

# Find files
find / -name "*.log" 2>/dev/null          # Find by name
find /var -type f -size +100M             # Files > 100MB
find /etc -type f -mtime -7               # Modified in last 7 days
find /home -type f -perm /u+x             # Executable files
find / -user alice                        # Files owned by alice
find / -name "*.conf" -exec grep -l "ssh" {} \;  # Find and grep

# Locate (uses database, must run updatedb first)
updatedb                                  # Build/update database
locate sshd_config                        # Fast search

# which, whereis, type
which python3                             # /usr/bin/python3
whereis nginx                             # binary, source, man page locations
type ls                                   # alias ls='ls --color=auto'
```

### Hard Links vs Symbolic Links

This is one of the most important Linux file concepts.

#### Inodes

Every file has an **inode** — a data structure storing file metadata (size, permissions, timestamps, disk block locations). The inode number is the file's true identity. **Filenames are just labels pointing to inodes.**

```bash
ls -li                 # Shows inode numbers
stat file.txt          # Shows inode number, link count
```

#### Hard Links

A **hard link** is an additional filename pointing to the **same inode** (same data on disk).

```bash
# Create hard link
ln original.txt hardlink.txt

# Both point to same inode
ls -li original.txt hardlink.txt
# 1234567 -rw-r--r-- 2 user group 100 original.txt
# 1234567 -rw-r--r-- 2 user group 100 hardlink.txt
# Same inode (1234567), link count = 2
```

**Properties of hard links:**
- Same inode as original — same data, same permissions
- Link count increments (tracks how many names point to inode)
- If you delete one, the other remains (data persists until link count = 0)
- **Cannot span filesystems** (inodes are local to filesystem)
- **Cannot link to directories** (prevents circular structures)

#### Symbolic (Soft) Links

A **symlink** is a separate file that contains the **path** to another file — like a Windows shortcut.

```bash
# Create symlink
ln -s /etc/nginx/nginx.conf nginx.conf
ln -s /usr/local/python3.11 /usr/bin/python3

# Symlink shows arrow
ls -la
# lrwxrwxrwx 1 user group 24 nginx.conf -> /etc/nginx/nginx.conf

# Check what symlink points to
readlink nginx.conf
readlink -f nginx.conf    # Resolve all symlinks to absolute path
```

**Properties of symbolic links:**
- Different inode from target
- If target is deleted → symlink becomes **dangling** (broken)
- **Can span filesystems**
- **Can link to directories**
- Can point to relative or absolute paths

#### Hard Link vs Symlink Comparison

| Feature | Hard Link | Symbolic Link |
|---------|-----------|---------------|
| Inode | Same as original | Different (own inode) |
| Cross-filesystem | No | Yes |
| Link to directory | No | Yes |
| If original deleted | Data remains | Becomes dangling |
| `ls -l` display | Same as regular file | Shows `->` target |
| Relative paths | N/A | Supported |

### File Compression and Archiving

```bash
# tar (Tape Archive) — most common
tar -cvf archive.tar files/       # Create archive
tar -xvf archive.tar              # Extract archive
tar -tvf archive.tar              # List contents
tar -czvf archive.tar.gz files/   # Create + gzip compress
tar -xzvf archive.tar.gz          # Extract gzip
tar -cjvf archive.tar.bz2 files/  # Create + bzip2
tar -cJvf archive.tar.xz files/   # Create + xz (best compression)

# Flags: c=create, x=extract, t=list, v=verbose, f=filename
# z=gzip, j=bzip2, J=xz

# Standalone compression
gzip file.txt           # Compress → file.txt.gz (original gone)
gzip -d file.txt.gz     # Decompress
gunzip file.txt.gz      # Same as above

bzip2 file.txt          # Better compression, slower
xz file.txt             # Best compression, slowest

zip archive.zip files/  # Create zip
unzip archive.zip       # Extract zip
```

### Cross-Questions & Answers

**Q1: If I delete the original file and a hard link exists, what happens?**
> **A:** The data is **preserved**. Deleting a filename just decrements the link count on the inode. The actual data is only removed when the link count reaches 0 (no more names point to the inode) AND no process has the file open. The hard link continues to work perfectly as an independent reference to the same data.

**Q2: Why can't hard links cross filesystems?**
> **A:** Inode numbers are only meaningful within a single filesystem. An inode number 12345 on `/dev/sda1` has no meaning on `/dev/sdb1`. Since hard links reference inodes directly, and inode numbers aren't unique across filesystems, cross-filesystem hard links would be ambiguous. Symlinks work across filesystems because they reference paths (which resolve across the entire directory tree), not inode numbers.

**Q3: What is a dangling symlink?**
> **A:** A symlink whose **target no longer exists**. Example: `link.txt -> /tmp/data.txt` where `/tmp/data.txt` has been deleted. Accessing the symlink gives "No such file or directory." Find dangling symlinks: `find . -type l -! -e` (symlinks that don't resolve).

---

## 7. Linux Users and Groups

### User Model

Linux is a multi-user OS. Every process runs as a **user**, every file has an **owner**. The user system enforces access control.

### User Types

| Type | UID | Description |
|------|-----|-------------|
| **root** | 0 | Superuser, full system access |
| **System users** | 1–999 | Services (apache, mysql, nobody) |
| **Regular users** | 1000+ | Human users |

### User Database Files

#### /etc/passwd — User accounts
```
username:password:UID:GID:GECOS:home:shell
alice:x:1001:1001:Alice Smith:/home/alice:/bin/bash
```

Fields:
- **username:** Login name
- **password:** `x` means password is in `/etc/shadow`
- **UID:** User ID
- **GID:** Primary Group ID
- **GECOS:** Full name, comment info
- **home:** Home directory
- **shell:** Login shell (`/sbin/nologin` = no interactive login for system accounts)

#### /etc/shadow — Encrypted passwords
```
alice:$6$salt$hashedpassword:19000:0:99999:7:::
```

Fields: username:encrypted_password:last_change:min_age:max_age:warn:inactive:expire

- `$6$` = SHA-512 hash
- `$1$` = MD5 (old, insecure)
- `$y$` = yescrypt (modern)
- `!!` or `!` = account locked

#### /etc/group — Group definitions
```
groupname:password:GID:member1,member2
developers:x:1002:alice,bob
```

### User Management Commands

```bash
# Add users
useradd alice                         # Create user (minimal)
useradd -m -s /bin/bash -c "Alice Smith" alice  # With home, shell, comment
useradd -u 1500 -g developers alice   # Specific UID and primary group
adduser alice                         # Interactive (Debian/Ubuntu — friendlier)

# Set/change password
passwd alice                          # Set password for alice
passwd                                # Change own password
passwd -l alice                       # Lock account
passwd -u alice                       # Unlock account
passwd -e alice                       # Expire password (force change on next login)

# Modify users
usermod -s /bin/zsh alice             # Change shell
usermod -d /data/alice alice          # Change home directory
usermod -l newname alice              # Change username
usermod -aG sudo alice                # Add to sudo group (-a = append, don't remove existing)
usermod -G group1,group2 alice        # Set groups (replaces existing supplementary groups)
usermod -L alice                      # Lock account
usermod -U alice                      # Unlock account

# Delete users
userdel alice                         # Delete user (keep home)
userdel -r alice                      # Delete user AND home directory
userdel -r -f alice                   # Force delete even if logged in

# User info
id alice                              # UID, GID, groups
whoami                                # Current username
w                                     # Who is logged in and what they're doing
who                                   # Who is logged in
last                                  # Login history
lastlog                               # Last login for all users
finger alice                          # User info (if installed)
```

### Group Management Commands

```bash
# Add group
groupadd developers
groupadd -g 2000 sysadmins            # Specific GID

# Modify group
groupmod -n newname developers        # Rename group
groupmod -g 2001 developers           # Change GID

# Delete group
groupdel developers

# Add user to group
gpasswd -a alice developers           # Add alice to developers
gpasswd -d alice developers           # Remove alice from developers
gpasswd -M alice,bob developers       # Set members list

# View groups
groups alice                          # Groups alice belongs to
getent group developers               # Group entry from database
cat /etc/group | grep alice           # Manual check
```

### sudo — Superuser Do

```bash
# Run command as root
sudo apt update
sudo systemctl restart nginx

# Run command as another user
sudo -u alice /home/alice/script.sh

# Open root shell
sudo su -             # Root shell with root's environment
sudo -i               # Same
sudo su               # Root shell but keep current environment

# Edit sudoers safely
visudo                # Always use visudo — validates syntax before saving

# Common sudoers entries:
alice ALL=(ALL:ALL) ALL           # alice can run any command as any user
%sudo ALL=(ALL:ALL) ALL           # All members of sudo group
alice ALL=(ALL) NOPASSWD: ALL     # No password required (risky)
alice ALL=(ALL) /usr/bin/systemctl # Only allow specific command
```

### Cross-Questions & Answers

**Q1: What is the difference between UID 0 and the username root?**
> **A:** It's the **UID 0** that grants superuser privileges, not the name "root." You could rename the root account to anything — as long as its UID is 0, it has full privileges. Similarly, if an attacker creates a user with UID=0, they have root access even with a different name. This is why monitoring `/etc/passwd` for unauthorized UID 0 accounts is a security best practice.

**Q2: What is the difference between primary group and supplementary groups?**
> **A:** Every user has one **primary group** (stored in `/etc/passwd` GID field) — new files created by the user belong to this group by default. **Supplementary (secondary) groups** are additional group memberships (stored in `/etc/group`) that grant extra permissions. `id alice` shows both. Use `usermod -aG group alice` to add supplementary groups.

**Q3: Why use `usermod -aG` instead of `usermod -G`?**
> **A:** `usermod -G group alice` **replaces** alice's supplementary groups with only "group" — removing all existing supplementary group memberships. `usermod -aG group alice` **appends** "group" while keeping existing memberships. The `-a` flag is critical — forgetting it is a common mistake that locks users out of groups they needed.

**Q4: What is `visudo` and why must it be used instead of directly editing /etc/sudoers?**
> **A:** `visudo` opens the sudoers file in an editor but **validates syntax before saving**. A syntax error in `/etc/sudoers` can lock **all users out of sudo** — potentially rendering the system unmanageable if root login is also disabled. `visudo` catches errors and warns you, preventing this. It also applies file locking to prevent simultaneous edits.

---

## 8. File Permissions

### Permission Model

Linux uses a discretionary access control (DAC) model. Every file has:
- An **owner** (user)
- An **owning group**
- **Permission bits** for owner, group, and others

### Reading Permissions

```bash
ls -l file.txt
# -rwxr-xr-- 1 alice developers 4096 Jan 1 12:00 file.txt
# |||||||||
# |||||||||__ Others: r-- (read only)
# ||||||_____ Group: r-x (read, execute)
# |||________ Owner: rwx (read, write, execute)
# ||_________ File type: - (regular file)
```

### Permission Breakdown

| Symbol | Meaning for File | Meaning for Directory |
|--------|-----------------|----------------------|
| `r` (4) | Read file contents | List directory contents (ls) |
| `w` (2) | Modify file | Create/delete files in dir |
| `x` (1) | Execute file | Enter directory (cd), access contents |
| `-` (0) | Permission denied | Permission denied |

### Octal Notation

```
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
-wx = 0+2+1 = 3
-w- = 0+2+0 = 2
--x = 0+0+1 = 1
--- = 0+0+0 = 0

Example: rwxr-xr-- = 754
  Owner: rwx = 7
  Group: r-x = 5
  Others: r-- = 4
```

### chmod — Change Permissions

```bash
# Octal notation (absolute)
chmod 755 file         # rwxr-xr-x
chmod 644 file         # rw-r--r-- (typical for files)
chmod 700 dir          # rwx------ (private)
chmod 777 file         # rwxrwxrwx (dangerous!)

# Symbolic notation (relative)
chmod u+x file         # Add execute for user/owner
chmod g-w file         # Remove write for group
chmod o=r file         # Set others to read only
chmod a+r file         # Add read for all (a = all)
chmod u+x,g-w file     # Multiple changes
chmod +x script.sh     # Add execute for all

# Recursive
chmod -R 755 directory/
```

### chown and chgrp — Change Ownership

```bash
chown alice file.txt              # Change owner
chown alice:developers file.txt   # Change owner and group
chown :developers file.txt        # Change group only
chgrp developers file.txt         # Change group only
chown -R alice:devs project/      # Recursive
```

### Special Permission Bits

#### SUID — Set User ID (4xxx)
- **On executable:** Runs with **owner's privileges**, not the caller's
- Example: `/usr/bin/passwd` is SUID root — regular users can run it to change their password, and it temporarily gets root privileges to write to `/etc/shadow`
- Shown as `s` in owner execute position: `-rwsr-xr-x`
- If set without execute permission: `S` (uppercase = not effective)

```bash
chmod 4755 program      # Set SUID
chmod u+s program       # Same
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root ... passwd
```

#### SGID — Set Group ID (2xxx)
- **On executable:** Runs with **group's privileges**
- **On directory:** New files inherit the **directory's group** (useful for shared directories)
- Shown as `s` in group execute position: `-rwxr-sr-x`

```bash
chmod 2755 program      # Set SGID on file
chmod g+s /shared/      # Set SGID on directory
```

#### Sticky Bit (1xxx)
- **On directory:** Users can only **delete their own files**, even if they have write permission on the directory
- Essential for `/tmp` — everyone can create files, but only the owner can delete them
- Shown as `t` in others execute position: `drwxrwxrwt`

```bash
chmod 1777 /tmp         # drwxrwxrwt
chmod +t /shared/       # Set sticky bit
ls -ld /tmp
# drwxrwxrwt 12 root root ... /tmp
```

### umask — Default Permissions

**umask** defines the default permission mask for newly created files/directories. The permissions granted = default maximum MINUS umask.

```bash
umask            # Show current umask (typically 0022)
umask 0027       # Set new umask (persistent only in session)

# Default max: files=666, dirs=777
# umask 022:
#   Files:   666 - 022 = 644 (rw-r--r--)
#   Dirs:    777 - 022 = 755 (rwxr-xr-x)

# umask 027:
#   Files:   666 - 027 = 640 (rw-r-----)
#   Dirs:    777 - 027 = 750 (rwxr-x---)
```

Set persistent umask in `~/.bashrc` or `/etc/profile`.

### ACLs — Access Control Lists

Standard permissions only allow one owner and one group. **ACLs** allow fine-grained permissions for multiple users/groups.

```bash
# Check if filesystem supports ACL (check mount options: acl)
mount | grep sda

# View ACLs
getfacl file.txt

# Set ACL
setfacl -m u:bob:rw file.txt         # Give bob read-write
setfacl -m g:devs:r file.txt         # Give devs group read
setfacl -m o::- file.txt             # Remove others' access
setfacl -x u:bob file.txt            # Remove bob's ACL
setfacl -b file.txt                  # Remove all ACLs
setfacl -R -m u:bob:rx directory/    # Recursive
setfacl -d -m g:devs:rw directory/   # Default ACL for new files in dir

# A '+' appears at end of ls -l when ACL is set
ls -l file.txt
# -rw-rw-r--+ 1 alice alice 100 file.txt
```

### Cross-Questions & Answers

**Q1: Why does `/usr/bin/passwd` need SUID?**
> **A:** The `passwd` command must write to `/etc/shadow` (owned by root, permissions 640). Regular users can't write to it directly. SUID makes `passwd` run with root's effective UID — giving it permission to write shadow. The program is carefully written to only allow users to change their own password, preventing abuse of the root privilege.

**Q2: What is the risk of world-writable SUID files?**
> **A:** Extremely dangerous. A world-writable SUID file means anyone can replace the binary with malicious code, and that code runs with the file owner's (often root's) privileges. Find these with: `find / -perm -4002 -type f` (SUID + world-writable). This is a common security misconfiguration that attackers exploit for privilege escalation.

**Q3: If a directory has write permission but no execute permission, can you list files in it?**
> **A:** No. To list directory contents (`ls`), you need **read (`r`) permission**. To access files within a directory or `cd` into it, you need **execute (`x`) permission**. Write (`w`) permission allows creating/deleting files. This is why the common `644` for files but `755` for directories — directories need execute for traversal.

**Q4: What is the difference between effective UID and real UID?**
> **A:** **Real UID** is who you actually are (your logged-in user ID). **Effective UID** is what the OS uses for permission checks — usually same as real, but changes when running SUID programs. When a regular user runs `passwd` (SUID root), real UID = user, effective UID = 0 (root). The program sees `euid=0` and gets root privileges while the kernel knows the real user via `ruid`.

---

## 9. Processes Hierarchy

### What is a Process?

A **process** is a running instance of a program. It has:
- **PID** (Process ID) — unique identifier
- **PPID** (Parent Process ID) — who created it
- **UID/GID** — owner
- **Memory space** — private virtual address space
- **File descriptors** — open files
- **State** — running, sleeping, stopped, zombie

### Process Tree

Linux processes form a **tree** rooted at PID 1 (systemd or init):

```
systemd (PID 1)
├── sshd (PID 234)
│   └── sshd (PID 891) — fork for connection
│       └── bash (PID 892)
│           └── vim (PID 1234)
├── cron (PID 456)
├── nginx (PID 567)
│   ├── nginx worker (PID 568)
│   └── nginx worker (PID 569)
└── login (PID 678)
    └── bash (PID 679)
```

```bash
pstree                    # Show process tree
pstree -p                 # Include PIDs
pstree -u                 # Include usernames
```

### Process States

| State | Symbol | Meaning |
|-------|--------|---------|
| Running | R | Actively using CPU or ready to run |
| Sleeping | S | Interruptible sleep (waiting for event) |
| Deep Sleep | D | Uninterruptible sleep (I/O wait — can't be killed easily) |
| Stopped | T | Paused (Ctrl+Z or SIGSTOP) |
| Zombie | Z | Process finished but parent hasn't collected exit status |
| Idle | I | Kernel idle thread |

### Process Monitoring Commands

```bash
# ps — Process Snapshot
ps                          # Current terminal's processes
ps aux                      # All processes, all users, full info
ps aux | grep nginx         # Find nginx processes
ps -ef                      # Full format with PPID
ps -u alice                 # Processes owned by alice
ps --forest                 # Tree view

# Output columns:
# USER   PID  %CPU %MEM    VSZ   RSS TTY  STAT START   TIME COMMAND
# root     1   0.0  0.1 168088 13264 ?    Ss   10:00   0:02 /usr/lib/systemd/systemd

# top — Interactive process monitor
top
# Keys:
# q = quit
# k = kill process (enter PID)
# M = sort by memory
# P = sort by CPU
# r = renice (change priority)
# 1 = show individual CPUs
# u = filter by user

# htop — Enhanced top (install separately)
htop                        # Colorful, mouse-supported

# Specific info
cat /proc/1234/status        # Process 1234 status
cat /proc/1234/cmdline       # Command line
ls -la /proc/1234/fd/        # Open file descriptors
```

### Signals

Signals are software interrupts sent to processes.

```bash
# Send signals
kill -SIGTERM 1234           # Graceful termination (default)
kill 1234                    # Same as SIGTERM
kill -9 1234                 # SIGKILL — force kill (unignorable)
kill -SIGHUP 1234            # Reload configuration
kill -SIGSTOP 1234           # Pause process
kill -SIGCONT 1234           # Resume process
killall nginx                # Kill all processes named nginx
pkill -f "python script.py"  # Kill by full command pattern

# Common signals
# SIGTERM (15) — Graceful shutdown (can be caught/ignored)
# SIGKILL (9) — Force kill (cannot be caught or ignored)
# SIGHUP (1) — Hangup / reload config
# SIGINT (2) — Ctrl+C (interrupt)
# SIGSTOP (19) — Stop/pause (unignorable)
# SIGCONT (18) — Continue stopped process
# SIGUSR1/2 — User-defined
```

### Foreground and Background Processes

```bash
# Run in background
command &                    # Start in background
sleep 300 &                  # Runs in background
jobs                         # List background jobs
[1]+  Running   sleep 300 &

# Bring to foreground
fg                           # Bring last background job to foreground
fg %1                        # Bring job 1 to foreground

# Send to background
Ctrl+Z                       # Suspend foreground process
bg                           # Resume suspended process in background
bg %1                        # Resume job 1 in background

# nohup — keep running after logout
nohup long_process.sh &      # Won't be killed when terminal closes
nohup long_process.sh > output.log 2>&1 &

# disown — detach job from shell
long_process.sh &
disown %1                    # Shell no longer tracks it
```

### Process Priority (nice/renice)

```bash
# Nice value: -20 (highest priority) to 19 (lowest)
# Default: 0
# Only root can set negative values

nice -n 10 command           # Start with lower priority
nice -n -5 command           # Higher priority (root only)
renice 15 -p 1234            # Change running process priority
renice -n -10 -u alice       # Change all alice's processes

# Check priority
ps -o pid,ni,pri,comm        # Show nice and priority values
top                          # NI column shows nice value
```

### Zombie Processes

A **zombie** is a process that has finished executing but its entry remains in the process table because the parent hasn't called `wait()` to collect its exit status.

```bash
ps aux | grep Z              # Find zombie processes
# You cannot kill a zombie directly — kill the parent
kill -SIGCHLD <parent_PID>   # Signal parent to reap children
```

If zombies accumulate, it indicates a bug in the parent process. Killing the parent (or waiting for it to call `wait()`) resolves zombies.

### Cross-Questions & Answers

**Q1: What is the difference between `kill -9` and `kill -15`?**
> **A:** `SIGTERM (15)` is a polite request to terminate — the process can catch it, save data, release resources, and exit cleanly. `SIGKILL (9)` is delivered directly by the kernel — the process **cannot** catch, ignore, or handle it. The kernel forcibly terminates it. Always try SIGTERM first; use SIGKILL only if the process doesn't respond. SIGKILL can leave incomplete writes and corrupted files.

**Q2: What is a zombie process, and how do you remove it?**
> **A:** A zombie exists when a process exits but its parent hasn't read its exit status. The process is dead but occupies a PID entry. You **cannot** kill a zombie with any signal (it's already dead). Fix: send `SIGCHLD` to the parent, or kill the parent. If the parent is running a buggy program, fix the bug. In practice, zombies are harmless if few in number.

**Q3: What is an orphan process?**
> **A:** When a parent process dies before its children, the children become orphans. Linux automatically **re-parents** orphans to PID 1 (systemd/init), which then properly reaps them when they exit. This prevents zombie accumulation.

**Q4: What is the difference between a process and a thread?**
> **A:** A **process** has its own memory space, file descriptors, and resources — isolated from other processes. A **thread** is a lightweight execution unit within a process — threads share the same memory space and resources. Creating a thread is faster than a process (`fork`). In Linux, both are implemented as tasks (`clone()` syscall) — threads just share more with the parent than processes do.

---

## 10. Memory Diagnostics

### Linux Memory Architecture

```
Physical RAM
├── Kernel memory (non-swappable)
├── User process memory
│   ├── Code (text segment)
│   ├── Data (BSS, heap)
│   └── Stack
├── Page cache (disk cache — can be reclaimed)
├── Buffers (metadata cache — can be reclaimed)
└── Swap space (on disk — used when RAM full)
```

### Key Memory Concepts

| Term | Meaning |
|------|---------|
| **RSS** (Resident Set Size) | Physical RAM currently used by process |
| **VSZ** (Virtual Memory Size) | Total virtual address space (may not all be in RAM) |
| **Shared memory** | Memory shared between processes (libraries) |
| **Page cache** | Kernel caches disk reads in RAM for performance |
| **Swap** | Disk space used as overflow when RAM is full |
| **OOM Killer** | Kernel mechanism to kill processes when RAM exhausted |

### Memory Diagnostic Commands

```bash
# free — Memory overview
free -h                    # Human-readable (MB/GB)
free -m                    # In megabytes

#              total        used        free      shared  buff/cache   available
# Mem:          7.7G        2.1G        1.2G       234M        4.4G        5.1G
# Swap:         2.0G          0B        2.0G

# Key: "available" is the important number — what can be used without swapping
# buff/cache can be reclaimed when needed — so "free" is misleading

# /proc/meminfo — Detailed memory info
cat /proc/meminfo

# vmstat — Virtual memory statistics
vmstat 1 5                 # Report every 1 second, 5 times
vmstat -s                  # Summary

# Columns: r=runnable, b=blocked, swpd=swap used, free, buff, cache
# si=swap in, so=swap out (high = memory pressure)
# us=user, sy=system, id=idle, wa=IO wait

# top/htop
top                        # Press M to sort by memory
# VIRT=virtual, RES=resident, SHR=shared, %MEM=% of total RAM

# ps — per process
ps aux --sort=-%mem | head -10    # Top 10 memory consumers
ps -o pid,user,rss,vsz,comm      # Custom format

# smem — better memory report
smem -r -k | head -10            # Sort by PSS (proportional)

# Shared library memory
pmap -x 1234                     # Memory map of process 1234
cat /proc/1234/maps               # Raw memory maps
```

### Memory Pressure and Swapping

```bash
# Check swap usage
swapon --show                    # Show swap devices
cat /proc/swaps                  # Same

# Swappiness — kernel preference for swapping (0-100)
cat /proc/sys/vm/swappiness      # Default: 60
sysctl vm.swappiness=10          # Temporary change
echo "vm.swappiness=10" >> /etc/sysctl.conf  # Permanent

# Lower swappiness = prefer keeping data in RAM
# swappiness=0 means avoid swap as much as possible
# swappiness=1 recommended for databases

# Clear page cache (safe — kernel reclaims as needed)
sync && echo 3 > /proc/sys/vm/drop_caches
# 1=page cache, 2=dentries+inodes, 3=all

# OOM Killer logs
dmesg | grep -i "oom"
journalctl -k | grep -i "oom kill"
```

### Memory Management — Practical

```bash
# Find memory-hungry processes
ps aux --sort=-%mem | head -20

# Check if system is swapping heavily (bad sign)
vmstat 1 | awk '{print $7, $8}'   # si=swap in, so=swap out

# Huge pages (for databases)
cat /proc/sys/vm/nr_hugepages
sysctl -w vm.nr_hugepages=128

# NUMA memory info
numactl --hardware                # NUMA topology
numastat                          # NUMA statistics

# Memory bandwidth/latency testing
apt install mbw
mbw -n 5 256                      # Test 256MB

# Check virtual memory settings
sysctl -a | grep vm               # All vm parameters
```

### Cross-Questions & Answers

**Q1: What does "available" mean in `free -h` output vs "free"?**
> **A:** **Free** is RAM completely unused. **Available** is an estimate of RAM available for new processes without swapping — this includes "free" plus most of buff/cache (which can be instantly reclaimed by the kernel when needed). Always look at "available," not "free." A system with 100MB "free" but 4GB "available" has plenty of RAM — the cache is just being used efficiently.

**Q2: What is the OOM Killer and how does it decide which process to kill?**
> **A:** The **Out-of-Memory Killer** activates when the system has no free memory and can't allocate more. It scores each process with an **OOM score** (0–1000) based on memory usage, runtime, child processes, and other factors. The process with the highest score is killed. You can view and adjust scores: `cat /proc/PID/oom_score`. Set `oom_score_adj` (-1000 to 1000) to protect important processes: `echo -1000 > /proc/PID/oom_score_adj`.

**Q3: What is the difference between VSZ and RSS?**
> **A:** **VSZ (Virtual Size)** is the total virtual address space allocated — includes shared libraries mapped in, memory-mapped files, and pages not yet loaded into RAM. **RSS (Resident Set Size)** is the actual physical RAM currently in use. VSZ is always ≥ RSS. High VSZ with low RSS is normal (virtual memory is just reserved but not yet committed). RSS is what matters for actual RAM pressure.

---

## 11. CPU Diagnostics

### CPU Architecture Basics

```bash
# View CPU info
lscpu                          # CPU architecture info
cat /proc/cpuinfo              # Raw CPU info (all cores)
nproc                          # Number of processing units

# lscpu output includes:
# Architecture, Thread(s) per core, Core(s) per socket, Socket(s)
# CPU MHz, Cache sizes, Virtualization support
```

### Load Average

**Load average** represents the average number of processes in the run queue over 1, 5, and 15 minutes.

```bash
uptime
# 15:30:01 up 5 days, 2:30, 1 user, load average: 1.23, 0.89, 0.72
#                                                  1min  5min  15min

# Interpretation:
# Load = number of CPUs means 100% utilization
# On 4-core system: load of 4.0 = fully utilized, 8.0 = overloaded

# Rule: load average / CPU count
# < 1.0: System is under-utilized
# = 1.0: Fully utilized
# > 1.0: Overloaded (processes waiting for CPU)
```

### CPU Monitoring Commands

```bash
# top — real-time CPU monitoring
top
# %Cpu(s): 5.2 us, 1.1 sy, 0.0 ni, 92.7 id, 0.8 wa, 0.0 hi, 0.2 si
# us=user, sy=system, ni=nice, id=idle, wa=IO wait, hi=hardware IRQ, si=software IRQ

# mpstat — per-CPU statistics
mpstat 1 5                    # Every 1 sec, 5 times
mpstat -P ALL 1               # All CPUs individually

# sar — historical CPU stats
sar 1 10                      # CPU every 1 sec, 10 times
sar -u -f /var/log/sa/sa01    # Read historical data from day 1

# iostat — CPU and I/O
iostat 1 5
# avg-cpu: %user, %nice, %system, %iowait, %steal, %idle

# vmstat — CPU context
vmstat 1
# cs=context switches/sec, in=interrupts/sec
# us=user, sy=system, id=idle, wa=IO wait

# pidstat — per-process CPU
pidstat 1 5                   # All processes every 1 sec
pidstat -p 1234 1             # Specific PID
```

### CPU Performance Analysis

```bash
# Find CPU-hungry processes
ps aux --sort=-%cpu | head -10
top   # press P to sort by CPU

# CPU stress testing
stress --cpu 4 --timeout 60   # Stress 4 CPUs for 60 seconds
apt install stress-ng
stress-ng --cpu 0 --timeout 30s   # All CPUs

# CPU frequency scaling
cpufreq-info                  # Current frequency info
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
# Governors: performance, powersave, ondemand, schedutil

# Set governor
cpupower frequency-set -g performance   # Max performance
echo "performance" > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# CPU affinity — pin process to specific CPU(s)
taskset -c 0,1 command        # Run on CPUs 0 and 1
taskset -cp 0,1 1234          # Set affinity of running PID

# perf — advanced profiling
perf stat command             # CPU stats for command
perf top                      # Real-time function profiling
perf record command           # Record profile
perf report                   # Analyze recording
```

### Cross-Questions & Answers

**Q1: What does high `%wa` (IO wait) in top mean?**
> **A:** `%wa` (IO wait) is the % of CPU time the processor is idle while waiting for disk I/O. High iowait means processes are blocked waiting for slow disk operations — the bottleneck is storage, not CPU. Solutions: faster storage (SSD), RAID, tuning I/O scheduler, adding more RAM for better caching, or moving I/O-heavy operations off the system.

**Q2: What is CPU steal time (`%st`) in a virtual machine?**
> **A:** **Steal time** is CPU time that a VM "wants" to use but is being withheld by the hypervisor (because other VMs on the same host are using the physical CPU). High steal time means your VM is on an over-provisioned host. It indicates the hypervisor is scheduling other VMs ahead of yours — a performance problem beyond your control within the VM.

**Q3: What is a context switch?**
> **A:** A context switch occurs when the OS switches the CPU from one process to another. The CPU state (registers, program counter) of the current process is saved, and the state of the next process is loaded. Very frequent context switches (high `cs` in vmstat) indicate too many processes competing for CPU or inefficient process scheduling — adds overhead.

---

## 12. Systemd

### What is Systemd?

**Systemd** is the init system and service manager for most modern Linux distributions. It replaced SysV init and Upstart. PID 1 is systemd — it's the first process started by the kernel and the ancestor of all other processes.

**Systemd responsibilities:**
- Starting/stopping/managing system services (daemons)
- Parallel service startup (faster boot)
- Dependency management between services
- System logging (journald)
- System state management (shutdown, reboot, suspend)
- Mount management
- Timer (cron-like scheduling)
- Socket activation

### systemctl — Systemd Control

```bash
# Service management
systemctl start nginx           # Start service
systemctl stop nginx            # Stop service
systemctl restart nginx         # Stop + start
systemctl reload nginx          # Reload config without restart
systemctl enable nginx          # Start on boot
systemctl disable nginx         # Don't start on boot
systemctl enable --now nginx    # Enable AND start immediately
systemctl disable --now nginx   # Disable AND stop immediately
systemctl mask nginx            # Prevent starting (even manually)
systemctl unmask nginx          # Remove mask

# Status and info
systemctl status nginx          # Detailed status + recent logs
systemctl is-active nginx       # active/inactive
systemctl is-enabled nginx      # enabled/disabled
systemctl is-failed nginx       # Check if service failed
systemctl list-units --type=service          # All loaded services
systemctl list-units --type=service --all    # Include inactive
systemctl list-unit-files --type=service     # All service files

# System state
systemctl poweroff              # Shutdown
systemctl reboot                # Reboot
systemctl suspend               # Suspend
systemctl hibernate             # Hibernate
systemctl rescue                # Rescue mode (single-user)
systemctl emergency             # Emergency mode

# Analyze boot
systemd-analyze                 # Total boot time
systemd-analyze blame           # Services sorted by startup time
systemd-analyze critical-chain  # Critical path
systemd-analyze plot > boot.svg # Generate SVG boot chart
```

### Unit Files

Systemd manages **units** — these are configuration files defining services, mounts, timers, sockets, etc.

**Unit file locations (priority: last wins):**
1. `/etc/systemd/system/` — admin-defined, highest priority
2. `/run/systemd/system/` — runtime units
3. `/usr/lib/systemd/system/` — package-installed, don't edit

```bash
# View a unit file
systemctl cat nginx.service
cat /usr/lib/systemd/system/nginx.service

# Edit unit file (creates override)
systemctl edit nginx.service          # Creates drop-in override
systemctl edit --full nginx.service   # Edit full copy

# Reload after manual changes
systemctl daemon-reload
```

### Service Unit File Structure

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
Documentation=https://myapp.example.com
After=network.target postgresql.service    # Start after these
Requires=postgresql.service                # Hard dependency
Wants=redis.service                        # Soft dependency

[Service]
Type=simple                # simple, forking, notify, oneshot, dbus
User=myapp                 # Run as this user
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/start.sh     # Command to start
ExecStop=/opt/myapp/bin/stop.sh       # Command to stop
ExecReload=/bin/kill -HUP $MAINPID    # Reload command
Restart=always             # Restart on failure
RestartSec=5               # Wait 5s before restarting
StandardOutput=journal     # Send stdout to journal
StandardError=journal      # Send stderr to journal
Environment=NODE_ENV=production   # Environment variables
EnvironmentFile=/etc/myapp/env    # Load env from file
LimitNOFILE=65536          # File descriptor limit

[Install]
WantedBy=multi-user.target    # Enable for multi-user mode
```

**Service Types:**
| Type | Meaning |
|------|---------|
| `simple` | Main process is ExecStart (default) |
| `forking` | ExecStart forks; parent exits; child is daemon |
| `notify` | Process sends sd_notify() when ready |
| `oneshot` | Runs once and exits (like scripts) |
| `dbus` | Ready when D-Bus name is acquired |

### Targets — Systemd Runlevels

```bash
# Targets replace SysV runlevels
systemctl get-default                  # Current default target
systemctl set-default multi-user.target  # Set default
systemctl isolate rescue.target        # Switch to rescue mode

# Runlevel to target mapping
# 0 → poweroff.target
# 1 → rescue.target
# 2,3,4 → multi-user.target
# 5 → graphical.target
# 6 → reboot.target
```

### Timers (Systemd Cron)

```bash
# List timers
systemctl list-timers

# Example timer unit
cat /etc/systemd/system/backup.timer
# [Unit]
# Description=Daily backup
#
# [Timer]
# OnCalendar=daily
# OnCalendar=*-*-* 02:00:00   # 2am every day
# RandomizedDelaySec=300       # Randomize by 5 minutes
# Persistent=true              # Run if missed
#
# [Install]
# WantedBy=timers.target

systemctl enable --now backup.timer
systemctl status backup.timer
```

### Cross-Questions & Answers

**Q1: What is the difference between `restart` and `reload` in systemctl?**
> **A:** `restart` stops and starts the service — all connections are dropped, there's a brief downtime. `reload` sends a signal (usually SIGHUP) to the process to re-read its configuration file without stopping — no downtime, existing connections maintained. Not all services support reload. Use `reload` for live changes (nginx, apache, sshd).

**Q2: What does `WantedBy=multi-user.target` mean in [Install] section?**
> **A:** It defines where to "install" the service when enabled. When you run `systemctl enable service`, systemd creates a symlink in the `multi-user.target.wants/` directory. This makes the service start when the system reaches multi-user mode (normal operation). `WantedBy=graphical.target` would start it only in GUI mode.

**Q3: What is the difference between `Requires` and `Wants` in a unit file?**
> **A:** `Requires=` is a **hard dependency** — if the required unit fails to start or stops, this unit also stops. `Wants=` is a **soft dependency** — this unit *prefers* the other to be started, but if it fails, this unit continues anyway. Use `Wants` for optional dependencies; `Requires` for mandatory ones.

**Q4: How do you troubleshoot a service that fails to start?**
> **A:** Step-by-step: (1) `systemctl status servicename` — shows last error. (2) `journalctl -u servicename -n 50` — detailed logs. (3) `journalctl -u servicename --since "5 minutes ago"` — recent logs. (4) Run the ExecStart command manually as the service user to see real-time errors. (5) Check `strace -p PID` for system call failures. (6) Verify file permissions, paths, and environment variables.

---

## 13. Using Journalctl

### What is the Journal?

`journald` is systemd's logging daemon. It collects log messages from:
- Kernel (dmesg)
- System services (stdout/stderr of systemd services)
- Syslog
- Audit daemon
- Boot messages

Logs are stored in **binary format** at `/var/log/journal/` (persistent) or `/run/log/journal/` (volatile, lost on reboot).

### Basic journalctl Usage

```bash
# View all logs (oldest first)
journalctl

# Follow new entries in real-time
journalctl -f                          # Like tail -f

# Show most recent 50 lines
journalctl -n 50

# Since last boot
journalctl -b                          # Current boot
journalctl -b -1                       # Previous boot
journalctl --list-boots                # List all recorded boots

# Kernel messages
journalctl -k                          # Kernel ring buffer (like dmesg)
dmesg | tail -20                       # Alternative
```

### Filtering Journal Logs

```bash
# By service unit
journalctl -u nginx.service
journalctl -u nginx -u mysql           # Multiple units
journalctl -u nginx --since "1 hour ago"

# By time
journalctl --since "2024-01-01 10:00:00"
journalctl --until "2024-01-01 11:00:00"
journalctl --since "1 hour ago"
journalctl --since yesterday
journalctl --since today

# By priority (log level)
journalctl -p err                      # Error and above
journalctl -p warning                  # Warning and above
# Levels: emerg, alert, crit, err, warning, notice, info, debug

# By PID, UID
journalctl _PID=1234
journalctl _UID=1001                   # All logs from UID 1001
journalctl _COMM=sshd                  # All logs from sshd

# Grep-like
journalctl | grep -i "error"
journalctl -u nginx | grep "404"

# Output formats
journalctl -o short                    # Default
journalctl -o verbose                  # All fields
journalctl -o json                     # JSON format
journalctl -o json-pretty             # Pretty JSON
journalctl -o cat                      # Just the message, no metadata
```

### Journal Maintenance

```bash
# Check journal size
journalctl --disk-usage

# Rotate/cleanup old logs
journalctl --vacuum-size=500M         # Keep at most 500MB
journalctl --vacuum-time=30d          # Keep last 30 days
journalctl --vacuum-files=5           # Keep max 5 files

# Configure persistent journal
mkdir -p /var/log/journal
systemd-tmpfiles --create --prefix /var/log/journal
# Or edit /etc/systemd/journald.conf:
# Storage=persistent    (default=auto)
# SystemMaxUse=500M
# SystemKeepFree=100M

# Flush journal to disk
journalctl --flush

# Verify journal integrity
journalctl --verify
```

### Cross-Questions & Answers

**Q1: What is the advantage of binary journal format over plain text logs?**
> **A:** Binary format enables: (1) **Structured fields** — filter by exact field values (UID, PID, unit) without regex; (2) **Compression** — binary is more compact; (3) **Cryptographic sealing** — detect log tampering; (4) **Metadata-rich** — each entry carries source, timing, priority, unit, etc. The downside is you need `journalctl` to read it (though you can export to JSON/text).

**Q2: How do you forward journald logs to a remote syslog server?**
> **A:** Configure `rsyslog` or `syslog-ng` with a forwarding rule, and systemd will forward to `/run/systemd/journal/syslog` socket. Or use `systemd-journal-remote` for native journal-to-journal forwarding. Or install `rsyslog` and configure it to read from journald and forward: `$ModLoad imjournal` in rsyslog.conf.

**Q3: What is `journalctl -b` and how is it different from `journalctl -b -1`?**
> **A:** `journalctl -b` (or `journalctl -b 0`) shows logs from the **current boot** only. `journalctl -b -1` shows logs from the **previous boot** (one boot before current). This is invaluable for diagnosing crashes — you can see what was logged before the system went down. `--list-boots` shows all boot IDs available.

---

## 14. Software Management

### Package Managers by Distro Family

| Distro Family | Package Manager | Package Format | Low-level tool |
|--------------|----------------|----------------|----------------|
| Debian/Ubuntu | apt | .deb | dpkg |
| RHEL/CentOS/Fedora | dnf (yum) | .rpm | rpm |
| Arch | pacman | .pkg.tar.zst | — |
| openSUSE | zypper | .rpm | rpm |

### APT — Debian/Ubuntu

```bash
# Update package list (metadata only, not packages)
apt update

# Upgrade installed packages
apt upgrade                        # Safe upgrade
apt full-upgrade                   # May remove packages to upgrade

# Install packages
apt install nginx
apt install nginx -y               # Auto-yes
apt install nginx mysql-server php # Multiple packages
apt install ./package.deb          # Local .deb file

# Remove packages
apt remove nginx                   # Remove but keep config
apt purge nginx                    # Remove including config files
apt autoremove                     # Remove unused dependencies

# Search and info
apt search nginx
apt show nginx                     # Package details
apt list --installed               # All installed packages
apt list --upgradable              # Packages with updates

# Package files (dpkg)
dpkg -l | grep nginx               # Check if installed
dpkg -L nginx                      # Files installed by package
dpkg -S /usr/bin/nginx             # Which package owns a file
dpkg -i package.deb                # Install deb file
dpkg -r nginx                      # Remove package
dpkg --configure -a                # Fix broken installations

# Repositories
cat /etc/apt/sources.list          # Repository list
ls /etc/apt/sources.list.d/        # Additional repos
add-apt-repository ppa:user/ppa    # Add PPA (Ubuntu)
apt-key add keyfile                # Add signing key (deprecated)
```

### DNF — Red Hat/CentOS/Fedora

```bash
# Update
dnf check-update                   # Check for updates
dnf update                         # Update all
dnf update nginx                   # Update specific package

# Install
dnf install nginx
dnf install nginx -y
dnf localinstall package.rpm       # Local RPM file

# Remove
dnf remove nginx                   # Remove package
dnf autoremove                     # Remove unused deps

# Search and info
dnf search nginx
dnf info nginx
dnf list installed                 # All installed
dnf list available                 # Available in repos
dnf provides /usr/bin/nginx        # Which package provides file
dnf repolist                       # List repositories

# Groups
dnf group list                     # List package groups
dnf group install "Development Tools"

# History
dnf history                        # Transaction history
dnf history undo 5                 # Undo transaction 5

# RPM (low-level)
rpm -qa                            # All installed packages
rpm -qi nginx                      # Package info
rpm -ql nginx                      # Files in package
rpm -qf /usr/sbin/nginx            # Package owning file
rpm -ivh package.rpm               # Install
rpm -e nginx                       # Remove

# Repositories
ls /etc/yum.repos.d/               # Repo configuration files
dnf config-manager --add-repo URL  # Add repo
```

### Compilation from Source

```bash
# General pattern
wget https://example.com/software-1.0.tar.gz
tar -xzvf software-1.0.tar.gz
cd software-1.0/

# Check requirements
cat INSTALL README

# Configure build
./configure --prefix=/usr/local    # Set installation directory
./configure --help                 # See all options

# Compile
make                               # Compile (use -j N for N parallel jobs)
make -j$(nproc)                    # Use all CPU cores

# Install
sudo make install

# Uninstall (if supported)
sudo make uninstall
```

### Snap and Flatpak (Universal Packages)

```bash
# Snap (Ubuntu/Canonical)
snap install code --classic        # Install VS Code
snap list                          # Installed snaps
snap refresh                       # Update snaps
snap remove code                   # Remove

# Flatpak
flatpak install flathub org.gimp.GIMP
flatpak update
flatpak list
flatpak uninstall org.gimp.GIMP
```

### Cross-Questions & Answers

**Q1: What is the difference between `apt update` and `apt upgrade`?**
> **A:** `apt update` **downloads the package index** (list of available packages and versions) from repositories — it does NOT install/upgrade anything. `apt upgrade` uses that updated index to **upgrade installed packages** to newer versions. Always run `apt update` before `apt upgrade` to ensure you're seeing the latest available versions.

**Q2: What happens if you install a broken package with dpkg?**
> **A:** dpkg will report errors about missing dependencies. The package manager enters a broken state. Fix with: `apt install -f` (fix broken packages — installs missing dependencies) or `dpkg --configure -a` (configure any unconfigured packages). Never install `.deb` files directly with dpkg unless you're sure dependencies are satisfied; use `apt install ./package.deb` instead so apt resolves dependencies.

**Q3: What is a PPA (Personal Package Archive)?**
> **A:** PPAs are unofficial package repositories hosted on Launchpad (Ubuntu's hosting platform). Individuals and organizations use PPAs to distribute software not in official Ubuntu repos (often newer versions). Security consideration: PPAs are not officially vetted — only add PPAs from trusted sources, as they have full system access. Use `add-apt-repository ppa:name/ppa` to add them.

---

## 15. Network Configuration

### Network Interface Naming

Linux uses **predictable network interface names** (since systemd):
- `eth0` — old-style (still seen in some VMs)
- `enp3s0` — Ethernet, PCIe bus 3, slot 0
- `ens33` — Ethernet, slot 33
- `wlan0` / `wlp2s0` — Wireless
- `lo` — Loopback (127.0.0.1)

```bash
# List interfaces
ip link show
ip addr show
nmcli device status               # NetworkManager status
```

### ip Command (Modern)

```bash
# IP addresses
ip addr show                       # All interfaces
ip addr show eth0                  # Specific interface
ip addr add 192.168.1.10/24 dev eth0    # Add IP
ip addr del 192.168.1.10/24 dev eth0   # Remove IP

# Links (Layer 2)
ip link show
ip link set eth0 up               # Bring interface up
ip link set eth0 down             # Bring interface down
ip link set eth0 mtu 9000         # Set MTU (jumbo frames)

# Routes
ip route show                      # Routing table
ip route add default via 192.168.1.1   # Set default gateway
ip route add 10.0.0.0/8 via 192.168.1.1  # Static route
ip route del 10.0.0.0/8           # Delete route

# Neighbors (ARP table)
ip neigh show                      # ARP cache
ip neigh flush all                 # Flush ARP cache
```

### NetworkManager (nmcli/nmtui)

Most modern distros use **NetworkManager** for network management.

```bash
# nmcli — Command line
nmcli device                       # Device status
nmcli connection show              # All connections
nmcli connection show "enp3s0"    # Specific connection

# Connect/disconnect
nmcli device connect eth0
nmcli device disconnect eth0

# Manage connections
nmcli connection add type ethernet ifname eth0 con-name myconn
nmcli connection modify myconn ipv4.addresses 192.168.1.100/24
nmcli connection modify myconn ipv4.gateway 192.168.1.1
nmcli connection modify myconn ipv4.dns "8.8.8.8 8.8.4.4"
nmcli connection modify myconn ipv4.method manual   # Static
nmcli connection modify myconn ipv4.method auto     # DHCP
nmcli connection up myconn
nmcli connection down myconn
nmcli connection delete myconn

# nmtui — Text UI (easier for beginners)
nmtui
```

### Static IP Configuration

#### RHEL/CentOS/AlmaLinux — /etc/sysconfig/network-scripts/

```ini
# /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
BOOTPROTO=static          # or dhcp
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.1.100
PREFIX=24
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4
```

```bash
systemctl restart NetworkManager
nmcli connection reload
```

#### Ubuntu — Netplan (Ubuntu 18.04+)

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

```bash
netplan try                        # Test for 120 seconds
netplan apply                      # Apply configuration
```

### DNS Configuration

```bash
# /etc/resolv.conf — DNS resolver
cat /etc/resolv.conf
# nameserver 8.8.8.8
# nameserver 8.8.4.4
# search example.com               # Append to unqualified hostnames

# /etc/hosts — static hostname resolution (checked before DNS)
cat /etc/hosts
# 127.0.0.1 localhost
# 192.168.1.100 myserver.example.com myserver

# Resolution order — /etc/nsswitch.conf
grep hosts /etc/nsswitch.conf
# hosts: files dns myhostname     (files=hosts file, then dns)
```

### Cross-Questions & Answers

**Q1: What is the difference between `ip` and `ifconfig`?**
> **A:** `ifconfig` is the older net-tools command — deprecated and not installed by default on modern distros. `ip` is from iproute2 suite — modern, more feature-rich, consistent syntax, and supports all current Linux networking features. Use `ip` for all new scripts. Key mappings: `ifconfig eth0` → `ip addr show eth0`, `route -n` → `ip route show`, `arp -n` → `ip neigh show`.

**Q2: Changes made with `ip addr add` don't persist after reboot. Why?**
> **A:** `ip` commands modify the **live kernel networking state** directly — they're not written to configuration files. On reboot, the kernel starts fresh and reads configuration from files (NetworkManager, netplan, ifcfg scripts). To persist changes, modify the appropriate configuration file and reload NetworkManager, or use `nmcli` which writes to files automatically.

---

## 16. SSH Overview and SSH Clients

### SSH Architecture (Recap + Linux Focus)

```bash
# SSH server (sshd)
systemctl status sshd
cat /etc/ssh/sshd_config           # Server configuration

# Key server config options
Port 22                            # Listen port
PermitRootLogin no                 # Deny root login
PasswordAuthentication yes/no      # Password auth
PubkeyAuthentication yes           # Key-based auth
AllowUsers alice bob               # Whitelist users
DenyUsers root hacker              # Blacklist users
MaxAuthTries 3                     # Limit failed attempts
ClientAliveInterval 300            # Keep-alive every 5 min
ClientAliveCountMax 2              # Disconnect after 2 missed
```

### SSH Key Management

```bash
# Generate key pair
ssh-keygen -t ed25519 -C "alice@example.com"     # Modern, recommended
ssh-keygen -t rsa -b 4096 -C "alice@example.com" # RSA 4096-bit
ssh-keygen -t ecdsa -b 521                        # ECDSA

# Keys stored in:
ls ~/.ssh/
# id_ed25519       → Private key (NEVER share)
# id_ed25519.pub   → Public key (share freely)
# authorized_keys  → Server: list of allowed public keys
# known_hosts      → Client: known server fingerprints
# config           → SSH client configuration

# Copy public key to server
ssh-copy-id alice@server           # Appends to ~/.ssh/authorized_keys
ssh-copy-id -i ~/.ssh/mykey.pub alice@server  # Specific key
# Manual alternative:
cat ~/.ssh/id_ed25519.pub | ssh alice@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Permissions MUST be correct (SSH is strict)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519        # Private key
chmod 644 ~/.ssh/id_ed25519.pub    # Public key
```

### SSH Client Configuration

```ini
# ~/.ssh/config — client-side aliases and settings
Host myserver
    HostName 192.168.1.100
    User alice
    Port 2222
    IdentityFile ~/.ssh/myserver_key
    ServerAliveInterval 60

Host jump-host
    HostName jump.example.com
    User alice

Host internal-server
    HostName 10.0.0.50
    User bob
    ProxyJump jump-host           # Connect via jump host
```

```bash
# Connect using config alias
ssh myserver                      # Uses Host alias
ssh -J jump.example.com internal  # Jump host (inline)
ssh -L 8080:localhost:80 server   # Local port forward
ssh -R 9090:localhost:3000 server # Remote port forward
ssh -D 1080 server                # SOCKS proxy
ssh -N -f -L 8080:localhost:80 server  # Background, no shell
```

### SSH Agent

```bash
# SSH agent stores decrypted keys in memory
eval $(ssh-agent)                  # Start agent
ssh-add ~/.ssh/id_ed25519          # Add key to agent
ssh-add -l                         # List loaded keys
ssh-add -d ~/.ssh/id_ed25519       # Remove key from agent
ssh-add -D                         # Remove all keys

# Agent forwarding — use local keys on remote server
ssh -A server                      # Forward agent
# Or in ~/.ssh/config: ForwardAgent yes

# ssh-agent one-liner
eval "$(ssh-agent -s)" && ssh-add
```

### SCP and SFTP

```bash
# SCP — Secure Copy (file transfer over SSH)
scp file.txt alice@server:/tmp/           # Upload file
scp alice@server:/tmp/file.txt .          # Download file
scp -r directory/ alice@server:/backup/  # Recursive
scp -P 2222 file.txt alice@server:/tmp/  # Custom port
scp -i ~/.ssh/mykey file.txt server:     # Specific key

# SFTP — Secure FTP (interactive)
sftp alice@server
# sftp> ls, get, put, cd, lcd, mkdir, rm, quit
sftp> get remotefile localfile
sftp> put localfile remotefile
sftp> mget *.txt                  # Multiple get
sftp> mput *.txt                  # Multiple put

# rsync over SSH — efficient sync
rsync -avz -e ssh source/ alice@server:/dest/
rsync -avz --progress file server:/path/  # Show progress
```

### Cross-Questions & Answers

**Q1: You copied your public key to server but SSH still asks for password. What's wrong?**
> **A:** Check in order: (1) `~/.ssh/` on server must be `chmod 700`; (2) `~/.ssh/authorized_keys` must be `chmod 600`; (3) `authorized_keys` must contain your public key exactly; (4) `/etc/ssh/sshd_config` must have `PubkeyAuthentication yes`; (5) `StrictModes yes` requires correct ownership; (6) Check `/var/log/auth.log` or `journalctl -u sshd` for the actual error.

**Q2: What is SSH agent forwarding and what are its risks?**
> **A:** Agent forwarding (`ssh -A`) allows the remote server to use your local SSH agent to authenticate to further servers. Useful for jump hosts. **Risk:** A malicious root user on the intermediate server can use your agent socket to authenticate as you to other servers — they can't steal your private key, but they can impersonate you as long as you're connected. Only use agent forwarding to trusted servers.

---

## 17. VirtualBox Networking

### VirtualBox Network Modes

VirtualBox provides several networking modes for VMs, each with different visibility and routing characteristics.

| Mode | VM↔Host | VM↔VM | VM↔Internet | External↔VM |
|------|---------|--------|------------|-------------|
| **NAT** | No* | No | Yes | No (port forward only) |
| **NAT Network** | No | Yes (same network) | Yes | No (port forward only) |
| **Bridged** | Yes | Yes | Yes | Yes |
| **Internal Network** | No | Yes (same name) | No | No |
| **Host-Only** | Yes | Yes | No | No |
| **Not Attached** | No | No | No | No |

#### NAT (Default)
```
VM ──NAT engine──► Host NIC ──► Internet
VM gets: 10.0.2.x (VirtualBox built-in)
Gateway: 10.0.2.2
DNS: 10.0.2.3
Host: 10.0.2.2
```
- VirtualBox acts as a NAT router
- VM can reach internet but is not reachable from outside
- Use **port forwarding** to expose services: Host:2222 → VM:22

#### Bridged Adapter
```
VM ─────────────► Physical NIC ──► Network
VM appears as physical device on network
VM gets IP from real DHCP server
```
- VM gets same network access as host
- Visible to all devices on network
- Best for servers that need to be accessible

#### Host-Only
```
VM ←──Host-Only Adapter──► Host
VMs and host can communicate
No internet access
```
- Creates virtual network between host and VMs
- Great for lab environments
- Multiple VMs can communicate via same host-only network

#### Internal Network
```
VM1 ←──Internal Network──► VM2
Named internal networks: isolated islands
```
- VMs on same internal network name can communicate
- No host access, no internet
- Perfect for isolated test environments

### Port Forwarding in NAT Mode

```
VirtualBox Manager → VM Settings → Network → NAT → Port Forwarding
Host IP: 127.0.0.1 (or leave blank for all)
Host Port: 2222
Guest IP: (leave blank — VM's IP)
Guest Port: 22

# Then connect: ssh -p 2222 user@127.0.0.1
```

### VirtualBox Command Line (VBoxManage)

```bash
# List VMs
VBoxManage list vms
VBoxManage list runningvms

# Start/stop
VBoxManage startvm "MyVM" --type headless
VBoxManage controlvm "MyVM" poweroff
VBoxManage controlvm "MyVM" savestate

# Network config
VBoxManage modifyvm "MyVM" --nic1 bridged
VBoxManage modifyvm "MyVM" --bridgeadapter1 eth0
VBoxManage modifyvm "MyVM" --nic2 hostonly
VBoxManage modifyvm "MyVM" --hostonlyadapter2 vboxnet0

# Port forwarding (NAT mode)
VBoxManage modifyvm "MyVM" --natpf1 "ssh,tcp,,2222,,22"
```

### Cross-Questions & Answers

**Q1: When should you use Bridged vs NAT networking in VirtualBox?**
> **A:** Use **Bridged** when the VM needs to be accessible from other machines on the network (as a server, for testing network services, or to simulate a real network device). Use **NAT** when the VM just needs internet access for updates/downloads and doesn't need to be reached from outside — it's simpler and more secure.

**Q2: What is the difference between NAT and NAT Network in VirtualBox?**
> **A:** **NAT** gives each VM its own isolated NAT — VMs cannot communicate with each other. **NAT Network** shares one NAT among multiple VMs — they can communicate with each other (like a real LAN behind a router) and all reach the internet, but are NAT'd to the outside. NAT Network is better for multi-VM labs where VMs need to talk to each other.

---

## 18. Basics of Linux Networking

### Network Interface Configuration

```bash
# View all interfaces
ip addr show
ip link show

# Interface information
ethtool eth0                       # Physical NIC info (speed, duplex)
ethtool -i eth0                    # Driver info
mii-tool eth0                      # Media type (older)

# View ARP table
ip neigh show                      # ARP/NDP cache
arp -n                             # Old command

# View connections
ss -tuln                           # Listening TCP/UDP sockets
ss -tunp                           # With process info
ss -s                              # Summary statistics
netstat -tuln                      # Old equivalent (net-tools)
netstat -rn                        # Routing table (old)
```

### Network Diagnostics Fundamentals

```bash
# Ping — ICMP echo test
ping 8.8.8.8                      # Basic connectivity
ping -c 4 8.8.8.8                 # 4 packets only
ping -i 0.2 8.8.8.8              # Fast (0.2s interval)
ping -s 1400 8.8.8.8              # Large packet (test MTU)
ping -6 ipv6.google.com           # IPv6 ping
ping6 ipv6.google.com             # Same

# traceroute — Path discovery
traceroute 8.8.8.8                # UDP by default
traceroute -T 8.8.8.8            # TCP mode
traceroute -I 8.8.8.8            # ICMP mode
tracepath 8.8.8.8                 # Similar, also shows MTU
mtr 8.8.8.8                       # Continuous traceroute + ping stats

# DNS testing
dig google.com                    # Full DNS query
dig google.com A                  # A record
dig google.com MX                 # MX records
dig @8.8.8.8 google.com          # Query specific DNS server
dig +short google.com             # Just the IP
nslookup google.com               # Simpler DNS lookup
host google.com                   # Simple lookup
dig -x 8.8.8.8                   # Reverse DNS lookup
```

### Network Statistics

```bash
# Network interface statistics
ip -s link show eth0              # RX/TX counters
cat /proc/net/dev                 # Raw stats
watch -n 1 'ip -s link show eth0' # Live monitoring

# Connection tracking
cat /proc/net/nf_conntrack        # NAT connection table
conntrack -L                      # If conntrack tools installed

# Bandwidth monitoring
iftop -i eth0                     # Interactive bandwidth monitor
nethogs eth0                      # Per-process bandwidth
nload eth0                        # Simple bandwidth meter
bmon                              # Multi-interface bandwidth
```

### /etc/hosts and Name Resolution Order

```bash
# /etc/hosts — override DNS for specific names
cat /etc/hosts
127.0.0.1       localhost
192.168.1.100   myserver.local myserver

# Resolution order (nsswitch.conf)
grep ^hosts /etc/nsswitch.conf
# hosts: files dns myhostname
# 1. /etc/hosts
# 2. DNS (/etc/resolv.conf)
# 3. myhostname (machine's own hostname)
```

### Cross-Questions & Answers

**Q1: What is the difference between `ss` and `netstat`?**
> **A:** Both show socket/connection information. `netstat` (from net-tools) is older and deprecated — it reads `/proc/net/tcp` directly, which is slow on systems with many connections. `ss` (socket statistics, from iproute2) uses kernel netlink API — faster, more detailed, and actively maintained. Use `ss` for all new work. Key equivalent: `netstat -tuln` → `ss -tuln`.

**Q2: Why does ping sometimes fail even when the host is up?**
> **A:** (1) **ICMP blocked** — firewall rules drop ICMP echo requests; (2) **Host firewall** — remote server drops ICMP; (3) **Rate limiting** — server throttles ICMP responses; (4) **Routing issue** — packet reaches server but reply takes wrong path; (5) **IPv6 vs IPv4** — resolving to IPv6 when IPv4 is expected. Test with `traceroute` and TCP-based tools (curl, nc) to distinguish network vs ICMP issues.

---

## 19. DHCP in Linux Networking

### DHCP Server — ISC DHCP / Kea

```bash
# Install ISC DHCP server (traditional)
dnf install dhcp-server            # RHEL/CentOS
apt install isc-dhcp-server        # Debian/Ubuntu

# Configure
cat /etc/dhcp/dhcpd.conf
```

```conf
# /etc/dhcp/dhcpd.conf
default-lease-time 600;            # 10 minutes
max-lease-time 7200;               # 2 hours

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option domain-name-servers 8.8.8.8, 8.8.4.4;
    option domain-name "example.com";
    option broadcast-address 192.168.1.255;
}

# Static reservation
host myserver {
    hardware ethernet AA:BB:CC:DD:EE:FF;
    fixed-address 192.168.1.50;
}
```

```bash
# Start/enable
systemctl enable --now dhcpd
journalctl -u dhcpd -f             # Monitor logs

# View leases
cat /var/lib/dhcpd/dhcpd.leases    # Lease database
```

### DHCP Client

```bash
# Manual DHCP request
dhclient eth0                      # Get IP via DHCP
dhclient -r eth0                   # Release DHCP lease
dhclient -v eth0                   # Verbose — see DORA

# NetworkManager DHCP
nmcli connection modify eth0 ipv4.method auto
nmcli connection up eth0

# DHCP client configuration
cat /etc/dhcp/dhclient.conf
# request subnet-mask, broadcast-address, routers, domain-name-servers;
# prepend domain-name-servers 127.0.0.1;   # Add local DNS first

# View assigned lease
cat /var/lib/dhcp/dhclient.leases  # Client-side lease file
```

### dnsmasq — Lightweight DHCP + DNS

```bash
# Install
apt install dnsmasq

# /etc/dnsmasq.conf
interface=eth0
dhcp-range=192.168.1.100,192.168.1.200,24h
dhcp-option=3,192.168.1.1         # Gateway
dhcp-option=6,8.8.8.8,8.8.4.4   # DNS
dhcp-host=AA:BB:CC:DD:EE:FF,192.168.1.50,myserver  # Reservation

systemctl enable --now dnsmasq
```

### Cross-Questions & Answers

**Q1: What is the DHCP lease renewal process in detail?**
> **A:** At **T1 (50% of lease time)**, client sends unicast DHCP Request to the original server to renew. If server responds with ACK, lease is renewed. If no response by **T2 (87.5%)**, client broadcasts DHCP Request to any DHCP server. At **T3 (100%)**, lease expires — client must release the IP and restart the full DORA discovery process, during which it cannot use the expired IP.

**Q2: How does DHCP snooping work in a Linux bridge?**
> **A:** Linux bridge with `br_netfilter` and ebtables/nftables can enforce DHCP snooping. Mark trusted ports (toward DHCP server) and untrusted (toward clients). Block DHCP OFFER and ACK packets incoming on untrusted ports. This prevents rogue DHCP servers. In production, this is usually enforced on managed switches via native DHCP snooping features.

---

## 20. Linux IP Routing

### Routing Table

The **routing table** determines where packets go based on destination IP.

```bash
# View routing table
ip route show
# default via 192.168.1.1 dev eth0 proto dhcp metric 100
# 192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100

# Columns: destination, via (gateway), dev (interface), metric

# Check which route would be used
ip route get 8.8.8.8
# 8.8.8.8 via 192.168.1.1 dev eth0 src 192.168.1.100

route -n                          # Old command (still useful)
```

### Route Management

```bash
# Add routes
ip route add 10.0.0.0/8 via 192.168.1.1         # Static route
ip route add 10.0.0.0/8 dev eth0                 # Direct (no gateway)
ip route add default via 192.168.1.1              # Default gateway
ip route add 192.168.2.0/24 via 192.168.1.254 metric 100  # With metric

# Delete routes
ip route del 10.0.0.0/8
ip route del default

# Flush all routes (careful!)
ip route flush table main

# Multiple default routes (policy routing)
ip route add default via 192.168.1.1 table 100
ip rule add from 192.168.2.0/24 table 100

# Persistent routes (RHEL)
# /etc/sysconfig/network-scripts/route-eth0:
# 10.0.0.0/8 via 192.168.1.254

# Persistent routes (Ubuntu/Netplan)
# In /etc/netplan/01-netcfg.yaml under the interface:
# routes:
#   - to: 10.0.0.0/8
#     via: 192.168.1.254
```

### IP Forwarding (Routing Between Interfaces)

To make a Linux system **route packets** between interfaces (act as a router):

```bash
# Check current setting
cat /proc/sys/net/ipv4/ip_forward  # 0=disabled, 1=enabled

# Enable temporarily
sysctl -w net.ipv4.ip_forward=1
echo 1 > /proc/sys/net/ipv4/ip_forward

# Enable permanently
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p                           # Apply sysctl.conf

# IPv6 forwarding
sysctl -w net.ipv6.conf.all.forwarding=1
```

### Dynamic Routing Protocols

For dynamic routing (routes learned automatically):

```bash
# FRRouting (modern routing suite — supports OSPF, BGP, RIP, etc.)
apt install frr
systemctl enable --now frr

# Configure via vtysh (Quagga/FRR CLI similar to Cisco IOS)
vtysh
router# configure terminal
router(config)# router ospf
router(config-router)# network 192.168.1.0/24 area 0
router(config-router)# end
router# write memory
```

### Cross-Questions & Answers

**Q1: What is a metric in routing and why does it matter?**
> **A:** A **metric** is a cost/preference value assigned to a route. When multiple routes exist to the same destination, the one with the **lowest metric** is preferred. For example, if eth0 has metric 100 and eth1 has metric 200 to the same network, eth0 is used. If eth0 fails, traffic falls back to eth1. DHCP assigns metrics automatically; manually configured routes may need explicit metrics.

**Q2: What is policy routing (RPDB)?**
> **A:** Linux supports multiple routing tables and **rules** (the Routing Policy Database — RPDB) to determine which table to use. You can route traffic differently based on source IP, TOS, or firewall marks. Example: traffic from 192.168.2.x always goes out eth1 (even if the main table says eth0). This is used for multi-homing, VPN, and traffic shaping.

**Q3: What does `ip route get 8.8.8.8` do?**
> **A:** It shows exactly which route the kernel would use to reach 8.8.8.8 — including the gateway, outgoing interface, and source IP. It's the most reliable way to debug routing because it uses the actual kernel routing algorithm, not just displays the table. Very useful when you have complex routing rules.

---

## 21. Linux Network Troubleshooting

### Systematic Troubleshooting Approach

```
Layer 1: Physical — Is interface up? Cable connected?
Layer 2: Data Link — ARP working? MAC reachable?
Layer 3: Network — Can ping gateway? Routing correct?
Layer 4: Transport — Can TCP connect? Port open?
Layer 7: Application — Service running? Config correct?
```

### Layer-by-Layer Diagnostic Commands

```bash
# Layer 1/2 — Interface and Link
ip link show eth0                  # Interface state (UP/DOWN)
ethtool eth0                       # Link speed, duplex, connected
ip -s link show eth0              # Packet/error counters

# Check for errors
ip -s link show eth0 | grep -E "RX|TX|errors|dropped"
# High error/dropped counts = hardware or duplex mismatch

# Layer 3 — IP and Routing
ip addr show                       # Is IP assigned?
ip route show                      # Is default route set?
ping 192.168.1.1                  # Can reach gateway?
ping 8.8.8.8                      # Can reach internet?
ping google.com                    # DNS working?

# Trace the path
traceroute -n 8.8.8.8             # Where does it stop?
mtr 8.8.8.8                       # Continuous trace

# Layer 4 — Port connectivity
nc -zv 192.168.1.100 22           # Can we connect to port 22?
nc -zv 8.8.8.8 443                # Can we reach HTTPS?
curl -v telnet://server:25         # Test SMTP
ss -tuln                           # What's listening locally?

# Layer 7 — Application
curl -v http://server/             # HTTP test with details
wget -O- http://server/            # Download test
openssl s_client -connect server:443  # TLS test
```

### DNS Troubleshooting

```bash
# Test DNS resolution
dig google.com                     # Full details
dig @8.8.8.8 google.com           # Test specific server
dig google.com +trace              # Trace full delegation

# Check resolv.conf
cat /etc/resolv.conf

# Check nsswitch order
cat /etc/nsswitch.conf | grep hosts

# Test with different methods
host google.com                    # Simple
nslookup google.com               # Interactive
getent hosts google.com            # Uses nsswitch (same as app)

# Flush DNS cache
systemd-resolve --flush-caches    # Systemd-resolved
/etc/init.d/nscd restart          # nscd (if used)
```

### Packet Capture (tcpdump)

```bash
# Basic capture
tcpdump -i eth0                    # All traffic on eth0
tcpdump -i any                     # All interfaces

# Filters
tcpdump -i eth0 host 192.168.1.100    # Traffic to/from IP
tcpdump -i eth0 port 80               # HTTP traffic
tcpdump -i eth0 tcp port 22           # SSH only
tcpdump -i eth0 src 192.168.1.100     # Source IP only
tcpdump -i eth0 dst 8.8.8.8          # Destination only
tcpdump -i eth0 icmp                  # ICMP (ping) only
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'  # SYN packets

# Output options
tcpdump -i eth0 -n                 # Don't resolve names
tcpdump -i eth0 -v                 # Verbose
tcpdump -i eth0 -vv                # More verbose
tcpdump -i eth0 -c 100            # Capture 100 packets
tcpdump -i eth0 -w capture.pcap   # Write to file
tcpdump -r capture.pcap           # Read from file
tcpdump -i eth0 -A port 80        # ASCII output (HTTP)
tcpdump -i eth0 -X port 80        # Hex+ASCII output

# Complex filters
tcpdump -i eth0 'port 80 and host 192.168.1.100'
tcpdump -i eth0 'not port 22'     # Exclude SSH
```

### Common Problems and Solutions

```bash
# "Network unreachable"
ip route show                      # Check routing table
# Solution: ip route add default via GATEWAY dev INTERFACE

# "No route to host"
ping -c 1 192.168.1.1             # Test gateway
iptables -L                        # Check firewall
# Solution: check firewall, check remote firewall

# Interface not getting IP
journalctl -u NetworkManager -f   # Watch NM logs
dhclient -v eth0                  # Manual DHCP verbose
# Solution: check DHCP server, check interface config

# Slow DNS
dig +time=2 +tries=1 google.com  # Set timeout
cat /etc/resolv.conf              # Check DNS servers
# Solution: change DNS servers in resolv.conf

# Port not reachable (service running but inaccessible)
ss -tlnp | grep :80               # Is nginx listening?
iptables -L INPUT -n -v           # Firewall blocking?
firewall-cmd --list-all           # firewalld rules
# Solution: open port in firewall, check bind address (0.0.0.0 vs 127.0.0.1)
```

### Cross-Questions & Answers

**Q1: A server can ping 8.8.8.8 but can't reach google.com. What's wrong?**
> **A:** DNS resolution is failing. The server can reach the internet (ping by IP works) but can't resolve hostnames. Check: `cat /etc/resolv.conf` (valid DNS servers configured?), `dig @8.8.8.8 google.com` (can reach DNS port 53?), `dig google.com` (using configured servers — does it work?). Fix: ensure correct DNS servers in `/etc/resolv.conf` and port 53 is not blocked in firewall.

**Q2: How do you capture only HTTP traffic that doesn't include your SSH connection?**
> **A:** `tcpdump -i eth0 port 80 and not port 22` — but actually if your session IS on port 22, filtering out port 22 just removes your own control traffic. Better: `tcpdump -i eth0 port 80 -w http.pcap` then analyze with Wireshark. Or use a filter: `tcpdump -i eth0 'port 80 and not src host YOUR_IP'` to exclude your own IP.

---

## 22. Linux Firewall and NAT

### Netfilter Architecture

Linux packet filtering is provided by the **Netfilter** kernel framework. Tables contain chains, chains contain rules.

```
Incoming packet ──► PREROUTING ──► FORWARD ──► POSTROUTING ──► Out
                                    │
                         INPUT ◄────┘
                           │
                       Local process
                           │
                         OUTPUT ──► POSTROUTING ──► Out
```

**Tables:**
| Table | Purpose |
|-------|---------|
| **filter** | Packet filtering (accept/drop/reject) — default |
| **nat** | NAT (source, destination, masquerade) |
| **mangle** | Packet modification (TTL, TOS, marks) |
| **raw** | Connection tracking bypass |
| **security** | SELinux marking |

**Chains:**
| Chain | Where packets are processed |
|-------|---------------------------|
| **INPUT** | Packets destined for local system |
| **OUTPUT** | Packets generated by local system |
| **FORWARD** | Packets routed through system |
| **PREROUTING** | Before routing decision (DNAT) |
| **POSTROUTING** | After routing decision (SNAT/Masquerade) |

### iptables (Traditional)

```bash
# List rules
iptables -L                       # List filter table
iptables -L -n -v                 # With counters, no name resolve
iptables -L INPUT -n --line-numbers  # With line numbers
iptables -t nat -L -n -v          # NAT table

# Basic rules — filter table (default)
# Allow established connections (stateful)
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow ping
iptables -A INPUT -p icmp -j ACCEPT

# Drop everything else
iptables -A INPUT -j DROP

# Insert rule at specific position
iptables -I INPUT 1 -p tcp --dport 8080 -j ACCEPT  # Position 1

# Delete rules
iptables -D INPUT 3                # Delete by line number
iptables -D INPUT -p tcp --dport 80 -j ACCEPT  # Delete by spec

# Flush (delete all rules) — CAREFUL
iptables -F INPUT                  # Flush INPUT chain
iptables -F                        # Flush all chains in filter table

# Default policies
iptables -P INPUT DROP             # Default drop incoming
iptables -P FORWARD DROP           # Default drop forwarded
iptables -P OUTPUT ACCEPT          # Default accept outgoing
```

### NAT with iptables

```bash
# Enable IP forwarding first
echo 1 > /proc/sys/net/ipv4/ip_forward

# MASQUERADE — dynamic SNAT (good for dynamic public IPs)
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
# All traffic going out eth0 has source IP replaced with eth0's IP

# SNAT — static source NAT (fixed public IP)
iptables -t nat -A POSTROUTING -o eth0 -s 192.168.1.0/24 \
    -j SNAT --to-source 203.0.113.1

# DNAT — destination NAT (port forwarding)
# Forward external port 80 to internal server
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 \
    -j DNAT --to-destination 192.168.1.10:80

# REDIRECT — redirect to local port (transparent proxy)
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080

# View NAT table
iptables -t nat -L -n -v

# Save rules (RHEL)
service iptables save
iptables-save > /etc/iptables/rules.v4  # Manual save
iptables-restore < /etc/iptables/rules.v4  # Restore
```

### nftables (Modern Replacement for iptables)

```bash
# nftables uses a unified syntax for all tables
nft list ruleset                   # Show all rules

# Basic nftables configuration
cat /etc/nftables.conf
```

```
#!/usr/sbin/nft -f
flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        ct state established,related accept
        iif lo accept
        tcp dport 22 accept
        tcp dport { 80, 443 } accept
        icmp type echo-request accept
    }
    
    chain forward {
        type filter hook forward priority 0; policy drop;
    }
    
    chain output {
        type filter hook output priority 0; policy accept;
    }
}

table ip nat {
    chain prerouting {
        type nat hook prerouting priority -100;
    }
    
    chain postrouting {
        type nat hook postrouting priority 100;
        oif eth0 masquerade
    }
}
```

```bash
systemctl enable --now nftables
nft -f /etc/nftables.conf

# Ad-hoc nft commands
nft add rule inet filter input tcp dport 8080 accept
nft delete rule inet filter input handle 5
```

### firewalld (Red Hat's Dynamic Firewall)

firewalld is a frontend for nftables/iptables with zone-based policies.

```bash
# Status
firewall-cmd --state
firewall-cmd --get-active-zones

# Zones
firewall-cmd --list-all                      # Default zone rules
firewall-cmd --list-all --zone=public        # Specific zone
firewall-cmd --get-zones                     # All zones

# Open ports/services
firewall-cmd --permanent --add-service=ssh
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --permanent --add-port=5000-5100/tcp  # Range

# Remove
firewall-cmd --permanent --remove-service=ssh
firewall-cmd --permanent --remove-port=8080/tcp

# Apply changes
firewall-cmd --reload

# Rich rules (complex rules)
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" \
    source address="192.168.1.0/24" service name="ssh" accept'
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" \
    source address="10.0.0.0/8" drop'

# NAT/Masquerade
firewall-cmd --permanent --add-masquerade
firewall-cmd --permanent --zone=public --add-masquerade

# Port forwarding
firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080
firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toaddr=192.168.1.10:toport=80

# Logging dropped packets
firewall-cmd --set-log-denied=all
```

### Building a Linux Router/NAT Gateway

```bash
# Step 1: Enable IP forwarding (permanent)
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# Step 2: Configure interfaces
# WAN: eth0 (public/uplink)
# LAN: eth1 (private network: 192.168.1.1/24)

# Step 3: Add NAT rule
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Step 4: Save rules
iptables-save > /etc/iptables/rules.v4
# Install iptables-persistent on Debian/Ubuntu for automatic restore

# Step 5: Add DHCP for LAN (dnsmasq or isc-dhcp-server)
# (see Section 19)

# Now clients on eth1 with gateway 192.168.1.1 can reach internet via eth0
```

### UFW — Uncomplicated Firewall (Ubuntu)

```bash
# UFW is a simplified frontend for iptables
ufw status
ufw enable
ufw disable

# Rules
ufw allow ssh
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow from 192.168.1.0/24 to any port 22
ufw deny 3389
ufw delete allow 80/tcp

# Default policies
ufw default deny incoming
ufw default allow outgoing

# Logging
ufw logging on
ufw logging high
```

### Cross-Questions & Answers

**Q1: What is the difference between REJECT and DROP in iptables?**
> **A:** **DROP** silently discards the packet — the sender gets no response and must wait for timeout. **REJECT** discards the packet AND sends an ICMP "port unreachable" or TCP RST back to the sender — they immediately know the connection failed. DROP is better against attackers (doesn't reveal anything, slows down scanning). REJECT is better for internal networks (faster failure, easier debugging).

**Q2: What is connection tracking (conntrack) and why is it important?**
> **A:** Connection tracking is a Netfilter module that tracks the state of network connections (NEW, ESTABLISHED, RELATED, INVALID). This enables **stateful** firewalling — you can allow ESTABLISHED/RELATED traffic without opening individual ports for return traffic. For example, you can drop all INPUT by default but allow `ESTABLISHED,RELATED` — this permits responses to connections you initiated while blocking unsolicited inbound traffic.

**Q3: What is the difference between SNAT and MASQUERADE?**
> **A:** Both replace the source IP of outbound packets. **SNAT** (`--to-source x.x.x.x`) uses a fixed IP — more efficient because the kernel knows the source IP without checking the interface. **MASQUERADE** dynamically looks up the outbound interface's IP at packet time — slightly slower but works when your public IP changes (DHCP, PPPoE). Use SNAT for static public IPs; MASQUERADE for dynamic IPs.

**Q4: How do you block an IP address from accessing your server?**
> **A:** 
> ```bash
> # iptables
> iptables -I INPUT -s 203.0.113.1 -j DROP
> 
> # nftables
> nft add rule inet filter input ip saddr 203.0.113.1 drop
> 
> # firewalld
> firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="203.0.113.1" drop'
> firewall-cmd --reload
> 
> # For many IPs: use ipset
> ipset create blocklist hash:ip
> ipset add blocklist 203.0.113.1
> iptables -I INPUT -m set --match-set blocklist src -j DROP
> ```

**Q5: A web server is running and listening on port 80, but external clients can't reach it. The firewall allows port 80. What else could be wrong?**
> **A:** Check: (1) **Bind address** — if nginx binds to `127.0.0.1:80` instead of `0.0.0.0:80`, it only accepts local connections (`ss -tlnp | grep :80`); (2) **Cloud/host firewall** — VMs on AWS/Azure have security groups/NSGs separate from the OS firewall; (3) **SELinux** — SELinux might block the service from binding or serving (`sealert`, `ausearch -m avc`); (4) **Port forwarding** — if behind NAT, port forwarding must be set up on the router; (5) **ISP blocking** — some ISPs block port 80 for residential connections.

---

## 🔑 Quick Reference: Essential Linux Commands

```bash
# System Info
uname -a          # Kernel info
lsb_release -a    # Distro info
hostnamectl       # Hostname and OS info
uptime            # Load and uptime
df -h             # Disk usage
du -sh /var/*     # Directory sizes
lsblk             # Block devices

# Process
ps aux            # All processes
top / htop        # Interactive monitor
kill -9 PID       # Force kill
pgrep nginx       # Get PID by name

# Network
ip addr           # IP addresses
ip route          # Routing table
ss -tuln          # Open ports
ping 8.8.8.8      # Connectivity test
traceroute HOST   # Path tracing
dig DOMAIN        # DNS lookup

# Files
find / -name "*.conf"     # Find files
grep -r "pattern" /etc/   # Search in files
tar -czvf out.tar.gz dir/ # Compress
chmod 755 file             # Permissions
chown user:group file      # Ownership

# Logs
journalctl -f           # Follow system log
journalctl -u service   # Service logs
tail -f /var/log/syslog # Traditional log
dmesg | tail            # Kernel messages
```

---

## 📊 Systemd Service States Reference

| State | Meaning |
|-------|---------|
| `active (running)` | Service is running |
| `active (exited)` | Oneshot service completed successfully |
| `active (waiting)` | Waiting for event |
| `inactive (dead)` | Not running (never started or stopped) |
| `failed` | Exited with error |
| `enabled` | Will start on boot |
| `disabled` | Won't start on boot |
| `masked` | Prevented from starting |
| `static` | Not enabled/disabled (dependency-started) |

---

*Guide compiled by Claude | Covers all major Linux fundamentals for system administration, DevOps, and security work. Applies to RHEL/CentOS/AlmaLinux/Rocky, Debian/Ubuntu, and most Linux distributions.*
