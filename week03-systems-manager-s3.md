# Week 3: Systems Manager and S3

## HarborTech Ticket Summary
Bright Path Nonprofits is incurring unnecessary overhead and maintenance time managing dedicated EC2 instances for hosting a simple, static public resource page. To resolve this, the workload will be transitioned to a serverless static website hosted on Amazon S3, with AWS Systems Manager (SSM) utilized for operational management and maintenance automation.

## Client Impact
Repeated manual administration leads to configuration drift, human error, and time-consuming tasks that scale poorly with growth. It also creates operational risks by relying on tribal knowledge, delaying critical security updates, and increasing the risk of service downtime.

## AWS Services Involved
Identify the AWS services and concepts involved in the investigation, such as:
- AWS Systems Manager - The central operations hub used to track, automate, and control infrastructure resources without requiring manual login access.
- Run Command - SSM capability used to remotely and securely execute scripts, commands, or configuration changes across managed EC2 instances, eliminating the need to log in manually via SSH.
- Session Manager - Provides secure, browser-based shell access to managed instances without opening SSH/RDP ports or managing bastion hosts.
- Inventory - Collects metadata from managed EC2 instances such as installed application versions, OS configurations, and system properties.
- Parameter Store - A secure, centralized configuration management service used to store plain-text or encrypted key-value parameters so scripts and services do not need to rely on hardcoded variables. 
- Amazon S3 - Highly scalable object storage used to store the web page's static files securly and cost-effective. 
- Static Website Hosting - Enables direct HTTP web traffic serving for static assets, replacing the need for dedicated EC2 instances.

## Virtualization Connection
AWS Systems Manager (SSM) provides a centralized management layer by using a lightweight agent to unify visibility and control across your entire infrastructure.

## Evidence Reviewed
Document the evidence you reviewed, such as:
- Current manual administration steps - Dana performs repetitive manual administration tasks across multiple EC2 instances every week.
- Number of affected systems - Workload spans several EC2 instances requirind repeated maintenance.
- Managed node requirements - SSM agent must be installed, active, and running. IAM role permissions and outbound conectivity over HTTPS to Systems Manger endpoints.
- Systems Manager feature fit - "Run Command" which is ideal for Dana's weekly automated execution of non-interactive scripts across multiple instances simultaneously without direct SSH logins. 
- Interactive versus non-interactive access needs - Non-interactive access is a primary need for routine patches and tasks.
- Configuration or parameter needs - Target S3 bucket names (brightpath-id-22), static access source directories (brightpath-site), and enviroment flags and script execution parameters.
- Static content requirements - Public resource page that only includes static assets, no server-side processing, and offload to Amazon S3 Static Web Hosting, completely removing the EC2 instance. 
- Evidence available through the AWS environment - AWS CloudTrail Logs, SSM Command History, and S3 Access Logs and Command Metrics. 

## Operational Analysis
Explain which tasks are better suited for centralized management, automation, interactive access, or static object hosting.
Support your analysis with the evidence you reviewed.

For centralized management, bucket names and execution paths are invloved in this category because the scripts need dynamic settings that should not be hardcoded into individual servers. For automation, the task that would go with this is Run Command since Dana is currently doing repetitive work across the EC2 instances which takes too much time. For interactive access, it would be be emergency troubleshooting and the tool would be AWS Session Manager. For static object hosting, which involve web assets or downloadable files that do not need server-side code. The best tool for this is Static Hosting. 

## Recommendation
Recommend the AWS service or feature that best fits each identified operational need.
Explain why the recommendation is appropriate for the workload.

## Escalation Notes
Document any prerequisite, permission, configuration, or environment issue that requires additional approval or support.
If no escalation is required, state that clearly.

## Lessons Learned
Week 3 provided practical, real-world lessons in operational efficiency, cloud security, and system architecture through the evaluation of HarborTech’s workflows and Bright Path Nonprofits’ resource hosting.

## Professional Vocabulary
Define the important Week 3 terms in your own words.
Include terms such as:
- Systems Manager - Provieds visibility and control over infrastructure without requiring direct server access.
- Managed Node - A machine configurated to participate in Systems Manager operations
- Run Command - Command that lets you remotely execute commands on one or many EC2 instances without requiring SSH access or manual login.
- Session Manager - Feature that provides secure browser-based terminal access to EC2 instances without opening inbound ports, maintaining SSH keys, or using bastion hosts. 
- Inventory - Collects metadata about managed nodes so the operator can inspect enviroment state.
- Parameter Store - Feature that provides centralized storage for data and secrets. 
- Automation - Automation uses software, scripts, and AI to perform repetitive technical tasks and streamline workflows with minimal human intervention.
- Static Website Hosting - A capability that serves static webstie files such as HTML, CSS, images, and clinet-side JavaScript from a bucket website endpoint. Does not execute server-side application code.
- Object Storage - A data architecture that manages information as distinct, self-contained units called "objects" instead of using traditional file hierarchies or block mappings.
- Management Plane - The administrative layer of a network, cloud, or IT system used to configure, monitor, and manage infrastructure
