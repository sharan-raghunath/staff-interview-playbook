# Best Time to Buy and Sell Stock

## Pattern

Running Minimum.

## Core Idea

At each day, the best selling profit depends on the minimum buying price seen before that day.

Maintain:

- `minPrice`
- `maxProfit`

## Complexity

- Time: O(n)
- Space: O(1)

## Interview Insight

This is not a generic two-loop comparison problem. It is an ordered scan where previous state matters.
