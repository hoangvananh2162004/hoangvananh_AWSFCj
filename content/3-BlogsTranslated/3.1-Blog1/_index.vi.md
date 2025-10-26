---
title: "Blog 1"
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Mở rộng deployment pipeline với Amazon ECS blue/green deployments và lifecycle hooks.

Bởi Olly Pomeroy | vào ngày 11 THÁNG 9, 2025 | trong [Amazon Elastic Container Service](https://aws.amazon.com/blogs/containers/category/compute/amazon-elastic-container-service/), [Containers](https://aws.amazon.com/blogs/containers/category/containers/) | [Permalink](https://aws.amazon.com/blogs/containers/extending-deployment-pipelines-with-amazon-ecs-blue-green-deployments-and-lifecycle-hooks/)

Chiến lược [blue/green deployment](https://docs.aws.amazon.com/whitepapers/latest/blue-green-deployments/welcome.html) cho phép bạn release ứng dụng bằng cách chuyển hướng traffic giữa hai môi trường giống hệt nhau đang chạy các phiên bản khác nhau của ứng dụng. Điều này giúp bạn giảm thiểu các rủi ro thường gặp khi deploy phần mềm, bởi vì bạn có thể rollback về một deployment trước đó ngay lập tức. Amazon Elastic Container Service ([Amazon ECS](https://aws.amazon.com/ecs/)) gần đây đã công bố hỗ trợ native cho blue/green deployments, loại bỏ nhu cầu quản lý và tích hợp với các công cụ deployment khác. Như một phần của lần phát hành này, Amazon ECS cũng đã giới thiệu lifecycle hooks cho deployments. Lifecycle hooks cho phép bạn tích hợp test suites, manual approvals, và metrics vào deployment pipeline của mình. Trong bài viết này, chúng tôi sẽ đi sâu vào lifecycle hooks và chỉ ra cách chúng có thể được tích hợp vào deployment workflows.

## Bối Cảnh

Khi một Amazon ECS [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html) được tạo hoặc cập nhật, chúng tôi sẽ tạo ra một immutable object chứa chính xác specification của nó, được gọi là một Amazon ECS [service revision](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-revision.html). Sau đó, control plane sẽ cố gắng deploy service revision này thông qua một Amazon ECS [service deployment](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-deployment.html). Một deployment sẽ trải qua [multiple lifecycle states](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-deployment.html#service-deployments-states), chẳng hạn như `IN_PROGRESS` và `SUCCESSFUL`. Trong lần phát hành blue/green gần đây, có một subcategory mới nằm bên dưới các states này, được gọi là lifecycle stages. Khi một deployment bắt đầu, chúng tôi sẽ xử lý tuần tự qua từng lifecycle stage, với tùy chọn mở rộng deployment bằng custom logic được đóng gói trong lifecycle hooks. Hình minh họa sau đây cho thấy mối quan hệ giữa lifecycle states, lifecycle stages, và lifecycle hooks.

![anh](/static/images/C45-1.png)

Lifecycle hooks là các synchronous [AWS Lambda](https://aws.amazon.com/lambda/) functions mà Amazon ECS control plane sẽ invoke thay cho bạn. Bạn có thể viết bất kỳ logic nào bạn muốn trong các function này bằng bất kỳ runtime language nào bạn chọn. Khi function đã thực thi xong logic của nó, nó phải return một `hookStatus` để Amazon ECS deployment có thể tiếp tục. Nếu `hookStatus` không được return, hoặc nếu function bị lỗi, thì Amazon ECS deployment sẽ rollback.

Các giá trị của hookStatus có thể là:

- `SUCCEEDED`: deployment tiếp tục sang lifecycle stage tiếp theo.
- `FAILED`: deployment sẽ rollback về deployment cuối cùng đã thành công.
- `IN_PROGRESS`: Amazon ECS control plane sẽ invoke lại function sau một khoảng thời gian ngắn. Mặc định là 30 giây, nhưng có thể điều chỉnh bằng cách return một `callBackDelay`.

![anh](/static/images/C45-2.png)

#### IN_PROGRESS hook status

`IN_PROGRESS` status cung cấp một cơ chế mạnh mẽ và linh hoạt để giám sát và kiểm soát Amazon ECS deployment. Function này là một synchronous event, do đó deployment chỉ có thể tạm dừng trong thời gian thực thi tối đa của một function (15 phút). Có thể có những tình huống bạn cần thực thi complex logic hoặc tạm dừng để chờ thêm dữ liệu, chẳng hạn như chờ một full test suite chạy trên test endpoint. Phản hồi `hookStatus` này cho phép bạn đạt được điều đó. Khi `IN_PROGRESS` hook status được return, Amazon ECS control plane sẽ invoke lại function sau 30 giây (hoặc khoảng thời gian được return trong key `callBackDelay`). Amazon ECS sẽ tiếp tục invoke function cho đến khi một `SUCCEEDED` hoặc `FAILED` hook status được return; do đó cho phép thực thi validation logic cần nhiều thời gian hơn so với thời gian thực thi của một function duy nhất.

#### Truyền trạng thái với hookDetails

Lifecycle hooks thường được định nghĩa một lần và tái sử dụng trên nhiều ECS services trong một tổ chức. Để làm cho lifecycle hooks độc lập với ECS service, bạn có thể sử dụng `hookDetails` dictionary để truyền vào dữ liệu cụ thể của service khi tạo hoặc cập nhật một service. Dưới đây là một trích đoạn từ một [ECS service](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_CreateService.html) nơi chúng tôi đã định nghĩa một lifecycle hook và truyền vào một [Amazon S3 Bucket](https://aws.amazon.com/s3/).

```yaml
{
  "hookTargetArn": "arn:aws:lambda:<region>:<account_id>:function:<function_name>",
  "roleArn": "arn:aws:iam::<account_id>:role/<iam_role_name>",
  "lifecycleStages": ["POST_TEST_TRAFFIC_SHIFT"],
  "hookDetails": { "S3_BUCKET_NAME": "mys3bucket" },
}
```

Ngoài ra, bạn cũng có thể sử dụng `hookDetails` dictionary để truyền state giữa các lần invocation của cùng một lifecycle hook. Nếu một hook return `IN_PROGRESS` status, bạn cũng có thể bao gồm `hookDetails` dictionary với nhiều cặp key/value, loại bỏ nhu cầu phải lưu state ở nơi khác. Các trường hợp sử dụng phổ biến bao gồm truyền metric counters hoặc [Amazon Resource Names (ARNs)](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html) của các external resources giữa các lần invocation. Lưu ý rằng dữ liệu được thêm vào `hookDetails` theo cách này chỉ tồn tại giữa các lần invocation của cùng một hook trong một deployment. Dữ liệu sẽ không được mang sang giữa các hook khác nhau trong cùng một deployment, hoặc cùng một hook ở các deployment khác nhau.

```yaml
{
  "hookStatus": " IN_PROGRESS",
  "callBackDelay": 90,
  "hookDetails":
    {
      "SFN_EXECUTION_ARN": "arn:aws:states:<region>:<account_id>:execution:<sfn>:<id>",
    },
}
```

#### PRE_SCALE_UP hook

`PRE_SCALE_UP` lifecycle hook là duy nhất trong Amazon ECS deployments vì nó là hook duy nhất có sẵn cho cả [blue green deployment strategy](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-bluegreen.html) mới cũng như [rolling update strategy](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-ecs.html) hiện tại. `PRE_SCALE_UP` hook chạy trước khi các new tasks được tạo, cho phép bạn áp dụng admission logic trong Amazon ECS. Một cách sử dụng phổ biến của hook này là đánh giá Amazon ECS task definition dựa trên governance policy trước khi các tasks được scheduled. Trước đây, nếu bạn muốn áp dụng một policy cho các tasks, bạn chỉ có thể phản hồi khi task đã được scheduled. Một ví dụ về mô hình trước đó có thể được tìm thấy trong [GitHub repository](https://github.com/aws-samples/lambda-based-signature-verification). Mô hình trước đây có những hạn chế khi các tasks không an toàn hoặc có lỗ hổng có thể đang chạy trước khi policy của bạn phát hiện ra vi phạm. Hook mới này cho phép các trường hợp sử dụng như xác minh container image signature hoặc container image repository trước khi workload được scheduled.

## Walkthrough

Trong walkthrough này, chúng tôi sẽ trình bày các patterns khác nhau để minh họa sức mạnh của lifecycle hooks. Trước tiên, chúng tôi sẽ trình diễn cách một `PRE_SCALE_UP` hook có thể được sử dụng như một dạng admission hook, tiếp theo, chúng tôi sẽ chỉ ra cách một `POST_TEST_TRAFFIC_SHIFT `hook có thể được sử dụng để áp dụng bước manual approval trước khi production traffic được chuyển đổi.

#### Điều kiện tiên quyết

Để triển khai walkthrough này, bạn cần quyền truy cập vào một AWS account với một workstation đã cài đặt AWS Command Line Interface [(AWS CLI)](https://aws.amazon.com/cli/), AWS Cloud Development Kit [(AWS CDK)](https://aws.amazon.com/cdk/), và Docker Engine.

Trong suốt walkthrough này, chúng tôi sẽ deploy các resources lên một Amazon ECS cluster và vào một Amazon Virtual Private Cloud [(Amazon VPC)](https://aws.amazon.com/vpc/). Code cho walkthrough này có thể được tìm thấy trong một AWS CDK app trong sample [GitHub repository](https://github.com/aws-samples/sample-amazon-ecs-blue-green-deployment-patterns#) của chúng tôi.

1. Clone GitHub Repository về máy

```yaml
$ git clone https://github.com/aws-samples/sample-amazon-ecs-blue-green-deployment-patterns.git
```

2. Triển khai AWS CDK app. AWS CDK app này chứa ba [AWS CloudFormation stacks](https://aws.amazon.com/cloudformation/) riêng biệt. Nó tạo một Amazon VPC và [Application Load Balancer (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) trong stack đầu tiên. Trong stack thứ hai, nó tạo một ECS cluster và các AWS Identity and Access Management ([IAM](https://aws.amazon.com/iam/)) roles cần thiết. Thứ ba, nó tạo một stack chứa các functions của chúng ta.

```yaml
$ cd sample-amazon-ecs-blue-green-deployment-patterns/ecs-bluegreen-lifecycle-hooks
$ npm install
$ npm run build
$ npx cdk deploy --all
```

You can verify that the resources were created correctly with the following:

```yaml
$ aws cloudformation list-stacks \
--query "StackSummaries[?starts_with(StackName, 'Ecs')].[StackName, StackStatus]" \
--output table
```

3. Trong kho lưu trữ này, chúng tôi có một template task definition và một service definition. Chúng tôi đã tạo một populate templates bash script trong repository để điền các template này với các giá trị phù hợp với môi trường của bạn.

```yaml
$ ./populate_templates.sh
```

#### Admission hook demonstration

Để minh họa cách một `PRE_SCALE_UP` hook có thể được sử dụng như một admission hook, chúng tôi sẽ khởi chạy một Amazon ECS service sử dụng container image từ một Amazon Elastic Container Registry ([Amazon ECR](https://aws.amazon.com/ecr/)) public repository. Trong tình huống của chúng tôi, chúng tôi đã viết một policy trong function để đảm bảo rằng tất cả container images đều đến từ một private hoặc public Amazon ECR repository. Chúng tôi có thể thấy policy này được thực thi trong logs của lifecycle hook.

1. Đầu tiên, chúng ta xem xét các resources mà chúng ta dự định deploy:
   Bên trong service definition của chúng ta, bạn sẽ thấy một `PRE_SCALE_UP lifecycle` hook.

```yaml
$ cat outputs/service.json | jq -r '.deploymentConfiguration.lifecycleHooks[0]'
{
  "hookTargetArn": "arn:aws:lambda:us-east-1:111222333444:function:EcsBluegreenHookStack-admissionFunction1D626CB0-FepCuhdYPhNn",
  "roleArn": "arn:aws:iam::111222333444:role/EcsBluegreenEcsStack-ECSLambdaInvokeRole2A82A552-yEE1nb8PewjH",
  "lifecycleStages": [
    "PRE_SCALE_UP"
  ]
}

```

Function cho hook này xác minh container image của chúng ta bằng cách sử dụng một regex expression. Bạn có thể xem đoạn code này trên máy local bằng cách in source code ra.

```yaml
$ grep -A 20 "def validate_container_images" src/admissionFunction/app.py
def validate_container_images(container_image_list):
    # Regex patterns to match Amazon ECR repository URLs
    # Format for private ECR: [account-id].dkr.ecr.[region].amazonaws.com/[repository-name]:[tag]
    # Format for public ECR: public.ecr.aws/aws-containers/[repository-name]:[tag]
    private_ecr_pattern = r"^(\d+)\.dkr\.ecr\.([a-z0-9-]+)\.amazonaws\.com/.*"
    public_ecr_pattern = r"^public\.ecr\.aws/aws-containers/.*"

    valid_images = True
    for container_image in container_image_list:#
        image_url = container_image["image"]
        logger.info(f"Validating image: {image_url}")

        if re.match(private_ecr_pattern, image_url) or re.match(
            public_ecr_pattern, image_url
        ):
            logger.info(f"Image {image_url} is from an Amazon ECR repository")
        else:
            logger.warning(f"Image {image_url} is NOT from an Amazon ECR repository")
            valid_images = False

    return valid_images
```

Cuối cùng, bạn có thể thấy container image mà chúng ta đang cố gắng deploy trong task definition của mình.

```yaml
$ cat outputs/taskdef.json | jq -r '.containerDefinitions[0].image'
public.ecr.aws/aws-containers/retail-store-sample-ui:1.1.0
```

2. Chúng tôi tạo **Amazon ECS service** và quan sát **lifecycle hook** hoạt động.

```yaml
aws ecs \
register-task-definition \
--cli-input-json file://outputs/taskdef.json
aws ecs \
create-service \
--cli-input-json file://outputs/service.json
```

3. Trong Amazon ECS [console](https://us-west-2.console.aws.amazon.com/ecs/home), bạn có thể theo dõi deployment này. **Current Deployment stage** trong **Deployment Tab** cho chúng ta biết deployment đang ở lifecycle stage nào. Hãy nhớ rằng lifecycle hook đầu tiên của chúng ta chạy ở `PRE_SCALE_UP` stage.

![anh](/static/images/C45-3.png)

4. Bạn có thể xác minh rằng **lifecycle hook** đã thực thi thành công bằng cách truy cập [Amazon CloudWatch Logs console](https://aws.amazon.com/cloudwatch/) và chọn **Log Group** có tên bắt đầu bằng `/aws/lambda/EcsBlugreenHookStack-admissionFunction`.

![anh](/static/images/C45-4.png)

Thành công! Container images được sử dụng trong task definition của chúng ta đã được xác minh theo policy trước khi task được scheduled.

#### Manual approval demonstration

Phần thứ hai của walkthrough tập trung vào bước manual approval ở `POST_TEST_TRAFFIC_SHIFT` stage. Đây là một stage quan trọng trong deployment pipeline, vì nó diễn ra sau khi test traffic endpoint đã được chuyển sang các green tasks của chúng ta (cho phép bạn chạy test suite), nhưng trước khi production traffic endpoint được di chuyển.
Trong walkthrough này, chúng tôi thực hiện một bước manual approval trước khi di chuyển production traffic. Có nhiều cách để thực hiện bước manual approval, nhưng trong minh họa này, chúng tôi sử dụng một file và một [Amazon S3](https://aws.amazon.com/s3/) bucket. Function trong lifecycle hook của chúng tôi sẽ cố gắng tìm một file trong S3 bucket có tên trùng với Amazon ECS service revision. Nếu file tồn tại, deployment pipeline sẽ tiếp tục. Nếu file không tồn tại, hook sẽ return trạng thái `IN_PROGRESS` và chúng tôi sẽ thử lại sau 30 giây.

1. Đầu tiên, chúng ta xem xét các resources mà chúng ta dự định deploy.
   Bên trong service definition của chúng ta, bạn sẽ thấy một `POST_TEST_TRAFFIC_SHIFT` lifecycle hook.

```yaml
$ cat outputs/service.json | jq -r '.deploymentConfiguration.lifecycleHooks[1]'
{
  "hookTargetArn": "arn:aws:lambda:us-east-1:111222333444:function:EcsBluegreenHookStack-approvalFunctionC7093965-J3YoEQtxR679",
  "roleArn": "arn:aws:iam::111222333444:role/EcsBluegreenEcsStack-ECSLambdaInvokeRole2A82A552-9OcKcmXkLe9c",
  "lifecycleStages": [
    "POST_TEST_TRAFFIC_SHIFT"
  ]
}
```

Xem source code của function trên máy local cho phép chúng ta kiểm tra logic để tìm file Amazon S3. Chúng tôi đang sử dụng [boto3client](https://aws.amazon.com/sdk-for-python/) để cố gắng tìm head object của một file có tên trùng với Amazon ECS service revision ID của chúng ta.

```yaml
$ grep -A 20 "def check_s3_file" src/approvalFunction/app.py
def check_s3_file(s3_bucket, revision_arn):

    # Extract the revision from the ARN (everything after the last '/')
    revision = revision_arn.split("/")[-1]
    file_name = f"{revision}.txt"
    logger.info(f"Checking if file {file_name} exists in bucket {s3_bucket}")
    # Create S3 client
    s3_client = boto3.client("s3")
    try:
        # Use head_object to check if the file exists
        s3_client.head_object(Bucket=s3_bucket, Key=file_name)
        logger.info(f"File {file_name} exists in bucket {s3_bucket}")
        return True
    except ClientError as e:
        if e.response["Error"]["Code"] == "404":
            # The file does not exist
            logger.info(f"File {file_name} does not exist in bucket {s3_bucket}")
            return False

```

2. Để kích hoạt một deployment, chúng ta cập nhật Amazon ECS service với một force action. Điều này buộc Amazon ECS tạo một service revision mới và redeploy tất cả các tasks, ngay cả khi service specification không thay đổi.

```yaml
export CLUSTER_NAME=$(cat outputs/service.json | jq -r '.cluster')
export SERVICE_NAME=$(cat outputs/service.json | jq -r '.serviceName')

aws ecs \
update-service \
--service $SERVICE_NAME \
--cluster $CLUSTER_NAME \
--force
```

3. Bạn có thể theo dõi deployment đang diễn ra trong Amazon [ECS console](https://us-west-2.console.aws.amazon.com/ecs/home), quan sát service di chuyển qua các lifecycle stages khác nhau. Khi deployment đạt đến `POST_TEST_TRAFFIC_SHIFT` stage, chúng ta có thể kiểm tra tiến trình của hook.

![anh](/static/images/C45-5.png)

4. CloudWatch Logs console cung cấp cho chúng ta cái nhìn về những gì đang diễn ra ở stage này. Trong một log group có tên bắt đầu bằng `/aws/lambda/EcsBlugreenHookStack-approvalFunction`, chúng ta sẽ thấy các invocation của function mỗi 30 giây, cố gắng truy xuất file từ Amazon S3.

![anh](/static/images/C45-6.png)

5. Để cho phép deployment pipeline tiếp tục, chúng ta tải một file lên S3 bucket. Trước tiên, chúng ta cần lấy Amazon ECS service revision ARN, rút gọn chỉ còn service revision ID, và sử dụng nó làm tên file.

```yaml
export S3_BUCKET_NAME=$(aws cloudformation describe-stacks \
--stack-name EcsBluegreenHookStack \
--query 'Stacks[0].Outputs[?OutputKey==`ApprovalBucketName`].OutputValue' \
--output text)
export ECS_SERVICE_REVISION=$(aws ecs \
list-service-deployments \
--service $SERVICE_NAME \
--cluster $CLUSTER_NAME \
--query 'serviceDeployments[?status==`IN_PROGRESS`].targetServiceRevisionArn' \
--output text)

SERVICE_REVISION_ID=$(echo "$ECS_SERVICE_REVISION" | awk -F/ '{print $NF}')
touch $SERVICE_REVISION_ID.txt

aws s3 cp $SERVICE_REVISION_ID.txt s3://$S3_BUCKET_NAME/$SERVICE_REVISION_ID.txt
```

6. Bây giờ khi file đã tồn tại trong Amazon S3, chúng ta có thể quay lại Amazon ECS console và quan sát deployment hoàn tất.

![anh](/static/images/C45-7.png)

## Dọn dẹp

Để dọn dẹp các AWS resources đã sử dụng trong suốt walkthrough này, chúng ta phải xóa Amazon ECS service.

```yaml
$ aws ecs \
delete-service \
--service $SERVICE_NAME \
--cluster $CLUSTER_NAME \
--force
```

Tiếp theo, chúng ta xóa các **CloudFormation** stacks bằng **AWS CDK**.

```yaml
$ cdk destroy --all
```

## Kết luận

Lifecycle hooks trong Amazon ECS cung cấp vô vàn khả năng để tùy chỉnh deployment pipeline của bạn. Trong bài viết này, chúng tôi đã đề cập đến một vài trường hợp sử dụng phổ biến của các hook này, chẳng hạn như đánh giá một deployment theo governance policies, cũng như thực hiện một bước manual approval. Để tìm hiểu thêm, hãy truy cập Amazon ECS lifecycle hooks [documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-bluegreen.html) và xem các minh họa trên kênh Containers from the Couch trên [YouTube](https://www.youtube.com/ContainersFromTheCouch).
