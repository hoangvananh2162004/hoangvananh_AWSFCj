---
title: "Blog 3"
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# Xây dựng kiến trúc backup tập trung cross-Region với AWS Control Tower

Bởi Chris Falk, Lei Shi, và Pujah Goviel | vào ngày 11 THÁNG 9, 2025 | trong [Advanced (300)](https://aws.amazon.com/blogs/storage/category/learning-levels/advanced-300/), [AWS Backup](https://aws.amazon.com/blogs/storage/category/storage/aws-backup/), [AWS Control Tower](https://aws.amazon.com/blogs/storage/category/management-tools/aws-control-tower/), [AWS Key Management Service (KMS)](https://aws.amazon.com/blogs/storage/category/security-identity-compliance/aws-key-management-service/), [AWS Organizations](https://aws.amazon.com/blogs/storage/category/security-identity-compliance/aws-organizations/), [Best Practices](https://aws.amazon.com/blogs/storage/category/post-types/best-practices/),[Enterprise governance and control](https://aws.amazon.com/blogs/storage/category/management-and-governance/enterprise-governance-and-control/), [Resilience](https://aws.amazon.com/blogs/storage/category/resilience/), [Storage](https://aws.amazon.com/blogs/storage/category/storage/) | [Permalink](https://aws.amazon.com/blogs/storage/build-centralized-cross-region-backup-architecture-with-aws-control-tower/) | [Bình luận](https://aws.amazon.com/blogs/storage/build-centralized-cross-region-backup-architecture-with-aws-control-tower/#Comments)

Quản lý bảo vệ dữ liệu ở quy mô lớn là một thách thức quan trọng đối với các doanh nghiệp hiện đại. Khi tổ chức phát triển, dữ liệu của họ ngày càng phân tán, khiến cho việc triển khai các chính sách hỗ trợ nhất quán để đảm bảo phạm vi bao phủ toàn diện trở nên khó khăn. Các nhóm IT phải cân bằng giữa những yêu cầu cạnh tranh như tuân thủ quy định, bảo vệ tài nguyên và hiệu quả vận hành – đồng thời phải đối mặt với khó khăn trong việc xác thực và điều phối quy trình hỗ trợ trên một hạ tầng số ngày càng mở rộng.

[AWS Backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html) mang đến một giải pháp mạnh mẽ cho những thách thức này với một dịch vụ tập trung, được quản lý hoàn toàn, giúp đơn giản hóa việc bảo vệ dữ liệu ở quy mô lớn. Doanh nghiệp có thể tận dụng [AWS Organizations](https://aws.amazon.com/organizations/) cùng với AWS Backup để triển khai các chính sách backup tự động và nhất quán trên toàn bộ môi trường cloud của họ. Sự tích hợp với [AWS Control Tower](https://docs.aws.amazon.com/controltower/latest/userguide/what-is-control-tower.html) còn đơn giản hóa quá trình này hơn nữa bằng cách cho phép tổ chức tích hợp quản lý backup trên toàn doanh nghiệp trực tiếp vào multi-account landing zone được thiết kế theo kiến trúc chuẩn.

Trong bài viết này, chúng tôi sẽ trình bày cách triển khai AWS Backup thông qua tích hợp với AWS Control Tower. Chúng tôi sẽ khám phá kiến trúc, các prerequisites, và quy trình triển khai từng bước. Bằng cách làm theo hướng dẫn này, bạn sẽ học cách tự động triển khai và quản lý các backup policies trên toàn tổ chức, giúp đáp ứng yêu cầu compliance, bảo vệ các critical resources, và giảm thiểu chi phí quản trị. Giải pháp này đặc biệt hữu ích cho các doanh nghiệp đang muốn chuẩn hóa hoạt động backup trong khi vẫn mở rộng môi trường cloud của mình.

## Tổng quan giải pháp

AWS Control Tower hiện cung cấp [built-in capabilities](https://aws.amazon.com/about-aws/whats-new/2024/11/aws-control-tower-prescriptive-backup-plans-landing-zone/) để tối ưu hóa việc backup management ở quy mô lớn thông qua tích hợp trực tiếp với AWS Backup. Tính năng này sẽ tự động tạo một central backup vault trong mỗi [AWS Region](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) bên trong một central backup account chuyên dụng. Khi tổ chức của bạn mở rộng ra nhiều accounts và regions, AWS Control Tower sẽ tự động tạo các local backup vaults trong từng workload account thuộc các organizational units (OUs) đã được AWS Backup-enabled trên tất cả các governed AWS Regions. Bạn có thể tùy chỉnh chiến lược backup của mình bằng cách cấu hình backup policies để sao chép dữ liệu từ local vaults sang central vaults, có thể trong cùng một Region hoặc cross-Region. Ngoài ra, bạn không cần đến các custom deployment pipelines hoặc complex automation. Bạn có thể sử dụng AWS Backup policies để triển khai một chiến lược backup toàn diện, đáp ứng các yêu cầu cụ thể của tổ chức trong khi vẫn duy trì tính governance nhất quán trên toàn bộ môi trường AWS.

![anh](/images/001-ST-1631-solution-overview.png)

## Điều kiện tiên quyết

Trước khi triển khai tích hợp **AWS Control Tower** và **AWS Backup,** hãy đảm bảo rằng bạn đã có sẵn các thành phần nền tảng sau:

- Một **AWS Organization** hiện có được **governed** bởi **AWS Control Tower**
- Quyền truy cập **Administrative** vào **AWS Organizations management account**

## Walkthrough

Phần này sẽ hướng dẫn bạn qua các bước thực hiện giải pháp.

### Bước 1: Cấp phát dedicated backup accounts

Trước tiên, bạn cần tạo hai specialized accounts đóng vai trò là nền tảng cho hạ tầng backup của bạn:

- **Backup administrator account:** Tài khoản này quản lý backup policies và configurations cho toàn bộ tổ chức của bạn.
- **Central backup account**: Tài khoản này lưu trữ centralized backup vaults và các cross-account backup copies.

Tạo hai accounts này thông qua Organizations console hoặc  [AWS Command Line Interface (AWS CLI)](https://aws.amazon.com/cli/). Không sử dụng AWS Control Tower Account Factory hoặc bất kỳ quy trình AWS account vending nào khác mà sẽ tự động enroll các new accounts vào AWS Control Tower governance. Ở bước ba, khi bạn enable AWS Backup trong AWS Control Tower, các accounts này sẽ được enrolled và tự động đưa vào Security OU.

### Bước 2: Tạo một multi-Region AWS KMS Key

Backup encryption là một yêu cầu bảo mật quan trọng trong các enterprise organizations. Hãy tạo một multi-Region [AWS Key Management Service (AWS KMS)](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html) key trong management account của bạn để kích hoạt mã hóa dữ liệu backup khi lưu trữ (at rest) và đảm bảo khả năng cross-account backup an toàn. Key này cần được tạo trong AWS Control Tower Home Region và được replicate đến tất cả các governed AWS Regions. Multi-Region AWS KMS key này sẽ được sử dụng để mã hóa backups trong central backup account vaults, cũng như tất cả local vaults trong các workload accounts trên tất cả các Regions. Mặc dù bạn có thể tạo key này thủ công thông qua [AWS Management Console](https://aws.amazon.com/console/), chúng tôi khuyến nghị sử dụng infrastructure as code (IaC) như ([AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html), [AWS Cloud Development Kit (AWS CDK)](https://aws.amazon.com/cdk/), or [Terraform](https://aws.amazon.com/cdk/)).

Tham khảo tài liệu [AWS Control Tower](https://docs.aws.amazon.com/controltower/latest/userguide/backup-prerequisites.html) để biết thêm chi tiết về cách tạo AWS KMS key policy liên quan.

### Bước 3: Enable AWS Backup trong AWS Control Tower

Bây giờ bạn có thể tích hợp AWS Backup với AWS Control Tower landing zone. Điều hướng đến **AWS Control Tower console**, chọn **Landing zone setting**s, và chọn Update landing zone. Trong phần backup configuration, nhập hoặc chọn các thông tin sau:

- **Central backup account ID**: ID gồm 12 chữ số của Central Backup account mà bạn đã tạo ở Bước 1.
- **Backup administrator account ID**: ID gồm 12 chữ số của backup administrator account mà bạn đã tạo ở Bước 1.
- **KMS Key ARN**: [Amazon Resource Name (ARN)](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html) của AWS KMS key mà bạn đã tạo ở Bước 2.

Bạn có thể chọn các accounts đã tạo ở Bước 1 và AWS KMS key ở Bước 2 từ drop-down list trong mỗi text box, như minh họa trong hình dưới đây.

![anh](/images/002-ST-1631-aws-backup-settings.png)

Quá trình landing zone update có thể mất từ 30–45 phút để hoàn tất. Khi hoàn thành, bạn sẽ thấy trạng thái AWS Backup là **Enabled**, và các accounts cùng AWS KMS keys mà bạn chỉ định sẽ xuất hiện trong AWS Control Tower console, như minh họa trong hình dưới đây.

![anh](/images/003-ST-1631-aws-backup-status.png)

### Bước 4: Cấu hình service opt-in và enable delegated administrator

Sau khi enable AWS Backup trong AWS Control Tower, cấu hình service opt-in trong AWS Backup console.
Điều hướng đến AWS Backup console, chọn Settings, và trong danh sách Service opt-in:

- Bật opt-in cho các dịch vụ cần thiết cho workloads mà bạn muốn bao phủ bằng AWS Backup, chẳng hạn như [Amazon Relational Database Service (Amazon RDS)](https://aws.amazon.com/rds/), [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/), [Amazon DynamoDB](https://aws.amazon.com/dynamodb/), [Amazon Elastic Block Store (Amazon EBS)](https://aws.amazon.com/ebs/), [Amazon Elastic File System (Amazon EFS)](https://aws.amazon.com/efs/).
- Điều này phải được thực hiện thủ công trong AWS Backup console.
- Backup service opt-in là một cài đặt theo AWS Region, và bạn phải lặp lại quy trình này cho tất cả các AWS Control Tower governed AWS Regions.

![anh](/images/004-ST-1631-backup-service-opt-in.png)

Cuối cùng, đặt backup administrator account làm delegated administrator để kích hoạt việc organization-wide backup task monitoring và quản lý backup policies từ tài khoản này, như minh họa trong hình dưới đây.

![anh](/images/005-ST-1631-backup-delegated-admin.png)

### Bước 5: Enable AWS Backup baseline trên các OUs cụ thể

Sau khi cấu hình xong hạ tầng core backup trước đó, bạn cần enable AWS Backup baseline trên các OUs mà bạn mong muốn. Điều hướng đến **AWS Control Tower console**, chọn **Organization** từ thanh điều hướng bên trái, chọn OU mà bạn muốn kích hoạt backup, sau đó tìm và bật tùy chọn **AWS Backup on this OU**, như minh họa trong hình dưới đây.

![anh](/images/006-ST-1631-backup-on-this-ou.png)

Thứ tự enablement là rất quan trọng, vì vậy khi enable AWS Backup baseline trên toàn tổ chức, hãy tuân theo cách tiếp cận theo hierarchy. Bắt đầu với các top-level OUs trước khi tiếp tục với các child OUs. Trong ví dụ minh họa trong hình dưới đây, bạn sẽ enable AWS Backup baseline trước trên Workloads top-level OU, sau đó tiếp tục với các child OUs như Workloads X, Workloads Y, v.v.

![anh](/images/007-ST-1631-multi-account-structure.png)

### Bước 6: Gắn tag cho các resources để backup

Bước cuối cùng trong quá trình triển khai của bạn là áp dụng tags cho các resources cần được backup.

Khi bạn enable AWS Backup thông qua AWS Control Tower, một bộ [default backup plans](https://docs.aws.amazon.com/controltower/latest/userguide/enable-backup.html#backups-on-ous) với các resource tags được định nghĩa sẵn sẽ tự động được tạo cho các bản backup theo giờ, ngày, tuần và tháng. Các kế hoạch này cung cấp mức bảo vệ hợp lý cho các kịch bản phổ biến. Tuy nhiên, hầu hết các doanh nghiệp sẽ muốn triển khai các custom backup plans dựa trên mức độ quan trọng của workload và các yêu cầu compliance bằng cách sử dụng AWS Organizations [backup policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_backup.html).

## Dọn Dẹp

Các bước sau sẽ hướng dẫn bạn dọn dẹp sau khi hoàn tất giải pháp này.

### Bước 1: Tắt backup baseline trên các OUs cụ thể

Trước khi gỡ tích hợp ở cấp landing zone, bạn nên disable backup controls tại từng OU trước.

1. Điều hướng đến **AWS Control Tower console**
2. Chọn **Organization** từ thanh điều hướng bên trái
3. Theo thứ tự ngược lại với quá trình **enablement**, bắt đầu từ các **child OU**s trước rồi tiếp tục đến các **parent OUs**:
   a. Chọn **OU**
   b. Chọn tab **Controls**
   c. Tìm và **disable** tùy chọn **AWS Backup on this OU**

### Bước 2: Gỡ tích hợp AWS Backup

Bây giờ bạn đã sẵn sàng để gỡ tích hợp AWS Backup khỏi AWS Control Tower landing zone của mình.

1. Điều hướng đến **AWS Control Tower console**
2. Chọn **Landing zone settings**
3. Chọn **Update landing zone**
4. Trong phần **backup configuration**, chọn **AWS Backup is not enabled**
5. Chọn **Next**, xem lại các thay đổi, sau đó chọn **Update landing zone** để áp dụng các thay đổi này

**AWS Control Tower** sẽ gỡ tích hợp trong khi vẫn giữ nguyên backup data hiện có của bạn, như minh họa trong hình dưới đây.

![anh](/images/008-ST-1631-disable-backups.png)

### Bước 3: Dọn dẹp các resources bổ sung

Sau khi gỡ tích hợp, bạn có thể muốn dọn dẹp thêm các resources:

- Lên lịch deletion cho AWS KMS key đã tạo
- Xóa bất kỳ [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) roles liên quan đến backup mà không còn cần thiết
- Quyết định xem có repurpose hay đóng các dedicated backup accounts
- Xóa bất kỳ recovery points và vaults nào không còn cần thiết

Theo mặc định, việc gỡ tích hợp AWS Backup sẽ không xóa các backup vaults hoặc recovery points hiện có của bạn. Điều này được thiết kế nhằm ngăn ngừa mất dữ liệu ngoài ý muốn trong quá trình thay đổi cấu hình. Tuy nhiên, điều này có nghĩa là bạn phải quản lý các resources này một cách rõ ràng nếu muốn xóa chúng.Để quản lý các backup vaults và recovery points sau khi gỡ tích hợp, điều hướng đến **AWS Backup console** trong central backup account của bạn, và chọn **Backup vaults** từ thanh điều hướng bên trái. Để xóa toàn bộ vault và tất cả các recovery points, trước tiên xóa tất cả recovery points bên trong vault, sau đó chọn vault và chọn **Delete**.

## Kết Luận

Bằng cách tích hợp AWS Backup với AWS Control Tower, bạn có thể tự động hóa và chuẩn hóa việc bảo vệ dữ liệu trên toàn bộ tổ chức mà không cần phát triển tùy chỉnh. Các tổ chức sử dụng AWS Control Tower có thể enable AWS Backup như một baseline landing zone service để tự động tạo backup vaults và triển khai policies trên nhiều accounts và AWS Regions, đảm bảo bảo vệ dữ liệu nhất quán trong khi giảm bớt khối lượng quản trị. Cách tiếp cận quản lý tập trung giúp tối ưu hóa hoạt động, trong khi các tùy chọn policies linh hoạt cho phép bạn tùy chỉnh chiến lược bảo vệ phù hợp với nhu cầu kinh doanh cụ thể. Quan trọng nhất, giải pháp này nâng cao khả năng resilience tổng thể của dữ liệu thông qua các best practices tích hợp sẵn. Khi môi trường cloud của bạn mở rộng, bạn có thể yên tâm duy trì các thực hành backup mạnh mẽ, nhất quán và phù hợp với các nguyên tắc well-architected.

#### TAGS: [AWS Backup](https://aws.amazon.com/blogs/storage/tag/aws-backup/), [AWS Cloud Storage](https://aws.amazon.com/blogs/storage/tag/aws-cloud-storage/), [AWS Control Tower](https://aws.amazon.com/blogs/storage/tag/aws-control-tower/), [AWS Key Management Service (AWS KMS)](https://aws.amazon.com/blogs/storage/tag/aws-key-management-service-aws-kms/), [data resiliency](https://aws.amazon.com/blogs/storage/tag/data-resiliency/)
