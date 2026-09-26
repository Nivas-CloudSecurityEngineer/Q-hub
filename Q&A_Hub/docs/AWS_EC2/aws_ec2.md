<div align="center" markdown="1">

# 🖥️ AWS EC2
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_EC2-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is EC2?</b></summary>
<br>

Elastic Compute Cloud (EC2) is an AWS service providing resizable virtual servers (instances) in the cloud, billed by usage, letting you run applications without owning physical hardware.

</details>

<details markdown="1">
<summary>❓ <b>2. What are AMIs?</b></summary>
<br>

Amazon Machine Images are templates containing the OS, application server, and applications used to launch EC2 instances. You can use AWS-provided, Marketplace, or custom AMIs (e.g., baked with Packer).

</details>

<details markdown="1">
<summary>❓ <b>3. What are the EC2 instance purchasing options?</b></summary>
<br>

On-Demand (pay per hour/second, no commitment), Reserved Instances (1/3-year commitment for discount), Savings Plans (flexible commitment-based discount), Spot Instances (spare capacity at steep discount, can be interrupted), Dedicated Hosts/Instances (physical isolation).

</details>

<details markdown="1">
<summary>❓ <b>4. Difference between Spot, On-Demand, and Reserved Instances - when to use each?</b></summary>
<br>

On-Demand: unpredictable/short-term workloads. Reserved/Savings Plans: steady-state, predictable long-running workloads for cost savings. Spot: fault-tolerant, flexible workloads (batch processing, CI runners, stateless web tiers) where interruption is acceptable, at up to 90% discount.

</details>

<details markdown="1">
<summary>❓ <b>5. What are instance types/families and how do you choose one?</b></summary>
<br>

Instance types are grouped by family optimized for use case: General purpose (T/M - balanced), Compute optimized (C - CPU-heavy), Memory optimized (R/X - in-memory DBs, caching), Storage optimized (I/D - high IOPS/throughput), Accelerated computing (P/G - GPU/ML). Choose based on the workload's CPU/memory/network/storage profile.

</details>

## 💾 Storage

<details markdown="1">
<summary>❓ <b>6. Difference between EBS and Instance Store.</b></summary>
<br>

EBS (Elastic Block Store) is persistent, network-attached block storage that survives instance stop/terminate (unless configured otherwise) and can be detached/reattached. Instance Store is ephemeral, physically attached storage that's lost on stop/terminate - used for temporary data/caches needing very high IOPS.

</details>

<details markdown="1">
<summary>❓ <b>7. What are the EBS volume types?</b></summary>
<br>

gp3/gp2 (general purpose SSD), io1/io2 (provisioned IOPS SSD, for high-performance DBs), st1 (throughput-optimized HDD, big data/logs), sc1 (cold HDD, infrequent access, cheapest).

</details>

<details markdown="1">
<summary>❓ <b>8. Can an EBS volume be attached to multiple instances?</b></summary>
<br>

Standard EBS volumes attach to a single instance at a time (except io1/io2 with Multi-Attach enabled, which allows attachment to multiple instances in the same AZ for clustered applications, but requires cluster-aware filesystems).

</details>

<details markdown="1">
<summary>❓ <b>9. How do you increase the size of an EBS volume without downtime?</b></summary>
<br>

Modify the volume size via console/CLI (`aws ec2 modify-volume`) while the instance is running, then extend the filesystem/partition on the OS (`growpart` + `resize2fs`/`xfs_growfs`) - no reboot needed for most modern Linux setups.

</details>

<details markdown="1">
<summary>🎯 <b>10. Scenario: You need to recover data from a terminated instance's EBS volume. Is this possible?</b></summary>
<br>

Only if "Delete on Termination" was disabled for the root volume, or if it was a non-root volume (which by default isn't deleted). If enabled and deleted, recovery is only possible from snapshots - which is why regular EBS snapshots are critical.

</details>

## 🌐 Networking & Security

<details markdown="1">
<summary>❓ <b>11. What is a Security Group and how does it differ from a NACL?</b></summary>
<br>

A Security Group is a stateful virtual firewall at the instance/ENI level - allows rules only (return traffic automatically permitted). A NACL is a stateless firewall at the subnet level, supporting both allow and deny rules, and requires explicit rules for both inbound and outbound/return traffic.

</details>

<details markdown="1">
<summary>❓ <b>12. What is an Elastic IP and when would you use one?</b></summary>
<br>

A static public IPv4 address you can allocate and associate with an instance, persisting even through stop/start (unlike the default public IP which changes). Used when you need a fixed IP for DNS/whitelisting purposes.

</details>

<details markdown="1">
<summary>🎯 <b>13. Scenario: You can SSH to an instance from your office but not from home. What could be wrong?</b></summary>
<br>

Likely the Security Group's inbound rule restricts SSH (port 22) to a specific IP/CIDR (office IP) - it needs updating to include the home IP, or use a bastion host/VPN/Session Manager instead of opening SSH broadly.

</details>

<details markdown="1">
<summary>❓ <b>14. What is instance metadata and what is IMDSv2?</b></summary>
<br>

Instance metadata (accessible via `169.254.169.254`) provides info about the instance (IAM role credentials, instance ID, etc.) to apps running on it. IMDSv2 requires session-oriented requests (token-based, PUT then GET) instead of simple GET requests (IMDSv1), mitigating SSRF attacks that could steal instance role credentials.

</details>

<details markdown="1">
<summary>❓ <b>15. How do you connect to an EC2 instance without opening SSH port 22 to the internet?</b></summary>
<br>

Use AWS Systems Manager Session Manager, which uses the SSM agent and IAM permissions to establish a secure shell session without any inbound ports open, or use a bastion host in a public subnet with tightly scoped SG rules, or a VPN/Direct Connect.

</details>

## 📈 High Availability & Scaling

<details markdown="1">
<summary>❓ <b>16. What is the difference between stopping, terminating, and hibernating an instance?</b></summary>
<br>

Stop: shuts down the instance, EBS-backed data persists, you stop paying for compute but still pay for EBS. Terminate: instance is permanently deleted, root EBS volume deleted by default. Hibernate: saves RAM contents to the EBS root volume so the instance can resume exactly where it left off (faster than a cold boot with warm memory state).

</details>

<details markdown="1">
<summary>❓ <b>17. How does EC2 Auto Recovery work?</b></summary>
<br>

CloudWatch can monitor instance status checks and automatically recover (stop/start) an instance on a new host if the underlying hardware fails, preserving instance ID, IP, metadata - useful for stateful single instances without HA architecture.

</details>

<details markdown="1">
<summary>❓ <b>18. What is a placement group and what are its types?</b></summary>
<br>

Controls how instances are placed on underlying hardware. Cluster: packs instances close together in one AZ for low latency/high throughput (HPC). Spread: spreads instances across distinct hardware to reduce correlated failure risk (small number of critical instances). Partition: groups instances into logical partitions with separate hardware, used for large distributed systems (Hadoop/Kafka/Cassandra).

</details>

<details markdown="1">
<summary>🎯 <b>19. Scenario: Your application on EC2 needs to scale based on demand. How would you design this?</b></summary>
<br>

Put instances in an Auto Scaling Group across multiple AZs behind an Application Load Balancer, define scaling policies (target tracking on CPU/requests, or step scaling on CloudWatch alarms), use a Launch Template with a baked AMI or user-data bootstrap, and make the app stateless (session data in ElastiCache/DynamoDB, not local disk).

</details>

## 🥾 User Data & Bootstrapping

<details markdown="1">
<summary>❓ <b>20. What is EC2 User Data used for?</b></summary>
<br>

A script/cloud-init config passed at launch time that runs once on first boot, used to bootstrap the instance (install packages, configure services, join a cluster, fetch app code).

</details>

<details markdown="1">
<summary>❓ <b>21. Difference between "baking" an AMI vs using User Data scripts for configuration.</b></summary>
<br>

Baking (Golden AMI via Packer) pre-installs everything into the image for fast, consistent, immutable boot (better for scaling speed and consistency). User Data configures at boot time (more flexible/dynamic but slower start and dependent on external repos/networking being available at boot).

</details>

## 🔍 Monitoring & Troubleshooting

<details markdown="1">
<summary>🎯 <b>22. Scenario: An EC2 instance fails status checks. How do you troubleshoot?</b></summary>
<br>

Distinguish system status check failures (AWS infrastructure issue - fix by stop/start to move to new host) vs instance status check failures (OS-level issue - check via EC2 Serial Console/system logs, kernel panic, disk full, misconfigured network). Use `Get System Log` and `EC2 Serial Console` for boot-level diagnostics.

</details>

<details markdown="1">
<summary>❓ <b>23. How do you monitor EC2 performance and what metrics matter?</b></summary>
<br>

CloudWatch provides CPUUtilization, NetworkIn/Out, DiskReadOps/WriteOps, StatusCheckFailed by default (5-min, or 1-min with detailed monitoring). Memory and disk usage require the CloudWatch Agent since these aren't tracked natively.

</details>

<details markdown="1">
<summary>🎯 <b>24. Scenario: An EC2-hosted app suddenly becomes unreachable. What's your triage process?</b></summary>
<br>

Check instance status checks in console. Check Security Group/NACL rules haven't changed. Check the app process/service status via Session Manager. Check CloudWatch metrics (CPU/memory/disk spikes). Check recent deployments/changes. Check route tables/IGW/NAT if it's a networking-layer issue.

</details>

<details markdown="1">
<summary>❓ <b>25. What is the difference between basic and detailed CloudWatch monitoring for EC2?</b></summary>
<br>

Basic monitoring provides metrics at 5-minute intervals for free; detailed monitoring provides 1-minute granularity for a small cost - useful for faster autoscaling reaction time and more precise troubleshooting.

</details>

## 🎯 Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>26. Scenario: You need to migrate an on-prem application to EC2 with minimal downtime. What's your approach?</b></summary>
<br>

Use AWS Application Migration Service (MGN) or CloudEndure for continuous block-level replication, test with a non-disruptive drill/cutover instance, then perform final cutover during a maintenance window, updating DNS to point to the new EC2-hosted app.

</details>

<details markdown="1">
<summary>❓ <b>27. How do you patch EC2 instances at scale securely?</b></summary>
<br>

Use AWS Systems Manager Patch Manager to define patch baselines and maintenance windows, patch instances in batches (rolling across AZs to maintain availability), and use Patch Compliance reports to track status.

</details>

<details markdown="1">
<summary>🎯 <b>28. Scenario: Cost review shows EC2 spend is much higher than expected. How do you investigate and optimize?</b></summary>
<br>

Use Cost Explorer/Trusted Advisor to identify idle/underutilized instances (low CPU), right-size instance types, move steady workloads to Reserved Instances/Savings Plans, use Spot for fault-tolerant workloads, stop non-prod instances outside business hours (via Instance Scheduler), and check for orphaned resources (unattached EBS volumes, unused Elastic IPs).

</details>

<details markdown="1">
<summary>❓ <b>29. What is EC2 Spot Instance interruption and how do applications handle it gracefully?</b></summary>
<br>

AWS can reclaim Spot capacity with a 2-minute warning (via instance metadata/CloudWatch event). Applications should listen for this termination notice and checkpoint work, drain connections, or let the ASG replace the instance, making workloads stateless/fault-tolerant (e.g., using Spot Fleet/mixed instance policies for resilience).

</details>

<details markdown="1">
<summary>🎯 <b>30. Scenario: You need to run a stateful legacy application that can't be clustered, but requires high availability. What EC2-level options exist?</b></summary>
<br>

Use Auto Recovery (CloudWatch alarm-based) to recover on hardware failure while preserving instance state/IP, deploy across Multi-AZ with a warm standby instance and failover automation (Route 53 health checks), and take frequent EBS snapshots for backup/DR.

</details>

<details markdown="1">
<summary>❓ <b>31. What is the difference between an ENI (Elastic Network Interface) and using multiple IPs on the same NIC?</b></summary>
<br>

An ENI is a virtual network card that can be attached/detached across instances (useful for failover, low-level network appliances). A single ENI can also have multiple private IPs assigned, useful for hosting multiple SSL certs/services without extra ENIs, subject to instance ENI/IP limits per instance type.

</details>

<details markdown="1">
<summary>❓ <b>32. How does EC2 pricing differ between different regions, and how does that affect architecture decisions?</b></summary>
<br>

Pricing varies by region due to infrastructure/operating costs and demand; teams often choose regions balancing latency to users, compliance/data residency requirements, and cost - sometimes deploying non-latency-sensitive workloads (like batch processing) in cheaper regions.

</details>

<details markdown="1">
<summary>🎯 <b>33. Scenario: You need zero-downtime deployment for an app running on EC2 instances behind an ALB. How do you achieve this?</b></summary>
<br>

Use a Blue/Green deployment strategy: launch a new ASG/instances with the updated version, verify health checks pass, gradually shift ALB target group traffic (or swap target groups), then terminate old instances - or use rolling deployment via ASG instance refresh with a minimum healthy percentage.

</details>

<details markdown="1">
<summary>❓ <b>34. What are EC2 Dedicated Hosts and when are they required?</b></summary>
<br>

Physical servers dedicated entirely to your use, giving visibility/control over physical cores/sockets - required for licensing that's tied to physical cores/sockets (e.g., some Windows Server/SQL Server BYOL licenses) or strict compliance requiring physical isolation.

</details>

<details markdown="1">
<summary>❓ <b>35. How do you secure SSH access and credentials for EC2 at scale in an organization?</b></summary>
<br>

Avoid distributing private keys broadly; use EC2 Instance Connect or Session Manager for auditable, keyless, IAM-controlled access, rotate/manage key pairs centrally, disable password authentication, and enforce SGs allowing SSH only from bastion/VPN CIDR ranges, plus CloudTrail logging of all session activity.

</details>

