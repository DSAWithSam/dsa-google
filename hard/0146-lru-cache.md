# LRU Cache

**LeetCode #146 · Difficulty: Hard · Category: Design & Data Structures**

---

## Problem Statement

Design a data structure that follows the constraints of a **Least Recently Used (LRU) cache**.

Implement the `LRUCache` class:
- `LRUCache(int capacity)` — initialise the LRU cache with positive size `capacity`.
- `int get(int key)` — return the value of `key` if it exists, otherwise return `-1`.
- `void put(int key, int value)` — update the value of `key` if it exists. Otherwise add the key-value pair. If the number of keys exceeds `capacity`, evict the least recently used key.

Both `get` and `put` must each run in **O(1) average time complexity**.

**Example 1:**
```
Input:
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1,1], [2,2], [1], [3,3], [2], [4,4], [1], [3], [4]]

Output:
[null, null, null, 1, null, -1, null, -1, 3, 4]

Walkthrough:
LRUCache(2)       # capacity = 2
put(1, 1)         # cache: {1=1}
put(2, 2)         # cache: {1=1, 2=2}
get(1)            # return 1,  cache: {2=2, 1=1}  (1 is now most recent)
put(3, 3)         # evict 2 (LRU), cache: {1=1, 3=3}
get(2)            # return -1  (2 was evicted)
put(4, 4)         # evict 1 (LRU), cache: {3=3, 4=4}
get(1)            # return -1  (1 was evicted)
get(3)            # return 3
get(4)            # return 4
```

**Constraints:**
- `1 <= capacity <= 3000`
- `0 <= key <= 10^4`
- `0 <= value <= 10^5`
- At most `2 * 10^5` calls will be made to `get` and `put`

---

## Clarifying Questions

Before writing a single line of code, ask these in an interview:

- **What should `get` return for a missing key?** Return `-1`.
- **Does a `get` count as a "use" that updates recency?** Yes. Both `get` and `put` on an existing key make it the most recently used.
- **What happens when `put` is called on a key that already exists?** Update the value and move it to most recently used. Do not evict anything.
- **What is the minimum capacity?** `1`. Your solution must handle a cache that holds only a single item.
- **Are keys and values always non-negative integers?** Yes, per the constraints.
- **What should happen when `put` is called and the cache is already at capacity?** Evict the least recently used key before inserting the new one.

---

## What the Problem Is Really Asking

An LRU cache must do two things simultaneously in O(1) time:

1. **Look up a key instantly** — this points to a hash map.
2. **Track the order of recent use and evict the oldest instantly** — this points to a doubly linked list.

Neither structure alone is sufficient:
- A hash map gives O(1) lookup but has no ordering.
- A linked list gives O(1) insertion and deletion at known positions but O(n) lookup.

The solution is to combine them. The hash map stores each key mapped to its corresponding node in the doubly linked list. The linked list maintains the usage order, with the most recently used node at one end and the least recently used at the other. When you need to access, update, or evict a node, the hash map gives you a direct pointer to it in O(1), and the linked list lets you reorder or remove it in O(1).

---

## The Data Structure Design

```
Hash map:  { key -> node }

Doubly linked list (head = LRU, tail = MRU):

  [dummy_head] <-> [node_A] <-> [node_B] <-> [node_C] <-> [dummy_tail]
       LRU end                                                MRU end
```

Two sentinel (dummy) nodes anchor the list at both ends. They have no real data but eliminate the need to check for null pointers when inserting or removing at the boundaries. Every real node sits between the two sentinels.

**Operations:**
- `get(key)`: look up node in map. If found, move to tail (most recent). Return value. If not found, return -1.
- `put(key, value)`: if key exists, update value and move to tail. If key does not exist, create a new node, add to tail, add to map. If at capacity, remove the node just after the dummy head (the LRU node), remove from map.

---

## Pseudocode

```
CLASS Node:
    key, value, prev, next

CLASS LRUCache:
    capacity, size, map
    dummy_head <-> dummy_tail  (sentinels)

    FUNCTION _remove(node):
        # Detach node from its current position
        node.prev.next = node.next
        node.next.prev = node.prev

    FUNCTION _insert_at_tail(node):
        # Insert node just before dummy_tail
        node.prev = dummy_tail.prev
        node.next = dummy_tail
        dummy_tail.prev.next = node
        dummy_tail.prev = node

    FUNCTION get(key):
        IF key not in map: RETURN -1
        node = map[key]
        _remove(node)
        _insert_at_tail(node)
        RETURN node.value

    FUNCTION put(key, value):
        IF key in map:
            node = map[key]
            node.value = value
            _remove(node)
            _insert_at_tail(node)
        ELSE:
            node = new Node(key, value)
            map[key] = node
            _insert_at_tail(node)
            size += 1
            IF size > capacity:
                lru = dummy_head.next
                _remove(lru)
                del map[lru.key]
                size -= 1
```

---

## Brute Force Approach

**Idea:** Use an ordered dictionary or a plain list to track usage order. Python's `collections.OrderedDict` maintains insertion order and allows moving items to the end, giving a simpler but less instructive implementation.

```python
from collections import OrderedDict

class LRUCache(object):
    def __init__(self, capacity):
        """
        :type capacity: int
        """
        self.capacity = capacity
        self.cache = OrderedDict()

    def get(self, key):
        """
        :type key: int
        :rtype: int
        """
        if key not in self.cache:
            return -1
        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key, value):
        """
        :type key: int
        :type value: int
        :rtype: None
        """
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)
```

`OrderedDict.move_to_end()` and `popitem(last=False)` are O(1) operations in CPython's implementation, so this technically meets the O(1) constraint. However, this is not the answer an interviewer is looking for. `OrderedDict` is a built-in abstraction over exactly the doubly linked list and hash map combination described above. Knowing how to build it from scratch is what differentiates candidates.

**Time Complexity:** O(1) average for both `get` and `put`.  
**Space Complexity:** O(capacity) — the cache holds at most `capacity` entries.

---

## Optimal Solution — Doubly Linked List and Hash Map

**Idea:** Build the underlying data structure explicitly. A doubly linked list with sentinel nodes maintains usage order in O(1). A hash map provides O(1) access to any node by key.

```python
class LRUCache(object):

    class Node(object):
        def __init__(self, key=0, value=0):
            self.key = key
            self.value = value
            self.prev = None
            self.next = None

    def __init__(self, capacity):
        """
        :type capacity: int
        """
        self.capacity = capacity
        self.size = 0
        self.map = {}

        # Sentinel nodes — never hold real data
        self.head = self.Node()   # dummy head (LRU end)
        self.tail = self.Node()   # dummy tail (MRU end)
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node):
        """Detach a node from its current position in the list."""
        node.prev.next = node.next
        node.next.prev = node.prev

    def _insert_at_tail(self, node):
        """Insert a node just before the dummy tail (most recently used position)."""
        node.prev = self.tail.prev
        node.next = self.tail
        self.tail.prev.next = node
        self.tail.prev = node

    def get(self, key):
        """
        :type key: int
        :rtype: int
        """
        if key not in self.map:
            return -1
        node = self.map[key]
        self._remove(node)
        self._insert_at_tail(node)
        return node.value

    def put(self, key, value):
        """
        :type key: int
        :type value: int
        :rtype: None
        """
        if key in self.map:
            node = self.map[key]
            node.value = value
            self._remove(node)
            self._insert_at_tail(node)
        else:
            node = self.Node(key, value)
            self.map[key] = node
            self._insert_at_tail(node)
            self.size += 1

            if self.size > self.capacity:
                lru = self.head.next          # node just after dummy head is LRU
                self._remove(lru)
                del self.map[lru.key]
                self.size -= 1
```

**Time Complexity:** O(1) for both `get` and `put`. Every operation is a fixed number of pointer reassignments and hash map lookups.  
**Space Complexity:** O(capacity) — the map and list each hold at most `capacity` real nodes.

---

## Why a Doubly Linked List

A singly linked list is not sufficient. When you remove a node from the middle of a singly linked list, you need to access its predecessor to update its `next` pointer. Finding the predecessor requires an O(n) scan. With a doubly linked list, every node holds a direct reference to its predecessor via `node.prev`, so removal is always O(1).

The sentinel (dummy) nodes at both ends eliminate all boundary condition checks. Without them, inserting at the tail and removing from the head require special handling when the list is empty or has one element. With sentinels, the pointer logic is identical in all cases.

---

## Step-by-Step Trace

Input: `capacity = 2`, operations from Example 1

```
Initial state:
  map: {}
  list: [HEAD] <-> [TAIL]

put(1, 1):
  New node (k=1,v=1). Insert at tail.
  list: [HEAD] <-> [1,1] <-> [TAIL]
  map: {1: node(1,1)}
  size=1, not over capacity.

put(2, 2):
  New node (k=2,v=2). Insert at tail.
  list: [HEAD] <-> [1,1] <-> [2,2] <-> [TAIL]
  map: {1: node(1,1), 2: node(2,2)}
  size=2, not over capacity.

get(1):
  key=1 found. Remove from current position, insert at tail.
  list: [HEAD] <-> [2,2] <-> [1,1] <-> [TAIL]
  Return 1.

put(3, 3):
  New node (k=3,v=3). Insert at tail.
  list: [HEAD] <-> [2,2] <-> [1,1] <-> [3,3] <-> [TAIL]
  size=3 > capacity=2. Evict head.next = node(2,2).
  Remove node(2,2), delete map[2].
  list: [HEAD] <-> [1,1] <-> [3,3] <-> [TAIL]
  map: {1: node(1,1), 3: node(3,3)}
  size=2.

get(2):
  key=2 not in map. Return -1.

put(4, 4):
  New node (k=4,v=4). Insert at tail.
  list: [HEAD] <-> [1,1] <-> [3,3] <-> [4,4] <-> [TAIL]
  size=3 > capacity=2. Evict head.next = node(1,1).
  Remove node(1,1), delete map[1].
  list: [HEAD] <-> [3,3] <-> [4,4] <-> [TAIL]
  map: {3: node(3,3), 4: node(4,4)}
  size=2.

get(1): -1 (evicted)
get(3): return 3, move to tail. list: [HEAD] <-> [4,4] <-> [3,3] <-> [TAIL]
get(4): return 4, move to tail. list: [HEAD] <-> [3,3] <-> [4,4] <-> [TAIL]
```

---

## Complexity Analysis

| Operation | Time | Space | Notes |
|-----------|------|-------|-------|
| `get` | O(1) | O(1) | Hash map lookup + two pointer reassignments. |
| `put` (existing key) | O(1) | O(1) | Hash map lookup + remove + insert at tail. |
| `put` (new key) | O(1) | O(1) | New node + insert at tail + optional eviction. |
| Overall space | O(capacity) | | Map and list each hold at most capacity nodes. |

---

## Edge Cases

| Scenario | Expected Behaviour |
|----------|--------------------|
| `get` on a key that was never inserted | Return `-1`. |
| `put` on a key that already exists | Update value, move to most recently used. Do not increment size. |
| `put` when cache is exactly at capacity | Evict LRU before inserting the new key. |
| `capacity = 1` | Every `put` of a new key evicts the previous single entry. |
| `get` after a key is evicted | Return `-1`. |
| `put` the same key twice in a row | Second call updates value and marks it as most recently used. No eviction. |

---

## Common Mistakes

**Mistake 1 — Storing only the value in the map instead of the node:**  
If `map[key] = value`, you cannot remove the node from the linked list in O(1) because you have no pointer to it. The map must store `map[key] = node` so that both the value and the list position are accessible instantly.

**Mistake 2 — Forgetting to store the key inside the node:**  
When evicting the LRU node (`head.next`), you need to remove it from the map using `del self.map[lru.key]`. If the node does not store its own key, you cannot do this without an O(n) reverse lookup. Always store the key inside the node.

**Mistake 3 — Not moving an existing key to the tail on `put`:**  
When `put` is called with a key that already exists, you must update the value and move the node to the most recently used position. Failing to move it means the node retains its old position in the recency order, which leads to incorrect eviction.

**Mistake 4 — Not moving a node to the tail on `get`:**  
A successful `get` is a use of that key and must update its recency. Returning the value without repositioning the node causes it to be evicted too early.

**Mistake 5 — Incrementing size for an existing key on `put`:**  
If the key already exists, you are updating, not inserting. The size does not change. Only increment size when a genuinely new key is added.

**Mistake 6 — Using sentinel nodes incorrectly:**  
The `_insert_at_tail` function inserts just before the dummy tail, not after it. The `_remove` function works on any node including the real nodes, but must never be called on the sentinels themselves. Mixing up head/tail directions causes the LRU end and MRU end to be reversed, resulting in evicting the most recently used key instead of the least.

---

## Follow-Up Questions

**Q: How would you implement an LFU (Least Frequently Used) cache? (LC #460)**  
LFU evicts the key with the lowest access frequency rather than the least recent access. It requires tracking frequency counts and, within the same frequency, still evicting the least recently used. This requires a hash map of frequencies to doubly linked lists, plus a pointer to the minimum frequency bucket.

**Q: What if multiple threads call `get` and `put` concurrently?**  
The current implementation is not thread-safe. You would need to add a lock (e.g. Python's `threading.Lock`) around each operation, or use a thread-safe ordered dictionary abstraction.

**Q: How would you support a `peek` operation that returns a value without updating recency?**  
Add a `peek(key)` method that looks up the node in the map and returns `node.value` without calling `_remove` or `_insert_at_tail`. The recency order remains unchanged.

---

## Test It Yourself

```python
class LRUCache(object):

    class Node(object):
        def __init__(self, key=0, value=0):
            self.key = key
            self.value = value
            self.prev = None
            self.next = None

    def __init__(self, capacity):
        """
        :type capacity: int
        """
        self.capacity = capacity
        self.size = 0
        self.map = {}
        self.head = self.Node()
        self.tail = self.Node()
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def _insert_at_tail(self, node):
        node.prev = self.tail.prev
        node.next = self.tail
        self.tail.prev.next = node
        self.tail.prev = node

    def get(self, key):
        """
        :type key: int
        :rtype: int
        """
        if key not in self.map:
            return -1
        node = self.map[key]
        self._remove(node)
        self._insert_at_tail(node)
        return node.value

    def put(self, key, value):
        """
        :type key: int
        :type value: int
        :rtype: None
        """
        if key in self.map:
            node = self.map[key]
            node.value = value
            self._remove(node)
            self._insert_at_tail(node)
        else:
            node = self.Node(key, value)
            self.map[key] = node
            self._insert_at_tail(node)
            self.size += 1
            if self.size > self.capacity:
                lru = self.head.next
                self._remove(lru)
                del self.map[lru.key]
                self.size -= 1


# ==================== Test Cases ====================
lru = LRUCache(2)

lru.put(1, 1)
lru.put(2, 2)
print(lru.get(1))    # Expected: 1
lru.put(3, 3)
print(lru.get(2))    # Expected: -1  (evicted)
lru.put(4, 4)
print(lru.get(1))    # Expected: -1  (evicted)
print(lru.get(3))    # Expected: 3
print(lru.get(4))    # Expected: 4

print("---")

# Capacity 1
lru2 = LRUCache(1)
lru2.put(1, 1)
print(lru2.get(1))   # Expected: 1
lru2.put(2, 2)
print(lru2.get(1))   # Expected: -1  (evicted)
print(lru2.get(2))   # Expected: 2

print("---")

# Update existing key
lru3 = LRUCache(2)
lru3.put(1, 1)
lru3.put(2, 2)
lru3.put(1, 10)       # update key 1, should not evict anything
print(lru3.get(1))   # Expected: 10
print(lru3.get(2))   # Expected: 2
```

---

## Key Takeaways

- LRU cache requires O(1) lookup and O(1) ordered eviction simultaneously. Neither a hash map nor a linked list alone achieves both. The combination does.
- The hash map stores `key -> node` (not `key -> value`) so that the linked list position is directly accessible.
- Every node stores its own key so that eviction can remove the node from the map without a reverse lookup.
- Sentinel (dummy) head and tail nodes eliminate boundary checks and make the pointer logic uniform for all insertions and removals.
- This design pattern (hash map plus doubly linked list) also appears in LFU Cache (LC #460) and is directly relevant to production systems work involving caching layers, CDN design, and database buffer pool management.

---

*Google Hard Track · Tutorial 03 of 05*
