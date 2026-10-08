# Sorting

---

## Сводная таблица

| Алгоритм | Среднее | Худшее | Память | Стабильная | Когда |
|----------|---------|--------|--------|------------|-------|
| **Merge Sort** | O(n log n) | O(n log n) | O(n) | ✅ | Нужна стабильность, linked list |
| **Quick Sort** | O(n log n) | O(n²) | O(log n) | ❌ | In-place, общий случай |
| **Heap Sort** | O(n log n) | O(n log n) | O(1) | ❌ | Гарантированный O(n log n), in-place |
| **Timsort** (Python) | O(n log n) | O(n log n) | O(n) | ✅ | Встроенный `sorted()` |
| Bubble / Insertion | O(n²) | O(n²) | O(1) | ✅ | Почти отсортированные, малые n |
| Counting / Radix | O(n+k) | O(n+k) | O(k) | ✅ | Целые в ограниченном диапазоне |

**Стабильная** = равные элементы сохраняют исходный порядок.

---

## Merge Sort

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return merge(left, right)

def merge(a, b):
    i = j = 0
    result = []
    while i < len(a) and j < len(b):
        if a[i] <= b[j]:      # <= для стабильности
            result.append(a[i]); i += 1
        else:
            result.append(b[j]); j += 1
    return result + a[i:] + b[j:]
```

**Плюсы:** гарантированный O(n log n), стабильный  
**Минусы:** O(n) доп. память  
**Применение:** LC 148 (sort list), inversion count, external sort

---

## Quick Sort

```python
def quicksort(arr, lo, hi):
    if lo >= hi:
        return
    p = partition(arr, lo, hi)
    quicksort(arr, lo, p - 1)
    quicksort(arr, p + 1, hi)

def partition(arr, lo, hi):
    pivot = arr[hi]
    i = lo
    for j in range(lo, hi):
        if arr[j] <= pivot:
            arr[i], arr[j] = arr[j], arr[i]
            i += 1
    arr[i], arr[hi] = arr[hi], arr[i]
    return i
```

**Худший случай O(n²):** уже отсортированный массив + плохой pivot  
**Фикс:** random pivot, median-of-three

**Применение:** LC 215 (quickselect — K-й элемент за O(n) среднее)

---

## Quickselect (K-й элемент)

```python
import random

def quickselect(nums, k):
    """k-й наименьший (0-indexed)."""
    def partition(lo, hi):
        pivot_idx = random.randint(lo, hi)
        nums[pivot_idx], nums[hi] = nums[hi], nums[pivot_idx]
        pivot = nums[hi]
        i = lo
        for j in range(lo, hi):
            if nums[j] <= pivot:
                nums[i], nums[j] = nums[j], nums[i]
                i += 1
        nums[i], nums[hi] = nums[hi], nums[i]
        return i

    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        p = partition(lo, hi)
        if p == k:
            return nums[p]
        elif p < k:
            lo = p + 1
        else:
            hi = p - 1
```

---

## Counting Sort

```python
def counting_sort(arr, max_val):
    count = [0] * (max_val + 1)
    for x in arr:
        count[x] += 1
    result = []
    for val, cnt in enumerate(count):
        result.extend([val] * cnt)
    return result
```

O(n + k), где k — диапазон значений. Только целые неотрицательные.

---

## Когда что на собесе

| Ситуация | Выбор |
|----------|-------|
| «Отсортируй» в Python | `sorted()` / `.sort()` — Timsort |
| Нужен K-й элемент | Quickselect или Heap |
| Слить K отсортированных списков | Heap (LC 23) |
| Подсчёт инверсий | Merge sort |
| Строки лексикографически | Встроенная сортировка O(n·m log n) |

---

## Вопросы на собесе

**Почему Quick Sort быстрее Merge Sort на практике?**
> In-place, лучшая cache locality, меньше копирований.

**Стабильность — зачем?**
> Сортировка по второму ключу: сначала по имени, потом по возрасту — стабильная сохранит порядок имён.

**Timsort?**
> Гибрид merge + insertion. Ищет уже отсортированные runs. Встроен в Python и Java.

**Сложность сортировки сравнением?**
> Нижняя граница **Ω(n log n)** для general comparison sort.

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
