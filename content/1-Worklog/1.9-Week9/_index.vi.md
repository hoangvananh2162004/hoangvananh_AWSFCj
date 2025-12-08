---
title: "Worklog Tuần 9"
weight: 1
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9: Xây dựng kiến trúc hệ thống

#### Nội dung làm việc

- Nghiên cứu chi tiết các dịch vụ AWS phục vụ pipeline thời gian thực:

  - Kinesis Stream / Firehose (streaming ingest)
  - AWS Lambda (xử lý real-time)
  - S3 (lưu trữ)
  - Glue / EMR (ETL & phân tích)
  - Athena / QuickSight (reporting)

- Mỗi thành viên đưa ra đề xuất kiến trúc: streaming hay batch, các bước xử lý session, data schema.
- Thảo luận về các giải pháp MLOps end-to-end cho phần processing & automation.

#### Kết quả

- Xác định được kiến trúc tổng thể phù hợp với nhu cầu: ingest → transform → sessionization → lưu trữ → analyze.
- Team thống nhất lựa chọn mô hình kiến trúc streaming-first.

Link sheet báo cáo team https://docs.google.com/spreadsheets/d/1Pvmgz3RBpyuaOQCA2mRM7u0Ei3s6-L72XaAuXFXOPac/edit?usp=sharing
