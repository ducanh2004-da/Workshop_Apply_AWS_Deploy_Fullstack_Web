---
title: "AWS Cloud Architecture: Deploying AI-Integrated Full-Stack Applications"
weight: 1
chapter: false
---

# Introduction

#### Executive Summary
This project demonstrates the deployment of a modern, high-availability full-stack AI web application (ReactJS, NestJS, PostgreSQL, Python AI) on AWS. The objective is to implement and compare two distinct infrastructure patterns: a **Service-Oriented Architecture** leveraging AWS managed services, and a **Secure Network Architecture** focusing on strict VPC segmentation and self-hosted resources.

---

#### Lab 01: Service-Oriented Architecture (Managed Services Focus)
*Architecture: Frontend on S3 (Static Hosting) | Backend & AI on EC2 | Database on RDS*

In this approach, the focus is on leveraging AWS managed services to reduce operational overhead and ensure high availability for the database layer.

1. **VPC Network Design:** Designed a custom VPC with a Multi-AZ architecture. Configured Public and Private subnets with custom Route Tables and Internet Gateways to manage traffic flow effectively.

2. **AI Service Containerization (Docker on EC2):** Provisioned a `t2.medium` EC2 instance to host the Python-based AI model. Containerized the AI service using Docker to ensure environment consistency and seamless integration with the backend.

3. **Backend Deployment (Node.js/NestJS on EC2):** Deployed the NestJS API on a `t2.micro` instance. Configured Prisma ORM for database schema management and established secure connectivity between the Backend EC2 and the RDS instance.

4. **Frontend Distribution (S3 Static Hosting):** Optimized cost and performance by hosting the ReactJS build artifacts on Amazon S3. Configured Bucket Policies for public read access, serving the UI directly to users while communicating with the backend API.

5. **Managed Database (Amazon RDS):** Provisioned a PostgreSQL instance using Amazon RDS to leverage automated backups and managed maintenance. Secured database access using strictly scoped Security Groups (allowing traffic only from the Backend EC2).

---

#### Lab 02: Secure Network Architecture (IaaS & VPC Segmentation Focus)
*Architecture: Frontend on Public EC2 (Nginx) | Backend & Database on Private EC2 | NAT Gateway*

This scenario simulates a highly secure enterprise environment where critical resources (Backend APIs and Databases) are isolated from the public internet, requiring advanced networking configuration.

1. **Advanced Networking & Security:** Architected a "VPC and More" setup with strict Public/Private subnet separation. Implemented a **NAT Gateway** to allow private instances to access the internet for updates/installations without exposing them to inbound traffic. Defined granular Inbound/Outbound Security Group rules to prevent unauthorized access.

2. **Frontend & Reverse Proxy (Public Subnet):** Provisioned a `t3.small` EC2 instance in the Public Subnet. Configured **Nginx** as a web server to host the ReactJS application, acting as the entry point for user traffic.

3. **Secure Backend & Self-Hosted DB (Private Subnet):** Provisioned a `t3.small` EC2 instance completely isolated in the Private Subnet. Deployed the NestJS backend and a **self-hosted PostgreSQL database (via Docker)** on this private instance. Demonstrated capability in managing database persistence and networking within a restricted environment.

4. **Cost Management & Resource Teardown:** Implemented lifecycle management policies to terminate Elastic IPs, NAT Gateways, and EC2 instances to prevent budget overruns.

---

#### Technical Skills Demonstrated
By completing these deployments, the following core cloud competencies were applied:

* **Infrastructure:** VPC Design, Subnetting (Public vs. Private), NAT Gateways, Route Tables.
* **Compute & Containers:** EC2 Administration (Ubuntu), Docker, Docker Compose.
* **Database Management:** Amazon RDS (Managed) vs. Self-hosted PostgreSQL (Docker).
* **Storage & Web Hosting:** Amazon S3 Static Website Hosting.
* **Security:** Security Groups, Network ACLs, IAM Roles.
* **Web Server Administration:** Nginx configuration and Reverse Proxy.

---

#### Lab Sections Navigation

1. [VPC Networking & Security (Section I)](1-VPC-networking/)
2. [AI Service on EC2 (Section II)](2-ai-on-ec2/)
3. [NestJS Backend on EC2 (Section III)](3-backend-NestJS-on-ec2/)
4. [ReactJS Frontend on S3 (Section IV)](4-frontend-ReactJS-on-s3/)
5. [PostgreSQL on RDS (Section V)](5-postgresql-on-rds/)
6. [Resource Cleanup (Section VI)](6-cleanup/)

---

*Ready to explore the infrastructure? Let’s dive into Section I.*