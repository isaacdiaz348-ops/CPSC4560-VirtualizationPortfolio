# Week 11: Monitoring and Change Evidence

## HarborTech Ticket Summary
Riverside Goods reported severe website slowdowns and noted that its production web tier Auto Scaling group (rg-web-asg) no longer matched its approved baseline capacity. Because no team member recalled making a modification and no approved change request was on file, HarborTech reconstructed the incident using cloud telemetry, configuration snapshots, and audit logs.

## Client Impact
When monitoring, change control, and audit logging are weak or missing, small infrastructure changes can quickly trigger major operational failures.

## AWS Services Involved
Identify the AWS services and concepts involved in the investigation, such as:
- Amazon CloudWatch: The overarching observability and monitoring platform used to collect telemetry, track system performance, store application logs, and trigger automated operational alerts.

- CloudWatch Metrics: Captured quantitative performance data over time—specifically showing TargetResponseTime spiking from 0.3 s to 2.4 s at 14:07:00 UTC, pinpointing the exact onset of user-facing latency.

- CloudWatch Logs: Stored the application-level text logs (/rg/web/app), providing the WARN upstream response 2.38 s, healthy targets 1 entry at 14:07:30 UTC that linked the latency spike directly to reduced backend target count.

- CloudWatch Alarms: Monitored metric thresholds continuously and transitioned rg-web-latency-high from OK to ALARM at 14:09:00 UTC after two consecutive datapoints exceeded 1 second, establishing the sustained severity of the incident.

- AWS CloudTrail: Recorded control-plane management API calls across the AWS account. It provided the critical audit trail showing the UpdateAutoScalingGroup call at 14:05:12 UTC, including the assumed IAM role (RG-WebOps-Role/session-7f3a), source IP (203.0.113.24), and requested parameter changes (desiredCapacity: 1, maxSize: 1).

- AWS Config: Tracked resource inventory and state changes over time. It confirmed at 14:06:03 UTC that rg-web-asg transitioned from its baseline (DesiredCapacity: 3, MaxSize: 4) down to a single instance, proving the CloudTrail API request took effect.

- Amazon Athena: A serverless interactive query service commonly used in incident investigations to run SQL queries directly against large volumes of historical CloudTrail logs or VPC Flow Logs stored in Amazon S3—useful if investigators need to search broader time windows or correlate session activity across multiple logs.

- Monitoring: The continuous collection and evaluation of real-time system health telemetry (metrics, logs, and alarms) to detect performance degradation, threshold breaches, and service availability issues as they happen.

- Audit Evidence: Immutable records of control-plane activity (who or what called which API, from where, and with what parameters) used to establish accountability, verify whether changes followed approved change-management workflows, and separate verified facts from unverified assumptions.

- Configuration History: A chronological record of a resource's actual state before and after a change event, enabling investigators to detect unauthorized configuration drift and verify that an API request actually altered the live infrastructure environment.

## Virtualization Connection
Since cloud infrastructure is software-defined, a single API call can alter servers or networks in seconds without anyone touching physical hardware. Monitoring, configuration, and audit services give teams visibility into these invisible changes across three layers: audit logs (like CloudTrail) record which API commands were run, configuration history (like AWS Config) tracks how resource settings changed before and after, and monitoring (like CloudWatch) measures how those changes affect live system performance. Together, they create a complete record of what happened and why.

## Evidence Reviewed
Document the evidence you reviewed, such as:
- CloudWatch metrics: Reviewed TargetResponseTime metrics for the web tier load balancer target group, observing baseline latency of 0.3 seconds prior to 14:06 UTC followed by a sharp eightfold spike to 2.4 seconds at 14:07:00 UTC.

- CloudWatch alarm history: Reviewed state transition history for alarm rg-web-latency-high, confirming it transitioned from OK to ALARM at 14:09:00 UTC after breaching the > 1 second threshold for two consecutive evaluation datapoints.

- Application or system logs: Examined application log group /rg/web/app around the onset of latency, capturing the warning entry at 14:07:30 UTC: WARN upstream response 2.38 s, healthy targets 1. This corroborated the CloudWatch metric spike and confirmed that traffic was hitting only one active backend target.

- CloudTrail management events: At 14:05:12 UTC, CloudTrail recorded an UpdateAutoScalingGroup management API call targeting rg-web-asg that lowered both desiredCapacity and maxSize to 1. The request was executed through temporary credentials under assumed-role/RG-WebOps-Role/session-7f3a originating from IP address 203.0.113.24. Because the call used a shared operational session without a corresponding change ticket, the log confirms the exact parameter change and session identity without identifying the specific person or script behind it.
 
- AWS Config resource history: At 14:06:03 UTC, AWS Config recorded a configuration state change for rg-web-asg showing that DesiredCapacity dropped from 3 to 1 and MaxSize dropped from 4 to 1. This snapshot confirmed that the 14:05:12 UTC CloudTrail API call actually altered the live Auto Scaling group settings just under a minute later. Having both the before and after states establishes clear configuration drift from the baseline and links the management API action directly to the capacity drop seen in monitoring.
  
- Configuration changes: Verified that the live resource configuration transitioned from its approved baseline (Before: DesiredCapacity: 3, MaxSize: 4) to a degraded single-instance state (After: DesiredCapacity: 1, MaxSize: 1), confirming that the 14:05 CloudTrail request was applied to the infrastructure.

- Event timestamps: The event timestamps across CloudTrail, AWS Config, CloudWatch metrics, CloudWatch logs, and CloudWatch alarms line up tightly within a four-minute window: 14:05:12 UTC for the API call, 14:06:03 UTC for the configuration change, 14:07:00 UTC for the response-time spike, 14:07:30 UTC for the warning log, and 14:09:00 UTC for the alarm trigger. Seeing the exact minute-and-second sequence makes it clear that the capacity reduction came first and the latency degradation followed right behind it. This tight chronological order is what ties the control-plane change, resource state shift, and customer-facing slowdown together as one incident.
  
- User or role identity: Inspected userIdentity and sourceIPAddress fields, confirming the request was executed via temporary credentials under assumed-role/RG-WebOps-Role/session-7f3a originating from IP address 203.0.113.24. (No direct named IAM user or approved change ticket was associated with the session.)

- API actions: Reviewed CloudTrail management events for rg-web-asg, identifying an UpdateAutoScalingGroup API call executed at 14:05:12 UTC that explicitly set desiredCapacity to 1 and maxSize to 1.

- Resource state before and after a change: Reviewed the AWS Config timeline and configuration item snapshots for rg-web-asg at 14:06:03 UTC.

- Athena query results: Correlated filtered CloudTrail event queries (as would be retrieved via Amazon Athena over S3 log archives) isolating Auto Scaling control-plane modifications during the 14:00–14:15 UTC window to confirm no other competing scaling or deployment API calls occurred.
  
- Incident timeline: At 14:05:12 UTC, CloudTrail recorded an API call from RG-WebOps-Role/session-7f3a reducing the Auto Scaling group's desired capacity and max size to 1, which AWS Config confirmed took effect at 14:06:03 UTC as capacity dropped from 3 to 1. By 14:07:00 UTC, CloudWatch metrics showed target response times spiking from 0.3 to 2.4 seconds, corroborated 30 seconds later by application logs warning of 2.38-second responses with only one healthy target. Finally, at 14:09:00 UTC, the high-latency CloudWatch alarm breached its threshold and entered an ALARM state.

## Operational Analysis
Correlating the evidence across a four-minute window (14:05:12 to 14:09:00 UTC) confirms that an UpdateAutoScalingGroup API call via RG-WebOps-Role/session-7f3a lowered rg-web-asg from 3 desired / 4 max instances down to 1, which AWS Config captured at 14:06:03 UTC. Immediately afterward, CloudWatch metrics and logs showed response times jumping from 0.3 s to 2.4 s with only one healthy target, triggering the rg-web-latency-high alarm at 14:09:00 UTC. While the logs clearly prove the timeline and parameter changes, it remains an unverified assumption whether a human operator or an automated script initiated that shared session—leaving exact attribution, source IP legitimacy (203.0.113.24), and whether the action was accidental or malicious as open questions.

## Recommendation
The primary recommended action is correction: restoring rg-web-asg back to its approved baseline capacity of 3 desired and 4 maximum instances to immediately relieve web tier latency. Because the change came from a shared role session (RG-WebOps-Role/session-7f3a) and IP (203.0.113.24) without an approved ticket, this correction should be paired with containment checks on that session and a configuration review to ensure no other settings were altered. Afterward, continued monitoring of CloudWatch latency and alarm states—alongside an identity and change audit—will confirm full recovery and establish who or what initiated the change.

## Escalation Notes
Escalation to Cloud Engineering and Security Operations is required because a production Auto Scaling group (rg-web-asg) was scaled down to a single instance without an approved change ticket via a shared role session (RG-WebOps-Role/session-7f3a) from an unverified IP address (203.0.113.24). This creates both an unverified configuration drift/access concern and a lingering risk of unauthorized credentials or repeat scaling disruption. Confirming baseline capacity restoration, normal latency recovery, and session/IP legitimacy remain open items that require senior engineering and security follow-up.

## Lessons Learned
Week 11 showed that troubleshooting a cloud issue requires connecting observability, audit logs, and configuration history rather than relying on symptoms alone. CloudWatch metrics, logs, and alarms revealed the timing and runtime impact of the slowdown, while CloudTrail and AWS Config captured the exact API call, session identity, and resource drift behind it. Putting these timestamps and records together establishes a clear cause-and-effect timeline and separates hard facts from assumptions so teams can fix the problem safely and escalate what remains unverified.

## Professional Vocabulary
Define the important Week 11 terms in your own words.
Include terms such as:
- Monitoring: Continuously watching a live system's behavior, health, and performance.
- Metric: A numerical measurement tracked over time.
- Log: A timestamped record of discrete events or messages emitted by an application, operating system, or service, capturing specific details like errors, warnings, or status lines.
- Alarm: An automated rule that watches a metric over defined intervals and triggers a state change or notification when a threshold is breached.
- CloudWatch: AWS's native observability service that collects metrics, aggregates log streams, and evaluates alarms to show how cloud workloads behave at runtime.
- CloudTrail: AWS's control-plane auditing service that records API calls and management actions—including who or what made the request, when it happened, from which IP, and what parameters were passed.
- AWS Config: AWS's resource-state and compliance tracking service that captures configuration snapshots and records how cloud resources change from a previous state to a new state.
- Audit Trail: A chronological, tamper-evident record of actions and requests taken against a system, used to trace accountability and verify change activity.
- Configuration History: The record of how a resource's settings and attributes looked before and after changes over time, making it possible to spot configuration drift.
- Incident Timeline: The ordered sequence of key events, state changes, and alerts reconstructed from exact timestamps to show how an incident unfolded from trigger to impact.
- Evidence Correlation: Comparing and lining up data across multiple sources—such as audit logs, configuration snapshots, metrics, logs, and alarms—so findings reinforce one another and separate verified cause-and-effect from guesswork.
