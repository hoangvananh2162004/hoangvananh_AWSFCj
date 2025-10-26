---
title: "Worklog Tuần 1"
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu cần làm tuần 1:

- Lên văn phòng cùng team và hiểu nội quy của văn phòng.
- Hiểu cách viết workshop.
- Tìm hiểu AWS tạo tài khoản.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                                                                                  | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                                                                 |
| --- | ---------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ---------------------------------------------------------------------------------------------- |
| 1   | - Hiểu rõ nội quy của văn phòng và thực hiện đúng cho những buổi sau                                       | 08/09/2025   | 09/09/2025      |
| 2   | - Viết workshop cách tạo sườn                                                                              | 10/09/2025   | 13/09/2025      | <https://www.youtube.com/watch?v=mXRqgMr_97U&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=3>  |
| 3   | - Tạo tài khoản AWS và làm nhiệm vụ nhận 200 credit free <br> - Thực hành sơ bộ với các nhiệm vụ nhận 200$ | 14/09/2025   | 14/09/2025      |                                                                                                |
| 4   | - Học và hiểu về cloud                                                                                     | 15/09/2025   | 17/09/2025      | <https://www.youtube.com/watch?v=HxYZAK1coOI&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=4>  |
| 5   | - Thực hiện tạo IAM user để admin user và admin group                                                      | 16/09/2025   | 16/09/2025      | <https://www.youtube.com/watch?v=b9pK1oG534Q&list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&index=13> |

### Kết quả đạt được tuần 1:

- Lên văn phòng kết nối được với các đồng nghiệp cùng làm tại AWS First Cloud Journey, Làm quen được với 2 bạn học về 1 bạn học BE và 1 bạn Chung ngành khác nhóm.

- Đã chạy được hugo và test thành công, hiểu được cách viết của workshop hỗ trợ cho việc viết cho các tuần sau cũng cung như proposal và translated blogs(Done và hiểu).

- Hiểu về cloud là gì:

  - Là sự phân phối tài nguyên Công Nghệ Thông Tin thông qua Internet.
  - Thanh toán tài nguyên theo mức độ sử dụng.
  - Tối ưu hóa được chi phí, xài bao nhiêu tính bấy nhiêu, tốc độ nhanh.
  - Linh hoạt trong việc thêm hoặc bớt tài nguyên 1 cách hợp lý.
  - Quy mô rộng, ứng dụng trên toàn cầu.

- Để học được Cloud thì:

  - Kết bạn học hỏi cùng nhau, học từ các người đi trước
  - Trải nghiệm và sử dụng các dịch vụ của AWS
  - Học qua các nền tảng online học tư học

- Tạo tài khoản hiểu về AWS và các công cụ:

  - Tạo thành công tài khoản và nhận được 200$ sau khi làm các nhiệm vụ activate.
  - Hiểu rõ root user và IAM user, root user là tài khoản gốc quản lý tất cả, IAM user được tạo ra bởi root user và được tài khoản gốc cấp quyền quản lý.
  - Tạo MFA cho tài khoản đảm bảo an toàn bảo mật.

- Tạo user:
  - Để tạo vào console của AWS tìm kiếm **IAM (Identity and Access Management)**.
  - Sau đó thấy Dashboard của nó và chú ý đến hai yếu tố là **User** và **User Group**'
  - Tạo User chọn User để tạo và bấm vào **create user**.
  - Nhập tên user và tích chọn **Provide user access to the AWS Management Console - optional**, để tạo mật khẩu chọn **user I want to create an IAM user** => **I want to create an IAM user** => **next**
  - Bấm next tiếp và create user
- Tạo user:
  - Tạo User Group vào console của AWS tìm kiếm **IAM (Identity and Access Management)**.
  - Chọn **User Group** => **Create user group** => **Nhập tên group và tạo**.
  - Chọn vào group đã tạo, chon add user để add, chon user cần add và add user.
    ![anh](/static/images/1.png)
    ![anh](/static/images/2.png)
    ![anh](/static/images/3.png)
    ![anh](/static/images/4.png)
    ![anh](/static/images/5.png)
    ![anh](/static/images/6.png)
    ![anh](/static/images/7.png)
    ![anh](/static/images/8.png)
    ![anh](/static/images/9.png)
    ![anh](/static/images/10.png)
    ![anh](/static/images/11.png)
