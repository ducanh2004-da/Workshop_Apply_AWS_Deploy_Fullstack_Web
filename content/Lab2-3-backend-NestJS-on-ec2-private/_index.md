---
title: "Lab 2 Part III: Backend NestJS on Private EC2"
weight: 9
chapter: false
--------------

# Deploying NestJS Backend and PostgreSQL

# Overview

In this step, we will provision a **Private EC2 Instance**. This acts as the "vault" of the system, hosting all business logic (Backend) and data (Database). This instance:
1.  **Has NO Public IP:** Cannot be accessed directly from the Internet.
2.  **Accepts traffic only from Frontend:** Controlled via Security Groups.
3.  **Accessed via Bastion Host:** Using the "SSH Jump" technique from the Public Instance.

---

### Step 1: Provision Private EC2 Instance

Access the **EC2 Dashboard** > **Launch Instances** and configure as follows:

#### 1. Basic Configuration
* **Name:** `Fullstack-Private-Backend`
* **OS Images:** Ubuntu Server **24.04 LTS**.
* **Instance Type:** `t3.small` (Required to run Node.js, Prisma, and Docker Containers simultaneously).
* **Key pair:** Select `fullstack-lab-key` again (Reuse the key from the Frontend).

#### 2. Network Settings (Critical) 🔒
Click **Edit**:
* **VPC:** Select your VPC (`My-Fullstack-Lab-vpc`).
* **Subnet:** Select the Subnet labeled **Private** (e.g., `My-Fullstack-Lab-subnet-private1...`).
* **Auto-assign public IP:** Select **Disable** (Absolutely no Public IP assignment).

#### 3. Security Group (Firewall)
Select **Create security group**:
* **Name:** `Private-SG-Backend`
* **Description:** Allow traffic from Frontend only.
* **Inbound Rules:**
    1.  **SSH (22):** Source select **Custom** -> Type `Public-SG-React` (The Frontend's SG) and select it.
        > *Meaning:* Allows SSH access only from the Frontend server machine.
    2.  **Custom TCP (App Port):** Port `10000` (or `3000` depending on your code) | Source select `Public-SG-React`.
        > *Meaning:* Allows the Frontend to call the API.

#### 4. Storage
* **Configure storage:** Increase to **20 GiB** (gp3) to accommodate Docker Images.

=> Click **Launch instance**.

---

### Step 2: SSH Jump Technique

Since the Private Instance has no Public IP, you cannot SSH directly. You must "jump" through the Public Instance.

**On your Local Terminal:**

1.  SSH into Public Instance (Frontend) first:
    ```bash
    ssh -i fullstack-lab-key.pem ubuntu@<PUBLIC_IP_FRONTEND>
    ```

2.  On the Public Instance, create the private key file:
    ```bash
    nano private-key.pem
    ```
    * Open the `.pem` file on your computer using Notepad/TextEdit.
    * Copy all content and Paste it into the terminal window.
    * Press `Ctrl+O` -> `Enter` -> `Ctrl+X` to save.

3.  Set permissions for the key file (Required):
    ```bash
    chmod 400 private-key.pem
    ```

4.  SSH from Public to Private:
    ```bash
    # Replace <PRIVATE_IP_BACKEND> with the Private IP of the Backend machine (e.g., 10.0.140.251)
    ssh -o StrictHostKeyChecking=no -i private-key.pem ubuntu@<PRIVATE_IP_BACKEND>
    ```
    *If you see the command prompt hostname change, you have successfully accessed the Private Server!*

---

### Step 3: Environment & Database Setup

#### 1. Install Node.js and Tools
Since the Private Subnet has a NAT Gateway (configured in Lab 1), it can download packages from the Internet.

```bash
sudo apt update && sudo apt upgrade -y
curl -fsSL [https://deb.nodesource.com/setup_20.x](https://deb.nodesource.com/setup_20.x) | sudo -E bash -
sudo apt-get install -y nodejs
sudo npm install --global yarn pm2

# Install Docker
curl -fsSL [https://get.docker.com](https://get.docker.com) -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu
newgrp docker

# Run PostgreSQL Container
docker run -d \
  --name postgres-db \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=12345 \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  --restart always \
  postgres:14

git clone [https://github.com/ducanh2004-da/learnify-be.git](https://github.com/ducanh2004-da/learnify-be.git)
cd ~/learnify-be
yarn install --frozen-lockfile --production=false

nano .env
DATABASE_URL="postgresql://postgres:12345@localhost:5432/mydb?schema=public"
PORT=10000
# Other variables...

nano src/main.ts
# You need to edit main.ts for the App to listen on IP 0.0.0.0 instead of localhost

// Change to:
app.enableCors({
    origin: '*', // Or specify specific Frontend domain
    credentials: true
});
await app.listen(10000, '0.0.0.0'); // CRITICAL: Must be '0.0.0.0'

npx prisma generate
npx prisma migrate deploy
yarn build

pm2 start dist/main.js --name "backend-api"
pm2 save
pm2 startup
# Copy the command printed by pm2 startup and run it to set startup on boot

# Verify and Debug
# Install net-tools if not already present
sudo apt install net-tools -y

# Check port
sudo netstat -tulpn | grep 10000
```