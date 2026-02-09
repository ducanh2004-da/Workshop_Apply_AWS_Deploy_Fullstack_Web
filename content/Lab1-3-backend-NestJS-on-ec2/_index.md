---

title: "Lab 1 Section III: NestJS Backend on EC2 Public"
weight: 3
chapter: false
--------------

# Deploying the NestJS Backend

#### Why It Matters

Your backend glues together AI, database, and frontend. Here you’ll provision a lightweight EC2 instance, sync your NestJS code, configure Prisma and PostgreSQL, and launch your API in production mode.

---

## 1. Launch the Backend Instance (Public Subnet)

* **Name**: `BackendServer`
* **AMI**: Ubuntu LTS
* **Type**: `t2.micro`
* **Key pair**: same or new RSA PEM
* **Network**: click **Edit** button then choose **MyVPC** → `PublicSubnet1` → `MyVPC-sg`
* **Auto-assign Public IPv4**: Enabled
* **Storage**: 8 GiB
* => Launch

Wait for **2/2 checks**.

![Host Fullstack Web A.I](../images/3/3-1.png?featherlight=false&width=90pc)

## 2. Install Node.js & Sync Backend Code from local to ec2

Choose that instance we have created -> Connect, then SSH(Connect) in EC2 via EC2 Instance Connect

![Host Fullstack Web A.I](../images/2/2-2.png?featherlight=false&width=90pc)

then we set EC2 Instance

```bash
sudo apt update
sudo apt upgrade
```

we install NodeJS
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

![Host Fullstack Web A.I](../images/3/3-3.png?featherlight=false&width=90pc)

**rsync**:

On your **local** machine:
* We open terminal, then open wsl:
```bash
wsl -d Ubuntu-22.04
```
* delete all ssh folder in the past(if you have the one in the past):
```bash
sudo rm -rf /home/ducanh/.ssh
```
* then we configure for sync:
```bash
mkdir -p /home/ducanh/.ssh
cp /mnt/c/Users/<name user of computer>/Downloads/<name of your key>.pem ~/.ssh/
chmod 400 ~/.ssh/<name of your key>.pem
```
* finally, sync from local computer to ec2
```bash
rsync -avz \
  --exclude 'node_modules' \
  --exclude '.git' \
  -e "ssh -i ~/.ssh/<your key>.pem \
      -o StrictHostKeyChecking=no \
      -o UserKnownHostsFile=/dev/null" \
  . ubuntu@<DNS of your Ec2>.amazonaws.com:~/app
```

* then check all the folder,file on EC2

![Host Fullstack Web A.I](../images/3/3-4.png?featherlight=false&width=90pc)

## 3. App Setup & Database

On EC2:

```bash
cd ~/app
npm install --global yarn
yarn install
npx prisma generate
```
> **If having error in fastify version**: 
```bash
 npm install --legacy-peer-deps
 ```

Install Postgres:

```bash
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo -i -u postgres
CREATE DATABASE my_app;
CREATE ROLE my_app_role WITH LOGIN PASSWORD 'some_password';
GRANT ALL PRIVILEGES ON DATABASE "my_app" TO my_app_role;
```

access to `src/main.ts` to edit to bind on `0.0.0.0`:
```bash
nano src/main.ts
```

change:

```bash
const address =
    process.env.NODE_ENV === 'production' ? '0.0.0.0' : 'localhost';
```
to:

```bash
const address = '0.0.0.0';
```
## 4. Launch Your API

```bash
npm run start:prod
```

![Host Fullstack Web A.I](../images/3/3-5.png?featherlight=false&width=90pc)

Visit `http://<BackendServer-IPv4>:10000/graphql` to confirm GraphQL playground is live.

![Host Fullstack Web A.I](../images/3/3-6.png?featherlight=false&width=90pc)

Then we go to source code of front end, configure backend url to frontend call backend to get backend API:

![Host Fullstack Web A.I](../images/3/3-7.png?featherlight=false&width=90pc)

Finally, we have the result front-end(Local) call back-end(AWS):

![Host Fullstack Web A.I](../images/3/3-8.png?featherlight=false&width=90pc)

---

> **Did you know?** You can auto‑restart your backend with `pm2` or a simple `systemd` unit to ensure uptime after reboots.
