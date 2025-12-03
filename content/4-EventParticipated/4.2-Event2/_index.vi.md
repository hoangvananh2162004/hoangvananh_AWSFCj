---
title: "Event 2"
weight: 1
chapter: false
pre: " <b> 4.2. </b> "
---

# Bài thu hoạch “​Theo AWS Well-Architected Security Pillar”

### Mục Đích Của Sự Kiện

- Welcome & Introduction.
- AWS AI/ML Services Overview.
- Generative AI with Amazon Bedrock.

### Danh Sách Diễn Giả

- **Le Vu Xuan An** – AWS Cloud Club Captain, HCMUTE, First Cloud AI Journey
- **Tran Duc Anh** – AWS Cloud Club Captain, SGU, First Cloud AI Journey
- **Tran Doan Cong Ly** – AWS Cloud Club Captain, PITT, First Cloud AI Journey
- **Danh Hoang Huu Nghi** – AWS Cloud Club Captain, HUFLIT, First Cloud AI Journey

### Tầm nhìn của Cloud Club

### Mục tiêu của Cloud Club

AWS Cloud Club hướng đến việc hỗ trợ sinh viên:

- Khám phá và phát triển kỹ năng điện toán đám mây
- Xây dựng năng lực lãnh đạo kỹ thuật
- Kết nối cộng đồng ở quy mô toàn cầu

### Cloud Club hỗ trợ sinh viên qua:

- Hoạt động trải nghiệm thực tế với AWS
- Mentorship từ các chuyên gia AWS
- Quyền lợi độc quyền cho thành viên
- Hỗ trợ lộ trình nghề nghiệp dài hạn

---

## Lộ trình Huy hiệu Captain

AWS Cloud Club cung cấp một lộ trình thăng cấp dành cho Captain:

### Các cấp độ Captain

- **Bronze**
- **Silver**
- **Gold**
- **Platinum**
- **Diamond**

### Quyền lợi theo lộ trình

- Voucher thi chứng chỉ AWS
- AWS Credits trị giá $200
- License học tập QA
- Điểm đổi swag và bộ swag dành cho Core Team
- Được pre-approval cho Student Community Day
- Có thể chuyển giao vai trò Captain cho thế hệ tiếp theo

---

## Dịch vụ Identity & Access Management (IAM)

IAM là dịch vụ quan trọng của AWS để quản lý quyền truy cập an toàn.

### IAM chịu trách nhiệm về:

- Kiểm soát truy cập bảo mật
- Quản lý **users**, **groups**, **roles**, **permissions**
- Đảm bảo **authentication** và **authorization**

### Ví dụ về nhóm IAM

- **Developers** – ví dụ: Susan
- **Security** – ví dụ: Dave
- **Admins** – ví dụ: Bob

Mỗi nhóm có mức độ quyền hạn khác nhau.

---

## Best Practices với IAM

Để vận hành hệ thống AWS an toàn, nên tuân thủ:

- Nguyên tắc **Least Privilege**
- Xóa **root access keys** ngay sau khi tạo
- Sử dụng IAM Users/Roles thay cho tài khoản root
- Tránh sử dụng ký tự `"*"` trong Action/Resource
- Đăng nhập và quản trị bằng `aws login` đúng chuẩn
