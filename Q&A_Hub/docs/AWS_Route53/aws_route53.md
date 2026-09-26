<div align="center" markdown="1">

# 🌍 AWS Route 53
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_Route53-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is Amazon Route 53?</b></summary>
<br>

A highly available, scalable DNS web service that also provides domain registration, health checking, and traffic routing/management capabilities.

</details>

<details markdown="1">
<summary>❓ <b>2. What are the main features of Route 53?</b></summary>
<br>

DNS management (hosted zones/records), Domain Registration, Health Checks, Traffic Flow/Routing Policies, and DNS Resolver (for hybrid/private DNS resolution).

</details>

<details markdown="1">
<summary>❓ <b>3. What is a Hosted Zone?</b></summary>
<br>

A container for DNS records for a specific domain, defining how traffic is routed for that domain and its subdomains. Public Hosted Zones route internet traffic; Private Hosted Zones route traffic within one or more VPCs.

</details>

<details markdown="1">
<summary>❓ <b>4. What are the common DNS record types supported?</b></summary>
<br>

A (IPv4 address), AAAA (IPv6 address), CNAME (alias to another domain name, can't be used at zone apex), MX (mail servers), TXT (arbitrary text, e.g., domain verification/SPF), NS (name servers), SOA (start of authority), and Alias (AWS-specific, similar to CNAME but works at zone apex and free for AWS targets).

</details>

<details markdown="1">
<summary>❓ <b>5. Difference between a CNAME and an Alias record.</b></summary>
<br>

CNAME can't be used at the zone apex (root domain, e.g., `example.com`) per DNS standards, and every lookup incurs an extra DNS query. Alias records are an AWS extension that can be used at the apex, resolve directly to the target's IP at the DNS level (no extra latency), and are free of charge when pointing to AWS resources (ALB, CloudFront, S3, etc.).

</details>

## 🧭 Routing Policies

<details markdown="1">
<summary>❓ <b>6. What are the Route 53 routing policies?</b></summary>
<br>

Simple, Weighted, Latency-based, Failover, Geolocation, Geoproximity (with traffic flow), and Multi-value Answer.

</details>

<details markdown="1">
<summary>❓ <b>7. Explain Weighted Routing with a use case.</b></summary>
<br>

Distributes traffic across multiple resources based on assigned weights (e.g., 90/10 split) - commonly used for canary/blue-green deployments, gradually shifting traffic to a new version.

</details>

<details markdown="1">
<summary>❓ <b>8. Explain Latency-based Routing.</b></summary>
<br>

Routes users to the AWS region providing the lowest network latency, based on latency measurements between users and AWS regions - improves user experience for globally distributed applications with resources in multiple regions.

</details>

<details markdown="1">
<summary>❓ <b>9. Explain Failover Routing and its use case.</b></summary>
<br>

Routes traffic to a primary resource while it's healthy (per health checks), and automatically fails over to a secondary resource if the primary becomes unhealthy - used for active-passive DR architectures.

</details>

<details markdown="1">
<summary>❓ <b>10. Explain Geolocation vs Geoproximity Routing - difference?</b></summary>
<br>

Geolocation routes based on the user's geographic location (country/continent) - useful for content restriction/localization/compliance. Geoproximity routes based on the geographic location of resources and users, with the ability to shift traffic between resources using a "bias" value, requires Traffic Flow.

</details>

<details markdown="1">
<summary>❓ <b>11. What is Multi-value Answer routing?</b></summary>
<br>

Returns multiple IP addresses (up to 8 healthy records) in response to a DNS query, allowing basic client-side load balancing/redundancy - unlike simple routing, it can be combined with health checks to only return healthy records.

</details>

<details markdown="1">
<summary>🎯 <b>12. Scenario: You need to run a canary deployment shifting 5% of traffic to a new app version, then gradually increase it. Which routing policy?</b></summary>
<br>

Weighted routing - assign a small weight to the new version's record and a larger weight to the old version, then gradually adjust weights as confidence grows, eventually shifting 100% to the new version.

</details>

<details markdown="1">
<summary>🎯 <b>13. Scenario: You have an active-passive DR setup across two regions. How do you configure Route 53 to fail over automatically?</b></summary>
<br>

Use Failover routing policy with a Primary record pointing to the main region's endpoint and a Secondary record pointing to the DR region's endpoint, both associated with Health Checks - Route 53 automatically serves the secondary if the primary's health check fails.

</details>

## 🩺 Health Checks

<details markdown="1">
<summary>❓ <b>14. What is a Route 53 Health Check?</b></summary>
<br>

A mechanism that monitors the health/reachability of an endpoint (via HTTP/HTTPS/TCP, or CloudWatch alarm state) and can be used to influence DNS routing (excluding unhealthy resources) and trigger CloudWatch alarms/notifications.

</details>

<details markdown="1">
<summary>❓ <b>15. What types of health checks does Route 53 support?</b></summary>
<br>

Endpoint health checks (monitor a specific IP/domain), Calculated health checks (combine the status of multiple health checks with AND/OR/NOT logic), and CloudWatch alarm health checks (health based on a CloudWatch alarm's state, useful for monitoring resources without a public IP, like RDS).

</details>

<details markdown="1">
<summary>🎯 <b>16. Scenario: Your health check shows the endpoint as unhealthy, but the application is actually up. What could be wrong?</b></summary>
<br>

Common causes: Security Group/NACL blocking the Route 53 health checker IP ranges, the health check path returning a non-2xx/3xx status code, SSL/TLS certificate issues (if HTTPS health check), or the health checker's request timing out due to slow response times exceeding the configured threshold.

</details>

## 🌐 Domain Registration & DNS Management

<details markdown="1">
<summary>❓ <b>17. Can you transfer an existing domain to Route 53? What's the process?</b></summary>
<br>

Yes - initiate a transfer from your current registrar (unlock the domain, get the authorization code), then use Route 53 to request the transfer; DNS records need to be recreated/imported into a Route 53 hosted zone since transfer doesn't automatically bring DNS config.

</details>

<details markdown="1">
<summary>🎯 <b>18. Scenario: You migrated your DNS to Route 53 but want zero downtime during the cutover. What's your approach?</b></summary>
<br>

Create the hosted zone and all DNS records in Route 53 first (matching the existing DNS exactly), verify everything resolves correctly using the assigned Route 53 name servers directly, then update the domain's NS records at the registrar to point to Route 53 - propagation may take time due to TTLs, so lower TTLs in advance of the migration.

</details>

<details markdown="1">
<summary>❓ <b>19. What is a Private Hosted Zone and its use case?</b></summary>
<br>

A hosted zone that only responds to DNS queries from within specified VPCs, used for internal service discovery/naming (e.g., `db.internal.company.com`) without exposing internal hostnames/IPs to the public internet.

</details>

<details markdown="1">
<summary>🎯 <b>20. Scenario: Two VPCs in different accounts need to resolve a shared private hosted zone. How?</b></summary>
<br>

Associate the Private Hosted Zone with both VPCs using authorization - the hosted zone owner authorizes the association, then the other account's VPC owner associates their VPC with the zone (via `create-vpc-association-authorization` and `associate-vpc-with-hosted-zone`).

</details>

## 🔀 Route 53 Resolver

<details markdown="1">
<summary>❓ <b>21. What is Route 53 Resolver and Resolver Endpoints?</b></summary>
<br>

Route 53 Resolver answers DNS queries for VPC resources by default (via the `.2` reserved IP). Resolver Endpoints (inbound/outbound) enable hybrid DNS resolution: Inbound endpoints let on-prem systems query AWS private DNS; Outbound endpoints let AWS resources resolve on-prem DNS via conditional forwarding rules.

</details>

<details markdown="1">
<summary>🎯 <b>22. Scenario: On-prem servers need to resolve AWS private hosted zone records, and EC2 instances need to resolve on-prem domain names. How do you architect this?</b></summary>
<br>

Set up a Route 53 Resolver Inbound Endpoint (on-prem queries AWS via this endpoint over VPN/Direct Connect) and an Outbound Endpoint with Resolver Rules forwarding queries for the on-prem domain to on-prem DNS servers - enabling bidirectional hybrid DNS resolution.

</details>

## 🎯 Troubleshooting & Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>23. Scenario: DNS changes aren't reflecting for some users after updating a record. Why?</b></summary>
<br>

DNS caching/TTL - resolvers and clients cache records for the duration of the TTL. If TTL was high (e.g., 24 hours) before the change, some users/resolvers will continue seeing the old value until their cache expires. Lower TTL in advance of planned changes to reduce propagation delay.

</details>

<details markdown="1">
<summary>❓ <b>24. How do you troubleshoot a domain that isn't resolving at all?</b></summary>
<br>

Check `dig`/`nslookup` for NS records matching Route 53's assigned name servers, verify the hosted zone has the correct records, check domain registration status/expiration, verify no typos in record names, and check if DNSSEC (if enabled) is misconfigured.

</details>

<details markdown="1">
<summary>❓ <b>25. What is DNSSEC and does Route 53 support it?</b></summary>
<br>

DNSSEC adds cryptographic signatures to DNS records to prevent spoofing/cache poisoning attacks. Route 53 supports DNSSEC signing for public hosted zones and can validate DNSSEC for domains registered elsewhere.

</details>

<details markdown="1">
<summary>🎯 <b>26. Scenario: You want to block users from certain countries from reaching your application at the DNS level. Is Route 53 Geolocation sufficient, or do you need more?</b></summary>
<br>

Geolocation routing can direct blocked countries to a "no service" endpoint or return NXDOMAIN, but it's not a complete security control since users can bypass DNS-level blocks (using a different DNS resolver, VPN, or direct IP access) - combine with WAF geo-blocking rules and/or CloudFront geo-restriction for actual enforcement.

</details>

<details markdown="1">
<summary>❓ <b>27. What is the difference between Alias records to an ALB vs a CloudFront distribution, cost/behavior-wise?</b></summary>
<br>

Both are free Alias record types with no extra DNS query overhead. Functionally, an ALB alias routes directly to regional load balancer IPs, while a CloudFront alias routes to the nearest edge location globally - choice depends on whether you need a CDN/global caching layer or direct regional load balancing.

</details>

<details markdown="1">
<summary>🎯 <b>28. Scenario: You need blue/green deployment across two independent environments with instant rollback capability at the DNS layer. How do you use Route 53?</b></summary>
<br>

Use Weighted routing with two record sets (blue/green) each pointing to its environment; shift weight to 100/0 for a hard cutover with instant rollback (just flip weights back) - combined with low TTL for faster switch propagation, and health checks to prevent Route 53 from serving an unhealthy target.

</details>

<details markdown="1">
<summary>❓ <b>29. What is Traffic Flow in Route 53?</b></summary>
<br>

A visual tool for creating complex routing configurations combining multiple routing policies (e.g., geoproximity + failover + weighted) into a single traffic policy, versioned and reusable across multiple hosted zones.

</details>

<details markdown="1">
<summary>🎯 <b>30. Scenario: Applications need service discovery for dynamically scaling microservices/containers (no fixed IPs). How does Route 53 help?</b></summary>
<br>

AWS Cloud Map (which integrates with Route 53) provides service discovery for dynamic resources - services register/deregister automatically (e.g., ECS/EKS services), and Route 53 private DNS records are updated automatically so consumers can resolve current, healthy endpoints instead of hardcoded IPs.

</details>

<details markdown="1">
<summary>❓ <b>31. What's the difference between "Simple" and "Multivalue Answer" routing when both can return multiple records?</b></summary>
<br>

Simple routing can list multiple values but doesn't support health checks - all values are always returned regardless of health, and typically the client picks one/first. Multivalue Answer explicitly supports health checks (only healthy records returned, up to 8 random ones per query), giving basic DNS-level load balancing and failover.

</details>

<details markdown="1">
<summary>🎯 <b>32. Scenario: Cost review shows high Route 53 charges. What contributes to Route 53 cost and how do you optimize?</b></summary>
<br>

Costs come from hosted zones (per zone/month), DNS queries (per million, cheaper for Alias-to-AWS-resource queries which are free), health checks (per check, more for global/HTTPS checks), and Resolver endpoints. Optimize by consolidating unnecessary hosted zones, using Alias records instead of CNAMEs to AWS resources, and removing unused health checks.

</details>

