# Best Time to Buy and Sell Stock

**LeetCode #121 · Difficulty: Easy · Category: Arrays**

---

## Problem Statement

You are given an array `prices` where `prices[i]` is the price of a given stock on the `i`th day. You want to maximize your profit by choosing a single day to buy one stock and choosing a different day in the future to sell that stock.

Return the maximum profit you can achieve from this transaction. If you cannot achieve any profit, return `0`.

**Example 1:**
```
Input:  prices = [7,1,5,3,6,4]
Output: 5
Explanation: Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6 - 1 = 5.
Note that buying on day 2 and selling on day 1 is not allowed because you must buy before you sell.
```

**Example 2:**
```
Input:  prices = [7,6,4,3,1]
Output: 0
Explanation: Prices only decrease. No profitable transaction exists, so return 0.
```

**Constraints:**
- `1 <= prices.length <= 10^5`
- `0 <= prices[i] <= 10^4`

---

## Clarifying Questions

Before writing a single line of code, ask these in an interview:

- **Can prices be negative?** No, the constraints guarantee `prices[i] >= 0`.
- **Must I buy before I sell?** Yes. You must buy on an earlier day and sell on a later day.
- **Can I buy and sell on the same day?** No. The problem says "a different day in the future."
- **Can I make multiple transactions?** No. Exactly one buy and one sell.
- **What if no profit is possible?** Return `0`. Never return a negative number.
- **Can the array have just one element?** Yes, `prices.length >= 1`. With only one price, no transaction is possible, so return `0`.

---

## What the Problem Is Really Asking

To maximize profit on any given sell day, you want to have bought at the lowest price seen so far before that day. As you scan left to right, you are tracking two things simultaneously:

1. The minimum price seen so far (the best possible buy day up to this point)
2. The maximum profit achievable if you sold today

You never need to look backwards. At every price, you already know the best possible buy price up to that point. The profit at each step is simply `current_price - min_price_so_far`. Keep the best one seen and you have your answer in a single pass.

---

## Pseudocode

### Brute Force

```
SET max_profit = 0

FOR each index i from 0 to end of array:
    FOR each index j from i+1 to end of array:
        profit = prices[j] - prices[i]
        IF profit > max_profit:
            UPDATE max_profit = profit

RETURN max_profit
```

### Optimal (Running Minimum)

```
SET min_price = infinity
SET max_profit = 0

FOR each price in prices:
    IF price < min_price:
        UPDATE min_price = price        # found a cheaper buy day
    ELSE IF price - min_price > max_profit:
        UPDATE max_profit = price - min_price

RETURN max_profit
```

---

## Brute Force Solution

**Idea:** Try every possible combination of buy day `i` and sell day `j` where `j > i`. Track the best profit seen.

```python
class Solution(object):
    def maxProfit(self, prices):
        """
        :type prices: List[int]
        :rtype: int
        """
        max_profit = 0

        for i in range(len(prices)):
            for j in range(i + 1, len(prices)):
                profit = prices[j] - prices[i]
                if profit > max_profit:
                    max_profit = profit

        return max_profit
```

This correctly solves the problem but ignores a fundamental property: we do not need to try every pair. For a given sell day `j`, the best buy day is simply whichever earlier day had the lowest price. Scanning backwards for it on every iteration is the waste the optimal solution eliminates.

**Time Complexity:** O(n²) — for each of n prices, we scan up to n others.  
**Space Complexity:** O(1) — no extra data structures.

---

## Optimal Solution — Single Pass (Running Minimum)

**Idea:** Track the minimum price seen so far as you iterate. At each price, compute what your profit would be if you sold today. Update the best profit if it beats the current maximum.

```python
class Solution(object):
    def maxProfit(self, prices):
        """
        :type prices: List[int]
        :rtype: int
        """
        min_price = float('inf')
        max_profit = 0

        for price in prices:
            if price < min_price:
                min_price = price
            elif price - min_price > max_profit:
                max_profit = price - min_price

        return max_profit
```

This is one of the rare optimal solutions with both O(n) time and O(1) space. Two scalar variables, one pass, no extra storage of any kind.

---

## Step-by-Step Trace

Input: `prices = [7, 1, 5, 3, 6, 4]`

| Day | Price | `min_price` | `price - min_price` | `max_profit` |
|-----|-------|-------------|----------------------|--------------|
| 0 | 7 | 7 | 0 | 0 |
| 1 | 1 | 1 | 0 | 0 |
| 2 | 5 | 1 | 4 | 4 |
| 3 | 3 | 1 | 2 | 4 |
| 4 | 6 | 1 | 5 | 5 |
| 5 | 4 | 1 | 3 | 5 |

Return `5`. Buy on day 1 (price = 1), sell on day 4 (price = 6). ✓

Input: `prices = [7, 6, 4, 3, 1]`

| Day | Price | `min_price` | `price - min_price` | `max_profit` |
|-----|-------|-------------|----------------------|--------------|
| 0 | 7 | 7 | 0 | 0 |
| 1 | 6 | 6 | 0 | 0 |
| 2 | 4 | 4 | 0 | 0 |
| 3 | 3 | 3 | 0 | 0 |
| 4 | 1 | 1 | 0 | 0 |

Return `0`. Prices only decrease, no profitable transaction possible. ✓

---

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force | O(n²) | O(1) | All pairs checked. For `n = 10^5`, this is 10 billion operations. |
| Single Pass (Optimal) | O(n) | O(1) | One loop, two scalars. No extra storage regardless of input size. |

This is one of the few problems where the optimal solution improves both time and space over a more complex approach. There is nothing to trade off here.

---

## Edge Cases

| Input | Expected | Why |
|-------|----------|-----|
| `[7,6,4,3,1]` | `0` | Strictly decreasing. No profitable transaction. |
| `[1]` | `0` | Single element. Cannot buy and sell on the same day. |
| `[2,4]` | `2` | Minimum length valid transaction. |
| `[1,2]` | `1` | Buy on day 0, sell on day 1. |
| `[2,1,2,1,2]` | `1` | Multiple local peaks. Best profit is still only 1. |
| `[0,0,0]` | `0` | All zeros. Profit is always 0. |

---

## Common Mistakes

**Mistake 1 — Initializing `min_price = 0` instead of `float('inf')`:**  
If you start with `min_price = 0`, then on the first iteration `price < min_price` is false for any non-negative price, so `min_price` never gets updated. The result will always be `0`. Using `float('inf')` guarantees that the very first price in the array always becomes the initial minimum.

**Mistake 2 — Allowing a negative profit to be returned:**  
Initializing `max_profit = 0` handles this correctly. If every possible transaction results in a loss, `max_profit` stays at `0` and that is what gets returned. Never initialize `max_profit` to a negative value or to `prices[0]`.

**Mistake 3 — Updating `min_price` after computing the profit:**  
The order within the loop matters conceptually. The correct mental model is: first check if today is a better buy day, then check if selling today beats your current best. If you compute the profit before updating `min_price`, you are asking "what is my profit if I buy and sell on the same day" at the first step.

**Mistake 4 — Starting `j` at `i` instead of `i + 1` in the brute force:**  
This allows buying and selling on the same day, which the problem forbids. Always ensure `j > i`.

---

## Follow-Up Questions

**Q: What if you can make at most two transactions? (LC #123)**  
Track four variables across a single pass: `buy1`, `sell1`, `buy2`, `sell2`. Update them greedily. This is a direct escalation of the running minimum pattern into a two-phase problem.

**Q: What if you can make unlimited transactions? (LC #122)**  
Sum every positive day-over-day gain. If `prices[i] > prices[i-1]`, add the difference. One pass, O(n) time, O(1) space.

**Q: What if there is a cooldown period after selling? (LC #309)**  
The greedy approach breaks down. This becomes a dynamic programming problem with states representing whether you are holding, sold, or in cooldown.

**Q: What if you can make at most k transactions? (LC #188)**  
Full DP with a 2D state table: `dp[k][n]` representing maximum profit using at most `k` transactions through day `n`.

---

## Test It Yourself

```python
class Solution(object):
    def maxProfit(self, prices):
        """
        :type prices: List[int]
        :rtype: int
        """
        min_price = float('inf')
        max_profit = 0

        for price in prices:
            if price < min_price:
                min_price = price
            elif price - min_price > max_profit:
                max_profit = price - min_price

        return max_profit


# ==================== Test Cases ====================
sol = Solution()

print(sol.maxProfit([7, 1, 5, 3, 6, 4]))   # Expected: 5
print(sol.maxProfit([7, 6, 4, 3, 1]))       # Expected: 0
print(sol.maxProfit([1]))                   # Expected: 0
print(sol.maxProfit([2, 4]))                # Expected: 2
print(sol.maxProfit([2, 1, 2, 1, 2]))       # Expected: 1
print(sol.maxProfit([0, 0, 0]))             # Expected: 0
print(sol.maxProfit([1, 2]))                # Expected: 1
```

---

## Key Takeaways

- The **running minimum** pattern reduces an O(n²) pair search to a single O(n) pass. Any time you need the best value seen so far to compute something at the current position, consider tracking a running minimum or maximum.
- Initializing `min_price = float('inf')` is the safe, universal approach for finding a minimum across any input.
- This problem is the entry point to the entire stock series (LC #122, #123, #188, #309). Understanding the running minimum here makes each variant approachable.

---

*Google Easy Track · Tutorial 04 of 08*
