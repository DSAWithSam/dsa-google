# Invert Binary Tree

**LeetCode #226 · Difficulty: Easy · Category: Binary Trees**

---

## Problem Statement

Given the `root` of a binary tree, invert the tree and return its root.

**Example 1:**
```
Input:  root = [4,2,7,1,3,6,9]
Output: [4,7,2,9,6,3,1]

Before:         After:
      4               4
     / \             / \
    2   7           7   2
   / \ / \         / \ / \
  1  3 6  9       9  6 3  1
```

**Example 2:**
```
Input:  root = [2,1,3]
Output: [2,3,1]

Before:    After:
    2          2
   / \        / \
  1   3      3   1
```

**Example 3:**
```
Input:  root = []
Output: []
```

**Constraints:**
- The number of nodes in the tree is in the range `[0, 100]`
- `-100 <= Node.val <= 100`

---

## Clarifying Questions

Before writing a single line of code, ask these in an interview:

- **Can the tree be empty?** Yes, Example 3 shows an empty tree. Return `None`.
- **Is this a BST or a general binary tree?** A general binary tree. The BST property does not hold after inversion anyway.
- **Should I modify in-place or return a new tree?** Modify in-place by swapping child pointers, then return the same root.
- **What does "invert" mean exactly?** Mirror the tree left to right. Every node's left and right children are swapped, recursively all the way down.
- **Can node values be negative?** Yes, the constraints allow values down to `-100`.

---

## What the Problem Is Really Asking

Inverting a binary tree means mirroring it. For every node in the tree, its left child and right child are swapped. This operation applies to every node, not just the root.

The key observation is that this is a naturally recursive operation. To invert a tree rooted at node `X`: swap `X`'s left and right children, then invert the left subtree, then invert the right subtree. The base case is when the node is `None` — do nothing and return. That structure matches the structure of the tree itself, which is why recursion feels so natural here.

This problem also has a clean iterative solution using BFS. Both are worth knowing.

---

## Pseudocode

### Recursive DFS

```
FUNCTION invertTree(node):
    IF node is None:
        RETURN None

    SWAP node.left and node.right

    invertTree(node.left)
    invertTree(node.right)

    RETURN node
```

### Iterative BFS

```
IF root is None:
    RETURN None

CREATE a queue containing [root]

WHILE queue is not empty:
    node = dequeue from front
    SWAP node.left and node.right
    IF node.left exists:  enqueue node.left
    IF node.right exists: enqueue node.right

RETURN root
```

---

## About This Problem's Structure

This problem does not have a meaningful brute force vs. optimal distinction. You must visit every node exactly once to swap its children. There is no shortcut. Every node must be processed, so any correct solution is O(n) time.

The interesting comparison is between the two implementation strategies:

- **Recursive DFS** — concise, mirrors the recursive definition of a tree
- **Iterative BFS** — explicit queue, avoids call stack depth issues on very deep trees

Both are shown below.

---

## Solution 1 — Recursive DFS

**Idea:** Swap the children of the current node, then recursively invert both subtrees. Return the node.

```python
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def invertTree(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: Optional[TreeNode]
        """
        if not root:
            return None

        # Swap left and right children
        root.left, root.right = root.right, root.left

        # Recursively invert both subtrees
        self.invertTree(root.left)
        self.invertTree(root.right)

        return root
```

The Python tuple swap `root.left, root.right = root.right, root.left` requires no temporary variable. Python evaluates the entire right-hand side before performing any assignments, so there is no risk of overwriting a value before it is read.

**Time Complexity:** O(n) — every node is visited exactly once.  
**Space Complexity:** O(h) — where `h` is the height of the tree. This is the call stack depth. For a balanced tree, h = O(log n). For a completely skewed tree (essentially a linked list), h = O(n).

---

## Solution 2 — Iterative BFS

**Idea:** Use a queue to process nodes level by level. At each node, swap its children and enqueue them for processing.

```python
from collections import deque

# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def invertTree(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: Optional[TreeNode]
        """
        if not root:
            return None

        queue = deque([root])

        while queue:
            node = queue.popleft()

            # Swap children at this node
            node.left, node.right = node.right, node.left

            # Enqueue children for processing
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)

        return root
```

**Time Complexity:** O(n) — every node is visited exactly once.  
**Space Complexity:** O(n) — the queue holds at most O(n/2) nodes at the widest level of the tree.

---

## Step-by-Step Trace

Input: `root = [4, 2, 7, 1, 3, 6, 9]`

```
Initial tree:
        4
       / \
      2   7
     / \ / \
    1  3 6  9

Step 1: Visit node 4. Swap children.
        4
       / \
      7   2
     / \ / \
    6  9 1  3

Step 2: Visit node 7 (was right, now left). Swap its children.
        4
       / \
      7   2
     / \ / \
    9  6 1  3

Step 3: Visit node 2 (was left, now right). Swap its children.
        4
       / \
      7   2
     / \ / \
    9  6 3  1

Steps 4-7: Visit leaf nodes 9, 6, 3, 1. No children to swap.

Final: [4, 7, 2, 9, 6, 3, 1] ✓
```

---

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Recursive DFS | O(n) | O(h) | Call stack depth equals tree height. O(log n) balanced, O(n) skewed. |
| Iterative BFS | O(n) | O(n) | Queue holds up to O(n/2) nodes at the widest level. |

For most inputs, the recursive solution uses less memory than BFS. The exception is a very deep, skewed tree where recursion depth approaches n and risks hitting Python's recursion limit (default 1,000 frames). In that case, the iterative solution is the safer choice.

---

## Edge Cases

| Input | Expected | Why |
|-------|----------|-----|
| `[]` | `[]` | Empty tree. Return `None` immediately. |
| `[1]` | `[1]` | Single node. No children to swap. |
| `[1, 2]` | `[1, null, 2]` | Only a left child. After inversion it becomes a right child. |
| `[1, null, 2]` | `[1, 2]` | Only a right child. After inversion it becomes a left child. |
| `[1, 2, 3]` | `[1, 3, 2]` | Simple three-node tree. |

---

## Common Mistakes

**Mistake 1 — Not returning `root` at the end:**  
The function signature requires returning the root of the inverted tree. Forgetting `return root` causes the function to return `None`, which looks identical to the empty tree case. Always return root in tree problems that ask for it.

**Mistake 2 — Swapping children at only the root:**  
The swap must happen at every node, not just the top level. Failing to recurse means only the root's immediate children are swapped while the rest of the tree remains unchanged.

**Mistake 3 — Not handling the `None` base case:**  
Without `if not root: return None`, the recursive calls on leaf nodes' children will crash with an `AttributeError` when trying to access `None.left` and `None.right`.

**Mistake 4 — Using recursion on a tree that could be very deep:**  
Python's default recursion limit is around 1,000 frames. For a skewed tree with 100,000 nodes, the recursive solution will hit a `RecursionError`. In a production context or when tree depth is unbounded, the iterative BFS solution is safer. Mention this in your interview.

---

## Follow-Up Questions

**Q: Can you verify that a tree has been correctly inverted?**  
Apply `invertTree` twice and check that the result equals the original tree. Alternatively, do an in-order traversal of the original and the inverted tree and verify the inverted values are the original values in reverse.

**Q: How does inverting a BST affect its properties?**  
Inverting a BST produces a tree where the in-order traversal is in descending order instead of ascending. It is no longer a valid BST by the standard definition unless you redefine the ordering property.

**Q: What if the tree is extremely deep (say, 100,000 nodes in a skewed structure)?**  
Use the iterative BFS solution to avoid a `RecursionError`. Alternatively, increase Python's recursion limit with `sys.setrecursionlimit()`, but this is generally discouraged in production code.

---

## Test It Yourself

The `TreeNode` class and helper functions for building and printing a tree are included so you can run this directly in any Python interpreter.

```python
from collections import deque


class TreeNode(object):
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


class Solution(object):
    def invertTree(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: Optional[TreeNode]
        """
        if not root:
            return None

        root.left, root.right = root.right, root.left

        self.invertTree(root.left)
        self.invertTree(root.right)

        return root


# ==================== Helpers ====================
def build_tree(values):
    """Builds a tree from a level-order list. None represents a missing node."""
    if not values or values[0] is None:
        return None
    root = TreeNode(values[0])
    queue = deque([root])
    i = 1
    while queue and i < len(values):
        node = queue.popleft()
        if i < len(values) and values[i] is not None:
            node.left = TreeNode(values[i])
            queue.append(node.left)
        i += 1
        if i < len(values) and values[i] is not None:
            node.right = TreeNode(values[i])
            queue.append(node.right)
        i += 1
    return root

def level_order(root):
    """Returns a level-order list representation of the tree."""
    if not root:
        return []
    result = []
    queue = deque([root])
    while queue:
        node = queue.popleft()
        result.append(node.val)
        if node.left:
            queue.append(node.left)
        if node.right:
            queue.append(node.right)
    return result


# ==================== Test Cases ====================
sol = Solution()

print(level_order(sol.invertTree(build_tree([4, 2, 7, 1, 3, 6, 9]))))  # Expected: [4, 7, 2, 9, 6, 3, 1]
print(level_order(sol.invertTree(build_tree([2, 1, 3]))))               # Expected: [2, 3, 1]
print(level_order(sol.invertTree(build_tree([]))))                      # Expected: []
print(level_order(sol.invertTree(build_tree([1]))))                     # Expected: [1]
print(level_order(sol.invertTree(build_tree([1, 2]))))                  # Expected: [1, 2] (was left, now right)
print(level_order(sol.invertTree(build_tree([1, None, 2]))))            # Expected: [1, 2] (was right, now left)
```

---

## Key Takeaways

- The **recursive DFS skeleton** for trees is: check base case (`if not root`), operate on the node, recurse left, recurse right, return node. This skeleton applies to nearly every tree problem.
- Python's tuple swap `a, b = b, a` requires no temporary variable and is idiomatic.
- Always consider recursion depth on tree problems. For deep or skewed trees, the iterative solution avoids stack overflow.
- This pattern of visiting every node and performing an operation appears in: Maximum Depth of Binary Tree (LC #104), Same Tree (LC #100), Symmetric Tree (LC #101), and Path Sum (LC #112).

---

*Google Easy Track · Tutorial 05 of 08*
