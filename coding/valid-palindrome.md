# Valid Palindrome

## Pattern

Two Pointers.

## Problem

Determine whether a string is a palindrome after ignoring non-alphanumeric characters and treating case insensitively.

## Brute Force

1. Build a normalized character array.
2. Reverse it.
3. Compare original normalized and reversed arrays.

Complexity:

- Time: O(n)
- Space: O(n)

## Optimized Approach

Use two pointers:

- left starts at beginning
- right starts at end
- skip non-alphanumeric characters
- compare lowercase/invariant characters
- move inward

## Final Solution

```csharp
public class Solution
{
    public bool IsPalindrome(string s)
    {
        int left = 0;
        int right = s.Length - 1;

        while (left < right)
        {
            if (!char.IsLetterOrDigit(s[left]))
            {
                left++;
            }
            else if (!char.IsLetterOrDigit(s[right]))
            {
                right--;
            }
            else
            {
                if (char.ToLowerInvariant(s[left]) != char.ToLowerInvariant(s[right]))
                {
                    return false;
                }

                left++;
                right--;
            }
        }

        return true;
    }
}
```

## Complexity

- Time: O(n)
- Space: O(1)

## Interview Insight

The optimized solution reduces auxiliary space from O(n) to O(1), but pointer logic becomes slightly more complex. In production, clarity is more important than making the code unnecessarily compact.
