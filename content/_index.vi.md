---
title: "Kiến trúc Đám mây AWS: Triển khai Ứng dụng Full-Stack tích hợp AI"
weight: 1
chapter: false
---

# Giới thiệu

#### Tóm tắt Dự án
Dự án này minh họa quy trình triển khai một ứng dụng web Full-stack hiện đại, có tính sẵn sàng cao (High Availability) bao gồm ReactJS, NestJS, PostgreSQL và mô hình AI Python trên nền tảng AWS. Mục tiêu là thực hiện và so sánh hai mô hình kiến trúc hạ tầng khác biệt: **Kiến trúc Hướng Dịch vụ (Service-Oriented Architecture)** tận dụng các dịch vụ được quản lý (Managed Services), và **Kiến trúc Mạng Bảo mật (Secure Network Architecture)** tập trung vào việc phân đoạn mạng VPC và tự quản trị tài nguyên.

---

#### Lab 01: Kiến trúc Hướng Dịch vụ (Tập trung vào Managed Services)
*Mô hình: Frontend trên S3 (Static Hosting) | Backend & AI trên EC2 | Database trên RDS*

Trong phương pháp tiếp cận này, trọng tâm là tận dụng các dịch vụ được AWS quản lý để giảm thiểu gánh nặng vận hành và đảm bảo tính sẵn sàng cao cho lớp cơ sở dữ liệu.

1.  **Thiết kế Mạng VPC:**
    * Thiết kế VPC tùy chỉnh với kiến trúc Đa vùng sẵn sàng (Multi-AZ).
    * Cấu hình các Subnet Public và Private với Bảng định tuyến (Route Tables) và Internet Gateway tùy chỉnh để quản lý luồng truy cập hiệu quả.

2.  **Đóng gói Dịch vụ AI (Docker trên EC2):**
    * Cấp phát (Provision) một instance EC2 `t2.medium` để vận hành mô hình AI dựa trên Python.
    * Đóng gói dịch vụ AI sử dụng Docker để đảm bảo tính nhất quán của môi trường và khả năng tích hợp liền mạch với backend.

3.  **Triển khai Backend (Node.js/NestJS trên EC2):**
    * Triển khai NestJS API trên một instance `t2.micro`.
    * Cấu hình Prisma ORM để quản lý schema cơ sở dữ liệu và thiết lập kết nối bảo mật giữa Backend EC2 và RDS instance.

4.  **Phân phối Frontend (S3 Static Hosting):**
    * Tối ưu hóa chi phí và hiệu suất bằng cách lưu trữ các bản build ReactJS trên Amazon S3.
    * Cấu hình Bucket Policies cho phép quyền đọc công khai (public read), phục vụ giao diện trực tiếp cho người dùng trong khi giao tiếp với Backend API.

5.  **Cơ sở dữ liệu được quản lý (Amazon RDS):**
    * Khởi tạo một instance PostgreSQL sử dụng Amazon RDS để tận dụng tính năng sao lưu tự động và bảo trì được quản lý bởi AWS.
    * Bảo mật quyền truy cập database bằng các Security Group nghiêm ngặt (chỉ cho phép lưu lượng từ Backend EC2).

---

#### Lab 02: Kiến trúc Mạng Bảo mật (Tập trung vào IaaS & Phân đoạn VPC)
*Mô hình: Frontend trên Public EC2 (Nginx) | Backend & Database trên Private EC2 | NAT Gateway*

Kịch bản này mô phỏng một môi trường doanh nghiệp có tính bảo mật cao, nơi các tài nguyên quan trọng (Backend API và Database) được cô lập khỏi internet công cộng, yêu cầu cấu hình mạng nâng cao.

1.  **Mạng & Bảo mật Nâng cao:**
    * Kiến trúc hóa thiết lập "VPC and More" với sự phân tách nghiêm ngặt giữa Public và Private Subnet.
    * Triển khai **NAT Gateway** để cho phép các instance trong Private Subnet truy cập internet (để cập nhật/cài đặt) mà không để lộ chúng trước các kết nối đi vào (inbound traffic).
    * Định nghĩa các quy tắc Security Group Inbound/Outbound chi tiết để ngăn chặn truy cập trái phép.

2.  **Frontend & Reverse Proxy (Public Subnet):**
    * Cấp phát một instance EC2 `t3.small` trong Public Subnet.
    * Cấu hình **Nginx** làm web server để host ứng dụng ReactJS, đóng vai trò là điểm truy cập (entry point) cho lưu lượng người dùng.

3.  **Backend Bảo mật & Database Tự quản trị (Private Subnet):**
    * Cấp phát một instance EC2 `t3.small` hoàn toàn cô lập trong Private Subnet.
    * Triển khai NestJS backend và **cơ sở dữ liệu PostgreSQL tự host (thông qua Docker)** trên instance riêng tư này.
    * Chứng minh khả năng quản lý dữ liệu bền vững (persistence) và kết nối mạng trong môi trường bị hạn chế.

4.  **Quản lý Chi phí & Dọn dẹp Tài nguyên:**
    * Thực hiện các chính sách quản lý vòng đời để hủy bỏ Elastic IPs, NAT Gateways và EC2 instances nhằm ngăn chặn vượt ngân sách.

---

#### Các Kỹ năng Kỹ thuật Đạt được
Thông qua việc hoàn thành các bài triển khai này, các năng lực điện toán đám mây cốt lõi sau đã được áp dụng:

* **Cơ sở hạ tầng (Infrastructure):** Thiết kế VPC, Chia Subnet (Public vs Private), NAT Gateways, Route Tables.
* **Tính toán & Container:** Quản trị EC2 (Ubuntu), Docker, Docker Compose.
* **Quản lý Cơ sở dữ liệu:** Amazon RDS (Managed) so với PostgreSQL tự host (Docker).
* **Lưu trữ & Web Hosting:** Amazon S3 Static Website Hosting.
* **Bảo mật:** Security Groups, Network ACLs, IAM Roles.
* **Quản trị Web Server:** Cấu hình Nginx và Reverse Proxy.

---

#### Điều hướng Các phần Bài Lab

1. [Mạng VPC & Bảo mật (Phần I)](1-VPC-networking/)
2. [Dịch vụ AI trên EC2 (Phần II)](2-ai-on-ec2/)
3. [NestJS Backend trên EC2 (Phần III)](3-backend-NestJS-on-ec2/)
4. [ReactJS Frontend trên S3 (Phần IV)](4-frontend-ReactJS-on-s3/)
5. [PostgreSQL trên RDS (Phần V)](5-postgresql-on-rds/)
6. [Dọn dẹp Tài nguyên (Phần VI)](6-cleanup/)

---

*Sẵn sàng khám phá hạ tầng? Hãy bắt đầu với Phần I.*