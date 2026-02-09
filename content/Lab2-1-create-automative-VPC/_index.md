---
title: "Lab 2 Section I: VPC Networking & Security"
weight: 1
chapter: false
---

# VPC Networking

In this section, we will establish the foundational **Networking Layer** using Amazon VPC. To implement a "Defense in Depth" security strategy, we will architect a network with strict segmentation:

* **Public Subnet:** Hosted in a Demilitarized Zone (DMZ) for the Frontend and resources requiring direct internet ingress.
* **Private Subnet:** A completely isolated environment for the Backend and Database, protected from direct external access.

We will leverage the **"VPC and more"** feature to automate the provisioning of the VPC, Route Tables, Internet Gateways, and NAT Gateways, reducing manual configuration errors.



---

### Step 1: VPC Workflow Initiation

1.  Navigate to the **AWS Management Console** and search for **VPC**.
2.  On the VPC Dashboard, click the orange **Create VPC** button.

---

### Step 2: Network Configuration

In the VPC creation wizard, configure the following parameters carefully to balance performance with cost-efficiency for this lab environment:

#### 1. VPC Settings
* **Resources to create:** Select **VPC and more**.
    > *Note: This option automatically visualizes the architecture and provisions dependent resources (Subnets, Route Tables) in a single workflow.*
* **Name tag auto-generation:** Enter `My-Fullstack-Lab`.
* **IPv4 CIDR block:** Keep default `10.0.0.0/16`.

#### 2. Availability Zones (AZs)
* **Number of Availability Zones (AZs):** Select **1**.
    > **Architectural Decision:**
    > While Production environments typically require Multi-AZ (2+) for High Availability, selecting **1 AZ** for this lab **reduces infrastructure costs by approximately 50%** (specifically regarding NAT Gateway hourly charges) while simplifying the network topology for learning purposes.

#### 3. Subnet Configuration
* **Number of public subnets:** Select **1**.
* **Number of private subnets:** Select **1**.
    > AWS will automatically calculate and assign CIDR blocks for these subnets based on the VPC CIDR.

#### 4. Gateways Configuration - CRITICAL ⚠️
* **NAT gateways ($):** Select **In 1 AZ**.
    > *Technical Rationale:* This is critical. The NAT Gateway provides **egress-only** internet access for the Private Subnet. This allows your Backend/Database instances to fetch updates and install packages (e.g., `apt update`, `npm install`) without exposing them to inbound internet traffic. Selecting "None" will leave your private backend completely isolated and unable to install software.
* **VPC endpoints:** Select **None**.

#### 5. DNS Options
* **Enable DNS hostnames:** ☑️ Check.
* **Enable DNS resolution:** ☑️ Check.
    > Essential for internal service discovery and hostname resolution within the VPC.

---

### Step 3: Review & Provision

1.  Review the **Preview** pane on the right. You should visualize the traffic flow:
    * Public Subnet routing to the **Internet Gateway**.
    * Private Subnet routing to the **NAT Gateway**.
2.  Click **Create VPC**.

AWS will initiate the automated provisioning workflow:
* ✅ Creating VPC...
* ✅ Creating Subnets...
* ✅ Creating Route Tables...
* ✅ Creating Internet Gateway...
* ✅ Allocating Elastic IP...
* ✅ Creating NAT Gateway...

*Please wait approximately 1-2 minutes for the provisioning to complete.*

---