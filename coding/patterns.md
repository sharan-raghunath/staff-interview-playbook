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


## Linked List Pointer Manipulation

Representative problem: Reverse Linked List.

Core idea:

> Preserve the next reference before changing the current node's pointer, then move through the list while maintaining the already-reversed part.

Iterative reversal step:

```text
next = curr.next
curr.next = prev
prev = curr
curr = next
```

Recursive reversal key step:

```text
head.next.next = head
head.next = null
```

Complexity for iterative solution:

```text
Time: O(n)
Space: O(1)
```

Complexity for recursive solution:

```text
Time: O(n)
Space: O(n) due to call stack
```


## Linked List Merge / Dummy Head

Representative problem: Merge Two Sorted Lists.

Core idea:

> Use a dummy head and a tail pointer so the first append and later appends follow the same logic. Compare the current heads of both sorted lists, attach the smaller node, and advance only that source list.

Dummy-head return rule:

```text
return dummy.next
```

The dummy node is not part of the real result.

Complexity:

```text
Time: O(n + m)
Space: O(1)
```


## Fixed-gap Two Pointers

Representative problem: Remove Nth Node From End of List.

Core idea:

> Move one pointer ahead to create a fixed gap, then advance both pointers without restarting traversal. A dummy head lets the slower pointer land immediately before the deletion target, including when the original head must be removed.

Interview clarification:

> One pass does not mean one loop; it means the traversal is not restarted from the head after first computing the full length.

Complexity:

```text
Time: O(n)
Space: O(1)
```
