# Docker & Containerization Guide 🐳

> **Think of Docker as a shipping company for your applications** - it packages everything your app needs into standardized containers that can ship anywhere and run exactly the same way.

**🎯 Status:** LEARNING IN PROGRESS - Started September 19, 2025

---

## �️ Your Docker Learning Journey

```
📦 Fundamentals → 🏗️ Building → 🌐 Networking → 🚀 Orchestration
     ↓               ↓             ↓              ↓
  Containers      Dockerfiles    Volumes &      Docker
  & Images                       Networks       Compose
```

## 📋 Quick Navigation
- [🐳 Docker Fundamentals](#-docker-fundamentals) - *What Docker is and why it matters*
- [🚀 Quick Start Guide](#-quick-start-guide) - *Get hands-on in 5 minutes*
- [🏗️ Building Your First App](#️-building-your-first-app) - *Real example walkthrough*
- [🎯 Common Use Cases](#-common-use-cases) - *When and why to use Docker*
- [📚 Deep Dive Resources](#-deep-dive-resources) - *Advanced topics and guides*
- [⚡ Quick Reference](#-quick-reference) - *Commands you'll use daily*

---

## 🐳 Docker Fundamentals

### The Big Picture: Why Docker Exists

**The Problem:** *"But it works on my machine!"*  
Ever spent hours debugging why your app works perfectly on your laptop but crashes on the server? Docker solves this.

**The Solution:** *Standardized shipping containers for software*  
Just like shipping containers revolutionized global trade by creating a standard way to transport goods, Docker containers standardize how we package and run software.

### Core Concepts (With Real-World Analogies)

#### 🏗️ **Docker Images** → *Recipe/Blueprint*
```bash
# Think: A recipe for chocolate chip cookies
# - Lists all ingredients (dependencies)
# - Provides step-by-step instructions
# - Anyone can use it to make identical cookies
docker pull nginx:latest  # Download the "nginx recipe"
```

#### � **Containers** → *Baked Cookies/Running Applications*
```bash
# Think: The actual cookies made from the recipe
# - Created from the image (recipe)
# - Each cookie is independent
# - You can make many from one recipe
docker run nginx:latest  # "Bake" a container from the image
```

#### 💾 **Volumes** → *External Hard Drive*
```bash
# Think: External storage that survives computer crashes
# - Data persists even when container stops
# - Can be shared between containers
# - Like plugging in a USB drive
docker run -v /my-data:/app/data nginx
```

#### 🌐 **Networks** → *Office WiFi Network*
```bash
# Think: How containers talk to each other
# - Like devices on your home WiFi
# - Containers can find and communicate
# - Isolated from other networks
docker network create my-app-network
```

---

## 🚀 Quick Start Guide

### Step 1: Verify Your Setup
```bash
# Like checking if your oven is working before baking
docker --version        # Should show Docker version
docker run hello-world  # The "Hello World" of Docker
```

### Step 2: Run Your First Real Container
```bash
# Run a web server (like setting up a lemonade stand)
docker run -d -p 8080:80 --name my-webserver nginx

# Visit http://localhost:8080 - You're now serving a website!
```

### Step 3: Peek Inside Your Container
```bash
# Like looking inside your running lemonade stand
docker ps                    # See what's running
docker logs my-webserver     # Check the activity log
docker exec -it my-webserver bash  # Step inside and look around
```

### Step 4: Clean Up
```bash
# Close the lemonade stand when done
docker stop my-webserver
docker rm my-webserver
```

---

## 🏗️ Building Your First App

Let's build a simple "Hello DevOps" web application from scratch!

### The Scenario
You're building a personal website that says "Hello from DevOps!" - let's containerize it.

### Step 1: Create Your App
```bash
# Create project directory (your workspace)
mkdir hello-devops && cd hello-devops
```

Create `index.html`:
```html
<!DOCTYPE html>
<html>
<head><title>Hello DevOps</title></head>
<body>
    <h1>🚀 Hello from DevOps!</h1>
    <p>This website is running in a Docker container!</p>
    <p>Built on: <span id="date"></span></p>
    <script>
        document.getElementById('date').textContent = new Date().toLocaleDateString();
    </script>
</body>
</html>
```

### Step 2: Write Your "Recipe" (Dockerfile)
Create `Dockerfile`:
```dockerfile
# Start with a base image (like starting with pre-made cookie dough)
FROM nginx:alpine

# Copy your website into the container (add your special ingredients)
COPY index.html /usr/share/nginx/html/

# Expose port 80 (tell people which door to knock on)
EXPOSE 80

# The container will automatically start nginx (the oven timer goes off)
```

### Step 3: Build Your Image
```bash
# Build your custom "recipe"
docker build -t hello-devops .

# See your new image
docker images | grep hello-devops
```

### Step 4: Run Your Containerized App
```bash
# Start your application
docker run -d -p 3000:80 --name my-hello-app hello-devops

# Visit http://localhost:3000 - Your containerized app is live!
```

**🎉 Congratulations!** You just:
1. Built a web application
2. Containerized it with Docker
3. Made it portable and reproducible

---

## 🎯 Common Use Cases

### 🔧 **Development Environment**
```bash
# Spin up a database for development
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=dev postgres:13
# No PostgreSQL installation needed on your machine!
```

### 🧪 **Testing**
```bash
# Test your app against different Node.js versions
docker run -v $(pwd):/app -w /app node:14 npm test
docker run -v $(pwd):/app -w /app node:16 npm test
```

### 🚀 **Deployment**
```bash
# Same container runs identically everywhere
docker run -d -p 80:3000 myapp:production  # Local testing
# (Same command works on AWS, Azure, Google Cloud...)
```

---

## 📚 Deep Dive Resources

Ready to dive deeper? Check out our specialized guides:

- **[🏗️ Images & Containers Deep Dive](docker_deep_dive/images_and_containers.md)**  
  *Master image layers, container lifecycle, and optimization*

- **[💾 Volumes & Persistence](docker_deep_dive/volumes_and_persistence.md)**  
  *Data management, backup strategies, and storage patterns*

- **[🌐 Networking](docker_deep_dive/networking.md)**  
  *Container communication, custom networks, and security*

- **[� Dockerfile Instructions](docker_deep_dive/dockerfile_instructions.md)**  
  *Master every instruction, best practices, and optimization*

---

## ⚡ Quick Reference

### 🚀 Essential Commands You'll Use Daily

#### Container Operations
```bash
# The Big 4 - Master these first
docker run -d -p 8080:80 --name myapp nginx  # Run container
docker ps                                    # List running containers  
docker logs myapp                           # Check container logs
docker exec -it myapp bash                  # Jump inside container

# Lifecycle management
docker stop myapp && docker rm myapp        # Stop and remove
docker restart myapp                        # Restart container
```

#### Image Operations  
```bash
# Working with images
docker images                               # List local images
docker pull ubuntu:20.04                   # Download image
docker build -t myapp .                     # Build from Dockerfile
docker rmi myapp                            # Remove image
```

#### Quick Cleanup
```bash
# When things get messy
docker system prune                         # Clean up unused resources
docker container prune                      # Remove stopped containers  
docker image prune                          # Remove dangling images
```

### 🔧 Dockerfile Cheat Sheet
```dockerfile
FROM nginx:alpine               # Base image (like starting recipe)
WORKDIR /app                     # Set working directory (your workspace)  
COPY . .                         # Copy files (add ingredients)
RUN npm install                  # Run build commands (prep work)
EXPOSE 3000                      # Document port (which door to use)
CMD ["npm", "start"]             # Default command (what happens when started)
```

### 🌐 Docker Compose Essentials
```bash
# Project management made easy
docker-compose up -d             # Start all services in background
docker-compose logs -f           # Watch all service logs
docker-compose down              # Stop everything cleanly
```

---

## 🆘 Troubleshooting Quick Fixes

### Common Issues & Solutions

**🔥 Container won't start?**
```bash
docker logs container-name      # Check what went wrong
docker run -it image-name bash  # Debug interactively
```

**🐌 Image build too slow?**
```bash
docker build --no-cache .       # Force fresh build
# Add .dockerignore file to exclude large directories
```

**💾 Out of disk space?**
```bash
docker system df                # See Docker disk usage
docker system prune -a          # Nuclear cleanup option
```

**🌐 Can't reach your container?**
```bash
docker port container-name      # Check port mappings
docker inspect container-name | grep IPAddress  # Get container IP
```

---

## 🎓 Next Steps

**Ready to level up?** Here's your learning path:

1. **Master the basics** → Work through this guide's examples
2. **Build real projects** → Containerize your existing applications  
3. **Learn orchestration** → Explore Docker Compose and Kubernetes
4. **Go deeper** → Check out our [Deep Dive Resources](#-deep-dive-resources)

### 📖 Official Resources
- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/) - Browse thousands of pre-built images
- [Docker Compose Documentation](https://docs.docker.com/compose/)

---

**🎉 Happy Containerizing!** Remember: Every expert was once a beginner. Start small, practice often, and don't be afraid to break things in your development environment.

> *"Docker is like learning to drive - intimidating at first, but once you get it, you can't imagine working without it."*

### Security Best Practices
- Use official base images from trusted sources
- Run containers as non-root users
- Keep images updated and scan for vulnerabilities
- Use multi-stage builds to reduce image size
- Don't include secrets in images
- Limit container resources

### Performance Best Practices
- Use `.dockerignore` to exclude unnecessary files
- Leverage Docker layer caching
- Minimize number of layers
- Use Alpine-based images for smaller footprint
- Clean up package caches in same RUN command

```dockerfile
# Good practice example
FROM node:16-alpine
RUN apk add --no-cache git && \
    npm install -g npm@latest && \
    apk del git                    # Clean up in same layer
```

---

## 🔍 Troubleshooting

### Common Issues
```bash
# Debug container issues
docker logs container_name              # Check logs
docker exec -it container_name sh      # Get shell access
docker inspect container_name          # Detailed container info

# Debug networking issues
docker network ls                      # List networks
docker port container_name             # Show port mappings
docker exec container_name netstat -tlnp # Check listening ports

# Debug storage issues
docker system df                       # Disk usage
docker volume ls                       # List volumes
docker system prune                    # Clean up space

# Container won't start
docker events                          # Monitor Docker events
docker run --rm -it image_name sh      # Test image manually
```

### Performance Monitoring
```bash
# Monitor resource usage
docker stats                           # Real-time stats
docker stats --no-stream              # One-time snapshot
docker system df                       # Disk usage breakdown
docker system events                   # Monitor system events

# Container process monitoring
docker top container_name              # Process list
docker exec container_name ps aux      # Processes inside container
```

---

## ⚡ Quick Reference

### Essential Commands
| Command | Purpose | Example |
|---------|---------|---------|
| `docker run` | Create and run container | `docker run -d -p 80:80 nginx` |
| `docker ps` | List containers | `docker ps -a` |
| `docker images` | List images | `docker images` |
| `docker build` | Build image | `docker build -t myapp .` |
| `docker pull` | Download image | `docker pull ubuntu:20.04` |
| `docker push` | Upload image | `docker push myrepo/myapp` |
| `docker exec` | Execute in container | `docker exec -it web bash` |
| `docker logs` | View logs | `docker logs -f container` |
| `docker stop` | Stop container | `docker stop webapp` |
| `docker rm` | Remove container | `docker rm webapp` |

### Docker Run Options
| Option | Purpose | Example |
|--------|---------|---------|
| `-d` | Detached mode | `docker run -d nginx` |
| `-it` | Interactive terminal | `docker run -it ubuntu bash` |
| `-p` | Port mapping | `docker run -p 8080:80 nginx` |
| `-v` | Volume mount | `docker run -v /data:/app/data nginx` |
| `-e` | Environment variable | `docker run -e NODE_ENV=prod myapp` |
| `--name` | Container name | `docker run --name web nginx` |
| `--rm` | Auto-remove when stopped | `docker run --rm test-image` |
| `--network` | Network connection | `docker run --network mynet nginx` |

### Dockerfile Instructions
| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image | `FROM node:16-alpine` |
| `WORKDIR` | Working directory | `WORKDIR /app` |
| `COPY` | Copy files | `COPY . .` |
| `RUN` | Execute command | `RUN npm install` |
| `EXPOSE` | Expose port | `EXPOSE 3000` |
| `CMD` | Default command | `CMD ["npm", "start"]` |
| `ENV` | Environment variable | `ENV NODE_ENV=production` |
| `VOLUME` | Volume mount point | `VOLUME ["/data"]` |

### Common Port Mappings
| Service | Container Port | Common Host Port |
|---------|----------------|------------------|
| HTTP | 80 | 8080, 80 |
| HTTPS | 443 | 8443, 443 |
| MySQL | 3306 | 3306 |
| PostgreSQL | 5432 | 5432 |
| Redis | 6379 | 6379 |
| MongoDB | 27017 | 27017 |
| Node.js | 3000 | 3000, 8080 |
| Nginx | 80 | 8080, 80 |

---

## 🎯 Learning Exercises

### Beginner Exercises
1. **Hello Docker**: Run your first container with `hello-world`
2. **Interactive Container**: Start Ubuntu container with interactive bash
3. **Port Mapping**: Run nginx and access it from your browser
4. **Volume Mount**: Mount a local directory in a container
5. **Environment Variables**: Pass configuration to a container

### Intermediate Exercises
1. **Custom Dockerfile**: Create a Dockerfile for a simple web app
2. **Multi-container App**: Use docker-compose for web app + database
3. **Data Persistence**: Implement persistent data storage with volumes
4. **Container Networking**: Connect multiple containers
5. **Image Optimization**: Reduce image size using multi-stage builds

### Advanced Exercises
1. **CI/CD Integration**: Build and push images in a pipeline
2. **Health Checks**: Implement container health monitoring
3. **Resource Limits**: Configure memory and CPU constraints
4. **Security Hardening**: Run containers with least privileges
5. **Orchestration**: Deploy multi-container applications

---

## 🎯 Best Practices Summary

### Development Workflow
1. **Start with official base images**
2. **Use .dockerignore files**
3. **Implement health checks**
4. **Tag images meaningfully**
5. **Test containers locally before pushing**

### Production Considerations
1. **Use specific image tags, not 'latest'**
2. **Implement proper logging**
3. **Monitor resource usage**
4. **Regular security updates**
5. **Backup persistent data**

### Security Guidelines
1. **Run as non-root user**
2. **Scan images for vulnerabilities**
3. **Use secrets management**
4. **Limit network exposure**
5. **Keep minimal image footprint**

---

**🏆 Learning Goal:** Complete understanding of Docker containerization for DevOps workflows - 📚 **IN PROGRESS**

*Reference created: September 19, 2025*
*Status: Learning in Progress*