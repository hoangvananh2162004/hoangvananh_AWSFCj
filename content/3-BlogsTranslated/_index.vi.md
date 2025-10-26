---
title: "Các bài blogs đã dịch"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

### [Blog 1 - Mở rộng deployment pipeline với Amazon ECS blue/green deployments và lifecycle hooks.](3.1-Blog1/)

Bài blog giới thiệu cách mở rộng quy trình triển khai ứng dụng với Amazon ECS blue/green deployments kết hợp lifecycle hooks. Blue/green cho phép chuyển đổi lưu lượng an toàn giữa hai môi trường, trong khi lifecycle hooks (các hàm Lambda chèn vào từng giai đoạn triển khai) giúp thêm logic tuỳ chỉnh như kiểm thử, xác thực, hay phê duyệt thủ công. Nhờ đó, doanh nghiệp có thể kiểm soát tốt hơn quá trình triển khai, giảm rủi ro và dễ dàng rollback khi gặp sự cố.

### [Blog 2 - Tăng tốc việc kiểm thử serverless với tích hợp LocalStack trong VS Code IDE.](3.2-Blog2/)

Bài blog giới thiệu việc tích hợp LocalStack vào AWS Toolkit for VS Code, giúp lập trình viên dễ dàng kiểm thử và gỡ lỗi ứng dụng serverless ngay trên máy local mà không cần triển khai lên AWS thật. Nhờ đó, quá trình phát triển, kiểm thử tích hợp và triển khai ứng dụng serverless trở nên nhanh chóng, thuận tiện và liền mạch hơn ngay trong IDE.

### [Blog 3 - Xây dựng kiến trúc backup tập trung cross-Region với AWS Control Tower](3.3-Blog3/)

Bài blog trình bày cách xây dựng kiến trúc sao lưu tập trung đa vùng (cross-Region) bằng cách tích hợp AWS Backup với AWS Control Tower. Theo đó, khi bật tích hợp này, Control Tower sẽ tự động tạo các vault sao lưu trung tâm tại mỗi vùng trong một tài khoản “central backup”, và vault bản địa trong từng tài khoản workload. Với chính sách sao lưu (backup policies), bạn có thể cấu hình sao lưu cục bộ hoặc sao chép chéo vùng, áp dụng nhất quán trên toàn tổ chức — từ đó giảm độ phức tạp vận hành, đảm bảo tuân thủ và tăng khả năng dự phòng dữ liệu.
