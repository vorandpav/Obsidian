# Number Theory

> Базовая теория чисел — часто спрашивают на алго-собесе и в задачах на дроби/циклы.

---

## Определения

| | Русский | Англ. | Смысл |
|--|---------|-------|-------|
| **GCD** | НОД | Greatest Common Divisor | Наибольший общий делитель |
| **LCM** | НОК | Least Common Multiple | Наименьшее общее кратное |

```python
gcd(12, 18) = 6    # делители: {1,2,3,6} ∩ {1,2,3,6,9,18}
lcm(12, 18) = 36   # кратные: {12,24,36,...} ∩ {18,36,...}
```

---

## Главная формула (запомни!)

$$\text{GCD}(a, b) \times \text{LCM}(a, b) = |a \times b|$$

```python
def lcm(a, b):
    return abs(a * b) // gcd(a, b)   # // чтобы не было float
```

**LCM через GCD** — стандарт на собесе. Сначала GCD, потом LCM.

---

## GCD — алгоритм Евклида

**Идея:** $\text{GCD}(a, b) = \text{GCD}(b, a \bmod b)$, пока $b \neq 0$.

```python
def gcd(a, b):
    while b:
        a, b = b, a % b
    return abs(a)
```

### Рекурсивная версия

```python
def gcd(a, b):
    return a if b == 0 else gcd(b, a % b)
```

| | |
|--|--|
| **Сложность** | O(log min(a, b)) |
| **Память** | O(1) итеративно, O(log n) рекурсия |

### Пример по шагам: GCD(48, 18)

```
48 % 18 = 12  →  GCD(18, 12)
18 % 12 = 6   →  GCD(12, 6)
12 % 6  = 0   →  GCD(6, 0) = 6
```

---

## LCM — реализация

```python
def lcm(a, b):
    if a == 0 or b == 0:
        return 0
    return abs(a // gcd(a, b) * b)   # делим ДО умножения — без overflow
```

**Почему `a // gcd * b`, а не `a * b // gcd`?**
> Сначала делим — меньше риск переполнения для больших чисел.

### LCM для нескольких чисел

```python
from functools import reduce

def lcm_multiple(nums):
    return reduce(lcm, nums)
# lcm(a, b, c) = lcm(lcm(a, b), c)
```

---

## GCD / LCM массива

```python
from functools import reduce

def gcd_array(nums):
    return reduce(gcd, nums)

def lcm_array(nums):
    return reduce(lcm, nums)
```

**Задача:** LC 1071 (GCD of Strings) — если строка `str1 + str2 == str2 + str1`, ответ = повтор `s[:gcd(len1, len2)]`.

---

## Расширенный алгоритм Евклида

Находит GCD **и** коэффициенты Безу: $ax + by = \text{GCD}(a, b)$.

```python
def extended_gcd(a, b):
    if b == 0:
        return a, 1, 0      # gcd, x, y: a*1 + b*0 = a
    g, x1, y1 = extended_gcd(b, a % b)
    x = y1
    y = x1 - (a // b) * y1
    return g, x, y
```

**Зачем на собесе:**
- Модульное обратное: $a^{-1} \pmod m$ когда $\text{GCD}(a, m) = 1$
- Диофантовы уравнения
- LC 365 (Water and Jug Problem)

---

## Модульное обратное (бонус)

```python
def mod_inverse(a, m):
    """a^(-1) mod m, если gcd(a, m) == 1."""
    g, x, _ = extended_gcd(a % m, m)
    if g != 1:
        return None
    return x % m
```

---

## Взаимно простые числа

**Coprime:** $\text{GCD}(a, b) = 1$

- В задачах на вращение/циклы: «встретятся снова через LCM шагов»
- Euler: $\phi(n)$ — количество чисел $< n$ с GCD = 1

---

## Типичные задачи

### Сократить дробь

```python
g = gcd(numerator, denominator)
num //= g
den //= g
```

### N-ая итерация цикла (шаги по кругу)

Колёса/шестерёнки с зубцами $a$ и $b$ совпадут через **LCM(a, b)** шагов.

```python
# LC 365: можно ли налить target литров?
# jugs a, b → GCD(a, b) должен делить target
def can_measure(a, b, target):
    return target <= a + b and target % gcd(a, b) == 0
```

### Проверка «можно ли разбить массив»

Если нужно, чтобы все части делились на $d$ → $d$ должен делить GCD всех элементов.

```python
def can_divide(nums, k):
    g = reduce(gcd, nums)
    return g % k == 0
```

---

## Python stdlib

```python
import math

math.gcd(48, 18)           # 6  (Python 3.5+)
math.lcm(12, 18)           # 36 (Python 3.9+)

# Для нескольких:
math.gcd(12, 18, 24)       # 3  (Python 3.9+)
math.lcm(4, 6, 8)          # 24 (Python 3.9+)
```

**На собесе:** если разрешают — `math.gcd`. Если просят «написать» — алгоритм Евклида.

---

## Шпаргалка «одной фразой»

| Вопрос | Ответ |
|--------|-------|
| GCD(a, 0)? | a |
| GCD(a, b) = GCD(b, a)? | Да |
| LCM через GCD? | `a * b // gcd(a, b)` |
| GCD массива? | `reduce(gcd, nums)` |
| Сложность Евклида? | O(log min(a, b)) |
| Coprime? | GCD = 1 |
| Jug problem? | target делится на GCD(a, b) |

---

## Задачи для практики

| LC | Тема |
|----|------|
| 1071 | GCD of Strings |
| 365 | Water and Jug (GCD) |
| 914 | X of a Kind (GCD массива) |
| 2183 | Count array pairs (GCD) |

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
