<div align="center" markdown="1">

# 🔍 Trivy
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-Trivy-blue?style=for-the-badge&logo=aquasecurity&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is Trivy?</b></summary>
<br>

An open-source, all-in-one security scanner (by Aqua Security) used to detect vulnerabilities, misconfigurations, secrets, and license issues in container images, filesystems, Git repositories, Kubernetes clusters, and Infrastructure as Code files.

</details>

<details markdown="1">
<summary>❓ <b>2. What are the main scan targets Trivy supports?</b></summary>
<br>

Container Images (OS packages + application dependencies), Filesystem (local directories/files), Git Repositories (remote or local), Kubernetes clusters (live cluster resources), and IaC configurations (Terraform, CloudFormation, Dockerfile, Kubernetes manifests, Helm charts).

</details>

<details markdown="1">
<summary>❓ <b>3. What types of security issues can Trivy detect?</b></summary>
<br>

Vulnerabilities (known CVEs in OS packages and language-specific dependencies), Misconfigurations (insecure settings in IaC/Dockerfiles/Kubernetes manifests, e.g., via built-in Rego/OPA policies), Secrets (hardcoded API keys, passwords, tokens accidentally committed into code/images), and License compliance issues (identifying problematic open-source licenses).

</details>

<details markdown="1">
<summary>❓ <b>4. How does Trivy detect vulnerabilities in a container image?</b></summary>
<br>

It scans the image's OS package manager metadata (e.g., apt/yum/apk package lists) and language-specific dependency files/lockfiles (e.g., `package-lock.json`, `requirements.txt`, `go.sum`), then cross-references them against vulnerability databases (NVD and various OS-specific advisory databases) to identify known CVEs affecting those exact versions.

</details>

## 🐛 Vulnerability Scanning

<details markdown="1">
<summary>🎯 <b>5. Scenario: You want to scan a Docker image and fail the build if any CRITICAL vulnerabilities are found. How?</b></summary>
<br>

`trivy image --severity CRITICAL --exit-code 1 myapp:latest` - this scans the image, filters to only CRITICAL severity findings, and returns a non-zero exit code if any are found, which a CI pipeline can use to fail the build/deployment step.

</details>

<details markdown="1">
<summary>❓ <b>6. What is the difference between scanning OS packages vs Language-specific dependencies in Trivy?</b></summary>
<br>

OS package scanning checks vulnerabilities in packages installed via the OS package manager (e.g., `openssl`, `glibc` in a Debian/Alpine base image). Language-specific dependency scanning checks vulnerabilities in application-level dependencies (e.g., an npm package, a Python pip package, a Java JAR) - Trivy covers both in a single scan, giving full-stack vulnerability visibility.

</details>

<details markdown="1">
<summary>❓ <b>7. What is a Trivy vulnerability database and how does it stay up to date?</b></summary>
<br>

Trivy downloads and caches a vulnerability database (aggregated from sources like NVD, GitHub Security Advisories, and OS vendor security trackers) periodically, ensuring scans reflect the latest known CVEs without needing manual updates - important to keep this database fresh in CI environments for accurate results.

</details>

<details markdown="1">
<summary>🎯 <b>8. Scenario: A Trivy scan flags a vulnerability in a package, but you've confirmed your application doesn't use the vulnerable code path and can't upgrade immediately. How do you handle this without blocking your pipeline indefinitely?</b></summary>
<br>

Use a `.trivyignore` file (or `--ignorefile`) to explicitly suppress that specific CVE ID with a documented justification/expiration reminder, ensuring the risk acceptance is intentional, auditable, and doesn't silently mask future genuinely new vulnerabilities.

</details>

<details markdown="1">
<summary>❓ <b>9. What does "Fixed Version" vs "No Fix Available" mean in Trivy's output, and how does it affect remediation strategy?</b></summary>
<br>

"Fixed Version" means upgrading to that specific package version resolves the CVE - straightforward remediation. "No Fix Available" means the maintainers haven't released a patch yet - remediation options include finding an alternative package, applying a temporary mitigation/workaround, or accepting the risk (with compensating controls) until a fix is released.

</details>

## ⚠️ Misconfiguration Scanning (IaC)

<details markdown="1">
<summary>❓ <b>10. How does Trivy scan Infrastructure as Code (Terraform, Kubernetes manifests, Dockerfiles)?</b></summary>
<br>

It uses built-in policies (based on Rego/OPA-style rules) to statically analyze IaC files for insecure configurations (e.g., an S3 bucket without encryption, a security group open to 0.0.0.0/0, a container running as root, a Kubernetes Pod without resource limits), without needing to actually deploy the infrastructure.

</details>

<details markdown="1">
<summary>🎯 <b>11. Scenario: You want to catch a misconfigured Terraform module (e.g., a publicly accessible S3 bucket) before it's ever applied to a real AWS account. How does Trivy help?</b></summary>
<br>

Run `trivy config /path/to/terraform` (or integrate into CI on every PR touching Terraform files) to statically scan the `.tf` files for known insecure patterns, failing the pipeline/flagging the PR before `terraform apply` ever provisions the actual insecure resource.

</details>

<details markdown="1">
<summary>❓ <b>12. What is the benefit of scanning Dockerfiles directly (not just built images) with Trivy?</b></summary>
<br>

Catches issues at the SOURCE level before even building the image (e.g., using `ADD` instead of `COPY` for remote URLs, running as root, using `latest` tags, exposing unnecessary secrets via ARG) - shifting security feedback even earlier ("shift left") than scanning the final built image.

</details>

## 🔍 Secret Scanning

<details markdown="1">
<summary>❓ <b>13. How does Trivy detect secrets, and where can it scan for them?</b></summary>
<br>

Uses pattern matching/regex rules (and entropy analysis) to detect common secret formats (AWS keys, private keys, generic API tokens, database connection strings) within container image layers, filesystems, and Git repository history/commits.

</details>

<details markdown="1">
<summary>🎯 <b>14. Scenario: A developer accidentally committed an AWS access key to a Git repository, then removed it in a later commit. Would Trivy's filesystem scan catch it?</b></summary>
<br>

A standard filesystem scan of the CURRENT working directory would NOT catch it (since the file no longer contains the secret in its current state), but scanning the Git repository/history specifically (`trivy repo` or scanning the `.git` directory) can detect secrets present in the commit history even if removed later - highlighting why secret scanning should target repo history, not just the current file tree.

</details>

## ☸️ Kubernetes & Cluster Scanning

<details markdown="1">
<summary>❓ <b>15. What does `trivy k8s` (Kubernetes scanning) provide?</b></summary>
<br>

Scans a live Kubernetes cluster's actual running resources (Deployments, Pods, ConfigMaps, RBAC configuration) for both vulnerabilities (in the images actually running) AND misconfigurations (e.g., overly permissive RBAC roles, privileged containers, missing network policies) - giving a real-time security posture view of what's actually deployed, not just what's in source control.

</details>

<details markdown="1">
<summary>🎯 <b>16. Scenario: You want continuous visibility into vulnerabilities across all workloads running in a production Kubernetes cluster, not just at deploy time. How?</b></summary>
<br>

Deploy Trivy Operator (a Kubernetes-native operator that continuously scans workloads in the cluster and exposes results as Kubernetes Custom Resources), integrated with dashboards/alerting, providing ongoing runtime visibility rather than a one-time point-in-time scan at CI/CD time.

</details>

## 🎯 CI/CD Integration & Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>17. Scenario: Design a CI/CD pipeline stage using Trivy to enforce security before deployment.</b></summary>
<br>

After building the Docker image, run `trivy image --severity HIGH,CRITICAL --exit-code 1 --ignore-unfixed myapp:$TAG`, failing the pipeline if high/critical vulnerabilities with available fixes are found; additionally run `trivy config` against IaC changes and `trivy fs` for secret scanning on the repository, all as gating steps before the image is pushed/deployed.

</details>

<details markdown="1">
<summary>❓ <b>18. What does the `--ignore-unfixed` flag do and why might a team choose to use it?</b></summary>
<br>

Excludes vulnerabilities that don't yet have a fix available from causing a pipeline failure, since blocking on unfixable issues provides no actionable remediation path and would perpetually fail the build - teams track these separately for monitoring/risk acceptance rather than gating deployment on them.

</details>

<details markdown="1">
<summary>🎯 <b>19. Scenario: Your organization wants to reduce Docker Hub rate-limiting issues and speed up scans by avoiding repeated vulnerability database downloads in every CI run. How do you optimize Trivy usage?</b></summary>
<br>

Cache the Trivy vulnerability database between CI runs (mounting a persistent cache directory or using a shared Trivy server mode), or run a centralized Trivy Server that CI clients connect to remotely, avoiding each pipeline run from re-downloading the full DB independently.

</details>

<details markdown="1">
<summary>❓ <b>20. What is Trivy's Client/Server mode and when would you use it?</b></summary>
<br>

Trivy can run as a standalone CLI (downloads/updates its own DB locally) or in Client/Server mode, where a central Trivy Server hosts the vulnerability database and scanning logic, and lightweight clients (in CI pipelines) send scan requests to it - useful at scale to avoid redundant DB downloads/updates across many parallel CI jobs and to centralize database update management.

</details>

<details markdown="1">
<summary>🎯 <b>21. Scenario: A security audit requires proof that every container image deployed to production over the last quarter was scanned and met a defined vulnerability threshold. How do you provide this evidence using Trivy?</b></summary>
<br>

Ensure every CI/CD pipeline run archives Trivy scan reports (JSON/SARIF output) as build artifacts or forwards them to a centralized security dashboard/SIEM, tagged with the image digest and deployment timestamp - providing an auditable trail correlating each deployed image with its corresponding scan result and pass/fail Quality Gate decision.

</details>

<details markdown="1">
<summary>❓ <b>22. How does Trivy compare to/complement tools like SonarQube in a DevSecOps pipeline?</b></summary>
<br>

SonarQube focuses on SAST for application source code quality/security (code smells, logic vulnerabilities, coverage). Trivy focuses on vulnerability/misconfiguration/secret scanning for the broader supply chain - dependencies, container images, IaC, and running clusters. Together they form complementary layers: SonarQube catches issues in the code you write, Trivy catches issues in what you package/deploy and the third-party components you depend on.

</details>

<details markdown="1">
<summary>🎯 <b>23. Scenario: You need to scan images before they're even pushed to a registry, as part of a pre-commit or local developer workflow (shifting security left further). How?</b></summary>
<br>

Provide developers with a pre-commit hook or a local `make scan`/Makefile target running `trivy image` (or `trivy fs`/`trivy config` for code/IaC) against their local build, giving immediate feedback before code is even pushed, reducing the number of security issues that reach CI/CD or later stages.

</details>

<details markdown="1">
<summary>❓ <b>24. What output formats does Trivy support and why does this matter for integration?</b></summary>
<br>

Table (human-readable CLI output), JSON (machine-readable for custom tooling/dashboards), SARIF (Static Analysis Results Interchange Format, natively supported by GitHub Code Scanning/Security tab), and others (CycloneDX/SPDX for SBOM generation) - supporting SARIF/JSON enables seamless integration into existing developer workflows (e.g., surfacing findings directly in GitHub PR security tabs) rather than requiring a separate dashboard to view results.

</details>

<details markdown="1">
<summary>❓ <b>25. What is an SBOM (Software Bill of Materials) and how does Trivy relate to it?</b></summary>
<br>

An SBOM is a complete inventory of all components/dependencies (and their versions) that make up a piece of software - Trivy can generate SBOMs (in CycloneDX or SPDX format) for container images/repositories, which is increasingly a regulatory/compliance requirement (e.g., for software supply chain security) and can also be used later to re-check for newly disclosed vulnerabilities without re-scanning the full artifact.

</details>

