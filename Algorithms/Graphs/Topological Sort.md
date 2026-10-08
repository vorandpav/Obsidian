# Topological Sort

> Линейный порядок вершин DAG (Directed Acyclic Graph): для каждого ребра u→v, u идёт **до** v.

---

## Когда применять

- Зависимости: курсы, сборка, задачи
- «Можно ли выполнить все?» → если есть топосорт, то да (нет цикла)
- Порядок компиляции, build system, prerequisites

---

## Kahn's Algorithm (BFS по in-degree)

```python
from collections import deque, defaultdict

def topological_sort_kahn(n, edges):
    graph = defaultdict(list)
    indegree = [0] * n

    for u, v in edges:
        graph[u].append(v)
        indegree[v] += 1

    queue = deque(i for i in range(n) if indegree[i] == 0)
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            indegree[neighbor] -= 1
            if indegree[neighbor] == 0:
                queue.append(neighbor)

    if len(order) != n:
        return []   # есть цикл!
    return order
```

**Логика:** начинаем с вершин без входящих рёбер, «удаляем» их, уменьшаем in-degree соседей.

### Сложность: O(V + E)

---

## DFS-версия

```python
def topological_sort_dfs(n, edges):
    graph = defaultdict(list)
    for u, v in edges:
        graph[u].append(v)

    visited = set()
    stack = []   # результат в обратном порядке

    def dfs(node):
        visited.add(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                dfs(neighbor)
        stack.append(node)   # postorder — добавляем ПОСЛЕ соседей

    for i in range(n):
        if i not in visited:
            dfs(i)

    return stack[::-1]
```

### С детекцией цикла (3 цвета)

```python
def topo_dfs_cycle(n, edges):
    graph = defaultdict(list)
    for u, v in edges:
        graph[u].append(v)

    WHITE, GRAY, BLACK = 0, 1, 2
    color = [WHITE] * n
    order = []

    def dfs(node):
        color[node] = GRAY
        for neighbor in graph[node]:
            if color[neighbor] == GRAY:
                return False   # цикл
            if color[neighbor] == WHITE and not dfs(neighbor):
                return False
        color[node] = BLACK
        order.append(node)
        return True

    for i in range(n):
        if color[i] == WHITE and not dfs(i):
            return []
    return order[::-1]
```

---

## Course Schedule (LC 207, 210)

```python
# LC 207 — можно ли пройти все курсы?
def can_finish(numCourses, prerequisites):
    return len(topological_sort_kahn(numCourses, prerequisites)) == numCourses

# LC 210 — вернуть порядок курсов
def find_order(numCourses, prerequisites):
    order = topological_sort_kahn(numCourses, prerequisites)
    return order if len(order) == numCourses else []
```

---

## Alien Dictionary (LC 269)

Построить порядок букв по отсортированному словарю чужого языка.

```python
def alien_order(words):
    graph = defaultdict(set)
    indegree = {ch: 0 for word in words for ch in word}

    for i in range(len(words) - 1):
        w1, w2 = words[i], words[i + 1]
        if len(w1) > len(w2) and w1[:len(w2)] == w2:
            return ""   # invalid: "abc" before "ab"
        for c1, c2 in zip(w1, w2):
            if c1 != c2:
                if c2 not in graph[c1]:
                    graph[c1].add(c2)
                    indegree[c2] += 1
                break

    queue = deque(ch for ch in indegree if indegree[ch] == 0)
    order = []
    while queue:
        ch = queue.popleft()
        order.append(ch)
        for neighbor in graph[ch]:
            indegree[neighbor] -= 1
            if indegree[neighbor] == 0:
                queue.append(neighbor)

    return "".join(order) if len(order) == len(indegree) else ""
```

---

## Longest Path in DAG

Топосорт + DP:

```python
def longest_path_dag(n, edges):
    graph = defaultdict(list)
    for u, v, w in edges:
        graph[u].append((v, w))

    order = topological_sort_kahn(n, [(u, v) for u, v, w in edges])
    dist = [0] * n

    for u in order:
        for v, w in graph[u]:
            dist[v] = max(dist[v], dist[u] + w)

    return max(dist)
```

---

## Kahn vs DFS

| | Kahn (BFS) | DFS |
|--|------------|-----|
| Детект цикла | `len(order) < n` | GRAY node |
| Интуиция | «Удаляем» без зависимостей | Postorder → reverse |
| Лексикограф. мин. | Сортировать queue | — |

---

## Вопросы на собесе

**Что если цикл?**
> Топосорт невозможен. Kahn: order короче n. DFS: back edge (GRAY neighbor).

**Сложность?**
> O(V + E).

**Undirected граф?**
> Топосорт только для directed. Undirected → DFS/BFS.

**Связь с DP?**
> DAG + топосорт → DP по порядку за O(V+E).

---

## Задачи

| LC | Тема |
|----|------|
| 207 | Можно ли? |
| 210 | Порядок |
| 269 | Alien dictionary |
| 1203 | Sort items by groups |
| 329 | Longest increasing path (DAG на grid) |

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
