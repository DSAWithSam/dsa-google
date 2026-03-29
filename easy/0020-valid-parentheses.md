# Valid Parentheses

**LeetCode #20 · Difficulty: Easy · Category: Stack**

---

## Problem Statement

Given a string `s` containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.

An input string is valid if:
1. Open brackets must be closed by the same type of brackets.
2. Open brackets must be closed in the correct order.
3. Every close bracket has a corresponding open bracket of the same type.

**Example 1:**
```
Input:  s = "()"
Output: true
```

**Example 2:**
```
Input:  s = "()[]{}"
Output: true
```

**Example 3:**
```
Input:  s = "(]"
Output: false
```

**Example 4:**
```
Input:  s = "([])"
Output: true
```

**Example 5:**
```
Input:  s = "([)]"
Output: false
```

**Constraints:**
- `1 <= s.length <= 10^4`
- `s` consists of parentheses only `'()[]{}'`

---

## Clarifying Questions

Before writing a single line of code, ask these in an interview:

- **Can the input be empty?** The constraints say `s.length >= 1`, so no, but worth confirming. An empty string is technically valid (nothing to invalidate it).
- **Only these six characters?** Yes, guaranteed by the constraints. No letters, spaces, or other symbols.
- **What's the maximum length?** Up to `10^4`, so O(n) time and O(n) space are both perfectly acceptable.
- **Is whitespace possible?** No, constraints confirm only `'()[]{}'`.

---

## What the Problem Is Really Asking

The core challenge is **matching in order**. Every closing bracket must match the *most recently seen* open bracket, not just any open bracket.

Consider `"([)]"`: every bracket has a match by count, but it's invalid because `)` closes before `[` is resolved. That ordering constraint is the key.

The phrase **"most recently seen"** is your signal. Whenever you need to track the most recently seen item and match against it, think **stack**: Last In, First Out (LIFO).

---

## Pseudocode

### Brute Force

```
WHILE the string contains "()", "[]", or "{}":
    REMOVE all occurrences of those pairs from the string

IF string is empty:
    RETURN true
ELSE:
    RETURN false
```

### Optimal (Stack)

```
CREATE an empty stack
CREATE a mapping: closing bracket → its expected open bracket
    { ")": "(", "]": "[", "}": "{" }

FOR each character in s:
    IF character is a closing bracket:
        IF stack is empty OR top of stack != expected open bracket:
            RETURN false
        ELSE:
            POP the top of the stack
    ELSE (it's an open bracket):
        PUSH it onto the stack

RETURN true only if stack is empty
```

**Key insight:** When we see an open bracket, we push it and defer matching it until later. When we see a closing bracket, we immediately check whether the top of the stack is its correct partner. If the stack is empty at the end, every open bracket was matched and closed: valid. If anything remains, those brackets were never closed: invalid.

---

## Brute Force Solution

**Idea:** Repeatedly scan the string and remove valid adjacent pairs `()`, `[]`, `{}` until none remain. If the string becomes empty, it was valid.

```python
class Solution(object):
    def isValid(self, s):
        """
        :type s: str
        :rtype: bool
        """
        while "()" in s or "[]" in s or "{}" in s:
            s = s.replace("()", "").replace("[]", "").replace("{}", "")

        return s == ""
```

> **Why this works:** Valid pairs are always adjacent at some point. Removing innermost pairs first eventually reduces a valid string to empty.

**Time Complexity:** O(n²) — each `replace` is O(n), and the while loop runs up to O(n/2) times in the worst case.  
**Space Complexity:** O(n) — each `replace` creates a new string object in memory.

This passes LeetCode but a Google interviewer will ask you to improve it. The O(n²) comes from repeatedly scanning the string — we can do this in a single pass.

---

## Optimal Solution — Stack

**Idea:** One pass through the string. Push open brackets onto the stack. When a closing bracket is encountered, verify it matches the top of the stack. Return whether the stack is empty at the end.

```python
class Solution(object):
    def isValid(self, s):
        """
        :type s: str
        :rtype: bool
        """
        # Maps each closing bracket to its expected open bracket
        matches = {")": "(", "]": "[", "}": "{"}
        stack = []

        for char in s:
            if char in matches:
                # It's a closing bracket — check the top of the stack
                top = stack.pop() if stack else "#"
                if matches[char] != top:
                    return False
            else:
                # It's an open bracket — push onto stack
                stack.append(char)

        return not stack
```

> **Why `if stack else "#"`?**  
> If the stack is empty when we encounter a closing bracket (e.g. input `"]"`), there's nothing to pop. Instead of crashing with an `IndexError`, we assign a dummy value `"#"` that will never match any bracket, causing the comparison to fail cleanly and return `False`.

> **Why `return not stack`?**  
> After processing every character, any open brackets still on the stack were never matched. If the stack is empty, all brackets were matched — return `True`. If anything remains, return `False`. `not stack` evaluates to `True` when the list is empty and `False` when it has items.

---

## Step-by-Step Trace

Input: `s = "{[]}"`

| Step | `char` | Action | Stack after |
|------|--------|--------|-------------|
| 1 | `{` | Open bracket → push | `['{']` |
| 2 | `[` | Open bracket → push | `['{', '[']` |
| 3 | `]` | Closing → pop `[`, matches `]`'s expected `[` ✓ | `['{']` |
| 4 | `}` | Closing → pop `{`, matches `}`'s expected `{` ✓ | `[]` |
| End | — | Stack is empty → return `True` ✓ | |

Input: `s = "([)]"`

| Step | `char` | Action | Stack after |
|------|--------|--------|-------------|
| 1 | `(` | Open bracket → push | `['(']` |
| 2 | `[` | Open bracket → push | `['(', '[']` |
| 3 | `)` | Closing → pop `[`, expected `(` — **mismatch** → return `False` ✗ | |

---

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force (replace) | O(n²) | O(n) | Each replace is O(n); loop runs O(n/2) times. New string per iteration. |
| Stack (Optimal) | O(n) | O(n) | Single pass. Each character pushed/popped at most once. Stack holds at most n/2 open brackets in worst case. |

> **Can we do O(1) space?** Only for a single bracket type using a counter. With three types and nesting, the stack is irreducible — you cannot determine validity without remembering the sequence of open brackets seen so far.

---

## Edge Cases

| Input | Expected | Why |
|-------|----------|-----|
| `"()"` | `true` | Basic valid pair |
| `"([)]"` | `false` | Counts balance but order is wrong |
| `"]"` | `false` | Closing bracket with empty stack |
| `"((("` | `false` | Open brackets never closed — stack non-empty at end |
| `"([])"` | `true` | Nested brackets, correct order |
| `""` | `true` | Empty string — stack starts and ends empty |

---

## Common Mistakes

**Mistake 1 — Not handling the empty stack case on a closing bracket:**  
Calling `stack.pop()` on an empty list raises an `IndexError`. Always guard with `if stack else "#"` (or equivalent) before popping.

**Mistake 2 — Forgetting `return not stack` at the end:**  
Input `"((("` processes all characters without ever hitting a `return False` — but the string is clearly invalid. The stack check at the end is what catches unmatched open brackets. Never return `True` unconditionally after the loop.

**Mistake 3 — Using a counter instead of a stack:**  
A counter that increments on `(` and decrements on `)` works for a *single* bracket type but completely fails for mixed types. `"([)]"` would give a count of zero (appears balanced) but is invalid. Counters cannot track type — only stacks can.

**Mistake 4 — Checking `len(stack) == 0` instead of `not stack`:**  
Both are correct, but `not stack` is idiomatic Python. It reads more naturally and signals stronger Python fluency in a code review or interview setting.

---

## Follow-Up Questions

**Q: What if we added a new bracket type, like `<` and `>`?**  
Add one entry to the `matches` dictionary: `">": "<"`. The rest of the logic is completely unchanged. This is a deliberate design win — the hash map makes the solution extensible in O(1) effort.

**Q: What if the string could contain non-bracket characters (letters, spaces)?**  
Skip any character not in our known set. Add an early check: if `char not in matches and char not in "([{"`, continue to the next iteration. O(n) time is unchanged.

**Q: What is the minimum number of bracket additions to make a string valid? (LC #921)**  
After processing the string with the stack, the number of unmatched open brackets remaining on the stack equals the closing brackets needed. Track unmatched closing brackets (those that caused a mismatch) separately. Return `len(stack) + unmatched_close`.

---

## Test It Yourself

```python
class Solution(object):
    def isValid(self, s):
        """
        :type s: str
        :rtype: bool
        """
        matches = {")": "(", "]": "[", "}": "{"}
        stack = []

        for char in s:
            if char in matches:
                top = stack.pop() if stack else "#"
                if matches[char] != top:
                    return False
            else:
                stack.append(char)

        return not stack


# ==================== Test Cases ====================
sol = Solution()

print(sol.isValid("()"))        # Expected: True
print(sol.isValid("()[]{}"))    # Expected: True
print(sol.isValid("(]"))        # Expected: False
print(sol.isValid("([])"))      # Expected: True
print(sol.isValid("([)]"))      # Expected: False
print(sol.isValid("{[]}"))      # Expected: True
print(sol.isValid("]"))         # Expected: False  (closing bracket, empty stack)
print(sol.isValid("((("))       # Expected: False  (unmatched open brackets)
print(sol.isValid(""))          # Expected: True   (empty string)
```

---

## Key Takeaways

- **Stack = "most recently seen"**. Whenever a problem involves matching against the most recent item, a stack is the natural data structure.
- The `matches` dictionary approach makes the solution clean, readable, and easily extensible to new bracket types.
- The two critical guards: `if stack else "#"` before popping, and `return not stack` after the loop.
- This stack pattern appears in: Largest Rectangle in Histogram (LC #84), Daily Temperatures (LC #739), Decode String (LC #394), and many more.

---

*Google Easy Track · Episode 02 of 08*
