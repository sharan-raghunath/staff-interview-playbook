# Product of Array Except Self

## Pattern

Prefix / Suffix Computation.

## Problem

For each index, compute the product of all elements except the element at that index.

## Brute Force

For each index, multiply every other element.

Complexity:

- Time: O(n²)
- Space: O(1) excluding output

## Optimized Approach

The result at index `i` is:

```text
product of elements before i × product of elements after i
```

Use prefix and suffix products.

The final optimization reuses the output array:

1. Store prefix products in result.
2. Traverse from right with a running suffix product.
3. Multiply into result.

## Complexity

- Time: O(n)
- Space: O(1) extra excluding output array

## Interview Insight

The breakthrough is recognizing repeated work: each result repeatedly multiplies many of the same values.
