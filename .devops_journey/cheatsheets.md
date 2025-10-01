# DevOps Cheatsheets

This markdown file will contain concise cheatsheets for each DevOps topic you revise. Cheatsheets will be organized by topic and updated as you progress.

---

## Cheatsheet Index

- [Git Fundamentals](#git-fundamentals) ✅ **REVISED**
- [Linux/Unix Commands](#linuxunix-commands) ✅ **REVISED**  
- [Docker & Containerization](#docker--containerization) ✅ **REVISED**
- [CI/CD Fundamentals](#cicd-fundamentals) ✅ **REVISED**
- [Infrastructure as Code](#infrastructure-as-code) ✅ **REVISED**
- [Configuration Management](#configuration-management) ⚡ **IN PROGRESS**

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
**Study Status:** ✅ **COMPLETE**
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
**Study Status:** ✅ **COMPLETE**
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
**Study Status:** ✅ **COMPLETE**
You've completed Docker fundamentals including containerization, images, volumes, networking, Docker Compose, and best practices. Ready for production use!

---

## CI/CD Fundamentals ✅ **REVISED**

### Essential Pipeline Commands
```bash
# GitHub Actions - Trigger pipeline
git push origin main                    # Triggers on push to main
git push origin feature-branch          # Triggers on push to feature branch

# Jenkins CLI (if installed)
jenkins-cli build job-name              # Trigger Jenkins job
jenkins-cli console job-name            # View job console output

# GitLab CI - Local pipeline testing
gitlab-runner exec docker test-job      # Run specific job locally
```

### Common CI/CD Pipeline Stages
```yaml
# GitHub Actions Example Structure
name: CI/CD Pipeline
on: [push, pull_request]
jobs:
  build:    # Compile/build application
  test:     # Run automated tests  
  package:  # Create deployable artifacts
  deploy:   # Deploy to environments
```

### Jenkins Pipeline (Jenkinsfile)
```groovy
pipeline {
  agent any
  stages {
    stage('Build') { steps { sh 'npm run build' } }
    stage('Test')  { steps { sh 'npm test' } }
    stage('Deploy'){ steps { sh './deploy.sh' } }
  }
}
```

### GitLab CI (.gitlab-ci.yml)
```yaml
stages:
  - build
  - test
  - deploy

build-job:
  stage: build
  script: npm install && npm run build

test-job:
  stage: test
  script: npm test

deploy-job:
  stage: deploy
  script: ./deploy.sh
  only: [main]
```

### Quick Troubleshooting
- **Pipeline fails?** Check logs in provider UI (GitHub Actions/GitLab CI/Jenkins)
- **Tests fail?** Run tests locally first: `npm test` or equivalent
- **Deployment issues?** Verify credentials and environment access
- **Build timeouts?** Optimize steps, use caching, run stages in parallel

### 🎯 **Study Status: COMPLETE**
**Study Status:** ✅ **COMPLETE**
You've completed CI/CD fundamentals including pipeline automation, Jenkins, GitHub Actions, GitLab CI, and production deployment practices. Ready for advanced DevOps automation!

---

## Infrastructure as Code ⚡ **IN PROGRESS**

### Terraform Essentials
```bash
# Project Setup
terraform init                         # Initialize project (download providers)
terraform validate                     # Check syntax and configuration
terraform fmt                         # Format code to standard style
terraform plan                        # Preview changes before applying
terraform apply                       # Apply changes to infrastructure
terraform destroy                     # Destroy all managed infrastructure

# State Management
terraform state list                  # List all resources in state
terraform state show resource_name    # Show details of specific resource
terraform state mv old_name new_name  # Rename resource in state
terraform state rm resource_name      # Remove resource from state
terraform state pull > backup.tfstate # Backup state file
```

### Terraform Core Workflow
```hcl
# 1. Provider Configuration
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# 2. Resource Definition
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
    Environment = "production"
  }
}

# 3. Output Values
output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

### AWS CloudFormation Essentials
```bash
# Stack Operations
aws cloudformation create-stack --stack-name my-stack --template-body file://template.yaml
aws cloudformation update-stack --stack-name my-stack --template-body file://template.yaml  
aws cloudformation delete-stack --stack-name my-stack
aws cloudformation describe-stacks --stack-name my-stack

# Useful Commands
aws cloudformation validate-template --template-body file://template.yaml
aws cloudformation describe-stack-events --stack-name my-stack
aws cloudformation get-template --stack-name my-stack
```

### CloudFormation Template Structure
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Basic web server infrastructure'

Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues: [t2.micro, t2.small, t2.medium]

Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1d0
      InstanceType: !Ref InstanceType
      Tags:
        - Key: Name
          Value: WebServer

Outputs:
  InstanceId:
    Description: Instance ID
    Value: !Ref WebServer
```

### Ansible Essentials
```bash
# Inventory and Connection
ansible all -m ping                    # Test connection to all hosts
ansible-inventory --list               # Show inventory in JSON format
ansible-config dump                    # Show current configuration

# Ad-hoc Commands  
ansible webservers -m shell -a "uptime"           # Run shell command
ansible all -m copy -a "src=file.txt dest=/tmp/"  # Copy file to all hosts
ansible databases -m service -a "name=mysql state=restarted"  # Restart service

# Playbook Operations
ansible-playbook site.yml              # Run playbook
ansible-playbook site.yml --check      # Dry run (check mode)
ansible-playbook site.yml --tags web   # Run only specific tags
ansible-playbook site.yml --limit webservers  # Run on specific group
```

### Ansible Playbook Structure
```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  vars:
    app_name: mywebapp
    
  tasks:
    - name: Install nginx
      package:
        name: nginx
        state: present
        
    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
        
    - name: Copy configuration
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: restart nginx
      
  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

### Common IaC Patterns
```bash
# Multi-Environment Management
terraform workspace new production     # Create workspace for environment
terraform workspace select staging    # Switch to staging environment
terraform workspace list              # List all workspaces

# Module Usage (Terraform)
module "vpc" {
  source = "./modules/networking"
  
  cidr_block = "10.0.0.0/16"
  environment = "production"
}

# Variable Files
terraform apply -var-file="production.tfvars"  # Use specific variable file
terraform apply -var="instance_type=t3.large"  # Override single variable
```

### IaC Security Best Practices
```bash
# Secrets Management
export AWS_ACCESS_KEY_ID="your-key"           # Use environment variables
export AWS_SECRET_ACCESS_KEY="your-secret"    # Never hardcode credentials

# Terraform Security
terraform plan -detailed-exitcode             # Check for changes in CI/CD
tfsec .                                       # Security scanning for Terraform
checkov -f main.tf                           # Policy checking

# Ansible Vault
ansible-vault create secrets.yml              # Create encrypted file
ansible-vault encrypt vars.yml                # Encrypt existing file  
ansible-playbook site.yml --ask-vault-pass   # Run with encrypted vars
```

### Troubleshooting Common Issues
```bash
# Terraform Issues
terraform refresh                             # Sync state with real infrastructure
terraform force-unlock LOCK_ID               # Remove stuck state lock
terraform import aws_instance.web i-1234567  # Import existing resource

# CloudFormation Issues  
aws cloudformation cancel-update-stack --stack-name my-stack    # Cancel stuck update
aws cloudformation continue-update-rollback --stack-name my-stack  # Continue rollback

# Ansible Issues
ansible all -m setup                         # Gather facts for debugging
ansible-playbook site.yml -vvv               # Verbose output for troubleshooting
ansible all -m ping -u ubuntu --private-key ~/.ssh/id_rsa  # Test connection with specific key
```

### Quick Infrastructure Patterns
```bash
# Simple Web Server (Terraform)
terraform init
terraform plan -var="instance_type=t2.micro"
terraform apply -auto-approve

# Database Setup (Ansible)
ansible-playbook -i inventory database.yml --tags mysql

# Multi-tier Application (CloudFormation)
aws cloudformation create-stack \
  --stack-name webapp \
  --template-body file://three-tier-app.yaml \
  --parameters ParameterKey=Environment,ParameterValue=production
```

### Best Practices Reminders
- **Version Control**: Always commit IaC code to Git
- **State Security**: Use remote state with encryption (S3 + DynamoDB for Terraform)
- **Modular Design**: Create reusable modules/roles/templates
- **Environment Separation**: Use workspaces, separate accounts, or parameter files
- **Testing**: Validate templates/playbooks before production deployment
- **Documentation**: Document variables, outputs, and architectural decisions

### Quick Safety Tips
- **Always run plan/check first**: Preview changes before applying
- **Use specific versions**: Pin provider/module versions
- **Backup state files**: Before major infrastructure changes
- **Test in dev first**: Never test directly in production
- **Monitor resources**: Track costs and resource usage
- **Clean up unused resources**: Regular infrastructure hygiene

### 🎯 **Study Status: COMPLETE**
**Study Status:** ✅ **COMPLETE**
You've completed Infrastructure as Code fundamentals including Terraform, AWS CloudFormation, and Ansible for automated infrastructure management and deployment. Ready for advanced cloud automation!

---

## Configuration Management ⚡ **IN PROGRESS**

### Ansible Essentials
```bash
# Connection and Testing
ansible all -m ping                           # Test connectivity
ansible all -m setup                          # Gather facts
ansible-inventory --list                      # Show inventory

# Ad-hoc Commands
ansible webservers -a "uptime"                # Run shell command
ansible all -m package -a "name=nginx state=present"  # Install package
ansible databases -m service -a "name=mysql state=restarted"  # Restart service
ansible all -m copy -a "src=file.txt dest=/tmp/"  # Copy file

# Playbook Operations
ansible-playbook site.yml                     # Run playbook
ansible-playbook site.yml --check             # Dry run
ansible-playbook site.yml --tags web          # Run specific tags
ansible-playbook site.yml --limit webservers  # Run on specific group
ansible-playbook site.yml --ask-vault-pass    # Use encrypted variables
```

### Ansible Playbook Structure
```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  vars:
    app_name: webapp
    
  tasks:
    - name: Install nginx
      package:
        name: nginx
        state: present
        
    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
        
    - name: Copy configuration
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: restart nginx
      
  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

### Ansible Vault (Secrets Management)
```bash
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Encrypt existing file
ansible-vault encrypt vars.yml

# Decrypt file
ansible-vault decrypt secrets.yml

# View encrypted file
ansible-vault view secrets.yml

# Run playbook with vault
ansible-playbook site.yml --ask-vault-pass
ansible-playbook site.yml --vault-password-file ~/.vault_pass
```

### Chef Essentials
```bash
# Node Management
knife node list                               # List all nodes
knife node show web1.example.com             # Show node details
knife node edit web1.example.com             # Edit node attributes

# Cookbook Management
knife cookbook upload webapp                  # Upload cookbook
knife cookbook list                           # List cookbooks
knife cookbook show webapp 1.0.0             # Show cookbook version

# Environment Management
knife environment list                        # List environments
knife environment show production             # Show environment
knife environment from file environments/production.rb  # Upload environment

# Bootstrap New Nodes
knife bootstrap web1.example.com \
  --ssh-user ubuntu \
  --sudo \
  --identity-file ~/.ssh/aws-key.pem \
  --node-name web1 \
  --run-list 'role[webserver]' \
  --environment production

# Data Bag Management
knife data bag create secrets                # Create data bag
knife data bag from file secrets data_bags/secrets/database.json
knife data bag show secrets database         # Show data bag item
```

### Chef Recipe Structure
```ruby
# Install and configure nginx
package 'nginx' do
  action :install
end

service 'nginx' do
  action [:enable, :start]
  supports restart: true, reload: true
end

template '/etc/nginx/nginx.conf' do
  source 'nginx.conf.erb'
  owner 'root'
  group 'root'
  mode '0644'
  variables(
    worker_processes: node['nginx']['worker_processes']
  )
  notifies :reload, 'service[nginx]', :immediately
end
```

### Puppet Essentials
```bash
# Node Management
puppet node list                              # List nodes (PE)
puppet cert list                              # List certificates
puppet cert sign web1.example.com            # Sign certificate

# Catalog and Code
puppet parser validate manifests/site.pp     # Validate syntax
puppet apply manifests/site.pp                # Apply manifest locally
puppet agent --test                           # Run puppet agent
puppet agent --noop                           # Dry run

# Module Management
puppet module list                            # List installed modules
puppet module install puppetlabs-nginx       # Install module
puppet module search nginx                    # Search for modules

# Hiera (Data Management)
hiera webapp::version                         # Lookup hiera value
hiera -c /etc/puppetlabs/puppet/hiera.yaml webapp::version
```

### Puppet Manifest Structure
```puppet
# Install and configure nginx
package { 'nginx':
  ensure => installed,
}

service { 'nginx':
  ensure     => running,
  enable     => true,
  hasrestart => true,
  require    => Package['nginx'],
}

file { '/etc/nginx/nginx.conf':
  ensure  => file,
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
  content => template('nginx/nginx.conf.erb'),
  notify  => Service['nginx'],
  require => Package['nginx'],
}
```

### Testing Configuration Management
```bash
# Ansible Testing
molecule init scenario --driver-name docker   # Initialize testing
molecule test                                  # Run full test cycle
molecule converge                              # Run playbook
molecule verify                                # Run tests

# Chef Testing  
kitchen list                                   # List test suites
kitchen test                                   # Run full test cycle
kitchen converge                               # Run chef
kitchen verify                                 # Run InSpec tests

# Puppet Testing
bundle exec rake spec                          # Run rspec tests
bundle exec rake lint                          # Lint puppet code
bundle exec rake syntax                        # Validate syntax
```

### Common Configuration Patterns
```yaml
# Multi-environment deployment (Ansible)
- name: Deploy application
  hosts: "{{ target_environment }}"
  vars:
    app_version: "{{ versions[target_environment] }}"
  tasks:
    - name: Deploy app
      include_tasks: deploy.yml

# Rolling deployment
- name: Rolling update
  hosts: webservers
  serial: 1
  max_fail_percentage: 0
  pre_tasks:
    - name: Remove from load balancer
      uri:
        url: "http://lb.example.com/remove/{{ inventory_hostname }}"
```

### Troubleshooting Common Issues
```bash
# Ansible Debugging
ansible-playbook site.yml -vvv               # Verbose output
ansible all -m setup | grep ansible_ssh      # Check SSH connectivity
ansible all -m ping -u ubuntu --private-key ~/.ssh/key.pem  # Test with specific key

# Chef Debugging
chef-client -l debug                          # Debug mode
knife ssh 'role:webserver' 'chef-client' -x ubuntu  # Run chef on multiple nodes

# Puppet Debugging
puppet agent --test --debug                   # Debug mode
puppet agent --test --noop --verbose          # Dry run with details
```

### Best Practices Reminders
- **Idempotency**: Ensure configurations can run multiple times safely
- **Version Control**: Keep all configuration code in Git
- **Testing**: Test configurations before production deployment
- **Secrets Management**: Use proper encryption for sensitive data
- **Documentation**: Document your infrastructure as code
- **Monitoring**: Monitor configuration drift and compliance

### Quick Safety Tips
- **Test in dev first**: Always test configurations in development
- **Use check/dry-run modes**: Preview changes before applying
- **Backup before changes**: Backup critical systems before major changes
- **Monitor after deployment**: Watch for issues after configuration changes
- **Have rollback plans**: Always have a way to revert changes
- **Use proper authentication**: Secure access to configuration management systems

### 🎯 **Study Status: IN PROGRESS**
**Study Status:** ⚡ **IN PROGRESS**
Currently building comprehensive Configuration Management skills including Ansible automation, Chef cookbook development, and Puppet manifest management for scalable infrastructure configuration.

---

