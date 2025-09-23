# SERVERLESS

1. When you get a `429` and throttling exception  
2. Need to know where throttling is happening, Lambda or the service which is calling Lambda  
3. When Lambda retries after 3 times and fails, it puts the message in the Dead Letter Queue (DLQ)  
4. DLQ can be **SNS** or **SQS**  
5. How to deal with Lambda function concurrency:  
   1. Configure reserved concurrency  
   2. Use exponential backoff in your code  
6. Lambda Versions and Aliases:  
   1. Exam will test how to update Lambda Versions and Aliases  
   2. And how to point applications to those versions  
   3. Blue/Green Lambda scenarios  
7. What is a Lambda version → we are always working on the `$LATEST`  
8. `MyFunction:$LATEST` version of the code is mutable  
9. Later on, the version will increment to `MyFunction:1`, `MyFunction:2`, and so on  
10. Versions are immutable and cannot be changed  
11. Each version has its own **ARN**  
12. Because different versions have unique ARNs, this allows you to effectively manage different environments like Production, Staging, or Development  
13. A qualified ARN has a version suffix  
14. There is also an unqualified ARN  
15. You cannot create an Alias from an unqualified ARN  
16. What are Lambda Aliases:  
    1. You create a Lambda function → `MyFunction:1` and `MyFunction:2`  
    2. Then you create a Lambda Alias  
    3. Then you point your application to the Alias  
    4. If you want to change the Lambda version, you can then point the application to a Blue/Green version of Lambda by changing the Alias  

---

## Amazon SQS

### 1. Standard Queue → Best Effort Ordering  
1. Unlimited throughput → nearly unlimited number of transactions per second  
2. Best-effort ordering: messages might be delivered in an order different from how they were sent  
3. A message is delivered more than once occasionally  

### 2. FIFO Queue → First-In, First-Out Queue  
1. A FIFO queue supports up to 300 messages per second  
2. When you batch 10 messages per operation, FIFO queue can support 30,000 messages per second  
3. Order is preserved  
4. Messages are delivered just once and remain available until a consumer processes and deletes them (**Exam topic**)  
5. Message Group ID and Message Deduplication:  
   1. **Message Group ID** → A tag that specifies the specific group  
   2. **Message Deduplication** → Token used for deduplication of messages within the deduplication interval  

### 3. Dead Letter Queue  
1. Messages not processed successfully:  
   1. `ReceiveCount` exceeds  
   2. `maxReceiveCount` for queue  
2. Dead Letter Queues can be **Standard** or **FIFO**  
3. The main task is to analyze the failed messages  
4. It is not the queue type (Standard or FIFO) itself, but rather designated as a **Dead Letter Queue** in the configuration of another queue  

### Dead Letter Queue Settings:  
1. Enable **Redrive Policy** – controls what happens when a message in a queue can’t be processed successfully after several tries. You can set maximum receives; after failing that many times, it is moved to DLQ  
2. Specify the queue to use as the DLQ  
3. Specify the maximum receives before a message is sent to DLQ  

## 4. Delay Queue  
- You specify the **Visibility Timeout** → the message is not visible in the queue until that time  

## 5. Long Polling vs Short Polling  
1. **Short polling** checks a subset of servers and may not return all messages → `WaitTimeSeconds(0)` = received message wait time  
2. **Long polling** waits for `WaitTimeSeconds(1–20)` and eliminates empty responses (waits for messages to arrive, fewer API calls, reduces cost)  

## 6. Application Integration Services Comparison  

1. **Simple Queue Service (SQS)**  
   - Messaging queue → store and forward pattern  
   - Use case: Building distributed/decoupled applications  

2. **Simple Notification Service (SNS)**  
   - Setup, operate, and send notifications from the cloud  
   - Use case: Send email notifications when CloudWatch alarms trigger  

3. **Step Functions**  
   - Out-of-box coordination of AWS service components with visual workflow  
   - Use case: Order processing workflow  

4. **Simple Workflow Service (SWF)**  
   - Supports external processes or specialized execution logic  
   - Use case: Human-enabled workflows like order fulfillment or procedural requests  
   - Note: *AWS recommends using Step Functions instead of SWF for new customers*  

5. **Amazon MQ**  
   - Message broker service for **Apache MQ** and **RabbitMQ**  
   - Use case: Need a message queue that supports industry-standard APIs and protocols, or migrate existing queues to AWS  

6. **Amazon Kinesis**  
   - Collects, processes, and analyzes streams of data  
   - Use case: Collects data from IoT devices for later processing  

## 7. AppFlow  

1. Securely transfers data between SaaS applications  
   - Data sources → Slack, Salesforce  
   - Destinations → AWS services like Redshift, S3, Snowflake, and more  
   - Uses **flows** to connect a data source to a destination  
   - Flows can run on-demand or based on events  
   - Supports up to **100GB per flow**  
   - Performs data transformations (mapping, merging, filtering, validation)  

2. AppFlow Triggers:  
   1. On-demand  
   2. On a schedule  
   3. On an event  

3. Use Cases:  
   1. Synchronizing data from multiple sources for analytics  
   2. Data enrichment (e.g., pull data from SaaS → send back to app)  
   3. Event-based flows → automatically trigger processes based on other data  
   4. Transfer opportunities from Salesforce to populate real-time dashboards  
   5. Automate data backup from SaaS apps to AWS services like S3  
   6. Analyze Slack events using business intelligence tools  

---

## 8. Amazon EventBridge  

100% exam topic:  
**Event Source → Events → EventBridge Event Bus → Rules → Target**  

- Example 1:  
  - EC2 creates some events  
  - Event = EC2 instance terminated  
  - Event picked up by EventBridge Bus  
  - Rules forward to a target (e.g., SNS → send notification)  

- Example 2:  
  - CloudTrail creates event  
  - Event = `S3PutBucketPolicy` API  
  - EventBridge Bus picks up the event  
  - Rules forward event data → SNS → email notification  

---

## 9. API Gateway Overview  

1. **Edge-Optimized Endpoint** → Uses CloudFront + API Gateway → reduced latency globally  
2. **Regional Endpoint** → Reduced latency in same region  
3. **Private Endpoint** → Secure REST API exposure to other services inside VPC or via Direct Connect  

### How API Gateway Integration Works:  
1. Lambda → Lambda Proxy Integration or Lambda Custom Integration  
2. HTTP Endpoint → HTTP Proxy or HTTP Custom Integration  
3. AWS Service Action → Native AWS integration (non-proxy type)  

### API Caching  
1. Cache API calls by provisioning API Gateway cache (size in GB)  
2. Caching stores endpoint responses  
3. Caching reduces backend calls and improves request latency  

### API Throttling  
1. Steady state and burst limit of request submission against your account  
2. Request rate limit = **10,000 RPS**  
3. Max concurrent requests = 5,000–10,000 (per account) → exceeding results in `429` errors  
4. Solution: Clients must retry with backoff and respect throttling limits  

### API Usage Plans & API Keys  
1. **Premium** usage plans → higher API limits  
2. **Basic** usage plans → lower limits  
3. Users connect via API keys configured in a usage plan  

---

# Architectural Patterns for Serverless Applications

1. **EC2 and RDS experience traffic spikes causing dropped writes**  
   - Solution: Decouple EC2 and RDS using SQS. Lambda processes messages in queue  

2. **Migrate on-premises web app with NFS processing**  
   - Solution: Migrate to EC2, use SQS + EFS + Auto Scaling. Scale tier based on SQS queue length  

3. **Lambda execution time increases with data size**  
   - Solution: Increase Lambda memory (increases CPU proportionally)  

4. **API Gateway streaming data causes throttling (`TooManyRequestsException`)**  
   - Solution: Send data to Kinesis Data Stream → configure Lambda to process in batches  

5. **Highly variable load, ordered processing required**  
   - Solution: Use SQS FIFO Queue  

6. **Long-running media processing (>2 hours)**  
   - Solution: Lambda writes jobs to SQS. Step Functions coordinate multiple functions in parallel  

7. **S3 Object Upload must trigger processing**  
   - Solution: Configure S3 Event Notification → trigger Lambda  

8. **Root User API activity captured in third-party ticketing system**  
   - Solution: CloudTrail + EventBridge Rule → put events in SQS → process with Lambda  

9. **Legacy batch scripts chained** (hard to maintain)  
   - Solution: Migrate to Lambda functions + Step Functions for orchestration  

10. **Large S3 upload volume → prevent affecting critical Lambdas**  
    - Solution: Configure reserved concurrency; monitor via CloudWatch throttling metric  

11. **EC2 image processing with JS, storing in S3 — costly & variable load**  
    - Solution: Replace EC2 with Lambda  

12. **Lambda Canary Deployment Strategy**  
    - Solution: Create Alias, configure weighted routing between versions  

13. **API Gateway regional REST API expansion to global**  
    - Solution: Convert to Edge-Optimized API  

14. **API Gateway + Lambda during peak → many retries (no Lambda errors)**  
    - Solution: Increase throttle limits  

15. **Restrict API Gateway access to IAM users only**  
    - Solution: Set Authorization to `AWS_IAM`. Grant `execute-api:Invoke` in IAM policy  

# Docker Container and PaaS

## Docker Container and Microservice

1. Docker containers:
   1. Can start up very quickly
   2. A container includes all the code, settings, and dependencies for running an application
   3. Each container is isolated from other containers
   4. Containers are very resource efficient
2. Monolith Application:
   1. All the application runs in the same host
   2. The user interface, business logic, and data access layer are combined on a single platform
   3. Updates to or failure of any single component can take down the whole application
3. Microservice Architecture:
   1. A microservice is an independently deployed unit of code
   2. They are often loosely coupled
   3. They are organized around business units
   4. Many instances of each microservice can run on each host
   5. We can spread our containers across hosts

## Amazon Elastic Container Service

1. Amazon ECS (Elastic Container Service) is a multi-AZ (Availability Zone) service
2. An ECS task is a running Docker container
3. A task is created by a task definition
4. We host images in AWS EC Registry
5. ECS services are used to maintain the desired count of tasks
6. ECS container instances can add auto scaling
   1. Key features of ECS key services:
      1. Serverless with AWS Fargate - managed for you and fully scalable
      2. Fully managed container orchestration - control plane is managed for you
      3. Supports Docker - run and manage Docker containers with integration into the Docker Compose CLI
      4. Windows container support - ECS supports management of Windows containers
      5. Elastic Load Balancing integration - distribute traffic across containers using ALB or NLB
      6. Amazon ECS Anywhere (new) - Enables the use of Amazon ECS control plane to manage on-premises implementation
7. ## ECS Components
   1. Cluster - logical grouping of EC2 instances
   2. Container instances - EC2 instances running the ECS agent
   3. Task definition - Blueprint that describes how a Docker container should launch
   4. Task - A running container using settings from a task definition
   5. Service - Defines long-running tasks; can control task count with Auto Scaling and attach an ELB
8. ## ECS Launch Type
   1. ### ECS EC2 Launch Type
      1. You explicitly provision EC2 instances
      2. You are responsible for managing EC2 instances
      3. Charged per EC2 instance
      4. Docker volumes, EFS, and FSx for Windows
      5. You handle cluster optimization
      6. More granular control over infrastructure
   2. ### Fargate Cluster
      1. We create a Fargate cluster
      2. We have ECS service and then we have a task running
      3. There are no container instances to manage
      4. Fargate automatically provisions resources
      5. Fargate provisions and manages compute
      6. Charged for running tasks
      7. EFS integration
      8. Fargate handles cluster optimization
      9. Limited control; infrastructure is automated
   3.  Auto Scaling for ECS
      1. Two types of auto scaling:
         1. Service auto scaling
            1. Automatically adjusts the desired task count up or down using the application auto scaling service
         2. Cluster auto scaling
            1. Uses a capacity provider to scale the number of EC2 cluster instances using EC2 Auto Scaling
               1. Target scaling policies:
                  1. Increase or decrease the number of tasks that your service runs to a target value for a specific CloudWatch metric
                  2. Step scaling policies - Increase or decrease the number of tasks that your service runs in response to CloudWatch alarms. Step scaling is based on scaling adjustments known as step adjustments, which vary based on the size of the alarm breach
                  3. Scheduled Scaling - Increase or decrease the number of tasks that your service runs based on the date and time
      2. 
         1. Service Auto Scaling
            1. Service auto scaling automatically adjusts the desired task count up or down using the application auto scaling service
            2. Service auto scaling supports target tracking, step, and scheduled scaling policies
            3. Cluster auto scaling uses a capacity provider to scale the number of EC2 cluster instances using EC2 auto scaling
            4. Uses an ECS resource type called capacity provider
            5. A capacity provider can be associated with an EC2 Auto Scaling group
            6. Auto Scaling group scales using:
               1. Managed scaling - with an automatically created policy on your ASG
               2. Managed instance termination protection - enables container-aware termination of instances in the ASG when scale in happens
9. Amazon ECS with ALB
   1.  A dynamic port is allocated on the host
   2.  All connections to the web service come into the HTTP listener port 80
   3.  Each task is running a web service on port 80
   4.  NAT gateway required for tasks in a private subnet to access the internet
10. ## EKS
    1.  It also spans multi-AZ
    2.  EKS is a managed service for running Kubernetes applications in the cloud or on-premises
    3.  Kubernetes is an open source system for automating deployment, scaling, and management of containerized applications
    4.  Use when you need to standardize container orchestration across multiple environments using a managed Kubernetes implementation
        1. Features:
           1. Hybrid deployment - managed Kubernetes clusters and applications across hybrid environments (AWS + on-premises)
           2. Batch processing - run sequential or parallel batch workloads on your EKS cluster using the Kubernetes Job API. Plan, schedule, and execute batch workloads
           3. Machine learning - use Kubeflow with EKS to model machine learning workflows and efficiently run distributed training jobs using the latest EC2 GPU-powered instances including Inferentia
           4. Web application - Build web applications that automatically scale up and down and run in a highly available configuration across multiple zones
        2. Amazon EKS Auto Scaling
           1. Vertical Pod Autoscaler - Automatically adjust the CPU and memory reservation for your pods to help "right size" your applications
           2. Horizontal Pod Autoscaler - Automatically scales the number of pods in a deployment, replication controller, or replica set based on resources CPU utilization
           3. Cluster Auto Scaling
              1. Amazon EKS supports two autoscaling products:
                 1. Kubernetes cluster autoscaler
                 2. Karpenter open source autoscaling project
              2. The cluster autoscaler uses AWS scaling groups, while Karpenter works directly with the Amazon EC2 fleet
        3. EKS Load Balancer
           1. Amazon EKS supports Network Load Balancer and Application Load Balancer
           2. The AWS Load Balancer Controller manages AWS Elastic Load Balancer
           3. Install the AWS Load Balancer Controller using Helm v3 or later or by applying a Kubernetes manifest
           4. The controller provisions the following resources:
              1. AWS ALB when you create a Kubernetes service of type LoadBalancer
              2. AWS Network Load Balancer when you create a Kubernetes service of type LoadBalancer
           5. In the past, the Kubernetes Network Load Balancer was used for instance targets, but the AWS Load Balancer Controller was used for IP targets
           6. With the AWS Load Balancer Controller version 2.3.0 or later, you can create NLB using either target type
        4. EKS Distro
           1. AWS EKS Distro is a distribution of Kubernetes with the same dependencies as Amazon EKS
           2. Allows you to manually run Kubernetes clusters anywhere
           3. EKS Distro includes binaries and containers of open source Kubernetes, etcd, networking, and storage plugins tested for compatibility
           4. You can securely access EKS Distro releases as open source on GitHub or with AWS via Amazon S3 and Amazon ECR
           5. Amazon EKS Distro alleviates the need to track updates, determine compatibility, and standardize on a common Kubernetes version across distribution terms
           6. You can create an EKS Distro cluster in AWS on EC2 and on your own on-premises hardware using the tooling of your choice
        5. EKS and ECS Anywhere
           1. Run ECS or EKS on cluster-managed infrastructure, supported by AWS
           2. Customers can run Amazon ECS/EKS Anywhere on their own on-premises infrastructure on bare metal servers
           3. You can deploy ECS/EKS Anywhere using VMware vSphere

        1. Rolling updates
           1. Update a few instances at a time and then move onto the next batch once the first batch is healthy
           2. Application is running both versions simultaneously
           3. Each batch of instances is taken out of service while the deployment takes place
           4. Your environment capacity will be reduced by the number of instances in a batch while the deployment takes place
           5. Not ideal for performance-sensitive systems
           6. If the updates fail, you will need to perform additional rolling updates to roll back the changes
           7. No additional cost
           8. Long deployment time
        2. Rolling with Additional Batch Update
           1. Like Rolling but launches new instances in a batch ensuring full availability
           2. Application is running at capacity
           3. Can set the batch size
           4. Application is running both versions simultaneously
           5. Small additional cost
           6. Additional batch is removed at the end of the deployment
           7. Longer deployment
           8. Good for production environments
        3. Immutable Update
           1. Launches new instances in a new ASG and deploys the version updates to these instances before swapping traffic to these instances once healthy
           2. Zero downtime
           3. New code is deployed to new instances using an ASG
           4. High cost as double the number of instances run during updates
           5. Longest deployment time
           6. Quick rollback in case of failure
           7. Great for production
        4. Blue Green
           1.  Not a feature of Elastic Beanstalk
           2.  You create a new staging environment and deploy updates there
           3.  The new environment ("green") can be validated independently and you can roll back if there are issues
           4.  Route 53 can be set up using the weighted policy to redirect a percentage of traffic to the staging environment
           5.  Using Elastic Beanstalk you can "swap URLs" when done with environment testing
           6.  Zero downtime
11. App Runner
    1. Fully managed service for deploying container web apps and APIs
    2. Source is GitHub and ECR registry
        1. Source
        2. Auto scaling
        3. Load balancer
        4. Health check
        5. Networking
        6. Observability
        7. You will get a URL for the app
        8. PaaS solution with all components managed - just bring your code and container image
12. Architectural patterns for EKS, ECS, and App Runner (aka container services):
   1. Company plans to deploy Docker containers on AWS at lowest cost.
      - Use Amazon ECS with a cluster of spot instances and enable spot instances
   2. Company plans to migrate Docker containers to AWS and does not want to manage operating systems
      - Migrate to AWS ECS using Fargate launch type
   3. Fargate task is launched in a private subnet and fails with error "CannotPullContainer"
      - Disable auto-assignment of public IP address and configure a NAT gateway
   4. Application will be deployed in Amazon ECS and must scale based on memory
      - Use service auto scaling and use memory utilization
   5. Company requires standard Docker container automation and management services to be used across multiple environments
      - Deploy EKS

   Container use case:
   - Scaling with service auto scaling
   - Attaching load balancer to ECS service

   6. Company needs to move many simple web apps running on PHP, Java, and Ruby to AWS; utilization is very low
      - Deploy to single-instance Elastic Beanstalk environment
   7. Business critical application running on Elastic Beanstalk must be updated; require zero downtime and quick, complete rollback
      - Update using an immutable update with a new ASG and swap traffic
   8. A development application running on Elastic Beanstalk needs a cost-effective and quick updates; downtime is acceptable
      - Use an all-at-once update
   9. Need managed environment for running a simple web application. App processes incoming data which can take several minutes per task.
      - Use an Elastic Beanstalk environment with a web server for the app front-end and decoupled worker tier for the long running process

# Deployment and Management

## CICD Overview

1. Developers frequently merge code into a shared repository.  
2. Each merge triggers automated builds and tests to catch issues early.  
   1. ### Continuous Integration  
3. Helps detect integration problems sooner rather than later.  
   1. Developer commits code to GitHub repository.  
   2. Build server builds and tests the code, aka AWS CodeBuild.  
   3. Results returned to developers.  
4. Source code services:  
   1. GitHub  
   2. GitLab  
   3. Bitbucket  
   4. Azure Repos  
5. Build and test services include:  
   1. GitHub Actions  
   2. GitLab CI/CD  
   3. Bitbucket Pipelines  
   4. AWS CodeBuild  
   5. Jenkins  
6. ### Continuous Delivery  
   1. Ensures that code is always in a deployable state after passing automated tests.  
   2. After passing tests, code must be manually approved before deployment to production or staging.  
   3. Automates everything except the final release step.  
   4. Allows the team to decide when to deploy rather than deploying automatically.  
7. ### Continuous Deployment  
   1. Fully automated deployment to production without manual approval.  
   2. Every successful change that passes tests is automatically released.  
   3. Requires a high level of automation and confidence in testing.  
   4. Best suited for teams needing rapid updates with minimal risk.  
8. CodePipeline → combines source code (repository) + build and test (CodeBuild) + deployment (CodeDeploy) in a single event-driven pipeline.  

## CodePipeline

1. It provides tool integration for many AWS and 3rd party software at each stage of the pipeline including:  
   1. #### Source Stage  
      1. S3, ECR, GitHub, GitLab, Bitbucket, etc.  
   2. #### Build  
      1. Run shell commands or use AWS CodeBuild, ECR, or Jenkins.  
   3. #### Test  
      1. CodeBuild, Device Farm, Jenkins, Ghost Inspector UI testing, Micro Focus StormRunner Load, Runscope API monitoring.  
   4. #### Deploy Stage  
      1. AppConfig, CloudFormation, CodeDeploy, ECS, Elastic Beanstalk, AWS Service Catalog, S3, Alexa Skills Kit.  
2. Pipelines  
   1. A workflow that describes how software changes go through the release process.  
   2. Each pipeline is made up of a series of stages.  
   3. There are two types of pipelines:  
      1. V1 type pipeline has a JSON structure that contains standard pipeline, stage, and action-level parameters.  
      2. V2 type pipeline has the same structure as V1, along with additional parameters for release safety and trigger configuration.  
3. Artifacts  
   1. Files or changes that will be worked on by the actions and stages in the pipeline.  
   2. Each pipeline state can create "artifacts".  
   3. Artifacts are passed, stored in Amazon S3, and then passed on to the next stage.  
   4. Examples include application source code, built application dependencies, definition files, and templates.  
4. Stages  
   1. Pipelines are broken into stages.  
   2. Example sequence: source → build → staging → deployment.  
   3. Stage examples include source, build, test, deploy, and load test.  
   4. Manual approval can be defined at any stage.  
5. Actions  
   1. Stages contain at least one action.  
   2. For example, a source action triggered by a code change or an action deploying the application to instances.  
   3. Action affects artifacts and will have artifacts as input, output, or both.  
6. Transitions  
   1. The progression from one stage to another inside a pipeline.  
7. Pipeline executions  
   1. An execution is a set of changes released by the pipeline. Each execution is unique and has its own ID.  
   2. An execution corresponds to a set of changes, such as merged commits or a manual release of the latest commit.  
   3. Two executions can release the same set of changes at different times.  

## AWS CodeBuild

1. AWS CodeBuild is a fully managed continuous integration service.  
2. It compiles source code, runs tests, and produces software packages that are ready to deploy.  
   1. Trigger code changes →  
      1. AWS CodeBuild launches a temporary containerized environment based on the specified build image.  
      2. The build steps are executed based on instructions in the `buildspec.yml` file.  
      3. CodeBuild scales continuously and processes multiple builds concurrently.  
      4. You pay based on the time it takes to complete the builds.  
      5. CodeBuild accepts source code from GitHub, Bitbucket, CodePipeline, S3, etc.  
      6. Output logs can be sent to Amazon S3 and Amazon CloudWatch Logs.  
   2. AWS CodeBuild Components  
      1. Build Project  
         1. Defines how CodeBuild will run a build, defines settings including:  
            1. Location of the source code.  
            2. The build environment to use.  
            3. Where to store the output of the build.  
      2. Build Environment  
         1. The operating system, language runtime, and tools that CodeBuild uses for the build.  
      3. Build Specification  
         1. A YAML file that describes the collection of commands and settings for CodeBuild to run a build.  

## CodeDeploy

1. An AWS CodeDeploy application contains information about what to deploy and how to deploy it.  
2. Need to choose the compute platform:  
   1. EC2/On-premises  
      1. On EC2 or on-premises or both.  
      2. Traffic is directed using:  
         1. In-place - just one instance; it may mean downtime.  
         2. Blue/green - no downtime.  
   2. AWS Lambda  
      1. Used to deploy applications that consist of updated versions of Lambda functions.  
      2. You can manage the way traffic is shifted to the updated Lambda function version during deployment by choosing a canary, linear, or all-at-once configuration.  
   3. Amazon ECS  
      1. Used to deploy an Amazon ECS containerized application as a task set.  
      2. CodeDeploy performs a blue/green deployment by installing an updated version of the application as a new replacement task set.  
      3. CodeDeploy reroutes production traffic from the original task set to the replacement task set.  
      4. The original task set is terminated after a successful deployment.  
      5. You can manage the way traffic is shifted to the updated task set during deployment by choosing a canary, linear, or all-at-once configuration.  
3. IMP - Blue/Green Traffic Shifting  
   1. **AWS Lambda:** Traffic is shifted from one version of a Lambda function to a new version of the same function.  
   2. **Amazon ECS:** Traffic is shifted from a task set in your Amazon ECS service to an updated, replacement task set in the same service.  
   3. **EC2/On-premises:** Traffic is shifted from one set of instances in the original environment to a replacement set of instances.  
      1. All AWS Lambda and Amazon ECS deployments are blue/green. An EC2/On-premises deployment can be in-place or blue/green.  
      2. **For Amazon ECS and AWS Lambda, there are 3 ways traffic can be shifted during a deployment:**  
         1. **Canary:** Traffic is shifted in increments. You can choose from predefined canary options that specify the percentage of traffic shifted to your updated Amazon ECS task set/Lambda function in the first increment and the interval in minutes before the remaining traffic is shifted in the second increment.  
         2. **Linear:** Traffic is shifted in equal increments with an equal number of minutes between each increment. You can choose from predefined linear options that specify the percentage of traffic shifted in each increment and the number of minutes between increments.  
         3. **All-at-once:** All traffic is shifted from the original Amazon ECS task set/Lambda function to the updated Amazon ECS task set/Lambda function all at once.  

## CloudFormation

1. Infrastructure as code using templates.  
2. You pass the template to CloudFormation.  
3. The infrastructure is provisioned consistently with fewer mistakes.  
4. Less time and effort than configuring resources manually.  
5. You can use version control and peer review for your CloudFormation templates.  
6. Free to use - you are only charged for the resources provisioned.  
7. Can be used to manage updates and dependencies.  
8. Can be used to rollback and delete the entire stack as well.  
9. Key components:  
   1. Template - The JSON or YAML text file that contains the instructions for building out the AWS environment.  
   2. Stacks - The entire environment described by the template, created, updated, and deleted as a single unit.  
   3. StackSets - AWS CloudFormation StackSets extend the functionality of stacks by enabling you to create, update across multiple accounts and regions with a single operation.  
   4. Change Set - A summary of proposed changes to your stack; allows you to see how changes might impact existing resources before implementing them.  
10. Template is a YAML or JSON file used to describe the end-state of the infrastructure you are provisioning or changing.  
11. After creating the template, you upload it to CloudFormation directly or using Amazon S3.  
12. CloudFormation reads the template and makes API calls on your behalf.  
13. The resulting resources are called a "Stack".  
14. The logical ID is used to reference resources within the template.  
15. Physical ID identifies resources outside of AWS CloudFormation templates but only after the resources have been created.  

## CloudFormation Stacks

1. Deployed resources based on a template.  
2. Create, update, and delete stacks using the template.  
3. Deployed through the Management Console, CLI, or API.  
4. Stack creation errors:  
   1. Automatically roll back on error is enabled by default.  
   2. You will be charged for resources provisioned even if there is an error.  
5. AWS CloudFormation StackSets extend the functionality of stacks by enabling creation, update, or deletion across multiple accounts and regions with a single operation.  
6. Using an administrator account, define and manage an AWS CloudFormation template as the basis for provisioning stacks into selected target accounts across specified regions.  
7. The administrator account is the one you use to create the StackSets.  

## Nested Stacks

1. Nested stacks allow reuse of CloudFormation code for common use cases.  
2. For example, standard configuration for a load balancer, web server, application server, etc.  
3. Instead of copying code, create a standard template for each common use case and reference it from within your CloudFormation template.  

## AWS CloudFormation Change Set

1. CloudFormation provides two methods for updating stacks: direct update or creating and executing change sets.  
2. When you directly update a stack, you submit changes and CloudFormation immediately deploys them.  
3. Use direct updates when you want to quickly deploy your updates.  
4. With change sets, you can preview the changes CloudFormation will make to your stack and then decide whether to apply those changes.  

## AWS Service Catalog (Important)

1. AWS Service Catalog allows organizations to create and manage catalogs for IT services approved for use on AWS.  
2. IT services can include everything from virtual machine images, servers, software, and databases to complete multi-tier application architectures.  
3. AWS Service Catalog allows you to centrally manage commonly deployed IT services.  
4. Enables users to quickly deploy only the approved IT services they need.  
5. Catalog administrators author templates and create products.  
6. Products are added to portfolios.  
7. Users/groups are assigned permissions to portfolios.  
8. Constraints are added to portfolios/products.  
9. Catalog administrators:  
   1. Manage a catalog of products, organizing them into portfolios and granting access to end users.  
   2. End users receive AWS credentials and use the AWS Management Console to launch products to which they have been granted access.  
10. AWS Service Catalog Constraints (Important)  
    1. Constraints control the way specific AWS resources can be deployed for a product.  
    2. Use constraints for governance or cost control limits on products.  
    3. **Launch Constraints** - Specify a role for a product in a portfolio. This role provisions the resources at launch, restricting user permissions without impacting users' ability to provision products from the catalog.  
    4. **Notification Constraints** - Enable notifications about stack events using an Amazon SNS topic.  
    5. **Template Constraints** - Restrict configuration parameters available for users when launching products.  
11. AWS Service Catalog Sharing (Important)  
    1. You can share portfolios across accounts in AWS Organizations.  
    2. You can either share a reference to the catalog or deploy a copy of the catalog.  
    3. With copies, you must redeploy any updates.  
    4. CloudFormation StackSets can be used to deploy a catalog to multiple accounts at the same time.  

## AWS Cloud Development Kit (CDK)

1. Open source development framework to define your cloud application resources using familiar programming languages.  
2. Preconfigured cloud resources with proven defaults using constructs.  
3. Provision your resources using AWS CloudFormation.  
4. Enables modeling application infrastructure using TypeScript, Python, Java, and .Net.  
5. Use existing IDEs, testing tools, and workflow patterns.  
6. Download preconfigured application components.  
7. Model infrastructure using a programming language.  
8. Deploy with CloudFormation.  

## AWS Serverless Application Model (SAM)

1. Provides a shorthand syntax to express functions, APIs, databases, and event source mapping.  
2. Can be used to create Lambda functions, API endpoints, DynamoDB tables, and other resources.  
3. A SAM template file is a YAML configuration that represents the architecture of a serverless application.  
4. You use the template to declare all AWS resources that comprise your serverless application in one place.  
5. AWS SAM templates are an extension of AWS CloudFormation templates, so any resources declared in CloudFormation can also be used in SAM.  
6. The `Transform` header indicates it's a SAM template:  
   1. `Transform: 'AWS::Serverless-2016-10-31'`  
   2. Several resource types:  
      1. `AWS::Serverless::Function` (AWS Lambda)  
      2. `AWS::Serverless::Api` (API Gateway)  
      3. `AWS::Serverless::SimpleTable` (DynamoDB)  
      4. `AWS::Serverless::Application` (AWS Serverless Application Repository)  
      5. `AWS::Serverless::HttpApi` (API Gateway HTTP API)  
      6. `AWS::Serverless::LayerVersion` (Lambda Layers)  

## AWS Systems Manager (Important)

1. Manages many AWS resources including EC2, S3, and RDS.  
2. Systems Manager components:  
   1. Automation  
      1. Document - defines the action to perform, written in YAML or JSON.  
      2. Automation automates IT operation and management tasks across AWS resources – e.g., taking a snapshot of an RDS DB instance.  
   2. Run Command  
      1. Document types include command, automation packages, etc.  
      2. Runs commands on managed EC2 instances.  
      3. For example, checks for missing updates.  
   3. Inventory  
      1. Provides inventory of managed resources.  
   4. Patch Manager  
      1. Selects and deploys patches for operating systems and software automatically across large groups of Amazon EC2 or on-premises instances.  
      2. Patch Baseline:  
         1. Sets rules to auto-approve select categories of patches to be installed.  
         2. Specifies a list of patches that overrides these rules and are automatically approved or rejected.  
      3. You can schedule maintenance windows to apply patches during predefined times.  
      4. Systems Manager helps ensure your software is up to date and meets compliance policies.  
         1. Scans managed instances for patch compliance and configuration inconsistencies.  
         2. Collects and aggregates data from multiple AWS accounts and regions and drills down into specific non-compliant resources.  
         3. By default, displays data about patching and associations.  
         4. Can customize and create your own compliance types using AWS CLI, AWS Tools for Windows PowerShell, or SDKs.  
   5. Session Manager (important)  
      1. Secure remote management of instances at scale without logging into servers.  
      2. Replaces the need for bastion hosts, SSH, or remote PowerShell.  
      3. Integrates with IAM for granular permissions.  
      4. All actions with Systems Manager are recorded by AWS CloudTrail.  
      5. Can store session logs in S3 and output to CloudWatch Logs.  
      6. Requires IAM permission for EC2 instances to access SSM, S3, and CloudWatch Logs.  
   6. Parameter Store  
      1. Provides secure, hierarchical storage for configuration data management and secrets management.  
      2. Highly scalable, available, and durable.  
      3. Stores data such as passwords, database strings, license codes, and parameter values.  
      4. Stores values as plaintext (unencrypted) or ciphertext (encrypted).  
      5. Reference values by the unique name specified when created.  
      6. Does not provide native rotation of keys (unlike AWS Secrets Manager).  

## AWS RAM  

1. Shares resources with accounts or organizations.  
2. RAM can be used to share:  
   1. AWS Transit Gateways  
   2. Subnets  
   3. AWS License Manager configurations  
   4. AWS Route 53 Resolver rules  

## AWS Health API Dashboard

1. AWS Personal Health Dashboard provides alerts and remediation guidance when AWS experiences events that may impact you.  
2. Provides a personalized view into performance and availability of AWS services underlying your resources.  
3. Provides proactive notifications to help plan for scheduled activities.  

## AWS Service Health Dashboard

1. Not personalized information, so may not be relevant to you.  
2. No proactive notification of scheduled activities.  
3. Shows current status information on service availability.  

## AWS Well-Architected Tool

1. You define your workload.  
2. Workload can be defined in AWS Service Catalog.  
3. The application you define is provided in the App Registry feature.  
4. Conduct an architectural review.  
5. Uses AWS Trusted Advisor.  
6. Looks at 6 pillars of the Well-Architected Framework:  
   1. Operational excellence  
   2. Security  
   3. Reliability  
   4. Performance efficiency  
   5. Cost optimization  
   6. Sustainability  
7. Then you can apply reports, documentation, videos, and dashboards.  
8. Review the state of your applications and workloads against architectural best practices.  
9. Identify opportunities for improvement and track progress over time.  
10. API extends best practices, measurements, and learning into your existing governance processes, applications, and workflows.  
11. Create your own architecture governance process, applications, and workflows.  
12. Create your own custom lenses – add your own best practice pillars and questions based on what's most important to your organization.  
13. Integrate with AWS Organizations for sharing workloads and custom lenses with up to 300 individual IAM users or accounts.  
14. Integrate with Trusted Advisor to aid workload reviews, providing automated context for supported questions.  
    1. AWS Well-Architected lenses:  
       1. Extends guidance offered by AWS Well-Architected to specific industry and technology domains.  
       2. Lenses provide a way to consistently measure your architecture against best practices and identify areas for improvement.  
       3. The AWS Well-Architected Framework lens is automatically applied when a workload is defined.  
       4. The following lenses are provided by AWS:  
          1. The Serverless lens focuses on designing, deploying, and architecting your serverless application workload in the AWS Cloud.  
          2. The SaaS lens focuses on designing, deploying, and architecting your software-as-a-service workload in the AWS Cloud.  
          3. The FTR lens is designed for Independent Software Vendors (ISVs) preparing for a Foundational Technical Review (FTR) in the AWS Partner Network (APN).  

## RPO and RTO (Very Important)

1. Recovery Point Objective (RPO)  
   1. Measurement of the amount of data that can be acceptably lost.  
   2. Measured in seconds, minutes, or hours.  
      1. Example:  
         1. You can acceptably lose 2 hours of data in a database (2 hrs RPO).  
         2. This means backups must be taken every 2 hours.  
2. Recovery Time Objective (RTO)  
   1. Measurement of the amount of time it takes to restore after a disaster event.  
   2. Measured in seconds, minutes, or hours.  
      1. Example:  
         1. The IT department expects it to take 4 hours to bring your application online after a disaster (RTO of 4 hrs).  
3. Achievable RPO – **The RPO is determined by how you can take a backup of the data:**  

| Recovery Point Objective (RPO) | Technique                                    |
|--------------------------------|----------------------------------------------|
| Milliseconds to Seconds         | Synchronous Replication                       |
| Seconds to Minutes              | Asynchronous Replication                      |
| Minutes to Hours               | Snapshot, Cloud Backup, D2D (Disk-to-Disk)   |
| Hours to Days                  | Offsite Backup, Traditional Backup, Tape     |

4. Achievable RTO – **The RTO is determined by how quickly you can recover:**  

| Recovery Time Objective (RTO)  | Technique                                                 |
|-------------------------------|-----------------------------------------------------------|
| Milliseconds to Seconds        | Fault Tolerance                                           |
| Seconds to Minutes             | High Availability, Load Balancing, Auto Recovery         |
| Minutes to Hours              | Cross-site Recovery (Cloud), Automated Recovery           |
| Hours to Days                 | Cross-site Recovery (Cloud/On-Premises), Manual Recovery  |

5. Disaster Recovery Strategies  
   1. **Backup and Restore**  
      1. Low priority workloads  
      2. Provision/restore after events  
      3. Cost $  
   2. **Pilot Light**  
      1. Data replicated  
      2. Services idle/off  
      3. Resources activated after events  
      4. Cost $$  
   3. **Warm Standby**  
      1. Minimum resources always running  
      2. Business critical workloads  
      3. Scale up/out after events  
      4. Cost $$$  
   4. **Multi-site Active/Active**  
      1. Zero downtime  
      2. Near zero data loss  
      3. Mission critical workloads  
      4. Cost $$$  

6. AWS Elastic Disaster Recovery  
   1. Disaster recovery from:  
      1. On-premises to the cloud  
      2. Cloud to AWS  
      3. AWS region to AWS region  
   2. Recovery of:  
      1. Compute (to Amazon EC2)  
      2. Disk Storage (to Amazon EBS)  
   3. Services used:  
      1. EC2 - Replicates on-premises servers to Amazon EC2 instances, allowing for rapid failover during disasters  
      2. S3 - Stores recovery point data  
      3. AWS Lambda - Provides monitoring and alerting capabilities  
      4. Amazon CloudWatch - Provides monitoring and alerting capabilities  
      5. AWS KMS - For encryption of data in transit and at rest  
   4. How Elastic Disaster Recovery works:  
      1. Open outbound port 443 to Elastic Disaster Recovery and S3  
      2. Open outbound port 1500 to a staging subnet in an Amazon VPC  
      3. AWS Replication Agent must be installed on servers to be protected  
      4. Servers can be physical, virtual, on-premises, or Amazon EC2  
   5. Capabilities and limitations:  
      1. RTO in seconds  
      2. RTO of 5 to 20 minutes  
      3. Factor in OS boot times  
         1. Linux ~ 5 minutes  
         2. Windows ~ 20 minutes  

## Architectural Patterns – Deployment and Management

1. Global company needs to centrally manage creation of infrastructure services across accounts in AWS Organization.  
   - Define infrastructure in CloudFormation templates, create Service Catalog products and portfolios in a central account, and share using AWS Organizations.  

2. Company is concerned about malicious attacks on RDP and SSH ports for remote access to EC2.  
   - Deploy Systems Manager Agent and use Session Manager.  

3. Development team requires a method of deploying applications using CloudFormation. Developers typically use JavaScript and TypeScript.  
   - Define resources in JavaScript and TypeScript and use AWS Cloud Development Kit (CDK) to create CloudFormation templates.  

4. Need to automate the process of updating an application when code is updated.  
   - Create a CodePipeline that sources code from CodeCommit and uses CodeBuild and CodeDeploy.  

5. Need to safely deploy updates to EC2 through a CodePipeline. Resources defined in CloudFormation templates and app code stored in S3.  
   - Use CodeBuild to automate testing, use CloudFormation change sets to evaluate changes, and CodeDeploy using blue/green deployment patterns.  

6. Company currently uses Chef cookbooks to manage infrastructure and is moving to the cloud. Needs to minimize migration complexity.  
   - Use AWS OpsWorks for Chef Automate.

# Compute, Auto Scaling, and Load Balancing

1. Amazon EC2 Pricing Options  
   1. On-Demand  
      1. Standard rate - no discount; no commitments; dev/test, short-term, or unpredictable workload  
   2. Reserved  
      1. 1 or 3 years commitment; up to 75% discount; steady-state, predictable workload and reserved capacity  
   3. Spot Instances  
      1. Get discount of up to 90% for unused capacity. Can be terminated at any time  
   4. Dedicated Instances  
      1. Physical isolation at the host hardware level from instances belonging to other customers; pay per instance. This means other customers are not using the same hardware on which your EC2 is running; however, other EC2 instances in your account can share those instances.  
   5. Dedicated Hosts  
      1. Physical servers for your use; socket/core visibility, host affinity; pay per host; workload with server-bound software licenses  
   6. Savings Plan  
      1. Commitment to a consistent amount of usage (EC2 + Fargate + Lambda); pay by $/hour for 1 or 3 years commitment  

2. ### Amazon EC2 Billing  
   1. Billed per second with a minimum of 1 minute  
      1. Per-second billing is for Amazon Linux and Ubuntu in On-Demand, Reserved, and Spot forms  
   2. Billed per hour minimum  
      1. Commercial Linux distros such as Red Hat EL, SUSE ES use hourly pricing  
   3. What about EBS volumes attached to instances?  
      1. They are billed per second with a minimum of 1 minute  

3. ### Reserved Instances for 1 or 3 years  
   1. Standard RI:  
      1. Change AZ, instance size (Linux), networking type  
   2. Convertible RI:  
      1. Change AZ, instance size (Linux), networking type  
      2. Change the family, OS, tenancy, payment options  
   3. Scheduled RI:  
      1. Match capacity reservation to a recurring schedule  
      2. Minimum 1200 hours per year  
      3. Example: Reporting app that runs 6 hours a day, 4 days a week = 1248 hours per year  
   4. Use ExchangeReservedInstance API  
   5. Attributes required to match for reserved instance discount:  
      1. Tenancy: Default or dedicated  
      2. When the attributes of a used instance match the RI attributes, the discount is applied  
      3. Reserved capacity in specified AZ  
      4. Region does not reserve capacity; discount applies across all AZs  

4. ### Savings Plan  
   1. Compute Savings Plan:  
      1. 1 or 3 years; hourly commitment to usage of Fargate, Lambda, and EC2; Any region, family, size, tenancy, and OS  
   2. EC2 Savings Plan:  
      1. 1 or 3 years; hourly commitment to usage of EC2 within a selected region and instance family; any size, tenancy, and OS  

5. ### Spot Instances  
   1. One or more EC2 instances  
   2. Spot Fleet: Launches and maintains the number of Spot/On-Demand instances to meet specified target capacity  
   3. EC2 Fleet: Launches and maintains specified number of Spot/On-Demand/Reserved instances in a single API call  
   4. Can define separate On-Demand/Spot capacity targets, bids, instances, types, and AZ  
   5. 2-minute warning if AWS needs to reclaim capacity - available via instance metadata and CloudWatch Events  
   6. Spot Block  
      1. Requires uninterrupted usage for 1-6 hours  
      2. Pricing 30%-40% less than On-Demand  

6. Dedicated Instances and Dedicated Hosts  
   | Characteristics | Dedicated Instances | Dedicated Hosts |  

7. ### Amazon EC2 Pricing Use Cases:  
   1. On-Demand - Developer working on a small project for several hours; cannot be interrupted  
   2. Reserved - Steady-state, business-critical line-of-business application; continuous demand  
   3. Spot Instances - Compute-intensive, cost-sensitive distributed computing; can withstand interruption  
   4. Steady state - Business-critical, line-of-business application; continuous demand  
   5. Scheduled Reserved - Reporting application, runs for 6 hours a day, 4 days per week  
   6. Dedicated Host - Security-sensitive application requires dedicated hardware; pay per instance  

8. ### AMI and Bootstrapping  
   1. How to launch EC2 instance and install dependencies, application software, security updates, and configure customs quickly?  
   2. You create an OS AMI with the following customization:  
      1. OS customization  
      2. Application dependencies  
      3. Application and configuration  
      4. Software and security updates  
   3. You can use USER DATA for faster boot  
   4. You can also use automation and configuration tools:  
      1. AWS CloudFormation  
      2. AWS OpsWorks  
      3. AWS Systems Manager  
      4. AWS CodePipeline, CodeDeploy, etc.  
      5. Chef and Puppet  
      6. Jenkins  

9. EC2 Placement Group  
   1. Cluster - pack instances close together inside an availability zone. This strategy enables workload to achieve low latency network performance necessary for tightly coupled node-to-node communication typical of HPC applications.  
   2. Partition - spread your instances across logical partitions so groups of instances in one partition do not share underlying hardware with groups in different partitions. Used by large distributed and replicated workloads such as Hadoop, Cassandra, and Kafka.  
   3. Spread - strictly place a small group of instances across distinct underlying hardware to reduce correlated failures.  

10. EC2 Placement Group Use Cases  
    - Cluster - Tightly coupled application that requires low latency, high throughput network traffic between instances  
    - Partition - Distributed and replicated NoSQL database; requires separate hardware for node group  
    - Spread - Small number of critical instances that should be kept separate from each other  

11. ### Network Interface  
    1. Network interface is also known as network adapter  
    2. There can be multiple adapters connected to instances  
    3. EC2 instances:  
       1. Eth0 - primary network interface with private and public IP  
       2. You cannot attach ENI from subnets in different AZs  
       3. Additional ENIs can be attached from subnets within the same AZ  
    4. **Elastic Network Interface (ENI)**  
       1. Basic adapter type when no high performance requirements  
       2. Can be used with all instance types  
    5. **Elastic Network Adapter (ENA)**  
       1. Enhanced networking performance  
       2. High bandwidth and lower inter-instance latency  
       3. Must be chosen on supported instance types  
    6. **Elastic Fabric Adapter (EFA)**  
       1. Used with high performance computing, MPI, and ML use cases  
       2. Tightly coupled applications  
       3. Can be used with all instance types  

12. ### Public, Private, and Elastic IP Addresses  
    1. Elastic IP will not change  
    2. Both ENI and EIP can be remapped to different instances  
    3. Elastic IP can be remapped across AZs  
    4. Elastic Network Interface cannot be remapped; they are within the availability zone  

13. ### NAT for Public Addresses  

14. ### Advanced Auto Scaling  
    1. Instance metrics are not counted until warm-up time has expired  
    2. AWS recommends scaling on metrics with a 1-minute frequency  
    3. Target-tracking metrics:  
       1. ASGAverageCPUUtilization - Average CPU utilization of auto scaling group  
       2. ASGAverageNetworkIn - Average number of bytes received on all network interfaces  
       3. ASGAverageNetworkOut - Average number of bytes sent on all network interfaces  
       4. ALBRequestCountPerTarget - Number of requests completed per target in an Application Load Balancer target group  

15. ### Dynamic Scaling - Target Tracking SQS  
    1. You can scale based on the number of messages in the queue based on custom metrics  

16. ### Dynamic Scaling - Simple Scaling  
    1. Scaling based on alarm and then 300 sec cooldown period to launch new instances  

17. ### Dynamic Scaling - Step Scaling  
    1. Depends on the size of the alarm breach  
    2. When CPU usage is >= 60%, launch 1 EC2 instance  
    3. When CPU usage is == 80%, launch 4 EC2 instances  

18. ### Scheduled Scaling  
    1. Launch set number of instances at a particular time  
    2. ASG keeps you at your desired capacity  

19. ### Scaling Process  
    1. Launch - Adds a new EC2 instance to an Auto Scaling group  
    2. Terminate - Removes an EC2 instance from the Auto Scaling group attached to ELB target group  
    3. AddToLoadBalancer - Accept notification from CloudWatch alarm associated with group’s scaling policies  
    4. AlarmNotification - Accept notification from CloudWatch alarm associated with group's scaling policies  
    5. AZRebalance - Balances the number of EC2 instances in the group across specified Availability Zones  
    6. HealthCheck - Checks instance health and marks as unhealthy if EC2 or ELB reports unhealthy to Auto Scaling  
    7. ReplaceUnhealthy - Terminates unhealthy instances and creates new ones to replace them  
    8. ScheduledActions - Performs scheduled actions  

20. ### Additional Scaling Settings  
    1. Cooldown - Used with scaling policy to prevent launching or terminating before effects of previous activities are visible. Default: 300 seconds (5 mins)  
    2. Termination Policy - Controls which instance terminates first during scale-in events  
    3. Termination Protection - Prevents auto scaling from terminating protected instances  
    4. Standby State - Used to move an instance from InService to Standby for update or troubleshooting  

21. ### Lifecycle Hook  
    1. Used to perform custom actions by pausing instances as ASG launches or terminates them  

22. ### Types of Load Balancer  
    1. **Application Load Balancer (ALB)**  
       1. Operates at the request level (Layer 7 HTTP/HTTPS)  
       2. Routes based on request content  
       3. Supports path-based and source IP address-based routing  
       4. Supports instances, IP addresses, Lambda functions, and containers as targets  
    2. **Network Load Balancer (NLB)**  
       1. Operates at the connection level (Layer 4)  
       2. Does not support security groups  
       3. Routes connection based on IP protocol data  
       4. Offers ultra high performance, low latency, and TLS offloading at scale  
       5. Can have static IP or Elastic IP  
       6. Supports UDP and static IP addresses as targets  
    3. **Classic Load Balancer**  
       1. Older generation; not recommended for new applications  
       2. Performs routing at Layer 4 and Layer 7  
       3. Used for existing applications running in EC2-Classic  
    4. **Gateway Load Balancer**  
       1. Used in front of virtual appliances such as firewalls, IDS/IPS, and deep packet inspection systems  
       2. Operates at Layer 3 - Listener for all packets on all ports  
       3. Forwards traffic to targets specified in listener rules  
       4. Exchanges traffic with appliances using the Geneve protocol on port 6081  

23. ### Routing Application Load Balancer  
    1. Assume 3 target groups: TG1, TG2, TG3  
    2. A rule is configured on the listener (ALB listens on HTTP/HTTPS)  
    3. Path-based routing:  
       - https://example.com/specials → TG1  
       - https://example.com/order → TG2  
    4. Host-based routing:  
       - https://members.example.com/ → TG3  
    5. Target groups can be EC2 instances, IP addresses, Lambda functions, or containers  

24. ### Network Load Balancer Routing  
    1. NLB nodes can have Elastic IP in each subnet  
    2. NLB creates nodes in each subnet with Elastic IPs  
    3. NLB listens to TCP, TLS, UDP, or TCP_UDP  
    4. https://example.com routes to TG1  
    5. https://example.com:8080 (separate listener on a unique port required) routes to TG2  
    6. Targets can be EC2 instances or IP addresses  
    7. Targets can also be outside of VPC e.g. on-premises  

25. ### What is the source IP address the app sees?  
    1. When an NLB with a VPC endpoint or AWS GA, source IP is private IP of NLB nodes  
    2. Application to TCP and TLS - for UDP and TCP_UDP use should be IP= A  
    3. **X-Forwarded-For header can be used with ALB to capture client IP**  

26. ### SSL/TLS Certificates  
    1. What happens at ALB:  
       1. With ALB a new connection is established with the instance  
       2. ACM certificates imported into ACM or IAM  
       3. Self-signed certificates can be used  
    2. What happens at NLB:  
       1. Single encrypted connection  
       2. Public certificate must be used  
       3. Two options:  
          1. AWS NLB - single encrypted connection with certificate at EC2  
          2. AWS NLB with two certs - one installed on NLB and one at EC2; hence two encrypted connections  

27. ### Session State and Session Stickiness  
    1. Storing session data in DynamoDB  
    2. If we need to store data, we might have to store it outside EC2  
    3. Session data such as authentication details stored in DynamoDB table  
    4. Session data retrieved from DynamoDB table  
    5. User does not need to re-authenticate  
    6. Sticky session - using cookie  
       1. Cookie is generated and client bound to instance for cookie lifetime  
       2. Session data such as authentication details stored locally  
       3. If an instance fails, session state is lost - use with session state store for more resiliency  
       4. Client is directed to another instance  

28. ### AWS Batch  
    1. You launch a job that can be a shell script, executable, or Docker container image  
    2. For that you need a "job definition"  
    3. A job is submitted to a queue until scheduled onto a compute environment  
    4. Batch launches, manages, and terminates resources as required (EC2 and ECS/Fargate)  
    5. Managed or unmanaged resources used to run the job  

29. ### AWS LightSail  
    1. Very similar to EC2  
    2. Low cost and ideal for users with less technical expertise  
    3. Compute, storage, and network  
    4. Preconfigured virtual servers  
    5. Virtual servers, databases, and load balancers  
    6. SSH and RDP access  
    7. Can access Amazon VPC  
    8. Exam tip: Typically comes up in use cases where an easy method of deploying a virtual server is required by a user with little or no expertise  

30. ### Architecture Patterns - Compute  

    1. High availability and elastic scalability for web servers.  
       Use Amazon EC2 Auto Scaling and an application load balancer across multiple AZs.  

    2. Low-latency connection over UDP to a pool of instances running a gaming application.  
       Use a network load balancer with UDP listener.  

    3. Client needs to whitelist static IP address for a highly available load balancer application in an AWS Region.  
       Use an NLB and create static IP addresses in each AZ.  

    4. Application on EC2 in an Auto Scaling group requires disaster recovery across regions.  
       Create an ASG in a second region with capacity set to 0. Take snapshots and copy them across regions (Lambda or DLM).  

    5. Application on EC2 must scale in large increments if a big increase in traffic occurs, compared to small increases in traffic.  
       Use Auto Scaling with "step scaling policy" and configure a large capacity increase.  

    6. Need to scale EC2 instances behind an ALB based on number of requests completed by each instance.  
       Configure a target tracking policy using the ALBRequestCountPerTarget metric.  

    7. Need to run large batch computing jobs at the lowest cost. Must be managed. Nodes can pick up where others left off in case of interruption.  
       Use a managed AWS Batch job and use EC2 spot instances.  

    8. A tightly coupled high-performance computing workload requires low latency between nodes and optimized network performance.  
       Launch EC2 instances in a single AZ in a cluster placement group and use an Elastic Fabric Adapter (EFA).  

    9. Line-of-business application receives weekly bursts of traffic and must scale for short periods. Needs most cost-effective solution.  
       Use reserved instances for minimum required workload and then use spot instances for bursts in traffic.  

    10. Application must start up when launched by ASG but requires app dependencies and code to be installed.  
        Create AMI that includes the application dependencies and code.  

    11. Application runs on EC2 behind an ALB. Once authenticated, users should not need to reauthenticate if an instance fails.  
        Enable sticky sessions for the target group or alternatively use a session store such as DynamoDB.

# Amazon Storage Services

### AWS EBS Deployment
1. Limited support for attaching multiple instances  
2. EBS volumes are replicated within the AZ  
3. EC2 instances must be in the same AZ as the EBS volume  

   #### EBS Volume Multi-Attach
   1. Must be provisioned IOPS io1 volume  
   2. Available for Nitro system-based EC2 instances  
   3. Up to 16 instances can be attached to a single volume  
   4. Must be in a single AZ  

### EBS Copying, Sharing, and Encryption Use Cases
1. Snapshots are stored in a region in S3  
2. You can attach the snapshot to an EC2 of a different AZ  
3. You can also take a snapshot, create an AMI, and launch a new AMI  
4. When you have a volume to snapshot — encryption state retained  
5. When you have a snapshot → copy snapshot and encrypt the snapshot — can be encrypted and change region  
6. When you have an unencrypted snapshot you can encrypt the volume — can be encrypted, can change AZ  
7. Unencrypted snapshot → you can create an AMI → cannot be encrypted. Can be shared with other accounts and can be shared publicly  
8. When you have a snapshot — can change encryption keys, can change regions  
9. You have an encrypted snapshot → encrypted AMI can be shared with other accounts (custom keys only), cannot be shared publicly  
10. Encrypted AMI → copy to encrypted AMI, can change encryption keys, can change regions  
11. Encrypted AMI → create an EC2 instance → can change encryption keys, can change AZ  
12. Unencrypted AMI → EC2 instance, can change encryption state, can change AZ  
13. Encrypted Snapshot → encrypted volume → can be encrypted, can change AZ  

## EBS vs Instance Store
1. Instance store volumes are physically attached to the host  
2. EBS volumes are attached over the network  

   ### Instance Store
   1. Instance store volumes are high-performance local disks that are physically attached to the host computer on which an EC2 instance runs  
   2. If your instance is running on instance store you cannot stop the instance, you can only restart it  
   3. Instance store is ephemeral which means that data is lost when powered off (non-persistent)  
   4. Instance store is ideal for temporary storage of information that changes frequently such as buffers, caches, or scratch data  
   5. Instance store volume root devices are created from AMI templates stored on S3  
   6. Instance store volume cannot be detached or re-attached  

## EFS Refresher
1. File management — NAS devices are file-based storage systems  
2. EFS is Linux only  
3. Instances from different VPCs can connect to an EFS file system  
4. efs-mnt is the mount point for the file system  
5. Can simultaneously connect thousands of instances  
6. Can connect to other accounts as well  
7. You can also connect on-premises computers via VPN or Direct Connect  
8. NFS protocol is used and only available for Linux systems  
9. Can connect instances from a different region (or different account) via VPC peering — mount using mount target IP address (no DNS)  

## S3
1. S3 is a key-value store  
2. Objects are accessed using HTTP protocol with REST API (GET, PUT, POST, SELECT, DELETE)  
3. You can store any type of file in S3  
4. Files can be anywhere from 0 bytes to 5TB  
5. S3 is a universal namespace so bucket names must be unique globally  
6. However, you can create your bucket within a "REGION"  
7. It is best practice to create buckets in regions that are physically closest to your users to reduce latency  
8. There is no hierarchy for objects within a bucket  
9. Delivers strong read-after-write consistency  
10. Folders can be created within folders  
11. Buckets cannot be created within other buckets  
12. An object consists of:  
   - Key  
   - Version  
   - Value  
   - Metadata  
   - Subresources  
   - Access control information  
13. S3 is in AWS public space, it is not within a VPC, it has a public endpoint  
14. It always has access to the internet  
15. **We have an S3 Gateway Endpoint** used to access EC2 instances without internet  

## Durability and Availability
1. **Durability** is protection against:  
   - Data loss  
   - Data corruption  
   - S3 offers 11 9s durability (99.999999999)  
   - If you store 10 million objects then you can expect to lose one object every 10,000 years  

2. **Availability**  
   - Measurement of the amount of time the data is available  
   - Expressed as percent of time per year  
   - Example: 99.99%  
   - S3 One Zone-IA is least durable, every other class is 99.99%  

3. **S3 Lifecycle Management**  
   - 3 types of actions:  
     - Transition Action → defines when objects transition to another storage class  
     - Expiration Action → defines when objects expire (deleted by S3)  
   - Can create lifecycle policy through the console or CLI/API  
   - When configured using CLI/API, an XML or JSON file must be supplied  
   - API actions to create/update/delete lifecycle policies:  
     - **PutBucketLifecycleConfiguration** — create or replace lifecycle configuration  
     - **GetBucketLifecycleConfiguration** — return lifecycle configuration info  
     - **DeleteBucketLifecycle** — delete the lifecycle configuration from the specified bucket  

### Storage Class Transitions
You cannot transition from:  
1. Any storage class to S3 Standard  
2. Any storage class to Reduced Redundancy Storage (RRS)  
3. S3 Intelligent-Tiering to S3 Standard-IA  
4. S3 One Zone-IA to S3 Standard-IA or S3 Intelligent-Tiering  

## Versioning
1. Versioning keeps multiple variants of an object in the same bucket  
2. Use versioning to preserve and restore every version of every object stored in an S3 bucket  
3. Versioning-enabled buckets let you recover objects from accidental deletion or overwrite  

## Replication
1. Versioning is a dependency for replication  
2. Cross-Region Replication (CRR)  
3. Same-Region Replication (can be in different accounts)  

## Encryption
1. All S3 buckets have encryption configured by default  
2. All new objects uploaded to S3 are automatically encrypted  
3. No additional cost and no impact on performance  
4. Objects are automatically decrypted in S3. For bulk operations, you can use S3 Batch Operations  

### Presigned URL
Used for short-period secure sharing  

### Server Access Logging
Monitors requests for access transparency  

## Storage Gateway
1. Gateways are deployed on-premises (VMware, Hyper-V, KVM, or hardware appliance) and then connected to AWS Storage Gateway  
   - File Gateway  
   - Volume Gateway  
   - Tape Gateway  

2. Data is encrypted in transit  

3. AWS services connected:  
   - S3 → lifecycle → S3 storage classes  
   - S3 → Backup → EBS  
   - Tape Library → Tape Archives  

### File Gateway
1. File system mounted using NFS or SMB  
2. Local cache provides low-latency access to recently used data  
3. A virtual gateway appliance runs on Hyper-V, VMware, or EC2  
4. Files are stored as objects in S3  
5. File Gateway provides a virtual on-premises file server  
6. Stores and retrieves files as objects in S3  
7. Used with on-premises apps and EC2 apps needing S3 file storage for object-based workloads  
8. File Gateway offers SMB or NFS access to S3 with local caching  

### Volume Gateway
1. **Cached Volume Mode**  
   - Cache of recently used data on-premises with the rest in S3  
   - Uses iSCSI protocol (block-based storage)  

2. **Stored Volume Mode**  
   - Entire dataset stored on-premises  
   - Backup goes to S3  
   - Asynchronous replication  
   - Data backed up as point-in-time snapshots  

### Tape Gateway
1. Backup servers on-premises can connect with many common backup applications  
2. Uses iSCSI protocol to connect to a virtual tape library on Storage Gateway  
3. Backup data is written to S3  
4. Once tapes are ejected from backup, they are stored in one of these classes  
5. S3 Standard is used when writing to tapes  
6. Used for backup with popular software  
7. Each gateway is preconfigured with media changer and tape drives (supported by NetBackup, Backup Exec, Veeam, etc.)  
8. Virtual tape sizes: 100GB, 200GB, 400GB, 800GB, 1.5TB, 2.5TB  
9. Tape Gateway can have up to 1500 virtual tapes with a maximum aggregated capacity of 1PB  
10. Data transferred between gateway and AWS is encrypted using SSL  
11. Data stored by Tape Gateway in S3 is encrypted server-side with SSE-S3  


# DNS Caching and Performance Optimization

### Route 53 Hosted Zones
1. A hosted zone represents a set of records belonging to a domain (Public hosted zone)  
2. Private Hosted Zone  
   1. You have your domain name -- mycompany.local on Route 53  
   2. We have resources on public and private subnets in our VPC  
   3. We then associate our Domain name with VPC  
   4. DNS Hostname = Enable  
   5. DNS Resolution = Enable  
   6. Records are created in Route 53  
   7. App server asks for the address of the DB  
   8. Route 53 forwards the IP  

### Migration to/from Route 53 (Super important)
1. You can migrate from another DNS provider and import records  
2. You can migrate from another DNS provider and import records  
3. You can migrate from Route 53 to another AWS account  
4. You can also associate a Route 53 hosted zone with a VPC in another account  
   1. Authorize association with VPC in a second account  
   2. Create an association in the second account  

### Route 53 Routing Policy
1. **Simple** — DNS response providing the IP address associated with a name  
2. **Failover (important)** — If the primary is down (based on health check), route to secondary destination  
3. **Geolocation** — Uses the geographic location of the user (e.g., Europe) to route to the closest region  
4. **Geoproximity** — Routes you to the closest region within a geographic area  
5. **Latency (important)** — Directs you based on the lowest latency route to resources  
6. **Multivalue Answer** — Returns several IP addresses and functions as a basic load balancer  
7. **Weighted** — Uses relative weights assigned to resources to determine routing  

### AWS Resolver Route 53
1. **Forwarder** — if it can't find the query then it will forward it  

### CloudFront Origin and Distribution
1. CloudFront has the following origins:  
   - Amazon S3 bucket and S3 bucket configured as website  
   - Elastic Load Balancer (ELB)  
   - Network Load Balancer  
   - Classic Load Balancer  
   - Amazon EC2  
   - Custom origin — HTTP/HTTPS server  
     - Own web server  
     - On-premises server  
     - Other AWS services that connect via HTTP/HTTPS  
   - Lambda function URLs  
   - API Gateways  
   - AWS AppSync GraphQL API  
   - AWS Global Accelerator endpoint  
   - AWS Elemental MediaPackage channel  
   - AWS Elemental MediaStore container  

2. Content is pushed to the origins and cached  
3. Edge locations are distributed around the world  
4. Users are directed to the nearest edge location by default  

### CloudFront Origins and Distribution
1. S3 static website can also be an origin  
2. RTMP distributions were discontinued, so only web distributions are available  
3. Behaviors of CloudFront  
   - Path patterns  
   - Viewer protocol policy  
   - Cache policy  
   - Origin request policy  

### CloudFront Web Distribution
1. Speeds up distribution of static and dynamic content (e.g., .html, .css, .php, graphic files)  
2. Distributes media files using HTTP or HTTPS  
3. Add, update, or delete objects, and submit data from web forms  
4. Use live streaming to stream an event in real time  

### CloudFront Caching
1. You have an origin (EC2 or an S3 bucket)  
   - There are 12 regional edge caches  
   - There are 210 edge locations  
   - User connects to the edge location (Point of Presence)  
   - Once connected, they use the AWS Global Network  

2. If there is a miss in the edge location, the request moves to a regional edge location and then back to the origin  
3. Object is cached for TTL (default 24 hrs)  
4. Decreasing the TTL is best for dynamic content  
5. Increasing the TTL is better for performance (reduces load on origin)  
6. You can define maximum TTL and default TTL  
7. TTL is defined at the behavior level  
8. This can be used to define different TTLs for file types (.png, .jpg)  
9. After expiration, CloudFront checks origin for new requests (latest version)  
10. Headers can be used to control the cache:  
    - **Cache-Control max-age=(seconds)** — Specify how long before CloudFront gets the object again from origin  
    - **Expires** — Specify an expiration date and time  

#### CloudFront Path Patterns
- Path patterns determine where to send requests  
  - Behavior — .jpg → origin 1  
               .mp4 → origin 2  
               Default → origin 1  
  - HTTP GET beach.jpg  
  - HTTPS GET clip.mp4  
- The default origin is used for any request that does not match a path pattern  

#### Caching Based on Request Headers
1. Configure CloudFront to forward headers in the viewer request to origin  
2. CloudFront can then cache multiple versions of an object based on header values  
3. Controlled in a behavior:  
   - Forward all headers to origin (object not cached)  
   - Forward a whitelist of headers  
   - Forward a whitelist of headers (and don’t cache objects based on values in headers)  

### CloudFront Signed URLs
1. Signed URLs provide more control over access to content  
2. Can specify beginning and expiration date and time, IP address  
3. Signed URLs should be used for individual files and clients that don’t support cookies  
   - Example: Mobile app uses a signed URL to access distribution → authenticated → request CloudFront → CloudFront sends data  

### CloudFront Signed Cookies
1. Similar to signed URLs  
2. Use signed cookies when you don’t want to change URLs  
3. Use signed cookies to provide **access to multiple restricted files**  

### Origin Access Identity (OAI)
1. CloudFront allows only a special type of user called **Origin Access Identity (OAI)**  
2. S3 bucket configured as static website allows only OAI access via bucket policy  
3. Direct access to bucket fails without OAI  
4. OAI has been deprecated — AWS now uses **Origin Access Control (OAC)**  

### Origin Access Control (OAC)
1. Like OAI but supports additional use cases  
2. AWS recommends using OAC instead of OAI  
3. Requires an S3 bucket policy that allows CloudFront service principals  

### CloudFront SSL/TLS and SNI
1. Certificates for CloudFront must be issued in **us-east-1**  
2. Default CloudFront domain name can be changed using CNAMEs  
3. Certificate can be ACM or from a trusted third-party CA  
4. S3 has its own certificate (cannot be changed)  
5. Certificates can be ACM (ALB) or third-party (EC2)  
6. Origin certificate must be a public certificate  

#### Server Name Indication (SNI)
1. Request URL includes domain name that matches certificate  
2. Multiple certificates share the same IP with SNI  
3. SNI works with browsers/clients after 2010, otherwise requires a dedicated IP  

### Lambda@Edge
1. Run Node.js and Python Lambda functions to customize content  
2. Executes closer to the viewer  
   - Can be run at the following points:  
     - After CloudFront receives request from viewer (Viewer request)  
     - Before CloudFront forwards request (Origin request)  
     - After CloudFront receives response from origin (Origin response)  
     - Before CloudFront forwards response to viewer (Viewer response)  

# RDS

### Scaling UP (Vertically)
1. Increase the instance size  
2. Horizontal  
   1. RDS master for standby if the primary fails  
   2. Scaling — you have read replicas — READ performance only  
   3. App server only writes to the master  
3. Multi-AZ Deployment  
   1. Synchronous replication — highly durable  
   2. Only the database engine on the primary instance is active  
   3. Automated backups are taken from standby  
   4. Always spans two availability zones within a single Region  
   5. Database engine version upgrades happen on primary  
   6. Automatically fails over to standby when a problem is detected  
4. Read Replica  
   1. Asynchronous replication — highly scalable  
   2. All read replicas are accessible and can be used for read scaling  
   3. No backup configured by default  
   4. Can be within an Availability Zone, Cross-AZ, or Cross-Region  
   5. Database engine version upgrade is independent from source instances  
   6. Can be manually promoted to a standalone DB instance  

### Amazon RDS Automated Backup
1. "Backup" is retained for 0–35 days  
2. You can restore the snapshot and recreate a new DB  
3. Backs up the entire DB instance, not just individual databases  
4. For single AZ DB instance there is brief suspension of I/O  
5. For multi-AZ SQL, I/O activity is briefly suspended on primary  
6. For Multi-AZ MariaDB, MySQL, Oracle, and PostgreSQL the snapshot is taken from the standby (no suspension)  
7. Snapshots do not expire (no retention period)  

### Maintenance Window
1. Operating system and DB patching can require taking the database offline  
2. These tasks take place during a maintenance window  
3. By default a weekly maintenance window is configured  
4. You can choose your own maintenance window  

---

## Secure RDS
1. Run in the VPC of your choice  
2. They can have public and private IPs  
3. Specify a security group  
4. SSL/TLS encryption in transit  
5. We can enable encryption at actual data at rest  
6. Any snapshot will be encrypted at rest  
7. You can only enable encryption for an Amazon RDS DB instance when you create it, not after the instance is created  
8. Encrypted DB instances cannot be modified to disable encryption  
9. Uses AES-256 encryption and encryption is transparent with minimal performance impact  
10. RDS for Oracle and SQL Server also supports Transparent Data Encryption (TDE) which may have performance impact  
11. AWS KMS is used for managing encryption keys  
12. You can't have an encrypted read replica of an unencrypted DB instance or an unencrypted read replica of an encrypted instance  
13. The read replica inherits the same encryption status as the main database  
14. The same key is used in the same region as the master  
15. If the read replica is in a different region, a different key is used  
16. You can't restore an unencrypted backup or snapshot to an encrypted DB instance  

**How to Encrypt Database When It Is Not Encrypted (Important)**  
1. You have its EBS volume  
2. Create a snapshot  
3. Copy the snapshot  
4. Restore from snapshot  
5. RDS encrypted — new instance with new endpoint  

---

## Aurora
1. High Performance and Scalability — Offers high performance, self-healing storage that scales up to 64TB, point-in-time recovery, and continuous backup to S3  
2. DB Compatibility — Compatible with MySQL and PostgreSQL databases  
3. Aurora Replica — In-region read scaling and failover target — up to 15 (supports auto scaling)  
4. MySQL Read Replica — Cross-region cluster with read scaling and failover target — up to 5 (each with up to 15 Aurora replicas)  
5. Global Database — Cross-region cluster with read scaling (fast replication/low latency reads). Can remove secondary and promote  
6. Multi-Master — Scales out writes within a region. In preview (not on exam)  
7. Serverless — On-demand, auto-scaling configuration. Does not support read replica or public IP (access only via VPC or Direct Connect, not VPN)  

---

# Amazon Aurora Replica
1. Number of replicas: 15  
2. Replication type: Asynchronous (milliseconds)  
3. Performance impact on primary is low  
4. Replication location is in-region  
5. Acts as a failover: Yes  
6. Automated failover: Yes  
7. Support for user-defined replication delay: Yes  
8. Support for different data/schema vs primary: No  

---

# MySQL Replicas
1. Support for user-defined replication delay: Yes  
2. Support for different data/schema vs primary: Yes  
3. Number of replicas: 15  
4. Replication type: Asynchronous (milliseconds)  
5. Performance impact on primary is low  
6. Replication location is in-region  
7. Acts as failover: Yes  
8. Automated failover: Yes  

---

## Aurora Fault Tolerance and Replication
1. Fault tolerance across 3 AZs  
2. Single logical volume  
3. Aurora replica scales out read requests  
4. Up to 15 Aurora replicas with sub-10ms replica lag  
5. Aurora replicas are independent endpoints  
6. Can be promoted to primary or new primary created  
7. Set priorities (tiers) on replicas to control promotion order  
8. Auto-scaling for replicas supported  

---

## Cross Region with Aurora MySQL
1. MySQL replicas across region  

---

## Global Database
1. Writes happen in one region  
2. Reads happen globally  
3. Cross-region replication is always asynchronous  
4. Replication uses Aurora storage layer  
5. Database in secondary region can be promoted to full read/write in less than 1 minute  
6. Applications can connect to cluster read endpoint  
7. Secondary clusters can use write forwarding  
8. Each Aurora Capacity Unit = 2 GB memory + CPU  
9. Capacity auto-scales seamlessly  

---

## Aurora Serverless Use Cases
1. Infrequently used applications  
2. New applications  
3. Variable workloads  
4. Unpredictable workloads  
5. Development and test databases  
6. Multi-tenant applications  

---

### When Not to Use Amazon RDS
1. Anytime you need a database type other than:  
   - MySQL  
   - MariaDB  
   - SQL Server  
   - Oracle  
   - PostgreSQL  
2. When you need root access (RDS does not give root access)  
3. Requirement alternatives:  
   - Lots of large binary objects → Use **S3**  
   - Automated scalability → **DynamoDB**  
   - Name/Value data structure → **DynamoDB**  
   - Data not well structured/unpredictable → **DynamoDB**  
   - Other database platforms (IBM DB2, SAP HANA) → **EC2**  
   - Full control of database → **EC2**  

**Alternative to Amazon RDS Replication (Disaster Recovery)**  
Requirement: DR for MySQL DB, RPO = 10 min, RTO = 5 min  
1. Create a read replica in a different region  
2. Use database mirroring for MySQL DBs  
3. Promote to primary — fails RTO (>5 minutes to activate primary)  
4. Failover to mirrored DB — meets RTO (very fast failover)  

---

## ElastiCache Core Knowledge
1. Fully managed implementation of Redis and Memcached  
2. ElastiCache is a key/value store  
3. In-memory database offering high performance and low latency  
4. Can be used in front of RDS or DynamoDB  

### Use Cases
1. Data that is relatively static and frequently accessed  
2. Applications tolerant of stale data  
3. When data is slow/expensive for memory writes/reads  
4. Requires push-button scalability for memory, writes, and reads  
5. Often used for storing session state  

---

### ElastiCache Scalability
1. **Memcached**  
   - Add nodes to a cluster  
   - Scale vertically (must create a new cluster manually)  

2. **Redis**  
   - Cluster mode disabled: add replica or change node type (creates new cluster + migrates data)  
   - Cluster mode enabled:  
     - Online resharding → add/remove shards, vertical scaling  
     - Offline resharding → add/remove shard, change node type, upgrade engine (more flexible)  

---

## Amazon DynamoDB Core Knowledge
1. Fully managed NoSQL database service  
2. Key/Value store and document store  
3. Non-relational, serverless service  
4. Push-button scaling  
5. Components:  
   - Table  
   - Items  
   - Attributes (e.g., userid, orderid, book price, date)  
   - Attributes can have TTL for auto-deletion  
   - With TTL, you set per-item deletion timestamp  
   - No extra cost, does not consume RCU/WCU  

### DynamoDB Features
1. Serverless — fully managed fault-tolerant  
2. Highly Available — 99.99% SLA, 99.999% for global tables  
3. Flexible schema — great for non-structured data  
4. Horizontally scaling seamlessly  
5. DynamoDB Streams — captures ordered sequence of item-level modifications  
6. DynamoDB Accelerator (DAX) — in-memory cache for better performance  
7. Transaction options — strongly consistent or eventually consistent, supports ACID  
8. Backup, point-in-time recovery down to the second within 35 days  
9. Global Tables — multi-region, multi-master solution  

---

### DynamoDB Provisioned Capacity
1. Default setting  
2. Specify read/write per second  
3. Supports auto scaling for dynamic load  
4. Metrics:  
   - RCU (Read Capacity Unit)  
   - WCU (Write Capacity Unit)  

**RCU Examples**  
- For items ≤4KB:  
  - 1 RCU = 1 strongly consistent read/sec  
  - 2 eventually consistent reads/sec  
  - 0.5 transactional reads/sec  
- Items larger than 4KB require additional RCU  

**WCU Examples**  
- For items ≤1KB:  
  - 1 standard write/sec  
  - 0.5 transactional writes/sec (1 transactional write = 2 WCU)  

---

## DAX
1. Fully managed, highly available in-memory cache for DynamoDB  
2. Improves performance from milliseconds to microseconds  
3. Used for READ performance (not writes)  
4. Fully compatible with DynamoDB API calls  

**DAX vs ElastiCache**  
- DAX optimized for DynamoDB  
- DAX does not support lazy loading  
- ElastiCache requires app code modifications  
- ElastiCache supports more datastores  

---

## DynamoDB Global Table
1. Multi-region multi-active database  
2. Each replica table stores the same data items  
3. Application logic used to failover to replicas  

---

## Architectural Patterns

1. Relational DB → Migrate MySQL to RDS, Multi-AZ standby for HA  
2. High query traffic → Add read replica and route reads  
3. Approaching capacity → Scale up DB instance (larger storage/CPU)  
4. RDS unencrypted but requires encrypted cross-region replica → Snapshot → encrypt copy → restore → new DB → encrypted replica  
5. Aurora DB cross-region replica → Deploy Aurora MySQL replica in second region  
6. Aurora minimal sync latency read replica → Deploy Aurora Replica in another AZ in the same region  
7. Aurora app in another region needs low latency read → Use Aurora Global Database with reader endpoint  
8. Aurora multi-master required → Use Aurora Multi-Master in-region  
9. Session-store with low latency → Use ElastiCache or DynamoDB  
10. Multi-threaded in-memory unstructured data → Use ElastiCache Memcached  
11. Microsecond latency in-memory store → Use DynamoDB DAX  
12. Persistent in-memory datastore with HA → ElastiCache Redis  
13. Serverless NoSQL key-value → DynamoDB  
14. Serverless Database with MySQL/PostgreSQL → Aurora Serverless  
15. RDBMS with unpredictable patterns → Aurora Serverless  
16. Key-value DB with multi-region writes → DynamoDB Global Tables  

# Architecture Pattern — Analytics

1. Athena is being used to analyze a large volume of data based on date ranges. Performance must be optimized  
   - Store data using Apache Hive partitioning with a key based on the date  
   - Use Apache Parquet and ORC storage formats  

2. Lambda is processing streaming data from API Gateway and is generating a TooManyRequestException as volume increases  
   - Stream the data into Kinesis Data Stream from API Gateway and process in batches  

3. Lambda function is processing streaming data that must be analyzed with SQL  
   - Load the data into Kinesis Data Stream and then analyze with Kinesis Data Analytics  

4. Security logs generated by AWS WAF must be sent to a third-party auditing application  
   - Send logs to Kinesis Data Firehose and configure the auditing application using an endpoint  

5. Real-time streaming data must be stored for future analysis  
   - Ingest data into Kinesis Data Stream and then use Firehose to load the data for later analysis  

6. Company runs several production databases and must run complex queries across a consolidated dataset for business forecasting  
   - Load the data from the OLTP databases into Redshift data warehouse for OLAP  

---

# Architecture Pattern — Monitoring, Logging, and Auditing

1. Need to stream logs from Amazon EC2 instances in an Auto Scaling Group  
   - Install the unified CloudWatch Agent and collect log files in Amazon CloudWatch  

2. Need to collect metrics from EC2 instances with 1-second granularity  
   - Create a custom metric with high resolution  

3. Application logs from on-premises servers must be processed by AWS Lambda in real time  
   - Install the Unified CloudWatch Agent on the server  
   - Use a subscription filter in CloudWatch to connect a Lambda function  

4. CloudWatch log entries must be transformed with Lambda and then loaded into Amazon S3  
   - Configure a Kinesis Firehose destination  
   - Transform with Lambda and then load into an S3 bucket  

5. CloudWatch log entries must be analyzed and stored centrally in a security account  
   - Use cross-account sharing and configure a Kinesis Data Stream in the security account to collect the log files  
   - Use Lambda to analyze and store  

6. Access auditing must be enabled and records must be stored for a minimum of 5 years. Any attempt to modify the log files must be identified  
   - Create a trail in CloudTrail that stores the data in an S3 bucket  
   - Enable log file validation  

7. API activity must be captured from multiple accounts and stored in a central security account  
   - Use CloudTrail in each account to record API activity  
   - Use cross-account access to a security account to store the log files in a central S3 bucket  

8. Need to trace and debug an application with distributed components  
   - Use AWS X-Ray to trace and debug the application  

# Security 

## AWS CloudHSM

| Feature              | CloudHSM                                   | AWS KMS                                  |
|-----------------------|---------------------------------------------|-------------------------------------------|
| Tenancy              | Single-tenant HSM                          | Multi-tenant AWS Service                  |
| Availability         | Customer-managed durability and availability | Highly available and durable key storage and management |
| Roots of Trust       | Customer-managed root of trust             | AWS-managed root of trust                 |
| FIPS Level           | Level 3                                    | Level 2 / Level 3                         |
| 3rd Party Support    | Broad 3rd-party support                    | Broad AWS service support                 |

### Exam Importance
1. FIPS 140-2 Level 3 validated HSM  
2. You can configure AWS Key Management Service to use your AWS CloudHSM cluster as a custom key store rather than the default KMS key store  
3. Managed service and automatically scales  
4. Retain control of your encryption keys — you control access (AWS has no visibility of your encryption keys)  

---

## Macie
Macie enables security compliance and preventative security as follows:  
1. Identifies a variety of data types, including PII, protected health information, regulatory documents, API keys, and secret keys  
2. Identifies changes to policy and access control lists  
3. Continuously monitors the security posture of S3  
4. Generates security findings that you can view using the Macie console, AWS Security Hub, or EventBridge  
5. Fully managed data security and data privacy service  
6. Uses machine learning and pattern matching to discover, monitor, and help you protect your sensitive data on S3  

---

## AWS WAF
1. Protects against SQL injection and cross-site scripting attacks  
2. **Match Statement** compares the web request to its origin against conditions that you provide  
3. Can be put in front of a load balancer  

| Match Statement       | Description                                                             |
|------------------------|-------------------------------------------------------------------------|
| Geographic Match       | Inspect the request’s country of origin                                 |
| IP Set Match           | Inspect the request against a set of IP addresses and ranges            |
| Regex Pattern Set      | Compares regex patterns against a request component                     |
| Size Constraints       | Check size constraints against a request component                      |
| SQL Injection Attack   | Inspect for malicious SQL code in the request component                 |
| String Match           | Compares a string to a request component                                |
| XSS Scripting Attack   | Inspect for cross-site scripting attack in a request component          |

---

## AWS Shield
1. AWS Shield is a managed Distributed Denial of Service (DDoS) protection service  
2. Safeguards web applications running on AWS with always-on detection and automatic inline mitigation  
3. Helps to minimize application downtime and latency  
4. Two tiers:  
   - **Standard** — no cost  
   - **Advanced** — $3k per month and 1-year commitment  
5. Integrated with AWS CloudFront  

---

## GuardDuty
1. Intelligent threat detection service  
2. Detects account compromise, instance compromise, malicious reconnaissance, and bucket compromise  
3. Continuous monitoring for events across:  
   - AWS CloudTrail Management Events  
   - AWS CloudTrail S3 Data Events  
   - Amazon VPC Flow Logs  
   - DNS Logs  

---

## Security Architecture Pattern
1. Need to enable custom domain name and encryption in transit for an application running behind an Application Load Balancer  
   - Use AWS Route 53 to create an Alias record to the ALB’s DNS name and attach an SSL/TLS certificate issued by ACM  

2. Company records customer information in CSV files in an Amazon S3 bucket and must not store PII data  
   - Use Amazon Macie to scan the S3 bucket for any PII data  

3. For compliance reasons all S3 buckets must have encryption enabled, and any non-compliant bucket must be auto-remediated  
   - Use AWS Config to check the encryption status of the bucket and use remediation to enable as required  

4. EC2 instances must be checked against CIS benchmark every 7 days  
   - Install the Amazon Inspector agent and configure a host assessment every 7 days  

5. Website running on EC2 instances behind an ALB must be protected against well-known web exploits  
   - Create a Web ACL in AWS WAF to protect against exploits and attach to the ALB  

6. Need to block access to an application running on an ALB from connections originating in a specific list of countries  
   - Create a Web ACL in AWS WAF with a geographic match and block traffic that matches the list of countries  

# Systems Manager Patch Manager

This tool automates patching of your EC2 and on-premises servers.

**What does it handle?**

- ✅ Automatically scans for missing patches  
- ✅ Defines patch rules (baselines)  
- ✅ Groups servers for patching (Patch Groups)  
- ✅ Applies patches in a controlled, environment-aware way  

### Key Concepts — *Systems Manager Patch Manager*

1️⃣ **Patch Baseline**  
- Defines which patches should be applied  
- Example:  
  - Baseline for Dev: all non-critical + critical patches  
  - Baseline for Prod: only critical patches, with a 7-day wait  
💡 Think of it as your “approved patch rulebook” per environment  

2️⃣ **Patch Group**  
- Logical grouping of EC2 instances based on tags  
- **MUST use the tag key:** `Patch Group` (AWS requirement)  
- Example:  
  - Dev instances → Patch Group = Dev-Patch  
  - Prod instances → Patch Group = Prod-Patch  
💡 Patch Groups connect specific servers to specific baselines  

3️⃣ **Tagging**  
- You must tag EC2 instances properly:  
  - ✅ By environment  
  - ✅ By OS type (if needed)  
  - Example:  
    - Environment: Dev, Patch Group: Dev-Patch  
    - Environment: Prod, Patch Group: Prod-Patch  
💡 Without the right tags, Patch Manager can’t target the correct servers  

4️⃣ **Patch Compliance**  
After patching, you can verify:  
- What was patched  
- What failed  
- What’s missing  

---

### Dedicated Instances vs Dedicated Hosts

| Characteristics                             | Dedicated Instances | Dedicated Hosts |
|----------------------------------------------|--------------------|----------------|
| Enable the use of dedicated physical servers | X                  | X              |
| Per instance billing (subject to $2/region) | X                  |                |
| Per host billing                            |                    | X              |
| Visibility of socket, cores, host ID        |                    | X              |
| Affinity between a host and instances        |                    | X              |
| Targeted instance placement                  |                    | X              |
| Automatic instance placement                 | X                  | X              |
| Add capacity using an allocation request     |                    | X              |

---

# VPC & Networking (Peering, TGW, Direct Connect)

1️⃣ **Trap:** VPC Peering is NOT transitive  
➡️ Even if A ↔️ B and B ↔️ C, A 🚫 cannot talk to C  

2️⃣ **Trap:** Direct Connect does NOT allow VPC-to-VPC traffic  
➡️ It connects on-premises to AWS — NOT VPC-to-VPC  
➡️ Use Transit Gateway + Direct Connect Gateway for multi-VPC routing  

---

# IAM & Security (Cross-Account Access, Policies)

1️⃣ **Trap:** Resource-based policies allow cross-account access WITHOUT switching roles  
   - ➡️ Identity-based policies require `AssumeRole`  

2️⃣ **Trap:** Trust policy’s **Principal** must point to the external account/user ARN  
   - NOT your own account or role name  

---

# CloudFront & Encryption

1️⃣ **Trap:** Field-Level Encryption is needed when encrypting specific sensitive fields (e.g., credit card numbers), not just HTTPS  

2️⃣ **Trap:** Adding headers (User-Agent, Host) to the CloudFront cache key can **reduce** cache hit ratio, NOT improve it  

---

# Migration Strategies (6 R’s)

1️⃣ **Trap:** If you're migrating NOW with no code changes, it’s always **Rehost** — even if you plan to refactor later  

2️⃣ **Trap:** **Replatform** = minor tweaks now (e.g., moving DB to RDS) — NOT a full rewrite (that’s Refactor)  

🧠 **Blue/Green works** best when you can duplicate everything and switch over cleanly.  
❌ Not ideal when apps need live state coordination, feature flags, or have tight upgrade paths (like COTS apps).  

---

# Security Groups vs NACLs

1️⃣ **Trap:**  
- Security Groups are **stateful** (return traffic auto-allowed)  
- NACLs are **stateless** (must allow return traffic explicitly)  

2️⃣ **Trap:**  
- NACLs evaluate rules **in order** (lowest # first). First match WINS  
- SGs evaluate all rules together  

---

# Direct Connect & VPN

1️⃣ **Trap:** VPN is over the internet (IPSec encrypted), while Direct Connect is a dedicated private connection  

2️⃣ **Trap:** Public VIF (Virtual Interface) on Direct Connect is for **public AWS services** (e.g., S3) — NOT for private VPC access  
➡️ For VPC access, you need a **Private VIF**  

---

# S3 Access & Encryption

1️⃣ **Trap:** Bucket policies (resource-based) can grant cross-account access directly  
➡️ IAM user policies alone cannot, unless combined with `AssumeRole`  

2️⃣ **Trap: S3 encryption**  
- **SSE-S3** = AWS manages keys  
- **SSE-KMS** = You manage keys via KMS  
➡️ **SSE-KMS requires extra permissions** (e.g., `kms:Decrypt`)  

---

# IPSec VPN and VPC

1️⃣ IPSec tunnels are built for “on-prem to AWS,” not AWS VPC to AWS VPC  
2️⃣ Instead of IPSec, AWS gives you:  
- VPC Peering  
- Transit Gateway (TGW)  

➡️ These use the AWS backbone network — no need for IPSec tunnels  

| Scenario                           | Solution(s)                    | Notes                      |
|-----------------------------------|---------------------------------|-----------------------------|
| Connect on-prem ↔️ AWS (encrypted) | ✅ Site-to-Site VPN (IPSec)     | AWS Native Solution         |
| VPC ↔️ VPC (same/other region)     | ✅ VPC Peering or Transit GW    | No IPSec tunnel used        |
| Connect multiple VPCs + scale     | ✅ Transit Gateway (TGW)        |                             |
| Multi-region + on-prem combo      | ✅ Direct Connect + DX Gateway  |                             |

🚨 **Key phrase to memorize:**  
👉 AWS does NOT provide native IPSec VPN tunnels **between two VPCs** — use VPC Peering or Transit Gateway instead  

---

# AWS Config vs Systems Manager (SSM)

| AWS Config                                 | AWS Systems Manager (SSM)                                   |
|--------------------------------------------|-------------------------------------------------------------|
| 🛡️ **Think:** Continuous Compliance & Auditing | 🔧 **Think:** Managing, operating & patching infrastructure |
| Tracks and records resource configurations | Manages servers, installs patches, runs commands            |
| Checks if resources meet compliance rules  | Used to automate ops tasks across fleets                    |
| Example: Are S3 buckets public?            | Example: Patch EC2 Linux fleet, run scripts                 |
| ✅ Best for compliance, auditing, security | ✅ Best for automation, maintenance, operations             |

🚨 **Exam Trap:**  
- Compliance/auditing → **AWS Config**  
- Operational tasks → **SSM**  

Mnemonic: **Config = Compliance. SSM = SysAdmin tasks**  

---

# How to Improve Cache Hit Ratio

| Technique                         | Why It Helps                                                                 |
|-----------------------------------|-------------------------------------------------------------------------------|
| Cache-Control max-age (long TTL)  | Longer CloudFront cache = fewer origin fetches                               |
| Minimize headers/query strings    | Unique requests reduce cache ratio                                           |
| Compress objects (gzip/brotli)    | Smaller size = faster, better cache reuse                                    |
| Use Origin Shield                  | Extra caching layer closer to origin                                         |
| Versioned URLs (e.g., /v1/file.js)| Cache “forever” + force refresh on URL change                                |

---

# AWS Security Services — Use Cases

| Use Case                                   | Service                | Status          |
|--------------------------------------------|------------------------|-----------------|
| Infra-layer DDoS protection (SYN, UDP)     | AWS Shield Advanced    | ✅              |
| Enforce config compliance                  | AWS Config             | ✅              |
| App-level threats (SQLi, XSS, bots)        | AWS WAF                |                 |
| Org-wide firewall/DDoS policy              | AWS Firewall Manager   |                 |
| Patch, automate EC2 ops                    | AWS Systems Manager    | ✅              |

---

# WAF Core Building Blocks

| Component       | What It Does                                                                 |
|-----------------|-----------------------------------------------------------------------------|
| Web ACL         | Main WAF rule group to attach (ALB, API GW, CloudFront)                     |
| Rules           | Logic like “block SQLi”, “allow US-only”, “rate limit per IP”               |
| Rule Groups     | Reusable sets of rules (AWS Managed Rule Groups available)                  |

---

# Federation Scenarios

| Scenario                        | Federation Type     | STS Call                      |
|---------------------------------|---------------------|--------------------------------|
| Enterprise users (AD/ADFS)      | ✅ SAML Federation   | AssumeRoleWithSAML            |
| Mobile/web app users (Google/FB)| ❌ Web Identity Fed. | AssumeRoleWithWebIdentity     |
| OAuth 2.0 Access                | ❌ Not for Console   | —                              |

Mnemonic:  
**SAML = Corporate**  
**Web Identity = Mobile/Web Apps**  
🔑 `STS + AssumeRoleWithSAML` = enterprise SSO login  

---

# EBS

| Data Pattern                        | Volume Type  |
|-------------------------------------|--------------|
| Random access (DB)                  | gp3/gp2, io1 |
| Sequential (logs, streaming data)   | st1          |
| Infrequent/archival/cold            | sc1          |


# EBS Volume Types — Nuances & Use Cases

| **Type** | **Performance Metric** | **Best Use Cases** | **Nuances** | **Exam Traps** |
|----------|-------------------------|--------------------|-------------|----------------|
| **gp2 (General Purpose SSD)** | IOPS/GB (3 IOPS/GB, up to 16,000 IOPS) | Boot volumes, dev/test DBs, general workloads | Older default; performance tied to size (small volumes = lower baseline) | Don’t pick gp2 if question mentions **predictable IOPS regardless of size** → gp3 |
| **gp3 (General Purpose SSD)** | Baseline: 3,000 IOPS, 125 MB/s throughput; scalable up to 16,000 IOPS / 1,000 MB/s independently of size | Modern default: boot, app servers, small-to-medium DBs, EMR clusters | Cheaper than gp2, predictable baseline even for small disks | If Q says “cost-effective, predictable IOPS” → gp3 (not gp2) |
| **io1 (Provisioned IOPS SSD)** | Provision up to 64,000 IOPS (Nitro instances) | Mission-critical DBs (Oracle, SAP, SQL Server, DynamoDB on EC2) | You pay for provisioned IOPS; can also enable Multi-Attach | Wrong if cost-efficiency or throughput is the priority |
| **io2 (Provisioned IOPS SSD, Gen2)** | Same as io1 but 99.999% durability, up to 500 IOPS/GB | Enterprise-grade, latency-sensitive, Tier-1 OLTP DBs | Higher durability SLA; fewer failures | Wrong if question only needs io1 durability |
| **st1 (Throughput Optimized HDD)** | Throughput (MB/s), up to 500 MB/s per volume | Big data, data warehouse, ETL, log processing — **frequently accessed, sequential workloads** | Cheaper than SSD, best for **streaming throughput** | Not for boot, random access, or infrequent workloads |
| **sc1 (Cold HDD)** | Throughput (MB/s), up to 250 MB/s per volume | Archival data, cold storage, infrequently accessed | Lowest-cost EBS; designed for “rarely read” | If data is accessed often, sc1 will bottleneck → pick st1 instead |

# EBS Mnemonics & Exam Shortcuts

| **Mnemonic** | **Meaning** | **Exam Shortcut** |
|--------------|-------------|--------------------|
| **GP** | General apps (gp2/gp3) | gp2 vs gp3 → gp3 is modern default (cheaper, predictable baseline, independent scaling). gp2 is legacy. |
| **IO** | IOPS-hungry DBs (io1/io2) | io1 vs io2 → both for high IOPS DBs; io2 = higher durability, SLA, newer gen. |
| **ST** | Streaming workloads (frequent) | st1 vs sc1 → both are HDD, throughput-focused; st1 = frequent big data, sc1 = infrequent cold. |
| **SC** | Cold, infrequent workloads | sc1 for archival/infrequent, lowest cost option. |


# Aurora

When performance bottlenecks, zero-downtime scaling, multi-AZ durability, or global DR with MySQL apps are mentioned, think **Aurora MySQL**.

Aurora has drop-in compatibility for two major engines:
- Aurora MySQL → MySQL-compatible  
- Aurora PostgreSQL → PostgreSQL-compatible  

Aurora pricing modes:
- **Aurora Standard** → Charged for read/write I/O operations + storage  
- **Aurora I/O-Optimized** → No charge for I/O operations, only pay for storage + compute  

---

# Scaling Compute in Amazon Aurora — 3 Options

## 1. Aurora Serverless (v2 is common in exam questions)
- **Auto-scales** compute capacity **up and down** based on workload  
- Best for **infrequent, unpredictable, or variable workloads**  
- Pay **per second** for the capacity used  
- No manual intervention — scaling is automatic  
- Exam Trigger: *“Sporadic traffic”* or *“must scale without manual changes”* → **Aurora Serverless**  

## 2. Aurora PostgreSQL Limitless Database *(PostgreSQL only)*
- **Horizontally scales** compute and storage beyond a single cluster  
- Designed for **massive scale** — millions of writes/sec, petabytes of data  
- Built for **highly concurrent OLTP workloads**  
- Exam Trigger: *“Extremely high transaction rates”* or *“multi-writer scaling” with PostgreSQL* → **Aurora Limitless**  

## 3. Manual Scaling
- Manually change **DB instance class** (upsize/downsize)  
- Requires **reboot** for instance class change  
- Best for **predictable workloads**  
- Exam Trigger: *“predictable growth”* or *“fixed workload”* → **Manual Scaling**  

### Common Exam Patterns
- Unpredictable load + cost efficiency → **Aurora Serverless**  
- Extreme scale OLTP + PostgreSQL → **Aurora Limitless**  
- Predictable growth → **Manual scaling**  

---

# CloudFront – File Expiration at Edge Locations

## Default Behavior
- If no `Cache-Control` header → Edge checks origin every **24 hours**  

## Custom Expiration
- Set `Cache-Control` headers at origin to control TTL  
- TTL can be as low as **0 seconds** or indefinitely long  

### Special Cases
- **0 seconds** → CloudFront revalidates every request with origin  
- **Long expiration** → High performance for rarely updated files  
  - Update method: Use **versioned file names** (e.g., `file_v3.css`)  

### Exam Patterns
- No header → 24-hr refresh  
- Frequent file updates → Low/zero TTL  
- Rare updates → Long TTL + versioning  

❌ *CloudFront cannot cache POST/PUT responses* (must reach origin).  
✅ OPTIONS method → can be cached.  

---

# CloudFront – HTTP Version Comparison

When setting up a distribution, you can choose supported protocols:

### 1. HTTP/1.0
- Year: 1996  
- Basic, no persistent connections  
- Rarely used today (legacy systems)  

### 2. HTTP/1.1
- Year: 1997  
- Added persistent connections, pipelining, caching improvements  
- Widely supported fallback  

### 3. HTTP/2
- Year: 2015  
- Features: Multiplexing, binary format, header compression  
- Default in CloudFront — improves performance significantly  
- Recommended for most modern apps  

### 4. HTTP/3 (QUIC-based)
- Emerging standard (UDP-based QUIC protocol)  
- Faster and more reliable (good for high-latency / lossy mobile networks)  
- Great for streaming and gaming  
- Use when scenario mentions *“unreliable mobile networks”* or *“real-time apps”*  

**Best Practice:**  
- Primary: HTTP/2  
- Fallback: HTTP/1.1  
- Consider HTTP/3 when triggered by scenario  

---

# CloudFront SaaS Manager

Feature for **managing multiple websites at scale**:  
- Ideal for **SaaS providers**, **web platforms**, or **large enterprises**  
- When **hundreds/thousands** of sub-sites exist  

### When to Use  
- Multi-tenant SaaS  
- Need **standardization** across many websites  

### When NOT to Use  
- Just a handful of websites  
- Different configs needed per site → Use individual CloudFront distributions  

### Exam Triggers  
- “Multi-tenant”  
- “SaaS hosting many websites”  
- → **Answer: CloudFront SaaS Manager**  

---

# CloudFront – Field-Level Encryption (FLE)

Encrypts **specific fields** at CloudFront before sending to origin.  

### Example
- Encrypt only **credit card numbers** or **SSNs**, not entire request  

### Benefits
- Provides extra security beyond TLS  
- Helps with compliance (PCI DSS, HIPAA)  

### Exam Triggers
- “Encrypt only certain form fields” → **CloudFront FLE**  
- Distinguish From:  
  - HTTPS/TLS = encrypts the entire request  
  - S3/RDS server-side encryption = encrypts at rest  

---

# CloudFront DDoS Protection

### Default
- **AWS Shield Standard** (free, always enabled)  
- Protects against common infra attacks: SYN/UDP Floods, reflection, etc.  

### Advanced
- **AWS Shield Advanced** ($$$)  
- Covers ELB, CloudFront, Route 53  
- Adds DDoS cost protection, attack visibility, SOC support  

### Exam Triggers
- “Protect from common L3/L4 attacks” → Shield Standard  
- “Large-scale attacks, cost protection needed” → Shield Advanced  

---

# CloudFront + AWS WAF – Web App Protection

Protects CloudFront-delivered apps from **Layer 7 application attacks** (SQLi, XSS, bots, etc.).

### How It Works
1. Attach WAF Web ACL to CloudFront  
2. Define rules to allow/block based on:  
   - IP addresses/ranges  
   - HTTP headers  
   - URI strings (e.g., block `/admin`)  
   - Rate limiting policies  

### Example Exam Scenario  
- Bot attacks at `/login` with brute-force attempts  
- Solution: Add WAF rule to block >10 failed login attempts from same IP in 5 mins  
- Result: Requests blocked **before reaching origin**  

---

# AWS Application Discovery & Migration Assessment Tools

| Tool/Service                          | When To Use                        | Data Collected | Data Usable In | Agent? |
|---------------------------------------|-------------------------------------|----------------|----------------|--------|
| **Discovery Service – Agentless**     | VMware configs, network, DB info    | VM/DB config   | Migration Hub  | No     |
| **Discovery Service – Agent**         | Process-level dependencies          | Deep process   | Athena, Hub    | Yes    |
| **Migration Evaluator**               | Guided AWS expert migration assess. | Multi-platform | Migration Hub  | No     |

### Exam Triggers
- “VMware/DB info, no installs” → **Agentless Collector**  
- “Process-level dependencies” → **Agent**  
- “AWS-guided assessment” → **Migration Evaluator**  

# AWS Storage Gateway – Tape Gateway

## Purpose
Modernize on-premises **tape backups** by moving them to **Amazon S3 + Glacier** using **virtual tapes**.

## How It Works
1. Existing backup software (e.g., Veeam, NetBackup) writes to a virtual tape (via iSCSI).  
2. Tape data is stored in Amazon S3.  
3. Archived tapes move to **Virtual Tape Shelf (VTS)** in Glacier.  

## Benefits
- Minimal changes to legacy backup processes  
- High durability (11 9s) in Glacier  
- Pay-as-you-go archival  
- Avoids physical tape management  

## Notes
- VTS = Glacier storage location for tapes  
- Retrieval requires restore first  
- ❌ Don’t use File Gateway or Volume Gateway for tape backups  

---

# Schema Conversion Tool + Database Migration Service

### Use Case: Heterogeneous migrations (Oracle → PostgreSQL, SQL Server → Aurora, etc.)

**Steps:**
1. Use **SCT** to convert schema (tables, indexes, procs).  
2. Deploy converted schema on target (RDS/Aurora).  
3. Use **DMS** to migrate data (full load + CDC).  
4. Cut over when apps are ready.  

---

# AWS DataSync

## Purpose
High-speed automated data transfer to AWS.

## Use Cases
- Migrate NFS/SMB file shares  
- Move logs, backups, archives to S3/EFS/FSx  
- Ongoing data sync  

## Key Features

| Feature              | Description                          |
|-----------------------|--------------------------------------|
| ✅ SMB/NFS Support    | Connects to on-prem file shares      |
| 🔄 Incremental Syncs  | Transfers only changed data          |
| 🚀 High Performance   | 10x faster than DIY scripts          |
| 🔐 Security + Sched   | TLS + IAM, supports cron             |

Destinations: **S3, FSx, EFS**  

---

# VM Import/Export

## Purpose
Convert on-prem VM images into **EC2 AMIs**.

## Use Cases
- Lift-and-shift VM-based apps  
- Preserve OS/app configuration  
- Create custom AMIs  

## Supported Formats  
OVA, VMDK, VHD  

## Requirements
- Upload image to S3  
- IAM role for import permissions  
- Use CLI/API for import  

---

# DataSync + VM Import/Export Combo

## Use Case
- Use DataSync for **file shares**  
- Use VM Import/Export for **full VM migration**  

### Exam Tip
✅ DataSync → file server migration  
✅ VM Import/Export → full VM lift-and-shift  

---

# Amazon SQS vs Amazon MQ

| Feature                  | Amazon SQS                         | Amazon MQ                                             |
|---------------------------|-------------------------------------|------------------------------------------------------|
| Purpose                   | Fully managed queue service        | Managed broker for migration of JMS/AMQP apps        |
| Best for                  | Cloud-native apps, decoupling      | Legacy broker-compatible workloads                  |
| Protocols                 | Proprietary AWS API only           | AMQP, MQTT, STOMP, JMS                               |
| Ordering                  | FIFO queues                        | Supports via topics + queues                         |
| Scalability               | Virtually unlimited                | Vertical (depends on broker size)                   |
| Latency                   | Milliseconds                       | Higher than SQS                                      |
| Cost Model                | Pay per request, cheap             | Broker instance hourly charges                       |
| Management Overhead       | Very low                           | Medium (broker clusters to manage)                   |

**Exam Tip**  
- Use **SQS** for decoupled, scalable apps.  
- Use **MQ** for **lift & shift JMS/AMQP** apps.  

---

# AWS Direct Connect (DX)

Dedicated, private link between **on-premises ↔️ AWS**.

- Reduces latency, avoids the internet, reliable  
- Use for SAP, Oracle DBs, large data movement  

**Why BGP?**  
Border Gateway Protocol (BGP) exchanges routes between your router and AWS DX router.  
- Without BGP, no routing  
- Supports MD5 auth for trust  

---

# AWS Snowball Edge

## Optimization Tips
- Use multiple parallel copy sessions  
- Batch small files into archives (`.zip/.tar`)  
- Avoid modifying files mid-transfer  
- Minimize switches/hops  
- Use 10/40/100 Gbps NICs (no USB)  

## Key Concepts
- Clustering = capacity & durability, NOT speed  
- File interface = file copy layer  
- S3 adapter = object interface  

**When to pick Snowball**  
- >10TB data  
- Network too slow  
- Deadline <1 month  
- Initial bulk transfer, followed by DMS for sync  

Mnemonic:  
**“Big Data, Slow Pipe → Snowball first, DMS after.”**

---

# IBM MQ vs Amazon MQ vs SQS

| Feature             | IBM MQ             | Amazon MQ                        | SQS (Amazon)           |
|---------------------|-------------------|----------------------------------|-------------------------|
| Purpose             | Enterprise broker | Managed JMS-compatible broker     | Serverless queue        |
| Protocols           | JMS, AMQP, MQTT   | JMS, STOMP, etc.                 | Proprietary only        |
| Migration           | Lift/replatform   | Drop-in replacement              | ❌ Not protocol-compatible |
| FIFO Support        | Yes               | Yes                              | Yes (FIFO queues)       |

---

# The 6 Rs of Migration

1. **Rehost (Lift & Shift)** → Migrate as-is.  
2. **Replatform (Tinker & Shift)** → Slight optimizations (e.g., DB → RDS).  
3. **Repurchase (Drop & Shop)** → Replace software, e.g., Salesforce.  
4. **Refactor/Re-architect** → Rebuild into cloud-native (microservices).  
5. **Retire** → Remove unnecessary apps.  
6. **Retain** → Keep some apps on-prem temporarily.  

**Mnemonic:** *Rehost, Replatform, Repurchase, Refactor, Retire, Retain*  

---

# CloudFormation DeletionPolicy Options

| Policy    | Behavior                                        |
|-----------|-------------------------------------------------|
| Delete    | Resource deleted with stack                     |
| Retain    | Keep resource intact after stack deletion       |
| Snapshot  | Backup created before deletion (only for EBS/RDS/etc.) |

---

# Networking Nuggets

| Component          | Function                       | Example              |
|--------------------|--------------------------------|----------------------|
| AmazonProvidedDNS  | VPC-level DNS endpoint         | VPC_CIDR + .2        |
| Route 53 Resolver  | Handles internal + external    | —                    |
| Private Zones      | Custom domains for VPCs        | db.internal.company  |
| Recursive Lookup   | Used for public Internet DNS   | google.com           |

---

# iSCSI & Storage Gateway

- iSCSI = block storage protocol over IP.  
- AWS Storage Gateway Volume/Tape gateway uses it.  

**Risks:** Replay, spoofing, MITM  
**Fix:** Enable **CHAP authentication** + TLS  

---

# EBS Volume Types

| Volume | Burstable?   | Notes                                            |
|--------|--------------|--------------------------------------------------|
| gp2    | ✅ (credits) | 3 IOPS per GB, up to 3K burst IOPS               |
| gp3    | ❌           | Fixed baseline 3000 IOPS + adjustable throughput |
| io1/io2| ❌           | Provisioned IOPS guarantee, premium performance |
| st1    | Linear       | HDD, good for streaming/logs                     |
| sc1    | Linear       | Cold HDD, archival                               |

---

# RDS High Availability & Scaling

## Multi-AZ Deployment
- Synchronous replication  
- One primary, one standby  
- Automatic failover  
- Backups are done from standby  

## Read Replica
- Asynchronous replication  
- Used for scaling reads (reporting, analytics)  
- No auto failover (manual promote required)  

## Global Database
- Cross-region reads + DR  
- Asynchronous replication  
- Regional failover (semi-manual)  

---

# FSx Variants

| Variant                   | Ideal For                                   |
|----------------------------|---------------------------------------------|
| FSx for Windows FileServer| SMB, Active Directory, Windows apps         |
| FSx for Lustre            | HPC, ML, big data                           |
| FSx for NetApp ONTAP      | SnapMirror, FlexClone, enterprises          |
| FSx for OpenZFS           | ZFS features, snapshots, cloning            |

---

# CloudFront – HTTPS + SSL

- Redirect HTTP → HTTPS at edge  
- Use CloudFront default cert or ACM cert  
- ❌ Do not use self-signed ELB certs  
- Exam Trigger: *“force HTTPS at viewer layer”*  

---

# GuardDuty + Trusted IPs

- GuardDuty only supports IP allow-listing  
- EC2 IDs not supported  
- Use **Elastic IPs** → add those IPs to Trusted IP list  

---

# ALB Target Types

| Target Type     | Notes                                     |
|-----------------|-------------------------------------------|
| EC2 Instances   | Standard target group                     |
| IP Addresses    | Within VPC subnets                        |
| Lambda          | ALB can invoke Lambda for HTTP/HTTPS req  |
| ECS Tasks       | Works with awsvpc Task Networking         |

❌ Cannot directly target RDS, DynamoDB, or arbitrary external IP  

---

# Gateway Load Balancer (GWLB)

- Operates at **Layer 3** (IP)  
- Forwards packets → security appliances (firewall, IDS/IPS)  
- Uses **GENEVE protocol** for encapsulation  
- Shared across accounts via **PrivateLink**  

**Flow Stickiness** uses **5-tuple (src/dst IP+port, protocol)**.  

---

# Exam Mnemonics & Hooks

- *“Config = Compliance, SSM = SysAdmin”*  
- *“Snowball for >10TB bulk, DMS for sync”*  
- *“Multi-AZ = HA, not scaling”*  
- *“Read Replica = scaling, not HA”*  
- *“CloudFront FLE = encrypt just fields”*  
- *“WAF at edge stops the wedge”*  

