# Median of Two Sorted Arrays

**LeetCode #4 · Difficulty: Hard · Category: Binary Search**

---

## Problem Statement

Given two sorted arrays `nums1` and `nums2` of size `m` and `n` respectively, return the median of the two sorted arrays.

The overall run time complexity should be `O(log(m+n))`.

**Example 1:**
```
Input:  nums1 = [1,3], nums2 = [2]
Output: 2.00000
Explanation: merged array = [1,2,3] and median is 2
```

**Example 2:**
```
Input:  nums1 = [1,2], nums2 = [3,4]
Output: 2.50000
Explanation: merged array = [1,2,3,4] and median is (2 + 3) / 2 = 2.5
```

**Constraints:**
- `nums1.length == m`
- `nums2.length == n`
- `0 <= m <= 1000`
- `0 <= n <= 1000`
- `1 <= m + n <= 2000`
- `-10^6 <= nums1[i], nums2[i] <= 10^6`

---

## Clarifying Questions

Before writing a single line of code, ask these in an interview:

- **Can either array be empty?** Yes, `m` and `n` can both be `0`, though `m + n >= 1` is guaranteed. Your solution must handle one empty array cleanly.
- **Are the arrays already sorted?** Yes, both are sorted in non-decreasing order per the problem statement.
- **Can there be duplicates within or across arrays?** Yes. The constraints do not exclude duplicates.
- **What precision is expected for the output?** LeetCode accepts answers within `10^-5` of the true answer. Returning a Python float is sufficient.
- **Why does the problem require O(log(m+n))?** This rules out the O(m+n) merge approach and requires a binary search strategy. The constraint is telling you the algorithm class to reach for.
- **Can values be negative?** Yes, values range from `-10^6` to `10^6`.

---

## What the Problem Is Really Asking

The median of a combined sorted array of total length `L` is:
- The middle element if `L` is odd
- The average of the two middle elements if `L` is even

The naive approach merges both arrays and picks the middle. That is O(m+n) time. The problem requires O(log(m+n)), which means you cannot look at every element.

The key insight is this: you do not need to find the median by constructing the merged array. You need to find a **partition point** in each array such that:

1. The left halves of both arrays together contain exactly the correct number of elements for the left side of the merged array.
2. Every element on the left side is less than or equal to every element on the right side.

Once you find the correct partition, the median can be read directly from the four boundary values without constructing anything.

This is a binary search problem on the partition index of the smaller array.

---

## Understanding the Partition Approach

Imagine cutting both arrays into a left half and a right half:

```
nums1: [... nums1_left ... | ... nums1_right ...]
nums2: [... nums2_left ... | ... nums2_right ...]
```

For this to represent a valid split of the combined sorted array:

1. The total number of elements on the left must equal `(m + n + 1) // 2`.
2. `max(nums1_left) <= min(nums2_right)`
3. `max(nums2_left) <= min(nums1_right)`

If both conditions hold, the partition is correct and the median is:
- `max(left side)` if the total length is odd
- `(max(left side) + min(right side)) / 2` if the total length is even

You binary search on the partition index of `nums1` (the smaller array). For each candidate partition of `nums1`, the partition of `nums2` is determined automatically since the total left count is fixed. You then check the two inequality conditions and adjust the search window.

---

## Pseudocode

### Brute Force (Merge and Find)

```
MERGE nums1 and nums2 into a single sorted array
total = m + n

IF total is odd:
    RETURN merged[total // 2]
ELSE:
    RETURN (merged[total // 2 - 1] + merged[total // 2]) / 2
```

### Optimal (Binary Search on Partition)

```
ENSURE nums1 is the shorter array (swap if needed)

SET left = 0, right = m (length of shorter array)
SET half = (m + n + 1) // 2

WHILE left <= right:
    partition1 = (left + right) // 2       # elements taken from nums1
    partition2 = half - partition1          # elements taken from nums2

    # Boundary values around the partition
    max_left1  = nums1[partition1 - 1] if partition1 > 0 else -infinity
    min_right1 = nums1[partition1]     if partition1 < m else +infinity
    max_left2  = nums2[partition2 - 1] if partition2 > 0 else -infinity
    min_right2 = nums2[partition2]     if partition2 < n else +infinity

    IF max_left1 <= min_right2 AND max_left2 <= min_right1:
        # Correct partition found
        IF (m + n) is odd:
            RETURN max(max_left1, max_left2)
        ELSE:
            RETURN (max(max_left1, max_left2) + min(min_right1, min_right2)) / 2

    ELSE IF max_left1 > min_right2:
        right = partition1 - 1   # too many elements from nums1, move left

    ELSE:
        left = partition1 + 1    # too few elements from nums1, move right
```

---

## Brute Force Solution

**Idea:** Merge both sorted arrays using the standard two-pointer merge, then pick the median from the combined array.

```python
class Solution(object):
    def findMedianSortedArrays(self, nums1, nums2):
        """
        :type nums1: List[int]
        :type nums2: List[int]
        :rtype: float
        """
        merged = []
        i, j = 0, 0

        while i < len(nums1) and j < len(nums2):
            if nums1[i] <= nums2[j]:
                merged.append(nums1[i])
                i += 1
            else:
                merged.append(nums2[j])
                j += 1

        while i < len(nums1):
            merged.append(nums1[i])
            i += 1

        while j < len(nums2):
            merged.append(nums2[j])
            j += 1

        total = len(merged)
        mid = total // 2

        if total % 2 == 1:
            return float(merged[mid])
        else:
            return (merged[mid - 1] + merged[mid]) / 2.0
```

This is clean and correct, but it touches every element in both arrays. O(m+n) time and O(m+n) space for the merged array. The problem explicitly requires O(log(m+n)), so this will not satisfy the constraint in an interview.

**Time Complexity:** O(m+n)  
**Space Complexity:** O(m+n)

---

## Optimal Solution — Binary Search on Partition

**Idea:** Binary search on the number of elements to take from the smaller array. For each candidate, check whether the partition boundaries satisfy the sorted invariant. Adjust the search window based on which boundary condition fails.

```python
class Solution(object):
    def findMedianSortedArrays(self, nums1, nums2):
        """
        :type nums1: List[int]
        :type nums2: List[int]
        :rtype: float
        """
        # Always binary search on the shorter array
        if len(nums1) > len(nums2):
            nums1, nums2 = nums2, nums1

        m, n = len(nums1), len(nums2)
        half = (m + n + 1) // 2

        left, right = 0, m

        while left <= right:
            partition1 = (left + right) // 2
            partition2 = half - partition1

            # Left and right boundary values around the partition
            max_left1  = nums1[partition1 - 1] if partition1 > 0 else float('-inf')
            min_right1 = nums1[partition1]     if partition1 < m else float('inf')
            max_left2  = nums2[partition2 - 1] if partition2 > 0 else float('-inf')
            min_right2 = nums2[partition2]     if partition2 < n else float('inf')

            if max_left1 <= min_right2 and max_left2 <= min_right1:
                # Valid partition found
                if (m + n) % 2 == 1:
                    return float(max(max_left1, max_left2))
                else:
                    return (max(max_left1, max_left2) + min(min_right1, min_right2)) / 2.0

            elif max_left1 > min_right2:
                # Too many elements from nums1, move partition left
                right = partition1 - 1
            else:
                # Too few elements from nums1, move partition right
                left = partition1 + 1
```

**Time Complexity:** O(log(min(m, n))) — binary search runs on the shorter array only.  
**Space Complexity:** O(1) — only a fixed number of scalar variables.

---

## Why Binary Search on the Shorter Array

The search space is `[0, m]` where `m` is the length of `nums1`. By ensuring `nums1` is always the shorter array, the search space is `O(log(min(m, n)))` rather than `O(log(max(m, n)))`. This is a minor optimisation but it is also required for correctness: `partition2 = half - partition1` must always be a valid index into `nums2`, which is only guaranteed when `nums1` is the shorter array.

---

## Step-by-Step Trace

Input: `nums1 = [1, 2]`, `nums2 = [3, 4]`

```
m=2, n=2, half=(2+2+1)//2=2
left=0, right=2

Iteration 1:
  partition1 = (0+2)//2 = 1
  partition2 = 2 - 1 = 1
  max_left1  = nums1[0] = 1
  min_right1 = nums1[1] = 2
  max_left2  = nums2[0] = 3
  min_right2 = nums2[1] = 4

  Check: max_left1 (1) <= min_right2 (4)? YES
  Check: max_left2 (3) <= min_right1 (2)? NO  (3 > 2)

  max_left2 > min_right1, so left = partition1 + 1 = 2

Iteration 2:
  partition1 = (2+2)//2 = 2
  partition2 = 2 - 2 = 0
  max_left1  = nums1[1] = 2
  min_right1 = +inf     (partition1 == m)
  max_left2  = -inf     (partition2 == 0)
  min_right2 = nums2[0] = 3

  Check: max_left1 (2) <= min_right2 (3)? YES
  Check: max_left2 (-inf) <= min_right1 (+inf)? YES

  Valid partition found.
  (m+n) = 4 (even)
  max_left  = max(2, -inf) = 2
  min_right = min(+inf, 3) = 3
  Return (2 + 3) / 2.0 = 2.5 ✓
```

Input: `nums1 = [1, 3]`, `nums2 = [2]`

```
m=2, n=1, half=(2+1+1)//2=2
left=0, right=2

Iteration 1:
  partition1 = 1, partition2 = 1
  max_left1=1, min_right1=3, max_left2=2, min_right2=+inf

  Check: 1 <= +inf? YES
  Check: 2 <= 3?   YES

  Valid partition.
  (m+n) = 3 (odd)
  Return float(max(1, 2)) = 2.0 ✓
```

---

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force (merge) | O(m+n) | O(m+n) | Touches every element. Fails the stated constraint. |
| Binary Search on Partition | O(log(min(m,n))) | O(1) | Search runs only on the shorter array. Constant extra space. |

---

## Edge Cases

| Input | Expected | Why |
|-------|----------|-----|
| `nums1=[], nums2=[1]` | `1.0` | One empty array. Median is the only element. |
| `nums1=[], nums2=[1,2]` | `1.5` | One empty array, even total. |
| `nums1=[2], nums2=[]` | `2.0` | Other array is empty. |
| `nums1=[1,2], nums2=[3,4]` | `2.5` | Even total, median spans both arrays. |
| `nums1=[1,3], nums2=[2]` | `2.0` | Odd total, median from nums2. |
| `nums1=[1,2,3], nums2=[4,5,6]` | `3.5` | No interleaving, boundary falls between arrays. |
| `nums1=[-1000000], nums2=[1000000]` | `0.0` | Extreme values, even total. |

---

## Common Mistakes

**Mistake 1 — Not ensuring nums1 is the shorter array:**  
`partition2 = half - partition1` can produce a negative index or an index exceeding `n` if `nums1` is longer than `nums2`. Always swap so that binary search runs on the shorter array. This single check prevents out-of-bounds errors on all edge case inputs.

**Mistake 2 — Using `float('inf')` and `float('-inf')` inconsistently:**  
When `partition1 == 0`, there are no elements to the left of the cut in `nums1`, so `max_left1` should be `-inf` (it contributes nothing to the max). When `partition1 == m`, there are no elements to the right, so `min_right1` should be `+inf`. Getting these backwards produces incorrect comparisons at the array boundaries.

**Mistake 3 — Integer division producing the wrong half size:**  
Use `(m + n + 1) // 2` for the left half size, not `(m + n) // 2`. The `+ 1` ensures that when the total length is odd, the extra element goes to the left side. This makes the odd-length median calculation consistent: you always return `max(max_left1, max_left2)`.

**Mistake 4 — Returning an integer instead of a float:**  
For the even-length case, `(a + b) / 2` in Python 2 performs integer division. Use `/ 2.0` explicitly or convert with `float()`. LeetCode's Python 2 environment will silently truncate integer division otherwise.

**Mistake 5 — Attempting to binary search on the larger array:**  
If `partition2` ever goes out of bounds for `nums2`, the solution crashes. This only happens when `nums1` is longer than `nums2`. The swap at the start of the function prevents this entirely.

---

## Follow-Up Questions

**Q: What if there are more than two arrays?**  
The binary search on partition approach does not extend directly beyond two arrays. The general approach is to merge pairs of arrays repeatedly, each merge taking O(log(min(m,n))). Alternatively, use a min-heap of size k to process k sorted arrays simultaneously.

**Q: What if the arrays are not sorted?**  
Sort them first at O(m log m + n log n), then apply the binary search partition. The overall complexity is dominated by the sort.

**Q: Can you solve this with a different binary search formulation?**  
Yes. An alternative approach binary searches for the k-th smallest element across both arrays, where k is the index of the median. Each step eliminates k//2 elements from one of the arrays. This approach is arguably easier to reason about but slightly harder to implement cleanly.

---

## Test It Yourself

```python
class Solution(object):
    def findMedianSortedArrays(self, nums1, nums2):
        """
        :type nums1: List[int]
        :type nums2: List[int]
        :rtype: float
        """
        if len(nums1) > len(nums2):
            nums1, nums2 = nums2, nums1

        m, n = len(nums1), len(nums2)
        half = (m + n + 1) // 2
        left, right = 0, m

        while left <= right:
            partition1 = (left + right) // 2
            partition2 = half - partition1

            max_left1  = nums1[partition1 - 1] if partition1 > 0 else float('-inf')
            min_right1 = nums1[partition1]     if partition1 < m else float('inf')
            max_left2  = nums2[partition2 - 1] if partition2 > 0 else float('-inf')
            min_right2 = nums2[partition2]     if partition2 < n else float('inf')

            if max_left1 <= min_right2 and max_left2 <= min_right1:
                if (m + n) % 2 == 1:
                    return float(max(max_left1, max_left2))
                else:
                    return (max(max_left1, max_left2) + min(min_right1, min_right2)) / 2.0
            elif max_left1 > min_right2:
                right = partition1 - 1
            else:
                left = partition1 + 1


# ==================== Test Cases ====================
sol = Solution()

print(sol.findMedianSortedArrays([1, 3], [2]))            # Expected: 2.0
print(sol.findMedianSortedArrays([1, 2], [3, 4]))         # Expected: 2.5
print(sol.findMedianSortedArrays([], [1]))                # Expected: 1.0
print(sol.findMedianSortedArrays([], [1, 2]))             # Expected: 1.5
print(sol.findMedianSortedArrays([2], []))                # Expected: 2.0
print(sol.findMedianSortedArrays([1, 2, 3], [4, 5, 6]))   # Expected: 3.5
print(sol.findMedianSortedArrays([0, 0], [0, 0]))         # Expected: 0.0
print(sol.findMedianSortedArrays([-1000000], [1000000]))  # Expected: 0.0
```

---

## Key Takeaways

- The median problem reduces to finding a valid partition across two arrays such that the left halves together form the correct lower portion of the combined sorted array.
- Binary search runs on the shorter array only. The partition of the longer array is derived automatically. This guarantees O(log(min(m,n))) time and prevents out-of-bounds access.
- `float('-inf')` and `float('inf')` act as sentinel values at the array boundaries. They ensure the inequality checks behave correctly when a partition sits at the very start or end of an array.
- `(m + n + 1) // 2` for the left half size is the correct formula for both odd and even total lengths.
- This partition-based binary search is one of the more conceptually demanding problems at the Easy-to-Hard spectrum. Understanding it deeply builds intuition for any problem that involves splitting sorted structures at a boundary.

---

*Google Hard Track · Tutorial 01 of 05*
