<div align="center" markdown="1">

# 📨 AWS SNS, SQS & SES
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_Messaging-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 📢 SNS (Simple Notification Service)

<details markdown="1">
<summary>❓ <b>1. What is Amazon SNS?</b></summary>
<br>

A fully managed pub/sub messaging service that allows publishers to send messages to a Topic, which then fans out (pushes) the message to multiple subscribers (email, SMS, Lambda, SQS, HTTP/S endpoints, mobile push) simultaneously.

</details>

<details markdown="1">
<summary>❓ <b>2. What is a Topic and what are the types of Topics?</b></summary>
<br>

A Topic is a communication channel/access point for message delivery. Standard Topics offer high throughput, best-effort ordering, and at-least-once delivery. FIFO Topics guarantee strict ordering and exactly-once delivery, but with lower throughput, and require a `MessageGroupId` for ordering.

</details>

<details markdown="1">
<summary>❓ <b>3. What is the Fan-Out pattern using SNS + SQS?</b></summary>
<br>

Publishing a single message to an SNS topic that has multiple SQS queues subscribed to it, so each queue (and its independent consumers) receives a copy of the message - enabling multiple decoupled services to process the same event independently without the publisher knowing about each consumer.

</details>

<details markdown="1">
<summary>🎯 <b>4. Scenario: An order placement event needs to trigger inventory update, email confirmation, and analytics logging, each processed independently and at its own pace. How do you architect this?</b></summary>
<br>

Publish the order event to a single SNS topic, subscribe three separate SQS queues (one per downstream service: inventory, email, analytics), each consumed independently - if one service is slow/down, the others aren't affected, and messages queue up safely in their respective SQS queue.

</details>

<details markdown="1">
<summary>❓ <b>5. What is a Subscription Filter Policy in SNS?</b></summary>
<br>

A JSON policy attached to a subscription that filters which messages a subscriber receives based on message attributes, so a single topic can serve multiple subscribers who each only care about a subset of messages (e.g., only "high-priority" orders go to a specific queue).

</details>

<details markdown="1">
<summary>❓ <b>6. How does SNS ensure message delivery reliability, and what happens if a subscriber endpoint is unavailable?</b></summary>
<br>

SNS retries delivery with a backoff policy for HTTP/S/Lambda endpoints; for persistent failures, you can configure a Dead Letter Queue (DLQ) on the subscription to capture undeliverable messages instead of losing them.

</details>

## 📬 SQS (Simple Queue Service)

<details markdown="1">
<summary>❓ <b>7. What is Amazon SQS?</b></summary>
<br>

A fully managed message queuing service that decouples producers and consumers, allowing asynchronous processing - producers send messages to a queue, and consumers poll and process them at their own pace.

</details>

<details markdown="1">
<summary>❓ <b>8. Difference between Standard Queue and FIFO Queue.</b></summary>
<br>

Standard: nearly unlimited throughput, at-least-once delivery (possible duplicates), best-effort ordering (messages may arrive out of order). FIFO: strict ordering (per message group), exactly-once processing (with deduplication), but limited throughput (up to 3000 msg/sec with batching).

</details>

<details markdown="1">
<summary>❓ <b>9. What is Visibility Timeout and why is it critical?</b></summary>
<br>

The period during which a message, once received by a consumer, is hidden from other consumers to prevent duplicate processing. If the consumer doesn't delete the message within this window (e.g., due to a crash or slow processing), it becomes visible again and may be redelivered - the timeout must be set longer than the expected processing time.

</details>

<details markdown="1">
<summary>🎯 <b>10. Scenario: Messages are being processed twice by different consumers. What's likely misconfigured?</b></summary>
<br>

The Visibility Timeout is probably too short relative to actual processing time, causing the message to reappear in the queue and be picked up by another consumer before the first one finishes and deletes it - increase the timeout, or use `ChangeMessageVisibility` to extend it dynamically for long-running tasks.

</details>

<details markdown="1">
<summary>❓ <b>11. What is a Dead Letter Queue (DLQ) in SQS?</b></summary>
<br>

A separate queue where messages are automatically sent after failing processing a configured number of times (`maxReceiveCount`), preventing "poison pill" messages from blocking/endlessly retrying in the main queue, and allowing separate investigation/reprocessing.

</details>

<details markdown="1">
<summary>❓ <b>12. What is Long Polling vs Short Polling in SQS?</b></summary>
<br>

Short Polling returns immediately, even if no messages are available (can return empty responses, more API calls, higher cost). Long Polling waits (up to 20 seconds) for a message to arrive before returning, reducing empty responses, lowering cost, and reducing latency in message retrieval - almost always the recommended setting.

</details>

<details markdown="1">
<summary>❓ <b>13. What are the differences between SQS Standard and FIFO regarding deduplication?</b></summary>
<br>

FIFO queues support Deduplication (content-based, using a hash of the message body, or an explicit `MessageDeduplicationId`) within a 5-minute deduplication interval, preventing duplicate message delivery if the same message is sent multiple times - Standard queues have no deduplication and may deliver duplicates due to their at-least-once model.

</details>

<details markdown="1">
<summary>🎯 <b>14. Scenario: You need to process financial transactions where order and exactly-once processing are critical. Which queue type?</b></summary>
<br>

FIFO Queue - guarantees strict ordering (using MessageGroupId to control ordering scope) and exactly-once processing (via deduplication), essential for financial transaction integrity where duplicate or out-of-order processing could cause incorrect balances.

</details>

<details markdown="1">
<summary>❓ <b>15. How does SQS integrate with Auto Scaling to build an elastic worker fleet?</b></summary>
<br>

Use the queue's `ApproximateNumberOfMessagesVisible` CloudWatch metric as the input to a Target Tracking or Step Scaling policy on the worker fleet's ASG, automatically scaling worker capacity up/down based on queue backlog.

</details>

<details markdown="1">
<summary>❓ <b>16. What is SQS Extended Client Library used for?</b></summary>
<br>

Since SQS has a max message size of 256KB, the Extended Client Library allows sending larger payloads by storing the actual message body in S3 and passing only a reference/pointer in the SQS message - transparently handled by the client library.

</details>

<details markdown="1">
<summary>🎯 <b>17. Scenario: A batch of messages must be processed with minimal cost and highest throughput, with occasional duplicates being acceptable. Which queue and settings?</b></summary>
<br>

Standard Queue with Long Polling and batch operations (`SendMessageBatch`/`ReceiveMessage` with `MaxNumberOfMessages` up to 10) to maximize throughput and minimize API call costs, since duplicates/ordering aren't a strict requirement here.

</details>

## 📧 SES (Simple Email Service)

<details markdown="1">
<summary>❓ <b>18. What is Amazon SES?</b></summary>
<br>

A cost-effective, scalable email service for sending and receiving transactional, marketing, and bulk email, providing high deliverability and detailed sending statistics.

</details>

<details markdown="1">
<summary>❓ <b>19. What is a Sandbox environment in SES and its limitations?</b></summary>
<br>

By default, new SES accounts start in the Sandbox, which only allows sending TO verified email addresses/domains, with a low daily sending quota - production access must be requested to send to arbitrary recipients at higher volume.

</details>

<details markdown="1">
<summary>❓ <b>20. How do you improve email deliverability and avoid being marked as spam with SES?</b></summary>
<br>

Set up SPF, DKIM, and DMARC records for your sending domain (SES can generate these), maintain a good sender reputation (low bounce/complaint rates), warm up new IPs/domains gradually, and use dedicated IPs for high-volume senders needing consistent reputation control.

</details>

<details markdown="1">
<summary>🎯 <b>21. Scenario: You need to track and automatically handle email bounces and complaints to keep your sender reputation healthy. How?</b></summary>
<br>

Configure SES Event Publishing (via SNS) for Bounce and Complaint notifications, subscribe a Lambda function to automatically suppress/unsubscribe those addresses from future sends (updating your application's user database), preventing repeated bounces/complaints from damaging your domain's reputation.

</details>

<details markdown="1">
<summary>❓ <b>22. What is SES Configuration Set used for?</b></summary>
<br>

A set of rules that applies to emails sent through it, enabling event tracking (sends, deliveries, opens, clicks, bounces, complaints) published to destinations like SNS, CloudWatch, or Kinesis Firehose, and allowing IP pool assignment for dedicated sending reputation management.

</details>

<details markdown="1">
<summary>🎯 <b>23. Scenario: Your application needs to send order confirmation emails and marketing newsletters separately, with independent reputation tracking. How?</b></summary>
<br>

Use separate Configuration Sets (and potentially separate sending domains/subdomains) for transactional vs marketing email, so a spike in marketing complaints/bounces doesn't affect the deliverability reputation of critical transactional emails.

</details>

## 🎯 Real-Time Cross-Service Scenarios

<details markdown="1">
<summary>🎯 <b>24. Scenario: Design a resilient order-processing pipeline using SQS, SNS, and Lambda that can handle backend outages gracefully.</b></summary>
<br>

Producer publishes order events to an SNS topic; SNS fans out to SQS queues per downstream consumer (inventory, billing, notifications); each Lambda consumer processes its queue with retries and a DLQ for failures; if a downstream dependency (e.g., billing API) is down, messages simply accumulate safely in that queue until the Lambda/service recovers, without blocking or losing other pipeline paths.

</details>

<details markdown="1">
<summary>🎯 <b>25. Scenario: You need to notify multiple teams (via email, Slack, and a ticketing system) whenever a critical alarm fires, using SNS. How?</b></summary>
<br>

Create an SNS topic that the CloudWatch alarm publishes to, with multiple subscriptions: an email subscription for one team, an HTTPS subscription pointing to a Slack webhook (or a Lambda that formats and posts to Slack), and another Lambda subscription that creates a ticket via the ticketing system's API - all triggered from the single publish event.

</details>

<details markdown="1">
<summary>❓ <b>26. What is the difference between SQS and SNS in one sentence, and how are they often used together?</b></summary>
<br>

SNS is push-based pub/sub (fan-out to many subscribers), while SQS is a pull-based queue (single logical consumer group, messages persist until processed); they're often combined so SNS fans out an event to multiple SQS queues, giving each downstream service its own durable, independently-consumed copy of the message.

</details>

<details markdown="1">
<summary>🎯 <b>27. Scenario: You need SES to handle high email volume (millions/day) reliably while controlling cost. What features help?</b></summary>
<br>

Use Configuration Sets with dedicated IP pools for reputation isolation, monitor sending statistics/reputation dashboard to catch issues early, implement suppression list management via bounce/complaint handling, and consider using SES's reputation-based sending recommendations to throttle automatically if metrics degrade, avoiding account-wide sending pauses.

</details>

<details markdown="1">
<summary>🎯 <b>28. Scenario: You need to guarantee that a message is processed exactly once even though your producer might accidentally send it multiple times (e.g., due to a retry on the client side). How do you prevent duplicate processing across SNS/SQS/Lambda?</b></summary>
<br>

Use a FIFO SNS topic with FIFO SQS subscription (deduplication ID based) for delivery-level dedup, and additionally implement idempotency at the consumer/Lambda level (e.g., checking a processed-message-ID table in DynamoDB with a conditional write) as a defense-in-depth measure, since end-to-end exactly-once semantics across an entire distributed system still benefit from application-level idempotency.

</details>

<details markdown="1">
<summary>❓ <b>29. How would you monitor and alert on a growing backlog in an SQS queue indicating consumers can't keep up?</b></summary>
<br>

Set a CloudWatch Alarm on `ApproximateNumberOfMessagesVisible` (and/or `ApproximateAgeOfOldestMessage`, which is often a better indicator of processing delay) exceeding a threshold, notify via SNS, and potentially trigger Auto Scaling for the consumer fleet automatically as part of the alerting/remediation flow.

</details>

<details markdown="1">
<summary>🎯 <b>30. Scenario: A partner integration requires you to guarantee message delivery even if their endpoint is down for hours. How do you design this with SQS/SNS?</b></summary>
<br>

Use an SQS queue (rather than direct HTTP push) as a buffer between your system and the partner integration Lambda/worker, configure a generous DLQ threshold and retry/backoff logic, and if messages fail persistently, they land in a DLQ for manual investigation/replay once the partner's endpoint recovers - ensuring no message loss during extended outages.

</details>

