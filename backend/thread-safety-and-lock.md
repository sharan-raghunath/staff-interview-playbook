# Thread Safety, Race Conditions and `lock`

## Why thread safety matters

Shared mutable state can produce incorrect results when multiple threads read and update it concurrently.

Example:

```csharp
public int Counter = 0;

public void Increment()
{
    Counter++;
}
```

`Counter++` is conceptually a read-modify-write sequence:

```text
read Counter
add 1
write Counter
```

Two threads can both read the same old value and then both write the same new value, losing one increment.

## Race condition

A race condition occurs when multiple threads access shared mutable state concurrently and the result depends on execution order.

## Critical section

The entire read-modify-write sequence must be protected, not only the final write.

> A critical section is code that accesses shared mutable state and must be executed by only one thread at a time.

## Mutual exclusion with `lock`

```csharp
private readonly object _counterLock = new();

public void Increment()
{
    lock (_counterLock)
    {
        Counter++;
    }
}
```

- `private` prevents external code from acquiring and holding the class's synchronization object.
- `readonly` keeps the reference stable so every thread synchronizes on the same object.
- Any methods using the same lock object exclude one another while a different thread owns that lock.

## Current curriculum boundary

Covered:

- race conditions;
- shared mutable state;
- critical sections;
- mutual exclusion;
- basic C# `lock` usage;
- ownership of a `private readonly` lock object.

Not yet covered:

- re-entrant locking details;
- `Monitor` internals;
- `Interlocked`;
- `SemaphoreSlim`;
- `ReaderWriterLockSlim`;
- deadlock analysis;
- distributed locking.
