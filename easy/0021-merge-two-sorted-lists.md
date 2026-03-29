# Merge Two Sorted Lists

**LeetCode #21 · Difficulty: Easy · Category: Linked Lists**

---

## Problem Statement

You are given the heads of two sorted linked lists `list1` and `list2`. Merge the two lists into one sorted list. The list should be made by splicing together the nodes of the first two lists. Return the head of the merged linked list.

**Example 1:**
```
Input:  list1 = [1,2,4], list2 = [1,3,4]
Output: [1,1,2,3,4,4]
```

**Example 2:**
```
Input:  list1 = [], list2 = []
Output: []
```

**Example 3:**
```
Input:  list1 = [], list2 = [0]
Output: [0]
```

**Constraints:**
- The number of nodes in both lists is in the range `[0, 50]`
- `-100 <= Node.val <= 100`
- Both `list1` and `list2` are sorted in non-decreasing order

---

## Clarifying Questions

Before writing a single line of code, ask these in an interview:

- **Can either list be empty?** Yes, as shown in Examples 2 and 3. Your solution must handle null heads gracefully.
- **Are duplicates possible?** Yes, Example 1 shows duplicate values across both lists.
- **Should I allocate new nodes or reuse existing ones?** Reuse the existing nodes by re-wiring their `next` pointers. Do not create new `ListNode` objects.
- **Are both lists guaranteed to be sorted?** Yes, in non-decreasing order per the constraints.
- **What should I return if both lists are empty?** Return `None` (an empty list).

---

## What the Problem Is Really Asking

You have two sorted linked lists and need to weave them together into one sorted list. At each step, you compare the front nodes of both lists, take the smaller one, attach it to your result, and advance that list's pointer forward.

The merge logic itself is straightforward. The tricky part is: what does your result list start with? You don't know until the first comparison is made. Handling that "where does the result begin?" question without messy special cases is what the **dummy head technique** solves.

A dummy head is a throwaway node created at the start with value `0`. You build your result list starting after this placeholder. At the end you return `dummy.next`, skipping it. This means the first real node is attached exactly the same way as every other node, with no special casing required.

---

## Pseudocode

### Brute Force

```
COLLECT all values from list1 into an array
COLLECT all values from list2 into the same array
SORT the array
BUILD a new linked list from the sorted values
RETURN the head of the new list
```

### Optimal (Dummy Head + Two Pointers)

```
CREATE a dummy node (value = 0)
SET current = dummy

WHILE list1 is not null AND list2 is not null:
    IF list1.val <= list2.val:
        current.next = list1
        list1 = list1.next
    ELSE:
        current.next = list2
        list2 = list2.next
    current = current.next

# Attach whichever list still has nodes remaining
current.next = list1 if list1 else list2

RETURN dummy.next
```

The last line is O(1). Because both input lists are already sorted, whatever remains in the non-exhausted list is already in the correct order. We attach it directly without any further comparisons.

---

## Brute Force Solution

**Idea:** Collect all values from both lists into an array, sort it, then build a brand new linked list from scratch.

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def mergeTwoLists(self, list1, list2):
        """
        :type list1: Optional[ListNode]
        :type list2: Optional[ListNode]
        :rtype: Optional[ListNode]
        """
        vals = []

        while list1:
            vals.append(list1.val)
            list1 = list1.next
        while list2:
            vals.append(list2.val)
            list2 = list2.next

        vals.sort()

        dummy = ListNode(0)
        current = dummy
        for v in vals:
            current.next = ListNode(v)
            current = current.next

        return dummy.next
```

This approach throws away the fact that both input lists are already sorted. We pay the cost of re-sorting all values from scratch and allocate entirely new nodes, which violates the spirit of the problem ("splicing together the nodes of the first two lists").

**Time Complexity:** O((n + m) log(n + m)) where n and m are the lengths of the two lists. The sort dominates.  
**Space Complexity:** O(n + m) for the values array and the newly allocated nodes.

---

## Optimal Solution — Iterative (Dummy Head)

**Idea:** Exploit the fact that both lists are already sorted. Compare only the current front nodes of each list, take the smaller one, and advance. One pass, no sorting needed.

```python
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def mergeTwoLists(self, list1, list2):
        """
        :type list1: Optional[ListNode]
        :type list2: Optional[ListNode]
        :rtype: Optional[ListNode]
        """
        dummy = ListNode(0)
        current = dummy

        while list1 and list2:
            if list1.val <= list2.val:
                current.next = list1
                list1 = list1.next
            else:
                current.next = list2
                list2 = list2.next
            current = current.next

        # At most one list still has nodes — attach the remainder directly
        current.next = list1 if list1 else list2

        return dummy.next
```

There is also a recursive formulation worth knowing for follow-up discussions:

```python
class Solution(object):
    def mergeTwoLists(self, list1, list2):
        """
        :type list1: Optional[ListNode]
        :type list2: Optional[ListNode]
        :rtype: Optional[ListNode]
        """
        if not list1:
            return list2
        if not list2:
            return list1
        if list1.val <= list2.val:
            list1.next = self.mergeTwoLists(list1.next, list2)
            return list1
        else:
            list2.next = self.mergeTwoLists(list1, list2.next)
            return list2
```

The recursive solution is elegant to read but uses O(n + m) space on the call stack. The iterative solution is strictly better on space and is the preferred answer in an interview.

---

## Step-by-Step Trace

Input: `list1 = [1, 2, 4]`, `list2 = [1, 3, 4]`

```
dummy -> ?
current = dummy

Step 1: list1.val=1, list2.val=1  →  1 <= 1, take list1
        result: dummy -> 1
        list1 advances to [2, 4]

Step 2: list1.val=2, list2.val=1  →  2 > 1, take list2
        result: dummy -> 1 -> 1
        list2 advances to [3, 4]

Step 3: list1.val=2, list2.val=3  →  2 <= 3, take list1
        result: dummy -> 1 -> 1 -> 2
        list1 advances to [4]

Step 4: list1.val=4, list2.val=3  →  4 > 3, take list2
        result: dummy -> 1 -> 1 -> 2 -> 3
        list2 advances to [4]

Step 5: list1.val=4, list2.val=4  →  4 <= 4, take list1
        result: dummy -> 1 -> 1 -> 2 -> 3 -> 4
        list1 advances to None

While loop ends (list1 is now None)

Attach remainder: current.next = list2 = [4]
Final:  dummy -> 1 -> 1 -> 2 -> 3 -> 4 -> 4

Return dummy.next = [1, 1, 2, 3, 4, 4] ✓
```

---

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force | O((n+m) log(n+m)) | O(n+m) | Sort dominates. New nodes allocated. |
| Iterative (Optimal) | O(n+m) | O(1) | Single pass. Only the dummy node is extra storage. |
| Recursive | O(n+m) | O(n+m) | Same time, but each recursive call adds a frame to the call stack. |

---

## Edge Cases

| Input | Expected | Why |
|-------|----------|-----|
| `list1 = [], list2 = []` | `[]` | Both empty. While loop never runs. `current.next = None`. Return `None`. |
| `list1 = [], list2 = [0]` | `[0]` | One list empty from the start. While loop never runs. Remainder attached directly. |
| `list1 = [1], list2 = [2]` | `[1, 2]` | Single-node lists. One iteration, then remainder attached. |
| `list1 = [1, 1], list2 = [1]` | `[1, 1, 1]` | All duplicates across both lists. |
| `list1 = [5], list2 = [1, 2, 3]` | `[1, 2, 3, 5]` | list2 exhausted first, list1 remainder attached at end. |

---

## Common Mistakes

**Mistake 1 — Forgetting to advance `current` after attaching a node:**  
Inside the while loop you set `current.next` but forget `current = current.next`. Every new node overwrites the same position, and you return a list of length 1. This is the most common bug under pressure on this problem.

**Mistake 2 — Not handling the case where one list is exhausted before the other:**  
If you exit the while loop and don't attach the remaining nodes, you silently truncate the result. The line `current.next = list1 if list1 else list2` handles this in one step. It also covers the case where one list is empty from the start.

**Mistake 3 — Using `<` instead of `<=` in the comparison:**  
With strict less-than, equal values from different lists are ordered unpredictably depending on which list they appear in. Using `<=` gives stable, deterministic output and is the conventional correct implementation.

**Mistake 4 — Allocating new nodes instead of re-wiring existing ones:**  
The problem says to splice together the nodes of the input lists. Creating `ListNode(v)` for each value wastes memory and ignores the structure you were given. Re-wire `next` pointers instead.

---

## Follow-Up Questions

**Q: How would you merge K sorted lists instead of just two? (LC #23)**  
The naive approach merges them one by one: O(kN) total time where N is the total number of nodes. The optimal approach uses a min-heap of size k. Push the head of each list into the heap. Always pop the smallest node, attach it to the result, then push that node's `next` into the heap. Time: O(N log k). Space: O(k) for the heap.

**Q: What if the lists are not sorted?**  
You would have to sort them first at O(n log n + m log m), then apply the same merge. The interviewer is checking whether you understand that the sorted property is what enables the O(n + m) solution.

**Q: Can you do this without the dummy node?**  
Yes, but it requires additional if-statements to determine which list's head becomes the result head. The dummy head approach produces identical runtime and space with significantly cleaner code.

---

## Test It Yourself

The `ListNode` class and a helper to print the list are included so you can run this directly in any Python interpreter.

```python
class ListNode(object):
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next


class Solution(object):
    def mergeTwoLists(self, list1, list2):
        """
        :type list1: Optional[ListNode]
        :type list2: Optional[ListNode]
        :rtype: Optional[ListNode]
        """
        dummy = ListNode(0)
        current = dummy

        while list1 and list2:
            if list1.val <= list2.val:
                current.next = list1
                list1 = list1.next
            else:
                current.next = list2
                list2 = list2.next
            current = current.next

        current.next = list1 if list1 else list2

        return dummy.next


# ==================== Helper ====================
def make_list(values):
    if not values:
        return None
    head = ListNode(values[0])
    current = head
    for v in values[1:]:
        current.next = ListNode(v)
        current = current.next
    return head

def print_list(node):
    result = []
    while node:
        result.append(node.val)
        node = node.next
    print(result)


# ==================== Test Cases ====================
sol = Solution()

print_list(sol.mergeTwoLists(make_list([1, 2, 4]), make_list([1, 3, 4])))  # Expected: [1, 1, 2, 3, 4, 4]
print_list(sol.mergeTwoLists(make_list([]), make_list([])))                # Expected: []
print_list(sol.mergeTwoLists(make_list([]), make_list([0])))               # Expected: [0]
print_list(sol.mergeTwoLists(make_list([1]), make_list([2])))              # Expected: [1, 2]
print_list(sol.mergeTwoLists(make_list([5]), make_list([1, 2, 3])))        # Expected: [1, 2, 3, 5]
print_list(sol.mergeTwoLists(make_list([1, 1]), make_list([1])))           # Expected: [1, 1, 1]
```

---

## Key Takeaways

- The **dummy head technique** eliminates edge case handling for the first node. Any time you are building a linked list result from scratch, start with a dummy head and return `dummy.next`.
- Always advance `current` after attaching each node. Forgetting this is the most common linked list bug.
- The `current.next = list1 if list1 else list2` pattern handles both the mid-loop exhaustion case and the case where one list was empty from the start.
- This dummy head pattern reappears in: Add Two Numbers (LC #2), Odd Even Linked List (LC #328), and Merge K Sorted Lists (LC #23).

---

*Google Easy Track · Tutorial 03 of 08*
