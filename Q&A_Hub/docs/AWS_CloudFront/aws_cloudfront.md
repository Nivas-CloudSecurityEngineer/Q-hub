<div align="center" markdown="1">

# ⚡ AWS CloudFront
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_CloudFront-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is Amazon CloudFront?</b></summary>
<br>

A globally distributed Content Delivery Network (CDN) that caches content at edge locations close to users, reducing latency and origin load for web content, APIs, video streaming, and static assets.

</details>

<details markdown="1">
<summary>❓ <b>2. What is an Origin in CloudFront?</b></summary>
<br>

The source location CloudFront pulls content from when it's not already cached at an edge location - can be an S3 bucket, an ALB/EC2/on-prem server, API Gateway, or any custom HTTP(S) origin.

</details>

<details markdown="1">
<summary>❓ <b>3. What is an Edge Location vs a Regional Edge Cache?</b></summary>
<br>

Edge Locations are the many globally distributed CloudFront POPs that serve cached content directly to end users with lowest latency. Regional Edge Caches are a smaller number of larger caches positioned between edge locations and the origin, holding a larger cache to reduce the need to go back to the origin for less-popular content.

</details>

<details markdown="1">
<summary>❓ <b>4. What is a Distribution and its two types?</b></summary>
<br>

A Distribution is the CloudFront configuration tying origins, behaviors, and caching rules together to serve content. Web Distribution: for websites/APIs/general HTTP(S) content. RTMP Distribution: (deprecated) was for media streaming - modern use cases now use standard distributions with HLS/DASH.

</details>

## ⚡ Caching

<details markdown="1">
<summary>❓ <b>5. How does CloudFront determine what to cache and for how long?</b></summary>
<br>

Based on Cache Behaviors (path-pattern rules) and Cache Policies specifying TTLs (min/max/default), and which request elements (headers, query strings, cookies) are part of the cache key; origin `Cache-Control`/`Expires` headers also influence TTL if within policy bounds.

</details>

<details markdown="1">
<summary>❓ <b>6. What is a Cache Key and why does it matter?</b></summary>
<br>

The Cache Key determines what makes two requests "the same" for caching purposes (URL path + any included headers/cookies/query strings). Including unnecessary elements (like a session cookie) in the cache key can drastically reduce cache hit ratio by fragmenting the cache.

</details>

<details markdown="1">
<summary>🎯 <b>7. Scenario: Your cache hit ratio is very low despite mostly static content. What could be wrong?</b></summary>
<br>

Common causes: query strings or headers/cookies unnecessarily included in the cache key (e.g., analytics/tracking params), very short TTLs set at origin or in the cache policy, cache-busting query strings from the frontend, or too many unique URLs (e.g., per-user URLs) that are inherently uncacheable.

</details>

<details markdown="1">
<summary>❓ <b>8. How do you invalidate cached content in CloudFront?</b></summary>
<br>

Create an invalidation request specifying paths (`/images/*` or specific files) to force CloudFront to fetch fresh content from the origin on the next request - but this incurs cost/latency; better practice is versioned file names (cache-busting via filename/hash) to avoid needing invalidations.

</details>

<details markdown="1">
<summary>❓ <b>9. Difference between using Invalidation and Cache-Control headers/versioned URLs for content updates.</b></summary>
<br>

Invalidation actively purges cached content immediately (useful for emergencies) but costs money per path and takes a little time to propagate globally. Versioned URLs (e.g., `app.v2.js` or content-hashed filenames) avoid the problem entirely - new content gets a new URL, so there's never stale cached content to invalidate, which is the scalable best practice.

</details>

## 🛡️ Security

<details markdown="1">
<summary>❓ <b>10. What is Origin Access Control (OAC) and why is it important for S3 origins?</b></summary>
<br>

OAC allows CloudFront to authenticate requests to a private S3 bucket using SigV4, ensuring the S3 bucket can be kept fully private (no public access) while only CloudFront is authorized to fetch objects - the modern replacement for the older Origin Access Identity (OAI).

</details>

<details markdown="1">
<summary>❓ <b>11. How do you enforce HTTPS between users and CloudFront, and between CloudFront and the origin?</b></summary>
<br>

Set "Viewer Protocol Policy" to redirect-to-HTTPS or HTTPS-only for the user-facing side, and configure "Origin Protocol Policy" to HTTPS-only for CloudFront-to-origin communication, using ACM certificates for custom domains.

</details>

<details markdown="1">
<summary>❓ <b>12. What is AWS WAF's role with CloudFront?</b></summary>
<br>

WAF can be attached to a CloudFront distribution to inspect and filter requests at the edge (before reaching the origin), blocking SQL injection, XSS, bad bots, rate-limiting abusive IPs, and geo-blocking - protecting the origin from malicious/excessive traffic globally.

</details>

<details markdown="1">
<summary>🎯 <b>13. Scenario: You need to restrict content so only paid subscribers can access certain video files. How?</b></summary>
<br>

Use CloudFront Signed URLs or Signed Cookies - generate time-limited, cryptographically signed URLs/cookies server-side after verifying the user's subscription, so only holders of a valid signed URL/cookie can retrieve the protected content from CloudFront.

</details>

<details markdown="1">
<summary>❓ <b>14. Difference between Signed URLs and Signed Cookies.</b></summary>
<br>

Signed URLs grant access to a single specific file/path - good for individual file downloads (e.g., one PDF). Signed Cookies grant access to multiple restricted files (e.g., an entire video streaming session with many segments) without needing to sign every individual URL.

</details>

<details markdown="1">
<summary>❓ <b>15. What is Field-Level Encryption in CloudFront?</b></summary>
<br>

Allows encrypting specific sensitive form fields (e.g., credit card numbers) at the edge using a public key, so the data remains encrypted all the way to the application layer that holds the private key - adding an additional layer of protection beyond standard HTTPS.

</details>

<details markdown="1">
<summary>❓ <b>16. How does CloudFront support geo-restriction?</b></summary>
<br>

You can configure allow-lists or block-lists of countries at the distribution level, and CloudFront will return an error response for requests originating from restricted countries, using IP-geolocation databases.

</details>

## 🚀 Performance & Advanced Features

<details markdown="1">
<summary>❓ <b>17. What are Lambda@Edge and CloudFront Functions? Difference between them?</b></summary>
<br>

Both let you run code at CloudFront edge locations to customize requests/responses. CloudFront Functions are lightweight (JavaScript, microsecond execution, cheaper) for simple tasks (header manipulation, URL rewrites, redirects) at the viewer request/response stage. Lambda@Edge supports full Lambda (Node.js/Python), longer execution time, more memory, and can run at all 4 event points (viewer request/response, origin request/response) - used for more complex logic like A/B testing, auth, or dynamic origin selection.

</details>

<details markdown="1">
<summary>🎯 <b>18. Scenario: You need to add a security header to every response served by CloudFront with minimal latency overhead. Which do you use?</b></summary>
<br>

CloudFront Functions - they're designed for exactly this kind of lightweight, high-volume, low-latency manipulation (like adding `Strict-Transport-Security` headers) more cost-effectively than Lambda@Edge.

</details>

<details markdown="1">
<summary>❓ <b>19. What is Origin Failover in CloudFront?</b></summary>
<br>

Configuring an Origin Group with a primary and secondary origin - if the primary returns specific error status codes (e.g., 5xx), CloudFront automatically retries the request against the secondary origin, improving availability.

</details>

<details markdown="1">
<summary>❓ <b>20. How does CloudFront support dynamic content (not just static files)?</b></summary>
<br>

CloudFront can proxy dynamic requests to origins like ALB/API Gateway with caching disabled or very short TTLs for those paths (via separate cache behaviors), still benefiting from CloudFront's global network for TLS termination/connection reuse/DDoS protection even when caching isn't applicable.

</details>

<details markdown="1">
<summary>🎯 <b>21. Scenario: A global application serves both static assets and dynamic API calls. How do you configure a single CloudFront distribution?</b></summary>
<br>

Define multiple Cache Behaviors based on path patterns: e.g., `/static/*` and `/images/*` pointing to an S3 origin with long TTL caching, and `/api/*` pointing to an ALB/API Gateway origin with caching disabled (or a very short TTL) and all headers/cookies forwarded as needed for the dynamic backend.

</details>

## 🎯 Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>22. Scenario: Users report that after a deployment, they still see the old version of the website. How do you fix and prevent this?</b></summary>
<br>

Immediately create an invalidation for the affected paths (or `/*` if urgent) to clear stale cache. To prevent recurrence, adopt versioned/hashed static asset filenames referenced by an HTML file with a short TTL, so only the small HTML changes and assets are inherently cache-safe.

</details>

<details markdown="1">
<summary>❓ <b>23. How would you use CloudFront to protect and accelerate an API Gateway-based API?</b></summary>
<br>

Put CloudFront in front of API Gateway with caching disabled (or short TTL for cacheable GET endpoints), attach WAF for request filtering/rate-limiting, and benefit from CloudFront's edge network for reduced latency (TCP/TLS termination closer to users) and built-in DDoS protection (AWS Shield Standard).

</details>

<details markdown="1">
<summary>❓ <b>24. What is AWS Shield and how does it relate to CloudFront?</b></summary>
<br>

Shield Standard is automatically included with CloudFront (and Route 53/ALB) providing protection against common network/transport layer DDoS attacks at no extra cost. Shield Advanced is a paid tier offering enhanced protection, 24/7 DRT support, and cost protection for scaling during an attack.

</details>

<details markdown="1">
<summary>🎯 <b>25. Scenario: You need to serve different content to mobile vs desktop users at the edge, without changing the origin application. How?</b></summary>
<br>

Use CloudFront's device detection headers (`CloudFront-Is-Mobile-Viewer`, etc.) forwarded to the origin, or use a CloudFront Function/Lambda@Edge to inspect the User-Agent and rewrite the request/response accordingly (e.g., redirecting to a mobile-specific path).

</details>

<details markdown="1">
<summary>❓ <b>26. How do you monitor CloudFront performance and troubleshoot cache issues?</b></summary>
<br>

Use CloudFront metrics in CloudWatch (requests, bytes downloaded/uploaded, error rates, cache hit ratio), enable Access Logs (delivered to S3) for detailed per-request analysis, and check the `X-Cache` response header (Hit/Miss/RefreshHit) to debug caching behavior for specific requests.

</details>

<details markdown="1">
<summary>🎯 <b>27. Scenario: An origin server is getting overwhelmed by traffic even though CloudFront is in front of it. Why might this happen?</b></summary>
<br>

Possible causes: cache hit ratio is low (dynamic/personalized content or cache key fragmentation), TTLs are too short, cache behaviors aren't properly matching the traffic patterns, or there's a traffic spike of uncacheable requests (e.g., POST/API calls) that always pass through to origin regardless of CDN.

</details>

<details markdown="1">
<summary>❓ <b>28. What is the significance of `Cache-Control: no-store` vs `max-age` headers when working with CloudFront?</b></summary>
<br>

`no-store` tells CloudFront (and browsers) never to cache the response at all (e.g., sensitive/personalized data). `max-age=N` tells CloudFront/browsers how long (in seconds) the response can be considered fresh and served from cache - directly influencing cache TTL behavior alongside the distribution's cache policy settings.

</details>

<details markdown="1">
<summary>🎯 <b>29. Scenario: You need multi-region active-active origins behind a single CloudFront distribution for resilience and lower latency. How?</b></summary>
<br>

Use an Origin Group with multiple regional origins (e.g., ALBs in us-east-1 and eu-west-1) with failover configured, or use Lambda@Edge/CloudFront Functions with custom logic to route to the nearest healthy regional origin based on viewer location/latency, combined with Route 53 health checks feeding origin health status.

</details>

<details markdown="1">
<summary>❓ <b>30. How does CloudFront pricing work and what drives cost?</b></summary>
<br>

Pricing is based on data transfer out (per GB, varies by edge location/region), number of HTTP/HTTPS requests, and additional costs for features like Lambda@Edge invocations, invalidations beyond the free tier, and field-level encryption - optimizing cache hit ratio directly reduces both data transfer and request costs to the origin (though CloudFront's own request/data-out costs to users remain).

</details>

