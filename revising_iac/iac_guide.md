# Infrastructure as Code (IaC) Fundamentals Guide 🏗️

A comprehensive, hands-on guide to Infrastructure as Code using Terraform, CloudFormation, and Ansible for modern DevOps.

**🎯 Status:** LEARNING IN PROGRESS - Started September 25, 2025

---

## 🧭 Your IaC Learning Journey

```
Plan → Code → Deploy → Manage → Scale
  ↓      ↓       ↓        ↓       ↓
Design  Write   Apply   Monitor  Iterate
Resources Templates Infrastructure State Updates
```

## 📋 Quick Navigation
- [🏗️ IaC Fundamentals](#️-iac-fundamentals) - *What is Infrastructure as Code and why it matters*
- [🚀 Quick Start Guide](#-quick-start-guide) - *Get hands-on in 10 minutes*
- [🏢 Building Your First Infrastructure](#-building-your-first-infrastructure) - *Real example walkthrough*
- [🎯 Common Use Cases](#-common-use-cases) - *When and why to use IaC*
- [📚 Deep Dive Resources](#-deep-dive-resources) - *Advanced topics and guides*
- [🔐 Best Practices & Security](#-best-practices--security) - *Production-ready patterns*
- [🆘 Troubleshooting & Quick Reference](#-troubleshooting--quick-reference) - *Common issues and solutions*
- [Exercises & Next Steps](#exercises--next-steps)

---

## 🏗️ IaC Fundamentals

### The Big Picture: Why Infrastructure as Code Exists

**The Problem:** *"It works on my server, but not on yours!"*  
Ever spent days manually configuring servers, only to have them fail in production or be impossible to replicate? IaC solves this by treating infrastructure like software.

**The Solution:** *Code your infrastructure like you code your applications*  
Just like developers use version control for application code, IaC lets you version control, test, and automate your entire infrastructure.

### Core Concepts (With Real-World Analogies)

#### 📝 **Infrastructure as Code** → *Building Blueprints*
```hcl
# Think: Architectural blueprints for a house
# - Specifies exact materials and dimensions
# - Can be used to build identical houses
# - Changes are documented and tracked
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t2.micro"
}
```

#### 🏭 **Declarative vs Imperative** → *Recipe vs Instructions*
```yaml
# Declarative (WHAT you want) - like ordering from a menu
Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1d0
      InstanceType: t2.micro

# vs Imperative (HOW to do it) - like cooking instructions
# 1. Launch EC2 instance
# 2. Configure security groups  
# 3. Attach storage
# 4. Set up networking...
```

#### 🔄 **State Management** → *House Inventory*
```bash
# Think: A detailed inventory of everything in your house
# - Tracks what currently exists
# - Knows what needs to be added/removed/changed
# - Prevents conflicts and duplicates
terraform state list    # See what's currently managed
terraform plan         # Preview what will change
terraform apply        # Make the changes
```

#### 🌍 **Multi-Cloud & Providers** → *Universal Remote Control*
```hcl
# One tool to control different cloud providers
# Like a universal remote for all your devices

provider "aws" {
  region = "us-east-1"
}

provider "azure" {
  features {}
}

provider "google" {
  project = "my-project"
  region  = "us-central1"
}
```

---

## 🚀 Quick Start Guide

### Step 1: Verify Your Setup (Terraform)
```bash
# Install Terraform (if not already installed)
# Windows: choco install terraform
# macOS: brew install terraform  
# Linux: Download from terraform.io

# Check installation
terraform version        # Should show Terraform version
aws --version           # Check AWS CLI (if using AWS)
```

### Step 2: Your First Infrastructure
Create a simple directory structure:
```bash
mkdir my-first-iac && cd my-first-iac
```

Create `main.tf`:
```hcl
# Configure the provider
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

# Create a simple S3 bucket
resource "aws_s3_bucket" "example" {
  bucket = "my-first-iac-bucket-${random_string.suffix.result}"
}

resource "random_string" "suffix" {
  length  = 8
  special = false
  upper   = false
}

# Output the bucket name
output "bucket_name" {
  value = aws_s3_bucket.example.id
}
```

### Step 3: Deploy Your Infrastructure
```bash
# Initialize Terraform (download providers)
terraform init

# See what will be created
terraform plan

# Create the infrastructure
terraform apply
# Type 'yes' when prompted

# Your S3 bucket is now live!
```

### Step 4: Clean Up
```bash
# Remove the infrastructure
terraform destroy
# Type 'yes' when prompted
```

**🎉 Congratulations!** You just:
1. Defined infrastructure as code
2. Created real cloud resources
3. Managed infrastructure lifecycle
4. Cleaned up resources safely

---

## 🏢 Building Your First Infrastructure

Let's build a complete web application infrastructure with proper networking, security, and scaling.

### The Scenario
You're deploying a web application that needs:
- Web server with load balancing
- Database for data storage
- Proper networking and security
- Monitoring and logging

### Step 1: Project Structure
```bash
mkdir webapp-infrastructure && cd webapp-infrastructure
mkdir modules/{networking,compute,database}
```

### Step 2: Main Configuration (`main.tf`)
```hcl
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# Variables
variable "aws_region" {
  description = "AWS region for resources"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}

# VPC and Networking
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

resource "aws_subnet" "public" {
  count = 2

  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 1}.0/24"
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name        = "${var.environment}-public-subnet-${count.index + 1}"
    Environment = var.environment
    Type        = "Public"
  }
}

resource "aws_subnet" "private" {
  count = 2

  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 10}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name        = "${var.environment}-private-subnet-${count.index + 1}"
    Environment = var.environment
    Type        = "Private"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name        = "${var.environment}-igw"
    Environment = var.environment
  }
}

# Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name        = "${var.environment}-public-rt"
    Environment = var.environment
  }
}

resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)

  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

# Security Group for Web Servers
resource "aws_security_group" "web" {
  name_prefix = "${var.environment}-web-"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [aws_vpc.main.cidr_block]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "${var.environment}-web-sg"
    Environment = var.environment
  }
}

# Data sources
data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# Launch Template for Auto Scaling
resource "aws_launch_template" "web" {
  name_prefix   = "${var.environment}-web-"
  image_id      = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  vpc_security_group_ids = [aws_security_group.web.id]

  user_data = base64encode(<<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              echo "<h1>Hello from ${var.environment} Web Server</h1>" > /var/www/html/index.html
              EOF
  )

  tag_specifications {
    resource_type = "instance"
    tags = {
      Name        = "${var.environment}-web-server"
      Environment = var.environment
    }
  }

  lifecycle {
    create_before_destroy = true
  }
}

# Auto Scaling Group
resource "aws_autoscaling_group" "web" {
  name                = "${var.environment}-web-asg"
  vpc_zone_identifier = aws_subnet.public[*].id
  target_group_arns   = [aws_lb_target_group.web.arn]
  health_check_type   = "ELB"

  min_size         = 1
  max_size         = 4
  desired_capacity = 2

  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "${var.environment}-web-asg"
    propagate_at_launch = false
  }

  tag {
    key                 = "Environment"
    value               = var.environment
    propagate_at_launch = true
  }
}

# Application Load Balancer
resource "aws_lb" "web" {
  name               = "${var.environment}-web-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.web.id]
  subnets            = aws_subnet.public[*].id

  enable_deletion_protection = false

  tags = {
    Name        = "${var.environment}-web-alb"
    Environment = var.environment
  }
}

resource "aws_lb_target_group" "web" {
  name     = "${var.environment}-web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 2
  }

  tags = {
    Name        = "${var.environment}-web-tg"
    Environment = var.environment
  }
}

resource "aws_lb_listener" "web" {
  load_balancer_arn = aws_lb.web.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}

# Outputs
output "load_balancer_dns" {
  description = "DNS name of the load balancer"
  value       = aws_lb.web.dns_name
}

output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = aws_subnet.public[*].id
}
```

### Step 3: Variables File (`terraform.tfvars`)
```hcl
aws_region  = "us-east-1"
environment = "dev"
```

### Step 4: Deploy Complete Infrastructure
```bash
# Initialize and plan
terraform init
terraform plan

# Deploy (takes 3-5 minutes)
terraform apply

# Get the load balancer URL
terraform output load_balancer_dns

# Visit the URL in your browser - you now have a scalable web application!
```

**🎯 What You Just Built:**
- Virtual Private Cloud with public/private subnets
- Application Load Balancer for high availability  
- Auto Scaling Group for automatic scaling
- Security groups for proper access control
- Launch template for consistent server configuration

---

## 🎯 Common Use Cases

### 🏢 **Multi-Environment Management**
```hcl
# Easily deploy to dev, staging, production
# with environment-specific configurations

# terraform/environments/dev/main.tf
module "infrastructure" {
  source = "../../modules/webapp"
  
  environment     = "dev"
  instance_type   = "t3.micro"
  min_capacity    = 1
  max_capacity    = 2
}

# terraform/environments/prod/main.tf  
module "infrastructure" {
  source = "../../modules/webapp"
  
  environment     = "production"
  instance_type   = "t3.large"
  min_capacity    = 3
  max_capacity    = 10
}
```

### 🔄 **Disaster Recovery**
```hcl
# Define identical infrastructure in multiple regions
provider "aws" {
  alias  = "primary"
  region = "us-east-1"
}

provider "aws" {
  alias  = "dr"
  region = "us-west-2"
}

module "primary_region" {
  source = "./modules/webapp"
  providers = {
    aws = aws.primary
  }
}

module "dr_region" {
  source = "./modules/webapp"
  providers = {
    aws = aws.dr
  }
}
```

### 📊 **Cost Optimization**
```hcl
# Automatically provision resources based on schedules
resource "aws_autoscaling_schedule" "scale_down_evening" {
  scheduled_action_name  = "scale-down-evening"
  min_size               = 0
  max_size               = 1
  desired_capacity       = 0
  recurrence             = "0 22 * * MON-FRI"  # 10 PM weekdays
  autoscaling_group_name = aws_autoscaling_group.web.name
}

resource "aws_autoscaling_schedule" "scale_up_morning" {
  scheduled_action_name  = "scale-up-morning"
  min_size               = 1
  max_size               = 4
  desired_capacity       = 2
  recurrence             = "0 8 * * MON-FRI"   # 8 AM weekdays
  autoscaling_group_name = aws_autoscaling_group.web.name
}
```

---

## 📚 Deep Dive Resources

Ready to dive deeper? Check out our specialized guides:

- **[🏗️ Terraform Deep Dive](iac_deep_dive/terraform.md)**  
  *Master Terraform state, modules, and advanced patterns*

- **[☁️ AWS CloudFormation](iac_deep_dive/cloudformation.md)**  
  *Native AWS infrastructure automation and templates*

- **[⚙️ Ansible Configuration Management](iac_deep_dive/ansible.md)**  
  *Automate server configuration and application deployment*

---

## 🔐 Best Practices & Security

### Infrastructure Security
- **Use least privilege principles**: Grant minimal necessary permissions
- **Encrypt everything**: Data at rest and in transit  
- **Network segmentation**: Use VPCs, subnets, and security groups properly
- **Regular security updates**: Keep AMIs and base images updated

```hcl
# Example: Encrypted S3 bucket with restricted access
resource "aws_s3_bucket" "secure_storage" {
  bucket = "my-secure-app-data"
}

resource "aws_s3_bucket_encryption" "secure_storage" {
  bucket = aws_s3_bucket.secure_storage.id

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}

resource "aws_s3_bucket_public_access_block" "secure_storage" {
  bucket = aws_s3_bucket.secure_storage.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

### Code Organization
- **Use modules**: Create reusable infrastructure components
- **Separate environments**: Use workspaces or directories
- **Version control**: Store all IaC in Git with proper branching
- **Documentation**: Comment your code and maintain README files

### State Management
- **Remote state**: Store Terraform state in S3/Azure Storage, not locally
- **State locking**: Use DynamoDB/Azure Table Storage to prevent conflicts
- **Regular backups**: Backup your state files regularly

```hcl
# Backend configuration for remote state
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "webapp/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

---

## 🆘 Troubleshooting & Quick Reference

### Common Issues & Solutions

**🚨 "Resource already exists" errors:**
```bash
terraform import aws_s3_bucket.example my-existing-bucket
# Import existing resources into Terraform state
```

**🔍 State file corruption:**
```bash
terraform state list                    # See managed resources
terraform state show aws_instance.web   # Inspect specific resource
terraform state rm aws_instance.web     # Remove from state (careful!)
```

**💥 Apply failures:**
```bash
terraform refresh                       # Sync state with real world
terraform plan -refresh-only           # See what changed outside Terraform
terraform apply -target=aws_instance.web # Apply to specific resource only
```

**🧹 Clean up and debugging:**
```bash
terraform plan -destroy                 # Preview what will be destroyed  
terraform destroy -target=aws_instance.web # Destroy specific resource
terraform console                       # Interactive console for testing
```

### Quick Commands Reference
| Command | Purpose | Example |
|---------|---------|---------|
| `terraform init` | Initialize working directory | `terraform init` |
| `terraform plan` | Preview changes | `terraform plan -var="env=prod"` |
| `terraform apply` | Create/update infrastructure | `terraform apply -auto-approve` |
| `terraform destroy` | Remove infrastructure | `terraform destroy` |
| `terraform fmt` | Format code | `terraform fmt -recursive` |
| `terraform validate` | Validate syntax | `terraform validate` |
| `terraform output` | Show output values | `terraform output load_balancer_dns` |

---

## Exercises & Next Steps

### Beginner Exercises
1. **Exercise 1**: Create a simple S3 bucket with versioning enabled
2. **Exercise 2**: Deploy an EC2 instance with a security group allowing SSH access
3. **Exercise 3**: Set up a VPC with public and private subnets

### Intermediate Exercises  
1. **Exercise 4**: Build the complete web application infrastructure from this guide
2. **Exercise 5**: Create Terraform modules for reusable components
3. **Exercise 6**: Implement multi-environment deployments (dev/staging/prod)

### Advanced Exercises
1. **Exercise 7**: Set up remote state with locking and backup
2. **Exercise 8**: Implement automated testing for your infrastructure code
3. **Exercise 9**: Create a complete CI/CD pipeline that deploys infrastructure

### Next Steps
- Explore deep-dive modules for Terraform, CloudFormation, and Ansible
- Learn about infrastructure testing with tools like Terratest
- Dive into advanced topics like policy as code and compliance automation
- Practice with real-world scenarios and multi-cloud deployments

---

**🎉 Happy Infrastructure Coding!** Remember: Infrastructure as Code is like programming, but for servers and cloud resources. Start simple, iterate often, and always version control your infrastructure.

> *"Infrastructure as Code is not just about automation - it's about bringing software engineering practices to infrastructure management."*