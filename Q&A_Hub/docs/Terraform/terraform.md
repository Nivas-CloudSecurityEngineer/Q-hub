<div align="center" markdown="1">

# 🏗️ Terraform
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-Terraform-blue?style=for-the-badge&logo=terraform&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is Terraform and what problem does it solve?</b></summary>
<br>

An open-source Infrastructure as Code (IaC) tool by HashiCorp that lets you define, provision, and manage cloud/on-prem infrastructure using declarative configuration files, ensuring consistent, repeatable, and version-controlled infrastructure changes instead of manual/click-ops provisioning.

</details>

<details markdown="1">
<summary>❓ <b>2. What is the difference between declarative and imperative IaC, and which is Terraform?</b></summary>
<br>

Declarative (Terraform): you describe the DESIRED end state, and the tool figures out how to achieve it. Imperative (e.g., a bash script or older Chef/Puppet-style tools): you specify the exact STEPS to execute in order to reach a state. Terraform's declarative model makes it easier to reason about the final state and enables reliable diffing/planning.

</details>

<details markdown="1">
<summary>❓ <b>3. What is the Terraform workflow (core commands)?</b></summary>
<br>

`terraform init` (initialize working directory, download providers/modules), `terraform plan` (preview changes), `terraform apply` (execute changes to reach desired state), and `terraform destroy` (tear down managed infrastructure).

</details>

<details markdown="1">
<summary>❓ <b>4. What is a Provider in Terraform?</b></summary>
<br>

A plugin that allows Terraform to interact with a specific API/platform (AWS, Azure, GCP, Kubernetes, GitHub, etc.), translating Terraform configuration into actual API calls to create/read/update/delete resources.

</details>

## 🗄️ State Management

<details markdown="1">
<summary>❓ <b>5. What is Terraform State and why is it critical?</b></summary>
<br>

A JSON file (`terraform.tfstate`) that maps your configuration to real-world resources, tracking metadata and resource attributes - Terraform uses it to determine what changes are needed during `plan`/`apply` by comparing desired configuration against the last-known state.

</details>

<details markdown="1">
<summary>❓ <b>6. Why should Terraform State never be stored locally for team environments?</b></summary>
<br>

Local state creates single points of failure/conflict - if two team members run Terraform simultaneously without shared state, they can create conflicting/duplicate infrastructure or corrupt state; local state also risks being lost (laptop failure) or containing sensitive data unprotected in version control.

</details>

<details markdown="1">
<summary>❓ <b>7. What is Remote State and what backends are commonly used?</b></summary>
<br>

Storing state in a shared, durable location (e.g., S3 with DynamoDB for locking, Terraform Cloud/Enterprise, Azure Blob Storage, GCS) accessible to the whole team/CI pipeline, ensuring consistency and enabling collaboration.

</details>

<details markdown="1">
<summary>🎯 <b>8. Scenario: Two engineers run `terraform apply` at the same time using a shared S3 backend. What prevents them from corrupting the state?</b></summary>
<br>

State Locking - when using an S3 backend with a DynamoDB table configured for locking, Terraform acquires a lock before making changes; the second `apply` will wait or fail with a "state locked" error until the first operation completes, preventing concurrent conflicting writes.

</details>

<details markdown="1">
<summary>❓ <b>9. What is `terraform state` command used for, and give an example subcommand.</b></summary>
<br>

A set of commands for advanced state manipulation, e.g., `terraform state list` (list resources in state), `terraform state mv` (rename/move a resource in state without destroying/recreating it), `terraform state rm` (remove a resource from state without destroying the actual infrastructure).

</details>

<details markdown="1">
<summary>🎯 <b>10. Scenario: You renamed a resource in your `.tf` file (e.g., from `aws_instance.web` to `aws_instance.app_server`), and now Terraform wants to destroy and recreate it. How do you prevent this?</b></summary>
<br>

Use `terraform state mv aws_instance.web aws_instance.app_server` to update the state file's mapping to match the renamed resource address, so Terraform recognizes it as the same existing resource rather than planning a destroy/create - avoiding unnecessary downtime.

</details>

<details markdown="1">
<summary>❓ <b>11. What is `terraform import` used for?</b></summary>
<br>

Brings an existing resource (created manually or outside Terraform) under Terraform's management by adding it to the state file, matched against a resource block you've already written in configuration - without recreating the actual infrastructure.

</details>

<details markdown="1">
<summary>🎯 <b>12. Scenario: A critical production database was created manually in the AWS console before your team adopted Terraform. How do you bring it under IaC management safely?</b></summary>
<br>

Write a Terraform resource block matching the database's exact current configuration, run `terraform import` to link it to the existing resource in state, then run `terraform plan` to verify there's no drift (no unintended changes proposed) before considering it fully managed - adjusting the configuration until `plan` shows no diff.

</details>

## 🧩 Modules

<details markdown="1">
<summary>❓ <b>13. What is a Terraform Module and why use them?</b></summary>
<br>

A reusable, self-contained package of Terraform configuration (input variables, resources, outputs) that can be called multiple times with different parameters - promoting DRY (Don't Repeat Yourself) principles, consistency, and easier maintenance across environments/projects.

</details>

<details markdown="1">
<summary>🎯 <b>14. Scenario: You need to provision identical VPC architectures for dev, staging, and production, with only size/CIDR differences. How do you structure this with Terraform?</b></summary>
<br>

Create a reusable VPC module (encapsulating subnets, route tables, NAT gateways, etc.) accepting input variables (CIDR range, environment name, AZ count), then call the module three times (once per environment) with environment-specific variable values - avoiding copy-pasted configuration across environments.

</details>

<details markdown="1">
<summary>❓ <b>15. What is the difference between a root module and a child module?</b></summary>
<br>

The root module is the main working directory where you run `terraform apply` (the entry point). Child modules are called/referenced from the root (or other modules) via a `module` block, encapsulating reusable logic - Terraform builds a hierarchical graph combining root and all called child modules.

</details>

## 🔀 Variables, Outputs & State Isolation

<details markdown="1">
<summary>❓ <b>16. What are Input Variables and Output Values in Terraform?</b></summary>
<br>

Input Variables (`variable` blocks) parameterize a module/configuration, allowing customization without editing the code itself. Output Values (`output` blocks) expose specific values (e.g., a resource's ID or IP) from a module for use elsewhere (other modules, or displayed after `apply`, or consumed by remote state data sources).

</details>

<details markdown="1">
<summary>❓ <b>17. What is a `terraform.tfvars` file used for?</b></summary>
<br>

A file to set input variable values automatically (without needing `-var` flags on the CLI each time), often used to separate environment-specific values (e.g., `prod.tfvars`, `dev.tfvars`) from the reusable configuration logic.

</details>

<details markdown="1">
<summary>❓ <b>18. How do you manage multiple environments (dev/stage/prod) with Terraform - what are the common strategies?</b></summary>
<br>

Workspaces (same configuration, different state files per workspace - simpler but limited isolation, risk of applying to wrong environment if not careful), Directory-per-environment (separate directories/configuration per environment, more explicit and safer but some duplication), or a combination with shared modules and environment-specific `.tfvars`/backend configs - directory-per-environment with shared modules is generally considered the safer, more scalable pattern for production use.

</details>

<details markdown="1">
<summary>🎯 <b>19. Scenario: An engineer accidentally applied a change intended for "dev" against the "prod" workspace due to not checking their current workspace. How do you prevent this class of mistake going forward?</b></summary>
<br>

Move away from relying solely on `terraform workspace select` (error-prone) toward separate state files/backend configurations per environment (distinct directories or distinct backend key paths), use CI/CD pipelines with explicit environment targeting (not manual local applies for prod), and add safeguards like requiring manual approval gates for production applies.

</details>

## ⚙️ Terraform Plan/Apply Mechanics

<details markdown="1">
<summary>❓ <b>20. What does `terraform plan` actually do internally?</b></summary>
<br>

It refreshes the state (optionally, checking real infrastructure against last-known state), builds a dependency graph of resources, and computes the diff between desired configuration and current state, producing an execution plan showing what will be created/updated/destroyed - without making any actual changes.

</details>

<details markdown="1">
<summary>❓ <b>21. What is the Terraform dependency graph and why does it matter?</b></summary>
<br>

Terraform builds a directed acyclic graph (DAG) of all resources based on explicit and implicit references between them, determining the correct order of operations (e.g., creating a VPC before a subnet that references it) and enabling safe parallelization of independent resource creation.

</details>

<details markdown="1">
<summary>🎯 <b>22. Scenario: You need Resource B to wait for Resource A to be fully created, but there's no direct attribute reference between them. How do you enforce ordering?</b></summary>
<br>

Use the `depends_on` meta-argument explicitly on Resource B, referencing Resource A - forcing Terraform to respect the ordering in its dependency graph even without an implicit reference (e.g., an IAM policy attachment that must happen after an S3 bucket policy is applied, even if not directly referencing bucket attributes).

</details>

<details markdown="1">
<summary>❓ <b>23. What is `lifecycle` meta-argument and its common uses (`create_before_destroy`, `prevent_destroy`, `ignore_changes`)?</b></summary>
<br>

`create_before_destroy`: creates the replacement resource before destroying the old one (avoiding downtime during replacement, e.g., for an ASG launch template change). `prevent_destroy`: blocks any plan that would destroy the resource, as a safety guard for critical resources (e.g., production databases). `ignore_changes`: tells Terraform to ignore drift on specific attributes (e.g., attributes modified outside Terraform, like auto-scaling adjusting desired count).

</details>

<details markdown="1">
<summary>🎯 <b>24. Scenario: An RDS instance must never be accidentally destroyed by a `terraform destroy` or a misconfigured plan. How do you protect it?</b></summary>
<br>

Add `lifecycle { prevent_destroy = true }` to the resource block - any plan/apply/destroy attempting to destroy that resource will fail with an error, requiring the lifecycle block to be explicitly removed first as a deliberate, auditable action.

</details>

## 🎯 Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>25. Scenario: You need to run Terraform safely as part of a CI/CD pipeline (no manual local applies), with mandatory review before production changes. How do you design this?</b></summary>
<br>

Run `terraform plan` automatically on PR creation (posting the plan output as a PR comment for review), require manual approval (e.g., via a pipeline gate) before running `terraform apply` on merge to main, use a remote backend with state locking, and use a dedicated CI service role with least-privilege IAM permissions (via OIDC, not static credentials).

</details>

<details markdown="1">
<summary>❓ <b>26. How do you handle secrets (e.g., database passwords) in Terraform configuration securely?</b></summary>
<br>

Avoid hardcoding secrets in `.tf`/`.tfvars` files (especially if committed to version control); instead, use a secrets manager data source (e.g., `aws_secretsmanager_secret_version`) to fetch secrets at apply time, or mark sensitive variables with `sensitive = true` (hides them from CLI output/plan display, though they're still stored in plaintext in state unless the backend itself encrypts it).

</details>

<details markdown="1">
<summary>❓ <b>27. Why is Terraform State considered sensitive, and how do you protect it?</b></summary>
<br>

State files can contain sensitive data in plaintext (e.g., resource attributes like database passwords or private keys), so protect it with a backend that supports encryption at rest (S3 with SSE-KMS), strict access control (IAM policies limiting who can read the state bucket), and avoid ever committing state files to version control.

</details>

<details markdown="1">
<summary>🎯 <b>28. Scenario: A `terraform apply` partially fails midway (some resources created, others failed). What's the state of your infrastructure and how do you recover?</b></summary>
<br>

Terraform updates state incrementally as resources are created/modified, so successfully created resources ARE tracked in state even if the overall apply reports failure. Recovery: review the error, fix the underlying issue (e.g., a quota limit or naming conflict), and simply re-run `terraform apply` - Terraform will resume from the current state, only creating/fixing what's still needed.

</details>

<details markdown="1">
<summary>❓ <b>29. What is Terraform Drift and how do you detect/handle it?</b></summary>
<br>

Drift occurs when actual infrastructure differs from what's recorded in Terraform state (e.g., someone manually changed a security group rule in the console). Detect via `terraform plan` (shows unexpected changes) run regularly/on a schedule; handle by either updating the configuration to match the manual change (if intentional) and importing/reconciling, or reverting the drift by applying the original configuration, and enforcing policies (SCPs, restricted console access) to prevent unauthorized manual changes going forward.

</details>

<details markdown="1">
<summary>🎯 <b>30. Scenario: You need to test infrastructure changes without risking your actual cloud resources or incurring cost. What tools/approaches help?</b></summary>
<br>

Use `terraform plan` extensively before applying, use tools like `terraform validate`/`tflint`/`checkov` for static analysis and policy compliance checks in CI, consider a sandbox/dev AWS account for testing risky changes, and use tools like LocalStack for local AWS service emulation for very early-stage testing without touching real cloud resources.

</details>

<details markdown="1">
<summary>❓ <b>31. What is the purpose of `terraform fmt` and `terraform validate`?</b></summary>
<br>

`terraform fmt` automatically rewrites configuration files to a canonical formatting style (consistency across a team). `terraform validate` checks configuration syntax and internal consistency (e.g., correct argument types) without contacting any provider APIs or requiring valid credentials - useful as an early, fast CI check.

</details>

<details markdown="1">
<summary>❓ <b>32. How would you structure a large Terraform codebase for a company managing dozens of microservices and multiple environments?</b></summary>
<br>

Use a modular structure: reusable modules for common patterns (VPC, ECS service, RDS instance) in a shared modules repository/registry, environment-specific root configurations (composing those modules) per environment/account, remote state per environment/component (avoiding one giant monolithic state file which increases blast radius and slows down plan/apply), and a CI/CD pipeline enforcing plan review and approval gates for changes.

</details>

<details markdown="1">
<summary>❓ <b>33. What is a Terraform Workspace, precisely, and what's a common misconception about it?</b></summary>
<br>

A workspace is a named instance of state within the SAME backend/configuration, allowing you to manage multiple distinct sets of infrastructure from one configuration (e.g., `terraform workspace new dev`). A common misconception is treating workspaces as a full substitute for separate environments with different configurations/variables - workspaces only isolate STATE, not variable values or backend config, so many teams find directory-based separation safer/clearer for genuinely different environments like dev vs prod.

</details>

<details markdown="1">
<summary>🎯 <b>34. Scenario: Your Terraform apply for a large infrastructure change is taking a very long time and you want to speed it up safely. What options exist?</b></summary>
<br>

Increase parallelism (`-parallelism=n` flag, default is 10) if resources are largely independent, break the configuration into smaller, more focused state files/modules (reducing the blast radius and per-run resource count), and ensure you're not doing unnecessary `-refresh=true` full-state refreshes when not needed for very large state files.

</details>

<details markdown="1">
<summary>❓ <b>35. What is the difference between Terraform and tools like AWS CloudFormation/Pulumi, and when might you choose Terraform?</b></summary>
<br>

CloudFormation is AWS-native (no multi-cloud support, tightly integrated with AWS features often faster than Terraform to support new services). Pulumi allows using general-purpose programming languages (Python/TypeScript) instead of a DSL. Terraform's key advantages are strong multi-cloud/multi-provider support (consistent workflow across AWS/Azure/GCP/Kubernetes/etc.), a mature ecosystem/module registry, and a declarative HCL language that's simpler to reason about than full imperative code - often chosen for multi-cloud or provider-agnostic infrastructure teams.

</details>

