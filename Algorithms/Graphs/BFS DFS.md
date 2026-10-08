# BFS DFS

---

## BFS — обход в ширину

**Очередь.** Сначала все соседи, потом соседи соседей.

```python
from collections import deque

def bfs(graph, start):
    visited = {start}
    queue = deque([start])
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    return order
```

### Сложность: O(V + E)

---

## BFS — кратчайший путь (невзвешенный граф!)

```python
def bfs_shortest_path(graph, start, end):
    if start == end:
        return [start]
    visited = {start}
    queue = deque([(start, [start])])

    while queue:
        node, path = queue.popleft()
        for neighbor in graph[node]:
            if neighbor in visited:
                continue
            new_path = path + [neighbor]
            if neighbor == end:
                return new_path
            visited.add(neighbor)
            queue.append((neighbor, new_path))
    return []  # нет пути
```

**Оптимизация:** хранить `parent[node]` вместо полного path → O(V) память.

```python
def bfs_distance(graph, start):
    dist = {start: 0}
    queue = deque([start])
    while queue:
        node = queue.popleft()
        for neighbor in graph[node]:
            if neighbor not in dist:
                dist[neighbor] = dist[node] + 1
                queue.append(neighbor)
    return dist
```

---

## DFS — обход в глубину

**Рекурсия или стек.**

```python
def dfs_recursive(graph, node, visited=None):
    if visited is None:
        visited = set()
    visited.add(node)
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs_recursive(graph, neighbor, visited)
    return visited

def dfs_iterative(graph, start):
    visited = set()
    stack = [start]
    order = []
    while stack:
        node = stack.pop()
        if node in visited:
            continue
        visited.add(node)
        order.append(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                stack.append(neighbor)
    return order
```

### Сложность: O(V + E)

---

## DFS — шаблон с backtracking

```python
def dfs_paths(graph, node, target, path, result):
    if node == target:
        result.append(path[:])
        return
    for neighbor in graph[node]:
        if neighbor not in path:   # избежать цикла в пути
            path.append(neighbor)
            dfs_paths(graph, neighbor, target, path, result)
            path.pop()
```

---

## Number of Islands (LC 200)

```python
def num_islands(grid):
    rows, cols = len(grid), len(grid[0])
    count = 0

    def dfs(r, c):
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] != '1':
            return
        grid[r][c] = '0'   # visited
        dfs(r+1, c); dfs(r-1, c); dfs(r, c+1); dfs(r, c-1)

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                dfs(r, c)
                count += 1
    return count
```

**BFS-версия:** то же, но `deque` вместо рекурсии.

---

## Clone Graph (LC 133)

```python
def clone_graph(node):
    if not node:
        return None
    clones = {}
    def dfs(n):
        if n in clones:
            return clones[n]
        copy = Node(n.val)
        clones[n] = copy
        for neighbor in n.neighbors:
            copy.neighbors.append(dfs(neighbor))
        return copy
    return dfs(node)
```

---

## Detect Cycle

### Undirected — DFS

```python
def has_cycle_undirected(graph, n):
    visited = set()
    def dfs(node, parent):
        visited.add(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                if dfs(neighbor, node):
                    return True
            elif neighbor != parent:   # уже visited и не родитель → цикл
                return True
        return False

    for i in range(n):
        if i not in visited:
            if dfs(i, -1):
                return True
    return False
```

### Directed — DFS (3 цвета)

```python
def has_cycle_directed(graph, n):
    WHITE, GRAY, BLACK = 0, 1, 2
    color = [WHITE] * n

    def dfs(node):
        color[node] = GRAY
        for neighbor in graph[node]:
            if color[neighbor] == GRAY:   # back edge → цикл
                return True
            if color[neighbor] == WHITE and dfs(neighbor):
                return True
        color[node] = BLACK
        return False

    return any(dfs(i) for i in range(n) if color[i] == WHITE)
```

| Цвет | Значение |
|------|----------|
| WHITE | Не посещён |
| GRAY | В текущем стеке (в процессе) |
| BLACK | Полностью обработан |

---

## Connected Components

```python
def count_components(graph, n):
    visited = set()
    components = 0
    for i in range(n):
        if i not in visited:
            dfs_recursive(graph, i, visited)
            components += 1
    return components
```

---

## BFS vs DFS — когда что

| | BFS | DFS |
|--|-----|-----|
| Структура | Queue | Stack / рекурсия |
| Кратчайший путь | ✅ (невзвеш.) | ❌ |
| Память | O(ширина уровня) | O(глубина) |
| Топосорт | ❌ | ✅ |
| Все пути | Неудобно | ✅ backtracking |
| Уровни дерева | ✅ | ❌ |

---

## Multi-source BFS (LC 994 — Rotting Oranges)

```python
def oranges_rotting(grid):
    rows, cols = len(grid), len(grid[0])
    queue = deque()
    fresh = 0

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 2:
                queue.append((r, c, 0))
            elif grid[r][c] == 1:
                fresh += 1

    minutes = 0
    while queue:
        r, c, t = queue.popleft()
        minutes = max(minutes, t)
        for dr, dc in [(0,1),(0,-1),(1,0),(-1,0)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 1:
                grid[nr][nc] = 2
                fresh -= 1
                queue.append((nr, nc, t + 1))

    return minutes if fresh == 0 else -1
```

---

## Задачи

| LC | Тема |
|----|------|
| 200, 695 | Islands, area |
| 133, 323 | Clone, components |
| 207, 210 | Cycle, order |
| 994 | Multi-source BFS |
| 127 | Word ladder (BFS) |

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
