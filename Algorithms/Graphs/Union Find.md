# Union Find

> Быстрое объединение множеств и проверка «в одном ли компоненте». Почти O(1) на операцию.

---

## Операции

| Операция | Что делает |
|----------|------------|
| `find(x)` | Представитель (root) множества x |
| `union(x, y)` | Объединить множества x и y |
| `connected(x, y)` | В одном ли множестве? |

---

## Реализация

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n      # для union by rank
        # self.size = [1] * n    # альтернатива: размер компоненты
        self.components = n

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # path compression
        return self.parent[x]

    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py:
            return False   # уже в одном множестве

        # Union by rank
        if self.rank[px] < self.rank[py]:
            px, py = py, px
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1

        self.components -= 1
        return True

    def connected(self, x, y):
        return self.find(x) == self.find(y)
```

---

## Оптимизации (спрашивают!)

### Path Compression
При `find` — все узлы на пути указывают напрямую на root.

```python
self.parent[x] = self.find(self.parent[x])
```

### Union by Rank / Size
Меньшее дерево подвешиваем к большему → высота ≤ log n, с compression → **α(n) ≈ O(1)**.

**Амортизированная сложность:** O(α(n)) на операцию, α — обратная функция Аккермана (практически константа).

---

## Number of Connected Components (LC 323)

```python
def count_components(n, edges):
    uf = UnionFind(n)
    for u, v in edges:
        uf.union(u, v)
    return uf.components
```

---

## Detect Cycle in Undirected Graph

```python
def has_cycle(n, edges):
    uf = UnionFind(n)
    for u, v in edges:
        if uf.connected(u, v):
            return True   # уже связаны → ребро создаёт цикл
        uf.union(u, v)
    return False
```

**Kruskal MST** использует это же: добавляем ребро только если не создаёт цикл.

---

## Redundant Connection (LC 684)

```python
def find_redundant(edges):
    uf = UnionFind(len(edges) + 1)
    for u, v in edges:
        if not uf.union(u, v):
            return [u, v]   # первое ребро, создавшее цикл
```

---

## Accounts Merge (LC 721)

```python
def accounts_merge(accounts):
    uf = UnionFind(len(accounts))
    email_to_id = {}

    for i, account in enumerate(accounts):
        for email in account[1:]:
            if email in email_to_id:
                uf.union(i, email_to_id[email])
            else:
                email_to_id[email] = i

    # Собрать emails по компонентам
    groups = defaultdict(list)
    for email, acc_id in email_to_id.items():
        groups[uf.find(acc_id)].append(email)

    return [[accounts[root][0]] + sorted(emails)
            for root, emails in groups.items()]
```

---

## Kruskal MST (Minimum Spanning Tree)

```python
def kruskal(n, edges):
    """
    edges: [(weight, u, v), ...]
    Returns: total weight of MST
    """
    edges.sort()   # по весу
    uf = UnionFind(n)
    total = 0
    edges_used = 0

    for w, u, v in edges:
        if uf.union(u, v):
            total += w
            edges_used += 1
            if edges_used == n - 1:
                break

    return total if edges_used == n - 1 else -1
```

**Сложность:** O(E log E) — сортировка рёбер.

---

## Union by Size (для подсчёта размера)

```python
def union(self, x, y):
    px, py = self.find(x), self.find(y)
    if px == py:
        return False
    if self.size[px] < self.size[py]:
        px, py = py, px
    self.parent[py] = px
    self.size[px] += self.size[py]
    return True
```

**Задачи:** «размер компоненты», «друзья по кругу» (LC 547).

---

## DSU vs DFS для компонент

| | Union-Find | DFS |
|--|------------|-----|
| Online union | ✅ | ❌ (нужен полный пересчёт) |
| Статический граф | Оба O(V+E) | Проще код |
| Dynamic connectivity | ✅ | ❌ |
| Детект цикл undirected | ✅ при добавлении ребра | ✅ |

---

## Вопросы на собесе

**Зачем path compression?**
> Дерево find становится плоским → следующие find быстрее.

**Union by rank vs size?**
> Rank — глубина дерева. Size — число узлов. Size удобнее, если нужен размер компоненты.

**Сложность m операций на n элементах?**
> O(m · α(n)) ≈ O(m).

**DSU на grid?**
> `uf = UnionFind(m * n)`, индекс `r * cols + c`.

---

## Задачи

| LC | Тема |
|----|------|
| 323, 547 | Components |
| 684, 685 | Redundant connection |
| 721 | Accounts merge |
| 1135 | Connecting cities (MST) |
| 947 | Stones (connected by row/col) |
| 990 | Equal equations |

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
