# Coding Patterns

This file tracks patterns, not just problems.

## Completed Patterns

| Pattern | Representative Problem | Day | Status |
|---------|------------------------|-----|--------|
| Hash Lookup | Two Sum | Day 1 | ✅ |
| Running Minimum | Best Time to Buy and Sell Stock | Day 2 | ✅ |
| Frequency Counting | Valid Anagram | Day 3 | ✅ |
| Prefix / Suffix | Product of Array Except Self | Day 4 | ✅ |
| Two Pointers | Valid Palindrome | Day 5 | ✅ |
| Fixed-size Sliding Window | Maximum Average Subarray I | Day 6 | ✅ |
| Prefix Sum / Running Sum | Find Pivot Index | Day 7 | ✅ |

---

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

---

## Hash Lookup

Representative problem: Two Sum.

Recognition clues:

- Need to find a complement.
- Brute force repeatedly searches previous/future values.
- Previously seen values matter.

Core idea:

> Trade space for time by storing values in a hash map for O(1) average lookup.

Complexity:

- Time: O(n)
- Space: O(n)

---

## Running Minimum

Representative problem: Best Time to Buy and Sell Stock.

Recognition clues:

- Need best value seen so far.
- Ordering matters.
- Brute force compares all pairs.

Core idea:

> Track the minimum price seen so far and compute best profit at each position.

Complexity:

- Time: O(n)
- Space: O(1)

---

## Frequency Counting

Representative problem: Valid Anagram.

Recognition clues:

- Count occurrences.
- Compare frequencies.
- Words like anagram, duplicate, occurrence, frequency.

Core idea:

> Replace repeated searching with count storage.

Complexity:

- Time: O(n)
- Space: O(1) for fixed alphabet, O(k) otherwise

---

## Prefix / Suffix Computation

Representative problem: Product of Array Except Self.

Recognition clues:

- Answer at index depends on everything before and after it.
- Repeated range computation.
- Division may be disallowed.

Core idea:

> Precompute cumulative information from the left and right, and combine them.

Complexity:

- Time: O(n)
- Space: O(1) extra if output array is reused

---

## Two Pointers

Representative problem: Valid Palindrome.

Recognition clues:

- Work can be done from both ends.
- Need to compare pairs.
- Need to avoid extra memory.
- Sorted array problems.

Core idea:

> Maintain two pointers and move them based on problem-specific rules.

Complexity:

- Time: O(n)
- Space: O(1)

---

## Fixed-Size Sliding Window

Representative problem: Maximum Average Subarray I.

Recognition clues:

- Contiguous subarray or substring.
- Fixed size `k`.
- Adjacent windows overlap heavily.
- Brute force recalculates almost the same range.

Core idea:

> Maintain the state of the current window. When the window moves, remove the outgoing element and add the incoming element.

Complexity:

- Time: O(n)
- Space: O(1)


---

## Prefix Sum / Running Sum

Representative problem: Find Pivot Index.

Recognition clues:

- Need to compare sums or counts on the left and right of an index.
- Repeated range-sum calculation appears in brute force.
- Problem asks about balance, equilibrium, pivot, cumulative sum or range sum.

Core idea:

> Maintain cumulative state so that each index can be evaluated without recomputing previous ranges.

For Pivot Index:

```text
rightSum = totalSum - leftSum - nums[i]
```

Then after checking index `i`:

```text
leftSum += nums[i]
```

Complexity:

- Time: O(n)
- Space: O(1)

Common mistakes:

- Updating the left sum before evaluating the current index.
- Including the pivot element in either side.
- Forgetting that empty sides sum to 0.
