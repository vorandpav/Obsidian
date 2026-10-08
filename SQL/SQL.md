# SQL

> Кратко, с примерами. Стандарт ANSI SQL, где важно — PostgreSQL.

---

## 1. Порядок выполнения SELECT (главный вопрос!)

**Пишем:**
```sql
SELECT ... FROM ... JOIN ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... LIMIT
```

**Выполняется:**

```
1. FROM + JOIN (+ ON)     — собрать таблицы
2. WHERE                  — фильтр строк
3. GROUP BY               — группировка
4. HAVING                 — фильтр групп
5. SELECT                 — вычисление столбцов, агрегаты, window functions
6. DISTINCT               — убрать дубликаты
7. ORDER BY               — сортировка
8. LIMIT / OFFSET         — обрезка результата
```

### Ловушки из-за порядка

```sql
-- ❌ Ошибка: alias ещё не существует в WHERE
SELECT price * qty AS total
FROM orders
WHERE total > 100;

-- ✅ Повторить выражение
WHERE price * qty > 100;

-- ✅ Alias работает в ORDER BY (выполняется после SELECT)
ORDER BY total DESC;
```

```sql
-- ❌ Агрегат в WHERE
WHERE COUNT(*) > 5;

-- ✅ Агрегат в HAVING
HAVING COUNT(*) > 5;
```

---

## 2. WHERE vs HAVING

| | WHERE | HAVING |
|--|-------|--------|
| Когда | До группировки | После GROUP BY |
| Фильтрует | Строки | Группы |
| Агрегаты | ❌ нельзя | ✅ можно |

```sql
SELECT department_id, COUNT(*) AS cnt, AVG(salary) AS avg_sal
FROM employees
WHERE active = true          -- отсечь неактивных ДО группировки
GROUP BY department_id
HAVING COUNT(*) > 10         -- только отделы > 10 человек
ORDER BY avg_sal DESC;
```

**Правило:** фильтруй как можно раньше → меньше данных для агрегации.

---

## 3. JOIN — типы и примеры

```
INNER   — только совпадения с обеих сторон
LEFT    — все слева + NULL справа где нет пары
RIGHT   — все справа + NULL слева
FULL    — все с обеих + NULL где нет пары
CROSS   — декартово произведение (каждый с каждым)
```

### INNER JOIN
```sql
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;
```

### LEFT JOIN — найти «сирот»
```sql
-- Клиенты без заказов
SELECT c.id, c.name
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id
WHERE o.id IS NULL;
```

### Self JOIN — иерархия
```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### Fan-out (раздувание строк) — ловушка!

Если ключ join не уникален справа → строки умножаются → `SUM`/`COUNT` завышены.

```sql
-- ❌ Один клиент × 5 заказов = 5 строк → SUM(amount) × 5
SELECT c.name, SUM(o.amount)
FROM customers c
JOIN orders o ON o.customer_id = c.id
GROUP BY c.name;

-- ✅ Сначала агрегировать
SELECT c.name, t.total
FROM customers c
LEFT JOIN (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
) t ON t.customer_id = c.id;
```

---

## 4. Агрегатные функции

| Функция | Что делает | NULL |
|---------|------------|------|
| `COUNT(*)` | Число строк | Считает все |
| `COUNT(col)` | Число non-NULL | Игнорирует NULL |
| `SUM(col)` | Сумма | Игнорирует NULL |
| `AVG(col)` | Среднее non-NULL | Игнорирует NULL |
| `MIN/MAX` | Мин/макс | Игнорирует NULL |

```sql
SELECT
    COUNT(*)           AS all_rows,
    COUNT(bonus)       AS with_bonus,
    AVG(bonus)         AS avg_bonus,    -- только по non-NULL
    SUM(amount) / COUNT(*) AS avg_with_nulls_as_zero  -- другая логика!
FROM employees;
```

### GROUP BY — правило
Все столбцы в SELECT без агрегатной функции **должны быть** в GROUP BY.

```sql
-- ❌ department_name не в GROUP BY
SELECT department_id, department_name, COUNT(*)
FROM employees GROUP BY department_id;

-- ✅
SELECT e.department_id, d.name, COUNT(*)
FROM employees e
JOIN departments d ON e.department_id = d.id
GROUP BY e.department_id, d.name;
```

### DISTINCT vs GROUP BY
- `DISTINCT` — убрать дубликаты строк
- `GROUP BY` — группировка + агрегаты
- `SELECT DISTINCT col` ≈ `SELECT col GROUP BY col` (без агрегатов)

---

## 5. Подзапросы

### В WHERE (фильтрация)
```sql
-- Сотрудники с зарплатой выше средней
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### EXISTS vs IN vs NOT IN

```sql
-- Клиенты с заказами (EXISTS — часто быстрее и безопаснее с NULL)
SELECT c.name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.id
);

-- NOT IN ловушка с NULL!
SELECT name FROM products
WHERE id NOT IN (SELECT product_id FROM discontinued);
-- Если product_id содержит NULL → результат пустой!

-- ✅ NOT EXISTS
WHERE NOT EXISTS (SELECT 1 FROM discontinued d WHERE d.product_id = p.id);
```

### CTE (WITH) — читаемость
```sql
WITH monthly_sales AS (
    SELECT
        DATE_TRUNC('month', created_at) AS month,
        SUM(amount) AS total
    FROM orders
    GROUP BY 1
),
ranked AS (
    SELECT *, RANK() OVER (ORDER BY total DESC) AS rnk
    FROM monthly_sales
)
SELECT * FROM ranked WHERE rnk <= 3;
```

---

## 6. Window Functions (оконные функции)

Вычисляются **после** WHERE/GROUP BY, **без** схлопывания строк.

```sql
SELECT
    name,
    department_id,
    salary,
    -- Ранг внутри отдела
    RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dept_rank,
    -- Скользящая сумма
    SUM(salary) OVER (PARTITION BY department_id ORDER BY hire_date
                      ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_sum,
    -- Сравнение со средним по отделу
    salary - AVG(salary) OVER (PARTITION BY department_id) AS diff_from_avg
FROM employees;
```

### Частые функции

| Функция | Зачем |
|---------|-------|
| `ROW_NUMBER()` | Уникальный номер (1,2,3...) |
| `RANK()` | Ранг с пропусками (1,2,2,4) |
| `DENSE_RANK()` | Ранг без пропусков (1,2,2,3) |
| `LAG(col, n)` | Значение n строк назад |
| `LEAD(col, n)` | Значение n строк вперёд |
| `NTILE(k)` | Разбить на k групп |

### Типовая задача: N-я зарплата по отделу
```sql
SELECT department_id, name, salary
FROM (
    SELECT *,
           DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rnk
    FROM employees
) t
WHERE rnk = 2;  -- 2-я по величине в каждом отделе
```

### Типовая задача: running total
```sql
SELECT date, amount,
       SUM(amount) OVER (ORDER BY date) AS cumulative
FROM transactions;
```

---

## 7. UNION / INTERSECT / EXCEPT

```sql
-- UNION — объединение, убирает дубликаты (сортировка для dedup)
SELECT id FROM active_users
UNION
SELECT id FROM archived_users;

-- UNION ALL — быстрее, дубликаты остаются
SELECT id FROM active_users
UNION ALL
SELECT id FROM archived_users;

-- INTERSECT — пересечение
-- EXCEPT — разность (PostgreSQL / SQL Server)
```

---

## 8. Индексы (теория для собеса)

```sql
CREATE INDEX idx_emp_dept ON employees(department_id);
CREATE INDEX idx_emp_name_sal ON employees(last_name, salary);  -- составной
```

| Тип | Когда |
|-----|-------|
| B-tree (default) | `=`, `<`, `>`, `BETWEEN`, `ORDER BY` |
| Hash | Только `=` (не везде) |
| GIN | JSON, full-text, массивы (PostgreSQL) |

**Когда индекс не помогает:**
- `WHERE LOWER(name) = 'ivan'` — функция на столбце
- `WHERE salary + 1000 > 50000` — выражение на столбце
- `LIKE '%abc'` — wildcard в начале
- Маленькая таблица — seq scan быстрее

**Covering index:** индекс содержит все нужные столбцы → index-only scan.

---

## 9. NULL — ловушки

```sql
-- ❌ Не работает!
WHERE col = NULL    -- всегда UNKNOWN
WHERE col != NULL   -- всегда UNKNOWN

-- ✅
WHERE col IS NULL
WHERE col IS NOT NULL

-- COALESCE — замена NULL
SELECT COALESCE(bonus, 0) + salary AS total;

-- NULL в агрегатах
COUNT(*)    -- считает строки с NULL
COUNT(col)  -- не считает NULL
AVG(col)    -- NULL не участвуют
```

---

## 10. Транзакции и ACID

| Свойство | Смысл |
|----------|-------|
| **A**tomicity | Всё или ничего |
| **C**onsistency | Ограничения сохраняются |
| **I**solation | Транзакции не мешают друг другу |
| **D**urability | После COMMIT — данные на диске |

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- или ROLLBACK;
```

### Уровни изоляции (знать названия)
- READ UNCOMMITTED → dirty reads
- READ COMMITTED (default в PostgreSQL)
- REPEATABLE READ
- SERIALIZABLE

---

## 11. DELETE vs TRUNCATE vs DROP

| | DELETE | TRUNCATE | DROP |
|--|--------|----------|------|
| Что удаляет | Строки (с WHERE) | Все строки | Таблицу целиком |
| Откат | ✅ | Зависит от СУБД | ❌ |
| Триггеры | ✅ | Обычно ❌ | — |
| Скорость | Медленно | Быстро | Мгновенно |

---

## 12. Пагинация

```sql
-- OFFSET — медленно на глубоких страницах (сканирует и выбрасывает)
SELECT * FROM events ORDER BY created_at DESC LIMIT 20 OFFSET 100000;

-- Keyset (cursor) pagination — быстро
SELECT * FROM events
WHERE created_at < '2026-01-01 12:00:00'
ORDER BY created_at DESC
LIMIT 20;
```

---

## 13. Типовые задачи на собесе

### Топ-N в группе
```sql
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) AS rn
    FROM products
) t WHERE rn <= 3;
```

### Дубликаты
```sql
-- Найти email-ы, встречающиеся > 1 раза
SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;

-- Удалить дубликаты, оставить min id
DELETE FROM users
WHERE id NOT IN (
    SELECT MIN(id) FROM users GROUP BY email
);
```

### Самосоединение — найти пары
```sql
SELECT a.name, b.name
FROM employees a
JOIN employees b ON a.salary = b.salary AND a.id < b.id;
```

### Pivot (сводная) — через CASE
```sql
SELECT
    department_id,
    SUM(CASE WHEN year = 2024 THEN amount ELSE 0 END) AS y2024,
    SUM(CASE WHEN year = 2025 THEN amount ELSE 0 END) AS y2025
FROM sales
GROUP BY department_id;
```

### YoY growth (год к году)
```sql
WITH yearly AS (
    SELECT year, SUM(revenue) AS rev FROM sales GROUP BY year
)
SELECT
    year,
    rev,
    LAG(rev) OVER (ORDER BY year) AS prev_rev,
    ROUND(100.0 * (rev - LAG(rev) OVER (ORDER BY year))
          / LAG(rev) OVER (ORDER BY year), 2) AS growth_pct
FROM yearly;
```

### Retention / когорты (упрощённо)
```sql
SELECT
    DATE_TRUNC('month', first_order) AS cohort,
    DATE_TRUNC('month', order_date) AS month,
    COUNT(DISTINCT user_id) AS users
FROM (
    SELECT user_id, order_date,
           MIN(order_date) OVER (PARTITION BY user_id) AS first_order
    FROM orders
) t
GROUP BY 1, 2
ORDER BY 1, 2;
```

---

## 14. Полезные функции

### Строки
```sql
CONCAT(a, b)           -- склейка
SUBSTRING(s, 1, 5)     -- подстрока
UPPER/LOWER/TRIM
LENGTH(s)              -- или CHAR_LENGTH
REPLACE(s, 'a', 'b')
```

### Даты (PostgreSQL)
```sql
DATE_TRUNC('month', ts)     -- обрезать до месяца
EXTRACT(YEAR FROM ts)       -- извлечь год
NOW(), CURRENT_DATE
ts + INTERVAL '7 days'
```

### Условные
```sql
CASE WHEN score >= 90 THEN 'A'
     WHEN score >= 80 THEN 'B'
     ELSE 'C' END AS grade

COALESCE(val, default)
NULLIF(a, b)  -- NULL если a = b
```

---

## 15. Шпаргалка «одной фразой»

| Вопрос | Ответ |
|--------|-------|
| INNER vs LEFT? | INNER — только match; LEFT — все слева + NULL |
| WHERE vs HAVING? | WHERE — строки до GROUP BY; HAVING — группы после |
| Порядок выполнения? | FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT |
| COUNT(*) vs COUNT(col)? | * — все строки; col — только non-NULL |
| Почему NOT IN с NULL? | Любое сравнение с NULL = UNKNOWN → пустой результат |
| Индекс не работает? | Функция на столбце, `LIKE '%x'`, низкая селективность |
| UNION vs UNION ALL? | UNION dedup (медленнее); ALL — все строки |

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
