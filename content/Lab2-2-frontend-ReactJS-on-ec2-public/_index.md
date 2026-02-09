---
title: "Lab 2 Part II: ReactJS Frontend on Public EC2"
weight: 3
chapter: false
--------------

# Deploying ReactJS Frontend

# Overview

In this step, we will provision a **Public EC2 Instance** acting as the "entry point" (Frontend Server). This instance will:
1.  Host the built ReactJS source code (static artifacts).
2.  Run **Nginx** to serve the static web content to users.
3.  Configure a **Reverse Proxy** to forward API requests from users to the Backend located in the Private Subnet.

---

### Step 1: Provision Public EC2 Instance

Access the **EC2 Dashboard** > **Launch Instances** and configure as follows:

#### 1. Name and OS
* **Name:** `Fullstack-Public-Frontend`
* **OS Images (AMI):** Ubuntu Server **24.04 LTS** (or 22.04 LTS).
* **Architecture:** 64-bit (x86).

#### 2. Instance Type
* **Instance Type:** `t3.small` (2 vCPU, 2 GiB RAM).
    > **Note:** The React build process (Vite/Webpack) consumes significant RAM. Using `t2.micro` (1GB RAM) will often cause the instance to freeze. `t3.small` is the optimal choice for this Lab environment.

#### 3. Key Pair (Login)
* Click **Create new key pair**.
* **Name:** `fullstack-lab-key`
* **Type:** RSA.
* **Format:** `.pem`.
* ⚠️ **Important:** Download the `.pem` file and store it securely. If you lose this file, you will not be able to SSH into the server.

#### 4. Network Settings (Critical)
Click the **Edit** button on the right:
* **VPC:** Select the VPC you just created (e.g., `My-Fullstack-Lab-vpc`).
* **Subnet:** Select the Subnet labeled **Public** (e.g., `My-Fullstack-Lab-subnet-public1...`).
* **Auto-assign public IP:** Select **Enable**.
    > Must be **Enabled** for the server to have a Public IP to communicate with the Internet.

#### 5. Security Group (Firewall)
Select **Create security group**:
* **Security group name:** `Public-SG-React`
* **Description:** Allow SSH and Web traffic.
* **Inbound Rules:**
    1.  **Type:** `SSH` | **Port:** `22` | **Source:** `My IP` (Allows only your home IP for administration).
    2.  **Type:** `HTTP` | **Port:** `80` | **Source:** `Anywhere (0.0.0.0/0)` (Allows global user access to the Web).

#### 6. Storage
* **Configure storage:** Increase to **20 GiB** (gp3). The default 8GB may fill up when installing Node modules.

=> Click **Launch instance**.

---

### Step 2: Environment Setup

Use Terminal (or PuTTY) to SSH into the newly created server:
```bash
ssh -i "fullstack-lab-key.pem" ubuntu@<YOUR_PUBLIC_IP>

# Update package list and upgrade system
sudo apt update && sudo apt upgrade -y

# Install Node.js 20.x (Latest LTS version)
curl -fsSL [https://deb.nodesource.com/setup_20.x](https://deb.nodesource.com/setup_20.x) | sudo -E bash -
sudo apt-get install -y nodejs

# Install Nginx and Yarn
sudo apt-get install -y nginx
sudo npm install --global yarn

# Check Nginx status
sudo systemctl status nginx

# Create a 4GB swap file (Virtual RAM)
sudo fallocate -l 4G /swapfile

# Set permissions (read/write for root only)
sudo chmod 600 /swapfile

# Set up the file as swap area
sudo mkswap /swapfile

# Enable swap
sudo swapon /swapfile

# (Optional) Verify memory
free -h

# Clone the repository
git clone [https://github.com/ducanh2004-da/learnify-fe.git](https://github.com/ducanh2004-da/learnify-fe.git)

# Enter Username: ducanh2004-da
# Enter Password: <YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>
# (Note: Password characters will not appear when typing)

cd ~/learnify-fe
yarn install --frozen-lockfile

# Create/Edit .env file
nano .env

# Google Auth
VITE_GOOGLE_CLIENT_ID=

# API Backend (Point to the Public IP of THIS server; Nginx will handle the proxying)
VITE_API_BACKEND_URL="http://<PUBLIC_IP_EC2>/graphql"
VITE_API_CSRF="http://<PUBLIC_IP_EC2>/csrf-token"
VITE_API_CHATSOCKET="http://<PUBLIC_IP_EC2>/chat"

# AI Services Keys
VITE_AZURE_SPEECH_KEY=<YOUR_AZURE_KEY>
VITE_AZURE_SPEECH_REGION=

VITE_ELEVEN_API_KEY=<YOUR_ELEVEN_LABS_KEY>
VITE_ELEVEN_VOICE_ID=
VITE_ELEVEN_MODEL_ID=
VITE_ELEVEN_OUTPUT_FORMAT=
VITE_ELEVEN_ENABLE_TIMESTAMPS=

# Edit package.json
nano package.json
# Find the "scripts" section and ensure it says: "build": "vite build"

# Build the application
yarn build

# Remove old code (if any)
sudo rm -rf /var/www/html/*

# Copy new build artifacts
sudo cp -r dist/* /var/www/html/

# Configure Nginx Reverse Proxy
sudo nano /etc/nginx/sites-available/default

server {
    listen 80;
    server_name _; # Accept requests from any IP/Domain

    # 1. SERVE FRONTEND (REACTJS)
    location / {
        root /var/www/html;
        index index.html index.htm;
        # Configuration for SPA (Single Page Application)
        # If file not found, return index.html for React Router to handle
        try_files $uri $uri/ /index.html;
    }

    # 2. REVERSE PROXY TO BACKEND (GRAPHQL)
    # Nginx receives request from user -> Forwards to Private EC2
    location /graphql {
        # Replace the IP below with the Private IP of the Backend (e.g., 10.0.140.251)
        proxy_pass [http://10.0.140.251:10000/graphql](http://10.0.140.251:10000/graphql);
        
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
    
    # 3. REVERSE PROXY FOR OTHER APIS
    location /api {
        proxy_pass [http://10.0.140.251:10000/api](http://10.0.140.251:10000/api);
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

# Apply configuration

# Check syntax for errors
sudo nginx -t

# Restart Nginx to apply new configuration
sudo systemctl restart nginx