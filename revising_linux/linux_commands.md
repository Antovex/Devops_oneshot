# Linux/Unix Essential Commands Reference ✅ **MASTERED**

A comprehensive guide to essential Linux/Unix commands for system administration and daily operations.

**� Status:** EXPERT LEVEL - Complete mastery achieved on September 18, 2025

---

## 📋 Table of Contents
- [System Administration](#-system-administration)
- [File Operations](#-file-operations)
- [Process Management](#-process-management)
- [Network Commands](#-network-commands)
- [User Management](#-user-management)
- [Package Management](#-package-management)
- [Text Processing](#-text-processing)
- [System Monitoring](#-system-monitoring)
- [Security & Permissions](#-security--permissions)
- [Archive & Compression](#-archive--compression)
- [Quick Reference](#-quick-reference)

---

## 🔧 System Administration

### System Information Commands
```bash
# Display system information
uname -a                    # Complete system information
hostnamectl                 # System hostname and details
lsb_release -a             # Distribution information
cat /etc/os-release        # OS release information
```

### System Resource Monitoring
```bash
# Memory and disk usage
free -h                     # Memory usage (human readable)
df -h                       # Disk space usage
du -sh /path/to/directory   # Directory size
lscpu                       # CPU information
lsblk                       # Block devices information
```

### Service Management (systemd)
```bash
# Service control
systemctl status service_name    # Check service status
systemctl start service_name     # Start a service
systemctl stop service_name      # Stop a service
systemctl restart service_name   # Restart a service
systemctl enable service_name    # Enable service at boot
systemctl disable service_name   # Disable service at boot

# System control
systemctl reboot                 # Reboot system
systemctl poweroff              # Shutdown system
```

---

## 📁 File Operations

### Basic File Commands
```bash
# Navigation
ls                         # List directory contents
ls -la                     # Detailed list with hidden files
pwd                        # Print working directory
cd /path/to/directory      # Change directory
cd ~                       # Go to home directory
cd -                       # Go to previous directory

# File manipulation
cp source destination      # Copy files/directories
cp -r source destination   # Copy directories recursively
mv source destination      # Move/rename files
rm filename               # Remove file
rm -rf directory          # Remove directory recursively (DANGEROUS!)
mkdir directory_name      # Create directory
mkdir -p path/to/dir      # Create nested directories
rmdir directory_name      # Remove empty directory
```

### Advanced File Operations
```bash
# File content viewing
cat filename              # Display file content
less filename            # View file with pagination
head -n 10 filename      # Show first 10 lines
tail -n 10 filename      # Show last 10 lines
tail -f filename         # Follow file changes (logs)

# File searching
find /path -name "*.txt"     # Find files by name
find /path -type f -size +1M # Find files larger than 1MB
locate filename              # Quick file location search
which command_name           # Find command location
whereis command_name         # Find command, source, manual
```

### File Linking
```bash
# Create links
ln source_file hard_link        # Create hard link
ln -s source_file symbolic_link # Create symbolic link
readlink symbolic_link          # Read symbolic link target
```

---

## ⚙️ Process Management

### Process Monitoring
```bash
# Active processes
ps aux                    # All running processes
ps -ef                    # Process tree format
pgrep process_name        # Find process ID by name
pstree                    # Display process tree

# Real-time monitoring
top                       # Dynamic process viewer
htop                      # Enhanced process viewer (if installed)
jobs                      # List active jobs
```

### Process Control
```bash
# Background/foreground control
command &                 # Run command in background
fg %1                     # Bring job 1 to foreground
bg %1                     # Send job 1 to background
nohup command &           # Run command immune to hangups

# Process termination
kill PID                  # Terminate process by PID
kill -9 PID              # Force kill process
killall process_name     # Kill all processes by name
pkill process_name       # Kill processes by name pattern
```

---

## 🌐 Network Commands

### Network Connectivity
```bash
# Basic connectivity tests
ping hostname_or_ip       # Test connectivity
ping -c 4 google.com     # Ping 4 times and stop
traceroute destination   # Trace network path
mtr destination          # Continuous traceroute

# DNS operations
nslookup domain.com      # DNS lookup
dig domain.com           # Detailed DNS information
host domain.com          # Simple DNS lookup
```

### Network Information
```bash
# Network configuration
ip addr show             # Show IP addresses (modern)
ifconfig                 # Network interface config (legacy)
ip route show            # Show routing table
netstat -tuln           # Show listening ports
ss -tuln                # Modern alternative to netstat

# Network testing
wget URL                 # Download files from web
curl -I URL             # Get HTTP headers
curl -X GET URL         # Make HTTP requests
nc -zv host port        # Test port connectivity
```

---

## 👥 User Management

### User Operations
```bash
# User information
whoami                   # Current username
id                       # User and group IDs
users                    # List logged-in users
w                        # Who is logged in and what they're doing
last                     # Login history

# User management (requires sudo)
sudo useradd username    # Add new user
sudo userdel username    # Delete user
sudo usermod -aG group user # Add user to group
sudo passwd username     # Change user password
```

### Group Management
```bash
# Group operations
groups                   # Show current user's groups
groups username         # Show specific user's groups
sudo groupadd groupname # Create new group
sudo groupdel groupname # Delete group
```

---

## 📦 Package Management

### APT (Debian/Ubuntu)
```bash
# Package operations
sudo apt update              # Update package list
sudo apt upgrade             # Upgrade installed packages
sudo apt install package    # Install package
sudo apt remove package     # Remove package
sudo apt autoremove         # Remove unused packages
sudo apt search keyword     # Search for packages
apt list --installed       # List installed packages
```

### YUM/DNF (RHEL/Fedora)
```bash
# YUM commands (RHEL/CentOS)
sudo yum update             # Update packages
sudo yum install package    # Install package
sudo yum remove package     # Remove package
sudo yum search keyword     # Search packages

# DNF commands (Fedora)
sudo dnf update             # Update packages
sudo dnf install package    # Install package
sudo dnf remove package     # Remove package
sudo dnf search keyword     # Search packages
```

### Snap Packages (Universal)
```bash
# Snap operations
sudo snap install package   # Install snap package
snap list                   # List installed snaps
sudo snap remove package    # Remove snap package
snap find keyword           # Search snap store
```

---

## 📝 Text Processing

### Text Viewing and Editing
```bash
# Text editors
nano filename            # Simple text editor
vim filename            # Vi/Vim editor
emacs filename          # Emacs editor

# Text processing
grep "pattern" file     # Search for pattern in file
grep -r "pattern" dir   # Recursive search in directory
grep -i "pattern" file  # Case-insensitive search
grep -v "pattern" file  # Invert match (exclude pattern)
```

### Advanced Text Processing
```bash
# Stream editing
sed 's/old/new/g' file      # Replace all occurrences
sed -n '5,10p' file         # Print lines 5-10
awk '{print $1}' file       # Print first column
awk -F',' '{print $2}' file # Use comma as delimiter

# Text utilities
sort file                   # Sort lines alphabetically
sort -n file               # Sort numerically
uniq file                  # Remove duplicate lines
wc -l file                 # Count lines
wc -w file                 # Count words
cut -d',' -f1 file         # Extract first field (comma-delimited)
```

---

## 📊 System Monitoring

### Resource Monitoring
```bash
# CPU and memory
top                        # Real-time system stats
htop                       # Enhanced system monitor
uptime                     # System uptime and load
vmstat 5                   # Virtual memory statistics every 5 seconds
iostat 5                   # I/O statistics every 5 seconds

# Disk usage
df -h                      # Disk space usage
du -sh directory          # Directory size
iotop                      # I/O usage by process (requires install)
```

### Log Monitoring
```bash
# System logs
journalctl                 # View systemd logs
journalctl -u service_name # View specific service logs
journalctl -f              # Follow logs in real-time
tail -f /var/log/syslog    # Follow system log

# Log locations
ls /var/log/              # Common log directory
dmesg                     # Kernel messages
```

---

## 🔒 Security & Permissions

### File Permissions
```bash
# Permission viewing
ls -l filename            # View file permissions
ls -ld directory         # View directory permissions

# Permission modification
chmod 755 filename       # Set permissions (rwxr-xr-x)
chmod u+x filename       # Add execute for user
chmod g-w filename       # Remove write for group
chmod o-r filename       # Remove read for others

# Ownership changes
sudo chown user:group file    # Change owner and group
sudo chown user file         # Change owner only
sudo chgrp group file        # Change group only
```

### Security Commands
```bash
# User switching
su - username            # Switch to another user
sudo command            # Execute command as root
sudo -u username command # Execute as specific user

# File attributes
lsattr filename         # List file attributes
sudo chattr +i filename # Make file immutable
```

---

## 🗜️ Archive & Compression

### TAR Archives
```bash
# Create archives
tar -cvf archive.tar files     # Create tar archive
tar -czvf archive.tar.gz files # Create compressed tar archive
tar -cjvf archive.tar.bz2 files # Create bzip2 compressed archive

# Extract archives
tar -xvf archive.tar          # Extract tar archive
tar -xzvf archive.tar.gz      # Extract gzipped archive
tar -xjvf archive.tar.bz2     # Extract bzip2 archive
tar -tf archive.tar           # List archive contents
```

### Other Compression
```bash
# ZIP operations
zip -r archive.zip directory  # Create zip archive
unzip archive.zip            # Extract zip archive
unzip -l archive.zip         # List zip contents

# Individual file compression
gzip filename               # Compress file (creates .gz)
gunzip filename.gz          # Decompress gzip file
```

---

## ⚡ Quick Reference

### Essential Daily Commands
| Command | Purpose | Example |
|---------|---------|---------|
| `ls -la` | List files detailed | `ls -la /home` |
| `cd` | Change directory | `cd /var/log` |
| `pwd` | Show current path | `pwd` |
| `cp -r` | Copy recursively | `cp -r src/ dest/` |
| `rm -rf` | Remove forcefully | `rm -rf temp/` |
| `chmod` | Change permissions | `chmod 755 script.sh` |
| `ps aux` | Show processes | `ps aux | grep nginx` |
| `top` | System monitor | `top` |
| `grep -r` | Search recursively | `grep -r "error" logs/` |
| `tail -f` | Follow file | `tail -f app.log` |

### Keyboard Shortcuts
| Shortcut | Function |
|----------|----------|
| `Ctrl+C` | Interrupt/cancel command |
| `Ctrl+Z` | Suspend current job |
| `Ctrl+D` | End of input/logout |
| `Ctrl+L` | Clear screen |
| `Ctrl+R` | Search command history |
| `Tab` | Auto-complete |
| `!!` | Repeat last command |
| `!n` | Repeat command number n |

### File Permission Numbers
| Number | Permission | Description |
|--------|------------|-------------|
| `7` | rwx | Read, write, execute |
| `6` | rw- | Read, write |
| `5` | r-x | Read, execute |
| `4` | r-- | Read only |
| `0` | --- | No permissions |

### Common Directories
| Directory | Purpose |
|-----------|---------|
| `/` | Root directory |
| `/home` | User home directories |
| `/etc` | Configuration files |
| `/var` | Variable data (logs, temp) |
| `/usr` | User programs |
| `/bin` | Essential commands |
| `/tmp` | Temporary files |

---

## 🎯 Best Practices

### Safety Guidelines
1. **Always verify before destructive operations**: Use `ls` before `rm`
2. **Use `-i` flag for confirmations**: `rm -i filename`
3. **Test commands on non-critical files first**
4. **Regular backups**: Before major system changes
5. **Check command syntax**: Use `man command` for help

### Security Practices
1. **Principle of least privilege**: Don't run everything as root
2. **Use sudo instead of su**: Better logging and control
3. **Regular permission audits**: Check file permissions periodically
4. **Monitor system logs**: Watch for unusual activity
5. **Keep systems updated**: Regular package updates

### Performance Tips
1. **Use `htop` over `top`**: Better interface and features
2. **Learn keyboard shortcuts**: Faster navigation
3. **Use tab completion**: Faster command entry
4. **Combine commands with pipes**: More efficient workflows
5. **Use aliases for frequent commands**: Save time

---

**🏆 Mastery Goal:** Complete understanding of Linux/Unix command ecosystem for DevOps operations - ✅ **ACHIEVED**

*Reference created: September 18, 2025*
*Status: ✅ **MASTERED** - Expert Level*