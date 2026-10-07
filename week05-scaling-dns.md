# Week 5: Scaling, Load Balancing, and DNS

## HarborTech Ticket Summary
The Week 5 HarborTech ticket (TKT-2026-0005) invloves addressing a operational risk for Riverside Goods,  whose application is currently only running on a single server with a single endpoint. 

## Client Impact
Traffic sure and single-point-of-failure risk can affect users and business operations by having slow load times, timed-out requests, and complete website crashes. 

## Provided Ticket Evidence
The ticket evidence tells us that Riverside Goods relies on a single EC2 instance and a single exposed endpoint. With Riverside Goods having an upcoming seasonal surge, previous peak data tells us the single EC2 instance is already near it's performance limits. This creates a capacity ceiling which can cause server crashes under high loads and a single enpoint where any server outage will crash the entire website.

## AWS Commands Used
1.). aws autoscaling describe-auto-scaling-groups \
  --query 'AutoScalingGroups[].{Name:AutoScalingGroupName,Min:MinSize,Desired:DesiredCapacity,Max:MaxSize}' \
  --output table

2.)  aws elbv2 describe-target-groups \
  --query 'TargetGroups[].{Name:TargetGroupName,TargetGroupArn:TargetGroupArn,Protocol:Protocol,Port:Port}' \
  --output table
     ## AWS Evidence Collected

3.)  TARGET_GROUP_ARN=$(aws elbv2 describe-target-groups \
  --query 'TargetGroups[0].TargetGroupArn' \
  --output text)

echo "$TARGET_GROUP_ARN"

aws elbv2 describe-target-health \
  --target-group-arn "$TARGET_GROUP_ARN" \
  --query 'TargetHealthDescriptions[].{Target:Target.Id,Port:Target.Port,State:TargetHealth.State,Reason:TargetHealth.Reason}' \
  --output table

4.)  aws route53 list-hosted-zones \
  --query 'HostedZones[].{Name:Name,Id:Id,Private:Config.PrivateZone}' \
  --output table

## Virtualization Connection
Scaling and load balancing eliminate the dependency on a single server by treating servers as flexible and repleaceable sources. Auto Scaling automatically manages servers based on demand, expanding capacity during traffic surges and shrinking down during slower periods. An Elastic Load Balancer acts as a traffic controller, which constantly checks which servers are healthy and spreads out incoming requests evenly.

## Operational Analysis
The evidence shows that capacity, target health, and traffic distribution are properly set up, however DNS failover is incomplete. The AWS enviroment solves the single endpoint risk by using an Auto Scaling Group that is set to 2 / 2 / 6 range that is paired with an Elastic Load Balancer and Target Group that actively distribute traffic across healthy EC2 instances. Since there are no Route 53 health checks or failover policies, the system cannot perform automatic DNS failover. 

## Recommendation
HarborTech should configure a Route 53 Health Check for the main load balancer and also configure a secondary backup routing policy. This will help fix the remaining DNS availability risk by letting Route 53 detect outages and send user traffic to a backup location without the need of a technician. For cost and reliability, adding these health checks only adds on a few more dollars a month while completely protecting Riverside Goods from losing sales due to downtime.

## Escalation Notes
If any modifications are done to production scaling, load balancer limits, Route 53 requires formal change control approval and an escalation to senior Clout Operations before executing. 

## Lessons Learned
The lessons learned this week are that when we are building a reliable cloud infrastrucute, it requires linking compute, routing, and DNS services together, then validating each in the AWS CLI. Another lesson that was learned is that target health checks only confirm backend server status and not total site availability. 

## Professional Vocabulary
Elasticity: The system's ability to automatically grow or shrink depending on when traffic spikes or traffic drops. 
Scalability: The infrastructure's capacity to handle growing workloads by adding more resources when needed.
Load Balancer: A virtual traffic controller that spreads out incoming server requests across healthy instances.
Target Group: A list of backend resources that tells the load balancer where to send incoming traffic.
Health Check: An automated test that is sent periodically to confirm a server is responsive before sending traffic to it.
Auto Scaling Group: A tool that automatically maintans application availability my adding or removing EC2 instances based on traffic or CPU rules
Launch Template: A pre-configured blueprint that contains AMI, instance type, security settings, and scripts needed to launch new EC2 instances.
Desired Capacity: The target number of EC2 instances that are set for an Auto Scaling Group to maintain under normal conditions. 
Route 53: AWS's scalable DNS that translates web addresses into computer addresses.
Failover: A backup mechanism that autmatically shifts user traffic from an unhealthy primary system to a secondary system during an outage.
     
