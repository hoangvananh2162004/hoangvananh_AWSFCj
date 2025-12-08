---
title: "Worklog Tuần 12"
weight: 2
chapter: false
pre: " <b> 1.12 </b> "
---

### Mục tiêu tuần 12: Hoàn thiện Proposal, Kiểm thử Pipeline

#### Nội dung làm việc

- Check và hoàn chỉnh toàn bộ Proposal dự án:
  - Check lại mục tiêu – phạm vi – kiến trúc – chi phí.
  - Tối ưu phần mô tả luồng dữ liệu và logic sessionization.
  - Chính sửa proposal để chuẩn bị nộp chính thức.
- Kiểm thử pipeline ở mức logical (chưa triển khai thực tế):

  - Kiểm thử các trường hợp tạo session:
  - Người dùng tương tác liên tục.
  - Người dùng idle 30 phút.
  - Người dùng trở lại sau timeout.
  - Nhiều user, nhiều session song song.

- Kiểm thử luồng dữ liệu từ raw, processed, sessionized.
- Phân tích rủi ro & cách khắc phục:

  - Kẹt dữ liệu ở streaming layer.
  - Lambda timeout khi ingest quá lớn.
  - Tăng chi phí S3 khi lưu raw quá nhiều.

- Chuẩn bị tài liệu bàn giao và slide trình bày:
- Tạo slide kiến trúc.
- Tạo bảng chi phí ước tính cuối.
- Viết phần kết luận & đề xuất mở rộng.

#### Kết quả

- Proposal 100% hoàn chỉnh, sẵn sàng nộp chính thức.
- Pipeline logic được kiểm thử với nhiều tình huống, đảm bảo tính hợp lý.
- Tạo được bộ tài liệu đầy đủ để trình bày:
  - Slide kiến trúc
  - Mô tả sessionization logic
  - Bảng chi phí
  - Mô hình phân tầng dữ liệu
  - Hoàn thiện Risk Assessment và phương án dự phòng (backup plan).

Link sheet báo cáo team(https://docs.google.com/spreadsheets/d/1Pvmgz3RBpyuaOQCA2mRM7u0Ei3s6-L72XaAuXFXOPac/edit?usp=sharing)
