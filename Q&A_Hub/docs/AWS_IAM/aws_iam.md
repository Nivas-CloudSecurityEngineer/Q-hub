<div align="center" markdown="1">

# 🔐 AWS IAM
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_IAM-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is IAM in AWS?</b></summary>
<br>

Identity and Access Management (IAM) is a global AWS service that lets you securely manage access to AWS resources by controlling who (authentication) can do what (authorization) on which resources.

</details>

<details markdown="1">
<summary>❓ <b>2. What are the main components of IAM?</b></summary>
<br>

Users, Groups, Roles, Policies (JSON documents defining permissions), and Identity Providers (for federation).

</details>

<details markdown="1">
<summary>❓ <b>3. Difference between an IAM User and an IAM Role.</b></summary>
<br>

A User represents a permanent identity (person/app) with long-term credentials (password/access keys). A Role is an identity with temporary credentials assumed by trusted entities (users, services, or accounts) - no long-term keys, ideal for cross-account access and EC2/Lambda permissions.

</details>

<details markdown="1">
<summary>❓ <b>4. What is the root user and why should it be secured?</b></summary>
<br>

The root user is created when you open the AWS account and has unrestricted access to everything, including billing. Best practice: enable MFA, don't use it for daily tasks, lock away its access keys (or delete them), and create IAM users/roles for actual work.

</details>

<details markdown="1">
<summary>❓ <b>5. What is the Principle of Least Privilege?</b></summary>
<br>

Granting only the minimum permissions required to perform a task, reducing the blast radius if credentials are compromised.

</details>

## 📜 Policies

<details markdown="1">
<summary>❓ <b>6. What is an IAM Policy? What are its key elements?</b></summary>
<br>

A JSON document defining permissions. Key elements: `Version`, `Statement` (array), each with `Effect` (Allow/Deny), `Action`, `Resource`, and optional `Condition`.

</details>

<details markdown="1">
<summary>❓ <b>7. Difference between Identity-based and Resource-based policies.</b></summary>
<br>

Identity-based policies are attached to users/groups/roles (define what the identity can do). Resource-based policies are attached directly to resources (e.g., S3 bucket policy, KMS key policy) and define who can access that resource, including cross-account access.

</details>

<details markdown="1">
<summary>❓ <b>8. Explain Explicit Deny vs Implicit Deny, and evaluation order.</b></summary>
<br>

By default, everything is implicitly denied. An explicit `Allow` grants access. An explicit `Deny` always overrides any `Allow`, regardless of where it comes from (identity policy, resource policy, SCP, permission boundary). Evaluation: explicit deny > explicit allow > implicit deny (default).

</details>

<details markdown="1">
<summary>❓ <b>9. What are Managed Policies vs Inline Policies?</b></summary>
<br>

Managed policies are standalone, reusable, and can be attached to multiple identities (AWS-managed or customer-managed). Inline policies are embedded directly into a single user/group/role and are deleted when that identity is deleted - used for tightly-coupled, one-off permissions.

</details>

<details markdown="1">
<summary>❓ <b>10. What is a Permission Boundary?</b></summary>
<br>

An advanced feature that sets the maximum permissions an IAM entity (user/role) can have, regardless of its attached policies. It's used to delegate role/user creation safely (e.g., allowing a team to create roles but capping what those roles can ever do).

</details>

<details markdown="1">
<summary>❓ <b>11. What are Service Control Policies (SCPs)?</b></summary>
<br>

SCPs are used in AWS Organizations to set permission guardrails across accounts/OUs. They don't grant permissions themselves; they define the maximum available permissions, restricting what even account admins/root can do in member accounts.

</details>

<details markdown="1">
<summary>🎯 <b>12. Scenario: A user has an Allow-all policy attached but still can't access an S3 bucket. Why?</b></summary>
<br>

Possible causes: an explicit Deny somewhere (SCP, permission boundary, or the S3 bucket policy denying that principal), a bucket policy that doesn't allow the account/principal, cross-account access needing bucket policy + IAM policy both, or S3 Block Public Access settings interfering, or a Deny based on a `Condition` (e.g., MFA required, source IP restriction) that isn't met.

</details>

<details markdown="1">
<summary>❓ <b>13. How do you grant cross-account access using IAM?</b></summary>
<br>

Create a role in Account B with a trust policy allowing Account A's principal (user/role) to assume it (`sts:AssumeRole`), attach a permission policy defining what that role can do, then users in Account A assume the role via STS to get temporary credentials.

</details>

<details markdown="1">
<summary>❓ <b>14. What is the difference between a Trust Policy and a Permission Policy on a role?</b></summary>
<br>

Trust policy (assume role policy) defines *who* can assume the role. Permission policy defines *what* the role can do once assumed.

</details>

## 🎭 Roles & Federation

<details markdown="1">
<summary>❓ <b>15. Why should EC2 instances use IAM Roles instead of access keys?</b></summary>
<br>

Roles provide temporary, automatically rotated credentials via the instance metadata service, eliminating the risk of hardcoded/leaked long-term access keys and simplifying credential management.

</details>

<details markdown="1">
<summary>❓ <b>16. What is STS (Security Token Service)?</b></summary>
<br>

A service that issues temporary security credentials (access key, secret key, session token) for IAM users/roles/federated users, used via `AssumeRole`, `AssumeRoleWithSAML`, `AssumeRoleWithWebIdentity`, `GetSessionToken`.

</details>

<details markdown="1">
<summary>❓ <b>17. What is IAM Identity Federation and give an example.</b></summary>
<br>

Federation lets external identities (corporate AD via SAML, or web identities via OIDC like Google/Cognito) assume IAM roles without creating individual IAM users, enabling SSO into AWS.

</details>

<details markdown="1">
<summary>❓ <b>18. What is IRSA (IAM Roles for Service Accounts) in EKS?</b></summary>
<br>

A mechanism allowing Kubernetes pods to assume specific IAM roles via OIDC federation between the EKS cluster and IAM, giving pod-level least-privilege AWS permissions instead of node-wide roles.

</details>

<details markdown="1">
<summary>🎯 <b>19. Scenario: You need a Lambda function to read from a specific S3 bucket and write to a specific DynamoDB table only. How do you configure this?</b></summary>
<br>

Create an execution role for Lambda with a permission policy scoped to `s3:GetObject`/`s3:ListBucket` on the specific bucket ARN and `dynamodb:PutItem`/etc. on the specific table ARN only - not wildcard `*` resources, following least privilege.

</details>

## 🔐 Security & MFA

<details markdown="1">
<summary>❓ <b>20. What is MFA and why is it important in IAM?</b></summary>
<br>

Multi-Factor Authentication requires a second verification factor (virtual/hardware MFA device) beyond password, significantly reducing risk from compromised credentials - mandatory best practice for root and privileged users.

</details>

<details markdown="1">
<summary>❓ <b>21. How do you enforce MFA for sensitive actions?</b></summary>
<br>

Use an IAM policy with a `Condition` checking `aws:MultiFactorAuthPresent: true`, denying actions if MFA isn't present, often combined with a short-lived STS session requiring re-authentication.

</details>

<details markdown="1">
<summary>❓ <b>22. What are IAM Access Keys and best practices around them?</b></summary>
<br>

Access keys (access key ID + secret) allow programmatic API/CLI access. Best practices: avoid using them where roles can be used instead, rotate regularly, never hardcode in code/repos, use Secrets Manager/env vars, and delete unused keys.

</details>

<details markdown="1">
<summary>❓ <b>23. How do you audit IAM permissions and detect unused/over-privileged identities?</b></summary>
<br>

Use IAM Access Analyzer, IAM Credential Report (`generate-credential-report`), Access Advisor (last accessed services/actions), and CloudTrail logs to find unused permissions and right-size policies.

</details>

<details markdown="1">
<summary>❓ <b>24. What is IAM Access Analyzer?</b></summary>
<br>

A service that analyzes resource policies (S3, IAM roles, KMS, etc.) to identify resources shared with external entities, and can also validate policies against AWS best practices and generate least-privilege policies from CloudTrail activity.

</details>

<details markdown="1">
<summary>🎯 <b>25. Scenario: You suspect an IAM user's access keys have been leaked. What immediate steps do you take?</b></summary>
<br>

Immediately deactivate/delete the compromised access key, rotate any secrets, review CloudTrail for unauthorized activity, check for newly created resources/users/roles (a common attacker move), attach a Deny-all policy temporarily if needed, and enable/verify GuardDuty alerts.

</details>

## 🏢 Groups & Organization

<details markdown="1">
<summary>❓ <b>26. What are IAM Groups and why use them?</b></summary>
<br>

Groups are collections of users sharing the same permissions. Best practice: attach policies to groups (not individual users) for easier management at scale - add/remove users from groups instead of managing per-user policies.

</details>

<details markdown="1">
<summary>❓ <b>27. Can you assign a policy directly to a resource-based permission like S3 bucket without IAM roles?</b></summary>
<br>

Yes - S3 bucket policies, KMS key policies, and SNS/SQS resource policies are resource-based and can grant access independent of IAM identity policies (useful for cross-account or public access scenarios).

</details>

<details markdown="1">
<summary>❓ <b>28. What is the difference between `Action` and `NotAction` in a policy?</b></summary>
<br>

`Action` lists explicitly the actions the statement applies to. `NotAction` is the inverse - it applies the statement to everything EXCEPT the listed actions - useful for broad deny statements excluding a few safe actions, but must be used carefully.

</details>

<details markdown="1">
<summary>❓ <b>29. What is a wildcard (`*`) risk in IAM policies?</b></summary>
<br>

Using `*` for Action or Resource grants overly broad permissions (e.g., `"Action": "*", "Resource": "*"`), violating least privilege and increasing blast radius if the identity is compromised - should be scoped down for production.

</details>

## 🎯 Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>30. Scenario: Your CI/CD pipeline (e.g., Jenkins/GitHub Actions) needs to deploy to AWS. How do you avoid storing long-term credentials?</b></summary>
<br>

Use OIDC federation - configure an IAM role with a trust policy trusting the CI/CD provider's OIDC identity provider (e.g., GitHub Actions OIDC), so the pipeline assumes a role and gets short-lived STS credentials per run, with no static keys stored as secrets.

</details>

<details markdown="1">
<summary>🎯 <b>31. Scenario: Multiple teams need isolated AWS access within the same account. How do you design IAM?</b></summary>
<br>

Use IAM groups/roles scoped by team/project with resource tagging + `Condition` (`aws:ResourceTag`) to restrict access to only resources tagged for that team, combined with permission boundaries to prevent privilege escalation.

</details>

<details markdown="1">
<summary>🎯 <b>32. Scenario: You need to grant a third-party vendor temporary read-only access to specific resources. How?</b></summary>
<br>

Create a cross-account IAM role with least-privilege read-only permissions and a trust policy scoped to the vendor's AWS account ID (and optionally an `ExternalId` condition to prevent confused deputy issues), avoiding creation of IAM users for the vendor.

</details>

<details markdown="1">
<summary>❓ <b>33. What is the "Confused Deputy" problem and how does IAM mitigate it?</b></summary>
<br>

It occurs when a third party is tricked into misusing its permissions on behalf of an attacker (common in cross-account role assumption). AWS mitigates this with the `sts:ExternalId` condition in trust policies, ensuring only requests with the correct external ID can assume the role.

</details>

<details markdown="1">
<summary>❓ <b>34. How do you troubleshoot an "Access Denied" error in AWS effectively?</b></summary>
<br>

Use the IAM Policy Simulator, review CloudTrail's `errorMessage`/`errorCode` for the exact denied action, check for explicit Denies in identity policies, resource policies, SCPs, and permission boundaries, and verify conditions (MFA, IP, tags) are satisfied.

</details>

<details markdown="1">
<summary>❓ <b>35. What is the difference between `sts:AssumeRole` and `sts:AssumeRoleWithWebIdentity`?</b></summary>
<br>

`AssumeRole` is used by IAM users/roles (including cross-account) with valid AWS credentials. `AssumeRoleWithWebIdentity` is used for federated identities via OIDC providers (e.g., Cognito, Google, GitHub Actions) without needing AWS IAM credentials upfront.

</details>

<details markdown="1">
<summary>🎯 <b>36. Scenario: How would you design least-privilege IAM for a Terraform CI pipeline that manages infra across dev/stage/prod?</b></summary>
<br>

Use separate roles per environment with scoped permissions (e.g., prod role requires manual approval/step in pipeline), store Terraform state with restricted access, use OIDC-based role assumption per environment, and apply SCPs/permission boundaries to prevent the pipeline role from escalating its own privileges or modifying IAM outside its scope.

</details>

<details markdown="1">
<summary>❓ <b>37. What's the risk of granting `iam:PassRole` broadly, and how do you scope it safely?</b></summary>
<br>

`iam:PassRole` allows a user/service to pass an IAM role to an AWS service (e.g., attach a role to an EC2 instance or Lambda). Granting it broadly could let a user attach a highly privileged role to a resource they control, escalating privilege. Scope it with a `Resource` ARN restricting exactly which role(s) can be passed, and combine with `iam:PassedToService` condition.

</details>

<details markdown="1">
<summary>❓ <b>38. What is an IAM policy `Condition` and give practical examples.</b></summary>
<br>

Conditions add fine-grained control based on context keys, e.g., `aws:SourceIp` (restrict by IP), `aws:MultiFactorAuthPresent` (require MFA), `aws:RequestedRegion` (restrict region), `s3:x-amz-server-side-encryption` (enforce encryption on upload), `aws:PrincipalOrgID` (restrict to org members).

</details>

<details markdown="1">
<summary>❓ <b>39. How does IAM integrate with AWS Organizations for centralized management?</b></summary>
<br>

AWS Organizations lets you manage multiple accounts centrally, apply SCPs to OUs/accounts as guardrails, use consolidated billing, and enable services like IAM Identity Center (SSO) for centralized federated access across all accounts.

</details>

<details markdown="1">
<summary>❓ <b>40. What is AWS IAM Identity Center (formerly AWS SSO) and how does it differ from IAM?</b></summary>
<br>

IAM Identity Center provides centralized SSO access across multiple AWS accounts and business applications, integrating with external identity providers (Azure AD, Okta), whereas classic IAM manages users/roles/policies within a single account. Identity Center is the recommended approach for human user access at organization scale, while IAM roles/policies remain the mechanism for workload/service permissions.

</details>

