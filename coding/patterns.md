# Coding Patterns

This file tracks patterns, not just problems.

## Completed Patterns

| Pattern | Representative Problem | Day | Status |
|---|---|---:|---|
| Hash Lookup | Two Sum | Day 1 | ✅ |
| Running Minimum | Best Time to Buy and Sell Stock | Day 2 | ✅ |
| Frequency Counting | Valid Anagram | Day 3 | ✅ |
| Prefix / Suffix | Product of Array Except Self | Day 4 | ✅ |
| Two Pointers | Valid Palindrome | Day 5 | ✅ |
| Fixed-size Sliding Window | Maximum Average Subarray I | Day 6 | ✅ |
| Prefix Sum / Running Sum | Find Pivot Index | Day 7 | ✅ |
| Fast & Slow Pointers | Linked List Cycle | Day 8 | ✅ |

## Core Interview Process

For every coding problem:

1. Clarify the problem.
2. State the brute-force approach.
3. Analyze brute-force complexity.
4. Identify repeated work / bottleneck.
5. Derive the optimized pattern.
6. Code cleanly.
7. Analyze optimized complexity.
8. Discuss edge cases and trade-offs.

## Hash Lookup

Representative problem: Two Sum.

Core idea:

> Trade space for time by storing values in a hash map for O(1) average lookup.

Complexity:

```text
Time: O(n)
Space: O(n)
```

## Running Minimum

Representative problem: Best Time to Buy and Sell Stock.

Core idea:

> Track the minimum price seen so far and compute the best profit at each position.

Complexity:

```text
Time: O(n)
Space: O(1)
```

## Frequency Counting

Representative problem: Valid Anagram.

Core idea:

> Replace repeated searching with count storage.

Complexity:

```text
Time: O(n)
Space: O(1) for a fixed alphabet; O(k) otherwise
```

## Prefix / Suffix Computation

Representative problem: Product of Array Except Self.

Core idea:

> Precompute cumulative information from the left and right, then combine it.

Complexity:

```text
Time: O(n)
Space: O(1) extra when the output array is reused
```

## Two Pointers

Representative problem: Valid Palindrome.

Core idea:

> Maintain two pointers and move them according to problem-specific rules.

Complexity:

```text
Time: O(n)
Space: O(1)
```

## Fixed-Size Sliding Window

Representative problem: Maximum Average Subarray I.

Core idea:

> Maintain current-window state. Remove the outgoing element and add the incoming element when the window moves.

Complexity:

```text
Time: O(n)
Space: O(1)
```

## Prefix Sum / Running Sum

Representative problem: Find Pivot Index.

Core idea:

> Maintain cumulative state so each index can be evaluated without recalculating previous ranges.

For Pivot Index:

```text
rightSum = totalSum - leftSum - nums[i]
```

Then update:

```text
leftSum += nums[i]
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

## Fast & Slow Pointers

Representative problem: Linked List Cycle.

Recognition clues:

- linked list or sequence traversal;
- cycle detection;
- need to avoid storing every visited node;
- one traversal can move at a different rate than another.

Core idea:

> In a finite cycle, two pointers moving at different speeds eventually meet. Without a cycle, the fast pointer reaches the end.

Complexity:

```text
Time: O(n)
Space: O(1)
```
