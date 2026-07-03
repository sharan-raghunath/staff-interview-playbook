# Find Pivot Index

LeetCode: 724

Pattern: Prefix Sum / Running Sum

## Problem

Given an integer array, return the leftmost pivot index.

A pivot index is an index where:

```text
sum(left side) == sum(right side)
```

The pivot element itself is not included in either sum.

If no pivot exists, return `-1`.

## Clarifications

- Empty left or right side has sum `0`.
- Negative numbers are allowed.
- Duplicate numbers are allowed.
- Return the leftmost pivot if multiple exist.
- A single-element array returns `0`.

## Brute Force

For every index:

1. Sum all elements to the left.
2. Sum all elements to the right.
3. Compare.

Complexity:

- Time: O(n²)
- Space: O(1)

## Optimization Insight

The brute-force approach recalculates almost the same sums repeatedly.

Instead:

1. Compute the total sum once.
2. Maintain a running `leftSum`.
3. At index `i`, compute:

```text
rightSum = totalSum - leftSum - nums[i]
```

If `leftSum == rightSum`, return `i`.

After checking index `i`, update:

```text
leftSum += nums[i]
```

## C# Solution

```csharp
public class Solution
{
    public int PivotIndex(int[] nums)
    {
        int totalSum = 0;
        int leftSum = 0;

        for (int i = 0; i < nums.Length; i++)
        {
            totalSum += nums[i];
        }

        for (int i = 0; i < nums.Length; i++)
        {
            int rightSum = totalSum - leftSum - nums[i];

            if (leftSum == rightSum)
            {
                return i;
            }

            leftSum += nums[i];
        }

        return -1;
    }
}
```

## Complexity

- Time: O(n)
- Space: O(1)

## Interview Explanation

> The brute-force approach recalculates left and right sums for every index, giving O(n²). I can optimize by computing the total sum once and maintaining a running left sum. For each index, the right sum is total minus left sum minus the current value. If left and right match, I return that index. Since I scan left to right, the first match is the leftmost pivot.

## Common Mistakes

- Including the pivot value in either left or right sum.
- Updating `leftSum` before checking the current index.
- Forgetting that the left side of index `0` and right side of the last index are `0`.
- Returning the rightmost pivot instead of the leftmost.
