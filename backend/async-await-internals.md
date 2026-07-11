# Async/Await Internals in .NET

## Why Asynchronous I/O Matters

Synchronous database, network, or file I/O blocks a thread while the external operation is in progress. The thread is not doing useful CPU work during that wait.

Under high concurrency, many blocked ThreadPool threads can cause requests to queue, time out, or be rejected even when CPU usage is not especially high. This is ThreadPool starvation.

`async`/`await` improves scalability by avoiding blocked worker threads during asynchronous I/O. It does not make the database, network, or external service itself faster.

## Process, Thread, and Task

- A **process** is an execution boundary with its own virtual memory and resources.
- A **thread** is an execution path inside a process. It has its own call stack while sharing process memory.
- A **Task** represents an operation and its eventual completion.

A `Task` can expose whether the operation is incomplete, completed, faulted, or cancelled. A `Task<T>` also exposes the successful result.

A Task is not:

- a process;
- a dedicated thread;
- necessarily newly created work.

> A Task represents work. It is not the executor of that work.

## What Happens at Await

```csharp
var response = await httpClient.GetAsync(url);
ProcessResponse(response);
```

`GetAsync` starts the HTTP operation and returns a `Task<HttpResponseMessage>`.

If the task is already complete, execution continues synchronously. The state machine does not suspend at that await.

If the task is incomplete:

1. The async method records the state required to resume later.
2. The continuation is registered with the awaiter.
3. The current invocation of the state machine returns.
4. The current thread is free to perform other work.
5. The operating system and network stack continue the I/O wait without a .NET worker thread sitting blocked.
6. When the operation completes, a suitable thread executes the continuation.

The external system completes the I/O. A .NET thread is later required to run the continuation.

## Why State Cannot Remain on the Original Stack

After an incomplete await, the original thread may immediately execute unrelated work. The original async method's stack frame therefore cannot remain as the durable home of its execution state.

The compiler-generated state machine preserves information such as:

- where execution should resume;
- local variables needed later;
- method parameters and instance references needed later;
- the current awaiter;
- the builder for the Task returned to the caller;
- exception and completion state.

The state machine is generated as a value type in common compiler output. When the method actually suspends, its state must survive beyond the current stack frame and is maintained through heap-backed async infrastructure. If awaited work completes synchronously, some allocation can be avoided.

## The State Field

The state field acts as a bookmark.

Conceptually:

```text
-1  initial execution
 0  resume after first await
 1  resume after second await
-2  completed
```

Local variables alone are not enough. With multiple awaits, the runtime must know which suspension point should resume.

## MoveNext

The compiler transforms the async method into a state machine whose main method is conceptually `MoveNext()`.

`MoveNext()` runs until one of two things happens:

- the method reaches an incomplete await and suspends; or
- the method reaches completion.

When an awaited operation later completes, `MoveNext()` is invoked again. It reads the saved state and continues from the correct point.

For three awaits that all complete asynchronously, `MoveNext()` is commonly invoked four times:

1. initial execution until the first await;
2. resume after the first await until the second;
3. resume after the second await until the third;
4. resume after the third await and complete.

If an awaited task is already complete, that await does not cause another suspension or another later invocation.

## AsyncTaskMethodBuilder

An async method returning `Task<T>` exposes a Task to its caller immediately. The compiler-generated builder manages that Task.

Conceptually:

```csharp
builder.SetResult(result);
builder.SetException(exception);
```

The builder completes the caller-visible Task with a result, exception, or cancellation outcome.

## Starting Independent Operations Before Awaiting

Calling an async method generally starts the operation. `await` does not start it.

Sequential initiation:

```csharp
Customer customer = await GetCustomerAsync(id);
Orders orders = await GetOrdersAsync(id);
```

The orders operation starts only after the customer operation completes.

Concurrent initiation for independent operations:

```csharp
Task<Customer> customerTask = GetCustomerAsync(id);
Task<Orders> ordersTask = GetOrdersAsync(id);

await Task.WhenAll(customerTask, ordersTask);

Customer customer = await customerTask;
Orders orders = await ordersTask;
```

The I/O waits can overlap. This is appropriate only when the operations are truly independent and the downstream systems can tolerate the extra concurrency.

## SynchronizationContext

A `SynchronizationContext` provides a policy for where continuations should execute.

Examples:

- WPF and WinForms use a UI context because controls have thread affinity.
- Classic ASP.NET used a request context.
- ASP.NET Core intentionally has no request `SynchronizationContext`.

Server-side request code normally has no requirement to resume on a particular physical thread. In ASP.NET Core, continuations can generally run on any available ThreadPool thread, improving throughput and avoiding unnecessary scheduling constraints.

## ConfigureAwait

The default await behavior captures an applicable context or scheduler when one exists.

```csharp
await operation.ConfigureAwait(false);
```

This says that the continuation does not require the captured context.

In ordinary ASP.NET Core application code, `ConfigureAwait(false)` usually has no practical behavioral effect because there is no request `SynchronizationContext` to capture.

It remains useful in reusable libraries that may be consumed from UI or other context-sensitive environments.

`ConfigureAwait(true)` does not guarantee resumption on the same physical thread. It requests continuation through the captured context when applicable.

## Result, Wait, and Deadlocks

Blocking on async code is dangerous:

```csharp
var result = GetCustomerAsync().Result;
GetCustomerAsync().Wait();
```

In WPF, WinForms, or classic ASP.NET, a deadlock can occur when:

1. the original context thread blocks on `.Result` or `.Wait()`;
2. the async operation completes;
3. its continuation attempts to return to the captured context;
4. that context thread is still blocked waiting for the async method to finish.

The preferred solution is async all the way:

```csharp
var result = await GetCustomerAsync();
```

ASP.NET Core normally avoids this specific context deadlock because it has no request `SynchronizationContext`. Blocking remains harmful because it occupies ThreadPool threads and can contribute to starvation.

## TaskScheduler

A `TaskScheduler` decides where scheduled Task work executes.

`Task.Run` normally uses `TaskScheduler.Default`, which schedules work on the ThreadPool.

```text
Task.Run delegate
    ↓
TaskScheduler.Default
    ↓
ThreadPool thread
```

This is distinct from asynchronous I/O. `HttpClient.GetAsync` does not need a dedicated ThreadPool thread to wait for the network response.

A useful distinction:

- `TaskScheduler` schedules Task work, particularly explicitly scheduled CPU work.
- `SynchronizationContext` provides continuation affinity or dispatch policy.
- `await` uses awaiter machinery and the applicable context/scheduler behavior to arrange continuation execution.

## Staff Interview Summary

> A Task represents an operation, not a thread. For asynchronous I/O, the operation is initiated and the OS handles the wait without a worker thread remaining blocked. If an awaited task is incomplete, the compiler-generated state machine stores the execution point, required locals, awaiter, and Task builder, then returns the thread for other work. When the operation completes, the state machine's `MoveNext()` runs again on a suitable thread and resumes from the saved state. UI frameworks may marshal continuations through a SynchronizationContext, whereas ASP.NET Core has no request SynchronizationContext and typically resumes on any available ThreadPool thread.
