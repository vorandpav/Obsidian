# Shortest Path

---

## Сводка алгоритмов

| Алгоритм | Веса | Сложность | Когда |
|----------|------|-----------|-------|
| **BFS** | Все = 1 | O(V+E) | Невзвешенный граф |
| **Dijkstra** | ≥ 0 | O((V+E) log V) | Один источник, неотриц. |
| **Bellman-Ford** | Любые* | O(VE) | Отриц. веса, detect cycle |
| **Floyd-Warshall** | Любые* | O(V³) | Все пары, V ≤ 400 |

*Отрицательные — без отрицательных циклов.

---

## Dijkstra — главный для собеса

```python
import heapq

def dijkstra(graph, start):
    """
    graph: adjacency list, graph[u] = [(v, weight), ...]
    Returns: dict {node: min_distance}
    """
    dist = {start: 0}
    heap = [(0, start)]   # (distance, node)

    while heap:
        d, u = heapq.heappop(heap)
        if d > dist.get(u, float('inf')):
            continue   # устаревшая запись в heap

        for v, weight in graph[u]:
            nd = d + weight
            if nd < dist.get(v, float('inf')):
                dist[v] = nd
                heapq.heappush(heap, (nd, v))

    return dist
```

### С восстановлением пути

```python
def dijkstra_with_path(graph, start, end):
    dist = {start: 0}
    parent = {start: None}
    heap = [(0, start)]

    while heap:
        d, u = heapq.heappop(heap)
        if u == end:
            break
        if d > dist.get(u, float('inf')):
            continue
        for v, w in graph[u]:
            nd = d + w
            if nd < dist.get(v, float('inf')):
                dist[v] = nd
                parent[v] = u
                heapq.heappush(heap, (nd, v))

    # Восстановить путь
    path = []
    node = end
    while node is not None:
        path.append(node)
        node = parent.get(node)
    return dist.get(end, float('inf')), path[::-1]
```

### Почему не работает с отрицательными весами?

Жадный выбор «минимальный dist» необратим — отрицательное ребро позже может дать более короткий путь.

---

## Bellman-Ford

```python
def bellman_ford(n, edges, start):
    """
    edges: list of (u, v, weight)
    """
    dist = [float('inf')] * n
    dist[start] = 0

    for _ in range(n - 1):          # V-1 релаксаций
        for u, v, w in edges:
            if dist[u] != float('inf') and dist[u] + w < dist[v]:
                dist[v] = dist[u] + w

    # Проверка отрицательного цикла
    for u, v, w in edges:
        if dist[u] != float('inf') and dist[u] + w < dist[v]:
            return None   # отрицательный цикл

    return dist
```

**Применение:** отрицательные веса, детект отрицательного цикла, V ≤ 1000.

---

## Floyd-Warshall (все пары)

```python
def floyd_warshall(dist):
    """
    dist: V×V матрица, dist[i][j] = вес или inf
    """
    n = len(dist)
    for k in range(n):
        for i in range(n):
            for j in range(n):
                if dist[i][k] + dist[k][j] < dist[i][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]
    return dist
```

Инициализация: `dist[i][j] = weight` если ребро, `0` если i==j, `inf` иначе.

---

## 0-1 BFS (веса только 0 и 1)

```python
from collections import deque

def bfs_01(graph, start, n):
    dist = [float('inf')] * n
    dist[start] = 0
    dq = deque([start])

    while dq:
        u = dq.popleft()
        for v, w in graph[u]:
            nd = dist[u] + w
            if nd < dist[v]:
                dist[v] = nd
                if w == 0:
                    dq.appendleft(v)   # вес 0 → в начало
                else:
                    dq.append(v)
    return dist
```

O(V + E) — быстрее Dijkstra для 0-1 графов.

---

## Network Delay Time (LC 743) — классика Dijkstra

```python
def network_delay(times, n, k):
    graph = defaultdict(list)
    for u, v, w in times:
        graph[u].append((v, w))

    dist = dijkstra(graph, k)
    result = max(dist.get(i, float('inf')) for i in range(1, n + 1))
    return result if result != float('inf') else -1
```

---

## Cheapest Flights within K Stops (LC 787) — Bellman-Ford вариант

```python
def find_cheapest_price(n, flights, src, dst, k):
    dist = [float('inf')] * n
    dist[src] = 0

    for _ in range(k + 1):          # максимум k+1 рёбер
        tmp = dist[:]
        for u, v, w in flights:
            if dist[u] != float('inf'):
                tmp[v] = min(tmp[v], dist[u] + w)
        dist = tmp

    return dist[dst] if dist[dst] != float('inf') else -1
```

---

## Path with Maximum Probability (LC 1514)

Dijkstra, но **максимизируем** произведение вероятностей → логарифмы или `dist` = max prob, `nd = dist[u] * prob`.

---

## Сравнение на собесе

| Вопрос | Ответ |
|--------|-------|
| Все рёбра вес 1? | BFS |
| Неотрицательные веса? | Dijkstra |
| Отрицательные? | Bellman-Ford |
| Все пары, V маленькое? | Floyd-Warshall |
| Почему heap в Dijkstra? | O(log V) extract-min вместо O(V) |

---

## Задачи

| LC | Алгоритм |
|----|----------|
| 743 | Dijkstra |
| 787 | Bellman-Ford / modified Dijkstra |
| 1514 | Dijkstra (max) |
| 1631 | Dijkstra на grid |
| 127 | BFS (word ladder) |

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
