# Maximum Average Subarray I

## Pattern

Fixed-size Sliding Window.

## Problem

Given an integer array `nums` and an integer `k`, find the maximum average of any contiguous subarray of length `k`.

## Key Insight

Since `k` is constant, maximizing the average is equivalent to maximizing the sum.

Therefore, maintain `maxSum` and divide once at the end.

## Brute Force

For every starting index, calculate the sum of the next `k` elements.

Complexity:

- Time: O(n × k), O(n²) worst case
- Space: O(1)

## Optimized Approach

Adjacent windows overlap.

Example:

```text
[2, 5, 1] → [5, 1, 8]
```

The new window:

- removes `2`
- adds `8`

So:

```text
newSum = oldSum - outgoing + incoming
```

## Final Solution

```csharp
public class Solution
{
    public double FindMaxAverage(int[] nums, int k)
    {
        int sum = 0;

        for (int i = 0; i < k; i++)
        {
            sum += nums[i];
        }

        int maxSum = sum;

        for (int i = k; i < nums.Length; i++)
        {
            sum = sum - nums[i - k] + nums[i];
            maxSum = Math.Max(maxSum, sum);
        }

        return (double)maxSum / k;
    }
}
```

## Common Bug

The outgoing element is:

```csharp
nums[i - k]
```

not:

```csharp
nums[k - i]
```

`k - i` becomes negative after the first iteration and causes an index error.

## Complexity

- Time: O(n)
- Space: O(1)

## Interview Insight

The key phrase:

> Adjacent windows overlap. I can maintain the current window sum and update it in O(1) as the window slides.
