---

title: "Lab 1 Section V: PostgreSQL on RDS"
weight: 5
chapter: false
--------------

# Provisioning PostgreSQL on Amazon RDS

#### Why It Matters

Using a managed database service like Amazon RDS offloads operational responsibilities—backups, scaling, patching—so you can focus on building application features. In this section, you’ll launch a production‑grade PostgreSQL instance inside your VPC, secure network access, and integrate it with your NestJS backend through Prisma migrations.

---

## 1. Prerequisites

* **VPC**: MyVPC configured across at least two AZs with public and private subnets.
* **VPC Settings**: DNS resolution and DNS hostnames enabled for the VPC.

![Host Fullstack Web A.I](../images/5/5-1.png?featherlight=false&width=90pc)

* **Security Groups**: You should already have `MyVPC-sg` for general EC2 access.

## 2. Create a Database Security Group

1. Navigate to **EC2 → Security Groups → Create security group**.
2. **Name**: `RDS-Postgres-sg`
3. **VPC**: MyVPC
4. **Inbound rule**:

   * **Type**: PostgreSQL
   * **Port range**: 5432
   * **Source**: `MyVPC-sg` (or your trusted CIDR/IP range)
5. **Outbound rules**: leave as default (allow all).

## 3. Launch the RDS Instance

![Host Fullstack Web A.I](../images/5/5-2.png?featherlight=false&width=90pc)

1. Open **Aurora and RDS service → Databases → Create database button**.
2. **Choose a database creation method**: Stadard create
2. **Engine options**: PostgreSQL (choose the latest freetier supported version).
3. **Templates**: Free tier 
4. **DB instance identifier**: `demo-postgres`
5. **Credentials**: set a master username (e.g., `admin`) and a secure password.
6. **DB instance class**: choose `db.t4g.micro`
7. **Storage**: General Purpose SSD (gp3), allocate 20 GiB or as needed and extend Additional storage configuration then uncheck Enable storage autoscaling
8. **Connectivity**:

   * **VPC**: MyVPC
   * **Subnet group**: select your public subnets
   * **Public access**: Yes
   * **VPC security group**: `RDS-Postgres-sg`
9. **Additional configuration**:

   * Enable automatic backups (retain 7 days).
   * Set backup window and maintenance window per your preference.
10. Click **Create database** and wait until the status shows **Available**.

## 4. Connect and Run Migrations

1. In the RDS console, select **demo-postges** and copy the **Endpoint** (host name).
2. Open **pgAdmin** or another SQL client and create a new server connection:

   * **Host**: `<your-endpoint>`
   * **Port**: 5432
   * **Username**: your master username
   * **Password**: your master password

![Host Fullstack Web A.I](../images/5/5-3.png?featherlight=false&width=90pc)

- After that we save to connect:

![Host Fullstack Web A.I](../images/5/5-4.png?featherlight=false&width=90pc)

3. In your backend EC2 instance, update the `.env` file at `~/app/.env`:

   ```env
   DATABASE_URL="postgresql://my_app_role:some_password@<your-endpoint>:5432/my_app"
   ```

   ![Host Fullstack Web A.I](../images/5/5-5.png?featherlight=false&width=90pc)

4. Navigate to your app folder and run Prisma migrations:

   ```bash
   cd ~/app
   npx prisma migrate deploy
   npx prisma generate
   ```
5. Optionally, open Prisma Studio to verify your schema and data:

   ```bash
   npx prisma studio
   ```

> **Pro Tip:** Enable automated backups and a 7‑day retention policy. You can also scale storage vertically without downtime as your data footprint grows.
