# Gradient Descent

> Алгоритм минимизации loss путём итеративного обновления параметров в направлении **антиградиента**.

---

## Идея

$$\theta_{t+1} = \theta_t - \eta \cdot \nabla_\theta L(\theta_t)$$

- $\theta$ — параметры модели (веса)
- $\eta$ — **learning rate** (шаг)
- $\nabla_\theta L$ — градиент loss по параметрам

**Аналогия:** спуск с горы в тумане — идёшь в сторону наибольшего уклона вниз.

---

## Три варианта (спрашивают всегда!)

| Вариант | Как считает градиент | Плюсы | Минусы |
|---------|---------------------|-------|--------|
| **Batch GD** | По всему датасету | Стабильный градиент | Медленно, много памяти |
| **SGD** | По 1 примеру | Быстро, онлайн-обучение | Шумный, нестабильный |
| **Mini-batch** | По батчу (32–512) | Баланс скорости и стабильности | Нужно подобрать batch size |

**На практике:** mini-batch SGD — стандарт в DL.

---

## Learning Rate — главный гиперпараметр

| Слишком большой η | Слишком маленький η |
|-------------------|---------------------|
| Расходится, loss скачет | Сходится очень медленно |
| Может «перепрыгнуть» минимум | Застревает в local minima/saddle |

**Типичные значения:** 1e-3 … 1e-4 (Adam), 1e-2 … 1e-1 (SGD + momentum)

### Learning Rate Schedules
```python
# Step decay: уменьшить lr каждые N эпох
# Cosine annealing: плавное снижение
# Warmup: маленький lr в начале, потом рост (трансформеры)
```

---

## Momentum

$$v_t = \beta \cdot v_{t-1} + \nabla L$$
$$\theta_{t+1} = \theta_t - \eta \cdot v_t$$

- $\beta \approx 0.9$ — «инерция»
- Сглаживает колебания, ускоряет в стабильном направлении
- Аналог шара, катящегося с горы

---

## Adam (Adaptive Moment Estimation)

**Стандарт в 2024–2026 для нейросетей.**

Для каждого параметра:
1. $m_t = \beta_1 m_{t-1} + (1-\beta_1) g_t$ — momentum (1-й момент)
2. $v_t = \beta_2 v_{t-1} + (1-\beta_2) g_t^2$ — скользящее среднее $g^2$ (2-й момент)
3. Bias correction: $\hat{m}_t$, $\hat{v}_t$
4. $\theta_{t+1} = \theta_t - \eta \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon}$

**Дефолты:** $\beta_1=0.9$, $\beta_2=0.999$, $\eta=10^{-3}$

### AdamW
Adam + **decoupled weight decay** (L2 отдельно от адаптивного lr).
- Стандарт для трансформеров, LLM

---

## Сравнение оптимизаторов

| Оптимизатор | Когда |
|-------------|-------|
| SGD + Momentum | CV (ResNet), когда нужна лучшая генерализация |
| Adam / AdamW | NLP, трансформеры, быстрый старт |
| Adagrad | Разреженные данные (NLP embeddings), lr затухает |
| RMSprop | RNN (между SGD и Adam) |

---

## Линейная регрессия — пример руками

Модель: $\hat{y} = wx + b$

Loss (MSE, 1 пример): $L = \frac{1}{2}(y - \hat{y})^2$

Градиенты:
$$\frac{\partial L}{\partial w} = -(y - \hat{y}) \cdot x$$
$$\frac{\partial L}{\partial b} = -(y - \hat{y})$$

```python
# Псевдокод mini-batch GD
for epoch in range(epochs):
    for batch_x, batch_y in dataloader:
        y_pred = w * batch_x + b
        error = batch_y - y_pred
        
        grad_w = -2 * (error * batch_x).mean()
        grad_b = -2 * error.mean()
        
        w -= lr * grad_w
        b -= lr * grad_b
```

---

## Backpropagation (для нейросетей)

1. **Forward pass** — вычислить $\hat{y}$ и loss
2. **Backward pass** — chain rule от loss к каждому весу
3. **Update** — оптимизатор применяет градиенты

```
loss → ∂loss/∂w₃ → ∂loss/∂w₂ → ∂loss/∂w₁
       (chain rule, от выхода к входу)
```

**На собесе:** backprop = эффективное вычисление градиентов через chain rule, не отдельный алгоритм оптимизации.

---

## Проблемы и решения

| Проблема | Решение |
|----------|---------|
| Vanishing gradients (глубокие сети) | ReLU, residual connections, BatchNorm |
| Exploding gradients | Gradient clipping: `clip_grad_norm_(params, max_norm=1.0)` |
| Local minima | В высоких размерностях saddle points важнее; SGD + noise помогает |
| Plateaus | Momentum, Adam, lr schedule |

---

## Код (PyTorch)

```python
import torch
import torch.nn as nn
from torch.optim import AdamW

model = MyModel()
optimizer = AdamW(model.parameters(), lr=1e-3, weight_decay=0.01)
criterion = nn.CrossEntropyLoss()

for batch_x, batch_y in train_loader:
    optimizer.zero_grad()          # обнулить градиенты!
    
    logits = model(batch_x)        # forward
    loss = criterion(logits, batch_y)
    
    loss.backward()                # backward (градиенты)
    torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)  # опционально
    optimizer.step()               # обновить веса
```

---

## Вопросы на собесе

**Чем SGD отличается от Batch GD?**
> SGD — градиент по 1 примеру (или mini-batch). Быстрее, шумнее, лучше обобщает.

**Зачем `zero_grad()`?**
> PyTorch **накапливает** градиенты. Без обнуления — суммируются с прошлого шага.

**Что делает weight decay?**
> Штраф $\lambda\|\theta\|^2$ → веса не растут бесконтрольно → меньше overfitting.

**SGD vs Adam на собесе?**
> SGD + momentum: лучше финальная генерализация в CV, дольше настраивать. Adam: быстрая сходимость, стандарт для NLP/трансформеров.

## Если спросят

- **Зачем:**
- **Когда ломается:**
- **Путают с:**

## Связанное

-
