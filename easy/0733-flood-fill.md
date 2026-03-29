# Flood Fill

**LeetCode #733 · Difficulty: Easy · Category: Graphs & Grid Traversal**

---

## Problem Statement

You are given an image represented by an `m x n` grid of integers `image`, where `image[i][j]` represents the pixel value of the image. You are also given three integers `sr`, `sc`, and `color`. Your task is to perform a flood fill on the image starting from the pixel `image[sr][sc]`.

To perform a flood fill:
1. Begin with the starting pixel and change its color to `color`.
2. Perform the same process for each pixel that is directly adjacent (pixels that share a side, either horizontally or vertically) and shares the same color as the starting pixel.
3. Keep repeating this process for the updated pixels, modifying their neighbours if they match the original color.
4. Stop when there are no more adjacent pixels of the original color to update.

Return the modified image after performing the flood fill.

**Example 1:**
```
Input:  image = [[1,1,1],[1,1,0],[1,0,1]], sr = 1, sc = 1, color = 2
Output: [[2,2,2],[2,2,0],[2,0,1]]

Before:         After:
1  1  1         2  2  2
1  1  0         2  2  0
1  0  1         2  0  1

The bottom-right 1 is not filled because it is not horizontally
or vertically connected to the starting pixel.
```

**Example 2:**
```
Input:  image = [[0,0,0],[0,0,0]], sr = 0, sc = 0, color = 0
Output: [[0,0,0],[0,0,0]]

The starting pixel already has color 0, which equals the target color.
No changes are made.
```

**Constraints:**
- `m == image.length`
- `n == image[i].length`
- `1 <= m, n <= 50`
- `0 <= image[i][j], color < 2^16`
- `0 <= sr < m`
- `0 <= sc < n`

---

## Clarifying Questions

Before writing a single line of code, ask these in an interview:

- **Is connectivity 4-directional or 8-directional?** 4-directional only (up, down, left, right). Diagonal neighbours do not count.
- **What if the starting pixel already has the target color?** Return the image unchanged. This is a critical edge case that prevents an infinite loop.
- **Are we modifying the image in place or returning a new one?** Modify in place and return the same grid.
- **Can the grid be a single cell?** Yes, `m` and `n` can both be `1`. The solution must handle this correctly.
- **Can color values be zero?** Yes, `color >= 0` per the constraints. This matters for the edge case check.

---

## What the Problem Is Really Asking

This is a connected-component traversal problem. Starting from a given cell, you need to visit every cell reachable through a chain of 4-directional neighbours that all share the original color, and repaint each one.

The structure is identical to Depth First Search (DFS) or Breadth First Search (BFS) on a graph, where each cell is a node and edges connect cells that are adjacent and share the original color.

The key insight is that painting a cell to the new color as you visit it serves as your visited marker. You do not need a separate visited set. When your traversal revisits a cell and checks whether it matches the original color, it will not match because it has already been repainted, so it will not be processed again.

---

## Pseudocode

### DFS (Recursive)

```
FUNCTION dfs(image, row, col, original_color, new_color):
    IF row or col is out of bounds: RETURN
    IF image[row][col] != original_color: RETURN

    image[row][col] = new_color   # paint and mark as visited

    dfs(image, row + 1, col, original_color, new_color)
    dfs(image, row - 1, col, original_color, new_color)
    dfs(image, row, col + 1, original_color, new_color)
    dfs(image, row, col - 1, original_color, new_color)

MAIN:
    IF image[sr][sc] == color: RETURN image   # already the target color

    CALL dfs(image, sr, sc, image[sr][sc], color)
    RETURN image
```

### BFS (Iterative)

```
IF image[sr][sc] == color: RETURN image

SET original = image[sr][sc]
CREATE queue containing [(sr, sc)]
SET image[sr][sc] = color

WHILE queue is not empty:
    row, col = dequeue from front
    FOR each of the 4 directions:
        nr, nc = row + dr, col + dc
        IF in bounds AND image[nr][nc] == original:
            image[nr][nc] = color
            enqueue (nr, nc)

RETURN image
```

---

## About This Problem's Structure

Like Invert Binary Tree, Flood Fill does not have a meaningful brute force versus optimal distinction. Every connected cell must be visited exactly once. There is no shortcut. Any correct solution is O(m x n) time.

The interesting comparison is between two implementation strategies:

- **Recursive DFS** — concise, mirrors the recursive definition of the traversal
- **Iterative BFS** — explicit queue, avoids Python's call stack depth limit on large grids

Both are shown below.

---

## Solution 1 — Recursive DFS

**Idea:** From the starting cell, recursively visit all 4 neighbours that share the original color. Paint each cell before recursing so it is not visited again.

```python
class Solution(object):
    def floodFill(self, image, sr, sc, color):
        """
        :type image: List[List[int]]
        :type sr: int
        :type sc: int
        :type color: int
        :rtype: List[List[int]]
        """
        original = image[sr][sc]

        # If the starting cell is already the target color, do nothing
        if original == color:
            return image

        def dfs(r, c):
            if r < 0 or r >= len(image) or c < 0 or c >= len(image[0]):
                return
            if image[r][c] != original:
                return
            image[r][c] = color
            dfs(r + 1, c)
            dfs(r - 1, c)
            dfs(r, c + 1)
            dfs(r, c - 1)

        dfs(sr, sc)
        return image
```

**Time Complexity:** O(m x n) — every cell is visited at most once.  
**Space Complexity:** O(m x n) — the call stack can reach depth m x n in the worst case if the entire grid is one connected region.

---

## Solution 2 — Iterative BFS

**Idea:** Use a queue to process cells level by level. Paint each cell when it is added to the queue, not when it is dequeued, to prevent duplicate entries.

```python
from collections import deque

class Solution(object):
    def floodFill(self, image, sr, sc, color):
        """
        :type image: List[List[int]]
        :type sr: int
        :type sc: int
        :type color: int
        :rtype: List[List[int]]
        """
        original = image[sr][sc]

        if original == color:
            return image

        rows, cols = len(image), len(image[0])
        queue = deque([(sr, sc)])
        image[sr][sc] = color

        directions = [(1, 0), (-1, 0), (0, 1), (0, -1)]

        while queue:
            r, c = queue.popleft()
            for dr, dc in directions:
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols and image[nr][nc] == original:
                    image[nr][nc] = color
                    queue.append((nr, nc))

        return image
```

**Time Complexity:** O(m x n) — every cell is visited at most once.  
**Space Complexity:** O(m x n) — the queue holds at most m x n cells in the worst case.

---

## The 4-Directions Pattern

Both solutions use the same set of directional offsets:

```python
directions = [(1, 0), (-1, 0), (0, 1), (0, -1)]
```

This represents moving down, up, right, and left respectively. Looping over this list is the standard way to enumerate 4-connected neighbours in grid problems. Memorise it. You will use it in Number of Islands (LC #200), Pacific Atlantic Water Flow (LC #417), Surrounded Regions (LC #130), and many others.

For 8-directional problems (where diagonals also count), add the four diagonal offsets:

```python
directions = [(1,0),(-1,0),(0,1),(0,-1),(1,1),(1,-1),(-1,1),(-1,-1)]
```

Everything else in the traversal remains identical.

---

## Step-by-Step Trace

Input: `image = [[1,1,1],[1,1,0],[1,0,1]]`, `sr = 1`, `sc = 1`, `color = 2`

```
original = image[1][1] = 1
original != color (1 != 2), so proceed

Call dfs(1, 1):
  image[1][1] = 2
  Grid: [[1,1,1],[1,2,0],[1,0,1]]

  Call dfs(2, 1):
    image[2][1] = 0, 0 != original (1), RETURN

  Call dfs(0, 1):
    image[0][1] = 1 == original, paint
    image[0][1] = 2
    Grid: [[1,2,1],[1,2,0],[1,0,1]]
    dfs(1,1) -> already color 2, RETURN
    dfs(-1,1) -> out of bounds, RETURN
    dfs(0,2) -> image[0][2]=1, paint -> 2
    dfs(0,0) -> image[0][0]=1, paint -> 2
    ...

  Call dfs(1, 2):
    image[1][2] = 0, 0 != original, RETURN

  Call dfs(1, 0):
    image[1][0] = 1 == original, paint -> 2
    ...continues to fill remaining connected 1s

Final: [[2,2,2],[2,2,0],[2,0,1]] ✓
```

The bottom-right cell `image[2][2] = 1` is never reached because it is only diagonally connected to the starting cell, and diagonal neighbours are not included.

---

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Recursive DFS | O(m x n) | O(m x n) | Call stack depth equals the size of the connected region in the worst case. |
| Iterative BFS | O(m x n) | O(m x n) | Queue holds at most m x n cells. Safer for large grids. |

Both approaches have the same asymptotic complexity. The practical difference is that the recursive DFS can hit Python's default recursion limit of approximately 1,000 frames on a 50 x 50 grid with a large connected region (2,500 cells). The iterative BFS handles any grid size safely.

---

## Edge Cases

| Input | Expected | Why |
|-------|----------|-----|
| `image=[[0,0,0],[0,0,0]], sr=0, sc=0, color=0` | `[[0,0,0],[0,0,0]]` | Starting color already equals target color. Return immediately. |
| `image=[[1]], sr=0, sc=0, color=2` | `[[2]]` | Single cell grid. Paint and return. |
| `image=[[1,1],[1,1]], sr=0, sc=0, color=2` | `[[2,2],[2,2]]` | Entire grid is one connected region. |
| `image=[[1,0,1]], sr=0, sc=0, color=2` | `[[2,0,1]]` | The `0` acts as a barrier. Only the left `1` is filled. |
| `image=[[1,0,1]], sr=0, sc=2, color=2` | `[[1,0,2]]` | Starting from the isolated right cell. Only it is filled. |

---

## Common Mistakes

**Mistake 1 — Not handling the case where `original == color`:**  
If the starting pixel already has the target color, the condition `image[r][c] != original` will never be true (since the first cell is immediately painted to `color`, which equals `original`). The DFS recurses forever. Always check `if original == color: return image` before starting the traversal.

**Mistake 2 — Using a separate visited set instead of painting in place:**  
Adding a `visited = set()` works but uses O(m x n) extra space unnecessarily. Painting the cell to the new color as you visit it serves as the visited marker. When the traversal revisits a cell, it checks the color, sees it no longer matches `original`, and returns immediately.

**Mistake 3 — Painting when dequeuing rather than when enqueuing in BFS:**  
If you enqueue a cell without immediately painting it, the same cell can be added to the queue multiple times before it is dequeued and painted. This leads to redundant processing and incorrect results on some inputs. Paint (and mark as visited) at the moment of enqueue.

**Mistake 4 — Checking bounds after accessing the array:**  
Accessing `image[r][c]` before verifying that `r` and `c` are within bounds causes an `IndexError`. Always check bounds first: `if 0 <= nr < rows and 0 <= nc < cols` before reading `image[nr][nc]`.

---

## Follow-Up Questions

**Q: How would you solve Number of Islands? (LC #200)**  
Iterate over every cell in the grid. When you find an unvisited land cell (`'1'`), increment an island counter and run DFS/BFS to mark the entire connected island as visited. The same 4-directional traversal applies. O(m x n) time.

**Q: What if connectivity is 8-directional?**  
Extend the directions list to include the four diagonal offsets. Everything else remains identical.

**Q: What if the grid is very large and most cells are not the original color?**  
BFS is preferable in this case. It processes cells level by level and the queue stays small when connectivity is sparse. DFS could descend deep into a long chain before backtracking. For large, sparsely connected grids, BFS uses less memory in practice.

**Q: What is multi-source BFS and when does it apply?**  
Instead of starting from a single cell, you initialise the queue with multiple source cells simultaneously. This solves problems like "spread infection from all initially infected cells at once" or "find the distance from any land cell to the nearest water cell." The traversal logic is identical; only the initialisation changes.

---

## Test It Yourself

```python
class Solution(object):
    def floodFill(self, image, sr, sc, color):
        """
        :type image: List[List[int]]
        :type sr: int
        :type sc: int
        :type color: int
        :rtype: List[List[int]]
        """
        original = image[sr][sc]

        if original == color:
            return image

        def dfs(r, c):
            if r < 0 or r >= len(image) or c < 0 or c >= len(image[0]):
                return
            if image[r][c] != original:
                return
            image[r][c] = color
            dfs(r + 1, c)
            dfs(r - 1, c)
            dfs(r, c + 1)
            dfs(r, c - 1)

        dfs(sr, sc)
        return image


# ==================== Test Cases ====================
sol = Solution()

print(sol.floodFill([[1,1,1],[1,1,0],[1,0,1]], 1, 1, 2))
# Expected: [[2,2,2],[2,2,0],[2,0,1]]

print(sol.floodFill([[0,0,0],[0,0,0]], 0, 0, 0))
# Expected: [[0,0,0],[0,0,0]]  (no change, already target color)

print(sol.floodFill([[1]], 0, 0, 2))
# Expected: [[2]]

print(sol.floodFill([[1,1],[1,1]], 0, 0, 2))
# Expected: [[2,2],[2,2]]

print(sol.floodFill([[1,0,1]], 0, 0, 2))
# Expected: [[2,0,1]]  (barrier at index 1 blocks the fill)

print(sol.floodFill([[1,0,1]], 0, 2, 2))
# Expected: [[1,0,2]]  (only the isolated right cell is filled)
```

---

## Key Takeaways

- Flood Fill is DFS or BFS on a grid. The grid is a graph where edges connect adjacent cells with the same original color.
- Painting a cell to the new color as you visit it is the visited marker. No separate set is needed.
- The `original == color` early exit is not optional. Without it the traversal loops indefinitely.
- The 4-directions pattern `[(1,0),(-1,0),(0,1),(0,-1)]` is the standard neighbour enumeration for any 4-connected grid problem. Memorise it.
- This traversal pattern is the foundation for: Number of Islands (LC #200), Pacific Atlantic Water Flow (LC #417), Surrounded Regions (LC #130), and Rotting Oranges (LC #994).

---

*Google Easy Track · Tutorial 08 of 08*
