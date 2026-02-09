---
title: "Lab 1 Section I: VPC Networking & Security"
weight: 1
chapter: false
---

# VPC Networking & Security

#### Why It Matters  
A rock‑solid network is the foundation of every cloud application. In this section, you’ll design a multi‑AZ Virtual Private Cloud (VPC) with both public and private subnets, attach an Internet Gateway, configure route tables, and lock down traffic with security groups. By the end, your environment will be ready to host everything from internet‑facing web servers to private AI workloads—securely and reliably.

---
#### Objective  

build a VPC spanning 3 Availability Zones (AZs). Each AZ will have one public subnet, but the VPC needs only one private subnet.

---

## 1. Create Your VPC  
1. Navigate to **VPC** service → on the side page choose **Your VPCs** → **Create VPC**  
2. Name it **MyVPC**, give it a CIDR (e.g. `10.0.0.0/24`), and enable tenancy **default**.  
3. Click **Create**—you now have a VPC spanning all AZs in the region.
> **Note:** we create about 4 subnet: 3 Public subnet and 1 private subnet 

![Host Fullstack Web A.I](../images/1/1-1.png?featherlight=false&width=90pc)

## 2. Build First Public Subnets  
1. Go to **VPC** service → on the side page choose **Subnets → Create Subnet**  
2. For each AZ (`us-east-1a`, `1b`, `1c`):  
   - **Name**: `PublicSubnet1` (then `PublicSubnet2`, `PublicSubnet3` for other public subnet we want to create)  
   - **VPC**: MyVPC  
   - **CIDR**: pick non‑overlapping ranges, e.g. `10.0.0.32/27`, `10.0.0.64/27`, `10.0.0.96/27`  

![Host Fullstack Web A.I](../images/1/1-2.png?featherlight=false&width=90pc)

3. After creation, select each subnet → **Actions → Edit auto-assign IP settings** → check **Enable auto-assign public IPv4 address** to set it to public
> **Note:** we build continue with other public subnet

![Host Fullstack Web A.I](../images/1/1-3.png?featherlight=false&width=90pc)

## 3. Build Your Private Subnet  
1. **Create Subnet** as before:  
   - **Name**: `PrivateSubnet1` 
   - **VPC**: MyVPC  
   - **CIDR**: e.g. `10.0.1.0/27`  
   - Choose any one AZ.  

![Host Fullstack Web A.I](../images/1/1-4.png?featherlight=false&width=90pc)

2. Leave **auto‑assign public IP** disabled—this subnet is for internal services only.

## 4. Internet Gateway & Route Tables  
- **Internet Gateway**: Create and attach an Internet Gateway to the VPC for connect internet  
  1. Go to **VPC** service → **Internet Gateways → Create** 
  2. Name: `myIGW`  
  3. Click **Create internet gateway** button

![Host Fullstack Web A.I](../images/1/1-5.png?featherlight=false&width=90pc)

  4. Select that internet gateway we have created → **Actions → Attach to VPC** → MyVPC  

- **Public Route Table**: for routing in public subnet
  1. Go to **VPC** service → **Route Tables → Create** 
  2. Name: `MyPublic-rtb`, VPC: MyVPC  
  3. Click **Create** button

![Host Fullstack Web A.I](../images/1/1-6.png?featherlight=false&width=90pc)

  4. Under **Routes** tab → **Edit** → Add Destination IP: `0.0.0.0/0` and Target: `myIGW`  

![Host Fullstack Web A.I](../images/1/1-7.png?featherlight=false&width=90pc)

  5. Under **Subnet Associations** tab → **Edit** → attach all three `PublicSubnet*`.

![Host Fullstack Web A.I](../images/1/1-8.png?featherlight=false&width=90pc)

- **Private Route Table**: for routing in private subnet 
  1. **VPC → Route Tables → Create**
  2. Name: `MyPrivate-rtb`, VPC: MyVPC  
  3. Leave default local route only.  
  4. **Subnet Associations** → attach `PrivateSubnet1`.

## 5. Security Group  
1. Go to **EC2** service → **Security Groups → Create**  
2. **Name**: `MyVPC-sg`, **VPC**: MyVPC 
3. **Description**: `allow SSH and user access within the VPC` 
3. **Inbound**:  
   - Type: SSH, Protocol: TCP, port: 22 from anywhere IPv4  
   - Type: HTTP, Protocol: TCP, port: 80 from anywhere IPv4  
   - Type: All ICMP - IPv4, Protocol: ICMP, port: all from anywhere IPv4 
4. **Outbound**: allow all.

![Host Fullstack Web A.I](../images/1/1-9.png?featherlight=false&width=90pc)

---

> **Tip:** Always tag your resources with `Project: Lab-AI` (or similar) — it makes cleanup and billing far simpler!
