# Task: Terraform Basics & Resource Provisioning
This file documents the completion of the Terraform foundational task under the 90 Days of DevOps challenge.

## 1. Installation Steps
Downloaded the Terraform binary package for Windows x64.

Configured the system environment variables path to access Terraform globally via the command line.

Verified the installation using the terminal:

Bash
terraform -version

## 2. Configuration File (main.tf)
The following configuration was created to provision a basic AWS EC2 instance using a stable Ubuntu 24.04 LTS image:

Terraform
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

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }
}

resource "aws_instance" "my_ec2" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"

  tags = {
    Name = "90DaysOfDevOps-EC2"
  }
}

## 3. Terraform Apply Output
The project directory was initialized using terraform init, and the infrastructure configuration was applied.

Terminal Output:
Plaintext
PS E:\terraform\terraform_1.15.7_windows_amd64\terraform-demo> terraform apply
No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
Screenshot Evidence:
Below is the execution screenshot from the VS Code terminal verifying the successful state comparison:

Note to add your photo: Save your terminal screenshot inside your project folder as terraform_apply.png so it displays perfectly right above!

## 4. Interview Answers
### Q1: How does Terraform manage resource creation and state?
Answer: Terraform tracks live cloud resources using a local or remote tracker called the State File (terraform.tfstate). When you run commands, Terraform compares your written code templates, the existing logs inside your state file, and the real-world settings pulled directly from your cloud provider's API. It calculates any differences and only modifies what is necessary to make reality match your code.

### Q2: What is the significance of the terraform init command in a new project?
Answer: terraform init is the mandatory first step for any project directory. It scans your code files, detects which cloud platforms you are using (like AWS), and automatically downloads the proper provider plugin binaries into a hidden .terraform folder so your commands can communicate with your cloud account.
