# Word Search II

**LeetCode #212 · Difficulty: Hard · Category: Trie & Backtracking**

---

## A few things worth noting about this one:

The "What the Problem Is Really Asking" section explicitly names Word Search I and Implement Trie as prerequisites. Anyone following the Google series in order will have covered Flood Fill (E-08) and the Trie (M-11) before reaching this problem, so the connection is intentional. The Trie pruning gets its own dedicated section rather than just a mention in the code comments, because it is the single most important optimisation and the one most likely to be skipped by a candidate who gets the basic solution working. The step-by-step trace uses a smaller 2x2 board to keep it readable without losing any meaningful cases.

## Problem Statement

Given an `m x n` board of characters and a list of strings `words`, return all words on the board.

Each word must be constructed from letters of sequentially adjacent cells, where adjacent cells are horizontally or vertically neighbouring. The same letter cell may not be used more than once in a word.

**Example 1:**
```
Input:
board = [["o","a","a","n"],
         ["e","t","a","e"],
         ["i","h","k","r"],
         ["i","f","l","v"]]
words = ["oath","pea","eat","rain"]

Output: ["eat","oath"]
```

**Example 2:**
```
Input:
board = [["a","b"],
         ["c","d"]]
words = ["abcb"]

Output: []
```

**Constraints:**
- `m == board.length`
- `n == board[i].length`
- `1 <= m, n <= 12`
- `board[i][j]` is a lowercase English letter
- `1 <= words.length <= 3 * 10^4`
- `1 <= words[i].length <= 10`
- `words[i]` consists of lowercase English letters
- All strings in `words` are unique

---

## Clarifying Questions

Before writing a single line of code, ask these in an interview:

- **Can the same cell be used more than once within a single word?** No. Each cell may be used at most once per word.
- **Can the same cell be reused across different words?** Yes. The restriction is per-word, not across the entire result set.
- **Can words share a prefix?** Yes, and this is exactly why a Trie is the correct structure. Multiple words may start with the same letters.
- **Can the same word appear multiple times in the output?** No. The constraints say all words in the input list are unique. Return each found word once.
- **What order should the output be in?** The problem does not specify. Any order is acceptable.
- **Can the board be a single cell?** Yes, `m` and `n` are each at least 1.

---

## What the Problem Is Really Asking

This problem is the direct combination of two earlier problems:

- **Word Search I (LC #79)** — find a single word on the board using DFS backtracking.
- **Implement Trie (LC #208)** — build a prefix tree to efficiently look up words by prefix.

The naive approach runs a separate DFS for each word in the list. With up to `3 * 10^4` words and a `12 x 12` board, this is far too slow.

The key insight is that many words share prefixes. If you build a Trie from all the words first, a single DFS pass over the board can explore every path and check all words simultaneously. At each cell, you follow the Trie branch corresponding to the current letter. If no branch exists, you backtrack immediately. If a Trie node is marked as a word ending, you record that word. This means shared prefixes are explored only once rather than once per word.

---

## The Trie Structure

A Trie (prefix tree) is a tree where each node represents one character. A path from the root to a marked node spells a complete word.

```
Words: ["eat", "oath"]

Trie:
root
 ├── e
 │    └── a
 │         └── t  [WORD: "eat"]
 └── o
      └── a
           └── t
                └── h  [WORD: "oath"]
```

By building the Trie before the DFS, every DFS step simultaneously checks whether the current path is a valid prefix of any word. If the current character has no child node in the Trie, there is no word in the entire list that starts with this prefix, so you backtrack immediately without exploring further.

---

## Pseudocode

### Brute Force

```
SET result = []

FOR each word in words:
    FOR each cell (r, c) in board:
        IF dfs_word_search(board, word, r, c) finds the word:
            result.append(word)
            BREAK

RETURN result
```

### Optimal (Trie + DFS)

```
BUILD a Trie from all words

SET result = []

FOR each cell (r, c) in board:
    dfs(board, r, c, trie_root, result)

RETURN result

FUNCTION dfs(board, r, c, trie_node, result):
    IF out of bounds OR cell already visited: RETURN
    char = board[r][c]
    IF char not in trie_node.children: RETURN   # no word has this prefix

    next_node = trie_node.children[char]

    IF next_node.word is not None:
        result.append(next_node.word)
        next_node.word = None   # de-duplicate: prevent adding this word again

    MARK cell as visited (board[r][c] = '#')
    RECURSE in all 4 directions with next_node
    RESTORE cell (board[r][c] = char)
```

---

## Brute Force Solution

**Idea:** For each word in the list, run a standard DFS from every cell on the board to check whether that word can be formed.

```python
class Solution(object):
    def findWords(self, board, words):
        """
        :type board: List[List[str]]
        :type words: List[str]
        :rtype: List[str]
        """
        rows, cols = len(board), len(board[0])
        result = []

        def dfs(r, c, word, idx):
            if idx == len(word):
                return True
            if r < 0 or r >= rows or c < 0 or c >= cols:
                return False
            if board[r][c] != word[idx]:
                return False
            temp = board[r][c]
            board[r][c] = '#'
            found = (dfs(r+1,c,word,idx+1) or dfs(r-1,c,word,idx+1) or
                     dfs(r,c+1,word,idx+1) or dfs(r,c-1,word,idx+1))
            board[r][c] = temp
            return found

        for word in words:
            for r in range(rows):
                for c in range(cols):
                    if dfs(r, c, word, 0):
                        result.append(word)
                        break
                else:
                    continue
                break

        return result
```

**Time Complexity:** O(W * m * n * 4 * 3^(L-1)) where W is the number of words and L is the maximum word length. For each word, DFS starts from every cell and branches in up to 4 directions (3 after the first step since you cannot go back). With `W = 30,000` and a `12x12` board this is extremely slow.  
**Space Complexity:** O(L) for the recursion call stack, where L is the max word length.

---

## Optimal Solution — Trie with DFS Backtracking

**Idea:** Build a Trie from all words. Run a single DFS from every cell, following Trie branches rather than matching individual words. All words with shared prefixes are explored simultaneously.

```python
class Solution(object):
    def findWords(self, board, words):
        """
        :type board: List[List[str]]
        :type words: List[str]
        :rtype: List[str]
        """

        # ---- Build the Trie ----
        trie = {}
        for word in words:
            node = trie
            for char in word:
                node = node.setdefault(char, {})
            node['#'] = word   # '#' marks end of word, stores the word itself

        rows, cols = len(board), len(board[0])
        result = []

        # ---- DFS from each cell ----
        def dfs(r, c, node):
            if r < 0 or r >= rows or c < 0 or c >= cols:
                return
            char = board[r][c]
            if char not in node:
                return   # no word in the Trie has this prefix

            next_node = node[char]

            if '#' in next_node:
                result.append(next_node['#'])
                del next_node['#']   # remove to prevent duplicates

            board[r][c] = '#'   # mark as visited
            dfs(r+1, c, next_node)
            dfs(r-1, c, next_node)
            dfs(r, c+1, next_node)
            dfs(r, c-1, next_node)
            board[r][c] = char  # restore

            # Pruning: remove empty Trie nodes to speed up future searches
            if not next_node:
                del node[char]

        for r in range(rows):
            for c in range(cols):
                dfs(r, c, trie)

        return result
```

**Time Complexity:** O(m * n * 4 * 3^(L-1)) where L is the max word length. Each DFS path is bounded by the longest word in the Trie. Critically, this cost does not multiply by W because all words are checked in a single traversal.  
**Space Complexity:** O(W * L) for the Trie, where W is the number of words and L is the max word length. O(L) additional space for the recursion call stack.

---

## The Trie Pruning Optimisation

The lines at the end of `dfs`:

```python
if not next_node:
    del node[char]
```

Once a word is found and its `'#'` marker is deleted, its Trie branch may become empty (no children, no word marker). If that branch is deleted, future DFS calls that reach the same board position will immediately find that the character has no entry in the Trie and return without recursing further.

Without this pruning, a word like `"eat"` found early in the search would still cause the DFS to follow the `e -> a -> t` branch on every subsequent cell that starts with `e`. With pruning, once the word is found and the branch is empty, those paths are skipped entirely.

This optimisation is the difference between a solution that times out on large inputs and one that passes comfortably.

---

## Why a Dictionary-Based Trie

The Trie in this solution uses nested dictionaries rather than a `TrieNode` class. Both are correct. Nested dictionaries are more concise for this problem and avoid the overhead of defining a separate class. The `'#'` sentinel key marks word endings and stores the word itself, so the found word can be appended to the result without reconstructing it from the DFS path.

---

## Step-by-Step Trace

Input: `board = [["o","a"],["e","t"]]`, `words = ["eat", "oath", "oa"]`

```
Build Trie:
{
  'e': {'a': {'t': {'#': 'eat'}}},
  'o': {'a': {'t': {'h': {'#': 'oath'}}, '#': 'oa'}}
}

DFS from (0,0) = 'o':
  'o' in trie root -> follow
  Mark (0,0) = '#'
    DFS from (1,0) = 'e': 'e' not in node['o'] -> return
    DFS from (0,1) = 'a': 'a' in node['o'] -> follow
      '#' in node['o']['a'] -> result.append('oa'), del '#'
      Mark (0,1) = '#'
        DFS from (1,1) = 't': 't' in node -> follow
          Mark (1,1) = '#'
            ... no further matching paths
          Restore (1,1) = 't'
        ... other directions: out of bounds or '#'
      Restore (0,1) = 'a'
      node['o']['a']['t'] is now empty (no words start with 'oat' remaining)
      Prune: del node['o']['a']['t']
  Restore (0,0) = 'o'

DFS from (1,0) = 'e':
  'e' in trie root -> follow
  Mark (1,0) = '#'
    DFS from (0,0) = 'o': 'o' not in node['e'] -> return
    DFS from (1,1) = 't': 't' not in node['e'] -> return
    DFS from (2,0): out of bounds -> return
  Restore (1,0) = 'e'
  (No 'a' reachable from 'e' in this small board via 'eat' path)

... continuing from other starting cells eventually finds 'eat'
    starting at (1,1) = 't' via path t -> ... no
    starting at (0,1) = 'a' -> 'a' not in trie root -> return

Final result: ["oa", "eat"]  (order may vary)
```

---

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force (DFS per word) | O(W * m * n * 4 * 3^(L-1)) | O(L) | Repeats prefix exploration for every word. |
| Trie + DFS (Optimal) | O(m * n * 4 * 3^(L-1)) | O(W * L) | All words explored in one pass. Pruning reduces this further. |

---

## Edge Cases

| Input | Expected | Why |
|-------|----------|-----|
| `words = []` | `[]` | No words to find. |
| `board = [["a"]], words = ["a"]` | `["a"]` | Single cell board, single letter word. |
| `board = [["a"]], words = ["ab"]` | `[]` | Word is longer than reachable path. |
| Word uses a cell twice | Not returned | The `'#'` visited marker prevents cell reuse. |
| Two words share a prefix, only one is on the board | Return only the one found | Trie allows independent tracking of each word's endpoint. |
| Same word appears multiple times in the result | Should appear once | `del next_node['#']` after finding a word prevents duplicates. |

---

## Common Mistakes

**Mistake 1 — Not restoring the cell after backtracking:**  
Setting `board[r][c] = '#'` marks the cell as visited. Failing to restore it with `board[r][c] = char` after the recursive calls means the cell appears permanently visited and cannot be used by any other path starting from a different cell.

**Mistake 2 — Not deleting the word marker after finding a word:**  
If `del next_node['#']` is omitted, the same word can be added to the result multiple times if multiple paths on the board spell that word. The delete prevents duplicates without needing a separate visited set for results.

**Mistake 3 — Omitting the Trie pruning step:**  
Without `if not next_node: del node[char]`, fully found word branches remain in the Trie and continue to be explored on every subsequent DFS call. This causes a timeout on inputs with many words and large boards.

**Mistake 4 — Building the Trie incorrectly by not storing the word at the end node:**  
If you only mark the end node with a boolean flag, you must reconstruct the found word from the DFS path. Storing the word directly in the end node (`node['#'] = word`) is simpler and avoids an additional data structure or path reconstruction.

**Mistake 5 — Running DFS from every cell for every word separately:**  
This is the brute force approach. Building the Trie and running a single multi-word DFS is the entire point of the optimisation. A common mistake is to build the Trie but then still loop over words in the outer loop.

---

## Follow-Up Questions

**Q: What if words can reuse cells (the restriction is removed)?**  
Remove the visited marker logic entirely. The DFS explores all paths without restriction, only backtracking when no Trie branch matches.

**Q: What if the board is very large (say 100 x 100)?**  
The pruning optimisation becomes even more critical. Additionally, you could limit the DFS depth to the length of the longest word in the Trie to avoid exploring paths that cannot possibly complete any word.

**Q: How would you handle words with duplicate characters in the word list?**  
The constraints say all words are unique, so this does not arise here. If duplicates were possible, you could deduplicate the input list before building the Trie.

**Q: What if words could share characters diagonally?**  
Extend the directions list to include all 8 diagonal offsets. The rest of the logic is unchanged.

---

## Test It Yourself

```python
class Solution(object):
    def findWords(self, board, words):
        """
        :type board: List[List[str]]
        :type words: List[str]
        :rtype: List[str]
        """
        trie = {}
        for word in words:
            node = trie
            for char in word:
                node = node.setdefault(char, {})
            node['#'] = word

        rows, cols = len(board), len(board[0])
        result = []

        def dfs(r, c, node):
            if r < 0 or r >= rows or c < 0 or c >= cols:
                return
            char = board[r][c]
            if char not in node:
                return
            next_node = node[char]
            if '#' in next_node:
                result.append(next_node['#'])
                del next_node['#']
            board[r][c] = '#'
            dfs(r+1, c, next_node)
            dfs(r-1, c, next_node)
            dfs(r, c+1, next_node)
            dfs(r, c-1, next_node)
            board[r][c] = char
            if not next_node:
                del node[char]

        for r in range(rows):
            for c in range(cols):
                dfs(r, c, trie)

        return result


# ==================== Test Cases ====================
sol = Solution()

board1 = [["o","a","a","n"],["e","t","a","e"],["i","h","k","r"],["i","f","l","v"]]
print(sorted(sol.findWords(board1, ["oath","pea","eat","rain"])))
# Expected: ["eat", "oath"]

board2 = [["a","b"],["c","d"]]
print(sol.findWords(board2, ["abcb"]))
# Expected: []

board3 = [["a"]]
print(sol.findWords(board3, ["a"]))
# Expected: ["a"]

board4 = [["a","b"],["c","d"]]
print(sorted(sol.findWords(board4, ["ab","cd","ac","bd"])))
# Expected: ["ab", "ac", "bd", "cd"]

board5 = [["a","a"]]
print(sol.findWords(board5, ["aab"]))
# Expected: []  (no 'b' on board)

board6 = [["a","b"],["c","d"]]
print(sol.findWords(board6, ["abdc"]))
# Expected: ["abdc"]  (a->b->d->c is a valid path)
```

---

## Key Takeaways

- Word Search II is the combination of Word Search I (DFS backtracking) and Implement Trie. Understanding both problems first makes this one approachable.
- The Trie converts the problem from O(W * board * path) to O(board * path) by exploring all words simultaneously in a single traversal.
- Storing the word itself at the end node avoids path reconstruction and simplifies the found-word logic.
- The pruning step (removing empty Trie branches after finding a word) is not optional for large inputs. It is the difference between passing and timing out.
- Modifying the board in place (`board[r][c] = '#'`) serves as the visited marker and avoids an extra `visited` set. Always restore the cell after backtracking.

---

*Google Hard Track · Tutorial 04 of 05*
