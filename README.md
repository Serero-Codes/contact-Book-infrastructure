# contact-Book-infrastructure

# AWS 3-Tier Enterprise Infrastructure (Terraform)

This repository contains the Infrastructure as Code (IaC) written in Terraform to provision a highly available, secure, and auto-scaling 3-tier cloud architecture on AWS. 

This infrastructure is designed to host the containerized **Contact Book** application. 
🔗 **Application Repository:** [Serero-Codes/Contact-Book](https://github.com/Serero-Codes/Contact-Book)

---

## 🏛️ Architecture Overview

The infrastructure is deployed in the **AWS Africa (Cape Town) region (`af-south-1`)** and enforces strict network isolation, security group chaining, and automated self-healing.

### Key Components

1. **Networking (VPC & Subnets)**
   * **VPC:** Custom Virtual Private Cloud (`10.0.0.0/16`) with DNS support.
   * **Public Subnets:** Two public subnets spanning multiple Availability Zones (`af-south-1a`, `af-south-1b`) hosting the Application Load Balancer (ALB) and a NAT Gateway.
   * **Private Subnets:** Two private subnets hosting the EC2 compute instances and the RDS database. No public IP addresses are assigned to the application or database tiers.
   * **Routing:** Internet Gateway for public ingress to the ALB; NAT Gateway for secure outbound internet access (required for instances to pull Docker images and OS updates).

2. **Compute & Load Balancing**
   * **Application Load Balancer (ALB):** Public-facing entry point routing HTTP traffic (Port 80) across Availability Zones.
   * **Auto Scaling Group (ASG):** Dynamically scales EC2 instances (`t3.micro`) between a minimum of 2 and maximum of 4 nodes. 
   * **Launch Template:** Uses Ubuntu 22.04 LTS. The `user_data` script automatically bootstraps Docker, installs Nginx, configures the reverse proxy, and launches the application container on boot.

3. **Data Tier**
   * **Amazon RDS for PostgreSQL:** Managed PostgreSQL 15 database (`db.t3.micro`) deployed inside an isolated database subnet group.

4. **Security & Access Management**
   * **Security Group Chaining:** 
     * `alb_sg`: Allows HTTP in from `0.0.0.0/0`.
     * `web_sg`: Allows HTTP in **only** from `alb_sg`.
     * `db_sg`: Allows PostgreSQL (Port 5432) in **only** from `web_sg`.
   * **IAM SSM Role:** EC2 instances are provisioned with the `AmazonSSMManagedInstanceCore` policy, completely eliminating the need for SSH (Port 22) and enabling GitHub Actions to trigger zero-downtime SSM deployments.

---

## 🚀 Deployment Guide

### Prerequisites
* [Terraform](https://developer.hashicorp.com/terraform/downloads) (`>= 1.5.0`) installed locally.
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) installed and configured with appropriate IAM permissions (`aws configure`).

### 1. Initialize Terraform
Downloads the required HashiCorp AWS provider plugins and initializes the state directory.
```bash
terraform init
