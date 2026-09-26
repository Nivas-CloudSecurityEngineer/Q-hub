<div align="center" markdown="1">

# 🐳 AWS ECS (Elastic Container Service)
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_ECS-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is Amazon ECS?</b></summary>
<br>

A fully managed container orchestration service that lets you run, stop, and manage Docker containers on a cluster of EC2 instances or serverlessly via Fargate, handling scheduling, scaling, and integration with load balancers/service discovery/IAM.

</details>

<details markdown="1">
<summary>❓ <b>2. What are the core building blocks of ECS?</b></summary>
<br>

Cluster (logical grouping of resources), Task Definition (blueprint describing containers, CPU/memory, networking, IAM roles - like a docker-compose spec), Task (a running instance of a task definition), and Service (maintains a desired number of running tasks, handles load balancer registration and rolling deployments).

</details>

<details markdown="1">
<summary>❓ <b>3. Difference between EC2 launch type and Fargate launch type.</b></summary>
<br>

EC2 launch type runs tasks on a cluster of self-managed EC2 instances (you manage capacity/patching/scaling of the underlying instances, but have more control and potentially lower cost at scale/GPU support). Fargate is serverless - AWS manages the underlying infrastructure entirely, you just specify CPU/memory per task, simplifying operations but at a higher per-unit cost and less low-level control.

</details>

<details markdown="1">
<summary>❓ <b>4. What is a Task Definition and what does it specify?</b></summary>
<br>

A JSON blueprint specifying one or more container definitions (image, CPU/memory, port mappings, environment variables, secrets, logging config), the Task Role (IAM permissions for the application), the Task Execution Role (permissions for ECS agent to pull images/write logs), network mode, and launch type compatibility.

</details>

## 📋 Task Definitions & Roles

<details markdown="1">
<summary>❓ <b>5. Difference between Task Role and Task Execution Role.</b></summary>
<br>

Task Role grants permissions to the application code running INSIDE the container (e.g., to call S3/DynamoDB). Task Execution Role grants permissions to the ECS agent itself to perform actions ON BEHALF of the task before/during startup (e.g., pulling the image from ECR, fetching secrets from Secrets Manager, writing logs to CloudWatch) - a common interview trip-up is confusing these two.

</details>

<details markdown="1">
<summary>🎯 <b>6. Scenario: A container can't access Secrets Manager to retrieve a database password injected as a secret. Which role needs the permission?</b></summary>
<br>

The Task Execution Role - since ECS Agent (not the application) is responsible for retrieving the secret value and injecting it as an environment variable before the container starts; the Task Role would need permission only if the application itself calls Secrets Manager's API directly at runtime.

</details>

<details markdown="1">
<summary>❓ <b>7. What are the networking modes available for ECS tasks?</b></summary>
<br>

`awsvpc` (each task gets its own ENI and private IP, required for Fargate, supports Security Groups per task), `bridge` (Docker's default virtual network on the host, EC2 launch type only), `host` (container shares the host's network namespace directly, EC2 only), and `none` (no external networking).

</details>

<details markdown="1">
<summary>❓ <b>8. Why is `awsvpc` mode recommended/required, and what are its benefits?</b></summary>
<br>

It gives each task a dedicated ENI with its own IP and Security Group, providing network isolation at the task level (rather than sharing the host's network stack), enabling fine-grained security group rules per service and consistent networking behavior between EC2 and Fargate launch types - it's mandatory for Fargate.

</details>

## 🚀 Services & Deployments

<details markdown="1">
<summary>❓ <b>9. What is an ECS Service and how does it maintain desired state?</b></summary>
<br>

A Service ensures a specified number of task instances are running at all times, automatically replacing failed/unhealthy tasks, integrating with a load balancer's target group for traffic distribution, and supporting deployment strategies for updates.

</details>

<details markdown="1">
<summary>❓ <b>10. What deployment strategies does ECS support for Services?</b></summary>
<br>

Rolling Update (default - gradually replaces old tasks with new ones based on min/max healthy percent), Blue/Green (via CodeDeploy integration - shifts traffic between two separate task sets, enabling instant rollback), and External (managed entirely outside ECS's built-in deployment controller, e.g., via a custom orchestration).

</details>

<details markdown="1">
<summary>🎯 <b>11. Scenario: You need zero-downtime deployment with the ability to instantly roll back if the new version has issues, using ECS. How?</b></summary>
<br>

Use Blue/Green deployment via AWS CodeDeploy integration - it launches a new task set (green) alongside the running one (blue), shifts ALB traffic gradually or all at once after passing health checks/bake time, and can automatically roll back to blue if CloudWatch alarms trigger during the bake period.

</details>

<details markdown="1">
<summary>❓ <b>12. What are `minimumHealthyPercent` and `maximumPercent` in a Rolling Update deployment?</b></summary>
<br>

`minimumHealthyPercent` defines the minimum percentage of the desired task count that must remain healthy/running during a deployment (e.g., 100% means no reduction in capacity during rollout). `maximumPercent` defines the upper limit of tasks that can run simultaneously (e.g., 200% allows doubling capacity temporarily to replace tasks without ever going below desired count).

</details>

<details markdown="1">
<summary>❓ <b>13. What is ECS Service Auto Scaling and how does it work?</b></summary>
<br>

Uses Application Auto Scaling to adjust the desired task count of a service based on Target Tracking (e.g., average CPU/memory utilization, or ALB request count per target) or Step Scaling policies tied to CloudWatch alarms.

</details>

## ⚔️ Fargate vs EC2 - Deeper

<details markdown="1">
<summary>🎯 <b>14. Scenario: You're running a workload with highly variable, spiky traffic and want to minimize operational overhead. Fargate or EC2 launch type?</b></summary>
<br>

Fargate - since it eliminates the need to manage/scale the underlying EC2 fleet, automatically providing exactly the compute needed per task, ideal for variable workloads where managing EC2 capacity planning/scaling would add unnecessary operational complexity.

</details>

<details markdown="1">
<summary>🎯 <b>15. Scenario: You need GPU support for an ML inference workload on ECS. Which launch type must you use?</b></summary>
<br>

EC2 launch type - Fargate does not support GPU-accelerated instances; you must use EC2 instances with GPU support (e.g., P3/G4 family) registered to the ECS cluster.

</details>

<details markdown="1">
<summary>❓ <b>16. How do EC2-backed ECS clusters handle capacity management?</b></summary>
<br>

Via an Auto Scaling Group of container instances registered to the cluster, often paired with ECS Capacity Providers, which can automatically manage the ASG's scaling based on the cluster's actual task resource requirements (scaling the underlying instances up/down as tasks are scheduled/removed).

</details>

<details markdown="1">
<summary>❓ <b>17. What is an ECS Capacity Provider?</b></summary>
<br>

An abstraction that manages the relationship between a cluster and the infrastructure it runs on (an ASG for EC2, or Fargate/Fargate Spot), determining how tasks are placed and how the underlying capacity scales - allows mixing multiple capacity providers (e.g., Fargate + Fargate Spot) with a defined strategy/weighting for cost optimization.

</details>

<details markdown="1">
<summary>🎯 <b>18. Scenario: You want to run most of your ECS tasks on cheaper Fargate Spot capacity, but ensure a baseline number always run on regular Fargate for resilience. How?</b></summary>
<br>

Configure a Capacity Provider Strategy on the service specifying a `base` count on the standard `FARGATE` provider (guaranteed baseline) and a `weight` for `FARGATE_SPOT` for additional capacity, balancing cost savings with resilience against Spot interruptions.

</details>

## 🧭 Service Discovery & Load Balancing

<details markdown="1">
<summary>❓ <b>19. How does ECS integrate with Application Load Balancer?</b></summary>
<br>

An ECS Service registers/deregisters its tasks with an ALB Target Group automatically as tasks start/stop (using the task's IP in `awsvpc` mode or dynamic port mapping in `bridge` mode), enabling the ALB to route traffic and perform health checks against running tasks.

</details>

<details markdown="1">
<summary>❓ <b>20. What is Service Discovery (via AWS Cloud Map) in ECS?</b></summary>
<br>

Allows services to be discovered by DNS name (e.g., `orders.internal`) rather than hardcoded IPs, automatically updating DNS records as tasks scale up/down or are replaced, useful for internal service-to-service communication in a microservices architecture without needing a load balancer for every internal call.

</details>

<details markdown="1">
<summary>🎯 <b>21. Scenario: Two internal microservices need to communicate with each other, but you don't want to provision a load balancer for purely internal service-to-service traffic. How?</b></summary>
<br>

Enable Service Discovery (Cloud Map) for both services, allowing one to resolve the other via a private DNS name that's automatically kept up to date with healthy task IPs, avoiding the cost/complexity of an internal ALB for simple internal calls.

</details>

## 🧾 Logging & Monitoring

<details markdown="1">
<summary>❓ <b>22. How does ECS handle container logging?</b></summary>
<br>

Typically via the `awslogs` log driver, sending container stdout/stderr directly to CloudWatch Logs (configurable per task definition with log group/stream prefix), or other drivers like `splunk`/`fluentd` for external log aggregation.

</details>

<details markdown="1">
<summary>❓ <b>23. What is Container Insights for ECS?</b></summary>
<br>

A CloudWatch feature providing aggregated CPU/memory/network/storage metrics and logs at the cluster, service, and task level, giving deeper operational visibility than default ECS metrics alone.

</details>

<details markdown="1">
<summary>🎯 <b>24. Scenario: A task keeps stopping shortly after starting, with `CannotPullContainerError` or similar. How do you troubleshoot?</b></summary>
<br>

Check the Task Execution Role has ECR pull permissions, verify the image URI/tag exists in the registry, check network connectivity (subnets need a route to ECR - either NAT Gateway or ECR VPC endpoints if in a private subnet), and review the "Stopped reason" field in the ECS console/API for the exact error.

</details>

## 🎯 Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>25. Scenario: Design a secure, scalable, cost-optimized architecture for a containerized microservices app using ECS.</b></summary>
<br>

Use Fargate (or Fargate Spot for non-critical workloads) launch type for reduced operational overhead, `awsvpc` networking with per-service Security Groups for isolation, ALB with path/host-based routing per microservice, Service Auto Scaling based on request count/CPU, Cloud Map for internal service discovery, Secrets Manager for credentials (via Task Execution Role), and Container Insights + centralized CloudWatch Logs for observability.

</details>

<details markdown="1">
<summary>🎯 <b>26. Scenario: You need to run a scheduled batch job (e.g., nightly report generation) using ECS rather than a long-running service. How?</b></summary>
<br>

Use ECS "RunTask" via an EventBridge Scheduled Rule targeting the ECS cluster/task definition directly (no Service needed since it's a one-off/periodic task, not a continuously running desired-count-maintained workload) - EventBridge handles the scheduling and task launch parameters.

</details>

<details markdown="1">
<summary>❓ <b>27. What is the difference between ECS and EKS, and when would you choose one over the other?</b></summary>
<br>

ECS is AWS's native, simpler container orchestrator with tighter AWS integration and less operational complexity/learning curve. EKS is managed Kubernetes, offering the full Kubernetes ecosystem (Helm, Operators, broad community tooling) and portability across cloud/on-prem, at the cost of additional complexity - choose ECS for AWS-native simplicity, EKS when you need Kubernetes-specific features, multi-cloud portability, or your team already has strong Kubernetes expertise.

</details>

<details markdown="1">
<summary>🎯 <b>28. Scenario: You want to reduce Fargate costs for fault-tolerant background workers. What do you use?</b></summary>
<br>

Fargate Spot - offering discounted pricing (up to 70% off) for interruption-tolerant tasks, appropriate for workloads like batch processing, queue consumers, or CI runners that can handle a task being stopped and restarted elsewhere.

</details>

<details markdown="1">
<summary>❓ <b>29. How do you perform blue/green or canary testing of a new container image version in ECS without a full CodeDeploy setup?</b></summary>
<br>

Create a new task definition revision with the updated image, update the service to use it with a controlled rolling deployment (adjusting `minimumHealthyPercent`/`maximumPercent` for gradual replacement), monitor CloudWatch/ALB metrics during rollout, and roll back by reverting the service to the previous task definition revision if issues arise.

</details>

<details markdown="1">
<summary>🎯 <b>30. Scenario: Your ECS tasks in a private subnet need to pull images from ECR and write logs to CloudWatch, but there's no NAT Gateway (cost reasons). How?</b></summary>
<br>

Create VPC Interface Endpoints for `ecr.api`, `ecr.dkr`, `logs` (CloudWatch Logs), and a Gateway Endpoint for S3 (since ECR uses S3 for image layer storage) - allowing tasks in the private subnet to reach these AWS services privately without needing a NAT Gateway at all.

</details>

