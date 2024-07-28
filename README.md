# Multi-Tier Architecture on AWS using Terraform

Deploy a scalable and resilient multi-tier architecture on AWS using Terraform.

# ![](https://www.jenkins.io/images/logos/JCa)

# 📌What is Terraform ?

Terraform is an infrastructure as code tool that lets you define both cloud and on-prem resources in human-readable configuration files that you can version, reuse, and share. You can then use a consistent workflow to provision and manage all of your infrastructure throughout its lifecycle.

# 🚀 Multi-Tier Architecture Overview

A multi-tier architecture typically consists of three layers: a presentation layer (web server), an application layer (app server), and a data layer (database server). Each layer serves a specific purpose and can be scaled independently, providing flexibility and efficiency.

- **Web Tier:** This tier handles incoming user requests and can be horizontally scaled for increased capacity. It typically includes web servers and a load balancer for distributing traffic.

- **Application Tier:** Application servers run our business logic and interact with the database tier. They can also be horizontally scaled to meet demand.

- **Database Tier:** The database stores and manages our application data. In this architecture, we use Amazon RDS for a managed database service.


# 🚦 Getting Started

## Prerequisites

Before you get started, make sure you have the following prerequisites in place:

- Terraform installed.
- AWS IAM credentials configured.

# ✨ Features

- **High Availability:** The architecture is designed for fault tolerance and redundancy.

- **Scalability:** Easily scale the web and application tiers to handle varying workloads.

- **Security:** Security groups and network ACLs are configured to ensure a secure environment.

# 🔧 Terraform Configuration
The Terraform configuration for this project is organized into different sections and resources to create the necessary AWS infrastructure components. Key resources include:

- Virtual Private Cloud (VPC)
- Subnets and Route Tables
- Security Groups and Network ACLs
- Load Balancers
- Auto Scaling Groups
- RDS Database Instances

# 🚀 Deployment

Follow these steps to deploy the architecture:

Creating a Separate folder for this Project is Recommended.

1. Provider Section

 
 What is Providers?

A provider in Terraform is a plugin that enables interaction with an API. This includes Cloud providers and Software-as-a-service providers. The providers are specified in the Terraform configuration code. They tell Terraform which services it needs to interact with.

- Create a provider.tf file using the below code

```bash
  provider "aws" {
  region = var.region-name
}
```

2. VPC Section

We are going to create a Separate VPC for deploying our Architecture.

What is VPC?

Amazon Virtual Private Cloud (Amazon VPC) gives you full control over your virtual networking environment, including resource placement, connectivity, and security. Get started by setting up your VPC in the AWS service console.

Step 1: Creating VPC

- Create a vpc.tf file for creating VPC

```bash
resource "aws_vpc" "vpc" {
  cidr_block = var.vpc-cidr-block
  tags = {
    Name = var.vpc-name
  }
}
```
**Step 2: Creating Subnets**

- We have to Create two subnets for each tiers.
- Create a file web-subnets.tf file for creating Web tier Subnets

```bash
  resource "aws_subnet" "web-subnet1" {
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = var.web-subnet1-cidr
  availability_zone       = var.az-1
  map_public_ip_on_launch = true

  tags = {
    Name = var.web-subnet1-name
  }
}

resource "aws_subnet" "web-subnet2" {
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = var.web-subnet2-cidr
  availability_zone       = var.az-2
  map_public_ip_on_launch = true

  tags = {
    Name = var.web-subnet2-name
  }
}
```

- Create a file app-subnets.tf file for creating App tier Subnets


```bash
  resource "aws_subnet" "app-subnet1" {
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = var.app-subnet1-cidr
  availability_zone       = var.az-1
  map_public_ip_on_launch = false

  tags = {
    Name = var.app-subnet1-name
  }
}

resource "aws_subnet" "app-subnet2" {
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = var.app-subnet2-cidr
  availability_zone       = var.az-2
  map_public_ip_on_launch = false

  tags = {
    Name = var.app-subnet2-name
  }
}
```

- Create a file db-subnets.tf file for creating Database tier Subnets

```bash
  resource "aws_subnet" "db-subnet1" {
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = var.db-subnet1-cidr
  availability_zone       = var.az-1
  map_public_ip_on_launch = false

  tags = {
    Name = var.db-subnet1-name
  }
}

resource "aws_subnet" "db-subnet2" {
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = var.db-subnet2-cidr
  availability_zone       = var.az-2
  map_public_ip_on_launch = true

  tags = {
    Name = var.db-subnet2-name
  }
}
```
**Step 3:  Creating Internet gateway**

 - Create a internet-gw.tf file for creating Internet Gateway

```bash
  resource "aws_internet_gateway" "internet-gw" {
  vpc_id = aws_vpc.vpc.id
  tags = {
    Name = var.igw-name
  }
}
```



**Step 4: Creating NAT Gateway**

- Before going to create a NAT Gateway , We need Elastic IP for creating NAT gateway. Create a eip.tf file for creating Elastic IP

```bash
resource "aws_eip" "eip" {
  domain = "vpc"
}
```
- Create a nat-gw.tf file for creating NAT Gateway

```bash
  resource "aws_nat_gateway" "nat-gw" {
  allocation_id     = aws_eip.eip.id
  connectivity_type = "public"
  subnet_id         = aws_subnet.web-subnet1.id

  tags = {
    Name = var.nat-gw-name
  }

  depends_on = [aws_internet_gateway.internet-gw]
}
```
**Step 5: Creating Route Tables and making Route table association**

- We have to Create route tables for Routing traffic
- Create a public-rt.tf file for creating Route table for Web Subnets , Since the Web Servers have to connected to Internet.

```bash
  resource "aws_route_table" "public-route-table" {
  vpc_id = aws_vpc.vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.internet-gw.id
  }

  tags = {
    Name = var.public-rt-name
  }
}

resource "aws_route_table_association" "pub-rt-asscociation-1" {
  subnet_id      = aws_subnet.web-subnet1.id
  route_table_id = aws_route_table.public-route-table.id
}

resource "aws_route_table_association" "pub-rt-asscociation-2" {
  subnet_id      = aws_subnet.web-subnet2.id
  route_table_id = aws_route_table.public-route-table.id
}
```
- Create a private-rt.tf file for creating Route table for App Subnets , So the Application Severs can be able connect to Internet.

```bash
  resource "aws_route_table" "private-route-table" {
  vpc_id = aws_vpc.vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_nat_gateway.nat-gw.id
  }

  tags = {
    Name = var.private-rt-name
  }
}

resource "aws_route_table_association" "pri-rt-asscociation-1" {
  subnet_id      = aws_subnet.app-subnet1.id
  route_table_id = aws_route_table.private-route-table.id
}

resource "aws_route_table_association" "pri-rt-asscociation-2" {
  subnet_id      = aws_subnet.app-subnet2.id
  route_table_id = aws_route_table.private-route-table.id
}
```
**3. Web Tier Section**

The Web Tier is the entry point for incoming user requests. It typically includes:

- **Load Balancer:** Distributes traffic across multiple web servers.

- **Auto Scaling:** Automatically adjusts the number of web servers based on traffic.

- **Security Groups:** Controls incoming and outgoing traffic to the web servers.

**Web Tier Configurations**

- **Step 1:** Create a lauch-template-web.tf file for creating Launch template for ASG in Web tier
 
```bash
  resource "aws_launch_template" "template-web" {
  name          = var.launch-template-web-name
  image_id      = var.image-id
  instance_type = var.instance-type
  key_name      = var.key-name

  network_interfaces {
    device_index    = 0
    security_groups = [aws_security_group.asg-security-group-web.id]
  }

  user_data = filebase64("user-data.sh")
  tag_specifications {

    resource_type = "instance"
    tags = {
      Name = var.web-instance-name
    }
  }
}
```
- **Step 2:** Create a user-data.sh script file file for user data

```bash
#!/bin/bash
# Use this for your user data (script from top to bottom)
# install httpd (Linux 2 version)
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
```
- **Step 3:** Create a alb-web.tf file for creating Application Load balancer for Web tier

```bash
  resource "aws_lb" "alb-web" {
  name               = var.alb-web-name
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb-security-group-web.id]
  subnets            = [aws_subnet.web-subnet1.id, aws_subnet.web-subnet2.id]
}
```

- **Step 4:** Create a tg-web.tf file for creating Target Group Configuration for Load balancer


```bash
  resource "aws_lb_target_group" "target-group-web" {
  name     = var.tg-web-name
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.vpc.id
  health_check {
    path    = "/"
    matcher = 200

  }
}

resource "aws_lb_listener" "alb_listener-web" {
  load_balancer_arn = aws_lb.alb-web.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.target-group-web.arn
  }
}
```

- **Step 5:** Create a asg-web.tf file for creating Auto Scaling Group for Web tier


```bash
  resource "aws_autoscaling_group" "asg-web" {
  name                = var.asg-web-name
  desired_capacity    = 2
  max_size            = 4
  min_size            = 1
  target_group_arns   = [aws_lb_target_group.target-group-web.arn]
  health_check_type   = "EC2"
  vpc_zone_identifier = [aws_subnet.web-subnet1.id, aws_subnet.web-subnet2.id]


  launch_template {
    id      = aws_launch_template.template-web.id
    version = aws_launch_template.template-web.latest_version
  }
}
```

- **Step 6:** Create a alb-sg-web.tf file for creating Security Group Configuration for Load balancer


```bash
  resource "aws_security_group" "alb-security-group-web" {
  name        = var.alb-sg-web-name
  description = "ALB Security Group"
  vpc_id      = aws_vpc.vpc.id

  ingress {
    description = "HTTP from Internet"
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

  tags = {
    Name = var.alb-sg-web-name
  }
}
```

- **Step 7:** Create a asg-sg-web.tf file for creating Security Group Configuration for Auto Scaling Group

```bash
  resource "aws_security_group" "asg-security-group-web" {
  name        = var.asg-sg-web-name
  description = "ASG Security Group"
  vpc_id      = aws_vpc.vpc.id

  ingress {
    description     = "HTTP from ALB"
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.alb-security-group-web.id]
  }

  ingress {
    description = "SSH From Anywhere or Your-IP"
    from_port   = 22
    to_port     = 22
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
    Name = var.asg-sg-web-name
  }
}
```

**4. App Tier Section**

The Application Tier hosts the application servers responsible for running business logic and interacting with the database tier. Key components include:

- **Application Servers:** These run your application code and can be horizontally scaled.

- **Load Balancer:** Distributes traffic to the application servers.

- **Auto Scaling:** Automatically adjusts the number of web servers based on traffic.

- **Security Groups:** Controls incoming and outgoing traffic to the application servers.

**Application Tier Configurations**

- **Step 1:** Create a lauch-template-app.tf file for creating Launch template for ASG in App tier


```bash
  resource "aws_launch_template" "template-app" {
  name          = var.launch-template-app-name
  image_id      = var.image-id
  instance_type = var.instance-type
  key_name      = var.key-name

  network_interfaces {
    device_index    = 0
    security_groups = [aws_security_group.asg-security-group-app.id]
  }

  tag_specifications {

    resource_type = "instance"
    tags = {
      Name = var.app-instance-name
    }
  }
}
```
- **Step 2:** Create a alb-app.tf file for creating Application Load balancer for

```bash
  resource "aws_lb" "alb-app" {
  name               = var.alb-app-name
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb-security-group-app.id]
  subnets            = [aws_subnet.app-subnet1.id, aws_subnet.app-subnet2.id]
}
```

- **Step 3:** Create a tg-app.tf file for creating Target Group Configuration for Load balancer

```bash
  resource "aws_lb_target_group" "target-group-app" {
  name     = var.tg-app-name
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.vpc.id
  health_check {
    path    = "/"
    matcher = 200

  }
}

resource "aws_lb_listener" "alb_listener-app" {
  load_balancer_arn = aws_lb.alb-app.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.target-group-app.arn
  }
}
```
- **Step 4:** Create a asg-web.tf file for creating Auto Scaling Group for Web tier

```bash
  resource "aws_autoscaling_group" "asg-app" {
  name                = var.asg-app-name
  desired_capacity    = 2
  max_size            = 4
  min_size            = 1
  target_group_arns   = [aws_lb_target_group.target-group-app.arn]
  health_check_type   = "EC2"
  vpc_zone_identifier = [aws_subnet.app-subnet1.id, aws_subnet.app-subnet2.id]


  launch_template {
    id      = aws_launch_template.template-app.id
    version = aws_launch_template.template-app.latest_version
  }
}
```
- **Step 5:** Create a alb-sg-app.tf file for creating Security Group Configuration for Load balancer

```bash
  resource "aws_security_group" "alb-security-group-app" {
  name        = var.alb-sg-app-name
  description = "ALB Security Group"
  vpc_id      = aws_vpc.vpc.id

  ingress {
    description     = "HTTP from Internet"
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.asg-security-group-web.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = var.alb-sg-app-name
  }
}
```
- **Step 6:** Create a asg-sg-app.tf file for creating Security Group Configuration for Auto Scaling Group

```bash
  resource "aws_security_group" "asg-security-group-app" {
  name        = var.asg-sg-app-name
  description = "ASG Security Group"
  vpc_id      = aws_vpc.vpc.id

  ingress {
    description     = "HTTP from ALB"
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.alb-security-group-app.id]
  }

  ingress {
    description     = "SSH From Anywhere or Your-IP"
    from_port       = 22
    to_port         = 22
    protocol        = "tcp"
    security_groups = [aws_security_group.asg-security-group-web.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = var.asg-sg-app-name
  }
}
```

5. **Database Tier Section**

The Database Tier stores and manages our application data. We use Amazon RDS for a managed database service. Key components include:

- **Amazon RDS:** A managed database service for MySQL/PostgreSQL/SQL Server databases.

- **Security Groups:** Control incoming and outgoing traffic to the database.

**Database Tier Configurations**

- **Step 1:** Create a db-subnet-group.tf file for creating DB Subnet Group for our RDS Database

```bash
  resource "aws_db_subnet_group" "subnet-grp" {
  name       = var.db-subnet-grp-name
  subnet_ids = [aws_subnet.db-subnet1.id,aws_subnet.db-subnet2.id]

  tags = {
    Name = var.db-subnet-grp-name
  }
}
```

- **Step 2:** Create a rds.tf file for creating RDS Database

```bash
  resource "aws_db_instance" "rds-db" {
  allocated_storage      = 10
  db_name                = var.db-name
  engine                 = "mysql"
  engine_version         = "5.7"
  instance_class         = var.instance-class
  username               = var.db-username
  password               = var.db-password
  parameter_group_name   = "default.mysql5.7"
  multi_az               = true
  db_subnet_group_name   = aws_db_subnet_group.subnet-grp.name
  vpc_security_group_ids = [aws_security_group.db-sg.id]
  skip_final_snapshot    = true
}
```
- **Step 3:** Create a db-sg.tf file for creating Security Group Configuration for RDS

```bash
  resource "aws_security_group" "db-sg" {
  name        = var.db-sg-name
  description = "DB SEcurity Group"
  vpc_id      = aws_vpc.vpc.id

  ingress {
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.asg-security-group-app.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = var.db-sg-name
  }
}
```
6. **Variables Section**

If we examine the files we have created so far, we have not hardcoded any values; instead, we have employed the approach of using variables. Now, we are going to assign values to these variables

**Step 1:** Create a variables.tf file for Variables Declaration

```bash
  variable "region-name" {
  description = "Region name"
}
variable "vpc-cidr-block" {
  description = "CIDR Block for VPC"
}

variable "vpc-name" {
  description = "Name for Virtual Private Cloud"
}

variable "igw-name" {
  description = "Name for Internet Gateway"
}

variable "nat-gw-name" {
  description = "Name for NAT Gateway"
}

variable "web-subnet1-cidr" {
  description = "CIDR Block for Web-tier Subnet-1"
}

variable "web-subnet1-name" {
  description = "Name for Web-tier Subnet-1"
}

variable "web-subnet2-cidr" {
  description = "CIDR Block for Web-tier Subnet-2"
}

variable "web-subnet2-name" {
  description = "Name for Web-tier Subnet-2"
}

variable "app-subnet1-cidr" {
  description = "CIDR Block for Application-tier Subnet-1"
}

variable "app-subnet1-name" {
  description = "Name for app-tier Subnet-1"
}

variable "app-subnet2-cidr" {
  description = "CIDR Block for Application-tier Subnet-2"
}

variable "app-subnet2-name" {
  description = "Name for Application-tier Subnet-2"
}


variable "db-subnet1-cidr" {
  description = "CIDR Block for Database-tier Subnet-1"
}

variable "db-subnet1-name" {
  description = "Name for Database-tier Subnet-1"
}

variable "db-subnet2-cidr" {
  description = "CIDR Block for Database-tier Subnet-2"
}

variable "db-subnet2-name" {
  description = "Name for Database-tier Subnet-2"
}

variable "az-1" {
  description = "Availabity Zone 1"
}

variable "az-2" {
  description = "Availabity Zone 2"
}

variable "public-rt-name" {
  description = "Name for Public Route table"
}

variable "private-rt-name" {
  description = "Name for Private Route table"
}

variable "launch-template-web-name" {
  description = "Name for Launch-template-1"
}

variable "image-id" {
  description = "Value for Image-id"
}

variable "instance-type" {
  description = "Value for Instance type"
}

variable "key-name" {
  description = "Value for Key name"
}

variable "web-instance-name" {
  description = "Value for Web Instances"
}

variable "alb-web-name" {
  description = "Name the Load Balancer for the Web Tier"
}

variable "alb-sg-web-name" {
  description = "Name for alb security group 1"
}

variable "asg-web-name" {
  description = "Name the Auto Scaling group in Web Tier"
}

variable "asg-sg-web-name" {
  description = "Name for asg security group 1"
}

variable "tg-web-name" {
  description = "Name for Target group web"
}

variable "launch-template-app-name" {
  description = "Name for Launch-template-1"
}

variable "app-instance-name" {
  description = "Value for App Instances"
}

variable "alb-app-name" {
  description = "Name the Load Balancer for the App Tier"
}

variable "alb-sg-app-name" {
  description = "Name for alb security group 1"
}

variable "asg-app-name" {
  description = "Name the Auto Scaling group in app Tier"
}

variable "asg-sg-app-name" {
  description = "Name for asg security group 1"
}

variable "tg-app-name" {
  description = "Name for Target group app"
}

variable "db-username" {
  description = "Username for db instance"
}

variable "db-password" {
  description = "Password for db instance"
}

variable "db-name" {
  description = "Name for Database"
}

variable "instance-class" {
  description = "Value for DB instance class"
}

variable "db-sg-name" {
  description = "Name for DB Security group"
}

variable "db-subnet-grp-name" {
  description = "Name for DB Subnet Group"
}

variable "app-db-sg-name" {
  description = "Name for App-DB SEcurity group"
}
```
**Step 2:** Create a terraform.tfvars file for Assigning Values to Variables

```bash
  region-name              = "us-east-1"
vpc-cidr-block           = "10.0.0.0/16"
vpc-name                 = "three-tier-vpc"
igw-name                 = "three-tier-igw"
nat-gw-name              = "three-tier-nat-gw"
web-subnet1-cidr         = "10.0.1.0/24"
web-subnet1-name         = "three-tier-web-subnet-1"
web-subnet2-cidr         = "10.0.2.0/24"
web-subnet2-name         = "three-tier-web-subnet-2"
app-subnet1-cidr         = "10.0.3.0/24"
app-subnet1-name         = "three-tier-app-subnet-1"
app-subnet2-cidr         = "10.0.4.0/24"
app-subnet2-name         = "three-tier-app-subnet-2"
db-subnet1-cidr          = "10.0.5.0/24"
db-subnet1-name          = "three-tier-db-subnet-1"
db-subnet2-cidr          = "10.0.6.0/24"
db-subnet2-name          = "three-tier-db-subnet-2"
az-1                     = "us-east-1a"
az-2                     = "us-east-1b"
public-rt-name           = "three-tier-public-route-table"
private-rt-name          = "three-tier-private-rote-table"
launch-template-web-name = "three-tier-launch-template-web"
image-id                 = "ami-0df435f331839b2d6"
instance-type            = "t2.micro"
key-name                 = "jenkins"
web-instance-name        = "three-tier-web-instances"
alb-web-name             = "three-tier-alb-web"
alb-sg-web-name          = "three-tier-alb-sg-web"
asg-web-name             = "three-tier-asg-web"
asg-sg-web-name          = "three-tier-asg-sg-web"
tg-web-name              = "three-tier-tg-web"
launch-template-app-name = "three-tier-launch-template-app"
app-instance-name        = "three-tier-app-instances"
alb-app-name             = "three-tier-alb-app"
alb-sg-app-name          = "three-tier-alb-sg-app"
asg-app-name             = "three-tier-asg-app"
asg-sg-app-name          = "three-tier-asg-sg-app"
tg-app-name              = "three-tier-tg-app"
db-name                  = "mydb"
instance-class           = "db.t3.micro"
db-sg-name               = "three-tier-db-sg"
db-subnet-grp-name       = "three-tier-db-subnet-grp"
app-db-sg-name           = "three-tier-app-db"
```
**Step 3:** Create a secret.tfvars file for Assigning Values to Username and Password Variables

- We need keep DB username and DB Password safe , So that’s why we assign this values in separate secret.tfvars file

```bash
  db-username = "dbusername"
db-password = "dbpassword123456789"
```
**7. Output Section**

- **Step 1:** Create a output.tf file for getting Web Server ALB DNS as Output

```bash
  output "web-server-dns" {
  value = aws_lb.alb-web.dns_name
}
```

**8. Remote Backend Configuration**

**Step 1:** Create a backend.tf file for Remote Backend

```bash
  terraform {
  backend "s3" {
    bucket = "three-tier-architecture-bucket"
    key    = "terraform.tfstate"
    region = "us-east-1"
  }
}
```
**9. Deploying the Architecture**

- Open the terminal
- Go to the folder where the terraform configurations files are saved
- Initialize the terraform directory by run this below command:

```bash
  terraform init  
```
- See what are the resources are going to be created using the below command:

```bash
  terraform plan -var-file=secret.tfvars
```
- Run the terraform apply command for Creating Resources in AWS
```bash
  terraform apply -var-file=secret.tfvars
```
**10. Resource Creation Verification**

- Now , You can Log into your AWS account and check whether our resources are created or not.

**11. Multi Tier Architecture Verification**

You can Verify our work by the below methods,

- Paste the Output DNS into your browser and check whether our Web application is running or not.
- Try to log into Application Servers through our Web Servers using SSH
- Try to log into MySQL Server from Application Server.

By this methods you can verify the Architecture work.

If you like this Post , Don’t forget to give the applause 👏.

Let’s Connect on LinkedIn : https://www.linkedin.com/in/imshiva411/
