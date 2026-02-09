---

title: "Lab 1 Phần II: Dịch vụ AI trên EC2"
weight: 2
chapter: false
--------------

# Triển Khai Dịch vụ AI trên EC2

#### Tại Sao Quan Trọng

Đóng gói mô hình AI của bạn trong Docker đảm bảo tính di động, xây dựng lặp lại được và dễ dàng cập nhật. Trong phần này, bạn sẽ khởi động một EC2 instance, cài Docker & Git, kéo mã AI, và khởi chạy mô hình của bạn phía sau một endpoint công khai.

---

## 1. Khởi Tạo EC2 Instance

1. **EC2 → Instances → Launch Instances**
2. **Name tag**: `ServerAI`
3. **AMI**: Ubuntu LTS
4. **Instance Type**: `t2.medium`
5. **Key pair**: Tạo hoặc chọn key RSA PEM của bạn nếu đã có
6. **Network setting**: MyVPC → `PublicSubnet2` → Security group: `MyVPC-sg`
7. **Auto-assign Public IPv4**: Đã bật
8. **Storage**: 20 GiB
9. Nhấn **Launch**. Chờ đến khi **2/2 checks** thành công.

![Host Fullstack Web A.I](../../images/2/2-1.png?featherlight=false\&width=90pc)

## 2. Cài Đặt Docker & Git

* Để cấu hình và triển khai mã nguồn AI trên AWS:
  SSH vào instance qua nút **Connect** => **EC2 Instance Connect**

![Host Fullstack Web A.I](../../images/2/2-2.png?featherlight=false\&width=90pc)

Chạy lệnh để cập nhật và cài Docker:

```bash
sudo apt update
sudo apt install -y \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
```

* Thêm khóa GPG chính thức của Docker:

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

* Thêm repository Docker vào APT sources:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

* Cập nhật lại và cài Docker Engine:

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

* Khởi động lại và kích hoạt dịch vụ Docker tự động:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

* Kiểm tra phiên bản Docker:

```bash
docker version
```

![Host Fullstack Web A.I](../../images/2/2-3.png?featherlight=false\&width=90pc)

**Cài đặt Git**:

```bash
sudo apt update
sudo apt install -y git
git --version
```

**Clone mã nguồn AI**:

```bash
git clone https://github.com/<tên-dự-án-github>
```

sử dụng 'ls' để xem các file, folder trong ec2 mà mình đã clone:

```bash
ls
```

![Host Fullstack Web A.I](../../images/2/2-4.png?featherlight=false\&width=90pc)

**Khởi chạy dự án bằng Docker**:

```bash
sudo docker compose up -d
```

![Host Fullstack Web A.I](../../images/2/2-5.png?featherlight=false\&width=90pc)

## 3. Kiểm Tra Kết Nối & Cấu Hình Endpoint

* Mở địa chỉ Public IPv4 của EC2 chạy AI trên tab trình duyệt để kiểm tra kết nối.

![Host Fullstack Web A.I](../../images/2/2-6.png?featherlight=false\&width=90pc)

* Khi thành công, thêm địa chỉ IPv4 đó vào file `.env` trong mã nguồn backend để gọi API AI.

![Host Fullstack Web A.I](../../images/2/2-7.png?featherlight=false\&width=90pc)
