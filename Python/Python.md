# Python

> Кратко, по делу. Для AI-инженера 2 курса.

---

## 1. Сложность операций (Big O)

### list — динамический массив

| Операция | Сложность | Почему |
|----------|-----------|--------|
| `lst[i]` | O(1) | Прямой доступ по индексу |
| `lst.append(x)` | O(1) амортиз. | Иногда перераспределение ~1.125× |
| `lst.pop()` | O(1) | С конца |
| `lst.pop(0)`, `insert(0, x)` | O(n) | Сдвиг всех элементов |
| `x in lst` | O(n) | Линейный поиск |
| `lst.sort()` | O(n log n) | Timsort |
| `len(lst)` | O(1) | Хранится в объекте |

### dict / set — хеш-таблица

| Операция | Сложность | Почему |
|----------|-----------|--------|
| `get/set/del`, `x in d` | O(1) среднее | Хеш → слот |
| `x in d` (худший случай) | O(n) | Все коллизии в одном бакете |
| Итерация | O(n) | Обход всех слотов |

**Ключ должен быть hashable** (immutable): `int`, `str`, `tuple` — да; `list`, `dict` — нет.

### tuple — неизменяемая последовательность

- Быстрее list, меньше памяти
- O(1) доступ по индексу
- Нельзя менять, но **содержимое mutable-объектов** внутри можно (например, `([1,2], 3)`)

### collections.deque — двусторонняя очередь

| Операция | Сложность |
|----------|-----------|
| `append`, `appendleft`, `pop`, `popleft` | O(1) |
| `x in deque` | O(n) |

**На собесе:** для очереди BFS / sliding window — `deque`, не `list.pop(0)`.

### collections — что знать

| Класс | Зачем |
|-------|-------|
| `Counter` | Подсчёт элементов, `most_common(k)` |
| `defaultdict` | Авто-значение по умолчанию для ключа |
| `deque` | Очередь, O(1) с обоих концов |
| `OrderedDict` | Порядок вставки (в 3.7+ обычный dict тоже ordered) |

---

## 2. Устройство структур (что спрашивают)

### list
```
[объект PyList]
  → массив указателей на PyObject*
  → over-allocated (запас ~12.5%)
```
- Динамический массив указателей, не связный список
- `append` амортизированно O(1) за счёт переаллокации

### dict (Python 3.7+)
```
хеш(key) → индекс в compact array → key/value
```
- Open addressing (не цепочки как в Java HashMap)
- Порядок вставки сохраняется
- При коллизии — probing (поиск следующего свободного слота)

### set
- Тот же механизм, что dict, но без value
- Элементы уникальны

---

## 3. Генераторы и итераторы

### Итератор
Объект с `__iter__()` и `__next__()`. `StopIteration` — конец.

### Генератор
Функция с `yield` → возвращает generator object (ленивый итератор).

```python
# Генератор — O(1) память
def read_lines(path):
    with open(path) as f:
        for line in f:
            yield line.strip()

# Generator expression
squares = (x**2 for x in range(10**6))  # не создаёт список в памяти

# List comprehension — O(n) память
squares_list = [x**2 for x in range(10**6)]
```

| | List comprehension `[]` | Generator `()` |
|--|--------------------------|----------------|
| Память | O(n) | O(1) |
| Многократный проход | Да | Нет (одноразовый) |
| Индексация, `len()` | Да | Нет |
| Когда | Нужен результат целиком | Поток, большие данные, `sum/any/all` |

### `yield from` — делегирование
```python
def chain(*iterables):
    for it in iterables:
        yield from it
```

**Типичный вопрос:** чем генератор отличается от итератора?
- Генератор — **подвид** итератора, создаётся функцией с `yield`
- Итератор — любой объект с `__next__`

---

## 4. GIL (Global Interpreter Lock)

- Один поток выполняет Python bytecode в момент времени (CPython)
- **I/O-bound** → `threading` (GIL отпускается при ожидании I/O)
- **CPU-bound** → `multiprocessing` (отдельные процессы, свои GIL)
- **Async I/O** → `asyncio` (один поток, event loop, корутины)

```python
# CPU-bound — процессы
from multiprocessing import Pool
with Pool(4) as p:
    results = p.map(heavy_func, data)

# I/O-bound — async
async def fetch(url):
    async with session.get(url) as resp:
        return await resp.text()
```

---

## 5. OOP — частые вопросы

### MRO (Method Resolution Order)
Порядок поиска метода при множественном наследовании. C3 linearization.

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass
print(D.__mro__)  # D → B → C → A → object
```

### `@staticmethod` vs `@classmethod` vs обычный метод

| | Первый аргумент | Зачем |
|--|----------------|-------|
| `def m(self)` | `self` (экземпляр) | Работа с данными объекта |
| `@classmethod` | `cls` (класс) | Альтернативные конструкторы |
| `@staticmethod` | нет | Утилита в namespace класса |

```python
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y

    @classmethod
    def from_polar(cls, r, theta):
        return cls(r * cos(theta), r * sin(theta))
```

### Дандер-методы (знать основные)

| Метод | Когда вызывается |
|-------|------------------|
| `__init__` | Создание объекта |
| `__repr__` | Для разработчика (`repr(obj)`) |
| `__str__` | Для пользователя (`print(obj)`) |
| `__eq__` | `==` |
| `__hash__` | В `set`/`dict` как ключ (нужен вместе с `__eq__`) |
| `__len__` | `len(obj)` |
| `__getitem__` | `obj[i]` |
| `__enter__/__exit__` | `with` statement |

### `*args` и `**kwargs`
```python
def f(a, *args, b=10, **kwargs):
    # args — tuple позиционных
    # kwargs — dict именованных
```

---

## 6. Декораторы

```python
def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__}: {time.time() - start:.3f}s")
        return result
    return wrapper

@timer
def train():
    ...
```

**Суть:** функция, принимающая функцию и возвращающая обёртку.  
**С `@functools.wraps(func)`** — сохраняет имя и docstring оригинала.

---

## 7. Копирование

```python
import copy

a = [[1, 2], [3, 4]]
b = a              # ссылка — одна и та же память
c = a.copy()       # shallow — вложенные объекты общие
d = copy.deepcopy(a)  # deep — полная копия
```

---

## 8. Context manager (`with`)

```python
class Database:
    def __enter__(self):
        self.conn = connect()
        return self.conn
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.conn.close()
        return False  # не подавлять исключение
```

Или: `from contextlib import contextmanager` + `yield`.

---

## 9. Полезные модули stdlib

| Модуль | Что знать |
|--------|-----------|
| `itertools` | `chain`, `groupby`, `combinations`, `permutations`, `islice` |
| `functools` | `lru_cache`, `partial`, `reduce` |
| `collections` | см. выше |
| `typing` | `List`, `Dict`, `Optional`, `Union`, `Protocol` |
| `pathlib` | `Path("data/file.txt").read_text()` |
| `dataclasses` | `@dataclass` — boilerplate для классов данных |

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)
```

---

## 10. Типичные ловушки (predict the output)

```python
# 1. Изменяемый default argument
def append_to(item, lst=[]):  # BAD — один список на все вызовы
    lst.append(item)
    return lst

# 2. Замыкание в цикле
funcs = [lambda: i for i in range(3)]
[f() for f in funcs]  # [2, 2, 2] — все ссылаются на финальный i

# 3. is vs ==
a = [1, 2, 3]
b = [1, 2, 3]
a == b  # True (значения)
a is b  # False (разные объекты)

# 4. Integer interning
a = 256; b = 256; a is b  # True (кэш -5..256)
a = 257; b = 257; a is b  # может быть False

# 5. List multiplication
[[0]*3]*3  # [[0,0,0],[0,0,0],[0,0,0]] — но все строки ОДНА ссылка!
```

---

## 11. Алгоритмы — что уметь объяснить

| Алгоритм        | Сложность         | Когда                                |
| --------------- | ----------------- | ------------------------------------ |
| Binary search   | O(log n)          | Отсортированный массив               |
| Two pointers    | O(n)              | Отсортированный массив, пары         |
| Sliding window  | O(n)              | Подмассив фикс. длины / условие      |
| BFS             | O(V+E)            | Кратчайший путь в невзвешенном графе |
| DFS             | O(V+E)            | Обход, связность, топосорт           |
| Hash map lookup | O(1)              | Поиск пары, подсчёт частот           |
| Heap (heapq)    | push/pop O(log n) | Top-K, медиана потока                |

---

## 12. Быстрый выбор структуры

```
Нужен индекс / порядок / дубликаты?     → list
Нужен быстрый поиск / уникальность?     → set
Нужна связь ключ → значение?            → dict
Данные фиксированы, не меняются?        → tuple
Очередь (добавить/убрать с обоих концов)? → deque
Счётчик элементов?                      → Counter
```

---

## 13. Частые вопросы одной фразой

| Вопрос                    | Ответ                                                        |
| ------------------------- | ------------------------------------------------------------ |
| Mutable vs immutable?     | list/dict/set — mutable; int/str/tuple/frozenset — immutable |
| `__name__ == "__main__"`? | Код выполняется только при прямом запуске, не при import     |
| `*a, b = [1,2,3]`?        | `a=[1,2]`, `b=3` — распаковка                                |
| Декоратор класса?         | `@dataclass`, `@property` — тоже декораторы                  |
| `enumerate`?              | `(index, value)` при итерации                                |
| `zip`?                    | Параллельная итерация, длина = min длин                      |
| `map` vs list comp?       | `map` — ленивый итератор в Python 3                          |

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
