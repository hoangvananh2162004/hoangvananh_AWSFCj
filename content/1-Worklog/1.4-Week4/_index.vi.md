---
title: "Worklog Tuần 4"
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

- Tìm hiểu về máy tính ảo trên AWS .
- Có các hệ điều hành nào và cái nào dùng phổ biến nhất trên AWS.
- EC2 được điều hành như thế nào và tại sao lại dùng nhiều.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                                             | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | --------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 1   | - Amazon Elastic Compute Cloud ( EC2 ) - Instance type AWS.           | 29/09/2025   | 29/09/2025      |
| 2   | - Amazon Elastic Compute Cloud ( EC2 ) - AMI / Backup / Key Pair EC2. | 30/09/2025   | 30/09/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | - Amazon Elastic Compute Cloud ( EC2 ) - Elastic block store.         | 30/09/2025   | 30/09/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | - Amazon Elastic Compute Cloud ( EC2 ) - Instance store               | 30/09/2025   | 30/09/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 5   | - Amazon Elastic Compute Cloud ( EC2 ) - User data                    | 30/09/2025   | 30/09/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 6   | - Amazon Elastic Compute Cloud ( EC2 ) - Meta data                    | 30/09/2025   | 30/09/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 7   | - Amazon Elastic Compute Cloud ( EC2 ) - EC2 auto scaling             | 1/10/2025    | 1/10/2025       | <https://cloudjourney.awsstudygroup.com/> |
| 8   | - EC2 Autoscaling - EFS/FSx - Lightsail - MGN                         | 1/10/2025    | 1/10/2025       | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 4:

- Khái niệm máy ảo và các dịch vụ về máy ảo:

  - Amazon Elastic Computer Cloud (EC2)
  - Amazon Lightsail
  - Amazon EFS/FSX
  - AWS Application Migration Service(MGN)

- Amazon Elastic Compute Cloud (EC2):

  - EC2 là dịch vụ về máy ảo và nó gần giống với máy chủ vật lý và máy chủ ảo truyền thống. Nhưng EC2 khởi tạo nhanh hơn, khả năng co giãn tài nguyên linh hoạt hơn.
  - EC2 có thể hỗ trợ các workload để lưu trữ website, ứng dụng, cơ sở dữ liệu và các dịch vụ nào máy chủ truyền thống làm được thì EC2 điều làm được.
  - Để được gọi là Instance type phải có đủ yếu tố như sau.
    - CPU/GPU.
    - Memory.
    - Network.
    - Storage
  - AMI(Amazon Machine Image)/Backup/Keypair.
    - AMI(Amazon Machine Image) dùng để provisitor ra 1 hoặc nhiều EC2 Instance cùng 1 lúc.
    - AMI có sẵn của AWS, trên AWS Market Place và custom AMI tự tạo từ EC2 Instances.
    - AMI bao gồm root OS volumes, quyền sử dụng AMI quy định tài khoản AWS được sử dụng và mapping EBS volume sẽ được tạo và gán vào EC2 Instances.
    - EC2 instance có thể được backup bằng cách tạo snapshot.
    - Key pair (public key và private key) dùng để mã hóa thông tin đăng nhập cho EC2 Instances.
  - Elastic block store.

    - Amazon EBS cung cấp block storage và được gắn trực tiếp vào EC2 Instance, tuy được gắn trực tiếp như 1 RAW device, EBS về bản chất hoạt động độc lập với EC2 và được kết nối thông qua mạng riêng của EBS.
    - EBS có hai nhóm đĩa chính là HDD và SSD, được thiết kế để đạt độ sẵn sàng 99.999% bằng cách replicate dữ liệu giữa 3 Storage Node trong 1 AZ.
    - Có một số EC2 Instances đặc thù được tối ưu hóa hiệu năng của EBS. (Optimized EBS Instances)
    - EBS volumes, mặc định chỉ được gắn vào 1 EC2 Instances, EC2 Instances chạy trên Hypervisor Nitro có thể dùng 1 EBS volume gắn vào nhiều EC2 Instances. (EBS Multi attach)
    - EBS được backup bằng cách thực hiện snapshot vào S3 (Simple Storage Service).
      - Snapshot đầu tiên là full, tất cả các snapshot tiếp theo là incremental.

  - Instance store là vùng dữ liệu NVMe tốc độ cao, nằm trên physical node chứa các máy ảo EC2.

    - Instance store sẽ bị xóa hết dữ liệu khi chương trình ta thực hiện stop EC2 instance.
    - nstance store không bị xóa dữ liệu khi chương trình ta thực hiện restart máy, nhưng bị crash.
    - Instance store không replicate dữ liệu để phòng né tránh trường hợp khuyên kích lưu trữ dữ liệu quan trọng.
    - Sử dụng trong môi trường eps thi điều co môt cai tinh @trieu lops
    - Khi sử dụng thường dự replicate dữ liệu vào một EBS volume để bảo đảm an toàn.

  - EC2 user data là đoạn script chạy một lần khi provision EC2 instance từ AMI.

    - Tùy hệ điều hành máy chủ ta sẽ dùng bash shell scripts (Linux) / powershell (Windows).

  - EC2 Metadata, là các thông tin liên quan đến bàn thân EC2 instances, ví dụ địa chỉ IP Private, Public, Hostname, Security groups....

  - EC2 Auto Scaling là tính năng hỗ trợ tăng giảm số lượng EC2 Instance dựa theo các điều kiện cụ thể (scaling policy).
    - EC2 Auto Scaling có thể đăng ký các EC2 Instance vào Elastic Load Balancer.
    - EC2 Auto Scaling hoạt động trên nhiều AWS Availability Zone.
    - EC2 Auto Scaling có thể hỗ trợ Pricing options khác nhau.

- EC2 Autoscaling - EFS/FSx - Lightsail - MGN:
  - EC2 bao gồm 4 tùy chọn giá.
    - On-demand Trả theo giờ / phút / giây, xài nhiêu tính nhiêu, máy nhật. Phù hợp cho các workload chạy lén tối 6 tiếng 1 ngày.
    - Reserved Instance : Cam kết sử dụng theo kì han 1-3 năm để lấy discount, tuy nhiên bị giới hạn theo EC2 Instance type/family.
    - Saving Plans : Cam kết sử dụng theo kì han 1-3 năm để lấy discount, có thể ko bị giới hạn bộ EC2 Instance type family.
    - Spot Instance : Tận dụng tài nguyên dư, giá rẻ tùy nhiên khi cán thị AWS sẽ terminate instance trong 2 phút.
  - Amazon Lightsail.
    - Là dịch vụ tính toán chi phí thấp (giá tính theo tháng, bắt đầu từ 3.5 USD/tháng). Ngoài ra, mỗi Instance Lightsail tạo ra cũng sẽ có một mức data transfer đi kèm (data transfer này có mức giá rẻ hơn so với data transfer từ EC2 khá nhiều).
    - Amazon Lightsail phù hợp cho các workload nhẹ, môi trường test/dev, không yêu cầu CPU cao liên tục (hơn 2 giờ mỗi ngày).
    - Amazon Lightsail cũng có khả năng backup bằng snapshot tương tự như EC2.
    - Amazon Lightsail chạy trong một môi trường VPC độc lập và có thể kết nối với VPC thông qua việc thiết lập VPC Peering.
  - Amazon EFS/FSX.
    - EFS
      - EFS (Elastic File System) cho phép tạo các NFSv4 Network volume và gắn vào nhiều EC2 Instances cùng lúc, quy mô lưu trữ lên đến hàng petabyte. EFS chỉ support Linux.
      - Sử dụng EFS chỉ tính chi phí theo dung lượng sử dụng (trong khi EBS tính phí theo dung lượng cấp phát).
      - EFS có thể được cấu hình để mount vào môi trường on-premise qua DX hoặc VPN.
    - FSX
      - FSx cho phép tạo các NTFS volume và gắn vào nhiều EC2 Instances cùng lúc sử dụng giao thức SMB (Server Message Block), FSx support Windows và Linux.
      - Sử dụng FSx chỉ tính chi phí theo dung lượng sử dụng (trong khi EBS tính phí theo dung lượng cấp phát).
      - FSx hỗ trợ tính năng deduplication, giúp giảm chi phí 30-50% cho các trường hợp sử dụng thông thường.
- AWS Application Migration Serveice(MGN):
  - AWS Application Migration Service (MGN) được dùng để migrate và replicate nhằm mục đích xây dựng Disaster Recovery Site cho các máy chủ thực, vào trong môi trường AWS.
  - Application Migration Service (MGN) liên tục sao chép các máy chủ nguồn sang EC2 Instances trên tài khoản AWS (asynchronous/synchronous).
  - Trong quá trình sao chép, MGN sẽ sử dụng các máy staging có số lượng và quy mô cấu hình nhỏ hơn máy chủ gốc rất nhiều.
  - Khi thực hiện cut-over, MGN sẽ dừng và khởi chạy các máy chủ EC2 trên AWS.
