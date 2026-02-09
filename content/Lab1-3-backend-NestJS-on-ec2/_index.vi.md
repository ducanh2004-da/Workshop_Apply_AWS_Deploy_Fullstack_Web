---

title: "Lab 1 Phần III: Backend NestJS trên EC2 Public"
weight: 3
chapter: false
--------------

# Triển Khai Backend NestJS

#### Tại Sao Quan Trọng

Backend của bạn kết nối các thành phần AI, cơ sở dữ liệu và frontend. Ở đây bạn sẽ cấp phát một EC2 instance nhẹ, đồng bộ mã NestJS, cấu hình Prisma và PostgreSQL, và khởi chạy API ở chế độ production.

---

## 1. Khởi Tạo EC2 Instance Cho Backend (Subnet Công Cộng)

* **Name tag**: `BackendServer`
* **AMI**: Ubuntu LTS
* **Instance Type**: `t2.micro`
* **Key pair**: dùng key RSA PEM cũ hoặc tạo mới
* **Network**: nhấn **Edit**, chọn **MyVPC** → `PublicSubnet1` → Security group: `MyVPC-sg`
* **Auto-assign Public IPv4**: Đã bật
* **Storage**: 8 GiB

Nhấn **Launch** và chờ đến khi **2/2 checks** thành công.

![Host Fullstack Web A.I](../../images/3/3-1.png?featherlight=false\&width=90pc)

## 2. Cài Đặt Node.js & Đồng Bộ Mã Backend

SSH (Connect) vào EC2 để kết nối đến console của ec2

![Host Fullstack Web A.I](../../images/2/2-2.png?featherlight=false\&width=90pc)

- Tiếp theo ta update và tải nodejs về:

```bash
sudo apt update && sudo apt upgrade -y
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```


**Đồng bộ mã với rsync**:

Trên máy **local**:

- Mở WSL nếu dùng Windows
```bash
wsl -d Ubuntu-22.04
```

- Xoá thư mục ~/.ssh cũ nếu có

```bash
sudo rm -rf /home/ducanh/.ssh
```
- Tạo lại thư mục ~/.ssh và copy key vào

```bash
mkdir -p /home/ducanh/.ssh
cp /mnt/c/Users/<tên-user>/Downloads/<your-key>.pem ~/.ssh/
chmod 400 ~/.ssh/<your-key>.pem
```
- Đồng bộ mã qua rsync

```bash
rsync -avz \
  --exclude 'node_modules' \
  --exclude '.git' \
  -e "ssh -i ~/.ssh/<your-key>.pem \
      -o StrictHostKeyChecking=no \
      -o UserKnownHostsFile=/dev/null" \
  . ubuntu@<DNS-EC2>.amazonaws.com:~/app
```
- Sau đó ta dùng 'ls' để kiểm tra file, folder

![Host Fullstack Web A.I](../../images/3/3-4.png?featherlight=false\&width=90pc)

## 3. Cấu Hình Ứng Dụng & Cơ Sở Dữ Liệu

Trên EC2:

```bash
cd ~/app
npm install --global yarn
yarn install
npx prisma generate
```

> Nếu gặp lỗi fastify version:

```bash
npm install --legacy-peer-deps
```

**Cài PostgreSQL**:

```bash
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo -i -u postgres
CREATE DATABASE my_app;
CREATE ROLE my_app_role WITH LOGIN PASSWORD 'some_password';
GRANT ALL PRIVILEGES ON DATABASE "my_app" TO my_app_role;
```

Chỉnh `src/main.ts` để bind vào `0.0.0.0`:

```bash
nano src/main.ts
```

Thay đoạn:

```ts
const address =
    process.env.NODE_ENV === 'production' ? '0.0.0.0' : 'localhost';
```

Bằng:

```ts
const address = '0.0.0.0';
```

## 4. Khởi Chạy API

```bash
npm run start:prod
```

![Host Fullstack Web A.I](../../images/3/3-5.png?featherlight=false\&width=90pc)

Truy cập `http://<IPv4-BackendServer>:10000/graphql` để xác nhận GraphQL Playground đang hoạt động.

![Host Fullstack Web A.I](../../images/3/3-6.png?featherlight=false\&width=90pc)

Sau đó, vào mã frontend và cấu hình URL backend để frontend gọi API:

![Host Fullstack Web A.I](../../images/3/3-7.png?featherlight=false\&width=90pc)

Kết quả: frontend (local) gọi thành công backend (AWS).

![Host Fullstack Web A.I](../../images/3/3-8.png?featherlight=false\&width=90pc)

---

> **Mẹo**: Bạn có thể tự động khởi động lại backend với `pm2` hoặc tạo một unit `systemd` đơn giản để đảm bảo uptime sau khi máy khởi động lại.
