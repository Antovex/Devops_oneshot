# DevOps Cheatsheets

This markdown file will contain concise cheatsheets for each DevOps topic you revise. Cheatsheets will be organized by topic and updated as you progress.

---

## Cheatsheet Index

- [Git Fundamentals](#git-fundamentals) ✅ **REVISED**
- [Linux/Unix Commands](#linuxunix-commands) ✅ **REVISED**  
- [Docker & Containerization](#docker--containerization) ✅ **REVISED**

---

## Git Fundamentals ✅ **REVISED**

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

### 🎯 **Study Status: COMPLETE**
You've completed all fundamental Git concepts including local workflows, branching, remote repositories, and team collaboration patterns. Ready for production use!

---

## 🚀 Next Topic Preparation

Ready to add your next DevOps topic here! Common next topics in DevOps learning paths:
- Docker & Containerization
- CI/CD Pipelines
- Infrastructure as Code
- Monitoring & Logging
- Cloud Platforms (AWS/Azure/GCP)

---

## Linux/Unix Commands ✅ **REVISED**

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

### 🎯 **Study Status: COMPLETE**
You've completed essential Linux/Unix commands for DevOps operations including system administration, file management, process control, network troubleshooting, and security practices. Ready for production environments!

---

## Docker & Containerization ✅ **REVISED**

### Essential Commands
```bash
# Container Lifecycle (The Big 4)
docker run -d -p 8080:80 --name myapp nginx  # Run container
docker ps                                    # List running containers  
docker logs myapp                           # Check container logs
docker exec -it myapp bash                  # Jump inside container

# Container Management
docker stop myapp                           # Stop container
docker start myapp                          # Start stopped container
docker restart myapp                        # Restart container
docker rm myapp                             # Remove stopped container
docker rm -f myapp                          # Force remove running container
```

### Image Operations
```bash
# Working with Images
docker images                               # List local images
docker pull ubuntu:20.04                   # Download image
docker build -t myapp .                     # Build from Dockerfile
docker build -t myapp:v1.0 .               # Build with tag
docker rmi myapp                            # Remove image
docker rmi -f myapp                         # Force remove image

# Image Information
docker inspect myapp                        # Detailed image info
docker history myapp                        # Show image layers
```

### Volume Management
```bash
# Named Volumes (Best for production data)
docker volume create mydata                 # Create named volume
docker volume ls                            # List volumes
docker run -v mydata:/app/data nginx        # Use named volume

# Bind Mounts (Best for development)
docker run -v ${PWD}:/app nginx             # Mount current directory
docker run -v /host/path:/container/path nginx  # Specific path

# Volume Operations
docker volume inspect mydata                # Volume details
docker volume prune                         # Remove unused volumes
```

### Network Management
```bash
# Network Operations
docker network create mynetwork             # Create custom network
docker network ls                           # List networks
docker run --network=mynetwork nginx        # Connect to network

# Container Communication
docker network connect mynetwork container  # Connect running container
docker network disconnect mynetwork container # Disconnect container
docker network inspect mynetwork            # Network details
```

### Docker Compose Essentials
```bash
# Project Management
docker-compose up -d                        # Start all services in background
docker-compose down                         # Stop and remove everything
docker-compose logs -f                      # Watch all service logs
docker-compose ps                           # List project containers

# Service Management  
docker-compose up -d --scale web=3          # Scale web service to 3 instances
docker-compose exec web bash                # Execute command in service
docker-compose restart web                  # Restart specific service
```

### Quick Cleanup Commands
```bash
# Emergency Cleanup (When things get messy)
docker system prune                         # Clean up unused resources
docker container prune                      # Remove stopped containers  
docker image prune                          # Remove dangling images
docker volume prune                         # Remove unused volumes

# Nuclear Option (Clean everything)
docker system prune -a --volumes           # Remove everything unused
```

### Dockerfile Quick Reference
```dockerfile
# Essential Instructions
FROM node:16-alpine                         # Base image (like starting recipe)
WORKDIR /app                               # Set working directory (your workspace)  
COPY package*.json ./                      # Copy files (add ingredients)
RUN npm install                            # Run build commands (prep work)
COPY . .                                   # Copy application code
EXPOSE 3000                                # Document port (which door to use)
USER node                                  # Security (run as non-root)
CMD ["npm", "start"]                       # Default command (what happens when started)
```

### Container Debugging
```bash
# Troubleshooting
docker logs container-name                  # Check application logs
docker inspect container-name               # Detailed container info
docker exec -it container-name sh           # Interactive shell access
docker port container-name                  # Check port mappings

# Resource Monitoring
docker stats                                # Real-time resource usage
docker stats container-name                 # Specific container stats
```

### Common Use Cases
```bash
# Development Database
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=dev postgres:13

# Quick Web Server
docker run -d -p 8080:80 -v ${PWD}:/usr/share/nginx/html nginx

# Temporary Test Environment
docker run --rm -it -v ${PWD}:/workspace ubuntu:20.04 bash
```

### Best Practices Reminders
- **Use specific image tags** (not `latest`) in production
- **Run as non-root user** for security
- **Use named volumes** for persistent data
- **Clean up regularly** to save disk space
- **One process per container** principle
- **Keep images small** with multi-stage builds

### Quick Safety Tips
- **Always check logs first**: `docker logs container-name`
- **Test locally before production**: Use same images
- **Backup volumes**: Before major changes
- **Use health checks**: For production containers
- **Monitor resources**: With `docker stats`

### 🎯 **Study Status: COMPLETE**
You've completed Docker fundamentals including containerization, images, volumes, networking, Docker Compose, and best practices. Ready for production use!

---

