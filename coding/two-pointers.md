# Two Pointers

## Representative Problem

LeetCode 125 — Valid Palindrome

## Problem

Given a string, determine whether it is a palindrome after:

- ignoring non-alphanumeric characters
- treating uppercase and lowercase letters as equal

---

## Brute Force

1. Build a normalized character array.
2. Ignore non-alphanumeric characters.
3. Convert remaining characters to lowercase.
4. Reverse the array.
5. Compare original normalized array with reversed array.

### Complexity

- Time: O(n)
- Space: O(n)

---

## Optimized Approach

Use two pointers directly on the original string.

```text
left = 0
right = n - 1
```

At each step:

1. If left is not alphanumeric, move left.
2. If right is not alphanumeric, move right.
3. Compare both characters case-insensitively.
4. If mismatch, return false.
5. Otherwise move both inward.

Stop when pointers meet or cross.

---

## C# Solution

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
                if (char.ToLowerInvariant(s[left]) !=
                    char.ToLowerInvariant(s[right]))
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

---

## Complexity

- Time: O(n)
- Space: O(1)

Each character is visited at most once by one of the two pointers.

---

## Readability vs Compactness

This compact version is correct:

```csharp
if (char.ToLowerInvariant(s[left++]) !=
    char.ToLowerInvariant(s[right--]))
{
    return false;
}
```

But the clearer version separates comparison from pointer movement.

Staff-level takeaway:

> The shortest code is not always the best code. Once time and space complexity are optimal, readability becomes the deciding factor.

---

## Recognition Clues

Think of Two Pointers when:

- you can compare from both ends
- the input is sorted
- you need pairs
- you want to avoid extra memory
- the brute-force solution compares many pairs unnecessarily

---

## Related Problems

- Two Sum II
- Container With Most Water
- Move Zeroes
- Remove Duplicates from Sorted Array
- 3Sum
- Trapping Rain Water
