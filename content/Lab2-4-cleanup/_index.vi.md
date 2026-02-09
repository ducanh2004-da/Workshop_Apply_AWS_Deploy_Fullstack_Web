---

title: "Lab 2 Phần IV: Dọn Dẹp Tài Nguyên"
weight: 10
chapter: false
--------------

# Dọn Dẹp Tài Nguyên AWS

#### Tại Sao Quan Trọng

Để lại tài nguyên trên đám mây sau khi hoàn thành lab có thể phát sinh chi phí không mong muốn. Trong bước cuối này, bạn sẽ học cách an toàn loại bỏ tất cả những gì đã tạo—giúp tài khoản AWS của bạn gọn gàng và tiết kiệm chi phí.

---

## 1. Xóa Nội Dung & Xóa Bucket S3

1. **S3 → Buckets → webreact2**
2. **Empty**: nhấn **Empty bucket**, gõ `permanently delete` để xác nhận.
3. **Delete** bucket khi đã rỗng.

## 2. Terminate Các EC2 Instance

1. **EC2 → Instances**
2. Chọn `ServerAI` và `BackendServer` → **Instance state → Terminate instance**.
3. Xác nhận terminate.

## 3. Xóa Cơ Sở Dữ Liệu RDS

1. **RDS → Databases → LabPostgres**
2. **Actions → Delete**
3. Tuỳ chọn tạo snapshot cuối cùng hoặc bỏ qua để xóa hoàn toàn.
4. Xác nhận thao tác xóa.

## 4. Xóa VPC & Các Tài Nguyên Liên Quan

1. **VPC → Your VPCs → MyVPC** → **Actions → Delete VPC**
2. Hành động này sẽ tự động xóa cascade các subnet, IGW, route table và security group.

---

> **Mẹo Cuối**: Kiểm tra **AWS Cost Explorer** vào ngày hôm sau để xác nhận không còn phí giờ nào tồn đọng.
