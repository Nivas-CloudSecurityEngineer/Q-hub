<div align="center" markdown="1">

# 🛡️ AWS WAF
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_WAF-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is AWS WAF?</b></summary>
<br>

A web application firewall that protects web applications/APIs from common exploits and bots by inspecting HTTP(S) requests, allowing you to define rules that filter/block/rate-limit traffic before it reaches your application.

</details>

<details markdown="1">
<summary>❓ <b>2. What AWS services can WAF be attached to?</b></summary>
<br>

CloudFront, Application Load Balancer (ALB), API Gateway, AWS AppSync, and Amplify - WAF operates at Layer 7 so it's not attachable to NLB.

</details>

<details markdown="1">
<summary>❓ <b>3. What is a Web ACL?</b></summary>
<br>

The top-level resource in WAF - a Web Access Control List containing an ordered set of rules (and rule groups) that define what to allow, block, or count, along with a default action for requests that don't match any rule.

</details>

<details markdown="1">
<summary>❓ <b>4. What are the possible actions for a WAF rule?</b></summary>
<br>

Allow, Block, Count (log/monitor without blocking - useful for testing rules before enforcing), and CAPTCHA/Challenge (require human/browser verification before proceeding).

</details>

## 📏 Rules & Rule Groups

<details markdown="1">
<summary>❓ <b>5. What are Managed Rule Groups and why use them?</b></summary>
<br>

Pre-configured rule sets maintained by AWS or AWS Marketplace sellers (e.g., AWS Managed Rules - Core Rule Set for OWASP Top 10, Known Bad Inputs, SQL Database, Linux/Windows OS-specific rules) - save time versus writing custom rules and are kept updated against emerging threats.

</details>

<details markdown="1">
<summary>❓ <b>6. What is the AWS Managed Core Rule Set (CRS) and what does it protect against?</b></summary>
<br>

A baseline rule group protecting against common web exploits like SQL injection, XSS, and other OWASP Top 10 vulnerabilities, providing broad protection out of the box without custom rule authoring.

</details>

<details markdown="1">
<summary>🎯 <b>7. Scenario: A managed rule group is blocking legitimate traffic (false positives) for your specific application. How do you handle this?</b></summary>
<br>

Use Rule Group override actions to set the specific problematic rule (within the managed group) to "Count" instead of "Block" (or exclude it entirely), test with Count mode to confirm behavior, and consider adding a custom rule/exception for the legitimate traffic pattern before re-enabling Block.

</details>

<details markdown="1">
<summary>❓ <b>8. What is Rate-Based Rule and a common use case?</b></summary>
<br>

A rule that tracks the request rate from a single IP (or custom aggregation key) over a 5-minute window and blocks/challenges it if it exceeds a defined threshold - commonly used to mitigate brute-force login attempts, credential stuffing, or basic DDoS/scraping behavior.

</details>

<details markdown="1">
<summary>🎯 <b>9. Scenario: Your login endpoint is being hit by a credential-stuffing attack from many different IPs (so IP rate limiting isn't fully effective). How do you use WAF to help?</b></summary>
<br>

Combine a rate-based rule (to catch high-volume single IPs) with CAPTCHA/Challenge actions on the login path to distinguish bots from humans, and consider aggregating rate limits by other keys (e.g., a custom header or cookie) rather than just source IP, plus integrating with AWS WAF Bot Control for more sophisticated bot detection.

</details>

<details markdown="1">
<summary>❓ <b>10. What is AWS WAF Bot Control?</b></summary>
<br>

A managed rule group specifically designed to identify and manage bot traffic (search engine crawlers, scrapers, scanners), letting you allow good bots (e.g., Googlebot) while blocking/challenging malicious or unwanted automated traffic.

</details>

<details markdown="1">
<summary>❓ <b>11. What is Fraud Control - Account Takeover Prevention (ATP) in WAF?</b></summary>
<br>

A specialized rule group that monitors login pages for credential stuffing and account takeover attempts, using AWS's threat intelligence on compromised credentials and behavioral signals, and can trigger CAPTCHA challenges for suspicious login attempts.

</details>

## 🗺️ Geo-Blocking & IP Filtering

<details markdown="1">
<summary>❓ <b>12. How do you block traffic from specific countries using WAF?</b></summary>
<br>

Create a Geographic Match rule specifying the country codes to block (or allow-list), using MaxMind/AWS's geo-IP database to determine the request's origin country.

</details>

<details markdown="1">
<summary>❓ <b>13. What is an IP Set in WAF and how is it used?</b></summary>
<br>

A reusable list of IP addresses/CIDR ranges that can be referenced in rules to allow or block specific IPs - useful for blocking known malicious IPs (e.g., from threat intel feeds) or allow-listing trusted partner IPs.

</details>

<details markdown="1">
<summary>🎯 <b>14. Scenario: You want to automatically block IPs that are found to be scanning for vulnerabilities (e.g., repeatedly hitting `/admin`, `/.env`, `/wp-login.php`). How?</b></summary>
<br>

Create a rule matching those known malicious paths, set the action to update an IP Set dynamically via a Lambda function triggered by WAF logs (or CloudWatch), effectively creating an automated "fail2ban"-style blocklist that grows based on detected scanning behavior.

</details>

## 🔍 Custom Rules & Request Inspection

<details markdown="1">
<summary>❓ <b>15. What request components can WAF rules inspect?</b></summary>
<br>

Headers, query strings, URI path, request body, cookies, and HTTP method - rules can match using string match, regex, SQL injection detection, XSS detection, or size constraints on any of these components.

</details>

<details markdown="1">
<summary>🎯 <b>16. Scenario: You need to block requests containing a specific malicious pattern in the request body (e.g., a known exploit payload). How?</b></summary>
<br>

Create a custom rule with a Byte Match or Regex Match statement targeting the request body, with the action set to Block - test in Count mode first to validate it doesn't affect legitimate traffic, being mindful of the body inspection size limit (default 8KB, can be increased for CloudFront/ALB).

</details>

<details markdown="1">
<summary>❓ <b>17. What is Label Matching in WAF rules?</b></summary>
<br>

Rules (especially in managed rule groups) can add "labels" to a request when matched, which subsequent rules in the same Web ACL can then check for - allowing you to build layered logic (e.g., "if this rule already labeled it as suspicious, escalate to block" instead of just relying on ordered priority alone).

</details>

## 🧾 Logging & Monitoring

<details markdown="1">
<summary>❓ <b>18. How do you monitor and analyze WAF activity?</b></summary>
<br>

Enable WAF Logging (to S3, CloudWatch Logs, or Kinesis Data Firehose) capturing full request details and which rule matched; visualize with CloudWatch metrics (`AllowedRequests`, `BlockedRequests`, per-rule counts) and dashboards, or feed logs into Athena/OpenSearch for deeper analysis.

</details>

<details markdown="1">
<summary>🎯 <b>19. Scenario: You want to test a new WAF rule in production without risking blocking legitimate traffic. What's the best practice?</b></summary>
<br>

Deploy the rule with the "Count" action first, monitor logs/metrics over a representative period (including peak traffic) to validate it only matches intended malicious traffic, then switch the action to "Block" once confidence is established.

</details>

<details markdown="1">
<summary>❓ <b>20. How do you troubleshoot WAF blocking legitimate user traffic in production?</b></summary>
<br>

Check WAF logs for the specific request (terminating IP/timestamp/path), identify which rule/rule group matched, verify if it's a true positive or false positive, and adjust (exclude the specific managed rule, add an exception/allow rule with higher priority, or refine the custom rule's match conditions).

</details>

## 🎯 Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>21. Scenario: You're building a public-facing API and need protection against SQL injection, DDoS, and scraping, without writing custom rules from scratch. How do you architect WAF?</b></summary>
<br>

Attach WAF to CloudFront/ALB/API Gateway with AWS Managed Rule Groups (Core Rule Set + SQL Database rule group for injection protection), add a rate-based rule for basic DDoS/scraping mitigation, and enable Bot Control if scraping by automated tools is a significant concern - combined with AWS Shield for network/transport layer DDoS protection.

</details>

<details markdown="1">
<summary>🎯 <b>22. Scenario: A specific internal admin tool needs to bypass WAF rules entirely for a known corporate IP range, while all other traffic goes through full inspection. How?</b></summary>
<br>

Create an IP Set with the corporate CIDR range, add a high-priority rule matching that IP Set with an "Allow" (or "Count"/bypass) action placed before the blocking rules in the Web ACL's rule evaluation order (rules are evaluated in priority order, first match for terminating actions wins).

</details>

<details markdown="1">
<summary>❓ <b>23. How does WAF rule evaluation order work, and why does it matter?</b></summary>
<br>

Rules are evaluated in the priority order you define; the first rule with a "terminating" action (Block or Allow, unless overridden) determines the outcome, so higher-priority allow-listing/bypass rules must come before broader blocking rules to take effect as intended.

</details>

<details markdown="1">
<summary>🎯 <b>24. Scenario: You need consistent WAF protection across dozens of AWS accounts/applications without configuring each one manually. How do you scale WAF management?</b></summary>
<br>

Use AWS Firewall Manager to centrally define and enforce WAF policies (Web ACLs) across multiple accounts/resources in an AWS Organization, ensuring consistent baseline protection and automatically applying policies to newly created resources.

</details>

<details markdown="1">
<summary>❓ <b>25. What is the difference between AWS WAF and AWS Shield?</b></summary>
<br>

WAF operates at Layer 7, filtering requests based on content/patterns/rate (application-layer attacks like SQLi/XSS/bot traffic). Shield protects against Layer 3/4 (network/transport layer) DDoS attacks (volumetric/protocol attacks); Shield Standard is automatic/free with CloudFront/Route 53, while Shield Advanced adds enhanced DDoS protection, cost protection, and 24/7 DDoS Response Team (DRT) access - the two are complementary layers of defense.

</details>

<details markdown="1">
<summary>🎯 <b>26. Scenario: Cost review shows high WAF charges due to logging. How do you optimize?</b></summary>
<br>

Filter WAF logs at the source (log only specific fields/redact sensitive data), use logging filters to only log Blocked/Count'd requests rather than all requests, adjust the S3/Kinesis Firehose destination's storage class/retention lifecycle, and review whether all rule groups (each incurring evaluation cost) are actually necessary for your threat model.

</details>

