# Terraform Deep Dive

A comprehensive guide to advanced Terraform concepts, state management, modules, and production practices.

## What is Terraform?
- Open-source Infrastructure as Code tool by HashiCorp
- Uses HashiCorp Configuration Language (HCL) for human-readable infrastructure definitions
- Multi-cloud provider support (AWS, Azure, GCP, VMware, etc.)
- Declarative approach: describe desired end state, Terraform figures out how to get there

## Key Concepts

### State Management
Terraform tracks infrastructure in a state file (`terraform.tfstate`) that maps real resources to your configuration.

```bash
# State commands
terraform state list                    # List all resources in state
terraform state show aws_instance.web   # Show details of specific resource
terraform state mv old_name new_name    # Rename resource in state
terraform state rm aws_instance.web     # Remove resource from state
terraform state pull > backup.tfstate  # Backup current state
```

### Remote State Configuration
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-state-locks"
  }
}
```

## Advanced Terraform Features

### Modules for Reusability
```hcl
# modules/vpc/main.tf
variable "cidr_block" {
  description = "CIDR block for VPC"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
}

resource "aws_vpc" "main" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

output "vpc_id" {
  value = aws_vpc.main.id
}

# Using the module
module "production_vpc" {
  source = "./modules/vpc"
  
  cidr_block  = "10.0.0.0/16"
  environment = "production"
}
```

### Conditional Resources and Loops
```hcl
# Conditional resource creation
resource "aws_instance" "web" {
  count = var.create_instance ? 1 : 0
  
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t2.micro"
}

# For_each loop
resource "aws_s3_bucket" "example" {
  for_each = toset(var.bucket_names)
  
  bucket = each.value
}

# Dynamic blocks
resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

### Workspaces for Environment Management
```bash
# Create and manage workspaces
terraform workspace new development
terraform workspace new staging  
terraform workspace new production

# List workspaces
terraform workspace list

# Switch workspaces
terraform workspace select production

# Use workspace in configuration
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = terraform.workspace == "production" ? "t3.large" : "t2.micro"
  
  tags = {
    Environment = terraform.workspace
  }
}
```

## Best Practices

### Code Organization
```
project/
├── modules/
│   ├── networking/
│   ├── compute/
│   └── database/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
├── terraform.tf          # Provider versions
├── variables.tf          # Variable definitions
├── outputs.tf           # Output definitions
├── main.tf              # Main configuration
└── terraform.tfvars     # Variable values
```

### Security Best Practices
- Store sensitive data in variable files or environment variables
- Use remote state with encryption
- Implement least privilege access policies
- Regular security scanning of infrastructure code

```hcl
# Using sensitive variables
variable "database_password" {
  description = "Database password"
  type        = string
  sensitive   = true
}

# Referencing secrets from AWS Secrets Manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "production/database/password"
}

resource "aws_db_instance" "example" {
  # ... other configuration ...
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}
```

### Testing Infrastructure Code
```bash
# Validate syntax
terraform validate

# Format code
terraform fmt -recursive

# Security scanning with tfsec
tfsec .

# Plan with detailed output
terraform plan -detailed-exitcode

# Testing with Terratest (Go-based testing framework)
# test/terraform_example_test.go
func TestTerraformExample(t *testing.T) {
    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../examples/terraform-example",
    })

    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)
    
    // Validate outputs
    instanceId := terraform.Output(t, terraformOptions, "instance_id")
    assert.NotEmpty(t, instanceId)
}
```

## Troubleshooting Common Issues

### State Lock Issues
```bash
# Force unlock (use carefully)
terraform force-unlock LOCK_ID

# Check who has the lock
terraform plan  # Will show lock information in error
```

### Import Existing Resources
```bash
# Import existing AWS S3 bucket
terraform import aws_s3_bucket.example my-existing-bucket

# Import EC2 instance
terraform import aws_instance.web i-1234567890abcdef0
```

### Debugging
```bash
# Enable detailed logging
export TF_LOG=DEBUG
terraform plan

# Log to file  
export TF_LOG=DEBUG
export TF_LOG_PATH=./terraform.log
terraform apply
```

## Advanced Patterns

### Blue-Green Deployments
```hcl
variable "active_environment" {
  description = "Currently active environment (blue or green)"
  type        = string
  default     = "blue"
}

module "blue_environment" {
  source = "./modules/app"
  
  environment = "blue"
  active      = var.active_environment == "blue"
}

module "green_environment" {
  source = "./modules/app"
  
  environment = "green"  
  active      = var.active_environment == "green"
}
```

### Multi-Region Deployments
```hcl
provider "aws" {
  alias  = "us_east"
  region = "us-east-1"
}

provider "aws" {
  alias  = "us_west"  
  region = "us-west-2"
}

module "us_east_infrastructure" {
  source = "./modules/regional-infrastructure"
  
  providers = {
    aws = aws.us_east
  }
  
  region = "us-east-1"
}

module "us_west_infrastructure" {
  source = "./modules/regional-infrastructure"
  
  providers = {
    aws = aws.us_west
  }
  
  region = "us-west-2"
}
```

## Exercises & Next Steps

- Exercise: Create a reusable VPC module and use it in multiple environments
- Next: Build a complete three-tier application infrastructure with modules
- Advanced: Implement automated testing and validation pipelines

## Resources
- [Terraform Documentation](https://www.terraform.io/docs/)
- [Terraform Registry](https://registry.terraform.io/)
- [HashiCorp Learn](https://learn.hashicorp.com/terraform)