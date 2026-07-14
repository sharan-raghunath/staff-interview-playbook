# Session 16 — ValueTask Usage, Load Balancers and Min Stack

## Scope

- Backend theory: safe `ValueTask<T>` usage at the current curriculum depth.
- System-design building block: load-balancer fundamentals.
- Coding: LeetCode 155 — Min Stack.

The session remained guided learning rather than interview simulation.

## Backend — `ValueTask<T>` Usage Guidance

Session 15 established why `ValueTask<T>` exists and when it may reduce allocations. Session 16 added one safe usage rule:

> Await a `ValueTask<T>` once and use the result immediately.

Covered:

- a caller should not assume which representation a `ValueTask<T>` contains;
- code should not store one `ValueTask<T>` and await that same value repeatedly;
- calling a method twice and awaiting the same returned awaitable twice are different questions;
- `Task<T>` remains the default return type;
- `ValueTask<T>` remains an optimization for measured hot paths with frequent synchronous completion.

Intentionally deferred because their prerequisites have not been covered:

- `AsTask()`;
- `IValueTaskSource`;
- object-pooling internals;
- exact multiple-await behavior for every representation;
- detailed public-API guidance.

## System Design Building Block — Load Balancers

### Why load balancers exist

A single application instance creates:

- a capacity bottleneck;
- a single point of failure;
- resource contention and unfair latency between users or tenants;
- deployment and maintenance risk when the only instance restarts.

A load balancer provides one stable endpoint and distributes work across healthy instances.

### Routing strategies covered

- **Round robin:** rotate across healthy, similarly sized instances.
- **Least connections / least load:** prefer the instance currently handling less work.
- **Geographic or latency-based routing:** choose a nearer or faster region.

The routing decision must match the available information and the problem being solved.

### Health checks

A load balancer must maintain a view of healthy backends and stop routing new work to failed instances. Health-check frequency is a trade-off between faster failure detection and monitoring overhead.

### Layer 4 versus Layer 7

- **Layer 4:** routes using transport/network information such as IP, port and TCP/UDP. It is sufficient when all healthy backends provide the same service and application-level inspection is unnecessary.
- **Layer 7:** understands application protocols such as HTTP and can route using host, URL path, headers, cookies or method.

Examples:

- `/api/invoices` and `/api/users` to different services requires Layer 7.
- Raw TCP distribution across identical proxy instances can use Layer 4.

### Sticky sessions

Sticky sessions route a client repeatedly to the same backend instance. They may be required when an application keeps session state in instance memory, but they make load distribution and failover harder.

Preferred architecture:

> Keep critical session and workflow state outside the application instance so any healthy replica can serve the next request.

If a sticky-session target fails:

- the load balancer must route subsequent requests to a healthy instance;
- server-local state may be lost;
- durable business state can be reconstructed or resumed safely by the new instance.

## Coding — LeetCode 155: Min Stack

### Requirements clarified

- implement the stack without the built-in `Stack<T>`;
- store integers only;
- grow dynamically;
- `Pop` and `Top` on an empty stack may throw `InvalidOperationException`;
- `Push`, `Pop`, `Top` and `GetMin` should all be O(1).

### Brute-force design

A normal stack can be implemented with `List<int>`:

- `Push`: append;
- `Pop`: remove the last item;
- `Top`: read the last item;
- `GetMin`: scan the list.

The bottleneck is `GetMin`, which is O(n). Maintaining only one global minimum still requires a scan when that minimum is popped.

### Optimized insight

Store the minimum associated with every stack level:

```text
(value, minimumAtThisLevel)
```

Example:

```text
Push 10 -> (10, 10)
Push 99 -> (99, 10)
Push 5  -> (5, 5)
Push 7  -> (7, 5)
```

The top entry always contains both the top value and the current minimum. Popping naturally reveals the previous level's minimum.

### Submitted solution

The user implemented the structure with `List<int[]>`, storing:

```text
index 0 -> value
index 1 -> minimum at that level
```

The implementation was correct and accepted the O(1) requirement. Code-quality review suggested a named tuple or small record/struct instead of numeric array indexes for readability.

### Complexity

```text
Push:   O(1) amortized
Pop:    O(1)
Top:    O(1)
GetMin: O(1)
Space:  O(n)
```

## Coaching and Process Notes

- Sessions are logical units and may span multiple calendar days.
- Feedback and references must use session numbers rather than “today” or “yesterday.”
- Assumptions established in a problem must not be changed silently.
- When an explanation requires uncovered prerequisites, stop at the curriculum boundary and defer the internals.

## Session Status

Completed:

- safe single-consumption guidance for `ValueTask<T>` at the current depth;
- load-balancer fundamentals;
- Min Stack derivation, implementation and review;
- durable repository reconciliation.

Pending for later sessions:

- advanced `ValueTask<T>` internals and `AsTask()`;
- weighted routing and other advanced load-balancer policies;
- the next coding pattern/problem;
- detailed DR and circuit-breaker topics according to the roadmap.
