# Backtracking

> Перебор с откатом: пробуем → если не подходит → откатываемся.

---

## Шаблон

```python
def backtrack(path, choices):
    if is_solution(path):
        result.append(path[:])   # копия! не path
        return

    for choice in choices:
        if not is_valid(choice):
            continue
        path.append(choice)       # выбор
        backtrack(path, next_choices)
        path.pop()                # откат
```

**Ключ:** `path[:]` при сохранении, `path.pop()` после рекурсии.

---

## Subsets (LC 78)

```python
def subsets(nums):
    result = []
    def backtrack(start, path):
        result.append(path[:])
        for i in range(start, len(nums)):
            path.append(nums[i])
            backtrack(i + 1, path)
            path.pop()
    backtrack(0, [])
    return result
```

**Комбинации:** `start` index — не перебираем назад (нет дубликатов).

---

## Permutations (LC 46)

```python
def permute(nums):
    result = []
    def backtrack(path, remaining):
        if not remaining:
            result.append(path[:])
            return
        for i in range(len(remaining)):
            path.append(remaining[i])
            backtrack(path, remaining[:i] + remaining[i+1:])
            path.pop()
    backtrack([], nums)
    return result
```

**Оптимизация:** swap in-place вместо `remaining[:i] + remaining[i+1:]`.

```python
def permute_swap(nums):
    result = []
    def backtrack(start):
        if start == len(nums):
            result.append(nums[:])
            return
        for i in range(start, len(nums)):
            nums[start], nums[i] = nums[i], nums[start]
            backtrack(start + 1)
            nums[start], nums[i] = nums[i], nums[start]
    backtrack(0)
    return result
```

---

## Combination Sum (LC 39)

```python
def combination_sum(candidates, target):
    result = []
    def backtrack(start, path, remaining):
        if remaining == 0:
            result.append(path[:])
            return
        for i in range(start, len(candidates)):
            if candidates[i] > remaining:
                break   # отсортированный массив — pruning
            path.append(candidates[i])
            backtrack(i, path, remaining - candidates[i])  # i, не i+1 — reuse
            path.pop()
    candidates.sort()
    backtrack(0, [], target)
    return result
```

---

## N-Queens (LC 51)

```python
def solve_n_queens(n):
    result = []
    cols = set()
    diag1 = set()  # row + col
    diag2 = set()  # row - col

    def backtrack(row, board):
        if row == n:
            result.append(["".join(r) for r in board])
            return
        for col in range(n):
            if col in cols or (row+col) in diag1 or (row-col) in diag2:
                continue
            cols.add(col); diag1.add(row+col); diag2.add(row-col)
            board[row][col] = 'Q'
            backtrack(row + 1, board)
            board[row][col] = '.'
            cols.remove(col); diag1.remove(row+col); diag2.remove(row-col)

    backtrack(0, [['.'] * n for _ in range(n)])
    return result
```

---

## Word Search (LC 79)

```python
def exist(board, word):
    rows, cols = len(board), len(board[0])

    def backtrack(r, c, idx):
        if idx == len(word):
            return True
        if r < 0 or r >= rows or c < 0 or c >= cols:
            return False
        if board[r][c] != word[idx]:
            return False

        temp = board[r][c]
        board[r][c] = '#'   # visited
        found = (backtrack(r+1,c,idx+1) or backtrack(r-1,c,idx+1) or
                 backtrack(r,c+1,idx+1) or backtrack(r,c-1,idx+1))
        board[r][c] = temp  # откат
        return found

    for r in range(rows):
        for c in range(cols):
            if backtrack(r, c, 0):
                return True
    return False
```

---

## Pruning (отсечение)

Ускоряет backtracking:

```python
# 1. Сортировка + break при превышении
if candidates[i] > remaining: break

# 2. Подсчёт оставшихся — если не хватит, return
if len(path) + remaining_choices < needed: return

# 3. Дубликаты — skip одинаковые на одном уровне
if i > start and nums[i] == nums[i-1]: continue
```

---

## Backtracking vs DP

| | Backtracking | DP |
|--|--------------|-----|
| Цель | Все решения / одно | Оптимум / количество |
| Сложность | Экспоненциальная | Полиномиальная |
| Память | O(глубина рекурсии) | O(таблица) |

---

## Вопросы на собесе

**Почему `path[:]`?**
> `path` — один список, мутируется. Без копии все результаты — ссылка на один объект.

**Subsets vs Permutations?**
> Subsets: порядок не важен → `start` index. Permutations: порядок важен → swap / used[].

**Сложность N-Queens?**
> O(n!) — в каждом ряду до n выборов, глубина n.

---

## Задачи

| LC | Тема |
|----|------|
| 78, 90 | Subsets |
| 46, 47 | Permutations |
| 39, 40 | Combination sum |
| 51 | N-Queens |
| 79, 212 | Word search |
| 17 | Letter combinations |

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
