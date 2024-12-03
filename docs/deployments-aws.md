# Deploying services to AWS

The code to deploy our [Live site](https://cudl.lib.cam.ac.uk) is stored in GitHub at https://github.com/cambridge-collection/cudl-terraform. 

## Components

The Terraform code deploys the following as a series of Terraform modules:

- Base architecture, including an AWS Virtual Private Cloud (VPC), Web Application Firewall (WAF), Autoscaling Group (ASG), Application Load Balancer (ALB) and an Elastic Container Service (ECS) cluster.
- Data processing, including S3 Buckets, Lambda Functions and Simple Queue Service (SQS) queues to deliver data for use by the Viewer.
- ECS workloads, including task definitions, Amazon Certificate Manager (ACM) certificates, Route 53 records (for DNS), CloudFront Distribution (handling CDN), ALB listeners and target groups, Identity Access Management (IAM) permissions for tasks and Elastic File System (EFS) volumes.

## Terraform Modules

Three Terraform modules are used in the deployment:

- Base Architecture: https://github.com/cambridge-collection/terraform-aws-architecture-ecs
- Data Processing: https://github.com/cambridge-collection/cudl-terraform/tree/main/modules/cudl-data-processing
- ECS Workloads: https://github.com/cambridge-collection/terraform-aws-workload-ecs

## Container Images

The ECS workloads and several lambdas are dependent on container images deployed to the AWS Elastic Container Repository (ECR). Images are managed outside of Terraform. The following container images are used:

- Lambdas:
    - SOLR listener lambdas: https://github.com/cambridge-collection/cudl-solr-listener
    - TEI Processing: https://github.com/cambridge-collection/cudl-data-processing-xslt
    - Transkribus: https://github.com/cambridge-collection/curious-cures-transkribus-lambda
- ECS:
    - Viewer: https://github.com/cambridge-collection/cudl-viewer
    - Content Loader User Interface: https://github.com/cambridge-collection/dl-loading-ui
    - Content Loader Database: https://github.com/cambridge-collection/dl-loading-ui/tree/main/docker/dl-loading-db
    - Services: https://github.com/cambridge-collection/cudl-services
    - SOLR Search API: https://github.com/cambridge-collection/cudl-search (API)
    - SOLR https://github.com/cambridge-collection/cudl-solr

Image builds and pushes to ECR are currently a largely manual process from the command line.

## Lambda JARs

Other lambdas use a Java JAR artefact. This is built using the code here https://github.com/cambridge-collection/data-lambda-transform/. Packaging is managed using Maven. JAR files are deployed to an S3 bucket in the target AWS account. The S3 bucket allows access from lambdas deployed in the same account.

Uploads to S3 are currently done manually.

## Deploying using Terraform

Each environment is deployed in a separate Terraform root module, for example the Staging environment is deployed using the code in the `cul-cudl-staging` directory. Authentication is done by assuming IAM roles in the target AWS account. Terraform state is stored in an S3 bucket for each account. State is locked using a DynamoDB table. Several environments may be deployed to one AWS account, although we have separated our production deployments (staging/live) from non-production (sandbox/dev/test) in different accounts.

To deploy new versions of components such as lambdas or ECS services based on container images, the value of the SHA256 image hash should be updated to reflect the new version. When Terraform is reapplied the new version of the service will be deployed.

For lambdas, the new image will be used on the next invocation after the resource has been updated by Terraform. For ECS deployments, a new task will be created with the new version running. The ECS capacity provider will scale out a new EC2 instance and the new task will be deployed. If this passes ALB health checks connections will be drained from the old task and the service will be scaled in and the old task stopped.

## Deploying a new environment

A minimal deployment of a new environment will require two modules:

```hcl
module "example_base" {
    source = "git::https://github.com/cambridge-collection/terraform-aws-architecture-ecs.git?ref=v2.1.0"
}

module "example_processing" {
    source = "git::https://github.com/cambridge-collection/cudl-terraform.git//modules/cudl-data-processing?ref=v1.0.0"

    providers = {
        aws.us-east-1 = aws.us-east-1
    }
}
```

Notice that the processing module requires an additional provider to deploy CloudFront resources in the `us-east-1` AWS region (the only valid region for these resources).

Using this minimal deployment a static website can be deployed to an S3 bucket (output as `destination_bucket`) using the input variable `cloudfront_origin_path` to change the path in the bucket that will serve as the root of the website. This website can be configured to refer to various files output by data transform lambdas.

## Deploying a new service

A new ECS service can be deployed alongside the base and processing modules.

```hcl
module "example_base" {
    source = "git::https://github.com/cambridge-collection/terraform-aws-architecture-ecs.git?ref=v2.1.0"
}

module "example_processing" {
    source = "git::https://github.com/cambridge-collection/cudl-terraform.git//modules/cudl-data-processing?ref=v1.0.0"

    providers = {
        aws.us-east-1 = aws.us-east-1
    }
}

module "example_service" {
    source = "git::https://github.com/cambridge-collection/terraform-aws-workload-ecs.git?ref=v3.4.0"

    vpc_id = module.example_base.vpc_id
    ...

    providers = {
        aws.us-east-1 = aws.us-east-1
    }
}
```

As can be seen many values required by the service module can be found as outputs from the base module.

ECS service deployments can deploy any container and can therefore adapt to suit any workload. References to specific container images and configuration will need to be provided in the argument `ecs_task_def_container_definitions` following the structure described [here](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_ContainerDefinition.html).

## Issues in deployments

### Debugging ECS

If the task fails to start after deploying to an EC2 host, it may be necessary to access the host instance to view the `ecs-agent` logs.

The host for the ECS deployment can be accessed with the following command:

```sh
aws ssm start-session --target $(aws ecs describe-container-instances --cluster $ECS_CLUSTER_NAME --container-instances $(aws ecs describe-tasks --cluster $ECS_CLUSTER_NAME --tasks $(aws ecs list-tasks --cluster $ECS_CLUSTER_NAME --service $ECS_SERVICE_NAME --query 'taskArns[0]' --output text) --query 'tasks[].containerInstanceArn[]' --output text) --query 'containerInstances[].ec2InstanceId' --output text)
```

This will provide a terminal shell inside the target host. ECS agent logs can be viewed with the command:

```sh
sudo docker logs ecs-agent
```

Common issues with deployments include errors around permissions, where for example the ECS task execution IAM role is missing permission to pull a container image from ECR, and network issues where the container is unable to perform a step specified in the entrypoint script.

Other issues can occur where an EFS file system fails to mount correctly, which may be due to either network issues connecting to EFS or permissions on the target host. Details about EFS mounts can be viewed in the log `/var/log/amazon/efs/mount.log`

### Tasks fail to deploy

If a task is started but does not deploy to an EC2 instance this is likely because no available container instance has capacity for the task defined.

Unfortunately if this is the case AWS does not report an error with the deployment: the deployment will simply hang until a suitable container instance is available.

The ECS capacity provider will also not scale out a new instance if the EC2 launch template does not define an instance with capacity to scale out a task.

This issue can be diagnosed by manually scaling out a new EC2 instance comparing the memory specified in the container definition with the memory available on the new container instance. If the container definition is higher, the task will not deploy.

Get the ARN of the autoscaling group 

```sh
ASG_NAME=$(aws ecs describe-capacity-providers --capacity-providers $(aws ecs describe-clusters --clusters $ECS_CLUSTER_NAME --query 'clusters[].capacityProviders' --output text) --query 'capacityProviders[].autoScalingGroupProvider.autoScalingGroupArn' --output text | awk -F '/' '{ print $2 }')
```

Increase the desired capacity of the autoscaling group: 

```sh
aws autoscaling set-desired-capacity --auto-scaling-group-name $ASG_NAME --desired-capacity 3
```

Note there is no output of this command, other than the return code.

Now you can view the available capacity of the container instances:

```sh
aws ecs describe-container-instances --cluster $ECS_CLUSTER_NAME --container-instances $(aws ecs list-container-instances --cluster $ECS_CLUSTER_NAME --query 'containerInstanceArns' --output text) --query 'containerInstances[].registeredResources[?name==`MEMORY`].integerValue | []'
```

This will return a list of numbers showing the maximum memory available on each instance.

Now you can view the memory required by the task itself:

```sh
aws ecs describe-tasks --cluster $ECS_CLUSTER_NAME --tasks $(aws ecs list-tasks --cluster $ECS_CLUSTER_NAME --service-name $ECS_SERVICE_NAME --query 'taskArns' --output text) --query 'tasks[].memory' --output text 
```

If this number is higher than all of the container instance memory values, you should edit the container definition to reduce the required memory, or increase the instance size associated with the EC2 Launch Template.
