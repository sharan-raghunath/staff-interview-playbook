# Session 15 — ValueTask Fundamentals and Valid Parentheses

## Scope

- Backend theory: `Task<T>` versus `ValueTask<T>` fundamentals.
- Coding: Valid Parentheses and stack fundamentals.
- System-design cadence decision: small building blocks every normal weekday session; simulations on Friday.

No standalone system-design building block was completed in this session. That part was discussed as a curriculum change and begins from Session 16.

## Backend — `ValueTask<T>`

`ValueTask<T>` was new to the user and was taught step by step.

Covered:

- `Task<T>` is the default return type for asynchronous value-producing operations.
- `ValueTask<T>` is a value type that can represent an already available value or an underlying asynchronous operation.
- The synchronous-completion path, such as a cache hit, is the path most likely to benefit.
- Caching itself is not the criterion; hot-path frequency, synchronous-completion rate and measured allocation cost are.
- `ValueTask<T>` has additional complexity, so it is not a general replacement for `Task<T>`.
- Selection should be profiling-driven rather than a premature optimization.

Interview rule:

> Default to `Task<T>`. Consider `ValueTask<T>` only for a hot, allocation-sensitive method that completes synchronously most of the time and where profiling shows completed-task allocations matter.

Intentionally deferred:

- multiple-await restrictions;
- `AsTask()`;
- `IValueTaskSource`;
- detailed public-API guidance.

## Coding — Valid Parentheses

### Requirement gathering

The user asked:

- whether characters other than brackets can appear;
- whether the empty string is valid;
- whether standard data structures are allowed.

The agreed constraints were:

- only `()[]{}` appear;
- empty string is valid;
- standard library collections are allowed.

### Solution derivation

The user independently identified the stack-based solution:

- push opening brackets;
- on a closing bracket, inspect the most recent opening bracket;
- fail on an empty stack or mismatched pair;
- return true only if the stack is empty after the scan.

The proposed solution was already optimal rather than brute force.

Complexity:

```text
Time: O(n)
Space: O(n)
```

### Implementation review

The original implementation was correct. Review improvements were:

- replace `ContainsValue` with key-based lookup;
- prefer `TryGetValue` for mapping lookup;
- prefer `TryPop` when peek is immediately followed by pop;
- prefer `Count == 0` over `Any() ? false : true`.

A switch and dictionary were both considered valid for the fixed contract. The dictionary was selected because it centralizes the mapping, while avoiding unnecessary configuration-driven over-engineering.

## Curriculum decision

The normal weekday structure was updated to include smaller system-design practice every day:

1. one backend theory topic;
2. one coding problem;
3. one small system-design building block;
4. journal/repository reconciliation.

Friday remains application/simulation day:

- unseen coding interview;
- system-design interview;
- behavioral question(s);
- weekly review.

Saturday and Sunday remain optional.

## Session status

Completed:

- `ValueTask<T>` fundamentals;
- stack fundamentals;
- Valid Parentheses;
- revised recurring session cadence.

Pending:

- advanced `ValueTask<T>` semantics;
- next stack problem;
- first dedicated system-design building block under the new cadence.
