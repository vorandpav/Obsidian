# Two Pointers

> Два индекса, двигающихся по массиву/строке. O(n) вместо O(n²).

---

## Когда применять

- Отсортированный массив + пара с суммой/разностью
- Палиндромы, reverse
- Слияние двух отсортированных массивов
- Удаление дубликатов in-place
- «Разделить массив» (Dutch flag)

---

## Паттерн 1: Сходящиеся указатели (opposite ends)

```python
def two_sum_sorted(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        s = nums[lo] + nums[hi]
        if s == target:
            return [lo, hi]
        elif s < target:
            lo += 1          # нужна большая сумма
        else:
            hi -= 1          # нужна меньшая сумма
    return []
```

**Задачи:** LC 1 (с hash map), 15 (3Sum), 11 (Container With Most Water), 42 (Trapping Rain Water)

---

## Паттерн 2: Быстрый + медленный (Floyd)

```python
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False
```

**Задачи:** LC 141, 142 (найти начало цикла), 876 (middle of list)

**Начало цикла (LC 142):**
```python
# После встречи: slow = head, оба по 1 шагу → встреча = начало цикла
```

---

## Паттерн 3: Оба с начала (merge / filter)

```python
def merge_sorted(a, b):
    i = j = 0
    result = []
    while i < len(a) and j < len(b):
        if a[i] <= b[j]:
            result.append(a[i])
            i += 1
        else:
            result.append(b[j])
            j += 1
    result.extend(a[i:])
    result.extend(b[j:])
    return result
```

**Задачи:** LC 88 (merge in-place), 977 (squares of sorted array)

---

## Паттерн 4: In-place удаление дубликатов

```python
def remove_duplicates(nums):
    if not nums:
        return 0
    write = 1
    for read in range(1, len(nums)):
        if nums[read] != nums[write - 1]:
            nums[write] = nums[read]
            write += 1
    return write  # новая длина
```

**Задачи:** LC 26, 27 (remove element), 80 (до 2 копий)

---

## Палиндром

```python
def is_palindrome(s):
    lo, hi = 0, len(s) - 1
    while lo < hi:
        while lo < hi and not s[lo].isalnum():
            lo += 1
        while lo < hi and not s[hi].isalnum():
            hi -= 1
        if s[lo].lower() != s[hi].lower():
            return False
        lo += 1
        hi -= 1
    return True
```

---

## 3Sum (LC 15) — шаблон

```python
def three_sum(nums):
    nums.sort()
    result = []
    for i in range(len(nums) - 2):
        if i > 0 and nums[i] == nums[i-1]:
            continue  # пропуск дубликатов
        lo, hi = i + 1, len(nums) - 1
        while lo < hi:
            s = nums[i] + nums[lo] + nums[hi]
            if s == 0:
                result.append([nums[i], nums[lo], nums[hi]])
                while lo < hi and nums[lo] == nums[lo+1]: lo += 1
                while lo < hi and nums[hi] == nums[hi-1]: hi -= 1
                lo += 1; hi -= 1
            elif s < 0:
                lo += 1
            else:
                hi -= 1
    return result
```

---

## Dutch National Flag (LC 75)

Три зоны: 0 | 1 | 2. Три указателя: `lo`, `mid`, `hi`.

```python
def sort_colors(nums):
    lo, mid, hi = 0, 0, len(nums) - 1
    while mid <= hi:
        if nums[mid] == 0:
            nums[lo], nums[mid] = nums[mid], nums[lo]
            lo += 1; mid += 1
        elif nums[mid] == 1:
            mid += 1
        else:  # 2
            nums[mid], nums[hi] = nums[hi], nums[mid]
            hi -= 1
```

---

## Сложность

| Паттерн | Время | Память |
|---------|-------|--------|
| Opposite ends | O(n) | O(1) |
| Fast/slow | O(n) | O(1) |
| Merge | O(n+m) | O(n+m) или O(1) in-place |

---

## Вопросы на собесе

**Two pointers vs hash map для Two Sum?**
> Отсортированный → two pointers O(n). Неотсортированный → hash map O(n).

**Почему 3Sum = O(n²)?**
> Внешний цикл O(n) × two pointers O(n).

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
