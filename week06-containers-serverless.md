# Week 6: Containers and Serverless Computing

## HarborTech Ticket Summary
Bright Path Community Services is running two small workloads, a reporting task that only runs once each morning and a small API on a shared EC2 instance. Although the server sits idle for too long, Bright Path still carries the full overhead of managing the operating system and application runtime dependencies. 

## Client Impact
A poorly matched compute model can affect cost by forcing the client to pay for 24/7 idle server capacity, affect reliability by making applications harder to patch and recover, limit scalability during traffic spikes, and burdening the client's team with manual server upkeep.

## AWS Services Involved
Identify the AWS services and concepts involved in the investigation, such as:
- Containers: A packaging method that bundles application with it's dependencies, allowing it to run reliably in an isolated enviroment while sharing a host OS kernel. 
- Serverless Computing: A cloud execution model where AWS manages all underlying servers, patching, and scaling. Only pay for compute resources consumed.
- AWS Lambda: A serverless compute service that runs code in response to events without provisioning or managing servers, ideal for short-lived tasks or backend APIs.
- Amazon API Gateway: Fully managed server that acts as a "front door" for APIs, public traffic entry, SSL termination, authorization, and request routing to backend compute like Lambda. 
- AWS Step Functions: A visual workflow service used to coordinate multiple AWS services into structured, step-by-step state machines. 
- IAM Execution Roles: Roles that are assigned to services to grant them secure, least-privilege permissions to access other AWS resources.
- Event-Driven Computing: Architectural pattern where workloads are triggered by specific events or schedules automatically rather than running continuously. 
- VPC-Connected Lambda: Configuration that allows Lambda function to run inside a secure VPC, which enables private access to internal databases or resources.
- Monitoring and Logs: Observability tools used to keep track of application health, analyze execution logs, and monitor performance metrics.

## Virtualization Connection
Containers can reduce infrastrucutre management by packing an application and it's dependencies together while sharing the host operating system. On the other hand, seamless computing removes infrastrucutre management entirely by letting AWS handle the servers, operating systems, and scaling so the client can only focus on the code. 

## Evidence Reviewed
Document the evidence you reviewed, such as:
- Workload duration and frequency: The reporting task runs once each morning and completes quickly,, whereas the API processes intermittent requests throughout the day. 
- Current server utilization: The EC2 instance is only utilized for small amounts of time, which leaves the instance idle for long periods of time. 
- Trigger or event pattern: The reporting task is a scheduled, event-based start, ehile the API responds to unpredictable external web traffic. 
- Runtime and dependency requirements: The API requires specific package dependencies and runtime enviroments, while the reporting task operates statelessly without local disk persistence. 
- IAM permissions: Evaluate least-privilege IAM execution roles to ensure services like Lambda have secure, restricted access o downstream data stores and secrets. 
- API request behavior: Small API experiences low-frequency, intermittent traffic patterns, which makes it a candidate for scale-to-zero capabilities or lightweight containerization. 
- Lambda logs and errors: Monitor execution logs for timeout limits, memory efficiency, and error traces to validate serverless compatibility. 
- VPC connectivity requirements: Assess whether workloads require private network access to internal databases, which determines if VPC-connected configurations are neccessary. 
- Workflow steps and failure points: Analyze the sequential execution of the morning report to identify potential bottlenecks and ensure reliable error handling. 
- Monitoring evidence: Review metrics and utilixation telemetry confirming low average resource consumption and idle overhead on the legacy EC2 host. 

## Operational Analysis
The evidence shows that the current compute model is inefficent, which is forcing the client to pay 24/7 for a underutilized EC2 instance while also managing guest operating system overhead. Based on the information, the morning task is best-suited for a serverless function because of it's scheduled and stateless nature. On the other hand, the API will be best handled by a API-driven service that dynamically scales with traffic. 

## Recommendation
The recommended approach would be using AWS Lambda for the morning report task and Fargate containes for the small API. 

## Escalation Notes
Deploying these changes requires approval for setting up least-privilege IAM security roles, configuring VPC networking and security groups, and establishing new container deployment pipelines.

## Lessons Learned
Week 6 taught how containers package applications with their dependencies for isolated runtime consistency, how serverless computing eliminates server management by scaling ephemeral code on demand, how APIs connect front-end requests to backend logic, and how event-driven workloads use automated triggers like schedules to optimize resource usage, altogether emphasizing that selecting the right compute model depends entirely on matching a workload's unique behavior to avoid wasted idle capacity and operational overhead.

## Professional Vocabulary
Define the important Week 6 terms in your own words.
Include terms such as:
- Container: A lightweight, portable package that bundles an application together with its code, libraries, and dependencies so it can run reliably anywhere while sharing the host operating system's kernel.
- Serverless Computing: A cloud execution model where AWS manages all the underlying servers, operating systems, and scaling automatically, allowing you to run code without provisioning or maintaining infrastructure and pay only for active use.
- AWS Lambda: A serverless compute service that runs your application code in response to specific events, handling execution and scaling automatically without the need for dedicated servers.
- API Gateway: A managed service that acts as the front door for web applications, handling incoming HTTP traffic, security, authorization, rate limiting, and request routing to backend services.
- Execution Role:  An AWS Identity and Access Management (IAM) role that grants permissions to a service (like a Lambda function) so it can securely access other AWS resources like databases or storage.
- Event Trigger: An automated signal or schedule that kicks off a specific workload or function to run.
- Step Functions: A visual orchestration service that connects multiple services and tasks together into a structured, sequential workflow or state machine.
- VPC-Connected Lambda: A configuration that allows a serverless Lambda function to run securely inside a private Virtual Private Cloud, giving it access to internal network resources like private databases.
- Runtime: The underlying software environment and programming language support provided to execute your application code.
- Abstraction: The process of hiding complex, low-level infrastructure details (like physical hardware, hypervisors, and operating systems) so you can focus strictly on building and deploying applications.
