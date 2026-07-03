# Queue Fundamentals

Status: Pending next session.

This document is intentionally a placeholder for the next distributed-systems module.

The learning approach should be first principles before Azure Service Bus.

## Planned Sequence

```text
Why queue?
  ↓
Producer / consumer
  ↓
Message ownership
  ↓
ACK / NACK
  ↓
Message lock / visibility timeout
  ↓
Retry
  ↓
Dead Letter Queue
  ↓
Azure Service Bus mapping
  ↓
Kafka / RabbitMQ comparison
```

## Key Questions To Answer

- When is work considered accepted?
- Who owns the message while it is being processed?
- What happens if the consumer crashes?
- When should the message be removed from the queue?
- What makes retry safe?
- What belongs in a DLQ?
- How does queue behavior affect idempotency and recovery?
