# Session 13 — Async/Await Internals and Variable-Size Sliding Window

## Session goals

- Understand `async`/`await` from first principles rather than memorizing rules.
- Explain the compiler-generated async state machine, continuation scheduling and ASP.NET Core behavior.
- Complete LeetCode 3: Longest Substring Without Repeating Characters.
- System design and mock interview were explicitly deferred to Sunday by the user.

## Backend / .NET — async and await internals

### Why asynchronous I/O matters

Synchronous database, network, or file I/O blocks a ThreadPool thread while an external dependency is doing the actual work. The blocked thread consumes resources without performing useful CPU work.

Under load, blocked worker threads can cause queued requests, timeouts and ThreadPool starvation even when CPU utilization is not especially high.

The key distinction learned today was:

> Async I/O improves scalability. It does not make the external I/O operation itself faster.

### Process, thread and Task

- A process is an execution boundary with its own virtual memory and resources.
- A thread is an execution path inside a process with its own stack.
- A Task represents an operation and its eventual completion.

A Task is not a dedicated thread and does not execute itself.

### What happens at await

When an awaited Task is incomplete:

- the method records the state required to continue later;
- its current execution returns;
- the worker thread becomes available for other work;
- the OS/network stack continues the asynchronous I/O wait;
- after completion, a suitable thread runs the continuation.

When the awaited Task is already complete, execution continues synchronously without suspending.

### Compiler-generated state machine

The original stack frame cannot remain attached to a ThreadPool thread while that thread handles unrelated work. The compiler therefore transforms the async method into a state machine that preserves information such as:

- the execution point to resume from;
- local variables required later;
- parameters and instance references required later;
- the awaiter;
- Task completion and exception state.

The state field acts as a bookmark. With multiple awaits, it identifies which suspension point must resume.

### MoveNext

`MoveNext()` contains the transformed method logic. It runs until the method either reaches an incomplete await or completes.

For three awaits that all complete asynchronously, `MoveNext()` is commonly invoked four times: initial execution plus one resumption after each await.

If an awaited Task is already complete, that await does not cause another suspension.

### AsyncTaskMethodBuilder

The builder manages the Task returned to the caller and completes it with the final result, exception or cancellation outcome.

### Independent asynchronous operations

Calling an asynchronous method starts the operation; awaiting it does not start it.

If operations are independent, initiating them before awaiting allows their I/O waits to overlap:

```csharp
Task<Customer> customerTask = GetCustomerAsync(id);
Task<Orders> ordersTask = GetOrdersAsync(id);

await Task.WhenAll(customerTask, ordersTask);
```

This is appropriate only after verifying that there is no data, transactional or business-ordering dependency and that the downstream system can handle the added concurrency.

### SynchronizationContext

A SynchronizationContext describes where a continuation should execute.

- WPF and WinForms marshal continuations back to the UI thread because UI controls have thread affinity.
- Classic ASP.NET had a request context.
- ASP.NET Core has no request SynchronizationContext.

ASP.NET Core continuations can generally run on any available ThreadPool thread because server request code does not normally need physical-thread affinity.

### ConfigureAwait

`ConfigureAwait(false)` tells the awaiter that the continuation does not require the captured context.

In ordinary ASP.NET Core application code it usually has no practical behavioral impact because there is no request SynchronizationContext to capture. It remains useful in reusable libraries that may run inside WPF, WinForms or another context-sensitive host.

`ConfigureAwait(true)` does not guarantee continuation on the same physical thread.

### Result and Wait

Blocking with `.Result` or `.Wait()` can deadlock in UI frameworks and classic ASP.NET when the continuation needs the captured context but the context thread is blocked waiting for the Task.

ASP.NET Core usually avoids that specific deadlock, but blocking still wastes ThreadPool threads and can cause starvation under load.

### TaskScheduler

`TaskScheduler` decides where scheduled Task work executes. `Task.Run` normally uses `TaskScheduler.Default`, backed by the ThreadPool.

This is distinct from asynchronous I/O, where no .NET worker thread needs to remain blocked during the external wait.

## Coding — LeetCode 3: Longest Substring Without Repeating Characters

### Clarified requirements

- Return the length of the longest contiguous substring with no repeated characters.
- Do not normalize case unless the requirement explicitly requests case-insensitive behavior.
- Do not assume the input contains only alphabetic characters.

### Approach

Use a variable-size sliding window with:

- `left` and `right` pointers;
- a `HashSet<char>` for characters in the current window;
- a maintained invariant that the current window contains unique characters.

When the right character is duplicated, remove characters from the left until the duplicate is gone. Then add the right character and update the maximum length.

```csharp
public class Solution
{
    public int LengthOfLongestSubstring(string s)
    {
        int left = 0;
        int maxLength = 0;
        HashSet<char> seen = new HashSet<char>();

        for (int right = 0; right < s.Length; right++)
        {
            while (seen.Contains(s[right]))
            {
                seen.Remove(s[left]);
                left++;
            }

            seen.Add(s[right]);
            maxLength = Math.Max(maxLength, right - left + 1);
        }

        return maxLength;
    }
}
```

Complexity:

```text
Time: O(n)
Space: O(k), O(n) worst case
```

The inner loop does not make the solution quadratic because each character enters and leaves the window at most once.

### Coding reflection

Concentration was lower than usual. The first proposal detected duplicates but did not fully define how the valid window should be preserved after a repetition.

Once the counterexample `abcad` was considered, the sliding-window requirement became clear. The final implementation was functionally correct. It was then simplified so that `maxLength` is updated only after the unique-window invariant has been restored.

The main process improvement is:

> Validate the proposed algorithm against a few edge cases before declaring its complexity.

## Newly learned today

- Async I/O scalability versus I/O speed.
- Task versus Thread versus Process.
- Compiler-generated async state machine and state preservation.
- `MoveNext()`, awaiters and `AsyncTaskMethodBuilder`.
- Await does not always suspend.
- Starting independent operations before awaiting them.
- SynchronizationContext and why ASP.NET Core does not use one for requests.
- `ConfigureAwait(false)` behavior and reusable-library rationale.
- `.Result` / `.Wait()` context deadlocks and ThreadPool starvation.
- TaskScheduler versus SynchronizationContext.
- Variable-size Sliding Window using a HashSet.

## Explicitly deferred

- Session 13 system-design discussion.
- Session 13 mock interview.

The user plans to see whether these can be continued on Sunday. They were not marked complete.

## Session 13 status

Backend theory and coding are complete. System design and mock interview are deferred by explicit user choice.
