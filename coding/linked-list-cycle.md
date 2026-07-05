# LeetCode 141 — Linked List Cycle

## Pattern

Fast and slow pointers (Floyd's cycle detection / tortoise and hare).

## Problem

Given the head of a singly linked list, determine whether following `next` pointers eventually revisits a node.

A cycle does not have to point back to the head. Any node may point to a previously visited node.

## Clarifications

- A repeated **value** is not proof of a cycle.
- The same **node reference** appearing again proves a cycle.
- An empty list or a single node whose `next` is `null` has no cycle.

## Brute Force

Keep a `HashSet<ListNode>` of visited node references.

```text
Visit node
  ↓
Already in set? → cycle exists
  ↓
next is null? → no cycle
```

Complexity:

```text
Time: O(n)
Space: O(n)
```

## Optimized Approach

Use two pointers:

```text
slow moves one node at a time
fast moves two nodes at a time
```

No cycle:

```text
fast reaches null
```

Cycle:

```text
both pointers enter the finite cycle
fast gains one position relative to slow per iteration
therefore fast eventually meets slow
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

## C# Solution

```csharp
public class Solution
{
    public bool HasCycle(ListNode head)
    {
        ListNode slow = head;
        ListNode fast = head;

        while (fast != null && fast.next != null)
        {
            slow = slow.next;
            fast = fast.next.next;

            if (slow == fast)
                return true;
        }

        return false;
    }
}
```

## Common Mistakes

- Storing node values instead of node references.
- Returning the wrong boolean for cycle/no-cycle.
- Writing `while (fast != null)` and hiding null handling through conditional access instead of checking `fast.next` explicitly.
- Calling complexity `O(n/2)` rather than `O(n)`.

## Interview Explanation

> If there is no cycle, the fast pointer reaches the end. If there is a cycle, both pointers eventually enter it. Since fast moves one node ahead of slow relative to slow on every iteration, it must eventually catch slow in the finite cycle.
