# Deploying My First Web Server on AWS Using Terraform

## 🚀 Introduction
In this project, I deployed a basic web server on AWS using Terraform. This hands-on exercise helped me understand Infrastructure as Code (IaC), cloud provisioning, and automation.

---

## 🧰 Tools Used
- Terraform
- AWS (EC2, VPC, Security Groups)
- Draw.io (for architecture diagram)

---

## 🌍 Step 1: Provider Configuration

```hcl
provider "aws" {
  region = "us-east-1"
}
```

---

## 🌐 Step 2: Create a VPC

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}
```

---

## 🧩 Step 3: Create a Public Subnet

```hcl
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
}
```

---

## 🌍 Step 4: Internet Gateway

```hcl
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}
```

---

## 🛣️ Step 5: Route Table

```hcl
resource "aws_route_table" "rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}
```

---

## 🔗 Step 6: Associate Route Table

```hcl
resource "aws_route_table_association" "rta" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.rt.id
}
```

---

## 🔐 Step 7: Security Group

```hcl
resource "aws_security_group" "web_sg" {
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

## 🖥️ Step 8: EC2 Instance

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

resource "aws_instance" "web" {
  ami                         = data.aws_ami.amazon_linux.id
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.public.id
  vpc_security_group_ids      = [aws_security_group.web_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              echo "<h1>Hello from Terraform!</h1>" > /var/www/html/index.html
              EOF
}
```

---

## 🌐 Step 9: Output Public IP

```hcl
output "public_ip" {
  value = aws_instance.web.public_ip
}
```

---

## 🏗️ Architecture Overview

- AWS Cloud (us-east-1)
- Custom VPC
- Public Subnet
- Internet Gateway
- Route Table
- Security Group (Port 80 open)
- EC2 Instance (Web Server)

---

## 🎯 Conclusion

This project helped me understand:
- How to provision infrastructure using Terraform
- Networking basics in AWS (VPC, subnets, routing)
- Automating server setup with user data scripts

---

## 📊 Next Steps
- Add HTTPS (port 443)
- Use Load Balancer
- Deploy multiple instances
