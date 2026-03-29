# Binary Search

**LeetCode #704 · Difficulty: Easy · Category: Binary Search**

---

## Problem Statement

Given an array of integers `nums` which is sorted in ascending order, and an integer `target`, write a function to search `target` in `nums`. If `target` exists, then return its index. Otherwise, return `-1`.

You must write an algorithm with `O(log n)` runtime complexity.

**Example 1:**
```
Input:  nums = [-1,0,3,5,9,12], target = 9
Output: 4
Explanation: 9 exists in nums and its index is 4
```

**Example 2:**
```
Input:  nums = [-1,0,3,5,9,12], target = 2
Output: -1
Explanation: 2 does not exist in nums so return -1
```

**Constraints:**
- `1 <= nums.length <= 10^4`
- `-10^4 < nums[i], target < 10^4`
- All the integers in `nums` are unique
- `nums` is sorted in ascending order

---

## Clarifying Questions

Before writing a single line of code, ask these in an interview:

- **Is the array guaranteed to be sorted?** Yes, ascending order is guaranteed by the constraints.
- **Can the array contain duplicates?** No, all integers are unique per the constraints.
- **Can the array be empty?** The constraints say `nums.length >= 1`, so no. But confirm anyway.
- **What should be returned if the target is not found?** Return `-1`.
- **Can the target be outside the range of values in the array?** Yes. If it is smaller than `nums[0]` or larger than `nums[-1]`, return `-1`.
- **Why does the problem require O(log n)?** This signals that a linear scan is not acceptable. You must exploit the sorted property to eliminate half the search space on each step.

---

## What the Problem Is Really Asking

A linear scan through a sorted array works, but it ignores the most important property: the array is sorted. If you look at the middle element and the target is larger, you know with certainty that the target cannot be in the left half. You can discard it entirely. The same logic applies in reverse. Each comparison eliminates half the remaining search space.

This is binary search. For an array of one billion elements, it finds the answer in at most 30 comparisons. A linear scan would take up to one billion.

The challenge with binary search is not the idea, it is getting the implementation exactly right. Off-by-one errors in the pointer updates and loop condition are the most common source of bugs.

---

## Pseudocode

### Brute Force (Linear Search)

```
FOR each index i from 0 to end of array:
    IF nums[i] == target:
        RETURN i

RETURN -1
```

### Optimal (Binary Search)

```
SET left = 0
SET right = len(nums) - 1

WHILE left <= right:
    SET mid = left + (right - left) // 2

    IF nums[mid] == target:
        RETURN mid
    ELSE IF nums[mid] < target:
        SET left = mid + 1      # target is in the right half
    ELSE:
        SET right = mid - 1     # target is in the left half

RETURN -1
```

---

## Brute Force Solution

**Idea:** Scan every element from left to right. Return the index when the target is found.

```python
class Solution(object):
    def search(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        for i in range(len(nums)):
            if nums[i] == target:
                return i
        return -1
```

This completely ignores the fact that the array is sorted. The problem statement explicitly requires O(log n), so this approach fails the stated constraint. It is shown here only to illustrate what we are improving upon.

**Time Complexity:** O(n) — every element may be visited in the worst case.  
**Space Complexity:** O(1) — no extra storage.

---

## Optimal Solution — Binary Search

**Idea:** Maintain a search window defined by `left` and `right` pointers. At each step, examine the midpoint. Depending on whether the midpoint value is less than, greater than, or equal to the target, either return the index or shrink the window by half.

```python
class Solution(object):
    def search(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        left, right = 0, len(nums) - 1

        while left <= right:
            # Safe midpoint calculation — avoids integer overflow in other languages
            mid = left + (right - left) // 2

            if nums[mid] == target:
                return mid
            elif nums[mid] < target:
                left = mid + 1   # target must be in the right half
            else:
                right = mid - 1  # target must be in the left half

        return -1
```

**Time Complexity:** O(log n) — the search space halves on every iteration.  
**Space Complexity:** O(1) — only three integer variables regardless of input size.

---

## Why `left + (right - left) // 2` Instead of `(left + right) // 2`

Both formulas compute the midpoint. In Python, integers have arbitrary precision and cannot overflow, so either works. However, in languages like Java and C++ where integers are 32-bit, `left + right` can overflow when both values are large. Using `left + (right - left) // 2` avoids this entirely and is considered the safe, portable midpoint formula. Mentioning this in an interview demonstrates awareness beyond the immediate problem.

---

## Step-by-Step Trace

Input: `nums = [-1, 0, 3, 5, 9, 12]`, `target = 9`

```
Initial: left=0, right=5

Iteration 1:
  mid = 0 + (5 - 0) // 2 = 2
  nums[2] = 3
  3 < 9, so left = mid + 1 = 3
  Window: [5, 9, 12]

Iteration 2:
  mid = 3 + (5 - 3) // 2 = 4
  nums[4] = 9
  9 == 9, return 4 ✓
```

Input: `nums = [-1, 0, 3, 5, 9, 12]`, `target = 2`

```
Initial: left=0, right=5

Iteration 1:
  mid = 2, nums[2] = 3
  3 > 2, so right = mid - 1 = 1
  Window: [-1, 0]

Iteration 2:
  mid = 0 + (1 - 0) // 2 = 0
  nums[0] = -1
  -1 < 2, so left = mid + 1 = 1
  Window: [0]

Iteration 3:
  mid = 1 + (1 - 1) // 2 = 1
  nums[1] = 0
  0 < 2, so left = mid + 1 = 2

left (2) > right (1), loop ends.
Return -1 ✓
```

---

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Linear Search | O(n) | O(1) | Ignores sorted property. Fails the stated O(log n) requirement. |
| Binary Search | O(log n) | O(1) | Search space halves each iteration. For n = 10^4, at most 14 iterations. |

To understand how dramatic the difference is: for an array of one billion elements, linear search takes up to one billion steps. Binary search takes at most 30.

---

## Edge Cases

| Input | Expected | Why |
|-------|----------|-----|
| `nums = [5], target = 5` | `0` | Single element, target found. |
| `nums = [5], target = 3` | `-1` | Single element, target not present. |
| `nums = [-1,0,3,5,9,12], target = -1` | `0` | Target is the first element. |
| `nums = [-1,0,3,5,9,12], target = 12` | `5` | Target is the last element. |
| `nums = [-1,0,3,5,9,12], target = 13` | `-1` | Target is larger than all elements. |
| `nums = [-1,0,3,5,9,12], target = -2` | `-1` | Target is smaller than all elements. |

---

## Common Mistakes

**Mistake 1 — Using `left < right` instead of `left <= right` as the loop condition:**  
With strict less-than, the loop exits when `left == right`, which means you never check that one remaining element. This causes missed-target bugs when the target is the last element left in the search window. Always use `left <= right`.

**Mistake 2 — Setting `left = mid` instead of `left = mid + 1`:**  
If `left` equals `right` and `nums[mid]` is less than the target, setting `left = mid` means `left` never advances. The loop runs forever. Always move past the midpoint: `left = mid + 1` and `right = mid - 1`.

**Mistake 3 — Initialising `right = len(nums)` instead of `len(nums) - 1`:**  
`len(nums)` is one past the last valid index. With `right = len(nums)`, the first midpoint calculation may produce an out-of-bounds index when the array has an even number of elements. Always initialise `right` to the last valid index.

**Mistake 4 — Using `(left + right) // 2` without thinking about overflow:**  
Correct in Python, but a bug in Java and C++ for large inputs. Build the habit of writing `left + (right - left) // 2` so the pattern carries over cleanly to other languages.

---

## Follow-Up Questions

**Q: How do you find the first and last position of a target in a sorted array with duplicates? (LC #34)**  
Run binary search twice. For the leftmost occurrence, when `nums[mid] == target` set `right = mid - 1` and record `mid`. For the rightmost, when `nums[mid] == target` set `left = mid + 1` and record `mid`. Two passes, still O(log n).

**Q: How do you search in a rotated sorted array? (LC #33)**  
At each midpoint, one half of the current window is always sorted. Determine which half is sorted using the boundary values, then check whether the target falls within that sorted half. If yes, search there. If no, search the other half. Still O(log n).

**Q: Can binary search be applied when there is no explicit array?**  
Yes. Any problem where the answer space is monotonic (if value X is valid, all values above or below X are also valid) can use binary search on the answer range rather than an array. Problems like Koko Eating Bananas (LC #875) and Capacity to Ship Packages (LC #1011) use this technique.

---

## Test It Yourself

```python
class Solution(object):
    def search(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        left, right = 0, len(nums) - 1

        while left <= right:
            mid = left + (right - left) // 2

            if nums[mid] == target:
                return mid
            elif nums[mid] < target:
                left = mid + 1
            else:
                right = mid - 1

        return -1


# ==================== Test Cases ====================
sol = Solution()

print(sol.search([-1, 0, 3, 5, 9, 12], 9))   # Expected: 4
print(sol.search([-1, 0, 3, 5, 9, 12], 2))   # Expected: -1
print(sol.search([5], 5))                     # Expected: 0
print(sol.search([5], 3))                     # Expected: -1
print(sol.search([-1, 0, 3, 5, 9, 12], -1))  # Expected: 0  (first element)
print(sol.search([-1, 0, 3, 5, 9, 12], 12))  # Expected: 5  (last element)
print(sol.search([-1, 0, 3, 5, 9, 12], 13))  # Expected: -1 (larger than all)
print(sol.search([-1, 0, 3, 5, 9, 12], -2))  # Expected: -1 (smaller than all)
```

---

## Key Takeaways

- Binary search requires three things: a sorted structure, a way to determine which half to discard, and correct pointer updates that always move past the midpoint.
- The loop condition is `left <= right`. The pointer updates are `left = mid + 1` and `right = mid - 1`. These two details are where most implementations go wrong.
- `left + (right - left) // 2` is the safe midpoint formula. Use it consistently across all binary search problems.
- Memorise this template. Every binary search variant (rotated array, first/last position, search insert position, binary search on answer) is a modification of these same ten lines. The skeleton never changes.

---

*Google Easy Track · Tutorial 07 of 08*
