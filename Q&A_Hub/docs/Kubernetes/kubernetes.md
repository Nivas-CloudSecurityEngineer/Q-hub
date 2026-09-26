<div align="center" markdown="1">

# ☸️ Kubernetes
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-Kubernetes-blue?style=for-the-badge&logo=kubernetes&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is Kubernetes and what problem does it solve?</b></summary>
<br>

An open-source container orchestration platform that automates deployment, scaling, networking, and management of containerized applications across a cluster of machines, solving problems like self-healing, load balancing, rolling updates, and declarative infrastructure management at scale.

</details>

<details markdown="1">
<summary>❓ <b>2. What is the Kubernetes architecture - Control Plane vs Worker Nodes?</b></summary>
<br>

Control Plane components (API Server, etcd, Scheduler, Controller Manager) manage the cluster's desired state and make global decisions. Worker Nodes run the actual application workloads (Pods), managed by the Kubelet (node agent), Kube-proxy (networking), and the container runtime.

</details>

<details markdown="1">
<summary>❓ <b>3. What is a Pod and why is it the smallest deployable unit (not a container)?</b></summary>
<br>

A Pod is a group of one or more containers that share the same network namespace (IP address, port space) and storage volumes, always scheduled together on the same node - it's the smallest unit because tightly coupled containers (e.g., an app + a sidecar log shipper) need to share resources/lifecycle, which a single container alone can't model.

</details>

<details markdown="1">
<summary>❓ <b>4. What is etcd and why is it critical?</b></summary>
<br>

A distributed, consistent key-value store that holds the entire cluster state (all objects' desired/current state) - it's the single source of truth for Kubernetes; losing etcd data means losing the cluster's state entirely, so it must be backed up regularly and run with an odd number of replicas (typically 3 or 5) for quorum-based high availability.

</details>

## 🧱 Core Objects

<details markdown="1">
<summary>❓ <b>5. Difference between a Deployment, ReplicaSet, and Pod.</b></summary>
<br>

Pod: the actual running unit of containers. ReplicaSet: ensures a specified number of identical Pod replicas are running at all times (self-healing). Deployment: manages ReplicaSets, providing declarative updates, rollout history, and rollback capability - you typically manage Deployments directly, letting them handle ReplicaSets/Pods underneath.

</details>

<details markdown="1">
<summary>❓ <b>6. What is a Service and why is it needed?</b></summary>
<br>

An abstraction providing a stable network identity (ClusterIP/DNS name) for a set of Pods, since Pods are ephemeral and get new IPs when recreated - the Service uses label selectors to dynamically route traffic to the currently healthy matching Pods, providing load balancing and service discovery.

</details>

<details markdown="1">
<summary>❓ <b>7. What are the types of Kubernetes Services?</b></summary>
<br>

ClusterIP (default, internal-only virtual IP reachable within the cluster), NodePort (exposes the service on a static port on every node's IP, for basic external access), LoadBalancer (provisions an external cloud load balancer, e.g., an AWS ELB, pointing to the service), and ExternalName (maps a service to an external DNS name, no proxying).

</details>

<details markdown="1">
<summary>🎯 <b>8. Scenario: You need to expose a web application to the internet with a stable, cloud-provider-managed load balancer. Which Service type?</b></summary>
<br>

LoadBalancer - it automatically provisions an external cloud load balancer (e.g., an ALB/NLB in AWS via a controller) that routes external traffic to the service's backing Pods, without manual load balancer configuration.

</details>

<details markdown="1">
<summary>❓ <b>9. What is an Ingress and how does it differ from a Service?</b></summary>
<br>

Ingress is a Layer 7 routing rule set (host/path-based routing, TLS termination) that sits in front of one or more Services, typically requiring an Ingress Controller (e.g., NGINX, ALB Ingress Controller) to actually implement the routing - allowing a single external entry point (and load balancer) to route to many different Services based on hostname/path, unlike a basic Service which just exposes one set of Pods.

</details>

<details markdown="1">
<summary>❓ <b>10. What is a ConfigMap and a Secret, and how do they differ?</b></summary>
<br>

Both store configuration data outside the container image, injected as environment variables or mounted volumes. ConfigMaps store non-sensitive plain-text configuration (feature flags, config files). Secrets store sensitive data (passwords, tokens, certs) - base64-encoded (not encrypted by default at rest unless encryption at rest is explicitly configured), with tighter RBAC conventions typically applied.

</details>

<details markdown="1">
<summary>🎯 <b>11. Scenario: You store a database password in a Kubernetes Secret but a teammate says base64 encoding isn't real security. Are they right?</b></summary>
<br>

Yes - base64 is just an encoding, not encryption, and anyone with API access to read the Secret object can trivially decode it. Real protection requires enabling encryption at rest for Secrets in etcd, strict RBAC limiting who can read Secret objects, and ideally integrating with an external secrets manager (Vault, AWS Secrets Manager via External Secrets Operator) for stronger secret lifecycle management.

</details>

## ⚙️ Workloads

<details markdown="1">
<summary>❓ <b>12. What is a StatefulSet and when do you use it instead of a Deployment?</b></summary>
<br>

Used for stateful applications needing stable, unique network identities and persistent storage per replica (e.g., databases, Kafka, Zookeeper) - unlike Deployments, StatefulSet Pods get predictable, stable names (`pod-0`, `pod-1`), stable storage (via PersistentVolumeClaim templates that persist across rescheduling), and are created/scaled/deleted in strict ordinal order.

</details>

<details markdown="1">
<summary>❓ <b>13. What is a DaemonSet and its common use cases?</b></summary>
<br>

Ensures exactly one copy of a Pod runs on every (or a selected subset of) node in the cluster - commonly used for node-level agents like log collectors (Fluentd/Fluent Bit), monitoring agents (node-exporter), or network/storage plugins.

</details>

<details markdown="1">
<summary>❓ <b>14. What is a Job and a CronJob?</b></summary>
<br>

A Job runs a Pod to completion (for a batch/one-off task), retrying on failure up to a specified limit, and considered "done" once it succeeds. A CronJob schedules Jobs to run periodically based on a cron expression - useful for scheduled batch tasks like nightly reports or cleanup scripts.

</details>

<details markdown="1">
<summary>🎯 <b>15. Scenario: You need to run a database migration exactly once before deploying a new application version, ensuring it completes successfully first. How?</b></summary>
<br>

Use a Kubernetes Job (or a Helm pre-install/pre-upgrade hook wrapping a Job) to run the migration to completion, and gate the actual application Deployment rollout on the Job's successful completion (e.g., via CI/CD pipeline logic checking the Job status, or Helm hook weights/ordering).

</details>

## 📐 Scheduling & Resource Management

<details markdown="1">
<summary>❓ <b>16. What are Resource Requests and Limits, and why are both important?</b></summary>
<br>

Requests specify the minimum resources (CPU/memory) a Pod needs, used by the Scheduler to decide which node has capacity to place it. Limits specify the maximum resources a Pod can consume - exceeding a memory limit causes the Pod to be OOM-killed, while exceeding a CPU limit causes throttling (not killing). Setting both correctly prevents both under-provisioning (scheduling failures/instability) and a single Pod starving others on the same node.

</details>

<details markdown="1">
<summary>❓ <b>17. What happens to a Pod that exceeds its memory limit vs its CPU limit?</b></summary>
<br>

Exceeding the memory limit results in the Pod being OOMKilled (terminated) since memory can't be compressed/throttled. Exceeding the CPU limit results in CPU throttling (the process is slowed down via the kernel's CFS scheduler) rather than being killed, since CPU can be time-sliced.

</details>

<details markdown="1">
<summary>🎯 <b>18. Scenario: A Pod keeps getting OOMKilled in production despite the application seemingly not using much memory under normal load. How do you troubleshoot?</b></summary>
<br>

Check actual memory usage over time via `kubectl top pod` and Grafana/Prometheus dashboards to identify spikes (e.g., during traffic bursts or specific operations like large batch processing), verify the memory limit is appropriately sized for peak (not just average) usage, check for memory leaks in the application, and review if JVM/runtime-level memory settings are aligned with the container's cgroup limit (a common mismatch issue).

</details>

<details markdown="1">
<summary>❓ <b>19. What are Node Affinity, Pod Affinity, and Taints/Tolerations used for?</b></summary>
<br>

Node Affinity: constrains which nodes a Pod can be scheduled on based on node labels (e.g., only GPU nodes). Pod Affinity/Anti-Affinity: influences Pod placement relative to OTHER pods (e.g., spread replicas across different nodes/AZs for HA, or co-locate a cache with its consuming app). Taints/Tolerations: taints repel Pods from a node unless the Pod has a matching toleration - used to dedicate nodes for specific workloads (e.g., reserving nodes for a particular team or GPU workloads).

</details>

<details markdown="1">
<summary>🎯 <b>20. Scenario: You want to ensure replicas of a critical Deployment are never all scheduled on the same node or AZ (for high availability). How?</b></summary>
<br>

Use Pod Anti-Affinity rules (`podAntiAffinity` with `topologyKey: kubernetes.io/hostname` or `topology.kubernetes.io/zone`) to instruct the scheduler to spread replicas across different nodes/AZs, preventing a single node/AZ failure from taking down all replicas simultaneously.

</details>

## 🌐 Networking

<details markdown="1">
<summary>❓ <b>21. How does Pod-to-Pod networking work across nodes in Kubernetes?</b></summary>
<br>

Kubernetes requires a flat network model where every Pod can reach every other Pod's IP directly without NAT, regardless of which node they're on - implemented by a CNI (Container Network Interface) plugin (Calico, Cilium, AWS VPC CNI, etc.) that handles the underlying routing/overlay networking.

</details>

<details markdown="1">
<summary>❓ <b>22. What is a Network Policy and why is it needed?</b></summary>
<br>

A resource that defines rules restricting which Pods can communicate with which other Pods/namespaces/IP ranges (essentially a firewall at the Pod level) - by default, Kubernetes allows all Pod-to-Pod traffic, so Network Policies are needed to enforce least-privilege network segmentation (requires a CNI plugin that supports enforcement, like Calico or Cilium).

</details>

<details markdown="1">
<summary>🎯 <b>23. Scenario: You need to ensure only the `frontend` namespace's Pods can talk to the `backend` namespace's Pods on port 8080, and nothing else. How?</b></summary>
<br>

Define a Network Policy in the `backend` namespace with a `podSelector` matching backend Pods, an `ingress` rule allowing traffic from Pods in the `frontend` namespace (via `namespaceSelector`) restricted to port 8080, with a default-deny-all policy also applied to ensure no other traffic is implicitly allowed.

</details>

<details markdown="1">
<summary>❓ <b>24. What is CoreDNS's role in Kubernetes?</b></summary>
<br>

The default cluster DNS provider, resolving Service names (e.g., `myservice.mynamespace.svc.cluster.local`) to their ClusterIP, enabling Pods to discover and communicate with Services by name rather than hardcoded IPs.

</details>

## 💾 Storage

<details markdown="1">
<summary>❓ <b>25. Difference between a PersistentVolume (PV) and a PersistentVolumeClaim (PVC).</b></summary>
<br>

A PV is a piece of actual storage provisioned in the cluster (by an admin or dynamically via a StorageClass), representing the underlying storage resource. A PVC is a request FOR storage made by a user/Pod, specifying size/access mode requirements - Kubernetes binds a PVC to a matching (or dynamically provisioned) PV.

</details>

<details markdown="1">
<summary>❓ <b>26. What is a StorageClass and dynamic provisioning?</b></summary>
<br>

A StorageClass defines a "class" of storage (e.g., SSD vs HDD, specific cloud provider parameters) and enables dynamic provisioning - when a PVC requests that StorageClass, Kubernetes automatically provisions a new PV (e.g., an EBS volume) on demand, rather than requiring an admin to pre-create PVs manually.

</details>

## 🚀 Deployments & Rollouts

<details markdown="1">
<summary>❓ <b>27. How does a Kubernetes Deployment perform a Rolling Update, and what controls its behavior?</b></summary>
<br>

It gradually replaces old ReplicaSet Pods with new ones, controlled by `maxSurge` (how many extra Pods can be created above the desired count during rollout) and `maxUnavailable` (how many Pods can be unavailable during rollout) - balancing rollout speed against maintaining availability.

</details>

<details markdown="1">
<summary>🎯 <b>28. Scenario: A new Deployment rollout is causing errors in production. How do you quickly roll back?</b></summary>
<br>

`kubectl rollout undo deployment/<name>` reverts to the previous ReplicaSet revision immediately, or specify a specific revision with `--to-revision=N` using the rollout history (`kubectl rollout history`).

</details>

<details markdown="1">
<summary>❓ <b>29. What are Readiness Probes and Liveness Probes, and why are both needed?</b></summary>
<br>

Liveness Probe determines if a container is alive/healthy - if it fails, Kubernetes restarts the container. Readiness Probe determines if a container is ready to receive traffic - if it fails, the Pod is removed from the Service's endpoints (no traffic routed) but NOT restarted, useful for temporary states like warming up a cache or waiting on a dependency.

</details>

<details markdown="1">
<summary>🎯 <b>30. Scenario: During a deployment, users experience errors because new Pods start receiving traffic before the application has finished initializing. How do you fix this?</b></summary>
<br>

Add a properly configured Readiness Probe (checking an actual application health endpoint, not just a TCP port) so Kubernetes only routes traffic to a Pod once it reports ready, preventing premature traffic routing during startup/warm-up.

</details>

## 📈 Autoscaling

<details markdown="1">
<summary>❓ <b>31. What is the Horizontal Pod Autoscaler (HPA)?</b></summary>
<br>

Automatically scales the number of Pod replicas in a Deployment/StatefulSet based on observed metrics (CPU/memory utilization by default, or custom/external metrics via the Metrics API), maintaining a target value similar in concept to AWS Auto Scaling's target tracking.

</details>

<details markdown="1">
<summary>❓ <b>32. What is the Cluster Autoscaler and how does it differ from HPA?</b></summary>
<br>

HPA scales the number of PODS based on resource utilization within existing node capacity. Cluster Autoscaler scales the number of NODES in the cluster itself, adding nodes when Pods can't be scheduled due to insufficient capacity, and removing underutilized nodes - the two work together (HPA scales pods, Cluster Autoscaler ensures there's node capacity to run them).

</details>

<details markdown="1">
<summary>🎯 <b>33. Scenario: HPA wants to scale up Pods due to high CPU, but new Pods stay "Pending" indefinitely. What's likely happening, and how do you fix it?</b></summary>
<br>

The cluster likely doesn't have enough available node capacity to schedule the new Pods - Cluster Autoscaler should detect this and provision new nodes; if it's not configured/working, enable/troubleshoot it, or check for other scheduling constraints (taints, resource requests too large for any available node type, PodDisruptionBudget conflicts).

</details>

## 🎯 Real-Time Scenarios & Troubleshooting

<details markdown="1">
<summary>🎯 <b>34. Scenario: A Pod is stuck in `CrashLoopBackOff`. How do you troubleshoot?</b></summary>
<br>

`kubectl describe pod <name>` for events, `kubectl logs <pod> --previous` to see the crashed container's last logs, check the container's exit code (`OOMKilled` = 137, application error = often 1), verify liveness probe isn't killing it prematurely, and check for missing ConfigMap/Secret dependencies or misconfigured startup commands.

</details>

<details markdown="1">
<summary>🎯 <b>35. Scenario: A Pod is stuck in `Pending` state and never gets scheduled. How do you diagnose?</b></summary>
<br>

`kubectl describe pod <name>` shows scheduling events/reasons (e.g., "Insufficient cpu," "node(s) had taints the pod didn't tolerate," or no nodes matching affinity rules) - check resource requests against actual node capacity, taints/tolerations, and node selectors/affinity rules.

</details>

<details markdown="1">
<summary>❓ <b>36. What is a PodDisruptionBudget (PDB) and why is it important during node maintenance/upgrades?</b></summary>
<br>

Defines the minimum number/percentage of Pods that must remain available during VOLUNTARY disruptions (node drains, cluster upgrades) - ensuring that maintenance operations (like draining a node for patching) don't take down too many replicas of a critical service simultaneously, preserving availability.

</details>

<details markdown="1">
<summary>🎯 <b>37. Scenario: You're performing a Kubernetes cluster upgrade and need to safely drain nodes one at a time without causing an outage. What do you configure beforehand?</b></summary>
<br>

Ensure appropriate PodDisruptionBudgets are set on critical Deployments/StatefulSets, verify sufficient replica counts and Pod Anti-Affinity for spread across nodes, and use `kubectl drain` (which respects PDBs) node by node rather than draining multiple nodes simultaneously.

</details>

<details markdown="1">
<summary>❓ <b>38. How do you manage secrets more securely than native Kubernetes Secrets in a production environment?</b></summary>
<br>

Integrate an external secrets management system (HashiCorp Vault, AWS Secrets Manager/Parameter Store via the External Secrets Operator or CSI Secrets Store driver), enable encryption at rest for the Kubernetes Secrets API/etcd, enforce strict RBAC on who can `get`/`list` Secret objects, and avoid mounting secrets as environment variables where possible (env vars can leak via logs/crash dumps more easily than files).

</details>

<details markdown="1">
<summary>❓ <b>39. What is a Helm Chart and why is it commonly used with Kubernetes?</b></summary>
<br>

A packaging format for Kubernetes applications, bundling templated YAML manifests with configurable values (`values.yaml`), enabling reusable, versioned, parameterized deployments (similar to a package manager for Kubernetes) - simplifying complex multi-resource application deployment and upgrades/rollbacks.

</details>

<details markdown="1">
<summary>🎯 <b>40. Scenario: You need multi-tenant isolation for different teams sharing a single Kubernetes cluster, balancing cost efficiency with security. How do you design this?</b></summary>
<br>

Use Namespaces per team/application for logical isolation, enforce ResourceQuotas and LimitRanges per namespace to prevent one team from consuming excessive cluster resources, apply RBAC (Roles/RoleBindings scoped per namespace) so teams can only manage their own resources, and use Network Policies for network-level segmentation between namespaces - achieving reasonable multi-tenancy without the overhead of fully separate clusters per team.

</details>

