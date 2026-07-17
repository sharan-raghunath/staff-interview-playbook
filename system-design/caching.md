# Caching Fundamentals and Invoice Manager Strategy

## Why caching exists

Repeated reads of unchanged data increase application latency and consume database CPU, I/O, connections and network capacity.

A cache keeps reusable data closer to the caller, improving latency and scalability.

Core trade-off:

> Caching improves performance by accepting a controlled risk of stale data.

Before caching, ask:

- How frequently is the value read?
- How often does it change?
- How harmful is staleness?
- How large is the value?
- How expensive is rebuilding it?
- Is there already an appropriate durable store?

## Cache-aside

The application owns the cache workflow:

```text
check cache
  hit  -> return
  miss -> read source of truth
          apply business rules
          populate cache
          return
```

This is a strong default for enterprise microservices because the application retains responsibility for authorization, validation, tenant rules, mapping and data access.

## Read-through

The application reads only through the cache; on a miss, the cache loads from the backing store.

Trade-offs:

- simpler caller API;
- tighter cache-to-database coupling;
- cache requires data-access knowledge, credentials and network access;
- expands the privileged trust boundary and attack surface.

It may fit tightly integrated data grids or managed caching products, but cache-aside remains the preferred default for the current architecture.

## Write strategies

### Database first, then invalidate

```text
update source of truth
invalidate cache entry
next read repopulates it
```

This keeps the database authoritative and is easier to reason about than maintaining two synchronized copies.

A failed invalidation can still leave a stale cache hit, so TTL and retry/operational handling remain important.

### Updating cache after the database

This avoids the next miss but creates ordering races. Two concurrent writers can commit database updates in one order and cache updates in the reverse order, leaving the cache stale.

### Write-through

The cache synchronously writes the backing store before acknowledging success. Every write still waits for persistence, while the cache gains persistence responsibilities and privileges.

### Write-behind

The cache acknowledges before asynchronously persisting. It can improve write latency but risks loss of acknowledged data and requires durable pending writes and reconciliation. This is not the preferred default for ordinary transactional enterprise data.

## Cache stampede

When a popular key expires, many concurrent requests can all miss and query the source of truth.

Mitigation learned at the current depth:

- coordinate population per cache key;
- re-check the cache after acquiring the per-key lock;
- avoid one global lock that serializes unrelated keys.

An in-process per-key lock coordinates only one application instance. Cross-pod coordination is a future distributed-locking topic.

## In-memory and distributed caching

### L1: in-memory

Advantages:

- same-process lookup;
- no network or serialization overhead;
- lowest latency;
- no extra infrastructure dependency.

Trade-offs:

- private to each pod;
- duplicate data across replicas;
- lower aggregate hit ratio;
- invalidation does not automatically reach other pods.

### L2: distributed cache

Advantages:

- shared by all replicas;
- centralized invalidation;
- better aggregate hit ratio;
- supports horizontal scaling.

Trade-offs:

- network and serialization cost;
- additional infrastructure and failure mode;
- slower than L1, although usually faster than the source database.

## Multi-level caching

```text
L1 in-memory
  miss -> L2 distributed cache
            miss -> source of truth
                    populate L2
                    populate L1
```

L1 should normally have a shorter TTL than L2 because each replica's L1 can remain stale after the shared L2 entry is invalidated.

## Expiry and eviction

### TTL

TTL represents how long the system is willing to trust a cached copy.

Choose it from:

- data volatility;
- stale-data impact;
- propagation expectations;
- rebuild cost;
- source-of-truth capacity;
- availability of explicit invalidation.

Useful examples:

- feature flags may need short propagation windows;
- tenant configuration can use medium or component-specific TTLs plus explicit invalidation;
- sessions often use short or inactivity-based expiry, distinct from token expiry;
- static reference data can use long absolute TTLs;
- transient OCR workflow data must cover queue delay and retries if cached, though the chosen design does not cache the payload.

Avoid synchronizing every replica to expire and reload the same keys at exactly the same instant.

### LRU

Evicts the least recently accessed item. It adapts to changing access patterns but can remove data just before a predictable burst.

### LFU

Evicts the least frequently accessed item. It preserves consistently hot data but pure lifetime frequency can disadvantage newly hot data and overvalue ancient popularity.

Policy selection must match observed workload. Hybrid or aging approaches may adapt better, but their internals were not covered.

## Cache warming

Preload a selected set of known hot data before or during traffic activation.

Good candidates:

- active tenant settings;
- active feature flags;
- common reference data;
- templates for active tenants.

Do not load everything blindly; warming policy should follow business knowledge and measured access patterns.

## Distributed invalidation

For pod-local L1 caches, invalidating Redis/L2 alone does not remove stale local copies.

Current-level options:

- short L1 TTL and accept bounded staleness;
- broadcast an invalidation event to replicas for important changes.

A rolling restart clears local caches but is too heavy to use as the normal invalidation mechanism.

## Invoice Manager caching decisions

### Tenant configuration

Cache frequently used tenant settings, preferably in L2 when they must be shared. Split monolithic configuration into keys with distinct freshness needs, for example feature flags, limits, source-system settings and branding.

Use TTLs based on propagation requirements and explicit invalidation after administrative changes. LFU may suit skewed tenant traffic; LRU may adapt better when tenant activity changes.

### OCR payload

Do not cache by default:

- large payload;
- low reuse;
- durable Blob storage already exists;
- limited latency benefit relative to memory and complexity cost.

Cache only lightweight metadata or the durable object reference when useful. DE reads the durable artifact from Blob storage.

### Final extraction result

Do not rerun GPT on a normal refresh. Read the persisted, versioned result using tenant/invoice/job/result identifiers.

Do not cache by default because expected reuse is low and the durable result already exists. Reprocessing requires an explicit reason such as changed input, configuration, model/prompt version or prior failure.

### Reference data

Small global reference data can use L1 with a long absolute TTL and startup warming.

Policies should differ by volatility and scope:

- country and currency codes: very long TTL;
- document types: long TTL plus explicit invalidation;
- tenant-specific asset categories: tenant-scoped keys and potentially shorter TTL.

Use invalidation messages for rare administrative changes rather than pod restarts.

### Uploaded PDFs

Do not cache:

- large and infrequently reused;
- may contain PII;
- another copy expands the trust boundary, retention obligations and attack surface;
- Blob storage is the appropriate durable object store.

Persist only metadata and the authorized object reference.

## Current curriculum boundary

Covered:

- cache-aside, read-through, write-through and write-behind concepts;
- stale reads and invalidation;
- cache stampede and per-key coordination at a conceptual level;
- L1, L2 and multi-level caching;
- TTL, LRU, LFU and warming;
- distributed invalidation at a conceptual level;
- Invoice Manager cache/no-cache decisions.

Not yet covered:

- Redis product internals;
- exact eviction implementations;
- distributed locking;
- cache-coherence protocols;
- detailed invalidation messaging technology;
- negative caching, penetration and avalanche as dedicated topics;
- operational sizing and capacity modelling.
