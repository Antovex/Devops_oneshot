# 💾 Volumes & Persistence Deep Dive

> **Think of Docker volumes like external hard drives** - they store your important data safely outside your computer, so even if your computer crashes, your data survives.

---

## 🎯 What You'll Master

- **Volume Types** - Named volumes, bind mounts, and tmpfs explained
- **Data Persistence** - Keeping data alive across container restarts
- **Sharing Data** - How containers can share files and databases
- **Backup & Recovery** - Protecting your valuable data
- **Performance** - Optimizing data access patterns

---

## 🔍 The Data Persistence Problem

### The Disappearing Data Issue

Imagine you're running a blog in a container. You write amazing posts, get thousands of readers, but then your container crashes. When you restart it... **all your posts are gone!** 😱

```bash
# Run a database container
docker run -d --name mydb postgres:13

# Add some important data
docker exec -it mydb psql -U postgres -c "CREATE TABLE users(name TEXT);"
docker exec -it mydb psql -U postgres -c "INSERT INTO users VALUES('Alice');"

# Container crashes or gets updated
docker stop mydb && docker rm mydb

# Start a new container - DATA IS GONE! 
docker run -d --name mydb postgres:13
docker exec -it mydb psql -U postgres -c "SELECT * FROM users;"
# ERROR: relation "users" does not exist
```

**The Problem:** Container filesystems are ephemeral - they disappear when containers are removed.

**The Solution:** Docker volumes provide persistent storage that survives container lifecycle.

---

## 🗂️ Types of Docker Storage

Think of Docker storage like different ways to store your files:

| Type | Analogy | Use Case | Persistence |
|------|---------|----------|-------------|
| **Named Volumes** | Bank vault 🏦 | Database data, app state | ✅ Survives container removal |
| **Bind Mounts** | Shared folder 📁 | Development, config files | ✅ Lives on host filesystem |
| **tmpfs Mounts** | Sticky notes 📝 | Temporary data, caches | ❌ Memory only, disappears |

---

## 🏦 Named Volumes: The Bank Vault Approach

Named volumes are like safety deposit boxes - Docker manages them for you, and they're completely separate from containers.

### Creating and Using Named Volumes

```bash
# Create a named volume (like opening a bank vault)
docker volume create my-database-data

# List your vaults
docker volume ls

# Inspect vault details  
docker volume inspect my-database-data
```

**Output shows where Docker stores your volume:**
```json
[
    {
        "CreatedAt": "2025-09-19T10:30:00Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-database-data/_data",
        "Name": "my-database-data",
        "Options": {},
        "Scope": "local"
    }
]
```

### Practical Example: Persistent Database

Let's create a PostgreSQL database that keeps its data:

```bash
# Create volume for database data
docker volume create postgres-data

# Run database with the volume mounted
docker run -d \
  --name persistent-db \
  -e POSTGRES_PASSWORD=secret \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13

# Add some important data
docker exec -it persistent-db psql -U postgres -c "
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    hire_date DATE
);

INSERT INTO employees (name, department, hire_date) VALUES
('Alice Johnson', 'Engineering', '2023-01-15'),
('Bob Smith', 'Marketing', '2023-02-20'),
('Carol Wilson', 'HR', '2023-03-10');
"

# Verify data exists
docker exec -it persistent-db psql -U postgres -c "SELECT * FROM employees;"
```

Now let's simulate a disaster and recovery:

```bash
# Disaster strikes! Container gets corrupted
docker stop persistent-db
docker rm persistent-db

# Start a new container with the same volume
docker run -d \
  --name recovered-db \
  -e POSTGRES_PASSWORD=secret \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13

# Check if data survived - it should all be there!
docker exec -it recovered-db psql -U postgres -c "SELECT * FROM employees;"
```

**🎉 Magic!** Your data survived the container replacement because it was stored in a named volume.

---

## 📁 Bind Mounts: The Shared Folder Approach

Bind mounts are like creating shortcuts to folders on your computer - you directly share a host directory with the container.

### When to Use Bind Mounts

- **Development** - Live code editing while container runs
- **Configuration** - Share config files from host  
- **Logs** - Direct access to application logs
- **Static assets** - Serve files from host filesystem

### Development Workflow Example

Let's create a live-reloading web development setup:

**Create your project:**
```bash
mkdir my-dev-project && cd my-dev-project
```

**package.json:**
```json
{
  "name": "live-dev-app",
  "version": "1.0.0",
  "scripts": {
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.18.0",
    "nodemon": "^2.0.0"
  }
}
```

**server.js:**
```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send(`
    <h1>🚀 Live Development Server</h1>
    <p>Edit server.js and see changes instantly!</p>
    <p>Current time: ${new Date().toLocaleTimeString()}</p>
  `);
});

app.listen(port, () => {
  console.log(`Dev server running at http://localhost:${port}`);
});
```

**Dockerfile:**
```dockerfile
FROM node:16-alpine
WORKDIR /app

# Install dependencies 
COPY package*.json ./
RUN npm install

# Copy source code
COPY . .

# Run development server
CMD ["npm", "run", "dev"]
```

**Run with bind mount for live development:**
```bash
# Build the image
docker build -t my-dev-app .

# Run with bind mount - your local files are now inside the container!
docker run -d \
  --name dev-container \
  -p 3000:3000 \
  -v ${PWD}:/app \
  -v /app/node_modules \
  my-dev-app

# Now edit server.js on your host machine and see instant changes!
```

**The magic:** When you edit `server.js` on your computer, nodemon inside the container detects the change and restarts the server automatically.

### Bind Mount Best Practices

```bash
# Read-only bind mount (for configuration files)
docker run -v /host/config:/app/config:ro nginx

# Specific file binding
docker run -v /host/nginx.conf:/etc/nginx/nginx.conf:ro nginx

# Multiple bind mounts
docker run \
  -v /host/data:/app/data \
  -v /host/logs:/app/logs \
  -v /host/config:/app/config:ro \
  myapp
```

---

## 📝 tmpfs Mounts: The Sticky Notes Approach

tmpfs mounts store data in memory - perfect for temporary data that doesn't need to persist.

### When to Use tmpfs

- **Sensitive data** - Passwords, tokens that shouldn't hit disk
- **Cache data** - Temporary files that improve performance  
- **Session storage** - User session data
- **Temporary processing** - Data that's processed and discarded

```bash
# Mount tmpfs for sensitive data
docker run -d \
  --name secure-app \
  --tmpfs /tmp:rw,noexec,nosuid,size=100m \
  --tmpfs /app/cache:rw,size=50m \
  myapp

# Data in /tmp and /app/cache is stored in memory only
# It disappears when container stops
```

---

## 🔄 Volume Management Operations

### Volume Lifecycle Management

```bash
# Create volumes with specific options
docker volume create \
  --driver local \
  --opt type=none \
  --opt o=bind \
  --opt device=/host/path \
  my-bind-volume

# Inspect volume usage
docker volume inspect my-volume | jq '.[0].Mountpoint'

# See which containers are using a volume
docker ps --filter volume=my-volume

# Remove unused volumes
docker volume prune

# Remove specific volume (must not be in use)
docker volume rm my-volume
```

### Sharing Volumes Between Containers

Create a multi-container application where containers share data:

```bash
# Create shared volume
docker volume create shared-data

# Container 1: Data producer
docker run -d \
  --name data-producer \
  -v shared-data:/data \
  alpine sh -c 'while true; do echo "$(date): Hello from producer" >> /data/messages.log; sleep 5; done'

# Container 2: Data consumer  
docker run -d \
  --name data-consumer \
  -v shared-data:/data \
  alpine sh -c 'while true; do echo "--- Latest messages ---"; tail -5 /data/messages.log; sleep 10; done'

# Container 3: Web server serving the data
docker run -d \
  --name web-server \
  -v shared-data:/usr/share/nginx/html \
  -p 8080:80 \
  nginx

# All containers can access the same data!
docker logs data-consumer
```

---

## 🛡️ Backup and Recovery Strategies

### Volume Backup Methods

**Method 1: Using a helper container**
```bash
# Create backup
docker run --rm \
  -v my-important-data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/backup-$(date +%Y%m%d_%H%M%S).tar.gz /data

# Restore from backup
docker run --rm \
  -v my-important-data:/data \
  -v $(pwd):/backup \
  alpine sh -c "cd /data && tar xzf /backup/backup-20250919_103000.tar.gz --strip 1"
```

**Method 2: Database-specific backup**
```bash
# PostgreSQL backup
docker exec postgres-container pg_dump -U postgres mydb > backup.sql

# PostgreSQL restore  
docker exec -i postgres-container psql -U postgres mydb < backup.sql

# MySQL backup
docker exec mysql-container mysqldump -u root -p mydb > backup.sql
```

**Method 3: Direct volume copy**
```bash
# Copy data from one volume to another
docker run --rm \
  -v source-volume:/source \
  -v destination-volume:/destination \
  alpine cp -r /source/. /destination/
```

### Automated Backup Script

Create a backup script for regular data protection:

```bash
#!/bin/bash
# backup-volumes.sh

BACKUP_DIR="/backup/docker-volumes"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p "$BACKUP_DIR"

# List of volumes to backup
VOLUMES=("postgres-data" "app-config" "user-uploads")

for volume in "${VOLUMES[@]}"; do
    echo "Backing up volume: $volume"
    
    docker run --rm \
        -v $volume:/data \
        -v $BACKUP_DIR:/backup \
        alpine tar czf /backup/${volume}_${DATE}.tar.gz /data
        
    echo "Backup completed: ${volume}_${DATE}.tar.gz"
done

# Clean up old backups (keep last 7 days)
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete

echo "Backup process completed!"
```

Make it executable and schedule it:
```bash
chmod +x backup-volumes.sh

# Run daily at 2 AM (add to crontab)
# 0 2 * * * /path/to/backup-volumes.sh
```

---

## ⚡ Performance Optimization

### Volume Performance Characteristics

| Storage Type | Performance | Use Case |
|-------------|------------|----------|
| **Named Volumes** | Good | Production databases, persistent app data |
| **Bind Mounts** | Varies by OS | Development, configuration files |
| **tmpfs** | Excellent | Caches, temporary data, sensitive info |

### Performance Best Practices

**1. Use appropriate storage for your use case:**
```bash
# High-performance temporary storage
--tmpfs /app/cache

# Good persistent storage  
-v db-data:/var/lib/mysql

# Development convenience
-v ${PWD}:/app
```

**2. Optimize database containers:**
```bash
# PostgreSQL performance tuning
docker run -d \
  --name fast-postgres \
  -v postgres-data:/var/lib/postgresql/data \
  -v postgres-config:/etc/postgresql \
  --tmpfs /tmp:rw,noexec,nosuid \
  --shm-size=1g \
  postgres:13

# MySQL performance tuning
docker run -d \
  --name fast-mysql \
  -v mysql-data:/var/lib/mysql \
  --tmpfs /tmp:rw,noexec,nosuid \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0 --innodb-buffer-pool-size=1G
```

**3. Monitor volume usage:**
```bash
# Check volume sizes
docker system df -v

# Monitor I/O performance
docker stats --format "table {{.Container}}\t{{.BlockIO}}"
```

---

## 🛠️ Troubleshooting Guide

### Common Volume Issues

**❌ Problem: Permission denied when accessing bind mount**
```bash
# Check file permissions
ls -la /host/path

# Fix permissions
sudo chown -R $USER:$USER /host/path
chmod -R 755 /host/path

# Or run container with correct user ID
docker run --user $(id -u):$(id -g) -v /host/path:/container/path myapp
```

**❌ Problem: Volume appears empty in container**
```bash
# Check if volume is correctly mounted
docker inspect container-name | jq '.Mounts'

# Verify mount point exists inside container
docker exec container-name ls -la /mount/point

# Check volume exists and has data
docker volume inspect volume-name
```

**❌ Problem: Out of disk space with volumes**
```bash
# Check volume disk usage
docker system df -v

# Clean up unused volumes
docker volume prune

# Remove specific large volumes
docker volume rm large-unused-volume
```

**❌ Problem: Slow performance with bind mounts**
```bash
# On macOS, use cached or delegated options
-v /host/path:/container/path:cached

# Consider using named volumes for better performance
docker volume create fast-volume
-v fast-volume:/app/data
```

---

## 🎯 Hands-On Exercises

### Exercise 1: Multi-Container WordPress with Persistent Data

Set up a complete WordPress installation with persistent data:

```bash
# Create volumes
docker volume create wordpress-db
docker volume create wordpress-files

# Run MySQL database
docker run -d \
  --name wordpress-mysql \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wpuser \
  -e MYSQL_PASSWORD=wppass \
  -v wordpress-db:/var/lib/mysql \
  mysql:5.7

# Run WordPress
docker run -d \
  --name wordpress-site \
  --link wordpress-mysql:mysql \
  -e WORDPRESS_DB_HOST=mysql \
  -e WORDPRESS_DB_USER=wpuser \
  -e WORDPRESS_DB_PASSWORD=wppass \
  -e WORDPRESS_DB_NAME=wordpress \
  -v wordpress-files:/var/www/html \
  -p 8080:80 \
  wordpress:latest

# Visit http://localhost:8080 and set up WordPress
# Then test persistence by removing containers and recreating them
```

### Exercise 2: Development Environment with Hot Reloading

Create a Python Flask development environment with live code reloading:

```bash
mkdir flask-dev && cd flask-dev
```

**app.py:**
```python
from flask import Flask
import os

app = Flask(__name__)

@app.route('/')
def hello():
    return f"""
    <h1>🐍 Flask Development Server</h1>
    <p>Edit this file and see changes instantly!</p>
    <p>Environment: {os.getenv('FLASK_ENV', 'production')}</p>
    """

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

**requirements.txt:**
```
Flask==2.3.2
```

**Dockerfile:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

```bash
# Build and run with live reloading
docker build -t flask-dev .

docker run -d \
  --name flask-dev-container \
  -p 5000:5000 \
  -v ${PWD}:/app \
  -e FLASK_ENV=development \
  flask-dev

# Now edit app.py and see changes instantly at http://localhost:5000
```

---

## 📖 Key Takeaways

💾 **Named volumes are the best choice for production data persistence**  
📁 **Bind mounts are perfect for development workflows**  
📝 **tmpfs mounts provide secure, fast temporary storage**  
🔄 **Regular backups are essential for data protection**  
⚡ **Choose the right storage type for optimal performance**  
🛡️ **Always test your data persistence and recovery procedures**

**Next up:** Ready to connect your containers? Check out [Networking](networking.md) to learn how containers communicate with each other and the outside world!