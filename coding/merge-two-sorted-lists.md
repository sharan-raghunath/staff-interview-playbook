# Merge Two Sorted Lists

## Problem

Given the heads of two sorted singly linked lists, merge them into one sorted linked list and return the head of the merged list.

Example:

```text
1 → 2 → 4
1 → 3 → 4
```

Result:

```text
1 → 1 → 2 → 3 → 4 → 4
```

## Pattern

Linked List Pointer Manipulation / Two-list Merge.

## Core Idea

Because both input lists are already sorted, no sorting is needed.

Maintain:

```text
list1 → current node in first list
list2 → current node in second list
tail  → last node in merged list
```

At each step:

1. Compare `list1.val` and `list2.val`.
2. Attach the smaller node to `tail.next`.
3. Advance only the list whose node was chosen.
4. Advance `tail`.

When one list becomes null, attach the remaining part of the other list because it is already sorted.

## Why Use a Dummy Node?

A dummy head avoids special handling for the first inserted node.

```text
dummy → merged-list-start
```

The dummy node is not part of the result.

Therefore return:

```csharp
return dummy.next;
```

## C# Solution

```csharp
public class Solution {
    public ListNode MergeTwoLists(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode(0);
        ListNode tail = dummy;

        while (list1 != null && list2 != null)
        {
            if (list1.val <= list2.val)
            {
                tail.next = list1;
                list1 = list1.next;
            }
            else
            {
                tail.next = list2;
                list2 = list2.next;
            }

            tail = tail.next;
        }

        if (list1 != null)
        {
            tail.next = list1;
        }
        else
        {
            tail.next = list2;
        }

        return dummy.next;
    }
}
```

## Complexity

```text
Time: O(n + m)
Space: O(1)
```

where `n` and `m` are the lengths of the two lists.

## Interview Explanation

> Since both lists are already sorted, I keep one pointer on each list and repeatedly attach the smaller current node to the merged-list tail. I use a dummy head so the first insertion and later insertions follow the same logic. Once one list is exhausted, I attach the rest of the other list because it is already sorted. The dummy node is not part of the answer, so I return `dummy.next`.
