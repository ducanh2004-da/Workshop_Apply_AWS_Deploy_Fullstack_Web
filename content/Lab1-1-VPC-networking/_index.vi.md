---

title: "Lab 1 Phần I: Mạng & Bảo mật VPC"
weight: 1
chapter: false
--------------

# Mạng & Bảo mật VPC

#### Tại Sao Quan Trọng

Một mạng lưới vững chắc là nền tảng của mọi ứng dụng trên đám mây. Trong phần này, bạn sẽ thiết kế một Virtual Private Cloud (VPC) đa AZ với cả subnet công cộng và riêng tư, đính kèm Internet Gateway, cấu hình bảng định tuyến, và chặn lưu lượng với security group. Cuối cùng, môi trường của bạn sẽ sẵn sàng để lưu trữ từ các máy chủ web hướng internet đến workload AI nội bộ—một cách bảo mật và tin cậy.

---

#### Mục Tiêu

## Xây dựng một VPC trải rộng trên 3 Availability Zone (AZ). Mỗi AZ sẽ có một subnet công cộng, và VPC chỉ cần một subnet riêng.

## 1. Tạo VPC của Bạn

1. Điều hướng tới **VPC → Your VPCs → Create VPC**
2. Đặt tên **MyVPC**, nhập CIDR (ví dụ `10.0.0.0/16`), và chọn tenancy **default**.
3. Nhấn **Create**—bạn đã có một VPC trải khắp tất cả AZ trong vùng.

> **Lưu ý:** chúng ta sẽ tạo tổng cộng 4 subnet: 3 subnet công cộng và 1 subnet riêng.

![Host Fullstack Web A.I](../../images/1/1-1.png?featherlight=false\&width=90pc)

## 2. Xây dựng Các Subnet Công Cộng Đầu Tiên

1. Vào **VPC → Subnets → Create Subnet**
2. Đối với mỗi AZ (`us-east-1a`, `us-east-1b`, `us-east-1c`):

   * **Name**: `PublicSubnet1` (sau đó `PublicSubnet2`, `PublicSubnet3`)
   * **VPC**: MyVPC
   * **CIDR**: chọn các dải không chồng lấn, ví dụ `10.0.0.32/27`, `10.0.0.64/27`, `10.0.0.96/27`

![Host Fullstack Web A.I](../../images/1/1-2.png?featherlight=false\&width=90pc)

3. Sau khi tạo xong, chọn từng subnet → **Actions → Edit auto-assign IP settings** → bật **Enable auto-assign public IPv4 address** để biến thành subnet công cộng.

> **Lưu ý:** tiếp tục tương tự để tạo các subnet công cộng còn lại.

![Host Fullstack Web A.I](../../images/1/1-3.png?featherlight=false\&width=90pc)

## 3. Xây dựng Subnet Riêng

1. Tạo **Subnet** như trước:

   * **Name**: `PrivateSubnet1`
   * **CIDR**: ví dụ `10.0.1.0/27`
   * Chọn bất kỳ một AZ

![Host Fullstack Web A.I](../../images/1/1-4.png?featherlight=false\&width=90pc)

2. Giữ **auto‑assign public IP** ở trạng thái tắt—subnet này chỉ dành cho dịch vụ nội bộ.

## 4. Internet Gateway & Bảng Định Tuyến

* **Internet Gateway**: Tạo và gắn Internet Gateway vào VPC để kết nối Internet

  1. **VPC → Internet Gateways → Create**
  2. Tên: `myIGW`
  3. Chọn → **Actions → Attach to VPC** → MyVPC

![Host Fullstack Web A.I](../../images/1/1-5.png?featherlight=false\&width=90pc)

* **Bảng Định Tuyến Công Cộng**: định tuyến cho subnet công cộng

  1. **VPC → Route Tables → Create**
  2. Tên: `MyPublic-rtb`, VPC: MyVPC
  3. Trong **Routes** → **Edit** → Thêm tuyến: `0.0.0.0/0` → Target: `myIGW`
  4. **Subnet Associations** → **Edit** → gắn tất cả ba `PublicSubnet*`.

![Host Fullstack Web A.I](../../images/1/1-7.png?featherlight=false\&width=90pc)

* **Bảng Định Tuyến Riêng**: định tuyến cho subnet riêng

  1. **VPC → Route Tables → Create**
  2. Tên: `MyPrivate-rtb`, VPC: MyVPC
  3. Chỉ giữ tuyến local mặc định.
  4. **Subnet Associations** → gắn `PrivateSubnet1`.

## 5. Security Group

1. **EC2 → Security Groups → Create**
2. **Name**: `MyVPC-sg`, **VPC**: MyVPC
3. **Description**: `cho phép SSH và truy cập nội bộ trong VPC`
4. **Inbound**:

   * SSH (TCP, port 22) chỉ từ IP của bạn
   * Các port ứng dụng cần thiết (ví dụ HTTP/HTTPS)
5. **Outbound**: cho phép tất cả.

![Host Fullstack Web A.I](../../images/1/1-9.png?featherlight=false\&width=90pc)

---

> **Mẹo:** Luôn gắn tag tài nguyên với `Project: Lab-AI` (hoặc tương tự)—giúp việc dọn dẹp và quản lý chi phí dễ dàng hơn!
