# Binary Search

> Главная боль: `left`, `right`, `<=` vs `<`, `mid-1` vs `mid`.  
> Решение: **запомни 2 шаблона**, не пытайся каждый раз выводить.

---

## Когда применять

1. **Массив отсортирован** → ищем элемент / границу
2. **Ответ монотонен** → «если X работает, то X+1 тоже» (поиск по ответу)
3. **Пик / повёрнутый массив** → половина отсортирована

Сложность: **O(log n)**

---

## Два шаблона — запомни их!

### Шаблон 1: Точное совпадение (exact match)

```python
def binary_search(nums, target):
    lo, hi = 0, len(nums) - 1      # закрытый интервал [lo, hi]
    while lo <= hi:                  # <= потому что lo == hi ещё проверяем
        mid = lo + (hi - lo) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            lo = mid + 1             # mid уже проверили
        else:
            hi = mid - 1
    return -1
```

| Параметр | Значение |
|----------|----------|
| `hi` | `len(nums) - 1` |
| Условие цикла | `lo <= hi` |
| Возврат | `mid` при равенстве, иначе `-1` |

---

### Шаблон 2: Граница (boundary / lower bound) — **ГЛАВНЫЙ**

Ищем **первый** индекс, где условие `ok(i)` = True.  
Монотонность: `False False False True True True`

```python
def first_true(lo, hi, ok):
    """Первый индекс в [lo, hi), где ok(i) == True.
    hi — эксклюзивная граница (обычно len(arr))."""
    while lo < hi:                     # < потому что lo == hi → ответ
        mid = lo + (hi - lo) // 2
        if ok(mid):
            hi = mid                 # mid может быть ответом → НЕ mid-1
        else:
            lo = mid + 1             # mid точно не подходит
    return lo                        # lo == hi → ответ
```

| Параметр | Значение |
|----------|----------|
| `hi` | `len(arr)` (эксклюзивно!) |
| Условие цикла | `lo < hi` |
| При `ok(mid)` | `hi = mid` (оставляем mid) |
| При `not ok(mid)` | `lo = mid + 1` |
| Возврат | `lo` (всегда) |

---

## Lower Bound и Upper Bound

На отсортированном массиве `arr`:

| Функция | Определение | Предикат `ok(i)` |
|---------|-------------|------------------|
| **lower_bound** | Первый `i`, где `arr[i] >= target` | `arr[i] >= target` |
| **upper_bound** | Первый `i`, где `arr[i] > target` | `arr[i] > target` |

```python
def lower_bound(arr, target):
    lo, hi = 0, len(arr)
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if arr[mid] < target:      # ещё не дошли до target
            lo = mid + 1
        else:
            hi = mid               # arr[mid] >= target → mid кандидат
    return lo

def upper_bound(arr, target):
    lo, hi = 0, len(arr)
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if arr[mid] <= target:     # <= а не < — вот единственная разница!
            lo = mid + 1
        else:
            hi = mid
    return lo
```

### Пример: `arr = [1, 2, 2, 2, 3]`, `target = 2`

```
lower_bound → 1   (первый индекс с 2)
upper_bound → 4   (первый индекс > 2, т.е. за блоком двоек)
```

**Количество вхождений** = `upper_bound - lower_bound` = 3  
**Диапазон** = `[lower_bound, upper_bound)`

---

## Первое и последнее вхождение (LC 34)

```python
def search_range(nums, target):
    lo = lower_bound(nums, target)
    hi = upper_bound(nums, target) - 1   # последний индекс с target
    
    if lo == len(nums) or nums[lo] != target:
        return [-1, -1]
    return [lo, hi]
```

---

## Поиск по ответу (Binary Search on Answer)

Когда ответ — **число**, и условие монотонно:

> «Найти минимальный X, при котором feasible(X) = True»

```python
def min_feasible_answer(lo, hi, feasible):
    """lo, hi — диапазон ответа (не индексы массива!)."""
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if feasible(mid):
            hi = mid                 # mid работает → ищем меньше
        else:
            lo = mid + 1             # mid не работает → ищем больше
    return lo
```

**Примеры задач:**
- Koko Eating Bananas — минимальная скорость
- Capacity To Ship Packages — минимальная ёмкость
- Split Array Largest Sum — минимальный максимум

```python
# Пример: минимальная скорость поедания бананов
def min_eating_speed(piles, h):
    def can_finish(speed):
        hours = sum((p + speed - 1) // speed for p in piles)  # ceil
        return hours <= h

    lo, hi = 1, max(piles)
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if can_finish(mid):
            hi = mid
        else:
            lo = mid + 1
    return lo
```

---

## Правый boundary (последний True)

Когда нужен **последний** индекс, где `ok(i) = True`:

```python
def last_true(lo, hi, ok):
    """Последний индекс в [lo, hi), где ok(i) == True."""
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if ok(mid):
            lo = mid + 1             # mid подходит, но ищем правее
        else:
            hi = mid
    return lo - 1                    # откат на последний True
```

Или: `last_true_pred = first_true(pred) - 1`, где pred = «строго после нужного».

---

## Шпаргалка: что куда ставить

```
┌─────────────────────────────────────────────────────────┐
│  EXACT MATCH          │  BOUNDARY (lower/upper)           │
├───────────────────────┼─────────────────────────────────┤
│  hi = len - 1         │  hi = len (эксклюзив!)           │
│  while lo <= hi       │  while lo < hi                  │
│  ok → return mid      │  ok → hi = mid                  │
│  < target → lo=mid+1  │  not ok → lo = mid+1            │
│  > target → hi=mid-1  │  return lo                      │
└───────────────────────┴─────────────────────────────────┘
```

### Почему `hi = len`, а не `len - 1`?

Полуинтервал `[lo, hi)` — `hi` указывает **за** последний элемент.  
Если target больше всех элементов → ответ `len` (вставка в конец).  
С `hi = len - 1` этот случай потеряется.

### Почему `hi = mid`, а не `mid - 1`?

При boundary мы ищем **первый** True. Если `ok(mid)` — mid может быть ответом.  
`hi = mid - 1` отбросит его → баг на границе.

---

## Повёрнутый отсортированный массив (LC 33)

```python
def search_rotated(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if nums[mid] == target:
            return mid
        
        # Левая половина отсортирована
        if nums[lo] <= nums[mid]:
            if nums[lo] <= target < nums[mid]:
                hi = mid - 1
            else:
                lo = mid + 1
        # Правая половина отсортирована
        else:
            if nums[mid] < target <= nums[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    return -1
```

---

## Поиск пика (LC 162)

```python
def find_peak(nums):
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if nums[mid] < nums[mid + 1]:
            lo = mid + 1    # пик справа
        else:
            hi = mid        # пик слева или mid — пик
    return lo
```

---

## Типичные ошибки

| Ошибка | Симптом | Фикс |
|--------|---------|------|
| `hi = mid` в exact match | Бесконечный цикл | `hi = mid - 1` |
| `hi = mid - 1` в boundary | Пропуск ответа | `hi = mid` |
| `while lo <= hi` в boundary | Off-by-one | `while lo < hi` |
| `(lo + hi) // 2` overflow | Редко в Python, но спрашивают | `lo + (hi-lo)//2` |
| Путаешь шаблоны | WA на границах | Сначала определи: exact или boundary |

---

## Вопросы на собесе

**Когда `lo <= hi`, когда `lo < hi`?**
> Exact match → `<=`. Boundary / поиск по ответу → `<`.

**Чем lower_bound от upper_bound?**
> lower: `arr[mid] < target` → сдвиг вправо. upper: `arr[mid] <= target` → сдвиг вправо.

**Бинарный поиск без массива?**
> Да — поиск по ответу. Главное: предикат монотонен.

**Сложность?**
> O(log n) по размеру пространства поиска.

---

## Задачи для практики

| Уровень | Задачи |
|---------|--------|
| База | LC 704, 35 (Search Insert), 278 (First Bad Version) |
| Границы | LC 34, 81 (with duplicates), 153 (min in rotated) |
| По ответу | LC 875, 1011, 410 |
| Пик | LC 162, 852 |

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
