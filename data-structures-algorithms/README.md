# 🧮 Data Structures & Algorithms Reference

## Big-O Complexity Cheat Sheet

| Structure         | Access | Search | Insert | Delete | Space  |
|-------------------|--------|--------|--------|--------|--------|
| Array             | O(1)   | O(n)   | O(n)   | O(n)   | O(n)   |
| Dynamic Array     | O(1)   | O(n)   | O(1)*  | O(n)   | O(n)   |
| Linked List       | O(n)   | O(n)   | O(1)   | O(1)   | O(n)   |
| Stack             | O(n)   | O(n)   | O(1)   | O(1)   | O(n)   |
| Queue             | O(n)   | O(n)   | O(1)   | O(1)   | O(n)   |
| Hash Table        | N/A    | O(1)*  | O(1)*  | O(1)*  | O(n)   |
| Binary Search Tree| O(log n)| O(log n)| O(log n)| O(log n)| O(n) |
| Balanced BST      | O(log n)| O(log n)| O(log n)| O(log n)| O(n)|
| Heap (Binary)     | O(1)   | O(n)   | O(log n)| O(log n)| O(n) |
| Trie              | O(k)   | O(k)   | O(k)   | O(k)   | O(n*k) |

*amortized

## Sorting Algorithms

| Algorithm      | Best     | Average  | Worst    | Space  | Stable |
|----------------|----------|----------|----------|--------|--------|
| Bubble Sort    | O(n)     | O(n²)    | O(n²)    | O(1)   | Yes    |
| Selection Sort | O(n²)    | O(n²)    | O(n²)    | O(1)   | No     |
| Insertion Sort | O(n)     | O(n²)    | O(n²)    | O(1)   | Yes    |
| Merge Sort     | O(n log n)| O(n log n)| O(n log n)| O(n) | Yes  |
| Quick Sort     | O(n log n)| O(n log n)| O(n²)  | O(log n)| No   |
| Heap Sort      | O(n log n)| O(n log n)| O(n log n)| O(1)| No   |
| Tim Sort       | O(n)     | O(n log n)| O(n log n)| O(n)| Yes  |
| Counting Sort  | O(n+k)   | O(n+k)   | O(n+k)   | O(k)   | Yes    |
| Radix Sort     | O(nk)    | O(nk)    | O(nk)    | O(n+k) | Yes    |

## Arrays

```
Key operations:
- Two-pointer technique: O(n)
- Sliding window: O(n)
- Binary search (sorted): O(log n)
- Kadane's algorithm (max subarray): O(n)
- Prefix sums: O(n) preprocess, O(1) range query
```

```python
# Binary Search
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = left + (right - left) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1

# Prefix Sum
def prefix_sum(arr):
    prefix = [0] * (len(arr) + 1)
    for i, val in enumerate(arr):
        prefix[i+1] = prefix[i] + val
    return prefix

# range sum [l, r]
# prefix[r+1] - prefix[l]
```

## Linked List

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val  = val
        self.next = next

# Reverse
def reverse(head):
    prev, curr = None, head
    while curr:
        nxt       = curr.next
        curr.next = prev
        prev      = curr
        curr      = nxt
    return prev

# Floyd's cycle detection
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False
```

## Stack & Queue

```python
# Stack (LIFO) — use list
stack = []
stack.append(x)    # push
stack.pop()        # pop
stack[-1]          # peek

# Queue (FIFO) — use deque
from collections import deque
queue = deque()
queue.append(x)    # enqueue
queue.popleft()    # dequeue
queue[0]           # peek

# Monotonic stack (next greater element)
def next_greater(arr):
    res, stack = [-1] * len(arr), []
    for i, val in enumerate(arr):
        while stack and arr[stack[-1]] < val:
            res[stack.pop()] = val
        stack.append(i)
    return res
```

## Trees

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val   = val
        self.left  = left
        self.right = right

# DFS Traversals
def inorder(root):      # left-root-right
    if not root: return
    inorder(root.left)
    visit(root.val)
    inorder(root.right)

def preorder(root):     # root-left-right
    if not root: return
    visit(root.val)
    preorder(root.left)
    preorder(root.right)

def postorder(root):    # left-right-root
    if not root: return
    postorder(root.left)
    postorder(root.right)
    visit(root.val)

# BFS (Level-order)
from collections import deque
def bfs(root):
    if not root: return
    q = deque([root])
    while q:
        node = q.popleft()
        visit(node.val)
        if node.left:  q.append(node.left)
        if node.right: q.append(node.right)

# Height of tree
def height(root):
    if not root: return 0
    return 1 + max(height(root.left), height(root.right))
```

## Graphs

```python
# Adjacency list representation
graph = {
    0: [1, 2],
    1: [0, 3],
    2: [0],
    3: [1]
}

# DFS
def dfs(graph, node, visited=None):
    if visited is None: visited = set()
    visited.add(node)
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)
    return visited

# BFS
from collections import deque
def bfs(graph, start):
    visited = {start}
    queue   = deque([start])
    while queue:
        node = queue.popleft()
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    return visited

# Topological Sort (Kahn's algorithm)
from collections import deque
def topo_sort(graph, num_nodes):
    indegree = [0] * num_nodes
    for node in graph:
        for neighbor in graph[node]:
            indegree[neighbor] += 1

    queue = deque(n for n in range(num_nodes) if indegree[n] == 0)
    order = []
    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            indegree[neighbor] -= 1
            if indegree[neighbor] == 0:
                queue.append(neighbor)
    return order if len(order) == num_nodes else []  # empty if cycle
```

## Heap / Priority Queue

```python
import heapq

# Min-heap
heap = []
heapq.heappush(heap, val)
heapq.heappop(heap)     # returns smallest
heap[0]                 # peek smallest

# Max-heap (negate values)
heapq.heappush(heap, -val)
-heapq.heappop(heap)

# Heapify in-place
heapq.heapify(arr)

# K largest elements
heapq.nlargest(k, arr)

# K smallest elements
heapq.nsmallest(k, arr)
```

## Hash Map Patterns

```python
from collections import Counter, defaultdict

# Frequency count
freq = Counter(arr)
freq.most_common(3)     # top 3

# Default dict
d = defaultdict(int)
d = defaultdict(list)
d = defaultdict(set)

# Two-sum
def two_sum(nums, target):
    seen = {}
    for i, n in enumerate(nums):
        if target - n in seen:
            return [seen[target - n], i]
        seen[n] = i
```

## Dynamic Programming Patterns

```
Common patterns:
1. Fibonacci / 1D DP
2. 0/1 Knapsack (2D DP)
3. Longest Common Subsequence (LCS)
4. Longest Increasing Subsequence (LIS)
5. Coin Change (unbounded knapsack)
6. Matrix path (grid DP)
7. Interval DP
8. Bitmask DP
```

```python
# Fibonacci (bottom-up)
def fib(n):
    if n <= 1: return n
    dp = [0, 1]
    for i in range(2, n+1):
        dp.append(dp[-1] + dp[-2])
    return dp[n]

# 0/1 Knapsack
def knapsack(weights, values, capacity):
    n  = len(weights)
    dp = [[0]*(capacity+1) for _ in range(n+1)]
    for i in range(1, n+1):
        for w in range(capacity+1):
            dp[i][w] = dp[i-1][w]
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i][w], values[i-1] + dp[i-1][w-weights[i-1]])
    return dp[n][capacity]

# Longest Common Subsequence
def lcs(s1, s2):
    m, n = len(s1), len(s2)
    dp   = [[0]*(n+1) for _ in range(m+1)]
    for i in range(1, m+1):
        for j in range(1, n+1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]
```

## Backtracking Template

```python
def backtrack(state, choices):
    if is_goal(state):
        results.append(state[:])
        return
    for choice in choices:
        if is_valid(state, choice):
            state.append(choice)
            backtrack(state, choices)
            state.pop()       # undo
```

## Union-Find (Disjoint Set Union)

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank   = [0] * n

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # path compression
        return self.parent[x]

    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py: return False
        if self.rank[px] < self.rank[py]:
            px, py = py, px
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1
        return True
```
