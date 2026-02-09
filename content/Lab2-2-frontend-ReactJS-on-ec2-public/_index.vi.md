---

title: "Lab 2 Phần II: Frontend ReactJS trên EC2 Public"
weight: 3
chapter: false
--------------

# Triển Khai Frontend ReactJS

# Tổng quan

Trong bước này, chúng ta sẽ khởi tạo một **Public EC2 Instance** đóng vai trò là "mặt tiền" (Frontend Server). Instance này sẽ:
1.  Chứa mã nguồn ReactJS đã được build (tĩnh).
2.  Chạy **Nginx** để phục vụ web tĩnh cho người dùng.
3.  Cấu hình **Reverse Proxy** để chuyển tiếp các yêu cầu API từ người dùng vào Backend nằm ẩn trong Private Subnet.

---

### Bước 1: Khởi tạo Public EC2 Instance

Truy cập **EC2 Dashboard** > **Launch Instances** và cấu hình như sau:

#### 1. Tên và Hệ điều hành
* **Name:** `Fullstack-Public-Frontend`
* **OS Images (AMI):** Ubuntu Server **24.04 LTS** (hoặc 22.04 LTS).
* **Architecture:** 64-bit (x86).

#### 2. Instance Type
* **Instance Type:** `t3.small` (2 vCPU, 2 GiB RAM).
    > **Lưu ý:** Quá trình build React (Vite/Webpack) tốn khá nhiều RAM. Dùng `t2.micro` (1GB RAM) thường sẽ bị treo (freeze). `t3.small` là lựa chọn tối ưu cho môi trường Lab.

#### 3. Key Pair (Login)
* Nhấn **Create new key pair**.
* **Name:** `fullstack-lab-key`
* **Type:** RSA.
* **Format:** `.pem`.
* ⚠️ **Quan trọng:** Tải file `.pem` về và lưu kỹ. Nếu mất file này, bạn sẽ không thể SSH vào server.

#### 4. Network Settings (Rất quan trọng)
Nhấn nút **Edit** ở góc phải:
* **VPC:** Chọn VPC vừa tạo (VD: `My-Fullstack-Lab-vpc`).
* **Subnet:** Chọn Subnet có chữ **Public** (VD: `My-Fullstack-Lab-subnet-public1...`).
* **Auto-assign public IP:** Chọn **Enable**.
    > Bắt buộc phải Enable để server có IP Public giao tiếp với Internet.

#### 5. Security Group (Firewall)
Chọn **Create security group**:
* **Security group name:** `Public-SG-React`
* **Description:** Allow SSH and Web traffic.
* **Inbound Rules:**
    1.  **Type:** `SSH` | **Port:** `22` | **Source:** `My IP` (Chỉ cho phép IP nhà bạn truy cập để quản trị).
    2.  **Type:** `HTTP` | **Port:** `80` | **Source:** `Anywhere (0.0.0.0/0)` (Cho phép người dùng toàn cầu truy cập Web).

#### 6. Storage
* **Configure storage:** Tăng lên **20 GiB** (gp3). Mặc định 8GB có thể bị đầy khi cài đặt các thư viện Node modules.

=> Nhấn **Launch instance**.

---

### Bước 2: Thiết lập Môi trường (Environment Setup)

Sử dụng Terminal (hoặc PuTTY) để SSH vào server vừa tạo:
```bash
ssh -i "fullstack-lab-key.pem" ubuntu@<PUBLIC_IP_CUA_BAN>
# Cập nhật danh sách gói
sudo apt update && sudo apt upgrade -y

# Cài đặt Node.js 20.x (Phiên bản ổn định mới nhất)
curl -fsSL [https://deb.nodesource.com/setup_20.x](https://deb.nodesource.com/setup_20.x) | sudo -E bash -
sudo apt-get install -y nodejs

# Cài đặt Nginx và Yarn
sudo apt-get install -y nginx
sudo npm install --global yarn

# Kiểm tra trạng thái Nginx
sudo systemctl status nginx
# Tạo file swap 4GB
sudo fallocate -l 4G /swapfile

# Phân quyền chỉ root được đọc/ghi
sudo chmod 600 /swapfile

# Thiết lập file làm vùng swap
sudo mkswap /swapfile

# Kích hoạt swap
sudo swapon /swapfile

# (Tùy chọn) Kiểm tra lại
free -h

git clone [https://github.com/ducanh2004-da/learnify-fe.git](https://github.com/ducanh2004-da/learnify-fe.git)

# Nhập Username: ducanh2004-da
# Nhập Password: <YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>
# (Lưu ý: Password sẽ không hiện ký tự khi gõ)

cd ~/learnify-fe
yarn install --frozen-lockfile

#file env
nano .env

# Google Auth
VITE_GOOGLE_CLIENT_ID=

# API Backend (Trỏ về IP Public của chính server này, Nginx sẽ lo phần còn lại)
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

nano package.json
# Tìm dòng "scripts" và đảm bảo: "build": "vite build"

yarn build

# Xóa code cũ (nếu có)
sudo rm -rf /var/www/html/*

# Copy code mới
sudo cp -r dist/* /var/www/html/

#Cấu hình Nginx Reverse Proxy
sudo nano /etc/nginx/sites-available/default

server {
    listen 80;
    server_name _; # Chấp nhận mọi IP/Domain truy cập

    # 1. PHỤC VỤ FRONTEND (REACTJS)
    location / {
        root /var/www/html;
        index index.html index.htm;
        # Cấu hình cho SPA (Single Page Application)
        # Nếu không tìm thấy file, trả về index.html để React Router xử lý
        try_files $uri $uri/ /index.html;
    }

    # 2. REVERSE PROXY ĐẾN BACKEND (GRAPHQL)
    # Nginx nhận request từ user -> Chuyển tiếp vào Private EC2
    location /graphql {
        # Thay IP bên dưới bằng Private IP của Backend (VD: 10.0.140.251)
        proxy_pass [http://10.0.140.251:10000/graphql](http://10.0.140.251:10000/graphql);
        
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
    
    # 3. REVERSE PROXY CÁC API KHÁC
    location /api {
        proxy_pass [http://10.0.140.251:10000/api](http://10.0.140.251:10000/api);
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

# áp dụng cấu hình
# Kiểm tra cú pháp xem có lỗi không
sudo nginx -t

# Khởi động lại Nginx để nhận cấu hình mới
sudo systemctl restart nginx
```
