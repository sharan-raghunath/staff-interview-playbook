# Two Sum

## Pattern

Hash Lookup.

## Problem

Given an array of integers and a target, return indices of two numbers that add up to the target.

## Brute Force

Check every pair.

Complexity:

- Time: O(n²)
- Space: O(1)

## Optimized Approach

For each number, compute its complement:

```text
complement = target - current
```

If the complement was seen earlier, return both indices.

Otherwise store the current number and index in a dictionary.

## Complexity

- Time: O(n)
- Space: O(n)

## Interview Insight

This pattern is about replacing repeated searching with constant-time lookup.
