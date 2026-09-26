<div align="center" markdown="1">

# 🗃️ AWS DynamoDB
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_DynamoDB-blue?style=for-the-badge&logo=amazondynamodb&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is Amazon DynamoDB?</b></summary>
<br>

A fully managed, serverless NoSQL key-value and document database offering single-digit millisecond performance at any scale, with built-in high availability, automatic scaling, and multi-region replication options.

</details>

<details markdown="1">
<summary>❓ <b>2. What are the core components of a DynamoDB table?</b></summary>
<br>

Tables (collection of items), Items (rows, similar to a JSON document), Attributes (columns/fields within an item), and a Primary Key (uniquely identifies each item - either a simple Partition Key or a composite Partition Key + Sort Key).

</details>

<details markdown="1">
<summary>❓ <b>3. Difference between Partition Key and Sort Key.</b></summary>
<br>

Partition Key (hash key) determines which physical partition an item is stored on (via a hash function) - it must be unique if used alone. Sort Key (range key) allows multiple items to share the same partition key, sorted/ordered by the sort key value, enabling range queries (e.g., all orders for a customer, sorted by date).

</details>

<details markdown="1">
<summary>❓ <b>4. Is DynamoDB schema-less? What does this mean in practice?</b></summary>
<br>

Yes - other than the primary key attributes (which must be defined at table creation), items in the same table can have completely different sets of attributes, offering flexibility to evolve your data model without migrations, unlike a rigid relational schema.

</details>

## ⚙️ Capacity Modes & Performance

<details markdown="1">
<summary>❓ <b>5. Difference between On-Demand and Provisioned capacity modes.</b></summary>
<br>

On-Demand: pay-per-request, automatically scales to handle traffic with no capacity planning - ideal for unpredictable/spiky workloads. Provisioned: you specify Read/Write Capacity Units (RCU/WCU) ahead of time (optionally with Auto Scaling to adjust within bounds) - cheaper for predictable, steady workloads.

</details>

<details markdown="1">
<summary>❓ <b>6. What is a Read Capacity Unit (RCU) and Write Capacity Unit (WCU)?</b></summary>
<br>

1 RCU = one strongly consistent read (or two eventually consistent reads) per second for an item up to 4KB. 1 WCU = one write per second for an item up to 1KB. Larger items consume proportionally more units.

</details>

<details markdown="1">
<summary>❓ <b>7. Difference between Eventually Consistent and Strongly Consistent reads.</b></summary>
<br>

Eventually consistent reads (default) may return slightly stale data (replication lag across the 3 AZ copies, usually milliseconds) but consume half the RCU and have lower latency. Strongly consistent reads always return the most up-to-date data but consume more RCU, have slightly higher latency, and aren't supported on Global Secondary Indexes.

</details>

<details markdown="1">
<summary>🎯 <b>8. Scenario: You have a highly unpredictable, spiky workload (viral social media app) and don't want to manage capacity. Which mode?</b></summary>
<br>

On-Demand capacity mode - it scales instantly to handle traffic bursts without throttling risk (subject to a 2x-in-30-minutes soft scaling pattern awareness) or manual capacity planning, trading off slightly higher per-request cost for operational simplicity.

</details>

## 🔎 Indexes

<details markdown="1">
<summary>❓ <b>9. What is a Global Secondary Index (GSI)?</b></summary>
<br>

An index with a partition key (and optional sort key) different from the base table's, allowing queries on non-primary-key attributes; it has its own provisioned/on-demand capacity and is eventually consistent only, stored as a separate structure that's asynchronously updated.

</details>

<details markdown="1">
<summary>❓ <b>10. What is a Local Secondary Index (LSI)?</b></summary>
<br>

An index that shares the same partition key as the base table but a different sort key, allowing alternate sort orders/range queries within the same partition; it must be created at table creation time (cannot be added later) and shares the table's provisioned throughput, supporting strongly consistent reads.

</details>

<details markdown="1">
<summary>🎯 <b>11. Scenario: You need to query orders by customer (existing partition key) but ALSO need to query all orders across all customers by status. How?</b></summary>
<br>

Add a Global Secondary Index with `status` as the partition key (and perhaps `order_date` as sort key), since this is a fundamentally different access pattern than the base table's customer-based partition key - GSIs enable these alternate query patterns.

</details>

<details markdown="1">
<summary>❓ <b>12. Why can't you add an LSI after table creation, but you can add a GSI anytime?</b></summary>
<br>

LSIs share the base table's partition structure and storage internally, so they must be defined upfront (baked into the table's underlying architecture at creation). GSIs are essentially separate, independently-provisioned tables/indexes maintained via asynchronous replication, so they can be added, removed, or modified at any time.

</details>

## 🧬 Data Modeling

<details markdown="1">
<summary>❓ <b>13. Why is DynamoDB data modeling described as "design for your access patterns first"?</b></summary>
<br>

Unlike relational databases where you normalize data and query flexibly with JOINs, DynamoDB has no JOINs and limited query flexibility (only by primary key/index) - so you must know your application's exact query patterns upfront and design the table/keys/indexes specifically to satisfy them efficiently, often denormalizing data.

</details>

<details markdown="1">
<summary>❓ <b>14. What is a Single-Table Design and why is it used in DynamoDB?</b></summary>
<br>

A modeling pattern where multiple different entity types (e.g., Users, Orders, Products) are stored in one table, using generic partition/sort key naming (e.g., `PK`/`SK` with prefixes like `USER#123`/`ORDER#456`) so that related data can be fetched together in a single Query operation - reduces the number of round trips versus multiple tables, optimizing for DynamoDB's strengths (though it trades off some readability/flexibility).

</details>

<details markdown="1">
<summary>🎯 <b>15. Scenario: You need to model a one-to-many relationship (a customer has many orders) for efficient retrieval of a customer with all their orders in one query. How?</b></summary>
<br>

Use a composite key design: Partition Key = `CUSTOMER#<id>`, Sort Key = `PROFILE` for the customer's own record and `ORDER#<order_id>` for each order - a single Query on the partition key retrieves the customer profile and all their orders together, sorted naturally by the sort key.

</details>

<details markdown="1">
<summary>❓ <b>16. What is item collection and why does its size matter?</b></summary>
<br>

An item collection is the group of items sharing the same partition key (including any LSI entries). Since a partition key's data (and its LSI data) must fit reasonably within a partition, very large item collections (e.g., a "hot" customer with millions of orders under one partition key) can lead to performance issues or hit the 10GB LSI item-collection size limit.

</details>

## 🔄 Streams & Transactions

<details markdown="1">
<summary>❓ <b>17. What is DynamoDB Streams?</b></summary>
<br>

A feature that captures a time-ordered sequence of item-level changes (insert/update/delete) in a table, retained for 24 hours, which can be consumed by Lambda (for triggers/event-driven processing) or Kinesis Data Streams (for higher throughput/fan-out use cases).

</details>

<details markdown="1">
<summary>🎯 <b>18. Scenario: You need to replicate changes in a DynamoDB table to update a search index (e.g., OpenSearch) in near real-time. How?</b></summary>
<br>

Enable DynamoDB Streams on the table, configure a Lambda function as a trigger on the stream, and have the Lambda transform and push each change event to the OpenSearch index - a common CQRS/event-driven synchronization pattern.

</details>

<details markdown="1">
<summary>❓ <b>19. What are DynamoDB Transactions and when are they needed?</b></summary>
<br>

`TransactWriteItems`/`TransactGetItems` allow atomic, all-or-nothing operations across multiple items/tables (ACID guarantees) - needed when business logic requires multiple related changes to succeed or fail together (e.g., debit one account and credit another atomically).

</details>

<details markdown="1">
<summary>❓ <b>20. What is Conditional Write in DynamoDB and a practical use case?</b></summary>
<br>

A write operation (Put/Update/Delete) that only succeeds if a specified condition on the item is true (e.g., `attribute_not_exists(id)` to prevent overwriting, or checking a version number) - used to implement optimistic locking and prevent race conditions in concurrent updates.

</details>

## 🚀 Scaling, Performance & Troubleshooting

<details markdown="1">
<summary>❓ <b>21. What is a "Hot Partition" and how do you avoid it?</b></summary>
<br>

When a disproportionate amount of read/write traffic targets a single partition key value (e.g., a popular product ID, or a poorly chosen key like a fixed "date" for all of today's writes), it can throttle even if the table's overall provisioned capacity is sufficient. Avoid by choosing high-cardinality partition keys with evenly distributed access patterns, or adding a random/calculated suffix ("sharding") to spread load.

</details>

<details markdown="1">
<summary>🎯 <b>22. Scenario: Your DynamoDB table is experiencing throttling (`ProvisionedThroughputExceededException`) even though overall utilization metrics look fine. What's likely happening?</b></summary>
<br>

A hot partition - overall table-level metrics can look healthy while a single partition is being overwhelmed, since DynamoDB divides provisioned capacity evenly across partitions internally. Diagnose using CloudWatch's partition-level metrics or by reviewing the access pattern/key distribution, and fix via better key design or switching to On-Demand mode.

</details>

<details markdown="1">
<summary>❓ <b>23. What is DynamoDB Accelerator (DAX)?</b></summary>
<br>

A fully managed, in-memory caching layer for DynamoDB that can reduce read latency from milliseconds to microseconds for read-heavy workloads, transparently handling cache population/invalidation without requiring application-level cache logic (though it introduces eventual consistency for cached reads).

</details>

<details markdown="1">
<summary>🎯 <b>24. Scenario: A read-heavy application needs microsecond latency for frequently accessed items. How do you achieve this with DynamoDB?</b></summary>
<br>

Add a DAX cluster in front of the table - the application uses the DAX SDK client (a near drop-in replacement for the DynamoDB client) which caches item and query results, dramatically reducing read latency and offloading repeated reads from the base table.

</details>

<details markdown="1">
<summary>❓ <b>25. How do you back up and restore DynamoDB tables?</b></summary>
<br>

On-Demand Backups (manual, full table snapshot, no performance impact, retained until deleted) and Point-in-Time Recovery (PITR - continuous backups allowing restore to any point within the last 35 days), both restoring to a NEW table (not in-place) to avoid overwriting current data.

</details>

## 🎯 Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>26. Scenario: You need global, low-latency access to the same DynamoDB table from multiple regions with active-active writes. How?</b></summary>
<br>

Use DynamoDB Global Tables - a fully managed multi-region, multi-active replication feature that automatically propagates writes made in any region to all other replica regions (using last-writer-wins conflict resolution), providing local low-latency reads/writes in each region.

</details>

<details markdown="1">
<summary>🎯 <b>27. Scenario: Multiple concurrent users might update the same item at the same time, and you need to prevent lost updates. How?</b></summary>
<br>

Implement optimistic locking using a version number attribute and a Conditional Write (`ConditionExpression` checking the version matches the expected value before updating, then incrementing it) - if the condition fails, the app retries by re-reading and reapplying the update.

</details>

<details markdown="1">
<summary>❓ <b>28. How would you design a DynamoDB table for a high-traffic e-commerce order system, considering cost and performance?</b></summary>
<br>

Use single-table design for related entities (customer/order/order-items) to minimize round trips, choose partition keys with high cardinality to avoid hot partitions (e.g., customer ID, not order status), use On-Demand mode initially (or Provisioned + Auto Scaling once traffic is predictable), add GSIs for alternate access patterns (e.g., orders by status/date for fulfillment dashboards), and enable PITR/backups plus DAX if read latency is critical.

</details>

<details markdown="1">
<summary>🎯 <b>29. Scenario: You need to enforce that a specific attribute (e.g., email) is unique across all items in a table, but DynamoDB doesn't support unique constraints on non-key attributes. How?</b></summary>
<br>

Use the attribute itself (or a hash of it) as the partition key of a separate "uniqueness" table/item, and use a Conditional Write (`attribute_not_exists(pk)`) when creating the main record - wrapped in a Transaction (`TransactWriteItems`) with the main item creation, ensuring both succeed or fail together atomically.

</details>

<details markdown="1">
<summary>❓ <b>30. What are the cost drivers for DynamoDB and how do you optimize them?</b></summary>
<br>

Read/Write capacity (provisioned vs on-demand pricing), storage (per GB), GSI additional read/write capacity (each GSI consumes its own WCUs on every base table write), DAX cluster costs, Streams read requests, and data transfer. Optimize by right-sizing capacity mode to the traffic pattern, minimizing unnecessary GSIs, using TTL to automatically expire/delete stale items (reducing storage cost), and using efficient item sizes (avoid storing large blobs directly - use S3 with a DynamoDB reference instead).

</details>

<details markdown="1">
<summary>❓ <b>31. What is DynamoDB TTL (Time to Live) and a common use case?</b></summary>
<br>

A feature that automatically deletes items after a specified Unix timestamp attribute passes, at no additional write capacity cost (though there's a small delay in actual deletion) - commonly used for session data, temporary tokens, or expiring cache-like records without manual cleanup jobs.

</details>

