<div align="center" markdown="1">

# 📉 Grafana & Prometheus
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-Grafana_%26_Prometheus-blue?style=for-the-badge&logo=grafana&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Prometheus Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is Prometheus?</b></summary>
<br>

An open-source monitoring and alerting toolkit that collects and stores metrics as time-series data, using a pull-based model to scrape metrics from configured targets, with its own powerful query language (PromQL).

</details>

<details markdown="1">
<summary>❓ <b>2. What is the Pull vs Push model, and why does Prometheus default to pull?</b></summary>
<br>

Pull: the monitoring system actively requests (scrapes) metrics from targets at defined intervals. Push: targets actively send metrics to the monitoring system. Prometheus defaults to pull because it centralizes scrape configuration/control, makes it easy to tell if a target is down (failed scrape = clear signal), and avoids targets needing to know about the monitoring system's location - though it supports push via the Pushgateway for ephemeral/batch jobs that don't live long enough to be scraped.

</details>

<details markdown="1">
<summary>❓ <b>3. What is a Prometheus Exporter?</b></summary>
<br>

A component that exposes metrics in Prometheus's text format from a system/application that doesn't natively support it (e.g., `node_exporter` for host-level metrics, `mysqld_exporter` for MySQL) - Prometheus then scrapes the exporter's `/metrics` endpoint.

</details>

<details markdown="1">
<summary>❓ <b>4. What are the four core Prometheus metric types?</b></summary>
<br>

Counter (monotonically increasing value, e.g., total requests - only goes up or resets to 0), Gauge (a value that can go up or down, e.g., current memory usage), Histogram (samples observations into configurable buckets, e.g., request duration distribution, and provides `_sum`/`_count`), and Summary (similar to histogram but calculates configurable quantiles client-side rather than bucketed).

</details>

<details markdown="1">
<summary>❓ <b>5. Difference between Histogram and Summary in more depth.</b></summary>
<br>

Histogram buckets observations server-side, allowing you to aggregate/calculate quantiles across multiple instances after the fact using PromQL (`histogram_quantile`) - more flexible for aggregation. Summary calculates exact quantiles client-side per instance, which cannot be meaningfully aggregated across multiple instances (e.g., averaging p99s from 10 servers is not the same as the true p99 across all of them) - Histogram is generally preferred for anything you need to aggregate.

</details>

<details markdown="1">
<summary>❓ <b>6. What is PromQL and give an example query.</b></summary>
<br>

Prometheus Query Language - used to select and aggregate time-series data. Example: `rate(http_requests_total[5m])` calculates the per-second average rate of increase of the `http_requests_total` counter over the last 5 minutes, commonly used to convert a cumulative counter into a meaningful rate.

</details>

<details markdown="1">
<summary>🎯 <b>7. Scenario: You need to alert when the 95th percentile request latency exceeds 500ms over the last 5 minutes. How do you write this with PromQL, assuming a histogram metric?</b></summary>
<br>

`histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) > 0.5` - this calculates the estimated 95th percentile from the histogram buckets, aggregated across instances, and compares it to the 0.5 second threshold.

</details>

<details markdown="1">
<summary>❓ <b>8. What is the difference between `rate()` and `irate()` in PromQL?</b></summary>
<br>

`rate()` calculates the average per-second rate of increase over the entire specified time range, smoothing out short-term spikes - best for alerting/dashboards where stability matters. `irate()` calculates the rate using only the last two data points in the range, more sensitive to very recent/instantaneous changes - useful for volatile, fast-changing metrics but noisier for alerting.

</details>

## 🏗️ Prometheus Architecture & Service Discovery

<details markdown="1">
<summary>❓ <b>9. How does Prometheus discover targets to scrape in a dynamic environment (e.g., Kubernetes)?</b></summary>
<br>

Via Service Discovery mechanisms - static configs for fixed targets, or dynamic discovery (Kubernetes SD, EC2 SD, Consul SD, etc.) that automatically detects new/removed targets (e.g., new pods) and updates the scrape target list without manual reconfiguration.

</details>

<details markdown="1">
<summary>❓ <b>10. What is the Prometheus Pushgateway and when should you use it?</b></summary>
<br>

A component that allows short-lived/batch jobs (which may finish before Prometheus's next scrape interval) to push their metrics to an intermediary gateway, which Prometheus then scrapes as a regular target - used sparingly, only for jobs that can't be scraped directly (Prometheus maintainers explicitly discourage using it as a general push-based replacement for pull).

</details>

<details markdown="1">
<summary>❓ <b>11. What are Recording Rules and why use them?</b></summary>
<br>

Pre-computed, saved PromQL expressions evaluated at regular intervals and stored as new time series - used to speed up frequently-used or expensive queries (e.g., pre-aggregating a complex query used in many dashboards) and to keep dashboard/alert queries fast and simple.

</details>

<details markdown="1">
<summary>❓ <b>12. What are Alerting Rules and how do they relate to Alertmanager?</b></summary>
<br>

Alerting Rules define PromQL conditions that, when true for a specified duration, fire an alert. Prometheus sends firing alerts to Alertmanager, which handles deduplication, grouping, silencing, inhibition, and routing to notification channels (Slack, PagerDuty, email) - Prometheus itself doesn't send notifications directly.

</details>

<details markdown="1">
<summary>🎯 <b>13. Scenario: You're getting alert fatigue from Prometheus firing many related alerts for the same root cause (e.g., an entire node going down triggers alerts for every service on it). How do you fix this?</b></summary>
<br>

Configure Alertmanager's Inhibition rules (suppress lower-priority alerts when a related higher-priority alert, like "NodeDown," is already firing) and Grouping (bundle related alerts into a single notification) to reduce noise and highlight the actual root cause.

</details>

<details markdown="1">
<summary>❓ <b>14. What is the `for` clause in a Prometheus alerting rule?</b></summary>
<br>

Specifies how long a condition must remain true before the alert transitions from "pending" to "firing" - preventing alerts from firing on brief, transient spikes that resolve on their own, reducing false positives/noise.

</details>

## 📈 Storage & Scaling

<details markdown="1">
<summary>❓ <b>15. How does Prometheus store data, and what's a key limitation of its default storage?</b></summary>
<br>

It uses a local, custom time-series database (TSDB) on disk, optimized for fast writes/queries but NOT designed for indefinite long-term storage or high availability by default - a single Prometheus instance has limited retention and doesn't natively provide clustering/replication.

</details>

<details markdown="1">
<summary>🎯 <b>16. Scenario: You need metrics retained for 2 years for compliance/trend analysis, but local Prometheus storage isn't designed for that. How do you solve this?</b></summary>
<br>

Integrate Prometheus with a long-term remote storage solution (e.g., Thanos, Cortex, Mimir, or a managed service like Amazon Managed Service for Prometheus) using the `remote_write`/`remote_read` API, which handles long-term retention, downsampling, and global querying across multiple Prometheus instances.

</details>

<details markdown="1">
<summary>❓ <b>17. What is Thanos (or similar tools like Cortex/Mimir) used for?</b></summary>
<br>

Extends Prometheus with horizontal scalability, long-term storage (typically backed by object storage like S3), global query view across multiple Prometheus instances/clusters, and high availability - addressing Prometheus's single-node storage/HA limitations.

</details>

<details markdown="1">
<summary>🎯 <b>18. Scenario: You run Prometheus across multiple Kubernetes clusters in different regions and need a single pane of glass to query metrics from all of them. How?</b></summary>
<br>

Deploy Thanos (or a similar federated solution) with sidecars alongside each Prometheus instance, using the Thanos Querier to provide a unified query interface across all clusters' data, and Thanos Store Gateway backed by object storage for long-term historical data across regions.

</details>

## 📊 Grafana Fundamentals

<details markdown="1">
<summary>❓ <b>19. What is Grafana and how does it relate to Prometheus?</b></summary>
<br>

An open-source visualization and dashboarding platform that queries data sources (Prometheus, CloudWatch, Loki, Elasticsearch, SQL databases, etc.) and renders interactive dashboards/graphs/alerts - Grafana itself doesn't store metrics; it visualizes data from external sources.

</details>

<details markdown="1">
<summary>❓ <b>20. What is a Data Source in Grafana?</b></summary>
<br>

A configured connection to a backend system (Prometheus, InfluxDB, CloudWatch, etc.) that Grafana queries to retrieve data for dashboards - each panel in a dashboard is tied to a specific data source and query.

</details>

<details markdown="1">
<summary>❓ <b>21. What is a Grafana Dashboard, Panel, and Variable?</b></summary>
<br>

A Dashboard is a collection of Panels (individual visualizations - graphs, tables, gauges). Variables are dashboard-level parameters (e.g., `$environment`, `$instance`) that let users dynamically filter/change what data a dashboard displays without editing the underlying queries directly.

</details>

<details markdown="1">
<summary>🎯 <b>22. Scenario: You need one dashboard template that can show metrics for any of 20 microservices, selected via a dropdown. How?</b></summary>
<br>

Create a Grafana Template Variable (e.g., `$service`, populated via a PromQL label query like `label_values(up, job)`), and reference `$service` within panel queries (e.g., `rate(http_requests_total{job="$service"}[5m])`) - a single dashboard then dynamically adapts based on the dropdown selection.

</details>

<details markdown="1">
<summary>❓ <b>23. What is Grafana Alerting and how does it differ from Prometheus Alertmanager?</b></summary>
<br>

Grafana has its own unified alerting engine that can evaluate queries from ANY configured data source (not just Prometheus) and route notifications - useful when you want a single alerting system across heterogeneous data sources (SQL, CloudWatch, Prometheus), whereas Prometheus Alertmanager is specifically tied to Prometheus-originated alerts.

</details>

## 🎯 Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>24. Scenario: You want to build a complete observability stack for a Kubernetes-based microservices application. What components do you use and how do they fit together?</b></summary>
<br>

Prometheus (via Prometheus Operator/kube-prometheus-stack) scrapes metrics from application/exporters/kube-state-metrics/node-exporter, Grafana visualizes dashboards from Prometheus (and possibly Loki for logs, Tempo/Jaeger for traces), Alertmanager handles alert routing/notification, and Thanos/Mimir provides long-term storage/multi-cluster aggregation if needed at scale.

</details>

<details markdown="1">
<summary>🎯 <b>25. Scenario: A dashboard shows a metric spiking, but you can't tell if it's a real issue or a monitoring artifact (e.g., a scrape gap). How do you differentiate?</b></summary>
<br>

Check the `up` metric for the target (indicates if scrapes are succeeding), look for gaps/anomalies in the time series consistent with scrape failures rather than genuine data, cross-reference with related metrics (e.g., if CPU spikes align with a real deployment event or traffic spike in logs/traces), and check Prometheus's own scrape duration/error metrics for that target.

</details>

<details markdown="1">
<summary>❓ <b>26. How would you monitor Prometheus itself to ensure it's healthy and not silently failing to scrape targets?</b></summary>
<br>

Monitor Prometheus's own exposed metrics (`up{job="..."}` for each target, `prometheus_target_scrape_pool_...` metrics, `prometheus_rule_evaluation_failures_total`), set alerts on scrape failures or high scrape duration, and consider a secondary, independent monitoring path (e.g., a separate lightweight uptime check) to detect if Prometheus itself goes down (avoiding the classic "who watches the watchmen" blind spot).

</details>

<details markdown="1">
<summary>🎯 <b>27. Scenario: Your Prometheus instance's memory usage keeps growing and eventually gets OOM-killed. What's a common cause and how do you fix it?</b></summary>
<br>

High cardinality metrics (e.g., using unbounded label values like user IDs, request IDs, or raw URLs as labels) cause an explosion in unique time series, consuming excessive memory - fix by removing/reducing high-cardinality labels, using recording rules to pre-aggregate, and setting `sample_limit`/scrape configuration guards, or scaling out with Thanos/Cortex/Mimir for legitimately large cardinality needs.

</details>

<details markdown="1">
<summary>❓ <b>28. What is the RED method and the USE method, and how do they relate to Grafana/Prometheus dashboard design?</b></summary>
<br>

RED (Rate, Errors, Duration) is used for monitoring request-driven services (e.g., APIs) - tracking request rate, error rate, and latency distribution. USE (Utilization, Saturation, Errors) is used for monitoring resources (CPU, disk, memory) - tracking how busy/utilized a resource is, whether it's saturated (queued/backed up), and its error count. Both are common frameworks for structuring meaningful, actionable Grafana dashboards rather than ad-hoc metric displays.

</details>

<details markdown="1">
<summary>🎯 <b>29. Scenario: You need to alert your on-call team via Slack immediately when a critical Prometheus alert fires, but only during business-critical services, with different severities routed differently. How?</b></summary>
<br>

Configure Alertmanager's routing tree with label-based matching (e.g., `severity: critical` routes to PagerDuty, `severity: warning` routes to a Slack channel), use `receivers` for each notification channel, and use `group_by`/`group_wait`/`repeat_interval` to control notification timing/frequency and prevent alert spam.

</details>

<details markdown="1">
<summary>❓ <b>30. How do you avoid vendor lock-in or single points of failure when designing a Prometheus + Grafana monitoring architecture at scale?</b></summary>
<br>

Use Prometheus's open remote-write standard to feed data into scalable, HA-capable long-term storage backends (Thanos/Mimir/Cortex, or cloud-managed Prometheus-compatible services), run Prometheus in HA pairs (two identical instances scraping the same targets) for redundancy, and keep dashboards/alerting rules as code (version-controlled JSON/YAML) for portability and disaster recovery.

</details>

