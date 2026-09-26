<div align="center" markdown="1">

# ⚖️ AWS ELB (Elastic Load Balancing)
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_ELB-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is Elastic Load Balancing (ELB) and why is it used?</b></summary>
<br>

A managed service that automatically distributes incoming application traffic across multiple targets (EC2 instances, containers, IPs, Lambda functions) in one or more Availability Zones, improving fault tolerance, availability, and scalability.

</details>

<details markdown="1">
<summary>❓ <b>2. What are the types of load balancers AWS offers?</b></summary>
<br>

Application Load Balancer (ALB) - Layer 7 (HTTP/HTTPS), Network Load Balancer (NLB) - Layer 4 (TCP/UDP/TLS), Gateway Load Balancer (GWLB) - for deploying/scaling third-party virtual appliances (firewalls/IDS), and Classic Load Balancer (CLB) - legacy, Layer 4/7 hybrid, not recommended for new workloads.

</details>

<details markdown="1">
<summary>❓ <b>3. When would you choose ALB vs NLB?</b></summary>
<br>

ALB: HTTP/HTTPS traffic needing content-based routing (path/host-based rules), WebSocket support, and application-layer features. NLB: extreme performance/low latency requirements, static IP/Elastic IP support, TCP/UDP traffic (non-HTTP protocols), or when preserving client source IP is critical and millions of requests/sec are expected.

</details>

<details markdown="1">
<summary>❓ <b>4. What is a Target Group?</b></summary>
<br>

A logical grouping of targets (EC2 instances, IP addresses, Lambda functions, or containers) that a load balancer routes requests to, along with health check configuration specific to that group.

</details>

## ⚖️ ALB Specifics

<details markdown="1">
<summary>❓ <b>5. What routing capabilities does ALB support?</b></summary>
<br>

Path-based routing (`/api/*` to one target group, `/images/*` to another), Host-based routing (different domains to different target groups), HTTP header/query string/method-based routing, and weighted target groups for A/B testing/blue-green deployments.

</details>

<details markdown="1">
<summary>🎯 <b>6. Scenario: You have a single ALB serving multiple microservices under different paths (`/orders`, `/users`, `/payments`). How do you configure this?</b></summary>
<br>

Create separate target groups for each microservice, then configure listener rules on the ALB with path-pattern conditions (e.g., `/orders/*` -> orders target group) so a single ALB and domain can route to multiple backend services.

</details>

<details markdown="1">
<summary>❓ <b>7. What is SSL/TLS Termination and where does it happen with ALB?</b></summary>
<br>

ALB terminates SSL/TLS at the load balancer (decrypts incoming HTTPS traffic using a certificate from ACM), then can forward traffic to targets over plain HTTP (common, reduces backend CPU load) or re-encrypt via HTTPS to targets for end-to-end encryption if required by compliance.

</details>

<details markdown="1">
<summary>❓ <b>8. How does ALB support WebSockets and HTTP/2?</b></summary>
<br>

ALB natively supports WebSocket connections (upgrading from HTTP) and HTTP/2 (including gRPC in newer configurations), enabling real-time bidirectional communication and multiplexed requests without extra configuration.

</details>

<details markdown="1">
<summary>🎯 <b>9. Scenario: You need to route gRPC traffic through a load balancer. Which one and how?</b></summary>
<br>

ALB - it supports HTTP/2 and gRPC natively; configure a target group with the "gRPC" protocol version, and ALB can route based on gRPC service/method if needed.

</details>

## 🔀 NLB Specifics

<details markdown="1">
<summary>❓ <b>10. Why does NLB provide ultra-low latency and how does it preserve source IP?</b></summary>
<br>

NLB operates at Layer 4, performing simple TCP/UDP-level load balancing without parsing application-layer data, resulting in minimal processing overhead. It preserves the client's source IP by default (passed directly to the target), unlike ALB which requires the `X-Forwarded-For` header since it terminates the connection.

</details>

<details markdown="1">
<summary>❓ <b>11. What are Static IPs/Elastic IPs with NLB used for?</b></summary>
<br>

NLB supports assigning a static IP (or Elastic IP) per Availability Zone, useful for allow-listing scenarios (firewalls/clients that require known fixed IPs) or legacy systems that can't handle DNS-based load balancer failover.

</details>

<details markdown="1">
<summary>🎯 <b>12. Scenario: A financial application requires a fixed whitelisted IP for partner connections and extremely low latency for TCP-based trading protocol. Which load balancer?</b></summary>
<br>

NLB - due to static IP support (required for IP whitelisting by the partner) and Layer 4 performance characteristics ideal for non-HTTP, latency-sensitive protocols.

</details>

## 🩺 Health Checks

<details markdown="1">
<summary>❓ <b>13. How do health checks work in ELB and what happens to unhealthy targets?</b></summary>
<br>

The load balancer periodically sends requests (HTTP/HTTPS path check, or TCP connection for NLB) to each registered target; if a target fails the configured number of consecutive checks, it's marked unhealthy and removed from active traffic rotation until it passes checks again.

</details>

<details markdown="1">
<summary>🎯 <b>14. Scenario: All targets in a target group show "unhealthy" but the application works fine when accessed directly. How do you troubleshoot?</b></summary>
<br>

Check: 1) Security Group on targets allows inbound traffic from the load balancer (or its SG). 2) The health check path returns a 200 (or configured success code) - not behind auth. 3) The health check port matches the app's actual listening port. 4) Response time is within the configured timeout. 5) NACLs allow the traffic.

</details>

## 📈 High Availability & Scaling

<details markdown="1">
<summary>❓ <b>15. How does ELB achieve high availability?</b></summary>
<br>

ELB nodes are deployed across multiple Availability Zones (you enable at least 2 AZs for the load balancer), and AWS manages the load balancer's own scaling/redundancy - if one AZ's ELB node fails, others continue to serve traffic, and DNS (Route 53) directs clients to healthy nodes.

</details>

<details markdown="1">
<summary>❓ <b>16. What is Cross-Zone Load Balancing?</b></summary>
<br>

Determines whether each load balancer node distributes traffic evenly across targets in ALL enabled AZs (cross-zone on) or only to targets within its own AZ (cross-zone off) - ALB has this on by default (free), NLB has it off by default (enabling it incurs cross-AZ data transfer charges).

</details>

<details markdown="1">
<summary>🎯 <b>17. Scenario: One AZ has significantly more targets than another, causing uneven load. What setting addresses this?</b></summary>
<br>

Enable Cross-Zone Load Balancing so that traffic is distributed evenly across all healthy targets regardless of which AZ they're in, rather than being constrained to targets in the same AZ as the receiving load balancer node.

</details>

<details markdown="1">
<summary>❓ <b>18. How does ELB integrate with Auto Scaling Groups?</b></summary>
<br>

ASGs can register/deregister instances automatically with a target group as instances launch/terminate, and ASG health checks can be configured to use ELB health check status (not just EC2 status) to replace instances that are unhealthy at the application level.

</details>

## 🛡️ Security

<details markdown="1">
<summary>❓ <b>19. How do you secure an ALB/NLB from direct access, forcing traffic through it only?</b></summary>
<br>

Configure target instances' Security Groups to only allow inbound traffic from the load balancer's Security Group (for ALB) or from the VPC CIDR (for NLB, since it doesn't have its own SG by default at the ENI level in the same way) - blocking direct internet access to instance ports.

</details>

<details markdown="1">
<summary>❓ <b>20. What is AWS WAF's relationship with ALB?</b></summary>
<br>

WAF can be directly attached to an ALB (as well as CloudFront/API Gateway) to filter malicious requests (SQLi, XSS, rate limiting, geo-blocking) before they reach backend targets - not supported on NLB since WAF operates at Layer 7.

</details>

<details markdown="1">
<summary>🎯 <b>21. Scenario: You need mutual TLS (mTLS) where the load balancer verifies client certificates. Does ALB support this?</b></summary>
<br>

Yes, ALB supports mTLS authentication, allowing it to verify client certificates against a trust store you configure (useful for B2B APIs or IoT device authentication) in addition to standard server-side TLS termination.

</details>

## 🎯 Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>22. Scenario: You're migrating from Classic Load Balancer to ALB. What should you watch out for?</b></summary>
<br>

CLB's simple round-robin differs from ALB's routing algorithm; you'll need to recreate listener rules and target groups (CLB doesn't have target groups), update any code relying on CLB-specific headers, verify SSL certificate migration to ACM, and test path/host-based routing rules since CLB doesn't support them (if you were working around this with multiple CLBs).

</details>

<details markdown="1">
<summary>🎯 <b>23. Scenario: An ALB access log shows high 5xx error rates during peak traffic. How do you diagnose the source?</b></summary>
<br>

Distinguish `ELB 5XX` (load balancer-generated, e.g., no healthy targets, timeout) from `Target 5XX` (application errors) using ALB access logs' `elb_status_code` vs `target_status_code` fields; check target group health, target CPU/memory, application logs, and connection draining/deregistration delay settings if instances are being cycled during scale-in.

</details>

<details markdown="1">
<summary>❓ <b>24. What is Connection Draining (Deregistration Delay) and why does it matter?</b></summary>
<br>

When a target is deregistered (e.g., during scale-in or unhealthy marking), the load balancer waits for a configurable period (default 300s) allowing in-flight requests to complete before fully removing the target, preventing abrupt connection termination for active users.

</details>

<details markdown="1">
<summary>🎯 <b>25. Scenario: You need to migrate traffic gradually from an old application stack to a new one with the ability to roll back instantly. How do you use ELB features?</b></summary>
<br>

Use weighted target groups on the same ALB listener rule to split traffic by percentage (e.g., 90/10) between old and new target groups, monitor error rates/latency per target group via CloudWatch, and adjust weights gradually - rollback is just reverting the weight to 100% on the old target group.

</details>

<details markdown="1">
<summary>❓ <b>26. How do sticky sessions work in ELB and when are they needed?</b></summary>
<br>

Sticky sessions (session affinity) use a cookie (ALB-generated or app-generated, ALB tracks it) to ensure a client's requests are consistently routed to the same target - needed for stateful applications storing session data locally on the instance rather than in a shared store (though the better practice is externalizing session state to ElastiCache/DynamoDB to avoid needing stickiness at all).

</details>

<details markdown="1">
<summary>🎯 <b>27. Scenario: Your load balancer needs to support both HTTP-to-HTTPS redirect and enforce modern TLS policies. How?</b></summary>
<br>

Configure an ALB listener on port 80 with a redirect action to HTTPS (port 443, "HTTPS_301" redirect), and configure the HTTPS listener with an appropriate Security Policy (SSL/TLS policy, e.g., `ELBSecurityPolicy-TLS13-1-2-2021-06`) to disable outdated protocols/ciphers (e.g., TLS 1.0/1.1) for compliance.

</details>

<details markdown="1">
<summary>❓ <b>28. What monitoring/metrics are critical for ELB in production?</b></summary>
<br>

`HealthyHostCount`/`UnHealthyHostCount`, `RequestCount`, `TargetResponseTime`, `HTTPCode_ELB_5XX_Count` and `HTTPCode_Target_5XX_Count`, `RejectedConnectionCount`, and `ActiveConnectionCount`/`NewConnectionCount` (for NLB) - these help distinguish load balancer-side issues from application/target-side issues.

</details>

<details markdown="1">
<summary>🎯 <b>29. Scenario: You need to expose an internal-only application to specific corporate VPN users, not the public internet, but still want load balancing. How?</b></summary>
<br>

Create an Internal Load Balancer (not internet-facing) - it gets a private IP/DNS name only resolvable/reachable within the VPC (and connected networks via VPN/Direct Connect/peering), providing load balancing without any public internet exposure.

</details>

<details markdown="1">
<summary>❓ <b>30. How would you design a highly available, secure ingress architecture using ELB for a containerized (ECS/EKS) application?</b></summary>
<br>

Use an ALB integrated with ECS/EKS target groups (via AWS Load Balancer Controller for EKS) with path/host-based routing per service, TLS termination via ACM, WAF attached for request filtering, deployed across multiple AZs with Auto Scaling based on request count/CPU metrics, and health checks tied to application-level readiness endpoints (not just TCP reachability).

</details>

