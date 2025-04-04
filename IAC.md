# Infrastructure as Code (IaC) with Terraform

This guide outlines our organization's standards for infrastructure management using Terraform.

> 📌 **Return to**: [Main Development Guide](../README.md)

## Quick Start

```bash
# 1. Install Terraform
brew install tfenv
tfenv install 1.7.5
tfenv use 1.7.5

# 2. Clone infrastructure repository
git clone git@github.com:organization/infrastructure.git
cd infrastructure

# 3. Navigate to environment directory
cd environments/dev

# 4. Initialize Terraform
terraform init

# 5. Plan changes
terraform plan -var-file=terraform.tfvars -out=tfplan

# 6. Apply changes
terraform apply tfplan
```

## Environment Setup

### Terraform Installation

We use `tfenv` to manage Terraform versions:

```bash
# Install tfenv
brew install tfenv

# Install Terraform version
tfenv install 1.7.5

# Set as default version
tfenv use 1.7.5
```

### AWS CLI Configuration

For AWS deployments:

```bash
# Install AWS CLI
brew install awscli

# Configure AWS credentials
aws configure
```

## Repository Structure

Standard Terraform project structure:

```
infrastructure/
├── environments/              # Environment-specific configs
│   ├── dev/
│   │   ├── main.tf            # Main configuration
│   │   ├── variables.tf       # Variable definitions
│   │   ├── terraform.tfvars   # Variable values (gitignored)
│   │   ├── outputs.tf         # Output definitions
│   │   └── backend.tf         # Backend configuration
│   └── prod/
│       ├── main.tf
│       ├── variables.tf
│       ├── terraform.tfvars
│       ├── outputs.tf
│       └── backend.tf
├── modules/                   # Reusable modules
│   ├── networking/            # Networking resources
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── compute/              # Compute resources
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── database/             # Database resources
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── security/             # Security resources
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── .gitignore                # Git ignore file
└── README.md                 # Documentation
```

## State Management

### Remote State Configuration

Store state in a remote backend:

```hcl
# environments/dev/backend.tf
terraform {
  backend "s3" {
    bucket         = "organization-terraform-state"
    key            = "projects/project-name/dev/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

### State Locking

Use DynamoDB for state locking:

```hcl
# Create the DynamoDB table for state locking
resource "aws_dynamodb_table" "terraform_state_lock" {
  name           = "terraform-state-lock"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

## Module Organization

### Module Structure

Each module should include:

- `main.tf` - Main resource definitions
- `variables.tf` - Input variable definitions
- `outputs.tf` - Output value definitions
- `README.md` - Module documentation

Example `modules/networking/vpc/main.tf`:

```hcl
resource "aws_vpc" "main" {
  cidr_block           = var.cidr_block
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = merge(
    {
      Name = var.name
    },
    var.tags
  )
}

resource "aws_subnet" "public" {
  count = length(var.public_subnet_cidrs)

  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidrs[count.index]
  availability_zone       = var.availability_zones[count.index % length(var.availability_zones)]
  map_public_ip_on_launch = true

  tags = merge(
    {
      Name = "${var.name}-public-${count.index + 1}"
    },
    var.tags
  )
}

resource "aws_subnet" "private" {
  count = length(var.private_subnet_cidrs)

  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index % length(var.availability_zones)]

  tags = merge(
    {
      Name = "${var.name}-private-${count.index + 1}"
    },
    var.tags
  )
}
```

Example `modules/networking/vpc/variables.tf`:

```hcl
variable "name" {
  description = "Name of the VPC"
  type        = string
}

variable "cidr_block" {
  description = "CIDR block for the VPC"
  type        = string
}

variable "public_subnet_cidrs" {
  description = "CIDR blocks for public subnets"
  type        = list(string)
}

variable "private_subnet_cidrs" {
  description = "CIDR blocks for private subnets"
  type        = list(string)
}

variable "availability_zones" {
  description = "Availability zones for subnets"
  type        = list(string)
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {}
}
```

Example `modules/networking/vpc/outputs.tf`:

```hcl
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "IDs of the public subnets"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "IDs of the private subnets"
  value       = aws_subnet.private[*].id
}
```

### Module Usage

Use modules in environment configurations:

```hcl
# environments/dev/main.tf
module "vpc" {
  source = "../../modules/networking/vpc"

  name                 = "${var.project}-${var.environment}-vpc"
  cidr_block           = var.vpc_cidr
  public_subnet_cidrs  = var.public_subnet_cidrs
  private_subnet_cidrs = var.private_subnet_cidrs
  availability_zones   = var.availability_zones

  tags = {
    Environment = var.environment
    Project     = var.project
    ManagedBy   = "terraform"
  }
}
```

## Variables and Outputs

### Environment Variables

Define variables in `environments/dev/variables.tf`:

```hcl
variable "project" {
  description = "Project name"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
}

variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
}

variable "public_subnet_cidrs" {
  description = "CIDR blocks for public subnets"
  type        = list(string)
}

variable "private_subnet_cidrs" {
  description = "CIDR blocks for private subnets"
  type        = list(string)
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
}
```

Set values in `environments/dev/terraform.tfvars`:

```hcl
project               = "customer-portal"
environment           = "dev"
region                = "us-east-1"
vpc_cidr              = "10.0.0.0/16"
public_subnet_cidrs   = ["10.0.1.0/24", "10.0.2.0/24"]
private_subnet_cidrs  = ["10.0.3.0/24", "10.0.4.0/24"]
availability_zones    = ["us-east-1a", "us-east-1b"]
```

### Environment Outputs

Define outputs in `environments/dev/outputs.tf`:

```hcl
output "vpc_id" {
  description = "ID of the VPC"
  value       = module.vpc.vpc_id
}

output "public_subnet_ids" {
  description = "IDs of the public subnets"
  value       = module.vpc.public_subnet_ids
}

output "private_subnet_ids" {
  description = "IDs of the private subnets"
  value       = module.vpc.private_subnet_ids
}

output "api_endpoint" {
  description = "API Gateway endpoint URL"
  value       = module.api_gateway.api_endpoint
}
```

## Resource Naming

Follow this naming convention for all resources:

```
{project}-{environment}-{resource_type}-{purpose}
```

Examples:
- `customer-portal-dev-vpc-main`
- `customer-portal-prod-lambda-auth`
- `customer-portal-dev-rds-main`

## Infrastructure Management

### Managing Terraform State

```bash
# List resources in state
terraform state list

# Show state for a specific resource
terraform state show aws_vpc.main

# Remove a resource from state
terraform state rm aws_vpc.main

# Import an existing resource into state
terraform import aws_vpc.main vpc-12345678
```

### Managing Workspaces

```bash
# List workspaces
terraform workspace list

# Create a new workspace
terraform workspace new dev

# Select a workspace
terraform workspace select dev

# Delete a workspace
terraform workspace delete dev
```

## Deployment Process

### Development Workflow

1. **Create a feature branch**:
   ```bash
   git checkout -b feature/add-rds-instance
   ```

2. **Make changes to infrastructure code**:
   ```bash
   # Edit Terraform files
   ```

3. **Test changes locally**:
   ```bash
   terraform validate
   terraform fmt
   terraform plan -var-file=terraform.tfvars
   ```

4. **Commit changes**:
   ```bash
   git add .
   git commit -m "Add RDS instance"
   git push origin feature/add-rds-instance
   ```

5. **Create a pull request**:
   - Create a PR to merge into `development` branch
   - Wait for code review and CI checks to pass

6. **Apply changes in development environment**:
   ```bash
   # After PR is approved and merged
   terraform apply -var-file=terraform.tfvars
   ```

7. **Promote to production**:
   - Create a PR from `development` to `main`
   - After approval, apply changes to production

### CI/CD Integration

Integration with GitHub Actions:

```yaml
name: Terraform

on:
  pull_request:
    branches: [development, main]
  push:
    branches: [development, main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.7.5
      
      - name: Terraform Format
        run: terraform fmt -check -recursive
      
      - name: Terraform Init
        run: |
          cd environments/dev
          terraform init -backend=false
      
      - name: Terraform Validate
        run: |
          cd environments/dev
          terraform validate

  plan:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    needs: validate
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.7.5
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Terraform Plan
        run: |
          cd environments/dev
          terraform init
          terraform plan -var-file=terraform.tfvars -out=tfplan
      
      - name: Upload Plan
        uses: actions/upload-artifact@v4
        with:
          name: tfplan
          path: environments/dev/tfplan

  apply:
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    needs: validate
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.7.5
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Terraform Apply
        run: |
          cd environments/${{ github.ref == 'refs/heads/main' && 'prod' || 'dev' }}
          terraform init
          terraform apply -auto-approve -var-file=terraform.tfvars
```

## Documentation

### Module Documentation

Each module should include a README with:

1. **Purpose**: What the module is for
2. **Resources Created**: List of resources the module creates
3. **Usage Example**: How to use the module
4. **Input Variables**: All input variables with descriptions
5. **Outputs**: All outputs with descriptions

Example:

````markdown
# VPC Module

This module creates a VPC with public and private subnets across multiple availability zones.

## Resources Created

- VPC
- Public subnets
- Private subnets
- Internet Gateway
- NAT Gateway
- Route tables
- Route table associations

## Usage

```hcl
module "vpc" {
  source = "../../modules/networking/vpc"

  name                 = "my-vpc"
  cidr_block           = "10.0.0.0/16"
  public_subnet_cidrs  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnet_cidrs = ["10.0.3.0/24", "10.0.4.0/24"]
  availability_zones   = ["us-east-1a", "us-east-1b"]

  tags = {
    Environment = "dev"
    Project     = "my-project"
  }
}
```

## Input Variables

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| name | Name of the VPC | `string` | n/a | yes |
| cidr_block | CIDR block for the VPC | `string` | n/a | yes |
| public_subnet_cidrs | CIDR blocks for public subnets | `list(string)` | n/a | yes |
| private_subnet_cidrs | CIDR blocks for private subnets | `list(string)` | n/a | yes |
| availability_zones | Availability zones for subnets | `list(string)` | n/a | yes |
| tags | Tags to apply to resources | `map(string)` | `{}` | no |

## Outputs

| Name | Description |
|------|-------------|
| vpc_id | ID of the VPC |
| public_subnet_ids | IDs of the public subnets |
| private_subnet_ids | IDs of the private subnets |
````

## Security Best Practices

### Sensitive Data Management

- Never commit sensitive data (passwords, API keys, etc.) to version control
- Use AWS Secrets Manager or Parameter Store for sensitive values
- Reference sensitive values as variables:

```hcl
data "aws_secretsmanager_secret" "db_password" {
  name = "/${var.project}/${var.environment}/db/password"
}

data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = data.aws_secretsmanager_secret.db_password.id
}

resource "aws_db_instance" "main" {
  # ...
  password = jsondecode(data.aws_secretsmanager_secret_version.db_password.secret_string)["password"]
  # ...
}
```

### IAM Least Privilege

Follow the principle of least privilege:

```hcl
resource "aws_iam_policy" "lambda_policy" {
  name        = "${var.project}-${var.environment}-lambda-policy"
  description = "Policy for Lambda function"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Effect   = "Allow"
        Resource = "arn:aws:logs:*:*:*"
      },
      {
        Action = [
          "s3:GetObject"
        ]
        Effect   = "Allow"
        Resource = "${aws_s3_bucket.main.arn}/*"
      }
    ]
  })
}
```

### Network Security

Implement proper security groups:

```hcl
resource "aws_security_group" "web" {
  name        = "${var.project}-${var.environment}-web-sg"
  description = "Security group for web servers"
  vpc_id      = module.vpc.vpc_id

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

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "${var.project}-${var.environment}-web-sg"
    Environment = var.environment
    Project     = var.project
  }
}
```

## Troubleshooting

### Common Issues and Solutions

#### Error: No valid credential sources found

```
Error: No valid credential sources found
```

**Solution**:
```bash
# Configure AWS credentials
aws configure

# Or set environment variables
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_REGION="us-east-1"
```

#### Error: Error acquiring the state lock

```
Error: Error acquiring the state lock
```

**Solution**:
```bash
# Check if a previous process is still running
# If not, force unlock the state
terraform force-unlock <LOCK_ID>
```

#### Error: Error loading state

```
Error: Error loading state
```

**Solution**:
```bash
# Check if your backend configuration is correct
# Try re-initializing Terraform
terraform init -reconfigure
```

#### Error: The plan would destroy resources

```
Error: The plan would destroy X resource(s).
```

**Solution**:
```bash
# Review the plan carefully
terraform plan -out=tfplan

# If you're sure the changes are intended
terraform apply tfplan
```

## Version Management

Terraform version updates are handled as follows:
- We maintain compatibility with the latest stable Terraform release (currently 1.7.x)
- New projects should always use the current standardized version
- Existing projects should update to new Terraform versions within 3 months of their release
- All Terraform version upgrades require thorough testing of the infrastructure code
- We follow HashiCorp's release schedule to plan our upgrades