<div align="center" markdown="1">

# λ AWS Lambda
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_Lambda-blue?style=for-the-badge&logo=awslambda&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is AWS Lambda?</b></summary>
<br>

A serverless compute service that runs your code in response to events without provisioning or managing servers, automatically scaling and charging only for actual compute time consumed (per millisecond).

</details>

<details markdown="1">
<summary>❓ <b>2. What are the key components of a Lambda function?</b></summary>
<br>

Handler (entry point function), Runtime (language environment, e.g., Node.js/Python/Java/Go/custom runtime), Execution Role (IAM permissions), Trigger/Event Source (what invokes it), Layers (optional shared code/dependencies), and configuration (memory, timeout, environment variables).

</details>

<details markdown="1">
<summary>❓ <b>3. What triggers can invoke a Lambda function?</b></summary>
<br>

API Gateway, S3 events, DynamoDB Streams, SQS/SNS, EventBridge (scheduled/event-pattern), Kinesis Streams, Application Load Balancer, CloudFront (Lambda@Edge), Step Functions, and direct SDK/CLI invocation.

</details>

<details markdown="1">
<summary>❓ <b>4. What is the maximum execution timeout for a Lambda function?</b></summary>
<br>

15 minutes (900 seconds) - for longer-running tasks, you need Step Functions orchestration, ECS/Fargate, or breaking the work into smaller chunks.

</details>

<details markdown="1">
<summary>❓ <b>5. How does Lambda pricing work?</b></summary>
<br>

Charged based on the number of requests and duration (GB-seconds = memory allocated x execution time), with a generous perpetual free tier (1M requests and 400,000 GB-seconds/month).

</details>

## ⚡ Execution Model & Performance

<details markdown="1">
<summary>❓ <b>6. What is a Cold Start and why does it happen?</b></summary>
<br>

The latency incurred when Lambda needs to initialize a new execution environment (download code, start runtime, run init code) before handling the first invocation - happens when no warm instance is available (first invocation, scaling up, or after inactivity).

</details>

<details markdown="1">
<summary>❓ <b>7. How do you reduce cold start latency?</b></summary>
<br>

Use Provisioned Concurrency (pre-initialized environments always ready), reduce package size/dependencies, choose lighter runtimes (e.g., avoid heavy frameworks/Java class loading where latency-sensitive), keep initialization code outside the handler (reused across warm invocations), and use SnapStart (for Java) which caches a post-initialization snapshot.

</details>

<details markdown="1">
<summary>❓ <b>8. What is Provisioned Concurrency and how does it differ from Reserved Concurrency?</b></summary>
<br>

Provisioned Concurrency keeps a specified number of execution environments pre-warmed and ready to respond instantly (eliminating cold starts for that capacity, at an additional cost). Reserved Concurrency sets both a guaranteed and maximum limit on concurrent executions for a function (ensures capacity availability and/or limits blast radius/downstream load), without pre-warming.

</details>

<details markdown="1">
<summary>🎯 <b>9. Scenario: Your Lambda-based API has unpredictable latency spikes for the first request after idle periods. How do you fix this for a latency-sensitive production API?</b></summary>
<br>

Enable Provisioned Concurrency sized to expected baseline traffic (possibly combined with Application Auto Scaling to adjust provisioned concurrency on a schedule/target tracking basis), and optimize package size/init code to further reduce any residual cold start impact.

</details>

<details markdown="1">
<summary>❓ <b>10. What is the execution environment lifecycle (Init, Invoke, Shutdown)?</b></summary>
<br>

Init: environment created, runtime bootstrapped, code outside the handler runs (connections, SDK clients). Invoke: the handler function runs per event (can be reused across multiple invocations in the same warm environment). Shutdown: environment is eventually torn down after inactivity, running any cleanup logic if using extensions.

</details>

<details markdown="1">
<summary>❓ <b>11. Why should you initialize SDK clients/DB connections outside the handler function?</b></summary>
<br>

Code outside the handler runs only once per cold start (during Init), then is reused across subsequent warm invocations - initializing expensive resources (DB connections, SDK clients) here avoids re-creating them on every single invocation, significantly improving performance.

</details>

## 🔀 Concurrency & Scaling

<details markdown="1">
<summary>❓ <b>12. How does Lambda scale with incoming traffic?</b></summary>
<br>

Lambda automatically creates new execution environments in parallel to handle concurrent invocations, scaling up rapidly (with an initial burst limit, then a steady increase rate) up to the account/function concurrency limit.

</details>

<details markdown="1">
<summary>❓ <b>13. What is a Concurrency Limit and what happens when it's exceeded?</b></summary>
<br>

The maximum number of simultaneous executions allowed (account-level default limit, or a function-level Reserved Concurrency cap); when exceeded, additional invocation requests are throttled (synchronous callers get a `429 TooManyRequestsException`, asynchronous invocations are retried automatically then sent to a DLQ if configured).

</details>

<details markdown="1">
<summary>🎯 <b>14. Scenario: A single misbehaving Lambda function is consuming the entire account's concurrency limit, starving other functions. How do you prevent this?</b></summary>
<br>

Set a Reserved Concurrency limit on that specific function to cap its maximum concurrent executions, ensuring it can't exhaust the shared account-level pool needed by other functions.

</details>

<details markdown="1">
<summary>❓ <b>15. How does Lambda concurrency interact with a downstream resource like RDS that has limited connections?</b></summary>
<br>

A traffic spike can cause Lambda to scale out massively and open far more DB connections than the database can handle, exhausting its connection pool. Mitigate using RDS Proxy (connection pooling/multiplexing), setting Reserved Concurrency as a ceiling, or using SQS as a buffer to control the processing rate.

</details>

## 📡 Event Sources - Sync vs Async vs Stream-based

<details markdown="1">
<summary>❓ <b>16. Difference between synchronous and asynchronous Lambda invocation.</b></summary>
<br>

Synchronous (e.g., API Gateway, ALB): the caller waits for the response, errors are returned directly to the caller. Asynchronous (e.g., S3, SNS, EventBridge): Lambda queues the event internally, returns immediately to the caller, and retries on failure (up to 2 additional retries) - failed events after retries can go to a Dead Letter Queue (DLQ) or On-Failure Destination.

</details>

<details markdown="1">
<summary>❓ <b>17. How does Lambda process events from a stream (Kinesis/DynamoDB Streams/SQS)?</b></summary>
<br>

Lambda polls the stream/queue internally (poll-based model) and invokes your function with a batch of records; for streams, records are processed in order per shard, and a failure can block processing of subsequent records in that shard unless configured with bisect-on-error/retry limits.

</details>

<details markdown="1">
<summary>🎯 <b>18. Scenario: One malformed record in a Kinesis stream is causing your Lambda consumer to retry indefinitely and block all subsequent records in that shard. How do you fix this?</b></summary>
<br>

Configure `Maximum Retry Attempts` and `Bisect Batch on Function Error` on the event source mapping, and set up a Destination (on-failure) or DLQ for the shard to send the problematic batch aside, allowing the shard to continue processing subsequent records instead of stalling.

</details>

<details markdown="1">
<summary>❓ <b>19. What is a Dead Letter Queue (DLQ) vs a Destination in Lambda?</b></summary>
<br>

DLQ (legacy, SNS/SQS) captures the failed event payload after async invocation exhausts retries. Destinations (newer, more flexible) can route both success AND failure outcomes to SQS, SNS, EventBridge, or another Lambda function, with richer context (including response payload, not just the request).

</details>

## 🌐 Networking & VPC

<details markdown="1">
<summary>❓ <b>20. Does a Lambda function need to be in a VPC?</b></summary>
<br>

Only if it needs to access resources inside a VPC (RDS, ElastiCache, private services) - functions accessing only public AWS APIs (S3, DynamoDB via public endpoints) don't need VPC attachment and perform better/simpler without it.

</details>

<details markdown="1">
<summary>❓ <b>21. What are the implications of putting a Lambda function inside a VPC?</b></summary>
<br>

Lambda creates/uses ENIs in your specified subnets, requiring a NAT Gateway (or VPC endpoints) for internet/AWS-service access from a private subnet, and historically added cold-start latency for ENI creation (largely mitigated since 2019 with Hyperplane-based networking, but still adds slight overhead and IP address consumption considerations).

</details>

<details markdown="1">
<summary>🎯 <b>22. Scenario: A VPC-attached Lambda function needs to call the S3 API but times out. Why, and how do you fix it?</b></summary>
<br>

If the Lambda is in a private subnet without a route to the internet/S3, it can't reach the S3 public endpoint. Fix by adding a NAT Gateway (for general internet access) or, more efficiently, a Gateway VPC Endpoint for S3 (private, no NAT cost/latency).

</details>

## 🔏 Security & Permissions

<details markdown="1">
<summary>❓ <b>23. How does a Lambda function get permissions to interact with other AWS services?</b></summary>
<br>

Via its Execution Role - an IAM role attached to the function granting it permissions (e.g., `s3:GetObject`, `dynamodb:PutItem`) following least privilege, assumed automatically by the Lambda service at invocation.

</details>

<details markdown="1">
<summary>❓ <b>24. What is Resource-based Policy on a Lambda function, and how does it differ from the Execution Role?</b></summary>
<br>

The Execution Role defines what the Lambda function itself can DO (call other services). The Resource-based Policy defines who/what can INVOKE the Lambda function (e.g., allowing API Gateway or S3 to invoke it, or granting cross-account invoke permissions).

</details>

<details markdown="1">
<summary>🎯 <b>25. Scenario: How do you securely manage database credentials or API keys used by a Lambda function?</b></summary>
<br>

Store them in AWS Secrets Manager (or Parameter Store with SecureString), retrieve them at runtime using the execution role's permissions (with in-memory caching across warm invocations to avoid excessive API calls/cost), rather than hardcoding or storing them as plain environment variables.

</details>

<details markdown="1">
<summary>❓ <b>26. Are Lambda environment variables encrypted?</b></summary>
<br>

Yes, by default they're encrypted at rest using an AWS-managed KMS key; you can also use a customer-managed KMS key for additional control (e.g., separate encryption/decryption permissions, key rotation policies, audit trail via CloudTrail).

</details>

## 🎯 Real-Time Scenarios & Best Practices

<details markdown="1">
<summary>🎯 <b>27. Scenario: You need to process image uploads (resize/thumbnail) as soon as they land in S3. Design the architecture.</b></summary>
<br>

S3 `ObjectCreated` event triggers a Lambda function directly (for simple cases) or via SQS (for buffering/retry control under high volume), the Lambda resizes the image (using a layer with image processing libraries) and writes the result to an output bucket/prefix, with a DLQ/Destination configured for failures.

</details>

<details markdown="1">
<summary>❓ <b>28. What is Lambda Layers and why use them?</b></summary>
<br>

A mechanism to package and share common code/libraries/dependencies across multiple Lambda functions without bundling them into each function's deployment package, reducing duplication and package size, and enabling centralized dependency updates.

</details>

<details markdown="1">
<summary>❓ <b>29. How do you monitor and debug Lambda functions in production?</b></summary>
<br>

CloudWatch Logs (automatic per-invocation logging), CloudWatch Metrics (`Invocations`, `Errors`, `Duration`, `Throttles`, `ConcurrentExecutions`, `IteratorAge`), and AWS X-Ray for distributed tracing to pinpoint latency/errors across the function and downstream calls.

</details>

<details markdown="1">
<summary>🎯 <b>30. Scenario: Lambda function costs have unexpectedly spiked. How do you investigate and optimize?</b></summary>
<br>

Check CloudWatch for invocation count/duration spikes (possible retry storms, misconfigured triggers causing loops, or a traffic increase), right-size memory allocation (memory also scales CPU - sometimes increasing memory reduces duration enough to lower overall cost), and check for unnecessarily long timeouts masking hanging dependencies that inflate billed duration.

</details>

<details markdown="1">
<summary>❓ <b>31. How would you implement a long-running workflow (e.g., order processing with multiple steps and waiting periods) using Lambda?</b></summary>
<br>

Use AWS Step Functions to orchestrate multiple Lambda functions as states in a state machine, handling retries/error handling/parallel branches/wait states declaratively, since a single Lambda can't exceed 15 minutes and shouldn't hold long-lived state itself.

</details>

<details markdown="1">
<summary>❓ <b>32. What is Lambda SnapStart and what problem does it solve?</b></summary>
<br>

A feature (initially for Java) that takes a snapshot of the initialized execution environment (post-Init) and restores from that snapshot on subsequent cold starts instead of re-running the full initialization, dramatically reducing cold start latency for runtimes with heavy startup costs (like JVM class loading).

</details>

<details markdown="1">
<summary>🎯 <b>33. Scenario: Multiple Lambda functions need to share a common set of utility code and a specific version of a dependency, updated centrally. How?</b></summary>
<br>

Package the shared code/dependency as a Lambda Layer, publish versions of the layer, and reference the specific layer version ARN in each function's configuration - updating the layer (and bumping function references) centralizes dependency management.

</details>

<details markdown="1">
<summary>❓ <b>34. How do you implement idempotency in a Lambda function, and why is it important?</b></summary>
<br>

Important because retries (from async invocation, SQS at-least-once delivery, or client retries) can cause the same event to be processed more than once. Implement by tracking processed event/message IDs (e.g., in DynamoDB with a conditional write) before performing the side-effecting operation, ensuring repeated deliveries don't cause duplicate actions (e.g., double-charging a customer).

</details>

<details markdown="1">
<summary>🎯 <b>35. Scenario: You need Lambda functions in different environments (dev/stage/prod) with different configurations but the same code. How do you manage this?</b></summary>
<br>

Use environment variables for environment-specific configuration (with values injected via IaC/CI-CD per environment), use Lambda Aliases and Versions (e.g., `prod` alias pointing to a specific published version) to control which code version serves which environment/traffic split, and manage deployments via SAM/CDK/Terraform pipelines per environment.

</details>

