# Longest Substring Without Repeating Characters

## Pattern

Variable-size Sliding Window.

## Problem

Given a string `s`, return the length of the longest contiguous substring that contains no repeated characters.

The standard problem is case-sensitive and does not restrict the input to alphabetic characters. Therefore, characters should not be normalized unless the requirements explicitly say so.

## Initial Idea and Important Gap

Tracking whether a character has been seen is useful, but a duplicate cannot always be handled by clearing the complete set and restarting.

For:

```text
abcad
```

when the second `a` appears, the useful window becomes:

```text
bca
```

It then grows to:

```text
bcad
```

A full reset could miss valid substrings that overlap the earlier window.

## Sliding-Window Invariant

Maintain two pointers:

- `left`: beginning of the current valid window;
- `right`: character currently being examined.

Maintain a `HashSet<char>` containing exactly the characters in the current window.

Before updating the answer, preserve this invariant:

> The substring from `left` through `right` contains no duplicate characters.

When `s[right]` is already in the set, remove characters from the left until the duplicate is gone. Then add `s[right]` and calculate the valid window length.

## Final Solution

```csharp
public class Solution
{
    public int LengthOfLongestSubstring(string s)
    {
        int left = 0;
        int maxLength = 0;
        HashSet<char> seen = new HashSet<char>();

        for (int right = 0; right < s.Length; right++)
        {
            while (seen.Contains(s[right]))
            {
                seen.Remove(s[left]);
                left++;
            }

            seen.Add(s[right]);
            maxLength = Math.Max(maxLength, right - left + 1);
        }

        return maxLength;
    }
}
```

## Complexity

- Time: `O(n)`
- Space: `O(k)`, where `k` is the number of distinct characters in the active window; `O(n)` in the worst case

The nested loop does not make the algorithm `O(n²)`. Every character is added to the set once and removed at most once. Both pointers move only forward.

## Interview Insight

Do not declare complexity only from the visible loop nesting. Count how many times each pointer or element can move across the complete algorithm.

A strong explanation is:

> I maintain the longest valid window ending at each right pointer. If the new character is duplicated, I shrink from the left until uniqueness is restored. Each character enters and leaves the window at most once, so the total work is linear.
