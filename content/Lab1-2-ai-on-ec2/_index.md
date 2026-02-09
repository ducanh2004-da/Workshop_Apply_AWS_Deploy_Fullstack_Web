---
title: "Lab 1 Section II: AI Service on EC2"
weight: 2
chapter: false
---

# Deploying Your AI Service on EC2

#### Why It Matters  
Containerizing your AI model in Docker ensures portability, repeatable builds, and easy updates. In this section, you’ll launch an EC2 instance, install Docker & Git, pull your AI code, and spin up your model behind a public endpoint.

---

## 1. Launch the EC2 Instance  
1. Go to **EC2** service → **Instances → Launch Instances**  
2. **Name tag**: `ServerAI`  
3. **AMI**: Ubuntu LTS  
4. **Instance Type**: `t2.medium`  
5. **Key pair**: Create or select your RSA PEM key if you have one 
6. **Network setting**: MyVPC → `PublicSubnet2` → Security group: `MyVPC-sg`  
7. **Auto-assign Public IPv4**: Enabled  
8. **Storage**: 20 GiB  
9. Click **Launch**. Wait for **2/2 checks** to pass.

![Host Fullstack Web A.I](../images/2/2-1.png?featherlight=false&width=90pc)

## 2. Install Docker & Git  
- To configure and deploy source code of A.I on AWS to run
SSH into your instance via **Connect** button => **EC2 Instance Connect** 

![Host Fullstack Web A.I](../images/2/2-2.png?featherlight=false&width=90pc)

run to install docker

```bash
sudo apt update
```

```bash
sudo apt install -y \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
```
- Add offical GPG key of Docker
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
- Add epository Docker in APT sources
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
- Update again and install Docker Engine
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
- Reboot and activate Docker service automatically
```bash
sudo systemctl enable docker
sudo systemctl start docker
```
- Check whether we have installed it
```bash
docker version
```

![Host Fullstack Web A.I](../images/2/2-3.png?featherlight=false&width=90pc)

**Install git**:
- Update package:
```bash
sudo apt update
```
- Install git
```bash
sudo apt install -y git
```
- Check version
```bash
git --version
```
**Clone the folder A.I to run**:
```bash
git clone https://github.com/<tên dự án github>
```
- ls: to list all the folder file we have cloned

![Host Fullstack Web A.I](../images/2/2-4.png?featherlight=false&width=90pc)


**Run project with docker**:
```bash
sudo docker compose up -d
```

![Host Fullstack Web A.I](../images/2/2-5.png?featherlight=false&width=90pc)

## 3. Check the connection and import IP on source code backend to call API A.I:
- Open ublic IPv4 address of A.I EC2 on another tab to check

![Host Fullstack Web A.I](../images/2/2-6.png?featherlight=false&width=90pc)

- When successfully, we add that IPv4 on .env file of source code backend to call API A.I 

![Host Fullstack Web A.I](../images/2/2-7.png?featherlight=false&width=90pc)

