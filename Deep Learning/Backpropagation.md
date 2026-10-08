# Backpropagation

## Идея

Алгоритм backpropagation строится на основе правила цепочки для вычисления производных сложных функций. В контексте нейронных сетей, мы можем рассматривать функцию потерь $L$ как сложную функцию, зависящую от весов $w$ через несколько промежуточных переменных $z_1, z_2, ..., z_n$:

$$
L = L(z_1(z_2(...(z_n(w, z_{n+1}(...))))))
$$

Тогда правило цепочки позволяет нам выразить производную функции потерь по весу $w$ следующим образом:

$$
\frac{\partial L}{\partial w} = \frac{\partial L}{\partial z_1} \cdot \frac{\partial z_1}{\partial z_2} \cdot ... \cdot \frac{\partial z_{n-1}}{\partial z_n} \cdot \frac{\partial z_n}{\partial w}
$$

Таким образом, чтобы узнать градиент функции потерь по весу $w$ промежуточной переменной $z_n$, нам нужно знать значение входных переменных $z_{n+1}$ и производную функции потерь по выходной переменной $z_{n-1}$. Поэтому backpropagation работает в двух направлениях: сначала мы вычисляем значения всех промежуточных переменных в прямом направлении (forward pass), а затем, начиная с функции потерь, мы вычисляем градиенты в обратном направлении (backward pass).

## Пример:

Пусть у нас есть простая нейронная сеть:

2 входа $\rightarrow$ 2 скрытых нейрона $\rightarrow$ 1 выход

Исходные данные:
входы: $x_1 = 0.5$, $x_2 = 1.0$
истинный ответ: $y = 1.0$
learning rate: $\eta = 0.5$
функция потерь: $L = \frac{1}{2}(y - y_{pred})^2$

Веса и смещения:
нейрон $h_1$: $w_{11} = 0.1$, $w_{12} = 0.2$
нейрон $h_2$: $w_{21} = 0.3$, $w_{22} = 0.4$
выходной нейрон $y_{pred}$: $v_1 = 0.5$, $v_2 = 0.6$, $b_3 = 0.0$

Этап 1: Прямой проход (Forward Pass)

1. скрытый слой:

- $h_1$:

$$
  z_1 = w_{11} \cdot x_1 + w_{12} \cdot x_2 = 0.25
$$

$$
  h_1 = \sigma(z_1) \approx 0.562
$$

- $h_2$:

$$
  z_2 = w_{21} \cdot x_1 + w_{22} \cdot x_2 = 0.5
$$

$$
  h_2 = \sigma(z_2) \approx 0.634
$$

2. выходной слой:

$$
   z_y = v_1 \cdot h_1 + v_2 \cdot h_2 + b_3 = 0.5 \cdot 0.562 + 0.6 \cdot 0.634 + 0.0 \approx 0.661
$$

$$
   y_{pred} = \sigma(z_y) \approx 0.659
$$

3. ошибка сети:

$$
   L = \frac{1}{2}(y - y_{pred})^2 = \frac{1}{2}(1.0 - 0.659)^2 \approx 0.058
$$

Этап 2: Обратный проход (Backward Pass)

1. Ошибка на выходе:
   производная функции потерь:

$$
   \frac{\partial L}{\partial y_{pred}} = y_{pred} - y = 0.659 - 1.0 = -0.341
$$

   производная функции активации:

$$
   \frac{\partial y_{pred}}{\partial z_y} = y_{pred} \cdot (1 - y_{pred}) \approx 0.659 \cdot (1 - 0.659) \approx 0.225
$$

   таким образом, градиент по $z_y$:

$$
   \frac{\partial L}{\partial z_y} = \frac{\partial L}{\partial y_{pred}} \cdot \frac{\partial y_{pred}}{\partial z_y} \approx -0.341 \cdot 0.225 \approx -0.0767
$$

2. Градиенты весов выходного слоя:

- для $v_1$:

$$
  \frac{\partial L}{\partial v_1} = \frac{\partial L}{\partial z_y} \cdot \frac{\partial z_y}{\partial v_1} = -0.0767 \cdot h_1 \approx -0.0767 \cdot 0.562 \approx -0.0431
$$

- для $v_2$:

$$
  \frac{\partial L}{\partial v_2} = \frac{\partial L}{\partial z_y} \cdot \frac{\partial z_y}{\partial v_2} = -0.0767 \cdot h_2 \approx -0.0767 \cdot 0.634 \approx -0.0486
$$

- для $b_3$:

$$
  \frac{\partial L}{\partial b_3} = \frac{\partial L}{\partial z_y} \cdot \frac{\partial z_y}{\partial b_3} = -0.0767 \cdot 1 \approx -0.0767
$$

3. Ошибки на скрытом слое:
- для $h_1$:

$$
  \frac{\partial L}{\partial h_1} = \frac{\partial L}{\partial z_y} \cdot \frac{\partial z_y}{\partial h_1} = -0.0767 \cdot v_1 \approx -0.0767 \cdot 0.5 \approx -0.03835
$$

  производная функции активации:

$$
  \frac{\partial h_1}{\partial z_1} = h_1 \cdot (1 - h_1) \approx 0.562 \cdot (1 - 0.562) \approx 0.246
$$

  таким образом, градиент по $z_1$:

$$
    \frac{\partial L}{\partial z_1} = \frac{\partial L}{\partial h_1} \cdot \frac{\partial h_1}{\partial z_1} \approx -0.03835 \cdot 0.246 \approx -0.0094
$$

И так далее.

## Если спросят

- **Зачем:** chain rule: forward кэширует $z$, backward несёт $\partial L/\partial z$.
- **Когда ломается:** длинная цепь — затухание/взрыв; ошибка в производной активации.
- **Путают с:** градиентным спуском: backprop считает градиент, GD его применяет.
- **Фраза:** backward — chain rule с кэшем прямого прохода.

## Связанное

- [[Sigmoid]]
- [[Linear regression model]]
- [[Matrix differentiation]]
- [[RNN]]
- [[Gradient Descent]]
- [[Basic Loss]]
