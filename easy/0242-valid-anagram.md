# Valid Anagram

**LeetCode #242 · Difficulty: Easy · Category: Strings & Hash Maps**

---

## Problem Statement

Given two strings `s` and `t`, return `true` if `t` is an anagram of `s`, and `false` otherwise.

An anagram is a word formed by rearranging the letters of another word, using all original letters exactly once.

**Example 1:**
```
Input:  s = "anagram", t = "nagaram"
Output: true
```

**Example 2:**
```
Input:  s = "rat", t = "car"
Output: false
```

**Constraints:**
- `1 <= s.length, t.length <= 5 * 10^4`
- `s` and `t` consist of lowercase English letters

**Follow-up:** What if the inputs contain Unicode characters? How would you adapt your solution?

---

## Clarifying Questions

Before writing a single line of code, ask these in an interview:

- **Are the strings guaranteed to be lowercase only?** Yes, per the constraints. But the follow-up explicitly asks about Unicode, so be ready to address it.
- **Can the strings have different lengths?** Yes. If they do, they cannot be anagrams. This is a free O(1) early exit.
- **Are spaces or special characters possible?** Not per the constraints, but worth confirming. If yes, they count as characters and must be included in the frequency comparison.
- **Is the comparison case-sensitive?** Yes. The constraints only guarantee lowercase, so casing is not a concern here, but worth asking if the problem were expanded.
- **Can either string be empty?** The constraints say `length >= 1`, so no. Two empty strings would technically be anagrams of each other, but that case cannot appear here.

---

## What the Problem Is Really Asking

Two strings are anagrams if and only if they contain exactly the same characters with exactly the same frequencies. The order does not matter, only the counts.

This single insight gives you all three valid approaches:

1. **Sort both strings** and compare them. If they are identical after sorting, they are anagrams.
2. **Count character frequencies** using a hash map and compare the two maps.
3. **Use a single fixed-size array** of 26 slots (one per lowercase letter), incrementing for `s` and decrementing for `t`, then verify all slots are zero.

All three are correct. The interesting question is which approach you reach for first and why.

---

## Pseudocode

### Approach 1 — Sort and Compare

```
IF len(s) != len(t):
    RETURN false

RETURN sorted(s) == sorted(t)
```

### Approach 2 — Hash Map Frequency Count

```
IF len(s) != len(t):
    RETURN false

CREATE an empty hash map

FOR each character in s:
    INCREMENT count for that character

FOR each character in t:
    DECREMENT count for that character

RETURN true if all counts equal zero, false otherwise
```

### Approach 3 — Fixed 26-Slot Array (Optimal for Lowercase)

```
IF len(s) != len(t):
    RETURN false

CREATE an array of 26 zeros

FOR each character pair (a from s, b from t) simultaneously:
    INCREMENT array[ord(a) - ord('a')]
    DECREMENT array[ord(b) - ord('a')]

RETURN true if all values in array are zero
```

---

## Solution 1 — Sort and Compare (Brute Force)

**Idea:** Sort both strings. If they are identical after sorting, every character appeared the same number of times.

```python
class Solution(object):
    def isAnagram(self, s, t):
        """
        :type s: str
        :type t: str
        :rtype: bool
        """
        if len(s) != len(t):
            return False

        return sorted(s) == sorted(t)
```

Readable and concise. The length check short-circuits a large class of inputs for free. However, sorting throws away the fact that character frequencies can be compared directly without rearranging anything.

**Time Complexity:** O(n log n) where n is the length of the strings. The sort dominates.  
**Space Complexity:** O(n) because Python's `sorted()` creates a new list rather than sorting in place.

---

## Solution 2 — Hash Map Frequency Count

**Idea:** Count character frequencies in `s` using a hash map, then verify the same frequencies exist in `t`.

```python
class Solution(object):
    def isAnagram(self, s, t):
        """
        :type s: str
        :type t: str
        :rtype: bool
        """
        if len(s) != len(t):
            return False

        count = {}

        for char in s:
            count[char] = count.get(char, 0) + 1

        for char in t:
            if char not in count:
                return False
            count[char] -= 1
            if count[char] < 0:
                return False

        return True
```

This approach also generalises cleanly to Unicode (see the Follow-Up section below).

**Time Complexity:** O(n) — two separate passes through strings of length n.  
**Space Complexity:** O(k) where k is the number of unique characters. For lowercase only, k is at most 26, which is effectively O(1).

Python's standard library provides `Counter` from `collections`, which makes this even more concise:

```python
from collections import Counter

class Solution(object):
    def isAnagram(self, s, t):
        """
        :type s: str
        :type t: str
        :rtype: bool
        """
        return Counter(s) == Counter(t)
```

`Counter(s) == Counter(t)` compares every key-value pair in both maps. It also handles the length check implicitly. If the lengths differ, the counts cannot be equal.

**Time Complexity:** O(n)  
**Space Complexity:** O(k) — at most 26 keys for lowercase input, effectively O(1).

---

## Solution 3 — Fixed 26-Slot Array (Optimal for Lowercase)

**Idea:** Map each lowercase letter to an index 0 through 25 using `ord(char) - ord('a')`. Increment for each character in `s` and decrement for each character in `t` in a single pass using `zip`. If both strings are anagrams, every slot returns to zero.

```python
class Solution(object):
    def isAnagram(self, s, t):
        """
        :type s: str
        :type t: str
        :rtype: bool
        """
        if len(s) != len(t):
            return False

        count = [0] * 26

        for a, b in zip(s, t):
            count[ord(a) - ord('a')] += 1
            count[ord(b) - ord('a')] -= 1

        return all(x == 0 for x in count)
```

This is strictly O(1) space because the array is always exactly 26 integers regardless of input length. The `zip` loop processes both strings simultaneously in a single pass.

**Time Complexity:** O(n) — one pass through both strings together.  
**Space Complexity:** O(1) — always exactly 26 integers, no hash overhead.

---

## Comparing All Three Approaches

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Sort and compare | O(n log n) | O(n) | Simple to write. Sorting is the bottleneck. |
| Hash map / Counter | O(n) | O(k) | General purpose. Works for any character set. |
| Fixed 26-slot array | O(n) | O(1) | Best for lowercase. Strictly constant space, no hash overhead. |

**Which to present in an interview:** Lead with `Counter` for its clarity and idiomatic Python. Then offer the fixed array as an optimisation, noting it achieves strictly O(1) space and removes hash function overhead. This demonstrates both range and the ability to reason about tradeoffs.

---

## Step-by-Step Trace (Fixed Array)

Input: `s = "rat"`, `t = "car"`

`ord('r') - ord('a') = 17`, `ord('a') - ord('a') = 0`, `ord('t') - ord('a') = 19`, `ord('c') - ord('a') = 2`

| Step | `a` (from s) | `b` (from t) | Index incremented | Index decremented | Relevant slots after |
|------|-------------|-------------|-------------------|-------------------|----------------------|
| 1 | `r` | `c` | 17 | 2 | `r:+1, c:-1` |
| 2 | `a` | `a` | 0 | 0 | `a:0` |
| 3 | `t` | `r` | 19 | 17 | `t:+1, r:0` |

Final array has non-zero values at slots for `c` and `t`. Not all zeros, so return `False`. ✓

Input: `s = "anagram"`, `t = "nagaram"`

After processing all 7 pairs, every slot returns to zero, so return `True`. ✓

---

## Edge Cases

| Input | Expected | Why |
|-------|----------|-----|
| `s = "a", t = "a"` | `true` | Single character, identical. |
| `s = "a", t = "b"` | `false` | Single character, different. |
| `s = "ab", t = "a"` | `false` | Different lengths, immediate false. |
| `s = "aab", t = "baa"` | `true` | Duplicate characters, all counts balance. |
| `s = "aa", t = "bb"` | `false` | Same length, different characters. |

---

## Common Mistakes

**Mistake 1 — Skipping the early length check:**  
Strings of different lengths cannot be anagrams. Checking `len(s) != len(t)` first is O(1) and eliminates a large class of inputs before any character processing happens. While `Counter` handles this implicitly, the explicit check is cleaner and communicates intent.

**Mistake 2 — Assuming `Counter` comparison is O(1):**  
Comparing two `Counter` objects checks every key-value pair. For lowercase input with at most 26 unique keys, this is effectively O(1). For arbitrary Unicode input with many unique characters, it becomes O(k) where k is the number of distinct characters. Know this distinction if asked about complexity.

**Mistake 3 — Using the fixed array approach and claiming it works for Unicode:**  
The 26-slot array only handles `'a'` through `'z'`. If the input could contain digits, punctuation, or Unicode characters, the array approach breaks. For any input beyond lowercase letters, switch to the hash map approach.

**Mistake 4 — Decrementing below zero without catching it:**  
In the manual hash map solution, if a character appears in `t` but not in `s`, you must catch that case. Checking `if char not in count: return False` before decrementing prevents incorrect results for inputs like `s = "ab", t = "cb"`.

---

## Follow-Up: Unicode Characters

The constraints here guarantee lowercase English letters only. But the follow-up asks: what if the input contains Unicode?

The fixed 26-slot array breaks entirely. Unicode has over one million code points, far too many for a fixed array.

Switch to the hash map approach using `Counter` or a standard dictionary. The logic is identical; only the character set is larger.

```python
from collections import Counter

class Solution(object):
    def isAnagram(self, s, t):
        """
        :type s: str
        :type t: str
        :rtype: bool
        """
        return Counter(s) == Counter(t)
```

**Time Complexity:** O(n)  
**Space Complexity:** O(k) where k is the number of unique characters. For Unicode, k can be large, but it is always bounded by the length of the input strings.

This is the answer Google is looking for when they ask the follow-up. Not a new algorithm, just a recognition that the fixed array assumption no longer holds and the hash map generalises cleanly.

---

## Test It Yourself

```python
class Solution(object):
    def isAnagram(self, s, t):
        """
        :type s: str
        :type t: str
        :rtype: bool
        """
        if len(s) != len(t):
            return False

        count = [0] * 26

        for a, b in zip(s, t):
            count[ord(a) - ord('a')] += 1
            count[ord(b) - ord('a')] -= 1

        return all(x == 0 for x in count)


# ==================== Test Cases ====================
sol = Solution()

print(sol.isAnagram("anagram", "nagaram"))  # Expected: True
print(sol.isAnagram("rat", "car"))          # Expected: False
print(sol.isAnagram("a", "a"))              # Expected: True
print(sol.isAnagram("a", "b"))              # Expected: False
print(sol.isAnagram("ab", "a"))             # Expected: False  (different lengths)
print(sol.isAnagram("aab", "baa"))          # Expected: True
print(sol.isAnagram("aa", "bb"))            # Expected: False
print(sol.isAnagram("listen", "silent"))    # Expected: True
```

---

## Key Takeaways

- Two strings are anagrams if and only if their character frequency counts are identical. That one sentence gives you all three approaches.
- Always do the length check first. It is O(1) and eliminates a large class of inputs immediately.
- The fixed 26-slot array gives strictly O(1) space and is the most efficient solution for lowercase-only input. The hash map generalises to any character set.
- `ord(char) - ord('a')` maps `'a'` through `'z'` to indices `0` through `25`. This is a reusable pattern for any problem involving lowercase letter frequencies.
- This frequency counting pattern appears in: Group Anagrams (LC #49), Find All Anagrams in a String (LC #438), and Minimum Window Substring (LC #76).

---

*Google Easy Track · Tutorial 06 of 08*
