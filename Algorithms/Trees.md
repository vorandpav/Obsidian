# Trees

---

## Обходы (DFS)

```python
# Preorder:  root → left → right
def preorder(root):
    if not root: return
    process(root)
    preorder(root.left)
    preorder(root.right)

# Inorder:   left → root → right  (BST → отсортированный порядок!)
def inorder(root):
    if not root: return
    inorder(root.left)
    process(root)
    inorder(root.right)

# Postorder: left → right → root  (удаление, вычисление)
def postorder(root):
    if not root: return
    postorder(root.left)
    postorder(root.right)
    process(root)
```

### BFS (level order) — через очередь

```python
from collections import deque

def level_order(root):
    if not root: return []
    q = deque([root])
    result = []
    while q:
        level = []
        for _ in range(len(q)):
            node = q.popleft()
            level.append(node.val)
            if node.left:  q.append(node.left)
            if node.right: q.append(node.right)
        result.append(level)
    return result
```

---

## BST — свойства

- Для каждого узла: `left < root < right`
- **Inorder** = отсортированная последовательность
- Поиск / вставка / удаление: O(h), h = высота. Сбалансированное → O(log n)

### Search в BST

```python
def search(root, val):
    if not root or root.val == val:
        return root
    if val < root.val:
        return search(root.left, val)
    return search(root.right, val)
```

### Validate BST (LC 98)

```python
def is_valid_bst(root, lo=float('-inf'), hi=float('inf')):
    if not root:
        return True
    if not (lo < root.val < hi):
        return False
    return (is_valid_bst(root.left, lo, root.val) and
            is_valid_bst(root.right, root.val, hi))
```

---

## Высота / глубина / диаметр

```python
def max_depth(root):
    if not root: return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))

def diameter(root):
    """Длиннейший путь между любыми двумя узлами."""
    best = 0
    def depth(node):
        nonlocal best
        if not node: return 0
        l = depth(node.left)
        r = depth(node.right)
        best = max(best, l + r)
        return 1 + max(l, r)
    depth(root)
    return best
```

---

## LCA — Lowest Common Ancestor

### BST (LC 235)

```python
def lca_bst(root, p, q):
    while root:
        if p.val < root.val and q.val < root.val:
            root = root.left
        elif p.val > root.val and q.val > root.val:
            root = root.right
        else:
            return root
```

### Binary Tree (LC 236)

```python
def lca(root, p, q):
    if not root or root == p or root == q:
        return root
    left = lca(root.left, p, q)
    right = lca(root.right, p, q)
    if left and right:
        return root
    return left or right
```

---

## Path Sum (LC 112, 113)

```python
def has_path_sum(root, target):
    if not root:
        return False
    if not root.left and not root.right:
        return root.val == target
    return (has_path_sum(root.left, target - root.val) or
            has_path_sum(root.right, target - root.val))
```

---

## Serialize / Deserialize (LC 297)

```python
def serialize(root):
    if not root: return "null"
    return f"{root.val},{serialize(root.left)},{serialize(root.right)}"

def deserialize(data):
    vals = iter(data.split(","))
    def build():
        v = next(vals)
        if v == "null": return None
        node = TreeNode(int(v))
        node.left = build()
        node.right = build()
        return node
    return build()
```

---

## Построение из обходов

| Дано | Как |
|------|-----|
| Preorder + Inorder | Первый preorder = root, делим inorder на left/right |
| Postorder + Inorder | Последний postorder = root |
| Level order | BFS с очередью |

---

## Сбалансированность (LC 110)

```python
def is_balanced(root):
    def height(node):
        if not node: return 0
        l, r = height(node.left), height(node.right)
        if abs(l - r) > 1:
            return -1
        h = 1 + max(l, r)
        return -1 if h == -1 else h
    return height(root) != -1
```

---

## Сложности

| Операция | Сбалансированное | Вырожденное (список) |
|----------|------------------|----------------------|
| Search | O(log n) | O(n) |
| Insert | O(log n) | O(n) |
| Delete | O(log n) | O(n) |
| Обход | O(n) | O(n) |

---

## Вопросы на собесе

**Inorder BST — зачем?**
> Даёт отсортированный порядок за O(n).

**BFS vs DFS на дереве?**
> BFS — уровни, кратчайший путь (в дереве = по рёбрам). DFS — пути, backtracking.

**Рекурсия vs итерация?**
> DFS — стек (явный или call stack). BFS — очередь.

---

## Задачи

| Тема | LC |
|------|-----|
| Обходы | 94, 102, 104, 105 |
| BST | 98, 235, 230 (kth smallest) |
| Path | 112, 124 (max path sum) |
| LCA | 236, 235 |
| Построение | 105, 106 |

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
