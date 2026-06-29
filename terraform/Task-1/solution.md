🚀 90 Days of DevOps: Infrastructure as Code with TerraformThis repository documents the structural implementation 
for the Terraform foundational provisioning task. 
It demonstrates installing the Terraform CLI on Windows, 
establishing a modular workspace configuration, 
deploying an AWS EC2 instance resource using dynamic Amazon Machine Images (AMIs), 
and managing declarative state files.  

📋 Table of ContentsEnvironment & PrerequisitesArchitecture WorkflowStep 
1: Environment & Prerequisites
2: Architecture Workflow
3: Step 1: Installation & Verification
4: Step 2: Configuration Manifest (main.tf)
5: Step 3: Validation & Execution Lifecycle
6: Step4: Resource Cleanup
7: Core Concepts & Interview Technical Breakdown

💻 Environment & Prerequisites
Operating System: Windows 11 Enterprise / Professional x64  
IDE/Editor: Visual Studio Code (VS Code)  
Required Extensions: HashiCorp Terraform (hashicorp.terraform) 
Authentication: AWS CLI pre-configured with IAM programmatic access (AdministratorAccess policy scope) via aws configure.

🏗️ Architecture Workflow
The following pipeline represents the deployment sequence initiated from the VS Code workstation
:Plaintext+-------------------+      terraform init       +-----------------------+
|  Local Workstation| ------------------------> | Download Provider SDK |
| (VS Code + Winget)|                           | (.terraform/plugins/) |
+-------------------+                           +-----------------------+
          |                                                 |
          | terraform apply                                 |
          v                                                 v
+-------------------+      Updates State Engine  +-----------------------+
|  AWS EC2 Instance | <------------------------ |  terraform.tfstate    |
| (Ubuntu 24.04 LTS)|                           | (Local Tracking File) |
+-------------------+                           +-----------------------+

Step 1: Installation & Verification
Terraform was deployed using the Windows Package Manager (winget) via an administrative PowerShell terminal embedded within VS Code:

PowerShell# Install the official HashiCorp Terraform binary distribution
winget install HashiCorp.Terraform

⚠️ Configuration Note: After the package installation completes, restart the VS Code window 
(Ctrl + Shift + P -> Developer: Reload Window) 
to safely force the terminal process to discover the newly appended system environment binary paths.  

Version Affirmation Check
Bash
terraform -version

Output Capture:
Plaintext
Terraform v1.10.3
on windows_amd64
+ provider registry.terraform.io/hashicorp/aws v5.31.0

Step 2: Configuration Manifest (main.tf)
The infrastructure definitions are structured within main.tf. 
This manifest dynamically queries Canonical's registry to retrieve the most up-to-date,
secure Ubuntu 24.04 LTS HVM hardware image matching the architecture profile: 

Terraform#---------------------------------------------------------
# Terraform Block: Provider Source & Version Constraints
#---------------------------------------------------------
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

#---------------------------------------------------------
# Provider Block: Target Deployment Region
#---------------------------------------------------------
provider "aws" {
  region = "us-east-1"
}

#---------------------------------------------------------
# Data Source: Dynamic Lookup for Latest Ubuntu 24.04 AMI
#---------------------------------------------------------
data "aws_ami" "ubuntu_latest" {
  most_recent = true
  owners      = ["099720109477"] # Canonical ID

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

#---------------------------------------------------------
# Resource Block: Compute Instance Provisioning
#---------------------------------------------------------
resource "aws_instance" "devops_node" {
  ami           = data.aws_ami.ubuntu_latest.id
  instance_type = "t2.micro"

  root_block_device {
    volume_size           = 8
    volume_type           = "gp3"
    encrypted             = true
    delete_on_termination = true
  }

  tags = {
    Name        = "90DaysOfDevOps-EC2"
    Environment = "Staging"
    ManagedBy   = "Terraform-IaC"
  }
}

Step 3: Validation & Execution Lifecycle
Execute the deployment lifecycle commands sequentially within the root directory containing your main.tf file: 
1. Initialize Working Directory

Bash
terraform init
Downloads the specific AWS provider plugins and locks version requirements in .terraform.lock.hcl.

2. Format & Linting Validation

Bash
# Canonicalize styles to match HashiCorp standard rules
terraform fmt

# Programmatically evaluate semantic structures and syntax errors
terraform validate

Expected Output: 
Success! The configuration is valid.

3. Execution Plan Compilation

Bash
terraform plan -out=deploy.tfplan
Generates an explicit execution blueprint detailing precisely which elements will be created, updated,
or destroyed without modifying live clouds.

4. Apply Changes to the Cloud

Bash
terraform apply deploy.tfplan
Executes structural cloud tasks synchronously and pipes back deployment metadata outputs.

Step 4: Resource Cleanup
To prevent ongoing billing charges on your cloud account, destroy the provisioned computing nodes when testing finishes:

Bash
terraform destroy
Prompts for authorization confirmation before systematically wiping the assigned resource blocks.

