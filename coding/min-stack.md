# Min Stack

LeetCode 155.

## Problem

Implement an integer stack supporting:

```text
Push
Pop
Top
GetMin
```

All operations should run in O(1) time.

## Brute-force approach

Use a dynamic array/list for normal stack operations and scan all values during `GetMin`.

```text
Push:   O(1) amortized
Pop:    O(1)
Top:    O(1)
GetMin: O(n)
```

A single global `min` variable is insufficient: when the current minimum is popped, the previous minimum is unknown without rescanning.

## Optimized pattern: auxiliary state per stack level

Store each value together with the minimum at that level:

```text
(value, minAtThisLevel)
```

On push:

```text
newMin = stack empty ? value : min(value, currentMin)
```

On pop, removing the top entry automatically exposes the previous level's minimum.

## C# implementation

```csharp
public class MinStack
{
    private readonly List<(int Value, int Min)> _items = new();

    public void Push(int value)
    {
        int min = _items.Count == 0
            ? value
            : Math.Min(value, _items[^1].Min);

        _items.Add((value, min));
    }

    public void Pop()
    {
        EnsureNotEmpty();
        _items.RemoveAt(_items.Count - 1);
    }

    public int Top()
    {
        EnsureNotEmpty();
        return _items[^1].Value;
    }

    public int GetMin()
    {
        EnsureNotEmpty();
        return _items[^1].Min;
    }

    private void EnsureNotEmpty()
    {
        if (_items.Count == 0)
            throw new InvalidOperationException("Stack is empty.");
    }
}
```

## Complexity

```text
Push:   O(1) amortized
Pop:    O(1)
Top:    O(1)
GetMin: O(1)
Space:  O(n)
```

## Interview takeaway

> If removing the current top destroys information needed after the pop, preserve that information at each stack level.

This is a general pattern: augment each element with the summary state required for the remaining prefix.
