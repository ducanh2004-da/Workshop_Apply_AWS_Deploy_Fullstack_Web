---

title: "Lab 2 Phần III: Backend NestJS trên EC2 private"
weight: 3
chapter: false
--------------

# Triển Khai Backend NestJS và PostgreSQL

# Tổng quan

Trong bước này, chúng ta sẽ khởi tạo **Private EC2 Instance**. Đây được ví như "két sắt" của hệ thống, nơi chứa toàn bộ logic nghiệp vụ (Backend) và dữ liệu (Database). Instance này:
1.  **Không có Public IP:** Không thể truy cập trực tiếp từ Internet.
2.  **Chỉ nhận kết nối từ Frontend:** Thông qua Security Group.
3.  **Truy cập thông qua Bastion Host:** Sử dụng kỹ thuật "SSH Jump" từ Public Instance.

---

### Bước 1: Khởi tạo Private EC2 Instance

Truy cập **EC2 Dashboard** > **Launch Instances** và cấu hình như sau:

#### 1. Cấu hình Cơ bản
* **Name:** `Fullstack-Private-Backend`
* **OS Images:** Ubuntu Server **24.04 LTS**.
* **Instance Type:** `t3.small` (Cần thiết vì chạy đồng thời Node.js, Prisma và Docker Container).
* **Key pair:** Chọn lại key `fullstack-lab-key` (Dùng chung key với Frontend).

#### 2. Network Settings (Cực kỳ quan trọng) 🔒
Nhấn **Edit**:
* **VPC:** Chọn VPC của bạn (`My-Fullstack-Lab-vpc`).
* **Subnet:** Chọn Subnet có chữ **Private** (VD: `My-Fullstack-Lab-subnet-private1...`).
* **Auto-assign public IP:** Chọn **Disable** (Tuyệt đối không cấp IP Public).

#### 3. Security Group (Firewall)
Chọn **Create security group**:
* **Name:** `Private-SG-Backend`
* **Description:** Allow traffic from Frontend only.
* **Inbound Rules:**
    1.  **SSH (22):** Source chọn **Custom** -> Gõ tên `Public-SG-React` (SG của Frontend) và chọn nó.
        > *Ý nghĩa:* Chỉ cho phép SSH từ máy Frontend server.
    2.  **Custom TCP (Port App):** Port `10000` (hoặc `3000` tùy code của bạn) | Source chọn `Public-SG-React`.
        > *Ý nghĩa:* Chỉ cho phép Frontend gọi API.

#### 4. Storage
* **Configure storage:** Tăng lên **20 GiB** (gp3) để đủ chỗ chứa Docker Images.

=> Nhấn **Launch instance**.

---

### Bước 2: Kỹ thuật SSH Jump (Nhảy cầu)

Vì Private Instance không có IP Public, bạn không thể SSH trực tiếp. Bạn phải "nhảy" qua con Public Instance.

**Tại Terminal máy cá nhân của bạn:**

1.  SSH vào Public Instance (Frontend) trước:
    ```bash
    ssh -i fullstack-lab-key.pem ubuntu@<IP_PUBLIC_FRONTEND>
    ```

2.  Tại Public Instance, tạo file chứa khóa bí mật:
    ```bash
    nano private-key.pem
    ```
    * Mở file `.pem` trên máy tính của bạn bằng Notepad/TextEdit.
    * Copy toàn bộ nội dung và Paste vào cửa sổ terminal.
    * Nhấn `Ctrl+O` -> `Enter` -> `Ctrl+X` để lưu.

3.  Phân quyền cho file khóa (Bắt buộc):
    ```bash
    chmod 400 private-key.pem
    ```

4.  SSH từ Public sang Private:
    ```bash
    # Thay <IP_PRIVATE_BACKEND> bằng Private IP của máy Backend (VD: 10.0.140.251)
    ssh -o StrictHostKeyChecking=no -i private-key.pem ubuntu@<IP_PRIVATE_BACKEND>
    ```
    *Nếu thấy dấu nhắc lệnh đổi tên host, bạn đã vào thành công Private Server!*

---

### Bước 3: Cài đặt Môi trường & Database

#### 1. Cài đặt Node.js và Công cụ
Do Private Subnet đã có NAT Gateway (đã cấu hình ở Lab 1), nó có thể tải gói tin từ Internet.

```bash
sudo apt update && sudo apt upgrade -y
curl -fsSL [https://deb.nodesource.com/setup_20.x](https://deb.nodesource.com/setup_20.x) | sudo -E bash -
sudo apt-get install -y nodejs
sudo npm install --global yarn pm2

# Cài Docker
curl -fsSL [https://get.docker.com](https://get.docker.com) -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu
newgrp docker

# Chạy PostgreSQL Container
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
# Các biến khác...

nano src/main.ts
# Bạn cần sửa file main.ts để App lắng nghe IP 0.0.0.0 thay vì localhost

// Sửa thành:
app.enableCors({
    origin: '*', // Hoặc điền domain Frontend cụ thể
    credentials: true
});
await app.listen(10000, '0.0.0.0'); // QUAN TRỌNG: Phải có '0.0.0.0'

npx prisma generate
npx prisma migrate deploy
yarn build

pm2 start dist/main.js --name "backend-api"
pm2 save
pm2 startup
# Copy lệnh mà pm2 startup in ra và chạy nó để set khởi động cùng OS

#Kiểm tra và Gỡ lỗi
# Cài net-tools nếu chưa có
sudo apt install net-tools -y

# Kiểm tra port
sudo netstat -tulpn | grep 10000
```