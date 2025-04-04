# Infrastructure as Code (IaC)

This guide outlines our organization's standards for infrastructure management using Terraform or Pulumi.

> 📌 **Return to**: [Main Development Guide](../README.md)

## Quick Start

```bash
# For Terraform projects
# Install Terraform on macOS
brew install terraform

# Clone infrastructure repository
git clone git@github.com:organization/infrastructure.git
cd infrastructure

# Navigate to environment directory
cd environments/dev

# Initialize Terraform
terraform init

# Plan changes
terraform plan -var-file=terraform.tfvars -out=tfplan

# Apply changes
terraform apply tfplan

# For Pulumi projects
# Install Pulumi on macOS
brew install pulumi

# Navigate to Pulumi project directory
cd pulumi-project

# Preview changes
pulumi preview

# Apply changes
pulumi up
```

## Technology Choice

For each project, choose either Terraform or Pulumi based on the following considerations:

- **Terraform**: Ideal for infrastructure teams familiar with HCL, or for projects where declarative configuration is preferred
- **Pulumi**: Better suited for teams with strong programming skills in TypeScript/JavaScript, Python, Go, or C#, or when complex logic is required

Choose one technology per project - do not mix Terraform and Pulumi within the same infrastructure project.

## Environment Setup

### Terraform Installation

```bash
# macOS
brew install terraform
```

For other platforms, refer to the [official Terraform installation documentation](https://developer.hashicorp.com/terraform/downloads).

Use Git for version pinning by committing the Terraform lock file (`.terraform.lock.hcl`) to your repository, which ensures consistent provider versions across all environments and team members.

### Pulumi Installation

```bash
# macOS
brew install pulumi
```

For other platforms, refer to the [official Pulumi installation documentation](https://www.pulumi.com/docs/install/).

### AWS CLI Configuration

```bash
# Install AWS CLI on macOS
brew install awscli

# Configure AWS credentials
aws configure
```

## Repository Structure

### Terraform Project Structure

```
infrastructure/
├── modules/                   # Reusable modules
│   ├── networking/            # Networking resources
│   ├── compute/               # Compute resources
│   ├── database/              # Database resources
│   └── security/              # Security resources
├── envs/                      # Environment configuration
│   ├── common.tfvars          # Common variables across environments
│   ├── dev.tfvars             # Dev-specific variables
│   ├── staging.tfvars         # Staging-specific variables
│   ├── prod.tfvars            # Production-specific variables
├── main.tf                    # Main configuration
├── variables.tf               # Variable definitions
├── outputs.tf                 # Output definitions
├── providers.tf               # Provider configuration
├── versions.tf                # Version constraints
├── backend.tf                 # Backend configuration
├── .gitignore                 # Git ignore file
└── README.md                  # Documentation
```

### Pulumi Project Structure

```
infrastructure/
├── src/                      # Shared code and utilities
├── modules/                  # Reusable Pulumi components
│   ├── networking/
│   ├── compute/
│   ├── database/
│   └── security/
├── Pulumi.yaml               # Project configuration
├── Pulumi.dev.yaml           # Dev stack configuration
├── Pulumi.staging.yaml       # Staging stack configuration
├── Pulumi.prod.yaml          # Production stack configuration
├── index.ts                  # Main program 
├── package.json              # Dependencies (for TypeScript)
├── tsconfig.json             # TypeScript configuration
├── .gitignore                # Git ignore file
└── README.md                 # Documentation
```

## State Management

### Terraform Remote State Configuration

Store state in a remote backend:

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "organization-terraform-state"
    key            = "projects/project-name/${terraform.workspace}/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

### Pulumi State Management

Pulumi automatically manages state in the Pulumi Cloud by default, but you can also configure it to use a self-managed backend:

```bash
# Configure Pulumi to use AWS S3 for state storage
pulumi login s3://organization-pulumi-state
```

### State Locking

Use DynamoDB for Terraform state locking:

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

## Using Workspaces

### Terraform Workspaces

Leverage Terraform workspaces to reduce code duplication across environments:

```bash
# List workspaces
terraform workspace list

# Create new workspaces for each environment
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Select a workspace
terraform workspace select dev
```

Use workspace-specific variables in your configuration:

```hcl
# main.tf
locals {
  environment = terraform.workspace

  # Environment-specific configurations
  env_configs = {
    dev = {
      instance_type = "t3.small"
      instance_count = 1
    }
    staging = {
      instance_type = "t3.medium"
      instance_count = 2
    }
    prod = {
      instance_type = "t3.large"
      instance_count = 3
    }
  }

  # Use the current workspace's configuration or default to dev
  config = lookup(local.env_configs, local.environment, local.env_configs.dev)
}

module "compute" {
  source = "./modules/compute"
  
  instance_type = local.config.instance_type
  instance_count = local.config.instance_count
  environment = local.environment
}
```

### Pulumi Stacks

Use Pulumi stacks (similar to Terraform workspaces) for environment separation:

```bash
# Create stacks for different environments
pulumi stack init dev
pulumi stack init staging
pulumi stack init prod

# Select a stack
pulumi stack select dev

# Set stack-specific configuration
pulumi config set instanceType t3.small
pulumi config set instanceCount 1
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

### Pulumi Component Structure

For Pulumi, create reusable components:

```typescript
// modules/networking/vpc.ts
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

export interface VpcArgs {
    name: string;
    cidrBlock: string;
    publicSubnetCidrs: string[];
    privateSubnetCidrs: string[];
    availabilityZones: string[];
    tags?: {[key: string]: string};
}

export class Vpc extends pulumi.ComponentResource {
    public vpc: aws.ec2.Vpc;
    public publicSubnets: aws.ec2.Subnet[];
    public privateSubnets: aws.ec2.Subnet[];

    constructor(name: string, args: VpcArgs, opts?: pulumi.ComponentResourceOptions) {
        super("custom:networking:Vpc", name, {}, opts);

        this.vpc = new aws.ec2.Vpc(name, {
            cidrBlock: args.cidrBlock,
            enableDnsSupport: true,
            enableDnsHostnames: true,
            tags: {
                Name: args.name,
                ...args.tags,
            },
        }, { parent: this });

        this.publicSubnets = args.publicSubnetCidrs.map((cidr, i) => {
            const az = args.availabilityZones[i % args.availabilityZones.length];
            return new aws.ec2.Subnet(`${name}-public-${i + 1}`, {
                vpcId: this.vpc.id,
                cidrBlock: cidr,
                availabilityZone: az,
                mapPublicIpOnLaunch: true,
                tags: {
                    Name: `${args.name}-public-${i + 1}`,
                    ...args.tags,
                },
            }, { parent: this });
        });

        this.privateSubnets = args.privateSubnetCidrs.map((cidr, i) => {
            const az = args.availabilityZones[i % args.availabilityZones.length];
            return new aws.ec2.Subnet(`${name}-private-${i + 1}`, {
                vpcId: this.vpc.id,
                cidrBlock: cidr,
                availabilityZone: az,
                tags: {
                    Name: `${args.name}-private-${i + 1}`,
                    ...args.tags,
                },
            }, { parent: this });
        });

        this.registerOutputs({
            vpcId: this.vpc.id,
            publicSubnetIds: this.publicSubnets.map(s => s.id),
            privateSubnetIds: this.privateSubnets.map(s => s.id),
        });
    }
}
```

## Variables and Outputs

### Environment Variables

Define variables in `variables.tf`:

```hcl
variable "project" {
  description = "Project name"
  type        = string
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

Set common values in `envs/common.tfvars`:

```hcl
project               = "customer-portal"
availability_zones    = ["us-east-1a", "us-east-1b"]
```

Set environment-specific values in environment files:

```hcl
# envs/dev.tfvars
vpc_cidr              = "10.0.0.0/16"
public_subnet_cidrs   = ["10.0.1.0/24", "10.0.2.0/24"]
private_subnet_cidrs  = ["10.0.3.0/24", "10.0.4.0/24"]
```

```hcl
# envs/prod.tfvars
vpc_cidr              = "10.1.0.0/16"
public_subnet_cidrs   = ["10.1.1.0/24", "10.1.2.0/24"]
private_subnet_cidrs  = ["10.1.3.0/24", "10.1.4.0/24"]
```

Apply with both common and environment-specific variables:

```bash
terraform apply -var-file=envs/common.tfvars -var-file=envs/dev.tfvars
```

### Outputs

Define outputs in `outputs.tf`:

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

### Managing Pulumi State

```bash
# View the current stack state
pulumi stack

# View a specific resource
pulumi stack output vpc

# Import an existing resource
pulumi import aws:ec2/vpc:Vpc main vpc-12345678
```

## Deployment Process

### Development Workflow

1. **Create a feature branch**:
   ```bash
   git checkout -b feature/add-rds-instance
   ```

2. **Make changes to infrastructure code**:
   ```bash
   # Edit Terraform/Pulumi files
   ```

3. **Test changes locally**:
   ```bash
   # For Terraform
   terraform validate
   terraform fmt
   terraform plan -var-file=envs/common.tfvars -var-file=envs/dev.tfvars
   
   # For Pulumi
   pulumi preview
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
   # For Terraform
   terraform workspace select dev
   terraform apply -var-file=envs/common.tfvars -var-file=envs/dev.tfvars
   
   # For Pulumi
   pulumi stack select dev
   pulumi up
   ```

7. **Promote to production**:
   - Create a PR from `development` to `main`
   - After approval, apply changes to production

### CI/CD Integration

Integration with GitHub Actions:

```yaml
name: Infrastructure CI/CD

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
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
      
      - name: Terraform Format
        run: terraform fmt -check -recursive
      
      - name: Terraform Init
        run: terraform init -backend=false
      
      - name: Terraform Validate
        run: terraform validate

  plan:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    needs: validate
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      # Determine environment from branch
      - name: Set environment
        id: set-env
        run: |
          if [[ "${{ github.base_ref }}" == "main" ]]; then
            echo "env=prod" >> $GITHUB_OUTPUT
          else
            echo "env=dev" >> $GITHUB_OUTPUT
          fi
      
      - name: Terraform Init
        run: terraform init
        
      - name: Select Workspace
        run: terraform workspace select ${{ steps.set-env.outputs.env }} || terraform workspace new ${{ steps.set-env.outputs.env }}
      
      - name: Terraform Plan
        run: |
          terraform plan \
            -var-file=envs/common.tfvars \
            -var-file=envs/${{ steps.set-env.outputs.env }}.tfvars \
            -out=tfplan
      
      - name: Upload Plan
        uses: actions/upload-artifact@v4
        with:
          name: tfplan
          path: tfplan

  apply:
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    needs: validate
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      # Determine environment from branch
      - name: Set environment
        id: set-env
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo "env=prod" >> $GITHUB_OUTPUT
          else
            echo "env=dev" >> $GITHUB_OUTPUT
          fi
      
      - name: Terraform Init
        run: terraform init
        
      - name: Select Workspace
        run: terraform workspace select ${{ steps.set-env.outputs.env }} || terraform workspace new ${{ steps.set-env.outputs.env }}
      
      - name: Terraform Apply
        run: |
          terraform apply \
            -var-file=envs/common.tfvars \
            -var-file=envs/${{ steps.set-env.outputs.env }}.tfvars \
            -auto-approve
```

For Pulumi:

```yaml
name: Pulumi CI/CD

on:
  pull_request:
    branches: [development, main]
  push:
    branches: [development, main]

jobs:
  preview:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm install
      
      # Determine stack from branch
      - name: Set stack
        id: set-stack
        run: |
          if [[ "${{ github.base_ref }}" == "main" ]]; then
            echo "stack=prod" >> $GITHUB_OUTPUT
          else
            echo "stack=dev" >> $GITHUB_OUTPUT
          fi
      
      - uses: pulumi/actions@v4
        with:
          command: preview
          stack-name: ${{ steps.set-stack.outputs.stack }}
        env:
          PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: us-east-1

  update:
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm install
      
      # Determine stack from branch
      - name: Set stack
        id: set-stack
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo "stack=prod" >> $GITHUB_OUTPUT
          else
            echo "stack=dev" >> $GITHUB_OUTPUT
          fi
      
      - uses: pulumi/actions@v4
        with:
          command: up
          stack-name: ${{ steps.set-stack.outputs.stack }}
        env:
          PULUMI_ACCESS_TOKEN: ${{ secrets.PULUMI_ACCESS_TOKEN }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: us-east-1
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
  name = "/${var.project}/${terraform.workspace}/db/password"
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
  name        = "${var.project}-${terraform.workspace}-lambda-policy"
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

## Version Management

Infrastructure code version management best practices:

1. **Pin Provider Versions**: Always specify exact provider versions in `versions.tf`:
   ```hcl
   terraform {
     required_providers {
       aws = {
         source  = "hashicorp/aws"
         version = "= 5.11.0"
       }
     }
     required_version = "= 1.7.0"
   }
   ```

2. **Lock Files**: Commit `.terraform.lock.hcl` files to git to ensure consistent provider versions across the team

3. **Versioned Releases**: Tag infrastructure repositories with semantic version numbers after significant changes

4. **Upgrade Process**:
   - Plan upgrades in advance
   - Test in lower environments first
   - Document any breaking changes
   - Maintain a changelog

5. **Update Schedule**:
   - Review provider updates monthly
   - Apply security patches immediately
   - Plan major version upgrades quarterly

By following these practices, you'll maintain a consistent, reliable infrastructure codebase that evolves with your needs while minimizing disruption.