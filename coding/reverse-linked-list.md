# Reverse Linked List

## Problem

Given the head of a singly linked list, reverse the list and return the new head.

Example:

```text
1 → 2 → 3 → 4 → 5 → null
```

should become:

```text
5 → 4 → 3 → 2 → 1 → null
```

## Pattern

Linked list pointer manipulation.

## Brute Force

A simple extra-memory approach is to store nodes or values in an array/stack, then rebuild or relink in reverse order.

Complexity:

```text
Time: O(n)
Space: O(n)
```

## Iterative Solution

Use three pointers:

```text
prev → already reversed part
curr → node currently being processed
next → saved original curr.next
```

Loop invariant:

> `prev` is the head of the reversed prefix, and `curr` is the first node not yet reversed.

Core step:

```csharp
ListNode next = curr.next;
curr.next = prev;
prev = curr;
curr = next;
```

Full solution:

```csharp
public class Solution {
    public ListNode ReverseList(ListNode head) {
        if(head == null || head.next == null)
        {
            return head;
        }

        ListNode curr = head;
        ListNode prev = null;

        while(curr != null)
        {
            ListNode next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }

        return prev;
    }
}
```

The early return is optional; the loop naturally handles empty and single-node lists.

Complexity:

```text
Time: O(n)
Space: O(1)
```

## Recursive Solution

The recursive version is useful for interviews but is usually less preferred in production because it uses call-stack space.

Base case:

```csharp
if (head == null || head.next == null)
{
    return head;
}
```

Why:

- empty list is already reversed;
- single-node list is already reversed;
- the last node becomes the new head.

Solution:

```csharp
public class Solution {
    public ListNode ReverseList(ListNode head) {
        if (head == null || head.next == null)
        {
            return head;
        }

        ListNode newHead = ReverseList(head.next);

        head.next.next = head;
        head.next = null;

        return newHead;
    }
}
```

Key line:

```csharp
head.next.next = head;
```

Meaning:

> After recursion reverses the rest of the list, `head.next` is the last node of the reversed portion. Pointing `head.next.next` to `head` appends the current node to the end of the reversed list.

Then:

```csharp
head.next = null;
```

breaks the old forward link and prevents a cycle.

Complexity:

```text
Time: O(n)
Space: O(n) due to recursive call stack
```

## Interview Explanation

> I keep `prev` as the reversed part and `curr` as the current node. Before reversing `curr.next`, I save the original next node. Then I point `curr.next` backward to `prev`, move `prev` to `curr`, and move `curr` to the saved next node. When `curr` becomes null, `prev` is the new head.

For recursion:

> The base case is an empty or single-node list. For any other node, I recursively reverse the rest of the list, then point the next node back to the current head using `head.next.next = head`, and set `head.next = null` to make the current head the new tail of that sublist.
