---

title: "Lab 2 Phần I: Mạng & Bảo mật VPC"
weight: 1
chapter: false
--------------

# Mạng VPC

Trong phần này, chúng ta sẽ thiết lập lớp nền tảng mạng (Networking Layer) sử dụng **Amazon VPC**. Để đảm bảo mô hình bảo mật "Defense in Depth" (Phòng thủ chiều sâu), chúng ta sẽ thiết kế mạng với hai phân vùng rõ rệt:
* **Public Subnet:** Dành cho Frontend và các tài nguyên cần truy cập trực tiếp từ Internet.
* **Private Subnet:** Dành cho Backend và Database, cô lập hoàn toàn khỏi các kết nối trực tiếp từ bên ngoài.

Chúng ta sẽ sử dụng tính năng **"VPC and more"** của AWS để tự động hóa việc tạo Route Table, Internet Gateway và NAT Gateway, giúp giảm thiểu sai sót cấu hình thủ công.

---

### Bước 1: Khởi tạo VPC Workflow

1.  Truy cập **AWS Management Console** và tìm kiếm dịch vụ **VPC**.
2.  Tại màn hình Dashboard, nhấn nút màu cam **Create VPC**.
3.  

---

### Bước 2: Cấu hình Kiến trúc Mạng (Network Configuration)

Trong trình hướng dẫn tạo VPC, hãy cấu hình chính xác các thông số sau để tối ưu hóa chi phí cho bài Lab (tránh phát sinh phí cho tài nguyên không cần thiết):

#### 1. Cấu hình cơ bản (VPC Settings)
* **Resources to create:** Chọn **VPC and more**.
    > *Lưu ý: Tùy chọn này sẽ tự động vẽ sơ đồ kiến trúc và tạo đầy đủ các thành phần phụ trợ thay vì chỉ tạo một VPC rỗng.*
* **Name tag auto-generation:** Nhập `My-Fullstack-Lab`.
* **IPv4 CIDR block:** Giữ nguyên mặc định `10.0.0.0/16`.

#### 2. Cấu hình Vùng sẵn sàng (Availability Zones - AZs)
* **Number of Availability Zones (AZs):** Chọn **1**.
    > **Tại sao chọn 1 AZ?**
    > Trong môi trường Production, chúng ta thường chọn 2 AZ để đảm bảo High Availability (HA). Tuy nhiên, với bài Lab ngắn hạn này, việc chọn 1 AZ giúp **tiết kiệm 50% chi phí** hạ tầng (đặc biệt là chi phí NAT Gateway) và đơn giản hóa việc quản lý mạng.

#### 3. Cấu hình Subnet (Subnets)
* **Number of public subnets:** Chọn **1**.
* **Number of private subnets:** Chọn **1**.
    > Hệ thống sẽ tự động chia dải IP (CIDR) cho 2 subnet này dựa trên VPC CIDR tổng.

#### 4. Cấu hình Cổng kết nối (Gateways) - QUAN TRỌNG ⚠️
* **NAT gateways ($):** Chọn **In 1 AZ**.
    > *Giải thích kỹ thuật:* Đây là bước quyết định. NAT Gateway cho phép các instance trong **Private Subnet** (Backend/DB) có thể đi ra Internet để tải thư viện (npm install, apt update) mà không cần lộ IP ra ngoài. Nếu chọn "None", Backend của bạn sẽ không thể cài đặt được gì.
* **VPC endpoints:** Chọn **None** (Không cần thiết cho bài lab này).

#### 5. Tùy chọn DNS (DNS Options)
* **Enable DNS hostnames:** ☑️ Tick chọn.
* **Enable DNS resolution:** ☑️ Tick chọn.
    > Cần thiết để các dịch vụ AWS phân giải được tên miền nội bộ.

---

### Bước 3: Xem trước và Khởi tạo (Review & Create)

1.  Nhìn vào biểu đồ trực quan (Preview) ở bên phải màn hình. Bạn sẽ thấy luồng kết nối:
    * Public Subnet kết nối tới Internet Gateway.
    * Private Subnet kết nối tới NAT Gateway.
2.  Nhấn nút **Create VPC** ở dưới cùng.

AWS sẽ bắt đầu quy trình tự động (Workflow):
* ✅ Creating VPC...
* ✅ Creating Subnets...
* ✅ Creating Route Tables...
* ✅ Creating Internet Gateway...
* ✅ Allocating Elastic IP...
* ✅ Creating NAT Gateway...

*Vui lòng đợi khoảng 1-2 phút để quá trình hoàn tất.*

---