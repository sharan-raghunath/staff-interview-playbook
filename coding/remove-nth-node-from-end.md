# Remove Nth Node From End of List

Representative problem: LeetCode 19.

## Problem

Given the head of a singly linked list and an integer `n`, remove the `n`th node from the end and return the resulting head.

## Brute-force reasoning

1. Traverse once to count the nodes.
2. Compute the zero-based index from the front as `length - n`.
3. Traverse again and reconnect the predecessor to the target node's successor.
4. If `n == length`, remove the original head.

```text
Time: O(n)
Space: O(1)
```

## One-pass reasoning

Use a dummy node and two pointers. Build a gap of `n` nodes between the fast and slow positions. Continue traversal without restarting from the head. When the fast pointer reaches the last node, the slow pointer is immediately before the node to remove.

```csharp
public class Solution
{
    public ListNode RemoveNthFromEnd(ListNode head, int n)
    {
        ListNode dummy = new ListNode(0);
        dummy.next = head;

        ListNode slow = dummy;
        ListNode fast = head;
        int count = 1;

        while (fast.next != null)
        {
            fast = fast.next;

            if (count < n)
            {
                count++;
                continue;
            }

            slow = slow.next;
        }

        slow.next = slow.next.next;
        return dummy.next;
    }
}
```

```text
Time: O(n)
Space: O(1)
```

## Interview takeaway

> One pass does not require one loop. It means the traversal is not restarted after first computing the full list length.

The dummy node handles removal of the original head without a separate deletion branch.
