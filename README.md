# Terraform Capstone Project: Automated WordPress Deployment on AWS

## Project: High-Performance WordPress Site for DigitalBoost

### 📄 Project Description
This project automates the deployment of a scalable, secure, and cost-effective WordPress website on AWS using Terraform. It includes VPC setup, public/private subnets, NAT Gateway, RDS MySQL, EFS, Application Load Balancer, and Auto Scaling.

---

## 📁 Directory Structure
```
terraform-wordpress/
├── main.tf
├── variables.tf
├── outputs.tf
├── vpc.tf
├── subnets.tf
├── nat_gateway.tf
├── security_groups.tf
├── rds.tf
├── efs.tf
├── alb.tf
├── autoscaling.tf
├── userdata.sh
└── README.md
```

---

## 🚀 Prerequisites
- AWS CLI configured
- Terraform installed (v1.0+)
- SSH key pair (for EC2 access)

---

## ⚙️ Steps and Code Summary

### 1. **VPC Setup** - `vpc.tf`
```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  enable_dns_support = true
  enable_dns_hostnames = true
  tags = { Name = "main-vpc" }
}
```

### 2. **Public & Private Subnets with NAT Gateway** - `subnets.tf`, `nat_gateway.tf`
```hcl
resource "aws_subnet" "public" { ... }
resource "aws_subnet" "private" { ... }

resource "aws_eip" "nat" { ... }
resource "aws_nat_gateway" "nat" { ... }

resource "aws_route_table" "private" {
  route { cidr_block = "0.0.0.0/0" gateway_id = aws_nat_gateway.nat.id }
}
```

### 3. **RDS MySQL** - `rds.tf`
```hcl
resource "aws_db_instance" "wordpress" {
  engine = "mysql"
  instance_class = "db.t3.micro"
  ...
  vpc_security_group_ids = [aws_security_group.rds_sg.id]
}
```

### 4. **EFS for WordPress** - `efs.tf`
```hcl
resource "aws_efs_file_system" "wordpress" { ... }
resource "aws_efs_mount_target" "wordpress" { ... }
```

### 5. **Application Load Balancer** - `alb.tf`
```hcl
resource "aws_lb" "app_lb" {
  name = "wordpress-alb"
  ...
}

resource "aws_lb_target_group" "wordpress_tg" { ... }
```

### 6. **Auto Scaling Group** - `autoscaling.tf`
```hcl
resource "aws_launch_template" "wordpress" { ... }
resource "aws_autoscaling_group" "wordpress_asg" {
  desired_capacity = 2
  max_size         = 4
  min_size         = 1
  ...
}
```

### 7. **User Data Script** - `userdata.sh`
```bash
#!/bin/bash
sudo apt update
sudo apt install -y apache2 php php-mysql mysql-client nfs-common
sudo mkdir -p /var/www/html
sudo mount -t nfs4 -o nfsvers=4.1 ${efs_dns}:/ /var/www/html
```

---

## 🔐 Security Measures
- **Security Groups:**
  - RDS only accessible from EC2
  - EC2 only accessible via ALB and SSH (restricted IP)
- **Private Subnets:** RDS and sensitive components are in private subnets
- **NAT Gateway:** Allows outbound traffic for private resources without exposing them

---

## 🧪 Demonstration Guide
1. Run `terraform init` and `terraform apply`
2. Access the ALB DNS name to visit the WordPress site
3. Simulate traffic using ApacheBench or similar tool:
   ```bash
   ab -n 1000 -c 50 http://<alb_dns>/
   ```
4. Observe Auto Scaling Group increasing instances (CloudWatch logs/console)

---

## 📌 Commands to Run
```bash
terraform init
terraform plan
terraform apply -auto-approve
```

---

## 📘 Variables Example - `variables.tf`
```hcl
variable "vpc_cidr" { default = "10.0.0.0/16" }
variable "public_subnet_cidr" { default = "10.0.1.0/24" }
variable "private_subnet_cidr" { default = "10.0.2.0/24" }
...
```

---

## 📤 Outputs - `outputs.tf`
```hcl
output "alb_dns_name" {
  value = aws_lb.app_lb.dns_name
}
```

---

## ✅ Final Notes
- Use `terraform destroy` to clean up
- All Terraform modules are reusable and modular
- Validate logs and scaling using AWS Console or CLI

---

Happy Deploying 🚀
