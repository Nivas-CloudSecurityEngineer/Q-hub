<div align="center" markdown="1">

# 🤫 AWS Secrets Manager
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_Secrets_Manager-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is AWS Secrets Manager?</b></summary>
<br>

A managed service for securely storing, retrieving, and automatically rotating secrets such as database credentials, API keys, and other sensitive configuration, eliminating hardcoded secrets in application code.

</details>

<details markdown="1">
<summary>❓ <b>2. Difference between Secrets Manager and Systems Manager Parameter Store.</b></summary>
<br>

Parameter Store (SecureString) is a simpler, cheaper (free tier for standard parameters) key-value store supporting KMS encryption but without built-in automatic rotation. Secrets Manager costs more per secret but provides native automatic rotation (with built-in Lambda rotation templates for RDS/Redshift/DocumentDB), fine-grained resource policies, and cross-account sharing - Secrets Manager is preferred when rotation and secret lifecycle management matter most.

</details>

<details markdown="1">
<summary>❓ <b>3. How is data encrypted in Secrets Manager?</b></summary>
<br>

Every secret is encrypted at rest using AWS KMS (a default AWS managed key `aws/secretsmanager` or a customer-managed CMK you specify), and encrypted in transit via TLS during retrieval.

</details>

<details markdown="1">
<summary>❓ <b>4. What formats can a secret be stored in?</b></summary>
<br>

Secrets can be stored as key-value pairs (JSON, e.g., `{"username":"admin","password":"..."}`), or as plaintext (e.g., a single API key/token string).

</details>

## 🔁 Rotation

<details markdown="1">
<summary>❓ <b>5. How does automatic secret rotation work?</b></summary>
<br>

Secrets Manager invokes a Lambda rotation function on a defined schedule, which follows a 4-step process (createSecret, setSecret, testSecret, finishSecret) to generate a new credential, update it on the target system (e.g., the database), test it works, and mark it as the current version - all without application downtime.

</details>

<details markdown="1">
<summary>❓ <b>6. Explain the versioning stages used during rotation (AWSCURRENT, AWSPENDING, AWSPREVIOUS).</b></summary>
<br>

AWSCURRENT is the active version applications should use. AWSPENDING is the new version being created/tested during rotation (not yet promoted). AWSPREVIOUS retains the prior version temporarily after rotation completes, allowing rollback/graceful transition if some clients haven't picked up the new version yet.

</details>

<details markdown="1">
<summary>🎯 <b>7. Scenario: Rotation for an RDS database credential fails midway. What could go wrong and how do you troubleshoot?</b></summary>
<br>

Common causes: the rotation Lambda's execution role lacks permission to call Secrets Manager/RDS APIs, network issues (Lambda not in the same VPC/no route to the DB), the DB user lacks ALTER USER privileges needed to change its own password, or the Lambda function's VPC security group doesn't allow it to reach the DB - check CloudWatch Logs for the rotation Lambda to pinpoint the exact failure step.

</details>

<details markdown="1">
<summary>❓ <b>8. What is the difference between single-user and alternating-user rotation strategies?</b></summary>
<br>

Single-user rotation updates the password of the same database user in place (a brief window exists where old connections may fail until they pick up the new secret). Alternating-user rotation maintains two users (e.g., appuser1/appuser2), rotating between them each cycle - the previous user's credentials remain valid during the transition, providing true zero-downtime rotation with no connection interruption risk.

</details>

<details markdown="1">
<summary>🎯 <b>9. Scenario: You need to rotate a secret used by a third-party API (not a native AWS database). How?</b></summary>
<br>

Write a custom rotation Lambda function implementing the 4-step rotation lifecycle (create/set/test/finish) tailored to that API's credential rotation mechanism (e.g., calling the vendor's "rotate API key" endpoint), since built-in rotation templates only cover RDS/Redshift/DocumentDB natively.

</details>

## 🔐 Access Control & Retrieval

<details markdown="1">
<summary>❓ <b>10. How do applications retrieve secrets securely at runtime?</b></summary>
<br>

Via the Secrets Manager SDK/API (`GetSecretValue`) using the application's IAM role permissions (least privilege scoped to specific secret ARNs), or via the Secrets Manager Lambda extension/caching client for reduced latency and API call cost.

</details>

<details markdown="1">
<summary>🎯 <b>11. Scenario: You want to avoid calling Secrets Manager on every single request due to latency/cost, but still want credentials to update automatically after rotation. How?</b></summary>
<br>

Use the AWS Secrets Manager client-side caching library/Lambda extension, which caches the secret value in memory and refreshes it periodically (or on version change), balancing performance with automatic pickup of rotated credentials.

</details>

<details markdown="1">
<summary>❓ <b>12. What is a Resource Policy on a secret and when is it used?</b></summary>
<br>

A resource-based policy attached directly to a secret controlling which principals (including cross-account) can access it - used similarly to an S3 bucket policy, enabling secure sharing of a secret across AWS accounts without duplicating it.

</details>

<details markdown="1">
<summary>🎯 <b>13. Scenario: A Lambda function needs a database password, but you don't want it stored in an environment variable in plaintext. How do you handle this?</b></summary>
<br>

Store the password in Secrets Manager, grant the Lambda's execution role `secretsmanager:GetSecretValue` scoped to that secret's ARN only, and retrieve/cache it in the function's init code (outside the handler) at runtime - avoiding plaintext exposure in the Lambda console/environment variables.

</details>

## 🔏 Security & Compliance

<details markdown="1">
<summary>❓ <b>14. How do you audit who accessed a secret and when?</b></summary>
<br>

Every `GetSecretValue`, `PutSecretValue`, `UpdateSecret`, etc. call is logged in CloudTrail (as a management event, and can also enable data events for more granular tracking), showing the requesting principal, timestamp, and source IP.

</details>

<details markdown="1">
<summary>🎯 <b>15. Scenario: You suspect a secret has been compromised (leaked in logs or a public repo). What immediate steps do you take?</b></summary>
<br>

Immediately rotate the secret (force a manual rotation or update the value), review CloudTrail for any unauthorized `GetSecretValue` calls to assess the blast radius, invalidate/change the underlying credential at the source system if rotation alone isn't sufficient, and search/remove the leaked value from any exposed location (logs, repos, CI artifacts).

</details>

<details markdown="1">
<summary>❓ <b>16. What is the "Replicate Secret" feature and when would you use it?</b></summary>
<br>

Allows automatically replicating a secret to other AWS regions (read-only replicas, kept in sync with the primary), used for multi-region disaster recovery/active-active applications that need low-latency local secret access in each region.

</details>

<details markdown="1">
<summary>🎯 <b>17. Scenario: Multiple microservices need access to shared database credentials, but you want to minimize blast radius if one service is compromised. How do you design this?</b></summary>
<br>

Use separate secrets per service/database-user combination (least privilege, following the principle that each service should have its own scoped DB credentials rather than one shared secret), with IAM policies scoping each service's role to only its specific secret ARN.

</details>

## 🎯 Cost & Real-Time Scenarios

<details markdown="1">
<summary>❓ <b>18. How does Secrets Manager pricing work and how do you optimize cost?</b></summary>
<br>

Charged per secret per month plus API call charges; optimize by consolidating related config into fewer secrets where appropriate (JSON key-value within one secret), using client-side caching to reduce API call volume, and using Parameter Store for non-sensitive or non-rotating configuration instead of Secrets Manager.

</details>

<details markdown="1">
<summary>🎯 <b>19. Scenario: You're migrating an application from hardcoded credentials in config files to Secrets Manager. What's your migration approach?</b></summary>
<br>

Create secrets in Secrets Manager matching current credentials, update application code/IaC to fetch secrets via the SDK at startup (or use ECS/Lambda native integrations that inject secrets as environment variables from Secrets Manager at launch), test in a staging environment, remove hardcoded values from code/config repos, and finally enable automatic rotation once migration is verified stable.

</details>

<details markdown="1">
<summary>❓ <b>20. How do ECS Tasks and Lambda natively integrate with Secrets Manager without custom code?</b></summary>
<br>

ECS Task Definitions support a `secrets` field that injects a secret's value directly as a container environment variable at task launch (fetched securely by the ECS agent using the task execution role) - similarly, some AWS services allow referencing a Secrets Manager ARN directly in configuration, removing the need for the application to call the SDK explicitly for basic use cases.

</details>

<details markdown="1">
<summary>🎯 <b>21. Scenario: You need to grant a CI/CD pipeline temporary access to a deployment secret without storing long-lived credentials. How?</b></summary>
<br>

Use an OIDC-federated IAM role assumed by the CI/CD pipeline (e.g., GitHub Actions), scoped via IAM/resource policy to read only the specific secret ARN needed, ensuring credentials are short-lived (STS) and access is fully auditable via CloudTrail, rather than storing a long-term secret manager API key in the pipeline itself.

</details>

