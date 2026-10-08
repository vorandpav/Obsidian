# Heaps

> Минимум/максимум за O(log n). Top-K, merge, медиана потока.

---

## Основы

```python
import heapq

heap = []                    # мин-куча (по умолчанию в Python)
heapq.heappush(heap, 3)
heapq.heappush(heap, 1)
min_val = heapq.heappop(heap)  # 1

# Макс-куча: инвертировать знак
heapq.heappush(heap, -val)
max_val = -heapq.heappop(heap)

# Из списка за O(n)
heapq.heapify(nums)

# K-й наименьший без полной сортировки
kth = heapq.nsmallest(k, nums)
```

| Операция | Сложность |
|----------|-----------|
| push | O(log n) |
| pop | O(log n) |
| peek (min) | O(1) |
| heapify | O(n) |

---

## Top-K элементов (LC 215, 347)

### K-й наибольший — min-heap размера k

```python
def find_kth_largest(nums, k):
    heap = []
    for num in nums:
        heapq.heappush(heap, num)
        if len(heap) > k:
            heapq.heappop(heap)
    return heap[0]
```

**Логика:** в min-heap размера k остаются k наибольших.

### Top-K frequent (LC 347)

```python
from collections import Counter

def top_k_frequent(nums, k):
    count = Counter(nums)
    return heapq.nlargest(k, count.keys(), key=count.get)
```

---

## Merge K Sorted Lists (LC 23)

```python
def merge_k_lists(lists):
    heap = []
    for i, node in enumerate(lists):
        if node:
            heapq.heappush(heap, (node.val, i, node))

    dummy = curr = ListNode(0)
    while heap:
        val, i, node = heapq.heappop(heap)
        curr.next = node
        curr = curr.next
        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))
    return dummy.next
```

O(N log k), N — всего элементов, k — списков.

---

## K Closest Points (LC 973)

```python
def k_closest(points, k):
    heap = []
    for x, y in points:
        dist = -(x*x + y*y)   # макс-куча через инверсию
        heapq.heappush(heap, (dist, x, y))
        if len(heap) > k:
            heapq.heappop(heap)
    return [(x, y) for _, x, y in heap]
```

---

## Find Median from Data Stream (LC 295)

Две кучи: max-heap для левой половины, min-heap для правой.

```python
class MedianFinder:
    def __init__(self):
        self.lo = []   # max-heap (инверсия)
        self.hi = []   # min-heap

    def addNum(self, num):
        heapq.heappush(self.lo, -num)
        heapq.heappush(self.hi, -heapq.heappop(self.lo))
        if len(self.hi) > len(self.lo):
            heapq.heappush(self.lo, -heapq.heappop(self.hi))

    def findMedian(self):
        if len(self.lo) > len(self.hi):
            return -self.lo[0]
        return (-self.lo[0] + self.hi[0]) / 2
```

---

## Dijkstra (кратчайший путь) — через heap

```python
import heapq

def dijkstra(graph, start):
    dist = {start: 0}
    heap = [(0, start)]
    while heap:
        d, u = heapq.heappop(heap)
        if d > dist.get(u, float('inf')):
            continue
        for v, w in graph[u]:
            nd = d + w
            if nd < dist.get(v, float('inf')):
                dist[v] = nd
                heapq.heappush(heap, (nd, v))
    return dist
```

---

## Когда heap vs sort?

| | Sort | Heap |
|--|------|------|
| Top-1 | O(n) max | O(n) heapify + pop |
| Top-K | O(n log n) | O(n log k) |
| K-й элемент | O(n log n) | O(n log k) или quickselect O(n) |
| Streaming | ❌ | ✅ |

---

## Вопросы на собесе

**Почему Python только min-heap?**
> Исторически; макс-куча через `-val`.

**Heap vs BST?**
> Heap: только min/max за O(1), push/pop O(log n). BST: поиск произвольного за O(log n), но сложнее.

**k-й элемент — heap или quickselect?**
> Quickselect O(n) среднее, in-place. Heap O(n log k) — стабильнее, проще код.

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
