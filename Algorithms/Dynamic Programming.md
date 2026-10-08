# Dynamic Programming

> Разбить на перекрывающиеся подзадачи, сохранить результаты. Оптимальная подструктура.

---

## Как распознать DP

- «Максимум/минимум/количество способов»
- «Можно ли...?» (да/нет)
- Выбор на каждом шаге влияет на будущее
- Brute force O(2^n) → есть повторяющиеся подзадачи

---

## Подход

```
1. Определить состояние: dp[i] = что означает?
2. База: dp[0], dp[1], ...
3. Переход: dp[i] = f(dp[i-1], dp[i-2], ...)
4. Ответ: dp[n] или max/min по dp
5. Порядок обхода (чтобы зависимости уже посчитаны)
```

---

## 1D DP — классика

### Fibonacci / Climbing Stairs (LC 70)

```python
def climb_stairs(n):
    if n <= 2:
        return n
    a, b = 1, 2
    for _ in range(3, n + 1):
        a, b = b, a + b
    return b
# dp[i] = dp[i-1] + dp[i-2]
```

### House Robber (LC 198)

```python
def rob(nums):
    prev2 = prev1 = 0
    for num in nums:
        prev2, prev1 = prev1, max(prev1, prev2 + num)
    return prev1
# dp[i] = max(dp[i-1], dp[i-2] + nums[i])
```

### Coin Change (LC 322)

```python
def coin_change(coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a:
                dp[a] = min(dp[a], dp[a - c] + 1)
    return dp[amount] if dp[amount] != float('inf') else -1
```

### Longest Increasing Subsequence (LC 300)

```python
def length_of_lis(nums):
    dp = [1] * len(nums)
    for i in range(len(nums)):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)
# O(n²); с бинарным поиском — O(n log n)
```

---

## 2D DP

### Unique Paths (LC 62)

```python
def unique_paths(m, n):
    dp = [[1] * n for _ in range(m)]
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i-1][j] + dp[i][j-1]
    return dp[m-1][n-1]
```

### 0/1 Knapsack

```python
def knapsack(weights, values, capacity):
    dp = [0] * (capacity + 1)
    for w, v in zip(weights, values):
        for c in range(capacity, w - 1, -1):  # справа налево!
            dp[c] = max(dp[c], dp[c - w] + v)
    return dp[capacity]
```

**Важно:** цикл по `capacity` **в обратном порядке** — каждый предмет один раз.

### LCS — Longest Common Subsequence (LC 1143)

```python
def longest_common_subsequence(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]
```

### Edit Distance (LC 72)

```python
def min_distance(word1, word2):
    m, n = len(word1), len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(m + 1): dp[i][0] = i
    for j in range(n + 1): dp[0][j] = j
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i-1] == word2[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
    return dp[m][n]
# insert, delete, replace
```

---

## Паттерны

| Паттерн | Признак | Примеры |
|---------|---------|---------|
| **Linear** | dp[i] от dp[i-1], dp[i-2] | Fib, rob, decode ways |
| **Knapsack** | Выбор предметов с весом | 0/1, unbounded coin change |
| **Grid** | Движение по сетке | Unique paths, min path sum |
| **String** | Два указателя по строкам | LCS, edit distance, palindrome |
| **Interval** | dp[i][j] — отрезок [i,j] | Burst balloons, matrix chain |
| **Bitmask** | dp[mask] — подмножество | TSP, assign tasks |

---

## Мемоизация vs Tabulation

```python
# Top-down (мемоизация)
from functools import lru_cache

@lru_cache(maxsize=None)
def dp(i):
    if i == 0: return base
    return f(dp(i-1), dp(i-2))

# Bottom-up (табуляция) — обычно быстрее, без recursion limit
dp = [0] * (n + 1)
dp[0] = base
for i in range(1, n + 1):
    dp[i] = f(dp[i-1], ...)
```

---

## Оптимизация памяти

Если `dp[i]` зависит только от `dp[i-1]` и `dp[i-2]`:

```python
# O(n) → O(1) память
prev2, prev1 = 0, 1
for i in range(2, n + 1):
    prev2, prev1 = prev1, prev1 + prev2
```

---

## Вопросы на собесе

**DP vs Greedy?**
> DP — когда локальный выбор неочевиден. Greedy — когда можно доказать, что локальный = глобальный.

**Как найти переход?**
> «Что было на предыдущем шаге?» — взял предмет / не взял, пришёл сверху / слева.

**Сложность 2D DP?**
> O(m·n) время и память; часто сжимается до O(n).

---

## Задачи для практики

| Уровень | LC |
|---------|-----|
| 1D | 70, 198, 322, 300, 139 (word break) |
| 2D | 62, 64, 1143, 72, 5 (palindrome) |
| Knapsack | 416 (partition), 494 (target sum) |

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
