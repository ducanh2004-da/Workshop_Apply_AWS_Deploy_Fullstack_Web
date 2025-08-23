# 🏗️ Workshop: Ứng dụng AWS để deploy Full‑Stack web (ReactJS + NestJS)

**Ngôn ngữ:** Tiếng Việt + Tiếng Anh

**Mục tiêu:** Tài liệu giới thiệu và hướng dẫn tổng quan cho bài nghiên cứu / workshop về triển khai một ứng dụng Full‑Stack (ReactJS frontend + NestJS backend) trên AWS sử dụng **S3**, **EC2** và **RDS**. Tài liệu phù hợp cho buổi workshop từ 2–4 tiếng, hoặc làm tài liệu tham khảo để thực hành độc lập.

---

## 🔎 Tổng quan bài toán

Trong workshop này, người học sẽ:

* Xây dựng/chuẩn bị một ứng dụng React (frontend) và một API NestJS (backend) kết nối tới cơ sở dữ liệu PostgreSQL.
* Đóng gói và deploy frontend lên **Amazon S3 + CloudFront** (hosting static site).
* Triển khai backend trên **Amazon EC2** (hoặc dùng ECS/Elastic Beanstalk nếu muốn nâng cấp sau này).
* Lưu dữ liệu trên **Amazon RDS (PostgreSQL)**.
* Thiết lập cấu hình mạng (VPC / Subnet / Security Group), IAM role, biến môi trường và cơ chế backup/monitoring.

---

## 🎯 Mục tiêu học tập

* Hiểu kiến trúc tối thiểu để chạy một ứng dụng full‑stack trên AWS.
* Biết cách cấu hình S3 để host static assets và tích hợp CloudFront để tăng hiệu năng.
* Triển khai NestJS lên EC2, cấu hình reverse proxy (nginx), systemd và deploy tự động cơ bản.
* Thiết lập RDS PostgreSQL và cấu hình an toàn truy cập từ EC2.
* Áp dụng các best practices về bảo mật, logging, backup và tối ưu chi phí.

---

## 🧭 Kiến trúc (architecture) — mô tả

```
User
  ↓ HTTPS
CloudFront (CDN)
  ↓
S3 (React static files)

User -- HTTPS --> ELB / ALB --> EC2 (NestJS, Nginx) --> RDS (Postgres)

Optional: Route53 (DNS) + ACM (TLS cert)
Monitoring: CloudWatch
Backups: RDS Snapshots
```

**Ghi chú:** Trong workshop ta sử dụng EC2 cho backend để minh họa quá trình deploy thủ công; ở bước nâng cao có thể chuyển sang ECS/EKS/Elastic Beanstalk để học autoscaling.

---

## 🧩 Thành phần chính & vai trò

* **ReactJS (frontend)**: ứng dụng SPA build bằng `npm run build` → output là static files (`index.html`, JS bundle, CSS) upload lên S3.
* **S3**: lưu trữ static assets; kết hợp **CloudFront** để phân phối nhanh và bảo mật (HTTPS, WAF tuỳ chọn).
* **NestJS (backend)**: REST / GraphQL API chạy trên EC2; giao tiếp với RDS để đọc/ghi dữ liệu.
* **EC2**: host ứng dụng backend; quản lý bằng SSH, systemd, Nginx reverse proxy; có thể dùng AMI hoặc Docker.
* **RDS (Postgres)**: database managed, backup tự động, multi‑AZ (tuỳ chọn chi phí).
* **IAM**: roles & policies cho EC2 (ví dụ để truy cập S3 nếu cần), user để quản trị.
* **VPC & Security Groups**: tách subnet public/private — EC2 trong public/private tuỳ kiến trúc; RDS nên nằm trong private subnet.

---

## ✅ Các bước triển khai (tổng quan thực hành)

1. **Chuẩn bị project**

   * Frontend: có script build (`npm run build`).
   * Backend: có file `.env` cấu hình DB, PORT, JWT secret; script start `node dist/main.js` hoặc `npm run start:prod`.

2. **Thiết lập AWS cơ bản**

   * Tạo IAM user cho workshop (quyền hạn tối thiểu khi thực hành) hoặc dùng AWS Educate/Free Tier.
   * Tạo VPC + subnet (public/private), security groups.

3. **Tạo RDS (Postgres)**

   * Chọn engine PostgreSQL, tạo instance trong private subnet.
   * Thiết lập username/password, bật automated backups.
   * Thêm rule Security Group để cho phép kết nối từ EC2 (port 5432).

4. **Deploy backend trên EC2**

   * Tạo EC2 instance (Ubuntu), SSH vào, cài Node.js, PM2 hoặc systemd, Nginx.
   * Pull code từ GitHub (hoặc SCP file), cài dependencies và build.
   * Cấu hình `.env` để kết nối RDS.
   * Cấu hình Nginx làm reverse proxy và SSL (có thể dùng ACM + ALB hoặc Certbot nếu dùng IP trực tiếp).

5. **Host frontend trên S3 + CloudFront**

   * Tạo S3 bucket public hoặc private với static website hosting (prefer private + CloudFront).
   * Upload thư mục `build` từ React.
   * Tạo distribution CloudFront và gắn domain (tuỳ chọn), dùng ACM cho TLS.

6. **CI/CD (tự động hóa)**

   * Thiết lập GitHub Actions: pipeline build frontend → deploy S3; build backend → SSH & restart service trên EC2 (hoặc Docker push & deploy).

7. **Monitoring & Backup**

   * CloudWatch logs cho backend (via CloudWatch Agent hoặc ghi log file + forward).
   * RDS snapshots + automated backup policy.

---

## 🔒 Bảo mật & Best practices

* Đặt RDS trong **private subnet**; không mở public access.
* Sử dụng **Security Group** theo nguyên tắc ít đặc quyền (least privilege).
* Dùng **IAM role** cho EC2 khi cần truy cập tài nguyên AWS (ví dụ upload ảnh lên S3).
* Không lưu secrets trong mã nguồn — dùng **AWS Systems Manager Parameter Store** hoặc **Secrets Manager**.
* Bật TLS (HTTPS) cho frontend và backend (ACM + CloudFront / ALB).

---

## 📈 Monitoring, Logging & Cost

* **Logging:** CloudWatch Logs, X‑Ray (tracing nâng cao)
* **Alerting:** CloudWatch Alarms (CPU, Memory, Disk, DB connections)
* **Cost:** Dùng Free‑Tier hoặc nhỏ (t2.micro/t3.micro) cho EC2, Single‑AZ RDS nhỏ; chú ý bandwidth CloudFront/S3.

---

## 🛠️ Mẹo khắc phục sự cố thường gặp

* Không kết nối được DB: kiểm tra Security Group của RDS, endpoint & port, biến môi trường trên EC2.
* Backend 502/504: kiểm tra Nginx config, service status (`systemctl status yourapp`), xem log.
* S3 trả về 403 cho file: kiểm tra permission bucket / policy / CloudFront origin access identity.

---

## 🗂️ Tài nguyên tham khảo & mở rộng

* Nâng cấp backend: Dockerize app → dùng ECS hoặc EKS.
* Xây dựng autoscaling: ALB + Auto Scaling Group.
* Thực hành CI/CD an toàn: lưu secrets trong GitHub Secrets hoặc dùng OIDC + IAM role.

---

## ⏱️ Gợi ý lịch trình workshop (3 giờ — mẫu)

1. 00:00–00:20 — Giới thiệu kiến trúc & mục tiêu.
2. 00:20–01:00 — Tạo RDS & cấu hình VPC / IAM.
3. 01:00–01:40 — Deploy backend trên EC2 (SSH, config, chạy service).
4. 01:40–02:10 — Build & deploy frontend lên S3 + CloudFront.
5. 02:10–02:40 — Thiết lập CI/CD cơ bản (GitHub Actions).
6. 02:40–03:00 — Q\&A, troubleshooting & bước nâng cao.

---

## 📎 Appendix — Các lệnh/command thường dùng (ví dụ nhanh)

```bash
# Build frontend
cd frontend && npm install && npm run build

# Upload build lên S3 (AWS CLI)
aws s3 sync build/ s3://your-bucket --delete

# On EC2: chạy backend
git clone https://github.com/you/your-backend.git
cd your-backend
npm install
npm run build
# dùng systemd or pm2
pm2 start dist/main.js --name your-app

# Check logs
journalctl -u your-app -f
```

---

Nếu bạn muốn, mình có thể:

* Tạo một **phiên bản tiếng Anh** của tài liệu này.
* Thêm **mẫu file GitHub Actions** để deploy frontend → S3 và backend → EC2.
* Soạn sẵn **script Terraform** để provision VPC / EC2 / RDS / S3 (infra as code).

Bạn muốn mình làm tiếp phần nào ngay bây giờ?
