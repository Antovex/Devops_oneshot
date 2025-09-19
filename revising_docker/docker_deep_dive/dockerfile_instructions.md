# 📝 Dockerfile Instructions Deep Dive

> **Think of a Dockerfile like a detailed recipe** - each instruction is a step that transforms basic ingredients (base image) into a delicious final dish (your application container).

---

## 🎯 What You'll Master

- **Core Instructions** - FROM, RUN, COPY, CMD, and more with real examples
- **Best Practices** - Writing efficient, secure, and maintainable Dockerfiles
- **Optimization Techniques** - Reducing image size and build time
- **Security Considerations** - Avoiding common security pitfalls
- **Multi-stage Builds** - Advanced patterns for production images

---

## 🏗️ Dockerfile Structure: The Recipe Framework

Every Dockerfile follows a basic structure, like a recipe format:

```dockerfile
# 1. Base ingredients (what you start with)
FROM node:16-alpine

# 2. Preparation steps (setting up the workspace)
WORKDIR /app

# 3. Add ingredients (copy files)
COPY package*.json ./

# 4. Cooking instructions (install dependencies)
RUN npm install

# 5. Add main ingredients (application code)
COPY . .

# 6. Final touches (expose ports, set user)
EXPOSE 3000
USER node

# 7. Serving instructions (how to start)
CMD ["npm", "start"]
```

---

## 🧱 Core Instructions Deep Dive

### FROM: Choosing Your Foundation

`FROM` is like choosing your base ingredients - flour for bread, cream for soup.

```dockerfile
# Official images (recommended)
FROM node:16-alpine          # Node.js with Alpine Linux (small)
FROM ubuntu:20.04           # Full Ubuntu system
FROM python:3.9-slim        # Python with minimal Debian

# Version pinning (production best practice)
FROM node:16.14.0-alpine    # Specific version, predictable builds
FROM ubuntu:20.04           # LTS version, stable

# Multi-platform images
FROM --platform=linux/amd64 node:16-alpine  # Force specific architecture

# Scratch (empty base - advanced)
FROM scratch                # Start from nothing (for compiled languages)
```

**Choosing the right base image:**
- **Alpine** (~5MB) - Minimal, security-focused
- **Slim** (~100MB) - Debian-based, common tools included  
- **Standard** (~300MB+) - Full features, all tools available

### WORKDIR: Setting Up Your Workspace

`WORKDIR` is like clearing and organizing your kitchen counter before cooking.

```dockerfile
# Set working directory (creates if doesn't exist)
WORKDIR /app

# All subsequent commands run from this directory
# Think: "cd /app && <command>"

# Multiple WORKDIR changes are allowed
FROM node:16-alpine
WORKDIR /tmp
RUN wget https://example.com/file.tar.gz
WORKDIR /app  
RUN tar -xzf /tmp/file.tar.gz

# Relative paths work too
WORKDIR /app
WORKDIR src      # Now in /app/src
WORKDIR ../data  # Now in /app/data
```

**Best practices:**
- Always use absolute paths for WORKDIR
- Set WORKDIR early in your Dockerfile
- Use consistent paths across all your images

### RUN: The Cooking Instructions

`RUN` executes commands during the build process - like "mix ingredients for 5 minutes".

```dockerfile
# Single command
RUN npm install

# Multiple commands (creates multiple layers - inefficient)
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get clean

# Chained commands (single layer - efficient!)  
RUN apt-get update && \
    apt-get install -y curl wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Complex multi-line commands
RUN set -eux; \
    groupadd -r nodejs; \
    useradd -r -g nodejs nodejs; \
    mkdir -p /home/nodejs; \
    chown nodejs:nodejs /home/nodejs
```

**RUN vs CMD vs ENTRYPOINT:**
- **RUN** - Executes during build time (building the recipe)
- **CMD** - Default command when container starts (serving the dish)
- **ENTRYPOINT** - Fixed command, CMD becomes arguments (recipe always starts this way)

### COPY vs ADD: Adding Ingredients

Both copy files, but ADD has superpowers (that can be dangerous):

```dockerfile
# COPY - Simple, predictable (use this 95% of the time)
COPY package.json .                    # Copy single file
COPY src/ /app/src/                   # Copy directory
COPY ["file with spaces.txt", "."]    # Handle filenames with spaces

# ADD - Has special features (use carefully)
ADD https://example.com/file.tar.gz /tmp/  # Download from URL
ADD archive.tar.gz /app/                   # Auto-extract archives
ADD file.txt /app/                         # Same as COPY for local files
```

**When to use each:**
- **COPY** - For local files (safer, more explicit)
- **ADD** - Only when you need URL downloads or auto-extraction

**Security tip:** Avoid `ADD` with URLs in production - use `RUN wget` or `RUN curl` instead for better control.

### ENV: Setting the Environment

`ENV` sets environment variables - like setting the oven temperature.

```dockerfile
# Single variable
ENV NODE_ENV production

# Multiple variables
ENV NODE_ENV=production \
    PORT=3000 \
    DATABASE_URL=postgres://localhost/mydb

# Use in other instructions
ENV APP_HOME /app
WORKDIR $APP_HOME
COPY . $APP_HOME

# Runtime vs build time
ENV BUILD_TIME_VAR value    # Available at build AND runtime
ARG BUILD_ONLY_VAR         # Only available at build time
RUN echo $BUILD_ONLY_VAR   # Works during build
# Container can't see BUILD_ONLY_VAR at runtime
```

**Environment variable precedence:**
1. `docker run -e VAR=value` (highest)
2. `ENV` in Dockerfile  
3. Base image defaults (lowest)

### EXPOSE: Documenting the Doors

`EXPOSE` documents which ports your application uses - like putting a sign on your restaurant door.

```dockerfile
# Document single port
EXPOSE 3000

# Multiple ports
EXPOSE 80 443

# Specify protocol (TCP is default)
EXPOSE 53/udp
EXPOSE 80/tcp 443/tcp

# Use variables
ENV PORT=3000
EXPOSE $PORT
```

**Important:** `EXPOSE` is documentation only! It doesn't actually publish ports. Use `docker run -p` to publish.

### USER: Security First

`USER` sets which user runs your application - like choosing who's allowed in the kitchen.

```dockerfile
# Create and use non-root user (security best practice)
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser

# Switch users mid-build
USER root
RUN apt-get update && apt-get install -y sudo
USER appuser

# Use existing users (Alpine Linux)
USER node        # Most official images have non-root users
USER nobody      # Generic non-root user

# Use numeric IDs (works across all systems)
USER 1000:1000
```

**Security principle:** Never run applications as root in production containers.

### CMD vs ENTRYPOINT: The Startup Dance

The most confusing part of Dockerfiles! Think of it like restaurant service:

```dockerfile
# CMD - Default orders (can be overridden)
CMD ["npm", "start"]                    # Customer can order something else

# ENTRYPOINT - Fixed service (always happens)  
ENTRYPOINT ["docker-entrypoint.sh"]     # Always run this script first

# ENTRYPOINT + CMD - Fixed service with default options
ENTRYPOINT ["python", "app.py"]         # Always run Python app
CMD ["--port", "3000"]                  # Default arguments
```

**Practical examples:**

```dockerfile
# Web server - CMD only (simple)
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
CMD ["nginx", "-g", "daemon off;"]

# Database - ENTRYPOINT + CMD (flexible)
FROM postgres:13
ENTRYPOINT ["docker-entrypoint.sh"]     # Setup script
CMD ["postgres"]                        # Default command

# CLI tool - ENTRYPOINT only (tool behavior)
FROM alpine:latest
RUN apk add --no-cache curl
ENTRYPOINT ["curl"]                     # Always run curl
# docker run myimage https://google.com  # Becomes: curl https://google.com
```

---

## 🚀 Multi-Stage Builds: The Professional Kitchen

Multi-stage builds are like having separate prep and service kitchens - build in one stage, serve from another.

### Basic Multi-Stage Pattern

```dockerfile
# Stage 1: Build environment (the prep kitchen)
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install              # Install ALL dependencies (dev + prod)
COPY . .
RUN npm run build           # Build the application
RUN npm ci --production     # Install only production dependencies

# Stage 2: Production environment (the service kitchen)
FROM node:16-alpine AS production
WORKDIR /app

# Copy only what's needed from builder stage
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules  
COPY --from=builder /app/package.json ./

USER node
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

### Advanced Multi-Stage: Go Application

```dockerfile
# Build stage
FROM golang:1.19-alpine AS builder

# Install git (needed for go modules)
RUN apk add --no-cache git

WORKDIR /app

# Copy go modules files
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Build binary
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Final stage - minimal image
FROM alpine:latest AS production

# Add CA certificates for HTTPS requests
RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copy binary from builder
COPY --from=builder /app/main .

# Run as non-root user
RUN adduser -D -s /bin/sh user
USER user

CMD ["./main"]
```

**Result:** 1GB+ build image becomes a 10MB production image!

### Multi-Stage with External Dependencies

```dockerfile
# Base stage - common dependencies
FROM node:16-alpine AS base
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Development stage
FROM base AS development  
RUN npm ci  # Install dev dependencies too
COPY . .
CMD ["npm", "run", "dev"]

# Test stage
FROM development AS test
RUN npm run test
RUN npm run lint

# Production stage
FROM base AS production
COPY --from=development /app/dist ./dist
USER node
CMD ["npm", "start"]
```

**Build specific stages:**
```bash
# Build for development
docker build --target development -t myapp:dev .

# Build for production (default - last stage)
docker build -t myapp:prod .

# Run tests only
docker build --target test -t myapp:test .
```

---

## ⚡ Optimization Techniques

### 1. Layer Caching Strategy

Order instructions from least to most frequently changed:

```dockerfile
# ❌ Poor caching (rebuilds often)
FROM node:16-alpine
WORKDIR /app
COPY . .                    # Code changes frequently
COPY package*.json ./       # Dependencies change rarely  
RUN npm install

# ✅ Optimized caching (rebuilds only when needed)
FROM node:16-alpine
WORKDIR /app
COPY package*.json ./       # Cache this layer
RUN npm install             # Cache this layer
COPY . .                    # Only rebuild from here when code changes
```

### 2. Minimize Layer Count

```dockerfile
# ❌ Multiple layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget  
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*

# ✅ Single layer
RUN apt-get update && \
    apt-get install -y curl wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 3. Use .dockerignore

Create `.dockerignore` to exclude unnecessary files:

```dockerignore
# Version control
.git
.gitignore

# Dependencies  
node_modules
bower_components

# Logs
logs
*.log
npm-debug.log*

# Runtime data
pids
*.pid
*.seed
*.pid.lock

# Coverage directory used by tools like istanbul
coverage

# IDE files
.vscode
.idea
*.swp
*.swo

# OS files
.DS_Store
Thumbs.db

# Build outputs
dist/
build/
*.tar.gz
```

### 4. Multi-Architecture Builds

```dockerfile
# Support multiple architectures
FROM --platform=$BUILDPLATFORM node:16-alpine AS base

# Use build arguments for architecture-specific logic
ARG TARGETPLATFORM
ARG BUILDPLATFORM

RUN echo "Building for $TARGETPLATFORM on $BUILDPLATFORM"

# Install architecture-specific packages if needed
RUN case ${TARGETPLATFORM} in \
         "linux/amd64")  ARCH=amd64  ;; \
         "linux/arm64")  ARCH=arm64  ;; \
         "linux/arm/v7") ARCH=armv7  ;; \
    esac
```

---

## 🛡️ Security Best Practices

### 1. Use Specific Base Image Tags

```dockerfile
# ❌ Unpredictable (latest changes)
FROM ubuntu:latest

# ❌ Still unpredictable (rolling tag)  
FROM ubuntu:20.04

# ✅ Specific, immutable digest
FROM ubuntu:20.04@sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae288f3322
```

### 2. Run as Non-Root User

```dockerfile
# Create non-root user
RUN groupadd -r myapp && useradd -r -g myapp myapp

# Or use existing user
USER node

# Set file ownership correctly
COPY --chown=myapp:myapp . /app
```

### 3. Minimize Attack Surface

```dockerfile
# Use minimal base images
FROM alpine:3.16         # Instead of full Ubuntu

# Remove unnecessary packages
RUN apk add --no-cache curl && \
    apk del --purge curl     # Remove after use

# Don't install recommended packages (apt)
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    apt-get clean
```

### 4. Handle Secrets Securely

```dockerfile
# ❌ Never put secrets in Dockerfiles
ENV DATABASE_PASSWORD=secret123

# ✅ Use build-time secrets (Docker BuildKit)
# syntax=docker/dockerfile:1
FROM alpine
RUN --mount=type=secret,id=mypassword \
    cat /run/secrets/mypassword

# ✅ Use runtime environment variables
CMD ["sh", "-c", "echo $DATABASE_PASSWORD"]
```

**Build with secrets:**
```bash
echo "secret123" | docker build --secret id=mypassword,src=- .
```

---

## 🔧 Advanced Patterns

### Health Check Integration

```dockerfile
FROM nginx:alpine

# Copy custom health check script
COPY health-check.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/health-check.sh

# Add health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD /usr/local/bin/health-check.sh

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Init System for Containers

```dockerfile
FROM ubuntu:20.04

# Install tini (proper init system for containers)
RUN apt-get update && \
    apt-get install -y tini && \
    apt-get clean

# Use tini as entrypoint
ENTRYPOINT ["tini", "--"]

# Your application command
CMD ["my-application"]
```

### Configuration Templating

```dockerfile
FROM nginx:alpine

# Install envsubst for template processing
RUN apk add --no-cache gettext

# Copy template
COPY nginx.conf.template /etc/nginx/

# Create startup script
RUN echo '#!/bin/sh' > /docker-entrypoint.sh && \
    echo 'envsubst < /etc/nginx/nginx.conf.template > /etc/nginx/nginx.conf' >> /docker-entrypoint.sh && \
    echo 'nginx -g "daemon off;"' >> /docker-entrypoint.sh && \
    chmod +x /docker-entrypoint.sh

ENTRYPOINT ["/docker-entrypoint.sh"]
```

---

## 🎯 Hands-On Exercises

### Exercise 1: Optimize a Dockerfile

Given this inefficient Dockerfile:

```dockerfile
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y python3-pip
RUN apt-get install -y git
COPY . /app
WORKDIR /app
RUN pip3 install -r requirements.txt
EXPOSE 5000
CMD python3 app.py
```

**Your task:** Optimize it for:
- Smaller image size
- Better caching
- Security
- Fewer layers

**Solution:**
```dockerfile
# Use Python base image (smaller, optimized)
FROM python:3.9-slim

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Set working directory
WORKDIR /app

# Copy requirements first (better caching)
COPY requirements.txt .

# Install dependencies in single layer
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY --chown=appuser:appuser . .

# Switch to non-root user
USER appuser

# Document port
EXPOSE 5000

# Use exec form for better signal handling
CMD ["python", "app.py"]
```

### Exercise 2: Multi-Stage React Application

Build a React application with optimized production build:

```dockerfile
# Build stage
FROM node:16-alpine AS builder

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Build application  
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine AS production

# Copy built application
COPY --from=builder /app/build /usr/share/nginx/html

# Copy custom nginx config
COPY nginx.conf /etc/nginx/nginx.conf

# Add non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S -u 1001 -G nodejs nodejs

# Change ownership
RUN chown -R nodejs:nodejs /usr/share/nginx/html
RUN chown -R nodejs:nodejs /var/cache/nginx
RUN chown -R nodejs:nodejs /var/log/nginx
RUN chown -R nodejs:nodejs /etc/nginx/conf.d
RUN touch /var/run/nginx.pid
RUN chown -R nodejs:nodejs /var/run/nginx.pid

USER nodejs

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Exercise 3: Microservice with Health Checks

Create a Python Flask microservice with proper health checking:

```dockerfile
FROM python:3.9-slim

# Install health check dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create app user
RUN groupadd -r flask && useradd -r -g flask flask

WORKDIR /app

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY --chown=flask:flask . .

# Switch to non-root user
USER flask

# Add health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

EXPOSE 5000

CMD ["python", "app.py"]
```

---

## 📖 Key Takeaways

🏗️ **FROM chooses your foundation - pick the right base image for your needs**  
📝 **Layer order matters - put frequently changing instructions last**  
🔄 **Multi-stage builds create smaller, more secure production images**  
🛡️ **Always run as non-root user in production containers**  
⚡ **Use .dockerignore to exclude unnecessary files from build context**  
🎯 **COPY for local files, RUN for commands, CMD for default startup**

**Congratulations!** 🎉 You've mastered Docker fundamentals and advanced techniques. You're now ready to containerize any application efficiently and securely. Keep practicing with real projects and explore container orchestration with Docker Compose and Kubernetes!