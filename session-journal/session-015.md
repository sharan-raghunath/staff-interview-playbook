# Session 15

## Backend Theory
- Introduced `ValueTask<T>` from first principles.
- Learned that `Task<T>` is the default choice.
- `ValueTask<T>` is an optimization for hot paths that complete synchronously most of the time (for example, cache hits).
- Decision should be evidence-based through profiling rather than premature optimization.
- Deferred advanced topics:
  - Multiple awaits
  - AsTask()
  - IValueTaskSource
  - Public API guidance

## Coding
Problem: Valid Parentheses

### Interview Process
- Clarified input assumptions before solving.
- Recognized that a stack models the LIFO nature of nested brackets.
- Discussed switch vs dictionary trade-off and avoiding over-engineering.
- Implemented an optimal O(n) time / O(n) space solution.
- Review improvements:
  - Prefer TryGetValue/TryPop over separate lookups.
  - Prefer Count == 0 over Any().

## Coaching
- Finalized long-term session cadence:
  - Mon-Thu: Backend + Coding + System Design Building Block + Journal
  - Fri: Coding interview + System Design interview + Behavioural + Weekly review
  - Sat/Sun: Optional
- Agreed that new concepts should not be introduced before their planned session.
- Repository curriculum remains the source of truth.

## Next Session
- Backend: Continue ValueTask (advanced usage)
- Coding: Next stack problem
- System Design Building Block: Load Balancing
