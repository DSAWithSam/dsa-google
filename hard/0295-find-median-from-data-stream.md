# Find Median from Data Stream

**LeetCode #295 · Difficulty: Hard · Category: Heaps & Design**

---

## A few things worth noting about this one:

The negation trick gets its own dedicated section with a concrete example because it is the single most confusing part of implementing this in Python, and glossing over it causes more bugs than anything else. The step-by-step trace walks through all three operations from Example 1 in full detail, showing both the partition correction and the size rebalancing at each step. The follow-ups address both parts of the problem's own follow-up question directly, which is unusual since LeetCode includes them in the problem statement itself, so they deserve a complete answer rather than a brief mention.

## Problem Statement

The median is the middle value in an ordered integer list. If the size of the list is even, there is no middle value and the median is the mean of the two middle values.

- For `arr = [2,3,4]`, the median is `3`.
- For `arr = [2,3]`, the median is `(2 + 3) / 2 = 2.5`.

Implement the `MedianFinder` class:
- `MedianFinder()` initialises the `MedianFinder` object.
- `void addNum(int num)` adds the integer `num` from the data stream to the data structure.
- `double findMedian()` returns the median of all elements so far. Answers within `10^-5` of the actual answer will be accepted.

**Example 1:**
```
Input:
["MedianFinder", "addNum", "addNum", "findMedian", "addNum", "findMedian"]
[[], [1], [2], [], [3], []]

Output:
[null, null, null, 1.5, null, 2.0]

Explanation:
MedianFinder medianFinder = new MedianFinder()
medianFinder.addNum(1)     # arr = [1]
medianFinder.addNum(2)     # arr = [1, 2]
medianFinder.findMedian()  # return 1.5  (i.e., (1 + 2) / 2)
medianFinder.addNum(3)     # arr = [1, 2, 3]
medianFinder.findMedian()  # return 2.0
```

**Constraints:**
- `-10^5 <= num <= 10^5`
- There will be at least one element before `findMedian` is called
- At most `5 * 10^4` calls will be made to `addNum` and `findMedian`

**Follow-up:**
- If all integers from the stream are in the range `[0, 100]`, how would you optimise?
- If 99% of all integers from the stream are in `[0, 100]`, how would you optimise?

---

## Clarifying Questions

Before writing a single line of code, ask these in an interview:

- **Can numbers be negative?** Yes, values range from `-10^5` to `10^5`.
- **Can duplicate numbers be added?** Yes, duplicates are not excluded by the constraints.
- **Will `findMedian` ever be called on an empty structure?** No, the constraints guarantee at least one element exists before `findMedian` is called.
- **Is the stream finite or infinite?** The problem frames it as a stream, implying potentially infinite input. Your design must not store elements in a way that degrades over time.
- **Does the output need to be exact?** No, answers within `10^-5` are accepted. A Python float is sufficient.

---

## What the Problem Is Really Asking

You need to maintain a running median over a sequence of numbers that arrive one at a time. After each insertion, you must be able to report the current median in O(1) time.

The naive approach keeps a sorted list and finds the middle element. Inserting into a sorted list is O(n), which is too slow for `5 * 10^4` calls.

The key insight is that the median depends on only two things: the largest element of the lower half and the smallest element of the upper half. You do not need to keep the entire sorted sequence accessible. You only need fast access to those two boundary values.

This maps perfectly onto two heaps:
- A **max-heap** containing the lower half of the numbers. Its top is the largest value in the lower half.
- A **min-heap** containing the upper half of the numbers. Its top is the smallest value in the upper half.

If you maintain these two heaps in balance (sizes differ by at most 1), then:
- When total count is odd: the median is the top of the larger heap.
- When total count is even: the median is the average of both tops.

Every `addNum` call is O(log n). Every `findMedian` call is O(1).

---

## The Two-Heap Invariant

At all times, the following must hold:

1. Every element in the max-heap (lower half) is less than or equal to every element in the min-heap (upper half).
2. The sizes of the two heaps differ by at most 1. We allow the max-heap to hold one extra element when the total count is odd.

```
Stream so far: [1, 2, 3, 5, 7]

max_heap (lower half): [3, 2, 1]   top = 3
min_heap (upper half): [5, 7]      top = 5

Total = 5 (odd). Median = max_heap top = 3.
```

```
Stream so far: [1, 2, 3, 5]

max_heap (lower half): [2, 1]   top = 2
min_heap (upper half): [3, 5]   top = 3

Total = 4 (even). Median = (2 + 3) / 2 = 2.5.
```

---

## Pseudocode

### Brute Force

```
MAINTAIN a list of all numbers seen

addNum(num):
    INSERT num into the list (keep sorted or sort on demand)

findMedian():
    SORT the list
    n = length
    IF n is odd:  RETURN list[n // 2]
    IF n is even: RETURN (list[n//2 - 1] + list[n//2]) / 2.0
```

### Optimal (Two Heaps)

```
INITIALISE max_heap = []   (lower half, stored as negatives for Python)
INITIALISE min_heap = []   (upper half)

addNum(num):
    PUSH num onto max_heap (negate for Python's min-heap)
    PUSH the max_heap top onto min_heap (balance the partition)
    IF len(min_heap) > len(max_heap):
        PUSH min_heap top back onto max_heap

findMedian():
    IF len(max_heap) > len(min_heap):
        RETURN float(-max_heap[0])
    ELSE:
        RETURN (-max_heap[0] + min_heap[0]) / 2.0
```

---

## Brute Force Solution

**Idea:** Store all numbers in a list. Sort on every call to `findMedian` and return the middle element.

```python
class MedianFinder(object):
    def __init__(self):
        self.nums = []

    def addNum(self, num):
        """
        :type num: int
        :rtype: None
        """
        self.nums.append(num)

    def findMedian(self):
        """
        :rtype: float
        """
        self.nums.sort()
        n = len(self.nums)
        if n % 2 == 1:
            return float(self.nums[n // 2])
        else:
            return (self.nums[n // 2 - 1] + self.nums[n // 2]) / 2.0
```

Alternatively, you could maintain the list in sorted order by using `bisect.insort` in `addNum`, reducing each insertion to O(log n) for the search but still O(n) for the list shift. Either way, `findMedian` becomes O(1) but `addNum` remains O(n).

**Time Complexity:** O(n log n) per `findMedian` call (sort dominates). O(1) for `addNum`.  
**Space Complexity:** O(n) — all numbers stored.

---

## Optimal Solution — Two Heaps

**Idea:** Maintain two heaps that together represent the sorted sequence. The max-heap holds the lower half and the min-heap holds the upper half. After each insertion, rebalance so their sizes differ by at most 1.

Python's `heapq` module only provides a min-heap. To simulate a max-heap, negate values before pushing and negate again when reading the top.

```python
import heapq

class MedianFinder(object):
    def __init__(self):
        self.max_heap = []   # lower half (stored as negatives)
        self.min_heap = []   # upper half

    def addNum(self, num):
        """
        :type num: int
        :rtype: None
        """
        # Step 1: Push to max_heap (lower half)
        heapq.heappush(self.max_heap, -num)

        # Step 2: Ensure the partition is valid
        # The largest element in the lower half must not exceed
        # the smallest element in the upper half
        if self.min_heap and (-self.max_heap[0] > self.min_heap[0]):
            val = -heapq.heappop(self.max_heap)
            heapq.heappush(self.min_heap, val)

        # Step 3: Rebalance sizes — max_heap may have at most 1 extra element
        if len(self.max_heap) > len(self.min_heap) + 1:
            val = -heapq.heappop(self.max_heap)
            heapq.heappush(self.min_heap, val)
        elif len(self.min_heap) > len(self.max_heap):
            val = heapq.heappop(self.min_heap)
            heapq.heappush(self.max_heap, -val)

    def findMedian(self):
        """
        :rtype: float
        """
        if len(self.max_heap) > len(self.min_heap):
            return float(-self.max_heap[0])
        return (-self.max_heap[0] + self.min_heap[0]) / 2.0
```

**Time Complexity:** O(log n) for `addNum` — each call performs at most three heap operations.  
**Space Complexity:** O(n) — all numbers stored across the two heaps.  
**Time Complexity for `findMedian`:** O(1) — reading the top of a heap is constant time.

---

## Why the Negation Trick Works

Python's `heapq` always maintains a min-heap where the smallest element is at the top. To simulate a max-heap (where the largest element should be at the top), we negate every value before pushing. The smallest negated value corresponds to the largest original value. When we read `max_heap[0]`, we negate it again to recover the true value.

```
True values:     [1, 2, 3]   <- we want max-heap, top should be 3
Stored as:      [-3, -1, -2]  <- min-heap of negatives, top is -3
Read as:        -(-3) = 3    <- correct max-heap top
```

---

## Step-by-Step Trace

Input: `addNum(1)`, `addNum(2)`, `findMedian()`, `addNum(3)`, `findMedian()`

```
Initial: max_heap=[], min_heap=[]

addNum(1):
  Push -1 to max_heap. max_heap=[-1]
  min_heap is empty, no partition check needed.
  Sizes: max=1, min=0. Balanced (max can hold 1 extra). Done.

  State: max_heap=[-1]  (represents [1])
         min_heap=[]

addNum(2):
  Push -2 to max_heap. max_heap=[-2, -1]
  Partition check: -max_heap[0] = 2, min_heap[0] = N/A (empty). Skip.
  Sizes: max=2, min=0. max has 2 more than min (exceeds allowed 1 extra).
  Move top of max_heap to min_heap: pop -2, push 2.
  max_heap=[-1], min_heap=[2]
  Sizes: max=1, min=1. Balanced.

  State: max_heap=[-1]  (lower half: [1])
         min_heap=[2]   (upper half: [2])

findMedian():
  len(max_heap) == len(min_heap) == 1. Even total.
  Return (-(-1) + 2) / 2.0 = (1 + 2) / 2.0 = 1.5 ✓

addNum(3):
  Push -3 to max_heap. max_heap=[-3, -1]
  Partition check: -max_heap[0] = 3 > min_heap[0] = 2. Violation!
  Move 3 from max_heap to min_heap: pop -3, push 3.
  max_heap=[-1], min_heap=[2, 3]
  Sizes: max=1, min=2. min has more than max. Rebalance.
  Move top of min_heap to max_heap: pop 2, push -2.
  max_heap=[-2, -1], min_heap=[3]
  Sizes: max=2, min=1. Balanced (max holds 1 extra for odd total).

  State: max_heap=[-2, -1]  (lower half: [1, 2])
         min_heap=[3]        (upper half: [3])

findMedian():
  len(max_heap)=2 > len(min_heap)=1. Odd total.
  Return float(-max_heap[0]) = float(-(-2)) = 2.0 ✓
```

---

## Complexity Analysis

| Operation | Brute Force | Two Heaps | Notes |
|-----------|-------------|-----------|-------|
| `addNum` | O(1) append | O(log n) | Heap push/pop operations. |
| `findMedian` | O(n log n) | O(1) | Heap top access is constant. |
| Space | O(n) | O(n) | Both store all numbers. |

For `5 * 10^4` calls, the brute force sorts up to 50,000 elements on each `findMedian` call. The two-heap approach handles both operations efficiently regardless of call count.

---

## Edge Cases

| Scenario | Expected Behaviour |
|----------|--------------------|
| Single element added, `findMedian` called | Return that element as a float. |
| Two elements added, `findMedian` called | Return their average. |
| All elements are identical | Median equals that element. |
| Elements added in reverse sorted order | Heaps rebalance correctly regardless of insertion order. |
| Elements added in sorted order | Same as above. |
| Negative numbers mixed with positive | Negation trick handles negatives correctly because the relative ordering is preserved. |

---

## Common Mistakes

**Mistake 1 — Forgetting to negate values when using Python's `heapq` as a max-heap:**  
`heapq` is always a min-heap. Pushing `num` without negating means the "max-heap" actually returns the smallest element, which reverses the entire lower-half logic and produces wrong medians for every input.

**Mistake 2 — Not checking the partition invariant after pushing:**  
After pushing to `max_heap`, the new element might be larger than the top of `min_heap`, violating the rule that every lower-half element must be at most every upper-half element. Always check and correct this before rebalancing sizes.

**Mistake 3 — Rebalancing sizes before checking the partition invariant:**  
The order of operations matters. Correct the partition first (so values are in the right heap), then rebalance sizes (so the heaps have the right counts). Reversing this order can move elements to the wrong heap.

**Mistake 4 — Returning an integer instead of a float from `findMedian`:**  
The return type is `double`. For the even case, `(-max_heap[0] + min_heap[0]) / 2.0` uses `2.0` to force float division in Python 2. Using `/ 2` truncates to an integer. For the odd case, wrapping with `float()` is correct practice.

**Mistake 5 — Allowing `min_heap` to be larger than `max_heap`:**  
The convention here allows `max_heap` to hold one extra element when the total count is odd. Allowing `min_heap` to be larger breaks `findMedian`: the odd-case median should come from `max_heap`, not `min_heap`. Always rebalance so `max_heap` is the "dominant" heap.

---

## Follow-Up Questions

**Q: If all integers are in `[0, 100]`, how would you optimise?**  
Use a counting array of size 101. `count[i]` stores how many times value `i` has been added. `addNum` is O(1) — just increment `count[num]`. `findMedian` walks the count array to find the middle index, which is O(100) = O(1) since the range is fixed. Space is O(1) relative to stream size.

**Q: If 99% of integers are in `[0, 100]` but 1% are outside?**  
Use the counting array for the `[0, 100]` range. Maintain two small sorted structures (or heaps) for outliers below 0 and above 100. `findMedian` combines the total count from all three structures to determine where the median falls, then reads from the appropriate one. This keeps the common case O(1) while handling outliers correctly.

**Q: What if the stream is distributed across multiple machines?**  
Each machine maintains its own two-heap structure over its local partition of the stream. A central coordinator periodically aggregates approximate medians using a merge-sort-style approach or reservoir sampling. Exact distributed medians require more sophisticated algorithms such as the Greenwald-Khanna algorithm for approximate quantiles.

**Q: Can you use a sorted container instead of heaps?**  
Yes. A balanced BST or a sorted list with `bisect.insort` gives O(log n) insertion and O(1) median lookup once you track the size. In Python, `sortedcontainers.SortedList` achieves this. The two-heap approach is preferred in interviews because it uses only standard library tools.

---

## Test It Yourself

```python
import heapq

class MedianFinder(object):
    def __init__(self):
        self.max_heap = []   # lower half (negated)
        self.min_heap = []   # upper half

    def addNum(self, num):
        """
        :type num: int
        :rtype: None
        """
        heapq.heappush(self.max_heap, -num)

        if self.min_heap and (-self.max_heap[0] > self.min_heap[0]):
            val = -heapq.heappop(self.max_heap)
            heapq.heappush(self.min_heap, val)

        if len(self.max_heap) > len(self.min_heap) + 1:
            val = -heapq.heappop(self.max_heap)
            heapq.heappush(self.min_heap, val)
        elif len(self.min_heap) > len(self.max_heap):
            val = heapq.heappop(self.min_heap)
            heapq.heappush(self.max_heap, -val)

    def findMedian(self):
        """
        :rtype: float
        """
        if len(self.max_heap) > len(self.min_heap):
            return float(-self.max_heap[0])
        return (-self.max_heap[0] + self.min_heap[0]) / 2.0


# ==================== Test Cases ====================
mf = MedianFinder()
mf.addNum(1)
mf.addNum(2)
print(mf.findMedian())   # Expected: 1.5
mf.addNum(3)
print(mf.findMedian())   # Expected: 2.0

print("---")

# Single element
mf2 = MedianFinder()
mf2.addNum(5)
print(mf2.findMedian())  # Expected: 5.0

print("---")

# Reverse sorted insertion
mf3 = MedianFinder()
for num in [5, 4, 3, 2, 1]:
    mf3.addNum(num)
print(mf3.findMedian())  # Expected: 3.0

print("---")

# Negative numbers
mf4 = MedianFinder()
mf4.addNum(-1)
mf4.addNum(-2)
mf4.addNum(-3)
print(mf4.findMedian())  # Expected: -2.0

print("---")

# Even count
mf5 = MedianFinder()
mf5.addNum(1)
mf5.addNum(7)
mf5.addNum(3)
mf5.addNum(5)
print(mf5.findMedian())  # Expected: 4.0
```

---

## Key Takeaways

- The median depends only on the two middle boundary values. You do not need the entire sorted array accessible at all times.
- Two heaps partition the stream into a lower half (max-heap) and an upper half (min-heap). Maintaining the size and partition invariants gives O(log n) insertion and O(1) median retrieval.
- Python's `heapq` is always a min-heap. Negate values to simulate a max-heap. Always negate when pushing and when reading the top.
- The order of operations in `addNum` is: push to max-heap, correct any partition violation, then rebalance sizes. Reversing these steps leads to subtle bugs.
- This two-heap technique generalises to any sliding-window or streaming percentile problem where you need O(1) access to a specific rank in a dynamic dataset.

---

*Google Hard Track · Tutorial 05 of 05*
