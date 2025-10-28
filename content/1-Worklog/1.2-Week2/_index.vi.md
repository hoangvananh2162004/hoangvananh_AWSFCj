---
title: "Worklog Tuần 2"
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

- Học và hiểu VPC,VPN là gì
- Cách tạo, kết nối và hoạt động của dịch vụ mạng
- Chạy được 1 server ảo

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                         | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 1   | - Tìm hiểu khái niệm VPC, VPN tập                 | 11/08/2025   | 11/08/2025      |
| 2   | - Thực hành: Sử dụng dịch vụ của AWS với VPC, VPN | 12/08/2025   | 12/08/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | - Tạo **Subnets**, **Internet Gateway**           | 13/08/2025   | 13/08/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | - Tạo **Route Table**, **security groups**        | 13/08/2025   | 13/08/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 5   | - Tạo **Route Table**, **Security groups**        | 13/08/2025   | 13/08/2025      | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 2:

- Hiểu khái niệm VPC và VPN:

  - Amazon VPC: Cho phép tạo 1 mạng riêng ảo, khởi tạo các tài nguyên ảo của AWS như là máy chủ ảo, mạng ảo và có quyền quản lý riêng, 1 VPC cho phép chạy nhiều AZ khác nhau VPC chỉ được nằm trong 1 Region, khi khởi tạo và khai báo phải có 1 lớp mạng **IPv4** là bắt buộc, **IPv6** là tùy chọn không bắt buộc

  - Max của VPC là đucợ 5VPC trong 1 Region và 1 Account.
  - Mục đích dùng phân tách môi trường (Production, Dev, Test, Staging).
  - Chia máy ảo và chia thành các mạng con, vàn tập con này chỉ năm trong 1 AZ cụ thể và chỉ định CIDR cho mạng con đó là 1 tập còn của khối VPC CIDR.
  - tập con có 5 IP:
    - **Địa chỉ Network** (10.10.1.0).
    - **Địa chỉ Boardcast** (10.10.1.255).
    - **Địa chỉ cho bộ tuyến tính** (10.10.1.1).
    - **Địa chỉ DNS (Domain Name System)** (10.10.1.2).
    - **Địa chỉ cho tính toán tương lai** (10.10.1.3).

- Bảng định tuyến(Route Table):

  - Tập hợp các Route, xác định đường đi cho mạng.
  - Default Route Table là mặc định không xóa được, chứ duy nhất 1 Route cho phép các tập con VPC kết nối với nhau, Route Table sẽ được gán cho tập con.
  - Custom Route Table để tạo và không xóa default route được.

- ENI (Elastic Network Interface) là card mạng ảo giúp chuyển sang EC2:

  - Khỉ chuyển máy chủ mới, ENI giúp mạng ảo đảm bảo vẫn sẽ duy trì.
  - Địa chỉ IP private.
  - Địa chỉ Elastics IP address.
  - Địa chỉ MAC.

- VPC Endpoint: Mục đích cho phép kết nối với các tài nguyên trong VPC mà không cần internet, có 2 loại VPC Endpoint: Interface Endpoint sử dụng cho và Gateway Endpoint.

- VPC Security Group: Là 1 tường lửa ảo giúp lưu trữ trạng thái giúp kiểm soát lượng truy cạp đến AWS.

  - Hạn chế Security Group rule: Địa chỉ nguồn, Cổng kết nối, Security Group khác.
  - Security Group rule chỉ cho phép rule allow, được áp dụng trên các ENI.

- Network Access Control List(NACL): Là tưởng ảo không lưu trữ trạng thái, giúp kiểm soát lượng truy cập đến và đi.

  - Hạn chế NACL: Địa chỉ nguồn và Cổng kết nối.
  - Áp dụng cho VPC Subnet.
  - Mặc định NACL chỉ cho phép mọi truy cập đến và đi.

- VPC Flow Logs: là tính năng cho phép nắm bắt thông tin về IP đến và đi.
  - Các tập sẽ được đưa lên Amazon CloudWatch Logs hoặc S3.
  - Không capture nội dung gói tin.
- VPC Peering: là tính năng giúp các VPC kết nối với nhau, kết nối cần tạo 1:1 giữa hai VPC và không hỗ trợ hai VPC bị overlap IP address space.
- Transit Gateway: dùng để kết nối các VPC với nhau.

- VPN Site to Site dùng cho mô hình hybrid tạo liên kết liên tục.
- VPN Client to Site cho phép truy cập 1 host với tài nguyên VPC
- Direct Connect cho phép kết nối riêng, độ trễ 20ms-30ms

  - Elastic Load Balancing là dịch vụ cân bằng quản lý bởi AWS
  - Sử dụng **HTTP**, **HTTPS**, **TCP**(bảo mật), **SSL**, có thể public hoặc private
  - có 4 loại ELB: **Application Load Balancer**, **Network Load Balancer**, **Classic Load Balancer**, **Gateway Load Balancer**

- Thực hành:
  - Tìm kiến VPC => create vpc => đặt tên
  - Chọn IPv4 và nhập địa chỉ **10.10.0.0/16**
  - Tạo **Subnets**
  - Tạo **Internet gateways**
  - Tạo **Security groups**

![anh](/images/a1.png)
![anh](/images/a2.png)
![anh](/images/a3.png)
![anh](/images/a4.png)
![anh](/images/a5.png)
![anh](/images/a6.png)
![anh](/images/a7.png)
