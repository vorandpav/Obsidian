# Greedy

> На каждом шаге — локально лучший выбор. Работает не всегда — нужно доказать корректность.

---

## Когда greedy работает

1. **Greedy choice property** — локальный оптимум → глобальный
2. **Optimal substructure** — оптимум подзадачи входит в общий оптимум

**Классические области:** интервалы, покрытие, минимальное остовное дерево (Kruskal/Prim), коды Хаффмана.

---

## Интервалы — главный паттерн

### Merge Intervals (LC 56)

```python
def merge_intervals(intervals):
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0]]
    for start, end in intervals[1:]:
        if start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])
    return merged
```

### Non-overlapping Intervals (LC 435) — минимум удалений

**Жадность:** сортируем по **концу** интервала, берём как можно больше непересекающихся.

```python
def erase_overlap(intervals):
    intervals.sort(key=lambda x: x[1])
    count = 0
    end = float('-inf')
    for start, finish in intervals:
        if start >= end:
            end = finish
        else:
            count += 1   # удаляем текущий (пересекается)
    return count
```

### Meeting Rooms II (LC 253) — минимум комнат

```python
def min_meeting_rooms(intervals):
    starts = sorted(s for s, e in intervals)
    ends = sorted(e for s, e in intervals)
    rooms = max_rooms = 0
    s = e = 0
    while s < len(starts):
        if starts[s] < ends[e]:
            rooms += 1
            max_rooms = max(max_rooms, rooms)
            s += 1
        else:
            rooms -= 1
            e += 1
    return max_rooms
```

---

## Activity Selection

> Максимум непересекающихся интервалов = минимум удалений (см. выше).

Сортировка по `end`, greedy: берём, если `start >= last_end`.

---

## Jump Game (LC 55, 45)

```python
def can_jump(nums):
    max_reach = 0
    for i, jump in enumerate(nums):
        if i > max_reach:
            return False
        max_reach = max(max_reach, i + jump)
    return True

def jump(nums):  # LC 45 — минимум прыжков
    jumps = end = farthest = 0
    for i in range(len(nums) - 1):
        farthest = max(farthest, i + nums[i])
        if i == end:
            jumps += 1
            end = farthest
    return jumps
```

---

## Partition Labels (LC 763)

```python
def partition_labels(s):
    last = {ch: i for i, ch in enumerate(s)}
    start = end = 0
    result = []
    for i, ch in enumerate(s):
        end = max(end, last[ch])
        if i == end:
            result.append(end - start + 1)
            start = i + 1
    return result
```

---

## Gas Station (LC 134)

```python
def can_complete_circuit(gas, cost):
    total = curr = start = 0
    for i in range(len(gas)):
        diff = gas[i] - cost[i]
        total += diff
        curr += diff
        if curr < 0:
            start = i + 1
            curr = 0
    return start if total >= 0 else -1
```

---

## Когда greedy НЕ работает

| Задача | Почему | Нужно |
|--------|--------|-------|
| Coin Change (общие номиналы) | Жадность по max монете не оптимальна | DP |
| 0/1 Knapsack | Нельзя брать дробные части | DP |
| Shortest path с отриц. рёбрами | Greedy не видит «обход» | Bellman-Ford |

---

## Greedy vs DP

| | Greedy | DP |
|--|--------|-----|
| Решение | Один проход, локальный выбор | Таблица, все подзадачи |
| Сложность | Обычно O(n log n) | O(n) — O(n²) |
| Корректность | Нужно доказать | Всегда, если есть overlapping subproblems |

---

## Вопросы на собесе

**Как доказать greedy?**
> Exchange argument: любое оптимальное решение можно преобразовать в greedy без ухудшения.

**Интервалы — сортировать по start или end?**
> Merge → по start. Non-overlapping / activity selection → по **end**.

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
