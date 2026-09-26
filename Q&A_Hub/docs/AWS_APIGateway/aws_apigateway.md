<div align="center" markdown="1">

# 🔌 AWS API Gateway
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_API_Gateway-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is Amazon API Gateway?</b></summary>
<br>

A fully managed service for creating, publishing, securing, and monitoring APIs at any scale, acting as the "front door" for backend services (Lambda, EC2, HTTP endpoints, other AWS services).

</details>

<details markdown="1">
<summary>❓ <b>2. What are the types of APIs supported by API Gateway?</b></summary>
<br>

REST API (full-featured, request/response transformation, API keys, usage plans), HTTP API (lightweight, lower cost, lower latency, fewer features - good for simple Lambda/HTTP proxy use cases), and WebSocket API (for real-time, bidirectional communication).

</details>

<details markdown="1">
<summary>❓ <b>3. Difference between REST API and HTTP API - when to choose which?</b></summary>
<br>

HTTP API is cheaper (up to 70% less) and faster (lower latency) but supports a reduced feature set (no request/response transformation, no usage plans/API keys natively, limited authorizers). REST API supports the full feature set (request validation, transformation, private APIs via resource policy, usage plans/API keys, caching, WAF integration). Choose HTTP API for simple proxy integrations to Lambda/HTTP backends; choose REST API when you need advanced features like caching, API keys/usage plans, or fine-grained request transformation.

</details>

<details markdown="1">
<summary>❓ <b>4. What are the API Gateway endpoint types?</b></summary>
<br>

Edge-Optimized (routed through CloudFront edge locations, best for geographically distributed clients), Regional (deployed in a specific region, best when clients are in the same region or you want to front it with your own CloudFront distribution), and Private (accessible only within a VPC via an Interface VPC Endpoint, not exposed to the public internet).

</details>

## 🔗 Integrations

<details markdown="1">
<summary>❓ <b>5. What integration types does API Gateway support?</b></summary>
<br>

Lambda Proxy Integration (passes the entire request to Lambda, Lambda returns a specifically formatted response), Lambda Custom Integration (manual mapping templates control request/response transformation), HTTP/HTTP Proxy Integration (forwards to an HTTP backend), Mock Integration (returns a response without hitting a backend, useful for testing/CORS preflight), and AWS Service Integration (directly calls other AWS services like SQS/Step Functions without a Lambda in between).

</details>

<details markdown="1">
<summary>❓ <b>6. Difference between Lambda Proxy and Lambda Custom (non-proxy) integration.</b></summary>
<br>

Proxy integration passes the full raw request (headers, query params, body) to Lambda as an event object, and Lambda must return a response in a specific format (`statusCode`, `headers`, `body`) - simpler, more common. Custom integration uses Velocity Template Language (VTL) mapping templates to transform the request/response between API Gateway and Lambda, giving more control but adding complexity.

</details>

<details markdown="1">
<summary>🎯 <b>7. Scenario: You want API Gateway to call SQS directly (to enqueue a message) without going through a Lambda function. Is this possible?</b></summary>
<br>

Yes - use an AWS Service Integration, configuring a mapping template that transforms the incoming request into the `SendMessage` SQS API format, and grant API Gateway's execution role permission to call SQS - eliminating the need for a Lambda just to relay to SQS, reducing cost and latency.

</details>

## 🛡️ Security

<details markdown="1">
<summary>❓ <b>8. What authorization/authentication mechanisms does API Gateway support?</b></summary>
<br>

IAM permissions (SigV4-signed requests), Lambda Authorizers (custom logic, e.g., validating a JWT/API key against custom logic), Amazon Cognito User Pools Authorizer (validates JWT tokens issued by Cognito), and native JWT Authorizers (for HTTP APIs, validating OIDC/OAuth2 tokens directly without a Lambda).

</details>

<details markdown="1">
<summary>❓ <b>9. What is a Lambda Authorizer and its two types?</b></summary>
<br>

A Lambda function invoked by API Gateway to determine if a request should be allowed, returning an IAM policy document. Token-based authorizer: receives a bearer token (e.g., `Authorization` header) and validates it. Request-based authorizer: receives full request context (headers, query params, source IP) for more complex authorization logic.

</details>

<details markdown="1">
<summary>🎯 <b>10. Scenario: You need to validate a JWT issued by your company's own OIDC identity provider (not Cognito) for API access. How?</b></summary>
<br>

For HTTP APIs, configure a native JWT Authorizer pointing to the issuer's JWKS endpoint (no custom code needed). For REST APIs, implement a Lambda Authorizer that validates the JWT signature/claims against the identity provider's public keys.

</details>

<details markdown="1">
<summary>❓ <b>11. How do you secure an API Gateway endpoint to only be callable by your own AWS account's services (e.g., internal microservices)?</b></summary>
<br>

Use IAM authorization on the API Gateway method, requiring the caller to sign requests with SigV4 using credentials from an IAM role/user with `execute-api:Invoke` permission on that specific API/resource - combined with a resource policy on the API Gateway restricting to specific principals/VPCs if needed.

</details>

<details markdown="1">
<summary>❓ <b>12. What is a Resource Policy on API Gateway used for?</b></summary>
<br>

Similar to S3 bucket policies - controls which principals/VPCs/IP ranges can invoke the API, used commonly for Private APIs (restricting access to specific VPCs/VPC Endpoints) or restricting public APIs to specific source IPs/AWS accounts.

</details>

## 🚦 Throttling & Usage Plans

<details markdown="1">
<summary>❓ <b>13. What is throttling in API Gateway and how is it configured?</b></summary>
<br>

API Gateway enforces rate limiting (requests per second) and burst limits (token bucket algorithm) at the account level (default) or per-API/stage/method/API-key level, protecting backends from being overwhelmed and returning `429 Too Many Requests` when exceeded.

</details>

<details markdown="1">
<summary>❓ <b>14. What are Usage Plans and API Keys used for?</b></summary>
<br>

Usage Plans define throttling and quota limits (e.g., 1000 requests/day) for specific API consumers, associated with API Keys that identify individual clients/customers - commonly used for monetized/partner APIs where you need to track and limit usage per customer.

</details>

<details markdown="1">
<summary>🎯 <b>15. Scenario: You're exposing an API to external partners and need to give each partner a different rate limit and track their usage for billing. How?</b></summary>
<br>

Issue each partner a unique API Key, create Usage Plans with appropriate throttle/quota settings per tier (e.g., Basic/Premium), associate each partner's key with the relevant usage plan, and use CloudWatch/API Gateway usage data for billing/reporting.

</details>

## ⚡ Caching & Performance

<details markdown="1">
<summary>❓ <b>16. How does API Gateway caching work?</b></summary>
<br>

For REST APIs, you can enable a cache (per stage) that stores responses for a configurable TTL, keyed by request parameters you specify - reducing latency and backend load for frequently requested, cacheable (typically GET) endpoints.

</details>

<details markdown="1">
<summary>🎯 <b>17. Scenario: A specific endpoint returns data that changes only once a day, but is called thousands of times per hour. How do you optimize cost/performance?</b></summary>
<br>

Enable API Gateway caching for that endpoint with a TTL matching the data's actual change frequency (or invalidate the cache programmatically when the underlying data updates), drastically reducing calls to the backend (Lambda/database) and lowering both latency and cost.

</details>

## 🔍 Monitoring & Troubleshooting

<details markdown="1">
<summary>❓ <b>18. What metrics/logs does API Gateway provide for monitoring?</b></summary>
<br>

CloudWatch metrics (`Count`, `4XXError`, `5XXError`, `Latency`, `IntegrationLatency`), Execution Logs (detailed request/response tracing per stage, useful for debugging integration issues), Access Logs (customizable format, e.g., for centralized log analysis), and X-Ray tracing for distributed request tracing.

</details>

<details markdown="1">
<summary>🎯 <b>19. Scenario: Your API is returning `502 Bad Gateway` errors intermittently. How do you troubleshoot?</b></summary>
<br>

502 typically indicates a malformed response from the backend integration (e.g., Lambda returning an invalid response format for proxy integration, or a Lambda timing out/crashing) - check CloudWatch Execution Logs for the specific integration error, verify the Lambda's response matches the expected proxy format, and check Lambda's own error logs/timeout settings.

</details>

<details markdown="1">
<summary>❓ <b>20. Difference between `4XXError` at the API Gateway level and errors from the backend integration.</b></summary>
<br>

API Gateway-level 4xx errors (e.g., 403 from a failed authorizer, 429 from throttling, 400 from request validation failure) occur before reaching the backend. Backend-originated errors (e.g., a Lambda function explicitly returning a 400/500 status in its proxy response) pass through API Gateway but represent an application-level decision, not a gateway-level rejection - distinguishing these is key to correct troubleshooting.

</details>

## 🎯 Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>21. Scenario: You need to design a public REST API with request validation (rejecting malformed requests before invoking Lambda) to save invocation costs. How?</b></summary>
<br>

Define a JSON Schema request model and enable Request Validation on the method (validate body and/or query string parameters/headers) - API Gateway rejects invalid requests with a 400 error before the Lambda is ever invoked, saving compute cost and improving response time for bad requests.

</details>

<details markdown="1">
<summary>🎯 <b>22. Scenario: You're migrating a monolithic API to microservices, each with a different Lambda/backend, but want to expose a single unified API to consumers. How?</b></summary>
<br>

Use a single API Gateway with different resource paths (e.g., `/orders`, `/users`, `/payments`) each routed to its own backend integration (Lambda/HTTP), effectively acting as an API composition/facade layer, potentially combined with request/response transformation to maintain a consistent external contract.

</details>

<details markdown="1">
<summary>❓ <b>23. How do you implement Canary Deployments in API Gateway?</b></summary>
<br>

Enable canary release on a stage, specifying a percentage of traffic to route to the new deployment version while the rest continues to the current stable deployment, monitor error rates/latency for the canary, then promote it to 100% (replacing the stable deployment) or roll back if issues are detected.

</details>

<details markdown="1">
<summary>🎯 <b>24. Scenario: You need to expose an internal API only accessible from within your VPC (not the public internet) for security/compliance reasons. How?</b></summary>
<br>

Create a Private API Gateway (endpoint type "Private"), associate it with an Interface VPC Endpoint for `execute-api`, and configure a resource policy restricting invocation to that specific VPC Endpoint - the API is never reachable from the public internet.

</details>

<details markdown="1">
<summary>❓ <b>25. How does API Gateway handle CORS and what's a common gotcha?</b></summary>
<br>

API Gateway can be configured to automatically handle CORS (adding required headers, and creating a Mock integration for OPTIONS preflight requests). A common gotcha is enabling CORS at the API Gateway level but forgetting to also include the CORS headers (e.g., `Access-Control-Allow-Origin`) in the actual Lambda proxy integration's response - since Lambda proxy responses bypass the gateway's automatic header injection for actual (non-OPTIONS) requests.

</details>

<details markdown="1">
<summary>🎯 <b>26. Scenario: You need real-time bidirectional communication (e.g., a chat application) via API Gateway. How?</b></summary>
<br>

Use a WebSocket API - define routes (e.g., `$connect`, `$disconnect`, `$default`, custom routes) each mapped to a backend (typically Lambda) that can push messages back to connected clients using the API Gateway Management API (`PostToConnection`), maintaining connection IDs (often in DynamoDB) to track active sessions.

</details>

<details markdown="1">
<summary>❓ <b>27. What is the relationship between API Gateway stages and deployments?</b></summary>
<br>

A Deployment is a snapshot of the API's configuration at a point in time; a Stage (e.g., `dev`, `prod`) is a named reference pointing to a specific deployment, with its own stage variables, throttling, caching, and logging settings - allowing you to promote the same deployment through environments or roll back a stage to a previous deployment quickly.

</details>

<details markdown="1">
<summary>🎯 <b>28. Scenario: How would you version an API to avoid breaking existing clients while introducing new features?</b></summary>
<br>

Use a versioning strategy such as URI path versioning (`/v1/orders`, `/v2/orders`) or header-based versioning, deploy the new version as a separate resource/stage while keeping the old version fully functional, and use stage variables/Lambda aliases to route each API version to the corresponding backend Lambda version - deprecating the old version only after clients migrate.

</details>

<details markdown="1">
<summary>❓ <b>29. How do you protect an API Gateway-fronted application from DDoS/malicious traffic?</b></summary>
<br>

Attach AWS WAF (for REST/regional/edge APIs) to filter malicious requests and apply rate-based rules, rely on AWS Shield Standard (automatically included) for network-layer DDoS protection, configure appropriate throttling/usage plans, and use CloudFront (for edge-optimized APIs) to absorb and cache traffic at the edge.

</details>

<details markdown="1">
<summary>🎯 <b>30. Scenario: A backend Lambda function occasionally times out under load, causing API Gateway to return 504 errors. How do you address this holistically?</b></summary>
<br>

Increase Lambda's memory/timeout if genuinely needed, but also investigate root cause (cold starts, downstream DB connection exhaustion, inefficient code), implement Provisioned Concurrency if cold starts are the issue, add caching at API Gateway for repeatable requests, and consider asynchronous patterns (API Gateway -> SQS -> Lambda) for long-running operations rather than a synchronous request/response that risks the fixed 29-second API Gateway integration timeout.

</details>

