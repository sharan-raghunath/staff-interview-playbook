# Load Balancers

A load balancer exposes one stable entry point and distributes incoming work across healthy backend instances.

## Problems solved

A single instance creates:

- limited capacity;
- a single point of failure;
- maintenance and deployment downtime risk;
- resource contention and unfair latency.

Adding replicas solves none of these unless clients have a mechanism that chooses a suitable healthy replica.

## Basic routing strategies

### Round robin

Rotate through healthy instances.

Best fit:

- similar instance capacity;
- roughly similar request cost;
- short-lived requests/connections.

Weakness: equal request counts do not guarantee equal work.

### Least connections / least load

Choose the backend handling the least current work.

Possible load signals include:

- active connections;
- queue length;
- CPU or memory pressure;
- observed latency.

More informed routing requires more measurement and coordination.

### Geographic or latency-based routing

Choose the nearest or fastest region. This is usually a global routing concern, followed by another balancing layer inside the selected region.

```text
Global users
    -> regional routing
    -> regional load balancer
    -> application replicas
```

## Health checks

The load balancer must stop routing new work to failed instances and reintroduce them only after they are healthy.

Health-check frequency trades off:

- faster failure detection;
- health-check traffic and false-positive risk.

## Layer 4 and Layer 7

### Layer 4

Routes using transport/network information:

```text
IP
port
TCP/UDP
```

Use when all backends provide the same service and application-level inspection is unnecessary.

### Layer 7

Understands an application protocol such as HTTP and can route using:

```text
host
URL path
method
headers
cookies
```

Use when routing decisions depend on HTTP semantics, such as sending `/api/invoices` and `/api/users` to different service pools.

Mental model:

> Layer 4 chooses where a connection goes. Layer 7 can choose where an HTTP request goes.

## Sticky sessions

Sticky sessions preserve backend affinity for a client.

They may be required when session state is stored only in application-instance memory. Trade-offs include:

- uneven load distribution;
- harder failover;
- loss of local state when the selected instance fails;
- reduced ability for any healthy replica to serve any request.

Preferred direction:

> Keep critical session and workflow state outside individual application instances so replicas remain interchangeable.

Sticky sessions are different from HTTP connection reuse. Connection reuse concerns an existing transport connection; stickiness concerns routing a client's requests to the same backend over time.

## Interview selection rule

Choose based on the information needed for the routing decision:

- identical TCP backends -> Layer 4 may be sufficient;
- path/header-aware routing -> Layer 7;
- uniform inexpensive work -> round robin;
- variable-duration work -> least connections/load;
- global users -> geographic/latency routing plus regional balancing.

## Pending extensions

Not covered yet:

- weighted routing;
- advanced failure thresholds;
- reverse-proxy internals;
- detailed global failover and DR.
