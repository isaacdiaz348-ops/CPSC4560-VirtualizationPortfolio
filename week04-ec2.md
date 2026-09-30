# Week 4: EC2 Evidence Lab

## HarborTech Ticket Summary:
Ticket ID - TKT-2026-0004
Client - Riverside Goods
Severity/Priority - High
Subject - EC2 Instance Failure and Network Diagnostic Audit
## Client Impact
Service interruption - HTTP traffic to "Riverside Goods" primary web service timed out, stopping order processing and external operational reporting

Operational Risk - Administrative SSH management access was blocked, which prevents system administrators from accessing guest OS diagnostic logs or being able to perform routine maintenance.

Business Severity - High operational impact. Downtime risks data synchronization delays between "Riverside Goods" and "HarborTech" partner endpoints.
## Environment and Resource Names
VPC - vpc-054de156fd410bf79
Subnet -  subnet-0fce11faec3c98817
Instance ID -  i-05c2579429dfcb78f
IPv4 Address(s) - 34.235.152.8. and  100.30.238.174
## AWS Documentation Evidence
AWS Security Groups - Security Groups act as firewalls at the ENI layer. They are stateful and return traffic is automatically allowed regardless of the inbound rules. By default, new security groups deny all inbound traffic unless explicitly authorized.

AWS Instance Metadata Service Version 2(IMDSv2) - Session-oriented authentication mechanism requiring a HTTP PUT request within the header to be able to generate a secret token. 

EC2 Instance Lifecycle- Stopping an instance transitions it to the stopping and stopped states. Haliting a non-Elastic IP instance releases its public IPv4 address back to the AWS public pool. Upon starting, AWS assigns a new public IPv4 address, while the private one remains persistent. 
## Root-Cause Analysis
Symptom - All incoming TCP socket initiation packets to public IP 34.235.152.8 on ports 22 and port 80 time out without receiving a TCP SYN-ACK or RST response.
## Corrective Action
Action - Apply minimal, scope-accurate remediation by adding targeted ingress rules to Security Group without modifying needed OS configurations, route tables, or network ACLs
## Cleanup Evidence
$ aws ec2 terminate instances --instance-ids i-05c2579429dfcb78f

"TerminatingInstances": [
        {
            "InstanceId": "i-05c2579429dfcb78f",
            "CurrentState": {
                "Code": 32,
                "Name": "shutting-down"
            },
            "PreviousState": {
                "Name": "shutting-down"
            },
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}

$ aws ec2 wait instance-terminated --instanceids i-05c2579429dfcb78f
no output 
$aws ec2 delete-security-group sg-08edc66f7615bc594
aws: [ERROR]: An error occurred (DependencyViolation) when calling the DeleteSecurityGroup operation: resource sg-08edc66f7615bc594 has a dependent object
## Escalation and Change-Control Notes
Notes - Change authority that is approved under Standard Infrastructure. If the Security Group rule authorization is failing to restore port connectivity, escalation will be designated to a higher tier HarborTech engineer. To eliminate public IP variance accross reboot cycles during the lifecycle audit, allocate an AWS Elastic IP(EIP).
## Lessons Learned
Learned - Security Groups drop unpermitted ingress traffic silently without returning a TCP RST packet, causing external requests to fail with generic time-out errors rather than immediate connection refusals.
Enforcing vulnerabilities by forcing callers to fetch a temporary IMDSv2 token via PUT before issuing GET requests.
Standard auto-assigned public IPv4 addresses are ephemeral. Systems relying on persistent IP endpoint configurations require Elastic IPs or DNS record updates via Amazon Route 53 post-lifecycle state changes.
## Professional Vocabulary
ENI(Elastic Network Interface) - A logical networking component in a VPC that represents a virtual network card.

Stateful filtering - A firewall mechanism where allowed incoming requests automatically permit corresponding outgoing return traffic.

IMDSv2(Instance Metadata Service Version 2) - An on-instance component used to securly obtain instance details using token-backed HTTP requests.

TCP Handshake Timeout - An error condition occuring when a client sends a SYN packet but receives no response before timing out.

Dependency Hierarchy - The structural ordering required when provisioning or destroying AWS cloud resources. 
