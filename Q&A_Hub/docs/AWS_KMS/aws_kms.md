<div align="center" markdown="1">

# 🔑 AWS KMS
### 🎤 Interview Questions & Answers

![Category](https://img.shields.io/badge/Category-AWS_KMS-blue?style=for-the-badge&logo=amazonaws&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner_to_Advanced-yellow?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Real--Time%20Scenarios-success?style=for-the-badge)

</div>

---

> 💡 **Pro Tip:** Skim the 🧩 fundamentals first, then master the 🎯 real-time scenarios — that's where interviews are actually won or lost.

---

## 🧩 Fundamentals

<details markdown="1">
<summary>❓ <b>1. What is AWS KMS?</b></summary>
<br>

Key Management Service is a managed service for creating, storing, and controlling cryptographic keys used to encrypt/decrypt data across AWS services and your own applications, backed by FIPS 140-2 validated hardware security modules (HSMs).

</details>

<details markdown="1">
<summary>❓ <b>2. What are the types of keys in KMS?</b></summary>
<br>

AWS Owned Keys (used internally by AWS services, not visible/manageable by you, no cost), AWS Managed Keys (created/managed by AWS on your behalf for a specific service, e.g., `aws/s3`, visible in your account but rotation/policy is AWS-controlled), and Customer Managed Keys (CMKs - fully controlled by you: key policy, rotation, aliases, enable/disable).

</details>

<details markdown="1">
<summary>❓ <b>3. Difference between symmetric and asymmetric KMS keys.</b></summary>
<br>

Symmetric keys use the same key for encrypt/decrypt (AES-256), never leave KMS unencrypted, and are used for the vast majority of use cases (S3/EBS/RDS encryption). Asymmetric keys have a public/private key pair (RSA or ECC) - the private key stays in KMS, but the public key can be exported/shared for encryption or signature verification outside AWS, used for use cases like sharing an encryption key with external parties or digital signing.

</details>

<details markdown="1">
<summary>❓ <b>4. What is a Customer Managed Key (CMK) and why choose it over an AWS Managed Key?</b></summary>
<br>

A CMK gives you full control: custom key policies, ability to enable/disable or schedule deletion, control rotation, and detailed CloudTrail audit logging of every use - chosen when you need fine-grained access control, cross-account sharing, or compliance requiring explicit key ownership/audit.

</details>

## ✉️ Envelope Encryption

<details markdown="1">
<summary>❓ <b>5. What is Envelope Encryption and why does KMS use it?</b></summary>
<br>

A technique where data is encrypted with a Data Encryption Key (DEK), and the DEK itself is encrypted with a KMS master key (the CMK never leaves KMS). This avoids sending large amounts of data to KMS for encryption (KMS has payload size limits) and is far more efficient/performant for encrypting large objects.

</details>

<details markdown="1">
<summary>❓ <b>6. Explain the `GenerateDataKey` API flow.</b></summary>
<br>

The application calls `GenerateDataKey`, KMS returns a plaintext DEK and the same DEK encrypted under the CMK. The app uses the plaintext DEK to encrypt the actual data, then discards the plaintext DEK and stores only the encrypted DEK alongside the encrypted data. To decrypt, the app calls `Decrypt` on the encrypted DEK (via KMS) to get the plaintext DEK back, then decrypts the data locally.

</details>

<details markdown="1">
<summary>🎯 <b>7. Scenario: You need to encrypt a 10GB file. Why can't you just call KMS `Encrypt` directly on it?</b></summary>
<br>

KMS's direct `Encrypt`/`Decrypt` API has a payload limit (4KB for symmetric keys), so it's unsuitable for large data. Instead, use envelope encryption: generate a DEK via `GenerateDataKey`, encrypt the 10GB file locally/client-side with that DEK (e.g., using AES-GCM), and store the encrypted DEK alongside the encrypted file.

</details>

## 🔑 Key Policies & Access Control

<details markdown="1">
<summary>❓ <b>8. How does access control work for KMS keys?</b></summary>
<br>

Through Key Policies (resource-based policies attached directly to the key - the primary and required access control mechanism) combined optionally with IAM policies and Grants. Unlike other AWS resources, if the key policy doesn't allow an action, IAM policies alone cannot grant it (the key policy must explicitly delegate to IAM, e.g., via `enable IAM policies` default statement).

</details>

<details markdown="1">
<summary>❓ <b>9. What is a KMS Grant and when is it used?</b></summary>
<br>

A temporary, programmatic way to delegate specific permissions on a key to a principal (often used by AWS services internally, e.g., to allow an EC2/EBS process temporary access to a key for a specific operation), which can be created/retired without modifying the key policy directly - useful for fine-grained, short-lived, application-managed permissions.

</details>

<details markdown="1">
<summary>🎯 <b>10. Scenario: A user has full `kms:*` permissions in their IAM policy but still can't use a specific CMK. Why?</b></summary>
<br>

The key's resource-based Key Policy must also explicitly allow that principal (or delegate to IAM via the default "enable IAM user permissions" statement) - IAM policies alone are insufficient for KMS; both the key policy AND the IAM policy must permit the action (this is the "double door" access model unique to KMS).

</details>

<details markdown="1">
<summary>❓ <b>11. How do you enable cross-account use of a KMS key?</b></summary>
<br>

Update the CMK's key policy to grant the external account (or specific principal) permission to use the key (e.g., `kms:Decrypt`, `kms:GenerateDataKey`), and the external account must also grant its own principals IAM permissions to use that external key ARN.

</details>

## 🔁 Key Rotation & Lifecycle

<details markdown="1">
<summary>❓ <b>12. How does automatic key rotation work for KMS CMKs?</b></summary>
<br>

For symmetric CMKs, you can enable automatic annual rotation - KMS generates new cryptographic material but keeps the same Key ID/ARN, retaining old key material to decrypt data encrypted under previous versions transparently (no re-encryption of existing data needed).

</details>

<details markdown="1">
<summary>❓ <b>13. Can you rotate an AWS Managed Key or an asymmetric CMK?</b></summary>
<br>

AWS Managed Keys are automatically rotated by AWS (typically annually) - not user-configurable. Asymmetric CMKs and HMAC keys do not support automatic rotation - if rotation is required, you must create a new key and manually migrate/re-encrypt as needed.

</details>

<details markdown="1">
<summary>🎯 <b>14. Scenario: You need to permanently delete a KMS key. What's the process and risk?</b></summary>
<br>

Schedule deletion with a mandatory waiting period (7-30 days) during which it can still be canceled; once deleted, ANY data encrypted under that key becomes permanently unrecoverable - AWS strongly recommends disabling the key first and confirming no resources depend on it before scheduling deletion.

</details>

## 🔗 Integration with AWS Services

<details markdown="1">
<summary>❓ <b>15. How does S3 use KMS for encryption (SSE-KMS)?</b></summary>
<br>

When an object is uploaded with SSE-KMS, S3 calls KMS to generate a data key, encrypts the object with it, and stores the encrypted data key with the object metadata; on retrieval, S3 calls KMS to decrypt the data key (transparently to the user, provided they have both S3 and KMS permissions).

</details>

<details markdown="1">
<summary>🎯 <b>16. Scenario: A user has S3 `GetObject` permission but gets "Access Denied" retrieving an SSE-KMS encrypted object. Why?</b></summary>
<br>

They likely lack `kms:Decrypt` permission on the specific CMK used to encrypt that object (both S3 permissions AND KMS key policy/IAM permissions are required for KMS-encrypted objects) - a very common real-world troubleshooting scenario.

</details>

<details markdown="1">
<summary>❓ <b>17. How does KMS integrate with EBS volume encryption?</b></summary>
<br>

When creating an encrypted EBS volume, you select a CMK; AWS uses envelope encryption to encrypt the volume's data, and any snapshots/AMIs created from it remain encrypted with the same (or a re-specified) key, with EC2 needing appropriate KMS grants to attach/use the encrypted volume.

</details>

<details markdown="1">
<summary>❓ <b>18. Can you enable default encryption for new EBS volumes/S3 buckets account-wide?</b></summary>
<br>

Yes - EBS supports "Always Encrypt" account-level default (all new volumes/snapshots encrypted automatically with a default or specified CMK), and S3 supports Default Bucket Encryption (SSE-S3 or SSE-KMS) so objects are encrypted even without the uploader specifying encryption headers.

</details>

## 🧾 Auditing & Compliance

<details markdown="1">
<summary>❓ <b>19. How do you audit KMS key usage?</b></summary>
<br>

Every KMS API call (Encrypt, Decrypt, GenerateDataKey, etc.) is logged in CloudTrail, including the requesting principal, so you can track exactly who used a key and when - critical for compliance and detecting anomalous decrypt patterns.

</details>

<details markdown="1">
<summary>🎯 <b>20. Scenario: A compliance requirement mandates that a specific key can only be used from within your corporate network/VPC. How do you enforce this?</b></summary>
<br>

Add a `Condition` in the key policy checking `aws:SourceVpce` (via a KMS VPC endpoint) or `aws:SourceIp`, denying usage from outside those conditions - combined with a VPC Endpoint for KMS to keep the traffic off the public internet entirely.

</details>

<details markdown="1">
<summary>❓ <b>21. What is a Multi-Region Key in KMS?</b></summary>
<br>

A CMK that can be replicated across multiple regions, sharing the same key material and Key ID structure, allowing you to encrypt data in one region and decrypt it in another without re-encrypting - useful for DR/global applications, though each replica is still managed independently for rotation/deletion at the regional level.

</details>

## 🎯 Real-Time Scenarios

<details markdown="1">
<summary>🎯 <b>22. Scenario: You need different teams to only be able to encrypt/decrypt their own data using separate keys, with full audit segregation. How do you design this?</b></summary>
<br>

Create separate CMKs per team/application with key policies scoped to that team's IAM roles only, tag keys accordingly, and use CloudTrail (filtered per key ARN) for team-specific audit trails - avoiding a single shared key that mixes access/audit trails across teams.

</details>

<details markdown="1">
<summary>🎯 <b>23. Scenario: Your application needs to decrypt data very frequently (thousands of times per second), but KMS API calls are becoming a bottleneck/cost concern. How do you optimize?</b></summary>
<br>

Use envelope encryption and cache the plaintext DEK in memory (with appropriate expiration/security controls) rather than calling KMS `Decrypt` for every single operation, or use the AWS Encryption SDK, which handles data key caching securely and efficiently out of the box.

</details>

<details markdown="1">
<summary>❓ <b>24. How would you migrate encrypted data from one KMS key to another (e.g., due to a compliance requirement to use a new key)?</b></summary>
<br>

Since existing ciphertext is bound to the original CMK, you must decrypt the data using the old key and re-encrypt it with the new key (re-encryption) - for services like S3 this typically means copying objects with the new encryption key specified (triggering server-side re-encryption), rather than an in-place key swap.

</details>

<details markdown="1">
<summary>❓ <b>25. What's the difference between KMS and AWS CloudHSM, and when would you choose CloudHSM?</b></summary>
<br>

KMS is a multi-tenant, fully managed service (shared HSM infrastructure, simpler, integrates natively with most AWS services). CloudHSM provides single-tenant, dedicated HSM instances you fully control (needed for specific compliance/regulatory requirements mandating dedicated hardware, certain cryptographic operations not supported by KMS, or when you need to import/manage keys outside AWS's control model entirely).

</details>

