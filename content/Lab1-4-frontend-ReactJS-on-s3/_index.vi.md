---

title: "Lab 1 Phần IV: Frontend ReactJS trên S3"
weight: 4
chapter: false
--------------

# Hosting Frontend ReactJS trên S3

#### Tại Sao Quan Trọng

Cung cấp ứng dụng React của bạn dưới dạng website tĩnh trên S3 mang lại hiệu năng cực nhanh và gần như không phải quản lý vận hành. Trong phần này, bạn sẽ biến build local của React thành một trang web được lưu trữ toàn cầu. Bạn sẽ cấu hình bucket S3 cho hosting tĩnh, áp dụng kiểm soát truy cập chi tiết, và triển khai artifact production sẵn sàng—biến quy trình CI/CD phức tạp thành vài bước đơn giản.

---

## 1. Tạo Bucket S3

1. Vào **S3 → Buckets → Create bucket**.
2. Cấu hình:

   * **Bucket name**: `webreact2` (phải toàn cục duy nhất).
   * **Region**: chọn gần người dùng nhất.
   * **Block Public Access**: Bỏ chọn **Block all public access**. Xác nhận bạn hiểu bucket này sẽ phục vụ nội dung công khai.
3. (Tuỳ chọn) Thêm tag như `Project: Lab-AI` hoặc `Environment: Production` để dễ theo dõi.

![Host Fullstack Web A.I](../../images/4/4-1.png?featherlight=false\&width=90pc)

## 2. Build Ứng Dụng React Local

Trong thư mục dự án ReactJS trên máy local, mở terminal và chạy:

```bash
npm run build
```

Lệnh này tạo ra các asset tối ưu trong thư mục `build/` (hoặc `dist/`), sẵn sàng để deploy.

## 3. Bật Static Website Hosting

1. Trong console **S3**, chọn bucket và vào **Properties**.
2. Cuộn xuống **Static website hosting** và chọn **Edit**.
3. Chọn **Enable**, đặt **Index document** là `index.html`, rồi **Save changes**.
4. Ghi lại URL **Bucket website endpoint**—đây là URL public cho app React của bạn.

![Host Fullstack Web A.I](../../images/4/4-2.png?featherlight=false\&width=90pc)

## 4. Cấu Hình Chính Sách Public-Read

Cho phép mọi người lấy file tĩnh, áp dụng policy:

1. Vào **Permissions → Bucket policy → Edit**.
2. Dán JSON sau, thay `webreact2` bằng tên bucket của bạn:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::webreact2/*"
    }
  ]
}
```

3. Nhấn **Save**.

![Host Fullstack Web A.I](../../images/4/4-3.png?featherlight=false\&width=90pc)

## 5. Upload Artifact Build

1. Trong console S3, mở bucket → **Objects → Upload**.
2. Chọn **Add files** và **Add folder**, sau đó chọn tất cả nội dung trong thư mục `build/`.
3. Nhấn **Upload**—ứng dụng React của bạn đã live tại endpoint bucket.

![Host Fullstack Web A.I](../../images/4/4-5.png?featherlight=false\&width=90pc)

## 6. Kết Quả

* Chuyển sang tab **Properties**, cuộn xuống và copy URL để chạy trang trên trình duyệt:

![Host Fullstack Web A.I](../../images/4/4-6.png?featherlight=false\&width=90pc)

* Dữ liệu từ backend (AWS) được gọi vào frontend (AWS) thành công:

![Host Fullstack Web A.I](../../images/4/4-7.png?featherlight=false\&width=90pc)

## 7. (Tuỳ chọn) Tăng Tốc Phân Phối với CloudFront

Để cache toàn cầu và hỗ trợ HTTPS, bạn có thể thêm CloudFront phía trước bucket S3:

1. Tạo phân phối CloudFront mới: Origin = endpoint static website của bucket.
2. Cấu hình cache behavior mặc định, chứng chỉ SSL (AWS Certificate Manager), và tên domain tuỳ ý.
3. Triển khai, sau đó sử dụng domain CloudFront để truy cập toàn cầu nhanh nhất.

> **Mẹo**: Tự động hóa deploy với lệnh `aws s3 sync build/ s3://webreact2/ --delete` trong pipeline CI để giữ bucket đồng bộ với build mới nhất.
