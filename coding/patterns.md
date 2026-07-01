# Coding Patterns

## Completed Patterns

| Pattern | Representative Problem |
|---------|------------------------|
| Hash Lookup | Two Sum |
| Running Minimum | Best Time to Buy and Sell Stock |
| Frequency Counting | Valid Anagram |
| Prefix / Suffix | Product of Array Except Self |
| Two Pointers | Valid Palindrome |

---

# Hash Lookup

## Representative Problem

Two Sum

## Recognition Clues

- Need to find complement.
- Repeated searching appears in brute force.
- Previously seen values matter.

## Core Idea

Store useful information in a hash map so that future lookups are O(1) average case.

## Complexity

- Time: O(n)
- Space: O(n)

---

# Running Minimum

## Representative Problem

Best Time to Buy and Sell Stock

## Recognition Clues

- Need best value seen so far.
- Constraint depends on ordering.
- Brute force compares all pairs.

## Core Idea

Track the best previous value while scanning once.

## Complexity

- Time: O(n)
- Space: O(1)

---

# Frequency Counting

## Representative Problem

Valid Anagram

## Recognition Clues

- Count occurrences.
- Compare character or element frequencies.
- Words like anagram, duplicate, frequency, occurrence.

## Core Idea

Replace repeated searching with count storage.

## Complexity

- Time: O(n)
- Space: O(1) for fixed alphabet, O(k) otherwise

---

# Prefix / Suffix Computation

## Representative Problem

Product of Array Except Self

## Recognition Clues

- Answer at index depends on everything before and after it.
- Repeated range computation.
- Division may be disallowed.
- Need to reuse intermediate products/sums.

## Core Idea

Precompute cumulative information from the left and right.

## Complexity

- Time: O(n)
- Space: O(1) extra if output array is reused

---

# Two Pointers

## Representative Problem

Valid Palindrome

## Recognition Clues

- Work can be done from both ends.
- Need to compare pairs.
- Sorted array problems.
- Need to avoid extra memory.
- Need to move inward based on conditions.

## Core Idea

Maintain two pointers and move them based on problem-specific rules.

## Complexity

- Time: O(n)
- Space: O(1)
