# 🌐 Docker Networking Deep Dive

> **Think of Docker networking like office building WiFi** - containers are like employees who need to communicate with each other, access the internet, and sometimes talk to people outside the building.

---

## 🎯 What You'll Master

- **Network Types** - Bridge, host, overlay, and none networks explained
- **Container Communication** - How containers find and talk to each other
- **Port Management** - Exposing services to the outside world
- **Network Security** - Isolating and protecting your applications
- **Multi-Host Networking** - Connecting containers across different machines

---

## 🏢 The Office Building Analogy

Imagine your Docker host is like an office building:

| Network Type | Office Analogy | Real Docker Use |
|-------------|----------------|-----------------|
| **Bridge** | Departmental WiFi 🏬 | Default container networking |
| **Host** | Using building's main internet 🌐 | Direct host network access |
| **None** | Isolation room 🔇 | No network access |
| **Custom** | Private conference room network 🏠 | Isolated application networks |

---

## 🌉 Bridge Networks: The Default Office WiFi

Bridge networks are like departmental WiFi - containers can talk to each other and access the internet, but they're isolated from the host network.

### Understanding the Default Bridge

When you run a container without specifying a network, it joins the default bridge network:

```bash
# Run containers on default bridge
docker run -d --name web1 nginx
docker run -d --name web2 nginx

# See the default bridge network
docker network ls
# NETWORK ID     NAME      DRIVER    SCOPE
# abc123456789   bridge    bridge    local

# Inspect the default bridge
docker network inspect bridge
```

**What you'll see:**
- Each container gets an IP address (like 172.17.0.2, 172.17.0.3)
- Containers can ping each other by IP
- But they can't find each other by name!

### The Name Resolution Problem

```bash
# This works (IP address)
docker exec web1 ping 172.17.0.3

# This fails on default bridge (name resolution)
docker exec web1 ping web2
# ping: web2: Name or service not known
```

### Custom Bridge Networks: The Solution

Custom bridge networks provide automatic name resolution:

```bash
# Create a custom bridge network
docker network create --driver bridge my-app-network

# Run containers on the custom network
docker run -d --name database --network my-app-network postgres:13
docker run -d --name backend --network my-app-network my-api:latest
docker run -d --name frontend --network my-app-network my-web:latest

# Now containers can find each other by name!
docker exec frontend ping database    # Works! ✅
docker exec backend ping database     # Works! ✅
```

### Practical Example: Three-Tier Web Application

Let's build a complete web application with proper networking:

**1. Create the network:**
```bash
docker network create webapp-network
```

**2. Start the database:**
```bash
docker run -d \
  --name webapp-db \
  --network webapp-network \
  -e POSTGRES_DB=myapp \
  -e POSTGRES_USER=appuser \
  -e POSTGRES_PASSWORD=secret \
  postgres:13
```

**3. Start the API backend:**

**api/app.js:**
```javascript
const express = require('express');
const { Pool } = require('pg');

const app = express();
const pool = new Pool({
  host: 'webapp-db',  // Container name becomes hostname!
  user: 'appuser',
  password: 'secret',
  database: 'myapp',
  port: 5432,
});

app.get('/users', async (req, res) => {
  try {
    const result = await pool.query('SELECT * FROM users');
    res.json(result.rows);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.listen(3000, () => {
  console.log('API server running on port 3000');
});
```

```bash
# Build and run API
docker build -t my-api api/
docker run -d \
  --name webapp-api \
  --network webapp-network \
  my-api
```

**4. Start the frontend:**

**frontend/nginx.conf:**
```nginx
server {
    listen 80;
    
    location /api/ {
        proxy_pass http://webapp-api:3000/;  # Container name!
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
```

```bash
# Build and run frontend with external access
docker build -t my-frontend frontend/
docker run -d \
  --name webapp-frontend \
  --network webapp-network \
  -p 8080:80 \
  my-frontend

# Visit http://localhost:8080 - full application is running!
```

**The magic:** Each container can reach others by name (`webapp-db`, `webapp-api`), but only the frontend is exposed to the host via port mapping.

---

## 🌐 Host Networking: Direct Building Internet Access

Host networking removes the network isolation - the container uses the host's network directly.

```bash
# Run with host networking
docker run -d --network host nginx

# Container binds directly to host port 80
# No port mapping needed - nginx is now available at localhost:80
```

### When to Use Host Networking

**✅ Good for:**
- High-performance networking applications
- Network monitoring tools
- Development/debugging scenarios

**❌ Avoid for:**
- Production web applications (security risk)
- Multiple containers needing the same port
- Portable applications that need to run anywhere

```bash
# Network monitoring tool example
docker run -d \
  --name network-monitor \
  --network host \
  --privileged \
  monitoring-tool

# Can see all host network interfaces and traffic
```

---

## 🔇 None Networking: The Isolation Room

Containers with no network access - useful for batch processing or security-sensitive tasks.

```bash
# Run with no network
docker run -d --network none my-batch-processor

# Container can only access its own filesystem
# No network communication possible
```

---

## 🔌 Port Management and Exposure

### Understanding Port Mapping

Port mapping is like having a receptionist redirect calls:

```bash
# Map host port 8080 to container port 80
docker run -d -p 8080:80 nginx
# "When someone calls 8080, redirect to container's 80"

# Map multiple ports
docker run -d \
  -p 8080:80 \
  -p 8443:443 \
  nginx

# Let Docker choose the host port
docker run -d -P nginx  # Maps to random available ports

# Check port mappings
docker port container-name
```

### Advanced Port Configuration

```bash
# Bind to specific interface
docker run -d -p 127.0.0.1:8080:80 nginx  # Only localhost

# Multiple containers, different ports
docker run -d -p 8081:80 --name web1 nginx
docker run -d -p 8082:80 --name web2 nginx

# UDP ports
docker run -d -p 5353:53/udp my-dns-server
```

### Service Discovery Example

Create a load-balanced setup with multiple backend instances:

```bash
# Create network
docker network create lb-network

# Start multiple backend instances
for i in {1..3}; do
  docker run -d \
    --name backend-$i \
    --network lb-network \
    my-backend-service
done

# Start load balancer that discovers backends
docker run -d \
  --name load-balancer \
  --network lb-network \
  -p 80:80 \
  nginx  # with config to proxy to backend-1, backend-2, backend-3
```

---

## 🛡️ Network Security and Isolation

### Network Segmentation

Isolate different parts of your application:

```bash
# Frontend network (public-facing)
docker network create frontend-net

# Backend network (internal only)
docker network create backend-net

# Database network (most restricted)
docker network create database-net

# Web server - only on frontend
docker run -d \
  --name web \
  --network frontend-net \
  -p 80:80 \
  nginx

# API server - bridge between frontend and backend
docker run -d \
  --name api \
  my-api

# Connect API to both networks
docker network connect frontend-net api
docker network connect backend-net api

# Database - only on backend network
docker run -d \
  --name db \
  --network backend-net \
  postgres:13
```

**Result:** Web can only talk to API, API can talk to both web and database, database is completely isolated from web.

### Firewall Rules with iptables

Docker automatically creates iptables rules, but you can add custom ones:

```bash
# Block specific traffic
iptables -I DOCKER-USER -s 192.168.1.0/24 -d 172.17.0.0/16 -j DROP

# Allow only specific ports from specific sources
iptables -I DOCKER-USER \
  -s 192.168.1.100 \
  -d 172.17.0.0/16 \
  -p tcp --dport 80 \
  -j ACCEPT
```

---

## 🌍 Multi-Host Networking with Overlay Networks

Overlay networks let containers on different machines talk to each other - like having WiFi that works across multiple office buildings.

### Setting Up Docker Swarm for Overlay Networks

```bash
# On manager node
docker swarm init --advertise-addr <manager-ip>

# On worker nodes (use token from init command)
docker swarm join --token <token> <manager-ip>:2377

# Create overlay network
docker network create --driver overlay my-overlay-net

# Deploy service across multiple nodes
docker service create \
  --name web-service \
  --network my-overlay-net \
  --replicas 3 \
  nginx

# Containers can communicate across different physical machines!
```

### Multi-Host Example: Distributed Database

```bash
# Create overlay network
docker network create --driver overlay --attachable db-cluster-net

# On node 1: Primary database
docker run -d \
  --name db-primary \
  --network db-cluster-net \
  -e POSTGRES_REPLICATION_MODE=master \
  postgres:13

# On node 2: Replica database  
docker run -d \
  --name db-replica \
  --network db-cluster-net \
  -e POSTGRES_REPLICATION_MODE=slave \
  -e POSTGRES_MASTER_SERVICE=db-primary \
  postgres:13

# Containers can find each other across nodes by name!
```

---

## 🔍 Network Troubleshooting

### Debugging Network Connectivity

```bash
# Check container networking
docker exec -it container-name ip addr show
docker exec -it container-name route -n

# Test connectivity between containers
docker exec -it web ping api
docker exec -it web telnet api 3000
docker exec -it web nslookup api

# Check DNS resolution
docker exec -it container-name nslookup google.com
docker exec -it container-name cat /etc/resolv.conf

# Inspect network details
docker network inspect network-name
docker inspect container-name | jq '.[0].NetworkSettings'
```

### Common Network Issues

**❌ Problem: Container can't reach another container**
```bash
# Check if both containers are on the same network
docker inspect container1 | jq '.[0].NetworkSettings.Networks'
docker inspect container2 | jq '.[0].NetworkSettings.Networks'

# If not on same network, connect them
docker network connect my-network container2
```

**❌ Problem: Can't reach container from host**
```bash
# Check port mappings
docker port container-name

# Verify service is listening inside container
docker exec container-name netstat -tlnp

# Test from inside container first
docker exec container-name curl localhost:3000
```

**❌ Problem: DNS resolution not working**
```bash
# Check DNS settings
docker exec container-name cat /etc/resolv.conf

# Try using IP addresses instead of names
docker exec container1 ping <container2-ip>

# Restart containers if DNS is corrupted
docker restart container-name
```

### Network Performance Testing

```bash
# Test bandwidth between containers
docker run --rm --network my-network nicolaka/netshoot \
  iperf3 -c other-container

# Test latency
docker exec container1 ping -c 10 container2

# Monitor network traffic
docker run --rm --network container:web-container \
  nicolaka/netshoot tcpdump -i eth0
```

---

## ⚙️ Advanced Networking Patterns

### Service Mesh Pattern

```bash
# Create service mesh network
docker network create service-mesh

# Run service discovery (Consul)
docker run -d \
  --name consul \
  --network service-mesh \
  -p 8500:8500 \
  consul agent -server -bootstrap -ui

# Run services with service mesh sidecar
docker run -d \
  --name app1 \
  --network service-mesh \
  -l "service=app1" \
  my-app:latest

docker run -d \
  --name app2 \
  --network service-mesh \
  -l "service=app2" \
  my-app:latest

# Service mesh handles routing, load balancing, security
```

### API Gateway Pattern

```bash
# Network for microservices
docker network create microservices

# API Gateway
docker run -d \
  --name api-gateway \
  --network microservices \
  -p 80:80 \
  nginx  # configured to route to microservices

# Microservices (not exposed to host)
docker run -d --name user-service --network microservices user-api:latest
docker run -d --name order-service --network microservices order-api:latest
docker run -d --name payment-service --network microservices payment-api:latest

# All external traffic goes through gateway
# Gateway routes requests to appropriate microservice
```

---

## 🎯 Hands-On Exercises

### Exercise 1: Multi-Tier Application with Network Isolation

Create a voting application with proper network segmentation:

```bash
# Create networks
docker network create frontend-tier
docker network create backend-tier

# Redis (backend only)
docker run -d \
  --name redis \
  --network backend-tier \
  redis:alpine

# Python voting app (needs both networks)
docker run -d \
  --name voting-app \
  --network frontend-tier \
  -p 5000:80 \
  dockersamples/examplevotingapp_vote:before

docker network connect backend-tier voting-app

# .NET results app (needs both networks)  
docker run -d \
  --name results-app \
  --network frontend-tier \
  -p 5001:80 \
  dockersamples/examplevotingapp_result:before

docker network connect backend-tier results-app

# Test isolation - voting-app can reach redis, but external traffic cannot
```

### Exercise 2: Service Discovery and Load Balancing

Set up automatic service discovery:

```bash
# Create network
docker network create discovery-net

# Start service registry (Consul)
docker run -d \
  --name consul \
  --network discovery-net \
  -p 8500:8500 \
  consul

# Register services with health checks
docker run -d \
  --name web1 \
  --network discovery-net \
  --label "service.name=web" \
  --label "service.port=80" \
  nginx

docker run -d \
  --name web2 \
  --network discovery-net \
  --label "service.name=web" \
  --label "service.port=80" \
  nginx

# Load balancer that discovers services
docker run -d \
  --name lb \
  --network discovery-net \
  -p 80:80 \
  nginx  # configured to query Consul for service endpoints
```

### Exercise 3: Network Monitoring Setup

Create a network monitoring solution:

```bash
# Monitoring network
docker network create monitoring

# Prometheus for metrics
docker run -d \
  --name prometheus \
  --network monitoring \
  -p 9090:9090 \
  prom/prometheus

# Grafana for visualization  
docker run -d \
  --name grafana \
  --network monitoring \
  -p 3000:3000 \
  grafana/grafana

# Node exporter for host metrics
docker run -d \
  --name node-exporter \
  --network monitoring \
  -p 9100:9100 \
  prom/node-exporter

# cAdvisor for container metrics
docker run -d \
  --name cadvisor \
  --network monitoring \
  -p 8080:8080 \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  google/cadvisor
```

---

## 📖 Key Takeaways

🌉 **Custom bridge networks provide automatic name resolution**  
🏢 **Use network segmentation to improve security and organization**  
🔌 **Port mapping controls external access to your services**  
🛡️ **Network isolation prevents unauthorized container communication**  
🌍 **Overlay networks enable multi-host container deployments**  
🔍 **Network troubleshooting requires understanding container DNS and routing**

**Next up:** Ready to master Dockerfile creation? Check out [Dockerfile Instructions](dockerfile_instructions.md) to learn the art of building efficient, secure container images!