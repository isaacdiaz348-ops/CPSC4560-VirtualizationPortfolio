# Week 7: Managed Database Services

## HarborTech Ticket Summary
In ticket TKT-2026-0007, HarborTech operates a relational database on an Amazon EC2 instance that requires heavy manual administrative overhead—including OS maintenance, patching, backups, storage management, and disaster recovery—prompting the team to evaluate whether to continue with a self-managed server or transition to a managed relational or NoSQL service.

## Client Impact
An incorrect database architecture or service choice directly harms both user experience and business operations by causing service downtime, slow application performance, data loss, and inflated costs. If HarborTech remains on a self-managed EC2 database, manual backups and unautomated failovers increase recovery time during outages while keeping engineering teams trapped in routine maintenance instead of feature development. Conversely, picking the wrong managed database—such as forcing a complex, relational workload into a NoSQL service like DynamoDB without redesigning the application—can trigger application errors, expensive query patterns, and risky data migrations that disrupt day-to-day operations.

## AWS Services Involved
Identify the AWS services and concepts involved in the investigation, such as:
- Amazon RDS: Relational database service that automates time-consulting administration tasks such as provisioning hardware, database setup, patching, and backups.
- Amazon Aurora: MySQL and PostgreSQL-compatiable database built for the cloud, offering high performance, automatic scaling, and distributed storage across multiple Availability Zones. 
- Amazon DynamoDB: A key-value and document NoSQL database designed for high-scale applications requiring consistent, single-digit millisecond latency and fully managed serverless scaling. 
- Relational Databases: Structured data stores utilizing tables, rows, columns, predefined schemes, standard SQL queries, and ACID transactions. 
- NoSQL Databases: Non-relational data stores optimized for unstructured or semi-structured data, flexible schemas, and horizontal scaling.
- Multi-AZ: A high availibility deployment option that replicates data synchronously across multiple Availability Zones to provide automated failover during infrastructure outages. 
- Database Migration Service: AWS Service used to migrate relational databases, data warehouses, and NoSQL data stores securly and reliably to AWS or between different database engines with minimal downtime. 
- Schema Conversion: The process of conveting source database schemas and stored procedures to match the target database engine format when changing platforms. 
- Security Groups: Virtual firewalls that control inbound and outbound network traffic at the instance or database cluster level to secure database connectivity. 
- Database Connectivity: The networking configurations, endpoints, connection strings, and VPC settings required for application servers to securly and reliably communicate with a database instance. 

## Virtualization Connection
Running a database on a virtual machine like EC2 requires your team to manually manage hardware provisioning, disk scaling, OS security patching, database engine updates, and complex multi server failover configurations. Managed database services like RDS or Aurora abstract all of this underlying infrastructure by automatically provisioning compute, dynamically scaling storage, applying OS and database patches, and handling synchronous replication and automated failovers across multiple availability zones. This shifts the burden of server and storage administration to the cloud provider, allowing engineering teams to focus strictly on data modeling, query optimization, and application code.

## Evidence Reviewed
Document the evidence you reviewed, such as:
- A relational database engine running directly on an Amazon EC2 instance supporting structured tables, SQL queries, and transaction processing.
- Highly structured relational schemas organized into tables, rows, and columns rather than unstructured document or key-value stores.
- Standard SQL queries involving multi-table joins and relational data retrieval patterns.
- High availability needs that currently rely on manual failover orchestration or custom scripts rather than native cloud multi-zone replication.
- Existing reliance on manual backup schedules and self-directed recovery testing to meet data protection objectives.
- Network routing, ports, and connection strings managed directly through the EC2 virtual private cloud network stack.
- Instance-level virtual firewall configurations controlling inbound and outbound network traffic to secure database access.
- Necessary steps, risks, and downtime thresholds for shifting data and application traffic away from a self-managed EC2 instance.
- Assessment of whether current database features align directly with managed relational services or require adjustments.
- Evaluation of whether a move to a managed relational engine requires minor adjustments or if a shift to a NoSQL alternative like DynamoDB requires a total data model rewrite.
- Heavy manual operational burden on the client team, including guest operating system maintenance, database software patching, storage management, backup execution, backup verification, and disaster recovery planning.
- Ongoing engineering resource allocation toward low-value infrastructure upkeep versus the potential cost and scalability benefits of automated cloud infrastructure.
  
## AWS Academy Activity: Actions and Results
Command: aws rds create-db-instance \
    --db-instance-identifier my-mysql-db \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --allocated-storage 20 \
    --master-username adminuser \
    --master-user-password SecurePassword123 \
    --backup-retention-period 7
Output: "DBInstance": {
        "DBInstanceIdentifier": "my-mysql-db",
        "DBInstanceClass": "db.t3.micro",
        "Engine": "mysql",
        "DBInstanceStatus": "creating",
        "MasterUsername": "adminuser",
        "AllocatedStorage": 20,
        "PreferredBackupWindow": "08:16-08:46",
        "BackupRetentionPeriod": 7,
        "DBSecurityGroups": [],
        "VpcSecurityGroups": [
            {
                "VpcSecurityGroupId": "sg-0f650cc2ea27d1ead",
                "Status": "active"
            }
        ],
        "DBParameterGroups": [
            {
                "DBParameterGroupName": "default.mysql8.4",
                "ParameterApplyStatus": "in-sync"
            }
        ],
        "DBSubnetGroup": {
            "DBSubnetGroupName": "default",
            "DBSubnetGroupDescription": "default",
            "VpcId": "vpc-054de156fd410bf79",
            "SubnetGroupStatus": "Complete",
            "Subnets": [
                {
                    "SubnetIdentifier": "subnet-03ad9a37498f493cf",
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1e"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
Explanation: This JSON output proves that your AWS CLI request was successfully accepted and that your new MySQL database is actively being built in the background. It confirms that your chosen settings—such as the database engine version, storage size, and instance class—were correctly applied, and it shows the exact network and security group placement assigned to your database within your virtual private cloud.

## Operational Analysis
The evidence shows that the workload relies on structured tables, SQL queries, and transactional consistency while suffering from heavy administrative overhead due to manual OS patching, backup management, and storage provisioning on EC2. Consequently, the operational need is best addressed by transitioning to a managed relational database service like Amazon Aurora or RDS using AWS Database Migration Service. This approach eliminates the low-level infrastructure burden and preserves existing relational query logic without requiring a costly application rewrite, whereas switching to a NoSQL database like DynamoDB would needlessly complicate the system with a total schema redesign.
## Recommendation
Migrating from the self-managed EC2 instance to Amazon Aurora using AWS Database Migration Service best fits the workload because it natively preserves structured relational tables, SQL queries, and transactional consistency while delivering built-in multi-zone high availability. This recommendation shifts low-level infrastructure duties like operating system maintenance, storage provisioning, patching, and manual backup execution to AWS, which relieves the client engineering team of heavy administrative burdens and allows them to focus exclusively on data modeling, query performance, and core application development.

##Escalation Notes
No escalation is strictly required to proceed with planning, but final execution requires administrative approval for a scheduled cutover maintenance window, budget authorization for shifting from EC2 to a managed multi-AZ cluster, IAM permission provisioning for AWS Database Migration Service, and security group adjustments to enable secure database connectivity.

## Lessons Learned
Week 7 demonstrated that while self-managed databases on EC2 provide full control, they burden engineering teams with routine maintenance like OS patching, manual backups, and storage management. Managed database services like Amazon Aurora eliminate this heavy lifting by abstracting infrastructure and providing built-in Multi-AZ high availability and dynamic scaling. Most importantly, selecting a database service requires evaluating clear evidence—such as query patterns, schema structure, and operational overhead—to ensure workload fit. Forcing a relational workload with multi-table joins into a NoSQL service like DynamoDB requires a costly application rewrite, whereas transitioning to a managed relational database using AWS Database Migration Service optimizes operations while preserving core business logic.

## Professional Vocabulary
Define the important Week 7 terms in your own words.
Include terms such as:
- Relational Database: A database model that organizes data into structured tables made of rows and columns with predefined schemas, utilizing SQL for querying and ensuring strict transactional consistency.
- NoSQL: A non-relational database design optimized for unstructured or semi-structured data, offering flexible schemas, high horizontal scalability, and high-throughput read/write operations without requiring standard SQL tables or joins.
- Amazon RDS: A fully managed cloud service that automates setup, operational maintenance, OS and engine patching, hardware scaling, and backups for traditional relational database engines like MySQL, PostgreSQL, and SQL Server.
- Amazon Aurora: A cloud-native relational database engine fully compatible with MySQL and PostgreSQL, designed for high performance, dynamic storage scaling, and continuous, automated replication across multiple locations.
- Amazon DynamoDB: A fully managed, serverless NoSQL key-value and document database built for predictable, single-digit millisecond latency at any scale.
- Multi-AZ: An AWS deployment architecture that synchronously replicates data across multiple physical data centers within a region to ensure automatic failover and high availability during hardware or network outages.
- Database Migration Service: An AWS tool designed to migrate databases securely and reliably between cloud environments, on-premises servers, or database engines with minimal downtime using continuous data replication.
- Schema Conversion: The process of translating database definitions, data types, indexes, and stored code from one database engine format to another when switching platforms.
- Managed Service: An operational cloud delivery model where the infrastructure provider takes ownership of underlying hardware provisioning, maintenance, security patching, and scaling, freeing the client to focus strictly on application logic and data design.
- Workload Fit: The practice of selecting a database engine, architecture, and deployment model based on direct evidence of application requirements, such as data structures, access patterns, performance needs, and administrative capacity.
