---
title: "Blog 1"
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Extending deployment pipelines with Amazon ECS blue green deployments and lifecycle hooks.

by Olly Pomeroy | on 11 SEP 2025 | in [Amazon Elastic Container Service](https://aws.amazon.com/blogs/containers/category/compute/amazon-elastic-container-service/), [Containers](https://aws.amazon.com/blogs/containers/category/containers/) | [Permalink](https://aws.amazon.com/blogs/containers/extending-deployment-pipelines-with-amazon-ecs-blue-green-deployments-and-lifecycle-hooks/)

The [blue/green deployment](https://docs.aws.amazon.com/whitepapers/latest/blue-green-deployments/welcome.html) enables you to release applications by shifting traffic between two identical environments running different versions of the application. This allows you to mitigate common risks associated with deploying software, because you can roll back to a previous deployment instantly. Amazon Elastic Container Service ([Amazon ECS](https://aws.amazon.com/ecs/)) recently announced native support for blue/green deployments, removing the need to manage and integrate with other deployment tools. As part of this release, Amazon ECS has also introduced lifecycle hooks for deployments. Lifecycle hooks allow you to integrate test suites, manual approvals, and metrics into your deployment pipeline. In this post we dive into lifecycle hooks and show how they can be integrated into deployment workflows.

## Background

When an Amazon ECS [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html) is created or updated, we create an immutable object of its exact specification, known as an Amazon ECS [service revision](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-revision.html). Then, the control plane attempts to deploy this service revision through an Amazon ECS [service deployment](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-deployment.html). A deployment goes through [multiple lifecycle states](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-deployment.html#service-deployments-states), such as IN_PROGRESS and SUCCESSFUL. As part of the recent blue/green release, there is a new subcategory underneath these states, known as lifecycle stages. When a deployment has started, we work through each lifecycle stage linearly, with the option to extend the deployment with custom logic packaged in lifecycle hooks. The following figure shows the relationship between lifecycle states, lifecycle stages, and lifecycle hooks.

![anh](/images/C45-1.png)

Lifecycle hooks are synchronous [AWS Lambda](https://aws.amazon.com/lambda/) functions that the Amazon ECS control plane invokes on your behalf. You can write any logic you’d like in these functions in any runtime language you choose. When the function has finished executing its logic, it must return a hookStatus for the Amazon ECS deployment to continue. If a hookStatus is not returned, or if the function fails, then the Amazon ECS deployment rolls back.

The values of the hookStatus can either be:

- `SUCCEEDED`: the deployment continues to the next lifecycle stage.
- `FAILED`: the deployment rolls back to the last successful deployment.
- `IN_PROGRESS`: the Amazon ECS control plane invokes the function again after a short period of time. By default, this is 30 seconds, but this can be tuned by returning a `callBackDelay`.

![anh](/images/C45-2.png)

#### IN_PROGRESS hook status

The `IN_PROGRESS` status provides a powerful and flexible mechanism to monitor and control the Amazon ECS deployment. The function is a synchronous event, thus the deployment can only pause for the maximum execution time of a function (15 minutes). There may be scenarios where you need to execute complex logic or pause for more data, such as waiting for a full test suite to run against the test endpoint. This `hookStatus` response allows you to achieve this. When the `IN_PROGRESS` hook status is returned, the Amazon ECS control plane will invoke the function again in 30 seconds (or the time returned in the `callBackDelay` key). Amazon ECS will continue to invoke the function until a `SUCCEEDED` or `FAILED` hook status is returned; therefore, allowing for validation logic that needs more time than the execution time of a single function.

#### Passing state with hookDetails

Lifecycle hooks are often defined once and reused across multiple ECS services within an organization. To make lifecycle hooks ECS service agnostic, you can use the `hookDetails` dictionary to pass in service specific data when creating or update a service. Below is an extract from a [ECS service](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_CreateService.html) where we have defined a lifecycle hook and passed in an [Amazon S3 Bucket](https://aws.amazon.com/s3/) name.

```yaml
{
  "hookTargetArn": "arn:aws:lambda:<region>:<account_id>:function:<function_name>",
  "roleArn": "arn:aws:iam::<account_id>:role/<iam_role_name>",
  "lifecycleStages": ["POST_TEST_TRAFFIC_SHIFT"],
  "hookDetails": { "S3_BUCKET_NAME": "mys3bucket" },
}
```

Furthermore, you can also use the `hookDetails` dictionary to pass state between invocations of the same lifecycle hook. If a hook returns the `IN_PROGRESS` status, you can also include the `hookDetails` dictionary with multiple key / value pairs, removing the need to store state elsewhere. Common use cases include passing metric counters or [Amazon Resource Names (ARNs)](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html) of external resources between invocations. Note data added to `hookDetails` in this way only persists between invocations of the same hook within a single deployment. Data doesn’t carry over between different hooks within the same deployment or the same hook in different deployments.

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

The `PRE_SCALE_UP` lifecycle hook is unique in Amazon ECS deployments as it is the only hook available for both the new [blue green deployment strategy](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-bluegreen.html) as well as the existing [rolling update strategy](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-ecs.html). The `PRE_SCALE_UP` hook runs before new tasks are created, allowing you to apply admission logic in Amazon ECS. A common use of this hook is to evaluate the Amazon ECS task definition against a governance policy before the tasks have been scheduled. Previously, if you wanted to enforce a policy to tasks, then you could only respond to a task once it has been scheduled. An example of the previous pattern can be found in this [GitHub repository](https://github.com/aws-samples/lambda-based-signature-verification). This previous pattern had limitations where insecure or vulnerable tasks could be running before your policy detected an infringement. This new hook enables use cases such as verifying a container image signature or a container image repository before the workload has been scheduled.

## Walkthrough

In this walkthrough we are going to show different patterns to demonstrate the power of lifecycle hooks. We will first showcase how a `PRE_SCALE_UP` hook can be used as form of admission hook, secondly, we will show a `POST_TEST_TRAFFIC_SHIFT` hook can be used to apply a manual approval step before production traffic is shifted.

#### Prerequisites

To deploy this walk through you need access to an AWS account with a workstation with the AWS Command Line Interface [(AWS CLI)](https://aws.amazon.com/cli/), AWS Cloud Development Kit [(AWS CDK)](https://aws.amazon.com/cdk/), and Docker Engine installed.

Throughout this walkthrough we deploy resources on to an Amazon ECS cluster and into an Amazon Virtual Private Cloud [(Amazon VPC)](https://aws.amazon.com/vpc/). The code for this walk through can be found in an AWS CDK app within our sample [GitHub repository](https://github.com/aws-samples/sample-amazon-ecs-blue-green-deployment-patterns#).

1. Clone down the GitHub Repository

```yaml
$ git clone https://github.com/aws-samples/sample-amazon-ecs-blue-green-deployment-patterns.git
```

2. Deploy the AWS CDK app. This AWS CDK app contains three separate [AWS CloudFormation stacks](https://aws.amazon.com/cloudformation/). It creates an Amazon VPC and [Application Load Balancer (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) in the first stack. In the second, it creates an ECS cluster and necessary AWS Identity and Access Management ([IAM](https://aws.amazon.com/iam/)) roles. Third, it creates a stack containing our functions.

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

3. Within this repository we have a template task definition and service definition. We have created a populate templates bash script within the repository to populate these templates with the values relevant to your environment.

```yaml
$ ./populate_templates.sh
```

#### Admission hook demonstration

To demonstrate how a `PRE_SCALE_UP` hook can be used as an admission hook, we are going to start an Amazon ECS service using a container image from an Amazon Elastic Container Registry ([Amazon ECR](https://aws.amazon.com/ecr/)) public repository. In our scenario, we have written a policy in our function to make sure that all container images come from either a private or public Amazon ECR repository. We can see this policy being enforced in the logs of our lifecycle hook.

1. First, we look at the resources we plan to deploy:
   Inside of our service definition you should find a `PRE_SCALE_UP` lifecycle hook.

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

The function for this hook verifies our container image using a regex expression. You can see that piece of code locally by printing the source code.

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

Finally, you can see what container image we are attempting to deploy within our task definition.

```yaml
$ cat outputs/taskdef.json | jq -r '.containerDefinitions[0].image'
public.ecr.aws/aws-containers/retail-store-sample-ui:1.1.0
```

2. We create the Amazon ECS service and see the lifecycle hook in action.

```yaml
aws ecs \
register-task-definition \
--cli-input-json file://outputs/taskdef.json
aws ecs \
create-service \
--cli-input-json file://outputs/service.json
```

3. In the Amazon [console](https://us-west-2.console.aws.amazon.com/ecs/home) you can monitor this deployment. The** Current Deployment stage** within the **Deployment tab** shows us which stage of the lifecycle the deployment is in. Remember our first lifecycle hook runs at the `PRE_SCALE_UP` stage.

![anh](/images/C45-3.png)

4. You can verify that the lifecycle hook executed successfully by browsing to the [Amazon CloudWatch Logs console](https://aws.amazon.com/cloudwatch/), and choosing the Log Group starting with the name `/aws/lambda/EcsBlugreenHookStack-admissionFunction`.

![anh](/images/C45-4.png)

Success! The container images used within our task definition were verified against our policy before the task was scheduled.

#### Manual approval demonstration

The second part of our walkthrough focuses on a manual approval step at the `POST_TEST_TRAFFIC_SHIFT` stage. This is a critical stage in our deployment pipeline, because it’s after the test traffic endpoint has been shifted to our green tasks (allowing you to a run a test suite), but before the production traffic endpoint has been moved.

In this walkthrough we implement a manual approval step before migrating the production traffic. There are lots of ways to implement a manual approval step, but in this demonstration, we use a file and an [Amazon S3](https://aws.amazon.com/s3/) bucket. The function in our lifecycle hook attempts to find a file in an S3 bucket named after the Amazon ECS service revision. If the file exists, then the deployment pipeline continues. If the file does not exist, then the hook returns an `IN_PROGRESS` state and we try again in 30 seconds.

1. First, we look at the resources that we plan to deploy.
   Inside of our service definition we should find a `POST_TEST_TRAFFIC_SHIFT` lifecycle hook.

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

Browsing the function source code locally allows us to look at the logic to find the Amazon S3 file. We are using the [boto3client](https://aws.amazon.com/sdk-for-python/) to attempt to find the head object of a file named after our Amazon ECS service revision ID.

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

2. To trigger a deployment, we update our Amazon ECS service with a force action. This forces Amazon ECS to create a new service revision and redeploy all the tasks, even though the service specification hasn’t changed.

```yaml
export CLUSTER_NAME=$(cat outputs/service.json | jq -r '.cluster')
export SERVICE_NAME=$(cat outputs/service.json | jq -r '.serviceName')

aws ecs \
update-service \
--service $SERVICE_NAME \
--cluster $CLUSTER_NAME \
--force
```

3. You can monitor the ongoing deployment in the Amazon [ECS console](https://us-west-2.console.aws.amazon.com/ecs/home), watching the service move through the various lifecycle stages. When the deployment reaches the `POST_TEST_TRAFFIC_SHIFT` stage, we can check the progress of our hook.

![anh](/images/C45-5.png)

4. The CloudWatch Logs console gives us insight into what is happening at this stage. In a log group starting with `/aws/lambda/EcsBlugreenHookStack-approvalFunction`, we should see invocations of our function every 30 seconds attempting to retrieving the file from Amazon S3.

![anh](/images/C45-6.png)

5. To allow our deployment pipeline to continue, we upload a file to the S3 bucket. First, we need to retrieve the Amazon ECS service revision ARN, shorten it to just the service revision ID, and use that as the file name.

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

6. Now that the file exists in Amazon S3, we can go back to the Amazon ECS console and watch the deployment complete.

![anh](/images/C45-7.png)

## Cleaning up

To clean up the AWS resources used throughout this walkthrough, we must remove the Amazon ECS service.

```yaml
$ aws ecs \
delete-service \
--service $SERVICE_NAME \
--cluster $CLUSTER_NAME \
--force
```

Next, we delete the **CloudFormation** stacks using **AWS CDK**.

```yaml
$ cdk destroy --all
```

## Conclusion

Lifecycle hooks in Amazon ECS provide endless possibilities to customize your deployment pipeline. In this post we have covered a few common use cases for these hooks, such as evaluating a deployment against governance policies, as well as implementing a manual approval step. To learn more, go to the Amazon ECS lifecycle hooks [documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-bluegreen.html), and watch demonstrations on the Containers from the Couch [YouTube](https://www.youtube.com/ContainersFromTheCouch) channel.





![anh](/images/blog1.jpg) 
**Olly Pomeroy** is a Senior Container Specialist Solution Architect at AWS.