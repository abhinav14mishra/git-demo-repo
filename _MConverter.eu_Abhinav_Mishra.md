Automated Transaction Processing System **AWS Step Functions · EC2 · ECS Fargate · GitHub**‑**Based Infrastructure as Code** GitHub Repository: 

https://github.com/abhinav14mishra/transaction-processor-automation.git 



1. Introduction 

ABC Startup requires a robust, automated, and auditable system to process daily transaction files. These files are critical for downstream business operations and must be handled reliably without human error or manual intervention. In traditional systems, such processing is often performed manually or through custom scripts, which leads to operational complexity, limited visibility, and significant audit challenges. 

To address these problems, ABC Startup designed and implemented a cloud‑native solution using Amazon Web Services \(AWS\). The system is built around an event‑driven architecture where infrastructure responds automatically to events, rather than relying on scheduled jobs or manual triggers. 

A key organizational requirement is that **all infrastructure must be created and** **managed exclusively through GitHub** using Infrastructure as Code \(IaC\). This ensures that every change is version‑controlled, reviewable, and traceable. Manual creation or modification of AWS resources through the AWS Console is strictly prohibited. 

This document provides a complete, end‑to‑end explanation of the system, written in simple language, focusing on: 

• System architecture 

• Execution flow 

• Infrastructure components 

• Security model 

• Networking design 

• Deployment strategy 

• Auditing and reliability 



2. Business Problem Statement 

The business challenges driving this solution are outlined below. 

1. Transaction files need to be processed automatically as soon as they are uploaded. 

2. The system must validate that required infrastructure dependencies are available before processing starts. 

3. Processing jobs must run in an isolated, repeatable, and short‑lived execution environment. 

4. Each processing attempt must be traceable for audit and troubleshooting purposes. 

5. Infrastructure must never be created manually, to avoid configuration drift. 

6. A **single IAM role** must be reused across multiple AWS services to simplify security and auditing. 

7. GitHub must be the only entry point for deploying, updating, or deleting infrastructure. 



3. Design Principles 

The solution is built using the following design principles: 3.1 Event‑Driven Architecture 

The system reacts automatically to events rather than running on a schedule. 

Uploading a file to S3 is the only action required to start the processing workflow. 

3.2 Ephemeral Compute 

Compute resources are created only when needed and terminated immediately after use. This reduces cost and security risk. 

3.3 Infrastructure as Code 

All AWS resources are defined using Terraform. This ensures consistency and repeatability. 

3.4 Centralized Orchestration 

AWS Step Functions coordinates all steps, ensuring clear sequencing, error handling, and visibility. 

3.5 Single IAM Role Strategy 

A single IAM role is reused across services to simplify security management and auditing. 



4. Solution Overview 

The solution implements a fully automated workflow using managed AWS services. 

At a high level: 

• A transaction file is uploaded to Amazon S3. 

• Amazon EventBridge detects the upload. 

• AWS Step Functions starts a workflow. 

• An EC2 instance is checked or created for preprocessing validation. 

• An ECS Fargate task processes the transaction file. 

• Optional cleanup or validation steps are executed. 

• The workflow ends with a success or failure state. 

Each file upload starts a **separate and independent execution**, allowing parallel processing and easy tracking. 



5. High‑Level Architecture 

The architecture includes the following AWS services: 

• **Amazon S3** – Entry point for transaction files 

• **Amazon EventBridge** – Event detection and routing 

• **AWS Step Functions** – Workflow orchestration 

• **Amazon EC2** – Pre‑processing dependency 

• **Amazon ECS \(Fargate\)** – Transaction processing engine 

• **Amazon VPC** – Network isolation 

• **AWS IAM** – Unified access control 

• **GitHub Actions** – CI/CD and deployment automation 

• **Terraform** – Infrastructure as Code Architecture Overview 





6. End‑to‑End System Flow \(Easy Explanation\) This section explains the full system flow in very simple terms. 

Step 1: File Upload 

A user uploads a transaction file \(such as a PDF or CSV\) to the designated Amazon S3 

bucket. 

• No API calls are required. 

• No scripts need to be executed. 

• Uploading multiple files triggers multiple parallel workflows. 





Step 2: Event Detection 

Amazon S3 automatically generates an **Object Created** event when the file is uploaded. 

Amazon EventBridge listens for these events. 

• EventBridge detects the event. 

• EventBridge forwards the event to AWS Step Functions. 

• No Lambda function is required. 





Step 3: Workflow Orchestration 

AWS Step Functions receives the event and starts the workflow. 

Step Functions ensures: 

• Correct order of execution 

• Clear visibility of each step 

• Error handling and retries 

• Auditable execution history 





7. EC2 Instance Step \(Pre‑Processing Validation\) Purpose of the EC2 Step 

The EC2 instance represents a required system dependency. In real‑world systems, such an instance might be responsible for: 



• Environment validation 

• Access to legacy systems 

• Configuration checks 

• Shared preprocessing logic 

In this design: 

• The EC2 instance is created using Terraform. 

• Step Functions checks or creates the instance. 

• The workflow ensures the instance is available before moving forward. 

EC2 Behavior 

• Instance type: t3.micro 

• Runs inside the project VPC 

• Uses a predefined IAM instance profile 

• No user login or SSH access required If the EC2 instance is not available or fails to start, the workflow stops and reports failure. 





8. ECS Task Step \(Transaction Processing\) Purpose of the ECS Task 

The ECS Fargate task performs the actual transaction processing logic. 



This could include: 

• Parsing transaction files 

• Data validation 

• Database updates 

• External API calls 

Why ECS Fargate? 

**Aspect **

**EC2 Scripts ECS Fargate Tasks **

**Execution **

Long‑running Short‑lived 

**Isolation **

Low 

High 

**Management **Manual 

Automatic 

**Cleanup **

Manual 

Automatic 

**Scaling **

Complex 

Managed 

The ECS task: 

• Runs in Fargate mode 

• Uses awsvpc networking 

• Shares the same IAM role 

• Stops automatically after completion 9. Networking Architecture 

The system runs inside a dedicated **Virtual Private Cloud \(VPC\)**. 



Components 

• VPC \(10.0.0.0/16\) 

• Public subnet \(10.0.1.0/24\) 

• Internet Gateway 

• Route table 

• Security group 

Why Networking Matters 

Networking ensures: 

• Isolation from other AWS workloads 

• Controlled access to external services 

• Secure internal communication 





10. Security Groups and Traffic Flow Inbound Rules 

• Only internal VPC traffic is allowed. 

• No public inbound access is permitted. 

Outbound Rules 

• Outbound traffic is allowed to enable: 



o ECS image downloads 

o API calls 

o OS updates 

This setup prevents unauthorized access while allowing required communication. 



11. IAM Design – Single Role Strategy A **single IAM role** named GitHubActions-IaC-Deployer is reused by: 

• GitHub Actions \(via OIDC\) 

• EC2 instance 

• ECS task 

• Step Functions 

• EventBridge 

Benefits 

• Simplified security model 

• Easier audits 

• Clear permission boundaries 

• Explicit compliance with requirements 





12. Infrastructure as Code \(Terraform\) All infrastructure is defined using Terraform files, including: 

• VPC and networking 

• IAM instance profiles 

• ECS cluster and task definition 

• Step Functions state machine 

• EventBridge rules 

• S3 buckets 

Manual infrastructure changes are not allowed. 



13. Terraform State Management 

Terraform state is stored remotely in Amazon S3. 

Characteristics: 

• Encrypted at rest 

• Centralized storage 

• Locking enabled 

• Prevents concurrent state corruption 





14. GitHub‑Based Deployment Model 

Deployment Workflow 

Workflow Name: **Deploy ABC Startup Infrastructure** Steps: 

1. Manual trigger 

2. GitHub OIDC authentication 

3. Terraform init 

4. Terraform apply 

No credentials are stored in GitHub Secrets. 





15. Infrastructure Teardown \(Destroy Workflow\) A separate workflow is used to destroy infrastructure safely. 

• Manual trigger only 

• Prevents accidental deletion 

• Uses the same IAM role 





16. Error Handling and Recovery 

• Step Functions clearly mark failed states 

• ECS task failures are visible 

• Failed executions do not affect others 

• Manual re‑execution is supported 



17. Observability and Auditability 

Audit trails are available through: 

• GitHub commit history 

• GitHub Actions logs 

• Terraform state 

• Step Functions execution history 

• ECS task lifecycle visibility 

This makes the system suitable for compliance‑driven environments. 



18. Security Best Practices 

• No static AWS credentials 

• OIDC‑based access 

• Least privilege IAM permissions 

• Ephemeral compute resources 

• Network isolation 



19. Benefits of the Solution 

• Fully automated 

• Easy to understand 

• Cost‑efficient 

• Secure and auditable 

• Highly maintainable 

• Scalable and extensible 



20. Final Summary 

This system provides ABC Startup with a complete, professional, and auditable solution for automatic transaction file processing. 

By combining: 

• Event‑based triggers 

• Centralized orchestration 

• Serverless batch processing 

• GitHub‑managed Infrastructure as Code The solution satisfies all stated requirements while remaining simple, transparent, and production‑ready.



