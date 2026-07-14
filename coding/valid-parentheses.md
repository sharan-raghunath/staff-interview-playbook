# Valid Parentheses

## Problem

Given a string containing only:

```text
( ) { } [ ]
```

determine whether every opening bracket is closed by the correct type in the correct order.

The empty string is valid.

## Reasoning

The most recently opened bracket must be closed first. This is a LIFO relationship, so a stack is the natural data structure.

Algorithm:

1. Push each opening bracket.
2. For a closing bracket, pop the latest opening bracket.
3. If the stack is empty or the types do not match, return `false`.
4. After the full scan, the stack must be empty.

## C# solution

```csharp
public class Solution
{
    public bool IsValid(string s)
    {
        Stack<char> openings = new();

        Dictionary<char, char> matchingBrackets = new()
        {
            { ')', '(' },
            { ']', '[' },
            { '}', '{' }
        };

        foreach (char current in s)
        {
            if (!matchingBrackets.TryGetValue(
                    current,
                    out char expectedOpening))
            {
                openings.Push(current);
                continue;
            }

            if (!openings.TryPop(out char actualOpening) ||
                actualOpening != expectedOpening)
            {
                return false;
            }
        }

        return openings.Count == 0;
    }
}
```

## Complexity

```text
Time: O(n)
Space: O(n)
```

Worst-case space occurs when every character is an opening bracket.

## Code-review lessons

- `TryGetValue` avoids separate key lookup and indexing.
- `TryPop` combines empty-stack checking and removal.
- `Count == 0` is clearer than `Any() ? false : true`.
- A dictionary centralizes the supported closing-to-opening mapping.
- Do not generalize bracket types beyond the stated contract unless configurability is a real requirement.

## Interview takeaway

> I use a stack because valid delimiters must close in reverse order of opening. Each character is processed once, and each opening bracket is pushed and popped at most once.
