---

title: "Lab 1 Phần V: PostgreSQL trên RDS"
weight: 5
chapter: false
--------------

# Cấp Phát PostgreSQL trên Amazon RDS

#### Tại Sao Quan Trọng

Sử dụng dịch vụ cơ sở dữ liệu quản lý như Amazon RDS giúp bạn giảm tải trách nhiệm vận hành—backup, scale, patch—để bạn tập trung vào phát triển tính năng ứng dụng. Trong phần này, bạn sẽ khởi tạo một instance PostgreSQL production trong VPC, bảo mật kết nối mạng, và tích hợp với backend NestJS thông qua các migration Prisma.

---

## 1. Yêu Cầu Tiên Quyết

* **VPC**: MyVPC đã cấu hình ít nhất hai AZ với subnet công cộng và riêng tư.
* **Cài Đặt VPC**: Đảm bảo DNS resolution và DNS hostnames đã được bật cho VPC.

![Host Fullstack Web A.I](../../images/5/5-1.png?featherlight=false\&width=90pc)

* **Security Groups**: Đã có sẵn `MyVPC-sg` cho truy cập EC2 chung.

## 2. Tạo Security Group cho Database

1. Vào **EC2 → Security Groups → Create security group**.
2. **Name**: `RDS-Postgres-sg`
3. **VPC**: MyVPC
4. **Inbound rule**:

   * **Type**: PostgreSQL
   * **Port range**: 5432
   * **Source**: `MyVPC-sg` (hoặc CIDR/IP đáng tin cậy)
5. **Outbound rules**: giữ mặc định (cho phép tất cả).

## 3. Khởi Tạo Instance RDS

![Host Fullstack Web A.I](../../images/5/5-2.png?featherlight=false\&width=90pc)

1. Vào **RDS → Databases → Create database**.
2. **Engine options**: PostgreSQL (chọn phiên bản mới nhất được hỗ trợ).
3. **Templates**: Free tier (nếu đủ điều kiện) hoặc Production.
4. **DB instance identifier**: `LabPostgres`
5. **Credentials**: đặt master username (ví dụ `admin`) và mật khẩu an toàn.
6. **DB instance class**: chọn `db.t3.micro` hoặc lớn hơn.
7. **Storage**: General Purpose SSD (gp3), cấp phát 20 GiB hoặc theo nhu cầu.
8. **Connectivity**:

   * **VPC**: MyVPC
   * **Subnet group**: chọn các private subnet
   * **Public access**: No (giữ database private)
   * **VPC security group**: `RDS-Postgres-sg`
9. **Additional configuration**:

   * Enable automatic backups (giữ 7 ngày)
   * Thiết lập backup window và maintenance window theo ý muốn.
10. Nhấn **Create database** và chờ trạng thái chuyển sang **Available**.

## 4. Kết Nối & Chạy Migration

1. Trong console RDS, chọn **LabPostgres** và copy **Endpoint** (hostname).
2. Mở **pgAdmin** hoặc client SQL khác và tạo kết nối server mới:

   * **Host**: `<your-endpoint>`
   * **Port**: 5432
   * **Username**: master username
   * **Password**: master password

![Host Fullstack Web A.I](../../images/5/5-3.png?featherlight=false\&width=90pc)

* Lưu cấu hình để kết nối:

![Host Fullstack Web A.I](../../images/5/5-4.png?featherlight=false\&width=90pc)

3. Trên EC2 backend, cập nhật file `.env` tại `~/app/.env`:

   ```env
   DATABASE_URL="postgresql://my_app_role:some_password@<your-endpoint>:5432/my_app"
   ```

   ![Host Fullstack Web A.I](../../images/5/5-5.png?featherlight=false\&width=90pc)

4. Vào thư mục ứng dụng và chạy Prisma migrations:

   ```bash
   cd ~/app
   npx prisma migrate deploy
   npx prisma generate
   ```

5. (Tuỳ chọn) Mở Prisma Studio để kiểm tra schema và dữ liệu:

   ```bash
   npx prisma studio
   ```

> **Mẹo**: Bật tự động backup với chính sách giữ 7 ngày. Bạn cũng có thể scale storage theo chiều dọc mà không downtime khi dữ liệu tăng lên.
