<div align="center" markdown="1">

# 🐳 Docker
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-Docker-blue?style=for-the-badge&logo=docker&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is Docker and what problem does it solve?</b></summary>
<br>

Docker is a containerization platform that packages an application with all its dependencies into a lightweight, portable container, ensuring it runs consistently across different environments (dev, test, prod) - solving the classic "it works on my machine" problem.

</details>

<details markdown="1">
<summary>❓ <b>2. Difference between a Container and a Virtual Machine.</b></summary>
<br>

A VM virtualizes hardware and runs a full guest OS on a hypervisor - heavier, slower to boot, more isolated. A Container virtualizes at the OS level, sharing the host kernel while isolating processes/filesystem - lightweight, starts in seconds, and much more resource-efficient since it doesn't need a full OS per instance.

</details>

<details markdown="1">
<summary>❓ <b>3. What is a Docker Image vs a Container?</b></summary>
<br>

An Image is a read-only, immutable template (built from a Dockerfile) containing the application code, runtime, libraries, and dependencies. A Container is a running (or stopped) instance of an image - the writable, live execution of that template.

</details>

<details markdown="1">
<summary>❓ <b>4. What is a Dockerfile?</b></summary>
<br>

A text file containing a sequence of instructions (`FROM`, `RUN`, `COPY`, `CMD`, etc.) that Docker uses to build an image layer by layer, defining the environment and steps needed to package the application.

</details>

## 🧱 Dockerfile & Image Building

<details markdown="1">
<summary>❓ <b>5. Explain the difference between `CMD` and `ENTRYPOINT`.</b></summary>
<br>

`ENTRYPOINT` defines the fixed executable that always runs when the container starts (harder to override, meant for the "main" command). `CMD` provides default arguments (to `ENTRYPOINT` or as the command itself if no `ENTRYPOINT`) that CAN be easily overridden at `docker run` time - often used together: `ENTRYPOINT` sets the binary, `CMD` sets default arguments.

</details>

<details markdown="1">
<summary>❓ <b>6. What is a multi-stage build and why use it?</b></summary>
<br>

A Dockerfile technique using multiple `FROM` stages, where you build/compile the application in one stage (with all build tools/dependencies) and copy only the final artifacts into a minimal final stage - dramatically reducing final image size and attack surface by excluding build-time dependencies from the production image.

</details>

<details markdown="1">
<summary>🎯 <b>7. Scenario: Your Node.js application's Docker image is 1.2GB, mostly due to build tools and dev dependencies. How do you optimize it?</b></summary>
<br>

Use a multi-stage build: Stage 1 uses a full Node image to `npm install` and build/transpile the app; Stage 2 uses a minimal base image (e.g., `node:20-alpine` or `distroless`) and copies only the compiled output and production `node_modules` from Stage 1 - stripping out build tools, dev dependencies, and unnecessary layers.

</details>

<details markdown="1">
<summary>❓ <b>8. What is Docker Layer Caching and how do you optimize Dockerfile instruction order for it?</b></summary>
<br>

Each instruction in a Dockerfile creates a cached layer; if an instruction and its inputs haven't changed, Docker reuses the cached layer instead of rebuilding it. Optimize by placing rarely-changing instructions (e.g., installing dependencies from a lockfile) BEFORE frequently-changing instructions (e.g., copying application source code), so code changes don't invalidate the expensive dependency-install layer cache.

</details>

<details markdown="1">
<summary>❓ <b>9. Why should you avoid running containers as root?</b></summary>
<br>

Running as root inside a container increases the security risk if an attacker escapes the container or exploits a vulnerability, since root in the container maps to elevated privileges on the host in certain misconfigurations - best practice is to create and switch to a non-root user (`USER` instruction) in the Dockerfile.

</details>

<details markdown="1">
<summary>❓ <b>10. What is a `.dockerignore` file and why is it important?</b></summary>
<br>

Similar to `.gitignore`, it excludes specified files/directories (e.g., `.git`, `node_modules`, secrets, local env files) from being sent to the Docker build context, reducing build time/image size and preventing accidental inclusion of sensitive files in the image.

</details>

## 🖼️ Images & Registries

<details markdown="1">
<summary>❓ <b>11. What is an image tag and what's the risk of using `latest`?</b></summary>
<br>

A tag is a human-readable label for a specific image version (e.g., `myapp:1.2.0`). Using `latest` is risky in production because it's mutable and ambiguous - it doesn't guarantee a specific, reproducible version, making rollbacks and debugging difficult; pin to explicit version tags or digests instead.

</details>

<details markdown="1">
<summary>🎯 <b>12. Scenario: You need to guarantee that exactly the same image content is deployed across environments, immune to tag reassignment. How?</b></summary>
<br>

Reference the image by its SHA256 digest (`myapp@sha256:abcd123...`) instead of a mutable tag - the digest is a cryptographic hash of the image content, so it's immutable and guarantees byte-for-byte identical content regardless of tag changes.

</details>

<details markdown="1">
<summary>❓ <b>13. What is Docker Content Trust (DCT)?</b></summary>
<br>

A feature that enables image signing and verification (using Notary/TUF under the hood), ensuring that only signed, trusted images can be pulled/run, protecting against tampered or malicious images in the supply chain.

</details>

## 🌐 Networking

<details markdown="1">
<summary>❓ <b>14. What are the Docker network drivers and their use cases?</b></summary>
<br>

`bridge` (default, isolated virtual network on a single host, containers communicate via internal IPs/DNS), `host` (container shares the host's network namespace directly, no isolation, used for performance-critical cases), `overlay` (multi-host networking for Docker Swarm/multi-node clusters), `none` (no networking), and `macvlan` (assigns a container a MAC address, appearing as a physical device on the network).

</details>

<details markdown="1">
<summary>❓ <b>15. How do containers on the same custom bridge network communicate with each other?</b></summary>
<br>

Docker provides automatic DNS resolution by container name on user-defined bridge networks - containers can reach each other using their container/service name as a hostname, without needing to know IP addresses (unlike the legacy default bridge network, which requires explicit linking or IP addresses).

</details>

<details markdown="1">
<summary>🎯 <b>16. Scenario: Two containers on the default bridge network can't resolve each other by name. Why, and how do you fix it?</b></summary>
<br>

The default `bridge` network doesn't support automatic DNS resolution by container name (a legacy limitation) - fix by creating a user-defined bridge network (`docker network create mynet`) and attaching both containers to it, enabling built-in DNS-based service discovery.

</details>

## 💽 Storage & Volumes

<details markdown="1">
<summary>❓ <b>17. Difference between a Docker Volume and a Bind Mount.</b></summary>
<br>

Volumes are managed entirely by Docker (stored in Docker's storage area, e.g., `/var/lib/docker/volumes`), portable, and the recommended way to persist data. Bind Mounts map a specific host filesystem path directly into the container, giving direct access to host files but tying the container to that host's directory structure (less portable, more error-prone).

</details>

<details markdown="1">
<summary>❓ <b>18. Why is data inside a container lost when it's removed, and how do volumes solve this?</b></summary>
<br>

Containers have an ephemeral, writable layer on top of the read-only image layers - when the container is removed, this layer (and any data written to it) is deleted. Volumes exist independently of the container's lifecycle, so data persists even if the container is deleted and can be reattached to a new container.

</details>

<details markdown="1">
<summary>🎯 <b>19. Scenario: You need a database container's data to survive container restarts/recreations. How do you configure this?</b></summary>
<br>

Mount a named Docker Volume to the database's data directory (e.g., `-v pgdata:/var/lib/postgresql/data`), ensuring the actual data lives in the volume rather than the container's writable layer, so recreating the container (e.g., during an image update) doesn't lose the data.

</details>

## 🎼 Docker Compose

<details markdown="1">
<summary>❓ <b>20. What is Docker Compose and why use it?</b></summary>
<br>

A tool for defining and running multi-container applications using a single YAML file (`docker-compose.yml`), specifying services, networks, volumes, and their relationships - simplifying local development and testing of applications with multiple interdependent containers.

</details>

<details markdown="1">
<summary>🎯 <b>21. Scenario: You need a local dev environment with a web app, a Redis cache, and a Postgres database, all networked together. How do you set this up?</b></summary>
<br>

Define a `docker-compose.yml` with three services (`web`, `redis`, `db`), each with their image/build context, environment variables, and `depends_on` relationships - Compose automatically creates a shared network so services can reach each other by service name, and you bring it all up with a single `docker compose up`.

</details>

<details markdown="1">
<summary>❓ <b>22. What is `depends_on` and its limitation regarding readiness?</b></summary>
<br>

`depends_on` controls the STARTUP ORDER of containers, but doesn't guarantee the dependent service is actually READY to accept connections (e.g., a DB container might be running but not yet accepting connections) - applications should implement their own retry/wait logic, or use a healthcheck-based `depends_on: condition: service_healthy`.

</details>

## 🛡️ Security

<details markdown="1">
<summary>❓ <b>23. What are common Docker security best practices?</b></summary>
<br>

Use minimal base images (alpine/distroless), don't run as root, scan images for vulnerabilities (Trivy/ECR scanning), avoid embedding secrets in images/Dockerfiles (use runtime secret injection), keep the Docker daemon/host patched, use read-only root filesystems where possible, and limit container capabilities (`--cap-drop`) to only what's needed.

</details>

<details markdown="1">
<summary>🎯 <b>24. Scenario: A security scan reveals your production image contains a leaked API key baked into a layer during build. How does this happen and how do you prevent it?</b></summary>
<br>

It happens when a secret is passed as a build ARG/ENV or copied into the image during a build step - even if later removed in a subsequent layer, it remains in the image history/layers. Prevent by using Docker BuildKit's `--secret` mount feature (secrets available only during the specific RUN step, never persisted to any layer), or fetching secrets at runtime rather than baking them into the image.

</details>

<details markdown="1">
<summary>❓ <b>25. What is the principle behind using read-only containers?</b></summary>
<br>

Running a container with a read-only root filesystem (`--read-only` flag) prevents any process inside from writing to the container's filesystem (except explicitly mounted writable volumes/tmpfs), reducing the impact of a compromised process trying to persist malware or tamper with the application.

</details>

## 🎯 Real-Time Scenarios & Troubleshooting

<details markdown="1">
<summary>🎯 <b>26. Scenario: A container exits immediately after starting, even though the image builds successfully. How do you troubleshoot?</b></summary>
<br>

Run `docker logs <container>` to see the exit reason/stack trace, check `docker inspect` for the exit code, verify the `CMD`/`ENTRYPOINT` process isn't a short-lived command (containers exit when the main process exits - e.g., forgetting a long-running foreground process), and try running interactively (`docker run -it ... /bin/sh`) to debug manually.

</details>

<details markdown="1">
<summary>🎯 <b>27. Scenario: A container is consuming excessive memory and getting OOM-killed. How do you diagnose and fix this?</b></summary>
<br>

Use `docker stats` to monitor real-time resource usage, check application-level memory leaks via logs/profiling, and set explicit memory limits (`--memory`) to catch issues early and prevent one container from starving the host - combined with proper application tuning (e.g., JVM heap sizing) matched to the container's memory limit.

</details>

<details markdown="1">
<summary>❓ <b>28. How do you debug networking issues between containers (e.g., one container can't reach another's exposed port)?</b></summary>
<br>

Verify both containers are on the same user-defined network (`docker network inspect`), check the target container is actually listening on the expected port (`docker exec ... netstat`/`ss`), verify no firewall/iptables rules on the host are blocking traffic, and test connectivity directly with `docker exec` + `curl`/`nc` between containers.

</details>

<details markdown="1">
<summary>🎯 <b>29. Scenario: You need to reduce the attack surface and image size for a production Go application as much as possible. What approach do you take?</b></summary>
<br>

Use a multi-stage build: compile the Go binary statically in a build stage, then copy only the compiled binary into a `scratch` or `distroless` final stage (no shell, no package manager, minimal OS utilities) - resulting in a tiny, hardened image with almost no attack surface.

</details>

<details markdown="1">
<summary>❓ <b>30. What is the difference between `docker stop` and `docker kill`?</b></summary>
<br>

`docker stop` sends a SIGTERM to the main process (allowing graceful shutdown), waits a grace period (default 10s), then sends SIGKILL if it hasn't exited. `docker kill` sends SIGKILL (or a specified signal) immediately, forcefully terminating the container without a graceful shutdown opportunity.

</details>

<details markdown="1">
<summary>🎯 <b>31. Scenario: You want to limit a container's CPU and memory usage to prevent it from starving other containers on the same host. How?</b></summary>
<br>

Use resource constraint flags at runtime: `--memory=512m` (hard memory limit) and `--cpus=1.0` (limit CPU usage), or configure equivalent limits in a Docker Compose file/orchestrator (Kubernetes resource requests/limits) for consistent enforcement across environments.

</details>

<details markdown="1">
<summary>❓ <b>32. How would you set up a CI pipeline step to build, scan, and push a Docker image securely?</b></summary>
<br>

Build the image using BuildKit with cache optimization, run a vulnerability scanner (Trivy/ECR scan) against the built image and fail the pipeline on critical/high findings, tag the image with an immutable identifier (git SHA), sign it (Docker Content Trust/cosign) if required, then push to the registry only after all checks pass - using short-lived, scoped registry credentials (OIDC) rather than static secrets.

</details>

<details markdown="1">
<summary>❓ <b>33. What is the difference between `docker exec` and `docker attach`?</b></summary>
<br>

`docker exec` starts a NEW process inside an already-running container (e.g., opening a new shell session) without affecting the container's main process. `docker attach` connects your terminal directly to the container's main process's stdin/stdout/stderr (the same process that's already running) - useful for viewing live output or interacting with the primary process itself, but risky since exiting incorrectly can stop the container.

</details>

<details markdown="1">
<summary>🎯 <b>34. Scenario: You need containers to communicate securely across multiple Docker hosts (not just a single machine). What's the solution?</b></summary>
<br>

Use an `overlay` network (via Docker Swarm mode), or move to a full orchestrator like Kubernetes which handles multi-host networking natively (via CNI plugins) - a single Docker host's bridge network alone cannot span multiple physical/virtual machines.

</details>

<details markdown="1">
<summary>❓ <b>35. How do health checks work in Docker and why are they useful?</b></summary>
<br>

The `HEALTHCHECK` instruction in a Dockerfile (or equivalent in Compose/orchestrators) defines a command Docker periodically runs inside the container to determine if it's actually healthy (not just "running") - orchestrators use this status to avoid routing traffic to, or to automatically restart, containers that are running but not functioning correctly (e.g., app hung but process still alive).

</details>

