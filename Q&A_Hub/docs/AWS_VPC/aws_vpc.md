<div align="center" markdown="1">

# 🌐 AWS VPC
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_VPC-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is a VPC?</b></summary>
<br>

A Virtual Private Cloud is a logically isolated virtual network within AWS where you launch resources, with full control over IP addressing, subnets, route tables, and gateways.

</details>

<details markdown="1">
<summary>❓ <b>2. What is CIDR notation and how do you choose a VPC CIDR range?</b></summary>
<br>

CIDR (Classless Inter-Domain Routing) defines an IP range and subnet mask (e.g., `10.0.0.0/16` = 65,536 IPs). Choose a range that doesn't overlap with other VPCs/on-prem networks you might connect to (via VPN/peering/Direct Connect), typically from RFC1918 private ranges, sized for future growth.

</details>

<details markdown="1">
<summary>❓ <b>3. Difference between public and private subnets.</b></summary>
<br>

A public subnet has a route to an Internet Gateway (IGW), allowing resources with public IPs to reach the internet directly. A private subnet has no direct IGW route - internet-bound traffic (if any) goes through a NAT Gateway, and resources aren't directly reachable from the internet.

</details>

<details markdown="1">
<summary>❓ <b>4. What determines whether a subnet is "public"?</b></summary>
<br>

Its route table has a route (`0.0.0.0/0`) pointing to an Internet Gateway - it's the route table association, not any inherent subnet property.

</details>

<details markdown="1">
<summary>❓ <b>5. What is a route table and how does routing work in a VPC?</b></summary>
<br>

A route table contains rules (routes) determining where network traffic is directed. Each subnet is associated with one route table; AWS uses the most specific matching route (longest prefix match) to direct traffic.

</details>

## 🚪 Gateways & Connectivity

<details markdown="1">
<summary>❓ <b>6. What is an Internet Gateway (IGW)?</b></summary>
<br>

A horizontally scaled, redundant VPC component that allows communication between resources in the VPC and the internet; it performs 1:1 NAT for instances with public IPs.

</details>

<details markdown="1">
<summary>❓ <b>7. What is a NAT Gateway and why is it needed?</b></summary>
<br>

A managed service allowing instances in private subnets to initiate outbound internet connections (e.g., for updates) while preventing unsolicited inbound connections from the internet. It resides in a public subnet and requires an Elastic IP.

</details>

<details markdown="1">
<summary>❓ <b>8. Difference between NAT Gateway and NAT Instance.</b></summary>
<br>

NAT Gateway is AWS-managed, highly available (within an AZ), scales automatically, no patching needed, but costs more. NAT Instance is a self-managed EC2 instance running NAT software - cheaper, more flexible (can double as bastion), but you manage patching/HA/scaling yourself.

</details>

<details markdown="1">
<summary>❓ <b>9. What is VPC Peering and its limitations?</b></summary>
<br>

A networking connection between two VPCs enabling private routing between them using private IPs, as if on the same network. Limitations: no transitive peering (A-B and B-C doesn't mean A-C), CIDR ranges can't overlap, and it can become complex to manage at scale (mesh of peerings).

</details>

<details markdown="1">
<summary>❓ <b>10. What is a Transit Gateway and how does it improve on VPC Peering?</b></summary>
<br>

A central hub that connects multiple VPCs and on-prem networks via a single gateway, enabling transitive routing between attached networks - avoids the full-mesh peering complexity, simplifying large-scale multi-VPC/multi-account architectures.

</details>

<details markdown="1">
<summary>❓ <b>11. What is VPC Endpoint and the difference between Gateway and Interface endpoints?</b></summary>
<br>

VPC Endpoints let you privately connect to supported AWS services without traversing the internet/NAT. Gateway Endpoints (S3, DynamoDB) are added as a route table target, free of charge. Interface Endpoints (most other services) create an ENI with a private IP in your subnet using PrivateLink, billed hourly + data processing.

</details>

<details markdown="1">
<summary>🎯 <b>12. Scenario: Instances in a private subnet need to access S3 without going through a NAT Gateway. How?</b></summary>
<br>

Create a Gateway VPC Endpoint for S3 and add it to the private subnet's route table - traffic to S3 stays within the AWS network, reducing NAT Gateway data processing costs and improving security.

</details>

## 🛡️ Security

<details markdown="1">
<summary>❓ <b>13. Difference between Security Groups and Network ACLs in depth.</b></summary>
<br>

Security Groups: stateful, instance/ENI-level, allow rules only, evaluates all rules before deciding. NACLs: stateless, subnet-level, supports allow AND deny rules, evaluated in rule number order (lowest first, first match wins), and require explicit rules for both directions since return traffic isn't automatically allowed.

</details>

<details markdown="1">
<summary>🎯 <b>14. Scenario: You want to explicitly block a malicious IP from reaching any resource in a subnet. SG or NACL?</b></summary>
<br>

NACL - because Security Groups only support Allow rules, you can't explicitly deny an IP with an SG. Add a low-numbered Deny rule in the NACL for that IP before the allow rules.

</details>

<details markdown="1">
<summary>❓ <b>15. What is a Bastion Host and how is it used securely?</b></summary>
<br>

A hardened jump server in a public subnet used to access instances in private subnets via SSH/RDP. Best practice today is to replace it with Session Manager (no open inbound ports, IAM-based access, full audit logging) but bastions are still common where SSM isn't feasible.

</details>

<details markdown="1">
<summary>❓ <b>16. What are VPC Flow Logs used for?</b></summary>
<br>

They capture metadata about IP traffic going to/from network interfaces in a VPC (source/dest IP, port, protocol, action, bytes) - used for security analysis, troubleshooting connectivity, and compliance auditing, exportable to CloudWatch Logs or S3.

</details>

## 🧠 Advanced Networking

<details markdown="1">
<summary>❓ <b>17. What is a multi-AZ VPC design and why is it recommended?</b></summary>
<br>

Deploying subnets (public/private) across multiple Availability Zones for high availability - if one AZ fails, resources in other AZs continue serving traffic, critical for production resilience.

</details>

<details markdown="1">
<summary>🎯 <b>18. Scenario: Design a 3-tier VPC architecture (web, app, DB) with high availability.</b></summary>
<br>

Create public subnets (per AZ) for ALB/NAT Gateways, private app subnets (per AZ) for application servers behind an internal or external ALB, and private/isolated DB subnets (per AZ) with no internet route, accessible only from the app tier's security group, all spread across at least 2 AZs with route tables tailored per tier.

</details>

<details markdown="1">
<summary>❓ <b>19. What is Direct Connect and how does it differ from a Site-to-Site VPN?</b></summary>
<br>

Direct Connect is a dedicated physical network connection from on-prem to AWS, offering consistent low latency/high bandwidth and no internet transit - used for large, steady-state hybrid workloads. Site-to-Site VPN is an encrypted tunnel over the public internet, quicker/cheaper to set up but with variable performance - often used as a backup to Direct Connect or for lower-traffic connections.

</details>

<details markdown="1">
<summary>❓ <b>20. What is an Elastic Network Interface (ENI) and typical use cases?</b></summary>
<br>

A virtual network card you can attach to an instance, supporting multiple IPs/security groups. Used for management network interfaces, failover (detach/reattach to another instance), or dual-homed instances in different subnets.

</details>

<details markdown="1">
<summary>❓ <b>21. What is the difference between IPv4 and IPv6 support in a VPC?</b></summary>
<br>

VPCs are IPv4 by default; you can optionally enable IPv6 (a /56 CIDR is assigned, then /64 per subnet). IGWs handle IPv6 directly (no NAT needed since IPv6 addresses are globally routable, but an "Egress-only Internet Gateway" is used for private IPv6 subnets to allow outbound-only internet access).

</details>

<details markdown="1">
<summary>❓ <b>22. What is DNS resolution/hostnames setting in a VPC?</b></summary>
<br>

Two VPC attributes: `enableDnsSupport` (whether Amazon-provided DNS resolves within the VPC) and `enableDnsHostnames` (whether instances get public DNS hostnames) - both usually need to be enabled for services like RDS/EFS/PrivateLink to resolve correctly.

</details>

## 🛠️ Troubleshooting

<details markdown="1">
<summary>🎯 <b>23. Scenario: An EC2 instance in a private subnet can't reach the internet for `yum update`. How do you troubleshoot?</b></summary>
<br>

Check: 1) Is there a NAT Gateway/Instance in the route table for `0.0.0.0/0`? 2) Is the NAT Gateway in a public subnet with a valid IGW route? 3) Security Group/NACL allow outbound HTTPS (and inbound ephemeral ports on NACL)? 4) DNS resolution working? 5) Any proxy config needed at OS level?

</details>

<details markdown="1">
<summary>🎯 <b>24. Scenario: Two peered VPCs can't communicate despite active peering connection. What could be wrong?</b></summary>
<br>

Check: 1) Route tables in both VPCs have routes pointing to the peering connection for the other VPC's CIDR. 2) Security Groups allow traffic from the peer VPC's CIDR/SG. 3) NACLs allow the traffic. 4) CIDR ranges don't overlap. 5) The peering connection status is "active," not pending/failed.

</details>

<details markdown="1">
<summary>❓ <b>25. How do you debug "Connection Timed Out" vs "Connection Refused" in AWS networking?</b></summary>
<br>

Timeout usually indicates a network-level block (SG, NACL, route table, or no path) - the packet never reaches/returns from the destination. "Connection Refused" means the packet reached the host but nothing is listening on that port (app not running, wrong port) - so it's an application-level issue, not a network ACL/SG issue.

</details>

<details markdown="1">
<summary>❓ <b>26. What tools does AWS provide to troubleshoot VPC connectivity?</b></summary>
<br>

VPC Reachability Analyzer (traces path between source/destination and identifies blocking component), VPC Flow Logs (traffic accept/reject logging), Network Access Analyzer, and traditional in-instance tools (`traceroute`, `curl`, `telnet`/`nc`).

</details>

## 🎯 Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>27. Scenario: Your company is merging two AWS environments with overlapping CIDR ranges. How do you resolve connectivity?</b></summary>
<br>

True overlapping CIDRs can't be peered directly. Options: re-IP one of the VPCs (disruptive), or use a solution like PrivateLink (service-level connectivity, no full network peering needed), or use a proxy/NAT-based translation approach for necessary connections, or migrate one environment.

</details>

<details markdown="1">
<summary>🎯 <b>28. Scenario: You need to allow a partner company on-prem network to reach specific internal services in your VPC without full network access. What's your approach?</b></summary>
<br>

Use AWS PrivateLink to expose only the specific service (via an NLB + VPC Endpoint Service) rather than a full VPN/peering that exposes broader network access - the partner creates an Interface Endpoint that connects only to the exposed service.

</details>

<details markdown="1">
<summary>❓ <b>29. How do you design a VPC to support hundreds of microservices while keeping IP address space manageable?</b></summary>
<br>

Use smaller subnet CIDRs sized appropriately for expected pod/task density (avoid oversized /24 subnets wasting IPs, but leave room for growth), use secondary CIDR blocks if needed, leverage VPC CNI custom networking (for EKS) or Fargate to reduce IP consumption per pod, and consider IPv6 for essentially unlimited address space.

</details>

<details markdown="1">
<summary>🎯 <b>30. Scenario: Compliance requires that database subnets have absolutely no route to the internet, even accidentally. How do you enforce this architecturally?</b></summary>
<br>

Don't attach an IGW route or NAT route in the DB subnet's route table at all (isolated subnet), use SCPs/AWS Config rules to detect/prevent route table modifications adding `0.0.0.0/0`, and use VPC Endpoints for any required AWS service access (S3, Secrets Manager, KMS) so the DB tier never needs internet egress.

</details>

<details markdown="1">
<summary>❓ <b>31. What is the difference between a VPC Endpoint (PrivateLink) and VPC Peering for accessing services?</b></summary>
<br>

PrivateLink/Interface Endpoints expose a single service privately without exposing the whole network (least privilege, no route table/CIDR overlap concerns). VPC Peering connects entire networks together (broader access, all resources potentially reachable subject to SG/NACL) - PrivateLink is generally preferred for service-to-service access across accounts/VPCs.

</details>

<details markdown="1">
<summary>❓ <b>32. What is Network Firewall in AWS and when would you use it over Security Groups/NACLs?</b></summary>
<br>

AWS Network Firewall is a managed, stateful, deep-packet-inspection firewall for VPC traffic supporting domain filtering, intrusion prevention (IPS) signatures, and centralized policy management across many VPCs - used when you need more advanced filtering (protocol-aware rules, threat intel feeds) beyond what SGs/NACLs provide.

</details>

<details markdown="1">
<summary>🎯 <b>33. Scenario: You need centralized egress internet control (e.g., all outbound traffic must go through a single inspected point) for dozens of VPCs. How do you architect this?</b></summary>
<br>

Use a hub-and-spoke model with Transit Gateway connecting all VPCs to a central "inspection VPC" containing AWS Network Firewall or a third-party firewall appliance, routing all egress traffic through it before reaching a shared NAT Gateway/IGW - centralizing security policy and logging.

</details>

<details markdown="1">
<summary>❓ <b>34. How does subnet sizing affect Kubernetes (EKS) networking with the VPC CNI?</b></summary>
<br>

By default, the VPC CNI assigns a VPC IP address to each pod, so subnet size directly limits pod density; undersized subnets can cause pod scheduling failures due to IP exhaustion. Solutions include custom networking with a separate CIDR for pods, prefix delegation (assigning /28 prefixes per ENI for more IPs), or switching to an overlay CNI.

</details>

<details markdown="1">
<summary>❓ <b>35. What are common cost drivers in VPC networking and how do you optimize them?</b></summary>
<br>

NAT Gateway data processing charges (per GB) - use VPC endpoints for AWS service traffic to avoid NAT; cross-AZ data transfer charges - co-locate chatty services in the same AZ where latency/HA tradeoffs allow; Interface Endpoint hourly + data charges - only use where justified; and unused/idle Elastic IPs and NAT Gateways left running.

</details>

