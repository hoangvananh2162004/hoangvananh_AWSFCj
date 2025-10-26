---
title: "Blog 2"
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Tăng tốc việc kiểm thử serverless với tích hợp LocalStack trong VS Code IDE.

Bởi Micah Walter | vào ngày 11 THÁNG 9, 2025 | trong [Announcements](https://aws.amazon.com/blogs/aws/category/post-types/announcements/), [AWS Lambda](https://aws.amazon.com/blogs/aws/category/compute/aws-lambda/), [AWS Serverless Application Model](https://aws.amazon.com/blogs/aws/category/compute/aws-serverless-application-model/), [Developer Tools](https://aws.amazon.com/blogs/aws/category/developer-tools/), [Launch](https://aws.amazon.com/blogs/aws/category/news/launch/), [News](https://aws.amazon.com/blogs/aws/category/news/), [Serverless](https://aws.amazon.com/blogs/aws/category/serverless/) | [Permalink](https://aws.amazon.com/blogs/aws/accelerate-serverless-testing-with-localstack-integration-in-vs-code-ide/) | [Bình luận](https://aws.amazon.com/blogs/aws/accelerate-serverless-testing-with-localstack-integration-in-vs-code-ide/#Comments)

Hôm nay, chúng tôi công bố việc tích hợp [LocalStack](https://localstack.cloud/) trong [AWS Toolkit for Visual Studio Code](https://aws.amazon.com/visualstudiocode/), giúp các developer dễ dàng hơn bao giờ hết trong việc test và debug ứng dụng serverless ngay trên môi trường local. Nâng cấp này được xây dựng dựa trên những cải tiến gần đây của chúng tôi đối với trải nghiệm phát triển [AWS Lambda](https://aws.amazon.com/blogs/compute/accelerating-local-serverless-development-with-console-to-ide-and-remote-debugging-for-aws-lambda/), bao gồm [console to IDE integration and remote debugging](https://aws.amazon.com/blogs/aws/simplify-serverless-development-with-console-to-ide-and-remote-debugging-for-aws-lambda/) được ra mắt vào tháng 7 năm 2025, tiếp tục cam kết của chúng tôi trong việc đơn giản hóa quá trình phát triển serverless trên Amazon Web Services (AWS).

Khi xây dựng ứng dụng serverless, các developer thường tập trung vào ba khía cạnh chính để tối ưu trải nghiệm testing: unit testing, integration testing, và debugging các tài nguyên đang chạy trên cloud. Mặc dù [AWS Serverless Application Model Command Line Interface (AWS SAM CLI)](https://aws.amazon.com/serverless/sam/) cung cấp khả năng local unit testing rất tốt cho từng hàm Lambda riêng lẻ, nhưng các developer làm việc với event-driven architectures liên quan đến nhiều dịch vụ AWS như [Amazon Simple Queue Service (Amazon SQS)](https://aws.amazon.com/sqs/), [Amazon EventBridge](https://aws.amazon.com/eventbridge/), và [Amazon DynamoDB](https://aws.amazon.com/eventbridge/) sẽ cần một giải pháp toàn diện hơn cho local integration testing. Mặc dù LocalStack cung cấp khả năng giả lập (emulation) các dịch vụ AWS trên local, developer phải quản lý nó như một công cụ standalone, đòi hỏi cấu hình phức tạp và thường xuyên phải chuyển đổi ngữ cảnh giữa nhiều giao diện khác nhau, làm chậm lại vòng đời phát triển.

## Tích hợp LocalStack trong AWS Toolkit cho VS Code

Để giải quyết những thách thức này, chúng tôi giới thiệu LocalStack integration để các developer có thể kết nối AWS Toolkit for VS Code trực tiếp với các endpoint của LocalStack. Với tích hợp này, developer có thể test và debug serverless applications mà không cần phải chuyển đổi qua lại giữa các công cụ hay quản lý các thiết lập LocalStack phức tạp. Developer có thể mô phỏng toàn bộ workflow event-driven end-to-end bao gồm các dịch vụ như Lambda, Amazon SQS, và EventBridge ngay tại môi trường local, mà không cần phải quản lý nhiều công cụ, thực hiện các cấu hình endpoint phức tạp, hoặc gặp phải các vấn đề về service boundary vốn trước đây đòi hỏi phải kết nối trực tiếp với cloud resources.

Lợi ích chính của tích hợp này là AWS Toolkit for VS Code giờ đây có thể kết nối đến custom endpoints như LocalStack, điều mà trước đây không thể thực hiện được. Trước đây, để trỏ AWS Toolkit for VS Code đến môi trường LocalStack, developer phải thực hiện cấu hình thủ công và liên tục chuyển đổi ngữ cảnh giữa các công cụ.

Bắt đầu với LocalStack trong VS Code rất đơn giản. Developer có thể bắt đầu với LocalStack Free version, phiên bản cung cấp local emulation cho các dịch vụ AWS cốt lõi, lý tưởng cho giai đoạn phát triển và testing ban đầu. Sử dụng guided application walkthrough trong VS Code, developer có thể cài đặt LocalStack trực tiếp từ giao diện của toolkit, thao tác này sẽ tự động cài đặt extension LocalStack và hướng dẫn họ trong suốt quá trình setup. Khi đã được cấu hình, developer có thể deploy các serverless applications trực tiếp vào môi trường emulated và test functions của họ tại local, tất cả mà không cần rời khỏi IDE.

## Hãy thử nghiệm

Đầu tiên, tôi sẽ cập nhật bản AWS Toolkit for VS Code của mình lên phiên bản mới nhất. Sau khi làm điều này, tôi có thể thấy một tùy chọn mới khi vào **Application Builder** và nhấp vào **Walkthrough** **of Application Builder**. Tùy chọn này cho phép tôi cài đặt LocalStack chỉ với một lần nhấp.

![anh](/static/images/image-01.png)

Sau khi tôi hoàn tất quá trình setup cho LocalStack, tôi có thể khởi chạy nó từ status bar và sau đó chọn LocalStack từ danh sách các [configured AWS profiles](https://docs.aws.amazon.com/cli/v1/userguide/cli-chap-configure.html). Trong ví dụ minh họa này, tôi đang sử dụng Application Composer để xây dựng một serverless architecture đơn giản với [Amazon API Gateway](https://aws.amazon.com/api-gateway/), Lambda, và DynamoDB. Thông thường, tôi sẽ deploy nó lên AWS bằng cách sử dụng AWS SAM. Trong trường hợp này, tôi sẽ dùng cùng một lệnh AWS SAM để deploy stack của mình trong môi trường local.

![anh](/static/images/image-02.png)

Tôi chỉ cần chạy lệnh `sam deploy –guided –profile localstack` từ command line và làm theo các bước hướng dẫn như bình thường. Deploying lên LocalStack bằng AWS SAM CLI mang lại trải nghiệm hoàn toàn giống với khi tôi deploy lên AWS. Trong ảnh chụp màn hình bên dưới, tôi có thể thấy standard output từ AWS SAM, cũng như các tài nguyên LocalStack mới của tôi được liệt kê trong AWS Toolkit Explorer.

![anh](/static/images/image-03.png)

Tôi thậm chí có thể truy cập vào một Lambda function và chỉnh sửa function code mà tôi đã deploy cục bộ!

![anh](/static/images/image-04.png)

Trên trang web LocalStack, tôi có thể đăng nhập và xem tất cả các resources mà tôi đang chạy cục bộ. Trong ảnh chụp màn hình bên dưới, bạn có thể thấy DynamoDB table cục bộ mà tôi vừa deploy.

![anh](/static/images/image-05.png)

## Quy trình phát triển được nâng cao

Những khả năng mới này bổ sung cho các tính năng console-to-IDE integration và remote debugging mà chúng tôi vừa ra mắt gần đây, tạo nên một trải nghiệm phát triển toàn diện đáp ứng nhiều nhu cầu kiểm thử khác nhau trong suốt vòng đời phát triển. AWS SAM CLI cung cấp khả năng testing cục bộ tuyệt vời cho từng Lambda function, xử lý hiệu quả các kịch bản unit testing. Đối với integration testing, LocalStack integration cho phép kiểm thử các workflow đa dịch vụ cục bộ mà không cần phải đối mặt với sự phức tạp của IAM permissions, cấu hình Amazon VPC, hoặc các vấn đề về service boundary vốn có thể làm chậm tốc độ phát triển.

Khi developers cần test với các AWS services trong môi trường phát triển, họ có thể sử dụng tính năng remote debugging, tính năng này cung cấp quyền truy cập đầy đủ vào Amazon VPC resources và IAM roles. Cách tiếp cận theo tầng này giúp developers tập trung vào business logic trong các giai đoạn phát triển ban đầu bằng LocalStack, sau đó chuyển đổi liền mạch sang kiểm thử dựa trên cloud khi cần validate hành vi và cấu hình của AWS services. Sự tích hợp này loại bỏ nhu cầu phải chuyển đổi giữa nhiều công cụ và môi trường, giúp developers xác định và khắc phục vấn đề nhanh hơn, đồng thời vẫn duy trì tính linh hoạt trong việc lựa chọn phương pháp testing phù hợp với nhu cầu cụ thể.

## Đã sẵn sàng để sử dụng

Bạn có thể bắt đầu sử dụng các tính năng mới này thông qua AWS Toolkit for VS Code bằng cách cập nhật lên phiên bản v3.74.0. LocalStack integration hiện đã khả dụng tại tất cả các commercial [AWS Regions](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) ngoại trừ [AWS GovCloud (US)](https://aws.amazon.com/govcloud-us/) Regions. Để tìm hiểu thêm, hãy truy cập tài liệu của [AWS Toolkit for VS Code](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/lambda-localstack.html) và [Lambda](https://docs.aws.amazon.com/lambda/latest/dg/) documentation.

Đối với các developer cần phạm vi dịch vụ rộng hơn hoặc các khả năng nâng cao, LocalStack cung cấp thêm các tier với nhiều tính năng mở rộng. Không có chi phí bổ sung từ AWS khi sử dụng integration này.

Những cải tiến này đánh dấu một bước tiến quan trọng khác trong cam kết liên tục của chúng tôi nhằm đơn giản hóa trải nghiệm phát triển serverless. Trong năm qua, chúng tôi đã tập trung biến VS Code trở thành công cụ lựa chọn hàng đầu cho các serverless developer, và LocalStack integration tiếp tục hành trình đó bằng cách cung cấp các công cụ giúp developer xây dựng và kiểm thử ứng dụng serverless hiệu quả hơn bao giờ hết.
