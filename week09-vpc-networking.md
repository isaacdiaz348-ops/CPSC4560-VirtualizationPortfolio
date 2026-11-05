# Week 9: VPC Networking and Troubleshooting

## HarborTech Ticket Summary
[The HarborTech Week 9 ticket involves an external client unable to reach a web application on TCP port 80, even though the instance is running and firewalls are properly configured. The root cause is a missing default route (0.0.0.0/0) targeting the Internet Gateway in the subnet's route table, which causes incoming traffic to be dropped at the routing boundary before reaching the application.] 

## Client Impact
[Incorrect routing, flawed subnet design, and misconfigured traffic filtering directly undermine cloud reliability, turning operational missteps into measurable business loss. In cloud environments like AWS, where infrastructure is defined by software, these errors create invisible barriers that disrupt services even when underlying workloads are fully healthy.]

## AWS Services Involved
Identify the AWS services and concepts involved in the investigation, such as:
- Amazon VPC:  [The foundational isolation boundary (vpc-0a91) that provisions a private, logically isolated section of the AWS cloud where all resources are launched.]
- Subnets: [Segments of the VPC IP address range (subnet-0a91) used to group and isolate resources.]
- Route Tables: [A set of routing rules (associations like rtb-0a91) that determine where network traffic from a subnet is directed. The absence of a default route (0.0.0.0/0 → igw) in the table was the root cause of the incident.]
- Internet Gateway: [A horizontally scaled, redundant VPC component (igw-0a91) that connects the VPC to the public internet, enabling inbound and outbound traffic for public resources.]
- NAT Gateway: [A managed Network Address Translation service used to provide secure outbound-only internet access for resources in private subnets without exposing them to unsolicited inbound connections.]
- Security Groups: [Stateful, virtual firewalls operating at the instance level (ENI) to control inbound and outbound traffic. The security group (sg-0a91) was correctly configured to permit inbound TCP port 80.]
- Network ACLs: [Stateless subnet-level firewalls acting as an additional layer of security guarding the subnet boundary, which correctly permitted the required web and ephemeral traffic.]
- CIDR Blocks: [The IP address allocation schema (10.42.0.0/16) defining the IP range for the VPC, establishing the boundary for internal routing and subnet sizing.]
- Public and Private Subnets: [A public subnet has its route table mapped to an Internet Gateway for external communication, while a private subnet is isolated from direct inbound internet access (often relying on NAT for updates). In this ticket, the subnet was incorrectly assumed to be public because of an attached IP, despite lacking the required route.]
- VPC Flow Logs: [A native feature used to capture information about IP traffic going to and from network interfaces in a VPC. In live troubleshooting scenarios, Flow Logs help verify whether packets successfully hit the network boundary or are being dropped by route tables or firewalls (REJECT vs. ACCEPT).]

## Virtualization Connection
Amazon VPC (Virtual Private Cloud) replaces physical hardware—such as routers, switches, physical firewalls, and copper cables—with software-defined networking (SDN). Instead of configuring physical patch panels or VLANs on hardware switches, cloud engineers use software constructs, APIs, and declarative configurations to build and secure a virtual network environment.

## Evidence Reviewed
Document the evidence you reviewed, such as:
- VPC CIDR: [10.42.0.0/16 (associated with VPC ID vpc-0a91). Establishes the core IP address space for internal routing.]
- Subnet CIDR: [Subnet subnet-0a91.] 
- Subnet type: [While assigned a public IPv4 address, it functioned functionally as a private subnet due to routing omissions.
- Route table entries.] 
- Internet gateway attachment: [Internet Gateway igw-0a91 is successfully created and attached to VPC vpc-0a91]
- NAT gateway configuration: [Not utilized in the direct inbound path, though reviewed as part of architectural alternatives for private subnet outbound updates.]
- Security group rules: [Security Group sg-0a91 is correctly configured to permit inbound TCP port 80 from 0.0.0.0/0 (leveraging stateful tracking for return traffic).]
- Network ACL rules: [Stateless NACL rules correctly permit inbound TCP port 80 and outbound ephemeral return ports (1024–65535).]
- Public or private IP addressing: [The target EC2 instance has a public IPv4 address (198.51.100.24) assigned alongside its private IP within the VPC range.]
- Application port: [The web application service is running and actively listening on TCP Port 80 inside the operating system.]
- Flow log evidence: [Indicate that incoming packets hitting the public IP lack a valid routing path and are dropped at the subnet routing boundary before reaching instance firewalls.]
- Connectivity test results: [External client requests (e.g., curl) fail with connection timeouts because traffic encounters a black hole at the routing layer.]
- 
## Operational Analysis
The evidence shows that compute resources, gateways, and firewalls are fully healthy and permissive, but the network path fails entirely at the routing layer. Specifically, the subnet's route table lacks a default route (0.0.0.0/0) pointing to the Internet Gateway, causing incoming traffic to be dropped at the routing boundary before it ever reaches the application.

## Recommendation
To resolve the issue, update the subnet's route table (rtb-0a91) by adding a single default route (0.0.0.0/0) pointing to the Internet Gateway (igw-0a91). This directly provides the missing forwarding path for external traffic without altering the security groups or network ACLs, which are already correctly configured.

## Escalation Notes
No architectural or firewall changes are required since gateways and security groups are already properly configured. The only necessary action is updating the subnet route table to restore application connectivity. Depending on internal organizational governance, this routing adjustment may require a standard Change Advisory Board (CAB) approval before execution in a live production environment.

## Lessons Learned
Week 9 taught us that VPC connectivity depends on explicit routing, proving that public IPs and attached gateways are useless without a default route. It reinforced that structured, evidence-based troubleshooting beats guesswork every time, preventing unnecessary changes to working firewalls and focusing remediation on the exact architectural bottleneck.

## Professional Vocabulary
Define the important Week 9 terms in your own words.
Include terms such as:
- VPC: [An isolated, dedicated section of a public cloud provider’s network where you build and manage your virtual cloud resources safely tucked away from other tenants.]
- Subnet: [A logically divided sub-section of an IP network within your VPC, used to group related systems together based on connectivity or security needs.]
- Route Table: [A virtual navigation table containing a list of IP destination rules that determine where network traffic entering or leaving a subnet is directed.]
- Internet Gateway: [A scalable VPC component that serves as a bridge, allowing bidirectionally open, direct communication between resources inside your VPC and the internet.]
- NAT Gateway: [A managed network service that allows private resources to send outbound traffic to the internet (for updates or patches) while preventing the public internet from initiating inbound connections back to them.]
- Security Group: [A stateful, virtual firewall operating directly at the server or instance level, regulating inbound and outbound traffic based on explicit allow rules.]
- Network ACL: [A stateless, subnet-level firewall that acts as a perimeter guard for an entire subnet, evaluating both inbound and outbound traffic line-by-line using ordered allow and deny rules.]
- CIDR: [A standard notation method (such as /16 or /24) used to allocate IP address ranges, defining how many IP addresses are available within a VPC or subnet network boundary.]
- Public Subnet: [A subnet whose associated route table has an explicit default route (0.0.0.0/0) mapped to an Internet Gateway, allowing its resources to send and receive public internet traffic directly.]
- Private Subnet: [A subnet whose route table lacks a direct route to an Internet Gateway, isolating its internal resources from unsolicited inbound internet access.]
