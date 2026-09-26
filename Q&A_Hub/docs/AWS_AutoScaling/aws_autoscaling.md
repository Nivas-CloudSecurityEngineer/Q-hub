<div align="center" markdown="1">

# 📈 AWS Auto Scaling
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_AutoScaling-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is AWS Auto Scaling?</b></summary>
<br>

A service that automatically adjusts compute capacity (EC2 instances, ECS tasks, DynamoDB throughput, etc.) up or down based on demand, health, or a schedule, ensuring application availability while optimizing cost.

</details>

<details markdown="1">
<summary>❓ <b>2. What is an Auto Scaling Group (ASG)?</b></summary>
<br>

A logical collection of EC2 instances treated as a scalable unit, defined by a Launch Template/Configuration, minimum/maximum/desired capacity, target subnets/AZs, and scaling policies.

</details>

<details markdown="1">
<summary>❓ <b>3. Difference between Launch Template and Launch Configuration.</b></summary>
<br>

Launch Configuration is the legacy way to define instance settings for an ASG (immutable, one active version, no support for newer features like multiple instance types or Spot). Launch Template is the modern, recommended approach, supporting versioning, mixed instance types/purchase options, and newer EC2 features - AWS recommends Launch Templates for all new ASGs.

</details>

<details markdown="1">
<summary>❓ <b>4. What are Min, Max, and Desired capacity in an ASG?</b></summary>
<br>

Min: the minimum number of instances the ASG will never go below. Max: the ceiling it will never exceed. Desired: the target number of instances the ASG tries to maintain at any given time (adjusted automatically by scaling policies within min/max bounds).

</details>

## 📐 Scaling Policies

<details markdown="1">
<summary>❓ <b>5. What are the types of scaling policies?</b></summary>
<br>

Target Tracking (maintain a specific metric value, e.g., 50% average CPU - simplest, recommended for most cases), Step Scaling (scale by different amounts based on the magnitude of alarm breach), Simple Scaling (single scaling adjustment per alarm, with a cooldown period), and Scheduled Scaling (scale at specific predictable times).

</details>

<details markdown="1">
<summary>❓ <b>6. Explain Target Tracking Scaling with an example.</b></summary>
<br>

You specify a target value for a metric (e.g., "keep average CPU utilization at 50%"), and Auto Scaling automatically creates and manages the CloudWatch alarms and adjusts capacity to maintain that target - simpler than manually defining thresholds/step adjustments.

</details>

<details markdown="1">
<summary>🎯 <b>7. Scenario: Traffic to your e-commerce site spikes predictably every day at 9 AM and drops at 11 PM. How do you configure Auto Scaling?</b></summary>
<br>

Use Scheduled Scaling to increase the minimum/desired capacity ahead of the 9 AM spike and decrease it after 11 PM, potentially combined with Target Tracking as a safety net to handle any unexpected additional demand within that window.

</details>

<details markdown="1">
<summary>❓ <b>8. What is a Cooldown Period and why is it important?</b></summary>
<br>

A period after a scaling activity during which the ASG suspends further scaling actions (for simple/step scaling) to allow newly launched instances time to start handling load before evaluating whether more scaling is needed - prevents over-aggressive scaling from lag in metrics reflecting new capacity.

</details>

<details markdown="1">
<summary>🎯 <b>9. Scenario: Your ASG is scaling out and in repeatedly within a short time (flapping). How do you fix this?</b></summary>
<br>

Increase the cooldown period, use a target tracking policy (which has smarter built-in logic than step/simple scaling), widen the target metric's acceptable range, or check if the metric being scaled on (e.g., CPU) is genuinely representative of load, or if a more suitable/custom metric (like request count per target) should be used instead.

</details>

<details markdown="1">
<summary>❓ <b>10. What is Predictive Scaling?</b></summary>
<br>

A feature that uses machine learning to forecast future traffic patterns based on historical data, proactively scaling capacity ahead of anticipated demand rather than purely reactively, useful for workloads with recurring patterns and long instance boot times.

</details>

## 🩺 Health Checks & Instance Management

<details markdown="1">
<summary>❓ <b>11. How does an ASG determine if an instance is unhealthy?</b></summary>
<br>

Via EC2 status checks (default), or ELB health checks if configured (recommended for load-balanced ASGs, since it checks application-level health, not just instance-level) - unhealthy instances are automatically terminated and replaced.

</details>

<details markdown="1">
<summary>🎯 <b>12. Scenario: An instance is failing ELB health checks but passing EC2 status checks. What happens, and why might this occur?</b></summary>
<br>

The ASG will terminate and replace it since ELB health check type takes precedence when configured (application is unreachable even though the instance itself is running) - could be caused by the app crashing, a misconfigured security group, or the app taking too long to become ready (needs Health Check Grace Period tuning).

</details>

<details markdown="1">
<summary>❓ <b>13. What is the Health Check Grace Period?</b></summary>
<br>

A configurable delay after an instance launches, during which failed health checks are ignored - gives the application/instance time to boot and initialize (install packages, start services) before being judged as unhealthy and prematurely terminated/replaced.

</details>

<details markdown="1">
<summary>❓ <b>14. What is Instance Refresh in an ASG?</b></summary>
<br>

A feature to roll out changes (new AMI/launch template version) to all instances in an ASG in a controlled way - gradually replacing old instances with new ones based on a minimum healthy percentage, rather than manually terminating instances one by one.

</details>

<details markdown="1">
<summary>🎯 <b>15. Scenario: You need to deploy a new AMI to your ASG with zero downtime and automatic rollback if health checks fail. How?</b></summary>
<br>

Use Instance Refresh with a defined minimum healthy percentage (e.g., 90%) and warm-up time, monitor the rollout, and if failures occur, cancel the refresh (or configure automatic rollback based on alarms) to revert to the previous launch template version.

</details>

## ⏹️ Termination Policies & Lifecycle Hooks

<details markdown="1">
<summary>❓ <b>16. What is a Termination Policy in ASG?</b></summary>
<br>

Determines which instance is chosen for termination during scale-in (e.g., `OldestInstance`, `NewestInstance`, `ClosestToNextInstanceHour`, `AllocationStrategy` for Spot/mixed instances, or `Default` which balances across AZs and picks the oldest launch template/config first).

</details>

<details markdown="1">
<summary>❓ <b>17. What are Lifecycle Hooks and when would you use them?</b></summary>
<br>

Hooks that pause an instance in a Pending or Terminating state before it enters service or is terminated, giving you time to perform custom actions (e.g., run configuration/bootstrap scripts, or drain connections/upload logs before termination) via a Lambda/SQS/SNS notification before completing the transition.

</details>

<details markdown="1">
<summary>🎯 <b>18. Scenario: Before an instance is terminated during scale-in, you need to ensure it finishes processing in-flight jobs and uploads logs to S3. How?</b></summary>
<br>

Configure a Terminating lifecycle hook that pauses termination, triggers a Lambda/SSM automation via SNS/EventBridge to drain jobs and upload logs, then calls `CompleteLifecycleAction` to allow the termination to proceed (with a timeout/heartbeat to avoid stalling indefinitely).

</details>

## 💰 Multi-AZ, Spot & Cost Optimization

<details markdown="1">
<summary>❓ <b>19. How does Auto Scaling maintain balance across Availability Zones?</b></summary>
<br>

By default, ASGs attempt to distribute instances evenly across all configured AZs/subnets, and will launch replacement instances in a way that maintains this balance (subject to available capacity).

</details>

<details markdown="1">
<summary>❓ <b>20. What is a Mixed Instances Policy?</b></summary>
<br>

Allows an ASG to launch a combination of different instance types and purchase options (On-Demand + Spot) within the same group, improving availability (diversification reduces risk of Spot interruption/capacity issues) and cost optimization.

</details>

<details markdown="1">
<summary>🎯 <b>21. Scenario: You want to run a cost-optimized, fault-tolerant web tier using a mix of Spot and On-Demand instances. How do you configure this?</b></summary>
<br>

Use a Mixed Instances Policy specifying a base On-Demand capacity (e.g., minimum guaranteed baseline) plus an On-Demand/Spot split percentage for additional capacity, with multiple instance types specified to increase Spot capacity pool diversity and reduce simultaneous interruption risk.

</details>

<details markdown="1">
<summary>❓ <b>22. What happens to an ASG-managed Spot Instance when it receives an interruption notice?</b></summary>
<br>

AWS provides a 2-minute warning; the ASG detects the impending interruption and proactively launches a replacement instance (potentially in a different AZ/instance type per the mixed instances policy) to maintain desired capacity before or as the Spot instance is reclaimed.

</details>

## 🎯 Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>23. Scenario: After a scale-out event, new instances take 5 minutes to become fully ready (bootstrapping), causing premature health check failures. How do you fix?</b></summary>
<br>

Increase the Health Check Grace Period to accommodate the bootstrap time, and/or bake a Golden AMI with pre-installed dependencies (via Packer) to reduce actual boot/readiness time instead of relying purely on User Data scripts at launch.

</details>

<details markdown="1">
<summary>🎯 <b>24. Scenario: You need to scale based on a custom application metric (e.g., queue depth) rather than CPU. How?</b></summary>
<br>

Push the custom metric (e.g., SQS `ApproximateNumberOfMessagesVisible`, which is already a CloudWatch metric) and create a Target Tracking or Step Scaling policy referencing that metric/alarm instead of the default CPU-based tracking - common for worker/queue-processing architectures.

</details>

<details markdown="1">
<summary>❓ <b>25. How do you prevent an ASG from scaling in during a critical batch job on a specific instance?</b></summary>
<br>

Use Instance Protection (scale-in protection) on that specific instance, which excludes it from being selected for termination during scale-in events until protection is removed.

</details>

<details markdown="1">
<summary>🎯 <b>26. Scenario: You're running a stateful legacy app that can only run as a single instance, but need auto-recovery on failure. Is ASG appropriate?</b></summary>
<br>

Yes - set Min=Max=Desired=1; the ASG will automatically detect instance failure (via health checks) and launch a replacement, effectively providing self-healing for a single-instance workload without needing full horizontal scaling logic.

</details>

<details markdown="1">
<summary>❓ <b>27. What's the difference between ASG scaling EC2 instances and Application Auto Scaling for other services?</b></summary>
<br>

EC2 Auto Scaling specifically manages EC2 instance fleets in an ASG. Application Auto Scaling is a broader framework that manages scaling for other AWS resources (ECS services, DynamoDB tables, Aurora replicas, Lambda provisioned concurrency, etc.) using similar target-tracking/step-scaling concepts adapted to each service's scalable dimension.

</details>

<details markdown="1">
<summary>🎯 <b>28. Scenario: How do you test that your Auto Scaling configuration correctly handles a sudden 5x traffic spike before it happens in production?</b></summary>
<br>

Use load testing tools (e.g., Locust, JMeter, or AWS Distributed Load Testing) against a staging environment mirroring production scaling config, observe scale-out speed/health check behavior/application performance during the ramp-up, and tune cooldowns/warm-up times/AMI boot time based on results.

</details>

<details markdown="1">
<summary>❓ <b>29. How does Auto Scaling Group interact with a Load Balancer's Target Group during scale-in, specifically around active connections?</b></summary>
<br>

The ASG first deregisters the instance from the target group (marking it "draining"), waits for the deregistration delay (connection draining period) to let in-flight requests finish, and only then proceeds to terminate the instance - preventing abrupt disruption of active user sessions.

</details>

<details markdown="1">
<summary>🎯 <b>30. Scenario: Cost review shows the ASG frequently runs near max capacity, throttling growth during peak periods. What do you check and adjust?</b></summary>
<br>

Review CloudWatch scaling activity history for repeated max-capacity hits, check if Max is set too conservatively (raise it, checking service/vCPU quotas), verify alarms/target tracking thresholds are appropriately tuned, and evaluate whether predictive scaling or scheduled scaling could pre-provision capacity ahead of known peak periods instead of reacting after the threshold is hit.

</details>

