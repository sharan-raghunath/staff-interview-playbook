# Stack Fundamentals

A stack follows **last in, first out (LIFO)** ordering.

Use a stack when the most recently seen unfinished item must be handled first.

Common signals:

- nested brackets or scopes;
- expression parsing;
- undo/redo history;
- depth-first traversal;
- monotonic-stack problems;
- recursive structure simulated iteratively.

Core operations:

```text
Push  -> add to the top
Peek  -> inspect the top without removing it
Pop   -> remove the top
```

In C#, prefer safe combined operations when available:

```csharp
stack.TryPeek(out char current);
stack.TryPop(out char current);
```

If code peeks and immediately pops, ask whether `TryPop` expresses the intent more directly.

## Interview reasoning

A stack is appropriate when correctness depends on reverse-order completion.

For nested delimiters:

> The last opening delimiter must be the first one closed, which is exactly LIFO behavior.

## Complexity

For standard stack operations:

```text
Push: O(1)
Pop:  O(1)
Peek: O(1)
```

A scan that pushes and pops each input item at most once is typically O(n) time and O(n) worst-case space.
