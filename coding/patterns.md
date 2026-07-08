# Coding Patterns

This file records patterns that have been learned through solved problems.

## Hash Lookup

Representative problem: Two Sum.

Core idea:

> Store previously seen values so a complement can be checked in constant expected time.

Complexity:

```text
Time: O(n)
Space: O(n)
```

## Running Minimum

Representative problem: Best Time to Buy and Sell Stock.

Core idea:

> Keep the best candidate from the prefix seen so far, then evaluate the current item against it.

Complexity:

```text
Time: O(n)
Space: O(1)
```

## Frequency Counting

Representative problem: Valid Anagram.

Core idea:

> Count occurrences, then compare counts rather than repeatedly searching.

Complexity:

```text
Time: O(n)
Space: O(1) for a fixed alphabet
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

Representative problems:

- Linked List Cycle
- Middle of the Linked List

Recognition clues:

- linked list or sequence traversal;
- need to avoid storing every visited node;
- one traversal can move at a different rate than another.

Core uses:

> In a finite cycle, two pointers moving at different speeds eventually meet. Without a cycle, the fast pointer reaches the end.

> When one pointer moves twice as fast as another from the same head, the slower pointer reaches the middle when the fast pointer reaches or passes the end.

Complexity:

```text
Time: O(n)
Space: O(1)
```
