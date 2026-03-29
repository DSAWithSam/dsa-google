# Two Sum

**LeetCode #1 · Difficulty: Easy · Category: Arrays & Hash Maps**

---

## Problem Statement

Given an array of integers `nums` and an integer `target`, return indices of the two numbers such that they add up to `target`.

You may assume that each input would have exactly one solution, and you may not use the same element twice. You can return the answer in any order.

**Example 1:**
```
Input:  nums = [2, 7, 11, 15], target = 9
Output: [0, 1]
Explanation: nums[0] + nums[1] = 2 + 7 = 9
```

**Example 2:**
```
Input:  nums = [3, 2, 4], target = 6
Output: [1, 2]
```

**Example 3:**
```
Input:  nums = [3, 3], target = 6
Output: [0, 1]
```

**Constraints:**
- `2 <= nums.length <= 10^4`
- `-10^9 <= nums[i] <= 10^9`
- `-10^9 <= target <= 10^9`
- Only one valid answer exists.

---

## Clarifying Questions

Before writing a single line of code, ask these in an interview:

- **Can the array contain negative numbers?** Yes, the constraints allow values down to `-10^9`.
- **Can it contain duplicates?** Yes, Example 3 shows `[3, 3]`.
- **Is the array sorted?** No, assume unsorted.
- **Will there always be exactly one solution?** Yes, guaranteed by the constraints.
- **Can I use the same element twice?** No, you must return two *different* indices.

---

## What the Problem Is Really Asking

For every number in the array, you need to find another number that adds up to `target`. That second number is called the **complement**: `complement = target - current_number`.

The challenge is finding that complement *efficiently*. That's the whole problem.

---

## Pseudocode

### Brute Force

```
FOR each index i from 0 to end of array:
    FOR each index j from i+1 to end of array:
        IF nums[i] + nums[j] == target:
            RETURN [i, j]
```

### Optimal

```
CREATE an empty hash map               # stores { number: index }

FOR each index i, number num in array:
    SET complement = target - num

    IF complement exists in hash map:
        RETURN [hash_map[complement], i]

    ELSE:
        STORE num → i in hash map
```

**Key insight:** Instead of searching the array for the complement on every iteration, we *remember* every number we've already seen using a hash map. Hash map lookups are O(1) average, so we reduce a nested loop to a single pass.

---

## Brute Force Solution

**Idea:** Check every possible pair of indices. If they add up to `target`, return them.

```python
class Solution(object):
    def twoSum(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        for i in range(len(nums)):
            for j in range(i + 1, len(nums)):
                if nums[i] + nums[j] == target:
                    return [i, j]
```

> **Why `j = i + 1`?**  
> Starting `j` at `i + 1` ensures we never use the same element twice and never check the same pair in reverse.

**Time Complexity:** O(n²) — for each of n elements, we scan up to n others.  
**Space Complexity:** O(1) — no extra data structures.

---

## Optimal Solution — Hash Map (One Pass)

**Idea:** For each number, calculate its complement and check whether we've already seen it. If yes, we're done. If no, store the current number for future lookups.

```python
class Solution(object):
    def twoSum(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        seen = {}  # maps number -> its index

        for i, num in enumerate(nums):
            complement = target - num

            if complement in seen:
                return [seen[complement], i]

            seen[num] = i
```

> **Why check *before* storing?**  
> We check if the complement exists *before* adding the current number to `seen`. This naturally prevents returning the same index twice. For example, with `nums = [4, 4]` and `target = 8`: on the first iteration, `seen` is empty so `4` is not found — we store `{4: 0}`. On the second iteration, we find `4` in `seen` and return `[0, 1]`. Correct. If we stored first, we'd find the number itself on the same iteration.

---

## Step-by-Step Trace

Input: `nums = [2, 7, 11, 15]`, `target = 9`

| Step | `i` | `num` | `complement` | `seen` before check | Action |
|------|-----|-------|--------------|---------------------|--------|
| 1 | 0 | 2 | 7 | `{}` | 7 not found → store `{2: 0}` |
| 2 | 1 | 7 | 2 | `{2: 0}` | 2 **found** at index 0 → return `[0, 1]` ✓ |

---

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force | O(n²) | O(1) | Nested loops. No extra storage. |
| Hash Map (Optimal) | O(n) | O(n) | Single pass. Hash map stores up to n entries. |

**The tradeoff:** We trade space for speed. The hash map uses O(n) extra memory, but cuts time from O(n²) to O(n). At Google scale, a slow algorithm is a production incident, the extra memory is worth it.

> **O(1) average for hash map lookups:** In the absolute worst case (all keys hash to the same bucket), lookup degrades to O(n). In practice, Python's `dict` uses a well-distributed hash function and this edge case is academic.

---

## Edge Cases

| Input | Expected Output | Why |
|-------|----------------|-----|
| `nums = [3, 3], target = 6` | `[0, 1]` | Duplicate values, both indices must be returned |
| `nums = [-1, -2, -3, -4, -5], target = -8` | `[2, 4]` | Negative numbers work fine |
| `nums = [0, 4, 3, 0], target = 0` | `[0, 3]` | Zero as a value and/or target |
| `nums = [2, 7], target = 9` | `[0, 1]` | Minimum length array (length 2) |

---

## Common Mistakes

**Mistake 1 — Starting `j` at `0` instead of `i + 1` in the brute force:**  
You'd compare `nums[0] + nums[0]`, using the same element twice. Always start the inner loop one step ahead of `i`.

**Mistake 2 — Storing before checking in the optimal solution:**  
If you call `seen[num] = i` before checking for the complement, you risk matching a number with itself when `num * 2 == target`. Always check first, store second.

**Mistake 3 — Returning the values instead of the indices:**  
The problem asks for *indices*, not the numbers themselves. Returning `[2, 7]` instead of `[0, 1]` is wrong.

---

## Test It Yourself

```python
class Solution(object):
    def twoSum(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        seen = {}
        for i, num in enumerate(nums):
            complement = target - num
            if complement in seen:
                return [seen[complement], i]
            seen[num] = i


# ==================== Test Cases ====================
sol = Solution()

print(sol.twoSum([2, 7, 11, 15], 9))          # Expected: [0, 1]
print(sol.twoSum([3, 2, 4], 6))               # Expected: [1, 2]
print(sol.twoSum([3, 3], 6))                  # Expected: [0, 1]
print(sol.twoSum([-1, -2, -3, -4, -5], -8))   # Expected: [2, 4]
print(sol.twoSum([0, 4, 3, 0], 0))            # Expected: [0, 3]
```

---

## Key Takeaways

- When you need to find a *complement* or *pair* in an array, a hash map is usually the right tool.
- The pattern `complement = target - num` → check map → store current is reusable across many problems.
- Always check before storing to avoid self-matching.
- This problem is the foundation for Three Sum (LC #15), Four Sum (LC #18), and many other pair-finding variants.

---

*Google Easy Track · Tutorial 01 of 08*
