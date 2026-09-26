<div align="center" markdown="1">

# 📦 AWS ECR (Elastic Container Registry)
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_ECR-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is Amazon ECR?</b></summary>
<br>

A fully managed Docker/OCI container registry service that lets you store, manage, and deploy container images securely, integrated natively with IAM, ECS, EKS, and Lambda.

</details>

<details markdown="1">
<summary>❓ <b>2. Difference between ECR Private and ECR Public.</b></summary>
<br>

ECR Private repositories require authentication and IAM permissions to push/pull, used for proprietary application images. ECR Public Gallery hosts publicly accessible images (like Docker Hub) that anyone can pull without authentication, used for sharing open-source images.

</details>

<details markdown="1">
<summary>❓ <b>3. How do you authenticate Docker to push/pull images to/from ECR?</b></summary>
<br>

Use `aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com`, which retrieves a temporary authentication token (valid for 12 hours) using your IAM credentials.

</details>

<details markdown="1">
<summary>❓ <b>4. What is the format of an ECR image URI?</b></summary>
<br>

`<account-id>.dkr.ecr.<region>.amazonaws.com/<repository-name>:<tag>` - identifying the account, region, repository, and specific tag/digest.

</details>

## 📦 Repository Management

<details markdown="1">
<summary>❓ <b>5. What is Image Tag Immutability and why enable it?</b></summary>
<br>

A repository setting that prevents a tag from being overwritten once pushed (any push with an existing tag is rejected) - important for ensuring that a given tag (e.g., `v1.2.3`) always refers to exactly the same image content, critical for auditability and reliable rollbacks in production.

</details>

<details markdown="1">
<summary>❓ <b>6. What is an ECR Lifecycle Policy?</b></summary>
<br>

A rule-based policy that automatically expires/deletes old images based on criteria (e.g., "keep only the last 10 tagged images" or "expire untagged images older than 14 days"), managing storage costs and repository clutter without manual cleanup.

</details>

<details markdown="1">
<summary>🎯 <b>7. Scenario: Your ECR repository is accumulating thousands of untagged images from CI builds, driving up storage costs. How do you fix this?</b></summary>
<br>

Apply a Lifecycle Policy rule targeting untagged images with an expiration after a short period (e.g., 7-14 days), since untagged images are typically build artifacts/intermediate layers no longer referenced by any deployment.

</details>

<details markdown="1">
<summary>❓ <b>8. How does ECR support vulnerability scanning?</b></summary>
<br>

Two options: Basic Scanning (uses the open-source Clair engine, scans on push for known CVEs), and Enhanced Scanning (integrates with Amazon Inspector for continuous, automated rescanning as new CVEs are published, covering OS packages and language-level dependencies).

</details>

<details markdown="1">
<summary>🎯 <b>9. Scenario: You want to prevent vulnerable images from ever being deployed to production. How do you enforce this in your pipeline?</b></summary>
<br>

Enable ECR scan-on-push, have the CI/CD pipeline query the scan findings (via API) after pushing and fail/block the deployment stage if critical/high severity vulnerabilities are found above a defined threshold, before promoting the image to a production tag/environment.

</details>

## 🔐 Security & Access Control

<details markdown="1">
<summary>❓ <b>10. How do you control who can push/pull to a specific ECR repository?</b></summary>
<br>

Use IAM policies (identity-based, scoping `ecr:PutImage`/`ecr:BatchGetImage` etc. to specific repository ARNs) and/or Repository Policies (resource-based, similar to S3 bucket policies) for cross-account access scenarios.

</details>

<details markdown="1">
<summary>🎯 <b>11. Scenario: A separate AWS account needs to pull images from your ECR repository (e.g., for a shared services/deployment account setup). How?</b></summary>
<br>

Add a Repository Policy granting the external account's principal `ecr:GetDownloadUrlForLayer`, `ecr:BatchGetImage`, and `ecr:BatchCheckLayerAvailability` permissions, and that account's IAM policy must also allow calling ECR - both are required for cross-account pull access.

</details>

<details markdown="1">
<summary>❓ <b>12. Are images in ECR encrypted?</b></summary>
<br>

Yes - by default with SSE-S3 (since ECR uses S3 as underlying storage), or optionally with SSE-KMS using a customer-managed key for additional control over encryption keys and audit logging via CloudTrail.

</details>

<details markdown="1">
<summary>❓ <b>13. How does ECR integrate with ECS/EKS/Lambda for pulling images securely?</b></summary>
<br>

The compute service's execution/task role (or node IAM role for EKS) must have ECR pull permissions (`ecr:GetAuthorizationToken`, `ecr:BatchGetImage`, `ecr:GetDownloadUrlForLayer`); no long-lived Docker credentials are needed since these services use the underlying IAM role to authenticate automatically.

</details>

## 🔁 Replication & Availability

<details markdown="1">
<summary>❓ <b>14. What is ECR Cross-Region/Cross-Account Replication?</b></summary>
<br>

A feature to automatically replicate images to another region or account as soon as they're pushed, useful for multi-region deployments (reducing pull latency and improving resilience) or centralizing images from a build account to be consumed by multiple downstream accounts.

</details>

<details markdown="1">
<summary>🎯 <b>15. Scenario: You deploy the same application across us-east-1 and eu-west-1 and want to avoid cross-region pull latency. How?</b></summary>
<br>

Configure ECR Replication Rules to automatically copy images to the eu-west-1 registry as soon as they're pushed to us-east-1, so ECS/EKS in each region pulls from its local regional replica instead of incurring cross-region data transfer latency/cost on every deployment.

</details>

## 🎯 CI/CD & Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>16. Scenario: Design a CI/CD pipeline that builds a Docker image, scans it, and pushes it to ECR only if it passes security checks.</b></summary>
<br>

CI pipeline builds the image, pushes to ECR (or a staging repo/tag), triggers/waits for ECR scan results via API polling or an EventBridge rule on `ECR Image Scan` completion event, and only proceeds to tag it as deployable (or push to a "promoted" repository) if no critical/high vulnerabilities are found - failing the pipeline otherwise.

</details>

<details markdown="1">
<summary>❓ <b>17. How would you implement immutable, traceable deployments using ECR image tags/digests?</b></summary>
<br>

Tag images with a unique identifier tied to the build (e.g., git commit SHA) rather than mutable tags like `latest`, enable tag immutability on the repository, and reference the image by digest (`sha256:...`) in deployment manifests for absolute certainty of exactly which image content is running, fully traceable back to source control.

</details>

<details markdown="1">
<summary>🎯 <b>18. Scenario: You need to automatically clean up old images but retain the last 5 production-tagged releases for rollback capability, while aggressively cleaning up dev/test images. How?</b></summary>
<br>

Use ECR Lifecycle Policy rules with tag prefix matching - one rule for `prod-*` tags keeping the last 5 images, and a separate, more aggressive rule for `dev-*`/untagged images expiring after a short period (e.g., a few days), applied to the same repository with different priorities/selection criteria.

</details>

<details markdown="1">
<summary>❓ <b>19. What is the difference between an ECR "public" pull-through cache and hosting your own mirror of Docker Hub images?</b></summary>
<br>

ECR Pull-Through Cache lets you configure ECR to automatically cache and serve images from upstream registries (like Docker Hub or public ECR) the first time they're requested, avoiding Docker Hub rate limits and providing local, faster, IAM-controlled access - without needing to manually build/maintain your own mirroring pipeline.

</details>

<details markdown="1">
<summary>🎯 <b>20. Scenario: Docker Hub rate limiting is causing CI/CD pipeline failures when pulling base images (e.g., `node:18-alpine`). How does ECR help solve this?</b></summary>
<br>

Set up an ECR Pull-Through Cache rule for Docker Hub, and update Dockerfiles/pipeline references to pull the base image through the ECR cache endpoint instead of directly from Docker Hub - ECR caches the image after the first pull, insulating your pipeline from Docker Hub's anonymous/authenticated rate limits.

</details>

