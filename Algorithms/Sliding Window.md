# Sliding Window

> Подмассив/подстрока фиксированной или переменной длины. O(n).

---

## Два типа

| Тип | Описание | Пример |
|-----|----------|--------|
| **Фиксированное** | Окно длины k | Max sum subarray of size k |
| **Переменное** | Расширяем/сжимаем по условию | Longest substring without repeat |

---

## Шаблон: переменное окно

```python
def sliding_window(s):
    left = 0
    state = {}          # или set, counter, сумма
    best = 0

    for right in range(len(s)):
        # 1. РАСШИРИТЬ: добавить s[right] в state
        state[s[right]] = state.get(s[right], 0) + 1

        # 2. СЖАТЬ: пока условие нарушено
        while not is_valid(state):
            state[s[left]] -= 1
            if state[s[left]] == 0:
                del state[s[left]]
            left += 1

        # 3. ОБНОВИТЬ ответ
        best = max(best, right - left + 1)

    return best
```

**Логика:** `right` всегда двигается вперёд; `left` догоняет, когда окно «плохое».

---

## Фиксированное окно k

```python
def max_sum_subarray(nums, k):
    window_sum = sum(nums[:k])
    best = window_sum

    for right in range(k, len(nums)):
        window_sum += nums[right] - nums[right - k]  # +новый -старый
        best = max(best, window_sum)
    return best
```

**Задачи:** LC 643, 567 (permutation in string)

---

## Longest substring without repeating (LC 3)

```python
def length_of_longest_substring(s):
    seen = {}
    left = best = 0

    for right, ch in enumerate(s):
        if ch in seen and seen[ch] >= left:
            left = seen[ch] + 1   # прыжок left (не по одному!)
        seen[ch] = right
        best = max(best, right - left + 1)
    return best
```

---

## Minimum window substring (LC 76)

Найти **минимальную** подстроку, содержащую все символы `t`.

```python
from collections import Counter

def min_window(s, t):
    need = Counter(t)
    missing = len(t)
    left = start = 0
    min_len = float('inf')

    for right, ch in enumerate(s):
        if need[ch] > 0:
            missing -= 1
        need[ch] -= 1

        while missing == 0:          # окно валидно → сжимаем
            if right - left + 1 < min_len:
                start, min_len = left, right - left + 1
            need[s[left]] += 1
            if need[s[left]] > 0:
                missing += 1
            left += 1

    return s[start:start + min_len] if min_len != float('inf') else ""
```

---

## Longest / shortest — что оптимизируем

| Задача | Обновлять ответ | Когда |
|--------|-----------------|-------|
| **Longest** | `max(best, right-left+1)` | Внутри цикла, когда окно валидно |
| **Shortest** | `min(best, right-left+1)` | Внутри `while` (при валидном окне), потом сжимаем |

---

## Subarray sum equals K (LC 560) — prefix sum + hash

Не чистое sliding window (если есть отрицательные числа), но часто путают:

```python
def subarray_sum(nums, k):
    count = 0
    prefix = 0
    seen = {0: 1}   # prefix_sum → count

    for num in nums:
        prefix += num
        count += seen.get(prefix - k, 0)
        seen[prefix] = seen.get(prefix, 0) + 1
    return count
```

**Чистый sliding window** работает только если все числа **≥ 0**.

---

## At most K distinct (LC 340)

```python
def at_most_k_distinct(s, k):
    count = {}
    left = result = 0

    for right, ch in enumerate(s):
        count[ch] = count.get(ch, 0) + 1

        while len(count) > k:
            count[s[left]] -= 1
            if count[s[left]] == 0:
                del count[s[left]]
            left += 1

        result += right - left + 1   # все подстроки, заканчивающиеся в right
    return result
```

**Exactly K** = `at_most(K) - at_most(K-1)`

---

## Сложность

- Время: **O(n)** — каждый элемент входит/выходит из окна максимум 1 раз
- Память: O(k) или O(алфавит)

---

## Вопросы на собесе

**Sliding window vs two pointers?**
> SW — подмассив/подстрока с условием на содержимое. TP — пары, палиндромы, merge.

**Когда SW не работает?**
> Отрицательные числа в сумме → prefix sum + hash map.

**Как не уйти в O(n²)?**
> `left` только увеличивается, никогда не уменьшается назад (кроме сжатия на 1).

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
