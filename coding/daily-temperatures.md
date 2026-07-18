# Daily Temperatures

Representative problem: LeetCode 739.

## Brute force

For each day, scan right until the first warmer day is found. Worst-case time is O(n²).

## Derived insight

Days without an answer are waiting for a future warmer temperature. The most recently unresolved day is checked first, so a stack is natural.

Store **indices**, not temperatures:

- compare with `temperatures[stack.Peek()]`;
- calculate distance with `currentIndex - previousIndex`.

When a warmer day arrives, pop and resolve every smaller temperature at the top. The unresolved indices therefore remain in non-increasing temperature order.

## C# solution

```csharp
public class Solution
{
    public int[] DailyTemperatures(int[] temperatures)
    {
        int[] answer = new int[temperatures.Length];
        Stack<int> stack = new();

        for (int i = 0; i < temperatures.Length; i++)
        {
            while (stack.TryPeek(out int index) &&
                   temperatures[i] > temperatures[index])
            {
                stack.Pop();
                answer[index] = i - index;
            }

            stack.Push(i);
        }

        return answer;
    }
}
```

The output array is zero-initialized, so unresolved days need no final cleanup loop.

## Complexity

Each index is pushed once and popped at most once.

- Time: O(n)
- Space: O(n)
