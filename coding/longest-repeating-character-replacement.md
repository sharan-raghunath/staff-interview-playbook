# Longest Repeating Character Replacement

LeetCode 424 — covered in Session 14.

## Problem

Given an uppercase English string and integer `k`, return the longest substring that can be made of one repeated character after replacing at most `k` characters.

## Final approach

Use a variable-size sliding window with a 26-entry frequency array.

For a window:

```text
required replacements = window length - highest character frequency
```

Shrink from the left while required replacements exceed `k`.

```csharp
public class Solution
{
    public int CharacterReplacement(string s, int k)
    {
        int left = 0;
        int maxLength = 0;
        int maxFrequency = 0;
        int[] frequencies = new int[26];

        for (int right = 0; right < s.Length; right++)
        {
            int index = s[right] - 'A';
            frequencies[index]++;
            maxFrequency = Math.Max(maxFrequency, frequencies[index]);

            while ((right - left + 1) - maxFrequency > k)
            {
                frequencies[s[left] - 'A']--;
                left++;
            }

            maxLength = Math.Max(maxLength, right - left + 1);
        }

        return maxLength;
    }
}
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

## Coaching accuracy note

The final implementation is correct, but the assistant disclosed the optimal validity formula, tracking fields and shrinking structure too early. Therefore this problem is recorded as **completed with excessive guidance**, not independently derived.

The agreed process for future problems is brute force → complexity → bottleneck → hint ladder → optimized solution → code.
