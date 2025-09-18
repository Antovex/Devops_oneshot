# DevOps Cheatsheets

This markdown file will contain concise cheatsheets for each DevOps topic you revise. Cheatsheets will be organized by topic and updated as you progress.

---

## Cheatsheet Index

- [Git Fundamentals](#git-fundamentals) ✅ **MASTERED**
- [Linux/Unix Commands](#linuxunix-commands) ✅ **MASTERED**

---

## Git Fundamentals ✅ **MASTERED**

### Essential Commands
```bash
# Setup
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Local Workflow
git init                    # Initialize repository
git status                  # Check status
git add .                   # Stage all files
git commit -m "message"     # Commit changes
git log --oneline           # View history

# Branching
git branch feature-name     # Create branch
git checkout feature-name   # Switch to branch
git checkout -b new-feature # Create and switch to branch

# Remote Repository
git branch -M main          # Rename branch to main
git remote add origin <url> # Link to remote repository
git push -u origin main     # Push and set upstream
git push                    # Subsequent pushes
git pull                    # Pull latest changes

# Navigation
git checkout <commit-hash>  # Go to specific commit
git checkout main           # Return to main branch
```

### Feature Branch Workflow
```bash
git pull                           # Get latest changes
git checkout -b feature-login      # Create feature branch
git add .                          # Stage changes
git commit -m "add login feature"  # Commit with good message
git push -u origin feature-login   # Push feature branch
```

### Commit Message Convention
**Rule:** "If applied to the codebase, this commit will ___"

Examples:
- `git commit -m "add user authentication"`
- `git commit -m "fix navigation menu bug"`
- `git commit -m "update API documentation"`

### Workflow Summary
- **Local:** `init` → `add` → `commit` → `log`
- **Branching:** `checkout -b` → develop → `add` → `commit` → `push -u`
- **Remote:** `branch -M main` → `remote add origin` → `push -u origin main`

### Quick Reference
- **Working Directory** → `git add` → **Staging Area** → `git commit` → **Local Repository** → `git push` → **Remote Repository**
- Always run `git status` before committing
- Pull before push to avoid conflicts
- Use descriptive branch names (`feature-login`, `fix-payment`)
- Write commit messages that complete: "If applied, this commit will ___"

### 🎯 **Mastery Status: COMPLETE**
You've mastered all fundamental Git concepts including local workflows, branching, remote repositories, and team collaboration patterns. Ready for production use!

---

## 🚀 Next Topic Preparation

Ready to add your next DevOps topic here! Common next topics in DevOps learning paths:
- Docker & Containerization
- CI/CD Pipelines
- Infrastructure as Code
- Monitoring & Logging
- Cloud Platforms (AWS/Azure/GCP)

---

## Linux/Unix Commands ✅ **MASTERED**

### Essential Daily Commands
```bash
# System Information
uname -a                    # System information
free -h                     # Memory usage
df -h                       # Disk space
lscpu                       # CPU information

# File Operations
ls -la                      # List files detailed
cp -r source dest           # Copy directories
mv old_name new_name        # Move/rename
rm -rf directory            # Remove recursively
chmod 755 file              # Set permissions

# Process Management
ps aux                      # Show all processes
top                         # Real-time process monitor
kill -9 PID                 # Force kill process
jobs                        # List active jobs

# Network Commands
ping -c 4 host              # Test connectivity
curl -I URL                 # Get HTTP headers
netstat -tuln               # Show listening ports
wget URL                    # Download files

# Text Processing
grep -r "pattern" dir       # Recursive search
sed 's/old/new/g' file      # Find and replace
awk '{print $1}' file       # Print first column
tail -f logfile             # Follow log file
```

### File Permissions Quick Reference
```bash
# Permission Numbers
chmod 755 file              # rwxr-xr-x (owner: all, group/others: read+execute)
chmod 644 file              # rw-r--r-- (owner: read+write, others: read)
chmod 600 file              # rw------- (owner: read+write, others: none)

# Permission Letters
chmod u+x file              # Add execute for user
chmod g-w file              # Remove write for group
chmod o-r file              # Remove read for others
```

### Package Management
```bash
# APT (Debian/Ubuntu)
sudo apt update             # Update package list
sudo apt install package   # Install package
sudo apt remove package    # Remove package

# YUM/DNF (RHEL/Fedora)
sudo yum install package    # Install package (RHEL/CentOS)
sudo dnf install package    # Install package (Fedora)
```

### System Monitoring
```bash
# Resource Monitoring
htop                        # Enhanced process viewer
iostat 5                    # I/O stats every 5 seconds
du -sh directory            # Directory size
uptime                      # System uptime and load

# Log Monitoring
journalctl -f               # Follow systemd logs
tail -f /var/log/syslog     # Follow system log
dmesg                       # Kernel messages
```

### Archive & Compression
```bash
# TAR operations
tar -czvf archive.tar.gz dir    # Create compressed archive
tar -xzvf archive.tar.gz        # Extract compressed archive
tar -tf archive.tar             # List archive contents

# ZIP operations
zip -r archive.zip directory    # Create zip archive
unzip archive.zip               # Extract zip archive
```

### Network Troubleshooting
```bash
# Connectivity Testing
ping google.com             # Basic connectivity test
traceroute destination      # Trace network path
nslookup domain.com         # DNS lookup
dig domain.com              # Detailed DNS info

# Port Testing
nc -zv host port            # Test port connectivity
telnet host port            # Manual port test
```

### Quick Safety Reminders
- **Always test destructive commands**: Use `ls` before `rm`
- **Use `-i` for confirmation**: `rm -i filename`
- **Don't run everything as root**: Use `sudo` when needed
- **Regular backups**: Before major changes
- **Check man pages**: `man command` for help

### 🎯 **Learning Status: COMPLETE**
You've mastered essential Linux/Unix commands for DevOps operations including system administration, file management, process control, network troubleshooting, and security practices. Ready for production environments!

---

