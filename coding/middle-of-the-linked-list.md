# Middle of the Linked List — LeetCode 876

## Problem

Given the head of a singly linked list, return its middle node.

When the list has an even number of nodes, return the second middle node.

```text
1 → 2 → 3 → 4 → 5
return 3

1 → 2 → 3 → 4 → 5 → 6
return 4
```

## Brute Force: Two Passes

1. Count the nodes in a first traversal.
2. Start from the head again and move `n / 2` zero-based hops.
3. Return the reached node.

Using zero-based hops avoids separate odd/even formulas:

```text
n = 5 → 5 / 2 = 2 hops → node 3
n = 6 → 6 / 2 = 3 hops → node 4
```

```text
Time: O(n)
Space: O(1)
```

## Optimal: Fast and Slow Pointers

Start `fast` and `slow` at the head.

- `slow` moves one node each iteration.
- `fast` moves two nodes each iteration.

When `fast` cannot safely advance two nodes, `slow` is at the middle. Since both start at head, `slow` naturally lands on the second middle for an even-length list.

```csharp
public class Solution
{
    public ListNode MiddleNode(ListNode head)
    {
        if (head == null)
            return null;

        ListNode slow = head;
        ListNode fast = head;

        while (fast != null && fast.next != null)
        {
            slow = slow.next;
            fast = fast.next.next;
        }

        return slow;
    }
}
```

```text
Time: O(n)
Space: O(1)
```

## Interview Explanation

> “I use two pointers. The fast pointer moves two nodes for every one node moved by slow. When fast reaches the end for an odd-length list, or passes the end for an even-length list, slow has moved halfway through the list. Starting both at head makes slow land on the second middle node when the length is even.”

## Pattern Connection

This is the second use of fast/slow pointers:

- **Linked List Cycle:** different speeds eventually meet inside a finite cycle.
- **Middle of the Linked List:** different speeds place one pointer at the halfway point.
