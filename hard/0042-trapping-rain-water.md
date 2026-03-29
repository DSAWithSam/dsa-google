# Trapping Rain Water

**LeetCode #42 · Difficulty: Hard · Category: Arrays & Two Pointers**

---

## A few things worth noting about this one:

This problem has three meaningfully distinct solutions rather than the usual two, so all three are included with their own code blocks and complexity rows in the table. The prefix array approach sits between brute force and optimal because it is the most direct translation of the water formula and makes the two-pointer logic much easier to understand in contrast. The "Why the Two-Pointer Logic Is Correct" section is its own block because this is genuinely the hardest part of the problem to justify. Most tutorials show the code without explaining why it works. The step-by-step trace uses Example 2 rather than Example 1 because it is shorter and easier to follow without losing any meaningful cases.

## Problem Statement

Given `n` non-negative integers representing an elevation map where the width of each bar is `1`, compute how much water it can trap after raining.

**Example 1:**
```
Input:  height = [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6

Elevation map (# = bar, ~ = trapped water):

     #
     #~#
  #~~###~#
  ########~##
0 1 0 2 1 0 1 3 2 1 2 1

Water trapped at each index: [0,0,1,0,1,2,1,0,0,1,0,0] = 6
```

**Example 2:**
```
Input:  height = [4,2,0,3,2,5]
Output: 9
```

**Constraints:**
- `n == height.length`
- `1 <= n <= 2 * 10^4`
- `0 <= height[i] <= 10^5`

---

## Clarifying Questions

Before writing a single line of code, ask these in an interview:

- **Can heights be zero?** Yes, `height[i] >= 0` per the constraints. A height of zero means no bar at that position.
- **Can the array be empty or have only one element?** The constraints say `n >= 1`, but with fewer than 3 elements no water can be trapped. Your solution should return `0` for these cases.
- **Is width always 1?** Yes, the problem states each bar has width 1.
- **Are all values non-negative?** Yes, guaranteed by the constraints.
- **What is the maximum possible answer?** With `n = 20000` bars each up to height `10^5`, the theoretical maximum is large but fits comfortably in a Python integer.

---

## What the Problem Is Really Asking

At any given index `i`, the amount of water that can sit above it is determined by the shorter of the tallest bar to its left and the tallest bar to its right, minus the height of the bar at `i` itself:

```
water[i] = max(0, min(max_left[i], max_right[i]) - height[i])
```

If the shorter surrounding wall is still taller than the current bar, water pools up to that wall's height. If the current bar is taller than either surrounding wall, no water sits above it (the formula gives a negative result, clamped to 0).

The total water is the sum of `water[i]` across all indices. The approaches differ only in how efficiently they compute `max_left[i]` and `max_right[i]` for each position.

---

## Pseudocode

### Brute Force

```
SET total = 0

FOR each index i from 0 to n-1:
    SET max_left  = max of height[0..i]
    SET max_right = max of height[i..n-1]
    total += max(0, min(max_left, max_right) - height[i])

RETURN total
```

### Prefix Array Approach

```
BUILD max_left[i]  = max height from index 0 to i (left to right pass)
BUILD max_right[i] = max height from index i to n-1 (right to left pass)

SET total = 0
FOR each index i:
    total += max(0, min(max_left[i], max_right[i]) - height[i])

RETURN total
```

### Optimal (Two Pointers)

```
SET left = 0, right = n - 1
SET max_left = 0, max_right = 0
SET total = 0

WHILE left < right:
    IF height[left] <= height[right]:
        IF height[left] >= max_left:
            max_left = height[left]
        ELSE:
            total += max_left - height[left]
        left += 1
    ELSE:
        IF height[right] >= max_right:
            max_right = height[right]
        ELSE:
            total += max_right - height[right]
        right -= 1

RETURN total
```

---

## Brute Force Solution

**Idea:** For each bar, scan left to find the tallest bar to its left, scan right to find the tallest bar to its right, then compute water at that position.

```python
class Solution(object):
    def trap(self, height):
        """
        :type height: List[int]
        :rtype: int
        """
        n = len(height)
        total = 0

        for i in range(n):
            max_left  = max(height[:i+1])
            max_right = max(height[i:])
            total += min(max_left, max_right) - height[i]

        return total
```

For each of n positions, we scan up to n elements to compute the left and right maximums. The water formula always produces a non-negative result here because `max_left >= height[i]` and `max_right >= height[i]` by definition.

**Time Complexity:** O(n²) — two scans per position.  
**Space Complexity:** O(1) — no extra storage beyond scalars.

---

## Better Approach — Prefix Arrays

**Idea:** Precompute the running maximum from the left and from the right in two separate passes, then combine them in a third pass. This trades space for time.

```python
class Solution(object):
    def trap(self, height):
        """
        :type height: List[int]
        :rtype: int
        """
        n = len(height)
        if n == 0:
            return 0

        max_left  = [0] * n
        max_right = [0] * n

        # Left pass: max_left[i] = tallest bar from index 0 to i
        max_left[0] = height[0]
        for i in range(1, n):
            max_left[i] = max(max_left[i-1], height[i])

        # Right pass: max_right[i] = tallest bar from index i to n-1
        max_right[n-1] = height[n-1]
        for i in range(n-2, -1, -1):
            max_right[i] = max(max_right[i+1], height[i])

        # Combine
        total = 0
        for i in range(n):
            total += min(max_left[i], max_right[i]) - height[i]

        return total
```

**Time Complexity:** O(n) — three separate linear passes.  
**Space Complexity:** O(n) — two arrays of length n for the precomputed maximums.

---

## Optimal Solution — Two Pointers

**Idea:** Eliminate the prefix arrays entirely. Use two inward-moving pointers and maintain running maximums as you go. The key observation is: if the left maximum is less than or equal to the right maximum, then the water above the left pointer is fully determined by `max_left` alone, regardless of what is to the right. You do not need to know the exact right maximum because you already know it is at least as tall as `max_left`. The same logic applies symmetrically from the right.

```python
class Solution(object):
    def trap(self, height):
        """
        :type height: List[int]
        :rtype: int
        """
        left, right = 0, len(height) - 1
        max_left, max_right = 0, 0
        total = 0

        while left < right:
            if height[left] <= height[right]:
                if height[left] >= max_left:
                    max_left = height[left]   # update the running max
                else:
                    total += max_left - height[left]   # water above left pointer
                left += 1
            else:
                if height[right] >= max_right:
                    max_right = height[right]   # update the running max
                else:
                    total += max_right - height[right]   # water above right pointer
                right -= 1

        return total
```

**Time Complexity:** O(n) — single pass, each element visited at most once.  
**Space Complexity:** O(1) — four scalar variables regardless of input size.

---

## Why the Two-Pointer Logic Is Correct

The two-pointer solution is the hardest part of this problem to reason about. Here is the justification:

When `height[left] <= height[right]`, we know:
- The right side has a wall at least as tall as `height[left]`
- Therefore the water above `left` is bounded by `max_left`, not `max_right`
- We can safely compute `max_left - height[left]` without knowing the exact right boundary

When `height[left] > height[right]`, the symmetric argument applies from the right side.

In both cases, the pointer on the side with the shorter current height moves inward. This is because the shorter side is the bottleneck: even if the other side were arbitrarily tall, the water level at the current position is constrained by the shorter side's maximum.

---

## Step-by-Step Trace (Two Pointers)

Input: `height = [4, 2, 0, 3, 2, 5]`

```
left=0, right=5, max_left=0, max_right=0, total=0

Step 1: height[0]=4, height[5]=5. 4 <= 5, process left.
  height[0]=4 >= max_left=0 -> max_left=4
  left=1

Step 2: height[1]=2, height[5]=5. 2 <= 5, process left.
  height[1]=2 < max_left=4 -> total += 4-2 = 2. total=2
  left=2

Step 3: height[2]=0, height[5]=5. 0 <= 5, process left.
  height[2]=0 < max_left=4 -> total += 4-0 = 4. total=6
  left=3

Step 4: height[3]=3, height[5]=5. 3 <= 5, process left.
  height[3]=3 < max_left=4 -> total += 4-3 = 1. total=7
  left=4

Step 5: height[4]=2, height[5]=5. 2 <= 5, process left.
  height[4]=2 < max_left=4 -> total += 4-2 = 2. total=9
  left=5

left (5) == right (5), loop ends.
Return 9 ✓
```

---

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force | O(n²) | O(1) | Recomputes left/right max for each position. |
| Prefix Arrays | O(n) | O(n) | Three passes. Two extra arrays of length n. |
| Two Pointers (Optimal) | O(n) | O(1) | Single pass. Four scalars. Best in class. |

---

## Edge Cases

| Input | Expected | Why |
|-------|----------|-----|
| `[1]` | `0` | Single bar. No water possible. |
| `[1, 2]` | `0` | Two bars. No enclosed space. |
| `[3, 0, 3]` | `3` | Symmetric walls, single valley. |
| `[0, 0, 0]` | `0` | No bars, nothing to trap water. |
| `[1, 0, 1]` | `1` | Minimum valid trapping scenario. |
| `[5, 4, 3, 2, 1]` | `0` | Strictly decreasing. No trapping. |
| `[1, 2, 3, 4, 5]` | `0` | Strictly increasing. No trapping. |
| `[4, 2, 0, 3, 2, 5]` | `9` | Example 2 from the problem. |

---

## Common Mistakes

**Mistake 1 — Clamping water to zero in the brute force:**  
The formula `min(max_left, max_right) - height[i]` is always non-negative because `max_left >= height[i]` and `max_right >= height[i]` by construction. However, in the two-pointer version you are computing running maximums that are always at least as tall as the current bar when water is added, so the subtraction is also always non-negative. Being explicit with `max(0, ...)` is safe but unnecessary here if the logic is correct.

**Mistake 2 — Moving both pointers inward when heights are equal:**  
In the two-pointer solution, when `height[left] == height[right]` it does not matter which pointer moves. The condition `height[left] <= height[right]` correctly handles this by consistently moving the left pointer. Moving both simultaneously skips elements and produces incorrect results.

**Mistake 3 — Not updating `max_left` or `max_right` before computing water:**  
The running maximum must be updated first. If `height[left] >= max_left`, the new bar sets the wall height and contributes no water. If you try to compute water before this check, you may subtract a stale maximum from a taller bar and get a negative result.

**Mistake 4 — Using the wrong termination condition:**  
The loop runs while `left < right`, not `left <= right`. When the pointers meet, there is no enclosed valley between them and processing stops. Using `<=` causes an off-by-one that can double-count the centre element.

**Mistake 5 — Forgetting to handle the empty array:**  
The constraints say `n >= 1`, but `height = []` will cause `len(height) - 1 = -1` for the right pointer, breaking the loop immediately and returning `0` correctly. However, explicitly checking `if not height: return 0` at the start is cleaner and communicates intent.

---

## Follow-Up Questions

**Q: How would you solve the 3D version of this problem (Trapping Rain Water II, LC #407)?**  
The 3D version uses a min-heap (priority queue) instead of two pointers. Start from all border cells, always process the cell with the smallest height first, and propagate inward. The water above each interior cell is determined by the minimum height seen along any path from that cell to the border.

**Q: Can you solve this using a monotonic stack?**  
Yes. Maintain a stack of indices with decreasing heights. When a taller bar is encountered, pop bars off the stack and compute the horizontal water trapped between the current bar and the new stack top. This is O(n) time and O(n) space and produces the correct answer through a different decomposition.

**Q: What if bar widths are not uniform?**  
The water calculation at each position scales with the width. Multiply `(min(max_left, max_right) - height[i])` by the width of bar `i`. The overall approach is unchanged.

---

## Test It Yourself

```python
class Solution(object):
    def trap(self, height):
        """
        :type height: List[int]
        :rtype: int
        """
        left, right = 0, len(height) - 1
        max_left, max_right = 0, 0
        total = 0

        while left < right:
            if height[left] <= height[right]:
                if height[left] >= max_left:
                    max_left = height[left]
                else:
                    total += max_left - height[left]
                left += 1
            else:
                if height[right] >= max_right:
                    max_right = height[right]
                else:
                    total += max_right - height[right]
                right -= 1

        return total


# ==================== Test Cases ====================
sol = Solution()

print(sol.trap([0,1,0,2,1,0,1,3,2,1,2,1]))  # Expected: 6
print(sol.trap([4,2,0,3,2,5]))               # Expected: 9
print(sol.trap([1]))                         # Expected: 0
print(sol.trap([1,2]))                       # Expected: 0
print(sol.trap([3,0,3]))                     # Expected: 3
print(sol.trap([1,0,1]))                     # Expected: 1
print(sol.trap([5,4,3,2,1]))                 # Expected: 0
print(sol.trap([1,2,3,4,5]))                 # Expected: 0
print(sol.trap([0,0,0]))                     # Expected: 0
```

---

## Key Takeaways

- Water above any bar is `min(max_left, max_right) - height[i]`. Everything else is an optimisation of how you compute those two maximums.
- The prefix array approach achieves O(n) time at the cost of O(n) space by precomputing the maximums.
- The two-pointer approach achieves O(n) time and O(1) space by observing that when one side's maximum is the bottleneck, you can safely process that side without knowing the exact height of the other.
- This problem is worth understanding through all three approaches. Each one teaches a distinct pattern: brute force for correctness, prefix arrays for the direct translation of the formula, and two pointers for the space-optimal insight.
- The two-pointer technique from this problem generalises to Container With Most Water (LC #11) and other problems involving decisions based on boundary heights.

---

*Google Hard Track · Tutorial 02 of 05*
