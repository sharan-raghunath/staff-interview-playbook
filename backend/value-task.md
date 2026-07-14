# Task<T> vs ValueTask<T>

## Why `ValueTask<T>` exists

`Task<T>` is the default representation of an asynchronous operation that eventually produces a value.

Some methods, however, complete synchronously very often. A cache lookup is a common example:

```csharp
public ValueTask<Customer> GetCustomerAsync(int id)
{
    if (cache.TryGetValue(id, out Customer customer))
    {
        return new ValueTask<Customer>(customer);
    }

    return new ValueTask<Customer>(repository.GetCustomerAsync(id));
}
```

On the cache-hit path, the value is already available. `ValueTask<T>` can carry that completed value directly. On the cache-miss path, it can wrap an underlying asynchronous operation.

## Mental model

```text
Task<T>
  -> reference-type object representing eventual completion

ValueTask<T>
  -> value type that can represent either:
       1. an already available T
       2. an underlying asynchronous operation
```

The caller can await either form:

```csharp
Customer customer = await GetCustomerAsync(id);
```

## When it may help

Consider `ValueTask<T>` only when all of the following are true:

1. The method is on a hot path.
2. It completes synchronously most of the time.
3. Allocation reduction is measurable and important.
4. Profiling shows that completed `Task<T>` allocations matter.

Caching can create this shape, but caching alone is not the deciding factor. Call frequency, synchronous-completion rate and measured allocation cost are the real criteria.

## Why `Task<T>` remains the default

`ValueTask<T>` has more complex representation and usage semantics because it can hold either a result or asynchronous work. When a method is mostly asynchronous, there may be little or no allocation benefit while the additional complexity remains.

Interview wording:

> I default to `Task<T>`. I consider `ValueTask<T>` only for hot, allocation-sensitive methods that complete synchronously most of the time, and only after profiling shows that task allocations are meaningful.

## Scope covered in Session 15

Covered:

- purpose of `ValueTask<T>`;
- synchronous-result path versus asynchronous path;
- why hot-path frequency and synchronous completion matter;
- why profiling should drive the choice;
- why `Task<T>` remains the default.

Intentionally pending:

- multiple-await restrictions;
- `AsTask()`;
- `IValueTaskSource`;
- detailed public-API guidance;
- framework-specific implementation details.
