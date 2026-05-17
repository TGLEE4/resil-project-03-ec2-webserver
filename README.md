# Project 3: EC2 Web Server

## Overview
Provisioned a live EC2 web server on AWS using Terraform. The server runs Ubuntu 22.04 LTS with Nginx installed automatically on first boot via a user data script. The webpage is accessible from any browser via a public IP address.

---

## Goals
- Launch a virtual server (EC2) on AWS using Terraform
- Configure a security group to control inbound and outbound traffic
- Automatically install and start Nginx on first boot using user data
- SSH into the server and verify Nginx is running
- Access the live webpage from a browser

---

## Tools Used
| Tool | Purpose |
|---|---|
| Terraform | Infrastructure as Code — provisioned all AWS resources |
| AWS EC2 | Virtual server running Ubuntu 22.04 LTS |
| AWS Security Group | Firewall controlling traffic on ports 22 and 80 |
| AWS Key Pair | SSH authentication to access the server |
| Nginx | Web server serving the webpage on port 80 |
| AWS CLI | Authenticated Terraform with AWS |
| Git + GitHub | Version controlled all infrastructure code |

---

## Infrastructure

### Security Group
- **Port 22 (SSH)** — allows remote login to the server
- **Port 80 (HTTP)** — allows the world to access the webpage
- **Egress** — allows all outbound traffic from the server

### EC2 Instance
- **AMI** — Ubuntu 22.04 LTS (official Canonical image)
- **Instance Type** — t2.micro (AWS Free Tier eligible)
- **Key Pair** — resil-ec2-key (RSA, chmod 400)
- **User Data** — bash script that installs and starts Nginx on first boot

---

## Steps Taken

### 1. Created Project Structure
- Initialized Git
- Created `.gitignore` to exclude `.terraform/`, state files, `.tfvars`, and `.pem` files

### 2. Wrote Terraform Code
- `variables.tf` — defined reusable variables for region, instance type, and key name
- `main.tf` — defined the security group and EC2 instance
- `outputs.tf` — configured Terraform to print the public IP and DNS after deployment

### 3. Created SSH Key Pair
- Generated key pair using AWS CLI
- Saved private key as `resil-ec2-key.pem`
- Set permissions to `chmod 400` to satisfy SSH security requirements

### 4. Deployed Infrastructure
- Ran `terraform init` to download the AWS provider plugin
- Ran `terraform plan` to preview the infrastructure before building
- Ran `terraform apply` to provision the security group and EC2 instance on AWS

### 5. Verified the Server
- SSH'd into the EC2 instance using the private key
- Ran `systemctl status nginx` to confirm Nginx was active and running
- Visited the public IP in a browser and confirmed the Nginx welcome page loaded

### 6. Destroyed Infrastructure
- Ran `terraform destroy` to terminate the EC2 and delete the security group
- All infrastructure code remains in GitHub and can be redeployed with `terraform apply`

---

## Lessons Learned
- Always get the latest official Ubuntu AMI dynamically from Canonical's AWS account instead of hardcoding an AMI ID
- The SSH username for Ubuntu AMIs is `ubuntu` — differs by OS (Amazon Linux uses `ec2-user`)
- `chmod 400` is required on `.pem` files — SSH will refuse to connect if permissions are too open
- `user_data` bootstraps the server automatically on first boot — no manual installation needed
- Always run `terraform destroy` after each session to avoid unexpected AWS charges
- Keep `.pem` files inside their project folder and never push them to GitHub

---

## File Structure

---

## How to Deploy

```bash
# Get latest Ubuntu AMI
aws ec2 describe-images \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
  --query "sort_by(Images, &CreationDate)[-1].ImageId" \
  --output text

# Create key pair
aws ec2 create-key-pair \
  --key-name resil-ec2-key \
  --query 'KeyMaterial' \
  --output text > resil-ec2-key.pem
chmod 400 resil-ec2-key.pem

# Deploy
terraform init
terraform plan
terraform apply

# SSH into server
ssh -i resil-ec2-key.pem ubuntu@<public_ip>

# Destroy when done
terraform destroy
```

---

## Author
**Teng Lee**