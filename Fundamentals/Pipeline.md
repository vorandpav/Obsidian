# Pipeline

---

## Supervised vs Unsupervised vs RL

| Тип | Данные | Задачи | Примеры |
|-----|--------|--------|---------|
| **Supervised** | X + y (метки) | Классификация, регрессия | Спам, цена дома, NER |
| **Unsupervised** | Только X | Кластеризация, снижение размерности | Сегментация, PCA, anomaly |
| **Semi-supervised** | Мало y, много X | Когда разметка дорогая | Псевдолейблы |
| **Self-supervised** | X без y, y из X | Pretext tasks | BERT MLM, contrastive |
| **Reinforcement** | Agent + env + reward | Оптимальная политика | Игры, роботы |

---

## ML Pipeline (что рассказать на собесе)

```
1. Постановка задачи     → что предсказываем? какая метрика?
2. Сбор данных           → EDA, пропуски, выбросы
3. Feature engineering   → создание/отбор признаков
4. Train/Val/Test split    → стратификация, временной split
5. Baseline              → majority class / mean / простая модель
6. Обучение моделей      → от простого к сложному
7. Hyperparameter tuning → grid/random/Bayesian search на val
8. Оценка на test        → один раз!
9. Деплой + мониторинг   → data drift, model decay
```

---

## EDA — что смотреть

- Распределения признаков и таргета
- Пропуски: `df.isnull().sum()`
- Корреляции: `df.corr()` — мультиколлинеарность
- Дисбаланс классов: `value_counts()`
- Выбросы: boxplot, IQR rule

---

## Feature Engineering

| Техника | Пример |
|---------|--------|
| Кодирование категорий | One-hot, Target encoding, Label encoding |
| Масштабирование | StandardScaler, MinMaxScaler |
| Полиномиальные | `x²`, `x₁·x₂` |
| Логарифм | `log(price)` — для скошенных распределений |
| Биннинг | Возраст → группы |
| Даты | day_of_week, is_weekend, hour |
| Текст | TF-IDF, embeddings |

### One-hot vs Label encoding
- **One-hot** — для номинальных (город, цвет)
- **Label** — для порядковых (low/med/high) или деревья (им не важен масштаб)

### Target Encoding — осторожно!
```python
# ❌ Утечка: средний target по всему датасету
# ✅ Только на train fold внутри CV
```

---

## Нормализация — зачем?

| Метод | Формула | Когда |
|-------|---------|-------|
| **StandardScaler** | $(x - \mu) / \sigma$ | LR, SVM, kNN, NN |
| **MinMaxScaler** | $(x - min) / (max - min)$ | Нужен диапазон [0,1] |
| **RobustScaler** | $(x - median) / IQR$ | Много выбросов |

**Деревья** (RF, XGBoost) — масштабирование не нужно.

---

## Train/Test Split

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    random_state=42,
    stratify=y          # сохранить пропорции классов
)
```

### Временные данные
```python
# ❌ random split для time series
# ✅ split по времени
train = df[df.date < '2025-01-01']
test  = df[df.date >= '2025-01-01']
```

---

## Cross-Validation

### k-Fold
```
Данные → [fold1|fold2|fold3|fold4|fold5]
         val  train train train train  → score₁
         train val  train train train  → score₂
         ...
Итог: mean(scores) ± std(scores)
```

### Stratified k-Fold
- Сохраняет пропорции классов в каждом fold

### Leave-One-Out (LOO)
- k = N — дорого, мало bias

---

## Hyperparameter Tuning

| Метод | Суть |
|-------|------|
| **Grid Search** | Перебор всех комбинаций из сетки |
| **Random Search** | Случайные комбинации (часто эффективнее grid) |
| **Bayesian Optimization** | Optuna, Hyperopt — умный поиск |

```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    'C': [0.1, 1, 10],
    'kernel': ['linear', 'rbf'],
}
search = GridSearchCV(SVC(), param_grid, cv=5, scoring='f1')
search.fit(X_train, y_train)
print(search.best_params_, search.best_score_)
```

**Важно:** tuning на train+val (через CV), финальная оценка на test.

---

## Baseline — всегда начинай с него

```python
from sklearn.dummy import DummyClassifier

dummy = DummyClassifier(strategy='most_frequent')
dummy.fit(X_train, y_train)
print(dummy.score(X_test, y_test))  # если модель не лучше — проблема
```

---

## Data Leakage — типичные случаи

| Утечка | Пример |
|--------|--------|
| Target leakage | Признак «был_возврат» при предсказании «купит_ли» |
| Preprocessing leakage | `fit` scaler на всём датасете |
| Temporal leakage | Train содержит данные из будущего |
| Duplicate leakage | Один пациент в train и test |

---

## Вопросы на собесе

**Supervised vs unsupervised?**
> Supervised — есть метки y. Unsupervised — только X, ищем структуру.

**Зачем baseline?**
> Понять, даёт ли модель реальное улучшение над тривиальным решением.

**Stratified split?**
> Сохраняет долю классов в train/test — важно при дисбалансе.

**Feature scaling обязателен?**
> Для kNN, SVM, LR, NN — да. Для деревьев — нет.

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
