---
title: "Worklog Tuần 10"
weight: 2
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10: Hoàn thiện kiến trúc

#### Nội dung làm việc

- Team thực hiện phiên presentation nội bộ:

  - rình bày flow dữ liệu từ click đến sessionization.
  - Biểu diễn luồng sự kiện, window time, logic session, cách detect user.

- Tổng hợp feedback và thống nhất phiên bản kiến trúc cuối:

  - Luồng dữ liệu real-time.
  - Cơ chế session cut-off (30 phút idle time).
  - Các lớp lưu trữ: raw, processed, sessionized.

- Kiểm tra tính khả thi triển khai thực tế (scalability, latency, fault tolerance).

#### Kết quả

- Hoàn thành kiến trúc cuối.
- Định nghĩa pipeline rõ ràng: ingestion → processing → session creation → storage → analytics.
- Tạo cơ sở tài liệu đầy đủ để bước sang phase proposal & tính chi phí.

Link sheet báo cáo team https://docs.google.com/spreadsheets/d/1Pvmgz3RBpyuaOQCA2mRM7u0Ei3s6-L72XaAuXFXOPac/edit?usp=sharing
