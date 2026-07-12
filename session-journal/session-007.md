# Session 7 Journal

## Theme

Friday consolidation day: rapid-fire interview practice, Invoice Manager async redesign part 1, and one coding problem.

The day deliberately stayed lighter than a full architecture deep dive. The focus was on strengthening recall and keeping the redesign disciplined rather than jumping too quickly into Azure Service Bus details.

---

## Rapid-Fire Interview Practice

### Question 1 — Browser to Invoice Manager

We walked through what happens when a user enters the Invoice Manager URL.

Canonical path:

```text
Browser
  ↓
DNS
  ↓
Akamai
  ↓
Azure Application Gateway
  ↓
Traefik Ingress
  ↓
IM Web / Edge API
  ↓
Microsoft Entra authentication if required
  ↓
Session established
  ↓
Angular application returned and bootstrapped
```

Important interview refinement:

- DNS resolves the public host to the edge path.
- Akamai handles edge security, DDoS/WAF and IP whitelisting.
- Azure Application Gateway and Traefik route traffic into AKS.
- IM Web / Edge API is the public-facing service boundary.
- If unauthenticated, the application redirects to the IdP.
- After authentication, a session is created and the browser stores the session cookie.
- The browser then downloads and runs Angular assets.

---

### Question 2 — Current vs Future Failure Handling

We corrected an important mistake: do not describe future-state recovery as if it exists today.

Current state:

- Synchronous processing.
- Fail-fast behavior.
- No queue.
- No persisted workflow checkpoints.
- No idempotent recovery.
- If Orchestrator crashes after OCR, the request fails and the OCR work is effectively wasted.

Future state:

- Idempotency.
- Persisted workflow state.
- Checkpointing.
- Resumability.
- Queued processing.

Interview principle:

> Always separate current production behavior from proposed target-state architecture.

---

### Question 3 — Compromised Text Extractor

Scenario: Text Extractor is compromised.

Answer:

- Every internal API must authenticate and authorize every request.
- Internal network location is not a trust boundary.
- TE should only have the minimum service permissions it needs.
- API Service and Source System should reject calls from TE unless the service token has the required audience and roles/scopes.

Staff-level phrasing:

> Internal services are not trusted merely because they are inside the network. Each service independently validates issuer, audience, signature, expiry and permissions. If TE is compromised, the attacker inherits only TE's limited permissions, reducing blast radius.

---

### Question 4 — Invoice Manager Data Ownership

Important clarification:

Invoice Manager is not the system of record for invoice business data.

Current data that Invoice Manager owns:

- User overrides of predictions.
- Pricing-related metadata.
- Operational/application metadata.

Future data that Invoice Manager may own:

- Job state.
- Processing lifecycle state.
- Retry count.
- Failure reason.
- Correlation ID.
- Idempotency key.
- Temporary processing artifact references.

It should not own:

- Invoice lifecycle.
- Accounting/payment state.
- Long-term business invoice data.
- Source-system master data.

---

### Question 5 — Edge API Compromise and Zero Trust Limits

Key lesson:

Zero Trust reduces blast radius but does not magically prevent all damage if a trusted service is fully compromised.

If Edge API is compromised, the attacker may act with Edge API's legitimate permissions.

Mitigations:

- Least privilege.
- Short-lived tokens.
- Credential rotation.
- Monitoring and anomaly detection.
- Rapid revocation of service credentials.
- WAF remains useful at the edge but does not protect service-to-service calls originating from a compromised internal service.

Staff-level phrasing:

> Zero Trust does not eliminate the impact of a compromised service. It minimizes blast radius through independent authorization, least privilege, short-lived credentials, monitoring, and revocation.

---

## Invoice Manager Architecture Updates

A proprietary architecture diagram was reviewed, but the image itself must not be reproduced.

The following relationships were captured in documentation instead:

```text
Asset Finance Web / Client Web
  ↓
Akamai
  ↓
Azure Application Gateway
  ↓
Traefik Ingress
  ↓
IM Web / Edge API
  ↓
IM REST API
  ↓
Orchestrator
  ├── Text Extraction Service
  └── Data Extraction Service
```

Platform dependencies:

- Azure SQL
- Azure Redis Cache
- Azure Key Vault
- Azure App Configuration
- Azure Application Insights
- Log Analytics
- Availability checks
- Azure Jump Box for operations
- Azure Computer Vision
- Azure Document Intelligence

Identity path:

```text
Client Identity Provider
  ↓
Federation
  ↓
FIS Identity Provider
  ↓
Microsoft Entra
  ↓
Invoice Manager
```

Text Extractor abstraction:

```text
Orchestrator
  ↓
Text Extraction Service
  ├── Azure Computer Vision
  └── Azure Document Intelligence
```

The Orchestrator should not care which OCR provider is used. Text Extraction owns that abstraction.

---

## Redis Clarification

Invoice Manager uses Azure Redis Cache as a distributed cache.

Architectural principle:

> Redis may accelerate reads and share transient state across replicas, but it should not be the authoritative store for recovery-critical processing state.

For future async processing:

- SQL should be the source of truth for job state and workflow status.
- Redis may be used as a cache for frequently polled status.
- Redis loss should degrade performance, not correctness.

---

## System Design — Async Redesign Part 1

### Queue Placement Discussion

Possible placements were discussed:

1. Queue after IM Web.
2. Queue after IM REST API.
3. Queue after Orchestrator.
4. Other variants.

Observation:

- Queueing immediately after IM Web keeps the user-facing request simple, but IM Web should not own workflow-level progress.
- IM REST API is closer to business validation and job creation.
- Orchestrator should own workflow sequencing and progress updates.

Tentative direction:

```text
IM Web / Edge API
  ↓
IM REST API
  ↓
Queue
  ↓
Orchestrator
  ↓
TE / DE
```

This remains a target-state discussion and will be finalized after queue fundamentals are covered.

---

### SQL vs Redis for Job State

Decision:

Use SQL as the authoritative store for job state.

Reasons:

- Better durability.
- Better consistency.
- Easier troubleshooting.
- No TTL-driven loss of state.
- Better auditability.
- Suitable for recovery-critical metadata.

Redis may be used as a caching layer only.

Important phrasing:

> SQL is the source of truth for processing state. Redis is an optimization layer, not the correctness boundary.

---

### Workflow State

Future target-state workflow may include:

```text
Accepted
  ↓
Queued
  ↓
Picked up for OCR
  ↓
OCR in progress
  ↓
OCR complete
  ↓
Field inference in progress
  ↓
Prediction complete
  ↓
Ready for user review
  ↓
Submitted to source system
```

Stage-completion rule:

> Do not mark a stage complete until the downstream operation has actually completed successfully.

---

### OCR Output Persistence

Future-state design should persist OCR output as a processing artifact.

Reason:

If OCR completes and the Orchestrator crashes before prediction, the system should resume from OCR complete rather than rerun OCR.

Suggested storage split:

```text
SQL
- JobId
- OCR status
- OCR artifact location
- checksum/hash
- timestamps

Blob Storage
- OCR JSON
- page-level OCR output
- extracted text
```

Important distinction:

Persisting OCR artifacts does not make Invoice Manager the invoice system of record. It means Invoice Manager owns temporary processing artifacts required for recovery and operational efficiency.

---

## Queue Learning Decision

We intentionally stopped before detailed Azure Service Bus mechanics.

Reason:

Before choosing queue technology, we want the reusable queue mental model:

```text
Why queue?
  ↓
Producer / consumer
  ↓
Message ownership
  ↓
ACK / NACK
  ↓
Message locks / visibility timeout
  ↓
Retry
  ↓
DLQ
  ↓
Then Azure Service Bus / Kafka / RabbitMQ mapping
```

This is pending for the next session.

---

## Coding — LeetCode 724: Find Pivot Index

Pattern: Prefix Sum / Running Sum.

Problem:

Find the leftmost index where the sum of elements to the left equals the sum of elements to the right.

Key clarifications:

- Pivot index is excluded from both sums.
- Empty side sum is `0`.
- Negative numbers are allowed.
- Duplicate numbers are allowed.
- Return leftmost pivot if multiple exist.
- Return `-1` if none exists.

Brute force:

- For each index, calculate left and right sums.
- Time: O(n²).
- Space: O(1).

Optimized insight:

- Compute `totalSum` once.
- Maintain `leftSum` while iterating.
- `rightSum = totalSum - leftSum - nums[i]`.
- If `leftSum == rightSum`, return `i`.
- Then update `leftSum += nums[i]`.

Final solution:

```csharp
public class Solution {
    public int PivotIndex(int[] nums) {
        int leftSum = 0, rightSum, totalSum = 0;

        for (int i = 0; i < nums.Length; i++) {
            totalSum += nums[i];
        }

        for (int i = 0; i < nums.Length; i++) {
            rightSum = totalSum - nums[i] - leftSum;

            if (leftSum == rightSum) {
                return i;
            }

            leftSum = leftSum + nums[i];
        }

        return -1;
    }
}
```

Complexity:

- Time: O(n).
- Space: O(1).

---

## Week 1 Status

Completed coding patterns:

- Hash Lookup — Two Sum
- Running Minimum — Best Time to Buy and Sell Stock
- Frequency Counting — Valid Anagram
- Prefix / Suffix — Product of Array Except Self
- Two Pointers — Valid Palindrome
- Fixed-size Sliding Window — Maximum Average Subarray I
- Prefix Sum / Running Sum — Find Pivot Index

Completed backend/security module:

- HTTP statelessness
- Sessions
- Cookies
- Distributed Sessions / Redis
- JWT
- OAuth 2.0
- OIDC
- Token Lifecycle
- Logout / SSO / SLO
- Zero Trust
- Identity architecture

Current system design level:

- Strong current vs target-state discipline.
- Strong data ownership thinking.
- Async redesign motivation understood.
- Queue fundamentals pending before technology selection.

---

## Pending for Next Session

- Build queue mental model from first principles.
- Derive ACK/NACK, locks, visibility timeout, retry and DLQ.
- Then map concepts to Azure Service Bus.
- Finalize queue placement for Invoice Manager async redesign.
- Continue with variable-size sliding window or next selected coding pattern depending on the plan.
