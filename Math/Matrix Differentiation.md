# Matrix Differentiation

https://education.yandex.ru/handbook/ml/article/matrichnoe-differencirovanie

Пусть у нас есть функция $f(x) = x^T A x$, где $x \in \mathbb{R}^{n \times 1}$, $A \in \mathbb{R}^{n \times n}$.

Тогда градиент функции $f(x)$ по $x$ равен:

$$
\nabla_{x} f(x) = 2Ax
$$

Это можно доказать используя дифференцирование:

$$
df = (\nabla_{f})^T dx
$$

Для нашей функции воспользуемся формулой для дифференцирования произведения:

$$
df = d(x^T) \cdot A x + x^T \cdot d(A x)
$$

$$
df = (dx)^T \cdot A x + x^T A (dx)
$$

Так как оба слагаемых являются скалярами - можем их транспонировать:

$$
df = x^T A^T dx + x^T A dx = x^T (A + A^T) dx
$$

Получаем, что $(\nabla_{x} f(x))^T = x^T (A + A^T)$.

Отсюда получаем, что $\nabla_{x} f(x) = (A^T + A) x$.

[[Quadratic form]] говорит о том, что любая квадратичная форма может быть приведена к виду $f(x) = x^T A_{sym} x$. А значит, $\nabla_{x} f(x) = 2A_{sym} x$.

## Если спросят

- **Зачем:** градиент $x^T A x$ нужен, чтобы закрыть нормальные уравнения линейной регрессии.
- **Когда ломается:** если $A$ не симметризовать, легко написать $\nabla=Ax$ вместо $(A+A^T)x$.
- **Путают с:** производной по матрице $A$, а не по вектору $x$.
- **Фраза:** $\nabla_x x^T A x=(A+A^T)x$, для симметричной — $2Ax$.

## Связанное

- [[Quadratic form]]
- [[Linear regression model]]
- [[Regularization]]
- [[Backpropagation]]
- [[Gradient Descent]]
